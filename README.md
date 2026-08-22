# Buying Point Receipts Map

An interactive map of buying points, crop receipts, and totals by catchment / hub / region / country — with a real choropleth layer where boundary polygons are available, and satellite or OpenStreetMap basemaps.

## Files

```
index.html
data/
  receipts.csv
  boundaries/
    country.geojson
    hub.geojson
```

- `index.html` — the map. No build step, no dependencies to install.
- `data/receipts.csv` — sample data. One row per buying point + crop. Replace with your real data (same columns): `name, lat, lon, country, region, hub, catchment, crop, qty`
- `data/boundaries/*.geojson` — **placeholder polygons only**, not real geography. Right now only `country` and `hub` have boundary files — `region` and `catchment` will show as bubble markers instead (still grouped and totaled correctly, just not shaded shapes). Drop a `region.geojson` or `catchment.geojson` into the same folder later if you want those shaded too — no code changes needed, the map picks up any level file that's present. Each polygon needs a `properties.name` that matches the corresponding value in `receipts.csv` exactly.

## Converting from GeoPackage (.gpkg)

Browsers can't read GeoPackage files directly — it's a SQLite-based format, not a web format. Convert each layer to GeoJSON once (or re-run whenever the source updates):

**Using GDAL's `ogr2ogr`** (command line, if you have GDAL installed):
```
ogr2ogr -f GeoJSON -t_srs EPSG:4326 data/boundaries/hub.geojson boundaries.gpkg hub_layer_name
ogr2ogr -f GeoJSON -t_srs EPSG:4326 data/boundaries/country.geojson boundaries.gpkg country_layer_name
```
(`-t_srs EPSG:4326` reprojects to plain lat/lon, which is what Leaflet expects — include it even if you think your data's already in that projection.)

**Using Python + geopandas**, if you don't have GDAL's command-line tools set up separately:
```python
import geopandas as gpd

for layer, out in [("hub_layer_name", "hub"), ("country_layer_name", "country")]:
    gdf = gpd.read_file("boundaries.gpkg", layer=layer).to_crs(epsg=4326)
    gdf.to_file(f"data/boundaries/{out}.geojson", driver="GeoJSON")
```
List your GeoPackage's actual layer names first with `gpd.list_layers("boundaries.gpkg")` (or `ogrinfo boundaries.gpkg` from the command line) if you're not sure what they're called.

Rename whichever field holds each polygon's name to exactly `name` if it isn't already (e.g. `gdf = gdf.rename(columns={"HUB_NAME": "name"})` before saving) — that's the field the map matches against your receipts.

## Run it locally

The filenames are fixed in the code (`data/receipts.csv`, `data/boundaries/hub.geojson`, etc.) — there's no picker or upload button. Because browsers block a plain HTML file from reading other local files for security reasons, `index.html` **must be served over `http://`** for this to work — double-clicking it directly won't load anything.

**Quick local server** (from inside this folder):
```
python -m http.server
```
Then open `http://localhost:8000` in your browser.

GitHub Pages (below) serves it the same way, so it works there without any extra setup.

## Publish it with GitHub Pages

1. Create a new GitHub repository and push this whole folder to it (`index.html` and `data/` at the root, or inside a `/docs` folder — either works).
2. In the repo: **Settings → Pages → Source** → choose your branch and the folder you used → **Save**.
3. GitHub gives you a live URL, usually `https://yourusername.github.io/repo-name/`, within a minute or two.
4. Share that link — anyone who opens it sees the live map, auto-loading straight from `data/receipts.csv` and `data/boundaries/*.geojson` in your repo.

## Updating the data later

Just replace the files in `data/` and push. The map re-reads them fresh every time someone opens the page — no rebuilding, no redeploying.

## Boundary name matching

If a region/hub/catchment/country shows as a bubble instead of a shaded polygon, its name in `receipts.csv` almost certainly doesn't exactly match the `name` property of the corresponding feature in that level's boundary file (check spelling, spacing, and capitalization). Points without a matching polygon automatically fall back to a bubble marker sized by total, so nothing goes missing — it just isn't shaded as a shape yet.

