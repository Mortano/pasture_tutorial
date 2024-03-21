# Buffer slices

The point buffers in `pasture` semantically behave like collections of points. Rust has built-in support for obtaining subsets of sequences of elements through slices, given that the elements are stored contiguously in memory. `pasture` can do a bit more and allow slicing even for non-contiguous memory regions, e.g. columnar buffers. It does so by providing *slice buffers*, which are lightweight wrappers around point buffers that handle the offset calculations for accessing subsets of points. In this section you will learn about slice buffers, how to use them, and some reasons for the design decisions of `pasture` regarding slicing.

## How to slice point buffers

Slicing buffers is as simple as calling [`slice`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/trait.SliceBuffer.html#tymethod.slice) on the buffer:

```rust
let buffer: VectorBuffer = ...;
let slice = buffer.slice(1..2);
// buffer supports by-reference access to points, so the slice also supports this:
for point_ref in sliced.view::<PointType>().iter() {
    println!("{:?}", point_ref);
}
```

A sliced buffer retains the memory layout of the buffer it was sliced from. In this example, the basic buffer is a `VectorBuffer` which has interleaved memory layout, causing the slice to also have interleaved memory layout. Most of the time, the sliced buffer behaves identical to the original buffer: You can obtain views from the slice, it will implement the same memory layout traits as the original buffer, and you can slice the slice again as often as you like. 

Slices are - by definition - borrows of their underlying point buffer, so they will never implement [`OwningBuffer<'a>`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/trait.OwningBuffer.html), meaning you can never resize a buffer through a slice. This is similar to how Rust built-in slices behave and should not be surprising. 

If the source buffer supports mutable access to its memory, you can also obtain a mutable slice:

```rust
let mut buffer: VectorBuffer = ...;
let slice = buffer.slice_mut(1..2);
for point_mut in sliced_mut.view_mut::<PointType>().iter_mut() {
    point_mut.intensity *= 2;
}
```

The usual borrowing rules apply, so you cannot obtain multiple mutable slices to the same buffer at the same time. 

```admonish
Slicing point buffers can be less powerful than the built-in Rust slices. Things like [`split_at_mut`](https://doc.rust-lang.org/std/primitive.slice.html#method.split_at_mut) are currently not supported in `pasture`.
```

## The `SliceBuffer` trait, and why we can't use the `Index` trait instead

The slicing syntax in `pasture` is more limited than the syntax you might be used to in Rust. In particular, `pasture` does not support the `[]` operator for slicing. This is a limitation in the design of the Rust standard library, or perhaps a mismatch between the intuition for what it means to call the `[]` operator to what it actually does. There is an interesting discussion [in the Rust internals forum](https://internals.rust-lang.org/t/the-return-type-of-the-index-trait-has-a-big-restriction/17451).

In a nutshell: `[]` is syntactic sugar for the [`Index`](https://doc.rust-lang.org/std/ops/trait.Index.html) trait, which requires the return value of the indexing operation to be a borrow. Since `pasture` slices are proxy objects, they have to be returned by value as they are constructed on-demand. This makes it impossible to use them with the `Index` trait, therefore preventing us from using the `[]` operator.

`pasture` instead provides its own indexing traits: [`SliceBuffer<'a>`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/trait.SliceBuffer.html) and [`SliceBufferMut<'a>`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/trait.SliceBufferMut.html). Here is what `SliceBuffer<'a>` looks like:

```rust
pub trait SliceBuffer<'a>
where
    Self: 'a,
{
    type SliceType: BorrowedBuffer<'a> + Sized;

    // Required method
    fn slice(&'a self, range: Range<usize>) -> Self::SliceType;
}
```

It provides the [`slice`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/trait.SliceBuffer.html#tymethod.slice) function, which returns an associated type [`SliceType`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/trait.SliceBuffer.html#associatedtype.SliceType). There are a bunch of different slice types that `pasture` provides for a multitude of reasons, but the most important reason for `SliceBuffer` to use an associated type for the slice object is to allow efficient slicing of slices themselves. 

Slices are thin wrappers around point buffers, so when slicing some point buffer `B`, you end up with a [`BufferSlice<B>`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/struct.BufferSlice.html) (or some variant thereof). If we now slice this slice, naively we would end up with a nested slice type: `BufferSlice<BufferSlice<B>>`. What we instead want is `BufferSlice<B>` with updated slice bounds. This is possible because `BufferSlice<B>` implements the `SliceBuffer` trait with a different `SliceType` than the regular point buffer types and therefore collapses the nested hierarchy at compile-time. 