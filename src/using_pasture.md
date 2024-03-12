# Using `pasture` in your code

In this section we will look at how the `pasture` library is structured and how you can include it in your code. 

## The structure of `pasture`

`pasture` consists of several related libraries:
- [`pasture-core`](https://crates.io/crates/pasture-core) contains all core data structures: Point buffer types, the [`PointLayout`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointLayout.html) type, predefined attribute definitions, as well as common math code for vectors and bounding boxes
- [`pasture-io`](https://crates.io/crates/pasture-io) contains code for doing point cloud I/O, i.e. reading and writing files in common formats
- [`pasture-derive`](https://crates.io/crates/pasture-derive) contains the `#[derive(PointType)]` macro, which auto-generates a [`PointLayout`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointLayout.html) from a Rust `struct` 
- [`pasture-algorithms`](https://crates.io/crates/pasture-algorithms) contains algorithms for working with point clouds. It is currently in a very early stage of development and contains only a limited set of algorithms. If you want to contribute, [pull requests are welcome!](https://github.com/Mortano/pasture/pulls)

## Using `pasture` in your code

For most projects, you will typically include `pasture-core`, `pasture-io`, and `pasture-derive` by adding the following code to your `Cargo.toml`:
```toml
pasture-core = "0.4.0"
pasture-io = "0.4.0"
pasture-derive = "0.4.0"
```

If you have your own I/O code, you probably won't need `pasture-io`.

Looking at the example shown in the [Overview](overview.md) section, we can understand the basic include structure:
```rust,editable
use pasture_core::{
    containers::{BorrowedBuffer, VectorBuffer},
    layout::attributes::POSITION_3D,
    nalgebra::Vector3,
};
use pasture_io::base::{read_all};
```

The two main modules that you will include from in `pasture-core` are [`containers`](https://docs.rs/pasture-core/0.4.0/pasture_core/containers/index.html) and [`layout`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/index.html). `containers` contains all traits and implementations for the various point buffer types that `pasture` supports. They are explained in detail in the [point buffers section of this tutorial](point_buffers.md). `layout` contains the `PointLayout` type and all its associated types, such as the predefined attribute definitions, which are explained in detail in the [point layout section of this tutorial](point_layout.md).

Since point clouds are spatial data, they require a bit of linear algebra, for which `pasture` uses the [`nalgebra` crate](https://docs.rs/nalgebra/latest/nalgebra/). `pasture-core` re-exports `nalgebra` so that you can interface with its types while using `pasture`. 

For doing I/O, `pasture-io` contains several built-in types. The simplest one is the `read_all` function included in the example, but the [`base`](https://docs.rs/pasture-io/0.4.0/pasture_io/base/index.html) module of `pasture-io` also contains traits for types that read from a point cloud file or write to a point cloud file. Point cloud I/O is explained in depth in [another section of this tutorial](point_io.md).