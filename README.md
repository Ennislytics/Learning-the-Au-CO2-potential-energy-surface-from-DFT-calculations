# Learning the Au–CO₂⁺ potential energy surface with Gaussian Process Regression

Can a small number of electronic-structure calculations reproduce a molecular potential energy surface, and does it matter which geometries are chosen?

## Overview

Calculating molecular energies at many different geometries can become expensive, particularly when more accurate electronic-structure methods are needed. I wanted to explore whether a Gaussian Process (GP) could learn the energy surface from a limited set of calculations, and whether choosing the training points more carefully would improve the result.

I used an existing set of 2,366 UB3P86/def2-TZVP calculations for Au–CO₂⁺, varying the Au–C distance from 2.5 to 5.0 Å and the approach angle from 0° to 90°. The CO₂ geometry was held fixed, so this is a two-dimensional slice of the potential energy surface rather than a full molecular PES.

The project compares three ways of selecting training geometries: random selection, choosing points where the model is most uncertain, and choosing points based on a combination of uncertainty and separation from existing training points. I tested each method against held-out DFT calculations and checked whether it recovered the lowest-energy geometry.

## The DFT energy surface

![Polar plot of the Au–CO₂⁺ potential energy surface](figures/au_co2_polar_pes.png)

*Polar representation of the calculated energy surface. The lowest-energy point on the original scan grid is at an Au–C separation of 3.20 Å and an angle of 23°. The full circle is obtained by reflecting the calculated 0–90° range. The inner region is masked for clarity, and the upper end of the colour scale is saturated; the underlying DFT energies are not changed.*

## Results

![Prediction error for the three sampling strategies](figures/sampling_rmse.png)

*Mean test-set prediction error across three different starting selections. Error bars show the standard deviation between runs. The vertical scale is logarithmic.*

Using 473 training geometries (20% of the calculations), the mean test RMSE was:

| Training-point selection | Mean RMSE (eV) |
| --- | ---: |
| Random | 0.00418 |
| Uncertainty only | 0.03122 |
| Uncertainty + diversity | 0.00190 |

The diversity-aware approach gave the lowest mean error at 118, 237 and 473 training points. At 473 points, its mean RMSE was about 55% lower than random sampling. This suggests that avoiding clusters of similar training geometries can be useful when trying to learn the surface efficiently.

However, predicting the overall surface accurately is not quite the same as finding its minimum.

![Recovery of the DFT minimum](figures/minimum_recovery.png)

*Number of runs, out of three, that identified the same minimum-energy point as the DFT scan (3.20 Å, 23°).*

Random sampling found the correct grid minimum more reliably at smaller training sizes, even though its average energy error was higher. By 473 points, both random and diversity-aware selection recovered the minimum in all three runs. The uncertainty-only method recovered it in one of three runs.

## What I took from this

The GP can reproduce the calculated energy surface with far fewer points than the full DFT grid, but the selection strategy matters. Choosing points solely from high predicted uncertainty was not consistently effective. Adding a preference for geometrically separated points improved the average prediction error, although it did not always improve the prediction of the lowest-energy geometry.

For me, the most interesting result is the difference between *overall energy accuracy* and *accuracy in a chemically important part of the surface*. This matters if a fitted potential is eventually going to be used to predict molecular structures or spectroscopic properties.

There are limits to what this small study shows. The comparison uses three initial selections and a test set drawn from the same regular DFT grid, so it mainly measures interpolation. The geometry of CO₂ is fixed, and the calculations use DFT rather than Quantum Monte Carlo. The results are therefore a starting point for thinking about more accurate and more efficient molecular potential fitting, not a demonstration of spectroscopic accuracy.

## Notebook

The analysis is in [`Au_CO2_Gaussian_Process_PES.ipynb`](Au_CO2_Gaussian_Process_PES.ipynb). It covers extraction of the Gaussian scan energies, visualisation of the PES, fitting the GP, comparing training-point selection methods and checking the predicted minimum.

The notebook expects the `Au_CO2_Scans` folder alongside it. The Gaussian output files are not included in this repository package; access to the underlying research data should be agreed with the project supervisor before public release.

## Tools

Python, NumPy, pandas, SciPy, scikit-learn and Matplotlib. Electronic-structure calculations were performed using Gaussian 16 at the UB3P86/def2-TZVP level.
