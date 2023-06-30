---
jupytext:
  notebook_metadata_filter: all,-language_info,-toc,-latex_envs
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.6
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Visualizing the cloud-capped boundary layer

Data is from a large eddy simulation of trade cumulus clouds.  

* Single timestep at 12.5 m x, y, z grid spacing

* Variable is QN -- cloud liquid water (g/kg)

+++

## link to data file

The netcdf file:  bomex_qv_qn.nc -- 527 Mbytes

[https://drive.google.com/file/d/1E0IORCCpZUGaMClbv2m4YFfiM34kC1FY/view?usp=sharing](https://drive.google.com/file/d/1E0IORCCpZUGaMClbv2m4YFfiM34kC1FY/view?usp=sharing)

```{code-cell} ipython3
import xarray as xr
from pathlib import Path
from vapor import session, renderer, dataset, camera
```

## adjust path to point to file 

```{code-cell} ipython3
the_file = Path().resolve() / 'bomex_qv_qn.nc'
print(the_file)
the_file.exists()
```

## Inspect the xarray dataset

```{code-cell} ipython3
bomex_data = xr.open_dataset(the_file)
bomex_data
```

## open as a Vapor dataset

```{code-cell} ipython3
ses = session.Session()
data = ses.OpenDataset(dataset.CF, [str(the_file)])
ses.Load('session.vs3')
```

```{code-cell} ipython3
print("Time Coordinate Variable Name:", data.GetTimeCoordVarName())
print("Coordinate Variable Names:", data.GetCoordVarNames())

print("Dimensions:")
for dim in data.GetDimensionNames():
    print(f"  {dim}:", data.GetDimensionLength(dim, 0))

print("Data Variables:")
for var in data.GetDataVarNames():
    print(f"  {var}")
    print(f"    Time Varying:", bool(data.IsTimeVarying(var)))
    print(f"    Dimensionality:", data.GetVarGeometryDim(var))
    print(f"    Coordinates:", data.GetVarCoordVars(var, True))
    print("     Data Range:", data.GetDataRange(var))
```

## Show gridcells

* Red: watervapor
* Grey: cloud liquid water
  
Top view of cloud field, with condensation threshold set to 0.01 g/kg

```{code-cell} ipython3
ren = data.NewRenderer(renderer.VolumeIsoRenderer)
ren.SetVariableName(data.GetDataVarNames(3)[0]) # Set to first 3D data variable
ren.SetIsoValues([0.01])

ses.GetCamera().ViewAll()
ses.Show()
```

## Create visualizer widget

```{code-cell} ipython3
from jupyter_vapor_widget import *

viz = VaporVisualizerWidget(ses)
viz
```

## Add a slider bar

```{code-cell} ipython3
tf = ren.GetPrimaryTransferFunction()
dataRange = tf.GetMinMaxMapValue()

def sliderChanged(change):
    ren.SetIsoValues([change.new])
    viz.Render(fast=True)

slider = widgets.FloatSlider(value=ren.GetIsoValues()[0], min=dataRange[0], max=dataRange[1], step=(dataRange[1]-dataRange[0])/100)
slider.observe(sliderChanged, names='value')

widgets.VBox([
    viz,
    widgets.HBox([widgets.Label("Iso value:"), slider])
])
```
