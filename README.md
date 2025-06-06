
## Setup
```shell
# should install requirements.txt packages
$ pip3 install -e stock-analysis # path to top level where setup.py is

# if not, install them explicitly
$ pip3 install -r requirements.txt
```

## Usage
This section will show some of the functionality of each class; however, it is by no means exhaustive.

### Getting data
```python
from stock_analysis import StockReader

reader = StockReader('2017-01-01', '2018-12-31')

# get bitcoin data in USD
bitcoin = reader.get_bitcoin_data('USD')

# get faang data
fb, aapl, amzn, nflx, goog = (
    reader.get_ticker_data(ticker)
    for ticker in ['META', 'AAPL', 'AMZN', 'NFLX', 'GOOG']
)

# get S&P 500 data
sp = reader.get_index_data('S&P 500')
```

### Grouping data
```python
from stock_analysis.utils import group_stocks, describe_group

faang = group_stocks(
    {
        'Facebook': fb,
        'Apple': aapl,
        'Amazon': amzn,
        'Netflix': nflx,
        'Google': goog
    }
)

# describe the group
describe_group(faang)
```

### Building a portfolio
Groups assets by date and sums columns to build a portfolio.
```python
from stock_analysis.utils import make_portfolio

faang_portfolio = make_portfolio(faang)
```

### Visualizing data
Be sure to check out the other methods here for different plot types, reference lines, shaded regions, and more!

#### Single asset
Evolution over time:
```python
import matplotlib.pyplot as plt
from stock_analysis import StockVisualizer

netflix_viz = StockVisualizer(nflx)

ax = netflix_viz.evolution_over_time(
    'close',
    figsize=(10, 4),
    legend=False,
    title='Netflix closing price over time'
)
netflix_viz.add_reference_line(
    ax,
    x=nflx.high.idxmax(),
    color='k',
    linestyle=':',
    label=f'highest value ({nflx.high.idxmax():%b %d})',
    alpha=0.5
)
ax.set_ylabel('price ($)')
plt.show()
```

<img src="images/netflix_line_plot.png?raw=true" align="center" width="600" alt="line plot with reference line">

After hours trades:
```python
netflix_viz.after_hours_trades()
plt.show()
```

<img src="images/netflix_after_hours_trades.png?raw=true" align="center" width="800" alt="after hours trades plot">

Differential in closing price versus another asset:
```python
netflix_viz.fill_between_other(fb)
plt.show()
```
<img src="images/nflx_vs_fb_closing_price.png?raw=true" align="center" width="600" alt="differential between NFLX and FB">

Candlestick plots with resampling (uses `mplfinance`):
```python
netflix_viz.candlestick(resample='2W', volume=True, xrotation=90, datetime_format='%Y-%b -')
```

<img src="images/candlestick.png?raw=true" align="center" width="600" alt="resampled candlestick plot">

*Note: run `help()` on `StockVisualizer` for more visualizations*

#### Asset groups
Correlation heatmap:
```python
from stock_analysis import AssetGroupVisualizer

faang_viz = AssetGroupVisualizer(faang)
faang_viz.heatmap(True)
```

<img src="images/faang_heatmap.png?raw=true" align="center" width="450" alt="correlation heatmap">

*Note: run `help()` on `AssetGroupVisualizer` for more visualizations. This object has many of the visualizations of the `StockVisualizer` class.*

### Analyzing data
Below are a few of the metrics you can calculate.

#### Single asset
```python
from stock_analysis import StockAnalyzer

nflx_analyzer = StockAnalyzer(nflx)
nflx_analyzer.annualized_volatility()
```

#### Asset group
Methods of the `StockAnalyzer` class can be accessed by name with the `AssetGroupAnalyzer` class's `analyze()` method.
```python
from stock_analysis import AssetGroupAnalyzer

faang_analyzer = AssetGroupAnalyzer(faang)
faang_analyzer.analyze('annualized_volatility')

faang_analyzer.analyze('beta', index=sp)
```

### Modeling
```python
from stock_analysis import StockModeler
```

#### Time series decomposition
```python
decomposition = StockModeler.decompose(nflx, 20)
fig = decomposition.plot()
plt.show()
```

<img src="images/nflx_ts_decomposition.png?raw=true" align="center" width="450" alt="time series decomposition">

#### ARIMA
Build the model:
```python
arima_model = StockModeler.arima(nflx, ar=10, i=1, ma=5)
```

Check the residuals:
```python
StockModeler.plot_residuals(arima_model)
plt.show()
```

<img src="images/arima_residuals.png?raw=true" align="center" width="650" alt="ARIMA residuals">

Plot the predictions:
```python
arima_ax = StockModeler.arima_predictions(
    nflx, arima_model,
    start='2019-01-01', end='2019-01-07',
    title='ARIMA'
)
plt.show()
```

<img src="images/arima_predictions.png?raw=true" align="center" width="450" alt="ARIMA predictions">

#### Linear regression
Build the model:
```python
X, Y, lm = StockModeler.regression(nflx)
```

Check the residuals:
```python
StockModeler.plot_residuals(lm)
plt.show()
```

<img src="images/lm_residuals.png?raw=true" align="center" width="650" alt="linear regression residuals">

Plot the predictions:
```python
linear_reg = StockModeler.regression_predictions(
    nflx, lm,
    start='2019-01-01', end='2019-01-07',
    title='Linear Regression'
)
plt.show()
```

<img src="images/lm_predictions.png?raw=true" align="center" width="450" alt="linear regression predictions">

---


