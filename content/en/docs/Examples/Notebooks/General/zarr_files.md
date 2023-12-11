---
    title: "Zarr files"
    linkTitle: "Zarr files"
    weight: 5

    description: >
        Zarr files example.
---
# Load Zarr files

This tutorial will show some examples on how to load zarr files on coast.
It will include:
- Creation of a Gridded object
- Loading data into the Gridded object.
- Combining Gridded output and Gridded domain data.
- Interrogating the Gridded object.
- Basic manipulation and subsetting
- Looking at the data with matplotlib

### Requirements

Coast also has the capability to allow you to open zarr files In order to do that, you need to install first the library zarr:

`pip install zarr xarray[complete] aiohttp requests`

After that, you can open the datasets

### Import

Begin by importing COAsT and define some file paths for NEMO output data and a NEMO domain, as an example of model data suitable for the Gridded object.


```python
import coast
import matplotlib.pyplot as plt
import datetime
import numpy as np
import xarray as xr
import zarr

root = "./"
fn_config_t_grid = root + "./config/example_nemo_monthly_climate.json"

# Define some file paths
fn_nemo_dom_mask = "https://noc-msm-o.s3-ext.jc.rl.ac.uk/n06-coast-testing/mask.zarr"
fn_nemo_dom_mesh_zgr = "https://noc-msm-o.s3-ext.jc.rl.ac.uk/n06-coast-testing/mesh_zgr.zarr"
fn_nemo_dom_mesh_hgr = "https://noc-msm-o.s3-ext.jc.rl.ac.uk/n06-coast-testing/mesh_hgr.zarr"
fn_nemo_dat_t = "https://noc-msm-o.s3-ext.jc.rl.ac.uk/n06-coast-testing/n06_T.zarr"
fn_nemo_dat_u = "https://noc-msm-o.s3-ext.jc.rl.ac.uk/n06-coast-testing/n06_U.zarr"
fn_nemo_dat_v = "https://noc-msm-o.s3-ext.jc.rl.ac.uk/n06-coast-testing/n06_V.zarr"

```

    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pydap/lib.py:5: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2871: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('pydap')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2871: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('pydap.responses')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2350: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('pydap')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2871: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('pydap.handlers')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2350: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('pydap')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2871: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('pydap.tests')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2350: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('pydap')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/pkg_resources/__init__.py:2871: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('sphinxcontrib')`.
    Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages



    ---------------------------------------------------------------------------

    ModuleNotFoundError                       Traceback (most recent call last)

    Cell In[1], line 6
          4 import numpy as np
          5 import xarray as xr
    ----> 6 import zarr
          8 root = "./"
          9 fn_config_t_grid = root + "./config/example_nemo_monthly_climate.json"


    ModuleNotFoundError: No module named 'zarr'


### Open the zarr files as a XARRAY

The zarr files that we are using in this example do not have all the variables on the same file. Because of that, we need to open each file separately and then add the variables to a central file


```python
dom = xr.open_zarr(fn_nemo_dom_mask)
mesh_zgr = xr.open_zarr(fn_nemo_dom_mesh_zgr)
mesh_hgr = xr.open_zarr(fn_nemo_dom_mesh_hgr)
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[2], line 1
    ----> 1 dom = xr.open_zarr(fn_nemo_dom_mask)
          2 mesh_zgr = xr.open_zarr(fn_nemo_dom_mesh_zgr)
          3 mesh_hgr = xr.open_zarr(fn_nemo_dom_mesh_hgr)


    NameError: name 'fn_nemo_dom_mask' is not defined



```python
for var_name in mesh_zgr.data_vars:
    dom[var_name] = mesh_zgr[var_name]
for var_name in mesh_hgr.data_vars:
    dom[var_name] = mesh_hgr[var_name]
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[3], line 1
    ----> 1 for var_name in mesh_zgr.data_vars:
          2     dom[var_name] = mesh_zgr[var_name]
          3 for var_name in mesh_hgr.data_vars:


    NameError: name 'mesh_zgr' is not defined



```python
u_grid = xr.open_zarr(fn_nemo_dat_u)
u_grid = u_grid.isel(time_counter=slice(0,119)).rename({'depthu': 'depth'})
v_grid = xr.open_zarr(fn_nemo_dat_v)
v_grid = v_grid.isel(time_counter=slice(0,119)).rename({'depthv': 'depth'})
t_grid = xr.open_zarr(fn_nemo_dat_t)
t_grid = t_grid.rename({'deptht': 'depth'})
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[4], line 1
    ----> 1 u_grid = xr.open_zarr(fn_nemo_dat_u)
          2 u_grid = u_grid.isel(time_counter=slice(0,119)).rename({'depthu': 'depth'})
          3 v_grid = xr.open_zarr(fn_nemo_dat_v)


    NameError: name 'fn_nemo_dat_u' is not defined



```python
for var_name in u_grid.data_vars:
    t_grid[var_name] = u_grid[var_name]
for var_name in v_grid.data_vars:
    t_grid[var_name] = v_grid[var_name]
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[5], line 1
    ----> 1 for var_name in u_grid.data_vars:
          2     t_grid[var_name] = u_grid[var_name]
          3 for var_name in v_grid.data_vars:


    NameError: name 'u_grid' is not defined



```python
dom
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[6], line 1
    ----> 1 dom


    NameError: name 'dom' is not defined



```python
t_grid
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[7], line 1
    ----> 1 t_grid


    NameError: name 't_grid' is not defined


### Slice the zarr files

Because zarr files are optimized for cloud, when we instantiate an xarray dataset, we do not open the zarr by it self. We only open some metadata related to the file. The files will only be downloaed when we need to perform some processing on the data


```python
dom = dom.isel(y=slice(500, 700), x=slice(1000,1200))
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[8], line 1
    ----> 1 dom = dom.isel(y=slice(500, 700), x=slice(1000,1200))


    NameError: name 'dom' is not defined



```python
t_grid = t_grid.isel(y=slice(500, 700), x=slice(1000,1200), time_counter=slice(0,24))
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[9], line 1
    ----> 1 t_grid = t_grid.isel(y=slice(500, 700), x=slice(1000,1200), time_counter=slice(0,24))


    NameError: name 't_grid' is not defined


### Loading and Interrogating

We can create a new Gridded object by simple calling `coast.Gridded()`. By passing this a NEMO data file and a NEMO domain file, COAsT will combine the two into a single xarray dataset within the Gridded object. Each individual Gridded object should be for a specified NEMO grid type, which is specified in a configuration file which is also passed as an argument. The Dask library is switched on by default, chunking can be specified in the configuration file.


```python
nemo_t = coast.Gridded(fn_data= t_grid, fn_domain = dom, config=fn_config_t_grid)
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[10], line 1
    ----> 1 nemo_t = coast.Gridded(fn_data= t_grid, fn_domain = dom, config=fn_config_t_grid)


    NameError: name 't_grid' is not defined


`!IMPORTANT!`: this dataset does not contain bathymetry data.

Our new Gridded object `nemo_t` contains a variable called dataset, which holds information on the two files we passed. Let’s have a look at this:


```python
# nemo_t.dataset # uncomment to print data object summary
```

This is an xarray dataset, which has all the information on netCDF style structures. You can see dimensions, coordinates and data variables. At the moment, none of the actual data is loaded to memory and will remain that way until it needs to be accessed.

As it is a zarr file, it will only get any data if you apply `compute()` on the data:


```python
ssh = nemo_t.dataset.ssh
ssh.compute()
# ssh # uncomment to print data object summary
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[12], line 1
    ----> 1 ssh = nemo_t.dataset.ssh
          2 ssh.compute()
          3 # ssh # uncomment to print data object summary


    NameError: name 'nemo_t' is not defined


Or as a numpy array:


```python
ssh_np = ssh.values
#ssh_np.shape # uncomment to print data object summary
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[13], line 1
    ----> 1 ssh_np = ssh.values
          2 #ssh_np.shape # uncomment to print data object summary


    NameError: name 'ssh' is not defined


Then lets plot up a single time snapshot of ssh using matplotlib:


```python
plt.pcolormesh(nemo_t.dataset.longitude, nemo_t.dataset.latitude, nemo_t.dataset.ssh[0])
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[14], line 1
    ----> 1 plt.pcolormesh(nemo_t.dataset.longitude, nemo_t.dataset.latitude, nemo_t.dataset.ssh[0])


    NameError: name 'nemo_t' is not defined



```python

```
