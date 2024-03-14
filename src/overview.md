# `pasture`

`pasture` is a Rust library for working with LiDAR point cloud data. Its main purpose is to provide data structures for handling collections of points with a wide range of attributes, as well as the serialization to and deserialization from well-known point cloud formats, such as [LAS](https://en.wikipedia.org/wiki/LAS_file_format) or [3D Tiles](https://github.com/CesiumGS/3d-tiles). The data structures that `pasture` provides are meant to be as efficient as possible in terms of memory usage and access speed, while maintaining a level of flexibility and safety that C/C++ libraries such as [PDAL](https://pdal.io/en/2.6.3/) do not have. Memory and I/O are the main focus of `pasture`, so it works best as a building block for tools and applications that require efficient handling of LiDAR data. If you are looking for a library that provides processing and analysis capabilities, `pasture` is probably not what you are looking for (though [`pasture-algorithms`](https://crates.io/crates/pasture-algorithms) eventually aims at providing more processing capabilities). 

## About this guide

In this guide you will learn how to use `pasture` in your own code and why `pasture` is designed the way it is. It requires some knowledge of the Rust programming language, but no prior knowledge of LiDAR point clouds. To get a feel for how `pasture` code looks, here is a first example showing how to read point data from a `LAS` file:

```rust
use anyhow::{bail, Context, Result};
use pasture_core::{
    containers::{BorrowedBuffer, VectorBuffer},
    layout::attributes::POSITION_3D,
    nalgebra::Vector3,
};
use pasture_io::base::{read_all};

fn main() -> Result<()> {
    // Reading a point cloud file is as simple as calling `read_all`
    let points = read_all::<VectorBuffer, _>("pointcloud.las").context("Failed to read points")?;

    if points.point_layout().has_attribute(&POSITION_3D) {
        for position in points
            .view_attribute::<Vector3<f64>>(&POSITION_3D)
            .into_iter()
            .take(10)
        {
            println!("({};{};{})", position.x, position.y, position.z);
        }
    } else {
        bail!("Point cloud file has no positions!");
    }

    Ok(())
}
```