# MotoGP Qualifying Time Prediction

How closely can practice lap times predict a MotoGP rider’s qualifying pace? This project uses `FP1`, `PR`, and `FP2` lap times, session conditions, and rider and team information to estimate each rider’s qualifying lap time (`Q`) in seconds. It then sorts the predictions to show the expected order by lap time.

## Data

- The historical workbook contains 1,199 rider-weekend records across 54 race weekends from 2024 to 2026 in `Arkusz1`. The training pipeline uses 1,037 complete records after converting lap times to seconds and filtering for dry qualifying sessions. [Historical results](Data/MotoGP%20Data.ods)
- The Portugal 2026 workbook covers 22 riders at the Autódromo Internacional do Algarve. `Arkusz1` contains the prediction inputs, including `Constructor`. `Arkusz2` also contains the recorded qualifying times. [Portugal 2026 test set](Test/PR_2026.ods)

## Results

| Evaluation | Model | MAE | RMSE |
| --- | --- | ---: | ---: |
| Random 80/20 split | Random Forest | 0.263 s | 0.341 s |
| Random 80/20 split | CatBoost (initial fit) | 0.286 s | 0.391 s |
| 7-fold cross-validation grouped by season and weekend | Random Forest | 0.457 ± 0.284 s | — |
| 7-fold cross-validation grouped by season and weekend | CatBoost | 0.494 ± 0.258 s | — |
| Held-out weekend (Portugal 2026) | Random Forest | 2.443 s | 3.955 s |

These results come from the saved notebook outputs. MAE is the average absolute error per rider, and RMSE gives larger errors more weight. The grouped experiments use only `FP1`, `PR`, and `FP2` and report the mean and standard deviation of MAE across folds. Grouping keeps all riders from a race weekend together within each split. The random split uses additional features and can place riders from the same weekend on both sides of the split.

With a small dataset, averaging predictions across many trees may have made Random Forest less sensitive to noise, helping explain its lower validation error compared with CatBoost.

The Portugal test covers a weekend with wet FP2 and dry qualifying. The model predicted times between 98.451 and 99.591 s, while the recorded times ranged from 97.201 to 111.233 s. The largest absolute error was 12.673 s. The Portugal 2026 test was an edge case because track conditions changed substantially between wet FP2 and dry qualifying. [Portugal notebook results](predict.ipynb)

## Repository structure

| File | Purpose |
| --- | --- |
| [`Main.ipynb`](Main.ipynb) | Exploratory analysis, Random Forest and CatBoost experiments, practice-time baselines, and grouped cross-validation. |
| [`train_model.ipynb`](train_model.ipynb) | Fits a scikit-learn pipeline with one-hot encoding and a 300-tree Random Forest, then saves the model and feature list with joblib. |
| [`predict.ipynb`](predict.ipynb) | Generates lap-time predictions and compares them with recorded times and the rankings derived from those times. |

## Run locally

The notebooks read the workbooks in the repository root. These are identical to the copies in `Data/` and `Test/`.

1. Run `train_model.ipynb` to train on `MotoGP Data.ods` and generate `motogp_rf_model.pkl` and `features.pkl`.
2. Run `predict.ipynb` through the ranking comparison. It reads `PR_2026.ods` and shows the predicted lap times, MAE, RMSE, and rankings. The comparison expects the two sheets to list riders in the same order. To plot errors for the corresponding riders, replace the assignments to `plot_data` and `plot_data["Error_ms"]` in the final plotting cell with this line.

   ```python
   plot_data = new_data.assign(Error_ms=new_data["Error"] * 1000).sort_values("Actual_Position")
   ```

3. Run `Main.ipynb` to explore the data and reproduce the model comparisons.

For a new event, use the columns listed in `features.pkl`, including `Constructor` and the qualifying-session conditions. To make predictions ahead of qualifying, supply weather estimates or assumptions for those conditions. Use a second sheet with measured times in the `Q` column if you want to run the evaluation cells.

## To-do
- [ ] Collect more data
- [ ] Improve dataset and features such as: delta times between drives, positions in each session etc.
- [ ] Account for changing sessions condition
