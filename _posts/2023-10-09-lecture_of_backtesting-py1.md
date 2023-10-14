---
layout: single
# classes: wide
title : "Backtesting.py 사용법"
categories: portfolio
tag: [python, backtesting]
toc: true
toc_label: "순서"
toc_icon: "cog"
author_profile: false
sidebar:
    nav: "docs"
Typora-root-url: ../




---

---





# backtesting.py -1

유투브 참고



https://www.youtube.com/watch?v=e4ytbIm2Xg0&ab_channel=ChadThackray

https://kernc.github.io/backtesting.py/doc/backtesting/backtesting.html#backtesting.backtesting.Order.limit


```python
import pandas as pd
import numpy as np
import seaborn as sns
from datetime import datetime
import matplotlib.pyplot as plt
```


```python
ticker = 'KODEX 200'
start_date = datetime(2010, 1, 1)
df = pd.read_pickle("../crawling/data/etf_price.pickle")
df.columns = ['NAV','Open','High','Low','Close','Volume','Amount','base','ticker','code','Date']
df['Date'] = pd.to_datetime(df['Date'])
cond = (df['ticker']==ticker) & (df['Date']>start_date)
data = df[cond].reset_index(drop=True)
data = data[['Date','Open','High','Low','Close','Volume']].set_index('Date')
for col in data.columns:
    data[col] = pd.to_numeric(data[col]).astype('float')
data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    DatetimeIndex: 3396 entries, 2010-01-04 to 2023-10-06
    Data columns (total 5 columns):
     #   Column  Non-Null Count  Dtype  
    ---  ------  --------------  -----  
     0   Open    3396 non-null   float64
     1   High    3396 non-null   float64
     2   Low     3396 non-null   float64
     3   Close   3396 non-null   float64
     4   Volume  3396 non-null   float64
    dtypes: float64(5)
    memory usage: 159.2 KB

### Strategy

- 전략을 설정하여 백테스트
- RSI 조건으로 매수, 매도




```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover

class MyStrategy(Strategy):

    upper_bound = 67
    lower_bound = 33
    rsi_window=14

    def init(self):
        self.rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)
    
    def next(self):
        if crossover(self.rsi, self.upper_bound):
            self.position.close()
        elif crossover(self.lower_bound, self.rsi):
            self.buy()
            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats = bt.run()
print(stats)
bt.plot()
```

    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   47.290931
    Equity Final [$]                  11091864.24
    Equity Peak [$]                   17417832.28
    Return [%]                          10.918642
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    0.771926
    Volatility (Ann.) [%]               13.341861
    Sharpe Ratio                         0.057857
    Sortino Ratio                         0.08228
    Calmar Ratio                          0.01836
    Max. Drawdown [%]                  -42.044703
    Avg. Drawdown [%]                   -3.154887
    Max. Drawdown Duration     2031 days 00:00:00
    Avg. Drawdown Duration       76 days 00:00:00
    # Trades                                   20
    Win Rate [%]                             70.0
    Best Trade [%]                      11.105913
    Worst Trade [%]                    -25.035937
    Avg. Trade [%]                       0.821925
    Max. Trade Duration         450 days 00:00:00
    Avg. Trade Duration         120 days 00:00:00
    Profit Factor                        1.551886
    Expectancy [%]                       1.124611
    SQN                                  0.228145
    _strategy                          MyStrategy
    _equity_curve                             ...
    _trades                       Size  EntryB...
    dtype: object




<div class="bk-root" id="7e90483b-0b25-454d-8aef-c5c4983a0f99" data-root-id="35273"></div>








<div style="display: table;"><div style="display: table-row;"><div style="display: table-cell;"><b title="bokeh.models.layouts.Row">Row</b>(</div><div style="display: table-cell;">id&nbsp;=&nbsp;'35273', <span id="35611" style="cursor: pointer;">&hellip;)</span></div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">align&nbsp;=&nbsp;'start',</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">aspect_ratio&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">background&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">children&nbsp;=&nbsp;[GridBox(id='35270', ...), ToolbarBox(id='35272', ...)],</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">cols&nbsp;=&nbsp;'auto',</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">css_classes&nbsp;=&nbsp;[],</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">disabled&nbsp;=&nbsp;False,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">height&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">height_policy&nbsp;=&nbsp;'auto',</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">js_event_callbacks&nbsp;=&nbsp;{},</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">js_property_callbacks&nbsp;=&nbsp;{},</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">margin&nbsp;=&nbsp;(0, 0, 0, 0),</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">max_height&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">max_width&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">min_height&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">min_width&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">name&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">sizing_mode&nbsp;=&nbsp;'stretch_width',</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">spacing&nbsp;=&nbsp;0,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">subscribed_events&nbsp;=&nbsp;[],</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">syncable&nbsp;=&nbsp;True,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">tags&nbsp;=&nbsp;[],</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">visible&nbsp;=&nbsp;True,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">width&nbsp;=&nbsp;None,</div></div><div class="35610" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">width_policy&nbsp;=&nbsp;'auto')</div></div></div>
<script>
(function() {
  let expanded = false;
  const ellipsis = document.getElementById("35611");
  ellipsis.addEventListener("click", function() {
    const rows = document.getElementsByClassName("35610");
    for (let i = 0; i < rows.length; i++) {
      const el = rows[i];
      el.style.display = expanded ? "none" : "table-row";
    }
    ellipsis.innerHTML = expanded ? "&hellip;)" : "&lsaquo;&lsaquo;&lsaquo;";
    expanded = !expanded;
  });
})();
</script>




### Optimizer

지표의 하이퍼파라미터를 최적화

- 파라미터 : RSI upper_bound, lower_bound, rsi_window
- 최적화 조건 설정 : optim_func
- 결과 저장 : bt.plot(filnename설정)


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover

def optim_func(series):
    
    if series['# Trades'] < 30: #최소 트레이드 회수 제한
        return -1
    return series['Equity Final [$]'] / series['Exposure Time [%]']
    #return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14

    def init(self):
        self.rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)
    
    def next(self):
        if crossover(self.rsi, self.upper_bound):
            self.position.close()
        elif crossover(self.lower_bound, self.rsi):
            self.buy()
            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats = bt.optimize(
    upper_bound=range(50, 85, 1),
    lower_bound=range(10, 45, 1),
    rsi_window=range(10,30,1),
    maximize = optim_func, #'Sharpe Ratio',
    constraint = lambda param : param.upper_bound > param.lower_bound,
    max_tries=100 # randomly simulations, 오버피팅 방지.. 
    )

lower_bound = stats['_strategy'].lower_bound
upper_bound = stats['_strategy'].upper_bound
rsi_window = stats['_strategy'].rsi_window
print(lower_bound,"-", upper_bound,"-", rsi_window)
print(stats)

bt.plot()
#filename있으면 저장을합니다. 

# bt.plot(filename=f'plots/{datetime.now().strftime("%Y%m%d")}_{lower_bound}_{upper_bound}.html')
```

    /Users/byeongsikbu/opt/anaconda3/envs/dct/lib/python3.10/site-packages/backtesting/backtesting.py:1375: UserWarning: For multiprocessing support in `Backtest.optimize()` set multiprocessing start method to 'fork'.
      warnings.warn("For multiprocessing support in `Backtest.optimize()` "



      0%|          | 0/9 [00:00<?, ?it/s]


    36 - 53 - 16
    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   25.912839
    Equity Final [$]                  12215340.55
    Equity Peak [$]                    12568343.2
    Return [%]                          22.153406
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    1.495976
    Volatility (Ann.) [%]               11.227801
    Sharpe Ratio                         0.133239
    Sortino Ratio                        0.192884
    Calmar Ratio                         0.046011
    Max. Drawdown [%]                  -32.513102
    Avg. Drawdown [%]                   -3.777746
    Max. Drawdown Duration     1934 days 00:00:00
    Avg. Drawdown Duration      136 days 00:00:00
    # Trades                                   39
    Win Rate [%]                        64.102564
    Best Trade [%]                       7.401241
    Worst Trade [%]                      -9.27555
    Avg. Trade [%]                       0.854879
    Max. Trade Duration         105 days 00:00:00
    Avg. Trade Duration          33 days 00:00:00
    Profit Factor                        1.859875
    Expectancy [%]                       0.920323
    SQN                                  0.928929
    _strategy                 MyStrategy(upper...
    _equity_curve                             ...
    _trades                       Size  EntryB...
    dtype: object




<div class="bk-root" id="924a4041-b592-4085-99eb-30276e81c48d" data-root-id="51852"></div>








<div style="display: table;"><div style="display: table-row;"><div style="display: table-cell;"><b title="bokeh.models.layouts.Row">Row</b>(</div><div style="display: table-cell;">id&nbsp;=&nbsp;'51852', <span id="52190" style="cursor: pointer;">&hellip;)</span></div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">align&nbsp;=&nbsp;'start',</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">aspect_ratio&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">background&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">children&nbsp;=&nbsp;[GridBox(id='51849', ...), ToolbarBox(id='51851', ...)],</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">cols&nbsp;=&nbsp;'auto',</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">css_classes&nbsp;=&nbsp;[],</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">disabled&nbsp;=&nbsp;False,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">height&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">height_policy&nbsp;=&nbsp;'auto',</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">js_event_callbacks&nbsp;=&nbsp;{},</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">js_property_callbacks&nbsp;=&nbsp;{},</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">margin&nbsp;=&nbsp;(0, 0, 0, 0),</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">max_height&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">max_width&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">min_height&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">min_width&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">name&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">sizing_mode&nbsp;=&nbsp;'stretch_width',</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">spacing&nbsp;=&nbsp;0,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">subscribed_events&nbsp;=&nbsp;[],</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">syncable&nbsp;=&nbsp;True,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">tags&nbsp;=&nbsp;[],</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">visible&nbsp;=&nbsp;True,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">width&nbsp;=&nbsp;None,</div></div><div class="52189" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">width_policy&nbsp;=&nbsp;'auto')</div></div></div>
<script>
(function() {
  let expanded = false;
  const ellipsis = document.getElementById("52190");
  ellipsis.addEventListener("click", function() {
    const rows = document.getElementsByClassName("52189");
    for (let i = 0; i < rows.length; i++) {
      const el = rows[i];
      el.style.display = expanded ? "none" : "table-row";
    }
    ellipsis.innerHTML = expanded ? "&hellip;)" : "&lsaquo;&lsaquo;&lsaquo;";
    expanded = !expanded;
  });
})();
</script>





```python
display(stats['_strategy'])
display(stats['_trades'])
```


    <Strategy MyStrategy(upper_bound=53,lower_bound=36,rsi_window=16)>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>466</td>
      <td>20</td>
      <td>44</td>
      <td>21427.77</td>
      <td>22180.0</td>
      <td>350539.18</td>
      <td>0.035105</td>
      <td>2010-02-01</td>
      <td>2010-03-09</td>
      <td>36 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>474</td>
      <td>87</td>
      <td>110</td>
      <td>21803.52</td>
      <td>22150.0</td>
      <td>164231.52</td>
      <td>0.015891</td>
      <td>2010-05-10</td>
      <td>2010-06-14</td>
      <td>35 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>392</td>
      <td>278</td>
      <td>304</td>
      <td>26813.52</td>
      <td>27075.0</td>
      <td>102500.16</td>
      <td>0.009752</td>
      <td>2011-02-14</td>
      <td>2011-03-23</td>
      <td>37 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>432</td>
      <td>444</td>
      <td>22234.38</td>
      <td>23880.0</td>
      <td>1645.62</td>
      <td>0.074012</td>
      <td>2011-09-27</td>
      <td>2011-10-14</td>
      <td>17 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>419</td>
      <td>398</td>
      <td>444</td>
      <td>25280.46</td>
      <td>23880.0</td>
      <td>-586792.74</td>
      <td>-0.055397</td>
      <td>2011-08-05</td>
      <td>2011-10-14</td>
      <td>70 days</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1</td>
      <td>604</td>
      <td>614</td>
      <td>23997.90</td>
      <td>25500.0</td>
      <td>1502.10</td>
      <td>0.062593</td>
      <td>2012-06-05</td>
      <td>2012-06-20</td>
      <td>15 days</td>
    </tr>
    <tr>
      <th>6</th>
      <td>385</td>
      <td>588</td>
      <td>614</td>
      <td>25991.88</td>
      <td>25500.0</td>
      <td>-189373.80</td>
      <td>-0.018924</td>
      <td>2012-05-11</td>
      <td>2012-06-20</td>
      <td>40 days</td>
    </tr>
    <tr>
      <th>7</th>
      <td>413</td>
      <td>631</td>
      <td>643</td>
      <td>23782.47</td>
      <td>24550.0</td>
      <td>316989.89</td>
      <td>0.032273</td>
      <td>2012-07-13</td>
      <td>2012-07-31</td>
      <td>18 days</td>
    </tr>
    <tr>
      <th>8</th>
      <td>407</td>
      <td>704</td>
      <td>726</td>
      <td>24919.74</td>
      <td>25250.0</td>
      <td>134415.82</td>
      <td>0.013253</td>
      <td>2012-10-29</td>
      <td>2012-11-28</td>
      <td>30 days</td>
    </tr>
    <tr>
      <th>9</th>
      <td>395</td>
      <td>803</td>
      <td>808</td>
      <td>26016.93</td>
      <td>26600.0</td>
      <td>230312.65</td>
      <td>0.022411</td>
      <td>2013-03-25</td>
      <td>2013-04-01</td>
      <td>7 days</td>
    </tr>
    <tr>
      <th>10</th>
      <td>411</td>
      <td>813</td>
      <td>841</td>
      <td>25601.10</td>
      <td>25880.0</td>
      <td>114627.90</td>
      <td>0.010894</td>
      <td>2013-04-08</td>
      <td>2013-05-20</td>
      <td>42 days</td>
    </tr>
    <tr>
      <th>11</th>
      <td>427</td>
      <td>858</td>
      <td>887</td>
      <td>24884.67</td>
      <td>24620.0</td>
      <td>-113014.09</td>
      <td>-0.010636</td>
      <td>2013-06-13</td>
      <td>2013-07-24</td>
      <td>41 days</td>
    </tr>
    <tr>
      <th>12</th>
      <td>422</td>
      <td>1016</td>
      <td>1029</td>
      <td>24909.72</td>
      <td>25670.0</td>
      <td>320838.16</td>
      <td>0.030521</td>
      <td>2014-02-05</td>
      <td>2014-02-24</td>
      <td>19 days</td>
    </tr>
    <tr>
      <th>13</th>
      <td>425</td>
      <td>1078</td>
      <td>1084</td>
      <td>25480.86</td>
      <td>26160.0</td>
      <td>288634.50</td>
      <td>0.026653</td>
      <td>2014-05-07</td>
      <td>2014-05-15</td>
      <td>8 days</td>
    </tr>
    <tr>
      <th>14</th>
      <td>435</td>
      <td>1177</td>
      <td>1206</td>
      <td>25571.04</td>
      <td>25025.0</td>
      <td>-237527.40</td>
      <td>-0.021354</td>
      <td>2014-10-01</td>
      <td>2014-11-13</td>
      <td>43 days</td>
    </tr>
    <tr>
      <th>15</th>
      <td>442</td>
      <td>1231</td>
      <td>1237</td>
      <td>24604.11</td>
      <td>25035.0</td>
      <td>190453.38</td>
      <td>0.017513</td>
      <td>2014-12-18</td>
      <td>2014-12-29</td>
      <td>11 days</td>
    </tr>
    <tr>
      <th>16</th>
      <td>456</td>
      <td>1242</td>
      <td>1247</td>
      <td>24288.48</td>
      <td>24890.0</td>
      <td>274293.12</td>
      <td>0.024766</td>
      <td>2015-01-07</td>
      <td>2015-01-14</td>
      <td>7 days</td>
    </tr>
    <tr>
      <th>17</th>
      <td>446</td>
      <td>1342</td>
      <td>1416</td>
      <td>25455.81</td>
      <td>24085.0</td>
      <td>-611381.26</td>
      <td>-0.053851</td>
      <td>2015-06-04</td>
      <td>2015-09-17</td>
      <td>105 days</td>
    </tr>
    <tr>
      <th>18</th>
      <td>443</td>
      <td>1456</td>
      <td>1462</td>
      <td>24243.39</td>
      <td>24740.0</td>
      <td>219998.23</td>
      <td>0.020484</td>
      <td>2015-11-17</td>
      <td>2015-11-25</td>
      <td>8 days</td>
    </tr>
    <tr>
      <th>19</th>
      <td>478</td>
      <td>1500</td>
      <td>1518</td>
      <td>22915.74</td>
      <td>23770.0</td>
      <td>408336.28</td>
      <td>0.037278</td>
      <td>2016-01-21</td>
      <td>2016-02-19</td>
      <td>29 days</td>
    </tr>
    <tr>
      <th>20</th>
      <td>474</td>
      <td>1575</td>
      <td>1590</td>
      <td>23982.87</td>
      <td>24435.0</td>
      <td>214309.62</td>
      <td>0.018852</td>
      <td>2016-05-16</td>
      <td>2016-06-07</td>
      <td>22 days</td>
    </tr>
    <tr>
      <th>21</th>
      <td>379</td>
      <td>1885</td>
      <td>1908</td>
      <td>30571.02</td>
      <td>31190.0</td>
      <td>234593.42</td>
      <td>0.020247</td>
      <td>2017-08-14</td>
      <td>2017-09-15</td>
      <td>32 days</td>
    </tr>
    <tr>
      <th>22</th>
      <td>368</td>
      <td>1972</td>
      <td>1977</td>
      <td>32129.13</td>
      <td>33070.0</td>
      <td>346240.16</td>
      <td>0.029284</td>
      <td>2017-12-22</td>
      <td>2018-01-03</td>
      <td>12 days</td>
    </tr>
    <tr>
      <th>23</th>
      <td>386</td>
      <td>2003</td>
      <td>2023</td>
      <td>31507.89</td>
      <td>32460.0</td>
      <td>367514.46</td>
      <td>0.030218</td>
      <td>2018-02-08</td>
      <td>2018-03-13</td>
      <td>33 days</td>
    </tr>
    <tr>
      <th>24</th>
      <td>412</td>
      <td>2089</td>
      <td>2137</td>
      <td>30380.64</td>
      <td>29860.0</td>
      <td>-214503.68</td>
      <td>-0.017137</td>
      <td>2018-06-20</td>
      <td>2018-08-28</td>
      <td>69 days</td>
    </tr>
    <tr>
      <th>25</th>
      <td>444</td>
      <td>2165</td>
      <td>2202</td>
      <td>27760.41</td>
      <td>27525.0</td>
      <td>-104522.04</td>
      <td>-0.008480</td>
      <td>2018-10-12</td>
      <td>2018-12-04</td>
      <td>53 days</td>
    </tr>
    <tr>
      <th>26</th>
      <td>437</td>
      <td>2305</td>
      <td>2328</td>
      <td>27945.78</td>
      <td>27330.0</td>
      <td>-269095.86</td>
      <td>-0.022035</td>
      <td>2019-05-09</td>
      <td>2019-06-12</td>
      <td>34 days</td>
    </tr>
    <tr>
      <th>27</th>
      <td>455</td>
      <td>2366</td>
      <td>2388</td>
      <td>26262.42</td>
      <td>26250.0</td>
      <td>-5651.10</td>
      <td>-0.000473</td>
      <td>2019-08-05</td>
      <td>2019-09-05</td>
      <td>31 days</td>
    </tr>
    <tr>
      <th>28</th>
      <td>418</td>
      <td>2502</td>
      <td>2540</td>
      <td>28536.96</td>
      <td>25890.0</td>
      <td>-1106429.28</td>
      <td>-0.092756</td>
      <td>2020-02-25</td>
      <td>2020-04-20</td>
      <td>55 days</td>
    </tr>
    <tr>
      <th>29</th>
      <td>358</td>
      <td>2672</td>
      <td>2676</td>
      <td>30270.42</td>
      <td>32220.0</td>
      <td>697949.64</td>
      <td>0.064405</td>
      <td>2020-11-02</td>
      <td>2020-11-06</td>
      <td>4 days</td>
    </tr>
    <tr>
      <th>30</th>
      <td>276</td>
      <td>2869</td>
      <td>2934</td>
      <td>41753.34</td>
      <td>39885.0</td>
      <td>-515661.84</td>
      <td>-0.044747</td>
      <td>2021-08-17</td>
      <td>2021-11-23</td>
      <td>98 days</td>
    </tr>
    <tr>
      <th>31</th>
      <td>293</td>
      <td>2940</td>
      <td>2945</td>
      <td>37620.09</td>
      <td>39975.0</td>
      <td>689988.63</td>
      <td>0.062597</td>
      <td>2021-12-01</td>
      <td>2021-12-08</td>
      <td>7 days</td>
    </tr>
    <tr>
      <th>32</th>
      <td>311</td>
      <td>2978</td>
      <td>3021</td>
      <td>37580.01</td>
      <td>36925.0</td>
      <td>-203708.11</td>
      <td>-0.017430</td>
      <td>2022-01-25</td>
      <td>2022-04-01</td>
      <td>66 days</td>
    </tr>
    <tr>
      <th>33</th>
      <td>335</td>
      <td>3047</td>
      <td>3063</td>
      <td>34363.59</td>
      <td>35240.0</td>
      <td>293597.35</td>
      <td>0.025504</td>
      <td>2022-05-10</td>
      <td>2022-06-02</td>
      <td>23 days</td>
    </tr>
    <tr>
      <th>34</th>
      <td>363</td>
      <td>3070</td>
      <td>3103</td>
      <td>32474.82</td>
      <td>32500.0</td>
      <td>9140.34</td>
      <td>0.000775</td>
      <td>2022-06-14</td>
      <td>2022-07-29</td>
      <td>45 days</td>
    </tr>
    <tr>
      <th>35</th>
      <td>379</td>
      <td>3131</td>
      <td>3163</td>
      <td>31147.17</td>
      <td>29690.0</td>
      <td>-552267.43</td>
      <td>-0.046783</td>
      <td>2022-09-08</td>
      <td>2022-10-28</td>
      <td>50 days</td>
    </tr>
    <tr>
      <th>36</th>
      <td>381</td>
      <td>3209</td>
      <td>3214</td>
      <td>29513.91</td>
      <td>31345.0</td>
      <td>697645.29</td>
      <td>0.062042</td>
      <td>2023-01-03</td>
      <td>2023-01-10</td>
      <td>7 days</td>
    </tr>
    <tr>
      <th>37</th>
      <td>361</td>
      <td>3365</td>
      <td>3375</td>
      <td>33050.97</td>
      <td>34010.0</td>
      <td>346209.83</td>
      <td>0.029017</td>
      <td>2023-08-21</td>
      <td>2023-09-04</td>
      <td>14 days</td>
    </tr>
    <tr>
      <th>38</th>
      <td>381</td>
      <td>3394</td>
      <td>3395</td>
      <td>32299.47</td>
      <td>32060.0</td>
      <td>-91238.07</td>
      <td>-0.007414</td>
      <td>2023-10-05</td>
      <td>2023-10-06</td>
      <td>1 days</td>
    </tr>
  </tbody>
</table>
</div>



```python

```

### hemtmap

최적화 결과를 히트맵으로 표현하여, 어떤 파라미터가 유용했는지 시각적으로 확인가능


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14

    def init(self):
        self.rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)
    
    def next(self):
        if crossover(self.rsi, self.upper_bound):
            self.position.close()
        elif crossover(self.lower_bound, self.rsi):
            self.buy()
            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats, heatmap = bt.optimize(
    upper_bound=range(55, 85, 5),
    lower_bound=range(10, 45, 5),
    rsi_window=14,
    maximize = optim_func,#'Sharpe Ratio',
    constraint = lambda param : param.upper_bound > param.lower_bound,
    return_heatmap=True
    )

#print(heatmap)

hm = heatmap.groupby(['upper_bound', 'lower_bound']).mean().unstack()
#print(hm)
sns.heatmap(hm, annot=True, cmap='viridis')
plt.show()

lower_bound = stats['_strategy'].lower_bound
upper_bound = stats['_strategy'].upper_bound
rsi_window = stats['_strategy'].rsi_window
print("best :", lower_bound,"-", upper_bound,"-", rsi_window)
print(stats)

bt.plot()
display(stats['_trades'])
```

    /Users/byeongsikbu/opt/anaconda3/envs/dct/lib/python3.10/site-packages/backtesting/backtesting.py:1375: UserWarning: For multiprocessing support in `Backtest.optimize()` set multiprocessing start method to 'fork'.
      warnings.warn("For multiprocessing support in `Backtest.optimize()` "



      0%|          | 0/9 [00:00<?, ?it/s]




![output_46_0](/images/2023-10-09-lecture_of_backtesting-py1/output_11_2.png)
    


    best : 15 - 80 - 14
    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   16.872792
    Equity Final [$]                  21689992.91
    Equity Peak [$]                   21689992.91
    Return [%]                         116.899929
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    5.913693
    Volatility (Ann.) [%]                7.728531
    Sharpe Ratio                         0.765177
    Sortino Ratio                        1.343684
    Calmar Ratio                         0.611046
    Max. Drawdown [%]                    -9.67799
    Avg. Drawdown [%]                   -1.620457
    Max. Drawdown Duration      163 days 00:00:00
    Avg. Drawdown Duration       16 days 00:00:00
    # Trades                                    3
    Win Rate [%]                            100.0
    Best Trade [%]                      56.254118
    Worst Trade [%]                     13.836122
    Avg. Trade [%]                      29.464016
    Max. Trade Duration         506 days 00:00:00
    Avg. Trade Duration         280 days 00:00:00
    Profit Factor                             NaN
    Expectancy [%]                      30.694484
    SQN                                  1.966811
    _strategy                 MyStrategy(upper...
    _equity_curve                             ...
    _trades                      Size  EntryBa...
    dtype: object




<div class="bk-root" id="729189db-1458-449d-b8ce-3de8d0586982" data-root-id="63212"></div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>413</td>
      <td>1188</td>
      <td>1316</td>
      <td>24188.28</td>
      <td>27535.0</td>
      <td>1382195.36</td>
      <td>0.138361</td>
      <td>2014-10-20</td>
      <td>2015-04-24</td>
      <td>186 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>510</td>
      <td>1399</td>
      <td>1741</td>
      <td>22304.52</td>
      <td>27210.0</td>
      <td>2501794.80</td>
      <td>0.219932</td>
      <td>2015-08-25</td>
      <td>2017-01-12</td>
      <td>506 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>665</td>
      <td>2520</td>
      <td>2620</td>
      <td>20866.65</td>
      <td>32605.0</td>
      <td>7806002.75</td>
      <td>0.562541</td>
      <td>2020-03-20</td>
      <td>2020-08-13</td>
      <td>146 days</td>
    </tr>
  </tbody>
</table>
</div>

자체 내장되어 있는 heatmap을 사용하는 것도 가능

```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14

    def init(self):
        self.rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)
    
    def next(self):
        if crossover(self.rsi, self.upper_bound):
            self.position.close()
        elif crossover(self.lower_bound, self.rsi):
            self.buy()
            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats, heatmap = bt.optimize(
    upper_bound=range(55, 85, 5),
    lower_bound=range(10, 45, 5),
    rsi_window=range(10, 20, 2),
    maximize = optim_func,#'Sharpe Ratio',
    constraint = lambda param : param.upper_bound > param.lower_bound,
    return_heatmap=True
    )

#print(heatmap)

hm = heatmap.groupby(['upper_bound', 'lower_bound']).mean().unstack()
#print(hm)
# sns.heatmap(hm, annot=True, cmap='viridis')
# plt.show()
plot_heatmaps(heatmap, agg='mean')

lower_bound = stats['_strategy'].lower_bound
upper_bound = stats['_strategy'].upper_bound
rsi_window = stats['_strategy'].rsi_window
print("best :", lower_bound,"-", upper_bound,"-", rsi_window)
print(stats)

bt.plot()
display(stats['_trades'])
```

    /Users/byeongsikbu/opt/anaconda3/envs/dct/lib/python3.10/site-packages/backtesting/backtesting.py:1375: UserWarning: For multiprocessing support in `Backtest.optimize()` set multiprocessing start method to 'fork'.
      warnings.warn("For multiprocessing support in `Backtest.optimize()` "



      0%|          | 0/9 [00:00<?, ?it/s]




<div class="bk-root" id="1b12f3fc-48db-4c4f-8b61-46d731f4edf0" data-root-id="63685"></div>





    best : 15 - 80 - 14
    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   16.872792
    Equity Final [$]                  21689992.91
    Equity Peak [$]                   21689992.91
    Return [%]                         116.899929
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    5.913693
    Volatility (Ann.) [%]                7.728531
    Sharpe Ratio                         0.765177
    Sortino Ratio                        1.343684
    Calmar Ratio                         0.611046
    Max. Drawdown [%]                    -9.67799
    Avg. Drawdown [%]                   -1.620457
    Max. Drawdown Duration      163 days 00:00:00
    Avg. Drawdown Duration       16 days 00:00:00
    # Trades                                    3
    Win Rate [%]                            100.0
    Best Trade [%]                      56.254118
    Worst Trade [%]                     13.836122
    Avg. Trade [%]                      29.464016
    Max. Trade Duration         506 days 00:00:00
    Avg. Trade Duration         280 days 00:00:00
    Profit Factor                             NaN
    Expectancy [%]                      30.694484
    SQN                                  1.966811
    _strategy                 MyStrategy(upper...
    _equity_curve                             ...
    _trades                      Size  EntryBa...
    dtype: object




<div class="bk-root" id="5493617d-bbf3-4bce-86ee-5643a5d2e94e" data-root-id="64357"></div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>413</td>
      <td>1188</td>
      <td>1316</td>
      <td>24188.28</td>
      <td>27535.0</td>
      <td>1382195.36</td>
      <td>0.138361</td>
      <td>2014-10-20</td>
      <td>2015-04-24</td>
      <td>186 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>510</td>
      <td>1399</td>
      <td>1741</td>
      <td>22304.52</td>
      <td>27210.0</td>
      <td>2501794.80</td>
      <td>0.219932</td>
      <td>2015-08-25</td>
      <td>2017-01-12</td>
      <td>506 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>665</td>
      <td>2520</td>
      <td>2620</td>
      <td>20866.65</td>
      <td>32605.0</td>
      <td>7806002.75</td>
      <td>0.562541</td>
      <td>2020-03-20</td>
      <td>2020-08-13</td>
      <td>146 days</td>
    </tr>
  </tbody>
</table>
</div>



```python

```

### multiple strategies

- 다른 시간대의 지표를 활용하는 전략
- 예를들어, rsi는 weekly, ma는 daily를 사용하는 전략
- resample_apply


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14

    def init(self):
        self.daily_rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)
        self.weekly_rsi = resample_apply(
            "W-FRI", talib.RSI, self.data.Close, self.rsi_window)
    
    def next(self):
        if (crossover(self.daily_rsi, self.upper_bound) 
                and self.weekly_rsi[-1] > self.upper_bound):
            self.position.close()
        elif (crossover(self.lower_bound, self.daily_rsi)
                  and self.weekly_rsi[-1] < self.lower_bound
                    and self.daily_rsi[-1] < self.weekly_rsi[-1]):
            self.buy()
            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats = bt.run()
# stats, heatmap = bt.optimize(
#     upper_bound=range(55, 85, 5),
#     lower_bound=range(10, 45, 5),
#     rsi_window=range(10, 20, 2),
#     maximize = optim_func,#'Sharpe Ratio',
#     constraint = lambda param : param.upper_bound > param.lower_bound,
#     return_heatmap=True
#     )

#print(heatmap)

#hm = heatmap.groupby(['upper_bound', 'lower_bound']).mean().unstack()
#print(hm)
# sns.heatmap(hm, annot=True, cmap='viridis')
# plt.show()
#plot_heatmaps(heatmap, agg='mean')

# lower_bound = stats['_strategy'].lower_bound
# upper_bound = stats['_strategy'].upper_bound
# rsi_window = stats['_strategy'].rsi_window
# print("best :", lower_bound,"-", upper_bound,"-", rsi_window)
print(stats)

bt.plot()
display(stats['_trades'])
```

    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   47.055359
    Equity Final [$]                  13323664.95
    Equity Peak [$]                   14628839.95
    Return [%]                          33.236649
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    2.152193
    Volatility (Ann.) [%]               10.862179
    Sharpe Ratio                         0.198136
    Sortino Ratio                        0.291684
    Calmar Ratio                         0.109967
    Max. Drawdown [%]                  -19.571261
    Avg. Drawdown [%]                   -3.781761
    Max. Drawdown Duration     3859 days 00:00:00
    Avg. Drawdown Duration      157 days 00:00:00
    # Trades                                    2
    Win Rate [%]                            100.0
    Best Trade [%]                      21.092631
    Worst Trade [%]                     10.046459
    Avg. Trade [%]                      15.437495
    Max. Trade Duration        1984 days 00:00:00
    Avg. Trade Duration        1179 days 00:00:00
    Profit Factor                             NaN
    Expectancy [%]                      15.569545
    SQN                                  3.716169
    _strategy                          MyStrategy
    _equity_curve                             ...
    _trades                      Size  EntryBa...
    dtype: object




<div class="bk-root" id="59d0e857-a6c3-4b94-9c62-13449886e955" data-root-id="66167"></div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>444</td>
      <td>408</td>
      <td>1751</td>
      <td>22519.95</td>
      <td>27270.0</td>
      <td>2109022.20</td>
      <td>0.210926</td>
      <td>2011-08-22</td>
      <td>2017-01-26</td>
      <td>1984 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>415</td>
      <td>3142</td>
      <td>3395</td>
      <td>29133.15</td>
      <td>32060.0</td>
      <td>1214642.75</td>
      <td>0.100465</td>
      <td>2022-09-27</td>
      <td>2023-10-06</td>
      <td>374 days</td>
    </tr>
  </tbody>
</table>
</div>



```python

```

### Reversal Starategies

- long, short을 함께 하는 전략
- buy : 롱포지션, sell : 숏포지션
- close : 롱이든 숏이든 포지션 청산하는 것
- is_long, is_short : 포지션이 롱인지, 숏인지 확인 필요


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14

    def init(self):
        self.daily_rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)

    def next(self):
        if crossover(self.daily_rsi, self.upper_bound) :
            if self.position.is_long:
                #print(self.position.size)
                #print(self.position.pl_pct)
                self.position.close() # sell long positions 그런데, short position일 때도, 포지션을 청산함 >> 조심해야함 
                self.sell() # buy short positions
        elif crossover(self.lower_bound, self.daily_rsi):
            if self.position.is_short or not self.position:
                self.position.close() # sell short positions 마찬가지로 조심해야함
                self.buy() # buy long positions

bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats = bt.run()

print(stats)

bt.plot()
display(stats['_trades'])
```

    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                    99.26384
    Equity Final [$]                    8422052.5
    Equity Peak [$]                   20604231.69
    Return [%]                         -15.779475
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                   -1.266248
    Volatility (Ann.) [%]               20.348414
    Sharpe Ratio                              0.0
    Sortino Ratio                             0.0
    Calmar Ratio                              0.0
    Max. Drawdown [%]                  -64.240721
    Avg. Drawdown [%]                   -3.076568
    Max. Drawdown Duration     2478 days 00:00:00
    Avg. Drawdown Duration       55 days 00:00:00
    # Trades                                   27
    Win Rate [%]                         70.37037
    Best Trade [%]                      15.154307
    Worst Trade [%]                    -42.439203
    Avg. Trade [%]                      -0.635676
    Max. Trade Duration         453 days 00:00:00
    Avg. Trade Duration         185 days 00:00:00
    Profit Factor                        1.037715
    Expectancy [%]                       0.132015
    SQN                                 -0.157789
    _strategy                          MyStrategy
    _equity_curve                             ...
    _trades                       Size  EntryB...
    dtype: object




<div class="bk-root" id="a2cc7990-ed24-4bb1-92a3-f2ea92dd8d7d" data-root-id="73120"></div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>475</td>
      <td>25</td>
      <td>183</td>
      <td>21036.99</td>
      <td>24225.0</td>
      <td>1514304.75</td>
      <td>0.151543</td>
      <td>2010-02-08</td>
      <td>2010-09-28</td>
      <td>232 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-476</td>
      <td>183</td>
      <td>399</td>
      <td>24176.55</td>
      <td>25080.0</td>
      <td>-430042.20</td>
      <td>-0.037369</td>
      <td>2010-09-28</td>
      <td>2011-08-08</td>
      <td>314 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>441</td>
      <td>399</td>
      <td>526</td>
      <td>25130.16</td>
      <td>26780.0</td>
      <td>727579.44</td>
      <td>0.065652</td>
      <td>2011-08-08</td>
      <td>2012-02-10</td>
      <td>186 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-441</td>
      <td>526</td>
      <td>589</td>
      <td>26726.44</td>
      <td>25520.0</td>
      <td>532040.04</td>
      <td>0.045140</td>
      <td>2012-02-10</td>
      <td>2012-05-14</td>
      <td>94 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>482</td>
      <td>589</td>
      <td>737</td>
      <td>25571.04</td>
      <td>26215.0</td>
      <td>310388.72</td>
      <td>0.025183</td>
      <td>2012-05-14</td>
      <td>2012-12-13</td>
      <td>213 days</td>
    </tr>
    <tr>
      <th>5</th>
      <td>-483</td>
      <td>737</td>
      <td>859</td>
      <td>26162.57</td>
      <td>24610.0</td>
      <td>749891.31</td>
      <td>0.059343</td>
      <td>2012-12-13</td>
      <td>2013-06-14</td>
      <td>183 days</td>
    </tr>
    <tr>
      <th>6</th>
      <td>543</td>
      <td>859</td>
      <td>920</td>
      <td>24659.22</td>
      <td>25790.0</td>
      <td>614013.54</td>
      <td>0.045856</td>
      <td>2013-06-14</td>
      <td>2013-09-10</td>
      <td>88 days</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-544</td>
      <td>920</td>
      <td>1079</td>
      <td>25738.42</td>
      <td>25195.0</td>
      <td>295620.48</td>
      <td>0.021113</td>
      <td>2013-09-10</td>
      <td>2014-05-08</td>
      <td>240 days</td>
    </tr>
    <tr>
      <th>8</th>
      <td>566</td>
      <td>1079</td>
      <td>1136</td>
      <td>25245.39</td>
      <td>26850.0</td>
      <td>908209.26</td>
      <td>0.063561</td>
      <td>2014-05-08</td>
      <td>2014-07-30</td>
      <td>83 days</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-568</td>
      <td>1136</td>
      <td>1178</td>
      <td>26796.30</td>
      <td>25110.0</td>
      <td>957818.40</td>
      <td>0.062930</td>
      <td>2014-07-30</td>
      <td>2014-10-02</td>
      <td>64 days</td>
    </tr>
    <tr>
      <th>10</th>
      <td>643</td>
      <td>1178</td>
      <td>1289</td>
      <td>25160.22</td>
      <td>26100.0</td>
      <td>604278.54</td>
      <td>0.037352</td>
      <td>2014-10-02</td>
      <td>2015-03-18</td>
      <td>167 days</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-644</td>
      <td>1289</td>
      <td>1350</td>
      <td>26047.80</td>
      <td>24935.0</td>
      <td>716643.20</td>
      <td>0.042721</td>
      <td>2015-03-18</td>
      <td>2015-06-16</td>
      <td>90 days</td>
    </tr>
    <tr>
      <th>12</th>
      <td>700</td>
      <td>1350</td>
      <td>1447</td>
      <td>24984.87</td>
      <td>25330.0</td>
      <td>241591.00</td>
      <td>0.013814</td>
      <td>2015-06-16</td>
      <td>2015-11-04</td>
      <td>141 days</td>
    </tr>
    <tr>
      <th>13</th>
      <td>-701</td>
      <td>1447</td>
      <td>1582</td>
      <td>25279.34</td>
      <td>23885.0</td>
      <td>977432.34</td>
      <td>0.055157</td>
      <td>2015-11-04</td>
      <td>2016-05-25</td>
      <td>203 days</td>
    </tr>
    <tr>
      <th>14</th>
      <td>782</td>
      <td>1582</td>
      <td>1726</td>
      <td>23932.77</td>
      <td>26265.0</td>
      <td>1823803.86</td>
      <td>0.097449</td>
      <td>2016-05-25</td>
      <td>2016-12-21</td>
      <td>210 days</td>
    </tr>
    <tr>
      <th>15</th>
      <td>-783</td>
      <td>1726</td>
      <td>1885</td>
      <td>26212.47</td>
      <td>30510.0</td>
      <td>-3364965.99</td>
      <td>-0.163950</td>
      <td>2016-12-21</td>
      <td>2017-08-14</td>
      <td>236 days</td>
    </tr>
    <tr>
      <th>16</th>
      <td>561</td>
      <td>1885</td>
      <td>1920</td>
      <td>30571.02</td>
      <td>32405.0</td>
      <td>1028862.78</td>
      <td>0.059991</td>
      <td>2017-08-14</td>
      <td>2017-10-11</td>
      <td>58 days</td>
    </tr>
    <tr>
      <th>17</th>
      <td>-562</td>
      <td>1920</td>
      <td>2003</td>
      <td>32340.19</td>
      <td>31445.0</td>
      <td>503096.78</td>
      <td>0.027680</td>
      <td>2017-10-11</td>
      <td>2018-02-08</td>
      <td>120 days</td>
    </tr>
    <tr>
      <th>18</th>
      <td>593</td>
      <td>2003</td>
      <td>2238</td>
      <td>31507.89</td>
      <td>28640.0</td>
      <td>-1700658.77</td>
      <td>-0.091021</td>
      <td>2018-02-08</td>
      <td>2019-01-28</td>
      <td>354 days</td>
    </tr>
    <tr>
      <th>19</th>
      <td>-595</td>
      <td>2238</td>
      <td>2306</td>
      <td>28582.72</td>
      <td>27320.0</td>
      <td>751318.40</td>
      <td>0.044178</td>
      <td>2019-01-28</td>
      <td>2019-05-10</td>
      <td>102 days</td>
    </tr>
    <tr>
      <th>20</th>
      <td>648</td>
      <td>2306</td>
      <td>2396</td>
      <td>27374.64</td>
      <td>27280.0</td>
      <td>-61326.72</td>
      <td>-0.003457</td>
      <td>2019-05-10</td>
      <td>2019-09-19</td>
      <td>132 days</td>
    </tr>
    <tr>
      <th>21</th>
      <td>-650</td>
      <td>2396</td>
      <td>2506</td>
      <td>27225.44</td>
      <td>27325.0</td>
      <td>-64714.00</td>
      <td>-0.003657</td>
      <td>2019-09-19</td>
      <td>2020-03-02</td>
      <td>165 days</td>
    </tr>
    <tr>
      <th>22</th>
      <td>644</td>
      <td>2506</td>
      <td>2570</td>
      <td>27379.65</td>
      <td>28965.0</td>
      <td>1020965.40</td>
      <td>0.057902</td>
      <td>2020-03-02</td>
      <td>2020-06-04</td>
      <td>94 days</td>
    </tr>
    <tr>
      <th>23</th>
      <td>-645</td>
      <td>2570</td>
      <td>2870</td>
      <td>28907.07</td>
      <td>41175.0</td>
      <td>-7912814.85</td>
      <td>-0.424392</td>
      <td>2020-06-04</td>
      <td>2021-08-18</td>
      <td>440 days</td>
    </tr>
    <tr>
      <th>24</th>
      <td>260</td>
      <td>2870</td>
      <td>3174</td>
      <td>41257.35</td>
      <td>32435.0</td>
      <td>-2293811.00</td>
      <td>-0.213837</td>
      <td>2021-08-18</td>
      <td>2022-11-14</td>
      <td>453 days</td>
    </tr>
    <tr>
      <th>25</th>
      <td>-261</td>
      <td>3174</td>
      <td>3394</td>
      <td>32370.13</td>
      <td>32235.0</td>
      <td>35268.93</td>
      <td>0.004175</td>
      <td>2022-11-14</td>
      <td>2023-10-05</td>
      <td>325 days</td>
    </tr>
    <tr>
      <th>26</th>
      <td>262</td>
      <td>3394</td>
      <td>3395</td>
      <td>32299.47</td>
      <td>32060.0</td>
      <td>-62741.14</td>
      <td>-0.007414</td>
      <td>2023-10-05</td>
      <td>2023-10-06</td>
      <td>1 days</td>
    </tr>
  </tbody>
</table>
</div>



```python

```

### Take Profits and Stop Losses

- 수익실현, 스탑로스 설정


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14

    def init(self):
        self.daily_rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)

    def next(self):
        
        price = self.data.Close[-1] # recent value
        
        if crossover(self.daily_rsi, self.upper_bound) :
            self.position.close() 

        elif crossover(self.lower_bound, self.daily_rsi):
            self.buy(tp=1.05*price, sl = 0.97*price) # sl : stop loss 

bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats = bt.run()

bt.plot()
# print(stats)
# display(stats['_trades'])
```



<div class="bk-root" id="7d5bb3f3-da29-4a2f-9e1b-e49919429863" data-root-id="77980"></div>








<div style="display: table;"><div style="display: table-row;"><div style="display: table-cell;"><b title="bokeh.models.layouts.Row">Row</b>(</div><div style="display: table-cell;">id&nbsp;=&nbsp;'77980', <span id="78318" style="cursor: pointer;">&hellip;)</span></div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">align&nbsp;=&nbsp;'start',</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">aspect_ratio&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">background&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">children&nbsp;=&nbsp;[GridBox(id='77977', ...), ToolbarBox(id='77979', ...)],</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">cols&nbsp;=&nbsp;'auto',</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">css_classes&nbsp;=&nbsp;[],</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">disabled&nbsp;=&nbsp;False,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">height&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">height_policy&nbsp;=&nbsp;'auto',</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">js_event_callbacks&nbsp;=&nbsp;{},</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">js_property_callbacks&nbsp;=&nbsp;{},</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">margin&nbsp;=&nbsp;(0, 0, 0, 0),</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">max_height&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">max_width&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">min_height&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">min_width&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">name&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">sizing_mode&nbsp;=&nbsp;'stretch_width',</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">spacing&nbsp;=&nbsp;0,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">subscribed_events&nbsp;=&nbsp;[],</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">syncable&nbsp;=&nbsp;True,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">tags&nbsp;=&nbsp;[],</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">visible&nbsp;=&nbsp;True,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">width&nbsp;=&nbsp;None,</div></div><div class="78317" style="display: none;"><div style="display: table-cell;"></div><div style="display: table-cell;">width_policy&nbsp;=&nbsp;'auto')</div></div></div>
<script>
(function() {
  let expanded = false;
  const ellipsis = document.getElementById("78318");
  ellipsis.addEventListener("click", function() {
    const rows = document.getElementsByClassName("78317");
    for (let i = 0; i < rows.length; i++) {
      const el = rows[i];
      el.style.display = expanded ? "none" : "table-row";
    }
    ellipsis.innerHTML = expanded ? "&hellip;)" : "&lsaquo;&lsaquo;&lsaquo;";
    expanded = !expanded;
  });
})();
</script>





```python

```

### Size

- 매수비중 조절


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14
    position_size=1

    def init(self):
        self.daily_rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)

    def next(self):
        
        price = self.data.Close[-1] # recent value
        
        if crossover(self.daily_rsi, self.upper_bound) :
            self.position.close() 

        # elif crossover(self.lower_bound, self.daily_rsi):
        #     self.buy(size=0.1) # fractions shares 
        elif self.lower_bound > self.daily_rsi[-1]:
            self.buy(size=self.position_size) # fractions shares 

bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats = bt.run()

bt.plot()
print(stats)
display(stats['_trades'])
```



<div class="bk-root" id="bba6c965-dd06-472b-be7b-28756d185892" data-root-id="85280"></div>





    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   48.468787
    Equity Final [$]                  10169269.44
    Equity Peak [$]                   10210942.37
    Return [%]                           1.692694
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    0.124633
    Volatility (Ann.) [%]                0.362115
    Sharpe Ratio                          0.34418
    Sortino Ratio                        0.518956
    Calmar Ratio                         0.087826
    Max. Drawdown [%]                   -1.419085
    Avg. Drawdown [%]                   -0.056055
    Max. Drawdown Duration      647 days 00:00:00
    Avg. Drawdown Duration       31 days 00:00:00
    # Trades                                  137
    Win Rate [%]                        84.671533
    Best Trade [%]                      42.892663
    Worst Trade [%]                    -21.383705
    Avg. Trade [%]                       5.458696
    Max. Trade Duration         453 days 00:00:00
    Avg. Trade Duration         144 days 00:00:00
    Profit Factor                        5.164782
    Expectancy [%]                       5.918196
    SQN                                  5.235569
    _strategy                          MyStrategy
    _equity_curve                             ...
    _trades                        Size  Entry...
    dtype: object



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>98</td>
      <td>183</td>
      <td>20761.44</td>
      <td>24225.0</td>
      <td>3463.56</td>
      <td>0.166827</td>
      <td>2010-05-26</td>
      <td>2010-09-28</td>
      <td>125 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>87</td>
      <td>183</td>
      <td>21803.52</td>
      <td>24225.0</td>
      <td>2421.48</td>
      <td>0.111059</td>
      <td>2010-05-10</td>
      <td>2010-09-28</td>
      <td>141 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>26</td>
      <td>183</td>
      <td>20741.40</td>
      <td>24225.0</td>
      <td>3483.60</td>
      <td>0.167954</td>
      <td>2010-02-09</td>
      <td>2010-09-28</td>
      <td>231 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>25</td>
      <td>183</td>
      <td>21036.99</td>
      <td>24225.0</td>
      <td>3188.01</td>
      <td>0.151543</td>
      <td>2010-02-08</td>
      <td>2010-09-28</td>
      <td>232 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>409</td>
      <td>526</td>
      <td>22284.48</td>
      <td>26780.0</td>
      <td>4495.52</td>
      <td>0.201733</td>
      <td>2011-08-23</td>
      <td>2012-02-10</td>
      <td>171 days</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>132</th>
      <td>1</td>
      <td>2873</td>
      <td>3174</td>
      <td>40500.84</td>
      <td>32435.0</td>
      <td>-8065.84</td>
      <td>-0.199152</td>
      <td>2021-08-23</td>
      <td>2022-11-14</td>
      <td>448 days</td>
    </tr>
    <tr>
      <th>133</th>
      <td>1</td>
      <td>2872</td>
      <td>3174</td>
      <td>40781.40</td>
      <td>32435.0</td>
      <td>-8346.40</td>
      <td>-0.204662</td>
      <td>2021-08-20</td>
      <td>2022-11-14</td>
      <td>451 days</td>
    </tr>
    <tr>
      <th>134</th>
      <td>1</td>
      <td>2870</td>
      <td>3174</td>
      <td>41257.35</td>
      <td>32435.0</td>
      <td>-8822.35</td>
      <td>-0.213837</td>
      <td>2021-08-18</td>
      <td>2022-11-14</td>
      <td>453 days</td>
    </tr>
    <tr>
      <th>135</th>
      <td>1</td>
      <td>3395</td>
      <td>3395</td>
      <td>32124.12</td>
      <td>32060.0</td>
      <td>-64.12</td>
      <td>-0.001996</td>
      <td>2023-10-06</td>
      <td>2023-10-06</td>
      <td>0 days</td>
    </tr>
    <tr>
      <th>136</th>
      <td>1</td>
      <td>3394</td>
      <td>3395</td>
      <td>32299.47</td>
      <td>32060.0</td>
      <td>-239.47</td>
      <td>-0.007414</td>
      <td>2023-10-05</td>
      <td>2023-10-06</td>
      <td>1 days</td>
    </tr>
  </tbody>
</table>
<p>137 rows × 10 columns</p>
</div>


### barssince
- position을 x일간 유지할 수 있도록 해줌


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply, barssince

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    upper_bound = 70
    lower_bound = 30
    rsi_window=14
    position_size=0.2

    def init(self):
        self.daily_rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)

    def next(self):
        
        price = self.data.Close[-1] # recent value
        
        if (self.daily_rsi[-1] > self.upper_bound and
            barssince(self.daily_rsi < self.upper_bound)==3):
            self.position.close() 

        # elif crossover(self.lower_bound, self.daily_rsi):
        #     self.buy(size=0.1) # fractions shares 
        elif self.lower_bound > self.daily_rsi[-1]:
            self.buy(size=self.position_size) # fractions shares 

bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats = bt.run()

bt.plot()
print(stats)
display(stats['_trades'])
```



<div class="bk-root" id="5835d9b7-b53b-4625-ab8f-7cff633b9b12" data-root-id="88520"></div>





    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   61.513545
    Equity Final [$]                  14904859.33
    Equity Peak [$]                   18202293.18
    Return [%]                          49.048593
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    3.005825
    Volatility (Ann.) [%]               12.129955
    Sharpe Ratio                         0.247802
    Sortino Ratio                        0.372816
    Calmar Ratio                         0.108518
    Max. Drawdown [%]                  -27.698991
    Avg. Drawdown [%]                   -2.020104
    Max. Drawdown Duration      760 days 00:00:00
    Avg. Drawdown Duration       40 days 00:00:00
    # Trades                                  135
    Win Rate [%]                        85.925926
    Best Trade [%]                      45.531971
    Worst Trade [%]                    -22.292634
    Avg. Trade [%]                       6.372823
    Max. Trade Duration         779 days 00:00:00
    Avg. Trade Duration         271 days 00:00:00
    Profit Factor                        5.544472
    Expectancy [%]                       6.904557
    SQN                                  2.777349
    _strategy                          MyStrategy
    _equity_curve                             ...
    _trades                        Size  Entry...
    dtype: object



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>49</td>
      <td>98</td>
      <td>187</td>
      <td>20761.44</td>
      <td>24545.0</td>
      <td>185394.44</td>
      <td>0.182240</td>
      <td>2010-05-26</td>
      <td>2010-10-04</td>
      <td>131 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>58</td>
      <td>87</td>
      <td>187</td>
      <td>21803.52</td>
      <td>24545.0</td>
      <td>159005.84</td>
      <td>0.125736</td>
      <td>2010-05-10</td>
      <td>2010-10-04</td>
      <td>147 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>77</td>
      <td>26</td>
      <td>187</td>
      <td>20741.40</td>
      <td>24545.0</td>
      <td>292877.20</td>
      <td>0.183382</td>
      <td>2010-02-09</td>
      <td>2010-10-04</td>
      <td>237 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>95</td>
      <td>25</td>
      <td>187</td>
      <td>21036.99</td>
      <td>24545.0</td>
      <td>333260.95</td>
      <td>0.166754</td>
      <td>2010-02-08</td>
      <td>2010-10-04</td>
      <td>238 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>604</td>
      <td>739</td>
      <td>23997.90</td>
      <td>26540.0</td>
      <td>5084.20</td>
      <td>0.105930</td>
      <td>2012-06-05</td>
      <td>2012-12-17</td>
      <td>195 days</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>130</th>
      <td>37</td>
      <td>2901</td>
      <td>3395</td>
      <td>39138.12</td>
      <td>32060.0</td>
      <td>-261890.44</td>
      <td>-0.180850</td>
      <td>2021-10-06</td>
      <td>2023-10-06</td>
      <td>730 days</td>
    </tr>
    <tr>
      <th>131</th>
      <td>47</td>
      <td>2900</td>
      <td>3395</td>
      <td>39163.17</td>
      <td>32060.0</td>
      <td>-333848.99</td>
      <td>-0.181374</td>
      <td>2021-10-05</td>
      <td>2023-10-06</td>
      <td>731 days</td>
    </tr>
    <tr>
      <th>132</th>
      <td>56</td>
      <td>2873</td>
      <td>3395</td>
      <td>40500.84</td>
      <td>32060.0</td>
      <td>-472687.04</td>
      <td>-0.208411</td>
      <td>2021-08-23</td>
      <td>2023-10-06</td>
      <td>774 days</td>
    </tr>
    <tr>
      <th>133</th>
      <td>70</td>
      <td>2872</td>
      <td>3395</td>
      <td>40781.40</td>
      <td>32060.0</td>
      <td>-610498.00</td>
      <td>-0.213857</td>
      <td>2021-08-20</td>
      <td>2023-10-06</td>
      <td>777 days</td>
    </tr>
    <tr>
      <th>134</th>
      <td>87</td>
      <td>2870</td>
      <td>3395</td>
      <td>41257.35</td>
      <td>32060.0</td>
      <td>-800169.45</td>
      <td>-0.222926</td>
      <td>2021-08-18</td>
      <td>2023-10-06</td>
      <td>779 days</td>
    </tr>
  </tbody>
</table>
<p>135 rows × 10 columns</p>
</div>



```python

```


```python

```


```python

```


```python

```

## strategies

### strategy : SAR, MACD


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply, barssince

class MyStrategy(Strategy):
    position_size=0.2
    
    def init(self):
        #Parabolic SAR 지표 계산
        self.sar = self.I(talib.SAR, self.data.High, self.data.Low, acceleration=0.02, maximum=0.2)
        # MACD 지표 계산
        self.macd_line, self.signal_line, self.macd_hist = self.I(talib.MACD, 
             self.data.Close, fastperiod=12, slowperiod=26, signalperiod=9)        

    def next(self):
        
        price = self.data.Close[-1] # recent value
        
        # 매수
        if (self.data.Close > self.sar and self.macd_hist > 0):
            self.buy(size=self.position_size)#, sl=0.95*price, tp=1.05*price)
            
        # 매도
        elif (self.sar > self.data.Close and self.macd_hist < 0):
            self.position.close()

            
bt = Backtest(data, MyStrategy, cash=1000000, commission=0.002)
stats = bt.run()

bt.plot()
print(stats)
display(stats['_trades'])
```



<div class="bk-root" id="8258222b-3a5b-4e19-87e2-7e10e06d99ec" data-root-id="115805"></div>





    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   55.742049
    Equity Final [$]                    1068093.2
    Equity Peak [$]                    1201059.73
    Return [%]                            6.80932
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    0.490022
    Volatility (Ann.) [%]                8.184861
    Sharpe Ratio                         0.059869
    Sortino Ratio                        0.084004
    Calmar Ratio                         0.019785
    Max. Drawdown [%]                   -24.76673
    Avg. Drawdown [%]                   -2.160225
    Max. Drawdown Duration     4547 days 00:00:00
    Avg. Drawdown Duration      236 days 00:00:00
    # Trades                                  977
    Win Rate [%]                        38.689867
    Best Trade [%]                       13.92485
    Worst Trade [%]                     -9.015166
    Avg. Trade [%]                       0.024117
    Max. Trade Duration          70 days 00:00:00
    Avg. Trade Duration          22 days 00:00:00
    Profit Factor                        1.064758
    Expectancy [%]                       0.083349
    SQN                                  0.555778
    _strategy                          MyStrategy
    _equity_curve                            E...
    _trades                        Size  Entry...
    dtype: object



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>52</td>
      <td>70</td>
      <td>22439.79</td>
      <td>22935.0</td>
      <td>495.21</td>
      <td>0.022068</td>
      <td>2010-03-19</td>
      <td>2010-04-14</td>
      <td>26 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>51</td>
      <td>70</td>
      <td>22389.69</td>
      <td>22935.0</td>
      <td>545.31</td>
      <td>0.024355</td>
      <td>2010-03-18</td>
      <td>2010-04-14</td>
      <td>27 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>50</td>
      <td>70</td>
      <td>22209.33</td>
      <td>22935.0</td>
      <td>725.67</td>
      <td>0.032674</td>
      <td>2010-03-17</td>
      <td>2010-04-14</td>
      <td>28 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>49</td>
      <td>70</td>
      <td>22038.99</td>
      <td>22935.0</td>
      <td>896.01</td>
      <td>0.040656</td>
      <td>2010-03-16</td>
      <td>2010-04-14</td>
      <td>29 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>48</td>
      <td>70</td>
      <td>22214.34</td>
      <td>22935.0</td>
      <td>720.66</td>
      <td>0.032441</td>
      <td>2010-03-15</td>
      <td>2010-04-14</td>
      <td>30 days</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>972</th>
      <td>2</td>
      <td>3377</td>
      <td>3389</td>
      <td>34093.05</td>
      <td>33005.0</td>
      <td>-2176.10</td>
      <td>-0.031914</td>
      <td>2023-09-06</td>
      <td>2023-09-22</td>
      <td>16 days</td>
    </tr>
    <tr>
      <th>973</th>
      <td>3</td>
      <td>3376</td>
      <td>3389</td>
      <td>34108.08</td>
      <td>33005.0</td>
      <td>-3309.24</td>
      <td>-0.032341</td>
      <td>2023-09-05</td>
      <td>2023-09-22</td>
      <td>17 days</td>
    </tr>
    <tr>
      <th>974</th>
      <td>4</td>
      <td>3375</td>
      <td>3389</td>
      <td>34078.02</td>
      <td>33005.0</td>
      <td>-4292.08</td>
      <td>-0.031487</td>
      <td>2023-09-04</td>
      <td>2023-09-22</td>
      <td>18 days</td>
    </tr>
    <tr>
      <th>975</th>
      <td>5</td>
      <td>3374</td>
      <td>3389</td>
      <td>33551.97</td>
      <td>33005.0</td>
      <td>-2734.85</td>
      <td>-0.016302</td>
      <td>2023-09-01</td>
      <td>2023-09-22</td>
      <td>21 days</td>
    </tr>
    <tr>
      <th>976</th>
      <td>6</td>
      <td>3373</td>
      <td>3389</td>
      <td>33697.26</td>
      <td>33005.0</td>
      <td>-4153.56</td>
      <td>-0.020544</td>
      <td>2023-08-31</td>
      <td>2023-09-22</td>
      <td>22 days</td>
    </tr>
  </tbody>
</table>
<p>977 rows × 10 columns</p>
</div>



```python
stats['_trades']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>387</td>
      <td>156</td>
      <td>460</td>
      <td>25821.54</td>
      <td>15530.0</td>
      <td>-3982825.98</td>
      <td>-0.398564</td>
      <td>2021-08-18</td>
      <td>2022-11-14</td>
      <td>453 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>399</td>
      <td>680</td>
      <td>681</td>
      <td>15075.09</td>
      <td>14920.0</td>
      <td>-61880.91</td>
      <td>-0.010288</td>
      <td>2023-10-05</td>
      <td>2023-10-06</td>
      <td>1 days</td>
    </tr>
  </tbody>
</table>
</div>



### MACD strategy


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
    return series['Sharpe Ratio']


class MyStrategy(Strategy):

    fast_period = 12
    slow_period = 26
    signal_period = 9

    def init(self):
        self.macd_line, self.signal_line, self.macd_hist = self.I(talib.MACD, 
             self.data.Close, fastperiod=self.fast_period, slowperiod=self.slow_period, signalperiod=self.signal_period)
    
    def next(self):
        if crossover(self.signal_line, self.macd_line):
            self.position.close()
        elif crossover(self.macd_line, self.signal_line):
            self.buy()
            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats, heatmap = bt.optimize(
    fast_period=range(9, 25, 3),
    slow_period=range(20, 50, 3),
    signal_period=range(8, 25, 3),
    maximize = optim_func,#'Sharpe Ratio',
    constraint = lambda param : param.fast_period < param.slow_period,
    return_heatmap=True
    )

#print(heatmap)

hm = heatmap.groupby(['fast_period', 'slow_period']).mean().unstack()
#print(hm)
# sns.heatmap(hm, annot=True, cmap='viridis')
# plt.show()
plot_heatmaps(heatmap, agg='mean')

fast_period = stats['_strategy'].fast_period
slow_period = stats['_strategy'].slow_period
signal_period = stats['_strategy'].signal_period
print("best :", fast_period,"-", slow_period,"-", signal_period)
print(stats)

bt.plot()
display(stats['_trades'])
```

    /Users/byeongsikbu/opt/anaconda3/envs/dct/lib/python3.10/site-packages/backtesting/backtesting.py:1488: UserWarning: Searching for best of 342 configurations.
      output = _optimize_grid()
    /Users/byeongsikbu/opt/anaconda3/envs/dct/lib/python3.10/site-packages/backtesting/backtesting.py:1375: UserWarning: For multiprocessing support in `Backtest.optimize()` set multiprocessing start method to 'fork'.
      warnings.warn("For multiprocessing support in `Backtest.optimize()` "



      0%|          | 0/9 [00:00<?, ?it/s]




<div class="bk-root" id="a4418719-90e0-4bda-9abc-94ef8faaf13c" data-root-id="113365"></div>





    best : 18 - 20 - 11
    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   53.739694
    Equity Final [$]                  15981353.49
    Equity Peak [$]                   17669529.69
    Return [%]                          59.813535
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    3.540232
    Volatility (Ann.) [%]               11.569808
    Sharpe Ratio                         0.305989
    Sortino Ratio                        0.452202
    Calmar Ratio                         0.141107
    Max. Drawdown [%]                  -25.088971
    Avg. Drawdown [%]                    -3.49389
    Max. Drawdown Duration     2195 days 00:00:00
    Avg. Drawdown Duration      125 days 00:00:00
    # Trades                                  108
    Win Rate [%]                        38.888889
    Best Trade [%]                      15.241274
    Worst Trade [%]                     -7.939919
    Avg. Trade [%]                        0.43567
    Max. Trade Duration          77 days 00:00:00
    Avg. Trade Duration          24 days 00:00:00
    Profit Factor                        1.439068
    Expectancy [%]                       0.515966
    SQN                                   1.09744
    _strategy                 MyStrategy(fast_...
    _equity_curve                             ...
    _trades                        Size  Entry...
    dtype: object




<div class="bk-root" id="89dfd04a-5abb-4ded-a9dd-291616a058f2" data-root-id="114073"></div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>457</td>
      <td>32</td>
      <td>74</td>
      <td>21858.63</td>
      <td>22860.0</td>
      <td>457626.09</td>
      <td>0.045811</td>
      <td>2010-02-18</td>
      <td>2010-04-20</td>
      <td>61 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>476</td>
      <td>104</td>
      <td>124</td>
      <td>21958.83</td>
      <td>21960.0</td>
      <td>556.92</td>
      <td>0.000053</td>
      <td>2010-06-04</td>
      <td>2010-07-02</td>
      <td>28 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>458</td>
      <td>131</td>
      <td>153</td>
      <td>22790.49</td>
      <td>22700.0</td>
      <td>-41444.42</td>
      <td>-0.003971</td>
      <td>2010-07-13</td>
      <td>2010-08-12</td>
      <td>30 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>444</td>
      <td>171</td>
      <td>195</td>
      <td>23446.80</td>
      <td>24505.0</td>
      <td>469840.80</td>
      <td>0.045132</td>
      <td>2010-09-07</td>
      <td>2010-10-14</td>
      <td>37 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>428</td>
      <td>211</td>
      <td>218</td>
      <td>25400.70</td>
      <td>25000.0</td>
      <td>-171499.60</td>
      <td>-0.015775</td>
      <td>2010-11-05</td>
      <td>2010-11-16</td>
      <td>11 days</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>481</td>
      <td>3214</td>
      <td>3237</td>
      <td>31407.69</td>
      <td>32825.0</td>
      <td>681726.11</td>
      <td>0.045126</td>
      <td>2023-01-10</td>
      <td>2023-02-14</td>
      <td>35 days</td>
    </tr>
    <tr>
      <th>104</th>
      <td>493</td>
      <td>3265</td>
      <td>3287</td>
      <td>31978.83</td>
      <td>32905.0</td>
      <td>456601.81</td>
      <td>0.028962</td>
      <td>2023-03-27</td>
      <td>2023-04-26</td>
      <td>30 days</td>
    </tr>
    <tr>
      <th>105</th>
      <td>479</td>
      <td>3304</td>
      <td>3323</td>
      <td>33882.63</td>
      <td>34230.0</td>
      <td>166390.23</td>
      <td>0.010252</td>
      <td>2023-05-23</td>
      <td>2023-06-21</td>
      <td>29 days</td>
    </tr>
    <tr>
      <th>106</th>
      <td>472</td>
      <td>3341</td>
      <td>3355</td>
      <td>34739.34</td>
      <td>34275.0</td>
      <td>-219168.48</td>
      <td>-0.013366</td>
      <td>2023-07-17</td>
      <td>2023-08-04</td>
      <td>18 days</td>
    </tr>
    <tr>
      <th>107</th>
      <td>482</td>
      <td>3374</td>
      <td>3390</td>
      <td>33551.97</td>
      <td>33105.0</td>
      <td>-215439.54</td>
      <td>-0.013322</td>
      <td>2023-09-01</td>
      <td>2023-09-25</td>
      <td>24 days</td>
    </tr>
  </tbody>
</table>
<p>108 rows × 10 columns</p>
</div>


### strategy : SAR


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps

def optim_func(series):
    
    if series['# Trades'] < 3: #최소 트레이드 회수 제한
        return -1
    #return series['Equity Final [$]'] / series['Exposure Time [%]']
#    return series['Sharpe Ratio']
    return series['Equity Final [$]']


class MyStrategy(Strategy):

    acceleration = 0.02
    maximum=0.2
    def init(self):
        self.sar = self.I(talib.SAR, self.data.High, self.data.Low, acceleration=self.acceleration, maximum=self.maximum)
    
    def next(self):
        if crossover(self.data.Close, self.sar):
            self.buy()
        elif crossover(self.sar, self.data.Close):
            self.position.close()
            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats, heatmap = bt.optimize(
    acceleration=0.02, #np.arange(0.01, 0.05, 0.01).tolist(),
    maximum=0.2, #np.arange(0.1, 0.25, 0.01).tolist(),
    maximize = optim_func,#'Sharpe Ratio',
    #constraint = lambda param : param.fast_period < param.slow_period,
    return_heatmap=True
    )

#print(heatmap)

hm = heatmap.groupby(['acceleration', 'maximum']).mean().unstack()
#print(hm)
# sns.heatmap(hm, annot=True, cmap='viridis')
# plt.show()
plot_heatmaps(heatmap, agg='mean')

acceleration = stats['_strategy'].acceleration
maximum = stats['_strategy'].maximum

print("best :", acceleration,"-", maximum)
print(stats)

bt.plot()
display(stats['_trades'])
```

    /Users/byeongsikbu/opt/anaconda3/envs/dct/lib/python3.10/site-packages/backtesting/backtesting.py:1375: UserWarning: For multiprocessing support in `Backtest.optimize()` set multiprocessing start method to 'fork'.
      warnings.warn("For multiprocessing support in `Backtest.optimize()` "



      0%|          | 0/1 [00:00<?, ?it/s]




<div class="bk-root" id="1e4fddd2-2554-4e88-9273-895e4ac0f9b0" data-root-id="121134"></div>





    best : 0.02 - 0.2
    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   58.186101
    Equity Final [$]                   11100358.3
    Equity Peak [$]                   12748303.25
    Return [%]                          11.003583
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    0.777651
    Volatility (Ann.) [%]               10.952246
    Sharpe Ratio                         0.071004
    Sortino Ratio                        0.102097
    Calmar Ratio                         0.023512
    Max. Drawdown [%]                  -33.074381
    Avg. Drawdown [%]                   -5.483589
    Max. Drawdown Duration     3638 days 00:00:00
    Avg. Drawdown Duration      332 days 00:00:00
    # Trades                                  150
    Win Rate [%]                        35.333333
    Best Trade [%]                      15.752356
    Worst Trade [%]                     -6.348752
    Avg. Trade [%]                       0.069374
    Max. Trade Duration          55 days 00:00:00
    Avg. Trade Duration          18 days 00:00:00
    Profit Factor                         1.10156
    Expectancy [%]                       0.130881
    SQN                                  0.259607
    _strategy                 MyStrategy(accel...
    _equity_curve                             ...
    _trades                        Size  Entry...
    dtype: object




<div class="bk-root" id="964d8b3d-91bc-4c7c-aa4a-857b956aff3f" data-root-id="121627"></div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>435</td>
      <td>11</td>
      <td>15</td>
      <td>22935.78</td>
      <td>22175.0</td>
      <td>-330939.30</td>
      <td>-0.033170</td>
      <td>2010-01-19</td>
      <td>2010-01-25</td>
      <td>6 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>451</td>
      <td>29</td>
      <td>38</td>
      <td>21402.72</td>
      <td>21250.0</td>
      <td>-68876.72</td>
      <td>-0.007136</td>
      <td>2010-02-12</td>
      <td>2010-02-26</td>
      <td>14 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>434</td>
      <td>43</td>
      <td>68</td>
      <td>22079.07</td>
      <td>23170.0</td>
      <td>473463.62</td>
      <td>0.049410</td>
      <td>2010-03-08</td>
      <td>2010-04-12</td>
      <td>35 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>431</td>
      <td>71</td>
      <td>74</td>
      <td>23346.60</td>
      <td>22860.0</td>
      <td>-209724.60</td>
      <td>-0.020842</td>
      <td>2010-04-15</td>
      <td>2010-04-20</td>
      <td>5 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>425</td>
      <td>76</td>
      <td>82</td>
      <td>23176.26</td>
      <td>22910.0</td>
      <td>-113160.50</td>
      <td>-0.011488</td>
      <td>2010-04-22</td>
      <td>2010-04-30</td>
      <td>8 days</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>145</th>
      <td>336</td>
      <td>3264</td>
      <td>3285</td>
      <td>31973.82</td>
      <td>33470.0</td>
      <td>502716.48</td>
      <td>0.046794</td>
      <td>2023-03-24</td>
      <td>2023-04-24</td>
      <td>31 days</td>
    </tr>
    <tr>
      <th>146</th>
      <td>337</td>
      <td>3302</td>
      <td>3320</td>
      <td>33326.52</td>
      <td>34555.0</td>
      <td>413997.76</td>
      <td>0.036862</td>
      <td>2023-05-19</td>
      <td>2023-06-16</td>
      <td>28 days</td>
    </tr>
    <tr>
      <th>147</th>
      <td>337</td>
      <td>3332</td>
      <td>3336</td>
      <td>34543.95</td>
      <td>33340.0</td>
      <td>-405731.15</td>
      <td>-0.034853</td>
      <td>2023-07-04</td>
      <td>2023-07-10</td>
      <td>6 days</td>
    </tr>
    <tr>
      <th>148</th>
      <td>325</td>
      <td>3340</td>
      <td>3355</td>
      <td>34609.08</td>
      <td>34275.0</td>
      <td>-108576.00</td>
      <td>-0.009653</td>
      <td>2023-07-14</td>
      <td>2023-08-04</td>
      <td>21 days</td>
    </tr>
    <tr>
      <th>149</th>
      <td>329</td>
      <td>3372</td>
      <td>3388</td>
      <td>33857.58</td>
      <td>33715.0</td>
      <td>-46908.82</td>
      <td>-0.004211</td>
      <td>2023-08-30</td>
      <td>2023-09-21</td>
      <td>22 days</td>
    </tr>
  </tbody>
</table>
<p>150 rows × 10 columns</p>
</div>


### SMA, RSI


```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply, barssince

class MyStrategy(Strategy):
    rsi_window=14
    short_period=60
    long_period=120
    upper_bound=65
    lower_bound=45
    
    def init(self):
        self.rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)
        self.ma_short = self.I(talib.SMA, self.data.Close, self.short_period)
        self.ma_long = self.I(talib.SMA, self.data.Close, self.long_period)

    def next(self):
        # 매수
        if (self.rsi < self.lower_bound and self.ma_short > self.ma_long):
            self.buy()
            
        # 매도
        elif (self.rsi > self.upper_bound and self.ma_short < self.ma_long):
            self.position.close()

            
bt = Backtest(data, MyStrategy, cash=1000000, commission=0.002)
stats = bt.run()

bt.plot()
print(stats)
display(stats['_trades'])
```



<div class="bk-root" id="4ece4a29-eb66-4579-b7bc-7065d670c198" data-root-id="136155"></div>





    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   77.532391
    Equity Final [$]                   1325733.23
    Equity Peak [$]                    1866205.41
    Return [%]                          32.573323
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    2.114367
    Volatility (Ann.) [%]               15.595352
    Sharpe Ratio                         0.135577
    Sortino Ratio                         0.19576
    Calmar Ratio                         0.058773
    Max. Drawdown [%]                   -35.97514
    Avg. Drawdown [%]                   -3.603827
    Max. Drawdown Duration     2222 days 00:00:00
    Avg. Drawdown Duration      108 days 00:00:00
    # Trades                                   13
    Win Rate [%]                        61.538462
    Best Trade [%]                      19.740756
    Worst Trade [%]                     -6.503583
    Avg. Trade [%]                       2.474779
    Max. Trade Duration         777 days 00:00:00
    Avg. Trade Duration         310 days 00:00:00
    Profit Factor                         4.40458
    Expectancy [%]                       2.711425
    SQN                                  1.122675
    _strategy                          MyStrategy
    _equity_curve                             ...
    _trades                       Size  EntryB...
    dtype: object



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>45</td>
      <td>124</td>
      <td>651</td>
      <td>22003.92</td>
      <td>25890.0</td>
      <td>174873.60</td>
      <td>0.176609</td>
      <td>2010-07-02</td>
      <td>2012-08-10</td>
      <td>770 days</td>
    </tr>
    <tr>
      <th>1</th>
      <td>46</td>
      <td>692</td>
      <td>918</td>
      <td>25450.80</td>
      <td>25420.0</td>
      <td>-1416.80</td>
      <td>-0.001210</td>
      <td>2012-10-11</td>
      <td>2013-09-06</td>
      <td>330 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>44</td>
      <td>958</td>
      <td>1062</td>
      <td>26117.13</td>
      <td>26400.0</td>
      <td>12446.28</td>
      <td>0.010831</td>
      <td>2013-11-08</td>
      <td>2014-04-10</td>
      <td>153 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>1182</td>
      <td>1279</td>
      <td>24829.56</td>
      <td>25625.0</td>
      <td>795.44</td>
      <td>0.032036</td>
      <td>2014-10-10</td>
      <td>2015-03-04</td>
      <td>145 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>45</td>
      <td>1104</td>
      <td>1279</td>
      <td>25801.50</td>
      <td>25625.0</td>
      <td>-7942.50</td>
      <td>-0.006841</td>
      <td>2014-06-16</td>
      <td>2015-03-04</td>
      <td>261 days</td>
    </tr>
    <tr>
      <th>5</th>
      <td>44</td>
      <td>1323</td>
      <td>1430</td>
      <td>26332.56</td>
      <td>24620.0</td>
      <td>-75352.64</td>
      <td>-0.065036</td>
      <td>2015-05-07</td>
      <td>2015-10-12</td>
      <td>158 days</td>
    </tr>
    <tr>
      <th>6</th>
      <td>45</td>
      <td>1470</td>
      <td>1529</td>
      <td>24423.75</td>
      <td>24460.0</td>
      <td>1631.25</td>
      <td>0.001484</td>
      <td>2015-12-07</td>
      <td>2016-03-08</td>
      <td>92 days</td>
    </tr>
    <tr>
      <th>7</th>
      <td>44</td>
      <td>1566</td>
      <td>1626</td>
      <td>24579.06</td>
      <td>25210.0</td>
      <td>27761.36</td>
      <td>0.025670</td>
      <td>2016-04-29</td>
      <td>2016-07-27</td>
      <td>89 days</td>
    </tr>
    <tr>
      <th>8</th>
      <td>44</td>
      <td>1659</td>
      <td>2157</td>
      <td>25350.60</td>
      <td>30355.0</td>
      <td>220193.60</td>
      <td>0.197408</td>
      <td>2016-09-13</td>
      <td>2018-09-28</td>
      <td>745 days</td>
    </tr>
    <tr>
      <th>9</th>
      <td>48</td>
      <td>2268</td>
      <td>2340</td>
      <td>28116.12</td>
      <td>27890.0</td>
      <td>-10853.76</td>
      <td>-0.008042</td>
      <td>2019-03-15</td>
      <td>2019-06-28</td>
      <td>105 days</td>
    </tr>
    <tr>
      <th>10</th>
      <td>48</td>
      <td>2446</td>
      <td>2568</td>
      <td>27930.75</td>
      <td>27305.0</td>
      <td>-30036.00</td>
      <td>-0.022404</td>
      <td>2019-12-02</td>
      <td>2020-06-02</td>
      <td>183 days</td>
    </tr>
    <tr>
      <th>11</th>
      <td>42</td>
      <td>2648</td>
      <td>3171</td>
      <td>31067.01</td>
      <td>31375.0</td>
      <td>12935.58</td>
      <td>0.009914</td>
      <td>2020-09-23</td>
      <td>2022-11-09</td>
      <td>777 days</td>
    </tr>
    <tr>
      <th>12</th>
      <td>41</td>
      <td>3247</td>
      <td>3395</td>
      <td>31993.86</td>
      <td>32060.0</td>
      <td>2711.74</td>
      <td>0.002067</td>
      <td>2023-02-28</td>
      <td>2023-10-06</td>
      <td>220 days</td>
    </tr>
  </tbody>
</table>
</div>



```python
import talib
from backtesting import Backtest, Strategy
from backtesting.lib import crossover, plot_heatmaps, resample_apply, barssince

class MyStrategy(Strategy):
    short_period=60
    long_period=120

    rsi_window = 14
    upper_bound = 70
    lower_bound = 30
    
    position_size = 1
    
    def init(self):
        self.ma_short = self.I(talib.SMA, self.data.Close, self.short_period)
        self.ma_long = self.I(talib.SMA, self.data.Close, self.long_period)
        self.rsi = self.I(talib.RSI, self.data.Close, self.rsi_window)
        
    def next(self):
        
        price = self.data.Close[-1]
        # 매수
        if (self.ma_short > self.ma_long and self.rsi < self.lower_bound):
            # if self.position.is_short or not self.position:
            #     self.position.close()
            self.buy(sl=0.8*price)
            
        # 매도
        elif (self.ma_short < self.ma_long and self.rsi > self.upper_bound):
            # if self.position.is_long:
            self.position.close()
              #   self.sell()

            
bt = Backtest(data, MyStrategy, cash=10000000, commission=0.002)
stats, heatmap = bt.optimize(
    short_period=range(5, 45, 10), #np.arange(0.01, 0.05, 0.01).tolist(),
    long_period=range(20, 80, 10), #np.arange(0.1, 0.25, 0.01).tolist(),
    rsi_window=range(10, 22, 2),
    maximize = 'Equity Final [$]', #'Sharpe Ratio', 
    constraint = lambda param : param.short_period < param.long_period,
    return_heatmap=True,
    max_tries=100
    )

#print(heatmap)

hm = heatmap.groupby(['short_period', 'long_period']).mean().unstack()
#print(hm)
# sns.heatmap(hm, annot=True, cmap='viridis')
# plt.show()
plot_heatmaps(heatmap, agg='mean')

short_period = stats['_strategy'].short_period
long_period = stats['_strategy'].long_period
rsi_window = stats['_strategy'].rsi_window
print("best :", short_period,"-", long_period, "-", rsi_window)
print(stats)

bt.plot()
display(stats['_trades'])
```

    /Users/byeongsikbu/opt/anaconda3/envs/dct/lib/python3.10/site-packages/backtesting/backtesting.py:1375: UserWarning: For multiprocessing support in `Backtest.optimize()` set multiprocessing start method to 'fork'.
      warnings.warn("For multiprocessing support in `Backtest.optimize()` "



      0%|          | 0/9 [00:00<?, ?it/s]




<div class="bk-root" id="a340a795-663e-4f2e-807b-94af0a5741c3" data-root-id="164597"></div>





    best : 5 - 70 - 14
    Start                     2010-01-04 00:00:00
    End                       2023-10-06 00:00:00
    Duration                   5023 days 00:00:00
    Exposure Time [%]                   97.438163
    Equity Final [$]                  14697467.84
    Equity Peak [$]                   20285067.84
    Return [%]                          46.974678
    Buy & Hold Return [%]                41.23734
    Return (Ann.) [%]                    2.898779
    Volatility (Ann.) [%]               17.463108
    Sharpe Ratio                         0.165994
    Sortino Ratio                        0.242547
    Calmar Ratio                         0.071368
    Max. Drawdown [%]                   -40.61749
    Avg. Drawdown [%]                   -3.581652
    Max. Drawdown Duration     2205 days 00:00:00
    Avg. Drawdown Duration       96 days 00:00:00
    # Trades                                    1
    Win Rate [%]                            100.0
    Best Trade [%]                      47.040478
    Worst Trade [%]                     47.040478
    Avg. Trade [%]                      47.040478
    Max. Trade Duration        4897 days 00:00:00
    Avg. Trade Duration        4897 days 00:00:00
    Profit Factor                             NaN
    Expectancy [%]                      47.040478
    SQN                                       NaN
    _strategy                 MyStrategy(short...
    _equity_curve                             ...
    _trades                      Size  EntryBa...
    dtype: object




<div class="bk-root" id="8f80ad16-2c76-41ac-b3a9-93e253ce025d" data-root-id="165311"></div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>EntryBar</th>
      <th>ExitBar</th>
      <th>EntryPrice</th>
      <th>ExitPrice</th>
      <th>PnL</th>
      <th>ReturnPct</th>
      <th>EntryTime</th>
      <th>ExitTime</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>458</td>
      <td>87</td>
      <td>3395</td>
      <td>21803.52</td>
      <td>32060.0</td>
      <td>4697467.84</td>
      <td>0.470405</td>
      <td>2010-05-10</td>
      <td>2023-10-06</td>
      <td>4897 days</td>
    </tr>
  </tbody>
</table>
</div>



```python

```
