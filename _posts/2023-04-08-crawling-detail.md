---

layout: single
# classes: wide
title : "주식 관련 데이터 크롤링 하기"
categories: portfolio
tag: [python, stock, etf, crawling]
toc: true
toc_label: "순서"
toc_icon: "cog"
author_profile: false
sidebar:
    nav: "docs"
Typora-root-url: ../

---

# 주식 관련 데이터 크롤링

## ❒ 섹터, 종목 (kor_ticker, kor_sector)

- "kor_ticker.pickle" : (krx) 코스피/코스닥 기준일의 종목코드, 종가, 시가총액, EPS, BPS, 업종명(소분류)
- "kor_sector.pickle" : (wiseindex) 코스피/코스닥 업종명

### 날짜 크롤링


```python
import requests as rq
from bs4 import BeautifulSoup
import re
```


```python
#네이버증권-증시자금동향 #deposit : 예탁금
url = 'https://finance.naver.com/sise/sise_deposit.nhn' 
data = rq.get(url)
data # Response[200] : 데이터를 제대로 받아왔음d
```




    <Response [200]>




```python
data_html = BeautifulSoup(data.content)
parse_day = data_html.select_one('span.tah').text #copy element
parse_day
```




    '\xa0\xa0|\xa0\xa02023.04.05'




```python
biz_day = re.findall('[0-9]+', parse_day) #['2023','04','05']
biz_day = "".join(biz_day)
biz_day
```




    '20230405'



(참고) <br>
re.findall() 함수는 첫 번째 인수로 지정된 정규표현식 패턴('[0-9]+' 이 경우)을 두 번째 인수인 문자열(parse_day 이 경우)에서 모두 찾아서, 찾은 모든 비중첩 매치(match)를 리스트로 반환합니다.<br>
[0-9] : 0부터 9까지의 숫자 중 하나를 의미합니다.
+ : 앞의 패턴([0-9])이 하나 이상 반복되는 것을 의미합니다.
따라서, [0-9]+ 패턴은 숫자(0~9)가 하나 이상 연속으로 나오는 문자열과 매치됩니다. 예를 들어, "123", "45", "6789" 등이 매치됩니다.

반대로, "abc", "12a", "23b4" 등은 숫자가 하나 이상 연속되는 문자열이 아니므로 매치되지 않습니다.

### ❍ 업종분류 현황 & 개별종목 지표(kor_ticker) 

#### 업종분류


```python
# data.krx.co.kr > 주식 > 세부안내 > 업종분류 현황
# 개발자 도구 열고, csv 다운로드 누르면 generate.cmd, download.cmd 파일이 생김
# generate.cmd > Headers > Request URL : 
```


```python
import requests as rq
from io import BytesIO
import pandas as pd
```


```python
def crawling_sector(market='STK'): 
    # generate.cmd > Headers > Request URL
    gen_otp_url = 'http://data.krx.co.kr/comm/fileDn/GenerateOTP/generate.cmd' 

    # generate.cmd > Payload
    gen_otp_stk = {
        'mktId' : market, # KOSPI : "STK", KOSDAQ : "KSQ"
        'trdDd' : biz_day,
        'money' : '1', 
        'csvxls_isNo' : 'false', 
        'name' : 'fileDown', 
        'url' : 'dbms/MDC/STAT/standard/MDCSTAT03901'
                  }

    # generate.cmd > Headers > Referer (로봇이 아님을..)
    headers = {'Referer' : 'http://data.krx.co.kr/contents/MDC/MDI/mdiLoader'}

    otp_stk = rq.post(gen_otp_url, gen_otp_stk, headers=headers).text

    # download.cmd > headers > Request URL
    down_url = 'http://data.krx.co.kr/comm/fileDn/download_csv/download.cmd'

    down_sector_stk = rq.post(down_url, {'code':otp_stk}, headers=headers)

    df= pd.read_csv(BytesIO(down_sector_stk.content), encoding='EUC-KR') # Binary String 형태로 변경
    return df

sector_stk = crawling_sector(market='STK') #KOSPI
sector_ksq = crawling_sector(market='KSQ') #KOSDAQ
krx_sector = pd.concat([sector_stk, sector_ksq]).reset_index(drop=True)
krx_sector['종목명'] = krx_sector['종목명'].str.strip() # 공백 제거
krx_sector['기준일'] = biz_day
krx_sector
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
      <th>시장구분</th>
      <th>업종명</th>
      <th>종가</th>
      <th>대비</th>
      <th>등락률</th>
      <th>시가총액</th>
      <th>기준일</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>095570</td>
      <td>AJ네트웍스</td>
      <td>KOSPI</td>
      <td>서비스업</td>
      <td>4670</td>
      <td>-65</td>
      <td>-1.37</td>
      <td>218660117650</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>1</th>
      <td>006840</td>
      <td>AK홀딩스</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>18450</td>
      <td>120</td>
      <td>0.65</td>
      <td>244417500450</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>2</th>
      <td>027410</td>
      <td>BGF</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>4480</td>
      <td>0</td>
      <td>0.00</td>
      <td>428811223680</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>3</th>
      <td>282330</td>
      <td>BGF리테일</td>
      <td>KOSPI</td>
      <td>유통업</td>
      <td>189300</td>
      <td>1700</td>
      <td>0.91</td>
      <td>3271843405800</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>4</th>
      <td>138930</td>
      <td>BNK금융지주</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>6510</td>
      <td>20</td>
      <td>0.31</td>
      <td>2121838451460</td>
      <td>20230405</td>
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
    </tr>
    <tr>
      <th>2574</th>
      <td>024060</td>
      <td>흥구석유</td>
      <td>KOSDAQ</td>
      <td>유통</td>
      <td>5890</td>
      <td>-260</td>
      <td>-4.23</td>
      <td>88350000000</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>2575</th>
      <td>010240</td>
      <td>흥국</td>
      <td>KOSDAQ</td>
      <td>기계·장비</td>
      <td>6260</td>
      <td>-40</td>
      <td>-0.63</td>
      <td>77140076960</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>2576</th>
      <td>189980</td>
      <td>흥국에프엔비</td>
      <td>KOSDAQ</td>
      <td>음식료·담배</td>
      <td>2695</td>
      <td>-60</td>
      <td>-2.18</td>
      <td>108171443765</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>2577</th>
      <td>037440</td>
      <td>희림</td>
      <td>KOSDAQ</td>
      <td>기타서비스</td>
      <td>9430</td>
      <td>-20</td>
      <td>-0.21</td>
      <td>131288939250</td>
      <td>20230405</td>
    </tr>
    <tr>
      <th>2578</th>
      <td>238490</td>
      <td>힘스</td>
      <td>KOSDAQ</td>
      <td>반도체</td>
      <td>7220</td>
      <td>-60</td>
      <td>-0.82</td>
      <td>81674343920</td>
      <td>20230405</td>
    </tr>
  </tbody>
</table>
<p>2579 rows × 9 columns</p>
</div>



bizday를 기준으로 코스피와 코스닥의 업종명을 확인할 수 있습니다.

#### 개별종목


```python
# 주식 > 세부안내 > PER/PBR/배당수익률(개별종목)

# generate.cmd > Headers > Request URL
gen_otp_url = 'http://data.krx.co.kr/comm/fileDn/GenerateOTP/generate.cmd' 

# generate.cmd > Payload
gen_otp_data = {
    'searchType':'1',
    'mktId' : 'ALL',
    'trdDd' : biz_day, #데이터가 안나올 경우 bizday 문제
    'csvxls_isNo' : 'false', 
    'name' : 'fileDown', 
    'url' : 'dbms/MDC/STAT/standard/MDCSTAT03501'
              }

# generate.cmd > Headers > Referer (로봇이 아님을..)
headers = {'Referer' : 'http://data.krx.co.kr/contents/MDC/MDI/mdiLoader'}
otp = rq.post(gen_otp_url, gen_otp_data, headers=headers).text

# download.cmd > headers > Request URL
down_url = 'http://data.krx.co.kr/comm/fileDn/download_csv/download.cmd'
krx_ind = rq.post(down_url, {'code': otp}, headers=headers)

krx_ind = pd.read_csv(BytesIO(krx_ind.content), encoding='EUC-KR') # Binary String 형태로 변경
krx_ind['종목명'] = krx_ind['종목명'].str.strip()
krx_ind['기준일'] = biz_day
krx_ind
```

#### sysmmetric difference
집합론에서, 두 집합의 대칭차 또는 대칭차집합은 둘 중 한 집합에는 속하지만 둘 모두에는 속하지는 않는 원소들의 집합이다.


```python
set(krx_sector['종목명']).symmetric_difference(set(krx_ind['종목명']))
```




    {'ESR켄달스퀘어리츠',
     'GRT',
     'JTC',
     'KB스타리츠',
     'NH올원리츠',
     'NH프라임리츠',
     'SBI핀테크솔루션즈',
     'SK리츠',
     '골든센츄리',
     '글로벌에스엠',
     '네오이뮨텍',
     '디앤디플랫폼리츠',
     '로스웰',
     '롯데리츠',
     '마스턴프리미어리츠',
     '맥쿼리인프라',
     '맵스리얼티1',
     '모두투어리츠',
     '미래에셋글로벌리츠',
     '미래에셋맵스리츠',
     '미투젠',
     '바다로19호',
     '소마젠',
     '신한서부티엔디리츠',
     '신한알파리츠',
     '씨케이에이치',
     '애머릿지',
     '에이리츠',
     '엑세스바이오',
     '엘브이엠씨홀딩스',
     '오가닉티코스메틱',
     '윙입푸드',
     '이리츠코크렙',
     '이스트아시아홀딩스',
     '이지스레지던스리츠',
     '이지스밸류리츠',
     '잉글우드랩',
     '제이알글로벌리츠',
     '컬러레이',
     '케이탑리츠',
     '코람코더원리츠',
     '코람코에너지리츠',
     '코오롱티슈진',
     '크리스탈신소재',
     '프레스티지바이오파마',
     '한국ANKOR유전',
     '한국패러랠',
     '한화리츠',
     '헝셩그룹'}



#### 데이터 머징


```python
# 섹터와 지표 데이터 간 공통된 칼럼
krx_sector.columns.intersection(krx_ind.columns).tolist()
```




    ['종목코드', '종목명', '종가', '대비', '등락률', '기준일']




```python
kor_ticker = pd.merge(krx_sector,
                      krx_ind,
                      on= krx_sector.columns.intersection(krx_ind.columns).tolist(),
                      how='outer')
kor_ticker
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
      <th>시장구분</th>
      <th>업종명</th>
      <th>종가</th>
      <th>대비</th>
      <th>등락률</th>
      <th>시가총액</th>
      <th>기준일</th>
      <th>EPS</th>
      <th>PER</th>
      <th>선행 EPS</th>
      <th>선행 PER</th>
      <th>BPS</th>
      <th>PBR</th>
      <th>주당배당금</th>
      <th>배당수익률</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>095570</td>
      <td>AJ네트웍스</td>
      <td>KOSPI</td>
      <td>서비스업</td>
      <td>4670</td>
      <td>-65</td>
      <td>-1.37</td>
      <td>218660117650</td>
      <td>20230405</td>
      <td>1707.0</td>
      <td>2.74</td>
      <td>619.0</td>
      <td>7.55</td>
      <td>8075.0</td>
      <td>0.58</td>
      <td>270.0</td>
      <td>5.78</td>
    </tr>
    <tr>
      <th>1</th>
      <td>006840</td>
      <td>AK홀딩스</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>18450</td>
      <td>120</td>
      <td>0.65</td>
      <td>244417500450</td>
      <td>20230405</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>45961.0</td>
      <td>0.40</td>
      <td>200.0</td>
      <td>1.08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>027410</td>
      <td>BGF</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>4480</td>
      <td>0</td>
      <td>0.00</td>
      <td>428811223680</td>
      <td>20230405</td>
      <td>684.0</td>
      <td>6.55</td>
      <td>700.0</td>
      <td>6.40</td>
      <td>16393.0</td>
      <td>0.27</td>
      <td>110.0</td>
      <td>2.46</td>
    </tr>
    <tr>
      <th>3</th>
      <td>282330</td>
      <td>BGF리테일</td>
      <td>KOSPI</td>
      <td>유통업</td>
      <td>189300</td>
      <td>1700</td>
      <td>0.91</td>
      <td>3271843405800</td>
      <td>20230405</td>
      <td>8547.0</td>
      <td>22.15</td>
      <td>12749.0</td>
      <td>14.85</td>
      <td>46849.0</td>
      <td>4.04</td>
      <td>3000.0</td>
      <td>1.58</td>
    </tr>
    <tr>
      <th>4</th>
      <td>138930</td>
      <td>BNK금융지주</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>6510</td>
      <td>20</td>
      <td>0.31</td>
      <td>2121838451460</td>
      <td>20230405</td>
      <td>2341.0</td>
      <td>2.78</td>
      <td>2637.0</td>
      <td>2.47</td>
      <td>28745.0</td>
      <td>0.23</td>
      <td>560.0</td>
      <td>8.60</td>
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
    </tr>
    <tr>
      <th>2574</th>
      <td>024060</td>
      <td>흥구석유</td>
      <td>KOSDAQ</td>
      <td>유통</td>
      <td>5890</td>
      <td>-260</td>
      <td>-4.23</td>
      <td>88350000000</td>
      <td>20230405</td>
      <td>96.0</td>
      <td>61.35</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5426.0</td>
      <td>1.09</td>
      <td>100.0</td>
      <td>1.70</td>
    </tr>
    <tr>
      <th>2575</th>
      <td>010240</td>
      <td>흥국</td>
      <td>KOSDAQ</td>
      <td>기계·장비</td>
      <td>6260</td>
      <td>-40</td>
      <td>-0.63</td>
      <td>77140076960</td>
      <td>20230405</td>
      <td>1027.0</td>
      <td>6.10</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7439.0</td>
      <td>0.84</td>
      <td>200.0</td>
      <td>3.19</td>
    </tr>
    <tr>
      <th>2576</th>
      <td>189980</td>
      <td>흥국에프엔비</td>
      <td>KOSDAQ</td>
      <td>음식료·담배</td>
      <td>2695</td>
      <td>-60</td>
      <td>-2.18</td>
      <td>108171443765</td>
      <td>20230405</td>
      <td>164.0</td>
      <td>16.43</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2004.0</td>
      <td>1.34</td>
      <td>30.0</td>
      <td>1.11</td>
    </tr>
    <tr>
      <th>2577</th>
      <td>037440</td>
      <td>희림</td>
      <td>KOSDAQ</td>
      <td>기타서비스</td>
      <td>9430</td>
      <td>-20</td>
      <td>-0.21</td>
      <td>131288939250</td>
      <td>20230405</td>
      <td>490.0</td>
      <td>19.24</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4769.0</td>
      <td>1.98</td>
      <td>150.0</td>
      <td>1.59</td>
    </tr>
    <tr>
      <th>2578</th>
      <td>238490</td>
      <td>힘스</td>
      <td>KOSDAQ</td>
      <td>반도체</td>
      <td>7220</td>
      <td>-60</td>
      <td>-0.82</td>
      <td>81674343920</td>
      <td>20230405</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6598.0</td>
      <td>1.09</td>
      <td>100.0</td>
      <td>1.39</td>
    </tr>
  </tbody>
</table>
<p>2579 rows × 17 columns</p>
</div>



#### 특정 종목들 찾기


```python
# 스팩 / 제x호를 포함하는 종목
cond = kor_ticker['종목명'].str.contains('스팩|제[0-9]+호')
kor_ticker.loc[cond, '종목명']
```




    577      엔에이치스팩19호
    960      DB금융스팩10호
    961       DB금융스팩8호
    962       DB금융스팩9호
    984     IBKS제17호스팩
               ...    
    2485       하이제7호스팩
    2504      한국제10호스팩
    2505      한국제11호스팩
    2531    한화플러스제2호스팩
    2532    한화플러스제3호스팩
    Name: 종목명, Length: 72, dtype: object




```python
# 우선주 찾기 : 우리나라 ticker는 끝자리가 0이면 보통주, 0이 아니면 우선주
cond = kor_ticker['종목코드'].str[-1:] != '0'
kor_ticker.loc[cond, '종목명']
```




    6           BYC우
    9       CJ4우(전환)
    12       CJ씨푸드1우
    13           CJ우
    15      CJ제일제당 우
              ...   
    944        흥국화재우
    1203      대호특수강우
    1310     루트로닉3우C
    1563       소프트센우
    2534      해성산업1우
    Name: 종목명, Length: 123, dtype: object




```python
# 리츠 : 리츠로 끝나는 종목명
cond = kor_ticker['종목명'].str.endswith('리츠')
kor_ticker.loc[cond, '종목명']
```




    35     ESR켄달스퀘어리츠
    64         KB스타리츠
    112        NH올원리츠
    115       NH프라임리츠
    143          SK리츠
    344      디앤디플랫폼리츠
    350          롯데리츠
    364     마스턴프리미어리츠
    375        모두투어리츠
    383     미래에셋글로벌리츠
    384      미래에셋맵스리츠
    534     신한서부티엔디리츠
    535        신한알파리츠
    570          에이리츠
    641     이지스레지던스리츠
    642       이지스밸류리츠
    669      제이알글로벌리츠
    717         케이탑리츠
    718       코람코더원리츠
    719      코람코에너지리츠
    871          한화리츠
    Name: 종목명, dtype: object



#### 종목 구분 및 저장


```python
import numpy as np

# 한쪽만 있는 데이터
diff = list(set(krx_sector['종목명']).symmetric_difference(set(krx_ind['종목명'])))

# 조건
kor_ticker['종목구분'] = np.where(kor_ticker['종목명'].str.contains('스팩|제[0-9]+호'), '스팩',
                              np.where(kor_ticker['종목코드'].str[-1:] != '0', '우선주',
                                       np.where(kor_ticker['종목명'].str.endswith('리츠'), '리츠',
                                                np.where(kor_ticker['종목명'].isin(diff), '기타',
                                                         '보통주'))))
kor_ticker = kor_ticker.reset_index(drop=True)
kor_ticker.columns = kor_ticker.columns.str.replace(" ", "") #공백제거
kor_ticker = kor_ticker[['종목코드','종목명','시장구분','업종명','종가','시가총액',
                         '기준일','EPS','선행EPS','BPS','주당배당금','종목구분']] # 열 선택
kor_ticker = kor_ticker.replace({np.nan : None}) # nan값을 None으로 변경 : SQL에서는 nan값을 저장할 수 없음
kor_ticker               
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
      <th>시장구분</th>
      <th>업종명</th>
      <th>종가</th>
      <th>시가총액</th>
      <th>기준일</th>
      <th>EPS</th>
      <th>선행EPS</th>
      <th>BPS</th>
      <th>주당배당금</th>
      <th>종목구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>095570</td>
      <td>AJ네트웍스</td>
      <td>KOSPI</td>
      <td>서비스업</td>
      <td>4670</td>
      <td>218660117650</td>
      <td>20230405</td>
      <td>1707.0</td>
      <td>619.0</td>
      <td>8075.0</td>
      <td>270.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>1</th>
      <td>006840</td>
      <td>AK홀딩스</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>18450</td>
      <td>244417500450</td>
      <td>20230405</td>
      <td>None</td>
      <td>None</td>
      <td>45961.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2</th>
      <td>027410</td>
      <td>BGF</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>4480</td>
      <td>428811223680</td>
      <td>20230405</td>
      <td>684.0</td>
      <td>700.0</td>
      <td>16393.0</td>
      <td>110.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>3</th>
      <td>282330</td>
      <td>BGF리테일</td>
      <td>KOSPI</td>
      <td>유통업</td>
      <td>189300</td>
      <td>3271843405800</td>
      <td>20230405</td>
      <td>8547.0</td>
      <td>12749.0</td>
      <td>46849.0</td>
      <td>3000.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>4</th>
      <td>138930</td>
      <td>BNK금융지주</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>6510</td>
      <td>2121838451460</td>
      <td>20230405</td>
      <td>2341.0</td>
      <td>2637.0</td>
      <td>28745.0</td>
      <td>560.0</td>
      <td>보통주</td>
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
    </tr>
    <tr>
      <th>2574</th>
      <td>024060</td>
      <td>흥구석유</td>
      <td>KOSDAQ</td>
      <td>유통</td>
      <td>5890</td>
      <td>88350000000</td>
      <td>20230405</td>
      <td>96.0</td>
      <td>None</td>
      <td>5426.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2575</th>
      <td>010240</td>
      <td>흥국</td>
      <td>KOSDAQ</td>
      <td>기계·장비</td>
      <td>6260</td>
      <td>77140076960</td>
      <td>20230405</td>
      <td>1027.0</td>
      <td>None</td>
      <td>7439.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2576</th>
      <td>189980</td>
      <td>흥국에프엔비</td>
      <td>KOSDAQ</td>
      <td>음식료·담배</td>
      <td>2695</td>
      <td>108171443765</td>
      <td>20230405</td>
      <td>164.0</td>
      <td>None</td>
      <td>2004.0</td>
      <td>30.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2577</th>
      <td>037440</td>
      <td>희림</td>
      <td>KOSDAQ</td>
      <td>기타서비스</td>
      <td>9430</td>
      <td>131288939250</td>
      <td>20230405</td>
      <td>490.0</td>
      <td>None</td>
      <td>4769.0</td>
      <td>150.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2578</th>
      <td>238490</td>
      <td>힘스</td>
      <td>KOSDAQ</td>
      <td>반도체</td>
      <td>7220</td>
      <td>81674343920</td>
      <td>20230405</td>
      <td>None</td>
      <td>None</td>
      <td>6598.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
  </tbody>
</table>
<p>2579 rows × 12 columns</p>
</div>




```python
# null값 확인
kor_ticker.isnull().sum()
```




    종목코드        0
    종목명         0
    시장구분        0
    업종명         0
    종가          0
    시가총액        0
    기준일         0
    EPS       960
    선행EPS    2069
    BPS       274
    주당배당금      49
    종목구분        0
    dtype: int64




```python
temp = pd.read_pickle("./data/kor_ticker.pickle")
temp = pd.concat([temp, kor_ticker]).drop_duplicates().reset_index(drop=True)
temp
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
      <th>시장구분</th>
      <th>종가</th>
      <th>시가총액</th>
      <th>기준일</th>
      <th>EPS</th>
      <th>선행EPS</th>
      <th>BPS</th>
      <th>주당배당금</th>
      <th>종목구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>095570</td>
      <td>AJ네트웍스</td>
      <td>KOSPI</td>
      <td>4735</td>
      <td>221703566825</td>
      <td>20230404</td>
      <td>1707.0</td>
      <td>619.0</td>
      <td>8075.0</td>
      <td>270.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>1</th>
      <td>006840</td>
      <td>AK홀딩스</td>
      <td>KOSPI</td>
      <td>18330</td>
      <td>242827793130</td>
      <td>20230404</td>
      <td>None</td>
      <td>None</td>
      <td>45961.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2</th>
      <td>027410</td>
      <td>BGF</td>
      <td>KOSPI</td>
      <td>4480</td>
      <td>428811223680</td>
      <td>20230404</td>
      <td>684.0</td>
      <td>700.0</td>
      <td>16393.0</td>
      <td>110.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>3</th>
      <td>282330</td>
      <td>BGF리테일</td>
      <td>KOSPI</td>
      <td>187600</td>
      <td>3242460765600</td>
      <td>20230404</td>
      <td>8547.0</td>
      <td>13676.0</td>
      <td>46849.0</td>
      <td>3000.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>4</th>
      <td>138930</td>
      <td>BNK금융지주</td>
      <td>KOSPI</td>
      <td>6490</td>
      <td>2115319746540</td>
      <td>20230404</td>
      <td>2341.0</td>
      <td>2644.0</td>
      <td>28745.0</td>
      <td>560.0</td>
      <td>보통주</td>
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
      <th>5153</th>
      <td>024060</td>
      <td>흥구석유</td>
      <td>KOSDAQ</td>
      <td>5890</td>
      <td>88350000000</td>
      <td>20230405</td>
      <td>96.0</td>
      <td>None</td>
      <td>5426.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>5154</th>
      <td>010240</td>
      <td>흥국</td>
      <td>KOSDAQ</td>
      <td>6260</td>
      <td>77140076960</td>
      <td>20230405</td>
      <td>1027.0</td>
      <td>None</td>
      <td>7439.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>5155</th>
      <td>189980</td>
      <td>흥국에프엔비</td>
      <td>KOSDAQ</td>
      <td>2695</td>
      <td>108171443765</td>
      <td>20230405</td>
      <td>164.0</td>
      <td>None</td>
      <td>2004.0</td>
      <td>30.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>5156</th>
      <td>037440</td>
      <td>희림</td>
      <td>KOSDAQ</td>
      <td>9430</td>
      <td>131288939250</td>
      <td>20230405</td>
      <td>490.0</td>
      <td>None</td>
      <td>4769.0</td>
      <td>150.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>5157</th>
      <td>238490</td>
      <td>힘스</td>
      <td>KOSDAQ</td>
      <td>7220</td>
      <td>81674343920</td>
      <td>20230405</td>
      <td>None</td>
      <td>None</td>
      <td>6598.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
  </tbody>
</table>
<p>5158 rows × 11 columns</p>
</div>




```python
temp.to_pickle("./data/kor_ticker.pickle")
```


```python
temp = pd.read_pickle("./data/kor_ticker.pickle")
temp
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
      <th>시장구분</th>
      <th>업종명</th>
      <th>종가</th>
      <th>시가총액</th>
      <th>기준일</th>
      <th>EPS</th>
      <th>선행EPS</th>
      <th>BPS</th>
      <th>주당배당금</th>
      <th>종목구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>095570</td>
      <td>AJ네트웍스</td>
      <td>KOSPI</td>
      <td>서비스업</td>
      <td>4670</td>
      <td>218660117650</td>
      <td>20230405</td>
      <td>1707.0</td>
      <td>619.0</td>
      <td>8075.0</td>
      <td>270.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>1</th>
      <td>006840</td>
      <td>AK홀딩스</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>18450</td>
      <td>244417500450</td>
      <td>20230405</td>
      <td>None</td>
      <td>None</td>
      <td>45961.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2</th>
      <td>027410</td>
      <td>BGF</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>4480</td>
      <td>428811223680</td>
      <td>20230405</td>
      <td>684.0</td>
      <td>700.0</td>
      <td>16393.0</td>
      <td>110.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>3</th>
      <td>282330</td>
      <td>BGF리테일</td>
      <td>KOSPI</td>
      <td>유통업</td>
      <td>189300</td>
      <td>3271843405800</td>
      <td>20230405</td>
      <td>8547.0</td>
      <td>12749.0</td>
      <td>46849.0</td>
      <td>3000.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>4</th>
      <td>138930</td>
      <td>BNK금융지주</td>
      <td>KOSPI</td>
      <td>기타금융</td>
      <td>6510</td>
      <td>2121838451460</td>
      <td>20230405</td>
      <td>2341.0</td>
      <td>2637.0</td>
      <td>28745.0</td>
      <td>560.0</td>
      <td>보통주</td>
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
    </tr>
    <tr>
      <th>2574</th>
      <td>024060</td>
      <td>흥구석유</td>
      <td>KOSDAQ</td>
      <td>유통</td>
      <td>5890</td>
      <td>88350000000</td>
      <td>20230405</td>
      <td>96.0</td>
      <td>None</td>
      <td>5426.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2575</th>
      <td>010240</td>
      <td>흥국</td>
      <td>KOSDAQ</td>
      <td>기계·장비</td>
      <td>6260</td>
      <td>77140076960</td>
      <td>20230405</td>
      <td>1027.0</td>
      <td>None</td>
      <td>7439.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2576</th>
      <td>189980</td>
      <td>흥국에프엔비</td>
      <td>KOSDAQ</td>
      <td>음식료·담배</td>
      <td>2695</td>
      <td>108171443765</td>
      <td>20230405</td>
      <td>164.0</td>
      <td>None</td>
      <td>2004.0</td>
      <td>30.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2577</th>
      <td>037440</td>
      <td>희림</td>
      <td>KOSDAQ</td>
      <td>기타서비스</td>
      <td>9430</td>
      <td>131288939250</td>
      <td>20230405</td>
      <td>490.0</td>
      <td>None</td>
      <td>4769.0</td>
      <td>150.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>2578</th>
      <td>238490</td>
      <td>힘스</td>
      <td>KOSDAQ</td>
      <td>반도체</td>
      <td>7220</td>
      <td>81674343920</td>
      <td>20230405</td>
      <td>None</td>
      <td>None</td>
      <td>6598.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
  </tbody>
</table>
<p>2579 rows × 12 columns</p>
</div>




```python
temp['업종명'].unique()
```




    array(['서비스업', '기타금융', '유통업', '섬유의복', '운수창고업', '음식료품', '증권', '보험', '전기전자',
           '건설업', '화학', '철강금속', '광업', '운수장비', '기계', '의약품', '비금속광물', '통신업',
           '기타제조업', '전기가스업', '종이목재', '은행', '의료정밀', '농업, 임업 및 어업', '기계·장비',
           '금융', '반도체', '통신장비', '운송장비·부품', '방송서비스', '기타서비스', '유통', '제약', '건설',
           '일반전기전자', '출판·매체복제', '섬유·의류', '오락·문화', '금속', '소프트웨어', 'IT부품',
           '디지털컨텐츠', '기타제조', '비금속', '운송', '인터넷', '정보기기', '음식료·담배', '종이·목재',
           '의료·정밀기기', '통신서비스', '컴퓨터서비스', '전기·가스·수도', '숙박·음식'], dtype=object)



### ❍ 섹터 데이터(kor_sector)


```python
### wiseindex.com/index
### WICS > 에너지 > Components > 개발자도구 > 날짜 선택 > 개발자도구 Headers 
### json형식 : https://www.wiseindex.com/Index/GetIndexComponets?ceil_yn=0&dt=20230224&sec_cd=G10
```


```python
import json
import requests as rq
import pandas as pd
```


```python
url = f'''https://www.wiseindex.com/Index/GetIndexComponets?ceil_yn=0&dt={biz_day}&sec_cd=G10'''
data = rq.get(url).json()
data.keys()
```




    dict_keys(['info', 'list', 'sector', 'size'])




```python
# 구성종목 data['list']
# 섹터코드 data['sector']
```


```python
data['list'][0]
```




    {'IDX_CD': 'G10',
     'IDX_NM_KOR': 'WICS 에너지',
     'ALL_MKT_VAL': 22080412,
     'CMP_CD': '096770',
     'CMP_KOR': 'SK이노베이션',
     'MKT_VAL': 9273926,
     'WGT': 42.0,
     'S_WGT': 42.0,
     'CAL_WGT': 1.0,
     'SEC_CD': 'G10',
     'SEC_NM_KOR': '에너지',
     'SEQ': 1,
     'TOP60': 3,
     'APT_SHR_CNT': 51780716}




```python
data_pd = pd.json_normalize(data['list'])
data_pd.head()
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
      <th>IDX_CD</th>
      <th>IDX_NM_KOR</th>
      <th>ALL_MKT_VAL</th>
      <th>CMP_CD</th>
      <th>CMP_KOR</th>
      <th>MKT_VAL</th>
      <th>WGT</th>
      <th>S_WGT</th>
      <th>CAL_WGT</th>
      <th>SEC_CD</th>
      <th>SEC_NM_KOR</th>
      <th>SEQ</th>
      <th>TOP60</th>
      <th>APT_SHR_CNT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>G10</td>
      <td>WICS 에너지</td>
      <td>22080412</td>
      <td>096770</td>
      <td>SK이노베이션</td>
      <td>9273926</td>
      <td>42.00</td>
      <td>42.00</td>
      <td>1.0</td>
      <td>G10</td>
      <td>에너지</td>
      <td>1</td>
      <td>3</td>
      <td>51780716</td>
    </tr>
    <tr>
      <th>1</th>
      <td>G10</td>
      <td>WICS 에너지</td>
      <td>22080412</td>
      <td>010950</td>
      <td>S-Oil</td>
      <td>3399100</td>
      <td>15.39</td>
      <td>57.39</td>
      <td>1.0</td>
      <td>G10</td>
      <td>에너지</td>
      <td>2</td>
      <td>3</td>
      <td>41655633</td>
    </tr>
    <tr>
      <th>2</th>
      <td>G10</td>
      <td>WICS 에너지</td>
      <td>22080412</td>
      <td>267250</td>
      <td>HD현대</td>
      <td>2601084</td>
      <td>11.78</td>
      <td>69.17</td>
      <td>1.0</td>
      <td>G10</td>
      <td>에너지</td>
      <td>3</td>
      <td>3</td>
      <td>44236128</td>
    </tr>
    <tr>
      <th>3</th>
      <td>G10</td>
      <td>WICS 에너지</td>
      <td>22080412</td>
      <td>078930</td>
      <td>GS</td>
      <td>1989504</td>
      <td>9.01</td>
      <td>78.19</td>
      <td>1.0</td>
      <td>G10</td>
      <td>에너지</td>
      <td>4</td>
      <td>3</td>
      <td>49245150</td>
    </tr>
    <tr>
      <th>4</th>
      <td>G10</td>
      <td>WICS 에너지</td>
      <td>22080412</td>
      <td>112610</td>
      <td>씨에스윈드</td>
      <td>1719244</td>
      <td>7.79</td>
      <td>85.97</td>
      <td>1.0</td>
      <td>G10</td>
      <td>에너지</td>
      <td>5</td>
      <td>3</td>
      <td>23615986</td>
    </tr>
  </tbody>
</table>
</div>




```python
import json
import time
import requests as rq
import pandas as pd
from tqdm import tqdm

sector_code = ['G25','G35','G50','G40','G10','G20','G55','G30','G15','G45']
data_sector = []

for i in tqdm(sector_code):
    url = f'''https://www.wiseindex.com/Index/GetIndexComponets?ceil_yn=0&dt={biz_day}&sec_cd={i}'''
    data = rq.get(url).json()
    data_pd = pd.json_normalize(data['list'])
    data_sector.append(data_pd)
    time.sleep(2)
```

    100%|███████████████████████████████████████████| 10/10 [00:28<00:00,  2.81s/it]



```python
# 데이터프레임 10개를 품은 리스트
type(data_sector)
```




    list




```python
kor_sector = pd.concat(data_sector, axis=0)
kor_sector = kor_sector[['IDX_CD','CMP_CD','CMP_KOR','SEC_NM_KOR']]
kor_sector['기준일'] = biz_day
kor_sector['기준일'] = pd.to_datetime(kor_sector['기준일'])
kor_sector
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
      <th>IDX_CD</th>
      <th>CMP_CD</th>
      <th>CMP_KOR</th>
      <th>SEC_NM_KOR</th>
      <th>기준일</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>G25</td>
      <td>005380</td>
      <td>현대차</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>1</th>
      <td>G25</td>
      <td>000270</td>
      <td>기아</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>2</th>
      <td>G25</td>
      <td>012330</td>
      <td>현대모비스</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>3</th>
      <td>G25</td>
      <td>051900</td>
      <td>LG생활건강</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>4</th>
      <td>G25</td>
      <td>090430</td>
      <td>아모레퍼시픽</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
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
      <th>643</th>
      <td>G45</td>
      <td>009140</td>
      <td>경인전자</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>644</th>
      <td>G45</td>
      <td>321260</td>
      <td>프로이천</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>645</th>
      <td>G45</td>
      <td>033200</td>
      <td>모아텍</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>646</th>
      <td>G45</td>
      <td>038340</td>
      <td>MIT</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>647</th>
      <td>G45</td>
      <td>174880</td>
      <td>장원테크</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
  </tbody>
</table>
<p>2384 rows × 5 columns</p>
</div>




```python
kor_sector['SEC_NM_KOR'].unique()
```




    array(['경기관련소비재', '건강관리', '커뮤니케이션서비스', '금융', '에너지', '산업재', '유틸리티',
           '필수소비재', '소재', 'IT'], dtype=object)




```python
temp = pd.read_pickle('./data/kor_sector.pickle')
temp = pd.concat([temp, kor_sector]).drop_duplicates().reset_index(drop=True)
temp
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
      <th>IDX_CD</th>
      <th>CMP_CD</th>
      <th>CMP_KOR</th>
      <th>SEC_NM_KOR</th>
      <th>기준일</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>G25</td>
      <td>005380</td>
      <td>현대차</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>1</th>
      <td>G25</td>
      <td>000270</td>
      <td>기아</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>2</th>
      <td>G25</td>
      <td>012330</td>
      <td>현대모비스</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>3</th>
      <td>G25</td>
      <td>051900</td>
      <td>LG생활건강</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>4</th>
      <td>G25</td>
      <td>090430</td>
      <td>아모레퍼시픽</td>
      <td>경기관련소비재</td>
      <td>2023-04-05</td>
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
      <th>2379</th>
      <td>G45</td>
      <td>009140</td>
      <td>경인전자</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>2380</th>
      <td>G45</td>
      <td>321260</td>
      <td>프로이천</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>2381</th>
      <td>G45</td>
      <td>033200</td>
      <td>모아텍</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>2382</th>
      <td>G45</td>
      <td>038340</td>
      <td>MIT</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>2383</th>
      <td>G45</td>
      <td>174880</td>
      <td>장원테크</td>
      <td>IT</td>
      <td>2023-04-05</td>
    </tr>
  </tbody>
</table>
<p>2384 rows × 5 columns</p>
</div>




```python
temp.to_pickle('./data/kor_sector.pickle')
```

### ❍ 월말기준으로 크롤링 (kor_ticker, kor_sector)


```python
import datetime
import pandas as pd
from pandas.tseries.offsets import BMonthEnd

# create a date range from January 2010 to today
date_range = pd.date_range(start='2010-01-01', end='2023-04-30', freq='BM')

# get the last business day of each month
last_business_days = [d.date() for d in date_range - BMonthEnd()]

# 12월 마지막 영업일은 주식시장 문 닫음
last_business_days = [i-datetime.timedelta(days=1) if i.month==12 else i for i in last_business_days]
last_business_days = [i.strftime("%Y%m%d") for i in last_business_days]

# 수동조절
manual_dict = {'20121230':'20121228','20140131':'20140129','20181230':'20181228', '20200430':'20200429', '20200930':'20200929', '20220131':'20220128'}
last_business_days = [manual_dict[i] if i in manual_dict.keys() else i for i in last_business_days]

print(last_business_days)
dates = last_business_days
```

    ['20091230', '20100129', '20100226', '20100331', '20100430', '20100531', '20100630', '20100730', '20100831', '20100930', '20101029', '20101130', '20101230', '20110131', '20110228', '20110331', '20110429', '20110531', '20110630', '20110729', '20110831', '20110930', '20111031', '20111130', '20111229', '20120131', '20120229', '20120330', '20120430', '20120531', '20120629', '20120731', '20120831', '20120928', '20121031', '20121130', '20121228', '20130131', '20130228', '20130329', '20130430', '20130531', '20130628', '20130731', '20130830', '20130930', '20131031', '20131129', '20131230', '20140129', '20140228', '20140331', '20140430', '20140530', '20140630', '20140731', '20140829', '20140930', '20141031', '20141128', '20141230', '20150130', '20150227', '20150331', '20150430', '20150529', '20150630', '20150731', '20150831', '20150930', '20151030', '20151130', '20151230', '20160129', '20160229', '20160331', '20160429', '20160531', '20160630', '20160729', '20160831', '20160930', '20161031', '20161130', '20161229', '20170131', '20170228', '20170331', '20170428', '20170531', '20170630', '20170731', '20170831', '20170929', '20171031', '20171130', '20171228', '20180131', '20180228', '20180330', '20180430', '20180531', '20180629', '20180731', '20180831', '20180928', '20181031', '20181130', '20181228', '20190131', '20190228', '20190329', '20190430', '20190531', '20190628', '20190731', '20190830', '20190930', '20191031', '20191129', '20191230', '20200131', '20200228', '20200331', '20200429', '20200529', '20200630', '20200731', '20200831', '20200929', '20201030', '20201130', '20201230', '20210129', '20210226', '20210331', '20210430', '20210531', '20210630', '20210730', '20210831', '20210930', '20211029', '20211130', '20211230', '20220128', '20220228', '20220331', '20220429', '20220531', '20220630', '20220729', '20220831', '20220930', '20221031', '20221130', '20221229', '20230131', '20230228', '20230331']


#### kor_ticker


```python
def crawling_kor_ticker(biz_day):
    def crawling_sector(market='STK'): 
        # generate.cmd > Headers > Request URL
        gen_otp_url = 'http://data.krx.co.kr/comm/fileDn/GenerateOTP/generate.cmd' 

        # generate.cmd > Payload
        gen_otp_stk = {
            'mktId' : market, # KOSPI : "STK", KOSDAQ : "KSQ"
            'trdDd' : biz_day,
            'money' : '1', 
            'csvxls_isNo' : 'false', 
            'name' : 'fileDown', 
            'url' : 'dbms/MDC/STAT/standard/MDCSTAT03901'
                      }

        # generate.cmd > Headers > Referer (로봇이 아님을..)
        headers = {'Referer' : 'http://data.krx.co.kr/contents/MDC/MDI/mdiLoader'}

        otp_stk = rq.post(gen_otp_url, gen_otp_stk, headers=headers).text

        # download.cmd > headers > Request URL
        down_url = 'http://data.krx.co.kr/comm/fileDn/download_csv/download.cmd'

        down_sector_stk = rq.post(down_url, {'code':otp_stk}, headers=headers)

        df= pd.read_csv(BytesIO(down_sector_stk.content), encoding='EUC-KR') # Binary String 형태로 변경
        return df

    sector_stk = crawling_sector(market='STK') #KOSPI
    sector_ksq = crawling_sector(market='KSQ') #KOSDAQ
    krx_sector = pd.concat([sector_stk, sector_ksq]).reset_index(drop=True)
    krx_sector['종목명'] = krx_sector['종목명'].str.strip() # 공백 제거
    krx_sector['기준일'] = biz_day

    # generate.cmd > Headers > Request URL
    gen_otp_url = 'http://data.krx.co.kr/comm/fileDn/GenerateOTP/generate.cmd' 

    # generate.cmd > Payload
    gen_otp_data = {
        'searchType':'1',
        'mktId' : 'ALL',
        'trdDd' : biz_day, #데이터가 안나올 경우 bizday 문제
        'csvxls_isNo' : 'false', 
        'name' : 'fileDown', 
        'url' : 'dbms/MDC/STAT/standard/MDCSTAT03501'
                  }

    # generate.cmd > Headers > Referer (로봇이 아님을..)
    headers = {'Referer' : 'http://data.krx.co.kr/contents/MDC/MDI/mdiLoader'}
    otp = rq.post(gen_otp_url, gen_otp_data, headers=headers).text

    # download.cmd > headers > Request URL
    down_url = 'http://data.krx.co.kr/comm/fileDn/download_csv/download.cmd'
    krx_ind = rq.post(down_url, {'code': otp}, headers=headers)

    krx_ind = pd.read_csv(BytesIO(krx_ind.content), encoding='EUC-KR') # Binary String 형태로 변경
    krx_ind['종목명'] = krx_ind['종목명'].str.strip()
    krx_ind['기준일'] = biz_day

    # 합치기
    kor_ticker = pd.merge(krx_sector, 
                          krx_ind,
                          on = krx_sector.columns.intersection(krx_ind.columns).tolist(), 
                          how='outer')

    # 한쪽만 있는 데이터
    diff = list(set(krx_sector['종목명']).symmetric_difference(set(krx_ind['종목명'])))

    # 조건
    kor_ticker['종목구분'] = np.where(kor_ticker['종목명'].str.contains('스팩|제[0-9]+호'), '스팩',
                                  np.where(kor_ticker['종목코드'].str[-1:] != '0', '우선주',
                                           np.where(kor_ticker['종목명'].str.endswith('리츠'),'리츠',
                                                    np.where(kor_ticker['종목명'].isin(diff), '기타',
                                                             '보통주'))))

    kor_ticker = kor_ticker.reset_index(drop=True)
    kor_ticker.columns = kor_ticker.columns.str.replace(" ", "") #공백제거
    kor_ticker = kor_ticker[['종목코드','종목명','시장구분','업종명','종가',
                             '시가총액','기준일','EPS','선행EPS','BPS','주당배당금','종목구분']] # 열 선택
    kor_ticker = kor_ticker.replace({np.nan : None}) # nan값을 None으로 변경 : SQL에 nan을 저장할 수 없음
    return kor_ticker
```


```python
kor_ticker = pd.DataFrame(columns = ['종목코드', '종목명', '시장구분', '업종명', 
                             '종가', '시가총액', '기준일', 'EPS', '선행EPS',
                             'BPS', '주당배당금', '종목구분'])
kor_ticker
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
      <th>시장구분</th>
      <th>업종명</th>
      <th>종가</th>
      <th>시가총액</th>
      <th>기준일</th>
      <th>EPS</th>
      <th>선행EPS</th>
      <th>BPS</th>
      <th>주당배당금</th>
      <th>종목구분</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
for date in tqdm(dates):
    temp = crawling_kor_ticker(date)
    kor_ticker = pd.concat([kor_ticker,temp])
kor_ticker = kor_ticker.drop_duplicates().reset_index(drop=True)
kor_ticker 
```

    100%|█████████████████████████████████████████| 160/160 [09:09<00:00,  3.44s/it]





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
      <th>시장구분</th>
      <th>업종명</th>
      <th>종가</th>
      <th>시가총액</th>
      <th>기준일</th>
      <th>EPS</th>
      <th>선행EPS</th>
      <th>BPS</th>
      <th>주당배당금</th>
      <th>종목구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>004560</td>
      <td>BNG스틸</td>
      <td>KOSPI</td>
      <td>철강금속</td>
      <td>8650</td>
      <td>130431715150.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>11755.0</td>
      <td>0.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>1</th>
      <td>004565</td>
      <td>BNG스틸우</td>
      <td>KOSPI</td>
      <td>철강금속</td>
      <td>8370</td>
      <td>919461240.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>0.0</td>
      <td>우선주</td>
    </tr>
    <tr>
      <th>2</th>
      <td>001460</td>
      <td>BYC</td>
      <td>KOSPI</td>
      <td>섬유의복</td>
      <td>125000</td>
      <td>78076875000.0</td>
      <td>20091230</td>
      <td>7741.0</td>
      <td>None</td>
      <td>288027.0</td>
      <td>750.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>3</th>
      <td>001465</td>
      <td>BYC우</td>
      <td>KOSPI</td>
      <td>섬유의복</td>
      <td>55000</td>
      <td>11846175000.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>800.0</td>
      <td>우선주</td>
    </tr>
    <tr>
      <th>4</th>
      <td>084680</td>
      <td>C&amp;우방랜드</td>
      <td>KOSPI</td>
      <td>서비스업</td>
      <td>1130</td>
      <td>37149976050.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>656.0</td>
      <td>0.0</td>
      <td>보통주</td>
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
    </tr>
    <tr>
      <th>428558</th>
      <td>024060</td>
      <td>흥구석유</td>
      <td>KOSDAQ</td>
      <td>유통</td>
      <td>5600</td>
      <td>84000000000</td>
      <td>20230331</td>
      <td>96.0</td>
      <td>None</td>
      <td>5426.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428559</th>
      <td>010240</td>
      <td>흥국</td>
      <td>KOSDAQ</td>
      <td>기계·장비</td>
      <td>6140</td>
      <td>75661353440</td>
      <td>20230331</td>
      <td>1027.0</td>
      <td>None</td>
      <td>7439.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428560</th>
      <td>189980</td>
      <td>흥국에프엔비</td>
      <td>KOSDAQ</td>
      <td>음식료·담배</td>
      <td>2720</td>
      <td>109174889440</td>
      <td>20230331</td>
      <td>164.0</td>
      <td>None</td>
      <td>2004.0</td>
      <td>30.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428561</th>
      <td>037440</td>
      <td>희림</td>
      <td>KOSDAQ</td>
      <td>기타서비스</td>
      <td>9120</td>
      <td>126972972000</td>
      <td>20230331</td>
      <td>490.0</td>
      <td>None</td>
      <td>4769.0</td>
      <td>150.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428562</th>
      <td>238490</td>
      <td>힘스</td>
      <td>KOSDAQ</td>
      <td>반도체</td>
      <td>7100</td>
      <td>80316875600</td>
      <td>20230331</td>
      <td>None</td>
      <td>None</td>
      <td>6598.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
  </tbody>
</table>
<p>428563 rows × 12 columns</p>
</div>




```python
kor_ticker.isnull().sum()
```




    종목코드          0
    종목명           0
    시장구분      87536
    업종명       87536
    종가            0
    시가총액      87536
    기준일           0
    EPS      215076
    선행EPS    419184
    BPS      124273
    주당배당금     95744
    종목구분          0
    dtype: int64




```python
# kor_ticker.to_pickle('./data/kor_ticker.pickle')
```


```python
kor_ticker_history = pd.read_pickle('./data/kor_ticker.pickle')
kor_ticker_history = pd.concat([kor_ticker_history, kor_ticker]).drop_duplicates().reset_index(drop=True)
kor_ticker_history
#kor_ticker_history.to_pickle('./data/kor_ticker.pickle')
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
      <th>시장구분</th>
      <th>업종명</th>
      <th>종가</th>
      <th>시가총액</th>
      <th>기준일</th>
      <th>EPS</th>
      <th>선행EPS</th>
      <th>BPS</th>
      <th>주당배당금</th>
      <th>종목구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>004560</td>
      <td>BNG스틸</td>
      <td>KOSPI</td>
      <td>철강금속</td>
      <td>8650</td>
      <td>130431715150.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>11755.0</td>
      <td>0.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>1</th>
      <td>004565</td>
      <td>BNG스틸우</td>
      <td>KOSPI</td>
      <td>철강금속</td>
      <td>8370</td>
      <td>919461240.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>0.0</td>
      <td>우선주</td>
    </tr>
    <tr>
      <th>2</th>
      <td>001460</td>
      <td>BYC</td>
      <td>KOSPI</td>
      <td>섬유의복</td>
      <td>125000</td>
      <td>78076875000.0</td>
      <td>20091230</td>
      <td>7741.0</td>
      <td>None</td>
      <td>288027.0</td>
      <td>750.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>3</th>
      <td>001465</td>
      <td>BYC우</td>
      <td>KOSPI</td>
      <td>섬유의복</td>
      <td>55000</td>
      <td>11846175000.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>800.0</td>
      <td>우선주</td>
    </tr>
    <tr>
      <th>4</th>
      <td>084680</td>
      <td>C&amp;우방랜드</td>
      <td>KOSPI</td>
      <td>서비스업</td>
      <td>1130</td>
      <td>37149976050.0</td>
      <td>20091230</td>
      <td>None</td>
      <td>None</td>
      <td>656.0</td>
      <td>0.0</td>
      <td>보통주</td>
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
    </tr>
    <tr>
      <th>428558</th>
      <td>024060</td>
      <td>흥구석유</td>
      <td>KOSDAQ</td>
      <td>유통</td>
      <td>5600</td>
      <td>84000000000</td>
      <td>20230331</td>
      <td>96.0</td>
      <td>None</td>
      <td>5426.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428559</th>
      <td>010240</td>
      <td>흥국</td>
      <td>KOSDAQ</td>
      <td>기계·장비</td>
      <td>6140</td>
      <td>75661353440</td>
      <td>20230331</td>
      <td>1027.0</td>
      <td>None</td>
      <td>7439.0</td>
      <td>200.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428560</th>
      <td>189980</td>
      <td>흥국에프엔비</td>
      <td>KOSDAQ</td>
      <td>음식료·담배</td>
      <td>2720</td>
      <td>109174889440</td>
      <td>20230331</td>
      <td>164.0</td>
      <td>None</td>
      <td>2004.0</td>
      <td>30.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428561</th>
      <td>037440</td>
      <td>희림</td>
      <td>KOSDAQ</td>
      <td>기타서비스</td>
      <td>9120</td>
      <td>126972972000</td>
      <td>20230331</td>
      <td>490.0</td>
      <td>None</td>
      <td>4769.0</td>
      <td>150.0</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>428562</th>
      <td>238490</td>
      <td>힘스</td>
      <td>KOSDAQ</td>
      <td>반도체</td>
      <td>7100</td>
      <td>80316875600</td>
      <td>20230331</td>
      <td>None</td>
      <td>None</td>
      <td>6598.0</td>
      <td>100.0</td>
      <td>보통주</td>
    </tr>
  </tbody>
</table>
<p>428563 rows × 12 columns</p>
</div>



#### kor_sector
- 2010년 1월 데이터부터 존재함


```python
def crawling_kor_sector(biz_day):
    sector_code = ['G25','G35','G50','G40','G10','G20','G55','G30','G15','G45']
    data_sector = []

    for i in sector_code:
        url = f'''https://www.wiseindex.com/Index/GetIndexComponets?ceil_yn=0&dt={biz_day}&sec_cd={i}'''
        data = rq.get(url).json()
        data_pd = pd.json_normalize(data['list'])
        data_sector.append(data_pd)
        time.sleep(2)
    kor_sector = pd.concat(data_sector, axis=0)
    kor_sector = kor_sector[['IDX_CD','CMP_CD','CMP_KOR','SEC_NM_KOR']]
    kor_sector['기준일'] = biz_day
    kor_sector['기준일'] = pd.to_datetime(kor_sector['기준일'])
    return kor_sector
```


```python
kor_sector = pd.DataFrame(columns = ['IDX_CD','CMP_CD','CMP_KOR','SEC_NM_KOR','기준일'])
kor_sector
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
      <th>IDX_CD</th>
      <th>CMP_CD</th>
      <th>CMP_KOR</th>
      <th>SEC_NM_KOR</th>
      <th>기준일</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
for date in tqdm(dates[1:]): # 2010-01 부터
    temp = crawling_kor_sector(date)
    kor_sector = pd.concat([kor_sector,temp])
kor_sector = kor_sector.drop_duplicates().reset_index(drop=True)
kor_sector 
```

    100%|█████████████████████████████████████████| 158/158 [17:09<00:00,  6.52s/it]





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
      <th>IDX_CD</th>
      <th>CMP_CD</th>
      <th>CMP_KOR</th>
      <th>SEC_NM_KOR</th>
      <th>기준일</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>G25</td>
      <td>005380</td>
      <td>현대차</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>1</th>
      <td>G25</td>
      <td>012330</td>
      <td>현대모비스</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>2</th>
      <td>G25</td>
      <td>004170</td>
      <td>신세계</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>3</th>
      <td>G25</td>
      <td>000270</td>
      <td>기아</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>4</th>
      <td>G25</td>
      <td>023530</td>
      <td>롯데쇼핑</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
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
      <th>294780</th>
      <td>G45</td>
      <td>174880</td>
      <td>장원테크</td>
      <td>IT</td>
      <td>2023-02-28</td>
    </tr>
    <tr>
      <th>294781</th>
      <td>G45</td>
      <td>043590</td>
      <td>크로바하이텍</td>
      <td>IT</td>
      <td>2023-02-28</td>
    </tr>
    <tr>
      <th>294782</th>
      <td>G45</td>
      <td>033200</td>
      <td>모아텍</td>
      <td>IT</td>
      <td>2023-02-28</td>
    </tr>
    <tr>
      <th>294783</th>
      <td>G45</td>
      <td>321260</td>
      <td>프로이천</td>
      <td>IT</td>
      <td>2023-02-28</td>
    </tr>
    <tr>
      <th>294784</th>
      <td>G45</td>
      <td>038340</td>
      <td>MIT</td>
      <td>IT</td>
      <td>2023-02-28</td>
    </tr>
  </tbody>
</table>
<p>294785 rows × 5 columns</p>
</div>




```python
#kor_sector.to_pickle("./data/kor_sector.pickle")
```


```python
kor_sector_history = pd.read_pickle("./data/kor_sector.pickle")
kor_sector_history = pd.concat([kor_sector_history, kor_sector]).drop_duplicates().reset_index(drop=True)
kor_sector_history
# kor_sector_history.to_pickle('./data/kor_sector.pickle')
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
      <th>IDX_CD</th>
      <th>CMP_CD</th>
      <th>CMP_KOR</th>
      <th>SEC_NM_KOR</th>
      <th>기준일</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>G25</td>
      <td>005380</td>
      <td>현대차</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>1</th>
      <td>G25</td>
      <td>012330</td>
      <td>현대모비스</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>2</th>
      <td>G25</td>
      <td>004170</td>
      <td>신세계</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>3</th>
      <td>G25</td>
      <td>000270</td>
      <td>기아</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
    </tr>
    <tr>
      <th>4</th>
      <td>G25</td>
      <td>023530</td>
      <td>롯데쇼핑</td>
      <td>경기관련소비재</td>
      <td>2010-01-29</td>
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
      <th>297166</th>
      <td>G45</td>
      <td>009140</td>
      <td>경인전자</td>
      <td>IT</td>
      <td>2023-03-31</td>
    </tr>
    <tr>
      <th>297167</th>
      <td>G45</td>
      <td>321260</td>
      <td>프로이천</td>
      <td>IT</td>
      <td>2023-03-31</td>
    </tr>
    <tr>
      <th>297168</th>
      <td>G45</td>
      <td>033200</td>
      <td>모아텍</td>
      <td>IT</td>
      <td>2023-03-31</td>
    </tr>
    <tr>
      <th>297169</th>
      <td>G45</td>
      <td>038340</td>
      <td>MIT</td>
      <td>IT</td>
      <td>2023-03-31</td>
    </tr>
    <tr>
      <th>297170</th>
      <td>G45</td>
      <td>174880</td>
      <td>장원테크</td>
      <td>IT</td>
      <td>2023-03-31</td>
    </tr>
  </tbody>
</table>
<p>297171 rows × 5 columns</p>
</div>




```python

```

## ❒ 주식 가격 크롤링 (stock_price)


```python
# 수정주가가 필요
# 네이버 증권을 추천함 (차트는 수정주가를 기준으로 함) 
# 네이버증권 > 개발자도구 > 차트 > 일 
# https://api.finance.naver.com/siseJson.naver?symbol=005930&requestType=1&startTime=20210414&endTime=20211220&timeframe=day
```


```python
import pandas as pd
from dateutil.relativedelta import relativedelta
import requests as rq
from io import BytesIO
from datetime import date
import time
from tqdm import tqdm

kor_sector = pd.read_pickle('./data/kor_sector.pickle')
kor_ticker = pd.read_pickle('./data/kor_ticker.pickle')

# 마지막 날 기준으로
kor_ticker_last_day = kor_ticker['기준일'].unique()[-1]
kor_sector_last_day = kor_sector['기준일'].unique()[-1]
cond = (kor_ticker['종목구분'] == '보통주') & \
        (kor_ticker['기준일']==kor_ticker_last_day)
ticker_list = kor_ticker[cond].reset_index(drop=True)

ticker_name_list = ticker_list[['종목코드','종목명']].set_index("종목코드")
ticker_name_list
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
      <th>종목명</th>
    </tr>
    <tr>
      <th>종목코드</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>095570</th>
      <td>AJ네트웍스</td>
    </tr>
    <tr>
      <th>006840</th>
      <td>AK홀딩스</td>
    </tr>
    <tr>
      <th>027410</th>
      <td>BGF</td>
    </tr>
    <tr>
      <th>282330</th>
      <td>BGF리테일</td>
    </tr>
    <tr>
      <th>138930</th>
      <td>BNK금융지주</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>024060</th>
      <td>흥구석유</td>
    </tr>
    <tr>
      <th>010240</th>
      <td>흥국</td>
    </tr>
    <tr>
      <th>189980</th>
      <td>흥국에프엔비</td>
    </tr>
    <tr>
      <th>037440</th>
      <td>희림</td>
    </tr>
    <tr>
      <th>238490</th>
      <td>힘스</td>
    </tr>
  </tbody>
</table>
<p>2334 rows × 1 columns</p>
</div>



### ❍ 한종목 가격 가져오기


```python
# 조회 시작일 ~ 종료일
fr = (date.today() + relativedelta(years=-5)).strftime("%Y%m%d")
to = (date.today()).strftime("%Y%m%d")
print(fr, to)

ticker = ticker_name_list.index[0]
print(ticker)

url = f'''https://fchart.stock.naver.com/siseJson.naver?symbol={ticker}&requestType=1&startTime={fr}&endTime={to}&timeframe=day'''
data = rq.get(url).content
data_price = pd.read_csv(BytesIO(data))
data_price
```

    20180408 20230408
    095570





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
      <th>[['날짜'</th>
      <th>'시가'</th>
      <th>'고가'</th>
      <th>'저가'</th>
      <th>'종가'</th>
      <th>'거래량'</th>
      <th>'외국인소진율']</th>
      <th>Unnamed: 7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>["20180409"</td>
      <td>6870.0</td>
      <td>7030.0</td>
      <td>6810.0</td>
      <td>6870.0</td>
      <td>26772.0</td>
      <td>8.34]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>["20180410"</td>
      <td>6870.0</td>
      <td>6960.0</td>
      <td>6590.0</td>
      <td>6940.0</td>
      <td>70302.0</td>
      <td>8.33]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>["20180411"</td>
      <td>6970.0</td>
      <td>7000.0</td>
      <td>6860.0</td>
      <td>6860.0</td>
      <td>31123.0</td>
      <td>8.32]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>["20180412"</td>
      <td>6810.0</td>
      <td>7000.0</td>
      <td>6810.0</td>
      <td>6950.0</td>
      <td>19823.0</td>
      <td>8.33]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>["20180413"</td>
      <td>6950.0</td>
      <td>7000.0</td>
      <td>6790.0</td>
      <td>6990.0</td>
      <td>35496.0</td>
      <td>8.33]</td>
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
    </tr>
    <tr>
      <th>1229</th>
      <td>["20230404"</td>
      <td>4680.0</td>
      <td>4800.0</td>
      <td>4580.0</td>
      <td>4735.0</td>
      <td>163332.0</td>
      <td>4.03]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1230</th>
      <td>["20230405"</td>
      <td>4685.0</td>
      <td>4765.0</td>
      <td>4635.0</td>
      <td>4670.0</td>
      <td>147912.0</td>
      <td>3.93]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1231</th>
      <td>["20230406"</td>
      <td>4685.0</td>
      <td>4730.0</td>
      <td>4590.0</td>
      <td>4660.0</td>
      <td>189535.0</td>
      <td>3.9]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1232</th>
      <td>["20230407"</td>
      <td>4715.0</td>
      <td>4745.0</td>
      <td>4615.0</td>
      <td>4660.0</td>
      <td>156870.0</td>
      <td>3.9]</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1233</th>
      <td>]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>1234 rows × 8 columns</p>
</div>




```python
import re
price = data_price.iloc[:, 0:6]
price.columns = ['날짜','시가','고가','저가','종가','거래량']
price = price.dropna()
price['날짜'] = price['날짜'].str.extract('(\d+)') #숫자만 뽑기
price['날짜'] = pd.to_datetime(price['날짜'])
price['종목코드'] = ticker
price.tail(3)
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
      <th>날짜</th>
      <th>시가</th>
      <th>고가</th>
      <th>저가</th>
      <th>종가</th>
      <th>거래량</th>
      <th>종목코드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1230</th>
      <td>2023-04-05</td>
      <td>4685.0</td>
      <td>4765.0</td>
      <td>4635.0</td>
      <td>4670.0</td>
      <td>147912.0</td>
      <td>095570</td>
    </tr>
    <tr>
      <th>1231</th>
      <td>2023-04-06</td>
      <td>4685.0</td>
      <td>4730.0</td>
      <td>4590.0</td>
      <td>4660.0</td>
      <td>189535.0</td>
      <td>095570</td>
    </tr>
    <tr>
      <th>1232</th>
      <td>2023-04-07</td>
      <td>4715.0</td>
      <td>4745.0</td>
      <td>4615.0</td>
      <td>4660.0</td>
      <td>156870.0</td>
      <td>095570</td>
    </tr>
  </tbody>
</table>
</div>



### ❍ 여러종목 가격 한번에 불러오기


```python
stock_price = pd.read_pickle('./data/stock_price.pickle')
stock_price.tail(1)
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
      <th>날짜</th>
      <th>시가</th>
      <th>고가</th>
      <th>저가</th>
      <th>종가</th>
      <th>거래량</th>
      <th>종목코드</th>
      <th>종목명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5873725</th>
      <td>2023-03-31</td>
      <td>4130.0</td>
      <td>4260.0</td>
      <td>4030.0</td>
      <td>4220.0</td>
      <td>637385.0</td>
      <td>450140</td>
      <td>코오롱모빌리티그룹</td>
    </tr>
  </tbody>
</table>
</div>




```python
df = pd.DataFrame(columns = ['날짜','시가','고가','저가','종가','거래량','종목코드'])
```


```python
error_list = []

#fr = "20100101"
fr = (pd.to_datetime(sorted(stock_price['날짜'].unique())[-1]) + relativedelta(days=1)).strftime("%Y%m%d")
to = (date.today()).strftime("%Y%m%d")
print(fr, to)


for i in tqdm(range(0, len(ticker_list))):
#for i in tqdm(range(0, 2)):    
    ticker = ticker_list['종목코드'][i]
#    ticker = ticker_list[0][i]

    try : 
        url = f'''https://fchart.stock.naver.com/siseJson.naver?symbol={ticker}&requestType=1&startTime={fr}&endTime={to}&timeframe=day'''

        # 데이터 다운로드
        data = rq.get(url).content
        data_price = pd.read_csv(BytesIO(data))

        # 데이터 클렌징
        price = data_price.iloc[:, 0:6]
        price.columns = ['날짜','시가','고가','저가','종가','거래량']
        price = price.dropna()
        price['날짜'] = price['날짜'].str.extract('(\d+)') #숫자만 뽑기
        price['날짜'] = pd.to_datetime(price['날짜'])
        price['종목코드'] = ticker
        df = pd.concat([df, price])
    
    except:
        error_list.append(ticker)
    
    time.sleep(2)
print('error list : ')    
print(error_list)
```

    20230401 20230408


    100%|███████████████████████████████████████████████████████████████████████████████████████████████████████| 2334/2334 [02:57<00:00, 13.12it/s]
    
    error list : 
    []


​    



```python
stock_price = pd.concat([stock_price, df]).sort_values(by=['날짜','종목코드']).drop_duplicates(['날짜','종목코드'],keep='last').reset_index(drop=True)

# 종목명 매핑
stock_price['종목명'] = stock_price['종목코드'].map(ticker_name_list['종목명'])
stock_price
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
      <th>날짜</th>
      <th>시가</th>
      <th>고가</th>
      <th>저가</th>
      <th>종가</th>
      <th>거래량</th>
      <th>종목코드</th>
      <th>종목명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2010-01-04</td>
      <td>7540.0</td>
      <td>7820.0</td>
      <td>7480.0</td>
      <td>7520.0</td>
      <td>177197.0</td>
      <td>000020</td>
      <td>동화약품</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2010-01-04</td>
      <td>3419.0</td>
      <td>3724.0</td>
      <td>3419.0</td>
      <td>3635.0</td>
      <td>1123647.0</td>
      <td>000040</td>
      <td>KR모터스</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2010-01-04</td>
      <td>9883.0</td>
      <td>10186.0</td>
      <td>9768.0</td>
      <td>9921.0</td>
      <td>721.0</td>
      <td>000050</td>
      <td>경방</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2010-01-04</td>
      <td>46309.0</td>
      <td>46516.0</td>
      <td>45789.0</td>
      <td>46517.0</td>
      <td>13390.0</td>
      <td>000070</td>
      <td>삼양홀딩스</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2010-01-04</td>
      <td>40100.0</td>
      <td>40300.0</td>
      <td>39600.0</td>
      <td>39950.0</td>
      <td>100021.0</td>
      <td>000080</td>
      <td>하이트진로</td>
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
    </tr>
    <tr>
      <th>5882123</th>
      <td>2023-04-07</td>
      <td>10820.0</td>
      <td>10920.0</td>
      <td>10250.0</td>
      <td>10330.0</td>
      <td>249003.0</td>
      <td>425420</td>
      <td>티에프이</td>
    </tr>
    <tr>
      <th>5882124</th>
      <td>2023-04-07</td>
      <td>10250.0</td>
      <td>10480.0</td>
      <td>10030.0</td>
      <td>10450.0</td>
      <td>336032.0</td>
      <td>441270</td>
      <td>파인엠텍</td>
    </tr>
    <tr>
      <th>5882125</th>
      <td>2023-04-07</td>
      <td>10360.0</td>
      <td>10480.0</td>
      <td>9980.0</td>
      <td>10030.0</td>
      <td>265553.0</td>
      <td>446070</td>
      <td>유니드비티플러스</td>
    </tr>
    <tr>
      <th>5882126</th>
      <td>2023-04-07</td>
      <td>4795.0</td>
      <td>4890.0</td>
      <td>4480.0</td>
      <td>4560.0</td>
      <td>1546455.0</td>
      <td>450140</td>
      <td>코오롱모빌리티그룹</td>
    </tr>
    <tr>
      <th>5882127</th>
      <td>2023-04-07</td>
      <td>2050.0</td>
      <td>2090.0</td>
      <td>2040.0</td>
      <td>2045.0</td>
      <td>6223645.0</td>
      <td>452260</td>
      <td>한화갤러리아</td>
    </tr>
  </tbody>
</table>
<p>5882128 rows × 8 columns</p>
</div>




```python
%%time
stock_price_pivot = stock_price.pivot(index='날짜', columns='종목명', values='종가')
stock_price_pivot
```

    CPU times: user 756 ms, sys: 123 ms, total: 879 ms
    Wall time: 879 ms





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
      <th>3S</th>
      <th>AJ네트웍스</th>
      <th>AK홀딩스</th>
      <th>APS홀딩스</th>
      <th>AP시스템</th>
      <th>AP위성</th>
      <th>BGF</th>
      <th>BGF리테일</th>
      <th>BGF에코머티리얼즈</th>
      <th>BNK금융지주</th>
      <th>...</th>
      <th>휴온스</th>
      <th>휴온스글로벌</th>
      <th>휴젤</th>
      <th>흥구석유</th>
      <th>흥국</th>
      <th>흥국에프엔비</th>
      <th>흥국화재</th>
      <th>흥아해운</th>
      <th>희림</th>
      <th>힘스</th>
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
      <th>2010-01-04</th>
      <td>1282.0</td>
      <td>NaN</td>
      <td>10331.0</td>
      <td>3282.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>5108.0</td>
      <td>NaN</td>
      <td>2655.0</td>
      <td>2330.0</td>
      <td>NaN</td>
      <td>6850.0</td>
      <td>6883.0</td>
      <td>10900.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-05</th>
      <td>1277.0</td>
      <td>NaN</td>
      <td>10206.0</td>
      <td>3278.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>5139.0</td>
      <td>NaN</td>
      <td>2670.0</td>
      <td>2195.0</td>
      <td>NaN</td>
      <td>6630.0</td>
      <td>6839.0</td>
      <td>10550.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-06</th>
      <td>1307.0</td>
      <td>NaN</td>
      <td>10206.0</td>
      <td>3275.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>5139.0</td>
      <td>NaN</td>
      <td>2700.0</td>
      <td>2150.0</td>
      <td>NaN</td>
      <td>6690.0</td>
      <td>6795.0</td>
      <td>10200.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-07</th>
      <td>1431.0</td>
      <td>NaN</td>
      <td>10300.0</td>
      <td>3323.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>5121.0</td>
      <td>NaN</td>
      <td>2680.0</td>
      <td>2150.0</td>
      <td>NaN</td>
      <td>6500.0</td>
      <td>7344.0</td>
      <td>10100.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-08</th>
      <td>1391.0</td>
      <td>NaN</td>
      <td>10362.0</td>
      <td>3442.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>5121.0</td>
      <td>NaN</td>
      <td>2680.0</td>
      <td>2150.0</td>
      <td>NaN</td>
      <td>6440.0</td>
      <td>7289.0</td>
      <td>10100.0</td>
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
      <th>2023-04-03</th>
      <td>2305.0</td>
      <td>4650.0</td>
      <td>18180.0</td>
      <td>14920.0</td>
      <td>22450.0</td>
      <td>21650.0</td>
      <td>4520.0</td>
      <td>180700.0</td>
      <td>7890.0</td>
      <td>6490.0</td>
      <td>...</td>
      <td>31900.0</td>
      <td>19740.0</td>
      <td>126900.0</td>
      <td>5720.0</td>
      <td>6200.0</td>
      <td>2760.0</td>
      <td>3180.0</td>
      <td>1379.0</td>
      <td>9300.0</td>
      <td>7110.0</td>
    </tr>
    <tr>
      <th>2023-04-04</th>
      <td>2355.0</td>
      <td>4735.0</td>
      <td>18330.0</td>
      <td>14650.0</td>
      <td>21900.0</td>
      <td>21800.0</td>
      <td>4480.0</td>
      <td>187600.0</td>
      <td>8040.0</td>
      <td>6490.0</td>
      <td>...</td>
      <td>33200.0</td>
      <td>19740.0</td>
      <td>130800.0</td>
      <td>6150.0</td>
      <td>6300.0</td>
      <td>2755.0</td>
      <td>3200.0</td>
      <td>1385.0</td>
      <td>9450.0</td>
      <td>7280.0</td>
    </tr>
    <tr>
      <th>2023-04-05</th>
      <td>2345.0</td>
      <td>4670.0</td>
      <td>18450.0</td>
      <td>14860.0</td>
      <td>22050.0</td>
      <td>22150.0</td>
      <td>4480.0</td>
      <td>189300.0</td>
      <td>8250.0</td>
      <td>6510.0</td>
      <td>...</td>
      <td>32900.0</td>
      <td>19920.0</td>
      <td>127400.0</td>
      <td>5890.0</td>
      <td>6260.0</td>
      <td>2695.0</td>
      <td>3210.0</td>
      <td>1388.0</td>
      <td>9430.0</td>
      <td>7220.0</td>
    </tr>
    <tr>
      <th>2023-04-06</th>
      <td>2290.0</td>
      <td>4660.0</td>
      <td>18480.0</td>
      <td>14180.0</td>
      <td>21350.0</td>
      <td>20850.0</td>
      <td>4410.0</td>
      <td>189400.0</td>
      <td>7970.0</td>
      <td>6440.0</td>
      <td>...</td>
      <td>32800.0</td>
      <td>19790.0</td>
      <td>129200.0</td>
      <td>5740.0</td>
      <td>6250.0</td>
      <td>2615.0</td>
      <td>3205.0</td>
      <td>1368.0</td>
      <td>9310.0</td>
      <td>7050.0</td>
    </tr>
    <tr>
      <th>2023-04-07</th>
      <td>2400.0</td>
      <td>4660.0</td>
      <td>18900.0</td>
      <td>14400.0</td>
      <td>21750.0</td>
      <td>21300.0</td>
      <td>4465.0</td>
      <td>183800.0</td>
      <td>8080.0</td>
      <td>6680.0</td>
      <td>...</td>
      <td>32850.0</td>
      <td>20300.0</td>
      <td>135400.0</td>
      <td>5750.0</td>
      <td>6150.0</td>
      <td>2620.0</td>
      <td>3190.0</td>
      <td>1352.0</td>
      <td>9480.0</td>
      <td>7260.0</td>
    </tr>
  </tbody>
</table>
<p>3275 rows × 2334 columns</p>
</div>




```python
# stock_price.to_pickle('./data/stock_price.pickle')
# stock_price_pivot.to_pickle('./data/stock_price_pivot.pickle')
```


```python

```

## ❒ ETF 가격 크롤링 (etf_price)


```python
from pykrx import stock
import requests
import json
from pandas.io.json import json_normalize
import pandas as pd
import datetime

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

    /var/folders/vr/g5rxclqn2qb11v47jh4p2lrw0000gn/T/ipykernel_1979/1657116227.py:14: FutureWarning: pandas.io.json.json_normalize is deprecated, use pandas.json_normalize instead.
      df = json_normalize(json_data['result']['etfItemList'])





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
      <th>688</th>
      <td>195980</td>
      <td>ARIRANG 신흥국MSCI(합성 H)</td>
    </tr>
    <tr>
      <th>689</th>
      <td>433250</td>
      <td>UNICORN R&amp;D 액티브</td>
    </tr>
    <tr>
      <th>690</th>
      <td>215620</td>
      <td>HK S&amp;P코리아로우볼</td>
    </tr>
    <tr>
      <th>691</th>
      <td>391670</td>
      <td>HK 베스트일레븐액티브</td>
    </tr>
    <tr>
      <th>692</th>
      <td>391680</td>
      <td>HK 하이볼액티브</td>
    </tr>
  </tbody>
</table>
<p>693 rows × 2 columns</p>
</div>




```python
etf[['종목명', '종목코드']].itertuples(index=False)
```




    <map at 0x17ed36d10>




```python
etf_price = pd.read_pickle("./data/etf_price.pickle")
etf_price
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
      <th>812503</th>
      <td>7211.78</td>
      <td>7245</td>
      <td>7245</td>
      <td>7180</td>
      <td>7205</td>
      <td>144</td>
      <td>1037685</td>
      <td>928.25</td>
      <td>히어로즈 리츠이지스액티브</td>
      <td>429870</td>
      <td>2023-04-03</td>
    </tr>
    <tr>
      <th>812504</th>
      <td>7243.46</td>
      <td>7205</td>
      <td>7220</td>
      <td>7175</td>
      <td>7190</td>
      <td>467</td>
      <td>3351265</td>
      <td>930.73</td>
      <td>히어로즈 리츠이지스액티브</td>
      <td>429870</td>
      <td>2023-04-04</td>
    </tr>
    <tr>
      <th>812505</th>
      <td>7231.48</td>
      <td>7285</td>
      <td>7285</td>
      <td>7190</td>
      <td>7190</td>
      <td>53</td>
      <td>381195</td>
      <td>929.02</td>
      <td>히어로즈 리츠이지스액티브</td>
      <td>429870</td>
      <td>2023-04-05</td>
    </tr>
    <tr>
      <th>812506</th>
      <td>7183.86</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>7190</td>
      <td>0</td>
      <td>0</td>
      <td>922.52</td>
      <td>히어로즈 리츠이지스액티브</td>
      <td>429870</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>812507</th>
      <td>7166.16</td>
      <td>7155</td>
      <td>7175</td>
      <td>7115</td>
      <td>7175</td>
      <td>133</td>
      <td>948425</td>
      <td>920.66</td>
      <td>히어로즈 리츠이지스액티브</td>
      <td>429870</td>
      <td>2023-04-07</td>
    </tr>
  </tbody>
</table>
<p>812508 rows × 11 columns</p>
</div>




```python
import time
from tqdm import tqdm
df = pd.DataFrame(columns=['NAV','시가','고가','저가','종가','거래량','거래대금','기초지수','종목명','종목코드'])
error_list = []

start_date = (pd.to_datetime(sorted(etf_price['날짜'].unique())[-1])+datetime.timedelta(days=1)).strftime("%Y%m%d")
end_date = datetime.datetime.now().strftime("%Y%m%d")
print(start_date, end_date)
for a, b in tqdm(etf[['종목명', '종목코드']].itertuples(index=False)):
    try:
        price = stock.get_etf_ohlcv_by_date(start_date, end_date, b)
        price['종목명']=a
        price['종목코드']=b
        price = price.reset_index()
        df = pd.concat([df,price])
    except:
        error_list.append(a)
    time.sleep(0.01)
df = df.reset_index(drop=True)
df
```


```python
error_list
```


```python
# 신규 추가 종목 
set(df['종목명']).symmetric_difference(set(etf_price['종목명']))
```


```python
etf_price = pd.concat([etf_price, df])
etf_price = etf_price.drop_duplicates(['종목명','종목코드','날짜'], keep='last').sort_values(by=['종목명','날짜']).reset_index(drop=True)
etf_price
```


```python
# 종목명에 null값이 있을 때,
temp = etf_price[['종목명','종목코드']].dropna().drop_duplicates(['종목코드'], keep='last').set_index('종목코드')['종목명']
etf_price.loc[etf_price['종목명'].isnull(),'종목명'] = etf_price.loc[etf_price['종목명'].isnull(),'종목코드'].map(temp)
etf_price = etf_price.sort_values(by=['종목명','날짜']).reset_index(drop=True)
etf_price
```


```python
%%time
etf_price_pivot = etf_price.pivot_table(index='날짜', columns = '종목명', values='종가')
etf_price_pivot
```

    CPU times: user 23.7 s, sys: 107 ms, total: 23.8 s
    Wall time: 23.8 s





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
      <th>ACE 200TR</th>
      <th>ACE 23-12 회사채(AA-이상)액티브</th>
      <th>ACE 24-12 회사채(AA-이상)액티브</th>
      <th>ACE ESG액티브</th>
      <th>ACE Fn5G플러스</th>
      <th>ACE Fn성장소비주도주</th>
      <th>ACE G2전기차&amp;자율주행액티브</th>
      <th>ACE KRX금현물</th>
      <th>ACE 골드선물 레버리지(합성 H)</th>
      <th>...</th>
      <th>에셋플러스 코리아플랫폼액티브</th>
      <th>파워 200</th>
      <th>파워 고배당저변동성</th>
      <th>파워 코스피100</th>
      <th>히어로즈 TDF2030액티브</th>
      <th>히어로즈 TDF2040액티브</th>
      <th>히어로즈 TDF2050액티브</th>
      <th>히어로즈 글로벌리츠이지스액티브</th>
      <th>히어로즈 단기채권ESG액티브</th>
      <th>히어로즈 리츠이지스액티브</th>
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
      <th>2010-01-04</th>
      <td>22550.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-05</th>
      <td>22555.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-06</th>
      <td>22715.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-07</th>
      <td>22455.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2010-01-08</th>
      <td>22515.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <th>2023-04-03</th>
      <td>32940.0</td>
      <td>20445.0</td>
      <td>103155.0</td>
      <td>105115.0</td>
      <td>7290.0</td>
      <td>7165.0</td>
      <td>6285.0</td>
      <td>9000.0</td>
      <td>11800.0</td>
      <td>17305.0</td>
      <td>...</td>
      <td>6960.0</td>
      <td>33010.0</td>
      <td>28360.0</td>
      <td>24730.0</td>
      <td>10390.0</td>
      <td>10535.0</td>
      <td>10550.0</td>
      <td>8905.0</td>
      <td>101925.0</td>
      <td>7205.0</td>
    </tr>
    <tr>
      <th>2023-04-04</th>
      <td>33075.0</td>
      <td>20525.0</td>
      <td>103140.0</td>
      <td>105180.0</td>
      <td>7320.0</td>
      <td>7115.0</td>
      <td>6365.0</td>
      <td>8875.0</td>
      <td>12000.0</td>
      <td>17600.0</td>
      <td>...</td>
      <td>7030.0</td>
      <td>33130.0</td>
      <td>28420.0</td>
      <td>24700.0</td>
      <td>10430.0</td>
      <td>10520.0</td>
      <td>10590.0</td>
      <td>8855.0</td>
      <td>101925.0</td>
      <td>7190.0</td>
    </tr>
    <tr>
      <th>2023-04-05</th>
      <td>33255.0</td>
      <td>20650.0</td>
      <td>103175.0</td>
      <td>105210.0</td>
      <td>7345.0</td>
      <td>7145.0</td>
      <td>6350.0</td>
      <td>8800.0</td>
      <td>12210.0</td>
      <td>18385.0</td>
      <td>...</td>
      <td>7065.0</td>
      <td>33315.0</td>
      <td>28435.0</td>
      <td>24870.0</td>
      <td>10390.0</td>
      <td>10485.0</td>
      <td>10530.0</td>
      <td>8795.0</td>
      <td>102000.0</td>
      <td>7190.0</td>
    </tr>
    <tr>
      <th>2023-04-06</th>
      <td>32680.0</td>
      <td>20270.0</td>
      <td>103210.0</td>
      <td>105310.0</td>
      <td>7215.0</td>
      <td>7000.0</td>
      <td>6320.0</td>
      <td>8680.0</td>
      <td>12275.0</td>
      <td>18420.0</td>
      <td>...</td>
      <td>7025.0</td>
      <td>32765.0</td>
      <td>28145.0</td>
      <td>24545.0</td>
      <td>10405.0</td>
      <td>10515.0</td>
      <td>10540.0</td>
      <td>8825.0</td>
      <td>102030.0</td>
      <td>7190.0</td>
    </tr>
    <tr>
      <th>2023-04-07</th>
      <td>33260.0</td>
      <td>20640.0</td>
      <td>103245.0</td>
      <td>105280.0</td>
      <td>7330.0</td>
      <td>7130.0</td>
      <td>6365.0</td>
      <td>8710.0</td>
      <td>12460.0</td>
      <td>18290.0</td>
      <td>...</td>
      <td>7070.0</td>
      <td>33300.0</td>
      <td>28070.0</td>
      <td>24850.0</td>
      <td>10390.0</td>
      <td>10525.0</td>
      <td>10550.0</td>
      <td>9000.0</td>
      <td>102060.0</td>
      <td>7175.0</td>
    </tr>
  </tbody>
</table>
<p>3275 rows × 697 columns</p>
</div>




```python
# etf_price.to_pickle("./data/etf_price.pickle")
# data.to_pickle("./data/etf_price_pivot.pickle")
```

## ❒ 재무제표 (data_fs)

- "data_fs.pickle" : FNGUIDE 에서 최근 4분기 재무제표 크롤링


```python
# 네이버증권에서 재무제표를 다운받으려면 동적으로 selenium을 써야하나 너무 느리다. 
# comp.fnguide.com
```

### ❍ 한 종목 재무제표 크롤링


```python
# 최근 시점의 보통주 데이터 불러오기

import pandas as pd
kor_ticker = pd.read_pickle('./data/kor_ticker.pickle')
kor_sector = pd.read_pickle('./data/kor_sector.pickle')

kor_ticker_last_day = kor_ticker['기준일'].unique()[-1]
kor_sector_last_day = kor_sector['기준일'].unique()[-1]
cond = (kor_ticker['종목구분'] == '보통주') & \
        (kor_ticker['기준일']==kor_ticker_last_day)
ticker_list = kor_ticker[cond].reset_index(drop=True)
```

#### 한 종목의 재무제표 데이터 다운로드


```python
i=0
ticker = ticker_list['종목코드'][0]
url = f'https://comp.fnguide.com/SVO2/ASP/SVD_Finance.asp?pGB=1&gicode=A{ticker}'
data = pd.read_html(url, displayed_only = False) 
#displayed_only = False하면 +로 숨겨져 있는 것들도 모두 가져옴
print(len(data))
```

    6



```python
print(data[0].columns.tolist(),'\n',  #연   -  손익계산서
      data[1].columns.tolist(), '\n', #분기  -  손익계산서
      data[2].columns.tolist(), '\n', #연   -  재무상태표
      data[3].columns.tolist(), '\n', #분기  -  재무상태표
      data[4].columns.tolist(),'\n',  #연   -  현금흐름표
      data[5].columns.tolist())       #분기  -  현금흐름표
```

    ['IFRS(연결)', '2019/12', '2020/12', '2021/12', '2022/12', '전년동기', '전년동기(%)'] 
     ['IFRS(연결)', '2022/03', '2022/06', '2022/09', '2022/12', '전년동기', '전년동기(%)'] 
     ['IFRS(연결)', '2019/12', '2020/12', '2021/12', '2022/12'] 
     ['IFRS(연결)', '2022/03', '2022/06', '2022/09', '2022/12'] 
     ['IFRS(연결)', '2019/12', '2020/12', '2021/12', '2022/12'] 
     ['IFRS(연결)', '2022/03', '2022/06', '2022/09', '2022/12']



```python
data[0].iloc[:, ~data[0].columns.str.contains('전년동기')].head(3)
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
      <th>IFRS(연결)</th>
      <th>2019/12</th>
      <th>2020/12</th>
      <th>2021/12</th>
      <th>2022/12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>매출액</td>
      <td>10003.0</td>
      <td>8720.0</td>
      <td>9819.0</td>
      <td>12084.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>매출원가</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>매출총이익</td>
      <td>10003.0</td>
      <td>8720.0</td>
      <td>9819.0</td>
      <td>12084.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
data[1].iloc[:, ~data[0].columns.str.contains('전년동기')].head(3)
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
      <th>IFRS(연결)</th>
      <th>2022/03</th>
      <th>2022/06</th>
      <th>2022/09</th>
      <th>2022/12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>매출액</td>
      <td>2958.0</td>
      <td>2842.0</td>
      <td>3451.0</td>
      <td>2833.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>매출원가</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>매출총이익</td>
      <td>2958.0</td>
      <td>2842.0</td>
      <td>3451.0</td>
      <td>2833.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
data[2].head(3)
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
      <th>IFRS(연결)</th>
      <th>2019/12</th>
      <th>2020/12</th>
      <th>2021/12</th>
      <th>2022/12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>자산</td>
      <td>18033.0</td>
      <td>15882.0</td>
      <td>13550.0</td>
      <td>14814.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유동자산계산에 참여한 계정 펼치기</td>
      <td>5011.0</td>
      <td>3547.0</td>
      <td>2739.0</td>
      <td>3295.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>재고자산</td>
      <td>586.0</td>
      <td>337.0</td>
      <td>197.0</td>
      <td>250.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
data[3].head(3)
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
      <th>IFRS(연결)</th>
      <th>2022/03</th>
      <th>2022/06</th>
      <th>2022/09</th>
      <th>2022/12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>자산</td>
      <td>13672.0</td>
      <td>13932.0</td>
      <td>15020.0</td>
      <td>14814.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유동자산계산에 참여한 계정 펼치기</td>
      <td>2619.0</td>
      <td>2702.0</td>
      <td>3120.0</td>
      <td>3295.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>재고자산</td>
      <td>289.0</td>
      <td>357.0</td>
      <td>428.0</td>
      <td>250.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
data[4].head(3)
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
      <th>IFRS(연결)</th>
      <th>2019/12</th>
      <th>2020/12</th>
      <th>2021/12</th>
      <th>2022/12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>영업활동으로인한현금흐름</td>
      <td>-379.0</td>
      <td>177.0</td>
      <td>-113.0</td>
      <td>183.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>당기순손익</td>
      <td>421.0</td>
      <td>-33.0</td>
      <td>767.0</td>
      <td>88.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>법인세비용차감전계속사업이익</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
data[5].head(3)
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
      <th>IFRS(연결)</th>
      <th>2022/03</th>
      <th>2022/06</th>
      <th>2022/09</th>
      <th>2022/12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>영업활동으로인한현금흐름</td>
      <td>30.0</td>
      <td>-231.0</td>
      <td>-312.0</td>
      <td>696.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>당기순손익</td>
      <td>109.0</td>
      <td>97.0</td>
      <td>103.0</td>
      <td>-221.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>법인세비용차감전계속사업이익</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



#### 연기준 재무제표 만들기


```python
# 결산월 확인하기 : 보통 12월 결산
import requests as rq
from bs4 import BeautifulSoup
import re

page_data = rq.get(url)
page_data_html = BeautifulSoup(page_data.content)
fiscal_data = page_data_html.select('div.corp_group1 > h2')
fiscal_data_text = fiscal_data[1].text
fiscal_data_text = re.findall('[0-9]+', fiscal_data_text)
print("결산월 : ", fiscal_data_text)
```

    결산월 :  ['12']



```python
# 데이터 클린징
def clean_fs(df, ticker, frequency): 
    # 계정을 제외하고 모든 열의 na인 행을 제외, 적어도 하나의 값 있는 행만 생존
    df = df[~df.loc[:, ~df.columns.isin(['계정'])].isna().all(axis=1)] 
    df = df.drop_duplicates(['계정'], keep='first') #중복되면 맨 앞 값만 생존
    df = pd.melt(df, id_vars="계정", var_name='기준일', value_name="값") 
    df = df[~pd.isnull(df['값'])]
    # + : 펼치기 버튼 삭제
    df['계정'] = df['계정'].replace({"계산에 참여한 계정 펼치기": ""}, regex=True)
    df['기준일'] = pd.to_datetime(df['기준일'], format="%Y-%m") + pd.tseries.offsets.MonthEnd() # 월말로 바꿔줌
    df['종목코드'] = ticker
    df['공시구분'] = frequency
    return df
```


```python
data_fs_y = pd.concat([data[0].iloc[:, ~data[0].columns.str.contains('전년동기')],
                       data[2],
                       data[4]
                       ])
data_fs_y = data_fs_y.rename(columns = {data_fs_y.columns[0] : "계정"})
# 12월 결산이 아니라면... 분기 재무제표(최근 결산기준) 함께 표시됨. 분기재무제표 열 제거
data_fs_y = data_fs_y.loc[:, (data_fs_y.columns == "계정") | 
                         (data_fs_y.columns.str[-2:].isin(fiscal_data_text))]
data_fs_y_clean = clean_fs(data_fs_y, ticker, 'y')
data_fs_y_clean
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
      <th>계정</th>
      <th>기준일</th>
      <th>값</th>
      <th>종목코드</th>
      <th>공시구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>매출액</td>
      <td>2019-12-31</td>
      <td>10003.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>1</th>
      <td>매출총이익</td>
      <td>2019-12-31</td>
      <td>10003.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2</th>
      <td>판매비와관리비</td>
      <td>2019-12-31</td>
      <td>9846.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인건비</td>
      <td>2019-12-31</td>
      <td>852.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>4</th>
      <td>유무형자산상각비</td>
      <td>2019-12-31</td>
      <td>109.0</td>
      <td>095570</td>
      <td>y</td>
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
      <th>586</th>
      <td>배당금지급(-)</td>
      <td>2022-12-31</td>
      <td>-121.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>588</th>
      <td>환율변동효과</td>
      <td>2022-12-31</td>
      <td>12.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>589</th>
      <td>현금및현금성자산의증가</td>
      <td>2022-12-31</td>
      <td>466.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>590</th>
      <td>기초현금및현금성자산</td>
      <td>2022-12-31</td>
      <td>852.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>591</th>
      <td>기말현금및현금성자산</td>
      <td>2022-12-31</td>
      <td>1319.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
  </tbody>
</table>
<p>544 rows × 5 columns</p>
</div>



#### 분기기준 재무제표 만들기


```python
# 분기 재무제표
data_fs_q = pd.concat(
    [data[1].iloc[:, ~data[1].columns.str.contains('전년동기')], data[3], data[5]])
data_fs_q = data_fs_q.rename(columns={data_fs_q.columns[0]: "계정"})
data_fs_q_clean = clean_fs(data_fs_q, ticker, 'q')
data_fs_q_clean
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
      <th>계정</th>
      <th>기준일</th>
      <th>값</th>
      <th>종목코드</th>
      <th>공시구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>매출액</td>
      <td>2022-03-31</td>
      <td>2958.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>1</th>
      <td>매출총이익</td>
      <td>2022-03-31</td>
      <td>2958.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>2</th>
      <td>판매비와관리비</td>
      <td>2022-03-31</td>
      <td>2766.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인건비</td>
      <td>2022-03-31</td>
      <td>259.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>4</th>
      <td>유무형자산상각비</td>
      <td>2022-03-31</td>
      <td>399.0</td>
      <td>095570</td>
      <td>q</td>
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
      <th>547</th>
      <td>배당금지급(-)</td>
      <td>2022-12-31</td>
      <td>0.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>548</th>
      <td>환율변동효과</td>
      <td>2022-12-31</td>
      <td>-17.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>549</th>
      <td>현금및현금성자산의증가</td>
      <td>2022-12-31</td>
      <td>559.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>550</th>
      <td>기초현금및현금성자산</td>
      <td>2022-12-31</td>
      <td>760.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>551</th>
      <td>기말현금및현금성자산</td>
      <td>2022-12-31</td>
      <td>1319.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
  </tbody>
</table>
<p>467 rows × 5 columns</p>
</div>



#### 연간 재무제표 + 분기 재무제표


```python
# 연간 재무제표 분기 재무제표 합치기
data_fs_bind = pd.concat([data_fs_y_clean, data_fs_q_clean]).reset_index(drop=True)
data_fs_bind
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
      <th>계정</th>
      <th>기준일</th>
      <th>값</th>
      <th>종목코드</th>
      <th>공시구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>매출액</td>
      <td>2019-12-31</td>
      <td>10003.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>1</th>
      <td>매출총이익</td>
      <td>2019-12-31</td>
      <td>10003.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2</th>
      <td>판매비와관리비</td>
      <td>2019-12-31</td>
      <td>9846.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>3</th>
      <td>인건비</td>
      <td>2019-12-31</td>
      <td>852.0</td>
      <td>095570</td>
      <td>y</td>
    </tr>
    <tr>
      <th>4</th>
      <td>유무형자산상각비</td>
      <td>2019-12-31</td>
      <td>109.0</td>
      <td>095570</td>
      <td>y</td>
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
      <th>1006</th>
      <td>배당금지급(-)</td>
      <td>2022-12-31</td>
      <td>0.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>1007</th>
      <td>환율변동효과</td>
      <td>2022-12-31</td>
      <td>-17.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>1008</th>
      <td>현금및현금성자산의증가</td>
      <td>2022-12-31</td>
      <td>559.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>1009</th>
      <td>기초현금및현금성자산</td>
      <td>2022-12-31</td>
      <td>760.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
    <tr>
      <th>1010</th>
      <td>기말현금및현금성자산</td>
      <td>2022-12-31</td>
      <td>1319.0</td>
      <td>095570</td>
      <td>q</td>
    </tr>
  </tbody>
</table>
<p>1011 rows × 5 columns</p>
</div>



### ❍ 여러 종목 재무제표 크롤링 


```python
from tqdm import tqdm
import time
import pandas as pd
df = pd.DataFrame(columns = ['계정','기준일','값','종목코드','공시구분'])
error_list = []
```


```python
for i in tqdm(range(0, len(ticker_list))): 
#for ticker in tqdm(diff):

    # ticker 선택
    ticker = ticker_list['종목코드'][i]
    
    try:     
        url = f'https://comp.fnguide.com/SVO2/ASP/SVD_Finance.asp?pGB=1&gicode=A{ticker}'
        # 데이터 받아오기
        data = pd.read_html(url, displayed_only=False)
        # 연간 재무제표
        data_fs_y = pd.concat([data[0].iloc[:, ~data[0].columns.str.contains('전년동기')],
                       data[2],
                       data[4]
                       ])
        data_fs_y = data_fs_y.rename(columns = {data_fs_y.columns[0] : "계정"})
        # 결산년 찾기
        page_data = rq.get(url)
        page_data_html = BeautifulSoup(page_data.content, 'html.parser')
        fiscal_data = page_data_html.select('div.corp_group1 > h2')
        fiscal_data_text = fiscal_data[1].text
        fiscal_data_text = re.findall('[0-9]+', fiscal_data_text)
        # 결산년에 해당하는 계정만 남기기
        data_fs_y = data_fs_y.loc[:, (data_fs_y.columns == '계정') | (data_fs_y.columns.str[-2:].isin(fiscal_data_text))]
        # 클렌징
        data_fs_y_clean = clean_fs(data_fs_y, ticker, 'y')
        
        # 분기 재무제표
        data_fs_q = pd.concat(
            [data[1].iloc[:, ~data[1].columns.str.contains('전년동기')], data[3], data[5]])
        data_fs_q = data_fs_q.rename(columns={data_fs_q.columns[0]: "계정"})
        # 클렌징
        data_fs_q_clean = clean_fs(data_fs_q, ticker, 'q')
        # 연간/분기 합치기
        data_fs_bind = pd.concat([data_fs_y_clean, data_fs_q_clean])
        # db 합치기
        df = pd.concat([df, data_fs_bind])
    except:
        print(ticker)
        error_list.append(ticker)
        break
    time.sleep(0.01)
```


```python
diff = list(set(ticker_list['종목코드']).symmetric_difference(set(df['종목코드'].unique())))
diff
# 한화갤러리아, 코오롱모빌리티그룹 >>> 신규상장한 기업들 
```




    ['452260', '450140']




```python
#df.to_pickle('./data/data_fs.pickle')
```


```python
data_fs_history = pd.read_pickle('./data/data_fs.pickle')
data_fs_history = pd.concat([data_fs_history, df])
data_fs_history = data_fs_history.drop_duplicates(['계정','기준일','종목코드','공시구분'], keep='last')
data_fs_history = data_fs_history.sort_values(by=['종목코드','기준일','공시구분','계정'])
data_fs_history = data_fs_history.reset_index(drop=True)
data_fs_history
#data_fs_history.to_pickle('./data/data_fs.pickle')
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
      <th>계정</th>
      <th>기준일</th>
      <th>값</th>
      <th>종목코드</th>
      <th>공시구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>(재무활동으로인한현금유출액)</td>
      <td>2019-12-31</td>
      <td>16.0</td>
      <td>000020</td>
      <td>y</td>
    </tr>
    <tr>
      <th>1</th>
      <td>(투자활동으로인한현금유출액)</td>
      <td>2019-12-31</td>
      <td>1080.0</td>
      <td>000020</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2</th>
      <td>(현금유입이없는수익등차감)</td>
      <td>2019-12-31</td>
      <td>99.0</td>
      <td>000020</td>
      <td>y</td>
    </tr>
    <tr>
      <th>3</th>
      <td>*영업에서창출된현금흐름</td>
      <td>2019-12-31</td>
      <td>164.0</td>
      <td>000020</td>
      <td>y</td>
    </tr>
    <tr>
      <th>4</th>
      <td>감가상각비</td>
      <td>2019-12-31</td>
      <td>104.0</td>
      <td>000020</td>
      <td>y</td>
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
      <th>2000595</th>
      <td>판매비와관리비</td>
      <td>2022-12-31</td>
      <td>32.0</td>
      <td>446070</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2000596</th>
      <td>현금및현금성자산</td>
      <td>2022-12-31</td>
      <td>548.0</td>
      <td>446070</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2000597</th>
      <td>현금및현금성자산의증가</td>
      <td>2022-12-31</td>
      <td>-270.0</td>
      <td>446070</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2000598</th>
      <td>현금유출이없는비용등가산</td>
      <td>2022-12-31</td>
      <td>23.0</td>
      <td>446070</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2000599</th>
      <td>환율변동효과</td>
      <td>2022-12-31</td>
      <td>0.0</td>
      <td>446070</td>
      <td>y</td>
    </tr>
  </tbody>
</table>
<p>2000600 rows × 5 columns</p>
</div>




```python

```

## ❒ 가치지표 계산 (data_value)

- "data_value.pickle" : data_fs로부터 TTM 데이터를 구한 후 계산
- PER(주가수익비율), PBR(주가순자산비율), PSR(주가매출비율), PCR(주가현금흐름비율)
- DY(배당률)
- 시가총액 기준일(ticker_list)을 기준으로 계산한 가치지표를 확인가능 (최신 재무제표 활용)

### PER, PBR, PSR, PCR, DY


```python
# 분기 재무제표 불러오기
import pandas as pd
import numpy as np
kor_fs = pd.read_pickle("./data/data_fs.pickle")
kor_fs.drop_duplicates(inplace=True)
cond = (kor_fs.공시구분 == 'q') & (kor_fs['계정'].isin(['당기순이익','자본','영업활동으로인한현금흐름','매출액']))
kor_fs = kor_fs[cond]

# TTM 구하기
kor_fs = kor_fs.sort_values(['종목코드','계정','기준일']).reset_index(drop=True)
kor_fs['ttm'] = kor_fs.groupby(['종목코드','계정'], as_index=False)['값'].rolling(window=4, min_periods=4).sum()['값']
#kor_fs_merge = kor_fs_merge.dropna()

# 자본은 평균으로
kor_fs['ttm'] = np.where(kor_fs['계정']=='자본',
                        kor_fs['ttm']/4, kor_fs['ttm'])
# tail(1) : 최근데이터만 남긴다 
kor_fs = kor_fs.groupby(['계정','종목코드']).tail(1).reset_index(drop=True)

# 합치기
kor_fs_merge = kor_fs[['계정','종목코드','ttm']].merge(ticker_list[['종목코드','시가총액','기준일']], on='종목코드')

# 시가총액 : 원단위를 억원으로 변경
kor_fs_merge['시가총액'] = kor_fs_merge['시가총액']/10**8

# 지표 계산

cond = kor_fs_merge['ttm']==0
kor_fs_merge.loc[cond, 'ttm'] = 0.1 #0인 값 처리

kor_fs_merge['value'] = kor_fs_merge['시가총액'] / kor_fs_merge['ttm']
kor_fs_merge['value'] = kor_fs_merge['value'].astype(float).round(4)
kor_fs_merge['지표'] = np.where(
    kor_fs_merge['계정'] == '매출액','PSR',
    np.where(
        kor_fs_merge['계정'] == '영업활동으로인한현금흐름','PCR',
        np.where(
            kor_fs_merge['계정'] == '자본', 'PBR',
            np.where(
                kor_fs_merge['계정'] == '당기순이익', 'PER', None))))

# value 중에 nan 이나 inf 를 None으로 변경하기, SQL에 nan, inf 저장 안됨
kor_fs_merge.rename(columns = {"value":"값"}, inplace=True)
kor_fs_merge = kor_fs_merge[['종목코드','기준일','지표','값']]
kor_fs_merge = kor_fs_merge.replace([np.inf, -np.inf, np.nan], None)


### 배당수익률
ticker_list['값'] = ticker_list['주당배당금'] / ticker_list['종가']
ticker_list['값'] = ticker_list['값'].astype(np.float64).round(4)
ticker_list['지표'] = "DY"

dy_list = ticker_list[['종목코드','기준일','지표','값']]
dy_list = dy_list.replace([np.inf, -np.inf, np.nan], None)
dy_list = dy_list[dy_list['값'] != 0] #배당을 0으로 주는 경우는 제외


# PER, PBR, PSR, PCR, DY 합치기
value_df = pd.concat([kor_fs_merge, dy_list]).reset_index(drop=True)
value_df
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
      <th>기준일</th>
      <th>지표</th>
      <th>값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PER</td>
      <td>10.7459</td>
    </tr>
    <tr>
      <th>1</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PSR</td>
      <td>0.6817</td>
    </tr>
    <tr>
      <th>2</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PCR</td>
      <td>7.9219</td>
    </tr>
    <tr>
      <th>3</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PBR</td>
      <td>0.629</td>
    </tr>
    <tr>
      <th>4</th>
      <td>000040</td>
      <td>20230331</td>
      <td>PER</td>
      <td>-3.7272</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>10299</th>
      <td>024060</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0179</td>
    </tr>
    <tr>
      <th>10300</th>
      <td>010240</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0326</td>
    </tr>
    <tr>
      <th>10301</th>
      <td>189980</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.011</td>
    </tr>
    <tr>
      <th>10302</th>
      <td>037440</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0164</td>
    </tr>
    <tr>
      <th>10303</th>
      <td>238490</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0141</td>
    </tr>
  </tbody>
</table>
<p>10304 rows × 4 columns</p>
</div>



여기서의 기준일은 시가총액 기준일 
- 재무제표는 2022.12.31 결산기준이며, 시가총액은 2023.03.31 기준 > 기준일 시가총액 기준


```python
# value_df.to_pickle("./data/data_value.pickle")
```


```python
kor_fs.head(1)
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
      <th>계정</th>
      <th>기준일</th>
      <th>값</th>
      <th>종목코드</th>
      <th>공시구분</th>
      <th>ttm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>당기순이익</td>
      <td>2022-12-31</td>
      <td>25.0</td>
      <td>000020</td>
      <td>q</td>
      <td>216.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
kor_fs_merge.head(1)
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
      <th>기준일</th>
      <th>지표</th>
      <th>값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PER</td>
      <td>10.7459</td>
    </tr>
  </tbody>
</table>
</div>




```python
value_df.head(1)
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
      <th>기준일</th>
      <th>지표</th>
      <th>값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PER</td>
      <td>10.7459</td>
    </tr>
  </tbody>
</table>
</div>




```python
temp = pd.read_pickle('./data/data_value.pickle')
temp = pd.concat([temp, value_df]).drop_duplicates(['종목코드','기준일','지표'], keep='last').reset_index(drop=True)
temp
#temp.to_pickle("./data/data_value.pickle")
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
      <th>기준일</th>
      <th>지표</th>
      <th>값</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PER</td>
      <td>10.7459</td>
    </tr>
    <tr>
      <th>1</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PSR</td>
      <td>0.6817</td>
    </tr>
    <tr>
      <th>2</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PCR</td>
      <td>7.9219</td>
    </tr>
    <tr>
      <th>3</th>
      <td>000020</td>
      <td>20230331</td>
      <td>PBR</td>
      <td>0.629</td>
    </tr>
    <tr>
      <th>4</th>
      <td>000040</td>
      <td>20230331</td>
      <td>PER</td>
      <td>-3.7272</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>10299</th>
      <td>024060</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0179</td>
    </tr>
    <tr>
      <th>10300</th>
      <td>010240</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0326</td>
    </tr>
    <tr>
      <th>10301</th>
      <td>189980</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.011</td>
    </tr>
    <tr>
      <th>10302</th>
      <td>037440</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0164</td>
    </tr>
    <tr>
      <th>10303</th>
      <td>238490</td>
      <td>20230331</td>
      <td>DY</td>
      <td>0.0141</td>
    </tr>
  </tbody>
</table>
<p>10304 rows × 4 columns</p>
</div>



## ❒ 상장폐지종목 (delistSymbol)


```python
import FinanceDataReader as fdr
from pykrx import stock
import pandas_datareader.data as pdr
import yfinance as yf

# 가장 최근 영업일의 시장별 종목리스트를 가져옴
stocks = fdr.StockListing('KRX') # 코스피, 코스닥, 코넥스 전체
stocks
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
      <th>Code</th>
      <th>ISU_CD</th>
      <th>Name</th>
      <th>Market</th>
      <th>Dept</th>
      <th>Close</th>
      <th>ChangeCode</th>
      <th>Changes</th>
      <th>ChagesRatio</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Volume</th>
      <th>Amount</th>
      <th>Marcap</th>
      <th>Stocks</th>
      <th>MarketId</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>005930</td>
      <td>KR7005930003</td>
      <td>삼성전자</td>
      <td>KOSPI</td>
      <td></td>
      <td>65000</td>
      <td>1</td>
      <td>2700</td>
      <td>4.33</td>
      <td>63800</td>
      <td>65200</td>
      <td>63800</td>
      <td>27476120</td>
      <td>1778208943700</td>
      <td>388035865750000</td>
      <td>5969782550</td>
      <td>STK</td>
    </tr>
    <tr>
      <th>1</th>
      <td>373220</td>
      <td>KR7373220003</td>
      <td>LG에너지솔루션</td>
      <td>KOSPI</td>
      <td></td>
      <td>580000</td>
      <td>3</td>
      <td>0</td>
      <td>0.00</td>
      <td>581000</td>
      <td>590000</td>
      <td>579000</td>
      <td>366225</td>
      <td>213290884200</td>
      <td>135720000000000</td>
      <td>234000000</td>
      <td>STK</td>
    </tr>
    <tr>
      <th>2</th>
      <td>000660</td>
      <td>KR7000660001</td>
      <td>SK하이닉스</td>
      <td>KOSPI</td>
      <td></td>
      <td>89100</td>
      <td>1</td>
      <td>5300</td>
      <td>6.32</td>
      <td>87900</td>
      <td>89400</td>
      <td>86000</td>
      <td>10867359</td>
      <td>959641473100</td>
      <td>64865010721500</td>
      <td>728002365</td>
      <td>STK</td>
    </tr>
    <tr>
      <th>3</th>
      <td>207940</td>
      <td>KR7207940008</td>
      <td>삼성바이오로직스</td>
      <td>KOSPI</td>
      <td></td>
      <td>796000</td>
      <td>2</td>
      <td>-9000</td>
      <td>-1.12</td>
      <td>804000</td>
      <td>805000</td>
      <td>792000</td>
      <td>61342</td>
      <td>48882068000</td>
      <td>56654504000000</td>
      <td>71174000</td>
      <td>STK</td>
    </tr>
    <tr>
      <th>4</th>
      <td>006400</td>
      <td>KR7006400006</td>
      <td>삼성SDI</td>
      <td>KOSPI</td>
      <td></td>
      <td>738000</td>
      <td>2</td>
      <td>-7000</td>
      <td>-0.94</td>
      <td>742000</td>
      <td>747000</td>
      <td>731000</td>
      <td>259085</td>
      <td>191613326000</td>
      <td>50748223140000</td>
      <td>68764530</td>
      <td>STK</td>
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
    </tr>
    <tr>
      <th>2709</th>
      <td>308700</td>
      <td>KR7308700004</td>
      <td>테크엔</td>
      <td>KONEX</td>
      <td>일반기업부</td>
      <td>363</td>
      <td>1</td>
      <td>43</td>
      <td>13.44</td>
      <td>367</td>
      <td>367</td>
      <td>363</td>
      <td>101</td>
      <td>36667</td>
      <td>1452000000</td>
      <td>4000000</td>
      <td>KNX</td>
    </tr>
    <tr>
      <th>2710</th>
      <td>322190</td>
      <td>KR7322190000</td>
      <td>베른</td>
      <td>KONEX</td>
      <td>일반기업부</td>
      <td>118</td>
      <td>5</td>
      <td>-20</td>
      <td>-14.49</td>
      <td>118</td>
      <td>137</td>
      <td>118</td>
      <td>532</td>
      <td>71838</td>
      <td>1053173246</td>
      <td>8925197</td>
      <td>KNX</td>
    </tr>
    <tr>
      <th>2711</th>
      <td>058420</td>
      <td>KR7058420001</td>
      <td>제이웨이</td>
      <td>KOSDAQ</td>
      <td>투자주의환기종목(소속부없음)</td>
      <td>215</td>
      <td>2</td>
      <td>-83</td>
      <td>-27.85</td>
      <td>260</td>
      <td>293</td>
      <td>210</td>
      <td>3697643</td>
      <td>845032403</td>
      <td>776330385</td>
      <td>3610839</td>
      <td>KSQ</td>
    </tr>
    <tr>
      <th>2712</th>
      <td>271850</td>
      <td>KR7271850000</td>
      <td>다이오진</td>
      <td>KONEX</td>
      <td>일반기업부</td>
      <td>125</td>
      <td>2</td>
      <td>-79</td>
      <td>-38.73</td>
      <td>249</td>
      <td>249</td>
      <td>125</td>
      <td>121711</td>
      <td>19095800</td>
      <td>400791625</td>
      <td>3206333</td>
      <td>KNX</td>
    </tr>
    <tr>
      <th>2713</th>
      <td>267060</td>
      <td>KR7267060002</td>
      <td>명진홀딩스</td>
      <td>KONEX</td>
      <td>일반기업부</td>
      <td>30</td>
      <td>2</td>
      <td>-151</td>
      <td>-83.43</td>
      <td>53</td>
      <td>53</td>
      <td>25</td>
      <td>1283696</td>
      <td>44440461</td>
      <td>274254120</td>
      <td>9141804</td>
      <td>KNX</td>
    </tr>
  </tbody>
</table>
<p>2714 rows × 17 columns</p>
</div>




```python
# KRX stock delisting symbol list 상장폐지 종목 전체 리스트
krx_delisting = fdr.StockListing('KRX-DELISTING')
krx_delisting
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
      <th>Symbol</th>
      <th>Name</th>
      <th>Market</th>
      <th>SecuGroup</th>
      <th>Kind</th>
      <th>ListingDate</th>
      <th>DelistingDate</th>
      <th>Reason</th>
      <th>ArrantEnforceDate</th>
      <th>ArrantEndDate</th>
      <th>Industry</th>
      <th>ParValue</th>
      <th>ListingShares</th>
      <th>ToSymbol</th>
      <th>ToName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>06031012</td>
      <td>3S R</td>
      <td>KOSDAQ</td>
      <td>신주인수권증서</td>
      <td>보통주</td>
      <td>2012-05-14</td>
      <td>2012-05-21</td>
      <td></td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>0.0</td>
      <td>1194422.0</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>00684014</td>
      <td>AK홀딩스8R</td>
      <td>KOSPI</td>
      <td>신주인수권증서</td>
      <td>보통주</td>
      <td>2014-07-28</td>
      <td>2014-08-04</td>
      <td></td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>0.0</td>
      <td>1278299.0</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>13893015</td>
      <td>BNK금융지주 8R</td>
      <td>KOSPI</td>
      <td>신주인수권증서</td>
      <td>보통주</td>
      <td>2015-12-24</td>
      <td>2016-01-05</td>
      <td></td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>0.0</td>
      <td>55969410.0</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>13893014</td>
      <td>BS금융지주5R</td>
      <td>KOSPI</td>
      <td>신주인수권증서</td>
      <td>보통주</td>
      <td>2014-06-18</td>
      <td>2014-06-25</td>
      <td></td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>0.0</td>
      <td>32791220.0</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>03204017</td>
      <td>C&amp;S자산관리 34R</td>
      <td>KOSDAQ</td>
      <td>신주인수권증서</td>
      <td>보통주</td>
      <td>2017-02-02</td>
      <td>2017-02-09</td>
      <td></td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>0.0</td>
      <td>3995063.0</td>
      <td></td>
      <td></td>
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
    </tr>
    <tr>
      <th>3487</th>
      <td>034370</td>
      <td>럭키소재</td>
      <td>KOSPI</td>
      <td>주권</td>
      <td></td>
      <td>1979-06-23</td>
      <td>1991-11-11</td>
      <td>해산 사유 발생</td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>NaN</td>
      <td>NaN</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3488</th>
      <td>028460</td>
      <td>태평양건설</td>
      <td>KOSPI</td>
      <td>주권</td>
      <td></td>
      <td>1977-06-09</td>
      <td>1991-10-05</td>
      <td>영업활동정지 6월 계속</td>
      <td>1991-08-28</td>
      <td>1991-10-04</td>
      <td></td>
      <td>NaN</td>
      <td>NaN</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3489</th>
      <td>028450</td>
      <td>금성투자금융</td>
      <td>KOSPI</td>
      <td>주권</td>
      <td></td>
      <td>1986-08-29</td>
      <td>1991-09-02</td>
      <td>해산 사유 발생</td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>NaN</td>
      <td>NaN</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3490</th>
      <td>028440</td>
      <td>삼화</td>
      <td>KOSPI</td>
      <td>주권</td>
      <td></td>
      <td>1974-06-12</td>
      <td>1991-07-12</td>
      <td>감사의견 의견거절</td>
      <td>1991-06-05</td>
      <td>1991-07-11</td>
      <td></td>
      <td>NaN</td>
      <td>NaN</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3491</th>
      <td>029260</td>
      <td>금성전기</td>
      <td>KOSPI</td>
      <td>주권</td>
      <td></td>
      <td>1976-06-29</td>
      <td>1991-06-13</td>
      <td>해산 사유 발생</td>
      <td>NaT</td>
      <td>NaT</td>
      <td></td>
      <td>NaN</td>
      <td>NaN</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<p>3492 rows × 15 columns</p>
</div>




```python
import datetime
import pandas as pd
cond = (krx_delisting['DelistingDate'] > "2010-01-01")&(krx_delisting['Kind']=='보통주')
temp = krx_delisting[cond].sort_values(by='DelistingDate', ascending=False)
temp = [i for i in temp.Symbol.tolist() if len(i)==6]
```


```python
# pd.DataFrame(temp, columns=['상장폐지종목코드']).to_pickle("./data/delistSymbol.pickle")
```


```python
temp = pd.read_pickle("./data/delistSymbol.pickle")
temp
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
      <th>상장폐지종목코드</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>267810</td>
    </tr>
    <tr>
      <th>1</th>
      <td>056000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>387310</td>
    </tr>
    <tr>
      <th>3</th>
      <td>353070</td>
    </tr>
    <tr>
      <th>4</th>
      <td>397500</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>696</th>
      <td>037510</td>
    </tr>
    <tr>
      <th>697</th>
      <td>083120</td>
    </tr>
    <tr>
      <th>698</th>
      <td>000610</td>
    </tr>
    <tr>
      <th>699</th>
      <td>015940</td>
    </tr>
    <tr>
      <th>700</th>
      <td>045820</td>
    </tr>
  </tbody>
</table>
<p>701 rows × 1 columns</p>
</div>

## ❒ 재무제표 - 다트 (API Key 필요)

- openapi를 활용해서 재무정보 다운로드

https://opendart.fss.or.kr/


```python
# !pip install keyring --quiet
```

### ❍ 다트 티커리스트 다운받기


```python
import keyring
import requests
from io import BytesIO
import zipfile

#keyring.set_password('dart_api_key', 'User Name', 'Password') # System , 본인이름, 발급받은 API키
keyring.set_password('dart_api_key', '###', '$$$$$$$$$$$$$$$$$$$$') # System , 본인이름, 발급받은 API키
api_key = keyring.get_password('dart_api_key', '###')

codezip_url = f'''https://opendart.fss.or.kr/api/corpCode.xml?crtfc_key={api_key}'''
codezip_data = requests.get(codezip_url)
codezip_data.headers
```




    {'Cache-Control': 'no-cache, no-store', 'Connection': 'keep-alive', 'Set-Cookie': 'WMONID=-Tq_CVAmWN2; Expires=Sun, 07-Apr-2024 20:6:17 GMT; Path=/', 'Pragma': 'no-cache', 'Expires': '0', 'Content-Transfer-Encoding': 'binary', 'Content-Disposition': ': attachment; filename=CORPCODE.zip', 'Date': 'Sat, 08 Apr 2023 11:06:17 GMT', 'Content-Type': 'application/x-msdownload;charset=UTF-8', 'Content-Length': '1657392'}




```python
codezip_data.headers['Content-Disposition']
```




    ': attachment; filename=CORPCODE.zip'




```python
codezip_file = zipfile.ZipFile(BytesIO(codezip_data.content)) # 바이너리스트림 형태로 변경 > 집파일 풀기 
codezip_file.namelist() # 어떤 파일이 있는지
```




    ['CORPCODE.xml']




```python
code_data = codezip_file.read("CORPCODE.xml").decode("utf-8") # utf-8로 디코딩 #한글이 보이기 시작
```


```python
# xml 형태를 dictionary 형태로 변경
import xmltodict
import json
import pandas as pd

data_odict = xmltodict.parse(code_data)
data_dict = json.loads(json.dumps(data_odict))
data = data_dict.get('result').get('list')
corp_list = pd.DataFrame(data)
corp_list
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
      <th>corp_code</th>
      <th>corp_name</th>
      <th>stock_code</th>
      <th>modify_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00434003</td>
      <td>다코</td>
      <td>None</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>1</th>
      <td>00434456</td>
      <td>일산약품</td>
      <td>None</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>2</th>
      <td>00430964</td>
      <td>굿앤엘에스</td>
      <td>None</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>3</th>
      <td>00432403</td>
      <td>한라판지</td>
      <td>None</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00388953</td>
      <td>크레디피아제이십오차유동화전문회사</td>
      <td>None</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>98028</th>
      <td>00646510</td>
      <td>미래에셋파트너스사호사모투자전문회사</td>
      <td>None</td>
      <td>20230228</td>
    </tr>
    <tr>
      <th>98029</th>
      <td>01184822</td>
      <td>미래에셋파트너스9호사모투자</td>
      <td>None</td>
      <td>20230228</td>
    </tr>
    <tr>
      <th>98030</th>
      <td>00755252</td>
      <td>시니안</td>
      <td>None</td>
      <td>20230228</td>
    </tr>
    <tr>
      <th>98031</th>
      <td>00227582</td>
      <td>청호나이스</td>
      <td>None</td>
      <td>20230228</td>
    </tr>
    <tr>
      <th>98032</th>
      <td>01615845</td>
      <td>메타버스월드</td>
      <td>None</td>
      <td>20230228</td>
    </tr>
  </tbody>
</table>
<p>98033 rows × 4 columns</p>
</div>




```python
corp_list = corp_list[~corp_list.stock_code.isin([None])].reset_index(drop=True) # None은 비상장이므로 필터링
corp_list
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
      <th>corp_code</th>
      <th>corp_name</th>
      <th>stock_code</th>
      <th>modify_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00260985</td>
      <td>한빛네트</td>
      <td>036720</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>1</th>
      <td>00264529</td>
      <td>엔플렉스</td>
      <td>040130</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>2</th>
      <td>00358545</td>
      <td>동서정보기술</td>
      <td>055000</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>3</th>
      <td>00231567</td>
      <td>애드모바일</td>
      <td>032600</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00247939</td>
      <td>씨모스</td>
      <td>037600</td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3560</th>
      <td>00413417</td>
      <td>우리손에프앤지</td>
      <td>073560</td>
      <td>20230403</td>
    </tr>
    <tr>
      <th>3561</th>
      <td>00440712</td>
      <td>어반리튬</td>
      <td>073570</td>
      <td>20230403</td>
    </tr>
    <tr>
      <th>3562</th>
      <td>00483735</td>
      <td>해성옵틱스</td>
      <td>076610</td>
      <td>20230403</td>
    </tr>
    <tr>
      <th>3563</th>
      <td>00516246</td>
      <td>알에프세미</td>
      <td>096610</td>
      <td>20230403</td>
    </tr>
    <tr>
      <th>3564</th>
      <td>00525679</td>
      <td>차바이오텍</td>
      <td>085660</td>
      <td>20230403</td>
    </tr>
  </tbody>
</table>
<p>3565 rows × 4 columns</p>
</div>




```python
# corp_list.to_pickle("./data/tickers_dart.pickle")
```

### ❍ 공시자료 다운로드

오픈다트 > 개발가이드 > 공시정보 > 공시검색


```python
# 전체 종목의 최근 100개의 공시에 해당하는 내역을 받아오게 됨

from datetime import date
from dateutil.relativedelta import relativedelta

bgn_date = (date.today() + relativedelta(days=-30)).strftime("%Y%m%d")
end_date = (date.today()).strftime("%Y%m%d")

notice_url = f'''https://opendart.fss.or.kr/api/list.json?crtfc_key={api_key}&bgn_de={bgn_date}&end_de={end_date}&page_no=1&page_count=100''' # 최대 100까지 가능

notice_data = requests.get(notice_url)
notice_data_df = notice_data.json().get('list')
notice_data_df = pd.DataFrame(notice_data_df)
```


```python
notice_data_df
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
      <th>corp_code</th>
      <th>corp_name</th>
      <th>stock_code</th>
      <th>corp_cls</th>
      <th>report_nm</th>
      <th>rcept_no</th>
      <th>flr_nm</th>
      <th>rcept_dt</th>
      <th>rm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01101722</td>
      <td>한송네오텍</td>
      <td>226440</td>
      <td>K</td>
      <td>[기재정정]기타시장안내(상장폐지 관련)</td>
      <td>20230407901312</td>
      <td>코스닥시장본부</td>
      <td>20230407</td>
      <td>코</td>
    </tr>
    <tr>
      <th>1</th>
      <td>00610490</td>
      <td>비디아이</td>
      <td>148140</td>
      <td>K</td>
      <td>[기재정정]주권매매거래정지기간변경(상장폐지 사유 해소, 사업보고서 미제출)</td>
      <td>20230407901311</td>
      <td>코스닥시장본부</td>
      <td>20230407</td>
      <td>코</td>
    </tr>
    <tr>
      <th>2</th>
      <td>00610490</td>
      <td>비디아이</td>
      <td>148140</td>
      <td>K</td>
      <td>기타시장안내(상장폐지 관련)</td>
      <td>20230407901308</td>
      <td>코스닥시장본부</td>
      <td>20230407</td>
      <td>코</td>
    </tr>
    <tr>
      <th>3</th>
      <td>00962393</td>
      <td>포인트모바일</td>
      <td>318020</td>
      <td>K</td>
      <td>[기재정정]기타시장안내(상장폐지 관련)</td>
      <td>20230407901309</td>
      <td>코스닥시장본부</td>
      <td>20230407</td>
      <td>코</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00610490</td>
      <td>비디아이</td>
      <td>148140</td>
      <td>K</td>
      <td>주권매매거래정지기간변경(상장폐지 사유 해소)</td>
      <td>20230407901306</td>
      <td>코스닥시장본부</td>
      <td>20230407</td>
      <td>코정</td>
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
    </tr>
    <tr>
      <th>95</th>
      <td>01653128</td>
      <td>네오플랜</td>
      <td></td>
      <td>E</td>
      <td>감사보고서 (2022.12)</td>
      <td>20230407003488</td>
      <td>현대회계법인</td>
      <td>20230407</td>
      <td></td>
    </tr>
    <tr>
      <th>96</th>
      <td>01685774</td>
      <td>두리랜드</td>
      <td></td>
      <td>E</td>
      <td>감사보고서 (2022.12)</td>
      <td>20230407003487</td>
      <td>인덕회계법인</td>
      <td>20230407</td>
      <td></td>
    </tr>
    <tr>
      <th>97</th>
      <td>00440712</td>
      <td>어반리튬</td>
      <td>073570</td>
      <td>K</td>
      <td>주식등의대량보유상황보고서(일반)</td>
      <td>20230407003486</td>
      <td>리튬인사이트</td>
      <td>20230407</td>
      <td></td>
    </tr>
    <tr>
      <th>98</th>
      <td>00990262</td>
      <td>피노텍</td>
      <td>150440</td>
      <td>N</td>
      <td>사업보고서 (2022.12)</td>
      <td>20230407003485</td>
      <td>피노텍</td>
      <td>20230407</td>
      <td></td>
    </tr>
    <tr>
      <th>99</th>
      <td>01476219</td>
      <td>코스텍시스</td>
      <td>355150</td>
      <td>K</td>
      <td>주식등의대량보유상황보고서(약식)</td>
      <td>20230407003484</td>
      <td>유비쿼스인베스트먼트</td>
      <td>20230407</td>
      <td></td>
    </tr>
  </tbody>
</table>
<p>100 rows × 9 columns</p>
</div>




```python
# 원하는 종목의 공시자료만 받아오기
corp_code = '00126380' # 종목티커(stock_code)와 별도로 종목코드(corp_code)가 있음

notice_url_ss = f'''https://opendart.fss.or.kr/api/list.json?crtfc_key={api_key}&corp_code={corp_code}&bgn_de={bgn_date}&end_de={end_date}&page_no=1&page_count=100''' # 최대 100까지 가능
notice_data_ss = requests.get(notice_url_ss)
notice_data_ss_df = notice_data_ss.json().get('list')
notice_data_ss_df = pd.DataFrame(notice_data_ss_df)
notice_data_ss_df.tail()
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
      <th>corp_code</th>
      <th>corp_name</th>
      <th>stock_code</th>
      <th>corp_cls</th>
      <th>report_nm</th>
      <th>rcept_no</th>
      <th>flr_nm</th>
      <th>rcept_dt</th>
      <th>rm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>005930</td>
      <td>Y</td>
      <td>연결재무제표기준영업(잠정)실적(공정공시)</td>
      <td>20230407800208</td>
      <td>삼성전자</td>
      <td>20230407</td>
      <td>유</td>
    </tr>
    <tr>
      <th>1</th>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>005930</td>
      <td>Y</td>
      <td>임원ㆍ주요주주특정증권등소유상황보고서</td>
      <td>20230406002362</td>
      <td>김이태</td>
      <td>20230406</td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>005930</td>
      <td>Y</td>
      <td>임원ㆍ주요주주특정증권등소유상황보고서</td>
      <td>20230324000884</td>
      <td>경계현</td>
      <td>20230324</td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>005930</td>
      <td>Y</td>
      <td>임원ㆍ주요주주특정증권등소유상황보고서</td>
      <td>20230316000646</td>
      <td>안재용</td>
      <td>20230316</td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>005930</td>
      <td>Y</td>
      <td>정기주주총회결과</td>
      <td>20230315802179</td>
      <td>삼성전자</td>
      <td>20230315</td>
      <td>유</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 공시번호를 이용해서 rcept_no 

notice_url_exam = notice_data_ss_df.loc[0, 'rcept_no']
notice_dart_url = f'https://dart.fss.or.kr/dsaf001/main.do?rcpNo={notice_url_exam}'
notice_dart_url
```




    'https://dart.fss.or.kr/dsaf001/main.do?rcpNo=20230407800208'




```python
# 개발가이드 > 사업보고서 주요정보 > 배당에 관한 사항 # https://opendart.fss.or.kr/guide/detail.do?apiGrpCd=DS002&apiId=2019005

corp_code = '00126380'
bsns_year = '2022'
report_code = '11011'

url_div = f'''https://opendart.fss.or.kr/api/alotMatter.json?crtfc_key={api_key}&corp_code={corp_code}&bsns_year={bsns_year}&reprt_code={report_code}'''

div_data_ss = requests.get(url_div)
div_data_ss_df = div_data_ss.json().get('list')
div_data_ss_df = pd.DataFrame(div_data_ss_df)
div_data_ss_df
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
      <th>rcept_no</th>
      <th>corp_cls</th>
      <th>corp_code</th>
      <th>corp_name</th>
      <th>se</th>
      <th>thstrm</th>
      <th>frmtrm</th>
      <th>lwfr</th>
      <th>stock_knd</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주당액면가액(원)</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>(연결)당기순이익(백만원)</td>
      <td>54,730,018</td>
      <td>39,243,791</td>
      <td>26,090,846</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>(별도)당기순이익(백만원)</td>
      <td>25,418,778</td>
      <td>30,970,954</td>
      <td>15,615,018</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>(연결)주당순이익(원)</td>
      <td>8,057</td>
      <td>5,777</td>
      <td>3,841</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>현금배당금총액(백만원)</td>
      <td>9,809,438</td>
      <td>9,809,438</td>
      <td>20,338,075</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주식배당금총액(백만원)</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>(연결)현금배당성향(%)</td>
      <td>17.90</td>
      <td>25.00</td>
      <td>78.00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>현금배당수익률(%)</td>
      <td>2.50</td>
      <td>1.80</td>
      <td>4.00</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>8</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>현금배당수익률(%)</td>
      <td>2.70</td>
      <td>2.00</td>
      <td>4.20</td>
      <td>우선주</td>
    </tr>
    <tr>
      <th>9</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주식배당수익률(%)</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>10</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주식배당수익률(%)</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>우선주</td>
    </tr>
    <tr>
      <th>11</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주당 현금배당금(원)</td>
      <td>1,444</td>
      <td>1,444</td>
      <td>2,994</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>12</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주당 현금배당금(원)</td>
      <td>1,445</td>
      <td>1,445</td>
      <td>2,995</td>
      <td>우선주</td>
    </tr>
    <tr>
      <th>13</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주당 주식배당(주)</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>보통주</td>
    </tr>
    <tr>
      <th>14</th>
      <td>20230307000542</td>
      <td>Y</td>
      <td>00126380</td>
      <td>삼성전자</td>
      <td>주당 주식배당(주)</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>우선주</td>
    </tr>
  </tbody>
</table>
</div>




```python
# API를 이용해서 전종목의 재무정보 다운받기
# 개발가이드 > 상장기업재무정보 > 단일회사 전체 재무제표 (금융회사 제외)
# https://opendart.fss.or.kr/guide/detail.do?apiGrpCd=DS003&apiId=2019020

url_fs = f'''https://opendart.fss.or.kr/api/fnlttSinglAcntAll.json?crtfc_key={api_key}
&corp_code={corp_code}&bsns_year={bsns_year}&reprt_code={report_code}&fs_div=OFS''' #OFS : 연결

fs_data_ss = requests.get(url_fs)
fs_data_ss_df = fs_data_ss.json().get('list')
fs_data_ss_df = pd.DataFrame(fs_data_ss_df)
fs_data_ss_df
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
      <th>rcept_no</th>
      <th>reprt_code</th>
      <th>bsns_year</th>
      <th>corp_code</th>
      <th>sj_div</th>
      <th>sj_nm</th>
      <th>account_id</th>
      <th>account_nm</th>
      <th>account_detail</th>
      <th>thstrm_nm</th>
      <th>thstrm_amount</th>
      <th>frmtrm_nm</th>
      <th>frmtrm_amount</th>
      <th>bfefrmtrm_nm</th>
      <th>bfefrmtrm_amount</th>
      <th>ord</th>
      <th>currency</th>
      <th>thstrm_add_amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>BS</td>
      <td>재무상태표</td>
      <td>ifrs-full_CurrentAssets</td>
      <td>유동자산</td>
      <td>-</td>
      <td>제 54 기</td>
      <td>59062658000000</td>
      <td>제 53 기</td>
      <td>73553416000000</td>
      <td>제 52 기</td>
      <td>73798549000000</td>
      <td>1</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>BS</td>
      <td>재무상태표</td>
      <td>ifrs-full_CashAndCashEquivalents</td>
      <td>현금및현금성자산</td>
      <td>-</td>
      <td>제 54 기</td>
      <td>3921593000000</td>
      <td>제 53 기</td>
      <td>3918872000000</td>
      <td>제 52 기</td>
      <td>989045000000</td>
      <td>2</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>BS</td>
      <td>재무상태표</td>
      <td>dart_ShortTermDepositsNotClassifiedAsCashEquiv...</td>
      <td>단기금융상품</td>
      <td>-</td>
      <td>제 54 기</td>
      <td>137000000</td>
      <td>제 53 기</td>
      <td>15000576000000</td>
      <td>제 52 기</td>
      <td>29101284000000</td>
      <td>3</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>BS</td>
      <td>재무상태표</td>
      <td>dart_ShortTermTradeReceivable</td>
      <td>매출채권</td>
      <td>-</td>
      <td>제 54 기</td>
      <td>20503223000000</td>
      <td>제 53 기</td>
      <td>33088247000000</td>
      <td>제 52 기</td>
      <td>24736740000000</td>
      <td>4</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>BS</td>
      <td>재무상태표</td>
      <td>dart_ShortTermOtherReceivables</td>
      <td>미수금</td>
      <td>-</td>
      <td>제 54 기</td>
      <td>2925006000000</td>
      <td>제 53 기</td>
      <td>1832488000000</td>
      <td>제 52 기</td>
      <td>1898583000000</td>
      <td>5</td>
      <td>KRW</td>
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
    </tr>
    <tr>
      <th>109</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>SCE</td>
      <td>자본변동표</td>
      <td>ifrs-full_Equity</td>
      <td>기말자본</td>
      <td>별도재무제표 [member]</td>
      <td>제 54 기</td>
      <td>209416191000000</td>
      <td>제 53 기</td>
      <td>193193732000000</td>
      <td>제 52 기</td>
      <td>183316724000000</td>
      <td>6</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>110</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>SCE</td>
      <td>자본변동표</td>
      <td>ifrs-full_Equity</td>
      <td>기말자본</td>
      <td>자본 [member]|기타자본항목</td>
      <td>제 54 기</td>
      <td>-273232000000</td>
      <td>제 53 기</td>
      <td>-882010000000</td>
      <td>제 52 기</td>
      <td>-268785000000</td>
      <td>6</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>111</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>SCE</td>
      <td>자본변동표</td>
      <td>ifrs-full_Equity</td>
      <td>기말자본</td>
      <td>자본 [member]|이익잉여금 [member]</td>
      <td>제 54 기</td>
      <td>204388016000000</td>
      <td>제 53 기</td>
      <td>188774335000000</td>
      <td>제 52 기</td>
      <td>178284102000000</td>
      <td>6</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>112</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>SCE</td>
      <td>자본변동표</td>
      <td>ifrs-full_Equity</td>
      <td>기말자본</td>
      <td>자본 [member]|자본금 [member]</td>
      <td>제 54 기</td>
      <td>897514000000</td>
      <td>제 53 기</td>
      <td>897514000000</td>
      <td>제 52 기</td>
      <td>897514000000</td>
      <td>6</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>113</th>
      <td>20230307000542</td>
      <td>11011</td>
      <td>2022</td>
      <td>00126380</td>
      <td>SCE</td>
      <td>자본변동표</td>
      <td>ifrs-full_Equity</td>
      <td>기말자본</td>
      <td>자본 [member]|주식발행초과금</td>
      <td>제 54 기</td>
      <td>4403893000000</td>
      <td>제 53 기</td>
      <td>4403893000000</td>
      <td>제 52 기</td>
      <td>4403893000000</td>
      <td>6</td>
      <td>KRW</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>114 rows × 18 columns</p>
</div>



## ❒ FRED 장단기 금리차, 공포탐욕지수

FRED : Federal Reserve Economic Data

대표적인 보조지표 : 장단기 금리차 > 장기로 돈을 빌리는 것이 단기로 빌리는 것보다 위험하므로, 장기 프리미엄이 붙게 됨. 주요 금융시장 및 경제지표 가운데 경기침체에 대한 예측력 좋다고 알려짐 >> 경기 침체 이전에 장단기 금리차가 역전됨

https://fred.stlouisfed.org/series/T10Y2Y


```python
import pandas_datareader as web
import pandas as pd

t10y2y = web.DataReader("T10Y2Y", 'fred', start='1990-01-01')
t10y3m = web.DataReader("T10Y3M", 'fred', start='1990-01-01')

rate_diff = pd.concat([t10y2y, t10y3m], axis=1)
rate_diff.columns = ['10Y-2Y', '10Y-3M']
rate_diff.tail()
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
      <th>10Y-2Y</th>
      <th>10Y-3M</th>
    </tr>
    <tr>
      <th>DATE</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2023-04-03</th>
      <td>-0.54</td>
      <td>-1.47</td>
    </tr>
    <tr>
      <th>2023-04-04</th>
      <td>-0.49</td>
      <td>-1.53</td>
    </tr>
    <tr>
      <th>2023-04-05</th>
      <td>-0.49</td>
      <td>-1.56</td>
    </tr>
    <tr>
      <th>2023-04-06</th>
      <td>-0.52</td>
      <td>-1.61</td>
    </tr>
    <tr>
      <th>2023-04-07</th>
      <td>-0.58</td>
      <td>-1.56</td>
    </tr>
  </tbody>
</table>
</div>




```python
import matplotlib.pyplot as plt
import numpy as np
import yfinance as yf

sp = yf.download("^GSPC", start='1990-01-01')
```

    [*********************100%***********************]  1 of 1 completed



```python
plt.rc('font', family='AppleGothic') # window:Malgun Gothic
plt.rc('axes', unicode_minus=False)

fig, ax1 = plt.subplots(figsize=(8,5))
ax1.plot(t10y2y, color='black', lw=0.5, label='10Y-2Y')
ax1.plot(t10y3m, color='gray', lw=0.5, label='10Y-3M')
ax1.legend()
ax1.axhline(0, color='r', ls='dashed')
ax1.set_ylabel("장단기 금리차")
ax1.legend(loc='lower right')

ax2 = ax1.twinx()
ax2.plot(sp['Close'], label='S&P500')
ax2.set_ylabel("S&P500 지수(로그)")
ax2.legend(loc='upper right')
​    
```


### ❍ Fear and Greed Index 

https://edition.cnn.com/markets/fear-and-greed

7가지 지표를 활용해서 산정함

동적 페이지라서 셀레니움 필요


```python
import selenium
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By
import time

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
driver.get(url='https://edition.cnn.com/markets/fear-and-greed')
time.sleep(2)

```

    [WDM] - Downloading: 100%|█████████████████████████████████████████████████████████████████████████████████| 8.01M/8.01M [00:00<00:00, 11.6MB/s]



```python
idx = driver.find_element(By.CLASS_NAME, 'market-fng-gauge__dial-number-value').text
driver.close()
```


```python
idx
```




    '57'



## ❒ 해외 종목 다운로드

### ❍ 미국 Ticker

한경글로벌마켓
https://www.hankyung.com/globalmarket/equities/amex/top-market-capitalization


```python
import json
import requests
import pandas as pd
import time

markets = ['nyse','nasdaq','amex']
stock = []

for market in markets:

    url = f"https://www.hankyung.com/globalmarket/data/price?type={market}&sort=market_cap_top&sector_nm=&industry_nm=&chg_net_text="
    data = requests.get(url).json() # json 형태의 데이터를 불러옴
    data_pd = pd.json_normalize(data['list'])
    data_pd
    data_pd['symbol'] = data_pd['symbol'].str.replace("-US","")
    data_pd['symbol'] = data_pd['symbol'].str.replace(".","-")
    stock.append(data_pd)
    time.sleep(2)
```

    /var/folders/vr/g5rxclqn2qb11v47jh4p2lrw0000gn/T/ipykernel_1979/770174442.py:16: FutureWarning: The default value of regex will change from True to False in a future version. In addition, single character regular expressions will *not* be treated as literal strings when regex=True.
      data_pd['symbol'] = data_pd['symbol'].str.replace(".","-")
    /var/folders/vr/g5rxclqn2qb11v47jh4p2lrw0000gn/T/ipykernel_1979/770174442.py:16: FutureWarning: The default value of regex will change from True to False in a future version. In addition, single character regular expressions will *not* be treated as literal strings when regex=True.
      data_pd['symbol'] = data_pd['symbol'].str.replace(".","-")
    /var/folders/vr/g5rxclqn2qb11v47jh4p2lrw0000gn/T/ipykernel_1979/770174442.py:16: FutureWarning: The default value of regex will change from True to False in a future version. In addition, single character regular expressions will *not* be treated as literal strings when regex=True.
      data_pd['symbol'] = data_pd['symbol'].str.replace(".","-")



```python
stock_bind = pd.concat(stock)
stock_bind
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
      <th>symbol</th>
      <th>hname</th>
      <th>primary_exchange_name</th>
      <th>sector_nm</th>
      <th>industry_nm</th>
      <th>market_cap</th>
      <th>close_price</th>
      <th>chg_net</th>
      <th>chg_rate</th>
      <th>volume</th>
      <th>avg_volume</th>
      <th>turnover</th>
      <th>dps</th>
      <th>l52w_price</th>
      <th>h52w_price</th>
      <th>per</th>
      <th>open_price</th>
      <th>locdt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>BRK-A</td>
      <td>버크셔 해서웨이</td>
      <td>NYSE</td>
      <td>금융</td>
      <td>상해보험</td>
      <td>700187.7787</td>
      <td>478005.0000</td>
      <td>6505.0000</td>
      <td>1.3796</td>
      <td>5095</td>
      <td>4814.4286</td>
      <td>2408407.7811</td>
      <td>0</td>
      <td>393012.2500</td>
      <td>531880.0000</td>
      <td>0</td>
      <td>473479.4488</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BRK-B</td>
      <td>버크셔 해서웨이</td>
      <td>NYSE</td>
      <td>금융</td>
      <td>상해보험</td>
      <td>686652.9097</td>
      <td>312.5100</td>
      <td>2.1200</td>
      <td>0.6830</td>
      <td>3132148</td>
      <td>4765457.7619</td>
      <td>975452.3787</td>
      <td>0</td>
      <td>259.8500</td>
      <td>354.3300</td>
      <td>0</td>
      <td>309.8200</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>UNH</td>
      <td>유나이티드헬스그룹</td>
      <td>NYSE</td>
      <td>헬스케어 서비스</td>
      <td>건강관리</td>
      <td>478373.2701</td>
      <td>512.8100</td>
      <td>3.5800</td>
      <td>0.7030</td>
      <td>3472551</td>
      <td>3381885.4762</td>
      <td>1775900.0744</td>
      <td>6.6</td>
      <td>449.7009</td>
      <td>558.1000</td>
      <td>25.027</td>
      <td>511.0000</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>XOM</td>
      <td>엑슨모빌</td>
      <td>NYSE</td>
      <td>에너지</td>
      <td>종합석유산업</td>
      <td>468367.3995</td>
      <td>115.0500</td>
      <td>-1.9400</td>
      <td>-1.6583</td>
      <td>15777957</td>
      <td>19682549.4286</td>
      <td>1824534.6842</td>
      <td>3.64</td>
      <td>79.2900</td>
      <td>119.6300</td>
      <td>8.3055</td>
      <td>116.8600</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TSM</td>
      <td>TSMC</td>
      <td>NYSE</td>
      <td>전기전자</td>
      <td>반도체</td>
      <td>468029.7600</td>
      <td>90.2400</td>
      <td>0.0400</td>
      <td>0.0443</td>
      <td>5874515</td>
      <td>9969092.4286</td>
      <td>530225.7386</td>
      <td>1.4112</td>
      <td>59.4300</td>
      <td>104.5000</td>
      <td>11.376</td>
      <td>89.6600</td>
      <td>2023-04-06</td>
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
    </tr>
    <tr>
      <th>230</th>
      <td>NBY</td>
      <td>NovaBay Pharmaceuticals, Inc.</td>
      <td>AMEX</td>
      <td>헬스케어 기술</td>
      <td>의약품</td>
      <td>3.2567</td>
      <td>1.6000</td>
      <td>-0.1000</td>
      <td>-5.8824</td>
      <td>2843</td>
      <td>14109.5238</td>
      <td>4.2799</td>
      <td>0</td>
      <td>1.2400</td>
      <td>12.6350</td>
      <td>0</td>
      <td>1.6100</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>231</th>
      <td>EFSH</td>
      <td>1847 Holdings LLC</td>
      <td>AMEX</td>
      <td>상업 서비스</td>
      <td>기타 산업서비스</td>
      <td>3.2225</td>
      <td>0.7900</td>
      <td>-0.0599</td>
      <td>-7.0479</td>
      <td>3806</td>
      <td>14881.9524</td>
      <td>2.9055</td>
      <td>0.525</td>
      <td>0.7007</td>
      <td>18.0000</td>
      <td>0</td>
      <td>0.7699</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>232</th>
      <td>CPHI</td>
      <td>China Pharma Holdings, Inc.</td>
      <td>AMEX</td>
      <td>헬스케어 기술</td>
      <td>의약품</td>
      <td>2.9519</td>
      <td>0.3490</td>
      <td>0.0080</td>
      <td>2.3460</td>
      <td>107812</td>
      <td>163925.3333</td>
      <td>35.4662</td>
      <td>0</td>
      <td>0.3200</td>
      <td>4.1480</td>
      <td>0</td>
      <td>0.3388</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>233</th>
      <td>DXF</td>
      <td>Dunxin Financial Holdings Ltd</td>
      <td>AMEX</td>
      <td>금융</td>
      <td>금융리스</td>
      <td>2.6183</td>
      <td>0.1254</td>
      <td>-0.0271</td>
      <td>-17.7705</td>
      <td>910720</td>
      <td>864171.1429</td>
      <td>116.0838</td>
      <td>0</td>
      <td>0.1213</td>
      <td>0.8000</td>
      <td>0</td>
      <td>0.1440</td>
      <td>2023-04-06</td>
    </tr>
    <tr>
      <th>234</th>
      <td>UFAB</td>
      <td>Unique Fabricating Inc</td>
      <td>AMEX</td>
      <td>제조업</td>
      <td>자동차 부품</td>
      <td>1.8268</td>
      <td>0.1557</td>
      <td>0.0015</td>
      <td>0.9728</td>
      <td>172462</td>
      <td>242471.7619</td>
      <td>11.7589</td>
      <td>0</td>
      <td>0.1400</td>
      <td>1.8600</td>
      <td>0</td>
      <td>0.1500</td>
      <td>2023-04-06</td>
    </tr>
  </tbody>
</table>
<p>5941 rows × 18 columns</p>
</div>




```python
temp = pd.read_pickle('./data/tickers_usa_hankyung.pickle')
temp
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
      <th>symbol</th>
      <th>hname</th>
      <th>primary_exchange_name</th>
      <th>sector_nm</th>
      <th>industry_nm</th>
      <th>market_cap</th>
      <th>close_price</th>
      <th>chg_net</th>
      <th>chg_rate</th>
      <th>volume</th>
      <th>avg_volume</th>
      <th>turnover</th>
      <th>dps</th>
      <th>l52w_price</th>
      <th>h52w_price</th>
      <th>per</th>
      <th>open_price</th>
      <th>locdt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>BRK-A</td>
      <td>버크셔 해서웨이</td>
      <td>NYSE</td>
      <td>금융</td>
      <td>상해보험</td>
      <td>690659.1723</td>
      <td>471500.0000</td>
      <td>-8715.0000</td>
      <td>-1.8148</td>
      <td>4412</td>
      <td>4584.5556</td>
      <td>2110422.5624</td>
      <td>0</td>
      <td>393012.2500</td>
      <td>544389.2600</td>
      <td>0</td>
      <td>479733.2450</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BRK-B</td>
      <td>버크셔 해서웨이</td>
      <td>NYSE</td>
      <td>금융</td>
      <td>상해보험</td>
      <td>683598.7753</td>
      <td>311.1200</td>
      <td>-5.8500</td>
      <td>-1.8456</td>
      <td>3610304</td>
      <td>3430777.6111</td>
      <td>1126597.7294</td>
      <td>0</td>
      <td>259.8500</td>
      <td>362.1000</td>
      <td>0</td>
      <td>316.3900</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>TSM</td>
      <td>TSMC</td>
      <td>NYSE</td>
      <td>전기전자</td>
      <td>반도체</td>
      <td>460783.2080</td>
      <td>88.8500</td>
      <td>-0.7300</td>
      <td>-0.8149</td>
      <td>8761066</td>
      <td>12403414.5000</td>
      <td>778860.1537</td>
      <td>1.4144</td>
      <td>59.4300</td>
      <td>109.7550</td>
      <td>11.376</td>
      <td>89.9500</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>XOM</td>
      <td>엑슨모빌</td>
      <td>NYSE</td>
      <td>에너지</td>
      <td>종합석유산업</td>
      <td>459642.3469</td>
      <td>111.6100</td>
      <td>-2.2000</td>
      <td>-1.9330</td>
      <td>11525289</td>
      <td>14504950.0000</td>
      <td>1286414.6486</td>
      <td>3.64</td>
      <td>76.2500</td>
      <td>119.6300</td>
      <td>8.3055</td>
      <td>112.8100</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>V</td>
      <td>비자</td>
      <td>NYSE</td>
      <td>상업 서비스</td>
      <td>기타 산업서비스</td>
      <td>459603.1114</td>
      <td>223.1700</td>
      <td>-3.5800</td>
      <td>-1.5788</td>
      <td>4386798</td>
      <td>5017618.5000</td>
      <td>984718.8474</td>
      <td>1.8</td>
      <td>174.6000</td>
      <td>234.3000</td>
      <td>29.6487</td>
      <td>226.7500</td>
      <td>2023-03-07</td>
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
    </tr>
    <tr>
      <th>226</th>
      <td>AMPE</td>
      <td>Ampio Pharmaceuticals Inc</td>
      <td>AMEX</td>
      <td>헬스케어 기술</td>
      <td>의약품</td>
      <td>4.2014</td>
      <td>0.2785</td>
      <td>-0.0014</td>
      <td>-0.5002</td>
      <td>45572</td>
      <td>114996.2222</td>
      <td>12.4635</td>
      <td>0</td>
      <td>0.1990</td>
      <td>9.7050</td>
      <td>0</td>
      <td>0.2611</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>227</th>
      <td>NBY</td>
      <td>NovaBay Pharmaceuticals, Inc.</td>
      <td>AMEX</td>
      <td>헬스케어 기술</td>
      <td>의약품</td>
      <td>4.1319</td>
      <td>2.0300</td>
      <td>-0.1300</td>
      <td>-6.0185</td>
      <td>9634</td>
      <td>12672.6667</td>
      <td>20.1074</td>
      <td>0</td>
      <td>1.2400</td>
      <td>12.6350</td>
      <td>0</td>
      <td>2.1900</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>228</th>
      <td>DXF</td>
      <td>Dunxin Financial Holdings Ltd</td>
      <td>AMEX</td>
      <td>금융</td>
      <td>금융리스</td>
      <td>3.4409</td>
      <td>0.1648</td>
      <td>-0.0041</td>
      <td>-2.4275</td>
      <td>55625</td>
      <td>148780.6667</td>
      <td>9.0143</td>
      <td>0</td>
      <td>0.1505</td>
      <td>0.8000</td>
      <td>0</td>
      <td>0.1625</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>229</th>
      <td>UFAB</td>
      <td>Unique Fabricating Inc</td>
      <td>AMEX</td>
      <td>제조업</td>
      <td>자동차 부품</td>
      <td>3.1679</td>
      <td>0.2700</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>292401</td>
      <td>254710.6111</td>
      <td>76.9324</td>
      <td>0</td>
      <td>0.2480</td>
      <td>2.2600</td>
      <td>0</td>
      <td>0.2882</td>
      <td>2023-03-07</td>
    </tr>
    <tr>
      <th>230</th>
      <td>CPHI</td>
      <td>China Pharma Holdings, Inc.</td>
      <td>AMEX</td>
      <td>헬스케어 기술</td>
      <td>의약품</td>
      <td>3.0815</td>
      <td>0.5967</td>
      <td>-0.0333</td>
      <td>-5.2857</td>
      <td>160661</td>
      <td>1687045.2778</td>
      <td>94.3365</td>
      <td>0</td>
      <td>0.5700</td>
      <td>4.8400</td>
      <td>0</td>
      <td>0.6110</td>
      <td>2023-03-07</td>
    </tr>
  </tbody>
</table>
<p>5987 rows × 18 columns</p>
</div>




```python
# stock_bind.to_pickle('./data/tickers_usa_hankyung.pickle')
```

### ❍ 그외 해외

investing.com > Market > Stocks > Stock screener

Select Equity Type > ORD : 보통주 DRC : DR종목, Prefrred : 우선주


```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import math
import pandas as pd
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
import numpy as np
from tqdm import tqdm
import time

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
nationcode = '5' # 국가코드 5: 미국

url = f"https://www.investing.com/stock-screener/?sp=country::{nationcode}|sector::a|industry::a|equityType::ORD|exchange::a%3Ceq_market_cap;1"
driver.get(url)

# driver 페이지가 열리고 엘리먼트가 보일 때까지 기다린다 ! 최대 10초 동안
WebDriverWait(driver, 10).until(EC.visibility_of_element_located((By.XPATH, '//*[@id="resultsTable"]/tbody')))

# 페이지 수 찾기
end_num = driver.find_element(By.CLASS_NAME, value = 'js-total-results').text
end_num = math.ceil(int(end_num) / 50) # 1페이지에 50개 종목

# 빈리스트 생성
all_data_df = []

for i in tqdm(range(1, end_num+1)):
    
    url = f"https://www.investing.com/stock-screener/?sp=country::{nationcode}|sector::a|industry::a|equityType::ORD|exchange::a%3Ceq_market_cap;{i}"
    driver.get(url)
    
    try : 
        # driver 페이지가 열리고 엘리먼트가 보일 때까지 기다린다 ! 최대 10초 동안
        WebDriverWait(driver, 10).until(EC.visibility_of_element_located((By.XPATH, '//*[@id="resultsTable"]/tbody')))
    
    except: # 오류가 있을 시
        time.sleep(1) # 쉬고
        driver.refresh() # 새로고침
        WebDriverWait(driver, 10).until(EC.visibility_of_element_located((By.XPATH, '//*[@id="resultsTable"]/tbody')))
        
    
    html = BeautifulSoup(driver.page_source, 'lxml')

    html_table = html.select('table.genTbl.openTbl.resultsStockScreenerTbl.elpTbl')
    
    df_table = pd.read_html(html_table[0].prettify()) # beautifulsoup로 파싱한 것을 unicode로 바꿔줍니다.
    df_table_select = df_table[0][["Name","Symbol","Exchange","Sector","Market Cap"]]
    
    all_data_df.append(df_table_select)
    time.sleep(1)

driver.quit()
```

    100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████| 171/171 [10:09<00:00,  3.56s/it]



```python
all_data_df_bind = pd.concat(all_data_df, axis=0)
all_data_df_bind['country']='United States'

from datetime import datetime
all_data_df_bind['date'] = datetime.today().strftime("%Y-%m-%d")
all_data_df_bind = all_data_df_bind[~all_data_df_bind['Name'].isnull()] # Name이 빈칸인건 제외
all_data_df_bind = all_data_df_bind[all_data_df_bind['Exchange'].isin(['NASDAQ','NYSE','NYSE Amex'])] # 거래소 필터링
all_data_df_bind = all_data_df_bind.drop_duplicates(["Symbol"])
all_data_df_bind.reset_index(inplace=True, drop=True)
all_data_df_bind = all_data_df_bind.replace({np.nan : None})

all_data_df_bind
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
      <th>Name</th>
      <th>Symbol</th>
      <th>Exchange</th>
      <th>Sector</th>
      <th>Market Cap</th>
      <th>country</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Apple</td>
      <td>AAPL</td>
      <td>NASDAQ</td>
      <td>Technology</td>
      <td>2.61T</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Microsoft</td>
      <td>MSFT</td>
      <td>NASDAQ</td>
      <td>Technology</td>
      <td>2.17T</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Alphabet C</td>
      <td>GOOG</td>
      <td>NASDAQ</td>
      <td>Technology</td>
      <td>1.39T</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alphabet A</td>
      <td>GOOGL</td>
      <td>NASDAQ</td>
      <td>Technology</td>
      <td>1.39T</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Amazon.com</td>
      <td>AMZN</td>
      <td>NASDAQ</td>
      <td>Consumer Cyclicals</td>
      <td>1.05T</td>
      <td>United States</td>
      <td>2023-04-08</td>
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
    </tr>
    <tr>
      <th>5521</th>
      <td>5E Advanced Materials</td>
      <td>FEAM</td>
      <td>NASDAQ</td>
      <td>Basic Materials</td>
      <td>-</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>5522</th>
      <td>MGO Global</td>
      <td>MGOL</td>
      <td>NASDAQ</td>
      <td>None</td>
      <td>-</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>5523</th>
      <td>CaliberCos</td>
      <td>CWD</td>
      <td>NASDAQ</td>
      <td>None</td>
      <td>-</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>5524</th>
      <td>Chanson International Holding</td>
      <td>CHSN</td>
      <td>NASDAQ</td>
      <td>None</td>
      <td>-</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
    <tr>
      <th>5525</th>
      <td>Hongli</td>
      <td>HLP</td>
      <td>NASDAQ</td>
      <td>None</td>
      <td>-</td>
      <td>United States</td>
      <td>2023-04-08</td>
    </tr>
  </tbody>
</table>
<p>5526 rows × 7 columns</p>
</div>




```python
# all_data_df_bind.to_pickle("./data/tickers_usa_investing.pickle")
```

## ❒ 해외종목 가격, 재무제표 


```python
# 시간이 너무 오래걸려서 돌리지 않음
```

### ❍ 가격


```python
# 미국 시장 데이터만 필요한 경우, 유료 데이터 벤더 이용하는 것도 좋음 
# stock data api
# polygon, alpha vantage, finhub, 
# tiingo > 10$/month >> api서비스 이용 , 안정적인 데이터를 더 빠르게 

# 미국 외의 데이터
# finance.yahoo.com
# 
```


```python
import yfinance as yf
import pandas as pd
import numpy as np
from tqdm import tqdm
import time
```


```python
#ticker_list = pd.read_pickle("./data/tickers_usa_hankyung.pickle")
ticker_list = pd.read_pickle("./data/tickers_usa_investing.pickle")
print(ticker_list.shape)
```

    (5526, 7)



```python
df = pd.DataFrame(columns = ['Date','Open','High','Low','Close','Adj Close','Volume','ticker'])
error_list = []
```


```python
for i in tqdm(range(0, len(ticker_list))):
    
    ticker = ticker_list['Symbol'][i]

    # 오류 무시
    try : 
        price = yf.download(ticker, progress=False) #progress=False : 진행상황 표기 없애줌
        price = price.reset_index()
        price['ticker'] = ticker
        df = pd.concat([df, price])
        
    except:
        print(ticker)
        error_list.append(ticker)
        
    time.sleep(1)

```




### ❍ 재무제표


```python
# !pip install yahooquery

from yahooquery import Ticker
import numpy as np
import pandas as pd
from tqdm import tqdm
import time

# 정보 다운로드
data = Ticker("AAPL")
# data.asset_profile
# data.summary_detail
# data.history()

# 연간 재무제표
data_y = data.all_financial_data(frequency='a') #'a': year, 'q':quarter
data_y.reset_index(inplace=True)
data_y = data_y.loc[:, ~data_y.columns.isin(['periodType','currencyCode'])] # 불필요열 삭제
data_y = data_y.melt(id_vars = ['symbol','asOfDate'])
data_y = data_y.replace([np.nan], None) # SQL에 np.nan은 저장안되므로 변경
data_y['freq'] = 'y'
data_y.columns = ['ticker','date','account','value','freq']

# 분기 재무제표
data_q = data.all_financial_data(frequency='q') #'a': year, 'q':quarter
data_q.reset_index(inplace=True)
data_q = data_q.loc[:, ~data_q.columns.isin(['periodType','currencyCode'])] # 불필요열 삭제
data_q = data_q.melt(id_vars = ['symbol','asOfDate'])
data_q = data_q.replace([np.nan], None) # SQL에 np.nan은 저장안되므로 변경
data_q['freq'] = 'q'
data_q.columns = ['ticker','date','account','value','freq']

# 데이터합치기
data_fs = pd.concat([data_y, data_q], axis=0)
data_fs
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
      <th>ticker</th>
      <th>date</th>
      <th>account</th>
      <th>value</th>
      <th>freq</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AAPL</td>
      <td>2019-09-30</td>
      <td>AccountsPayable</td>
      <td>46236000000.0</td>
      <td>y</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAPL</td>
      <td>2020-09-30</td>
      <td>AccountsPayable</td>
      <td>42296000000.0</td>
      <td>y</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAPL</td>
      <td>2021-09-30</td>
      <td>AccountsPayable</td>
      <td>54763000000.0</td>
      <td>y</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>2022-09-30</td>
      <td>AccountsPayable</td>
      <td>64115000000.0</td>
      <td>y</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AAPL</td>
      <td>2019-09-30</td>
      <td>AccountsReceivable</td>
      <td>22926000000.0</td>
      <td>y</td>
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
      <th>603</th>
      <td>AAPL</td>
      <td>2022-12-31</td>
      <td>TradeandOtherPayablesNonCurrent</td>
      <td>None</td>
      <td>q</td>
    </tr>
    <tr>
      <th>604</th>
      <td>AAPL</td>
      <td>2022-03-31</td>
      <td>WorkingCapital</td>
      <td>-9328000000.0</td>
      <td>q</td>
    </tr>
    <tr>
      <th>605</th>
      <td>AAPL</td>
      <td>2022-06-30</td>
      <td>WorkingCapital</td>
      <td>-17581000000.0</td>
      <td>q</td>
    </tr>
    <tr>
      <th>606</th>
      <td>AAPL</td>
      <td>2022-09-30</td>
      <td>WorkingCapital</td>
      <td>-18577000000.0</td>
      <td>q</td>
    </tr>
    <tr>
      <th>607</th>
      <td>AAPL</td>
      <td>2022-12-31</td>
      <td>WorkingCapital</td>
      <td>-8509000000.0</td>
      <td>q</td>
    </tr>
  </tbody>
</table>
<p>1216 rows × 5 columns</p>
</div>




```python
ticker_list = pd.read_pickle("./data/tickers_usa_investing.pickle")

df_fs= pd.DataFrame(columns = ['ticker','date','account','value','freq'])
error_list_fs = []

for i in tqdm(range(0, len(ticker_list))):
    
    ticker = ticker_list['Symbol'][i]
    
    # 오류 무시
    try : 

        # 정보 다운로드
        data = Ticker(ticker)
        
        # 연간 재무제표
        data_y = data.all_financial_data(frequency='a') #'a': year, 'q':quarter
        data_y.reset_index(inplace=True)
        data_y = data_y.loc[:, ~data_y.columns.isin(['periodType','currencyCode'])] # 불필요열 삭제
        data_y = data_y.melt(id_vars = ['symbol','asOfDate'])
        data_y = data_y.replace([np.nan], None) # SQL에 np.nan은 저장안되므로 변경
        data_y['freq'] = 'y'
        data_y.columns = ['ticker','date','account','value','freq']

        # 분기 재무제표
        data_q = data.all_financial_data(frequency='q') #'a': year, 'q':quarter
        data_q.reset_index(inplace=True)
        data_q = data_q.loc[:, ~data_q.columns.isin(['periodType','currencyCode'])] # 불필요열 삭제
        data_q = data_q.melt(id_vars = ['symbol','asOfDate'])
        data_q = data_q.replace([np.nan], None) # SQL에 np.nan은 저장안되므로 변경
        data_q['freq'] = 'q'
        data_q.columns = ['ticker','date','account','value','freq']

        # 데이터합치기
        data_fs = pd.concat([data_y, data_q], axis=0)
        
        # 데이터프레임 합치기
        df_fs = pd.concat([df_fs, data_fs])
        
    except:
        print(ticker)
        error_list_fs.append(ticker)
        
    time.sleep(1)
```

## ❒ (참고)

- 유투브 : 헨리의 퀀트대학 (https://www.youtube.com/channel/UCHfiWvw33aSBktAlWICfPKQ)

