# BEPMAT: Biomass Energy Potential Mapping and Analysis Tool

**Imperial College London, Climate Compatible Growth programme, May 2023 – Mar 2024**
Divyansh Singhal (IIT Delhi), Vignesh Sridharan and Adam Hawkes (Imperial College London).
The first two authors contributed equally.

- Code: [ClimateCompatibleGrowth/BEPMAT](https://github.com/ClimateCompatibleGrowth/BEPMAT)
- Paper draft: [`BEPMAT_JOSS_draft.pdf`](BEPMAT_JOSS_draft.pdf), submitted to the *Journal of Open Source Software*
  on 9 April 2024 (submission 6657, not yet published)

## Why it exists

Energy system models need spatially explicit resource potentials as inputs. Solar and wind have
open tools, such as Renewables.ninja, the Global Wind Atlas and the Global Solar Atlas. Biomass has
few, and most of those don't publish their code. BEPMAT fills that gap for **crop residues**,
producing outputs that energy system models can use directly.

## What it does

- Estimates the theoretical energy potential from crop residues for any country, down to
  **province level** using GADM administrative boundaries.
- Covers a baseline year and three future periods (2011–2040, 2040–2070, 2070–2100) under multiple
  **RCP climate pathways** and climate models.
- Splits available land into **current cropland**, where today's crops keep growing under changing
  climate yields, and **marginal land**. For marginal land, it searches the crop database for the
  crop with the highest attainable yield.
- Outputs interactive plotly maps and charts and downloadable **NetCDF4 / xarray** datasets.
  Region-specific residue factors can be overridden.

## Method

1. Start from the selected geography. Exclude land unfit for crops, protected areas, tree cover and
   water bodies to get the total available land.
2. Separate cropland from marginal land. On marginal land, choose the crop with the maximum
   agro-climatically attainable yield.
3. Convert production to energy:

```
Theoretical energy potential = Σ_i Yield_i × Area_i × RPR_i × SAF_i × LHV_i
```

RPR is the residue-to-product ratio, SAF is the share of residue available for energy, and LHV is
the lower heating value.

## Data

- FAO and IIASA **GAEZ v4**: agro-ecological zones, production, harvested area, exclusion areas,
  tree cover, and future yields by RCP and water supply.
- SEDAC pastureland; GADM boundaries through the `gadm` Python library.
- RPR, SAF and LHV values from published literature (cited in the paper and documentation).

## Stack

Python and Jupyter, GeoPandas, Rasterio, xarray and NetCDF4, plotly and bokeh, and QGIS for
exploration.
