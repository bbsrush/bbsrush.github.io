---
layout : single
classes: wide
title : "연금 포트폴리오 종목 선정하기 feat. 유전알고리즘"
categories: portfolio
tag: [python, portfolio, genetic algorithm, ga, pygad, crawling]
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
author_profile: false
sidebar:
    nav: "docs"
Typora-root-url: ../

---

# ETF Portfolio

- 가격데이터 크롤링
- 투자비중 : 샤프비율최대화(12개월 롤링)
- 종목선택 > 랜덤 5종목
- 종목선택 > 유전알고리즘 5종목 


### 라이브러리


```python
from pykrx import stock
import requests
import json
from pandas.io.json import json_normalize
import pandas as pd
import datetime
import time
from tqdm import tqdm
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings('ignore')
from scipy.optimize import minimize

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

## 1. Crawling

### ETF 종목명 불러오기


```python
# https://yobro.tistory.com/197
date = datetime.datetime.now().strftime("%Y%m%d")
tickers = stock.get_etf_ticker_list(date)
tickers=pd.DataFrame(tickers,columns=['종목코드'])

url = 'https://finance.naver.com/api/sise/etfItemList.nhn'
json_data = json.loads(requests.get(url).text)
df = json_normalize(json_data['result']['etfItemList'])
df=df[['itemcode','itemname']]
df=df.rename(columns={"itemcode": "종목코드", "itemname": "종목명"})

etf=pd.merge(left=tickers,right=df,how='left',on='종목코드' )
etf
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
      <th>종목코드</th>
      <th>종목명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>445690</td>
      <td>BNK 주주가치액티브</td>
    </tr>
    <tr>
      <th>1</th>
      <td>292340</td>
      <td>마이티 200커버드콜ATM레버리지</td>
    </tr>
    <tr>
      <th>2</th>
      <td>442260</td>
      <td>마이티 다이나믹퀀트액티브</td>
    </tr>
    <tr>
      <th>3</th>
      <td>159800</td>
      <td>마이티 코스피100</td>
    </tr>
    <tr>
      <th>4</th>
      <td>361580</td>
      <td>KBSTAR 200TR</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>673</th>
      <td>195980</td>
      <td>ARIRANG 신흥국MSCI(합성 H)</td>
    </tr>
    <tr>
      <th>674</th>
      <td>433250</td>
      <td>UNICORN R&amp;D 액티브</td>
    </tr>
    <tr>
      <th>675</th>
      <td>215620</td>
      <td>HK S&amp;P코리아로우볼</td>
    </tr>
    <tr>
      <th>676</th>
      <td>391670</td>
      <td>HK 베스트일레븐액티브</td>
    </tr>
    <tr>
      <th>677</th>
      <td>391680</td>
      <td>HK 하이볼액티브</td>
    </tr>
  </tbody>
</table>
<p>678 rows × 2 columns</p>
</div>

총 678개의 ETF 종목명을 불러왔습니다. 

### 주가 데이터 크롤링(일단위 OHLCV) 


```python
# df = pd.DataFrame(columns=['NAV','시가','고가','저가','종가','거래량','거래대금','기초지수','종목명','종목코드'])
# error_list = []
# for a, b in tqdm(etf[['종목명', '종목코드']].itertuples(index=False)):
#     try:
#         price = stock.get_etf_ohlcv_by_date("20100101", "20230309", b)
#         price['종목명']=a
#         price['종목코드']=b
#         price = price.reset_index()
#         df = pd.concat([df,price])
#     except:
#         error_list.append(a)
#     time.sleep(1)
# etf_price = df.reset_index(drop=True)
# etf_price
```

### 데이터 필터링


```python
#저장
#etf_price.to_pickle("../crawling/data/etf_price.pickle") 

#불러오기
df = pd.read_pickle("../crawling/data/etf_price.pickle")
print("<<종목 수>>")
print("최초 :", df.종목코드.nunique())

# 필터링
black_str = ['레버리지','인버스','원유',"채"] #채권은 샤프비율을 높이는데 도움은 되지만 실질적으로 수익률을 끌어올리지 못해서 제외
for black in black_str:
    cond = df['종목명'].str.contains(black)
    df = df[~cond]

print("레버리지 필터링 후 :",df.종목코드.nunique())

# 기간필터링
opendates = df.groupby("종목명")['날짜'].first().reset_index()
lastdates = df.groupby("종목명")['날짜'].last().reset_index()
lastdates.columns = ['종목명','날짜last']
oldates = pd.merge(opendates, lastdates, on='종목명')
cond = oldates['날짜']<'2017-01-01' ##### 2017년 이전 상장 
new_oldates = oldates[cond]
new_list = new_oldates['종목명'].tolist()
cond = df['종목명'].isin(new_list)
etf_price = df[cond]


print("기간 필터링 후 :", etf_price.종목코드.nunique())
etf_price
```

    <<종목 수>>
    최초 : 678
    레버리지 필터링 후 : 498
    기간 필터링 후 : 146

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
      <th>2020</th>
      <td>9067.76</td>
      <td>8915</td>
      <td>9010</td>
      <td>8850</td>
      <td>8925</td>
      <td>34137</td>
      <td>304969115</td>
      <td>978.07</td>
      <td>ARIRANG 신흥국MSCI(합성 H)</td>
      <td>195980</td>
      <td>2023-03-08</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>8937.23</td>
      <td>8925</td>
      <td>9035</td>
      <td>8910</td>
      <td>8955</td>
      <td>22681</td>
      <td>203142255</td>
      <td>974.15</td>
      <td>ARIRANG 신흥국MSCI(합성 H)</td>
      <td>195980</td>
      <td>2023-03-09</td>
    </tr>
    <tr>
      <th>2025</th>
      <td>12082.88</td>
      <td>12040</td>
      <td>12040</td>
      <td>12040</td>
      <td>12040</td>
      <td>19</td>
      <td>228760</td>
      <td>9466.48</td>
      <td>HK S&amp;P코리아로우볼</td>
      <td>215620</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>2026</th>
      <td>11991.12</td>
      <td>12035</td>
      <td>12035</td>
      <td>12020</td>
      <td>12020</td>
      <td>2</td>
      <td>24055</td>
      <td>9386.54</td>
      <td>HK S&amp;P코리아로우볼</td>
      <td>215620</td>
      <td>2023-03-08</td>
    </tr>
    <tr>
      <th>2027</th>
      <td>12023.64</td>
      <td>12040</td>
      <td>12050</td>
      <td>12040</td>
      <td>12050</td>
      <td>3</td>
      <td>36140</td>
      <td>9415.02</td>
      <td>HK S&amp;P코리아로우볼</td>
      <td>215620</td>
      <td>2023-03-09</td>
    </tr>
  </tbody>
</table>
<p>357042 rows × 11 columns</p>
</div>

## 2. 샤프비율 최대화

### 데이터 변환


```python
%%time
data = etf_price.pivot_table(index='날짜', columns='종목명', values='종가')

# 월말 데이터로 필터링
data= data.resample("M").last()

# 기간 필터링
data = data.loc[:"2023-02-28"]
data
```

    CPU times: user 10.4 s, sys: 42.6 ms, total: 10.5 s
    Wall time: 10.5 s

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
      <th>ACE 미국다우존스리츠(합성 H)</th>
      <th>ACE 밸류대형</th>
      <th>ACE 베트남VN30(합성)</th>
      <th>ACE 삼성그룹동일가중</th>
      <th>ACE 삼성그룹섹터가중</th>
      <th>ACE 인도네시아MSCI(합성)</th>
      <th>ACE 일본Nikkei225(H)</th>
      <th>ACE 중국본토CSI300</th>
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
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6505.0</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6650.0</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7035.0</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7100.0</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6690.0</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <td>76345.0</td>
      <td>7810.0</td>
      <td>16580.0</td>
      <td>15385.0</td>
      <td>15075.0</td>
      <td>11500.0</td>
      <td>18450.0</td>
      <td>24560.0</td>
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
      <td>79030.0</td>
      <td>8365.0</td>
      <td>15785.0</td>
      <td>16080.0</td>
      <td>15595.0</td>
      <td>10785.0</td>
      <td>18735.0</td>
      <td>25360.0</td>
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
      <td>75650.0</td>
      <td>7860.0</td>
      <td>15520.0</td>
      <td>15600.0</td>
      <td>14120.0</td>
      <td>9845.0</td>
      <td>17465.0</td>
      <td>25150.0</td>
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
      <td>81675.0</td>
      <td>8480.0</td>
      <td>16815.0</td>
      <td>16265.0</td>
      <td>15260.0</td>
      <td>10040.0</td>
      <td>18410.0</td>
      <td>27250.0</td>
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
      <td>78085.0</td>
      <td>8420.0</td>
      <td>16400.0</td>
      <td>16025.0</td>
      <td>15150.0</td>
      <td>10485.0</td>
      <td>18525.0</td>
      <td>27450.0</td>
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
<p>158 rows × 146 columns</p>
</div>

행은 날짜로, 열을 종목명으로 구성된 종가 데이터프레임을 만듭니다. 


```python
print("상장일이 다르다보니, 종목별로 누락치가 많이 보입니다.")
print(data.isnull().sum())
```

    상장일이 다르다보니, 종목별로 누락치가 많이 보입니다.
    종목명
    ACE 200                0
    ACE Fn성장소비주도주         67
    ACE 미국다우존스리츠(합성 H)    43
    ACE 밸류대형              17
    ACE 베트남VN30(합성)       78
                          ..
    마이다스 200커버드콜5%OTM     13
    마이티 코스피100            30
    파워 200                25
    파워 고배당저변동성            49
    파워 코스피100             16
    Length: 146, dtype: int64


### 임의 종목 선택하여 수익률 계산하기


```python
# 임의 종목 5개 선택
num=5
tickers = np.random.choice(data.columns, num, replace=False) # replace=False : 중복없이
tickers = ['마이다스 200커버드콜5%OTM', 'TIGER 현대차그룹+펀더멘털', 'KBSTAR 우량업종',
           'KBSTAR V&S셀렉트밸류', 'KODEX 철강']
print(tickers)
temp = data[tickers].dropna()
temp
```

    ['마이다스 200커버드콜5%OTM', 'TIGER 현대차그룹+펀더멘털', 'KBSTAR 우량업종', 'KBSTAR V&S셀렉트밸류', 'KODEX 철강']

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
      <th>마이다스 200커버드콜5%OTM</th>
      <th>TIGER 현대차그룹+펀더멘털</th>
      <th>KBSTAR 우량업종</th>
      <th>KBSTAR V&amp;S셀렉트밸류</th>
      <th>KODEX 철강</th>
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
      <th>2016-02-29</th>
      <td>9870.0</td>
      <td>17720.0</td>
      <td>9360.0</td>
      <td>10290.0</td>
      <td>8495.0</td>
    </tr>
    <tr>
      <th>2016-03-31</th>
      <td>10260.0</td>
      <td>18185.0</td>
      <td>9520.0</td>
      <td>10680.0</td>
      <td>9025.0</td>
    </tr>
    <tr>
      <th>2016-04-30</th>
      <td>10280.0</td>
      <td>18195.0</td>
      <td>9570.0</td>
      <td>10805.0</td>
      <td>9540.0</td>
    </tr>
    <tr>
      <th>2016-05-31</th>
      <td>10170.0</td>
      <td>17030.0</td>
      <td>9420.0</td>
      <td>10490.0</td>
      <td>8300.0</td>
    </tr>
    <tr>
      <th>2016-06-30</th>
      <td>10165.0</td>
      <td>16490.0</td>
      <td>9155.0</td>
      <td>10150.0</td>
      <td>8225.0</td>
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
      <td>11545.0</td>
      <td>20050.0</td>
      <td>9775.0</td>
      <td>11820.0</td>
      <td>7430.0</td>
    </tr>
    <tr>
      <th>2022-11-30</th>
      <td>12000.0</td>
      <td>21205.0</td>
      <td>10675.0</td>
      <td>12815.0</td>
      <td>8475.0</td>
    </tr>
    <tr>
      <th>2022-12-31</th>
      <td>11195.0</td>
      <td>19705.0</td>
      <td>10225.0</td>
      <td>12225.0</td>
      <td>7805.0</td>
    </tr>
    <tr>
      <th>2023-01-31</th>
      <td>12235.0</td>
      <td>21255.0</td>
      <td>10665.0</td>
      <td>13180.0</td>
      <td>8310.0</td>
    </tr>
    <tr>
      <th>2023-02-28</th>
      <td>12240.0</td>
      <td>22240.0</td>
      <td>10555.0</td>
      <td>13350.0</td>
      <td>8735.0</td>
    </tr>
  </tbody>
</table>
<p>85 rows × 5 columns</p>
</div>




```python
fig, (ax,ax2) = plt.subplots(nrows=2, 
                             sharex=True,
                             figsize=(8,6))

cum_rets=((1+temp[13:].pct_change()).cumprod().fillna(1)-1)
cum_rets.plot(ax=ax, color=sns.color_palette("muted", 5))
ax.axhline(0, ls='--', color='black', lw=5, alpha=0.5)
ax.legend(loc='upper left')
ax.set_title('개별 종목 누적 수익률 >> 최소:%d%%, 최대:%d%%' 
            % (cum_rets.iloc[-1,:].min()*100, cum_rets.iloc[-1,:].max()*100) 
            )

ew_cum_rets = ((1+temp[13:].pct_change()).cumprod().fillna(1).sum(axis=1)/num-1)
ew_cum_rets.plot(ax=ax2, color=sns.color_palette("muted", 5))
ax2.axhline(0, ls='--', color='black', lw=5, alpha=0.5)
ax2.set_title('동일비중 투자시 누적 수익률 : %d%%' % (100*ew_cum_rets[-1]))
```


​    ![output_18_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_18_1.png)

​    


### 샤프비율 최대화로 투자비중 선택하여 수익률 계산하기


```python
# 출처 : 퀀트대디
# 샤프비율 최대화 모델 가중치 계산 함수
def get_msr_weights(er, cov):
    # 자산 개수 
    noa = er.shape[0]

    # 초기 가중치
    init_guess = np.repeat(1/noa, noa)

    # 제약조건 및 상하한값
    bounds = ((0.0, 1), ) * noa
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
# 수익률
rets = data[tickers].pct_change().dropna().fillna(0)

# 빈 데이터 프레임 생성
msr_w_df = pd.DataFrame().reindex_like(rets) #rets의 인덱스를 가져오는

# 기대수익률 배열
er = np.array(rets * 12)

pal = sns.color_palette("Spectral", len(tickers))
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
ax.set_title('12개월 롤링 샤프지수 최대화 누적수익률 : %d%%' % (100*port_cum_rets[-1]) )
ax.set_xlabel("Date")
ax.set_ylabel("Weights")
ax2 = ax.twinx()
ax2.plot(port_cum_rets, lw=2, c='blue')#, ls='--')
ax2.axhline(0, color='black', alpha=0.5, ls='--')
ax.set_xlim(datetime.date(2016,11,29), datetime.date(2023,3,1))
```


![output_21_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_21_1.png)    

​    


세로 좌측 : 투자비중, 세로 우측 : 누적수익률 

투자기간 5년동안 종목 비중을 월마다 리밸런싱해주는 것이<br>
개별종목 최대 수익률(27%)보다보다 더 높은 수익률(37%)을 보여주지만, <br>
안정적으로 보이진 않습니다. 

## 3. 종목선택 : 몬테카를로 시뮬레이션

### 임의로 5개 종목 선택 시뮬레이션


```python
port = []
ret = []
ret_12m = []
sharpe = []

########################################
cnt = 1000 #시나리오 개수
num_of_tickers = 5 # 종목개수
rolling_window = 12 # 롤링윈도우

# common_tickers의 종목과 상관계수가 0.7 이상인 ticker들 필터링
threshold = 0.7 #상관계수 필터링 조건
common_tickers = [] # 
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
        top = ret[-1]

    if ret[-1] > top:
        print(port_cum_rets[-1].round(2), ret_12m[-1].round(2), port_sharpe.round(2), tickers) 
        top = ret[-1]
    
```

    5 0.7 146


      0%|                                          | 2/1000 [00:00<01:29, 11.16it/s]
    
    0.43 -0.06 0.38 ['TIGER 단기선진하이일드(합성 H)', 'TIGER 농산물선물Enhanced(H)', 'TIGER 200 생활소비재', 'TIGER 200 건설', 'KODEX 반도체']


      1%|▎                                         | 8/1000 [00:00<01:17, 12.84it/s]
    
    0.54 -0.17 0.33 ['ARIRANG 고배당주', 'KODEX 에너지화학', 'KBSTAR 중국본토대형주CSI100', 'KODEX 골드선물(H)', 'ACE 밸류대형']


      2%|▉                                        | 24/1000 [00:01<01:06, 14.60it/s]
    
    0.61 -0.01 0.32 ['KODEX 반도체', 'KBSTAR 5대그룹주', 'TIGER 200 중공업', 'ARIRANG 코스피50', '마이티 코스피100']


      5%|█▉                                       | 46/1000 [00:03<00:58, 16.26it/s]
    
    0.61 0.1 0.5 ['KODEX 코스피100', 'ACE 인도네시아MSCI(합성)', 'KODEX 미국S&P500산업재(합성)', 'KBSTAR 모멘텀밸류', 'ACE 코스닥(합성)']


      9%|███▌                                     | 86/1000 [00:06<00:58, 15.58it/s]
    
    1.04 0.1 0.76 ['KODEX 삼성그룹', 'TREX 200', 'TIGER S&P글로벌헬스케어(합성)', '파워 코스피100', 'ACE 삼성그룹섹터가중']


     14%|█████▌                                  | 138/1000 [00:09<00:55, 15.63it/s]
    
    1.13 0.17 0.77 ['ARIRANG 코스피50', 'ARIRANG 코스피', 'KODEX 일본TOPIX100', 'KODEX 코스피100', 'TIGER 200 산업재']


     21%|████████▌                               | 214/1000 [00:14<01:00, 13.03it/s]
    
    1.73 0.04 0.72 ['KODEX 반도체', 'TIGER 200 IT', 'KOSEF 미국달러선물', 'KOSEF 고배당', 'TIGER 미국나스닥100']


    100%|███████████████████████████████████████| 1000/1000 [01:10<00:00, 14.10it/s]


### 수익률 결과 히스토그램


```python
results = pd.DataFrame({"port":port,"sharpe":sharpe,"ret":ret,"ret_12m":ret_12m})
results['ret'].plot.hist(bins=100)
```

![output_27_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_27_1.png)


​    

```python
results = pd.DataFrame({"port":port,"ret":ret,"ret_12m":ret_12m,"sharpe":sharpe})
cond = results['ret'] > 1
results = results[cond].sort_values(by='sharpe', ascending=False)
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
      <th>ret</th>
      <th>ret_12m</th>
      <th>sharpe</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>949</th>
      <td>[TIGER LG그룹+펀더멘털, ARIRANG 미국다우존스고배당주(합성 H), KO...</td>
      <td>1.083995</td>
      <td>0.105965</td>
      <td>1.032625</td>
    </tr>
    <tr>
      <th>325</th>
      <td>[TIGER 200 IT, ACE 밸류대형, KODEX 200, KODEX 미국달러...</td>
      <td>1.041779</td>
      <td>0.091459</td>
      <td>0.838990</td>
    </tr>
    <tr>
      <th>145</th>
      <td>[KODEX 배당성장, TIGER 미국나스닥100, TIGER KTOP30, KOS...</td>
      <td>1.094986</td>
      <td>0.068243</td>
      <td>0.830157</td>
    </tr>
    <tr>
      <th>134</th>
      <td>[ARIRANG 코스피50, ARIRANG 코스피, KODEX 일본TOPIX100,...</td>
      <td>1.134787</td>
      <td>0.170098</td>
      <td>0.769744</td>
    </tr>
    <tr>
      <th>507</th>
      <td>[ACE 코스닥(합성), KOSEF 미국달러선물, KODEX 반도체, TIGER 유...</td>
      <td>1.004255</td>
      <td>0.045385</td>
      <td>0.768535</td>
    </tr>
    <tr>
      <th>84</th>
      <td>[KODEX 삼성그룹, TREX 200, TIGER S&amp;P글로벌헬스케어(합성), 파...</td>
      <td>1.042702</td>
      <td>0.095163</td>
      <td>0.755316</td>
    </tr>
    <tr>
      <th>490</th>
      <td>[KODEX 미국S&amp;P500산업재(합성), ACE 중국본토CSI300, TIGER ...</td>
      <td>1.131652</td>
      <td>0.202374</td>
      <td>0.752559</td>
    </tr>
    <tr>
      <th>220</th>
      <td>[TIGER 200 중공업, TIGER 200 금융, TIGER 단기선진하이일드(합...</td>
      <td>1.275585</td>
      <td>0.376153</td>
      <td>0.725623</td>
    </tr>
    <tr>
      <th>213</th>
      <td>[KODEX 반도체, TIGER 200 IT, KOSEF 미국달러선물, KOSEF ...</td>
      <td>1.729249</td>
      <td>0.035973</td>
      <td>0.720899</td>
    </tr>
    <tr>
      <th>174</th>
      <td>[KOSEF 블루칩, KODEX 코스닥150, TIGER 미국나스닥100, TIGE...</td>
      <td>1.109075</td>
      <td>-0.305920</td>
      <td>0.644526</td>
    </tr>
  </tbody>
</table>
</div>



### 상위 3개의 결과 시각화


```python
idxs = results[:3].index.tolist()
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

    949 ['TIGER LG그룹+펀더멘털', 'ARIRANG 미국다우존스고배당주(합성 H)', 'KODEX 미국달러선물', 'TIGER 단기선진하이일드(합성 H)', 'TIGER 200 IT']
    325 ['TIGER 200 IT', 'ACE 밸류대형', 'KODEX 200', 'KODEX 미국달러선물', 'KODEX 바이오']
    145 ['KODEX 배당성장', 'TIGER 미국나스닥100', 'TIGER KTOP30', 'KOSEF 미국달러선물', 'TIGER 코스닥150']

![output_30_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_30_1.png)

![output_30_2](/images/2023-03-12-etf_portfolio_by_using_GA/output_30_2.png)

![output_30_3](/images/2023-03-12-etf_portfolio_by_using_GA/output_30_3.png)



​    


1000개의 종목에 대해 시뮬레이션해도 결과가 신통치 않음 

모든 경우의 수를 다 해본다면 좋겠지만, 해를 개선해 나가기 위한 방법 고민됨

- 종목 몇개를 고정하고, 상관계수가 높은 종목들을 필터링해주기 
- 유전알고리즘으로 종목 선택 개선하기 

## 4. 종목선택 : 유전알고리즘


```python
import pygad
```


```python
##### hypter parameter
tic = time.time()
# 유전자집합
#ticker_list = data.columns.tolist()
num_select_stocks = 5
pal = sns.color_palette("Spectral", num_select_stocks)
print("유전자집합 :", len(ticker_list))
print("유전자개수 :", num_select_stocks)
print("common_tickers :", common_tickers)
print('-'*100)

def fitness_func(solution, solution_idx):
    
    #global tickers
    tickers = [ticker_list[i] for i in solution]+common_tickers 
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
    # print((1+port_rets.sum(axis=1)).cumprod()[-1])
    # print('-'*30)
    return (1+port_rets.sum(axis=1)).cumprod()[-1]-1 # 최대화


fitness_function = fitness_func

num_generations = 15 # Number of generations.
num_parents_mating = 5 # Number of solutions to be selected as parents in the mating pool.

# To prepare the initial population, there are 2 ways:
# 1) Prepare it yourself and pass it to the initial_population parameter. This way is useful when the user wants to start the genetic algorithm with a custom initial population.
# 2) Assign valid integer values to the sol_per_pop and num_genes parameters. If the initial_population parameter exists, then the sol_per_pop and num_genes parameters are useless.
sol_per_pop = 30 # Number of solutions in the population.
num_genes = num_select_stocks-len(common_tickers) #5

init_range_low = 0
init_range_high = data[ticker_list].shape[1]-1

parent_selection_type = "sss" # Type of parent selection. sss (for steady-state selection), rws (for roulette wheel selection), sus (for stochastic universal selection), rank (for rank selection), random (for random selection), and tournament (for tournament selection)
keep_parents = 1 # Number of parents to keep in the next population. -1 means keep all parents and 0 means keep nothing.

crossover_type = "single_point" # Type of the crossover operator.

# Parameters of the mutation operation.
mutation_type = 'random' # Type of the mutation operator.
mutation_percent_genes = 80 # Percentage of genes to mutate. This parameter has no action if the parameter mutation_num_genes exists or when mutation_type is None.

last_fitness = 0
def callback_generation(ga_instance):
    global last_fitness
    global tickers 
    tickers = [ticker_list[i] for i in ga_instance.best_solution()[0]]+common_tickers
    print("Generation = [{generation}]".format(generation=ga_instance.generations_completed), "\tFitness = {fitness}".format(fitness=ga_instance.best_solution()[1].round(4)), "\t Best tickers = {tickers}".format(tickers = [ticker_list[i] for i in ga_instance.best_solution()[0]]+common_tickers))
#    print("Best tickers = {tickers}".format(tickers = [ticker_list[i] for i in ga_instance.best_solution()[0]]+common_tickers))
    print('-'*100)
    #print("Change     = {change}".format(change=ga_instance.best_solution()[1] - last_fitness))
    last_fitness = ga_instance.best_solution()[1]
    
               
initial_population = [] # 없음

# Creating an instance of the GA class inside the ga module. Some parameters are initialized within the constructor.
if len(initial_population) > 0 :
    print('initial_population')
    ga_instance = pygad.GA(initial_population=initial_population, # 초기 집단 설정
                           num_generations=num_generations,
                           num_parents_mating=num_parents_mating, 
                           fitness_func=fitness_function,
                           sol_per_pop=sol_per_pop, 
                           num_genes=num_genes,
                           init_range_low=init_range_low,
                           init_range_high=init_range_high,
                           parent_selection_type=parent_selection_type,
                           keep_parents=keep_parents,
                           crossover_type=crossover_type,
                           mutation_type=mutation_type,
                           mutation_percent_genes=mutation_percent_genes,
                           on_generation=callback_generation,
                           gene_type=int, #좀옥을 선택하는 것으니 int로
                          save_best_solutions=True)
else:
    ga_instance = pygad.GA(
                       num_generations=num_generations,
                       num_parents_mating=num_parents_mating, 
                       fitness_func=fitness_function,
                       sol_per_pop=sol_per_pop, 
                       num_genes=num_genes,
                       init_range_low=init_range_low,
                       init_range_high=init_range_high,
                       parent_selection_type=parent_selection_type,
                       keep_parents=keep_parents,
                       crossover_type=crossover_type,
                       mutation_type=mutation_type,
                       mutation_percent_genes=mutation_percent_genes,
                       on_generation=callback_generation,
                       gene_type=int, #좀옥을 선택하는 것으니 int로
                      save_best_solutions=True) 

# Running the GA to optimize the parameters of the function.
ga_instance.run()

# After the generations complete, some plots are showed that summarize the how the outputs/fitenss values evolve over generations.
ga_instance.plot_result()

# Returning the details of the best solution.
solution, solution_fitness, solution_idx = ga_instance.best_solution()
print("Parameters of the best solution : {solution}".format(solution=solution))
print("Fitness value of the best solution = {solution_fitness}".format(solution_fitness=solution_fitness))
print("Index of the best solution : {solution_idx}".format(solution_idx=solution_idx))

# prediction = numpy.sum(numpy.array(function_inputs)*solution)
# print("Predicted output based on the best solution : {prediction}".format(prediction=prediction))

if ga_instance.best_solution_generation != -1:
    print("Best fitness value reached after {best_solution_generation} generations.".format(best_solution_generation=ga_instance.best_solution_generation))

# Saving the GA instance.
filename = 'genetic' # The filename to which the instance is saved. The name is without extension.
ga_instance.save(filename=filename)

# Loading the saved GA instance.
#loaded_ga_instance = pygad.load(filename=filename)
#loaded_ga_instance.run()
#loaded_ga_instance.plot_result()
#solution, solution_fitness, solution_idx = loaded_ga_instance.best_solution()
print(time.time()-tic,"seconds")

solution
```

    유전자집합 : 146
    유전자개수 : 5
    common_tickers : []
    ----------------------------------------------------------------------------------------------------
    Generation = [1] 	Fitness = 0.7765 	 Best tickers = ['KBSTAR 5대그룹주', 'TIGER 코스닥150바이오테크', 'KODEX 미국달러선물', 'KODEX 200', 'ACE 미국다우존스리츠(합성 H)']
    ----------------------------------------------------------------------------------------------------
    Generation = [2] 	Fitness = 0.8345 	 Best tickers = ['ACE Fn성장소비주도주', 'TIGER 200 에너지화학', 'KODEX 미국달러선물', 'KODEX 200', 'ACE Fn성장소비주도주']
    ----------------------------------------------------------------------------------------------------
    Generation = [3] 	Fitness = 1.0922 	 Best tickers = ['KODEX 건설', 'TIGER 우량가치', '마이티 코스피100', 'KODEX 미국S&P500에너지(합성)', 'ACE 미국다우존스리츠(합성 H)']
    ----------------------------------------------------------------------------------------------------
    Generation = [4] 	Fitness = 1.7465 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', '마이다스 200커버드콜5%OTM', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [5] 	Fitness = 1.7465 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', '마이다스 200커버드콜5%OTM', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [6] 	Fitness = 1.9916 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 펀더멘탈 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [7] 	Fitness = 1.9916 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 펀더멘탈 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [8] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [9] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [10] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [11] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [12] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [13] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [14] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------
    Generation = [15] 	Fitness = 2.2685 	 Best tickers = ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    ----------------------------------------------------------------------------------------------------

![output_34_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_34_1.png)



​    


    Parameters of the best solution : [ 23 132 139  85  78]
    Fitness value of the best solution = 2.268480819777148
    Index of the best solution : 0
    Best fitness value reached after 8 generations.
    246.91905093193054 seconds

누적수익률 227%라는 결과를 4분 만에 도달했지만, 지역최적해일 가능성이 높다. 

## 5. 결과 시각화


```python
# solution = [61 ,76, 78 ,81 ,68]
tickers = [ticker_list[i] for i in solution] + common_tickers
#tickers = ['KOSEF 미국달러선물', 'TIGER 200IT레버리지', 'TIGER 코스닥150 레버리지', 'KODEX 미국S&P500에너지(합성)', 'KODEX 미국S&P500산업재(합성)']

print(tickers)
# 수익률
rets = data[tickers].pct_change().dropna().fillna(0)
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
port_sharpe = port_rets.sum(axis=1).mean()*12 / (port_rets.sum(axis=1).std()*np.sqrt(12))
# 시각화
fig, ax = plt.subplots(figsize=(8,5))
ax.stackplot(msr_w_df.index, msr_w_df.T, labels=msr_w_df.columns, colors=pal)
ax.legend(loc='upper left')
ax.set_title("MSR Weights")
ax.set_xlabel("Date")
ax.set_ylabel("Weights")
ax2 = ax.twinx()
ax2.plot(port_cum_rets, lw=2, c='blue')#, ls='--')

print(((1+port_rets.sum(axis=1)).cumprod()[-1]-1).round(2), ((1+port_rets.sum(axis=1)[-12:]).cumprod()[-1]-1).round(2), port_sharpe.round(2))
```

    ['KBSTAR 200', 'TIGER 코스닥150', 'TREX 200', 'TIGER 200 IT', 'KOSEF 미국달러선물']
    2.27 0.22 1.13




![output_36_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_36_1.png)
    



```python
# cluster map
sns.clustermap(data[tickers].corr(),
               annot = True,      # 실제 값 화면에 나타내기
               cmap = 'RdYlBu_r',  # Red, Yellow, Blue 색상으로 표시
               vmin = -1, vmax = 1, #컬러차트 -1 ~ 1 범위로 표시
               figsize=(6,6) )
```


![output_37_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_37_1.png)
    



```python
# 그림 사이즈 지정
fig, ax = plt.subplots( figsize=(5,5) )

# 삼각형 마스크를 만든다(위 쪽 삼각형에 True, 아래 삼각형에 False)
mask = np.zeros_like(data[tickers].corr(), dtype=np.bool)
mask[np.triu_indices_from(mask)] = True

# 히트맵을 그린다
sns.heatmap(data[tickers].corr(), 
            cmap = 'RdYlBu_r', 
            annot = True,   # 실제 값을 표시한다
            mask=mask,      # 표시하지 않을 마스크 부분을 지정한다
            linewidths=.5,  # 경계면 실선으로 구분하기
            cbar_kws={"shrink": .5},# 컬러바 크기 절반으로 줄이기
            vmin = -1,vmax = 1   # 컬러바 범위 -1 ~ 1
           )  
plt.show()
```


![output_38_0](/images/2023-03-12-etf_portfolio_by_using_GA/output_38_0.png)
    


달러를 제외하고는 상관계수가 모두 빨간색으로 좋은 포트폴리오라고 보기엔 어려울 것 같습니다.


```python
# 시뮬레이션 투자비중, 수익률
periods=36
fig, ax = plt.subplots(figsize=(10,5+periods*0.2))
weights = msr_w_df.shift().iloc[-periods:].reset_index()
returns = pd.DataFrame(port_rets.sum(axis=1), columns=['수익률']).reset_index()
temp = pd.merge(weights, returns, on='날짜', how='inner').set_index("날짜")
sns.heatmap(temp.round(2), cmap='RdBu_r', vmin=-0.3, vmax=0.3, annot=True, ax=ax)
ax.set_yticklabels(weights.날짜.apply(lambda x:x.strftime("%Y-%m")))
;
```


![output_40_1](/images/2023-03-12-etf_portfolio_by_using_GA/output_40_1.png)    

   

```python
# 다음 추천 투자 비중 
msr_w_df.iloc[-1].round(2)
```


    종목명
    KBSTAR 200      0.00
    TIGER 코스닥150    0.00
    TREX 200        0.00
    TIGER 200 IT    0.78
    KOSEF 미국달러선물    0.22
    Name: 2023-02-28 00:00:00, dtype: float64

