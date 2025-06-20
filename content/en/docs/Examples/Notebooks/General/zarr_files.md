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

### Open the zarr files as a XARRAY

The zarr files that we are using in this example do not have all the variables on the same file. Because of that, we need to open each file separately and then add the variables to a central file


```python
dom = xr.open_zarr(fn_nemo_dom_mask)
mesh_zgr = xr.open_zarr(fn_nemo_dom_mesh_zgr)
mesh_hgr = xr.open_zarr(fn_nemo_dom_mesh_hgr)
```


```python
for var_name in mesh_zgr.data_vars:
    dom[var_name] = mesh_zgr[var_name]
for var_name in mesh_hgr.data_vars:
    dom[var_name] = mesh_hgr[var_name]
```


```python
u_grid = xr.open_zarr(fn_nemo_dat_u)
u_grid = u_grid.isel(time_counter=slice(0, 119)).rename({"depthu": "depth"})
v_grid = xr.open_zarr(fn_nemo_dat_v)
v_grid = v_grid.isel(time_counter=slice(0, 119)).rename({"depthv": "depth"})
t_grid = xr.open_zarr(fn_nemo_dat_t)
t_grid = t_grid.rename({"deptht": "depth"})
```


```python
for var_name in u_grid.data_vars:
    t_grid[var_name] = u_grid[var_name]
for var_name in v_grid.data_vars:
    t_grid[var_name] = v_grid[var_name]
```

    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/dask/array/core.py:4849: PerformanceWarning: Increasing number of chunks by factor of 15


    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/dask/array/core.py:4849: PerformanceWarning: Increasing number of chunks by factor of 15



```python
dom
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
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
Dimensions:       (t: 1, z: 75, y: 3059, x: 4322)
Dimensions without coordinates: t, z, y, x
Data variables: (12/42)
    fmask         (t, z, y, x) int8 dask.array&lt;chunksize=(1, 10, 383, 541), meta=np.ndarray&gt;
    fmaskutil     (t, y, x) int8 dask.array&lt;chunksize=(1, 765, 1081), meta=np.ndarray&gt;
    nav_lat       (y, x) float32 dask.array&lt;chunksize=(383, 541), meta=np.ndarray&gt;
    nav_lev       (z) float32 dask.array&lt;chunksize=(75,), meta=np.ndarray&gt;
    nav_lon       (y, x) float32 dask.array&lt;chunksize=(383, 541), meta=np.ndarray&gt;
    time_counter  (t) float64 dask.array&lt;chunksize=(1,), meta=np.ndarray&gt;
    ...            ...
    glamu         (t, y, x) float32 dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;
    glamv         (t, y, x) float32 dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;
    gphif         (t, y, x) float32 dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;
    gphit         (t, y, x) float32 dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;
    gphiu         (t, y, x) float32 dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;
    gphiv         (t, y, x) float32 dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;
Attributes:
    DOMAIN_number_total:  8972
    DOMAIN_size_global:   [4322, 3059]</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-e00108b2-ab44-4f9a-823d-b622fa373173' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-e00108b2-ab44-4f9a-823d-b622fa373173' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span>t</span>: 1</li><li><span>z</span>: 75</li><li><span>y</span>: 3059</li><li><span>x</span>: 4322</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-20187fee-54d9-41eb-b67e-bb2c0601fce6' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-20187fee-54d9-41eb-b67e-bb2c0601fce6' class='xr-section-summary'  title='Expand/collapse section'>Coordinates: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'></ul></div></li><li class='xr-section-item'><input id='section-4dcdb196-9561-44bc-be5c-83700e8aa549' class='xr-section-summary-in' type='checkbox'  ><label for='section-4dcdb196-9561-44bc-be5c-83700e8aa549' class='xr-section-summary' >Data variables: <span>(42)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>fmask</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 10, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-3531f0fe-db12-44ae-b47a-acb694e2a912' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-3531f0fe-db12-44ae-b47a-acb694e2a912' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-18de38bf-7f18-4b47-94df-9600121eb9ce' class='xr-var-data-in' type='checkbox'><label for='data-18de38bf-7f18-4b47-94df-9600121eb9ce' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 0.92 GiB </td>
                        <td> 1.98 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 10, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 512 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>fmaskutil</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 765, 1081), meta=np.ndarray&gt;</div><input id='attrs-a880df17-e723-40a5-b34b-981c420cfaba' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a880df17-e723-40a5-b34b-981c420cfaba' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-cb4cfdbf-077c-478c-94ef-40de6245b868' class='xr-var-data-in' type='checkbox'><label for='data-cb4cfdbf-077c-478c-94ef-40de6245b868' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 12.61 MiB </td>
                        <td> 807.58 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 765, 1081) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 16 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="70" y1="0" x2="84" y2="14" />
  <line x1="100" y1="0" x2="114" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="84" y1="14" x2="84" y2="99" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>nav_lat</span></div><div class='xr-var-dims'>(y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(383, 541), meta=np.ndarray&gt;</div><input id='attrs-046e4168-ee8e-4378-a06d-06bf2416ffa7' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-046e4168-ee8e-4378-a06d-06bf2416ffa7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-dd52232b-f39c-4624-9c67-8d10ead53826' class='xr-var-data-in' type='checkbox'><label for='data-dd52232b-f39c-4624-9c67-8d10ead53826' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (3059, 4322) </td>
                        <td> (383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="134" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="10" x2="120" y2="10" />
  <line x1="0" y1="21" x2="120" y2="21" />
  <line x1="0" y1="31" x2="120" y2="31" />
  <line x1="0" y1="42" x2="120" y2="42" />
  <line x1="0" y1="53" x2="120" y2="53" />
  <line x1="0" y1="63" x2="120" y2="63" />
  <line x1="0" y1="74" x2="120" y2="74" />
  <line x1="0" y1="84" x2="120" y2="84" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="84" style="stroke-width:2" />
  <line x1="15" y1="0" x2="15" y2="84" />
  <line x1="30" y1="0" x2="30" y2="84" />
  <line x1="45" y1="0" x2="45" y2="84" />
  <line x1="60" y1="0" x2="60" y2="84" />
  <line x1="75" y1="0" x2="75" y2="84" />
  <line x1="90" y1="0" x2="90" y2="84" />
  <line x1="105" y1="0" x2="105" y2="84" />
  <line x1="120" y1="0" x2="120" y2="84" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,84.93290143452106 0.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="104.932901" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="140.000000" y="42.466451" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,140.000000,42.466451)">3059</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>nav_lev</span></div><div class='xr-var-dims'>(z)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(75,), meta=np.ndarray&gt;</div><input id='attrs-a6e51354-9794-4a7c-8930-255b9ee80637' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a6e51354-9794-4a7c-8930-255b9ee80637' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-181db79e-fc04-4622-a4c1-45b34c32425b' class='xr-var-data-in' type='checkbox'><label for='data-181db79e-fc04-4622-a4c1-45b34c32425b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 300 B </td>
                        <td> 300 B </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (75,) </td>
                        <td> (75,) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="76" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="26" x2="120" y2="26" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="26" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="26" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,26.84266337678776 0.0,26.84266337678776" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="46.842663" font-size="1.0rem" font-weight="100" text-anchor="middle" >75</text>
  <text x="140.000000" y="13.421332" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,140.000000,13.421332)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>nav_lon</span></div><div class='xr-var-dims'>(y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(383, 541), meta=np.ndarray&gt;</div><input id='attrs-60be9500-a8ac-4a71-8723-eef8dde7e939' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-60be9500-a8ac-4a71-8723-eef8dde7e939' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-085750f6-08a6-4f0e-85a6-73b7e8124aa0' class='xr-var-data-in' type='checkbox'><label for='data-085750f6-08a6-4f0e-85a6-73b7e8124aa0' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (3059, 4322) </td>
                        <td> (383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="134" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="10" x2="120" y2="10" />
  <line x1="0" y1="21" x2="120" y2="21" />
  <line x1="0" y1="31" x2="120" y2="31" />
  <line x1="0" y1="42" x2="120" y2="42" />
  <line x1="0" y1="53" x2="120" y2="53" />
  <line x1="0" y1="63" x2="120" y2="63" />
  <line x1="0" y1="74" x2="120" y2="74" />
  <line x1="0" y1="84" x2="120" y2="84" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="84" style="stroke-width:2" />
  <line x1="15" y1="0" x2="15" y2="84" />
  <line x1="30" y1="0" x2="30" y2="84" />
  <line x1="45" y1="0" x2="45" y2="84" />
  <line x1="60" y1="0" x2="60" y2="84" />
  <line x1="75" y1="0" x2="75" y2="84" />
  <line x1="90" y1="0" x2="90" y2="84" />
  <line x1="105" y1="0" x2="105" y2="84" />
  <line x1="120" y1="0" x2="120" y2="84" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,84.93290143452106 0.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="104.932901" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="140.000000" y="42.466451" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,140.000000,42.466451)">3059</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>time_counter</span></div><div class='xr-var-dims'>(t)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1,), meta=np.ndarray&gt;</div><input id='attrs-4f2bc4c5-f6de-4974-8b66-43a49c9714ad' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-4f2bc4c5-f6de-4974-8b66-43a49c9714ad' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f076c40d-4d6c-4c10-aa7f-74d6f1a73c12' class='xr-var-data-in' type='checkbox'><label for='data-f076c40d-4d6c-4c10-aa7f-74d6f1a73c12' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 8 B </td>
                        <td> 8 B </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1,) </td>
                        <td> (1,) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="170" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="120" x2="120" y2="120" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="120" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="120" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,120.0 0.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="140.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="140.000000" y="60.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,140.000000,60.000000)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>tmask</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 10, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-6de9be31-6610-4520-9c76-f5b14cb665cb' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-6de9be31-6610-4520-9c76-f5b14cb665cb' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-6481482c-d79e-472d-ae33-d16751b6b5c4' class='xr-var-data-in' type='checkbox'><label for='data-6481482c-d79e-472d-ae33-d16751b6b5c4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 0.92 GiB </td>
                        <td> 1.98 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 10, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 512 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>tmaskutil</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 765, 1081), meta=np.ndarray&gt;</div><input id='attrs-5683520e-6dea-4817-8473-5172d17de84c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-5683520e-6dea-4817-8473-5172d17de84c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-2955eff5-25a4-4eb5-925c-fb71540513b5' class='xr-var-data-in' type='checkbox'><label for='data-2955eff5-25a4-4eb5-925c-fb71540513b5' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 12.61 MiB </td>
                        <td> 807.58 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 765, 1081) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 16 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="70" y1="0" x2="84" y2="14" />
  <line x1="100" y1="0" x2="114" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="84" y1="14" x2="84" y2="99" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>umask</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 10, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-0e2afeb2-f224-4597-a137-9e375ad414dd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-0e2afeb2-f224-4597-a137-9e375ad414dd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4458a56c-b602-416a-a448-bee90057f180' class='xr-var-data-in' type='checkbox'><label for='data-4458a56c-b602-416a-a448-bee90057f180' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 0.92 GiB </td>
                        <td> 1.98 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 10, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 512 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>umaskutil</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 765, 1081), meta=np.ndarray&gt;</div><input id='attrs-61e16362-ebe3-46e9-9648-e2286b55a198' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-61e16362-ebe3-46e9-9648-e2286b55a198' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-bc38b65c-e363-4e71-a40f-ca6439b58e14' class='xr-var-data-in' type='checkbox'><label for='data-bc38b65c-e363-4e71-a40f-ca6439b58e14' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 12.61 MiB </td>
                        <td> 807.58 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 765, 1081) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 16 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="70" y1="0" x2="84" y2="14" />
  <line x1="100" y1="0" x2="114" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="84" y1="14" x2="84" y2="99" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>vmask</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 10, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-41fe7039-8ae3-4ae7-8d63-f4e669e24bd5' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-41fe7039-8ae3-4ae7-8d63-f4e669e24bd5' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-65ef75a3-f884-43d4-b2a6-ba4e7db0821d' class='xr-var-data-in' type='checkbox'><label for='data-65ef75a3-f884-43d4-b2a6-ba4e7db0821d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 0.92 GiB </td>
                        <td> 1.98 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 10, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 512 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>vmaskutil</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>int8</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 765, 1081), meta=np.ndarray&gt;</div><input id='attrs-058d0a6b-2fe7-417d-bbdb-d761e7fa1905' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-058d0a6b-2fe7-417d-bbdb-d761e7fa1905' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-e1d3e05f-6779-48b9-9f92-a4c584145591' class='xr-var-data-in' type='checkbox'><label for='data-e1d3e05f-6779-48b9-9f92-a4c584145591' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 12.61 MiB </td>
                        <td> 807.58 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 765, 1081) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 16 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int8 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="70" y1="0" x2="84" y2="14" />
  <line x1="100" y1="0" x2="114" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="84" y1="14" x2="84" y2="99" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e3t_0</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-43471f38-de2d-46f4-816f-cecb2b3b6a88' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-43471f38-de2d-46f4-816f-cecb2b3b6a88' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-736f87aa-08c8-4b94-a108-8961a88341a0' class='xr-var-data-in' type='checkbox'><label for='data-736f87aa-08c8-4b94-a108-8961a88341a0' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 7.39 GiB </td>
                        <td> 3.96 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1920 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="5" x2="111" y2="21" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="15" x2="111" y2="32" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="26" x2="111" y2="43" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="37" x2="111" y2="53" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="47" x2="111" y2="64" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="58" x2="111" y2="75" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="69" x2="111" y2="85" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="79" x2="111" y2="96" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="21" x2="231" y2="21" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="32" x2="231" y2="32" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="43" x2="231" y2="43" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="53" x2="231" y2="53" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="64" x2="231" y2="64" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="75" x2="231" y2="75" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="85" x2="231" y2="85" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="96" x2="231" y2="96" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e3t_1d</span></div><div class='xr-var-dims'>(t, z)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 75), meta=np.ndarray&gt;</div><input id='attrs-a6d4f32a-ccd2-4ba9-ae14-d382f4c89905' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a6d4f32a-ccd2-4ba9-ae14-d382f4c89905' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-89f2cd98-d3ab-492b-b97c-8d24895edfb4' class='xr-var-data-in' type='checkbox'><label for='data-89f2cd98-d3ab-492b-b97c-8d24895edfb4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 600 B </td>
                        <td> 600 B </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75) </td>
                        <td> (1, 75) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="76" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="26" x2="120" y2="26" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="26" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="26" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,26.84266337678776 0.0,26.84266337678776" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="46.842663" font-size="1.0rem" font-weight="100" text-anchor="middle" >75</text>
  <text x="140.000000" y="13.421332" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,140.000000,13.421332)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e3u_0</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-193abd6b-0929-4698-8d19-e01f97976f27' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-193abd6b-0929-4698-8d19-e01f97976f27' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-065c2e5e-4f18-4981-82d2-8c1612469ac5' class='xr-var-data-in' type='checkbox'><label for='data-065c2e5e-4f18-4981-82d2-8c1612469ac5' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 7.39 GiB </td>
                        <td> 3.96 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1920 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="5" x2="111" y2="21" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="15" x2="111" y2="32" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="26" x2="111" y2="43" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="37" x2="111" y2="53" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="47" x2="111" y2="64" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="58" x2="111" y2="75" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="69" x2="111" y2="85" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="79" x2="111" y2="96" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="21" x2="231" y2="21" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="32" x2="231" y2="32" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="43" x2="231" y2="43" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="53" x2="231" y2="53" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="64" x2="231" y2="64" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="75" x2="231" y2="75" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="85" x2="231" y2="85" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="96" x2="231" y2="96" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e3v_0</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-3830ced0-43fc-444d-9e4c-810af63f988a' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-3830ced0-43fc-444d-9e4c-810af63f988a' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3d3a3c49-d846-4d54-9a81-d8f61c2c6ade' class='xr-var-data-in' type='checkbox'><label for='data-3d3a3c49-d846-4d54-9a81-d8f61c2c6ade' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 7.39 GiB </td>
                        <td> 3.96 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1920 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="5" x2="111" y2="21" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="15" x2="111" y2="32" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="26" x2="111" y2="43" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="37" x2="111" y2="53" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="47" x2="111" y2="64" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="58" x2="111" y2="75" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="69" x2="111" y2="85" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="79" x2="111" y2="96" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="21" x2="231" y2="21" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="32" x2="231" y2="32" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="43" x2="231" y2="43" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="53" x2="231" y2="53" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="64" x2="231" y2="64" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="75" x2="231" y2="75" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="85" x2="231" y2="85" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="96" x2="231" y2="96" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e3w_0</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-4aec30dc-a0d1-4f95-983e-a3830b967936' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-4aec30dc-a0d1-4f95-983e-a3830b967936' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-44675f5c-4456-4285-9e3e-f66a69967cd0' class='xr-var-data-in' type='checkbox'><label for='data-44675f5c-4456-4285-9e3e-f66a69967cd0' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 7.39 GiB </td>
                        <td> 3.96 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1920 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="5" x2="111" y2="21" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="15" x2="111" y2="32" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="26" x2="111" y2="43" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="37" x2="111" y2="53" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="47" x2="111" y2="64" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="58" x2="111" y2="75" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="69" x2="111" y2="85" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="79" x2="111" y2="96" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="21" x2="231" y2="21" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="32" x2="231" y2="32" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="43" x2="231" y2="43" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="53" x2="231" y2="53" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="64" x2="231" y2="64" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="75" x2="231" y2="75" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="85" x2="231" y2="85" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="96" x2="231" y2="96" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e3w_1d</span></div><div class='xr-var-dims'>(t, z)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 75), meta=np.ndarray&gt;</div><input id='attrs-750aa2a2-e7ce-4fde-8b68-8b7559eaccf8' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-750aa2a2-e7ce-4fde-8b68-8b7559eaccf8' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-26b75288-0e79-45ec-88b9-8a3b7708064e' class='xr-var-data-in' type='checkbox'><label for='data-26b75288-0e79-45ec-88b9-8a3b7708064e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 600 B </td>
                        <td> 600 B </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75) </td>
                        <td> (1, 75) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="76" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="26" x2="120" y2="26" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="26" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="26" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,26.84266337678776 0.0,26.84266337678776" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="46.842663" font-size="1.0rem" font-weight="100" text-anchor="middle" >75</text>
  <text x="140.000000" y="13.421332" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,140.000000,13.421332)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gdept_0</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-a2a95084-33c2-4fbd-85e5-a417157c3aca' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a2a95084-33c2-4fbd-85e5-a417157c3aca' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-1235d9f5-98a6-4f55-8081-56bc4a77894e' class='xr-var-data-in' type='checkbox'><label for='data-1235d9f5-98a6-4f55-8081-56bc4a77894e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 3.69 GiB </td>
                        <td> 3.95 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 960 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gdept_1d</span></div><div class='xr-var-dims'>(t, z)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 75), meta=np.ndarray&gt;</div><input id='attrs-fb4896d5-703a-4f90-a671-49454af987dd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-fb4896d5-703a-4f90-a671-49454af987dd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-9aed537a-c8ca-4f0b-8887-9189a3847eb1' class='xr-var-data-in' type='checkbox'><label for='data-9aed537a-c8ca-4f0b-8887-9189a3847eb1' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 600 B </td>
                        <td> 600 B </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75) </td>
                        <td> (1, 75) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="76" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="26" x2="120" y2="26" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="26" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="26" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,26.84266337678776 0.0,26.84266337678776" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="46.842663" font-size="1.0rem" font-weight="100" text-anchor="middle" >75</text>
  <text x="140.000000" y="13.421332" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,140.000000,13.421332)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gdepu</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-7115795f-3c0e-49e2-913c-0e57f6cd000a' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-7115795f-3c0e-49e2-913c-0e57f6cd000a' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-1ba76355-2671-4b4c-8956-c208f34458dc' class='xr-var-data-in' type='checkbox'><label for='data-1ba76355-2671-4b4c-8956-c208f34458dc' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 3.69 GiB </td>
                        <td> 3.95 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 960 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gdepv</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-095e2d05-35f6-488f-88a5-e2bf902d2f7b' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-095e2d05-35f6-488f-88a5-e2bf902d2f7b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-27dfba3c-b865-42e5-9175-e2961ebe48d0' class='xr-var-data-in' type='checkbox'><label for='data-27dfba3c-b865-42e5-9175-e2961ebe48d0' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 3.69 GiB </td>
                        <td> 3.95 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 960 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gdepw_0</span></div><div class='xr-var-dims'>(t, z, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-2e4111a3-9eca-4e57-b7aa-791d30356460' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-2e4111a3-9eca-4e57-b7aa-791d30356460' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-862f817f-4af6-44ed-8be6-9f0d25fb0c4c' class='xr-var-data-in' type='checkbox'><label for='data-862f817f-4af6-44ed-8be6-9f0d25fb0c4c' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 3.69 GiB </td>
                        <td> 3.95 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75, 3059, 4322) </td>
                        <td> (1, 5, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 960 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="376" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="95" y1="10" x2="111" y2="27" />
  <line x1="95" y1="21" x2="111" y2="37" />
  <line x1="95" y1="31" x2="111" y2="48" />
  <line x1="95" y1="42" x2="111" y2="59" />
  <line x1="95" y1="53" x2="111" y2="69" />
  <line x1="95" y1="63" x2="111" y2="80" />
  <line x1="95" y1="74" x2="111" y2="91" />
  <line x1="95" y1="84" x2="111" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="84" style="stroke-width:2" />
  <line x1="96" y1="1" x2="96" y2="86" />
  <line x1="97" y1="2" x2="97" y2="87" />
  <line x1="98" y1="3" x2="98" y2="88" />
  <line x1="99" y1="4" x2="99" y2="89" />
  <line x1="100" y1="5" x2="100" y2="90" />
  <line x1="101" y1="6" x2="101" y2="91" />
  <line x1="102" y1="7" x2="102" y2="92" />
  <line x1="103" y1="8" x2="103" y2="93" />
  <line x1="104" y1="9" x2="104" y2="94" />
  <line x1="106" y1="11" x2="106" y2="96" />
  <line x1="107" y1="12" x2="107" y2="97" />
  <line x1="108" y1="13" x2="108" y2="98" />
  <line x1="109" y1="14" x2="109" y2="99" />
  <line x1="110" y1="15" x2="110" y2="100" />
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 111.61338981631184,16.613389816311837 111.61338981631184,101.5462912508329 95.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="96" y1="1" x2="216" y2="1" />
  <line x1="97" y1="2" x2="217" y2="2" />
  <line x1="98" y1="3" x2="218" y2="3" />
  <line x1="99" y1="4" x2="219" y2="4" />
  <line x1="100" y1="5" x2="220" y2="5" />
  <line x1="101" y1="6" x2="221" y2="6" />
  <line x1="102" y1="7" x2="222" y2="7" />
  <line x1="103" y1="8" x2="223" y2="8" />
  <line x1="104" y1="9" x2="224" y2="9" />
  <line x1="106" y1="11" x2="226" y2="11" />
  <line x1="107" y1="12" x2="227" y2="12" />
  <line x1="108" y1="13" x2="228" y2="13" />
  <line x1="109" y1="14" x2="229" y2="14" />
  <line x1="110" y1="15" x2="230" y2="15" />
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="111" y2="16" style="stroke-width:2" />
  <line x1="110" y1="0" x2="126" y2="16" />
  <line x1="125" y1="0" x2="141" y2="16" />
  <line x1="140" y1="0" x2="156" y2="16" />
  <line x1="155" y1="0" x2="171" y2="16" />
  <line x1="170" y1="0" x2="186" y2="16" />
  <line x1="185" y1="0" x2="201" y2="16" />
  <line x1="200" y1="0" x2="216" y2="16" />
  <line x1="215" y1="0" x2="231" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 215.0,0.0 231.61338981631184,16.613389816311837 111.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="111" y1="16" x2="231" y2="16" style="stroke-width:2" />
  <line x1="111" y1="27" x2="231" y2="27" />
  <line x1="111" y1="37" x2="231" y2="37" />
  <line x1="111" y1="48" x2="231" y2="48" />
  <line x1="111" y1="59" x2="231" y2="59" />
  <line x1="111" y1="69" x2="231" y2="69" />
  <line x1="111" y1="80" x2="231" y2="80" />
  <line x1="111" y1="91" x2="231" y2="91" />
  <line x1="111" y1="101" x2="231" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="111" y1="16" x2="111" y2="101" style="stroke-width:2" />
  <line x1="126" y1="16" x2="126" y2="101" />
  <line x1="141" y1="16" x2="141" y2="101" />
  <line x1="156" y1="16" x2="156" y2="101" />
  <line x1="171" y1="16" x2="171" y2="101" />
  <line x1="186" y1="16" x2="186" y2="101" />
  <line x1="201" y1="16" x2="201" y2="101" />
  <line x1="216" y1="16" x2="216" y2="101" />
  <line x1="231" y1="16" x2="231" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="111.61338981631184,16.613389816311837 231.61338981631184,16.613389816311837 231.61338981631184,101.5462912508329 111.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="171.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="251.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,251.613390,59.079841)">3059</text>
  <text x="93.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,93.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gdepw_1d</span></div><div class='xr-var-dims'>(t, z)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 75), meta=np.ndarray&gt;</div><input id='attrs-7ee436fa-e402-4e81-b4d8-c9a1a3a05da7' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-7ee436fa-e402-4e81-b4d8-c9a1a3a05da7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-faf44fd6-233f-4064-b0f9-74f79dbba21b' class='xr-var-data-in' type='checkbox'><label for='data-faf44fd6-233f-4064-b0f9-74f79dbba21b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 600 B </td>
                        <td> 600 B </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 75) </td>
                        <td> (1, 75) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 1 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="76" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="26" x2="120" y2="26" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="26" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="26" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,26.84266337678776 0.0,26.84266337678776" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="46.842663" font-size="1.0rem" font-weight="100" text-anchor="middle" >75</text>
  <text x="140.000000" y="13.421332" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,140.000000,13.421332)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>mbathy</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>int16</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 1081), meta=np.ndarray&gt;</div><input id='attrs-f13d0f2b-9c0b-4399-b50e-e8a21d94fcd7' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-f13d0f2b-9c0b-4399-b50e-e8a21d94fcd7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-bbf6b414-6a3c-4292-8c62-237227792212' class='xr-var-data-in' type='checkbox'><label for='data-bbf6b414-6a3c-4292-8c62-237227792212' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 25.22 MiB </td>
                        <td> 808.64 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 1081) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 32 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> int16 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="70" y1="0" x2="84" y2="14" />
  <line x1="100" y1="0" x2="114" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="84" y1="14" x2="84" y2="99" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e1f</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-56c4b816-7856-4e34-9063-64ee7d2bbc34' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-56c4b816-7856-4e34-9063-64ee7d2bbc34' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-d6f11220-0f81-443a-9d35-ccebbfbb0164' class='xr-var-data-in' type='checkbox'><label for='data-d6f11220-0f81-443a-9d35-ccebbfbb0164' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e1t</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-fd0716ed-5fce-4e72-b5ff-7fa5456f796b' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-fd0716ed-5fce-4e72-b5ff-7fa5456f796b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-33d18a23-14d8-4788-86c5-5d6b695a7882' class='xr-var-data-in' type='checkbox'><label for='data-33d18a23-14d8-4788-86c5-5d6b695a7882' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e1u</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-98c6996e-a504-4c41-b608-b7a748d527b2' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-98c6996e-a504-4c41-b608-b7a748d527b2' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-cbce96fc-8372-49ac-886e-1bdc415f42de' class='xr-var-data-in' type='checkbox'><label for='data-cbce96fc-8372-49ac-886e-1bdc415f42de' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e1v</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-4fd23bfc-0d9c-4b88-91d6-20b7bb3bf7a0' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-4fd23bfc-0d9c-4b88-91d6-20b7bb3bf7a0' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-e4260f73-80df-4d80-b0fb-00c9f992dccb' class='xr-var-data-in' type='checkbox'><label for='data-e4260f73-80df-4d80-b0fb-00c9f992dccb' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e2f</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-d0211ce5-f3e0-479a-a5d2-9d5f1e9c2679' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-d0211ce5-f3e0-479a-a5d2-9d5f1e9c2679' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-7cbe818e-8293-4305-a384-3fd48ad6e87a' class='xr-var-data-in' type='checkbox'><label for='data-7cbe818e-8293-4305-a384-3fd48ad6e87a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e2t</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-6b18bdd8-0aa8-492a-a1a3-5beb878a0173' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-6b18bdd8-0aa8-492a-a1a3-5beb878a0173' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-d6b8c554-2d51-4fa3-9216-7d57b119fbaa' class='xr-var-data-in' type='checkbox'><label for='data-d6b8c554-2d51-4fa3-9216-7d57b119fbaa' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e2u</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-38ef0deb-6704-46ea-b37b-97146e94cddb' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-38ef0deb-6704-46ea-b37b-97146e94cddb' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b6ef933d-4ba3-4c25-891f-dea5983651ac' class='xr-var-data-in' type='checkbox'><label for='data-b6ef933d-4ba3-4c25-891f-dea5983651ac' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>e2v</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-e7ad48e6-f516-4c2d-9cca-61bb37c9d962' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-e7ad48e6-f516-4c2d-9cca-61bb37c9d962' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b5887e84-df34-44c6-83b0-71a4f390d433' class='xr-var-data-in' type='checkbox'><label for='data-b5887e84-df34-44c6-83b0-71a4f390d433' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>ff</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 192, 541), meta=np.ndarray&gt;</div><input id='attrs-5722867a-d759-4ec9-8add-91ca45990792' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-5722867a-d759-4ec9-8add-91ca45990792' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-a67af276-8e43-48b2-a147-9e912c3ba493' class='xr-var-data-in' type='checkbox'><label for='data-a67af276-8e43-48b2-a147-9e912c3ba493' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 100.87 MiB </td>
                        <td> 811.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 192, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 128 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float64 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="5" x2="24" y2="20" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="15" x2="24" y2="30" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="26" x2="24" y2="41" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="37" x2="24" y2="52" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="47" x2="24" y2="62" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="58" x2="24" y2="73" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="69" x2="24" y2="84" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="79" x2="24" y2="94" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="20" x2="144" y2="20" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="30" x2="144" y2="30" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="41" x2="144" y2="41" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="52" x2="144" y2="52" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="62" x2="144" y2="62" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="73" x2="144" y2="73" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="84" x2="144" y2="84" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="94" x2="144" y2="94" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>glamf</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-2372449b-bdd9-44bd-bf63-bde7bbbbcd5c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-2372449b-bdd9-44bd-bf63-bde7bbbbcd5c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-e7c293ee-dada-45f3-abec-9ba9525c9cf8' class='xr-var-data-in' type='checkbox'><label for='data-e7c293ee-dada-45f3-abec-9ba9525c9cf8' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>glamt</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-720f61f1-3e20-4c37-9df8-8e9b3e58913d' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-720f61f1-3e20-4c37-9df8-8e9b3e58913d' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-daca4054-2a80-4e1d-8e6f-96666f396947' class='xr-var-data-in' type='checkbox'><label for='data-daca4054-2a80-4e1d-8e6f-96666f396947' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>glamu</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-2cbfa9fe-0074-495d-817e-0d8fa3347e79' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-2cbfa9fe-0074-495d-817e-0d8fa3347e79' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-8a10ac4c-195e-4d5d-ae45-b1a6c4be637f' class='xr-var-data-in' type='checkbox'><label for='data-8a10ac4c-195e-4d5d-ae45-b1a6c4be637f' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>glamv</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-65943414-d830-41fb-967a-0f4c52cbbd67' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-65943414-d830-41fb-967a-0f4c52cbbd67' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-e665c4ce-3240-4070-bfd1-d267bd776169' class='xr-var-data-in' type='checkbox'><label for='data-e665c4ce-3240-4070-bfd1-d267bd776169' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gphif</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-4741056a-94b5-4659-b37a-d1708ee58eda' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-4741056a-94b5-4659-b37a-d1708ee58eda' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-0d02dd1b-aa52-4a45-a30c-70f109c3d626' class='xr-var-data-in' type='checkbox'><label for='data-0d02dd1b-aa52-4a45-a30c-70f109c3d626' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gphit</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-4ca8f599-d0a9-49b3-af9f-a2ef0dbc769c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-4ca8f599-d0a9-49b3-af9f-a2ef0dbc769c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-a3e7e8d2-65c9-4bc5-864f-c84a49aa06f2' class='xr-var-data-in' type='checkbox'><label for='data-a3e7e8d2-65c9-4bc5-864f-c84a49aa06f2' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gphiu</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-1db38436-a2a6-4c59-9151-c0fcb3cea278' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-1db38436-a2a6-4c59-9151-c0fcb3cea278' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4bb23a12-0e83-4870-bddb-fceb068d8010' class='xr-var-data-in' type='checkbox'><label for='data-4bb23a12-0e83-4870-bddb-fceb068d8010' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>gphiv</span></div><div class='xr-var-dims'>(t, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 383, 541), meta=np.ndarray&gt;</div><input id='attrs-5ab308a8-2415-451f-9033-c05f1c10563b' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-5ab308a8-2415-451f-9033-c05f1c10563b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-ef2c4eb8-9256-4997-8443-112d9fccf6cf' class='xr-var-data-in' type='checkbox'><label for='data-ef2c4eb8-9256-4997-8443-112d9fccf6cf' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 809.39 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 3059, 4322) </td>
                        <td> (1, 383, 541) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 64 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="149" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="10" x2="24" y2="25" />
  <line x1="10" y1="21" x2="24" y2="36" />
  <line x1="10" y1="31" x2="24" y2="46" />
  <line x1="10" y1="42" x2="24" y2="57" />
  <line x1="10" y1="53" x2="24" y2="68" />
  <line x1="10" y1="63" x2="24" y2="78" />
  <line x1="10" y1="74" x2="24" y2="89" />
  <line x1="10" y1="84" x2="24" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,99.88149938427546 10.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="25" y1="0" x2="39" y2="14" />
  <line x1="40" y1="0" x2="54" y2="14" />
  <line x1="55" y1="0" x2="70" y2="14" />
  <line x1="70" y1="0" x2="85" y2="14" />
  <line x1="85" y1="0" x2="100" y2="14" />
  <line x1="100" y1="0" x2="115" y2="14" />
  <line x1="115" y1="0" x2="130" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="25" x2="144" y2="25" />
  <line x1="24" y1="36" x2="144" y2="36" />
  <line x1="24" y1="46" x2="144" y2="46" />
  <line x1="24" y1="57" x2="144" y2="57" />
  <line x1="24" y1="68" x2="144" y2="68" />
  <line x1="24" y1="78" x2="144" y2="78" />
  <line x1="24" y1="89" x2="144" y2="89" />
  <line x1="24" y1="99" x2="144" y2="99" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="99" style="stroke-width:2" />
  <line x1="39" y1="14" x2="39" y2="99" />
  <line x1="54" y1="14" x2="54" y2="99" />
  <line x1="70" y1="14" x2="70" y2="99" />
  <line x1="85" y1="14" x2="85" y2="99" />
  <line x1="100" y1="14" x2="100" y2="99" />
  <line x1="115" y1="14" x2="115" y2="99" />
  <line x1="130" y1="14" x2="130" y2="99" />
  <line x1="144" y1="14" x2="144" y2="99" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,99.88149938427546 24.9485979497544,99.88149938427546" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="119.881499" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="164.948598" y="57.415049" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,57.415049)">3059</text>
  <text x="7.474299" y="112.407200" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,112.407200)">1</text>
</svg>
        </td>
    </tr>
</table></div></li></ul></div></li><li class='xr-section-item'><input id='section-f942a730-65c4-461a-8d41-ad8424254de4' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-f942a730-65c4-461a-8d41-ad8424254de4' class='xr-section-summary'  title='Expand/collapse section'>Indexes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'></ul></div></li><li class='xr-section-item'><input id='section-6f58dfb6-5ccf-40e5-b2c3-7658fb29ae44' class='xr-section-summary-in' type='checkbox'  checked><label for='section-6f58dfb6-5ccf-40e5-b2c3-7658fb29ae44' class='xr-section-summary' >Attributes: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>DOMAIN_number_total :</span></dt><dd>8972</dd><dt><span>DOMAIN_size_global :</span></dt><dd>[4322, 3059]</dd></dl></div></li></ul></div></div>




```python
t_grid
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
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
Dimensions:       (depth: 75, time_counter: 119, y: 3059, x: 4322)
Coordinates:
  * depth         (depth) float32 0.5058 1.556 2.668 ... 5.698e+03 5.902e+03
  * time_counter  (time_counter) datetime64[ns] 1960-01-06T12:00:00 ... 1969-...
Dimensions without coordinates: y, x
Data variables: (12/25)
    e3t           (time_counter, depth, y, x) float32 dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;
    mldkz5        (time_counter, y, x) float32 dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;
    mldr10_1      (time_counter, y, x) float32 dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;
    nav_lat       (y, x) float32 dask.array&lt;chunksize=(577, 577), meta=np.ndarray&gt;
    nav_lon       (y, x) float32 dask.array&lt;chunksize=(577, 577), meta=np.ndarray&gt;
    potemp        (time_counter, depth, y, x) float32 dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;
    ...            ...
    tauuo         (time_counter, y, x) float32 dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;
    uo            (time_counter, depth, y, x) float32 dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;
    uos           (time_counter, y, x) float32 dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;
    tauvo         (time_counter, y, x) float32 dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;
    vo            (time_counter, depth, y, x) float32 dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;
    vos           (time_counter, y, x) float32 dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;
Attributes:
    DOMAIN_number_total:  80
    DOMAIN_size_global:   [4322, 3059]
    conventions:          CF-1.1
    description:          ocean T grid variables
    ibegin:               1
    jbegin:               1
    name:                 ORCA0083-N06_1m_19591222_19601231
    ni:                   4322
    nj:                   39
    production:           An IPSL model
    timeStamp:            2014-Dec-03 05:19:35 GMT</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-8826f3f7-6298-4a83-b485-b091933e31f2' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-8826f3f7-6298-4a83-b485-b091933e31f2' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>depth</span>: 75</li><li><span class='xr-has-index'>time_counter</span>: 119</li><li><span>y</span>: 3059</li><li><span>x</span>: 4322</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-97098567-f8aa-4604-84cc-a7ee117e908a' class='xr-section-summary-in' type='checkbox'  checked><label for='section-97098567-f8aa-4604-84cc-a7ee117e908a' class='xr-section-summary' >Coordinates: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>depth</span></div><div class='xr-var-dims'>(depth)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>0.5058 1.556 ... 5.902e+03</div><input id='attrs-710a497e-33a5-4a06-a49b-e2c8b520c583' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-710a497e-33a5-4a06-a49b-e2c8b520c583' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-80e1a2df-0c51-4a7f-a8a2-64bb29994b41' class='xr-var-data-in' type='checkbox'><label for='data-80e1a2df-0c51-4a7f-a8a2-64bb29994b41' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>Z</dd><dt><span>long_name :</span></dt><dd>Vertical V levels</dd><dt><span>positive :</span></dt><dd>down</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><pre>array([5.057600e-01, 1.555855e+00, 2.667682e+00, 3.856280e+00, 5.140361e+00,
       6.543034e+00, 8.092519e+00, 9.822750e+00, 1.177368e+01, 1.399104e+01,
       1.652532e+01, 1.942980e+01, 2.275762e+01, 2.655830e+01, 3.087456e+01,
       3.574020e+01, 4.118002e+01, 4.721189e+01, 5.385064e+01, 6.111284e+01,
       6.902168e+01, 7.761116e+01, 8.692943e+01, 9.704131e+01, 1.080303e+02,
       1.200000e+02, 1.330758e+02, 1.474062e+02, 1.631645e+02, 1.805499e+02,
       1.997900e+02, 2.211412e+02, 2.448906e+02, 2.713564e+02, 3.008875e+02,
       3.338628e+02, 3.706885e+02, 4.117939e+02, 4.576256e+02, 5.086399e+02,
       5.652923e+02, 6.280260e+02, 6.972587e+02, 7.733683e+02, 8.566790e+02,
       9.474479e+02, 1.045854e+03, 1.151991e+03, 1.265861e+03, 1.387377e+03,
       1.516364e+03, 1.652568e+03, 1.795671e+03, 1.945296e+03, 2.101027e+03,
       2.262422e+03, 2.429025e+03, 2.600380e+03, 2.776039e+03, 2.955570e+03,
       3.138565e+03, 3.324641e+03, 3.513446e+03, 3.704657e+03, 3.897982e+03,
       4.093159e+03, 4.289953e+03, 4.488155e+03, 4.687581e+03, 4.888070e+03,
       5.089479e+03, 5.291683e+03, 5.494575e+03, 5.698061e+03, 5.902058e+03],
      dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time_counter</span></div><div class='xr-var-dims'>(time_counter)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>1960-01-06T12:00:00 ... 1969-12-...</div><input id='attrs-2773d62e-ac0f-4530-b137-b7664c0711c1' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-2773d62e-ac0f-4530-b137-b7664c0711c1' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-0d5ef830-0ad7-4e87-afa8-5e9a237cbba2' class='xr-var-data-in' type='checkbox'><label for='data-0d5ef830-0ad7-4e87-afa8-5e9a237cbba2' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>T</dd><dt><span>long_name :</span></dt><dd>Time axis</dd><dt><span>standard_name :</span></dt><dd>time</dd><dt><span>time_origin :</span></dt><dd>1950-01-01 00:00:00</dd><dt><span>title :</span></dt><dd>Time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;1960-01-06T12:00:00.000000000&#x27;, &#x27;1960-02-06T12:00:00.000000000&#x27;,
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
       &#x27;1961-11-16T00:00:00.000000000&#x27;, &#x27;1961-12-13T22:17:04.000000000&#x27;,
       &#x27;1962-01-16T12:00:00.000000000&#x27;, &#x27;1962-02-15T00:00:00.000000000&#x27;,
       &#x27;1962-03-16T12:00:00.000000000&#x27;, &#x27;1962-04-16T00:00:00.000000000&#x27;,
       &#x27;1962-05-16T12:00:00.000000000&#x27;, &#x27;1962-06-16T00:00:00.000000000&#x27;,
       &#x27;1962-07-16T12:00:00.000000000&#x27;, &#x27;1962-08-16T12:00:00.000000000&#x27;,
       &#x27;1962-09-13T00:00:00.000000000&#x27;, &#x27;1962-10-13T00:00:00.000000000&#x27;,
       &#x27;1962-11-23T00:00:00.000000000&#x27;, &#x27;1962-12-16T00:00:00.000000000&#x27;,
       &#x27;1963-01-16T12:00:00.000000000&#x27;, &#x27;1963-02-15T00:00:00.000000000&#x27;,
       &#x27;1963-03-16T12:00:00.000000000&#x27;, &#x27;1963-04-16T00:00:00.000000000&#x27;,
       &#x27;1963-05-16T12:00:00.000000000&#x27;, &#x27;1963-06-16T00:00:00.000000000&#x27;,
       &#x27;1963-07-16T12:00:00.000000000&#x27;, &#x27;1963-08-16T12:00:00.000000000&#x27;,
       &#x27;1963-09-16T00:00:00.000000000&#x27;, &#x27;1963-10-16T12:00:00.000000000&#x27;,
       &#x27;1963-11-16T00:00:00.000000000&#x27;, &#x27;1963-12-16T12:00:00.000000000&#x27;,
       &#x27;1964-01-16T12:00:00.000000000&#x27;, &#x27;1964-02-15T12:00:00.000000000&#x27;,
       &#x27;1964-03-16T12:00:00.000000000&#x27;, &#x27;1964-04-16T00:00:00.000000000&#x27;,
       &#x27;1964-05-16T12:00:00.000000000&#x27;, &#x27;1964-06-16T00:00:00.000000000&#x27;,
       &#x27;1964-07-16T12:00:00.000000000&#x27;, &#x27;1964-08-16T12:00:00.000000000&#x27;,
       &#x27;1964-09-16T00:00:00.000000000&#x27;, &#x27;1964-10-16T12:00:00.000000000&#x27;,
       &#x27;1964-11-16T00:00:00.000000000&#x27;, &#x27;1964-12-16T12:00:00.000000000&#x27;,
       &#x27;1965-01-16T12:00:00.000000000&#x27;, &#x27;1965-02-15T00:00:00.000000000&#x27;,
       &#x27;1965-03-16T12:00:00.000000000&#x27;, &#x27;1965-04-16T00:00:00.000000000&#x27;,
       &#x27;1965-05-16T12:00:00.000000000&#x27;, &#x27;1965-06-16T00:00:00.000000000&#x27;,
       &#x27;1965-07-16T12:00:00.000000000&#x27;, &#x27;1965-08-16T12:00:00.000000000&#x27;,
       &#x27;1965-09-16T00:00:00.000000000&#x27;, &#x27;1965-10-16T12:00:00.000000000&#x27;,
       &#x27;1965-11-16T00:00:00.000000000&#x27;, &#x27;1965-12-16T12:00:00.000000000&#x27;,
       &#x27;1966-01-16T12:00:00.000000000&#x27;, &#x27;1966-02-15T00:00:00.000000000&#x27;,
       &#x27;1966-03-16T12:00:00.000000000&#x27;, &#x27;1966-04-16T00:00:00.000000000&#x27;,
       &#x27;1966-05-16T12:00:00.000000000&#x27;, &#x27;1966-06-16T00:00:00.000000000&#x27;,
       &#x27;1966-07-16T12:00:00.000000000&#x27;, &#x27;1966-08-16T12:00:00.000000000&#x27;,
       &#x27;1966-09-16T00:00:00.000000000&#x27;, &#x27;1966-10-03T00:00:00.000000000&#x27;,
       &#x27;1966-11-02T12:00:00.000000000&#x27;, &#x27;1966-12-03T00:00:00.000000000&#x27;,
       &#x27;1967-01-16T12:00:00.000000000&#x27;, &#x27;1967-02-15T00:00:00.000000000&#x27;,
       &#x27;1967-03-16T12:00:00.000000000&#x27;, &#x27;1967-04-16T00:00:00.000000000&#x27;,
       &#x27;1967-05-16T12:00:00.000000000&#x27;, &#x27;1967-06-16T00:00:00.000000000&#x27;,
       &#x27;1967-07-16T12:00:00.000000000&#x27;, &#x27;1967-08-16T12:00:00.000000000&#x27;,
       &#x27;1967-09-16T00:00:00.000000000&#x27;, &#x27;1967-11-16T00:00:00.000000000&#x27;,
       &#x27;1967-11-25T00:00:00.000000000&#x27;, &#x27;1968-01-16T12:00:00.000000000&#x27;,
       &#x27;1968-02-15T12:00:00.000000000&#x27;, &#x27;1968-03-16T12:00:00.000000000&#x27;,
       &#x27;1968-04-16T00:00:00.000000000&#x27;, &#x27;1968-05-16T12:00:00.000000000&#x27;,
       &#x27;1968-06-16T00:00:00.000000000&#x27;, &#x27;1968-07-16T12:00:00.000000000&#x27;,
       &#x27;1968-08-16T12:00:00.000000000&#x27;, &#x27;1968-09-16T00:00:00.000000000&#x27;,
       &#x27;1968-10-16T12:00:00.000000000&#x27;, &#x27;1968-11-16T00:00:00.000000000&#x27;,
       &#x27;1968-12-16T12:00:00.000000000&#x27;, &#x27;1969-01-16T12:00:00.000000000&#x27;,
       &#x27;1969-02-15T00:00:00.000000000&#x27;, &#x27;1969-03-16T12:00:00.000000000&#x27;,
       &#x27;1969-04-16T00:00:00.000000000&#x27;, &#x27;1969-05-16T12:00:00.000000000&#x27;,
       &#x27;1969-06-16T00:00:00.000000000&#x27;, &#x27;1969-07-16T12:00:00.000000000&#x27;,
       &#x27;1969-08-16T12:00:00.000000000&#x27;, &#x27;1969-09-16T00:00:00.000000000&#x27;,
       &#x27;1969-10-16T12:00:00.000000000&#x27;, &#x27;1969-11-16T00:00:00.000000000&#x27;,
       &#x27;1969-12-16T12:00:00.000000000&#x27;], dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-d9390759-c391-426c-b8a7-514246b12120' class='xr-section-summary-in' type='checkbox'  ><label for='section-d9390759-c391-426c-b8a7-514246b12120' class='xr-section-summary' >Data variables: <span>(25)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>e3t</span></div><div class='xr-var-dims'>(time_counter, depth, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-1bc4fe8c-57b1-4dea-9473-b5df48add083' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-1bc4fe8c-57b1-4dea-9473-b5df48add083' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-30df0219-93e2-4812-8dff-9bfe2a361ac8' class='xr-var-data-in' type='checkbox'><label for='data-30df0219-93e2-4812-8dff-9bfe2a361ac8' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered deptht nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>Cell thickness</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 439.57 GiB </td>
                        <td> 6.35 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 75, 3059, 4322) </td>
                        <td> (1, 5, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 85680 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="386" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="30" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="30" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="0" y1="0" x2="0" y2="25" />
  <line x1="1" y1="0" x2="1" y2="25" />
  <line x1="2" y1="0" x2="2" y2="25" />
  <line x1="3" y1="0" x2="3" y2="25" />
  <line x1="4" y1="0" x2="4" y2="25" />
  <line x1="5" y1="0" x2="5" y2="25" />
  <line x1="6" y1="0" x2="6" y2="25" />
  <line x1="7" y1="0" x2="7" y2="25" />
  <line x1="8" y1="0" x2="8" y2="25" />
  <line x1="9" y1="0" x2="9" y2="25" />
  <line x1="10" y1="0" x2="10" y2="25" />
  <line x1="11" y1="0" x2="11" y2="25" />
  <line x1="12" y1="0" x2="12" y2="25" />
  <line x1="13" y1="0" x2="13" y2="25" />
  <line x1="14" y1="0" x2="14" y2="25" />
  <line x1="15" y1="0" x2="15" y2="25" />
  <line x1="16" y1="0" x2="16" y2="25" />
  <line x1="17" y1="0" x2="17" y2="25" />
  <line x1="18" y1="0" x2="18" y2="25" />
  <line x1="19" y1="0" x2="19" y2="25" />
  <line x1="20" y1="0" x2="20" y2="25" />
  <line x1="21" y1="0" x2="21" y2="25" />
  <line x1="22" y1="0" x2="22" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="24" y1="0" x2="24" y2="25" />
  <line x1="25" y1="0" x2="25" y2="25" />
  <line x1="26" y1="0" x2="26" y2="25" />
  <line x1="27" y1="0" x2="27" y2="25" />
  <line x1="28" y1="0" x2="28" y2="25" />
  <line x1="29" y1="0" x2="29" y2="25" />
  <line x1="30" y1="0" x2="30" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 30.879479818120135,0.0 30.879479818120135,25.412616514582485 0.0,25.412616514582485" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Text -->
  <text x="15.439740" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >119</text>
  <text x="50.879480" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,50.879480,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="100" y1="16" x2="116" y2="32" />
  <line x1="100" y1="32" x2="116" y2="48" />
  <line x1="100" y1="48" x2="116" y2="64" />
  <line x1="100" y1="64" x2="116" y2="80" />
  <line x1="100" y1="80" x2="116" y2="96" />
  <line x1="100" y1="84" x2="116" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="100" y2="84" style="stroke-width:2" />
  <line x1="101" y1="1" x2="101" y2="86" />
  <line x1="102" y1="2" x2="102" y2="87" />
  <line x1="103" y1="3" x2="103" y2="88" />
  <line x1="104" y1="4" x2="104" y2="89" />
  <line x1="105" y1="5" x2="105" y2="90" />
  <line x1="106" y1="6" x2="106" y2="91" />
  <line x1="107" y1="7" x2="107" y2="92" />
  <line x1="108" y1="8" x2="108" y2="93" />
  <line x1="109" y1="9" x2="109" y2="94" />
  <line x1="111" y1="11" x2="111" y2="96" />
  <line x1="112" y1="12" x2="112" y2="97" />
  <line x1="113" y1="13" x2="113" y2="98" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="115" y1="15" x2="115" y2="100" />
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 116.61338981631184,16.613389816311837 116.61338981631184,101.5462912508329 100.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="220" y2="0" style="stroke-width:2" />
  <line x1="101" y1="1" x2="221" y2="1" />
  <line x1="102" y1="2" x2="222" y2="2" />
  <line x1="103" y1="3" x2="223" y2="3" />
  <line x1="104" y1="4" x2="224" y2="4" />
  <line x1="105" y1="5" x2="225" y2="5" />
  <line x1="106" y1="6" x2="226" y2="6" />
  <line x1="107" y1="7" x2="227" y2="7" />
  <line x1="108" y1="8" x2="228" y2="8" />
  <line x1="109" y1="9" x2="229" y2="9" />
  <line x1="111" y1="11" x2="231" y2="11" />
  <line x1="112" y1="12" x2="232" y2="12" />
  <line x1="113" y1="13" x2="233" y2="13" />
  <line x1="114" y1="14" x2="234" y2="14" />
  <line x1="115" y1="15" x2="235" y2="15" />
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="116" y1="0" x2="132" y2="16" />
  <line x1="132" y1="0" x2="148" y2="16" />
  <line x1="148" y1="0" x2="164" y2="16" />
  <line x1="164" y1="0" x2="180" y2="16" />
  <line x1="180" y1="0" x2="196" y2="16" />
  <line x1="196" y1="0" x2="212" y2="16" />
  <line x1="212" y1="0" x2="228" y2="16" />
  <line x1="220" y1="0" x2="236" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 220.0,0.0 236.61338981631184,16.613389816311837 116.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />
  <line x1="116" y1="32" x2="236" y2="32" />
  <line x1="116" y1="48" x2="236" y2="48" />
  <line x1="116" y1="64" x2="236" y2="64" />
  <line x1="116" y1="80" x2="236" y2="80" />
  <line x1="116" y1="96" x2="236" y2="96" />
  <line x1="116" y1="101" x2="236" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />
  <line x1="132" y1="16" x2="132" y2="101" />
  <line x1="148" y1="16" x2="148" y2="101" />
  <line x1="164" y1="16" x2="164" y2="101" />
  <line x1="180" y1="16" x2="180" y2="101" />
  <line x1="196" y1="16" x2="196" y2="101" />
  <line x1="212" y1="16" x2="212" y2="101" />
  <line x1="228" y1="16" x2="228" y2="101" />
  <line x1="236" y1="16" x2="236" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="116.61338981631184,16.613389816311837 236.61338981631184,16.613389816311837 236.61338981631184,101.5462912508329 116.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="176.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="256.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,256.613390,59.079841)">3059</text>
  <text x="98.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,98.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>mldkz5</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-65d3f570-a81f-4dd4-90fc-412bbe30453c' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-65d3f570-a81f-4dd4-90fc-412bbe30453c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-743bf27e-abe4-464c-abe6-366e39e658f7' class='xr-var-data-in' type='checkbox'><label for='data-743bf27e-abe4-464c-abe6-366e39e658f7' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>mixing layer depth (Turbocline)</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>mldr10_1</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-4c5f894a-6561-4e9a-a298-b830e3dcdc8d' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-4c5f894a-6561-4e9a-a298-b830e3dcdc8d' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-56a23beb-8614-4bff-90b4-b081c4a8a391' class='xr-var-data-in' type='checkbox'><label for='data-56a23beb-8614-4bff-90b4-b081c4a8a391' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>Mixed Layer Depth 0.01 ref.10m</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>nav_lat</span></div><div class='xr-var-dims'>(y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(577, 577), meta=np.ndarray&gt;</div><input id='attrs-dfbf7d2d-59ec-4565-be45-030c8f76c960' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-dfbf7d2d-59ec-4565-be45-030c8f76c960' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-6b4cf489-3ca9-4129-a0ce-b30fc1b468aa' class='xr-var-data-in' type='checkbox'><label for='data-6b4cf489-3ca9-4129-a0ce-b30fc1b468aa' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>Y</dd><dt><span>long_name :</span></dt><dd>Latitude</dd><dt><span>nav_model :</span></dt><dd>grid_V</dd><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (3059, 4322) </td>
                        <td> (577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 48 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="134" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="16" x2="120" y2="16" />
  <line x1="0" y1="32" x2="120" y2="32" />
  <line x1="0" y1="48" x2="120" y2="48" />
  <line x1="0" y1="64" x2="120" y2="64" />
  <line x1="0" y1="80" x2="120" y2="80" />
  <line x1="0" y1="84" x2="120" y2="84" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="84" style="stroke-width:2" />
  <line x1="16" y1="0" x2="16" y2="84" />
  <line x1="32" y1="0" x2="32" y2="84" />
  <line x1="48" y1="0" x2="48" y2="84" />
  <line x1="64" y1="0" x2="64" y2="84" />
  <line x1="80" y1="0" x2="80" y2="84" />
  <line x1="96" y1="0" x2="96" y2="84" />
  <line x1="112" y1="0" x2="112" y2="84" />
  <line x1="120" y1="0" x2="120" y2="84" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,84.93290143452106 0.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="104.932901" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="140.000000" y="42.466451" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,140.000000,42.466451)">3059</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>nav_lon</span></div><div class='xr-var-dims'>(y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(577, 577), meta=np.ndarray&gt;</div><input id='attrs-21beb8ef-ca25-44d6-9e6d-04e55e5d383e' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-21beb8ef-ca25-44d6-9e6d-04e55e5d383e' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-aa10f668-3822-45f7-8b75-0f60ec1d65f5' class='xr-var-data-in' type='checkbox'><label for='data-aa10f668-3822-45f7-8b75-0f60ec1d65f5' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>X</dd><dt><span>long_name :</span></dt><dd>Longitude</dd><dt><span>nav_model :</span></dt><dd>grid_V</dd><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>units :</span></dt><dd>degrees_east</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 50.43 MiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (3059, 4322) </td>
                        <td> (577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 48 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="170" height="134" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="16" x2="120" y2="16" />
  <line x1="0" y1="32" x2="120" y2="32" />
  <line x1="0" y1="48" x2="120" y2="48" />
  <line x1="0" y1="64" x2="120" y2="64" />
  <line x1="0" y1="80" x2="120" y2="80" />
  <line x1="0" y1="84" x2="120" y2="84" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="84" style="stroke-width:2" />
  <line x1="16" y1="0" x2="16" y2="84" />
  <line x1="32" y1="0" x2="32" y2="84" />
  <line x1="48" y1="0" x2="48" y2="84" />
  <line x1="64" y1="0" x2="64" y2="84" />
  <line x1="80" y1="0" x2="80" y2="84" />
  <line x1="96" y1="0" x2="96" y2="84" />
  <line x1="112" y1="0" x2="112" y2="84" />
  <line x1="120" y1="0" x2="120" y2="84" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,84.93290143452106 0.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="104.932901" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="140.000000" y="42.466451" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,140.000000,42.466451)">3059</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>potemp</span></div><div class='xr-var-dims'>(time_counter, depth, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-192e6ed9-79fe-4fe0-9bae-edd13fd84bad' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-192e6ed9-79fe-4fe0-9bae-edd13fd84bad' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-439187b5-c831-468c-8df6-91f50eb20610' class='xr-var-data-in' type='checkbox'><label for='data-439187b5-c831-468c-8df6-91f50eb20610' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_instant deptht nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>1mo</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_water_potential_temperature</dd><dt><span>online_operation :</span></dt><dd>instant</dd><dt><span>units :</span></dt><dd>degC</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 439.57 GiB </td>
                        <td> 6.35 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 75, 3059, 4322) </td>
                        <td> (1, 5, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 85680 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="386" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="30" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="30" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="0" y1="0" x2="0" y2="25" />
  <line x1="1" y1="0" x2="1" y2="25" />
  <line x1="2" y1="0" x2="2" y2="25" />
  <line x1="3" y1="0" x2="3" y2="25" />
  <line x1="4" y1="0" x2="4" y2="25" />
  <line x1="5" y1="0" x2="5" y2="25" />
  <line x1="6" y1="0" x2="6" y2="25" />
  <line x1="7" y1="0" x2="7" y2="25" />
  <line x1="8" y1="0" x2="8" y2="25" />
  <line x1="9" y1="0" x2="9" y2="25" />
  <line x1="10" y1="0" x2="10" y2="25" />
  <line x1="11" y1="0" x2="11" y2="25" />
  <line x1="12" y1="0" x2="12" y2="25" />
  <line x1="13" y1="0" x2="13" y2="25" />
  <line x1="14" y1="0" x2="14" y2="25" />
  <line x1="15" y1="0" x2="15" y2="25" />
  <line x1="16" y1="0" x2="16" y2="25" />
  <line x1="17" y1="0" x2="17" y2="25" />
  <line x1="18" y1="0" x2="18" y2="25" />
  <line x1="19" y1="0" x2="19" y2="25" />
  <line x1="20" y1="0" x2="20" y2="25" />
  <line x1="21" y1="0" x2="21" y2="25" />
  <line x1="22" y1="0" x2="22" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="24" y1="0" x2="24" y2="25" />
  <line x1="25" y1="0" x2="25" y2="25" />
  <line x1="26" y1="0" x2="26" y2="25" />
  <line x1="27" y1="0" x2="27" y2="25" />
  <line x1="28" y1="0" x2="28" y2="25" />
  <line x1="29" y1="0" x2="29" y2="25" />
  <line x1="30" y1="0" x2="30" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 30.879479818120135,0.0 30.879479818120135,25.412616514582485 0.0,25.412616514582485" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Text -->
  <text x="15.439740" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >119</text>
  <text x="50.879480" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,50.879480,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="100" y1="16" x2="116" y2="32" />
  <line x1="100" y1="32" x2="116" y2="48" />
  <line x1="100" y1="48" x2="116" y2="64" />
  <line x1="100" y1="64" x2="116" y2="80" />
  <line x1="100" y1="80" x2="116" y2="96" />
  <line x1="100" y1="84" x2="116" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="100" y2="84" style="stroke-width:2" />
  <line x1="101" y1="1" x2="101" y2="86" />
  <line x1="102" y1="2" x2="102" y2="87" />
  <line x1="103" y1="3" x2="103" y2="88" />
  <line x1="104" y1="4" x2="104" y2="89" />
  <line x1="105" y1="5" x2="105" y2="90" />
  <line x1="106" y1="6" x2="106" y2="91" />
  <line x1="107" y1="7" x2="107" y2="92" />
  <line x1="108" y1="8" x2="108" y2="93" />
  <line x1="109" y1="9" x2="109" y2="94" />
  <line x1="111" y1="11" x2="111" y2="96" />
  <line x1="112" y1="12" x2="112" y2="97" />
  <line x1="113" y1="13" x2="113" y2="98" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="115" y1="15" x2="115" y2="100" />
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 116.61338981631184,16.613389816311837 116.61338981631184,101.5462912508329 100.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="220" y2="0" style="stroke-width:2" />
  <line x1="101" y1="1" x2="221" y2="1" />
  <line x1="102" y1="2" x2="222" y2="2" />
  <line x1="103" y1="3" x2="223" y2="3" />
  <line x1="104" y1="4" x2="224" y2="4" />
  <line x1="105" y1="5" x2="225" y2="5" />
  <line x1="106" y1="6" x2="226" y2="6" />
  <line x1="107" y1="7" x2="227" y2="7" />
  <line x1="108" y1="8" x2="228" y2="8" />
  <line x1="109" y1="9" x2="229" y2="9" />
  <line x1="111" y1="11" x2="231" y2="11" />
  <line x1="112" y1="12" x2="232" y2="12" />
  <line x1="113" y1="13" x2="233" y2="13" />
  <line x1="114" y1="14" x2="234" y2="14" />
  <line x1="115" y1="15" x2="235" y2="15" />
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="116" y1="0" x2="132" y2="16" />
  <line x1="132" y1="0" x2="148" y2="16" />
  <line x1="148" y1="0" x2="164" y2="16" />
  <line x1="164" y1="0" x2="180" y2="16" />
  <line x1="180" y1="0" x2="196" y2="16" />
  <line x1="196" y1="0" x2="212" y2="16" />
  <line x1="212" y1="0" x2="228" y2="16" />
  <line x1="220" y1="0" x2="236" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 220.0,0.0 236.61338981631184,16.613389816311837 116.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />
  <line x1="116" y1="32" x2="236" y2="32" />
  <line x1="116" y1="48" x2="236" y2="48" />
  <line x1="116" y1="64" x2="236" y2="64" />
  <line x1="116" y1="80" x2="236" y2="80" />
  <line x1="116" y1="96" x2="236" y2="96" />
  <line x1="116" y1="101" x2="236" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />
  <line x1="132" y1="16" x2="132" y2="101" />
  <line x1="148" y1="16" x2="148" y2="101" />
  <line x1="164" y1="16" x2="164" y2="101" />
  <line x1="180" y1="16" x2="180" y2="101" />
  <line x1="196" y1="16" x2="196" y2="101" />
  <line x1="212" y1="16" x2="212" y2="101" />
  <line x1="228" y1="16" x2="228" y2="101" />
  <line x1="236" y1="16" x2="236" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="116.61338981631184,16.613389816311837 236.61338981631184,16.613389816311837 236.61338981631184,101.5462912508329 116.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="176.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="256.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,256.613390,59.079841)">3059</text>
  <text x="98.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,98.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>rsntds</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-8813207a-cf5c-49d3-92f9-1fcb0d0db2f9' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-8813207a-cf5c-49d3-92f9-1fcb0d0db2f9' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-dd376de1-0e60-48b9-a15f-584ca2f5c619' class='xr-var-data-in' type='checkbox'><label for='data-dd376de1-0e60-48b9-a15f-584ca2f5c619' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>surface_net_downward_shortwave_flux</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>W/m2</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>salin</span></div><div class='xr-var-dims'>(time_counter, depth, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-8112c450-b439-4480-9e0c-0a905e70938c' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-8112c450-b439-4480-9e0c-0a905e70938c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-2383a64f-3b57-4a00-84f8-72383e7e65a0' class='xr-var-data-in' type='checkbox'><label for='data-2383a64f-3b57-4a00-84f8-72383e7e65a0' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_instant deptht nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>1mo</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_water_salinity</dd><dt><span>online_operation :</span></dt><dd>instant</dd><dt><span>units :</span></dt><dd>psu</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 439.57 GiB </td>
                        <td> 6.35 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 75, 3059, 4322) </td>
                        <td> (1, 5, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 85680 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="386" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="30" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="30" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="0" y1="0" x2="0" y2="25" />
  <line x1="1" y1="0" x2="1" y2="25" />
  <line x1="2" y1="0" x2="2" y2="25" />
  <line x1="3" y1="0" x2="3" y2="25" />
  <line x1="4" y1="0" x2="4" y2="25" />
  <line x1="5" y1="0" x2="5" y2="25" />
  <line x1="6" y1="0" x2="6" y2="25" />
  <line x1="7" y1="0" x2="7" y2="25" />
  <line x1="8" y1="0" x2="8" y2="25" />
  <line x1="9" y1="0" x2="9" y2="25" />
  <line x1="10" y1="0" x2="10" y2="25" />
  <line x1="11" y1="0" x2="11" y2="25" />
  <line x1="12" y1="0" x2="12" y2="25" />
  <line x1="13" y1="0" x2="13" y2="25" />
  <line x1="14" y1="0" x2="14" y2="25" />
  <line x1="15" y1="0" x2="15" y2="25" />
  <line x1="16" y1="0" x2="16" y2="25" />
  <line x1="17" y1="0" x2="17" y2="25" />
  <line x1="18" y1="0" x2="18" y2="25" />
  <line x1="19" y1="0" x2="19" y2="25" />
  <line x1="20" y1="0" x2="20" y2="25" />
  <line x1="21" y1="0" x2="21" y2="25" />
  <line x1="22" y1="0" x2="22" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="24" y1="0" x2="24" y2="25" />
  <line x1="25" y1="0" x2="25" y2="25" />
  <line x1="26" y1="0" x2="26" y2="25" />
  <line x1="27" y1="0" x2="27" y2="25" />
  <line x1="28" y1="0" x2="28" y2="25" />
  <line x1="29" y1="0" x2="29" y2="25" />
  <line x1="30" y1="0" x2="30" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 30.879479818120135,0.0 30.879479818120135,25.412616514582485 0.0,25.412616514582485" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Text -->
  <text x="15.439740" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >119</text>
  <text x="50.879480" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,50.879480,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="100" y1="16" x2="116" y2="32" />
  <line x1="100" y1="32" x2="116" y2="48" />
  <line x1="100" y1="48" x2="116" y2="64" />
  <line x1="100" y1="64" x2="116" y2="80" />
  <line x1="100" y1="80" x2="116" y2="96" />
  <line x1="100" y1="84" x2="116" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="100" y2="84" style="stroke-width:2" />
  <line x1="101" y1="1" x2="101" y2="86" />
  <line x1="102" y1="2" x2="102" y2="87" />
  <line x1="103" y1="3" x2="103" y2="88" />
  <line x1="104" y1="4" x2="104" y2="89" />
  <line x1="105" y1="5" x2="105" y2="90" />
  <line x1="106" y1="6" x2="106" y2="91" />
  <line x1="107" y1="7" x2="107" y2="92" />
  <line x1="108" y1="8" x2="108" y2="93" />
  <line x1="109" y1="9" x2="109" y2="94" />
  <line x1="111" y1="11" x2="111" y2="96" />
  <line x1="112" y1="12" x2="112" y2="97" />
  <line x1="113" y1="13" x2="113" y2="98" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="115" y1="15" x2="115" y2="100" />
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 116.61338981631184,16.613389816311837 116.61338981631184,101.5462912508329 100.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="220" y2="0" style="stroke-width:2" />
  <line x1="101" y1="1" x2="221" y2="1" />
  <line x1="102" y1="2" x2="222" y2="2" />
  <line x1="103" y1="3" x2="223" y2="3" />
  <line x1="104" y1="4" x2="224" y2="4" />
  <line x1="105" y1="5" x2="225" y2="5" />
  <line x1="106" y1="6" x2="226" y2="6" />
  <line x1="107" y1="7" x2="227" y2="7" />
  <line x1="108" y1="8" x2="228" y2="8" />
  <line x1="109" y1="9" x2="229" y2="9" />
  <line x1="111" y1="11" x2="231" y2="11" />
  <line x1="112" y1="12" x2="232" y2="12" />
  <line x1="113" y1="13" x2="233" y2="13" />
  <line x1="114" y1="14" x2="234" y2="14" />
  <line x1="115" y1="15" x2="235" y2="15" />
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="116" y1="0" x2="132" y2="16" />
  <line x1="132" y1="0" x2="148" y2="16" />
  <line x1="148" y1="0" x2="164" y2="16" />
  <line x1="164" y1="0" x2="180" y2="16" />
  <line x1="180" y1="0" x2="196" y2="16" />
  <line x1="196" y1="0" x2="212" y2="16" />
  <line x1="212" y1="0" x2="228" y2="16" />
  <line x1="220" y1="0" x2="236" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 220.0,0.0 236.61338981631184,16.613389816311837 116.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />
  <line x1="116" y1="32" x2="236" y2="32" />
  <line x1="116" y1="48" x2="236" y2="48" />
  <line x1="116" y1="64" x2="236" y2="64" />
  <line x1="116" y1="80" x2="236" y2="80" />
  <line x1="116" y1="96" x2="236" y2="96" />
  <line x1="116" y1="101" x2="236" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />
  <line x1="132" y1="16" x2="132" y2="101" />
  <line x1="148" y1="16" x2="148" y2="101" />
  <line x1="164" y1="16" x2="164" y2="101" />
  <line x1="180" y1="16" x2="180" y2="101" />
  <line x1="196" y1="16" x2="196" y2="101" />
  <line x1="212" y1="16" x2="212" y2="101" />
  <line x1="228" y1="16" x2="228" y2="101" />
  <line x1="236" y1="16" x2="236" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="116.61338981631184,16.613389816311837 236.61338981631184,16.613389816311837 236.61338981631184,101.5462912508329 116.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="176.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="256.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,256.613390,59.079841)">3059</text>
  <text x="98.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,98.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>soprecip</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-3d3b5654-6d2a-4eeb-be98-2aa47b9e2a30' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-3d3b5654-6d2a-4eeb-be98-2aa47b9e2a30' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-7aba89e9-819c-44e7-8c59-7a9778d68b54' class='xr-var-data-in' type='checkbox'><label for='data-7aba89e9-819c-44e7-8c59-7a9778d68b54' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>Total precipitation</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>kg/m2/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>sosflxdo</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-0d57544e-fcf8-4b21-8881-d28277fc1064' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-0d57544e-fcf8-4b21-8881-d28277fc1064' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-893216cc-7f58-4f35-a24b-94bc19a4f603' class='xr-var-data-in' type='checkbox'><label for='data-893216cc-7f58-4f35-a24b-94bc19a4f603' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>Downward salt flux</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>PSU/m2/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>sowindsp</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-44f9e83d-c499-4240-8d1d-645734e2692f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-44f9e83d-c499-4240-8d1d-645734e2692f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b3ebc34c-0021-4cde-a48c-361c61dd615d' class='xr-var-data-in' type='checkbox'><label for='data-b3ebc34c-0021-4cde-a48c-361c61dd615d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>Wind speed module at 10 m</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>ssh</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-b1d9354c-572a-4963-9449-03f4523c7e10' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-b1d9354c-572a-4963-9449-03f4523c7e10' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-61606767-7327-4a47-8037-ed393b752859' class='xr-var-data-in' type='checkbox'><label for='data-61606767-7327-4a47-8037-ed393b752859' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_surface_height_above_geoid</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>sss</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-f66f2c60-64dc-48b7-ad5a-d32008bc5dd9' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-f66f2c60-64dc-48b7-ad5a-d32008bc5dd9' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-2a338073-cc96-4f84-ba3f-ce7ac5daa77b' class='xr-var-data-in' type='checkbox'><label for='data-2a338073-cc96-4f84-ba3f-ce7ac5daa77b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_instant nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>1mo</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_surface_salinity</dd><dt><span>online_operation :</span></dt><dd>instant</dd><dt><span>units :</span></dt><dd>psu</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>sst</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-f2243541-24a8-43a9-b577-e20ba9bbdd4a' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-f2243541-24a8-43a9-b577-e20ba9bbdd4a' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c7204535-53ab-4286-bacb-5ab774660b97' class='xr-var-data-in' type='checkbox'><label for='data-c7204535-53ab-4286-bacb-5ab774660b97' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_instant nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>1mo</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_surface_temperature</dd><dt><span>online_operation :</span></dt><dd>instant</dd><dt><span>units :</span></dt><dd>degC</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>taum</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-2b18a78b-9ae4-4c96-9f5c-b499dc077914' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-2b18a78b-9ae4-4c96-9f5c-b499dc077914' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3b402bcd-9f55-4e6a-8be7-16dbfc055929' class='xr-var-data-in' type='checkbox'><label for='data-3b402bcd-9f55-4e6a-8be7-16dbfc055929' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>wind stress module</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>N/m2</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>tohfls</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-310319dd-e5b7-45e9-b577-e5df89f36b18' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-310319dd-e5b7-45e9-b577-e5df89f36b18' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-fca084e4-434f-4969-b5fa-8c3242a7787a' class='xr-var-data-in' type='checkbox'><label for='data-fca084e4-434f-4969-b5fa-8c3242a7787a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>surface_net_downward_total_heat_flux</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>W/m2</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>tossq</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-9fee6683-926d-4744-8cc1-4e696dfa7138' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-9fee6683-926d-4744-8cc1-4e696dfa7138' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-79e8e7d3-f1ff-448f-8626-b5c10a0ddc0a' class='xr-var-data-in' type='checkbox'><label for='data-79e8e7d3-f1ff-448f-8626-b5c10a0ddc0a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>square_of_sea_surface_temperature</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>degC2</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>wfo</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-4779947e-4812-499e-bd60-174fc737d546' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-4779947e-4812-499e-bd60-174fc737d546' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-07a68209-e332-4e40-8890-194cbb9fb270' class='xr-var-data-in' type='checkbox'><label for='data-07a68209-e332-4e40-8890-194cbb9fb270' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>water_flux_into_sea_water</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>kg/m2/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>zossq</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-b7339721-083c-4085-a837-b982ea537000' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-b7339721-083c-4085-a837-b982ea537000' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3dae21af-6e43-465a-b404-091cc325f222' class='xr-var-data-in' type='checkbox'><label for='data-3dae21af-6e43-465a-b404-091cc325f222' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>square_of_sea_surface_height_above_geoid</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m2</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 2 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>tauuo</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-1f0201d3-0777-4f01-8dd9-c4e738a6c01e' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-1f0201d3-0777-4f01-8dd9-c4e738a6c01e' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-46aa7d6a-1ccc-4dab-9b03-d316efc1406b' class='xr-var-data-in' type='checkbox'><label for='data-46aa7d6a-1ccc-4dab-9b03-d316efc1406b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>surface_downward_x_stress</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>N/m2</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 14 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>uo</span></div><div class='xr-var-dims'>(time_counter, depth, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-cdc3cb83-2c01-4e88-9c3b-3a588391a460' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-cdc3cb83-2c01-4e88-9c3b-3a588391a460' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3224332f-d907-4dd2-a127-b022983d1d81' class='xr-var-data-in' type='checkbox'><label for='data-3224332f-d907-4dd2-a127-b022983d1d81' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered depthu nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_water_x_velocity</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 439.57 GiB </td>
                        <td> 6.35 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 75, 3059, 4322) </td>
                        <td> (1, 5, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 85680 chunks in 17 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="386" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="30" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="30" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="0" y1="0" x2="0" y2="25" />
  <line x1="1" y1="0" x2="1" y2="25" />
  <line x1="2" y1="0" x2="2" y2="25" />
  <line x1="3" y1="0" x2="3" y2="25" />
  <line x1="4" y1="0" x2="4" y2="25" />
  <line x1="5" y1="0" x2="5" y2="25" />
  <line x1="6" y1="0" x2="6" y2="25" />
  <line x1="7" y1="0" x2="7" y2="25" />
  <line x1="8" y1="0" x2="8" y2="25" />
  <line x1="9" y1="0" x2="9" y2="25" />
  <line x1="10" y1="0" x2="10" y2="25" />
  <line x1="11" y1="0" x2="11" y2="25" />
  <line x1="12" y1="0" x2="12" y2="25" />
  <line x1="13" y1="0" x2="13" y2="25" />
  <line x1="14" y1="0" x2="14" y2="25" />
  <line x1="15" y1="0" x2="15" y2="25" />
  <line x1="16" y1="0" x2="16" y2="25" />
  <line x1="17" y1="0" x2="17" y2="25" />
  <line x1="18" y1="0" x2="18" y2="25" />
  <line x1="19" y1="0" x2="19" y2="25" />
  <line x1="20" y1="0" x2="20" y2="25" />
  <line x1="21" y1="0" x2="21" y2="25" />
  <line x1="22" y1="0" x2="22" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="24" y1="0" x2="24" y2="25" />
  <line x1="25" y1="0" x2="25" y2="25" />
  <line x1="26" y1="0" x2="26" y2="25" />
  <line x1="27" y1="0" x2="27" y2="25" />
  <line x1="28" y1="0" x2="28" y2="25" />
  <line x1="29" y1="0" x2="29" y2="25" />
  <line x1="30" y1="0" x2="30" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 30.879479818120135,0.0 30.879479818120135,25.412616514582485 0.0,25.412616514582485" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Text -->
  <text x="15.439740" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >119</text>
  <text x="50.879480" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,50.879480,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="100" y1="16" x2="116" y2="32" />
  <line x1="100" y1="32" x2="116" y2="48" />
  <line x1="100" y1="48" x2="116" y2="64" />
  <line x1="100" y1="64" x2="116" y2="80" />
  <line x1="100" y1="80" x2="116" y2="96" />
  <line x1="100" y1="84" x2="116" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="100" y2="84" style="stroke-width:2" />
  <line x1="101" y1="1" x2="101" y2="86" />
  <line x1="102" y1="2" x2="102" y2="87" />
  <line x1="103" y1="3" x2="103" y2="88" />
  <line x1="104" y1="4" x2="104" y2="89" />
  <line x1="105" y1="5" x2="105" y2="90" />
  <line x1="106" y1="6" x2="106" y2="91" />
  <line x1="107" y1="7" x2="107" y2="92" />
  <line x1="108" y1="8" x2="108" y2="93" />
  <line x1="109" y1="9" x2="109" y2="94" />
  <line x1="111" y1="11" x2="111" y2="96" />
  <line x1="112" y1="12" x2="112" y2="97" />
  <line x1="113" y1="13" x2="113" y2="98" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="115" y1="15" x2="115" y2="100" />
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 116.61338981631184,16.613389816311837 116.61338981631184,101.5462912508329 100.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="220" y2="0" style="stroke-width:2" />
  <line x1="101" y1="1" x2="221" y2="1" />
  <line x1="102" y1="2" x2="222" y2="2" />
  <line x1="103" y1="3" x2="223" y2="3" />
  <line x1="104" y1="4" x2="224" y2="4" />
  <line x1="105" y1="5" x2="225" y2="5" />
  <line x1="106" y1="6" x2="226" y2="6" />
  <line x1="107" y1="7" x2="227" y2="7" />
  <line x1="108" y1="8" x2="228" y2="8" />
  <line x1="109" y1="9" x2="229" y2="9" />
  <line x1="111" y1="11" x2="231" y2="11" />
  <line x1="112" y1="12" x2="232" y2="12" />
  <line x1="113" y1="13" x2="233" y2="13" />
  <line x1="114" y1="14" x2="234" y2="14" />
  <line x1="115" y1="15" x2="235" y2="15" />
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="116" y1="0" x2="132" y2="16" />
  <line x1="132" y1="0" x2="148" y2="16" />
  <line x1="148" y1="0" x2="164" y2="16" />
  <line x1="164" y1="0" x2="180" y2="16" />
  <line x1="180" y1="0" x2="196" y2="16" />
  <line x1="196" y1="0" x2="212" y2="16" />
  <line x1="212" y1="0" x2="228" y2="16" />
  <line x1="220" y1="0" x2="236" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 220.0,0.0 236.61338981631184,16.613389816311837 116.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />
  <line x1="116" y1="32" x2="236" y2="32" />
  <line x1="116" y1="48" x2="236" y2="48" />
  <line x1="116" y1="64" x2="236" y2="64" />
  <line x1="116" y1="80" x2="236" y2="80" />
  <line x1="116" y1="96" x2="236" y2="96" />
  <line x1="116" y1="101" x2="236" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />
  <line x1="132" y1="16" x2="132" y2="101" />
  <line x1="148" y1="16" x2="148" y2="101" />
  <line x1="164" y1="16" x2="164" y2="101" />
  <line x1="180" y1="16" x2="180" y2="101" />
  <line x1="196" y1="16" x2="196" y2="101" />
  <line x1="212" y1="16" x2="212" y2="101" />
  <line x1="228" y1="16" x2="228" y2="101" />
  <line x1="236" y1="16" x2="236" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="116.61338981631184,16.613389816311837 236.61338981631184,16.613389816311837 236.61338981631184,101.5462912508329 116.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="176.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="256.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,256.613390,59.079841)">3059</text>
  <text x="98.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,98.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>uos</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-134a3cdc-cc7c-455a-855a-6dd84480242f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-134a3cdc-cc7c-455a-855a-6dd84480242f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-1158eb7c-91fe-4678-9f52-c68166ad8e55' class='xr-var-data-in' type='checkbox'><label for='data-1158eb7c-91fe-4678-9f52-c68166ad8e55' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_surface_x_velocity</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 14 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>tauvo</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-3cc7218b-efa7-4341-aec6-b5265ec10c6f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-3cc7218b-efa7-4341-aec6-b5265ec10c6f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-47785a66-6106-4946-8484-f1c99ee07c05' class='xr-var-data-in' type='checkbox'><label for='data-47785a66-6106-4946-8484-f1c99ee07c05' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>surface_downward_y_stress</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>N/m2</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 14 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>vo</span></div><div class='xr-var-dims'>(time_counter, depth, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 5, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-d7089385-2185-48c0-a232-69837ea8c368' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-d7089385-2185-48c0-a232-69837ea8c368' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-10c667af-64c7-4f42-9427-f95d77dbbdd9' class='xr-var-data-in' type='checkbox'><label for='data-10c667af-64c7-4f42-9427-f95d77dbbdd9' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered depthv nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_water_y_velocity</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 439.57 GiB </td>
                        <td> 6.35 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 75, 3059, 4322) </td>
                        <td> (1, 5, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 85680 chunks in 17 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="386" height="151" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="30" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="30" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="0" y1="0" x2="0" y2="25" />
  <line x1="1" y1="0" x2="1" y2="25" />
  <line x1="2" y1="0" x2="2" y2="25" />
  <line x1="3" y1="0" x2="3" y2="25" />
  <line x1="4" y1="0" x2="4" y2="25" />
  <line x1="5" y1="0" x2="5" y2="25" />
  <line x1="6" y1="0" x2="6" y2="25" />
  <line x1="7" y1="0" x2="7" y2="25" />
  <line x1="8" y1="0" x2="8" y2="25" />
  <line x1="9" y1="0" x2="9" y2="25" />
  <line x1="10" y1="0" x2="10" y2="25" />
  <line x1="11" y1="0" x2="11" y2="25" />
  <line x1="12" y1="0" x2="12" y2="25" />
  <line x1="13" y1="0" x2="13" y2="25" />
  <line x1="14" y1="0" x2="14" y2="25" />
  <line x1="15" y1="0" x2="15" y2="25" />
  <line x1="16" y1="0" x2="16" y2="25" />
  <line x1="17" y1="0" x2="17" y2="25" />
  <line x1="18" y1="0" x2="18" y2="25" />
  <line x1="19" y1="0" x2="19" y2="25" />
  <line x1="20" y1="0" x2="20" y2="25" />
  <line x1="21" y1="0" x2="21" y2="25" />
  <line x1="22" y1="0" x2="22" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="23" y1="0" x2="23" y2="25" />
  <line x1="24" y1="0" x2="24" y2="25" />
  <line x1="25" y1="0" x2="25" y2="25" />
  <line x1="26" y1="0" x2="26" y2="25" />
  <line x1="27" y1="0" x2="27" y2="25" />
  <line x1="28" y1="0" x2="28" y2="25" />
  <line x1="29" y1="0" x2="29" y2="25" />
  <line x1="30" y1="0" x2="30" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 30.879479818120135,0.0 30.879479818120135,25.412616514582485 0.0,25.412616514582485" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Text -->
  <text x="15.439740" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >119</text>
  <text x="50.879480" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,50.879480,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="100" y1="16" x2="116" y2="32" />
  <line x1="100" y1="32" x2="116" y2="48" />
  <line x1="100" y1="48" x2="116" y2="64" />
  <line x1="100" y1="64" x2="116" y2="80" />
  <line x1="100" y1="80" x2="116" y2="96" />
  <line x1="100" y1="84" x2="116" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="100" y2="84" style="stroke-width:2" />
  <line x1="101" y1="1" x2="101" y2="86" />
  <line x1="102" y1="2" x2="102" y2="87" />
  <line x1="103" y1="3" x2="103" y2="88" />
  <line x1="104" y1="4" x2="104" y2="89" />
  <line x1="105" y1="5" x2="105" y2="90" />
  <line x1="106" y1="6" x2="106" y2="91" />
  <line x1="107" y1="7" x2="107" y2="92" />
  <line x1="108" y1="8" x2="108" y2="93" />
  <line x1="109" y1="9" x2="109" y2="94" />
  <line x1="111" y1="11" x2="111" y2="96" />
  <line x1="112" y1="12" x2="112" y2="97" />
  <line x1="113" y1="13" x2="113" y2="98" />
  <line x1="114" y1="14" x2="114" y2="99" />
  <line x1="115" y1="15" x2="115" y2="100" />
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 116.61338981631184,16.613389816311837 116.61338981631184,101.5462912508329 100.0,84.93290143452106" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="100" y1="0" x2="220" y2="0" style="stroke-width:2" />
  <line x1="101" y1="1" x2="221" y2="1" />
  <line x1="102" y1="2" x2="222" y2="2" />
  <line x1="103" y1="3" x2="223" y2="3" />
  <line x1="104" y1="4" x2="224" y2="4" />
  <line x1="105" y1="5" x2="225" y2="5" />
  <line x1="106" y1="6" x2="226" y2="6" />
  <line x1="107" y1="7" x2="227" y2="7" />
  <line x1="108" y1="8" x2="228" y2="8" />
  <line x1="109" y1="9" x2="229" y2="9" />
  <line x1="111" y1="11" x2="231" y2="11" />
  <line x1="112" y1="12" x2="232" y2="12" />
  <line x1="113" y1="13" x2="233" y2="13" />
  <line x1="114" y1="14" x2="234" y2="14" />
  <line x1="115" y1="15" x2="235" y2="15" />
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="100" y1="0" x2="116" y2="16" style="stroke-width:2" />
  <line x1="116" y1="0" x2="132" y2="16" />
  <line x1="132" y1="0" x2="148" y2="16" />
  <line x1="148" y1="0" x2="164" y2="16" />
  <line x1="164" y1="0" x2="180" y2="16" />
  <line x1="180" y1="0" x2="196" y2="16" />
  <line x1="196" y1="0" x2="212" y2="16" />
  <line x1="212" y1="0" x2="228" y2="16" />
  <line x1="220" y1="0" x2="236" y2="16" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="100.0,0.0 220.0,0.0 236.61338981631184,16.613389816311837 116.61338981631184,16.613389816311837" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="116" y1="16" x2="236" y2="16" style="stroke-width:2" />
  <line x1="116" y1="32" x2="236" y2="32" />
  <line x1="116" y1="48" x2="236" y2="48" />
  <line x1="116" y1="64" x2="236" y2="64" />
  <line x1="116" y1="80" x2="236" y2="80" />
  <line x1="116" y1="96" x2="236" y2="96" />
  <line x1="116" y1="101" x2="236" y2="101" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="116" y1="16" x2="116" y2="101" style="stroke-width:2" />
  <line x1="132" y1="16" x2="132" y2="101" />
  <line x1="148" y1="16" x2="148" y2="101" />
  <line x1="164" y1="16" x2="164" y2="101" />
  <line x1="180" y1="16" x2="180" y2="101" />
  <line x1="196" y1="16" x2="196" y2="101" />
  <line x1="212" y1="16" x2="212" y2="101" />
  <line x1="228" y1="16" x2="228" y2="101" />
  <line x1="236" y1="16" x2="236" y2="101" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="116.61338981631184,16.613389816311837 236.61338981631184,16.613389816311837 236.61338981631184,101.5462912508329 116.61338981631184,101.5462912508329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="176.613390" y="121.546291" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="256.613390" y="59.079841" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,256.613390,59.079841)">3059</text>
  <text x="98.306695" y="113.239596" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,98.306695,113.239596)">75</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>vos</span></div><div class='xr-var-dims'>(time_counter, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 577, 577), meta=np.ndarray&gt;</div><input id='attrs-bf0648c4-8bf5-4121-b6e9-ef8ad9371665' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-bf0648c4-8bf5-4121-b6e9-ef8ad9371665' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-6f3598b7-1994-45c1-b69c-37f5cfbcae28' class='xr-var-data-in' type='checkbox'><label for='data-6f3598b7-1994-45c1-b69c-37f5cfbcae28' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_surface_y_velocity</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m/s</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 5.86 GiB </td>
                        <td> 1.27 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (119, 3059, 4322) </td>
                        <td> (1, 577, 577) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 5712 chunks in 14 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="198" height="153" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="10" y1="16" x2="28" y2="34" />
  <line x1="10" y1="32" x2="28" y2="50" />
  <line x1="10" y1="48" x2="28" y2="66" />
  <line x1="10" y1="64" x2="28" y2="82" />
  <line x1="10" y1="80" x2="28" y2="98" />
  <line x1="10" y1="84" x2="28" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="84" style="stroke-width:2" />
  <line x1="10" y1="0" x2="10" y2="85" />
  <line x1="11" y1="1" x2="11" y2="86" />
  <line x1="12" y1="2" x2="12" y2="87" />
  <line x1="13" y1="3" x2="13" y2="88" />
  <line x1="14" y1="4" x2="14" y2="89" />
  <line x1="15" y1="5" x2="15" y2="90" />
  <line x1="16" y1="6" x2="16" y2="91" />
  <line x1="17" y1="7" x2="17" y2="92" />
  <line x1="18" y1="8" x2="18" y2="93" />
  <line x1="19" y1="9" x2="19" y2="94" />
  <line x1="20" y1="10" x2="20" y2="95" />
  <line x1="21" y1="11" x2="21" y2="96" />
  <line x1="22" y1="12" x2="22" y2="97" />
  <line x1="23" y1="13" x2="23" y2="98" />
  <line x1="24" y1="14" x2="24" y2="99" />
  <line x1="25" y1="15" x2="25" y2="100" />
  <line x1="26" y1="16" x2="26" y2="101" />
  <line x1="27" y1="17" x2="27" y2="102" />
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 28.164399893011844,18.164399893011844 28.164399893011844,103.0973013275329 10.0,84.93290143452106" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="10" y1="0" x2="130" y2="0" />
  <line x1="11" y1="1" x2="131" y2="1" />
  <line x1="12" y1="2" x2="132" y2="2" />
  <line x1="13" y1="3" x2="133" y2="3" />
  <line x1="14" y1="4" x2="134" y2="4" />
  <line x1="15" y1="5" x2="135" y2="5" />
  <line x1="16" y1="6" x2="136" y2="6" />
  <line x1="17" y1="7" x2="137" y2="7" />
  <line x1="18" y1="8" x2="138" y2="8" />
  <line x1="19" y1="9" x2="139" y2="9" />
  <line x1="20" y1="10" x2="140" y2="10" />
  <line x1="21" y1="11" x2="141" y2="11" />
  <line x1="22" y1="12" x2="142" y2="12" />
  <line x1="23" y1="13" x2="143" y2="13" />
  <line x1="24" y1="14" x2="144" y2="14" />
  <line x1="25" y1="15" x2="145" y2="15" />
  <line x1="26" y1="16" x2="146" y2="16" />
  <line x1="27" y1="17" x2="147" y2="17" />
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="28" y2="18" style="stroke-width:2" />
  <line x1="26" y1="0" x2="44" y2="18" />
  <line x1="42" y1="0" x2="60" y2="18" />
  <line x1="58" y1="0" x2="76" y2="18" />
  <line x1="74" y1="0" x2="92" y2="18" />
  <line x1="90" y1="0" x2="108" y2="18" />
  <line x1="106" y1="0" x2="124" y2="18" />
  <line x1="122" y1="0" x2="140" y2="18" />
  <line x1="130" y1="0" x2="148" y2="18" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 148.16439989301185,18.164399893011844 28.164399893011844,18.164399893011844" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="28" y1="18" x2="148" y2="18" style="stroke-width:2" />
  <line x1="28" y1="34" x2="148" y2="34" />
  <line x1="28" y1="50" x2="148" y2="50" />
  <line x1="28" y1="66" x2="148" y2="66" />
  <line x1="28" y1="82" x2="148" y2="82" />
  <line x1="28" y1="98" x2="148" y2="98" />
  <line x1="28" y1="103" x2="148" y2="103" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="28" y1="18" x2="28" y2="103" style="stroke-width:2" />
  <line x1="44" y1="18" x2="44" y2="103" />
  <line x1="60" y1="18" x2="60" y2="103" />
  <line x1="76" y1="18" x2="76" y2="103" />
  <line x1="92" y1="18" x2="92" y2="103" />
  <line x1="108" y1="18" x2="108" y2="103" />
  <line x1="124" y1="18" x2="124" y2="103" />
  <line x1="140" y1="18" x2="140" y2="103" />
  <line x1="148" y1="18" x2="148" y2="103" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="28.164399893011844,18.164399893011844 148.16439989301185,18.164399893011844 148.16439989301185,103.0973013275329 28.164399893011844,103.0973013275329" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="88.164400" y="123.097301" font-size="1.0rem" font-weight="100" text-anchor="middle" >4322</text>
  <text x="168.164400" y="60.630851" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.164400,60.630851)">3059</text>
  <text x="9.082200" y="114.015101" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,9.082200,114.015101)">119</text>
</svg>
        </td>
    </tr>
</table></div></li></ul></div></li><li class='xr-section-item'><input id='section-b4269249-f543-4f6c-b357-35aaf1ad4aa0' class='xr-section-summary-in' type='checkbox'  ><label for='section-b4269249-f543-4f6c-b357-35aaf1ad4aa0' class='xr-section-summary' >Indexes: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-index-name'><div>depth</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-a6fab649-3806-4044-927d-ad16f9a8d796' class='xr-index-data-in' type='checkbox'/><label for='index-a6fab649-3806-4044-927d-ad16f9a8d796' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(Index([0.5057600140571594, 1.5558552742004395, 2.6676816940307617,
       3.8562798500061035,  5.140361309051514,  6.543033599853516,
         8.09251880645752,  9.822750091552734, 11.773679733276367,
        13.99103832244873,  16.52532196044922,  19.42980194091797,
        22.75761604309082, 26.558300018310547, 30.874561309814453,
       35.740203857421875, 41.180023193359375,  47.21189498901367,
        53.85063552856445,  61.11283874511719,  69.02168273925781,
        77.61116027832031,  86.92942810058594,  97.04131317138672,
       108.03028106689453,              120.0, 133.07582092285156,
                147.40625, 163.16445922851562,  180.5499267578125,
        199.7899627685547, 221.14117431640625,         244.890625,
       271.35638427734375, 300.88751220703125,  333.8628234863281,
           370.6884765625,  411.7938537597656,  457.6256103515625,
         508.639892578125,  565.2922973632812,  628.0260009765625,
        697.2586669921875,  773.3682861328125,   856.678955078125,
        947.4478759765625,  1045.854248046875,    1151.9912109375,
       1265.8614501953125,     1387.376953125, 1516.3636474609375,
       1652.5684814453125, 1795.6707763671875, 1945.2955322265625,
        2101.026611328125,  2262.421630859375,  2429.025146484375,
         2600.38037109375,  2776.039306640625,       2955.5703125,
         3138.56494140625,  3324.640869140625,  3513.445556640625,
         3704.65673828125,   3897.98193359375,   4093.15869140625,
         4289.95263671875,   4488.15478515625,    4687.5810546875,
         4888.06982421875,     5089.478515625,   5291.68310546875,
          5494.5751953125,     5698.060546875,    5902.0576171875],
      dtype=&#x27;float32&#x27;, name=&#x27;depth&#x27;))</pre></div></li><li class='xr-var-item'><div class='xr-index-name'><div>time_counter</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-e1f93573-9839-4625-8bc2-38f10870de5e' class='xr-index-data-in' type='checkbox'/><label for='index-e1f93573-9839-4625-8bc2-38f10870de5e' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(DatetimeIndex([&#x27;1960-01-06 12:00:00&#x27;, &#x27;1960-02-06 12:00:00&#x27;,
               &#x27;1960-03-07 12:00:00&#x27;, &#x27;1960-04-06 12:00:00&#x27;,
               &#x27;1960-05-07 00:00:00&#x27;, &#x27;1960-06-06 12:00:00&#x27;,
               &#x27;1960-07-07 00:00:00&#x27;, &#x27;1960-08-06 12:00:00&#x27;,
               &#x27;1960-09-06 12:00:00&#x27;, &#x27;1960-10-07 00:00:00&#x27;,
               ...
               &#x27;1969-03-16 12:00:00&#x27;, &#x27;1969-04-16 00:00:00&#x27;,
               &#x27;1969-05-16 12:00:00&#x27;, &#x27;1969-06-16 00:00:00&#x27;,
               &#x27;1969-07-16 12:00:00&#x27;, &#x27;1969-08-16 12:00:00&#x27;,
               &#x27;1969-09-16 00:00:00&#x27;, &#x27;1969-10-16 12:00:00&#x27;,
               &#x27;1969-11-16 00:00:00&#x27;, &#x27;1969-12-16 12:00:00&#x27;],
              dtype=&#x27;datetime64[ns]&#x27;, name=&#x27;time_counter&#x27;, length=119, freq=None))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-eef3a926-0872-41d3-bd92-697cdf08cab3' class='xr-section-summary-in' type='checkbox'  ><label for='section-eef3a926-0872-41d3-bd92-697cdf08cab3' class='xr-section-summary' >Attributes: <span>(11)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>DOMAIN_number_total :</span></dt><dd>80</dd><dt><span>DOMAIN_size_global :</span></dt><dd>[4322, 3059]</dd><dt><span>conventions :</span></dt><dd>CF-1.1</dd><dt><span>description :</span></dt><dd>ocean T grid variables</dd><dt><span>ibegin :</span></dt><dd>1</dd><dt><span>jbegin :</span></dt><dd>1</dd><dt><span>name :</span></dt><dd>ORCA0083-N06_1m_19591222_19601231</dd><dt><span>ni :</span></dt><dd>4322</dd><dt><span>nj :</span></dt><dd>39</dd><dt><span>production :</span></dt><dd>An IPSL model</dd><dt><span>timeStamp :</span></dt><dd>2014-Dec-03 05:19:35 GMT</dd></dl></div></li></ul></div></div>



### Slice the zarr files

Because zarr files are optimized for cloud, when we instantiate an xarray dataset, we do not open the zarr by it self. We only open some metadata related to the file. The files will only be downloaed when we need to perform some processing on the data


```python
dom = dom.isel(y=slice(500, 700), x=slice(1000, 1200))
```


```python
t_grid = t_grid.isel(y=slice(500, 700), x=slice(1000, 1200), time_counter=slice(0, 24))
```

### Loading and Interrogating

We can create a new Gridded object by simple calling `coast.Gridded()`. By passing this a NEMO data file and a NEMO domain file, COAsT will combine the two into a single xarray dataset within the Gridded object. Each individual Gridded object should be for a specified NEMO grid type, which is specified in a configuration file which is also passed as an argument. The Dask library is switched on by default, chunking can be specified in the configuration file.


```python
nemo_t = coast.Gridded(fn_data=t_grid, fn_domain=dom, config=fn_config_t_grid)
```

    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/coast/data/gridded.py:237: UserWarning: The model domain loaded, '<xarray.Dataset>
    Dimensions:       (t: 1, z: 75, y: 200, x: 200)
    Dimensions without coordinates: t, z, y, x
    Data variables: (12/42)
        fmask         (t, z, y, x) int8 dask.array<chunksize=(1, 10, 200, 82), meta=np.ndarray>
        fmaskutil     (t, y, x) int8 dask.array<chunksize=(1, 200, 81), meta=np.ndarray>
        nav_lat       (y, x) float32 dask.array<chunksize=(200, 82), meta=np.ndarray>
        nav_lev       (z) float32 dask.array<chunksize=(75,), meta=np.ndarray>
        nav_lon       (y, x) float32 dask.array<chunksize=(200, 82), meta=np.ndarray>
        time_counter  (t) float64 dask.array<chunksize=(1,), meta=np.ndarray>
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
    coordinates:         time_centered nav_lon nav_lat
    interval_operation:  300s
    interval_write:      1mo
    long_name:           sea_surface_height_above_geoid
    online_operation:    average
    units:               m</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'ssh'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>t_dim</span>: 24</li><li><span>y_dim</span>: 200</li><li><span>x_dim</span>: 200</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-04ea5b62-562a-4f62-a14d-34e69a9f566c' class='xr-array-in' type='checkbox' checked><label for='section-04ea5b62-562a-4f62-a14d-34e69a9f566c' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>-1.621 -1.622 -1.623 -1.625 -1.628 ... -0.6946 -0.7192 -0.7428 -0.7652</span></div><div class='xr-array-data'><pre>array([[[-1.6210064 , -1.6217471 , -1.6231693 , ..., -1.5842544 ,
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
         -0.7428356 , -0.7651531 ]]], dtype=float32)</pre></div></div></li><li class='xr-section-item'><input id='section-6a0be372-2268-4d41-9874-7e15fdd10d94' class='xr-section-summary-in' type='checkbox'  checked><label for='section-6a0be372-2268-4d41-9874-7e15fdd10d94' class='xr-section-summary' >Coordinates: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(t_dim)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>1960-01-06T12:00:00 ... 1961-12-...</div><input id='attrs-a648d888-ef24-4df7-a1f1-b36bb7e7e405' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-a648d888-ef24-4df7-a1f1-b36bb7e7e405' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-95ddc456-00de-45ce-b6e9-46aabd80dafe' class='xr-var-data-in' type='checkbox'><label for='data-95ddc456-00de-45ce-b6e9-46aabd80dafe' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>T</dd><dt><span>long_name :</span></dt><dd>Time axis</dd><dt><span>standard_name :</span></dt><dd>time</dd><dt><span>time_origin :</span></dt><dd>1950-01-01 00:00:00</dd><dt><span>title :</span></dt><dd>Time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;1960-01-06T12:00:00.000000000&#x27;, &#x27;1960-02-06T12:00:00.000000000&#x27;,
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
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>longitude</span></div><div class='xr-var-dims'>(y_dim, x_dim)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>156.2 156.3 156.4 ... 172.8 172.8</div><input id='attrs-f6f9f688-be21-4c6e-84ea-5006ad0da8cf' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-f6f9f688-be21-4c6e-84ea-5006ad0da8cf' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-0051e622-5965-471f-a19d-089249b1307d' class='xr-var-data-in' type='checkbox'><label for='data-0051e622-5965-471f-a19d-089249b1307d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[156.25   , 156.33333, 156.41667, ..., 172.66667, 172.75   ,
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
        172.83333]], dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>latitude</span></div><div class='xr-var-dims'>(y_dim, x_dim)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>-63.49 -63.49 ... -55.07 -55.07</div><input id='attrs-8260da65-beac-4dda-9cb3-d3895acb106f' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-8260da65-beac-4dda-9cb3-d3895acb106f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c037c1ca-5ef0-4386-ba94-09639be6b4d4' class='xr-var-data-in' type='checkbox'><label for='data-c037c1ca-5ef0-4386-ba94-09639be6b4d4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[-63.48817 , -63.48817 , -63.48817 , ..., -63.48817 , -63.48817 ,
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
        -55.067184]], dtype=float32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-81c6f63e-50de-404b-9554-765ad0d9ca90' class='xr-section-summary-in' type='checkbox'  ><label for='section-81c6f63e-50de-404b-9554-765ad0d9ca90' class='xr-section-summary' >Indexes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-index-name'><div>time</div></div><div class='xr-index-preview'>PandasIndex</div><div></div><input id='index-fa733e73-86a0-491c-89fb-524b5bbe0808' class='xr-index-data-in' type='checkbox'/><label for='index-fa733e73-86a0-491c-89fb-524b5bbe0808' title='Show/Hide index repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-index-data'><pre>PandasIndex(DatetimeIndex([&#x27;1960-01-06 12:00:00&#x27;, &#x27;1960-02-06 12:00:00&#x27;,
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
              dtype=&#x27;datetime64[ns]&#x27;, name=&#x27;time&#x27;, freq=None))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-b5586c52-7205-4fd5-8593-34a7c802119a' class='xr-section-summary-in' type='checkbox'  checked><label for='section-b5586c52-7205-4fd5-8593-34a7c802119a' class='xr-section-summary' >Attributes: <span>(6)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>coordinates :</span></dt><dd>time_centered nav_lon nav_lat</dd><dt><span>interval_operation :</span></dt><dd>300s</dd><dt><span>interval_write :</span></dt><dd>1mo</dd><dt><span>long_name :</span></dt><dd>sea_surface_height_above_geoid</dd><dt><span>online_operation :</span></dt><dd>average</dd><dt><span>units :</span></dt><dd>m</dd></dl></div></li></ul></div></div>



Or as a numpy array:


```python
ssh_np = ssh.values
# ssh_np.shape # uncomment to print data object summary
```

Then lets plot up a single time snapshot of ssh using matplotlib:


```python
plt.pcolormesh(nemo_t.dataset.longitude, nemo_t.dataset.latitude, nemo_t.dataset.ssh[0])
```

    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/IPython/core/pylabtools.py:77: DeprecationWarning: backend2gui is deprecated since IPython 8.24, backends are managed in matplotlib and can be externally registered.
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/IPython/core/pylabtools.py:77: DeprecationWarning: backend2gui is deprecated since IPython 8.24, backends are managed in matplotlib and can be externally registered.
    /usr/share/miniconda/envs/coast/lib/python3.10/site-packages/IPython/core/pylabtools.py:77: DeprecationWarning: backend2gui is deprecated since IPython 8.24, backends are managed in matplotlib and can be externally registered.





    <matplotlib.collections.QuadMesh at 0x7f1b78eca8f0>




    
![png](/COAsT/zarr_files_files/zarr_files_33_2.png)
    



```python

```
