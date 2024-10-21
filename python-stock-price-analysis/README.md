# python-stock-price-analysis

`python-stock-price-analysis.ipynb` - Jupyter notebook demonstrates analyzing financial stock prices using data in Snowflake via the Snowflake Connector for Python. This notebook works while running in Workbench (using the managed credentials).

## Setup Steps

Ensure you have followed the projects main README.

Ensure you have access to the database/schema/table `FINANCIAL__ECONOMIC_ESSENTIALS.CYBERSYN.STOCK_PRICE_TIMESERIES` in Snowflake.

## Usage

Setup the `venv` environment:

```bash
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

Open the notebook and either run entirely or cell by cell.