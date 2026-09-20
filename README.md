# Stock Price Prediction (Google Finance + Linear Regression)

Predicts the future closing prices of a portfolio of NSE stocks using a Linear Regression model, built in Google Colab.

The stock data lives in an Excel / Google Sheet that pulls prices live from **Google Finance**. Enter the Google Finance company ID (e.g. `NSE:INFY`) in the sheet, run the notebook, and get the price forecast for each stock. No code changes needed.

![Forecast](images/forecast.png)

## How it works

The notebook follows the same workflow as my [Ford car price prediction](FordCarPredication.ipynb) project:

1. Load data and inspect it (`shape`, `info`, `describe`, `isnull`)
2. EDA: price history, return distributions, correlation heatmap, boxplots, scatter plots
3. Feature engineering (needed for time series): lag prices, 5 and 10 day moving averages, volatility, day of week, month
4. Split into `X` and `y`. The target is the **next trading day's closing price**
5. One-hot encoding and label encoding of the `Stock` column
6. Scaling with `StandardScaler`
7. Train/test split (33% test)
8. Linear Regression for both encodings, evaluated with R² and adjusted R²
9. Recursive forecast of the next 30 trading days for every stock, saved to `stock_forecasts.csv`

The test set is the **most recent** 33% of dates (`shuffle=False`), so the model is never trained on days that come after the days it is tested on.

## The Excel sheet is dynamic

`Test_Trading.xlsx` is exported from Google Sheets. Every price cell is a `GOOGLEFINANCE` formula, and each stock block is laid out as `Closing Price | Return Rate | Units`.

The company ID is the only thing you change to track a different stock:

1. Open the sheet in Google Sheets.
2. Replace the old ticker with the new Google Finance ID (e.g. `NSE:ADANIPOWER` → `NSE:TATAMOTORS`). The ticker sits inside the formulas, so use **Edit → Find and replace**, tick **Also search within formulas**, and replace it across the sheet. The company name in row 1 and all prices update automatically.
3. To add newer days, put the new dates in column A and fill the formulas down.
4. Download it as **File → Download → Microsoft Excel (.xlsx)**. Downloading from Google Sheets stores the fetched prices in the file, which is what the notebook reads.
5. Run the notebook. It reads the company names and prices from the sheet, so nothing in the code changes.

The notebook detects each stock by the `Closing Price` header and takes the company name from row 1. The portfolio columns and summary table at the bottom of the sheet are ignored.

## Quick start (Google Colab)

1. Open `StockPricePrediction.ipynb` in [Google Colab](https://colab.research.google.com) (**File → Upload notebook**).
2. Click **Runtime → Run all**.
3. When the first cell asks, upload your `Test_Trading.xlsx`.
4. Scroll to the end for the forecast table and plots. `stock_forecasts.csv` downloads automatically.

## Configuration

Both settings are in the forecasting cell:

| Setting | Default | Meaning |
|---|---|---|
| `FORECAST_DAYS` | 30 | Trading days (Mon to Fri) to forecast ahead |
| `MIN_ROWS` | 60 | Stocks with less history than this are skipped. They are still used for training |

## Sample results

Data: 5 stocks, 2026-02-17 to 2026-09-09. Train up to 2026-07-07, test from 2026-07-08.

| Model | R² | Adjusted R² |
|---|---|---|
| Linear Regression (one-hot) | 0.9995 | 0.9994 |
| Linear Regression (label encoded) | 0.9995 | 0.9994 |

Comparison with a naive baseline ("tomorrow's price = today's price") on the test period:

| | MAE | RMSE |
|---|---|---|
| Linear Regression | 9.82 | 15.67 |
| Naive baseline | 9.45 | 15.36 |

## Limitations

- R² is very high because "tomorrow ≈ today" already explains most of the variance. The model is about as accurate as the naive baseline, so compare errors, not R² alone.
- Forecasts are recursive: each predicted day feeds the next, so errors build up. Later days are rough trend estimates. They look like smooth curves that drift back toward recent averages, while real prices are noisier.
- ideaForge has only 38 rows of data (it ends 2026-04-16), so no forecast is produced for it by default.
- Market holidays are not removed from the forecast dates.
- This is a learning project. It is **not** investment advice.

## Project structure

```
├── StockPricePrediction.ipynb   # main notebook
├── Test_Trading.xlsx            # data (Google Finance driven)
├── FordCarPredication.ipynb     # the earlier project this workflow is based on
├── images/
│   └── forecast.png
└── stock_forecasts.csv          # generated when you run the notebook
```

## Tech stack

Python, pandas, NumPy, scikit-learn, seaborn, matplotlib, Google Colab, Google Sheets (`GOOGLEFINANCE`)

## Author

Atharva · [GitHub](https://github.com/batharva)
