# Reading LAS files

This section covers reading LAS files using `pasture`. 

## Reading in the default LAS point record layout

```rust,editable,noplayground
use anyhow::{bail, Context, Result};
use pasture_core::{
    containers::{BorrowedBuffer, BorrowedBufferExt, HashMapBuffer, VectorBuffer},
    layout::{attributes::POSITION_3D, PointLayout},
    nalgebra::Vector3,
};
use pasture_io::{
    base::PointReader,
    las::{LASReader, ATTRIBUTE_LOCAL_LAS_POSITION},
};

fn main() -> Result<()> {
    let args = std::env::args().collect::<Vec<_>>();
    if args.len() != 2 {
        bail!("Usage: las_io <INPUT_FILE>");
    }
    let input_file = args[1].as_str();

    let mut las_reader =
        LASReader::from_path(input_file, false).context("Could not open LAS file")?;
    let points = las_reader
        .read::<VectorBuffer>(10)
        .context("Error while reading points")?;

    for position in points
        .view_attribute::<Vector3<f64>>(&POSITION_3D)
        .into_iter()
        .take(10)
    {
        eprintln!("{}", position);
    }

    Ok(())
}
```

## Reading points with a custom `PointLayout`

```rust,editable,noplayground
// Imports redacted for brevity

fn main() -> Result<()> {
    let args = std::env::args().collect::<Vec<_>>();
    if args.len() != 2 {
        bail!("Usage: las_io <INPUT_FILE>");
    }
    let input_file = args[1].as_str();

    let mut las_reader = LASReader::from_path(input_file, false).context("Could not open LAS file")?;

    let layout = PointLayout::from_attributes(&[POSITION_3D]);
    let mut points = HashMapBuffer::with_capacity(10, layout);
    las_reader
        .read_into(&mut points, 10)
        .context("Can't read points in custom layout")?;

    for position in points.view_attribute::<Vector3<f64>>(&POSITION_3D).iter() {
        eprintln!("{}", position);
    }

    Ok(())
}
```

## Reading points with a memory layout that exactly matches the binary layout of LAS point records

```rust,editable,noplayground
fn main() -> Result<()> {
    let args = std::env::args().collect::<Vec<_>>();
    if args.len() != 2 {
        bail!("Usage: las_io <INPUT_FILE>");
    }
    let input_file = args[1].as_str();
    
    let mut native_las_reader =
        LASReader::from_path(input_file, true).context("Could not open LAS file")?;

    let native_points = native_las_reader
        .read::<VectorBuffer>(10)
        .context("Failed to read points")?;
    eprintln!(
        "PointLayout obtained from a native LAS reader: {}",
        native_points.point_layout()
    );

    for local_position in native_points
        .view_attribute::<Vector3<i32>>(&ATTRIBUTE_LOCAL_LAS_POSITION)
        .into_iter()
    {
        eprintln!("{local_position}");
    }

    Ok(())
}
```