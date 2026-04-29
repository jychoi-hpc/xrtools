# xrtools

Small CLI utilities for inspecting and merging NetCDF/Zarr datasets with xarray.

## Contents

- `xrls`: print a quick summary of a NetCDF/Zarr/GRIB2 file and optionally dump variables.
- `xrmerge`: merge/concat multiple NetCDF/Zarr files into a single output (NetCDF or Zarr). Supports optional MPI parallel write mode.

## Requirements

- Python 3.8+
- `xarray`
- `dask`
- `netCDF4`
- `tqdm`
- `pandas`
- Optional for MPI parallel write: `mpi4py` and netCDF4 built with parallel HDF5 support

## Install

Create a virtual environment and install the dependencies:

```
python -m venv .venv
source .venv/bin/activate
pip install xarray dask netCDF4 tqdm pandas
```

Optional extras:

```
pip install mpi4py
```

## Usage

### Inspect a file

```
./xrls path/to/file.nc
./xrls path/to/file.zarr
```

Dump specific variables:

```
./xrls path/to/file.nc --dump var1 var2
```

Show variable encodings:

```
./xrls path/to/file.nc --verbose
```

Open a group:

```
./xrls path/to/file.nc -g group_name
```

Check for NaN values per variable:

```
./xrls path/to/file.nc --check-nan
```

### Merge/concat files

Basic merge to Zarr:

```
./xrmerge inputs/*.nc --outfile merged.zarr
```

Concat along a dimension (e.g., `time`):

```
./xrmerge inputs/*.nc --concat time --outfile merged.nc
```

Include or exclude variables:

```
./xrmerge inputs/*.nc --include var1,var2 --outfile merged.nc
./xrmerge inputs/*.nc --exclude var1,var2 --outfile merged.nc
```

Control chunking:

```
./xrmerge inputs/*.nc --chunk "{'time': 1}" --outfile merged.zarr
```

MPI parallel write (NetCDF only):

```
mpirun -n 4 ./xrmerge inputs/*.nc --parallel --outfile merged.nc
```

Check for NaN values per variable (prompts to continue; use `-y` to skip prompt):

```
./xrmerge inputs/*.nc --check-nan --outfile merged.nc
./xrmerge inputs/*.nc --check-nan -y --outfile merged.nc
```

Apply a custom preprocess function to each file before merging:

```
./xrmerge inputs/*.nc --preprocess ./my_preprocess.py --outfile merged.nc
```

The file must define a function named `preprocess(ds)` that accepts and returns an `xarray.Dataset`:

```python
# my_preprocess.py
def preprocess(ds):
    return ds.isel(time=slice(0, 10))
```

Overwrite an existing output file:

```
./xrmerge inputs/*.nc --force --outfile merged.nc
```

Dry run:

```
./xrmerge inputs/*.nc --dryrun
```

## Notes

- For best MPI performance, disable filters/compression on output.
- When using `--parallel`, the script writes by splitting the first dimension across ranks.

## License

MIT.
