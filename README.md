# Initial Report README

## Project Title

**Spatial Downscaling of SMAP Soil Moisture Using Sentinel-1, MODIS NDVI, and Terrain Variables**

## Project Overview

This project investigates whether coarse-resolution SMAP soil moisture can be spatially downscaled using higher-resolution proxy variables from radar, vegetation, and topography. The workflow treats soil moisture downscaling as a supervised regression problem. SMAP soil moisture is used as the reference target variable, while Sentinel-1 backscatter, MODIS NDVI, and SRTM-derived terrain variables are used as predictors.

For the initial report, the work focuses on data preparation, exploratory data analysis, and preliminary model testing. The final goal is to train a model at the SMAP grid scale and then apply the trained model to finer-resolution predictors to generate a 1 km downscaled soil moisture product.

## Study Area and Time Period

- **Study area:** Southwestern Oklahoma
- **ROI:** `[-99.0, 34.5, -97.7, 35.8]`
- **Study period:** April 2024 through September 2024
- **Temporal structure:** Monthly raster stacks
- **Training scale:** SMAP scale, approximately 9 km
- **Prediction scale:** 1 km, for later downscaled prediction

## Data Sources

| Dataset | Variables Used | Role in Project |
|---|---|---|
| SMAP Level 3 Enhanced Soil Moisture | `soil_moisture_am` | Target/reference soil moisture variable |
| Sentinel-1 GRD | VV, VH, incidence angle | Radar proxy predictors |
| MODIS NDVI | NDVI | Vegetation proxy predictor |
| SRTM DEM | Elevation, slope, aspect | Terrain predictors |
| ISMN | 5 cm in-situ soil moisture | Planned external validation source |

## Workflow Summary

The workflow was revised to distinguish between the **training scale** and the **prediction scale**.

For model training, high-resolution proxy variables were aggregated to the SMAP grid. This prevents the model from treating multiple 1 km pixels within the same SMAP footprint as independent observations with repeated target values. Each training row represents one SMAP-scale pixel-month observation.

For later downscaled mapping, a separate 1 km prediction table can be used. This table contains finer-resolution predictor values and can be used after model training to generate 1 km soil moisture estimates.

## Google Earth Engine Processing

The Google Earth Engine workflow produced two main outputs:

1. **SMAP-scale training table**
   - File name: `oklahoma_smap_scale_training_Apr_Sep_2024.csv`
   - Use: EDA and model training
   - Each row: one SMAP-scale pixel-month observation

2. **1 km prediction table**
   - File name: `oklahoma_1km_prediction_Apr_Sep_2024.csv`
   - Use: later downscaled prediction
   - Each row: one 1 km pixel-month observation

The SMAP-scale training table is the priority file for the initial report.

## SMAP-Scale Training Table Columns

The main training CSV includes the following variables:

| Column | Description |
|---|---|
| `smap_sm` | Monthly mean SMAP soil moisture target |
| `mean_vv` | Sentinel-1 VV aggregated to SMAP scale |
| `mean_vh` | Sentinel-1 VH aggregated to SMAP scale |
| `mean_angle` | Sentinel-1 incidence angle aggregated to SMAP scale |
| `mean_ndvi` | MODIS NDVI aggregated to SMAP scale |
| `mean_elevation` | Elevation aggregated to SMAP scale |
| `mean_slope` | Slope aggregated to SMAP scale |
| `mean_vv_minus_vh` | VV minus VH radar contrast aggregated to SMAP scale |
| `mean_aspect_sin` | Sine-transformed aspect aggregated to SMAP scale |
| `mean_aspect_cos` | Cosine-transformed aspect aggregated to SMAP scale |
| `month` | Month number, April through September |
| `lon` | Longitude of sampled SMAP-scale point |
| `lat` | Latitude of sampled SMAP-scale point |

If a `.geo` column appears after export, it can be removed before modeling because longitude and latitude are already stored separately.

## Python Analysis Workflow

The Python notebook includes the following steps:

1. Import libraries
2. Load the SMAP-scale training CSV
3. Inspect dataset shape, column names, data types, and summary statistics
4. Check missing values and duplicate rows
5. Remove missing values, duplicates, and physically unreasonable values
6. Plot the distribution of SMAP soil moisture
7. Compute and plot monthly mean SMAP soil moisture over the ROI
8. Create a spatial scatter plot of SMAP-scale soil moisture samples
9. Compute a correlation matrix for the target and predictors
10. Define predictors and target variable
11. Split the data temporally into training and testing sets
12. Train and evaluate a mean baseline model
13. Train and evaluate Lasso Regression
14. Train and evaluate Random Forest Regression
15. Compare model performance using RMSE, MAE, and R²
16. Create predicted-versus-observed plots
17. Examine Random Forest feature importance
18. Examine Lasso coefficients
19. Save model results and prediction outputs

## Modeling Approach

The preliminary model uses a temporal train-test split:

- **Training data:** April through August 2024
- **Test data:** September 2024

The target variable is:

- `smap_sm`

The predictors are:

- `mean_vv`
- `mean_vh`
- `mean_angle`
- `mean_ndvi`
- `mean_elevation`
- `mean_slope`
- `mean_vv_minus_vh`
- `mean_aspect_sin`
- `mean_aspect_cos`
- `month`

Three models are compared:

1. **Mean Baseline**
   - Predicts the global April-August mean SMAP soil moisture for every September test pixel.
   - Does not use radar, NDVI, terrain, month-specific spatial patterns, or location information.

2. **Lasso Regression**
   - Linear model with L1 regularization.
   - Used for interpretable preliminary modeling and coefficient analysis.

3. **Random Forest Regression**
   - Nonlinear ensemble model.
   - Used to capture more complex relationships among radar, vegetation, terrain, and SMAP soil moisture.

## Evaluation Metrics

The preliminary models are evaluated using:

| Metric | Meaning |
|---|---|
| RMSE | Root mean squared error; lower is better |
| MAE | Mean absolute error; lower is better |
| R² | Proportion of variance explained; higher is better |

The machine-learning models should outperform the mean baseline to demonstrate that the predictor variables add useful information.

## Output Files from Python Analysis

The notebook saves the following files:

| File | Description |
|---|---|
| `initial_model_results.csv` | Model comparison metrics for baseline, Lasso, and Random Forest |
| `random_forest_feature_importance.csv` | Feature-importance values from Random Forest |
| `lasso_coefficients.csv` | Lasso coefficient table |
| `september_test_predictions.csv` | September test observations with model predictions |

These files can be reused in the final report.

## Initial Report Interpretation

The initial analysis demonstrates that the data pipeline is functional and that the project has progressed beyond data collection. The workflow successfully produced monthly SMAP-scale training samples, performed data cleaning and exploratory analysis, trained preliminary regression models, and generated model-evaluation outputs.

The results should be interpreted as a preliminary satellite-based feasibility analysis. The models are trained and tested against SMAP-scale soil moisture, not yet fully validated against independent ground observations.

## Validation Limitation

An ISMN station search identified only one station within the ROI with 5 cm soil moisture data for April-September 2024. Since the current workflow uses monthly stacks, this provides only six station-month observations for external ground validation.

Therefore, ISMN validation should be treated as preliminary rather than definitive. To strengthen final validation, the project may expand the time period, expand the validation search area, or include additional nearby stations if available.

## Next Steps

The next phase will refine the preliminary workflow for the final project. Recommended next steps include:

1. Expand the analysis period beyond April-September 2024 to include additional growing seasons.
2. Improve model testing by using additional temporal splits, such as testing across multiple months or years.
3. Apply the trained model to the 1 km prediction table to generate a downscaled soil moisture product.
4. Map the 1 km predicted soil moisture estimates.
5. Compare the downscaled predictions with the original SMAP product.
6. Use ISMN station data as an external validation check, while clearly acknowledging the limited number of available station observations.
7. Explore whether the downscaled model improves agreement with ISMN compared with the original SMAP value at the station location.

## Reproducibility Notes

To reproduce the initial analysis:

1. Run the Google Earth Engine script to create the SMAP-scale training table.
2. Export `oklahoma_smap_scale_training_Apr_Sep_2024.csv` to Google Drive.
3. Download the CSV and place it in the same folder as the Python notebook.
4. Run the notebook cells in order.
5. Confirm that `.geo`, if present, is removed before modeling.
6. Review the saved model-output CSV files for report tables and figures.

