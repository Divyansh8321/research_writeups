# Adjusted Heating Degree Days for a Home Energy Model

**MITACS Globalink Research Internship, University of Victoria, May–Aug 2024**
ReBuild Initiative with OPEN Technologies, Vancouver. Supervised by Prof. Ralph Evins and Prof. Michael Gutland.

## The problem

The ReBuild Home Energy Model (HEM) estimates monthly heat loss through each part of a house
(ceilings, main walls, windows, doors) with a simple Fourier conduction formula:

```
Q_component = (A_component × HDD_component / R_component) × 86400 s/day / 10^6 J/MJ
```

The weak point is the HDD (heating degree days) term. Published HDDs, and the ones stored in
HOT2000 files, use an 18 °C reference temperature, but houses are heated to about 20 °C. The quick
fix of adding 2 °C × 365 days breaks down in summer, when outdoor temperatures pass the reference
and fewer degree days should count.

The goal was to find an **adjusted HDD** for each component and city that makes this formula
reproduce what HOT2000, Natural Resources Canada's detailed simulator, actually calculates.

## Approaches tried

| Approach | Idea | Outcome |
|---|---|---|
| Constant monthly temperature | HDD = (T_ref − T_avg,month) × days, with T_ref = 20 °C | Exact for doors in winter; overcounts HDD in summer |
| Sine-curve daily temperature | Model daily temperature as a cosine around the monthly mean, using the weather file's amplitude, then integrate between the curve and T_ref | Matches the constant method in cold months and correctly counts fewer HDD in warm months |
| Sol-air temperature | Follow HOT2000's documentation: raise surface temperature by a ∝ irradiance | Unstable; produced negative heat loss in summer, so it was dropped |
| **Simulation plus regression (used)** | Simulate many homes in HOT2000, back-calculate the adjusted HDD from the simulated losses, and regress it on HDD, CDD and irradiance | Best results; became the production method |

## What I built

1. **Synthetic home generator.** A script that takes a template HOT2000 (`.h2k`) house and writes
   batches of variants. It samples footprint, insulation R-values, air leakage, window-to-wall
   ratio and heating systems from distributions (discrete, uniform, Gaussian, skewed) that I fitted
   to the **Ontario EnerGuide** dataset. It assigns each variant to an Ontario weather station and
   logs every combination.
2. **Batch simulation pipeline.** It drives HOT2000 from the command line over the generated files,
   then parses the results and the weather data (monthly HDD, CDD, irradiance, wind) into tables.
3. **Regression of adjusted HDD per component.** For each component, I back-calculated the
   adjusted HDD from the simulated heat loss and fitted linear, ridge, polynomial and polynomial
   ridge models on HDD and CDD (plus irradiance and the ceiling's insulation level for ceilings).
   The coefficients feed a single summary table that produces adjusted HDDs for any city.
4. **The U-value fix for ceilings.** Ceiling adjusted HDD depends on the house's insulation. The
   adjusted HDD is inversely related to the R-value but linearly related to the U-value (1/R), so
   regressing on U-value with polynomial terms gave a far better fit, **R² of 0.99+**. Per-house
   coefficients are then multiplied by each house's R-value from EnerGuide.
5. **Diagnostics.** Plots of percentage error against actual values for every component, method
   and HDD variant, used to choose the final models.
6. **Downstream attribute models.** The adjusted HDD dataset fed into a cleaned Ontario EnerGuide
   dataset, which was used to train **CatBoost** models predicting home attributes such as system
   efficiencies and RSI values. Data and feature engineering cut prediction loss by up to 50% on
   several attributes.

## Extending it to other provinces

The pipeline is province-agnostic. To extend it, you export that province's weather files from
HOT2000, refit the parameter ranges from its EnerGuide records, regenerate and simulate, then rerun
the regression and HDD steps. 

## Known limitations

- Only HOT2000's `wth110` weather set could be run in batch. The newer `wth2020` files had
  city-indexing problems in CLI mode, so their adjusted HDDs reuse the `wth110` coefficients.
- Sol-air corrections for walls remain an open problem.

## Note on code

The codebase belongs to the ReBuild project and is not included here. This repository documents my
methodology and results only.
