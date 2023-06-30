---
jupytext:
  notebook_metadata_filter: all,-language_info,-toc,-latex_envs
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Reduce to one variable

```{code-cell} ipython3
import xarray as xr
from pathlib import Path
```

```{code-cell} ipython3
the_path = Path.home() / 'repos/VAPOR'
filename = "BO*nc"
bomex_file = list(the_path.glob(filename))[0]
bomex_file
```

```{code-cell} ipython3
bomex_data = xr.open_dataset(bomex_file)
bomex_data
```

```{code-cell} ipython3
out = bomex_data.variables
var_list = list(out.keys())
var_list
good_vars = ['x',
 'y',
 'z',
 'time',
 'p','QV',
 'QN']
```

```{code-cell} ipython3
good_vars = set(good_vars)
all_vars = set(var_list)
bad_vars =all_vars - good_vars
bomex_new = bomex_data.drop(bad_vars)
bomex_new['x'].attrs['axis'] = 'X'
bomex_new['y'].attrs['axis'] = 'Y'
bomex_new['z'].attrs['axis'] = 'Z'
bomex_new['time'].attrs['axis'] = 'T'
```

```{code-cell} ipython3
bomex_new.variables
```

## compress the variables

```{code-cell} ipython3
comp = dict(zlib=True, complevel=5)
for name, var in bomex_new.items(): 
    var.encoding.update(comp)
```

```{code-cell} ipython3
bomex_new['QN'].encoding
```

Write a variable out with zlib compression

```{code-cell} ipython3
outfile = the_path / "compressed_data.nc"
help(bomex_new.to_netcdf)
```
