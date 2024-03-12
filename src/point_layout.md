# Understanding the `PointLayout` type

In this section you will learn all about the [`PointLayout`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointLayout.html) type in `pasture`. You will learn how it is used to represent the structure of the points in your point clouds, how it relates to individual point attributes, and how you can create your own `PointLayout` either from hand or from an existing `struct` definition. Additionally you will learn about all the built-in point attribute definitions that `pasture` provides, and how you can create your own point attributes. 

## Point attributes

In the [data model section](data_model.md) of this tutorial we learned that `pasture` models a point cloud as a collection of tuples of attributes. A point attribute is a uniquely identifiable piece of data, which `pasture` represents using two types: [`PointAttributeDefinition`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointAttributeDefinition.html) and [`PointAttributeMember`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointAttributeMember.html). Understanding how attributes combine to form a `PointLayout` will also answer the question why there are two distinct attribute types in `pasture`. 

When we talk about specific point attributes, we often refer to them by their *name*, which is why `pasture` uses strings as the unique identifiers for point attributes. To get a feel for the different types of point attributes that are commonly used, we can look at the [LAS file specification](https://www.asprs.org/wp-content/uploads/2019/07/LAS_1_4_r15.pdf), specifically under the section that defines the *point data record formats*. Looking at point data record format 0---the simplest format that LAS provides---we see the following list of point attributes:

|Item|Format|Size|Required|
|-|-|-|-|
|X|long|4 bytes|yes|
|Y|long|4 bytes|yes|
|Z|long|4 bytes|yes|
|Intensity|unsigned short|2 bytes|no|
|Return Number|3 bits (bits 0-2)|3 bits|yes|
|Number of Returns (Given Pulse)|3 bits (bits 3-5)|3 bits|yes|
|Scan Direction Flag|1 bit (bit 6)|1 bit|yes|
|Edge of Flight Line|1 bit (bit 7)|1 bit|yes|
|Classification|unsigned char|1 byte|yes|
|Scan Angle Rank (-90 to +90) – Left Side|signed char|1 byte|yes|
|User Data|unsigned char|1 byte|no|
|Point Source ID|unsigned short|2 bytes|yes|

We see the name of each attribute (the `Item` column), the data type used to represent the attribute (the `Format`) column, how many bits or bytes a single value of that attribute requires (the `Size` column), and whether or not it is required to include this value in the LAS point record (the `Required` column). We don't really care about the last column, but the first three columns are interesting. Not only do they explain what types of data a single point might contain, they also give information of the *memory representation* of the point data. `pasture` deals with efficient in-memory representations of point clouds, so the mapping of memory ranges to the actual point attributes is a central part of what `pasture` does. It does this by using precisely the information we saw in the table: 

Each point attribute in `pasture` has a unique name, a data type and a size! If you look at the [`PointAttributeDefinition`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointAttributeDefinition.html) type, you will see three corresponding accessors:
- [`fn name(&self) -> &str`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointAttributeDefinition.html#method.name) for accessing the name of the attribute
- [`fn datatype(&self) -> PointAttributeDataType`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointAttributeDefinition.html#method.datatype) for accessing the data type of the attribute
- [`fn size(&self) -> u64`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointAttributeDefinition.html#method.size) for accessing the size in bytes of the attribute

If we look into the [`layout::attributes` module](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/attributes/index.html) we see all the predefined point attributes that `pasture` provides:

|Name|Description|
|-|-|
|CLASSIFICATION|Attribute definition for a classification. Default datatype is U8|
|CLASSIFICATION_FLAGS|Attribute definition for the classification flags. Default datatype is U8|
|COLOR_RGB|Attribute definition for an RGB color. Default datatype is Vec3u16|
|EDGE_OF_FLIGHT_LINE|Attribute definition for an edge of flight line flag. Default datatype is Bool|
|GPS_TIME|Attribute definition for a GPS timestamp. Default datatype is F64|
|INTENSITY|Attribute definition for an intensity value. Default datatype is U16|
|NIR|Attribute definition for near-infrared records (NIR). Default datatype is U16|
|NORMAL|Attribute definition for a 3D point normal. Default datatype is Vec3f32|
|NUMBER_OF_RETURNS|Attribute definition for the number of returns. Default datatype is U8|
|POINT_ID|Attribute definition for a point ID. Default datatype is U64|
|POINT_SOURCE_ID|Attribute definition for a point source ID. Default datatype is U16|
|POSITION_3D|Attribute definition for a 3D position. Default datatype is Vec3f64|
|RETURN_NUMBER|Attribute definition for a return number. Default datatype is U8|
|RETURN_POINT_WAVEFORM_LOCATION|Attribute definition for the return point waveform location in the LAS format. Default datatype is F32|
|SCANNER_CHANNEL|Attribute definition for the scanner channel. Default datatype is U8|
|SCAN_ANGLE|Attribute definition for a scan angle with extended precision (like in LAS format 1.4). Default datatype is I16|
|SCAN_ANGLE_RANK|Attribute definition for a scan angle rank. Default datatype is I8|
|SCAN_DIRECTION_FLAG|Attribute definition for a scan direction flag. Default datatype is Bool|
|USER_DATA|Attribute definition for a user data field. Default datatype is U8|
|WAVEFORM_DATA_OFFSET|Attribute definition for the offset to the waveform data in the LAS format. Default datatype is U64|
|WAVEFORM_PACKET_SIZE|Attribute definition for the size of a waveform data packet in the LAS format. Default datatype is U32|
|WAVEFORM_PARAMETERS|Attribute definition for the waveform parameters in the LAS format. Default datatype is Vector3|
|WAVE_PACKET_DESCRIPTOR_INDEX|Attribute definition for the wave packet descriptor index in the LAS format. Default datatype is U8|

## Point attribute data types

The description of each built-in attribute includes a statement of the form "The default datatype is X". `pasture` makes an assumption what a good default datatype is for each attribute. Here are some examples:
- `POSITION_3D` -> `Vec3f64`
- `CLASSIFICATION` -> `U8`
- `COLOR_RGB` -> `Vec3u16`

These datatypes roughly correspond to primitive types that Rust supports, plus several vector types provided by `nalgebra`. `pasture` defines the [`PointAttributeDataType`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/enum.PointAttributeDataType.html) enum, which constrains the valid datatypes for point attributes. It includes all integer and floating-point types up to and including 64 bits, some three- and four-component vector types (themselves of integers and floating-point values), as well as two special types `ByteArray(u64)` and `Custom{ ... }`. Each of these datatypes has a well-known binary representation and corresponds to a specific Rust type, which is what allows conversions between untyped memory (`[u8]`) and strongly typed attribute data. This correspondence is established using the [`PrimitiveType`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/trait.PrimitiveType.html) trait, which is a trait that `pasture` implements for the Rust integer and floating-point primitive types and some of the `nalgebra` vector types. It allows code like this:

```rust
let datatype: PointAttributeDataType = f64::data_type();
```

For memory safety, `pasture` uses the [`bytemuck`](https://docs.rs/bytemuck/latest/bytemuck/) crate, which ensures that all memory transmutations from and to untyped memory are safe. This is done by requiring that all `pasture` primitive types have to implement [`bytemuck::Pod`](https://docs.rs/bytemuck/latest/bytemuck/trait.Pod.html). This has some interesting but also limiting side-effects, in particular it prevents `pasture` from supporting the Rust primitive type `bool`, which is not valid for any bit pattern (i.e. it is represented using 8 bits, but only the bit patterns `0` and `1` are valid). We will shortly see that this also has implications for which types of `struct`s you are allowed to use as point representations. 

## Custom attributes

Besides the built-in attribute definitions, you can easily create your own point attributes using [`PointAttributeDefinition::custom`](https://docs.rs/pasture-core/0.4.0/pasture_core/layout/struct.PointAttributeDefinition.html#method.custom):

```rust
let custom_attribute = PointAttributeDefinition::custom(Cow::Borrowed("Custom"), PointAttributeDataType::F32);
```

It is a `const fn`, so you can create compile-time constant point attributes, which is precisely what all the built-in attribute definitions are! `pasture-io` uses the same mechanism to add LAS-specific attribute definitions, for example the [`ATTRIBUTE_LOCAL_LAS_POSITION`](https://docs.rs/pasture-io/0.4.0/pasture_io/las/constant.ATTRIBUTE_LOCAL_LAS_POSITION.html) which corresponds to a point position in the local space of a LAS file, represented using 32-bit signed integers:

```rust
pub const ATTRIBUTE_LOCAL_LAS_POSITION: PointAttributeDefinition = PointAttributeDefinition::custom(
    Cow::Borrowed("LASLocalPosition"),
    PointAttributeDataType::Vec3i32,
);
```