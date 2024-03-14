# Understanding `pasture` point buffers in detail

In this section you will learn all the nitty-gritty details of the point buffer types in `pasture`, so that you can write the most efficient point cloud code that your application requires. The point buffer explanation comes in three levels of detail:
1. [An overview of the built-in point buffer types](point_buffers/builtin_types.md): A hands-on look at the built-in buffer types that `pasture` provides, and how you can use them in your code. This subsection is focused on usage patterns and thus describes *how* to use the point buffer types, ignoring *why* they work the way they do.
2. Explanations of the [buffer traits in `pasture`](point_buffers/buffer_traits.md), including an explanation of the memory layout and ownership model. Here you will learn about all the point buffer traits that `pasture` exposes, why they are designed the way they are, and how you can choose the right buffer type for your application.
3. A guide on [how to implement your own point buffer type](point_buffers/implementing_custom_buffers.md)

There is also a [special section on *buffer slices*](point_buffers/buffer_slices.md), which are similar to the Rust built-in slice type `[T]`.