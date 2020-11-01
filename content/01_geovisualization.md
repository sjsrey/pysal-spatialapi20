---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.5.1
  kernelspec:
    display_name: Python [conda env:pysal-workshop]
    language: python
    name: conda-env-pysal-workshop-py
---

<!-- #region -->
# Geovisualization with PySAL



## Introduction

When the [Python Spatial Analysis Library](https://github.com/pysal), `PySAL`, was originally planned, the intention was to focus on the computational aspects of exploratory spatial data analysis and spatial econometric methods, while relying on existing GIS packages and visualization libraries for visualization of computations. Indeed, we have partnered with [esri](https://geodacenter.asu.edu/arc_pysal) and [QGIS](http://planet.qgis.org/planet/tag/pysal/ ) towards this end.

However, over time we have received many requests for supporting basic geovisualization within PySAL so that the step of having to interoperate with an exertnal package can be avoided, thereby increasing the efficiency of the spatial analytical workflow.



In this notebook, we demonstrate several approaches towards a particular subset of geovisualization methods, namely **choropleth maps**. We start with a exploratory workflow introducing mapclassify and geopandas to create different choropleth classifications and maps for quick exploratory data analysis. We then introduce the **geoviews** package for interactive mapping in support of exploratory spatial data analysis.



<!-- #endregion -->

### PySAL Map Classifiers


```python
import mapclassify
import geopandas as gpd
import matplotlib.pyplot as plt
import contextily as cx
```

```python
shp_link = 'data/scag_region.gpkg'
gdf = gpd.read_file(shp_link)
```

```python
gdf = gdf.to_crs(3857)
```

```python
f, ax = plt.subplots(1, figsize=(12, 8))
gdf.plot(column='median_home_value', scheme='QUANTILES', ax=ax,
        edgecolor='white', legend=True, linewidth=0.3)
ax.set_axis_off()
plt.show()
```

As a first cut, `geopandas` makes it very easy to plot a map quickly. If you know the area well, this may do fine for quick explorataion. If you *don't* know a place extremely well (or you want to make a figure easy to understand for those who don't) it's often a good idea to add a basemap for context. We can do that easily using the `contextily` package

```python

```

```python
gdf.fillna(gdf.mean(), inplace = True) 
  
```

```python
f, ax = plt.subplots(1, figsize=(12, 8))
gdf.plot(column='median_home_value', scheme='QUANTILES', alpha=0.6, ax=ax, legend=True)
cx.add_basemap(ax, crs=gdf.crs.to_string(), source=cx.providers.Stamen.TonerLite)
ax.set_axis_off()
plt.show()
```

```python
hv = gdf['median_home_value']
```

```python
mapclassify.Quantiles(hv, k=5)
```

```python
mapclassify.Quantiles(hv, k=10)
```

```python
q10 = mapclassify.Quantiles(hv, k=10)
```

```python
dir(q10)
```

```python
q10.bins
```

```python
q10.counts
```

```python
fj10 = mapclassify.FisherJenks(hv, k=10)
```

```python
fj10
```

```python
fj10.adcm
```

```python
q10.adcm
```

```python
bins = [100000, 500000, 1000000, 1500000]
```

```python
ud4 = mapclassify.UserDefined(hv, bins=bins)
ud4
```

## GeoPandas: Choropleths

```python
f, ax = plt.subplots(1, figsize=(12, 8))
gdf.plot(column='median_home_value', scheme='QUANTILES', ax=ax, alpha=0.6, legend=True)

cx.add_basemap(ax, crs=gdf.crs.to_string(), source=cx.providers.Stamen.TonerLite)
ax.set_axis_off()
plt.show()
```

```python
f, ax = plt.subplots(1, figsize=(12, 8))
gdf.plot(column='median_home_value', scheme='FisherJenks', ax=ax,
        alpha=0.6, legend=True)

cx.add_basemap(ax, crs=gdf.crs.to_string(), source=cx.providers.Stamen.TonerLite)

ax.set_axis_off()
plt.show()
```

```python
f, ax = plt.subplots(1, figsize=(12, 8))
gdf.plot(column='median_home_value', scheme='FisherJenks', ax=ax,
        alpha=0.6, legend=True, cmap='Blues')

cx.add_basemap(ax, crs=gdf.crs.to_string(), source=cx.providers.Stamen.TonerLite)

ax.set_axis_off()
plt.show()
```

```python
f, ax = plt.subplots(1, figsize=(12, 8))
gdf.plot(column='median_home_value', scheme='FisherJenks', ax=ax,
        alpha=0.6, legend=True, cmap='Blues')
cx.add_basemap(ax, crs=gdf.crs.to_string(), source=cx.providers.Stamen.TonerLite)

ax.set_axis_off()
plt.show()
```

```python
f, ax = plt.subplots(1, figsize=(12, 8))
gdf.plot(column='median_home_value', scheme='FisherJenks', ax=ax,
        alpha=0.6, legend=True, cmap='Blues')
cx.add_basemap(ax, crs=gdf.crs.to_string(), source=cx.providers.Stamen.TonerLite)

ax.set_axis_off()
plt.show()
```

```python
gdf.geoid
```

```python
import numpy
```

```python
counties = set([geoid[:5] for geoid in gdf.geoid])
```

```python
counties
```

```python
for county in counties:
    cgdf = gdf[gdf['geoid'].str.match(f'^{county}')]
    f, ax = plt.subplots(1, figsize=(12, 8))
    cgdf.plot(column='median_home_value', scheme='FisherJenks', ax=ax,
        edgecolor='grey', legend=True, cmap='Blues', alpha=0.6)
    plt.title(county)
    ax.set_axis_off()
    plt.show()

```

## Geoviews

```python
import geoviews as gv
import geoviews.feature as gf
import xarray as xr
from cartopy import crs
import hvplot.pandas
```

```python
import geopandas as gpd
cities = gpd.read_file(gpd.datasets.get_path('naturalearth_cities'))

cities.hvplot(global_extent=True, frame_height=450, tiles=True)
```

```python
shp_link = 'data/scag_region.gpkg'
gdf = gpd.read_file(shp_link)
gdf.fillna(gdf.mean(), inplace = True)
```

```python
gdf.hvplot()
```

```python
gdf.hvplot(geo=True)
```

```python
def choro(df, field, scheme='Quantiles', ncolors=5, cmap='Blues',
          width=600, height=400, tools=['hover']):
    from holoviews.plotting.util import process_cmap
    cl = getattr(mapclassify, scheme)
    yb = cl(df[field], k=ncolors).yb
    pcmap = process_cmap(cmap, ncolors=ncolors)
    #yb, pcmap = choro(values, scheme=scheme, ncolors=ncolors, cmap=cmap)
    df[scheme] = yb
    return gv.Polygons(df, vdims=[scheme, field]).opts(cmap=pcmap, width=width, height=height, tools=tools, line_width=0.1)

    
```

```python
choro(gdf, 'median_home_value')
```

```python
choro(gdf, 'median_home_value', ncolors=9, cmap='Greens')
```

```python
(gv.tile_sources.CartoLight * choro(gdf, 'median_home_value', scheme='FisherJenks', ncolors=9, cmap='Greens'))
```

<div class="alert alert-success" style="font-size:120%">
<b>Exercise</b>: <br>
Create choropleth maps for each of the counties that use the FisherJenks classification defined on the entire Southern California region.
</div>

```python
mapclassify.Pooled?
```

```python
for county in counties:
    cgdf = gdf[gdf['geoid'].str.match(f'^{county}')]
    f, ax = plt.subplots(1, figsize=(12, 8))
    cgdf.plot(column='median_home_value', scheme='FisherJenks', ax=ax,
        edgecolor='grey', legend=True, cmap='Blues', alpha=0.6)
    plt.title(county)
    ax.set_axis_off()
    plt.show()
    

```

---

<a rel="license" href="http://creativecommons.org/licenses/by-nc-
sa/4.0/"><img alt="Creative Commons License" style="border-width:0"
src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span
xmlns:dct="http://purl.org/dc/terms/" property="dct:title">Geovisualization</span> by <a xmlns:cc="http://creativecommons.org/ns#"
href="http://sergerey.org" property="cc:attributionName"
rel="cc:attributionURL">Serge Rey</a> is licensed under a <a
rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative
Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
