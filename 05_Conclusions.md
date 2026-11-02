---
layout: page
title: "5. Conclusions"
permalink: /05_BurnMapping_Conclusions
nav_order: 5
---

# Conclusions & Key Takeaways

## Executive Summary
This **Burn Scar Mapping Training** provides an operational framework for mapping, evaluating, and reporting wildfire impacts across diverse global biomes.
By bridging satellite remote sensing theory with Google Earth Engine (GEE) workflows, these training materials equip analysts, regional forestry management agencies, and environmental researchers with transparent, reproducible, and cloud-tolerant monitoring tools.


## Key Takeaways & Core Training Outcomes

### 1. Foundational & Background Learning
* **Remote Sensing & Satellite Telemetry Principles**: Participants build a strong understanding of how satellite sensors interact with land surface features across optical, thermal, and active microwave spectrums.
* **Cloud Computing in Google Earth Engine (GEE)**: Hands-on mastery of GEE enables analysts to process massive regional satellite image collections on Google's cloud infrastructure without requiring heavy local hardware resources.

### 2. Optical Imagery for Burn Mapping & Severity Assessment
* **Spectral Index Differencing (**dNBR** & $	**dBAI**): Leverages Sentinel-2 multispectral bands (NIR, Red, SWIR1, SWIR2) to measure chlorophyll destruction, charring, and canopy water loss following wildfire events.
* **Standardized Severity Stratification**: Uses delta Normalized Burn Ratio (**dNBR** ) mapped directly to standard USGS burn severity thresholds (ranging from unburned/regrowth to high severity).

### 3. Synthetic Aperture Radar (SAR) for Cloud-Tolerant Burn Mapping
* **All-Weather & Cloud/Smoke Penetration**: Ingests Sentinel-1 C-band active microwave radar ($VV$ and $VH$ polarizations) that penetrates dense cloud cover, haze, and fire smoke, providing essential mapping capability in tropical and monsoonal biomes.
* **10-Index Radar Ensemble & Consensus Voting**: Calculates 10 complementary radar change metrics ($	ext{RBD}$, $	ext{RBDrel1}$, $	ext{RBDrel2}$, $	ext{RBR}$, $	ext{dRVI}$, $	ext{dDPSVI}$) and extracts burn scars only where at least 5 indices agree (greater than or equal to 5 votes), suppressing single-index speckle noise and soil moisture false positives.

## Next Steps
Two core methods for burn mapping are presented. Now that you have tested both, explore your results to see which option performs best in your area of interest! Maps of burned areas can be used for multiple purposes, such as: disaster resource allocation decision making, historical assessments of fire damage extent and severity, or as a map for sampling stratification for sample-based area estimation of burned areas. How might you use these results in your region?
