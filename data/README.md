# Source data

`XAUUSD_H1.csv` is the supplied headerless OHLCV file. Columns are timestamp, open, high, low, close, and volume. The supplied file does not specify timezone or vendor volume semantics.

Before publishing the raw CSV in a public GitHub repository, verify that the data provider's terms permit redistribution. The root `.gitignore` excludes `data/*.csv` by default. If redistribution is permitted, remove that ignore rule or explicitly force-add the file.
