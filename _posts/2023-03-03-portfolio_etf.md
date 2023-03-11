```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings('ignore')
```


```python
#matplotlib 한글 폰트 출력코드

import matplotlib
from matplotlib import font_manager, rc
import platform

try :
    if platform.system() == 'Windows':
        # 윈도우인 경우
        font_name = font_manager.FontProperties(fname="c:/Windows/Fonts/malgun.ttf").get_name()
        rc('font', family=font_name)
    else:
        # Mac 인 경우
        rc('font', family='AppleGothic')
except :
    pass
matplotlib.rcParams['axes.unicode_minus'] = False
```


```python
df = pd.read_pickle("../crawling/data/etf_price.pickle")
print("최초 :", df.종목코드.nunique())

# 필터링
black_str = ['레버리지','인버스','원유',]
for black in black_str:
    cond = df['종목명'].str.contains(black)
    df = df[~cond]

print("레버리지 필터링 후 :",df.종목코드.nunique())

# 기간필터링
opendates = df.groupby("종목명")['날짜'].first().reset_index()
lastdates = df.groupby("종목명")['날짜'].last().reset_index()
lastdates.columns = ['종목명','날짜last']
oldates = pd.merge(opendates, lastdates, on='종목명')
cond = oldates['날짜']<'2017-01-01' ##### 조건
new_oldates = oldates[cond]
new_list = new_oldates['종목명'].tolist()
cond = df['종목명'].isin(new_list)
df = df[cond]


print("기간 필터링 후 :", df.종목코드.nunique())
df
```

    최초 : 678
    레버리지 필터링 후 : 591
    기간 필터링 후 : 168





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
      <th>NAV</th>
      <th>시가</th>
      <th>고가</th>
      <th>저가</th>
      <th>종가</th>
      <th>거래량</th>
      <th>거래대금</th>
      <th>기초지수</th>
      <th>종목명</th>
      <th>종목코드</th>
      <th>날짜</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>22631.83</td>
      <td>22390</td>
      <td>22585</td>
      <td>22390</td>
      <td>22550</td>
      <td>58738</td>
      <td>1325723895</td>
      <td>223.49</td>
      <td>ACE 200</td>
      <td>105190</td>
      <td>2010-01-04</td>
    </tr>
    <tr>
      <th>1</th>
      <td>22566.21</td>
      <td>22550</td>
      <td>22725</td>
      <td>22475</td>
      <td>22555</td>
      <td>47769</td>
      <td>1081075485</td>
      <td>222.84</td>
      <td>ACE 200</td>
      <td>105190</td>
      <td>2010-01-05</td>
    </tr>
    <tr>
      <th>2</th>
      <td>22748.91</td>
      <td>22625</td>
      <td>22735</td>
      <td>22620</td>
      <td>22715</td>
      <td>74937</td>
      <td>1701637315</td>
      <td>224.67</td>
      <td>ACE 200</td>
      <td>105190</td>
      <td>2010-01-06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>22409.69</td>
      <td>22740</td>
      <td>22740</td>
      <td>22415</td>
      <td>22455</td>
      <td>26044</td>
      <td>587893595</td>
      <td>221.31</td>
      <td>ACE 200</td>
      <td>105190</td>
      <td>2010-01-07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>22544.98</td>
      <td>22365</td>
      <td>22515</td>
      <td>22230</td>
      <td>22515</td>
      <td>11803</td>
      <td>264454100</td>
      <td>222.66</td>
      <td>ACE 200</td>
      <td>105190</td>
      <td>2010-01-08</td>
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
      <td>...</td>
    </tr>
    <tr>
      <th>793589</th>
      <td>24618.20</td>
      <td>24400</td>
      <td>24715</td>
      <td>24370</td>
      <td>24505</td>
      <td>63</td>
      <td>1546790</td>
      <td>2403.87</td>
      <td>파워 코스피100</td>
      <td>140950</td>
      <td>2023-02-23</td>
    </tr>
    <tr>
      <th>793590</th>
      <td>24421.21</td>
      <td>24505</td>
      <td>24620</td>
      <td>24310</td>
      <td>24310</td>
      <td>41</td>
      <td>1002770</td>
      <td>2384.57</td>
      <td>파워 코스피100</td>
      <td>140950</td>
      <td>2023-02-24</td>
    </tr>
    <tr>
      <th>793591</th>
      <td>24194.67</td>
      <td>24240</td>
      <td>24240</td>
      <td>24050</td>
      <td>24205</td>
      <td>105</td>
      <td>2532590</td>
      <td>2362.62</td>
      <td>파워 코스피100</td>
      <td>140950</td>
      <td>2023-02-27</td>
    </tr>
    <tr>
      <th>793592</th>
      <td>24266.16</td>
      <td>24205</td>
      <td>24465</td>
      <td>24205</td>
      <td>24270</td>
      <td>53</td>
      <td>1291550</td>
      <td>2369.38</td>
      <td>파워 코스피100</td>
      <td>140950</td>
      <td>2023-02-28</td>
    </tr>
    <tr>
      <th>793593</th>
      <td>24359.09</td>
      <td>24195</td>
      <td>24450</td>
      <td>24195</td>
      <td>24265</td>
      <td>56</td>
      <td>1360090</td>
      <td>0.00</td>
      <td>파워 코스피100</td>
      <td>140950</td>
      <td>2023-03-02</td>
    </tr>
  </tbody>
</table>
<p>410157 rows × 11 columns</p>
</div>




```python
data = df.pivot_table(index='날짜', columns = '종목명', values='종가')
# 월말 데이터로 필터링
data= data.resample("M").last()

data = data.loc[:"2023-02-28"]
data


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
      <th>종목명</th>
      <th>ACE 200</th>
      <th>ACE Fn성장소비주도주</th>
      <th>ACE 국고채3년</th>
      <th>ACE 단기통안채</th>
      <th>ACE 미국다우존스리츠(합성 H)</th>
      <th>ACE 밸류대형</th>
      <th>ACE 베트남VN30(합성)</th>
      <th>ACE 삼성그룹동일가중</th>
      <th>ACE 삼성그룹섹터가중</th>
      <th>ACE 인도네시아MSCI(합성)</th>
      <th>...</th>
      <th>TIGER 헬스케어</th>
      <th>TIGER 현대차그룹+펀더멘털</th>
      <th>TIGER 화장품</th>
      <th>TREX 200</th>
      <th>TREX 펀더멘탈 200</th>
      <th>마이다스 200커버드콜5%OTM</th>
      <th>마이티 코스피100</th>
      <th>파워 200</th>
      <th>파워 고배당저변동성</th>
      <th>파워 코스피100</th>
    </tr>
    <tr>
      <th>날짜</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010-01-31</th>
      <td>21240.0</td>
      <td>NaN</td>
      <td>102515.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6505.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21190.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-02-28</th>
      <td>21160.0</td>
      <td>NaN</td>
      <td>103350.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6650.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21030.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-03-31</th>
      <td>22485.0</td>
      <td>NaN</td>
      <td>103520.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7035.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>22415.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-04-30</th>
      <td>22870.0</td>
      <td>NaN</td>
      <td>104545.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7100.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>22860.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-05-31</th>
      <td>21385.0</td>
      <td>NaN</td>
      <td>104700.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6690.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21465.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <td>...</td>
    </tr>
    <tr>
      <th>2022-10-31</th>
      <td>30315.0</td>
      <td>5305.0</td>
      <td>100330.0</td>
      <td>101175.0</td>
      <td>76345.0</td>
      <td>7810.0</td>
      <td>16580.0</td>
      <td>15385.0</td>
      <td>15075.0</td>
      <td>11500.0</td>
      <td>...</td>
      <td>26945.0</td>
      <td>20050.0</td>
      <td>1745.0</td>
      <td>30765.0</td>
      <td>32485.0</td>
      <td>11545.0</td>
      <td>22885.0</td>
      <td>30430.0</td>
      <td>26585.0</td>
      <td>22790.0</td>
    </tr>
    <tr>
      <th>2022-11-30</th>
      <td>32465.0</td>
      <td>5900.0</td>
      <td>102155.0</td>
      <td>101430.0</td>
      <td>79030.0</td>
      <td>8365.0</td>
      <td>15785.0</td>
      <td>16080.0</td>
      <td>15595.0</td>
      <td>10785.0</td>
      <td>...</td>
      <td>26840.0</td>
      <td>21205.0</td>
      <td>2110.0</td>
      <td>32950.0</td>
      <td>34875.0</td>
      <td>12000.0</td>
      <td>24385.0</td>
      <td>32640.0</td>
      <td>28590.0</td>
      <td>24410.0</td>
    </tr>
    <tr>
      <th>2022-12-31</th>
      <td>29850.0</td>
      <td>6175.0</td>
      <td>101395.0</td>
      <td>100145.0</td>
      <td>75650.0</td>
      <td>7860.0</td>
      <td>15520.0</td>
      <td>15600.0</td>
      <td>14120.0</td>
      <td>9845.0</td>
      <td>...</td>
      <td>26210.0</td>
      <td>19705.0</td>
      <td>2335.0</td>
      <td>30310.0</td>
      <td>32665.0</td>
      <td>11195.0</td>
      <td>22500.0</td>
      <td>29970.0</td>
      <td>27610.0</td>
      <td>22375.0</td>
    </tr>
    <tr>
      <th>2023-01-31</th>
      <td>32555.0</td>
      <td>6515.0</td>
      <td>102830.0</td>
      <td>100485.0</td>
      <td>81675.0</td>
      <td>8480.0</td>
      <td>16815.0</td>
      <td>16265.0</td>
      <td>15260.0</td>
      <td>10040.0</td>
      <td>...</td>
      <td>25995.0</td>
      <td>21255.0</td>
      <td>2470.0</td>
      <td>32900.0</td>
      <td>35135.0</td>
      <td>12235.0</td>
      <td>24555.0</td>
      <td>32665.0</td>
      <td>29000.0</td>
      <td>24460.0</td>
    </tr>
    <tr>
      <th>2023-02-28</th>
      <td>32335.0</td>
      <td>6500.0</td>
      <td>101815.0</td>
      <td>100715.0</td>
      <td>78085.0</td>
      <td>8420.0</td>
      <td>16400.0</td>
      <td>16025.0</td>
      <td>15150.0</td>
      <td>10485.0</td>
      <td>...</td>
      <td>25220.0</td>
      <td>22240.0</td>
      <td>2400.0</td>
      <td>32660.0</td>
      <td>34880.0</td>
      <td>12240.0</td>
      <td>24450.0</td>
      <td>32445.0</td>
      <td>28725.0</td>
      <td>24270.0</td>
    </tr>
  </tbody>
</table>
<p>158 rows × 168 columns</p>
</div>



- ticker 랜덤으로 8개 선택하기
- 샤프비율 최대화 딕셔너리로 저장하기



```python
# KODEX 코스피와 상관계수가 0.7 이상인 ticker들 필터링
threshold = 0.7
common_tickers = ['KODEX 코스피']#,'KODEX 코스닥150']

###
ticker_list = data.columns.tolist()
try:
    for ticker in common_tickers:

        temp = data.corr()[ticker]
        high_corr_tickers = temp[temp>threshold].index.tolist()
        ticker_list = [i for i in ticker_list if i not in high_corr_tickers]
except:
    pass

print(len(ticker_list))
random_tickers = ticker_list.copy()
random_tickers
```

    72





    ['ACE Fn성장소비주도주',
     'ACE 국고채3년',
     'ACE 단기통안채',
     'ACE 인도네시아MSCI(합성)',
     'ACE 필리핀MSCI(합성)',
     'ARIRANG 고배당저변동50',
     'ARIRANG 고배당주',
     'ARIRANG 우량회사채50 1년',
     'HK S&P코리아로우볼',
     'KBSTAR 국고채3년',
     'KBSTAR 단기통안채',
     'KBSTAR 중기우량회사채',
     'KBSTAR 차이나HSCEI(H)',
     'KODEX 건설',
     'KODEX 골드선물(H)',
     'KODEX 국고채3년',
     'KODEX 국채선물10년',
     'KODEX 기계장비',
     'KODEX 단기채권',
     'KODEX 단기채권PLUS',
     'KODEX 미국S&P500에너지(합성)',
     'KODEX 미국달러선물',
     'KODEX 바이오',
     'KODEX 밸류Plus',
     'KODEX 보험',
     'KODEX 선진국MSCI World',
     'KODEX 은선물(H)',
     'KODEX 은행',
     'KODEX 차이나H',
     'KODEX 철강',
     'KODEX 콩선물(H)',
     'KODEX 퀄리티Plus',
     'KOSEF 고배당',
     'KOSEF 국고채10년',
     'KOSEF 국고채3년',
     'KOSEF 단기자금',
     'KOSEF 미국달러선물',
     'KOSEF 인도Nifty50(합성)',
     'KOSEF 통안채1년',
     'SOL 차이나강소기업CSI500(합성 H)',
     'TIGER 200 건설',
     'TIGER 200 경기소비재',
     'TIGER 200 금융',
     'TIGER 200 산업재',
     'TIGER 200 생활소비재',
     'TIGER 200 중공업',
     'TIGER 200 철강소재',
     'TIGER 200 헬스케어',
     'TIGER S&P글로벌헬스케어(합성)',
     'TIGER 경기방어',
     'TIGER 경기방어채권혼합',
     'TIGER 국채3년',
     'TIGER 금은선물(H)',
     'TIGER 농산물선물Enhanced(H)',
     'TIGER 단기통안채',
     'TIGER 라틴35',
     'TIGER 로우볼',
     'TIGER 모멘텀',
     'TIGER 미국MSCI리츠(합성 H)',
     'TIGER 미국다우존스30',
     'TIGER 미디어컨텐츠',
     'TIGER 방송통신',
     'TIGER 우량가치',
     'TIGER 유로스탁스배당30',
     'TIGER 은행',
     'TIGER 중국소비테마',
     'TIGER 차이나HSCEI',
     'TIGER 차이나항셍25',
     'TIGER 코스닥150바이오테크',
     'TIGER 헬스케어',
     'TIGER 화장품',
     '파워 고배당저변동성']



### 랜덤종목선택하기


```python
# 종목개수
num_of_tickers = 5

tickers = common_tickers + np.random.choice(random_tickers,num_of_tickers-len(common_tickers)).tolist()
tickers
```




    ['KODEX 코스피',
     'TIGER 200 경기소비재',
     'ARIRANG 고배당주',
     'TIGER 200 철강소재',
     'TIGER 경기방어']



### 최적화


```python
from scipy.optimize import minimize
```


```python
data[tickers].dropna().plot()
```




    <Axes: xlabel='날짜'>




    
![png](output_10_1.png)
    



```python
# 수익률
rets = data[tickers].pct_change().dropna().fillna(0)
# 팔레트
pal = sns.color_palette("Spectral", len(tickers))
```


```python
# 초기값 설정
noa = rets.shape[1]
init_guess = np.repeat(1/noa, noa)

# 기대수익률 벡터
er = rets.mean() * 12

# 공분산 행렬
cov = rets.cov() * 12

# 각 ETF별 투자 가중치 상하한선 : 공매도 불가 조건
bounds = ((0.0, 1.0), ) * noa # 종목 개수만큼 

# 제약조건
weights_sum_to_1 = {'type':'eq',
                    'fun' : lambda weights : np.sum(weights)-1}

# 목적함수 : 마이너스 샤프비율
def neg_sharpe(weights, er, cov):
    r = weights.T @ er 
    vol = np.sqrt(weights.T @ cov @ weights)
    return -r / vol

# 최적화 알고리즘 구현
res = minimize(neg_sharpe, 
               init_guess,
               args = (er, cov),
               method = "SLSQP",
               constraints = (weights_sum_to_1,),
               bounds = bounds)

# 가중치 결과값 
weights = res.x

# 결과 출력
print(weights)
```

    [1.00000000e+00 0.00000000e+00 4.10609047e-15 0.00000000e+00
     0.00000000e+00]



```python
# 가중치 데이터프레임 생성
weights_df = pd.Series(np.round(weights,3), index=tickers)
weights_df = weights_df[weights_df > 0.0]

# 파이차트 시각화
plt.figure(figsize=(7,7))
wedgeprops = {'width':0.3, 'edgecolor':'w', 'linewidth':2}
plt.pie(weights_df, labels=weights_df.index, autopct="%.1f%%", wedgeprops=wedgeprops, colors=pal)
plt.show()
```


    
![png](output_13_0.png)
    



```python
# MSR 모델 가중치 계산 함수
def get_msr_weights(er, cov):
    # 자산 개수 
    noa = er.shape[0]

    # 초기 가중치
    init_guess = np.repeat(1/noa, noa)

    # 제약조건 및 상하한값
    bounds = ((0.0, 1.0), ) * noa
    weights_sum_to_1 = {'type' : 'eq',
                        'fun' : lambda weights:np.sum(weights)-1}
    
    # 목적함수 : 네거티브 샤프비율
    def neg_sharpe(weights, er, cov):
        r = weights.T @ er
        vol = np.sqrt(weights.T @ cov @ weights)
        return -r / vol

    # 최적화 수행
    res = minimize(neg_sharpe, 
                   init_guess,
                   args = (er, cov),
                   method = "SLSQP",
                   constraints = (weights_sum_to_1,),
                   bounds = bounds)
    
    return res.x #가중치 결과
```


```python
# 빈 데이터 프레임 생성
msr_w_df = pd.DataFrame().reindex_like(rets) #rets의 인덱스를 가져오는

# 기대수익률 배열
er = np.array(rets * 12)

# 공분산행렬 배열
cov = rets.rolling(12).cov().fillna(0) * 12 
print(cov.shape)
cov
```

    (390, 5)





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
      <th>종목명</th>
      <th>KODEX 코스피</th>
      <th>TIGER 경기방어채권혼합</th>
      <th>KBSTAR 중기우량회사채</th>
      <th>KODEX 선진국MSCI World</th>
      <th>KBSTAR 중기우량회사채</th>
    </tr>
    <tr>
      <th>날짜</th>
      <th>종목명</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">2016-09-30</th>
      <th>KODEX 코스피</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>TIGER 경기방어채권혼합</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>KBSTAR 중기우량회사채</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>KODEX 선진국MSCI World</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>KBSTAR 중기우량회사채</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">2023-02-28</th>
      <th>KODEX 코스피</th>
      <td>0.063821</td>
      <td>0.011485</td>
      <td>0.005284</td>
      <td>0.030469</td>
      <td>0.005284</td>
    </tr>
    <tr>
      <th>TIGER 경기방어채권혼합</th>
      <td>0.011485</td>
      <td>0.003876</td>
      <td>0.001281</td>
      <td>0.003598</td>
      <td>0.001281</td>
    </tr>
    <tr>
      <th>KBSTAR 중기우량회사채</th>
      <td>0.005284</td>
      <td>0.001281</td>
      <td>0.001310</td>
      <td>0.001844</td>
      <td>0.001310</td>
    </tr>
    <tr>
      <th>KODEX 선진국MSCI World</th>
      <td>0.030469</td>
      <td>0.003598</td>
      <td>0.001844</td>
      <td>0.036409</td>
      <td>0.001844</td>
    </tr>
    <tr>
      <th>KBSTAR 중기우량회사채</th>
      <td>0.005284</td>
      <td>0.001281</td>
      <td>0.001310</td>
      <td>0.001844</td>
      <td>0.001310</td>
    </tr>
  </tbody>
</table>
<p>390 rows × 5 columns</p>
</div>




```python
# 공분산 행렬을 3차원 배열로 변환
cov = cov.values.reshape(int(cov.shape[0] / cov.shape[1]), cov.shape[1], cov.shape[1])
print(cov.shape)
```

    (78, 5, 5)



```python
from tqdm import tqdm
for i in tqdm(range(12, len(msr_w_df))): # 처음 12개월은 값이 없음. cov계산에 사용되어있어서
    msr_w_df.iloc[i] = get_msr_weights(er[i-1], cov[i-1])
    
```

    100%|██████████████████████████████████████████| 66/66 [00:00<00:00, 687.68it/s]



```python
# 매달 투자 가중치
msr_w_df
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
      <th>종목명</th>
      <th>KODEX 코스피</th>
      <th>TIGER 경기방어채권혼합</th>
      <th>KBSTAR 중기우량회사채</th>
      <th>KODEX 선진국MSCI World</th>
      <th>KBSTAR 중기우량회사채</th>
    </tr>
    <tr>
      <th>날짜</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-09-30</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-10-31</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-11-30</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-12-31</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-01-31</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2022-10-31</th>
      <td>0.000000e+00</td>
      <td>1.360023e-15</td>
      <td>6.383782e-16</td>
      <td>1.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>2022-11-30</th>
      <td>0.000000e+00</td>
      <td>4.996004e-16</td>
      <td>4.440892e-16</td>
      <td>1.000000e+00</td>
      <td>2.220446e-16</td>
    </tr>
    <tr>
      <th>2022-12-31</th>
      <td>3.701717e-02</td>
      <td>4.567498e-01</td>
      <td>2.531165e-01</td>
      <td>0.000000e+00</td>
      <td>2.531166e-01</td>
    </tr>
    <tr>
      <th>2023-01-31</th>
      <td>4.881068e-09</td>
      <td>6.665019e-08</td>
      <td>4.999998e-01</td>
      <td>4.288940e-09</td>
      <td>5.000002e-01</td>
    </tr>
    <tr>
      <th>2023-02-28</th>
      <td>3.638968e-13</td>
      <td>4.425384e-13</td>
      <td>4.876049e-01</td>
      <td>2.479019e-02</td>
      <td>4.876049e-01</td>
    </tr>
  </tbody>
</table>
<p>78 rows × 5 columns</p>
</div>




```python
plt.figure(figsize=(20,5))
plt.stackplot(msr_w_df.index, msr_w_df.T, labels=msr_w_df.columns, colors=pal)
plt.legend(loc='upper left')
plt.title("MSR Weights")
plt.xlabel("Date")
plt.ylabel("Weights")
```




    Text(0, 0.5, 'Weights')




    
![png](output_19_1.png)
    



```python
# 포트폴리오 수익률 데이터프레임
port_rets = msr_w_df.shift() * rets
port_cum_rets = (1+port_rets.sum(axis=1)).cumprod() - 1

# MSR 백테스팅 시각화
plt.plot(port_cum_rets.iloc[12:])
plt.title("MSR Backtest")
plt.xlabel("Date")
plt.ylabel("Returns")
```




    Text(0, 0.5, 'Returns')




    
![png](output_20_1.png)
    



```python

```


```python

```


```python
from tqdm import tqdm
```

### 자동으로 돌리기


```python
port = []
ret = []
ret_12m = []
sharpe = []
########################################
cnt = 50 #시나리오 개수
num_of_tickers = 6 # 종목개수
rolling_window = 12 # 롤링윈도우
# KODEX 코스피와 상관계수가 0.7 이상인 ticker들 필터링
threshold = 1 #상관계수 필터링 조건
common_tickers = ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학'] # 무조건 넣는 ticker
########################################

ticker_list = data.columns.tolist()
for ticker in common_tickers:

    temp = data.corr()[ticker]
    high_corr_tickers = temp[temp>threshold].index.tolist()
    ticker_list = [i for i in ticker_list if i not in high_corr_tickers]
random_tickers = ticker_list.copy()
print(num_of_tickers, threshold, len(random_tickers))
for _ in tqdm(range(cnt)):

    tickers = common_tickers + np.random.choice(random_tickers, num_of_tickers-len(common_tickers), False).tolist() # False: 중복비허용
    
    # 수익률
    rets = data[tickers].pct_change().dropna().fillna(0) #최근 있는 데이터로만 모델링

    # 빈 데이터 프레임 생성
    msr_w_df = pd.DataFrame().reindex_like(rets) #rets의 인덱스를 가져오는

    # 기대수익률 배열
    er = np.array(rets * 12)

    # 공분산행렬 배열
    cov = rets.rolling(12).cov().fillna(0) * 12 

    # 공분산 행렬을 3차원 배열로 변환
    cov = cov.values.reshape(int(cov.shape[0] / cov.shape[1]), cov.shape[1], cov.shape[1])

    for i in range(rolling_window, len(msr_w_df)): # 처음 12개월은 값이 없음. cov계산에 사용되어있어서
        msr_w_df.iloc[i] = get_msr_weights(er[i-1], cov[i-1])
        #msr_w_df.iloc[i] = get_rp_weights(cov[i-1]) # 리스크패리티는 성과가 좋지 않음......


    # 포트폴리오 수익률 데이터프레임
    port_rets = msr_w_df.shift() * rets
    port_cum_rets = (1+port_rets.sum(axis=1)).cumprod() - 1
    port_sharpe = port_rets.sum(axis=1).mean()*12 / (port_rets.sum(axis=1).std()*np.sqrt(12))
    #port_cum_rets.plot()
    port.append(tickers)
    ret.append(port_cum_rets[-1])
    ret_12m.append((port_rets.sum(axis=1)[-12:]+1).cumprod()[-1]-1)
    sharpe.append(port_sharpe)
    # 최고값 초기화
    if _==0:
        top = port_cum_rets[-1]
        print(port_cum_rets[-1].round(2), ret_12m[-1].round(2), tickers) 
    if port_cum_rets[-1] > top:
        print(port_cum_rets[-1].round(2), ret_12m[-1].round(2), tickers) 
        top = port_cum_rets[-1]
    
```

    6 1 168


      2%|▉                                           | 1/50 [00:00<00:06,  7.33it/s]

    2.07 0.11 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'ACE 미국다우존스리츠(합성 H)']


      8%|███▌                                        | 4/50 [00:00<00:07,  6.31it/s]

    2.42 0.17 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER 구리실물']


     12%|█████▎                                      | 6/50 [00:00<00:05,  7.49it/s]

    2.44 0.18 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'KODEX 코스피']
    2.7 0.26 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER 유로스탁스50(합성 H)']


     50%|█████████████████████▌                     | 25/50 [00:03<00:03,  7.19it/s]

    3.41 0.33 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER 코스닥150']


    100%|███████████████████████████████████████████| 50/50 [00:06<00:00,  7.89it/s]



```python
results = pd.DataFrame({"port":port,"sharpe":sharpe,"ret":ret,"ret_12m":ret_12m})
results['ret'].plot.hist(bins=100)
```




    <Axes: ylabel='Frequency'>




    
![png](output_26_1.png)
    



```python
results = pd.DataFrame({"port":port,"sharpe":sharpe,"ret":ret,"ret_12m":ret_12m})
results = results.sort_values(by='ret', ascending=False)
results[:10]
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
      <th>port</th>
      <th>sharpe</th>
      <th>ret</th>
      <th>ret_12m</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>23</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.425542</td>
      <td>3.407336</td>
      <td>0.326655</td>
    </tr>
    <tr>
      <th>34</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.286572</td>
      <td>2.764599</td>
      <td>0.209691</td>
    </tr>
    <tr>
      <th>5</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.315019</td>
      <td>2.698864</td>
      <td>0.256279</td>
    </tr>
    <tr>
      <th>48</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.178905</td>
      <td>2.664425</td>
      <td>0.163214</td>
    </tr>
    <tr>
      <th>16</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.274011</td>
      <td>2.651698</td>
      <td>0.207132</td>
    </tr>
    <tr>
      <th>41</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.311626</td>
      <td>2.635712</td>
      <td>0.218854</td>
    </tr>
    <tr>
      <th>47</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.311626</td>
      <td>2.635712</td>
      <td>0.218854</td>
    </tr>
    <tr>
      <th>9</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.311626</td>
      <td>2.635712</td>
      <td>0.218854</td>
    </tr>
    <tr>
      <th>19</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.312296</td>
      <td>2.626763</td>
      <td>0.211982</td>
    </tr>
    <tr>
      <th>37</th>
      <td>[KOSEF 미국달러선물, KODEX 미국S&amp;P500산업재(합성), KODEX 미국...</td>
      <td>1.209887</td>
      <td>2.566055</td>
      <td>0.213306</td>
    </tr>
  </tbody>
</table>
</div>




```python
idxs = results[:10].index.tolist()
for idx in idxs:
    tickers = results.loc[idx]['port']
    print(idx, tickers)
    # 수익률
    rets = data[tickers].pct_change().dropna().fillna(0)
    # 팔레트
    pal = sns.color_palette("Spectral", len(tickers))
    # 빈 데이터 프레임 생성
    msr_w_df = pd.DataFrame().reindex_like(rets) #rets의 인덱스를 가져오는

    # 기대수익률 배열
    er = np.array(rets * 12)

    # 공분산행렬 배열
    cov = rets.rolling(12).cov().fillna(0) * 12 

    # 공분산 행렬을 3차원 배열로 변환
    cov = cov.values.reshape(int(cov.shape[0] / cov.shape[1]), cov.shape[1], cov.shape[1])

    for i in range(12, len(msr_w_df)): # 처음 12개월은 값이 없음. cov계산에 사용되어있어서
        msr_w_df.iloc[i] = get_msr_weights(er[i-1], cov[i-1])

    # 포트폴리오 수익률 데이터프레임
    port_rets = msr_w_df.shift() * rets
    port_cum_rets = (1+port_rets.sum(axis=1)).cumprod() - 1
    

    # 시각화
    fig, ax = plt.subplots(figsize=(8,5))
    ax.stackplot(msr_w_df.index, msr_w_df.T, labels=msr_w_df.columns, colors=pal)
    ax.legend(loc='upper left')
    ax.set_title(f"{idx} {tickers }MSR Weights", fontsize=8)
    ax.set_xlabel("Date")
    ax.set_ylabel("Weights")
    ax2 = ax.twinx()
    ax2.plot(port_cum_rets, lw=1, c='black')#, ls='--')
```

    23 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER 코스닥150']
    34 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'KBSTAR 5대그룹주']
    5 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER 유로스탁스50(합성 H)']
    48 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'KODEX 미국S&P바이오(합성)']
    16 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'ACE 밸류대형']
    41 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER KTOP30']
    47 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER KTOP30']
    9 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER KTOP30']
    19 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'KODEX KTOP30']
    37 ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER 미국나스닥바이오']



    
![png](output_28_1.png)
    



    
![png](output_28_2.png)
    



    
![png](output_28_3.png)
    



    
![png](output_28_4.png)
    



    
![png](output_28_5.png)
    



    
![png](output_28_6.png)
    



    
![png](output_28_7.png)
    



    
![png](output_28_8.png)
    



    
![png](output_28_9.png)
    



    
![png](output_28_10.png)
    



```python
ticker_number = 23
tickers = results.loc[ticker_number]['port']


print(tickers)
# 수익률
rets = data[tickers].pct_change().dropna().fillna(0)

# 빈 데이터 프레임 생성
msr_w_df = pd.DataFrame().reindex_like(rets) #rets의 인덱스를 가져오는

# 기대수익률 배열
er = np.array(rets * 12)

# 공분산행렬 배열
cov = rets.rolling(12).cov().fillna(0) * 12 

# 공분산 행렬을 3차원 배열로 변환
cov = cov.values.reshape(int(cov.shape[0] / cov.shape[1]), cov.shape[1], cov.shape[1])

for i in range(12, len(msr_w_df)): # 처음 12개월은 값이 없음. cov계산에 사용되어있어서
    msr_w_df.iloc[i] = get_msr_weights(er[i-1], cov[i-1])

# 포트폴리오 수익률 데이터프레임
port_rets = msr_w_df.shift() * rets
port_cum_rets = (1+port_rets.sum(axis=1)).cumprod() - 1
port_sharpe = port_rets.sum(axis=1).mean()*12 / (port_rets.sum(axis=1).std()*np.sqrt(12))
# 시각화
fig, ax = plt.subplots(figsize=(8,5))
ax.stackplot(msr_w_df.index, msr_w_df.T, labels=msr_w_df.columns, colors=pal)
ax.legend(loc='upper left')
ax.set_title("MSR Weights")
ax.set_xlabel("Date")
ax.set_ylabel("Weights")
ax2 = ax.twinx()
ax2.plot(port_cum_rets, lw=1, c='black')#, ls='--')
```

    ['KOSEF 미국달러선물', 'KODEX 미국S&P500산업재(합성)', 'KODEX 미국S&P500에너지(합성)', 'TIGER 200 IT', 'TIGER 200 에너지화학', 'TIGER 코스닥150']





    [<matplotlib.lines.Line2D at 0x16a2367d0>]




    
![png](output_29_2.png)
    



```python
# 시뮬레이션 투자비중, 수익률
periods=72
fig, ax = plt.subplots(figsize=(10,5+periods*0.2))
weights = msr_w_df.shift().iloc[-periods:].reset_index()
returns = pd.DataFrame(port_rets.sum(axis=1), columns=['수익률']).reset_index()
temp = pd.merge(weights, returns, on='날짜', how='inner').set_index("날짜")
sns.heatmap(temp.round(2), cmap='Blues', annot=True, ax=ax)
ax.set_yticklabels(weights.날짜.apply(lambda x:x.strftime("%Y-%m-%d")))
;
```




    ''




    
![png](output_30_1.png)
    



```python
# 투자 비중
msr_w_df.iloc[-1].round(2)
```




    종목명
    KOSEF 미국달러선물             0.22
    KODEX 미국S&P500산업재(합성)    0.00
    KODEX 미국S&P500에너지(합성)    0.00
    TIGER 200 IT             0.78
    TIGER 200 에너지화학          0.00
    TIGER 코스닥150             0.00
    Name: 2023-02-28 00:00:00, dtype: float64




```python
import quantstats as qs
benchmark = qs.utils.download_returns("SPY")
benchmark = benchmark.tz_convert(None)
# 슬리피지 0.003
qs.reports.full((port_rets.sum(axis=1)-0.001).iloc[12:], benchmark=benchmark)
```


<h4>Performance Metrics</h4>


                               Strategy    Benchmark
    -------------------------  ----------  -----------
    Start Period               2016-12-31  2016-12-31
    End Period                 2023-02-28  2023-02-28
    Risk-Free Rate             0.0%        0.0%
    Time in Market             100.0%      99.0%
    
    Cumulative Return          309.44%     95.34%
    CAGR﹪                     25.69%      11.47%
    
    Sharpe                     6.79        3.27
    Prob. Sharpe Ratio         100.0%      95.07%
    Smart Sharpe               6.34        3.05
    Sortino                    19.46       4.83
    Smart Sortino              18.18       4.51
    Sortino/√2                 13.76       3.42
    Smart Sortino/√2           12.86       3.19
    Omega                      3.72        3.72
    
    Max Drawdown               -12.55%     -23.93%
    Longest DD Days            184         393
    Volatility (ann.)          74.24%      78.72%
    R^2                        0.13        0.13
    Information Ratio          0.18        0.18
    Calmar                     2.05        0.48
    Skew                       1.15        -0.7
    Kurtosis                   1.73        1.23
    
    Expected Daily %           1.9%        0.9%
    Expected Monthly %         1.9%        0.9%
    Expected Yearly %          19.27%      8.73%
    Kelly Criterion            46.22%      24.44%
    Risk of Ruin               0.0%        0.0%
    Daily Value-at-Risk        -5.69%      -7.14%
    Expected Shortfall (cVaR)  -5.69%      -7.14%
    
    Max Consecutive Wins       8           13
    Max Consecutive Losses     3           3
    Gain/Pain Ratio            2.72        0.69
    Gain/Pain (1M)             2.72        0.69
    
    Payoff Ratio               1.81        0.65
    Profit Factor              3.72        1.69
    Common Sense Ratio         12.15       1.65
    CPC Index                  4.41        0.77
    Tail Ratio                 3.26        0.98
    Outlier Win Ratio          3.12        3.69
    Outlier Loss Ratio         4.07        1.71
    
    MTD                        11.83%      -2.51%
    3M                         6.21%       3.07%
    6M                         15.11%      -2.69%
    YTD                        21.46%      4.05%
    1Y                         30.39%      -10.5%
    3Y (ann.)                  33.0%       10.7%
    5Y (ann.)                  24.77%      9.87%
    10Y (ann.)                 25.69%      11.47%
    All-time (ann.)            25.69%      11.47%
    
    Best Day                   16.76%      12.7%
    Worst Day                  -7.01%      -16.12%
    Best Month                 16.76%      12.7%
    Worst Month                -7.01%      -16.12%
    Best Year                  41.39%      31.22%
    Worst Year                 -0.1%       -18.52%
    
    Avg. Drawdown              -3.98%      -9.43%
    Avg. Drawdown Days         80          121
    Recovery Factor            24.65       3.98
    Ulcer Index                0.03        0.07
    Serenity Index             86.77       4.39
    
    Avg. Up Month              4.35%       3.5%
    Avg. Down Month            -2.4%       -5.4%
    Win Days %                 65.33%      70.27%
    Win Month %                65.33%      70.27%
    Win Quarter %              80.77%      76.0%
    Win Year %                 87.5%       71.43%
    
    Beta                       0.35        -
    Alpha                      4.15        -
    Correlation                36.71%      -
    Treynor Ratio              893.82%     -



    None



<h4>5 Worst Drawdowns</h4>



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
      <th>Start</th>
      <th>Valley</th>
      <th>End</th>
      <th>Days</th>
      <th>Max Drawdown</th>
      <th>99% Max Drawdown</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2022-11-30</td>
      <td>2022-12-31</td>
      <td>2023-02-28</td>
      <td>90</td>
      <td>-12.552405</td>
      <td>-7.009379</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-02-28</td>
      <td>2018-03-31</td>
      <td>2018-07-31</td>
      <td>153</td>
      <td>-6.449524</td>
      <td>-5.984693</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-10-31</td>
      <td>2018-12-31</td>
      <td>2019-01-31</td>
      <td>92</td>
      <td>-5.909427</td>
      <td>-5.693330</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-06-30</td>
      <td>2019-10-31</td>
      <td>2019-12-31</td>
      <td>184</td>
      <td>-5.238342</td>
      <td>-3.652545</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2020-02-29</td>
      <td>2020-03-31</td>
      <td>2020-07-31</td>
      <td>153</td>
      <td>-4.926595</td>
      <td>-4.309929</td>
    </tr>
  </tbody>
</table>
</div>



<h4>Strategy Visualization</h4>



    
![png](output_32_6.png)
    



    
![png](output_32_7.png)
    



    
![png](output_32_8.png)
    



    
![png](output_32_9.png)
    



    
![png](output_32_10.png)
    



    
![png](output_32_11.png)
    



    
![png](output_32_12.png)
    



    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    Cell In[29], line 5
          3 benchmark = benchmark.tz_convert(None)
          4 # 슬리피지 0.003
    ----> 5 qs.reports.full((port_rets.sum(axis=1)-0.001).iloc[12:], benchmark=benchmark)


    File ~/opt/anaconda3/envs/dct/lib/python3.10/site-packages/quantstats/reports.py:317, in full(returns, benchmark, rf, grayscale, figsize, display, compounded, periods_per_year, match_dates)
        314     print('\n\n')
        315     print('[Strategy Visualization]\nvia Matplotlib')
    --> 317 plots(returns=returns, benchmark=benchmark,
        318       grayscale=grayscale, figsize=figsize, mode='full',
        319       periods_per_year=periods_per_year, prepare_returns=False)


    File ~/opt/anaconda3/envs/dct/lib/python3.10/site-packages/quantstats/reports.py:707, in plots(returns, benchmark, grayscale, figsize, mode, compounded, periods_per_year, prepare_returns, match_dates)
        700 if benchmark is not None:
        701     _plots.rolling_beta(returns, benchmark, grayscale=grayscale,
        702                         window1=win_half_year, window2=win_year,
        703                         figsize=(figsize[0], figsize[0]*.3),
        704                         show=True, ylabel=False,
        705                         prepare_returns=False)
    --> 707 _plots.rolling_volatility(
        708     returns, benchmark, grayscale=grayscale,
        709     figsize=(figsize[0], figsize[0]*.3), show=True, ylabel=False,
        710     period=win_half_year)
        712 _plots.rolling_sharpe(returns, grayscale=grayscale,
        713                       figsize=(figsize[0], figsize[0]*.3),
        714                       show=True, ylabel=False, period=win_half_year)
        716 _plots.rolling_sortino(returns, grayscale=grayscale,
        717                        figsize=(figsize[0], figsize[0]*.3),
        718                        show=True, ylabel=False, period=win_half_year)


    File ~/opt/anaconda3/envs/dct/lib/python3.10/site-packages/quantstats/_plotting/wrappers.py:543, in rolling_volatility(returns, benchmark, period, period_label, periods_per_year, lw, fontname, grayscale, figsize, ylabel, subtitle, savefig, show)
        539     benchmark = _utils._prepare_benchmark(benchmark, returns.index)
        540     benchmark = _stats.rolling_volatility(
        541         benchmark, period, periods_per_year, prepare_returns=False)
    --> 543 fig = _core.plot_rolling_stats(returns, benchmark,
        544                                hline=returns.mean(),
        545                                hlw=1.5,
        546                                ylabel=ylabel,
        547                                title='Rolling Volatility (%s)' % period_label,
        548                                fontname=fontname,
        549                                grayscale=grayscale,
        550                                lw=lw,
        551                                figsize=figsize,
        552                                subtitle=subtitle,
        553                                savefig=savefig, show=show)
        554 if not show:
        555     return fig


    File ~/opt/anaconda3/envs/dct/lib/python3.10/site-packages/quantstats/_plotting/core.py:435, in plot_rolling_stats(returns, benchmark, title, returns_label, hline, hlw, hlcolor, hllabel, lw, figsize, ylabel, grayscale, fontname, subtitle, savefig, show)
        430 fig.suptitle(title+"\n", y=.99, fontweight="bold", fontname=fontname,
        431              fontsize=14, color="black")
        433 if subtitle:
        434     ax.set_title("\n%s - %s                   " % (
    --> 435         df.index.date[:1][0].strftime('%e %b \'%y'),
        436         df.index.date[-1:][0].strftime('%e %b \'%y')
        437     ), fontsize=12, color='gray')
        439 if hline:
        440     if grayscale:


    IndexError: index 0 is out of bounds for axis 0 with size 0



    
![png](output_32_14.png)
    



```python

```


```python

```


```python
tickers
```




    ['KOSEF 미국달러선물',
     'KODEX 미국S&P500산업재(합성)',
     'KODEX 미국S&P500에너지(합성)',
     'TIGER 200 IT',
     'TIGER 200 에너지화학']




```python

```
