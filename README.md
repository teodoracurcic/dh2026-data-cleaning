# Turning Raw Data into Reliable Sources

This notebook was built for a [Dataharvest](https://dataharvest.eu/) workshop, but it works just as well as a standalone reference for anyone who needs to take messy government data and turn it into something mappable and story-ready.

The two datasets used here are both publicly available Serbian open data sources, but the techniques apply to almost any similar data: accident records, facility registries, administrative spreadsheets in a non-Latin script. If you have something like that, this workflow will get you most of the way there.

---

## What's in the notebook

### 0. Installing & importing libraries

Sets up the environment. All the libraries you need are listed here, with a single `pip install` line you can uncomment and run. Nothing unusual is required beyond standard data science tools plus `geopandas`, `geopy`, and `transliterate` for the geo and script-conversion steps.

---

### 1. Traffic accidents data

**Source: [Serbian police via data.gov.rs](https://data.gov.rs/sr/datasets/podatsi-o-saobratshajnim-nezgodama-po-politsijskim-upravama-i-opshtinama/)**

The source data comes as multiple `.xlsx` files, one per year or police district. The notebook reads all of them from a folder at once and concatenates them into a single DataFrame, which is a common pattern when you get data in annual batches.

From there the cleaning covers a few things that come up constantly with government data:

- **Column naming**: the original files have no header row, so columns get assigned manually
- **Date parsing**: a combined `date_time` string gets split into separate `date` and `time` columns and converted to proper datetime types, so you can later filter and group by year, month, or hour
- **Category translation**: accident status and type fields are in Serbian and get mapped to English equivalents; the notebook checks `.unique()` first before replacing, which is a good habit to catch unexpected variants or typos
- **Text normalization**: location fields are lowercased to avoid mismatches when filtering

The analysis section does some quick frequency checks, municipality breakdowns, and filters down to Belgrade-area accidents, which feeds into the geo export at the end.

---

### 2. School locations data

**Source: [Ministry of Education open data](https://opendata.mpn.gov.rs)**

This dataset starts in Serbian Cyrillic, which is the first thing to deal with. The `transliterate` library converts every cell from Cyrillic to Latin script in one pass using a lambda applied across the whole DataFrame.

After that the steps are similar to section 1: drop the many columns that aren't needed for a map (room counts, surface areas, administrative subdivisions), rename what's left to English, normalize text case, and translate the private/public category field.

The more interesting part here is **geocoding**. The schools data has addresses but no coordinates, so the notebook uses `geopy`'s Nominatim geocoder (powered by OpenStreetMap, no API key needed) to look up latitude and longitude for each address. It filters to Novi Sad first to keep the request count reasonable. A `time.sleep(1)` between requests is mandatory because Nominatim's usage policy caps requests at one per second. Addresses that can't be found get `None` stored instead of crashing the loop, and you can inspect those misses afterward.

---

### 3. Geo DataFrames and GeoJSON export

Both cleaned DataFrames get converted into `GeoDataFrame` objects using `geopandas`, with point geometry built from their latitude and longitude columns. The coordinate reference system is set to `EPSG:4326` (standard WGS84, the same system used by GPS and most web maps).

Both are then exported as `.geojson` files, which is the format you want if you're going to load the data into QGIS, Datawrapper, Flourish, or any other mapping tool.

---

## What this is useful for

- Cleaning and merging multi-file government datasets
- Parsing non-standard date and category fields from official sources
- Working with data originally in Cyrillic script
- Geocoding a list of addresses without an API key
- Producing map-ready GeoJSON output from tabular data

---

## Requirements

```
pandas
numpy
openpyxl
geopandas
transliterate
geopy
```

`pathlib` and `time` are part of the Python standard library and don't need to be installed separately.

---

Made by [Teodora Curcic](https://www.linkedin.com/in/teodora-curcic-27a93884/)  
