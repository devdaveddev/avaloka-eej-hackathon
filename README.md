# Can solar and geomagnetic drivers predict the equatorial electrojet?

Avaloka Engineering Intern Hackathon, 14–15 November 2026. Analysis run in [Avaloka](https://avaloka.ai).

## Question

Can solar and geomagnetic drivers predict hourly `delta_EEJ` (equatorial electrojet strength) better than a time-of-day baseline, and which drivers matter?

## Answer

No. At hourly resolution the drivers do not beat a baseline that only knows the hour of day, and adding them to a time-only model makes the error worse.

## Dataset

`eej_hourly_2001_2021.csv` — 145,188 rows, 16 columns, 9.0 MB, no missing values.

- Hourly means of one-minute data for 2001–2021 (no data for 2012). Hours with fewer than 30 minutes of data are dropped.
- Time is UT.

| Column | Meaning |
|---|---|
| `Year`, `Month`, `Day`, `DOY`, `Hour` | Date and hour (UT); `DOY` is day of year |
| `F107_daily` | Daily F10.7 solar radio flux |
| `IEF` | Interplanetary electric field |
| `Pdyn` | Solar wind dynamic pressure |
| `Speed` | Solar wind speed |
| `Bmag`, `BY`, `BZ_GSM` | Interplanetary magnetic field magnitude and components |
| `SYM_H` | SYM-H geomagnetic index |
| `AE` | Auroral electrojet index |
| `storm_flag` | 1 if the hour falls in a geomagnetic storm interval |
| `delta_EEJ` | Target: equatorial electrojet strength |

## Method

All models were trained on 2001–2018 and tested on 2019–2021 (22,525 test hours). The split is chronological, with no shuffling, so neighbouring hours cannot leak between train and test.

Two questions were put to Avaloka in plain language:

1. Can the nine drivers predict `delta_EEJ` better than the mean `delta_EEJ` for each hour?
2. Do the drivers add anything once hour and day of year are already features?

Avaloka wrote and ran gradient-boosted tree models for both.

## Results from Avaloka

Test RMSE, lower is better.

**Run 1: drivers only**

| Model | Test RMSE |
|---|---|
| Per-hour mean baseline | 8.92 |
| Gradient-boosted trees, 9 drivers | 10.44 |

Top drivers by importance: `BZ_GSM` (0.248), `Pdyn` (0.115), `BY` (0.050). `IEF` and `AE` were near zero (0.001 each).

**Run 2: time features, with and without drivers**

| Model | Test RMSE |
|---|---|
| Per-hour mean baseline | 9.63 |
| Hour + day of year | 9.44 |
| Hour + day of year + 9 drivers | 9.79 |

Driver importance in the last model: `F107_daily` 0.10, `SYM_H` 0.04, `Speed` 0.02, `AE` 0.02, the rest 0.01 or less.

## What this means

- The drivers-only model (10.44) is no better than predicting one constant: the overall training mean scores 10.46 on the same test years.
- Hour of day and season carry almost all the predictable signal at hourly resolution.
- Adding the drivers raises the error from 9.44 to 9.79, which suggests the model fits driver patterns from 2001–2018 that do not hold in 2019–2021.

## Checks

- The per-hour baseline of 8.92 in run 1 was recomputed independently with pandas on the full file and matches exactly.
- The same baseline was reported as 9.63 in run 2. This disagrees with run 1 and with the independent check, and the cause was not traced. Treat the run 2 baseline with caution; the comparison between the two models in run 2 is unaffected.
- The model RMSE values are Avaloka's and were not independently reproduced.

## What broke or was confusing

- Avaloka's first training plan was a 30-epoch neural network. It failed with "Script execution timed out after 60 seconds" and had to be replaced with gradient-boosted trees.
- The per-hour baseline changed between two runs on the same data and split (8.92, then 9.63).
- Auto Insights focused on `storm_flag` and the date columns and did not pick out `delta_EEJ` as the quantity of interest.
