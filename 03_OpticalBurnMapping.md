---
layout: page
title: "3. Burn Mapping with Optical Imagery"
permalink: /03_BurnMapping_Optical
nav_order: 3
---

# Optical Imagery Burn Scar & Fire Severity Mapping

---

## 1. Purpose of This Script

### High-Level Objective
This script provides an automated Earth Observation workflow in Google Earth Engine (GEE) to rapidly map burned areas and assess fire severity across custom geographic regions using **optical satellite imagery**. By leveraging passive multispectral sensors (Sentinel-2) alongside Dynamic World land cover data, the script isolates real fire impacts while filtering out cloud cover, non-burnable terrain, and small spatial noise.

*Note: This module represents the **Optical Imagery** approach to burn scar mapping. Optical sensors rely on visible (Red, Green, Blue), Near-Infrared (NIR), and Shortwave Infrared (SWIR) light to measure canopy greenness, moisture loss, and charcoal deposition.*

### What Is Included in the Script
* **Flexible Area of Interest (AOI) Selection**: Define study boundaries using FAO GAUL administrative units (Country or Province/State level) or a hand-drawn map polygon.
* **Automated Tiling Grid**: Splits large study areas into customizable spatial grid tiles to optimize processing speed and enable batch exports.
* **Pre- and Post-Fire Optical Compositing**: Filters Sentinel-2 optical scenes across customizable pre- and post-fire baseline windows and applies cloud masking.
* **Spectral Index Change Analysis**: Calculates Normalized Burn Ratio (NBR) and Burned Area Index (BAI) to derive Delta NBR (dNBR) for severity classification and Delta BAI (dBAI) for relative charcoal/ash detection.
* **Dynamic World Land Cover Masking**: Integrates Dynamic World land cover data to filter out non-burnable land covers (water, built land, bare ground, snow/ice).
* **Minimum Fire Size Filtering**: Converts burn scar rasters into vector polygons and filters out small, isolated detections below a user-defined minimum spatial area.
* **Interactive Visualization & Side-by-Side Legends**: Displays multi-layer maps alongside dedicated UI panels for USGS Burn Severity and Dynamic World Land Cover.
* **Data Exporting**: Exports refined raster GeoTIFFs and vector shapefiles (`.shp`) directly to Google Drive.

---

## 2. General Analysis Overview

The burn mapping workflow follows a sequential optical remote sensing methodology to transform raw satellite observations into refined burn scar vectors and severity maps.
<p align="center">
  <img src="{{ site.baseurl }}/images/optical_burn/optical_burn_mapping_workflow_v2.png"  vspace="10" width="300">
</p>

### Step 1: Baseline Time Windows
* **Pre-Fire Window:** Set relative to the fire start date using a customizable time window parameter (recommended default: **30 days**).
* **Post-Fire Window:** Set relative to the fire end date using the same customizable window duration (recommended default: **30 days**).
* **Purpose:** Combining multiple weeks of satellite overpasses ensures sufficient clear observations to construct representative pre- and post-fire baseline snapshots while minimizing seasonal vegetation changes.

### Step 2: Optical Image Compositing & Cloud Masking
* **Spectral Bands Used:** Sentinel-2 optical imagery utilizing the **Red, Green, Blue (RGB), Near-Infrared (NIR), and Shortwave Infrared (SWIR)** bands.
* **Cloud Masking:** Cloud masking is applied across all scenes to remove cloud contamination, cirrus haze, and cloud shadows (users can choose and adjust their preferred masking methods in the code).
* **Median Compositing:** Earth Engine calculates the median reflectance value for each pixel across all cloud-free scenes in each window, producing clean **Pre-Fire** and **Post-Fire** baseline composites.

### Step 3: Spectral Index Calculations (NBR & BAI)
For both pre-fire and post-fire composites, two complementary spectral indices are calculated:
* **Normalized Burn Ratio (NBR):** Calculated using the **NIR and SWIR2** bands ($\text{NBR} = \frac{\text{NIR} - \text{SWIR2}}{\text{NIR} + \text{SWIR2}}$). NBR measures canopy moisture loss and vegetation destruction.
* **Burned Area Index (BAI):** Calculated using the **Red and NIR** bands ($\text{BAI} = \frac{1}{(0.1 - \text{Red})^2 + (0.06 - \text{NIR})^2}$). BAI highlights charcoal and dark ash accumulation on the ground.

### Step 4: Measuring Spectral Change (dNBR & dBAI)
* **Delta NBR ($\text{dNBR}$):** Calculated as $\text{preFireNBR} - \text{postFireNBR}$. Higher positive values represent greater vegetation destruction and fire severity.
* **Delta BAI ($\text{dBAI}$):** Calculated as $\text{preFireBAI} - \text{postFireBAI}$. Measures relative change in charcoal and dark ash surface deposits.

### Step 5: Extracting Burned Area Polygons from Severity Thresholds
* **USGS Severity Classification:** Continuous dNBR values are stratified into standard USGS burn severity categories
<p align="center">
  <img src="{{ site.baseurl }}/images/optical_burn/USGS_BurnSeverity_Thresholds.png"  vspace="10" width="700">
</p>
[Image Source](https://un-spider.org/advisory-support/recommended-practices/recommended-practice-burn-severity/in-detail/normalized-burn-ratio)

* **Burn Polygon Generation:** Initial burn area polygons are created from pixels reaching the **Moderate-Low Severity threshold and above** ($\text{dNBR} \ge 0.270$). This captures moderate to severe fire impacts while excluding unburned terrain and background variation.

### Step 6: Land Cover Masking
Using Google's 10m Dynamic World land cover composite, detections are filtered based on land surface type:
* **Available Classes:** Water (Class 0), Trees (Class 1), Grass (Class 2), Flooded Vegetation (Class 3), Crops (Class 4), Shrub & Scrub (Class 5), Built (Class 6), Bare (Class 7), and Snow & Ice (Class 8).
* **Default Class Exclusions:** By default, **Water, Built, Bare, and Snow & Ice** are excluded. Detections over these surfaces are often false positives (e.g., shadow artifacts, dark asphalt, or water absorption) and are generally not of interest for vegetation fire mapping.
* **User Customization:** Users can toggle any combination of land cover classes to include or exclude based on their specific research or management focus.

### Step 7: Minimum Size Filtering
* **Noise & Small Fire Filtering:** Small, isolated detections below a user-defined area threshold (`minArea`) are filtered out.
* **Rationale:** Very small polygon fragments are more likely to represent spatial noise, misclassifications, or minor agricultural burns outside the scope of interest. Users can adjust this threshold (e.g., 5,000 m² / 0.5 hectares) to match their mapping objectives.

---

## 3. How to Use This Script

### Open the Script
Find the **users/ee-scripts/PC642-SERVIR-BPEmbeddings/part1/burnScar_optical** script.

You can also use the quick link: [https://code.earthengine.google.com/29baed42e7a78813a3af034f1f3a8ed5](https://code.earthengine.google.com/3b36286b4abbbf63af951d24e8e92bd3). 

If you make any changes to the script which you would like to save and easily access again, you can use `Save` (Save As) to save a version of the script to your own repository.

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
