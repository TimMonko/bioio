# BioIO

[![Build Status](https://github.com/bioio-devs/bioio/actions/workflows/ci.yml/badge.svg)](https://github.com/bioio-devs/bioio/actions)
[![Documentation](https://github.com/bioio-devs/bioio/actions/workflows/docs.yml/badge.svg)](https://bioio-devs.github.io/bioio)
[![PyPI version](https://badge.fury.io/py/bioio.svg)](https://badge.fury.io/py/bioio)
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)
[![Python 3.6](https://img.shields.io/badge/python-3.9,3.10,3.11-blue.svg)](https://www.python.org/downloads/release/python-360/)

Image Reading, Metadata Conversion, and Image Writing for Microscopy Images in Pure Python

---

## Migrating from AICSImageIO?

[See the migration document on the full documentation site](https://bioio-devs.github.io/bioio/MIGRATION.html)

## Features

- Image Reading
    - BioIO can read many various image formats (ex. `OME-TIFF`, `PNG`,  `ND2`), but only when paired with the appropriate plug-in for the image format. See [Reader Installation](#reader-installation) for a full list of the currently supported image formats as well as how to install them.
- Image Writing
    - Supports writing metadata and imaging data for:
        - `OME-TIFF`
        - `OME-ZARR`
        - `PNG`, `GIF`, [and other similar formats seen here](https://github.com/imageio/imageio)
- Various File Systems (HTTP URLs, s3, gcs, etc.)
    - Supports reading and writing to [fsspec](https://github.com/intake/filesystem_spec) supported file system wherever possible:
        -   Local paths (i.e. `my-file.png`)
        -   HTTP URLs (i.e. `https://my-domain.com/my-file.png`)
        -   [s3fs](https://github.com/dask/s3fs) (i.e. `s3://my-bucket/my-file.png`)
        -   [gcsfs](https://github.com/dask/gcsfs) (i.e. `gcs://my-bucket/my-file.png`)
    - See [Cloud IO Support](#cloud-io-support) for more details.

## Installation

**Stable Release:** `pip install bioio`<br>
**Development Head:** `pip install git+https://github.com/bioio-devs/bioio.git`

BioIO is supported on Windows, Mac, and Ubuntu.
For other platforms, you will likely need to build from source.

### Reader Installation

BioIO is a plug-in based system of readers meaning in addition to the `bioio` package you need to install the packages
that support the file types you are using. For example, if attempting to read `.ome.tiff` and `.zarr` files you'll want to install the `bioio-ome-tiff` & `bioio-ome-zarr` packages alongside `bioio` (ex. `pip install bioio bioio-ome-zarr bioio-ome-tiff`).
BioIO will then determine which reader to use for which file automatically.

This is a list of currently known and maintained reader plug-ins available:
| Package      | Supported file types |
| ------------ | -------------------- |
| [bioio-czi](https://github.com/bioio-devs/bioio-czi) | `CZI` |
| [bioio-dv](https://github.com/bioio-devs/bioio-dv)  | `DV` |
| [bioio-imageio](https://github.com/bioio-devs/bioio-imageio) | `PNG`, `GIF`, [& other similar formats seen here](https://github.com/imageio/imageio)        |
| [bioio-lif](https://github.com/bioio-devs/bioio-lif) | `LIF`        |
| [bioio-nd2](https://github.com/bioio-devs/bioio-nd2) | `ND2`        |
| [bioio-ome-tiff](https://github.com/bioio-devs/bioio-ome-tiff) | `OME-TIFF` (non-tiled)      |
| [bioio-ome-tiled-tiff](https://github.com/bioio-devs/bioio-ome-tiled-tiff) | `OME-TIFF` (tiled)       |
| [bioio-ome-zarr](https://github.com/bioio-devs/bioio-ome-zarr) | `ZARR`        |
| [bioio-sldy](https://github.com/bioio-devs/bioio-sldy) | `SLDY`        |
| [bioio-tifffile](https://github.com/bioio-devs/bioio-tifffile) | `TIFF` (non-globbed)       |
| [bioio-tiff-glob](https://github.com/bioio-devs/bioio-tiff-glob) | `TIFF` (globbed)       |
| [bioio-bioformats](https://github.com/bioio-devs/bioio-bioformats) | Files supported by [Bio-Formats](https://docs.openmicroscopy.org/bio-formats/latest/supported-formats.html) (Requires `java` and `maven`, see below for details)       |

## Documentation

For full package documentation please visit
[https://bioio-devs.github.io/bioio/](https://bioio-devs.github.io/bioio/).

## Quickstart

### Full Image Reading

If your image fits in memory:

```python
from bioio import BioImage

# Get an BioImage object
img = BioImage("my_file.tiff")  # selects the first scene found
img.data  # returns 5D TCZYX numpy array
img.xarray_data  # returns 5D TCZYX xarray data array backed by numpy
img.dims  # returns a Dimensions object
img.dims.order  # returns string "TCZYX"
img.dims.X  # returns size of X dimension
img.shape  # returns tuple of dimension sizes in TCZYX order
img.get_image_data("CZYX", T=0)  # returns 4D CZYX numpy array

# Get the id of the current operating scene
img.current_scene

# Get a list valid scene ids
img.scenes

# Change scene using name
img.set_scene("Image:1")
# Or by scene index
img.set_scene(1)

# Use the same operations on a different scene
# ...
```

#### Full Image Reading Notes

The `.data` and `.xarray_data` properties will load the whole scene into memory.
The `.get_image_data` function will load the whole scene into memory and then retrieve
the specified chunk.

### Delayed Image Reading

If your image doesn't fit in memory:

```python
from bioio import BioImage

# Get an BioImage object
img = BioImage("my_file.tiff")  # selects the first scene found
img.dask_data  # returns 5D TCZYX dask array
img.xarray_dask_data  # returns 5D TCZYX xarray data array backed by dask array
img.dims  # returns a Dimensions object
img.dims.order  # returns string "TCZYX"
img.dims.X  # returns size of X dimension
img.shape  # returns tuple of dimension sizes in TCZYX order

# Pull only a specific chunk in-memory
lazy_t0 = img.get_image_dask_data("CZYX", T=0)  # returns out-of-memory 4D dask array
t0 = lazy_t0.compute()  # returns in-memory 4D numpy array

# Get the id of the current operating scene
img.current_scene

# Get a list valid scene ids
img.scenes

# Change scene using name
img.set_scene("Image:1")
# Or by scene index
img.set_scene(1)

# Use the same operations on a different scene
# ...
```

#### Delayed Image Reading Notes

The `.dask_data` and `.xarray_dask_data` properties and the `.get_image_dask_data`
function will not load any piece of the imaging data into memory until you specifically
call `.compute` on the returned Dask array. In doing so, you will only then load the
selected chunk in-memory.

### Mosaic Image Reading

Read stitched data or single tiles as a dimension.

Known plug-in packages that support mosaic tile stitching:

-   `bioio-czi`
-   `bioio-lif`

#### BioImage

If the file format reader supports stitching mosaic tiles together, the
`BioImage` object will default to stitching the tiles back together.

```python
img = BioImage("very-large-mosaic.lif")
img.dims.order  # T, C, Z, big Y, big X, (S optional)
img.dask_data  # Dask chunks fall on tile boundaries, pull YX chunks out of the image
```

This behavior can be manually turned off:

```python
img = BioImage("very-large-mosaic.lif", reconstruct_mosaic=False)
img.dims.order  # M (tile index), T, C, Z, small Y, small X, (S optional)
img.dask_data  # Chunks use normal ZYX
```

If the reader does not support stitching tiles together the M tile index will be
available on the `BioImage` object:

```python
img = BioImage("some-unsupported-mosaic-stitching-format.ext")
img.dims.order  # M (tile index), T, C, Z, small Y, small X, (S optional)
img.dask_data  # Chunks use normal ZYX
```

#### Reader

If the file format reader detects mosaic tiles in the image, the `BioImage` object
will store the tiles as a dimension.

If tile stitching is implemented, the `BioImage` can also return the stitched image.

```python
reader = BioImage("ver-large-mosaic.lif")
reader.dims.order  # M, T, C, Z, tile size Y, tile size X, (S optional)
reader.dask_data  # normal operations, can use M dimension to select individual tiles
reader.mosaic_dask_data  # returns stitched mosaic - T, C, Z, big Y, big, X, (S optional)
```

#### Single Tile Absolute Positioning

There are functions available on the `BioImage` object
to help with single tile positioning:

```python
img = BioImage("very-large-mosaic.lif")
img.mosaic_tile_dims  # Returns a Dimensions object with just Y and X dim sizes
img.mosaic_tile_dims.Y  # 512 (for example)

# Get the tile start indices (top left corner of tile)
y_start_index, x_start_index = img.get_mosaic_tile_position(12)
```

### Metadata Reading

```python
from bioio import BioImage

# Get an BioImage object
img = BioImage("my_file.tiff")  # selects the first scene found
img.metadata  # returns the metadata object for this file format (XML, JSON, etc.)
img.channel_names  # returns a list of string channel names found in the metadata
img.physical_pixel_sizes.Z  # returns the Z dimension pixel size as found in the metadata
img.physical_pixel_sizes.Y  # returns the Y dimension pixel size as found in the metadata
img.physical_pixel_sizes.X  # returns the X dimension pixel size as found in the metadata
```

### Xarray Coordinate Plane Attachment

If `bioio` finds coordinate information for the spatial-temporal dimensions of
the image in metadata, you can use
[xarray](http://xarray.pydata.org/en/stable/index.html) for indexing by coordinates.

```python
from bioio import BioImage

# Get an BioImage object
img = BioImage("my_file.ome.tiff")

# Get the first ten seconds (not frames)
first_ten_seconds = img.xarray_data.loc[:10]  # returns an xarray.DataArray

# Get the first ten major units (usually micrometers, not indices) in Z
first_ten_mm_in_z = img.xarray_data.loc[:, :, :10]

# Get the first ten major units (usually micrometers, not indices) in Y
first_ten_mm_in_y = img.xarray_data.loc[:, :, :, :10]

# Get the first ten major units (usually micrometers, not indices) in X
first_ten_mm_in_x = img.xarray_data.loc[:, :, :, :, :10]
```

See `xarray`
["Indexing and Selecting Data" Documentation](http://xarray.pydata.org/en/stable/indexing.html)
for more information.

### Cloud IO Support

[File-System Specification (fsspec)](https://github.com/intake/filesystem_spec) allows
for common object storage services (S3, GCS, etc.) to act like normal filesystems by
following the same base specification across them all. AICSImageIO utilizes this
standard specification to make it possible to read directly from remote resources when
the specification is installed.

```python
from bioio import BioImage

# Get an BioImage object
img = BioImage("http://my-website.com/my_file.tiff")
img = BioImage("s3://my-bucket/my_file.tiff")
img = BioImage("gcs://my-bucket/my_file.tiff")

# Or read with specific filesystem creation arguments
img = BioImage("s3://my-bucket/my_file.tiff", fs_kwargs=dict(anon=True))
img = BioImage("gcs://my-bucket/my_file.tiff", fs_kwargs=dict(anon=True))

# All other normal operations work just fine
```

Remote reading requires that the file-system specification implementation for the
target backend is installed.

-   For `s3`: `pip install s3fs`
-   For `gs`: `pip install gcsfs`

See the [list of known implementations](https://filesystem-spec.readthedocs.io/en/latest/?badge=latest#implementations).

### Saving to OME-TIFF

The simpliest method to save your image as an OME-TIFF file with key pieces of
metadata is to use the `save` function.

```python
from bioio import BioImage

BioImage("my_file.czi").save("my_file.ome.tiff")
```

**Note:** By default `BioImage` will generate only a portion of metadata to pass
along from the reader to the OME model. This function currently does not do a full
metadata translation.

For finer grain customization of the metadata, scenes, or if you want to save an array
as an OME-TIFF, the writer class can also be used to customize as needed.

```python
import numpy as np
from bioio.writers import OmeTiffWriter

image = np.random.rand(10, 3, 1024, 2048)
OmeTiffWriter.save(image, "file.ome.tif", dim_order="ZCYX")
```

See
[OmeTiffWriter documentation](./aicsimageio.writers.html#aicsimageio.writers.ome_tiff_writer.OmeTiffWriter.save)
for more details.

#### Other Writers

In most cases, `BioImage.save` is usually a good default but there are other image
writers available. For more information, please refer to
[our writers documentation](https://bioio-devs.github.io/bioio/bioio.writers.html).

## Development

See our
[developer resources](https://bioio-devs.github.io/bioio/developer_resources.html)
for information related to developing the code.

## Citation

If you find `bioio` useful, please cite this repository as:

# TODO: should this remain the same??
> Eva Maxfield Brown, Dan Toloudis, Jamie Sherman, Madison Swain-Bowden, Talley Lambert, AICSImageIO Contributors (2021). AICSImageIO: Image Reading, Metadata Conversion, and Image Writing for Microscopy Images in Pure Python [Computer software]. GitHub. https://github.com/AllenCellModeling/aicsimageio

bibtex:

```bibtex
@misc{aicsimageio,
  author    = {Brown, Eva Maxfield and Toloudis, Dan and Sherman, Jamie and Swain-Bowden, Madison and Lambert, Talley and {AICSImageIO Contributors}},
  title     = {AICSImageIO: Image Reading, Metadata Conversion, and Image Writing for Microscopy Images in Pure Python},
  year      = {2021},
  publisher = {GitHub},
  url       = {https://github.com/AllenCellModeling/aicsimageio}
}
```

_Free software: BSD-3-Clause_

_(Each reader plug-in has its own license, including some that may be more restrictive than this package's BSD-3-Clause)_
