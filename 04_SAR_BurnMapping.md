---
layout: page
title: "4. Burn Mapping with SAR Imagery"
permalink: /04_BurnMapping_SAR
nav_order: 4
---

# Synthetic Aperture Radar (SAR) Burn Mapping

## 1. Purpose of This Script

### High-Level Objective
This script provides an automated Earth Observation workflow in Google Earth Engine (GEE) to map burned areas using **Synthetic Aperture Radar (SAR)** imagery from Sentinel-1. 

Unlike optical sensors that depend on sunlight and clear skies, SAR is an active microwave sensor that penetrates persistent cloud cover, atmospheric haze, and dense smoke. This makes SAR burn mapping indispensable for tropical regions, monsoonal zones, and active fire monitoring where cloud-free optical imagery is unavailable.

Because radar backscatter is sensitive to multiple landscape variables—such as surface roughness, soil moisture, and canopy geometry—this script uses a **multi-index consensus approach**. Rather than relying on a single index (such as dNBR in optical mapping), the workflow computes **10 distinct SAR change indices** across dual polarizations (VV and VH) and extracts burn scars only where **at least 5 indices agree**, significantly reducing false positives and radar speckle noise.

*Note: This module represents the **SAR (Radar) Imagery** approach to burn scar mapping. Radar signals measure physical surface structure, canopy volume, and moisture dielectric properties rather than optical solar reflectance.*

### What Is Included in the Script
* **Sentinel-1 SAR Data Processing**: Automatically filters Sentinel-1 C-band Ground Range Detected (GRD) imagery (IW mode, descending orbit) across baseline pre- and post-fire time windows.
* **Dual-Polarization Backscatter Analysis**: Leverages both **VV** (vertical-vertical) and **VH** (vertical-horizontal) polarizations to track changes in soil moisture, surface roughness, and canopy structure.
* **Speckle Reduction & Linear Scale Conversion**: Converts backscatter from decibels (dB) to a linear scale ($10^{(\text{dB}/10)}$) and applies spatial focal filtering (`focalMean`) to smooth high-frequency radar speckle noise.
* **Ensemble of 10 SAR Change Indices**: Calculates a comprehensive suite of radar change metrics across VV and VH polarizations:
  * Absolute Radar Burn Differences (`VH_RBD`, `VV_RBD`)
  * Relative Radar Burn Differences 1 & 2 (`VH_RBDrel1`, `VV_RBDrel1`, `VH_RBDrel2`, `VV_RBDrel2`)
  * Radar Burn Ratios (`VH_RBR`, `VV_RBR`)
  * Delta Radar Vegetation Index (`dRVI`)
  * Delta Dual-Polarization SAR Vegetation Index (`dDPSVI`)
* **Multi-Index Consensus Thresholding**: Computes dynamic percentile cutoffs (e.g., 10th percentile) across the study region for each index and constructs an agreement map requiring **5 or more agreeing indices** ($\ge 5$) to isolate burn scars.
* **High-Resolution Optical Validation**: Overlays SAR burn scar detections onto high-resolution PlanetScope optical basemaps within the interactive Earth Engine map interface for visual inspection and verification.


You can also open a copy of the code using this quick link: [https://code.earthengine.google.com/a118a23701afb37f840d135973526eec](https://code.earthengine.google.com/a118a23701afb37f840d135973526eec)
