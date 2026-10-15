# Community-TRR-Data-and-Analysis
# Community TRR Data and Analysis

This repository contains community road-network GIS files, community-level predictor data, and code for measuring and analyzing traffic recovery resilience under extreme snowfall (TRR) in Shenyang, Harbin, and Changchun. The analytical sequence is: road-speed recovery calculation, severity adjustment, reverse min–max normalization, and subsequent machine-learning regression and SHAP interpretation.

## 1. Repository contents

Folder names below follow the uploaded archive, including their original spelling.

| Folder                        | Contents and purpose                                                               |
| ----------------------------- | ---------------------------------------------------------------------------------- |
| `Com_roadmapcutting/`         | Community-clipped road-network GIS data for the three cities.                      |
| `Machine Learning code&data/` | Community panel and the three-city, four-community-type machine-learning notebook. |
| `TRR calulation code&data-1/` | TRR calculation notebook and input package.                                        |
| `TRR caluation code&data-2/`  | Saved Harbin calculation outputs, including `Harbin_TRR_results`.                  |

### 1.1 Community road-network GIS data

`Com_roadmapcutting/` contains three city subfolders:

| City      | Main road layer                  |
| --------- | -------------------------------- |
| Changchun | `Changchun/cuutincchangchun.shp` |
| Harbin    | `Harbin/hrbcutting.shp`          |
| Shenyang  | `Shenyang/sycuutingroad.shp`     |

These layers provide road geometry and attributes for spatial matching. Harbin also includes `hrbcutting_GeneratePointsAlongLine.shp`, a derived point layer along the roads. Keep each shapefile together with its `.dbf`, `.shx`, `.prj`, and any accompanying encoding or spatial-index files. The coordinate reference system is recorded in the `.prj` file.

### 1.2 Machine-learning code and community panel

| File                                  | Purpose                                                                                                                                                                                                                      |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Three_City_TRR_Analysis_Colab.ipynb` | Three-city, four-type community machine-learning training. Implements pooled reverse normalization before train/test separation and subsequent regression, model comparison, SHAP, and PDP analysis.                         |
| `Community_Datazz_plain_updated.xlsx` | Community panel with three city worksheets, identifiers, names, coordinates, TRR, and 22 candidate infrastructure and control variables. Use the updated replacement workbook containing Community_type, as specified below. |

The panel contains 2,700 Shenyang, 1,977 Harbin, and 1,689 Changchun communities: 6,366 rows in total. `Community_ID` is the key used to join the panel to city-level TRR calculation outputs.

The notebook first joins the original `Recovery_burden_B` values from the three city result CSVs, calculates one common minimum and maximum across available burdens, and applies reverse normalization. It writes `Pooled_TRR.csv` before model fitting. The regression outcome is this normalized TRR; recovery burden, meteorological severity, and normalization diagnostics are not predictors.

The machine-learning branch includes:

- Natural-log transformation of BH, Dis_M, Dis_B, DB, H_D, and Slope.
- Pearson correlation coefficients, two-sided p values, pairwise sample counts, and correlation heatmaps for the pooled sample and each community type.
- Separate models for CHC (commercial housing), UC (unit-based), OC (old residential), and UVC (urban village) communities, plus a pooled model.
- A 70% training/30% test split,  five-fold Bayesian hyperparameter tuning, comparison of XGBoost, DT, RF, SVM, and GBDT, and held-out R², MAE, RMSE, and MSE.
- SHAP importance, dependence and interaction outputs, and partial dependence plots.



### 1.3 TRR calculation code, inputs, and Harbin working record

The materials in this section are split between `TRR calulation code&data-1/` and `TRR caluation code&data-2/`. The TRR calculation notebook and input package are stored in the first folder, while `Harbin_TRR_results.zip` and `Harbin_TRR_results/` are stored in the second folder.

| File or folder                 | Purpose                                                                                                                                                                          |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Harbin_TRR_RAR_Colabzz.ipynb` | Colab-ready TRR calculation code with saved Harbin execution outputs, input checks, spatial matching, recovery calculations, reverse normalization, and quality-control outputs. |
| `Harbin_TRR_1km_Package.rar`   | Input archive uploaded to the Harbin calculation notebook.                                                                                                                       |
| `Harbin_TRR_1km_Package/`      | Extracted copy of the input package.                                                                                                                                             |
| `Harbin_TRR_results.zip`       | Downloadable archive of the six Harbin output files listed below.                                                                                                                |
| `Harbin_TRR_results/`          | Extracted calculation results and audit files.                                                                                                                                   |

The input package contains Harbin community AOI boundary shapefiles, `Harbinroadcutting.rar` with clipped road-network GIS data, `Road_Speed_Harbinzz.xlsx` with hourly road-speed series, and `Station_Weather_46_Hourly.xlsx` with station coordinates and hourly weather records. The road-speed workbook contains 1,963 road segments; the saved calculation assigns functional weights to 1,762 motorized-road segments.

The weather workbook contains `Station_list`, `Hourly_weather`, and `Event_summary` worksheets. It covers 46 stations: 20 in Harbin, 20 in Changchun, and 6 in Shenyang, with 32,016 hourly records. The Harbin worked example uses the relevant Harbin stations. Community centroids are matched to stations, and station-specific snowfall cessation defines the recovery-time origin.

## 2. Six Harbin result files

| File                         | Description                                                                                                                                                                                         |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Harbin_1977_TRR.csv`        | Final community-level calculation table. Includes recovery times at 85%, 90%, and 95% criteria, snowfall and environmental severity, recovery burden, TRR, status, normalization scope, and bounds. |
| `Community_Road_Mapping.csv` | Community-to-road matching table, with community/road identifiers, OSM identifiers, road weights, and distance to the community AOI.                                                                |
| `Road_Station_Recovery.csv`  | Road–station recovery calculations, recovery duration, status, and observation counts.                                                                                                              |
| `Road_weight_audit.csv`      | Road-class counts and recalculated functional weights.                                                                                                                                              |
| `Run_report.json`            | Input filenames and SHA-256 hashes, buffer setting, matching and baseline rules, status counts, normalization formula, scope, and bounds.                                                           |
| `TRR_quality_control.png`    | Diagnostic figure generated by the Harbin notebook for recovery-time and TRR distributions.                                                                                                         |

## 3. TRR calculation and reverse normalization

For each eligible road, the baseline is the same-hour mean over seven days preceding snowfall onset. Recovery time is measured from the matched station’s snowfall cessation to the first observed return to the specified fraction of baseline speed. The primary criterion is 90%; 85% and 95% are also exported.

Roads intersecting the community boundary or its 1 km outward buffer are selected. Recovery times are aggregated using road-hierarchy weights. In the supplied Harbin audit, trunk and primary roads/links have weight 0.4, secondary and tertiary roads/links have weight 0.3, and residential, service, living-street and unclassified roads have weight 0.2. Unweighted classes are excluded from this motorized-road calculation.

The severity-adjusted community recovery burden is:

```text
B_c = T90_c / E_c
```

The reverse-normalized outcome is:

```text
TRR_c = 100 × [1 − (B_c − B_min) / (B_max − B_min)]
```

Higher TRR therefore represents lower severity-adjusted recovery burden and stronger traffic-recovery performance. The computation is traceable through the original burden column, `TRR_Bmin`, `TRR_Bmax`, `TRR_normalization_scope`, the notebook outputs, and `Run_report.json`.

**Scope of the saved Harbin example:** `normalization_scope` is `harbin`. Its recorded bounds are approximately 0.2362609643 and 15.0454379044. This output demonstrates reverse normalization within Harbin. For the manuscript’s three-city outcome, merge the original burdens from all three cities and recalculate TRR with common pooled bounds; the machine-learning notebook places this operation before subsequent analyses.

## 4. Sample availability and reproduction

To reproduce the Harbin example, open `Harbin_TRR_RAR_Colabzz.ipynb` in Colab, upload `Harbin_TRR_1km_Package.rar`, run the cells in order, and download the result ZIP. Check the newly generated report against the saved working record.

To run the three-city models, use the replacement panel containing the original community-type labels, open `Three_City_TRR_Analysis_Colab.ipynb`, and upload the panel plus the Shenyang, Harbin, and Changchun calculation-result CSVs. The notebook verifies city membership by `Community_ID`, calculates pooled TRR, and then trains the models. Inspect the normalization audit, sample counts, and train/test membership files alongside the resulting metrics and SHAP figures.
