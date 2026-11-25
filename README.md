# Binance daily crypto data, 2019 to 2024

Daily prices and volumes of 524 Binance USDT pairs from 2019-01-01 to 2024-12-31 (UTC days), with a cleaned set of returns. The source is the Binance public spot archive.

To get started, use `processed/core_returns.csv`: daily log returns of 20 actively traded coins.

## Files

| File | Content |
|---|---|
| `processed/core_returns.csv` | Daily log returns of the 20 coins |
| `processed/core_close.csv` | Closing prices of the 20 coins |
| `processed/returns.parquet` | Daily log returns of all 455 coins |
| `processed/close.parquet` | Closing prices of all 455 coins |
| `processed/universe.csv` | One row per pair: type, dates, gaps, and whether it is one of the 20 |
| `raw/ohlcv_daily.parquet` | Open, high, low, close, volume and USDT volume of all 524 pairs, unedited |
| `raw/MANIFEST.csv` | Source file, checksum and odd-value counts for each pair |
| `raw/COLLECTION.json` | Row counts and checksum of the raw file |

The price and return tables have one row per day (2,192 days) and one column per coin.

## Loading

```python
import pandas as pd

core = pd.read_csv("processed/core_returns.csv", index_col="date", parse_dates=True)
everything = pd.read_parquet("processed/returns.parquet")  # needs pyarrow
```

## Cleaning

- The price and return tables hold ordinary coins only. Leveraged tokens (50), stablecoins (11), pegged tokens (5) and fiat currencies (3) remain in the raw file.
- Nothing is filled in. A missing price stays missing, and a return needs two traded days in a row.
- Returns are logarithmic. Use `exp(r) - 1` before averaging them across coins in a portfolio.

## The 20 coins

BTC, ETH, BNB, SOL, XRP, ADA, AVAX, DOGE, MATIC, NEAR, DOT, TRX, FTM, ATOM, LINK, SAND, LTC, ETC, FIL, MANA.

They are chosen with data up to 2022-12-31 only: at least 730 traded days, no missing day in the last 365, then the highest median daily USDT volume over those 365 days. All values of these 20 coins were compared with the files Binance publishes, and none differed.

Suggested split: train to the end of 2022, validate on 2023, test on 2024.

## Good to know

- All 20 coins have returns together from 2020-10-16 to 2024-09-10, which is 1,426 days.
- `MATICUSDT` ends on 2024-09-10, when the coin was renamed POL. After that, 19 of the 20 coins remain. The two series are not joined.
- `LUNAUSDT` is a different coin after its gap in May 2022. `FTTUSDT` has a gap of 311 days until September 2023.
- `BTTUSDT` has negative volumes on five days in early 2019 at the source. Those days are missing in the tables.
- The download kept only pairs averaging at least 1 million USDT a day over the whole period, so the 455 coins lean toward those that stayed actively traded. This does not affect the 20 coins.
- `in_h5_433` marks the 433 pairs used in an earlier project.
