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

    /mnt/code/.pyenv/versions/3.10.12/envs/coast-10/lib/python3.10/site-packages/utide/harmonics.py:16: RuntimeWarning: invalid value encountered in cast
    /mnt/code/.pyenv/versions/3.10.12/envs/coast-10/lib/python3.10/site-packages/utide/harmonics.py:17: RuntimeWarning: invalid value encountered in cast


### Open the zarr files as a XARRAY

The zarr files that we are using in this example do not have all the variables on the same file. Because of that, we need to open each file separately and then add the variables to a central file


```python
dom = xr.open_zarr(fn_nemo_dom_mask)
mesh_zgr = xr.open_zarr(fn_nemo_dom_mesh_zgr)
mesh_hgr = xr.open_zarr(fn_nemo_dom_mesh_hgr)
```

    /tmp/ipykernel_262236/396929176.py:1: RuntimeWarning: Failed to open Zarr store with consolidated metadata, but successfully read with non-consolidated metadata. This is typically much slower for opening a dataset. To silence this warning, consider:
    1. Consolidating metadata in this existing store with zarr.consolidate_metadata().
    2. Explicitly setting consolidated=False, to avoid trying to read consolidate metadata, or
    3. Explicitly setting consolidated=True, to raise an error in this case instead of falling back to try reading non-consolidated metadata.



```python
for var_name in mesh_zgr.data_vars:
    dom[var_name] = mesh_zgr[var_name]
for var_name in mesh_hgr.data_vars:
    dom[var_name] = mesh_hgr[var_name]
```


```python
u_grid = xr.open_zarr(fn_nemo_dat_u)
u_grid = u_grid.isel(time_counter=slice(0,119)).rename({'depthu': 'depth'})
v_grid = xr.open_zarr(fn_nemo_dat_v)
v_grid = v_grid.isel(time_counter=slice(0,119)).rename({'depthv': 'depth'})
t_grid = xr.open_zarr(fn_nemo_dat_t)
t_grid = t_grid.rename({'deptht': 'depth'})
```


```python
for var_name in u_grid.data_vars:
    t_grid[var_name] = u_grid[var_name]
for var_name in v_grid.data_vars:
    t_grid[var_name] = v_grid[var_name]
```

    /mnt/code/.pyenv/versions/3.10.12/envs/coast-10/lib/python3.10/site-packages/dask/array/core.py:4836: PerformanceWarning: Increasing number of chunks by factor of 15
    /mnt/code/.pyenv/versions/3.10.12/envs/coast-10/lib/python3.10/site-packages/dask/array/core.py:4836: PerformanceWarning: Increasing number of chunks by factor of 15



```python
#Uncomment to see the data
# dom
```


```python
#Uncomment to see the data
# t_grid
```

### Slice the zarr files

Because zarr files are optimized for cloud, when we instantiate an xarray dataset, we do not open the zarr by it self. We only open some metadata related to the file. The files will only be downloaed when we need to perform some processing on the data


```python
dom = dom.isel(y=slice(500, 700), x=slice(1000,1200))
```


```python
t_grid = t_grid.isel(y=slice(500, 700), x=slice(1000,1200), time_counter=slice(0,24))
```

### Loading and Interrogating

We can create a new Gridded object by simple calling `coast.Gridded()`. By passing this a NEMO data file and a NEMO domain file, COAsT will combine the two into a single xarray dataset within the Gridded object. Each individual Gridded object should be for a specified NEMO grid type, which is specified in a configuration file which is also passed as an argument. The Dask library is switched on by default, chunking can be specified in the configuration file.


```python
nemo_t = coast.Gridded(fn_data= t_grid, fn_domain = dom, config=fn_config_t_grid)
```

    /mnt/code/code/noc/coast/COAsT/coast/data/gridded.py:236: UserWarning: The model domain loaded, '<xarray.Dataset>
    Dimensions:       (t: 1, z: 75, y: 200, x: 200)
    Dimensions without coordinates: t, z, y, x
    Data variables: (12/34)
        e3t_0         (t, z, y, x) float64 dask.array<chunksize=(1, 5, 76, 82), meta=np.ndarray>
        e3t_1d        (t, z) float64 dask.array<chunksize=(1, 75), meta=np.ndarray>
        e3u_0         (t, z, y, x) float64 dask.array<chunksize=(1, 5, 76, 82), meta=np.ndarray>
        e3v_0         (t, z, y, x) float64 dask.array<chunksize=(1, 5, 76, 82), meta=np.ndarray>
        e3w_0         (t, z, y, x) float64 dask.array<chunksize=(1, 5, 76, 82), meta=np.ndarray>
        e3w_1d        (t, z) float64 dask.array<chunksize=(1, 75), meta=np.ndarray>
        ...            ...
        glamu         (t, y, x) float32 dask.array<chunksize=(1, 200, 82), meta=np.ndarray>
        glamv         (t, y, x) float32 dask.array<chunksize=(1, 200, 82), meta=np.ndarray>
        gphif         (t, y, x) float32 dask.array<chunksize=(1, 200, 82), meta=np.ndarray>
        gphit         (t, y, x) float32 dask.array<chunksize=(1, 200, 82), meta=np.ndarray>
        gphiu         (t, y, x) float32 dask.array<chunksize=(1, 200, 82), meta=np.ndarray>
        gphiv         (t, y, x) float32 dask.array<chunksize=(1, 200, 82), meta=np.ndarray>
    Attributes:
        DOMAIN_number_total:  8972
        DOMAIN_size_global:   [4322, 3059]', does not contain the bathy_metry' variable. This will result in the NEMO.dataset.bathymetry variable being set to zero, which may result in unexpected behaviour from routines that require this variable.


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




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body[data-theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-index-preview {
  grid-column: 2 / 5;
  color: var(--xr-font-color2);
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data,
.xr-index-data-in:checked ~ .xr-index-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-index-name div,
.xr-index-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2,
.xr-no-icon {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;ssh&#x27; (t_dim: 24, y_dim: 200, x_dim: 200)&gt;
array([[[-1.6210064 , -1.6217471 , -1.6231693 , ..., -1.5842544 ,
         -1.5886359 , -1.5930159 ],
        [-1.6208621 , -1.6217225 , -1.6233234 , ..., -1.5888053 ,
         -1.5935609 , -1.5981766 ],
        [-1.621304  , -1.6223003 , -1.6240481 , ..., -1.5930148 ,
         -1.5981003 , -1.6029042 ],
        ...,
        [-0.98735386, -0.9597808 , -0.931632  , ..., -0.6270069 ,
         -0.6278365 , -0.6290987 ],
        [-1.0102477 , -0.9808094 , -0.9505175 , ..., -0.63027424,
         -0.6316009 , -0.63331974],
        [-1.0307174 , -0.99985355, -0.96790063, ..., -0.63465285,
         -0.6367161 , -0.6389442 ]],

       [[-1.6141921 , -1.6149583 , -1.6157647 , ..., -1.5382599 ,
         -1.5428585 , -1.5479578 ],
        [-1.6137543 , -1.614481  , -1.6153185 , ..., -1.5421394 ,
         -1.5469443 , -1.5521828 ],
        [-1.6131831 , -1.6138711 , -1.6147525 , ..., -1.5456467 ,
         -1.5506693 , -1.5560651 ],
...
        [-0.5291973 , -0.53182423, -0.537423  , ..., -0.7970734 ,
         -0.8119881 , -0.82499063],
        [-0.53454167, -0.5373199 , -0.54308665, ..., -0.77014893,
         -0.7829233 , -0.7941438 ],
        [-0.53937477, -0.5424393 , -0.5484757 , ..., -0.7462123 ,
         -0.7570849 , -0.76668566]],

       [[-1.6791433 , -1.6803815 , -1.6808515 , ..., -1.5127497 ,
         -1.5175854 , -1.5226325 ],
        [-1.679447  , -1.6804127 , -1.6806444 , ..., -1.5093309 ,
         -1.5140619 , -1.5190879 ],
        [-1.6797307 , -1.6804144 , -1.6804197 , ..., -1.5054852 ,
         -1.5097626 , -1.5145139 ],
        ...,
        [-0.67837346, -0.6714207 , -0.6632468 , ..., -0.77248365,
         -0.79867   , -0.82295316],
        [-0.6826826 , -0.6778399 , -0.67128325, ..., -0.74587363,
         -0.7708025 , -0.794183  ],
        [-0.68843955, -0.68585885, -0.681205  , ..., -0.71919113,
         -0.7428356 , -0.7651531 ]]], dtype=float32)
Coordinates:
  * time       (t_dim) datetime64[ns] 1960-01-06T12:00:00 ... 1961-12-13T22:1...
    longitude  (y_dim, x_dim) float32 156.2 156.3 156.4 ... 172.7 172.8 172.8
    latitude   (y_dim, x_dim) float32 -63.49 -63.49 -63.49 ... -55.07 -55.07
Dimensions without coordinates: t_dim, y_dim, x_dim
Attributes:
    interval_operation:  300s
    interval_write:      1mo
    long_name:           sea_surface_height_above_geoid
    online_operation:    average
    units:               m</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'ssh'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>t_dim</span>: 24</li><li><span>y_dim</span>: 200</li><li><span>x_dim</span>: 200</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-7e8cb039-3a1d-4915-8626-4ff2c521ec6d' class='xr-array-in' type='checkbox' checked><label for='section-7e8cb039-3a1d-4915-8626-4ff2c521ec6d' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>-1.621 -1.622 -1.623 -1.625 -1.628 ... -0.6946 -0.7192 -0.7428 -0.7652</span></div><div class='xr-array-data'><pre>array([[[-1.6210064 , -1.6217471 , -1.6231693 , ..., -1.5842544 ,
         -1.5886359 , -1.5930159 ],
        [-1.6208621 , -1.6217225 , -1.6233234 , ..., -1.5888053 ,
         -1.5935609 , -1.5981766 ],
        [-1.621304  , -1.6223003 , -1.6240481 , ..., -1.5930148 ,
         -1.5981003 , -1.6029042 ],
        ...,
        [-0.98735386, -0.9597808 , -0.931632  , ..., -0.6270069 ,
         -0.6278365 , -0.6290987 ],
        [-1.0102477 , -0.9808094 , -0.9505175 , ..., -0.63027424,
         -0.6316009 , -0.63331974],
        [-1.0307174 , -0.99985355, -0.96790063, ..., -0.63465285,
         -0.6367161 , -0.6389442 ]],

       [[-1.6141921 , -1.6149583 , -1.6157647 , ..., -1.5382599 ,
         -1.5428585 , -1.5479578 ],
        [-1.6137543 , -1.614481  , -1.6153185 , ..., -1.5421394 ,
         -1.5469443 , -1.5521828 ],
        [-1.6131831 , -1.6138711 , -1.6147525 , ..., -1.5456467 ,
         -1.5506693 , -1.5560651 ],
...
        [-0.5291973 , -0.53182423, -0.537423  , ..., -0.7970734 ,
         -0.8119881 , -0.82499063],
        [-0.53454167, -0.5373199 , -0.54308665, ..., -0.77014893,
         -0.7829233 , -0.7941438 ],
        [-0.53937477, -0.5424393 , -0.5484757 , ..., -0.7462123 ,
         -0.7570849 , -0.76668566]],

       [[-1.6791433 , -1.6803815 , -1.6808515 , ..., -1.5127497 ,
         -1.5175854 , -1.5226325 ],
        [-1.679447  , -1.6804127 , -1.6806444 , ..., -1.5093309 ,
         -1.5140619 , -1.5190879 ],
        [-1.6797307 , -1.6804144 , -1.6804197 , ..., -1.5054852 ,
         -1.5097626 , -1.5145139 ],
        ...,
        [-0.67837346, -0.6714207 , -0.6632468 , ..., -0.77248365,
         -0.79867   , -0.82295316],
        [-0.6826826 , -0.6778399 , -0.67128325, ..., -0.74587363,
         -0.7708025 , -0.794183  ],
        [-0.68843955, -0.68585885, -0.681205  , ..., -0.71919113,
         -0.7428356 , -0.7651531 ]]], dtype=float32)</pre></div></div></li><li class='xr-section-item'><input id='section-4037eb1d-ee6f-4b12-87b3-b9b273073f5a' class='xr-section-summary-in' type='checkbox'  checked><label for='section-4037eb1d-ee6f-4b12-87b3-b9b273073f5a' class='xr-section-summary' >Coordinates: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(t_dim)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>1960-01-06T12:00:00 ... 1961-12-...</div><input id='attrs-678b713e-7fe5-4718-aa49-7b118d864235' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-678b713e-7fe5-4718-aa49-7b118d864235' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-6ed39e63-52f1-4b98-bc73-114ffca03cf9' class='xr-var-data-in' type='checkbox'><label for='data-6ed39e63-52f1-4b98-bc73-114ffca03cf9' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>T</dd><dt><span>long_name :</span></dt><dd>Time axis</dd><dt><span>standard_name :</span></dt><dd>time</dd><dt><span>time_origin :</span></dt><dd>1950-01-01 00:00:00</dd><dt><span>title :</span></dt><dd>Time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;1960-01-06T12:00:00.000000000&#x27;, &#x27;1960-02-06T12:00:00.000000000&#x27;,
       &#x27;1960-03-07T12:00:00.000000000&#x27;, &#x27;1960-04-06T12:00:00.000000000&#x27;,
       &#x27;1960-05-07T00:00:00.000000000&#x27;, &#x27;1960-06-06T12:00:00.000000000&#x27;,
       &#x27;1960-07-07T00:00:00.000000000&#x27;, &#x27;1960-08-06T12:00:00.000000000&#x27;,
       &#x27;1960-09-06T12:00:00.000000000&#x27;, &#x27;1960-10-07T00:00:00.000000000&#x27;,
       &#x27;1960-11-06T12:00:00.000000000&#x27;, &#x27;1960-12-07T00:00:00.000000000&#x27;,
       &#x27;1961-01-16T12:00:00.000000000&#x27;, &#x27;1961-02-15T00:00:00.000000000&#x27;,
       &#x27;1961-03-16T12:00:00.000000000&#x27;, &#x27;1961-04-16T00:00:00.000000000&#x27;,
       &#x27;1961-05-16T12:00:00.000000000&#x27;, &#x27;1961-06-16T00:00:00.000000000&#x27;,
       &#x27;1961-07-16T12:00:00.000000000&#x27;, &#x27;1961-08-16T12:00:00.000000000&#x27;,
       &#x27;1961-09-16T00:00:00.000000000&#x27;, &#x27;1961-10-16T12:00:00.000000000&#x27;,
       &#x27;1961-11-16T00:00:00.000000000&#x27;, &#x27;1961-12-13T22:17:04.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>longitude</span></div><div class='xr-var-dims'>(y_dim, x_dim)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>156.2 156.3 156.4 ... 172.8 172.8</div><input id='attrs-95274d7f-edc4-411e-a3e9-10bb1f4e3c37' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-95274d7f-edc4-411e-a3e9-10bb1f4e3c37' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b864d72e-e088-425e-be02-e0066c7e8ef5' class='xr-var-data-in' type='checkbox'><label for='data-b864d72e-e088-425e-be02-e0066c7e8ef5' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[156.25   , 156.33333, 156.41667, ..., 172.66667, 172.75   ,
        172.83333],
       [156.25   , 156.33333, 156.41667, ..., 172.66667, 172.75   ,
        172.83333],
       [156.25   , 156.33333, 156.41667, ..., 172.66667, 172.75   ,
        172.83333],
       ...,
       [156.25   , 156.33333, 156.41667, ..., 172.66667, 172.75   ,
        172.83333],
       [156.25   , 156.33333, 156.41667, ..., 172.66667, 172.75   ,
        172.83333],
       [156.25   , 156.33333, 156.41667, ..., 172.66667, 172.75   ,
        172.83333]], dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>latitude</span></div><div class='xr-var-dims'>(y_dim, x_dim)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>-63.49 -63.49 ... -55.07 -55.07</div><input id='attrs-11a12a66-0c12-4487-a797-479198b3882b' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-11a12a66-0c12-4487-a797-479198b3882b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c39589e7-749e-4012-9da6-6835bb4c87ef' class='xr-var-data-in' type='checkbox'><label for='data-c39589e7-749e-4012-9da6-6835bb4c87ef' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[-63.48817 , -63.48817 , -63.48817 , ..., -63.48817 , -63.48817 ,
        -63.48817 ],
       [-63.450947, -63.450947, -63.450947, ..., -63.450947, -63.450947,
        -63.450947],
       [-63.413673, -63.413673, -63.413673, ..., -63.413673, -63.413673,
        -63.413673],
       ...,
       [-55.162506, -55.162506, -55.162506, ..., -55.162506, -55.162506,
        -55.162506],
       [-55.114876, -55.114876, -55.114876, ..., -55.114876, -55.114876,
        -55.114876],
       [-55.067184, -55.067184, -55.067184, ..., -55.067184, -55.067184,
        -55.067184]], dtype=float32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-8d179e7b-e211-477b-8d70-709cd326b5de' class='xr-section-summary-in' type='checkbox'  ><label for='section-8d179e7b-e211-477b-8d70-709cd326b5de' class='xr-section-summary' >Indexes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-index-name'><div>time</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-9e5860d1-6088-475f-91c8-e6549d0fe7cd' class='xr-index-data-in' type='checkbox'/><label for='index-9e5860d1-6088-475f-91c8-e6549d0fe7cd' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(DatetimeIndex([&#x27;1960-01-06 12:00:00&#x27;, &#x27;1960-02-06 12:00:00&#x27;,
               &#x27;1960-03-07 12:00:00&#x27;, &#x27;1960-04-06 12:00:00&#x27;,
               &#x27;1960-05-07 00:00:00&#x27;, &#x27;1960-06-06 12:00:00&#x27;,
               &#x27;1960-07-07 00:00:00&#x27;, &#x27;1960-08-06 12:00:00&#x27;,
               &#x27;1960-09-06 12:00:00&#x27;, &#x27;1960-10-07 00:00:00&#x27;,
               &#x27;1960-11-06 12:00:00&#x27;, &#x27;1960-12-07 00:00:00&#x27;,
               &#x27;1961-01-16 12:00:00&#x27;, &#x27;1961-02-15 00:00:00&#x27;,
               &#x27;1961-03-16 12:00:00&#x27;, &#x27;1961-04-16 00:00:00&#x27;,
               &#x27;1961-05-16 12:00:00&#x27;, &#x27;1961-06-16 00:00:00&#x27;,
               &#x27;1961-07-16 12:00:00&#x27;, &#x27;1961-08-16 12:00:00&#x27;,
               &#x27;1961-09-16 00:00:00&#x27;, &#x27;1961-10-16 12:00:00&#x27;,
               &#x27;1961-11-16 00:00:00&#x27;, &#x27;1961-12-13 22:17:04&#x27;],
              dtype=&#x27;datetime64[ns]&#x27;, name=&#x27;time&#x27;, freq=None))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-2d81ef3a-a177-4c72-9511-ef84df1c6e07' class='xr-section-summary-in' type='checkbox'  checked><label for='section-2d81ef3a-a177-4c72-9511-ef84df1c6e07' class='xr-section-summary' >Attributes: <span>(5)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_surface_height_above_geoid</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m</dd></dl></div></li></ul></div></div>



Or as a numpy array:


```python
ssh_np = ssh.values
#ssh_np.shape # uncomment to print data object summary
```

Then lets plot up a single time snapshot of ssh using matplotlib:


```python
plt.pcolormesh(nemo_t.dataset.longitude, nemo_t.dataset.latitude, nemo_t.dataset.ssh[0])
```




    <matplotlib.collections.QuadMesh at 0x7fa6e3d77a60>




    
![png](/COAsT/zarr_files/zarr_files_33_1.png)
    



```python

```
