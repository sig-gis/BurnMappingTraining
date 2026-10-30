---
layout: page
title: "3. Burn Mapping with Optical Imagery"
permalink: /03_BurnMapping_Optical
nav_order: 3
---

## Open the Script
Find the **users/ee-scripts/PC642-SERVIR-BPEmbeddings/part1/burnScar_optical** script.

You can also use the quick link: https://code.earthengine.google.com/29baed42e7a78813a3af034f1f3a8ed5. 

If you make any changes to the script which you would like to save and easily access again, you can use `Save` (Save As) to save a version of the script to your own repository.

## 1. Purpose of This Script

### High-Level Objective
This script provides an automated, end-to-end Earth Observation workflow in Google Earth Engine (GEE) to rapidly identify burned areas and quantify fire severity across custom geographic regions. By leveraging multi-temporal optical imagery from Sentinel-2 and high-resolution land cover data from Google's Dynamic World model, the script isolates genuine fire impacts while filtering out atmospheric contamination, non-vegetated surfaces, and minor spatial noise.

### What Is Included in the Script
* **Flexible Area of Interest (AOI) Selection**: Choose between administrative boundaries from the FAO GAUL dataset (at country or province/state levels) or define custom boundaries using GEE hand-drawn map geometry.
* **Automated Tiling Grid**: Automatically divides large study areas into customizable square spatial grid cells to optimize memory usage, prevent compute timeouts, and enable tiled exporting.
* **Pre- and Post-Fire Compositing**: Automatically filters Sentinel-2 Level-2A imagery across baseline pre-fire and post-fire time windows, combining standard `QA60` bitmasking with Google Cloud Score+ (`CS+`) machine learning quality scores to eliminate cloud and shadow artifacts.
* **Spectral Difference Analysis (dNBR & dBAI)**: Computes pre- and post-fire Normalized Burn Ratio (NBR) and Burned Area Index (BAI) rasters, producing Delta NBR (dNBR) for standardized severity classification and Delta BAI (dBAI) for fine-scale charcoal/ash deposition mapping.
* **Dynamic World Land Cover Masking**: Integrates near-real-time Dynamic World land cover composites to mask out water bodies, urban/built land, bare soil, and snow/ice, isolating burn scar detection to relevant vegetation types.
* **Post-Processing & Minimum Area Filtering**: Converts classified burn rasters into vector polygons, calculates polygon surface areas, and removes small isolated noise fragments below a user-defined square-meter threshold.
* **Interactive Map Visualization & Side-by-Side Legends**: Renders multi-layer map displays with custom color palettes alongside dedicated, auto-stacked map UI panels for USGS Burn Severity and Dynamic World Land Cover.
* **Batch Data Exporting**: Generates ready-to-run Google Earth Engine export tasks for both raster GeoTIFF files and vector shapefiles (`.shp`) directed to Google Drive or Earth Engine Assets.

---

## 2. How to Use This Script

Executing the script involves three simple steps: configuring your input parameters at the top of the code, running the script to inspect the interactive map layers, and initiating your exports from the Earth Engine Tasks panel.

### Step 1: Configuring User Parameters
All user-adjustable settings are consolidated in the **USER CONFIGURATION** block at the top of the script (lines 35–95). Modify these variables to match your specific fire event and study region:

#### 1. Fire Dates & Baseline Windows
Set the calendar year and approximate date range of the active fire event, as well as the duration of the pre- and post-fire observation windows:
```javascript
var year = '2024';             // Calendar year of the fire event
var fireStartString = '01-30'; // Fire start date in 'MM-DD' format
var fireEndString = '03-30';   // Fire end date in 'MM-DD' format
var windowDays = 30;           // Duration of pre-and post-fire baseline windows (days)
```
*Note: The script automatically calculates the pre-fire window (`fireStart` minus `windowDays`) and post-fire window (`fireEnd` plus `windowDays`).*

#### 2. Area of Interest (AOI) Selection
Specify the target geographic region using FAO GAUL administrative names:
```javascript
var countryName = "Lao People's Democratic Republic"; // Country name (Admin 0)
var admin1Name = "Bokeo";                            // Province/State name (Admin 1)
```
In the script body under **AOI SELECTION**, uncomment **one** of the three AOI options:
* **Option 1 (Default)**: Filters by province/state (`admin1Name`).
* **Option 2**: Filters by entire country boundary (`countryName`).
* **Option 3**: Uses a polygon hand-drawn on the map interface with GEE drawing tools (named `geometry`).

#### 3. Export Grid Scale
Set the grid cell side-length (in meters) used to divide large study areas:
```javascript
var gridScale = 140000; // Grid cell size in meters (140,000 m = 140 km)
```
*Tip: For small provinces or drawn polygons, a single grid cell will cover the area. For large states or whole countries, adjust `gridScale` so the study area is covered by a manageable number of grid tiles (e.g., 2–6 tiles).*

#### 4. Cloud Masking Thresholds
Adjust the strictness of the Cloud Score+ quality filter:
```javascript
var csQualityThreshold = 0.60;  // Pixel quality threshold (0.50 to 0.65 recommended)
var cloudPercentThreshold = 30; // Scene-wide cloud filter threshold (if Method 1 is enabled)
```
*Higher values (e.g., `0.65`) remove thin haze and cirrus shadows more aggressively, while lower values retain more pixels in persistently cloudy tropical regions.*

#### 5. Dynamic World Land Cover Toggles
Toggle land cover classes to include (`true`) or exclude (`false`) from burn mapping:
```javascript
var includeWater      = false; // Class 0 : Water (Excluded by default)
var includeTrees      = true;  // Class 1 : Trees
var includeGrass      = true;  // Class 2 : Grass
var includeFloodedVeg = true;  // Class 3 : Flooded Vegetation
var includeCrops      = true;  // Class 4 : Crops
var includeShrub      = true;  // Class 5 : Shrub & Scrub
var includeBuilt      = false; // Class 6 : Built (Excluded by default)
var includeBare       = false; // Class 7 : Bare (Excluded by default)
var includeSnowIce    = false; // Class 8 : Snow & Ice (Excluded by default)
```

#### 6. Minimum Burn Scar Area Filter
Define the smallest burn scar polygon size to retain in the final output:
```javascript
var minArea = 5000; // Minimum area threshold in square meters (5,000 m² = 0.5 hectares)
```
*Polygons smaller than `minArea` are filtered out as spatial noise.*

#### 7. Export Asset Directory (Optional)
If exporting directly to Earth Engine Assets rather than Google Drive, specify your GEE username directory:
```javascript
var user_asset_path_start = 'users/YOUR_USERNAME/';
```

---

### Step 2: Running the Script & Examining Map Results
Click the **Run** button at the top of the Google Earth Engine Code Editor.

#### Inspecting the Console Tab
Look at the **Console** tab on the right panel to view:
1. **Calculated Date Ranges**: Verify the exact start and end dates for pre-fire and post-fire observation windows.
2. **Available Boundary Names**: Lists all valid country names (`ADM0_NAME`) and province names (`ADM1_NAME`) for spelling verification.
3. **Grid Summary**: Shows the total number of grid tiles generated across your AOI.

#### Interacting with Map Layers & Legends
Use the **Layers** control in the top-right corner of the map to toggle and inspect output rasters:
* **AOI Boundary & Grid**: Displays the study area outline and spatial tiling grid.
* **Dynamic World Land Cover**: Displays modal land cover classes across the observation period alongside the **Dynamic World Legend** (bottom-right).
* **Allowed Land Cover Mask**: Highlights all vegetated land cover types selected for burn analysis in green.
* **dNBR (Burn Severity)**: Shows classified USGS burn severity levels (Unburned, Low, Moderate, High Severity) with the **Burn Severity Legend** (bottom-left).
* **dBAI (Charred Ground Info)**: Renders relative charcoal and ash deposition using the *Charred Earth* color palette (dark charcoal, rust, copper, ochre, stone gray).
* **Burn Scars - LC Masked**: Shows detected burn scars filtered by land cover.
* **Burn Scars - Size Masked**: Shows final refined burn scars with small noise fragments below `minArea` removed.

---

### Step 3: Downloading & Exporting Results
Once you have inspected the map layers and confirmed your parameter settings, export the raster and vector datasets:

1. Click the **Tasks** tab in the top-right panel of the Earth Engine interface.
2. You will see generated export tasks for each grid cell:
   * **`BurnScars_[AOI]_[Dates]_raster_gridX`**: Tiff raster file of refined burn scars.
   * **`BurnScarsFC_[AOI]_[Dates]_vector_gridX`**: ESRI Shapefile (`.shp`) of vectorized burn scar polygons.
3. Click the **Run** button next to any task.
4. In the pop-up dialog, select your target **Google Drive** folder or **GEE Asset** location, verify the projection and scale (default is 10 meters for Sentinel-2), and click **GO**.
5. Once processing completes (indicated by a green checkmark), navigate to your Google Drive to download the output files for use in QGIS, ArcGIS, or analytical reports.
