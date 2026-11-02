---
layout: page
title: "4. Burn Mapping with SAR Imagery"
permalink: /04_BurnMapping_SAR
nav_order: 4
---

# Synthetic Aperture Radar (SAR) Burn Mapping

## 1. Purpose of This Script

### High-Level Objective
This script provides an automated Earth Observation workflow in Google Earth Engine (GEE) to map burned areas and assess fire extent across custom geographic regions using **Synthetic Aperture Radar (SAR)** imagery from Sentinel-1.

Unlike optical sensors that depend on visible sunlight and cloud-free skies, SAR is an active microwave system that operates independently of weather conditions, penetrating heavy cloud cover, tropical atmospheric haze, and dense wildfire smoke. This capability makes SAR burn mapping indispensable for monsoonal zones, tropical rainforests, and active fire monitoring where optical imagery is unavailable.

Because radar signals interact with physical land surface properties—such as canopy volume, branch geometry, surface roughness, and soil dielectric moisture—this script employs a **multi-index consensus voting model**. Rather than relying on a single index (like dNBR in optical mapping), the script calculates **10 distinct SAR change indices** across dual polarizations (VV and VH) and extracts burn scars only where **at least 5 indices agree**, drastically reducing false alarms and radar speckle noise.

* **Script Path:** `users/ee-scripts/PC642-SERVIR-BPEmbeddings/part1/burnScar_SAR`
* **Quick Link to GEE Script:** [Open Google Earth Engine Code Editor](https://code.earthengine.google.com/a118a23701afb37f840d135973526eec)

*Note: This module represents the **SAR (Radar) Imagery** approach to burn scar mapping. Radar microwave signals measure physical canopy volume and structural destruction rather than optical leaf reflectance.*

### What Is Included in the Script
* **Sentinel-1 SAR Processing Pipeline**: Automatically filters Sentinel-1 C-band Ground Range Detected (GRD) imagery (Interferometric Wide swath mode, Descending orbit) across baseline pre- and post-fire time windows.
* **Pre- & Post-Fire Image Count Diagnostics**: Automatically calculates and prints the exact number of Sentinel-1 scenes available in pre-fire and post-fire baseline periods to prevent empty composite errors.
* **Dual-Polarization Backscatter Analysis**: Evaluates both **VV** (vertical transmit / vertical receive) and **VH** (vertical transmit / horizontal receive) channels to track changes in canopy structure and ground exposure.
* **Linear Scale Conversion & Speckle Filtering**: Converts raw backscatter from decibels (dB) to a linear scale ($10^{(\text{dB}/10)}$) and applies spatial focal mean smoothing to suppress radar speckle noise.
* **10-Index SAR Ensemble**: Computes a suite of 10 complementary radar change metrics including Radar Burn Difference (`RBD`), Relative Radar Burn Differences (`RBDrel1`, `RBDrel2`), Radar Burn Ratio (`RBR`), Delta Radar Vegetation Index (`dRVI`), and Delta Dual-Polarization SAR Vegetation Index (`dDPSVI`).
* **Multi-Index Consensus Voting**: Computes dynamic 10th percentile cutoffs across the study region and overlays the 10 binary index maps, requiring $\ge 5$ agreeing indices to isolate candidate burn scars.
* **Dynamic World Land Cover Masking**: Integrates Dynamic World land cover composites to mask non-burnable surfaces (water bodies, urban pavement, bare ground, snow/ice) that produce radar specular reflection false positives.
* **Vector Polygon Generation & Minimum Size Filtering**: Converts classified burn rasters into vector polygons, calculates polygon surface areas ($m^2$), and filters out small isolated noise fragments below a user-defined threshold (`minArea`).
* **Automated Spatial Tiling Grid**: Subdivides large study regions into spatial grid tiles to optimize memory usage, avoid compute timeouts, and enable batch exports.
* **Discrete 10-Color Palette & 5-Milestone UI Legends**: Displays a 10-color discrete map layer alongside custom UI legend panels for SAR Index Agreement (1 to 10) and Dynamic World Land Cover.
* **Optional Optical Baseline Comparison**: Includes background Sentinel-2 optical compositing with Cloud Score+ quality filtering for visual cross-validation.
* **Batch Data Exporting**: Generates export tasks for refined raster GeoTIFF files and vector shapefiles (`.shp`) directed to Google Drive or GEE Assets.

---

## 2. General Analysis Overview

The SAR burn mapping workflow follows a sequential microwave remote sensing methodology to transform raw active radar backscatter observations into refined burn scar vectors and raster datasets.
<p align="center">
  <img src="{{ site.baseurl }}/images/sar_burn/sar_burn_mapping_workflow_short.png.png"  vspace="10" width="300">
</p>

---

### Comparison: Benefits & Pitfalls of SAR Burn Mapping

Understanding when and why to use SAR imagery instead of optical imagery is key to effective wildfire monitoring:

| Feature / Dimension | Active Microwave SAR (Sentinel-1) | Passive Optical (Sentinel-2 / Landsat) |
| :--- | :--- | :--- |
| **Atmospheric Penetration** | **All-Weather Capability**: Microwave C-band signals penetrate cloud cover, tropical haze, and dense smoke completely. | **Cloud-Restricted**: Obscured by cloud cover, cirrus haze, and thick wildfire smoke. |
| **Illumination Source** | **Active Day & Night Sensing**: Satellite carries its own microwave transmitter; operates during night overpasses. | **Passive Sensing**: Requires daytime solar illumination; limited by sun angle and shadows. |
| **Target Surface Interaction** | **Physical Structure & Moisture**: Responds to branch geometry, canopy density, surface roughness, and dielectric moisture. | **Biochemical & Spectral Reflectance**: Responds to chlorophyll absorption, canopy greenness, and SWIR moisture loss. |
| **Main Advantage** | Reliable burn detection in tropical, monsoonal, or active fire events where optical scenes are unusable. | Standardized burn severity stratification (e.g., USGS dNBR categories) and high sensitivity to ash/charcoal deposition. |
| **Primary Pitfalls** | • **Radar Speckle Noise**: Requires spatial filtering.<br/>• **Specular Reflection False Positives**: Calm water, smooth roads, and bare ground reflect radar signals away from the sensor, producing low backscatter that mimics severe burns.<br/>• **Topographic Distortion**: Terrain layover and shadow in steep mountains. | • High omission rates during rainy/cloudy burning seasons.<br/>• Cloud shadow artifacts misclassified as burn scars. |

---

### Deep Dive: How SAR Burn Mapping & Indices Work

#### 1. Physical Basis: Canopy Volume Scattering vs. Ground Backscatter
At C-band wavelengths (~5.6 cm), radar signals interact strongly with forest leaf canopies, twigs, and branches through **volume scattering** (multiple reflections off structural elements returning power to the satellite). 

When a wildfire destroys the forest canopy:
1. **Canopy Collapse**: Foliage and small branches burn away, destroying volume scattering.
2. **Signal Loss**: The radar beam passes through the open canopy and reflects away off exposed ground surfaces, causing a sharp **drop in returned backscatter power** ($VV$ and $VH$).
3. **Moisture Shifts**: Soil drying during fires further reduces the dielectric constant, decreasing returned signal intensity.

#### 2. Mathematical Orientation: The `-1` Multiplier
In standard remote sensing equations, some ratios (like $(Pre - Post) / Pre$) yield positive numbers for a post-fire drop. In this script, all relative change formulas (`RBDrel1` and `RBDrel2`) are intentionally multiplied by `-1`:

$$	ext{RBDrel1} = \left(rac{	ext{PreFire} - 	ext{PostFire}}{	ext{PreFire}}
ight) 	imes (-1)$$

**Why this matters:** Multiplying by `-1` forces **every single one of the 10 SAR indices to produce negative values for fire damage**. This creates a single uniform rule across the entire script: **pixels with the most negative change values represent burn candidates across all bands**.

#### 3. Statistical Justification for Dynamic 10th Percentile Thresholding
Rather than hardcoding a static cutoff (e.g., $-0.05$) that fails when seasonal soil moisture or baseline forest types change, the script calculates a **dynamic 10th percentile cutoff** across the study area using `reduceRegion`:

$$	ext{percentile} = 	ext{indices.reduceRegion}(	ext{reducer} = 	ext{ee.Reducer.percentile}([10]))$$

* **Statistical Logic**: Burned areas represent spatial anomalies covering a minority of the landscape. Taking the 10th percentile isolates the bottom 10% extreme drops in backscatter across the region.
* **Region-Adaptive**: Automatically adjusts the cutoff threshold for each specific image pair and geographic region.

#### 4. Multi-Index Voting Consensus ($\ge 5$ Agreeing Indices)
Because individual radar metrics can suffer from single-index noise or moisture fluctuations, the script overlays **10 distinct binary index layers**:

$$	ext{burnAgreement} = \sum_{i=1}^{10} \mathbf{1}_{\{	ext{index}_i \le 	ext{percentile}_i\}}$$

A pixel is classified as a **Potential Burn Scar** only when **at least 5 out of 10 indices agree** ($	ext{burnAgreement} \ge 5$). This voting consensus eliminates single-index false alarms without sacrificing detection sensitivity.

#### 5. Dynamic World Land Cover Masking
Smooth flat surfaces (calm water bodies, paved highways, airport runways, and bare soil) reflect radar signals away from the sensor like a mirror (**specular reflection**). This produces very low backscatter returns that look identical to severe burn scars.

To solve this, the script overlays Google's 10m Dynamic World land cover model and **excludes non-vegetated classes** (Water, Built, Bare, Snow/Ice by default), restricting burn mapping strictly to burnable land cover types (Trees, Grass, Flooded Veg, Crops, Shrub).

#### 6. Minimum Area Size Filtering (`minArea`)
Radar imagery contains inherent high-frequency speckle noise. The script converts classified burn rasters into vector polygons, calculates polygon surface areas ($m^2$), and removes small fragments below a user-defined threshold (e.g., $5,000 \, m^2$ / 0.5 hectares).

---

### Detailed Breakdown of the 10 SAR Change Indices

The script computes 10 complementary change metrics across **VH** (cross-polarization, highly sensitive to canopy volume) and **VV** (co-polarization, sensitive to surface roughness and moisture):

| # | Index Name | Polarization | Mathematical Formula | Physical & Remote Sensing Interpretation |
| :---: | :--- | :---: | :--- | :--- |
| **1** | `VH_RBD` | VH | $	ext{Post}_{VH} - 	ext{Pre}_{VH}$ | **Radar Burn Difference (VH)**: Absolute raw backscatter power change. Captures loss of canopy volume scattering. |
| **2** | `VH_RBDrel1` | VH | $\left(rac{	ext{Pre}_{VH} - 	ext{Post}_{VH}}{	ext{Pre}_{VH}}
ight) 	imes (-1)$ | **Relative Radar Burn Difference 1 (VH)**: Fractional backscatter change relative to pre-fire baseline level. |
| **3** | `VH_RBDrel2` | VH | $\left(rac{	ext{Pre}_{VH} - 	ext{Post}_{VH}}{\sqrt{	ext{Pre}_{VH}}}
ight) 	imes (-1)$ | **Relative Radar Burn Difference 2 (VH)**: Variance-scaled change that dampens background noise in low-return areas. |
| **4** | `VH_RBR` | VH | $rac{	ext{Post}_{VH}}{	ext{Pre}_{VH}}$ | **Radar Burn Ratio (VH)**: Simple post-to-pre backscatter ratio. |
| **5** | `VV_RBD` | VV | $	ext{Post}_{VV} - 	ext{Pre}_{VV}$ | **Radar Burn Difference (VV)**: Absolute raw change in vertical co-polarization. Captures surface roughness and moisture drop. |
| **6** | `VV_RBDrel1` | VV | $\left(rac{	ext{Pre}_{VV} - 	ext{Post}_{VV}}{	ext{Pre}_{VV}}
ight) 	imes (-1)$ | **Relative Radar Burn Difference 1 (VV)**: Fractional change in vertical co-polarization backscatter. |
| **7** | `VV_RBDrel2` | VV | $\left(rac{	ext{Pre}_{VV} - 	ext{Post}_{VV}}{\sqrt{	ext{Pre}_{VV}}}
ight) 	imes (-1)$ | **Relative Radar Burn Difference 2 (VV)**: Variance-scaled vertical co-polarization change. |
| **8** | `VV_RBR` | VV | $rac{	ext{Post}_{VV}}{	ext{Pre}_{VV}}$ | **Radar Burn Ratio (VV)**: Simple post-to-pre ratio of vertical backscatter intensity. |
| **9** | `dRVI` | VV & VH | $	ext{RVI}_{	ext{Post}} - 	ext{RVI}_{	ext{Pre}}$, where $	ext{RVI} = rac{4 \cdot VH}{VV + VH}$ | **Delta Radar Vegetation Index**: RVI estimates canopy volume scattering relative to total power. Negative $	ext{dRVI}$ tracks structural canopy collapse. |
| **10** | `dDPSVI` | VV & VH | $	ext{DPSVI}_{	ext{Post}} - 	ext{DPSVI}_{	ext{Pre}}$, where $	ext{DPSVI} = rac{VV + VH}{VV}$ | **Delta Dual-Pol SAR Vegetation Index**: Tracks shifts in total backscatter power relative to vertical co-polarization. |

---

## 3. How to Use This Script

Executing the script involves three simple steps: configuring your input parameters at the top of the code, running the script to inspect map layers and UI legends, and initiating batch exports from the Earth Engine Tasks panel.

### Step 1: Configuring User Parameters
All user-adjustable settings are consolidated in the **USER CONFIGURATION** block at the top of the script (lines 30–90):

#### 1. Fire Dates & Baseline Windows
Set the fire calendar year, start date, end date, and window duration:
```javascript
var year = '2024';                  // Year of the fire event
var fireStartString = '01-30';      // Fire start date in 'MM-DD' format
var fireEndString = '03-30';        // Fire end date in 'MM-DD' format
var windowDays = 30;                // Baseline window duration (days)
```

#### 2. Area of Interest (AOI) Selection
Specify your study region using FAO GAUL administrative boundaries:
```javascript
var countryName = "Lao People's Democratic Republic"; // Country name (Admin 0)
var admin1Name = "Bokeo";                            // Province/State name (Admin 1)
```
In the script body under **AOI SELECTION**, uncomment **one** of the four AOI options:
* **Option 1 (Default)**: Province / State level boundary (`admin1Name`).
* **Option 2**: Entire Country boundary (`countryName`).
* **Option 3**: Hand-drawn map polygon using GEE drawing tools (`geometry`).
* **Option 4**: Path to an existing GEE FeatureCollection asset.

#### 3. Dynamic World Land Cover Toggles
Toggle land cover classes to include (`true`) or exclude/mask out (`false`):
```javascript
var includeWater      = false; // Class 0 : Water (Excluded by default - prevents radar false positives)
var includeTrees      = true;  // Class 1 : Trees
var includeGrass      = true;  // Class 2 : Grass
var includeFloodedVeg = true;  // Class 3 : Flooded Vegetation
var includeCrops      = true;  // Class 4 : Crops
var includeShrub      = true;  // Class 5 : Shrub & Scrub
var includeBuilt      = false; // Class 6 : Built (Excluded by default - prevents pavement false positives)
var includeBare       = false; // Class 7 : Bare (Excluded by default)
var includeSnowIce    = false; // Class 8 : Snow & Ice (Excluded by default)
```

#### 4. Minimum Burn Scar Area Filter
Define the smallest burn scar polygon size to retain:
```javascript
var minArea = 5000; // Minimum area threshold in square meters (5,000 m² = 0.5 hectares)
```

#### 5. Export Grid Scale
Set the tile size for dividing large study areas:
```javascript
var gridScale = 140000; // Grid cell side-length in meters (140,000 m = 140 km)
```

---

### Step 2: Running the Script & Examining Map Results
Click the **Run** button at the top of the Google Earth Engine Code Editor.

#### Inspecting the Console Tab
Check the **Console** panel on the right side of the interface for key diagnostic prints:
1. **Calculated Date Ranges**: Displays the exact start and end dates for pre-fire and post-fire observation windows.
2. **Sentinel-1 Image Counts**: Prints `Number of Pre-fire Sentinel-1 images` and `Number of Post-fire Sentinel-1 images` to verify scene availability.
3. **Available Boundary Names**: Lists valid country and province names for spelling verification.
4. **Dynamic Percentiles**: Displays calculated 10th percentile cutoff values for all 10 SAR indices.

#### Interacting with Map Layers & Side-by-Side Legends
Use the **Layers** control in the top-right corner to toggle map layers:
* **AOI Boundary & Grid**: Outlines the study area and spatial tiling grid.
* **Pre & Post Fire S1 (VV / VH)**: Displays pre- and post-fire backscatter images.
* **10 Individual SAR Index Layers**: Allows inspection of unthresholded continuous rasters alongside binary thresholded layers (`.lte(percentile)`).
* **SAR Index Agreement (1 to 10)**: Renders the discrete multi-index agreement map alongside the **SAR Index Agreement Legend** (bottom-left).
* **Potential Burn Scar**: Highlights candidate burn areas where $\ge 5$ indices agree (in dark red).
* **Dynamic World Land Cover**: Displays modal land cover classes alongside the **Dynamic World Legend** (bottom-right).
* **Allowed Land Cover Mask**: Shows vegetated land cover types selected for analysis in green.
