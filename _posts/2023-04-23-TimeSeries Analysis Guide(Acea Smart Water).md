---
layout: single
# classes: wide
title : "시계열 데이터 분석 가이드 (Acea Smart Water)"
categories: portfolio
tag: [python, time-series, 시계열, ARIMA, LSTM, prophet]
toc: true
toc_label: "순서"
toc_icon: "cog"
author_profile: false
sidebar:
    nav: "docs"
Typora-root-url: ../


---

---



kaggle의 timeseries 가이드를 현재 실행가능한 버전으로 필사하였습니다. 해당 글은 2년 전 글이며, 출처는 다음과 같습니다. 

TimeSeries Analysis 📈A Complete Guide

https://www.kaggle.com/code/andreshg/timeseries-analysis-a-complete-guide


```pytho
from google.colab import drive
drive.mount('/content/drive')
```


### 목적

- 시계열 데이터를 해석
- 시계열 데이터를 다루기 
- 일반적인 시계열 모델 
    - ACF/PACF
    - ARIMA
    - Auto-ARIMA
    - Prophet
    - Augmented Dickey-Fuller (ADF)



```python
!pip install colorama
```

```python
import numpy as np
import pandas as pd

import seaborn as sns
import matplotlib.pyplot as plt
from colorama import Fore

from sklearn.metrics import mean_absolute_error, mean_squared_error
import math

import warnings
warnings.filterwarnings('ignore')

from datetime import datetime, date

np.random.seed(7)
```


```python
df = pd.read_csv('/content/drive/MyDrive/kaggle/Acea_Smart_Water_Analytics/data/Aquifer_Petrignano.csv')
df
```





  <div id="df-4b324b07-5095-4e43-8add-82921f970d38">
    <div class="colab-df-container">
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
      <th>Date</th>
      <th>Rainfall_Bastia_Umbra</th>
      <th>Depth_to_Groundwater_P24</th>
      <th>Depth_to_Groundwater_P25</th>
      <th>Temperature_Bastia_Umbra</th>
      <th>Temperature_Petrignano</th>
      <th>Volume_C10_Petrignano</th>
      <th>Hydrometry_Fiume_Chiascio_Petrignano</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>14/03/2006</td>
      <td>NaN</td>
      <td>-22.48</td>
      <td>-22.18</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>15/03/2006</td>
      <td>NaN</td>
      <td>-22.38</td>
      <td>-22.14</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>16/03/2006</td>
      <td>NaN</td>
      <td>-22.25</td>
      <td>-22.04</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17/03/2006</td>
      <td>NaN</td>
      <td>-22.38</td>
      <td>-22.04</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>18/03/2006</td>
      <td>NaN</td>
      <td>-22.60</td>
      <td>-22.04</td>
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
    </tr>
    <tr>
      <th>5218</th>
      <td>26/06/2020</td>
      <td>0.0</td>
      <td>-25.68</td>
      <td>-25.07</td>
      <td>25.7</td>
      <td>24.5</td>
      <td>-29930.688</td>
      <td>2.5</td>
    </tr>
    <tr>
      <th>5219</th>
      <td>27/06/2020</td>
      <td>0.0</td>
      <td>-25.80</td>
      <td>-25.11</td>
      <td>26.2</td>
      <td>25.0</td>
      <td>-31332.960</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>5220</th>
      <td>28/06/2020</td>
      <td>0.0</td>
      <td>-25.80</td>
      <td>-25.19</td>
      <td>26.9</td>
      <td>25.7</td>
      <td>-32120.928</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>5221</th>
      <td>29/06/2020</td>
      <td>0.0</td>
      <td>-25.78</td>
      <td>-25.18</td>
      <td>26.9</td>
      <td>26.0</td>
      <td>-30602.880</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>5222</th>
      <td>30/06/2020</td>
      <td>0.0</td>
      <td>-25.91</td>
      <td>-25.25</td>
      <td>27.3</td>
      <td>26.5</td>
      <td>-31878.144</td>
      <td>2.4</td>
    </tr>
  </tbody>
</table>
<p>5223 rows × 8 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-4b324b07-5095-4e43-8add-82921f970d38')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }
    
    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }
    
    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }
    
    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-4b324b07-5095-4e43-8add-82921f970d38 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';
    
        async function convertToInteractive(key) {
          const element = document.querySelector('#df-4b324b07-5095-4e43-8add-82921f970d38');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;
    
          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
# remove nan rows
df = df[df.Rainfall_Bastia_Umbra.notna()].reset_index(drop=True)
# remove not usefull columns 
df = df.drop(['Depth_to_Groundwater_P24', 'Temperature_Petrignano'], axis=1)
```


```python
# simplify column names
df.columns = ['date', 'rainfall','depth_to_groundwater','temperature','drainage_volume','river_hydrometry']
targets = ['depth_to_groundwater']
features = [feature for feature in df.columns if feature not in targets]
df.head()
```





  <div id="df-bd3e7b6b-fb3f-4c55-a55f-3fe4623d6153">
    <div class="colab-df-container">
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
      <th>date</th>
      <th>rainfall</th>
      <th>depth_to_groundwater</th>
      <th>temperature</th>
      <th>drainage_volume</th>
      <th>river_hydrometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/01/2009</td>
      <td>0.0</td>
      <td>-31.14</td>
      <td>5.2</td>
      <td>-24530.688</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>02/01/2009</td>
      <td>0.0</td>
      <td>-31.11</td>
      <td>2.3</td>
      <td>-28785.888</td>
      <td>2.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>03/01/2009</td>
      <td>0.0</td>
      <td>-31.07</td>
      <td>4.4</td>
      <td>-25766.208</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>04/01/2009</td>
      <td>0.0</td>
      <td>-31.05</td>
      <td>0.8</td>
      <td>-27919.296</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>05/01/2009</td>
      <td>0.0</td>
      <td>-31.01</td>
      <td>-1.9</td>
      <td>-29854.656</td>
      <td>2.3</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-bd3e7b6b-fb3f-4c55-a55f-3fe4623d6153')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }
    
    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }
    
    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }
    
    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-bd3e7b6b-fb3f-4c55-a55f-3fe4623d6153 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';
    
        async function convertToInteractive(key) {
          const element = document.querySelector('#df-bd3e7b6b-fb3f-4c55-a55f-3fe4623d6153');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;
    
          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
df['date'] = pd.to_datetime(df['date'], format='%d/%m/%Y')
df.head().style.set_properties(subset=['date'], **{'background-color':'dodgerblue'})
```




<style type="text/css">
#T_b6108_row0_col0, #T_b6108_row1_col0, #T_b6108_row2_col0, #T_b6108_row3_col0, #T_b6108_row4_col0 {
  background-color: dodgerblue;
}
</style>
<table id="T_b6108" class="dataframe">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_b6108_level0_col0" class="col_heading level0 col0" >date</th>
      <th id="T_b6108_level0_col1" class="col_heading level0 col1" >rainfall</th>
      <th id="T_b6108_level0_col2" class="col_heading level0 col2" >depth_to_groundwater</th>
      <th id="T_b6108_level0_col3" class="col_heading level0 col3" >temperature</th>
      <th id="T_b6108_level0_col4" class="col_heading level0 col4" >drainage_volume</th>
      <th id="T_b6108_level0_col5" class="col_heading level0 col5" >river_hydrometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_b6108_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_b6108_row0_col0" class="data row0 col0" >2009-01-01 00:00:00</td>
      <td id="T_b6108_row0_col1" class="data row0 col1" >0.000000</td>
      <td id="T_b6108_row0_col2" class="data row0 col2" >-31.140000</td>
      <td id="T_b6108_row0_col3" class="data row0 col3" >5.200000</td>
      <td id="T_b6108_row0_col4" class="data row0 col4" >-24530.688000</td>
      <td id="T_b6108_row0_col5" class="data row0 col5" >2.400000</td>
    </tr>
    <tr>
      <th id="T_b6108_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_b6108_row1_col0" class="data row1 col0" >2009-01-02 00:00:00</td>
      <td id="T_b6108_row1_col1" class="data row1 col1" >0.000000</td>
      <td id="T_b6108_row1_col2" class="data row1 col2" >-31.110000</td>
      <td id="T_b6108_row1_col3" class="data row1 col3" >2.300000</td>
      <td id="T_b6108_row1_col4" class="data row1 col4" >-28785.888000</td>
      <td id="T_b6108_row1_col5" class="data row1 col5" >2.500000</td>
    </tr>
    <tr>
      <th id="T_b6108_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_b6108_row2_col0" class="data row2 col0" >2009-01-03 00:00:00</td>
      <td id="T_b6108_row2_col1" class="data row2 col1" >0.000000</td>
      <td id="T_b6108_row2_col2" class="data row2 col2" >-31.070000</td>
      <td id="T_b6108_row2_col3" class="data row2 col3" >4.400000</td>
      <td id="T_b6108_row2_col4" class="data row2 col4" >-25766.208000</td>
      <td id="T_b6108_row2_col5" class="data row2 col5" >2.400000</td>
    </tr>
    <tr>
      <th id="T_b6108_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_b6108_row3_col0" class="data row3 col0" >2009-01-04 00:00:00</td>
      <td id="T_b6108_row3_col1" class="data row3 col1" >0.000000</td>
      <td id="T_b6108_row3_col2" class="data row3 col2" >-31.050000</td>
      <td id="T_b6108_row3_col3" class="data row3 col3" >0.800000</td>
      <td id="T_b6108_row3_col4" class="data row3 col4" >-27919.296000</td>
      <td id="T_b6108_row3_col5" class="data row3 col5" >2.400000</td>
    </tr>
    <tr>
      <th id="T_b6108_level0_row4" class="row_heading level0 row4" >4</th>
      <td id="T_b6108_row4_col0" class="data row4 col0" >2009-01-05 00:00:00</td>
      <td id="T_b6108_row4_col1" class="data row4 col1" >0.000000</td>
      <td id="T_b6108_row4_col2" class="data row4 col2" >-31.010000</td>
      <td id="T_b6108_row4_col3" class="data row4 col3" >-1.900000</td>
      <td id="T_b6108_row4_col4" class="data row4 col4" >-29854.656000</td>
      <td id="T_b6108_row4_col5" class="data row4 col5" >2.300000</td>
    </tr>
  </tbody>
</table>




## 1. Data Visualization

<features>

- __Rainfall(강우량)__ indicates the quantity of rain falling (mm)
- __Temperature(기온)__ indicates the temperature (℃)
- __Drainage Volume(배수량)__ indicates the volume of water taken from the drinking water treatment plant (㎥)
- __Hydrometry(강 유량계)__ indicates the groundwater level (m)

<target>

- __Depth to Groundwater(지하수 깊이)__ indicates the groundwater level (m from the ground floor)


```python
# To complete the date, as naive method, we well use ffill

f, ax = plt.subplots(nrows=5, ncols=1, figsize=(15,25))

for i, column in enumerate(df.drop('date', axis=1).columns):
    sns.lineplot(x=df['date'], y=df[column].fillna(method='ffill'), 
                 ax=ax[i], color='dodgerblue')
    ax[i].set_title('Feature : {}'.format(column), fontsize=14)
    ax[i].set_ylabel(ylabel=column, fontsize=14)
    ax[i].set_xlim([date(2009, 1,1), date(2020, 6, 30)])
```


​    ![output_11_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_11_0.png)



## 2. Data Preprocessing

- 시간 간격(time interval) 확인


```python
df = df.sort_values(by='date')

# check time intervals
df['delta'] = df['date'] - df['date'].shift(1)
df[['date', 'delta']].head()
```





  <div id="df-3e9fdfcb-3f4c-42b8-b490-a1f418ad188a">
    <div class="colab-df-container">
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
      <th>date</th>
      <th>delta</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2009-01-01</td>
      <td>NaT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2009-01-02</td>
      <td>1 days</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2009-01-03</td>
      <td>1 days</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2009-01-04</td>
      <td>1 days</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2009-01-05</td>
      <td>1 days</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-3e9fdfcb-3f4c-42b8-b490-a1f418ad188a')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }
    
    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }
    
    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }
    
    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-3e9fdfcb-3f4c-42b8-b490-a1f418ad188a button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';
    
        async function convertToInteractive(key) {
          const element = document.querySelector('#df-3e9fdfcb-3f4c-42b8-b490-a1f418ad188a');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;
    
          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
df['delta'].sum(), df['delta'].count()
```




    (Timedelta('4198 days 00:00:00'), 4198)




```python
df = df.drop('delta', axis=1)
```

### 2.1 Handle Missings


```python
df.isna().sum()
```




    date                     0
    rainfall                 0
    depth_to_groundwater    27
    temperature              0
    drainage_volume          1
    river_hydrometry         0
    dtype: int64




```python
# 일단 이상데이터(0)을 np.nan으로 바꾼 후 fillna를 통해 데이터를 채워줌
f, ax = plt.subplots(nrows=2, ncols=1, figsize=(15,15))

old_hydrometry = df['river_hydrometry'].copy()
df['river_hydrometry'] = df['river_hydrometry'].replace(0, np.nan)
sns.lineplot(x=df['date'], y=old_hydrometry, ax=ax[0], 
             color='darkorange', label='original')
sns.lineplot(x=df['date'], y=df['river_hydrometry'].fillna(np.inf), ax=ax[0],
                  color='dodgerblue', label='modified')
ax[0].set_title('Feature : Hydrometry', fontsize=14)
ax[0].set_ylabel(ylabel='Hydrometry', fontsize=14)
ax[0].set_xlim([date(2009, 1,1), date(2020, 6, 30)])

old_drainage = df['drainage_volume'].copy()
df['drainage_volume'] = df['drainage_volume'].replace(0, np.nan)
sns.lineplot(x=df['date'], y=old_drainage, ax=ax[1], color='darkorange', label='original')
sns.lineplot(x=df['date'], y=df['drainage_volume'].fillna(np.inf), ax=ax[1], color='dodgerblue', label='modified')
ax[1].set_title('Feature: Drainage', fontsize=14)
ax[1].set_ylabel(ylabel='Drainage', fontsize=14)
ax[1].set_xlim([date(2009, 1, 1), date(2020, 6, 30)])
```




    (14245.0, 18443.0)



​    

​    ![output_19_1](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_19_1.png)



```python
f, ax = plt.subplots(nrows=1, ncols=1, figsize=(16,5))

sns.heatmap(df.T.isna(), cmap='Blues')
ax.set_title("Missing Values", fontsize=16)

for tick in ax.yaxis.get_major_ticks():
    tick.label.set_fontsize(14)
plt.show()
```


​    ![output_20_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_20_0.png)

​    


__Missing Values, how to handle__
- Option 1: Fill NaN with Outlier or Zero <br>

In this specific example filling the missing value with an outlier value such as np.inf or 0 seems to be very naive. However, using values like -999, is sometimes a good idea.

- Option 2: Fill NaN with Mean Value <br>

Filling NaNs with the mean value is also not sufficient and naive, and doesn't seems to be a good option.

- Option 3: Fill NaN with Last Value with .ffill() <br>

Filling NaNs with the last value could be bit better.

- Option 4: Fill NaN with Linearly Interpolated Value with .interpolate() <br>

Filling NaNs with the interpolated values is the best option in this small examlple but it requires knowledge of the neighouring value


```python
f, ax = plt.subplots(nrows=4, ncols=1, figsize=(15, 12))

sns.lineplot(x=df['date'], y=df['drainage_volume'].fillna(0), ax=ax[0], color='darkorange', label = 'modified')
sns.lineplot(x=df['date'], y=df['drainage_volume'].fillna(np.inf), ax=ax[0], color='dodgerblue', label = 'original')
ax[0].set_title('Fill NaN with 0', fontsize=14)
ax[0].set_ylabel(ylabel='Volume C10 Petrignano', fontsize=14)

mean_drainage = df['drainage_volume'].mean()
sns.lineplot(x=df['date'], y=df['drainage_volume'].fillna(mean_drainage), ax=ax[1], color='darkorange', label = 'modified')
sns.lineplot(x=df['date'], y=df['drainage_volume'].fillna(np.inf), ax=ax[1], color='dodgerblue', label = 'original')
ax[1].set_title(f'Fill NaN with Mean Value ({mean_drainage:.0f})', fontsize=14)
ax[1].set_ylabel(ylabel='Volume C10 Petrignano', fontsize=14)

sns.lineplot(x=df['date'], y=df['drainage_volume'].ffill(), ax=ax[2], color='darkorange', label = 'modified')
sns.lineplot(x=df['date'], y=df['drainage_volume'].fillna(np.inf), ax=ax[2], color='dodgerblue', label = 'original')
ax[2].set_title(f'FFill', fontsize=14)
ax[2].set_ylabel(ylabel='Volume C10 Petrignano', fontsize=14)

sns.lineplot(x=df['date'], y=df['drainage_volume'].interpolate(), ax=ax[3], color='darkorange', label = 'modified')
sns.lineplot(x=df['date'], y=df['drainage_volume'].fillna(np.inf), ax=ax[3], color='dodgerblue', label = 'original')
ax[3].set_title(f'Interpolate', fontsize=14)
ax[3].set_ylabel(ylabel='Volume C10 Petrignano', fontsize=14)

for i in range(4):
    ax[i].set_xlim([date(2019, 5, 1), date(2019, 10, 1)])
    
plt.tight_layout()
plt.show()
```


​    ![output_22_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_22_0.png)

​    

```python
# interpolate method looks best options

df['drainage_volume'] = df['drainage_volume'].interpolate()
df['river_hydrometry'] = df['river_hydrometry'].interpolate()
df['depth_to_groundwater'] = df['depth_to_groundwater'].interpolate()
```

### 2.2 Smoothing data / Resampling 


```python
fig, ax = plt.subplots(ncols=2, nrows=3, sharex=True, figsize=(16,12))

sns.lineplot(x=df['date'], y=df['drainage_volume'], color='dodgerblue', ax=ax[0, 0])
ax[0, 0].set_title('Drainage Volume', fontsize=14)

resampled_df = df[['date','drainage_volume']].resample('7D', on='date').sum().reset_index(drop=False)
sns.lineplot(x=resampled_df['date'], y=resampled_df['drainage_volume'], color='dodgerblue', ax=ax[1, 0])
ax[1, 0].set_title('Weekly Drainage Volume', fontsize=14)

resampled_df = df[['date','drainage_volume']].resample('M', on='date').sum().reset_index(drop=False)
sns.lineplot(x=resampled_df['date'], y=resampled_df['drainage_volume'], color='dodgerblue', ax=ax[2, 0])
ax[2, 0].set_title('Monthly Drainage Volume', fontsize=14)

for i in range(3):
    ax[i, 0].set_xlim([date(2009, 1, 1), date(2020, 6, 30)])

sns.lineplot(x=df['date'], y=df['temperature'], color='dodgerblue', ax=ax[0, 1])
ax[0, 1].set_title('Daily Temperature (Acc.)', fontsize=14)

resampled_df = df[['date','temperature']].resample('7D', on='date').mean().reset_index(drop=False)
sns.lineplot(x=resampled_df['date'], y=resampled_df['temperature'], color='dodgerblue', ax=ax[1, 1])
ax[1, 1].set_title('Weekly Temperature (Acc.)', fontsize=14)

resampled_df = df[['date','temperature']].resample('M', on='date').mean().reset_index(drop=False)
sns.lineplot(x=resampled_df['date'], y=resampled_df['temperature'], color='dodgerblue', ax=ax[2, 1])
ax[2, 1].set_title('Monthly Temperature (Acc.)', fontsize=14)

for i in range(3):
    ax[i, 1].set_xlim([date(2009, 1, 1), date(2020, 6, 30)])
plt.show()

```


​    ![output_25_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_25_0.png)

​    



```python
# As we can see, downsample to weekly could smooth the data and hgelp with analysis
downsample = df[['date',
                 'depth_to_groundwater', 
                 'temperature',
                 'drainage_volume', 
                 'river_hydrometry',
                 'rainfall'
                ]].resample('7D', on='date').mean().reset_index(drop=False)

df = downsample.copy()
```

### 2.3. Stationarity

Some time-series models, such as such as ARIMA, assume that the underlying data is stationary. Stationarity describes that the time-series has

- constant mean and mean is not time-dependent
- constant variance and variance is not time-dependent
- constant covariance and covariance is not time-dependent

![stationarity](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABQoAAAGYCAIAAACF3khUAACAAElEQVR42uzd6XfcyJUo+HsjEFgSQO6ZXCRKKsuSyq7ntrvq2e/1eb145st86f+2v7/u8bzpd6bdZbtcLmthqSSKS5JM5gogsUbEnEyoUiwtFCmSKi33d+r4wFASGQkgkbi4EXGNJEmAEEIIIYQQQgj5uDHaBYQQQgghhBBCCIXHhBBCCCGEEEIIhceEEEIIIYQQQgiFx4QQQgghhBBCCAAYtAsIIYQQcuHyQsexBgDHQWEg7RBCCCHvPsoeE0IIIeTixbHe3su39/IySCaEEELefZQ9JoQQQsibKPPDcSLTRBoGNhoCAEajvCi0ZfNpoJ5sp8KAdpO5FcxznRdQSF0UWha6KFRRaNOc/5Vj08N6QgghFB4TQggh5L1V5od7vfRgP676/IvPfQD48g/BNJArq05ewONHUdVnt2+aec5HEzkNVRjpKJSzKI/CIgjyVkt88bnvrFm0MwkhhFB4TAghhJD3T5apIJCTQGWZjhPV62VRwJKfubbzLA8sDKj6rOpz49UDj5XUSaKDUBVSc4Y0SpkQQgiFx4QQQgh5nwSBvL8Z5zlsbFi4Lp48ehrTNhrii8/9snM1ANy+aZadroXARo17Ll90rjZkYZadq5UCZLh3kIeRti3cWBfCp/CYEEIIhceEEEIIeceUo4vTTD03Wrj8VyGg5jPbMq5dswHAdphjsx/2lBbLJc7RfmH7SaJGE5mFarmmzEsDgO9z06QxyYQQQt4qTJKE9gIhhBBCXjQN1PZePhgUz40WbrfEMoiVCkajvEwdn3WSLSn1csqusnP1dFLc34wB4M4tp9USdAgIIYS8TZQ9JoQQQsizYDXNdJoqzuahb5qpwaA47OeykHmmpHz6StNkrdazSPiN59biHF+aVc5SfTQopALLYpaJQsxfRgeIEEIIhceEEEIIeRvyXI8mcjDMDw9y28Y7txxZwCyax8atllGrWr7LqlW+7Fx9GXyf37nlHA2KXi/b3cu6K6LVFI0ap/CYEEIIhceEEEIIeaukhCRVAEyqp7NPmwKXYap9yTWKy7y0VLC7lyWpkhKyTB0NtWMhjUYmhBBC4TEhhBBC3oZyfmm2iEA5B8tiTplDPtbJ+e20xLJYd0VICasrIs/1o0cJ5+VoZAqPCSGEUHhMCCGEkEtQjjeOZmoyyZUC1zMYx0bDEAZaJj43xvhCZJCEOAYAT9dNsF8eHpvYagoAqPk8iiTndKAIIYS8DTRzNSGEEPLxKksrbT1J/vxVkOXwk5vu1Q17pcNrHrukCbGGuL+JfwKAW/pXTb16QtBeJrSl1FTqiRBCyNtB2WNCCCHkY1RWGJ5M5WgiB4M8zTTiPBjmDByLXd4Y4wySIeyXC696TTmj9XK51WJxoo4GeVFoy+aOzR0HhUGTdRFCCKHwmBBCCCHnFgTy/mZ8eJhnuXYc9t9/U200DNcz3Mo8+HzXWjsa5V/+IZgGcmXVWVuzNtaF8Ck8JoQQQuExIYQQQt5I2WlZqvlynGopgXN0OLaaxk8+sVst8c63f97saKaimTQFUD1kQgghFB4TQggh5E2UlY0XdZuAIXzyic0W0aVlou+/0/NfNRrii8/9IFRxqhEhiOR8JdVDJoQQQuExIYQQQk6pHGOcZhoAlAYpdbneNNlbqGN8URybOWtWOZFYGd4TQgghFB4TQggh5AzKMcaDQQEA9brxySd2u26UU3C9tTrGF6WszCwVf0/bTwghhMJjQgghhLwTOAfHQrfy42eMU4j7sMPRqOqWBc6p20+DjQkhhFB4TAghhJA34vv8zi0nva7fqTHGUxz8Ff+/oT74Ofy3jr5Kh4kQQgiFx4QQQgi5SMfnps4yFYUSAFyPNxv8R5/nWYPSOtc6NziXUBzCDgL7CfzirNvJCx3HOk5kmkjDwEZDOO/JCGpCCCEUHhNCCCHkLTk+N/VomD98OAOAmzcra2vWjz7Ps9a5lFPAyGKmBU6gxgUUGvRZtxPHensv7/XSg/246vMvPvedNYsOPSGEkPOjp62EEELIBxQeFzAN1TRQeaHftbYVkE1xMIPAAseDmgHivJ80UIOhHIzku/l5CSGEvHcoe0wIIYR8OAqpw0hLCd027zSNbluUnasdm/3o8zzHmPT4QYQTm3kN6A74gaEN1HjW/LHj4Ma6MBfB9SxW/aFmPN9YF8KnWbsIIYRQeEwIIYR8xI6PxZ3FGjS4FeZWeNVnjfo79ENfYBFiGGPsYtMGV4BgwOfh8RkJA4WPUhnTUI/GUilIUi0VZY8JIYRQeEwIIYR83I6Pxa3Y7Gc/99bXhON84KlUg6Pnol5E156LBhV8IoQQ8t6Fx2maBkGQ57lhGJZlOY4jhKDDQAj5SERR1O/3AaDT6biuSzuEXPzvusCqz6r+hz+3iDCg6rGyizVDmEyLLEXf56ZJ86q8i+I4Ho1GUkrP88rbP8457RZCyMceHgdBcO/evel06nleu92+evUqhceEkI9Hv9//t3/7NwD47W9/S+ExuSjlWNx2k92+aZaFjj6GTy0ENmpcqnmINRoXjx4lnMOdW06rReHxu2g0Gv3+979PkuSnP/3p+vp6o9Gg8JgQQuExRFH0+PHjJ0+ecM6vXbvmeV61WqXDQAj54JV9Z7799tvf//73AHD16lXP83zftywqSEPeXJyo0ShXElyP13wumsZZSzcpkAVm8xsCbTJ4n8IVzp+VcU5ipFDrnSWlzPO83+/fvXu33+8fHBzcuXPnl7/85erqKu0cQsjHHh4nSbK/v//111/3er3bC9euXaPDQAj54JV9Z/74xz9+8803AHDjxg3btj/99FMKj8l5jEb5l38IkkS9cWXjArMARwDgQ8PUznu6H3yf37nllAt0Vrxr8jwfjUa9Xm97e/vu3bv/8R//8ctf/rLb7VJ4TAj5qMPjPM/jOD44ONja2nrw4MH29raUcnNzc21trdFoOI5DB4MQ8kEqMyeDweDhw4ebm5u9Xi/P8//8z/+sVCqrq6vtdpt2ETnb7+linuqyzO80UEV+rhmbU4j7uAsAprZNOO1vsYQiw6SAXIPiYFi6cs4ixufEObouTzM9nkjOJI1AfqekaXp0dLS3t7e7u/vo0aPRaMQYe/To0cbGBvWgIYR8vOFxHMc7Ozubm5v379/f3t4uZ2j4/e9/7zjOr3/9awqPCSEfbDDzfebkyZMn+/v7SZJMp9Py6veb3/yG9g85+++p3t7Lp4Gax7QCfvZzz6uwN65sPINwGx4AQANWfGie8q8yTIa4H+Ekh9TR3gpcN3TtR/2W6dFEDob54UFu20gjkN8pSZIcHByUF8DRaJTnedmbptvtUg8aQsjHGx4HQfDgwYNvvvlmd3d3Op0CwHg8/vOf/1yv12/durW+vk4HgxDyoYbH0+n08PBwa2trd3c3/t7DhStXrlAPGvJaUuo813GiolCOprI/1EmqDY6tJl9ZEY3am/coziGdwKBcOEN7oIgxDHCUQixBtnT+buwlSFIFwKSiU+YdOW9lnufD4fDRo0ePHz8eDAZJkizvCVdXV69cuUI9aAghH2l4PB6P/7QwHo+XAfPdu3c7nU4QBHQkCCEfqqIooig6PDx8/Phx2XemXE89aMgZgthFdrTXSx8+nEUz7dct3xeei1Tvt1TOYs0WCWPOwbIodfyOnLf5aDTa3t7+61//+u2334ZhWK4Pw/Dbb79dX1//9a9/TXuJEPJOeRu/H1LKJEnKcXePHj2KoqhcX45F2draun///uPHj5frCSHkQxIEwf3797/55pudnZ3pdJrn+XL9gwV6REjOGApCzWetJm81edVj4k0fdGeQDHE/wJEDbgW8GMMpDnLI3s6nsLTVUp2aqiZwAe/LOdo2qzjcdniew/Z2srWVzGaSzpYfPTx+ru9MuT6O493d3bIHzd7e3nI9IYT86Iy3c3EcjUaHh4cHBwflmJPj/1pWAVVKURVQQsgHqd/v/+u//uvvfve7fr9/fH05Hu/g4KDsbUjIiSExNmrctuxuWyitLZubJjM4CgPeYLxxKcTxJv4pwGEDuwaIIfRySK/CLaFbb+ET+dq/U/ws0+nEOMjwYt63kDqM9NZWur01a9T5P/1D/fp1msj6x1T2nRkOh0dHR8fvAMs7w4cPH/77v/87IlIPGkLIxxUel2OM//jHP5Zz0jz3r2X+pNPpfPbZZ6urq0IIKhNPCPkwlDP293q9zc3Nx48fP/evaZr2+/3t7e1Hjx51Oh0agUxeqqxsXBTzkNix+coqF8bF9KbOIBnCfgrRiloTIHZxK4esCxun+VsJRQzhDAIFsoAsxVkG8ZkqJ1tQ8XUrhPEh3I8hOeX7nowztC0UguW5jmYqmqkkUUIgp/7nP5Ky78zdu3cPDw+P3wHKhbJf4ZUrV37+85/TviKEfEThca/X+5d/+Zff/e53vV7vxX8tiiIMw+Fw2O/3R6NRo9Gg8JgQ8oEENosZ+3d2dl7adZDyJ+Q0ysrG00CurDpra9bGuhD+RQZ7XKEjDQaQ85lCkCBPGVoP9P4I9k1wOBhTPOJgnKlyMqLBeRV0nmIGUJzyfU/mOLixLio21KsopdaIo4l8g1rQ5KK8qu/M8g4wWiiKgvYVIeSjCI/LzMnu7u4333zz4MGDV71mMpmU9ZBXVlbsBTowhJAPwMmji8v8Sb/fv3v3brPZvHHjRrPZpB40ZDlPdZrpNFWDQZGllzgRM9Mo5Dx6lKzQmGs41XuV2eMIAgDMIU0W2WOlq2d6Z4YmoijmgbE85fs+14YMk7JiM1/czwgDhY+IIitgMMz3emkQFOZNx7Zppq63LU3TIAiW88u86jX9fr/X6/X7/StXrjiOI4SgXUcI+ZDD45MzJ8vwuJzVsLxB7Ha7jUaDDgwh5APw4oz9LypHIG9vb/d6vfICSOExOV7FF7S+fdut1XjZudpxKBH6VFl7GQCasOpob7m+HIG8u5t/9zBs1Hi3bbSaBu2ut6ysbHzv3r0Tph6M43h7e/v+/fubm5vtdvvq1asUHhNCPvDwOIqixwsnzEpd5k/Kmnjr6+uffvppp9Oh/Akh5L12PHNyfMb+l76yzJ8cHR1Np1PP86gHzUcdGBc6jnU0k0Eky9GzboWtrJjdzvsdNiiQBWYZxAokB25px9TO6Qcqvzw8hqfhsafrDjwLj8sRyLbNOKNHCT9mePzamfnzhbKPYavVqtVq1WqVdh0h5EMOj8u6dscr3b0K5U8IIR/YreG9e/e+/vrrra2tF2fsf+4GcTQaHR0dDYdDGoNH4lhv7+XRTFkmWBbrrgi3wj6AKr4FZgGOQhwVkBna9HTDh4ahTcBzhccDPQ+PV+HG8fXHRyALgbUaJSR/BKe/Ayx72VSr1Vu3bl25coV2HSHkQw6PhRDVanVjYwMAptNpmSvO81xKuahSyJdZYtu2G41GOfKEjgoh5MPged7t27c971leq7wGlpfH4w8Bb9y40el0bNumJ4Pk+yq+aM7DOm5bzDLxlOFiiOMym2rCafsgIDLGLA5GBasKeQZJDOFyNO9FkVDEGCYYaQ0GWBZUTj+J14tyyGIMjmCvD7sCzAzSH9x7LEYgm0LYNssyFSdqMMh9n5smjUB+eyzLarfbN2/ebDab5ZzVr7r6lXeA1WqVelYTQj788LjT6fz2t7/91a9+FcdxeU1MkmQ0GpUXyvKCWHYjLENl3/fb7bbv+3SJJIS813zf//TTT1dXV3/zm98cn3yhvAYCwPLqV3Jdt9Pp1Ot1ekT4kSszn1JpgyNbRHOcnbaycVnHGABuwa+aevXU4bHgrOqg18IgwVkIYw7Gc6N5z0+ByiEtoDBQmGAxfa7HQDEGO7j5GO72YKuqm7nOXnxNWSn6aKgfPUo4hzu3nFaLwuO3Z21t7Z//+Z///u//fpkUedXVr7wDrNfra2trtN8IIR94eOwuHF8zmUyePHkSBIFhGLVabX193fd9OgyEkA+MtdBut59bPxgMHjx4IKX85JNPWq0WzbNAACDLVBDINNPzM8fERZ7zTc6Kso5xuXDyK8vs6wymNjqGNk3mCXArWJUgE4wMEFXdetXflvNFpxgbKCxwOHAlZZbHmY6VIeFYBJols2DUV0Vu2m5qJbExTVlY6FgxRwsF54hVc8imMBjBYajHJjgvLQrFOXKOjoWcg5QQp5pqIL9N/gJd/S7bLIXBdL7QqkLFov1ByDsfHr+oHGM8Go08zzMMg0bZEUI+KuPx+KuvvirzyYwxmmeBLEaqy/ub8WAw/0FstYy3kOcss68hjqrQrGLL1pVyvV7keHNITyizVM4XPcWhhU4VGlLLIsvC0ZEFolFbOd6nOxj1N//wuySctNdu8KYT2pOYT2M1NWyUzQyctxOk8Tu3nDjVUmqqgUxXvw/PYAr/7zfzU/p/fKYrHdofhLyH4XFRFGEYTqdTxliWZUopOgaEkI9HkiT7+/tRFI3H4yRJyj6H5CN0vLLxaFRkqT7P1sp5oROIUoiZ5kq/5re1zL7GEHbwiqdrBogCikV4rCVICVLDK9uTpOF+8O04P+DM4KhTHcbTWbiXaZF0b98A+9kduiryJJxMBwcMGY/tyIliHqUqNH1RVEJlp4hPB1IpWaRpmOipITjjBqLAVySXNSitc61TzRUCcjC4NvDVE3xxjq7LCyUHw4JzWXEYTQxPV78PySyB7cPFwk3aGYS8n+ExIYQQQo5XNmYIa2vmbZ8vO1efdWvlvNBj6E/VyNKOxEvsmTWbjLfv/nE02m2ZVznjoewHo8Pw0WHSGnzS/Vs41inbtN322g2GLE9m0/gwMIOUxVIXTtvOugPptzmvloFtniXT4b6t0K1VLMfjvIr48n6iWudSTgucapYzZEKbpjYZspP382E/391NbYt12wKAMpaEEEIoPCaEEELeMVJCkirbYo2GcZ7KxjlkUxyM9WEI4zIDfNYtcDAc7aUQZ5hInUmdKJ2+NIubR/H40d7kaL/VXbEcx1F2PONpMYtVKPUPwnLTqTTXbnBhhqM+ZEcpV5IBByVcB3/YqzZPk0m/h1E4qXCv3ml0bjnuy8PjcgbsGEMJBQduQ8WGCn9dxKsUZBlorWczRSOQCSGEUHhMCCGEvEPKeZWfzk3N4ZyVjXNIJ/poCIcRBgIcBWceuGRquwmrADCFYaajXI6ktl6axdWploeKTQz3erO9eqWu1yrtZr4OTm2F13/wYtOuNNY2vGanyNJQDw/ZbsBGOaQ10XYqa4uNi+/D43hysBtsZ3E8bqxsfPZ3Tcd9+dxgOcqJMZviLEfJwagw19Eu18bJ+7koYDxVea6DSFk0ApkQQgiFx4QQQsi7YDaT/X6eZsq2OeNYr3HTPG1l41eRIBOYSSw8qNrgBDAcwv6Zqh+X2WMbXFxMfD3DwAHPgQqD58NjtDjvOpZfr66ttlZuLO4k3AGMLcdjrvmDbRrC8WqOV5uHyljNGCBaKcSOrgvlMW09fV/wZtwpKilUCp2LPJXD3g7nlt/omHbluXdXqFKWp1iAZhWoVsB3sT6D6as+bzl/da0K3Y6YTmUQSkRNI5AJIYRQeEwIIYT8+Pr9/P/+f8aDQb66al29an/yid2o8VNWNj6ZC1Ufm4jY098lEJ2p+vFxOZNTnBl6Zmj5Yodv3rXs/6Ntynqjer1lzcPjApiHjxk3uDhz/3AT7BauQk0mP5/YN5zV7DoMioOt++PD3Vuf/2Nr7fpL/4oBN9HydK2tryiQPXw8gP0TPm9ZTfpQwMPvsmSGNAKZEEIIhceEEELIjyMvdBzrMk77QajJwbHQti+mjJMNTkt3Uki28EGh82tw5+TXM0ChmNAM9dNW6bjIJ2EUHyodZRgqntWd1YrfENazZCu63PAr87dTNUfX5ws4FujO/0kzOOMM3GX2uGLVVAcqut7Vt7WTjg93syQa9rbKmsmmUzHtCjfE07HHEOaQubpa0+0VuDaF4QSOAOCEzysMFD4mCRMGSqWThEYgk/ee0lBIzAooJDAETXuEEAqPCSGEkPdCHOvtvRwANtZFpyP+6R/qZedqt8LeYJ7qVwaBildlJQHQRpFjqvRrJujiCh1pONrg7GlpJDlK4j/2Rjv3B3kxMLxJ9erq1TsbP/uiZq1d6v4ps8EmWEzzSqNz6/N/HPa2eo/+urv55/bajebajcbaRtlJO4NkoPcl5Kv6RldftXRFYT/HdBEtvObzWhbrrogk0XFKNZDJe6+QGCUsmOkkUcLQWtHJTAiFx4QQQsi7LctUEMjRRE2n0rSYVLrq8+vXLywkllBkmCQYaQRT2xXtL1bjq2oXH3+90JalHVPbDHiWzIJR/+C7+4NvH4WDQ8evcp+fUE/4YiEgA85g/o6mXSn7VA96W0k4ebH9MYSgwdGuq2vLWs3lwmuCcIbC5Ekmg0gBoudyGoBM3uNrSw5HUzgc43QGngNS0S4hhMJjQggh5N0WBPL+ZhyEyq8ansuNi05XZpgMcX+MfUBtoit4gwMiGmd4va4iimC0v/mH3z16+J87g3uiXvnk139z9epnDb7qO52K33j7+81f5JCPd64+/zYLqcNIjyfzKBpRF5K6o5L32CyFnT48PoBBiNxAqel8JoTCY0IIIeTdJhUkiZZSuxVW9Zi46F/dsg5wgpEGzdHkaDOw8NXzTr309QCgZJHGoYTc6tYqG2331oq3vtZQ18oMbSmHLMYgxrACngBLgHV5+22ZQ34W2C/y24PiUeKMTNvVtjrrLQxnaFvoVhgAMITRqFCF8n1umoxOVPLeySVMIj2OMM4wl5qiY0IoPCaEEELedeV4VylhdUXU/IuZofoy2K6/euO2ueqtNTPZgqKhJ/qoASvHXxNjsIObAQ7buF7VrQp4b7OFwai/+Yff7QX3JyvT1up1faWA6tm2UM5fnRfzMGIyKba3030Bd245rRaFx4QQQig8JoQQQi5BnKjRKE/ixVhARLfCbZvVfH5RM1Sfh86VilNVxAAZGAU4T3Owpl1prl13sFW0IKhMDmE71JMC8uN/m0EyhP0U4qv6p219xQKnXG+C3YTVDJIQxkM8W73l00tUuJc+3Bt/y3JD61VoybOGx+X81U//j1L7gk5V8t6rmPpqSzV9GIfQG2LD07ZJe4UQCo8JIYSQd8ZolH/5h6C3lwLAyor5i1/4F1XZ+ALC4yjPt6f5ZKRBYa0CGwoWvaeFaVebqy4UWgBTez18HENUTnm1JEHGECktK7rqQ8PQT2/DPV2/Bb8awv5r6w+fR+6pyU+ziZnWdoAfKPzpubbm+/zOLadcoDOWvL+avr7eKhBha9+IEvz8Fqw1qZs1IRQeE0IIIe8AKXWe6yTR6vtpnxhH28bLyxtr0GoxSbUAU4CF8Mo3knmWxtG4t3v04NtwclRxa6BbsKq+b6dhOU87Swcw1qALyDWoH76XKvPJAixTO8v1JthNvZpifEL94XLG6QxSR7uerhlwhtStAllgljqZWjNgJtTDItkfD548MkXFb3TAeZP9Zpqs1WLfHy+qgUzeV46pr7RUksFXWxDn8Ok12iWEUHhMCCGEvBvyXI8mEhn+4hf+L/7LPEK2HdZoXGIvXg0qh1SDcsB1wTde/bOextHR7sMnD/60+Zd/z7Pk2s2/udiWKJAn1B9+sV7x6bdcYBbgKDVnXr1W9RuqOBgcPL73+/8ZTga3Pv/HNwuPjx8vAKAayOS9JhXEmWYGlXcihMJjQggh5EdVVjZOs3kwnCRqNJGWhRtXrFr1pF67Zf3hMkPLwbB05UwJ1TIw1jrPIZrpaYZZQ3fdRVaWAbfAWWRc81gGkCqVFUWWjfo725t/2n789XDWMysOq1ui7l7gVNon1x9+sV7xGfYwJEPcj3nQ5usV30hbbjoZTg53hGWt3LitGqayC8bP9kHKvPEkkIf9nHOoOMymIsgfnLzQcaxzCciYEOiY2vhAu9JrBUUBRaG1op7VhFB4TAghhPx4ysrGg0FRBl1Zrtst0W0LgJPuxMv6wxFOckgd7a3AdeOMQaPWuZTTBI8meAjasLXr64ahTYFmVTcyTBIMR3lPj9J0EIajwcHOt4/u/36cHrA1q/7J9erPNiqrHe5Y7/4eTmC2r7cyTDbUbc//hfpsv289uLf55SwaT452dcPJ28myZ/hpA6dF3viwn+/uprbFXnu8yPsojvX2Xh7EyC2z6uN6C3yHokdCCIXHhBBCyKUpKxtHMwUAjIEp0LaRv26scVl/eIz9AEYe1Ou668LZwmPQSum00LNUR0w7hjbL8cCmtlq6E+IoYqNCRW5iyDCOJ8N4NslZhg1h3Wy6P12rrq+5fsvQ5ou5XgOEB7UCs5meTnHgaF/AZU2Gy8FwtJdCnGEidSZ1onSKKI4Poi4gD2ECGqrQWnVWYaNtaXMyG6e6ME07Pce7KwVZBmkqd3bTolCNhnBsKvL04cgLPQ3UKETuAHIsahQbE0IoPCaEEEIuU1nZ2HLmYZVtsUaN16r8lPMhpxAPoJdCfA0+vaj2ONpeUytHIHeMHQOxwa75pld49dzYYGt2WA2Ta4XdalbtTpltfnELNlRW8foUBkdsN9Gzq3BL6NYl7T1T201YBYApDDMd5XIktcV5FfEVaW0hoF6v6p98BiBBV65e7zdHwjxzx2ghsFHjRQHjqTo4yL76KtxtG1987jtrFp3ShBBC4TEhhBBCTqscbxzNVJJIpcGxmevyZXh8+nmqc8gmephDHuE0g9jQJjt1F1+FumBaInA0ODxLtzLgNrgcxQxCgzHuWG6tmVkm075vdcb+6KB1YNiOpd3js0//IGRdzEQtsdiGBzFEHbh6eXuyzB7b4OJ8b6QpxhkkFrjlhykgT3EWY2SixbXgYADnwLnVaHdwcQNTr0+dnLEz38xwjpxjrQrdjsgyFQVZkigl6dR+S5IMekN008uq06s0FBKjBAchG0dYM2neNUIIhceEEELI5SjHG29tJfv7ab1u/N3f1TZW7HnQxeBM9Y2lljMdFVBMsB9Ax4fGq0LWl/wt6tgoMgSBFUu7/Pu4ulwfYyFRG6aAet101xqy4+kog7EQ5tiYvObOQJs+NAIYzzBMMM519nb2qmQ6xiLWhaF1eXeS4uwAtyZ4VMGqBzVTf58lXuSQny6cg+PgxrrwKlivojDQ9Wj48VsyCnFnF133sur0LmJjdjCBBz0dJnDbofCYEELhMSGEEHLhIdxixuM41fL7TCPn6FjMrZw2Y1zOOC0hznQssTDBFGAWkGUYK109fUsyTI/Y0RRDTzd8WTMzUEWYazlJj3rTzUNjJ+mEVsUF2+aW7wBYys4lRDA2XpdrZcBN7RggcsyeVmyCLMZgBlMbHUObL51ku5wxW2uJunhx/PBpKNA5U7lWWj2d/TqHbIrDGQYV8D04Vi15kUN+uj+lVlkOUmkVA0vmATM/bZQrDBQ+as2DkC+GjlMQ9ZYsssfgpvOFy6CUTlI1DqE3ZnGG14sP9siW159CaapMRgiFx4QQQsjbVs54LKX+5BP7+jUrSaRlsk7nDDnMcsbpFAcRDhUWbVxzwLWxAmdMoc0g3IYHuU5v6b9dybrOWOaz/alKdvc3v/nqf44qR/b/uV67sXIhnzrGYAc3QxxVoVnFlv2yqsUCzKpuzANjmUg9PWn88BkCZpnptIDcQEOAxV4Wb+tcFtMYkkSrAVhjqNdPHx6X0lQdHuQAQPNXfzCU1HlaJDFEKU8L9gFXAzYF1mvcMM/Wb4UQQuExIYQQ8qYh8aJ6atkXN830YJhLCasrotY0hDDPnLR5OuN0nOmYATaha0NFgUpgVtYNPm2rIJ3AAKR0UsMa6vH242AyCCE7PHoy2d+LmzORd87zqctssIQixXiqhyM4yCDp4BVP116aPbbA6cDVBCYRTDgaNTBtePPwuMyxK51IniGArV1He/xlNy0sBavP4sHsIN5kHrrXbzqtjhCCnzpI5gxsG8sF8mHQWstCFgVmBc8VfGDFgAsJcYazBDlDS2jXWWSP6ewlhMJjQggh5C0oq6cCwMa6KDONSarKskCNGj9Pn0YBVh3bDIyZnqLGBpw92ZvnMJ5Mt0fffP2/jo52QZjKgNrqqrvBteud51OX2eAYwwCHEfIMkpNf72j/KtwawN6OcW+gJ7egZZ+1VNUPwptcymmBU81yA0xPN3x4+TzbZizqu9X4u72/7P/Hk9ruJ1qvMaPRaJw+PPZ9fueWUy7Q2U7egytShnsDHExRGOg7YBhUsIoQCo8JIYSQtyWaya2teXDYrDPTKDONjF9EJLUIQVsK1BAO9GKi5tP/rQKV6yydDfe37xvf5YeH21ES1dprfqthNivZuh650fHXF1iELEwh87RvqdeXMi6zwRM4imCSY5bpxAT75M8idGuGQYgRYJTrDM5x017WhY4xlFAIsExwXjVpmcXdFftGxAf3Z18OZege3vAaLc+t2PZpqz2ZJmu1WDkbeZoV822a6PvcNCkfR95FRQHBDKMEOQNLAA06JoTCY0IIIeQthsdh8d3Deah5+6bZ2rDu3HKkAstilonnHO9naNPTjQLyAnMF6kydq6WWM4gOx0/kX/bdJwKFWL/x6fWf/6a22tVGPHQOE+e746+PMenxg0SHa2q9Bh2XuXBi28tssKWdR+ybqR4IME8Ojy9WjnJizKY4y1GKE8Psit/Y+NkXWUU9wvtRMImindlgTXa7cMbcdTkb+WAwD49bLePOLafVovCYEEIoPCaEEEI+euV447yYR2azWDv2PBIWBpaZxot6FwbcBAeBKVCLAZJnmEEIATlwzk20HbtRrza73Wu31m5+5reaUk4VGo7Rx2P1kDNMB9hXOl1XG029yl8X6y6zwbnOZhi6UH2b+1+hSlmeYgGaMTDw1aG8sOyatVbN142DSprvHR1+62u/sXLF9upnGoFMCCGEUHhMCCGEvEQ53ngazONVg+Ovf12r+WebofqyCTTr2LLb1p3/8YtuekWYllXxK34DUXBeNaHhYA0XFZCeBvyQTXGkQWpuc11FFO/+UWDATbRMsJh+TZTLDGa4Vqbj3Ydf64PCX7tp+u03GIGcXtfLztX0LSCEEAqPCSGEkI9aWdk4msloppJUCYFVn2+si6p/WV1tEZgBYlFeOM0wNrTJXlVeSErI8wRmEzGd8kETV6zKJ9ev/aqpV3+wQbQEuhWoFZjHEEUwsXRFgUwhBgSNBnujOaUZoFBMaIb60gc7SihiCHPIXF2t6uZrR0qDwdA3ocJzGWSzUM0iSBKQZ+isXvYLKI9+XkAU6yRVjoPCoJGdFylJ5P5+4qSzOEgFmpMJjBwwOArjaWmiPNcAzy+fdQK8rIBRxGYZNCpKz78P9LCDEELhMSGEEHJ2ZWXjIJKWCRWHN2rc97jjXGKMxIE74KYYz3Aa6JEPjVdNQ7WYp3o8ge0/1/6S2fAT9dkKXvN0/WXbFBX0Q5wMYC+HZAWuX0A7FTrScLTBGcIlx4wZJAO9LyFf1Te6+qr1skrLx2nBVNXgXb961V9lV1ecSuON8uPl0Z+GKoy0beHGuhA+hccXaTTO/vjVyPYHWPEw50+ecJkyz8Wqxxq1eRA7mkgAeG75rOFxlLJHfTYM1I124dqqNv/+0khyQgiFx4QQQsgZlZWNg1BxA2tVo+pz37vcG2sT7CasBjCc4EADmNo24YXweJE3TkdH0+3vnsCD7za+sRutz8R/b/LVl26TARNgMWAzDBjwls7P08LFOGdDgGVpx9T2K5PbL1NAnuIsxshEi2vBT3fjUWaPQYOjXVe/fpItLgyr5tWvXLkad6/pm432im3b8KYDj6XUSaIAmFRUMuetyguYhkpKbRiolD7s55xDxWHceFZ7nOHrs8q5xFGEs0xvVFXdA06ljwghFB4TQgghbxIeLyobjyay4hqc80Je+o11OUf0ATzZhD9OYNCAFR+aL9zv5zAeT7e/++br//UdfnvEgg6vqJqEt9JplAEXaJnoCt4467jlFGcHuDXBowpWPaiZ+lKmvxbCqtc7Favys9bnV/QnvlcHuwJCnH072Khxw0CAgnM0qGbORWvUzV/+suE0WtvTql+tXLsG11efda6OUxVG5bOJIs/U7m5qW6zbFoi4rD1uijfPKhNCCIXHhBBCyGuU81THiUwTOZlI0NqtsEqF2xZydsH332X94QQSR9um8gwwyjmipzCcYQgQ5pC+JMichZPtb/e+++vh4fa0Miqn1H4LOBgOeBkkJlgcTY72yeOWGXALHIWygCyD2NBmCnEfd2MMW3qtqVdfWh3K0IanPUQpWTaFYbFIHZtomdo24FQhLufcNiq2VenWbrb0jTf/vBw5xzTTSqooVLqQ1SpvNIRjU9fci2HbfHXVrrQqR9oyK6JW043as5OZM7Stsq44Kg62xWwbOQOpdJLOXyaVlgqTVC2WeZyo0SgvCm3Z3DDmx0gY6DioCpWFRTiWRxoU8ppJUTQhhMJjQggh5HTKeap7vfRgP7ZNvH3b7XRMbqBlsgsfdVzWH850UFe1GnQcZp9mHO9kOvjLN/9++GQThai1VvJmbFUdZlx67tgEu4WrBhoa9OnuKkwfGxkmCUQBjHxopBAPoCd1cU1/Og+PX5Y9drS9plbGwI6M0RE7KDA3UHhQa+Kq/bpRx5eh7EGw10vTRHa74ovPfWfNoq/JW+A4uLEupNIGR6WMbltwBr7P0ww8F8tp5I+/fjTKv/xDMA3kyqrjegIAqj7bWBd5nAW98f62HjX8bmbedig8JoRQeEwIIYS8QUBosVbLWF25rLpHy/rDq+pKVbf493MyC7Bq0MohjTGcwsDRPqRqFozSWZBn6cHW/b3dh2kSXbnyWfWakzd6tuVyPFV4nEEyxP1MJzXdMsG2wDl9a5fZ4xTi091VGBXwAHSC0VEyGg3MoY76zZHjMKEtR3uvCqqrupVhksLeAHohTKq6tYIbr8o2nyxPk8V+C4s8FZZTba1YzvPvm0MWY1D2bH9xWmzOwLbRFBgGOkmUkvS1eEuEgT+cC+3pGa6Uqnplcnj+f22LlYfpVYpMhsN4cKAws3MUnqkbFXa1hYsh/DrNdJqqMvA2zQ+hXwAyXIwIAJx/GhpoTQiFx4QQQsg5lDmrdpPdvmkaBjYal1gT+FX1hyvgbcDtKQ6G0MshvQq3dJBt3/3y8MnmdHgYTodJNG2sbFz/L7+Bm5VZ9UvBbK5P9SMe4ngIBza41/SdJqxWdesSb9MXU4IhRDFEhyO9/wcxUpP483h93Toh/1zWagYYp5iFehKqiQO+g1UfGoY2z9qGWTDavvtlf+dhMDmqdtY++2//V+fq8+FxjMEObgLAVbglXtghZQ3kbsfsD3LLRNejskA/dti8GBO+LPi0XG40xBef+y92rpZSRzM5nRQGxqC0CtGQ4idrdt1lo4kcDPPDg9y28c4tp9X6EMJjzsAx4eRHBoQQCo8JIYSQlyhr25a311muR6NcSXA9XvO5aBqXPdnPq+oPm9pqqzZivo87GnQO14//QtsVr9paaW5cs6+3dMdoshVDmyZYp0kUZZAM4aAG7Rr8bUdfPdttNxiO9hRIGyqO9pYjgZf1gYvFJM9RWDAGtZpIGB5GZqAs4eRBHw73iwnPeFrkhREXKihUIXVRaFnoolBFoU0Ty5G9iBaiKEAWkCvQTHMTXjaD96mls/BwezOJpjdvfwHtKyDE8YmsF/tkHwC6sPHi35Y1kA3BkkzFsTo4yOJY0gjkiwx3uW64yjDVJMKjKfqOtk58JFWOCT/+f8sFh+Nz/d6fnpkKnardaBe2h5ajxPfDEKSCJFVBqEYTac5wMJKFgjSRDNH15gF2ITVn+Nz82OepwPySMzOHIEaloGKBKcDg+vzzG9gCVhuYFhDG+jT7kxBC4TEhhBDyVFnbtsxBlWMXk0TdvFlZW7N+xLlwuV5UFcZFfL5oQsVvbPzsi+7123mWaqUYN1I3nTRGGcuq0Kxi6y2MyzW1XSacNSgOxrL+8PH6wAf76XcPI1PA3/zSV4b+6r41S6vXrmihzQqvKF9kBstyPZoWEOZhpKNQzqI8CosgyFut50f2cjQqzHW0e8rc+IvK/QaMHe0/0XECoxGMx1CvHw+PJcgYonLhlWEMjUC+NK6lPunksSx2Do0wYZ9ugCX0BX67c82v3mpVuso3pW0zYZvrq4bvYdnxmBtYcQ0poXcon+xkB/uxKfDmzYrri2W96+PzY5+nAvOLghjvbWNe4NUOtKvg2so8d90pvwK3N+BoirsDFsT6AvcnIRQeXzpEhmggWgA2gAXAy/lCv5+G4dloEGS4rHFXzptSLmulg0BKBZbFGMPlcz6xGHRx/Hm2UrDsZnP8KaDSr99a+Xq52IJSz7bGGBgc2eLxMWfPtnZy+4+/u5TP3rEcKFJup7zglu0//r4vPsWk9p+n/cdff3w7L7a/KJ5tjXP8qNp/nm9T2S/x5PaXk+6We6Ms5lG2/1VXgw9jbBh5d8RFtjcdA4Dt1gHelTl7mEZDoSE16FxhDkIJy65ZawBry9cMsHeAvRjCDl7xdO2UszqfK2hfZI/Lq0Ga6Wj+rcx9n5f1aaeBlBLzQk8DaYr5dYYfu62wHba+ZoZV67Ayv2Lhy3a1kjpJdDSbX2JirqSlubLNmevACrftN7tJKfdbOD7yqs1kdHS0+8gwHefaTbvZFkIoLmMMJtBPIDLA1KBe+dkXI5ANjsPZ/JMWBcUbF/fYxQDTVJio/gAKDenqhW15lsJ2H0axcWXDWF/XLE85A+6YrQa3TMWZtC1Wqxqc8yybH9A8e/nzkfIMBwDP5UrpwTCXcv6zVf4mvvhbf/y3+GRpDkeT+f/WPaxWwDn7aSX1/M/zQluGEgwQtSmwVcU0h94Asvwi92chIc7mH8oxtUGDDAiFx5cTHnPOHc4lYkNDFcAo5wtNUu25mGdyORpEmHxZ4w4Alst5Ju9vxkmiuytCmHz5nK+cy+H48+zywlfOYXj8KWCWv35r5evLygFZDsutmSZ6LpqLGxLbYsutndz+4+8eRc/e0XX5cjtPw4NF+4+/74tPMan952n/8dcf386L7Y+iZ1tzXf5Rtf883yYAeG37p4Fa7o2q96z9r7oafBhjw8i7I8zDzem3ANBtf9puNL/43C87Vzs2KztS/oh0JovJjKFWLXmOnsUXrLwaHB+xyQwWRlpKXFsxGjW2eOCFV69YZi1W3TRQU+HkPvNbeSsUbl61DY7Nulh1xaJztSELs+xcrRQgw8Oj+eVibMu8oXnq2ts/sfUNftWd3ya8KWFa1WY3mY7ub/6hN+lf0bqLvNFoZEa0g5v78HiGwcnDsMsRyPW6ePQks0y0bAoO3gPTmP11B9MCv7gNvql7PT1Ln32pyzHMnsuXj32Lwrh903yxc3UYzX/7FvGhzjN1eJCXv622o176W3/8t/iyFQVMZzqKtVfNHQMWU/Rd1vvGGe4N5htfb4Hv0BMiQuHxpRDIatywTatpmh5jRpzIXi+dTGW9xuNYfvdd7Lm82RSOo3d2U86g3Zz/7vZ6qVTgVTCO5b37cRjJnySO4/DxRNaqvN1kbgXzXE8CedjPjwbFeCJnsS4K3W4ZFRsqFXbYzzmHisNmsdraSqSCig1pqu7dj2exylXFstTBYe67rFln6PJpqIJwfk8fhvP1s5kyDPRc1moZlcq8Sb4HnsvL9geharWMOJabD2YVhz3XfimxbD+AOZk8a3+txotC13xuW7YQz9o/DYqi0GGoBkNZrXLbqnjuj9l+rdnyueloVCzb32gY5YNVxqB8ehpG8/aH0eJ9I3VwWHguu5D2h4tlz+MrXeF5z9ofzeTWVhJEaqUr0vRZ+y1Lffco4QyadaadRfvjjI+zMFL3H7Ew5ytdw3PZCe0fjYrl2ZgVemc34wxsy85y/dzZWLbfcdRgUPgeazcZ4rP2z2bz9WE0vxGsVNibt3+x/+ftfzCbTmWryT2PGQZWfaNMIwuBcaLm36ZAGgZOJvKN2+84GMd6MpVHg3w4evZtaja4sTiIR4NciGdHMy+0wSFN5F/vxQzhhPbPZvP1R4PCMLDiYL3G263j7X/+alCr8vU1s9WiQVTkvMr8Z5yoKJRb4fhAP3FslrPrjs3ekR6zWZKMB/v98ZPxdM+s1GJvklViQ5sM3iQqK7O+6fzmNil0nuk015nS6sw3x4u6stOpDCI1mcrBoKhVeV5AxUTbmv/XqHGo8WKRgWs1BavGqy3pYJZC7Gizo5gNwmHzHwuHMd96/lFXkqhlgGGA8KDOVcVKrlR0l8tzHRer4nev3crTZH/r/mCw7xzuOrWm51YSe3YAT/bxSQ5pFU4Kj8sRyMxgZfaMurG8F9Ic+tPFj6yj254eHsHsWB3xcgyz/dx35WW9ML6vwDxfUIt+BItqzPMrSZIoACaVPh6UZpk6GmrHwmWPrbNmlU9DaSgkJjkmGUqlXWv+MQW/xKg1yaA3nDe+6WvfofOLUHh8Gc/FwVJ6hRmF67ueXzGEEYTFwX58eJgHDTEN5F/vRlWfX7tm+T7feTKzbbx90wSAg/04SXS9ikEg792Ppos+nFWfj0Z5tytu3zTznI8m89hmdzc9PMzmP+eBjELZ7Zr1Klarxu5ualus2xZRWHz3MEoSVa/iIjyL4kRX66ZlqQf3wkad375p2jYLIz2ezK84g0H+4F4YTHPX462WCeBkuVhkwnUhdZrIg/14NFYAThDK+w9mjo3PtT9P8eHD2eLn35tMimX76zUeBEW3I7pt4dhs2f6joywIisEg3+3l7baoV1mrJX7E9iPA8rlpv58t29/tzA9NtcoX/zv/1+lU7u6m0+n8X48G+f0Hs2qVX0j7B4Nsfu81b79XSHPZ/nJro7EE8NJULdtvWez+3cC22e2bJmfGvP29IUt3jwb5l08aU169c7vSXgRdr2r/YT9bno1BaNy7F5btV1o/dzaW7fc9vbsTN+rzdzTNZ+0PpsXuTjwYZFEo/ap48/Yv9v9gkH/55eToKL+yJlot4ftGu22WXbgbNR6F8uHD2WE/931jPJFv3P445tt7+WE/n4zy8fjZt6neKPeYMRnlVf/Z0SzPhzTVf/kmMgWc0P7ptLh/Nzg8zFyPV30eNESaPGt/eTYevxp0O+K/fu7RlZqcX5n/7PXShw9nB6w/+3SvtmaxSvrutDCajB999acn+18f8UN/fW14pediy4eGqd/knrQcMzz/0sEw1lGkppauSCzOup1ybPbhYW7ZXJicG7ziCr4YZ1H27SoHQC2Xz7pDy2yeVPPrsMerQtyYFYX2O552DXGuW5RyBLJV8ef7Nhjn4WA22JPdbgbpUB0O8dBipyoZZXB8acVd8mE7fobbFr9zyykHImW5Big4R4Pj8bN3NC4ePUo4h2WPrcvIKhcSo4RFCRZSmwLrNd6oaSEwV5cYHu8P5zczN9fopCAUHl9agKx1rlW2eFzHAcxF+ovnmfI9jgjdjnBdblpMzNcz2346wLXqc1MoYeCiIqWwbVarcs/loFXV5+VrFs8F51ci35uvFwbYJlR9tnhuN19v28gZlFs2xfxXWcN8a1muF9vHRp3PX29g+dTQXWRZs4Q16tw0lOvyRp35HivX29b8ZWX7F8lANm9/V5gCX2y/bT+t0Xe8/W6FK6lsmzEOx9tfrs9S3qgtPt2P3X5+7LnpD9tfbofxH7ZfVvQiWJq/r+teTPth8fPTqHPf/UH7y63N2+8yy8Rn7RdP2y+W7dcpP+iJUVEzXF7jx/bDy9tfq/Ll2bjIu7LySDHA587Gsv2eyxp1Vp6Nx9sPar4eFLdNcL1ztH+x/435ei7z+ffFrcxPlePtZ3x+ppXrtYaXth+TrBIOWap4IUxLvLT9z56d23j82+R6/OnRsfH40SyHJWsNrZZxcvvL9bKYnxiex33vB+1/8Wow3yBlbMhFQyF5daZq0Qh3quh5uv4GxXXf/N0B+SIhvByLmyWzYNTvPby7u3lvEO4UV5mqQGRMI5w42itnb5ZQZJgkGGkEAZatK6Z2Tkgsl9ljG+aBXQ5pDFECsxOmoXr6EGEx/j+aySgshIGdjlheGx2bud48Nm61DMtkz9WnXS6fNTw+PiOxQIfhamKoolotImN3LzN47nqGW+HLORFO+rzgScxjiCKYWLry/chtSGdBMOxz03ZsmzMudRHrKIbION0tkDCg6rHZTO7uJQbHU7aHvO9+eIbjcoxPkqii4OWJcfzsTWLkL/s6Jonam0nQ8/sKQ6Bl8+GUj6esUJimEIUqGue2pU85L7rSkBWYFfMFMf9tZba9GEF9aeFxISGMny4QQuHxJQXHkZJbeToNp9XI6xRto9Hwv/jczzJtGFgU+r9+7nEDWy1hcOy2BeNQ1p9cjg0rpO62DVnoedy1+JOyLET5DK/isG5bZJkqCl3+Z5pYqwnDmG+tnObHcdg//UNdKl2rCaXnW1MKajXBGNy+aZY3BMLEjXVRTiCUbRi3b5pFMW+huRh9dLy2nhDPau4VxXwLjMGFtD/LVJpqYc73hinYj9h+zmD53HSlK5btL+MWw8DlNF1F3ei2RTl/SZap5dE8f/uzrBytylzvabxUtt/oiH/6h3peaNebb2fZfobYbRuczbdmGDhvvyGcUbxiqs5/qRVXOmdq/8lHs2y/YcCyYqphPGt/Oa4py3R5CN68/cf2f55py0LTZItjyixrHlovK0Ce/G0qeuPP2I40oV1vQqd6Qvu7bb4cH1j+V7bfMFAWpjCeHc3Tt78o5nusbGH538ntN022tmrSlZpcwM3u4hprW3a3LXaY/3XdCKH/Lf4pw+QW/KqpV99aSxhwoa1yoVwTjPqbf/jdzoOvBr3H0MTGp9erdzaKmo4gqMPTfG+GyRD3x9gH1BXtV3Xb16etCaxA5ZhlkCl4zU10Of5/ayv57mFU9eff7u6KufxWGgbjBlomKzPGF39Hok0fGhUOumY8Gef/+39Popn+yU33+nV7OSfCq5hgt3A1xMkYDjTIFbhu6Noyh1xkKTImTMfxqgCDNzhzJpOztYd82FeSZcGnpXKkernA+bOs8sFB9vXXwcFBBgCub6ysOkFhPdo1DAOutxTMsq3HUcVhNC86IR9veAw6Bz2RcpClRZY5ShUnjPtq1J817/hrOq8YiMg52nY5jOQVD5fLH1ETrl/nL9/ainjxWfj8D1fECc8Xj7ftypr4INt/bG4k3nnNQNC33H5+fGvH299qft/+NGvrAOQQshGYVusTAT91z9r+0x3NV7Qfztf+U+//FytAvtj+ArI1GAIHxyqMljix/a9/kv3c0TxF+3+w/vTtJ+SNlaNny4eAjs1XVrky7O9ATCHt464D3jW483ZakkI8xcEUBlVoCSkglUF8mETB4ZMH+4/+Go77Xr3j3Gh7166ZnToiz3W6nFdZQhFjmGCkQRtgWovs8Snf10SrDp0GdEz9yu/X07HZqZI/jKDPOjb7B2OeIUtxlkGiQJ5mBDUDPv9QDMACx5KLvFz5sFIfDfI8YyfMY19mj1OII5wisKZ+2hN0mUM+dh9ytqNWZgjLebmDUI4mshGonGaxfi8gosGRgVZKP83gntdzFZiP/Rqy4790y9fYNrKXvT5JoXekB1Lu7uV1n02n0nGKKJTlg2ytdL+fz+8xOsKy2LPqDxKSWP7/7L35cxxJlib2nnvcV15IXCTIqmKzOFUzfWzV7uyOSas1yWQmM/21+l0/zczKbEfStLa7pw82i1XFC3ciM+OO8HB/sogAQZAESADERSA+ww9BIJkZ6eHh/l58731floGUCJciyqEUleX+QTehOnTpcYcON2V/jCJ8/BgeP6aNLVgYgaxu82hwkI7Km0i0q5TqcCvQds+GkVxatldWzLVVHQPUUGPIKhACijpwvhSEOPkT/ksJ+T16FBR9NcnXN55t/vzX6ebzaLoTjJYf/Pp/CL66UyyUOeatK++5wIf+GO+MYMWH/nHJ4b5bbEVLYz4eOge1PKf9rMM9zylEIe4mEFVYnraDenyosibL1IsXxaYOH9WxVyBLKkoq1Hmvb72e/qtf+5M9SdjpV38+uz9j3DRQVyQqWQAp7SSPfc8XbVVU/o17UFy9HfJ5xV5sw+NXwITmM8O0KEpU+nP+9GlqWez773xR0j/+8wwA/st/7i8tGW/cH0qa7lXTPSxL3bEu47uICqJENQdduUSHLj3ucCpIWUdZzcM94N3eec1QFjCZwM4OxDEEPih1S+dh81fMMq0qgDOi7klwhxt8K7xx7Q4jVYm3ZjsCMuQcNR0MRrwpPD67RvTJUUC2g68AwKXekJZD2NxPKS1ntHJ/uPLF3a9/7a8uRjid0U7xhn3NTlhEfWzK2hQej2DZJOvtwJeyjIpSyYrSVE7nUtNweUkf9PjignY2MaG3ep6xyF+zx6d9H8d5U1kzmYhNvV7b5pECJt80B73X/UtACioFFR31GKDt3y4xN8i0weWnCYFch92/ZwU9OZ0pw9i3ne9wHe96gaaOrL7D650w8DEvYXeCYUpZgWWFGid2iYne+/UXFcN+gNMEFIFl6nd8c2VIts3KQh2+N1sHRFHRYQdmIVQ4F+EcKuBKsTxXeU66jpWgaC7DGVUZWAio2KdbPRUCogx351QIYAyirP5n537coUuPO5wYQsBsVh/0+1163OGazsP2r/MZiBK41Y1Whxt9K7xx7QaCb771HBvb4mrbxrB5jQ5GQAMHvBzjCKZn1og+G3TDCobLlhMMl7+oRNEkya4/GLf9twLKHXjVsq8cNB8GF3EObafxZFKlicgzWQryPH6gJM+vjVBz29U5j9Q8oq3d8sCD/bTdv23/doxTj4IKFgUWJy+0bjtOG5WKfdXi7i67nnd9mqJvo2WioYFt0OoIqgp/3tRDAUsJd11wLWVoV/90eODj13f5go8LLvoOmSZTig5UQpDJrx64AOB6WiXpwIFZVpQmIk2wMqkUNJtLR6NBj2epevkiW9+s8gJ81EDanx7hRxn+5QU+3UBgoOuwM8dgQp37cYcuPe5w8lW5gjACJUHToOOQrw9aNrUs68thWmiZZBhwgx/7lyVOJgBAjgOWdeQsxdkc0hQYv8kseofu1leQFyqK5WxOrsO+uGeMBu+uyQaZIzUOIChYHOLEIrfViL4cMK6Ztmfanj9YfO/EbB1MAHrNvmaKgjPu8aR55BnIXfQNsBnwlmFrG4yTVCapSlKZpkpWZOh4WEn++uC1/7CchiJJW5lGlKfvhCyKdDd5tideSIo1g6QPcGJWvu04lQpsq06MO/b4GiIrYWMPpjH0XBwEaDU8p2+TY2GhWCEwK7Gs4Jokd7YJqwu4OuI9Fyx9/5wOVEKEgPv3raZsgbdWF62dBOdo2dy2USFKSVmh8oKkqu/rPJN5KlRFVa7CUPdcdlBh0daJtCZVJ1dcLwTsziHOoe+RxmB7hoaGfbdzP+7QpccdToiqwjiGIm+lijsO+do8tmj40jgGw4Ber/5xXeA3dtpjnuPWFjR9exQER8/S2QxmISC75T3YHTrYZK3IZY96u9qUEAewdDO/o1pSoCy+YKKvkVE2DFvryCoqMg0YjXi/x4yGHfU8fqAkf+3ilcZ/mAibKtOz8LdFFO08+evm5EkFgkYcHrkw6u6Dm4OkYD/tsDjD+8vszhicz1nn8Thfccb53TUXHXyxx+GQED3X0HF1x2245aYqRBIeVFi0dSLt+5y25sI1acEVhYAftzVRsa869+MOXXrc4aRQCsoSpzOYzWk4rJMxqytevQ7pccPqxzEgQr8H/AvwPMwy2J2Q74N54xyDqgriZP/guFkKQP0euC6G0Y0dhw63Ei0vmqRqPhdpqoREIvBc5nvsSMJEI+4p1yaropLw457AZzyrSpR5WqRxFs83yx8ieGU5nloQ4JxykwEloFCgHPI96mknU63VwAioTgE5BQzM/Xd6jdfOrvsO8IMet85V7EdAMaMdScKHIKDhB3SzT4LWf5gzEIIQKU4k0enYMAlVBnGY7IrtGEPTvrt62vS49ZAvSzWdVXmGH1DS7nAFG77EaYKFQMvEwNkv5ru2IAJZKSXpSEHt43zFJaHf40EFekSk0DaZZRJnoOss6OlxjqVigEeXN4iK5pFsXc0Zaw0dsShUa7p53EzWOfVdFaeQZARInWZ7hy497nBKzOfw/AUOhzQew3DYjcd1SBcxjjFO6nRxMKCHD4EIt7chDOHRIzJvJXHQ68Gv+vU4bG5CWd7ecehw84Ljhhd99jz//e+iLKe1+87qqrWypA16/IJ8ek+CMk+nGy+2XzxZf/pvG5Mftsrni2sP5f+cwRenTLNBpBQpkCNaHdKSSSdKrxF1zoP2YD/Y1d84sr6T+J07Y5xQuKleOGR/qb5cgS99FnyKWlB75oaO07lMUjWbV66jTsWGoa/zRwGBGf/wHHKmZcOzncPuHv30U875x5W0O3Q4DkoqUVSiIOXws8loGTr2e3zQI11HBTgYmpky5pXUNbp7F++M8WDda7noeSQnU7n+qvjxaWLo8Ktf+57Ht7eEZWE3kzt06NLjC0NewNZWYxSSd4NxPfYfVWeAVQWaRkFAd+5gEsPjx/Wf7t+7nUNClkX9HmYZPP0RiuLWjkOHm4SyVFEk56GczuVkIoqSOEfLZL7HBj0e+MeEfcgZmgxNxAuJC6UoiyzZ23rx8snvd189DSebRRorPFFTgw5GACMGrKAspqlLrqA05XMkbpPrUu+kCSEwRLMdn6KsP9o00PfPmSU+DgXku7AxoGFAozHc4Z/W2n3gN1s2vdOVPDWLVV9ty8SBTkrBtIKXOQQF9BQcU+xFoIhE+3ABG469PQfbxK59qsMnIi3g+TZVkhyHnDM9v2MMLItZVn0jMIWWzS0HmMk1nYIeBf6bG+SAi27VsN9BqwlfKVHkUtNwMNA1DbOMwgiqimeZfPkij1KIY2bomOUqz/cfpR24MfNOqa5Dlx536PC5TXONPJc8DzQNKglJW358u11/qwriGIhu+zh0uBGIIvn4Sba9LUpBts3+098Hg4HmeprrfIg3RtQ4DzTyDpjVc04Os2T31dPnT3731z/810oU9x78arD6kA2fuqNlvvCRRNEm/y483KVXz9ifE5p4UpMgMzZHNM9QBN6Oz2RSp8ejkXZpTJECJbCsmGLocQjOZZxb/tZzeSWJMzxDXQDnumMHMCvYf08pC+nfCTimnZJISBk2/yVAfFMZ3ipptwfd3dfhbNiL8I8TbXVEiwu40L+MT2w55GGfff3AOCiuXlzQ40TNI/rx52xrMwt8/v13vufpL9bF+gTTjE0m4vd/nYcxyYBbJu5N5TSo70EAOHBj7tLjDl163OF4MAa6ARehjXx9fJXP90wu7nvte/ymmOd1Euh5EPiga9TyyQB0w3SbhYAsgzCqD3T96HEuclCqaZ6zqCjqX5YldfrVHT5PHHY2nkdKCOAcbY6jofbVl9Zo9PFMDIEzNDnaBtgSqwLSDGKDrJPb4bY+uo3W9BH/SxTZfGcj3N2oitxyg8W1h/i1s7eQ67bHlP5hSyEdDJ1GKUY5ZYqyUsX1G1LBkBGos43PZYIDt8CxwGmspDmi+brz+ZPfueFv9frbgaggTtR+kqx/OFGXFZYCCgbMdwa9+w8VpqUsIazg+F7KEvI5bgJADwzr0Pm3StqtGnCUfMiHuUOHt+YhQSVRVEiEuaDNaT2Zc3GKOoiqgjClOEPG0NTpVGlpyyEHPoOlw3cLtyyVFccuEGWFs4TPYmVakGVqd1JtO3WQqxS9fFU23fiWruOBHr5qKjvayOLk90UlISvrl3W+yh269PjmDSdHz6WL0Ea+Pr7K53smF/e92nfe3YXZFHSjTo/7/SPyxhuDLMOXr3B9HbOM3v+a7WjMGsdj3aiHArA+6NDhM34i9MbZmCGsrZmWaR0UD58mkdMd9DOM5zAhoCEs2+Sd8P+2ProAcOT/kpXIohkH9sUv/l1/8c7qF98mw1w3rM9rfM4GHcweLgxpcZOeG2Sw8y5fP/ztLBPXVnX2wdW9wjLCaYqhBvp4uLb0H34p7ifPdh6jY+LxdeYJJj/xHwHgIYws6L236NKLddFWq57Nh7nDbUMlMclZkmMl68zW0ZWtA8dTTJu0gJc7sBuCziFwUDsPA+eWVV4Ysq8fGAfF1fU9ZcL8OVqeufxgZMWUVbwo1PZW6TFqW1r+8pfYMtnigm5b7EAPvxQQJ1SWdKr7IitxfVK/rPNV7tClxzcCh3k5y4KlRfA8TOJz0wRu3z8McXsbOD/az/YykeewsVEfWNYnnclFfy8p61NNM8hyYPzma4mnKTx7DuvrICt439KpHY08B6nAZGBaYBXAOx2ODjchSQ4jaZns3l39fWfjk4AB08EsICswzcCQdArDswKyHXwpoJRQ9WFsk68fMtLluuEEA0TmDcbeeNEYB5kjXexx0E6oO/06tatiFmtgWOga4H/0/7adxllBLYH8ieNz1vTYCGjUh0UP+ibYHM75o9/xtRbVR0SxBZQhTlKMLLJte7By914ezLf8XWAmHJ8eCyynOG14+/J9tl8qygv6FB/mDue34Te1EiVYHHWGtsENDRhew/QYogyTHBhD2yTbBFM/3Vacl7C5R1EGXy3DQo/M83jm/4ZVfr22SEmGDoaOjKHvsX5gz2P6aV3pIGy70co+/obOMrm+IWYzWVW0MNIcCzjXD8T/pKQokm1jwmGt7DSHF9v1Bet8lTt06fENic7e8HKuS/0eEOHLVxAn56MJ3Lw/bm/jy5d1gnekn+1lIo7x6Y8AQONxQ0LelO/1uSNO4Mcf6/R41MlQd7gVaHtQlYJZeGUNAjkkm/AsxL0I9pbhi7vwUKc3N6DjD9a++V4KoRlmZVWxHaYYDXHJJs8C9xSfgvmGtuVBr8+WAhhbH9OsbjuNpYQvv7QGPXYl46OR4cOgD+OADRhxTlccbAgo5rSbQuhg0MeRxReEx1D3ABmYZ0zdP92HucP5BWI0ncssVn2dDJONXN21mMav3QMLUUGYUFqgbULfQ9tmug7ITnGeZQWTECpJo4BWR2AbdHHjOZtjKVjfg4d3KEwgzhkD/W8esYd3yDRZmkkAjzNwPX5YD78s1HQ3/+nnPInl4qLRD7CV5W6t45KkXp0A3lV9T3L4abP+Lp2vcocuPb4ROMTLkW3ReIxpBn/5C2TZ+WgCt+8fJzCvV8Sj/WwvE6WA6XT/4CZ9r89+HlaQJJBlxBggQhxDGIJt3+R68g63FWkqd3ZEUSrL4pUE12aGcfa2Tw6aTZ6AMsOopExSpsg+UCo+Pt2qXx/hVDWVMFPYMsBahLW3EnjT6pn7sV6Iewl7mWHkgB/A8DDJ/AEYYA1hWUIVsYiAr8HqkJYNOFEhDOdgm6gbfNBTLUF0mdeIATfIttA1wYZ6KBmcaxjf+g+bulJShqF89UpZRVGM1HHa2BJkDmkJRQ8MCz0dHWak++VdhO+fWwWiwDSEvQRCDfQj5dBaH2YgiiIpSqlUF1BdadoZqyRVpo6+A7ZJhnYdyXylIC/rs9V5Pft0DRkjPF3oRFlRHzjmBbKsQmKUYZIRU9JzcXWIvoM9DwH4eMwWx9S2abR+VLbFDlTlAcD32KDPwwG3DAh8dqRdnJSQFRTF6kBgTwg1nYomM9cButK2Dl16fJPQdnUyDmVZ3/2dJnCHSwPXwHWh16t/GMPNLagk3b3Tpccdbh52dsQ//vNsMhHLy+bqqrW2Zi6M9DM7GxtUp6AIGMFeQnuF9B3S3lEqfh8ZRi/xSYR7C7jqQbALGxkkH9CUbl2Lc0hMtHUw2cniP4/6D+E3LgSP8V9TiC3yfBpo9JHU+rCuMjJcW9Xb9sIbtdk2bFWRq+eF3Nounv8sveX0zr8X5jklDAWmW/jsJT7Zohcu9QSVx52DKNXz54IzXFzQATpNoatBJSlOKMmRuM5Njl169WmQhFmll0parHI5aKhxxu1m4TkoBW/nf3tw+P8OBvr33/nf/I1bVWQY2OvprY1cW1zdrk5t68f6ljjQDqCqpHgGAFT1AazuEnTo0uMbBNZ2dQrg/A2rfOVa09cTomGh53No/FXOn+1s3z9NIAig3wfDvPlzz9DrAQx8ME1I03o8xdL+XxVBKbDxf74QZfUOHS46XGt6C1tl1KxQ8rXnra5Dz2ets/FrdeJWoJi/0wl8HFr2OIVINcRMpTIFBScFH0wnBZQhTDIZjcsVU2iJ3KuyeCP6UwF7bm9gOYFpupr25tMVKAGFgBIBOXCEEyWrBlhDWs4xbRljgyyDjs7/WhXlLJcHzqX265baK9SLQmBtpzSeNx3UslUNT8XyjEeRIoWy5LLkdNLR3b8yShWVTCohEZhuWKxR1mx6lffmsJtDaqBdQFpCppHBDiXA7Tm4DnMddjht6HD5aCwpSFTI9eaqoALoWsHPDiIUiklFGioDiSG3dFgeYFbCNKINEwceWcbRdse2xewV8wN37mjE8lxN5zIv1fvXsf7TrEpiCQCuxxnDolCcvdurfHhHaBPv9mRabfCm94FY1+7QoUuPr9OgcnDdpnI4htns6rWmryeaHmbc3b0otrPtkS5L+ttv4c5d8v3bMfc08lzwXIyTt35fVRjHkBdgmfv+zx06fFZoe+FaZVRdZ//wDz2GYFncddiBDnOrThziZE67BtjvdAJfCAqlJoUWQi9zp5uvfvfX/8Pg1v2/+371i2/H468Op8efgkbmatAeHPeaVkV5Y6M4cC79QIR6aeDA7abLml8MrdoyUffWzKqiDPVIK9IoUh47ufAZkZAqzLPtZJ4yNIPhsml77aOWkgqF0kLbAD3DMKKpD4P3H090HsgdbgN8B75eg409fPKKbU3pu4ewMjz7A4j33ctRM9AbiFzNEqZ+zp8+TQHgwQPHMNn2lrAsfKdX+fCO0HY1t+lxqw1ep9aWup4F9h269Pi2wrJhZaXR65q2HrM3SjNZKSgFlAXu7EAQnF2du+1hns3r9Lhljw3jDdv56XjdI01+AAu3Ra2KGCOjdd7ODv8eZYVJAkVBw0Hr/7z/EEcpqKquxqHDdUbLEkSJms5lnEhZke+xtTtmL3h3xtYpDWYJzqew3bQT3z/5p7T61QrLlGUmxD72jlvXWq/jVM4LESe7E/V0pu8xJJR7yWTnOXO1frU2wHsSz01PwQR7DHfbgyMeHDS88XQuo1g1JYvX6Nq1vdPtwYW8f+M/3B5PU5VNPCEiZXKlyxKyEt/je4GbZBtkH/xSyqrI4vlutfX8laY5hum26TEBKag00Ee4bIA5h4kFrwyyjPcuAefourwoaTaXnMl3OK4OHS4HDMHQyNCQYcP9VlBW58mgGjqMAgxT2IsgySEvP+nd2sqLdlFoHY+jHCTT0FDAxIf3gja7Lkqa7IkoVlxD22RFrhyHmSbLBHvVGETdGanWIOowt9yhQ5ceX12K4vvw6BG8egl//BPO5p+q8HztYtVGAmp3gn/8Y51ZnYs6d4eLRlVBEoNUb/yfLQuWlpr6/6yrcehwndGyBPXPTIWhTJNq0OPn3ufZuh8LzLe1SU6kwciAo0tOWq/jHfEinGztPv0x+X9eaTMcj++Dw5wvh/pSYD4Y8iUH9XM7PZv8u/CwPXj/ry1vXIeJnNbu6F9/ZfgeGwyuhejAh8/8nHdeoYt5ICChvl5hGePUIOsdvlcjw6OBD03/dhMtSyGTWRo+nzz+f//FdILR8he9hTfKuQ54Y7gjoJzARgnFAJZ8GB45Pyd74kiOq0OHS4rmObmWci3UOFYSGndl+iwY1NbxeH2CWUmOiYOedm+h7eTfL65eXNDb4ur2XgOAQa91YBbTuXRcjXP1vJC2hYtLeljov31af+XvHsiV4bvccocOXXp8dTCNOmMMQ0ySOpP8RIXn/Z1fgRB1JpOmV8z1NS0+EMewswMLC1AW1zCahizDPAffb1ptO22q15x/w7bs1zJYFqys7M/SyeTq/bQ7dDgGrcOtEKTr6DocSVkWnnufZ+vTW2C2yX4WJJeoOK51sXXQTdSMpWgmRlIQQ83x+vbSwPLHuGjqi550FZ1f86MOxgeqxEVFYaTyQo1HfDTggx63rOuSnn34zM85mmHc0+0C7YxxCUWBWQmZoqBl+zOIJVQcNAPstwukGWOmqLJkPs2iaLL+zAsCy/fJLIgrE+wx3UkhfoVPFZCAY/c7KaGp82RSdbdshytAyx5bOlgGpAVLcogyNHUwrn2Yn5ewsYfrE0gyck0wTRz0tUH/8Hnvh7t5/ubuakoz0RXMcbhUlMaESK0g9vqOkoq+HEtHU0WhLJO1ciu6joogy0hU1LAG6n2lhg4duvT4s0JV1RlplsFksk8AdlzfccgyfPkKZjNYWaHAr4erw/uwbbp7B7d1fPpjPa863+kO133C4uIC1zSUlaFr59/n2epX55D+jH8WKI5UKt5PhBoNaknVCJdHwXDlm/uW7q18+Y07HFVaFVvhjr1RvwDEpSaiTS/foMePtFG5JTNkbVV3UH9pv1XX3rL9e7hZQm7Tu9uBbljBcFkueoOFcTzd2fjhd6DS1UcPqzEQExoYHg04GtrxXd/tyLfxN+dgml2c3eHqYnoNAgcJIC0gTKjnAJifQXq8uUcbE4gTNXCBFH74XmsP2p5/UQHXMM9pN6hz4OUlXU64rpMs6gS4ZZjbYFkpGPR4KeDFugijOs1OYnGtlBo6dOnxrQAyhkazoTL26RQCVU1JMwDFceOx3FlGfWCtzWFjA/Kc1u7C4mJHih63z9Q/cQxFUf90vtMdrhOyXE2nIs8argARGfruSXlRCVUCYQllBFMHfYMsfoKdrtWvtsAhVBWUBEeQgK0ydg5JASly1reX3QWn8oVpe4OVNdvrNf7GE4GyhHwOu4TkUf8i2m4Pj09ZgaGDZXHfu0a88eWDYVMqlGnz526JLOi7yjGJc8mLDOMMY0lHrHKMa6bt6Qv+nQffbj//azTZBF0693yOnoRKB9MAW5JkeOzjmAPH16aySkWxrITqOpAvDa1ScS5YLrlCDGzwHbq16pMcwdRB5yCqOu1UZ61lkASFAEXouSzw6UJd0ysJcQZRSqUgJYEI4Bjp+cPuyq0C9uugT7VXvOfzvZTpBkkFtkVvVOXfvn3r+zSSm5vFj0/TYZ/fv2e5rnagjw0AUSRbsb3uLu7S4w7nPrSN9FF7cD7rR7J/0OGj6fHWVr0tPHy432fboUOHzwrTqfjX30Yb6wUALC0Zv/ylf3JeNKd0k14QgM+GOhlDWH6fMzw2xQKuk9keHLEMN8rYMU4rKLWGdez7C6QU45phOe1r2m7bPdh8hU+34MVD+M2Qli90fMZj45tvvdUV/YY5G58WbV/iy+faX36/oED2fr2wcC+gngb84x1A/mD88Lv/yQ0Gf/pv/+dktt5Te7bGBEr9xM+2W15rd49++innHLoO5EtDq1Q8z9hcINNgcYgrI7KNTrL400a1gjClinB5UV8ekm2jvMYjephVRga6zhjD4QDXllqVCjBNZhqo68g1WFvVdU7hrNzbK9fXiyTVNrcF097oYwPA4ycZQHcXd+nxDUiHGo1iGAyugCpsHWXLEpV6a/UwDBgOoCghjHB3cnaF5/1PaXqP24PPHVKCaEaMc3CdOoN1XYgizHPIUsqdT+2srmTLtIOmHTEf7KbntiyxERj71Oty3cbT0N+tVmj+CmVZDylnneNxh2s/nUkIynNSr8MxxtGy8OS8aAUihrACEcH0OM7wOLQa0QVkIUw01AMaGWASiTyJZzs7MZunS0nplSY5Ph969tCxBu8Gak23bYZJa6ScwNzDnkbG6/QbdcV0Ykh4tpHJcpXEcn2jmM2qvFC6jpqOwWvn5w4IyEBTEosM8wykd6wF8mFPZsNyRiv3s2jGNS1MJtuTF86oEL509JO2nLS8lm3Wy3DbAJnnqtPLvYwQTFFeqDTHvOKWQa5NrV5xhzOHtJXEtMQoRURyHBb41IjYXd9zbu++vITtOW5MICvBNuotoxfAOwqOvPGBFyUfDfnSWIvvWa7LLfvYgLOt06kqMi3OGMqK2taekzgwd+jS46vGdIr/+lsAoO+/qzOfS0bjKItx8k55KrXKwLsTfPaszt47hecDCAGz2b6Tk+PQ/fugcZzPYTaF3d06p73QzurBgL7/DncnuL0NYXgTrsvh8dS0dz2ND//VsjrH4w7XfjrTdC6R4S9/6f/y76hxyrs8HWaP+g/hNzvw8jk+3oRn38J/HKmxlOHu5pPf/dM/7/BN7X8JFtz7D+nfLdG9D6gxt36/BWbpa7/c/d8rtKVmk8YZAp5lZDY2iqdP0zxXg6H+1QNn0OP9vnZNdKqvFi2DdO8XKl+MwjxjUiakKtL0D16jw57MjGuG7aq02vzhzyZt+49WB8PTeQ22/ZCNvVZ9sTq93MtI5ySJohIFKqkBdKP9ySFtw8aHCYZpnQpWnw8jM43xt0/wz89oZ0+tjD7Uw9zep6srxr//zuMajka6obMDfeyWN25ftjsR//rbKIzk0rJtGDxNROCzEzowd+jS46tG1vSatgdXEM2VdfYbRTAYQBCA/npIOSfXxXlYJydCXEeF56tC2xs8m9Yj1u/D4mI9OET15UuzC++sbhWbieCnnxpd8eJGjeeBMPU7fw0jcl3o98kwugnY4XqifQY/j+T2juAcjnQ2/sg7gMwhLaFo5Fo1fnrnJwOsIS2nGM1xAlIWxddFgtPZz69++MPGj3+ceHtGvujAyCYv+KAgc+v3m8A8h3QXX6UUFZBVWOlkNr67FjvNubXOxkkqo0RmhZKKLIutrpgrK+a10qm+WrQMkm/g0FHVXKQb+TwuxEBxUAIKBbJZHJ2DWcFBs8ETWCQ0N9CyyTcdb3HtoXIhduKmi/PUMW7rwxzFan1L5KXyXN5JX1w0iEhWUkok4l16fA6PGwjKCnMBhWiH9/MJhUrY2IP1CaT5R3qY2/t0NHrn0dmbNfmjNdWHGeM0e8uT37GZrtd/FRVUktqKT11D20aGcODb3KXQXXp8C5K9rS2oKnr4kO7eAdvuhuQjiGN8+iOUJf3tt3DnLvk+TrpnB+c0noMB5TnE8bt/rSr41a+6+dnhOqN9Br+9I169KiyTncHZWEAxp90UQwddHQwdjU+yVhICZvPoxezxX/7rq5dPQZZe4KmT1V+0Hcgz2tliz3ZhvUIBiBroLo51PuAUIJ6C722djZNUmQYMBvrigm6bzPW4bbFbq1N9bNhaaOnEmW2r2TpXNpWLoDdK4wIKD/s9WGh7yw+crkMoN9hPCYV34WHbgbwsvp3rk8iep356tnOoJMUJtQfdFenQ4fPFYKB//51/qLh63zfhMGOc5fSOJ79p4HQuw1jFCZVlvQgEPltb1Q0dDnybu/S4S48vCSgqCEMKwzoBuEw1pqbTlQDIceCwNQ7nzaNqG2yrPi7LK/YrPlVQmGVNiHfUSDIGhoGWCeZZv1cp2l5x8gNY6ArOPxmHx3M0gp2do/9qWdBZN3W4lmj7u6JYZQWlqeJNs/EZnI0FlHOYZBB70LPBM85qaaJACSqLdG/zxWPtB7Hz4nlWxMaqZ3/poxv0YKR/7J3bDmQCFcqtPcw28SUBLuEaB4OjxU55Yq2zcZKqfg8Dvw6/WjnWDu+DQCmoSlkmObAyf7WVuCya9uZolS4FAY10OOgDZ/V1RIxp2kye+4E1Gq3ct7BPzADQdGW6ytNAOy0lyRlaJrYH3RXpcF4oBEQZRhkFDpkamZf7aKz1VbYN8EzJkURJTeh37VhQQ4OBS56psgzDiNk2forstm2xIz2f3nJgrjcsJqs3nvytV3+SyiSlsrEIRKwTY4awOxG6Do7NGFNRJKUC02RKURLXafPBQ88uee7S43OMsDJ89QoMg+7euRZixboO/T5UFczmdcIZxzCbfR5+xa1vMMDRI8k1cF3o9eofw/icvleHDh2uJVod5ulMDhashaH+4Ct70GNncDYWUIY4zTEbw50Ahmd2VJIkU0i2Z8/lv226z3XU9f79u/zv+tb9UTBYXIB7DpxICtsma0UtFbD3jD/OWDGEpe5aX3gQbwhtYarJKU2cvan6/x6jM4vY307HpuFQ4MPgQCbt+FlUzGm3oHihGvRhbDMrOmWk2jowtwfdFelwXogy/MsL3JvT2rBaCCiwOcDlPSbTOLmWGrlqxRdlobIYp9eyz9Y16W5QjVw5neALxtZWdd3HCwjwcdDjUvFWMLeSVFXaG265akcMPRdUU7FHRBtbVZbJ+VQEPltc0CsBj59keU6LS3pZqKdPUwB48MBpW2a69LhLj88PRQG7uxgEsDi+FvVMnAPnZDtgWaAUtSzrZ+FXnL/u5V4YHcE3MgaGDqYJlgmIjeaTCa4HXYtVhw4dPm3JtE0MAr441s+mw6xAFpBVIEywbfA+YV/UPejNuBPacxh6dwYPrHuD5IE0x/0x3FugOybYJ3sfI6BRHxZd6CmIdTD4iTtaD/qNk7hKM9I4Dvrcc9EyGe+Y4w8NumR6bvQzf8GYp9XGZsardPRVPgJNB9Ogj1+4Qua7cr0son5mcQbow2mrEHQNdR/TVK6/KkVFrqe5Dv9EFusG3u8MdJO5gLamOFFZnJGNLCuYJiwtsedCPwDz5qrUpTm82Ia8gL+9C2tjcMzLffDUsscmBQ5ECpJUhSZcw+5624TVBebqNEsAOC4sQHAhu9WHJqpSyjLfWqZLAXFCZdG6MLypjZKK8pwme+LxX+v0OOjrlsPjuHIdNhjomoZZRlLV67+udR3LXXp8NsgKk4SS+B0F6Q5nSY+3tuqDh7/4yCurCuMEDJO6Me/QocNZcbi/y7b4lXNuLgZf4d/aY3P2P77y894X+n9kjvGD/8eKxAkZyBaIOueBD6v32DcxzgMc6WTiyQiftt/42bP8x6eJbeF/+A+9tTXjIEjq5sxHQmSb371j2bn11xd6FrGeOMWI5SLZCJ/F8x1rSpppaPdHZyvS39kR//jPszBSXz1w79+3LojF+nzR8m9mibO5KAVFIVr8LGxkUrCfdlgh4Nsv8O74Jrs6lRVMQlAKHYcPenQl6wAyxk2DMpnkwkzoGnbXGwYbDg3O6GUIIqGiwqua2y23/DphfpdhbvWxk1RlBa1v0Na2aGg+mk7Fi+eZ6/Dvv/M9T3+xLvKCPBcDr9PE7tLjs0EpKEsoxU1wBr5aHPgGV/JkY16easyRMWz1kw8ceg0TRiPgHNIEplPo97uLcA7bWFsFUBT7fuD2UQ95NQ08F0QFaQqX37ff4Vai1fwsSioKVZaqqsgwcDDQ7XNSYOag2eSZ4GQYlZRJyhTZiDqeuBDRInsZVtGu4B7oZHHyNdB8CHRlOOAbcFJ9OwSGaLo4vAMPMoh1Ml3qtV67HwqCSxVFch6pVtOlDbZ6fh0bdZPnpCGyzryBbq5Y+R03xsJEk0qNTsbcE1AFoqiyNIrjdOb4OxlLyFVc40SlouJUc6nDsfdp07EJDC1NiZKSjFsWeh6elo0UEqdJfV17Li0EdLNXzqzYXxAsCy8ulC5EHRQwBroG+PbnICJyRgyqQpQlXcNwu55Xdn175opIkCQCoMs/h5PksaMRc101nct+X7t7t574/b5mm4wzVJLynERVvXxV5LlaWtKlZHFc6Rpq9Q/j7QFHdmgp6lyXu/S4w2c96Ti47v5BG474Pjx6BK9ewh//hLM5jcddhnwOmE7x+XNyHPjyS1pdqZPk9wPB1p07DGF3gqW4Ln37HW40Ws3PyZ7Y3hLTqYgiMRrp33/nH6mAcpbUiKwhLbsQRLCX0F4hfYc0zgPEk76/TrwnHYF+xIOM5Rvwo0fecjUOYOQy97RCTSY5S3BfUoXANNAN+kj8H0Xy8ZNMCFhbM5cXna8fGLqG43F3Y54uPLUtbbhi37UGc4E/mz2ZMHI4nGAUdd3s98dMSnPPFXE+efVTXKS0JnRPJ5lICk84l8Zj/b/85/7h4uruuhyb+CnMKj2ruKRO6/uq1+cKwoTiDA0NHBO07kHQRaLlmb95ZC8uaE3CrGscFxf0UhAy3NwqXj5PpaJ+j00r+eJ5lmfK93XX0xxXdz3uuWgcWtM61+UuPf580Og8Q1liVVEpQF3vpV9KEAKqat/+p6rOWW27KDGKIIzq9Ng04MCD1zTIHEEYYpI0XsSimzhnjgrBsuofHkGawrPnMBzSb34DKyvHvZ5cF4oCZ/NmY+ykgzpcOIqS6tx4W8xDmeXq3AUZDthj1dAtlcoUFJzUsWlts+5VoixEDpybjs+1+h0cCEywU8ginGqEq7AypGV+epUFDXSNeh8LSSnLSFT1BjGPlBD1utvzG2fOpS4xPjUQQdPQ8cn2Ci3B7clIC3WmWx9OjyVUJeZSE4E+NH19MBzrAue7G1EyB62AsTEzt0zd7oFhnaDe2jTZ0pLROqNyBp2I9QdAgEIxoRiRunyu7zhUErI6doO20Mo2SLsFBRx5CZt7ME9o4MHiAKyL7HButbjnCTiGGgcACm2TqorynE7Fgu5XJBXIketafa9dz1i7Pc/Dvsctz2xZbHzIhHnQ1/JcTedS17CV4fBddpwSfllSFElREdfQc7mm4cGao+uoCA66l1urOc6w00Ho0uPrMMAaeR5WAvIC4/i69+gKAbMZ5Nk+05hn56tKjVGEjx9DGJLjwMKIrE7O67zR6qUrBbN5a74Jpgmy6wzvcJ3S40Jtb9W5ca/H7941fJcFAR8M9Ktd94r5ZCvcBstauve10xtwHmiQHLgTI+oaCziczq/45Gg7jcNINQwArq2ZPf8s2t0d3pppmIYwjSXn8xWXejxwP/z6EvM93Ixx5mN/aC8u31kr5HT7+ZP55itWOSJjz+79mFrqIYws6J1gWr1xRu1Ync8RWYnrE4zS+qr5Dq2O4AY3Nh8gyeGnTSor+PtH9OXKxX7lVot7FqoFr3Lugu9otgFZVk3np3MGbu+1KEaNt65IoKrrGWLTCX2PW1bZMq3G/x9cr94LFhf0StI7xdXTqXj+vN47HFcTAgEquym8b9ecUsBB9zIAxAnV+0ung9Clx9cgXdEg8KEscB4CIsiKzppmg+uCUqjUBbLQZYmTyT67qxROJiTlPht5Pu9fwGQCSQK9Hrnu+XtBNY7NmKZomqRpoN2+4LLRSwfTAsbqJFmcujO8Q4eLWVpUFMmiaaaNIskQegFfXNRHQ70OAqyrKeAjkIqKLJmEz3+Y72zs5XNruDBcWkMYIZocbQPsVqTaAJszm9FFMSlSUV5QkqpGWoYPBlrXaXzGrZI0jzxEKVk5g3QGu6XyNLnAst72OhVRceAs+n74U0C2gy8zjAMa9bVxP1iOe8ywXZxC8WxWqYjGnjboCSpPTnC2dQFZXge1h1mjDtcfjVI07kXg6CpzqarQaRMPAwYeWcanvr+pwzhQjgVRBrsh+jZdB9ltUdEsrg8ChxaCC36AJWB3DkkK90dq4ILtkKhwnuLGHln2KQLPooLdEMKEHEPaFrgmSwGvVb1GW4kwj3Bnihonx/1IGdIBqzzov1mnDh8fCquZ6zApwXHqnfSd5eX1ziIRseFNyHWYqKhdlw7M5w6Oq4qmU3FYJvPW8sxdenzR6XHD5uV5naXI6syq2miZuLRYv0lVwYWx0JjnuLUFUUyjUZ1ZPXuOk73PqRO4cWzG3V3wfQgC6NjpDh2uB9p+2smkXrh8j6+sGIOBZprMNPAKdZiJKinDvensz//2T7PNTTPoLZqWel1twUF30K9QAIADPqcLDF0bh0wkqofCc1HrMqizovWangHb1aa7uL2t1jkb3XV+Iebyd7/PdA0PnEXfD39ySDbhmaRqmb4Y0rJBluX6y198nYv4hz//t6RMg28fnmbzx/YZx2xexYkCqKqKdxzyZ4SWR92bqXsDIXL4eVMvFAOAlSF89xBWhp/KUgS2+vZuJRWs72q5wL9ZA1O/va3Xvk2LCzRN2J9+1qICFhdgcOL/mwu2FbJ5pvq2XOirkc8Rr9eN1lYibE/57pw13/QE9ScnHDefP3poiwreEetqi6tFVW8oiPXvy0O8WluvBACtQ/vBcRyLf/1tFEZyadleWTFvM898g9LjpowZTPN6afC2bJ5hgJT1z5l5PK6B6yJjlOeYJGdnoT/ygKuCOKkHcDwmAJzNQYjPqBMY99nvEFwXBgPQjdtwDx+hBN6hwzWGYeLCSBuNrnR9ThVM8yKZbIvH8KLYWH9apuWdpVW3N+L6PkWsN67FetNlapOnwzmvJ20fWpKq+VzkORGiazNdR9dhevfg+uwxTX3VSswLWJ/B7px2Xa47vuQhk5Lqn+M34QpEDPNGm820yavnquUMV+6nRbgzfa6cXX6aoKLlf1wBrqPCUM5mlZLKsVn32PZzQcujxjmZBrkWhAKKxiEsTOHVLuXlpzLJjgnDAcQZ/PElJAXcGwMEt3e0TR0WAlAAacnSEoQ8heJ0pTAuMBdgBlVgKcdgsUbXiz2uIEpxGuMsJWRUqXPT0zYMNhodW36laxR4rJXvKgUgkmV+iBCOY/nDD9nOpMoKRoAcKc+5abKqovlcKAW3x8X9Bm3CpkVLSxD4N1mDt+q8nT+2AiUxFAX1++R5oN2OGPM9JfAOHa4b2ifcxX1qlPjw6rtqJ4L+ZR6+ePpnsckLLNJssHT//t/9/eqDv3P8fdLCIGsIy5KqVu7ro4rTpw6+mz60Z8/z3/8ukgq++ca9e9ca9HidHnfOxmdF6zUNMCuwzCGRUBk6DnpaAAZAnfHevWMMelzX8SSbqG5YwXD5joVs1d7WXk7Gk9Oej23j2qq+rcPTH8s8xaaTsFuoPyfYFltb1VaGuJTwrEmJkxyevGL//Sl9IpOs69jvcYUQZZgUUHZhXYfzxmEH5tZ7uZXmOuCN3zkuCtraFptbZX9gWCZG87Lf0xaX9DiWv/9dVAq4PS7uNyF/2GfPHBsCn0zz8jR4G71TLEtsKeJL4O6uj7ezrkMQgOdBFGGeQ5ZS7nyixvX+dZQSypLOpphdj0+jvK1pYOhXKRXaaoCLppyes4uqZWh6rSHPwfdB194ogbewTFhaguEQzI6t6HDJ05+EoHr6SypLVeRS03Aw0D/wkPv89wVABppCVvKqIMFQHfXZjKFpu35vwV1ce7h47+vewhuN91YB++LGJytIvq524wwsi7lO/XNVndg3JD1uvKYR9QqkAKGgDgctEz1XC3r1sCuFpQD+duxDoEgVhCUjRGQH/siMa6bt9R3Ox7aGQQq/OznlU78nCc7B9/U8Z7qGUlGeqzxXXQfyuSAtYBLCzgx6LjgmWMa5x5atx2wjlDAC193PYCchRUmd00oJs5h+WqckA99BxzqdunWdqJhM16kQ9XyRkj6Hx0+kM6UzpaArVrtGyEuYxvWS8k4twwccmA/ntwfHlsVWVg3DZEtjree/2YmyTK1vFGlKbmC6XqVzEiX3fc45CkFFSUWjQWiarBVZuAEOzDeCXmvZs8bJ4VI/t9V5juM6LbGs28Xd2TbduQOOjfM5zKawu1sP/idqXLfXMc/rIT1XxewrQDs3wgjjGCzromoZml5rCEMKfAiCd5XAez34VZ+GQ+j3us2jw+VOf5rOZRirOKHpXrm1mQU+P0dn45OAATfQrDCPsOCUaiTfugNHOv5Dr/ebv/lW/v0irNVZkOP5g/Fljo9U5Dr84YN9x8teT+9444tDI+RPYSRnoRr01Nqqzg5NCCJRqZAg1+vc2GRvE7waGT4MEppreIoMjEhIGTYRamCa2uKSnueUFfWl7zqQzwWTEP6vP2Jewrf3aG2xTgzO9/05A9vYP9A4uZZqVZxtA3oOxDkmOdvag9/+lRDh4Rq7vww3Xt2aI9laZWuyYgTQzeHrgmmMv31SX45P7IpfWTb+9/9tVJbKdblhsKZvFU2TgaJBX0OmNA2jSIazcjTkjx7arsuncznZE9tbAgAWl3TX5TdDq/+K0mMpMcsgzeATLS9bH90kgdGoTgYuuacnz2Fjo04OB4M6l7Ps23Mrkq5REKBlAhFkeX0p8/zjV5MxMHRQFjiNKOE72a9lw8pKnVXOpi2ZcoEXtD0TUdWp+EV0qrdd0JM9zHMKgrPXMrQzHIB8H8z3IjNR1SefptTvwXtK4GRZ9e+HQzDNbvPocJnIcrWxUcwjpRtX9oSr6RweKpQhm0sCXy06h2MGh4FXLzGL6tFdengpm94bx8vW+VkIoBH2Ar62ZnUulBcSyoNmo2uDy1FrOeQkxSQhzpSo3lIhLyHfg00E7MGCCb1WrvytRy1ka8JQWSmkTLUk13Nd1/kHH+CWkM9xEwB6YDDm6wZP82pvWgnRdSCfDxpl6fqg9w2tDM/znVs/3jiDgY+mBpZODMHQ9lcQS4fAgbKCJCcp6S/P6lfuRTgMoOrdcE5V5+TblOcQxiyvMM3J7aohrhStBvXWBJ5vc13DvPykd/M8/vAXR+QyyVD76is7jOTSkmYYPE32K1ilgrxQSVr/KEW6ydOcZEW+xxyb6fqbXa91YH5fK/va7n1Xkx5jWWKWoWVh+UlXct9HtyxpdQXs5paN48v7GnGMT3+sP/1vv4U7d+sEpsNHppsGrlenwQsL0O+/k5HWA/joEbx6CX/8E87mF6uY3Z5JGOLmFlTy3NndfQ3wyV6dmgbBp85wAHj0iMxRN4M6fBZIYvn0aSoq+vWv/IWv7K8fGG1x9WWeg0nOEtwnoHX4KYLZMnwBML7KIOaQ42Xr/BxGahaqxbHeuVBeEAw0XRYMadEgs+0BdmzWarS+O2Mx+UkLfRiswMMRrXp0xNaj0jJ/NsnzfBJs2b3hYDD4cHpcvyf/EQAewohJL05oMpFpUg16vOtAvuZo/XjDFFdGuBCQ7xwVRDR88vIQvvmCzWL0HYRbUG/c9rLmkj+bYJLT7kxqrKuGuEq0GtQvNzHL2MXtI4OB/v13fmv4xBjKytA18H0umnYD02SLS3pZgpAQH1rlTAMPdr3Wgfl9rexru/ddEXucFzCdom5AUXzS+7Q+ugDgf029HuzsXOq3KAVMp01eF8BCl7p8JDaELIM4boTzLLKdI5hh06gzwDDEJIEkuVDFbMKmMQJxvzb+3DvV8wK2tuvJufRpvPH6ev1jmvVUPzkYq78UKXCcI1j6w+B8n6XnUTdJO5xzIKVhz+dLYx1Av4q9TdeoZ4KTY1rmydbkB1YKw7INyzVtFy5d1b59yt4c8KY4BkuBnHUx5QXCAmcMd5bwngWOrqHuo6hI17As1WQiTBJlTzXC5FBimWDIyPJpOKTlo69gUWY7s9nezob9J2PEbPqFNVz4gEaGwHKKdYQgqHQa7tp1OJLiHONEWZa6zZ6i1xytH28hYGUIowCNoxawlk/2LFzoXTh32rrm5iW5htI46Md/nKaB71BZQZhiIeBU7cwtZx5lFDhkamQe1ejR9rIaJkrErFJJXuUFtcpPHT50BRXEOcRZfSnPF0kOzzbh501KCggujKSzLXZkb5RSyjJZo7heJ8BxQgmTSMqykLN3dkDKC2oPlIIwUi31XZYqiqRUdY6tFCVxPUAH7vRX+NjlqtLjDDY2iDHI8u62uSUPuPDlK9zbq5PkW0CzU5HD1jZOJtA7Y9/vPm+8vo5RRKetjm5NzpyGpfe8DxHjrS934xbfTdIO5wXX4w8eOO3BdTifaLr75Lf/NJmPBkt3Ryv3F+48gCs1fTvsVGkarC0263DucMi/Sw+X4Qub3tp02t45c5T1HkkwT56iVGkeTbZ/xnmuDfcWEfvspIobLXe9uMBlZaSZmkeUFaKrGuhw0gCqcc2dRTR2hO+AY2gARwv42QatjurUfRJCmNKpTE5aznxvTmvDaiGgwObHfUqHUyc9JWzu0eYUzt20Lynw6Tb/aRsN6woWk/eVsatKO+CWOd/3fj9wYG5N/g87MEeRfPwky3NaXNLLQj19mgLAgTv9rUuPqaoojiE+V4OiwzyYUlDkcDb149uJVmm5LOvhspveYMOAhRFoGuY5fXp3btslG4a3cWzPNhvTFJ4/x5Y6PvWKpe0/RfS8j/Rvt6LrpgWMXeBd09YOSAWaVp9bd1feRLTPgLNcVRUpBYOBblnMvlwRZgTkTc3qgfLwvr/xj7vbj19WYtHx+5c/Mmkqd3ZEFEsh0XG5Uh9xquxwfvGNEcAooNGhpREDn4nyTe/cycF0rvVt5mliJ8ym2+lsN4tXDM/l8O4aW0Ie42wOuxw1DjyDpNRDR3OCpowijFRWiO7qdDg5GjYb4wwHPi4EH7JZ1jj4NoVWvd+KCpJM5TmdkIXLC9iYQF7g36zC3QVwzM8idCUhKM9RqTqVsk1mmcTZdUyPt6awOYXxeSullhVMEzZNYHAV+dwHlLEPXvB67aXAY218SlSvw+2CLA612B6qsYI8V+uptCwcDHRNwywjqUjj2ISQl8EqX5VydWO6Uf+c50OMNzyYKGE2qyP+z1r9+DJxWIXbdWFhATSNADBJYDZDgJvpI31x89u0cGmx3qks62yzseWfYWu7fp+z3QvtwSnmwIXdNa3Cdp6T59V5e3dX3kS0z4A3NsooEoO+9stf+q277GWeAwOuN4pLb5SHG39j+WQidwQsLQ4b6ti0XYDdSzurnR3xj/88m87k2n3n/n27kp0hypXhMItb6CLxT9FVYgTu8JsvKle4iuxsGMlimocDOX5fxybG2RP873PcNcEy0J7BFoFcgvsa9Q7O4UCipkOHj6KqIEwpr9jCyBgMwDBOtIaUgmZz6Wh0QhauUpAWoCT6Hh/06LNQ0W81HWZzLAUzGh/p63nmbXq8PaXAuaV3fcsztwdce+O0bJn80UO7La5OM3ngTp+l6g9/iBjH77/zPU9/sS7ygjwXA++SNLGvKD22TFhcrAPlvT3Y2IDB4Bw0ig/zYM2ThxNpKV8ODBNGo/p8JnuAeD7f95zXmArC6O3eYLNpqCbcndTDuDi+nJhu3/24aaD9LKPIlodXEoIAq4oAMMshTclxTjcbqwrimOIERtUZ74VTn/mF3TWvFbYBCAz9utyVHc5hsjf+vblKYrm7J6YzmeVKSmAcLQvPy7+3AlFgKqFCYBroBln8vZ2rgCzESQiTAEYm2CbYZZ5G052dH35I/ryBO6I/XF2897C/eMcJBpc2MrJhKLNCSUmco2Uyy+z6jS9lywVrCMvtwVshWtOB3JaM7pXaVqj2sCKfmMH0prIFj68mNU1nbN1XohSvduLN5NXkx8rg9njJ7g3en4078DKBcA1/YYGdYoTAhrTy9jl0+PRoAVuTVWTwuchiERFJCRI0DXVWB1wnWk8IClGnr4bJLRsYlyf5vlJSVqiT9wYTwYHSkmVd7Hi23dRJBoyUrSnXwmZtPH1wITHKMMmIKWlp6NT7TlvBC57d8OepCiO6Vn3+qqFGT87qH/s+BJVEqVDXlGHUsfN1nvmHeWb+lusyHtRSmQa2bmGDHicl2FGD07LKjNV3kG0x3983oLop6XEQUL9PyODJD5ik9P13sLJygxfxfU3mjQ148gSfPbuO37eqMI4xTt4td68kJEm9EJ9jGfxHpmTjftwefI44xMOTZWGeU8vHduhw49A+ud/YKJ4+TcuSxkvmo0eO77Ig4OeoU11guoXPMox1MF3qDWHZJu+d14Q4+RP+Swn5PXo0hrse9aPp1pPf/tNPf/q/t9afer3RN//pf/3y27/vjVYuc2TaOjFdZ//wDz2No+tprsM7zvAS4FH/IfymPTjuNXEknzzJJ5gvPZLuiFvQWEAdLyhtgbOM90s9fOr/NN14zp6IeDZd+uJvBktr74atIAUWEgWvU26tIlFSoaB7LHjeAfchX+LPJhpUShYVlmTraGuMI7+d7sFtN/XOnmKVGK1qGCEAAIAASURBVFhqoc8HvbO4vkvCrNJLJS1WuRw03O/KtgxYHmIYqd1dgYKuvM/fMmBpAJVCUz81q3/8IwZMclZUYNvou9Dk/593adJhhrnVym5FszUN11b1trh6Z6f8wx+i6azyfX1lxXj00L6gTqUrSo9Ns04YigKfvwDEmy/QdaDJ3OpsX8PvqxSUJbzvs9X+Xtfrg09H88AHDKM+OA6GAcMBFCWEEe5Ojvb7vWi0qtFKkmXV53OqXtnWDTuMyHURETY36wE81ei1/HPVrHmei5r2uddiIilsO9uVog88VsgyLMv6i2v61Vz3DidP/xqvxSiW07mcR0pU9P+z96bfcSNJnqCZH7gRd/DQRaWUyrO6qydre3pqj3lvv+6n+Y/3y8ybmX1VNd1Z1ZmpVKUkSuIdwYgAAjcctg9AiKR4iaRIipTCHj9ERroQgMHc3cx/ZvbTNOx1RN1Ow7jUkuMcsilOpjjWwWTAFR1zVJdAtINrAGBTs5n3syzzJ6PR1ptpMOB9w3643Px6xX2wLGiGJXIQFjgK8hTjCKbHItIfph/wpoVfNeF0HX73juE682Lj6xMNjJMaUB+1roA8DoULLQea4uQu6wKkA01H74i+AyOt2MgKUnScP0pAClQBigFDYEX5Od8beTCzgDOYc8ZeQOIURlMc+dC00dTAkLdgk6yxvjjFKAFVkGWAa5Lk9HmGx3kOflijx8oQZBniYrsGEWYFKwMnLDQkhjNXTTBwDEgiCsPCY5TlH9lCBAfbKG9J8HOj+qfMgo3dciJ0GlgA+dHtP/A6gDCbHA/2yt473fB9ZNeyYIr5OjuXa1whBDk2OU4ZJJ+02BkGLC7CYIirqzAafRS+31nX6CSGxUXqds9XK1uzYec5/OM/UqcDkwmcl9y7wp8xjrHTRqCLtOa6gTthMIUgPA1Fr3ubD4flSLcx53m+4VJzLY4mKstIavz3/+g2Xb5HxvCRQ/csG41GnjdlXGut3HWefOE8WAoXol3c3EOedTC7sBzhNIDJLmwei0h/kMkrmgY0npQ+GSLN641voDguf/K1oSFtOGsZ8T7ebWKvrl0/RQzLWbj/2BFNHWlBrNhO+/z2uZ9ZYOhszhl7ARlN8S/P0AtxoY3LneN5iW/cNlhhfZOITTLkjNwGtZtwK+p753IzxQ/h19fghbDchW4Tn77+LHaZGlVOU9pLrv4Uw2NNo3YHXPdyeiPP5WOEu+DYoBQkMQZBGQ0elQoLxSjEOAYiqJszndLbnnOybZx4MB5XzbSTj/BcNZ+2NwGlEIAs6xy14nts2IaBhl6a9HnD4xp/nkxgYYEaDYyiMjhP01vZib1GwsMQohiiEHwPXed4OwlDWH0F6+uoclpa+jjvfS5nf7EVh2Ecl/txw+V37sg6J+rmiG5a3eUV1+zhty4tyCn6jN7Y1DRhFh736e4YdjwcKsj3vv8wY9+vxJ6GBRDYVpXmN683vpGiCda1jd1EfzbgIUiy25rRZlw7ncuGS2k32xY6bdXs0rJpN04ayYBrZOhgcuAEBRVJAQmiPLA0klKFVgVIcwz5fJtkhZslGXyxfCIv8bU5QWfkGS4I0hyTHBLFdUmaTldd3zuXT1vSHAYeZXk5BTijwUgxgDBGP0JTI/GJ9j89iYH50wqPbQda7dLjn/dGvp0yQ3oRcTwpQ99+nxpHfIW6FncwgPEIpFaGx63WrXjRGCewtVWGo8c+19VJjT+nKX3/HegGvHlT6nY6LdV463o+129/PIEse48+pwE8fw7r69Cdg8a3QARHx0ai0qF3bBQ3ybOXUrbbbccy8oWFgufgyiFtPYU/h+AvwUOAfhkek7UIKwS0Di98GO99/2HGvl+JLTh++53z8IFWt2Ka1xvfxBU+5TjW1Vbbe72CoEcri/lig5riLEzImmn37j7u0nLVBf2EOUKaSx2TbIFrRFleeIpszhtSau0m1yTWyRejiUozmmPIt9VZ/wCe4bnM5cOFc7CNwpZq2c2CGHcnUgh2pwuuOT98ub3hsWnQwgIUBW5sXlpvZCmg8rwxSeAkPPP2iqaR64Lrgv/x6nIP3U+3i1EMa+vlGzwWJlUK4hjCCKIYGAdNu3Fduw/d7R7/MwCOJ2SYZ4V/64rlICgDPE0D0yjD2gvIHv7sNqDdhjAsY+MwBM8H25n1Ya3vM4mhKICzt4zf/CbqM45n+mQMw4iOtn+r9VY/8ul16XP5uGcdVb1xEKpgmueKpGTdNq8TROXV7yQ5pAlFSKwLd965qyQO/dE4eBWmW1I3il7GLc45NwwD3rYUjihMMEogyiF7u/NJQU0drBjDugD1Q+6tZjaeeCpKaDTKPE81GtyxZvqZyw0NjwE5CFbIIhHlx+Ic0akQmtVoW9Q++cqcA9fJ1KqC+bxc5uIUYh1s8ba+Ls2oTrGey82Rgx2VdQ664JqAU5I/DvIM5wrjDNIcBKdPO19El9BrgkAIUzYKqNUCe246H2EvRl1yKcjSydaKrlOQgkkgmcBug1xzrqfbGx5LCc3mLBpR+eX0RjZNuncXtyX+9hyi6LpxvysWcl36+mvQNVzfgMHw49dn1vy646q8VqlyY7n1q86BvtO6Xj5aEJzxuWYVy3FMT55Qr1tGtru7HzxBBTkO5hnECU6n+0fTM1R2DFkKUivfwk3G5Ktu5JTnGAQn6m13l754CHeWYTKZr8s3U+p649XV+PlvgW3hH//YvL9o7LUXuupfD2m6U6z5NFmCh23YJwMP/dHrn//84sWfB1s/NnrL6v+O4OF1a6ZmNt7ezjpdrdkUd+8ZC31pO/PY+GaHxxqIDukisrUtAKYv3RONJnIJ8KEL6VH+bcUowjyiXBCJ2eaJ7Save/Nczwyay5lWuQMdlV2Dmqa0DSY4nTG0DmIWxGQbhSY+ZezONemb+/BmB396KfwEFnrQnpvOte/Fgwk2TdZwmaFRnc81TWAyBT7PYrj14THnYBiAWEZWSl1Ob2Qpy7/pFJKk/LsiGzmKE16P6BUXcVFA3QF75cElXPNQbbDrQqMBUszejmWWT4dYqrSuDz/0BjknTavfIF3KG6x/92NhoQf4n8FxgIgsq9S2rr2frTqOYGOj/PD1VzPiLsZAk5CdDxF9h/mZIWiShMA8eKcDdo3KxjGoAnQGuvHhmDxWXcopzeDk9tIzy6+5ys6cuUClHjSUstTqUakrveMYvvyy/M+//32+Lt80mVXVJoUq9qYpmjqra2uvaWpCMqIdRSohr8AEUeZJGvqjrZe/vnr649r6L14xNHvdAoqPoxlFnINpYLslFhZltyNNY54HcbOFFyRzLHItKLIIJpNslGdOszjvfq6UyrKszupXPJviOIBJE3oaGDqYdcPqAihjRUYFFbMO1gd7tJ64ckJBlFXd3U4jZJ7LZa4zGXk+TYOCI5k6GBqdPdCtqoshzfHWpbUmGfgRTgKwtEKXIN/nfOkSdEl+iGHKwhQyRR+XUggRy+kk6Hgf45OTJIOBh14AlqPaFkhOtbPJBWYKTq+Bn8ttCI9vrRyDE16nzLiI4XLQ2oO1wVzQ0iItLZVhcM3Wm+c1goqbW5Arunf3yp+uRqTD8ONgoQf5n9ttevIEkgRevMCNjfezVR99L0KA7QDg+TicPxbzc/nsAWj6KQePM8sHuPzMhfK9N2cf5nLzXMbRRGU5LfZ5v2N99ViTAvv9a31TBRQpJikFaTFW4HHeqHHjV09/fPXs54G1m/7Rwq870NeuXzM1szFDMAxuGEzXma7hHA+84VJAkUGSZ8hGvWhd/jqyi5Zo/YDN5fPaQDaqakPa7XYoxs/wX1OIl+lhB5Ycavk4uvAdEmVKeVUs3UDU56/sGqTmJaYMUAquM/w8DiX8CH95jWOv6Dl5r0G2zuFWHccgZ1zXuE6fyfvKiQVKxgW1mDJFwXGeqfSphMeU53kwjUe70+EWVxmlPiPURgNh6tx2mbbv3+RZmoR+nh1uY4tMMKmLONaV4gdgxnq82t1Cf0gExWADJQpNE5ouNYNxceqVERGF1HXT5nL/HlSepXGosrQoFFFRjtvcwpfPys1roSUMqbN3bPPo+JmuJ0MzSw9euaiG5lmq8oyK/ViXS6mbBpdadWY8W7Mhy/JgGo128iTG8U753e4mOLJieBO6NIQ8wNBbYcLlM2ZxXuSH9SZ1KSXfGxnH+dRPJrtKE5SFlE4hnSIyxrhQmS44r9FjTYNsUZEqr5mlNN6GQbVhj7cxnCAy9Hb5tKMZFhf7fnOWZ+F0pLwdEY6RUhptU6WAUs+WK07Qc5H4pEJIC7b1RnDQm21hmO/VG462cTriWa4X+TtvpDrdz7OkyBKqtEHjCQW7LJroxKRtskLN9FwhqHkUJmmUMUV6H9IM114yxtnDu7zlnmZFu5s4GdTvBRsGF1KoVGs1GecQhMBH0Godtbpaz1xqpd4Uoe9XNcZ2ziCOg2yUqtEOjEfc8zG3aNcVgkq9HXTy8iyZTnKmDtnbzOpO1fNsvOdjOhVJYHh5+a7fHoIc1FuhcioK3NpiL34VXOgLPe7Ye/ZWoxwqS5MoVtl+GSeFUTEaMN830kSSEkC1nmfjoyiZTNTGRhGNgXNMfA4oVMILJrwJc5x5N/sbIklKw90sy6DXlc0G73WFFHgk2FA5pnVHor2c0svZL8rlTylQBESkiJIkmWShN3j9Zu3ZXwfrq8SksbhQPMnlF01G12QwdfXXxFODYSYl3L+rNxtzH+VWBUIVOzFj0NAsJUyeWkWsgzr3S0yjYHdjNUvifNoL2/5W86XUDQdaZyRePm3eQTSq2LzbIE2Yh8ewxx6cK+QcdQBDgiYutdCXiHJFRbkxl7+BxcfFRa9phc9gMCn9lJVu0bZJuz1dj/MCpjHEGbYd1nBJSvoc3lcBmBYso4KxXLLi0k9x6llWYTT02bIufJzwWCVx5E3G0/F64Y04JdNNvYDOc9ZMQuuLJ0zbR6WS0N969WtQRR0Hheu25i44QdLJUs7Nw+Of/4prfy8KlctUjDecdtft9BudJd10TrlyGahw4TS7vbuPrQPhRBqHo43XwWSQZbFSles/3MX1vwMVTFd2FvS/+NbS9NPGV+IMvcXQt5v7T5elsbe7GUyG0dTL03jve6vhdu8u2c0u542ZZeY5TKdxHm8/h2kSwuB1+eXzv0G4DQC27iw2FkTzAENvhQknk+GWtx0k00N6s5vddrt98FghztNtfxDkET1HGLWq42oppeEo7MTJwYzqWKXb0Wg6GcKLZv3rsLkNg9ecC7n2m23r7eX7ptPcGx/G09cbvwWv/q4PR8Iw6TeAUbu852Zv8cFX4oA2DuotnYyS6SbGsfbjsDl8uPAP/3wwPD5Jb7A7xq0XtjQW8gcHG0XUp/vBZJj62yoJSgsMw2SwrQdxX8mmFEaWineCgWhruD6dEKlqxxi8lppubL+y285pVrS1DcPqvbz4m8g902nYVqOx0DW4gB9/xJ0d6vcTVIesrtaz3ey1l+9bcY5Pn4LnkWXFGhsM1yaRFw13yJsYYcINk5Rnxw9LvQnjHb1tvpru0CF7q+V0Pc/GZzlEkRMGi4PAzhU8+fKo3rIyqs9xOBRvXrhmY2FnxXYbe/ZWoxyBPxyubYaevz/T4zgZ7+qZ6kmnqZhNhZh5INX44Zvhv/27v7GRZglZlr6KOnAjGDoom29e68W8m/2NcZ6SYnsr8/xi7BULfXn/jtzj6N93UzCtgTIX2hqZl+oKqJQSBZkgqZHOiEXeaOPZy43f/r65+rRAce+7PxRPtPX23y20OfDr8Y7q6q/tnWwyyhouW+hJgHl4fPvEsHDpAfZc4XTsRWFdoGI8CSY7L36c7GyMmm11j0Xf+1JfvpR789F7Kn4CgO+pY0Jr/rL22IPjDHVJQoeGXdgGnbE2eC6fnsQpbO5SruDxMt3pkqnNLeHSZlnpQH7qVfQ3KDzmCDorWBoE22+SIom8hpCQBFObycyfYJ7LVleaNkhZAGVpPN5Ze/XL/9rdXD28pTmdxtLjPjPcJDaqwKlGFGfjn/07rG8WRaHCgT5Y7CzfW3zwxLAaBwObaDpZe/bXnbXf9r5hnHOp9e4+slt9q7GfLx0H042Xvw7WnieRl+cV7ucHsLmJBbFo2FOJ3b9jtXunja+kH0FjXNhmY6+CNI3D3Y3VnbXn3mAzDveD2NbCIvLfMS5M22SAVaCSJt5oMNp8Ga2NsgA235Tj9AQ2GgDQaS7Ile8Yl5pj86q1cZ6lyWS4s/Z89fUvI29wWG/3HjmWYVTVqgogA9hNo9XR+u5oC+IB2BYASM0w7WbfbNla2+T7wViUp2+Cwc7mc9ASWK/ank082HwjpW4+t3umtJrtg+FxFE7XXj/b+e2vehBxqUEyrK/fv/u40Vk8eFiQhNPt188Ga8+jwIsDLwmmkCR6Viz5E3f5gd3s7GGVWRJ7g42dN89HW2+iwNs3iyCE7Z1Oa1Gk3/Gqv0rt6WRJVI/3Nn+Lp7vlU6dpEkzdjBAswYWYjEWnC1IqpBTVIPZebr0YxRMarkKSwOYbw7Ibb/r9hr1vRRU+Hw221n75153Nl5Ue/Nl7kYkxXm/0lvp3HxmPu7phoO+DKsDzo8TbH1/PQKHrZqN395HZaFsKYTiEIIBmM6LkzfOfN7dfRf6YwlBPcyE1Gq72I7/UW+ud8Hg43dyJxoG/mybRoZlyVM9pFO5uvNxZe35ofDcq9N1MABNpwo7oLZwMVJaDP+WDQafVp60H/XbLtExRW1GeRcFkd/PV6s9/G29v7YfHlZ5tzVSLD5ne1EjVK87e+Fc//2Ww9iYxNTBNffjaQtGI8r7ZMnVLd5qQLc43qo8lNTqapIXKaTLJq8zh0/h7U4h3cROokMQEsUuplqwR6QTCHBIEtMG1sclRlvYznaRxxA3HbS3c+/L7/CHznV2JGqNjfpSDsMBRkKcYRzDVyOAfsPdFcTEaZaOxGnmUZSQlVpqZm8wtEw7CJKfNW5pbFMzSyWyC1LQLvEhCUmk43p7sZDkmy9Jy2oVQtfkLEg45iEqxNIZQwTlKolJMhqzcvtMigbnbP0MKyPOLMARdMMsg6zy1wXP5FAM5mJZeDJk6fQ5URjWum+VIhHUHEEMnzkBdas+NNIOBV/5Qy0bHhE+YRfkGhceCUUMWRjEebD8NRlvilWAISuUtw2W9+zrKxuJ9sFxotTLKvd3NjRc/P/vLf9188cuh6zS6S/eejMzWnTyTNXtHjSjOxj/7G0QxAdCGZq33FgaPqKDO0kO3vd/vNJiMXvztT6s//WnvG5RCWPpKOL775B+7sLI/0p+8fPrXlz/9KZnuqjSazciqkRWu6yuk7v3uP/YO3Nsx4ytZ4e6KvNNrLYLK3wbS/ubLX1d/+tPOm5eht1+htPDgsa7bmuzK5a5kWnVCFm+NNlZf/vx0dTrIQgjDctzuKlSEKgt3H+mGzbv9turXqE2SxVve9urrX3751/822Hh1SG8Gh3xhYaY3gBHRWhz8urW6tfozvNbrEFS3nEZ/IV/8Yunu71rufjAWZPGLydbq659hOPt1yHIIQ82wXBalhlh8+HVr4e6BwwJ//cXTl7/8L54rZKy8fjXPVr4bPfjmD70DtczRdLL+299e/vxnfzRIgqlSOaiCE0Uqe/DNf+h1l/ewyiyJxltr68/+9vrZ37zhfjBWJYoniyvfGHEgiNpvw+M8DvyN51tP//Tm2Y/ecHNWX6TyLuqm1rFQ2FvfQLsLrVYCagDJajT8Ze3pzuZr0GS56oSh1Wj1n2nKsvetqMLng9UXL/7Xf199/jd4i8GWHwYvrVf9/r2HKsvL8bWXlKUwGQfR7v742kXTTN3pPPwuWPzim461jw9MJ7u//fg/Xr74d5VlpBQvilJ7UluJo1Jvrf24MYmCwc7L1a3nw/VXoT8+NFOO6jmNg8HGy9Vf/nJo/BKaDrY1p+lmqX5Eb5PBJlEBWY5p2o3uZet3s173Tr/vQrMamXnbk41nL579+b9vv9o/cqr13O4uCcM2OnqjmGV6zMa/Wvt19det1WeKM+Ccc2Ez2QNNLT3qtZfc+c7/UaVGR4fDPAwyQ8PlZa3ZFFygrrFjWXxjCDdplShxlDCJX0q1ZI1IT3GUQ6qB3mELLWhpyhCadFp9AqvNpN3qL688nrbHUp5YdayD2YXlCKcBTHZhswNLJjkXvqvRKPvzX3x/qpbvmMuL0rU112GuO4eOb5loZFTlwXai3FixiAy4UBqhYbtLD79SefLm179OtnazbctpLqhGVmdDm2QsF4tjYAMxIsQ2zI/8PsxxT4vRbu57aAitYTExj43n8nkdB2DVHR1zRZrEVpO3myQlquQyJ0KYwJsd8ENqu7jQgs+TRfnjJFczxqQhTcswNFOK8h4awrRRyizHOKr68b49eWXc1u2m1Tx0BddqOLZjmAZ/t8vubLzRgJwRUGEbpmObtqEZkr17vM+PXBkF54ZmaZZg7zg6jDPNMmzX0SkvhDY7WsmrjdRyLNPhXLxnfCUOmBwOXVnopmM7rcRqynz/8MfRXU3YDHV4W1FQdzMWpmUjSzgv8vKpmeGyqnuwYzWE5YBuwN4zVv2fheXYZiM78Ix7emPv9oVmyHSp24bNDKvmMNVsu3wE2+DvjhSMO9Jo6g6YThk91tpQTJqm5dpaxYFwcLwkaGbQRR3ab8dXYus2P6RnwTW71BsmaVKwIk3rOmFDM9i7nZ9nerObDauJUVzlDqgiS4kUmFJ3msx2Dva+ZpwbpuHYTsNqYLR/YNEiaZEhCDFOZla3N14zbdNhmo6cFc2maViWAiOMuXrn7L+M6HS7qTnlP0cGDQuEACFMq2k7Ld10qkLl4vD4A2+E6YZwndI+83TGD2waYJk8sy3DKS1ZVJxneQ6MgWEcqzfd1m3XSe2GPNKv8ITxxtHxTiGFOmSfB/RmR7N3TVNHmsK0Sg3vXbk0UF0TtqO7ycHZmmYwnTZBtxodsbCEuQC7Yu2OA+bHWsYcbiRms7xUZTYGcRuEoRmH7nku1yN1H+Ysh1zRNCiynFRBcVxuw+22WDi1F1cO2RQmMYwdNOiSqiULUClGCUaKFAdhojDB5sClbjT7y2YD0XANp9lqt3MjPsVmNNK6Rc9DFuPUw6FLnfMmf9eIcdV+SRYK4rhQihyb9bqi3eTGvEn1LZQaPdbLXUDErBC6gchOyQJgwDUy9WofLy0TohQjQZpmWJ3llSjwdl79portOAniOFB2/tbH0hrUTcs9Zj2D4owM2xmkEfoRTi1wZNUneP6+9taoKFJpgsIkXQLHTz4cOh8P81w+bSkKipMiilEVKAUYBjOM2oujS/QBRh6+3gIvxFyhqWHe/BwPoa47PM4L9DKWGZ3lJ//U1MSd7l3XcssgKkrskWdajqHN/BapGY3OkvzqP3S4nYyGh64jG03jzn0rU+bT346OTx/8HX57XkCefrmE9xaNXs/u3bOb75TudHvL//K//z/RV3/YD48ZImdmb7Hdeud8t9Xv//4//19fffcVjhKM86oF1Ah+e17+v8ePjCffdHvLp4+vxZpMu6ubYNvwNpx22/0nP/znew++Tjc31HS/YlNvdZwvvjR7i5phQZJW9VHO0r3Hze7iSr8TRtP411/KL7/6RnR7h8fXV7DcxQdfNazmg+6DdDI6pDezv2i5s+xxCdBGFE7LefgPyeIXxp37wm2Uz2hJ7Bomk+4Y4UB5b0d3/9PiNyFvw+NHVLXsrrWBpqB//s786ptG9yCUfsz4Wsze4iG9Nbq9b//4fz788gntxvnIT0aDPA5LLS0/aN1/dJDX13Lb97/9Q6937+tHv8+8SYVGRsl4mEchEFl37vcfPHbbbXny+JnTHMbOzq5lupo1g5J04H3QDXe5/e0f069zvdNjppZiKOLUGWeOvWTv5ZlXHba7j7/5F/gv8eN/ws2tMrh1HLJtcBzW6WhLy2Zv0W33IdqqDRSarW7jzr/Af4l2d/buoQzP27rpNFtCzvqf2Tb0et1eu7zyN/8RplMMghkV89KisfLokN6cVuvRo85S8btiJyiC9NBMOapnu9n64vf/tHR3+dB4Y+w3Xq6Zze5eH6+jeqvftWy0zO//k/n4G9OZkYprhtVevm9IbdHuJON9tud3xq98oRMjz8f1DT1LOk3X7K0sffWf0vv/SEuLVDXr5lmuB5HFpNvozDfC65e6D7M3LaYBcQ7dNu+2+KDBpARdP1ME6DPvKf40ofBKqyWlYbWXukAacMmFlO8rUC/DY9VmmG7xnRB9dbYQ5aDUiDEA/OEH13b448eWKuDeXa3d5PMO1bdaECXnDVMnwQUCO+VtCtJcaKcUC1zLMZ3iSCPDhXbte8RLXnfpgZ8GU22SQF689Vbr6wOME0yrYOdMydUR+m/wmY+7PbzToK4FzvxNfZ7yITzMc/kEw2NFWZJnCRZKAOAV+QCbO/B6k0UZ73zG/SavOzxWBEnBCunYDXex2314/8tO5QfjeIwvX0KWU4E182qNEOp90ZI2xPHhCxkGtFq4u4sv1+sv3hnfXMLGnSL2065R9Dp4/x5v9Tm33gkbnOa9J/8A958cc2W78c4XtrP88Aks3ed+waoEBtzawkwvY5vvf08rK+A0Tx8/k40N9HPQ5B4LrmZY3eUVaC9C/8E7z1g93YzJtgqPhdBko+O0+70vv8yiIPFK907/9n8Td+8dHl+/V6mJZtfW7Z7dOfHKlfDqT2im1bkDnOtff1eH3IWOymUQhjLZgjjY15vQLacHwqFv/zAjOhqNsHGHiiRvdUnYHN8xqmPG793Ju3rTLWfh/pfQv8f9ohj5yfZG7nuQ5aLb01vdg/csdaOpLzedNrSX6qfLp16y/ib3J0Ak+gt6qyNOHT9zXHZHKP8OSpGY9XMQBcm00DXXfvw7cB293UNTyzDE3ZH+bI1Jh9jbXK6K89nsLd4TOjYX0XkJaQa9HrVa5DjQcA/pGTgDw6jHH7yHmZ6Lgg8TSqc17E+mZdpWOXJxBUcjHI1hMgHToIcPaWmp1NsBELvCLhY7lsWX3rW3E/SsmTOrOzx+YwOnqvx1Jk7U29oajvLyJqVDYj+7gQtpOk1T6B2j8Y6G65liGLT4gJYqW81ew+Ymn0zMlRXTsruPfge6Rouz8BjiuHzeIIQ8n+//17oyV2fGQVgEYVFBo2DorOlyTUKV3wO6dqadOIHEY2NBZqy8gpJLqUAmVag0RSBHtg10POaBJttG28Szht+CuFUYEWilj4nJBbiR05SGw6z+0Gmz5eVy/Z/jxp9CeAwMUWeitPMoLra20zQlIdA0mOvyg6XINXqsocmAK1AJRilEBTUYN3XTsRsdt7Ngh1tCRkevjyjzMjBWdDbbSyHehc0Eonv0ZY/u6mDO39SHHYKAFJApyHJIsr32Lzfj3hiKigig2k4P73t5Dn6IYYycwXl5mM+vJayKnAivkj24rp4NU/RCSHJsWFXHafHJHjKGCQw92BlD066oF4oPelJVQFwRTUsBV5E9UV4/KbwpDD1Mq/kyD4+vVYjLQrcLtwedPrQrmEgIGE/A8zCOYTrdZ16tWXDVkQPX0kE/DjGoxxsG9fu4sa49/YmCAfQeQbeBKI8ZeYYrz05/DbuM++pk1DyHduWZdbsHUc0Tx89mSQTHVscdvZPTnq7JDd0wy1iCtTvQ779fG2e4Mkdmcg0MA9vt+prIkQvAWCDuHndlDZqt2a+3WtTvw9am+O1X2g7wD104kgz/zvgz6JkLUwfQpIHTKcrDydVHn45LaYwnlOQAhNJkx6ZZHtVGUZRfHvwmzyGYcgDj/kNa6HNNB84kKFQGyt0Tr1mzQxcFPVyhdqc0ZinO+EZqPUOc4CF6zHpkXVlF1SGhZUGvN7O3g1dAKViDjOZhezu7PZ/dPicTSFMIAlhdRcM43Fn6qIbrmcL4HoczJTFsbeP2dvlEKyu0sgK9Lu1lwkcR1bv0NJi7dNcp9ZlxEBZZRqaJCz1uW9w0kWEZBFbv9hz7MFGulKfIu5QK5CLLk4nHSd5pfAEGeyH+TcDWNffyFQJdV9YfpMQL6GQuN1/qHIHhMHNdubysff3E7HbPevzBuNBtx8SGyXd1EOzDgB0FKoKgIGVRw4W2IG3+dj5o/jKw9FkbJy+gpgU3J129jHu12YcTA9dr4fVlnEldSp3YVaLTdfXs0MO1AQHgo2V5v0/H9rP4NGTowX/7d4xT+O4BPVqGZ2t73YcutDIQRrnMCGyTVcV8V/KmFEGUsVQxVczD42s/LiOhk2aRaUGFGpFSsLCAjIHvYxCAyg9idOdZaXiNv5VuepqxKIcop4zDURftzFeuT39LVYmqIVNdvNpslSFQo/EOQnh0/EExjOPXv9PvpCBIq9NOywLXAcdBQFGVNJNh1tp7jzbee5SXZqwgtF1wHXLcGsfDOm+Dp6AZoGmQ54dXdMPY//VK2zgJcRJSnB6/Axwc/149AxOqwAIwTCDNKYopjvdZnY88HcaJEBry0n+tAN6z6flAffJ+wJxmpYabLerOgnkOgGaEhglZDtMpeN4+H2/Fk1wqRwgSgpqt2bnJmd/ITM9Ih1n465GqgnMNvbzVClU+am9QoxPCOuNsvqB9VpaP1ZNixYNNnne4s/RRDVsW1IxcewcctcbiGOKkfPJOmxYWDt0GaRqwaO7SXafUzMZjPytYajWUYRO4GpDLQePnOaDWQXcKu1E4Uxjt4mazXD4u4ofW7NxBOgryncDfCrY3Dd7k96Xq4K4+4pzv9fKtu1vnmEnQWGnBl5YMViPqM/8AcWFBqzoDMM6R83lg/ImLUhAlFMeFlGd73TqHvq7HzV7zXstY1vk7eC8HYYKjME/hTL3T6W2VsgT9cmnSPk8xdFjulqvDJCDJod+Epn2SQ4Qp8ZxAY4VkiHjlaUyGhKU2JjlMIxp46FY11Ue2bUTBkdOl8zAfrG02BFgGN3Rk7ArZnkuXMy89Oy8EXYJdoceXH+ozMDTQJE4jnhIV9NFW7DCG1xUFavNbcix8vQPBB4THmcJRyMIU2w71mseZyqWEaFU7GSggqbItFM3D448opkn37oLgOJlAMD0cjN0ciSJ8swaeRw0XGg06Jla59KO2HKelQqjbhXar/MXLBdaOXv8dAxHkOJAm143m1ThkHFdY5RQGAxBin9X5I0wUAbYDnoebW5CrfdS0sgfc3YUoAveT7rVcP+lgAA0XTAMubPlcgG1Ds1n+2c4sc3cuHz08rpiNN3b9VB8b5E0WgmV078ETSd1zXcctGivqoSQ5YIMA8yfQNaB5gfup2blHg9fj8fpwuOptvkid3kBtAhpZ9x2u+7q7dVw1MdLIkHBpOFuNqMdJUR1BwRdfGKaO8w7Vn7C02/IPP7h1cjUgKlUaQLvJzxIeFzZkK6gVrYfyXkcu2/ydc1INjC4uReVeO+YgPrB3+lzOK65J39wHXdD/9xT9EB6dzEudEwuUjAtqMWWKgiO/igrPd+7Ngq/uw8DDtSHzo+o+5fWFI59qbbMU0LAxjNnQ1wJV5PSJHGheD89zqT0LMIEoAS+kPJ+Hxx/TlmX5N50CvQVLb6ZkOXgehCG1mlWTrfO4SpqEdrsMR+OYDiKQ7/nFFEYjiGO6fw+6XdAuO8nq9OuXs8SFNAF/inEMUUhxDKqAq2b5rHFITavO8GMIo4Odpa9fSNOg20XEMgw+iJrW9uB5H/reKwwficCxwXFnjFmn+WLnHH8Jq3IMGxuwO4JGo4xpdf29sUWpqzAsR0qxHwYbOiwukGnC3TvQ7ZI2Txr86EsaRRFNg4JzNG0iPc2M8Q7fYOg0y+DWEqSdHZLVSO+qHgC84q9iRhml5wUhVJYmUbC79bqiE1svsjQPQ0iIWQzhGCgng9TDYQLTRtGwqaldRtJkmha+r/xp4QdFQWCa6Np8Xmn8yYtpMHN5Zj9BWGwPsjgpVHEm4ycJSgOGRhMXW7SoFfpBy9fA6NDSGHYmOEghtqlpzrttzbZ6NE0eZJgUeBacqkZ3C8SGCa5FZzxf1WUZcw4nEMYQVoveMStPlS0SxhjnvGBkm4VrkiwDxauNrDQJ3Ub57BvD0vlNlq5V/3Vtc4UeK4OTIbl2S8qAkwz8CCcBWFqhS5DvTlPBwTXJN5GQxTmGsYpjOmsmyA2Wq+Z5zguYJqwAvNtnXoQFlT+Xq3l4PJcrFcehx4/Q82E8RoDDdZunhCVbW1AU8OTJ0Trnywl7Trn+URQ3isqImhufmd9UZTdYJr5Zu5L3XmP4SsHSErVaZQj9vj3tfOMvy048j1ZWwLbxaLe8Q3JSnkWlDchyWFqE67nzubznRdHr9SxN6f59/YHpeCIY6GzTmuxiMIItA2wX2teZ3plEwWDtt1fP/u3Xv/7XNA7v3P/aWurnD4TbXe49WM7bdIjfOINkQoOEpr283YK+yYwP92Z9Xz19Fo3GhdR5qyUWenzeoXouHxQCVRzLKcQv4ScOwyV4CNCfq6XUjMbaHT1BmAzZWXCqGt0tBC50cPlS0bM6W2Q8wTRjBxll5+/oZoof4S+vcewVPSfvNcjW+cEKNcHJNgrbQMExSWk8UZagM2aCfM4SZ2zLY0mBXz9gUYovNymsGvHPw+OPvlLq0O2WkZjv42BIrgv6jQGXlIIsgyQuI8m6kvZo5erpYhiwvEyc4+s3ZZDZ60KjcabDoqBKbBZiltFqlteBNMUggA/X0tHrH5SjKG5Socdn8nNT9P3y4jUobd7siLooIHnLfnz0tdbZDWkKRG9RdAukRCowy4AI3DIIPB7FPcN7R5Vj9RbIss5iFecdf7YjgFPtqrITihMydHDs95c/1Gjz0ayEShtQ16sbxnw3ujFOKva6UmtIg8kE6RX5CRU+jlowtejcVQMchYkOoYjK+T/RyRJwvnM9zpgpdM3mzmJX3mkqW7MbPbvTSIzkEL+xAhVDqCg1C8MBR6A4JjyueLk5GgK1AiGGoK7/PHGxL0rLzTKyHLQtZltz3Pizk6oNBShFYVRwBufCndI4HA5XDXLcdr9mW6w5liXoEQY1SfhcwzMfVGDDZV6CyRapEMMU0ryMbY5S+x5EdxkD27xk9Kzu2RslqBQxHQ8wyt4+0SX0miAQwpSNAmq1wP7kLKeu6Y0T+P4eLHdm7c32hCFogqQAxIorOynihM6YCfI5S5bjKMBM4VIHsxzWdiDLSw93Hh5/ZCk98q+/huEA1zdgMISvvya9e2OsJoPxuPzLUpBa6dyfF8utkVjPh9EIGIMnX17wTtpt+sMPOBji9jZ43s3S0kGP1Pfx6dPSzXzyhHpdOEB6fCNXhfr9TsoPJ516VH2ty8Bvrxa6/qYoaGmRlpaOx0LP8t7r69QfznSocc7x129XJ2Ul1NqoP8zlBohp4v07sv5Qp1DllIc0rXrnhhlchAZJA6MLSyGmY9giUIuwIuisFci6affuPrZN925zOcVI3W+ErSzhKITN5AWdm7pbu8REw+0YozHsAEAHTsxi1HW2sCjbHeh1ZbPBP+GuqnM5cYlVNA0ojotykc35uXCn6Whn5y9PDXKe/PCfu8src2We5oNWKJ8hMckwVOAFLHDKb47SFx1Edw19PiVPk7rW+s0O/vRS+Aks9KD9yT1jmsPQg6JAy5rj/Je37hUQJlAQGLLQBHzOYPsNQ491jfQupgn4v1bmn3yY33epKKtSVbvdCjvVGejn705UI7FCzHpfXzidv8bflIKnT8v/XHlwQ40rjmBjo/zw9VfvMB6fXRgDTUKmAbt66CaKZqnjtl3Gb9rJFYzVIfMMZ64hhipn+EQU9yzvfe86Z6y6P+/467erk7ISztuLfi5XIDUOk+VlDMAZmibWtJM5kCrtWxEQQZFBkkF6kfCYtFbRZjj1cYLIOrR82rqaZbPjksowuNQsqVma2TWaMQajTpKZu5wmjCQQQ0AOnKggSmte5TOFx1W3doGmhmYKSYJhBFNF7xwt1fXGFe2zKghMg3U783rjz1eqFvs0GqvxKPU6XHtsntcS0jjY3VgtY5W3GPJZJIM0Qj+AiURNkMY/9Qq4GuUTFUtDkmGcQZrjsajwQXT3quuBb7vUtdaTAL2IQQRJTlfXifoj7mJRUm8daBh4W97LhVH9g/zJlg7G1WTWEtVcx1S3Z2Of8Tz7pFfeW4Kyfmj4cWOr5j/8Dut+0YB7rLlXKEGAvz0HIej77+DuPTq2DfV13s/cruZyZVLjMN60mAZk6Hj/jpQuViRJZUisUBlgKsgLLDJK6PzhsSTezC2izOcqxaQAdcqtwHgMdbL9wXOTOssAJbzLN86AS9KJElJBzat8dj8ZgcnKb8TjiN/qeuPV1XhzM2m1xB//2Gw3tTki8ZnLZJK9ehl023yhJ7qds/pLTrvf+mExXB9uvPhpuLF6Lgw5Qv8NPpvgwAS7gd1TSgDmMpf3BJAFROnsw1xugnwIqn+QP/n+ArQdmuvz8wuPz1KBXGMOaVqBUex4dLFGw4jgxYvS4/9ALLruFVzxvoJ2LXjm6Wc8NX9s/eEGCDKGdYlymtYcxR9+h3W/6PIVByHw0Swp94okTWF3BLZFbgN63fffDwxLAzupVvk8kQpEEUyngAhSHLarisQbNQ3r5IW3Nc+frF3VpOWGAdyvqsGrfI1DfNdzOfuiVXEClwv92+7TdZ9qf6pGExXFhVJ4ME0xh2wKEwXZAtxTFYYckpdRUJCDKI+NKo9/jSQEmAkpDkJhHsH0KNdrUVEbx8PBdPUF46L95FvjaJYB5odmhA5mH+7FMPZgV6DWPk9JMwIeIkYmoIJUlsMkVPGk8KdFFFM1odHU2Rw3/pxFVjWxDZdLgVlOnl94fmGaeLrFMUBZMFtz3KWFccxf/PW/B5ORPxnZ7UUpJRNcB7OalVkK0bE94TNIPRgGMHGxY5LD5/1T53Lx3XzW6oyKeSh1I6RG9f0Qw5SFKWTqHKj+Qf7k5c7l31vNgB0laGgoOAl+bogkTmE0Ld2JtkOXBW7XmW51jsA1Z3rfxJX3TBXINeYwnZaRqmFcB3tq3Ss4TsDQyZnztR41JQ62XQYz02n5ai4llK36RZcT4scfcWeH+v2rjZDPdT95RQ3B2Gm1ymeRPT5hxo7hAT6J//mT9Umr5y2Kqgg8LQ2JsY/Jd33LpeYEBoC97tN1n+rRRGUZmQZbXhTt5n5tbQrxkDYzyL6k3xPQK3g6oe1YLSgyOW8gnpUzaVbry9mhWt+DXK9ZGnu7mxvPf/n7n/+Hbjo/9PvL3d57r+xQ6wn80za8WhV/24Hd76nD4eKnRUSUU+4Hxav1tOUptyG+bwlTd5oN3u/Pa+M/a6lr8i0DWg2MYvIDeL2e3b8jTzc3XqCphEmCM2RcGKYTRbHnTfXRqN1uS6E1qJ1iHOPUh9E194Sfy1zmMpcTXdGKAXvXx6aNrgWGRhWD1DlkNMW/PCt9iR+ewHLnck5k6ky3MuS+9q7jNzLGqyuQ8wxevIAwhKkPjn0YQcpy8HwIQ9J1sG3iV/8ge/zArSY0roVp9naJpkGnDRMPfA+HBlnW5QRLUpIxRN+vnPfL7vYpRGlaSkES4zSYRbzvvZ+Gh0JAGOJwUD61lOVFLnxcUnd49jxot+koD/AN43++cqmfVzfKqLhqH/yJP+8VSwrxLm6WayqZGpg1elzjYIJjw63xsX2MVEEewRQIOrQEAK/gaUJBXoQFJJyKc5T7VZ2iNShcalCRhzhhyFzq1KFAjRtPBhvrL35e++2XnZ31Zm8pV2ea3TV/bIi+z6YA07RIzAuFx2laDCfZkLLIzpOUJr4yUup3Rbc9rzeeC9TosXSRcwkIw5EaT8qJcyxfbp2jkUJUgGLANTI0Mhhwoel2q1vgKJwMPdNyLEM3zT7cm8IogAlHsTcr53LbRTBydIICw5h5Idg6wC05XlMESQZ5gbrOTZPmvEe31XViYOjM1JFzJMIsP7ED/ElSodPoR9Bt0GKbDA3OGx7HKWzszj5cXtA+u6ZhXjfVyQ2O8eo6z4No5MHw+Pqx3KvmH77lQoYBi4uAiOMJEEG/T7frnodDSGKAM5Ak1TzG29s4DYBF1GrCwgJdeOLWdpXn9OQJ3bs75wGey2UuWhBu0ioAtGHROVDlJDg6dvknrtIZ0kjrqjbDdIvvhOirt2Q2NW68/uLnn//037zBjt1oLt1/aFjXyjzi++rNs2hYRP4Thcjq3K12c85vPJdzS52jMcVRDqkEXbAGhwailLrZWrxbEAXjdZ+yfGHBoYV78GRAa6vs55Cmh2blXG6vGLJYbChW4M5IpjlrWuDckp08z8ELKc5Zr6u126Bp8zTsWyn1/hXmqO1iriCIWRAf3wH+JAlieLFJuYJHy3SnCxWj+MffCoOEvdgpN+jr775+c8PjWZ3n1IckKcPjOgDew5DrCsk8J2GDJq+jvdrp/MC3Uc7OS1xXg0sJWXYiyso52TbqBsTVIcLuCIsCXLcq1b6pRwmaRt0uRjGsrePuiM6ITs86PBewMyj/8/59Osjre+JvSWi3Ic8xjsnzyjC4PmGp+YQvl8H4JtiV74Ftl2rR5gdJH8/1qWqJs5yGYZIFWTDNw4gEx26HOzY2HHZKEowGRgeWEpyELBrDqImucU6kS4B0qBnDNIM3OaisCAoVYg7RcHv9+V/f/PbTZLDFuVy8/2j54VeG5Z50/wlFNjQcaEu45GadiMCQaRpvurwN3HXmuPFc3t3WKkxGcBVFORSY58IErQFdBiyhaAoTFzoCoCrZixQprTQonZFeeiu60egtV91fUTcNxrkETVI3RD+mCCCasx+fS9IcRgELU2za0GqAfpM2Fl1ArwFRAqtDijJ6vHwL9FkQ5ArjDOMUCwLH5g2XhCiuusd1msHQoyDGjgsN66o6MH92KxVHztEwkLFypkRxESdknuewI8tpPK3Pesg1zx0e7dUt5wp2xmRoeCkVyJnCUYDVh+vuvn6D0eO6znM8hs1NHI2oPuaaVyFeomt4Zl7iWTX49jaursLW1ntRVowiePOmjPeWl6nhguPcUBXUla7jCaQpBcFZOYQP/CuoKZ3Okk1QY86eD+MxAtC9u59qAsLMrsZjWuhTt3tz3/5nI1FEb9az5GX4/LfANPCf/7l5/74mOEoBpyCldZXvENY3xLMJhU+ga0DznMGn5LwBME4wJQrSYqRiyf3CX/370z/9vzs7643m0tL9rx/9w39cuPfIdI5ZUmr0O8d0pfi2B3dNci9LJ67Ll56YI7CzhsEYPrir3Sc55zeey5HFHttN7nn5aBB7HL56rHXIPYIAHz8vpGY0Okum3eosPWRcWO4cKP4gqXGkJIPvHuK9Prkm3TQ7GQboRxgkZXxy8yVXWGGMmCsSHGyjsI0Znc+Vih/Cr68hSunJ3eJub96B+QoOPlSRJXmWUGHx60GAD9YtTyP66RXbmdAlViB/FBE3eV8CKSnLsSAMQkAkxsF24LzAbV1fmuUQhnAQtTu7xDGMRjgalQGkYZyGsp7FZaw7PFddnT6y4Zydl7iuBo8jCIIzoaxpCsMh5vks6v6wOPAKNVZVulJV2Vv3JMeqJzmd5V81G7CwAADlh7NkE9SYs5S4sQlKwUKfLtWurkk4e39n6TSB4bD8fnm5/LtAqoUU0GiAppWhdflb8xOxCztARahSL0lkMcrRzKGQ0mi6rN18v0rrKt8EIw99QH+FkvMe3dZsw4gyB1VQooqooKScPFzoptPsLd158N3dh98t3HvkdvrHXiGDdAJDIDDBbdCH0vIVGc8Syw/j6TSyWdFqi4at6VIwhKbLGzTHjedydLFHztF1WLvFC0V5DioWtuyE4v0IMKvsXDcvcj5Y83tz4Dgn+N1bDd7iSE2beo3L954+BJ2u7URKSEpzoIqZ+YODnLfobpSCKqBhkWvRJdYRFgRpjmlefkAkKejsibgf5HhmsFkBXm33mPCpZvCoP5x4hapD8iSAfgsMDaw599m7QkRKKZUTlZva+xeQk/RpaLDcgTCBnTEIDt0GWCd050wyGExwHKAmgDPc2KU4hW8efNBTXA/P8+0Mj9/x8nKcBqDplJ/7UG5WX+p5MBhiml0EtRuN8M9/Kd39L76gO8unoKxnU3nV4bn+8LF95/Mx3NZpwGdBWfMcg4Auq8fy9WhMCLRtsu2z/krFql1/OOtxz14PapWXOrxcu7qmQysNmi1oNd/fWbp83uYFq/SrzJHSIxACKohzvuFdTJIs3/GCsRp3Fl7f7cqvHt9t89a5ejIXoDJMKl9KXZL9uO0nv/vnfj9TuWk1yj+ncT2/riI92Oiuv6Rnv05tMzL/hYwvWM7wepzCudxeabflH35w45iQ4Wii2s0rp1tiwLSKBYbB/NTmmuSmodM1ujsOcOQTIvWbtNQ5X7rszZTT2ZgFm8Vg4mTDrzskhwl894CWu9RtzI33g+QkfbYd+uEJvN6Gn17B8038P74nq3+Syw9eSF4IDQvL95uU0ewHsm1/dJ7nGx8e1zhSnkMUYRjCZEycna99UVUTC0mC40n5n9niue8himuUlX744T0o61mk7vCcpOCdzOp8PSdM52S4rcdjeoaedPWV0xQuhTvXMEu1xzGMxmUYVmOtly6GUcaoy8vlz51t/PmM4WAPaqVKzcTJRewqz8H3T+RJvmqp0eOzdJbeYzC+yKyX85D44s7HAZ7APGHetpgojBcTvZGtdGSbn++NIDABWhWmpicxtb7HZECY4GQICc9DltpMmlbb7C6cac0BUqDqD+8GDwf4Yykq4D3Bc8327E0w9y2KjUypjCJFUUEJlP92HoHM5TQxDWYu66Nx/uJlnKQUdGXcztJGoWkXsZxqTslyTkGS4v6cOtgBu5w15Mx5j69TrhqdPq/U6G6clsGkLskyblY+ebm2I9myyBWGMfoRmhqdBVw4nY1ZcHDM0s0Jw8LzyTRRisP4516H5H4L7vdv+l6cZahLZAwrEOEmnm6cpM8KPSYvgJ1x+QrCxyc/adX/PMlKx7YgyhXlOX0g2/ZV8zzf/vC4xpEsE9+s1QyxEEblN7dWZmj2YIirqzAaHc/qPJeDGqsrnzc24NkzXF2lP1zGIcVRcV36+iv44lH5czdZogjHYwyj43mS5zJ38g7wBKpQi990/YBPwtZ0ycr7As7ZH1rChzK1amB0cWmKuz5GjEaSOh9+HHjwrjKMckj5eyYNvV7P/GnBOS09JPgyt8y81xwp6TPMAPS52czlvRJM1W+/hcNh3ulIcS8Kv1bd7kXCYw7cBDvBKETPp/05dbADtiDNobYLbUHzzkVzuaGii6Jv536EuxMpBLvThQ8P4IWAhoWeX2xuZ1lM9+9I6eKt3ovDEF0TDR21ubN2q+Tmo8dVBTIAhBFub8PODiQJ9LrAxfVYd41ao66TEJeT3Ft3eJ54MB6X10+TuRW+bw3WSO+C5+FwCDWYfxVBuK5Rt1ea1g2XKKqIoNTxPMmXsCRwsG2sDm/pYrX6c/moogqIk6L6wJETM3KmsoJlBeUE587m0OHiTK01v3GaTjHNch57jd1UK9pw78OTpCRoTegGMIlwWkCeUcpPbWpdsz0HYdFqotOGfr8wjMzKoghjhnNW7bmcSRgHw2BSYppRnlCVOlgURZJhGNNUQWoVpk2OOM6tyrM0Cf0siQqVB3Kit3TUIUTfg6FBdj2nClAHOmBzDczzHkXNZS7XKWaFLvIx7EwgUdiyz93x+BgHGUGXwJCmQSGRjmUavy1Sc/aOA+g3odW4eI10nf0UxeAYXAisarPnBUHz8Hg2CysMuSjw1asyPHjyJdjO9YQi+GYNBwNwXWg0PhE+p7ncakkS2NoGTbsqnuS3tfql2TP2CXfY/izW90bU+G49VduOITS9wbR75156j+nTe9Yi+ZrfeDR4Pd76bWhu7X4ThP3oAXzz4c8lQW9ij5BC8mIIM0gMmINsc7laqSuQPU/5QTEx5VBnRJkqvBjUCDcySnt5uwV9kxlHu+Ekob/16ldvsJ4EU2gJ47u+WGiE5CFhGxbnup3LrQyPTbx/R2YMfvkZh1N6tDxXyTtyWdXsdfbTaIyLLdZwmG3MY+N5eLzvDVUYcqsJlgVpiqMRxDGm6VXbCKYpDoczZuBe73LC47om0zLBrFrypukxvX/PIifx6H4sOfhcvg/TAIQAdRv4Dcy3VcTX1js6y8DzL8gLnecwDcimq+JJftthG3y/NP6FPp3ruaIIwhB0HaSYJ35fm9Q1TkFYTCZZHBMhCivdxWFo7LQNkChtzATFSOq8h84XY2pNwulovOvtjsaDteloJ/Gncc/zk4lWNHPMPrwXLwdugCVBSyGJITx2TJoWvq+CsIhjFScgOLZb3LEx05k/b4U+l4tsFMxc1l23WN/KfMXitOBRlmHMWOHTLiK34EGDuscmMhAUREk0He68fik86+4X3dp6OYQK5vkLn4XUfL9+hK4FDetm8TZf0DEXKF00xzCNSxu/1UjvlTh6l1TNXqHQ6EfY78JCly6xU/dBvmLB6QIJsrXvEcdYFFxycC3SJS23C02CH8HAQ9ekW2rqt8p/rZljh0N8swZFAYxdPnR2NBQJplAUtLRIS0uX83N1B+M8r3r/ZjCdwnh8ET7nm8aje+C5aDiC4ZCiiFZWboFdnbcH9Ycfu9xkXug6U0NwnExK4z9Xr/i6O4DnlQ/VaNA82+LatuGqxmn1Vfzjv/mqgG+/td2H4zX89xR3FvBuH+4UcK0HVZPB5tP/+T/Xnz/zpkMh9f7CQy4NZNPrvAffV0+fRaur8eZm0miIPbZnT/KJnJPlzOXCDiVNA9qN1TYlzUaaOoUwVAwhQx2YzaGBeMxGrJtG9+5SHO28/vVHVfgLacxhTgj/eUnN95vk9M394m4PblqfrbncWElythPIXOFXTbzTvczu5Qf5il0LjPNfufY9xhNMM2ZocKdDhixcqcYBrA9EnOE390GXNA+Pr1hqXItzePMGPA9yBY4NlnWFv1gUUHP8Os6lIXU1a65plY9TFFSjx0pdJBxtt0EpHAxnPLrnmG0p+v4MFde0s6KmNcoaxTAanf5caOhUdy22zMvirT2R/VgpyDKMQoxjIJqlwUtxbru6TrlUXujLP+aQEqbTUplpttd7/Ezs01leTswwpFYTbHvOV3wNUp/dRgmpt10iq+biDPVkl68RBl/A9zqYu7BF+P+z92ffcRxZniB8r5mb70vs2LkKpLbMylJ+WX2+zp5+mJc5NfPPzluffuvumq451aelqlTnIiWTkkhwwRKBiPDw3d3M5rg7CIAgCQZA7PDfwZFcIQuP6+a23Gt3+UECYQyBKvV3y+FW3q1cylwp3xrBt4s5K8Bs8ArMIun7ODKkw95xkdX1eAuRFSnPxuOt509Gr58xx7ZbvfbiHd6nu3pMCJmTx7WWB6FQpYpI31sum0MRgh9DaIGroqEQg0gNkBz0Cd8f1LjP9pwhoYi5FCnNU8g5NnzHDU6ygRPUNdQEQaGInEuBEkQBOQEFCStH4Hu/xVTL65pOGwCi2WRn4yfDDLJ2bui3y0hmFB0NeRAHOxubP5FsumlW6sfmrjEbtSVzBDdrvbQqXBClURAH0yJPAWB7BOlWuXps/ySNRDedtmbaTNUJfX/7/Z7XDFs3bc10oF61hIA0S3dnU3+UpgexJ8OxIseGbtqEtwAOlKI0Cqaj12l09HQPkRCqaKbttPuqfqCIpnGwvfHajGZH2r+YOM9eLmumbRtQ+xKzJJqNd/xpMA3EzlgEO5AzuUUlm5me2wXmgNChWvqKLJ3tTobJLEsiXhwN4dG0sr1m2vtxiHWue7Qb4DQo8mznZzCrB1WYZtieZtqqbqaZujWWowlnEKuFP34xFsP4oN8UpuqmZjiG7Sqqtr8mp1Hgj4ZZfNBvO1vAh0CoMtvUp4ZtOm2m6ce0H25TMVY00yGiC/BWv/mjrZ1hMNqB6ZTzNJMmvJKML7ped6l8ujcgPNbSSRIFw19iZXr05Fcz7f32GoOeB6qM+WxnHM/4tNDUgwIcimYablc3bcYYPdRvSRSkccDzA5YWhWm66WimrRkWZep+ZY00CqLZOE+PFsSp20e+zQuHKkfbb79K9keyGihM1TTT2e+3d0fd6xHsvkBEErdpptlauw/0oN/qUZRGgeDF9lgELzGOxW6QKz4RM73VcY+Mz8Ptp4F8MUS/ejlOz66YfQ76OU/LprPNQI4TwYv9UVQ93N7s44IkqQiCPJj4FKKZ7qOaKNNCTmF7ovg6MQNZLJxstkrFjoUzDbXL5bW+VuZx5Z+UNZNNXsCf/wJnxax7fbHHXSxP5OXD2Qx//BGSRK6v71lo86D2srbb+O13H2lpWfjgvmy1YGXllMy37xmqH2A/znOYTGA4hMkYqHKWfv5zfGtnygt9QUvFleHrbvDO2S0X0jLp+kNj0CuXdM9jEwuBZQjYkn0LvAyTDJMJ7ABABxYNeVQplzLn3AcMNaIS1Ojb5qgO5iLe9WE0JC8TGa3COpNHi9jV9XiTvEgmaeLHQsju0t3H//C/L977wjDdobW544wVoqDEeQK8a3kQEwcdUio57wlYTWS0KTdySB3sWNjRaJdKF5FldZ9wef++fveOliRcU8kRtueccB+jCKIciya3vsH8qPMtITX83CUkYZTBHNHRiIxSl6kd3WwNp69+/PaftEmn9w+PvOXB7eo9BVZcAju7m3/65/GT/2lZmlqdZY+LxRfJb7zFR0W2AuDUSvz49cb2xpNXT//oj8uFK0zkdLe0bf74nOyuLd354u8Ha+tuZ1Ez7Pe231NGvHZ/7eFgbX3hziOA7r7OMB0/+eOf/nl7+8XBmiAsWfSdtXUl/y3A8v7n09HrP/73/7T9/G/vmEBMNazB2vr6N/+xu3QQKDcdbv5x+z+9lE+OtJ+S9U3l/xzcWd/ng52Nd558999e/vzXyTiYBVmUAlMg/4FM1ta+/urfu4uPgNHaPI5D/9XTP+9Ef93dfB4H0yN3HgxWv/7q3w/WPtuPQ6xz3UdP/oq//C3zd/+yTV6YpT3mtvvLD78erK23l9YmofqHp9L3o4ed10by5OkP38WTzYM3ZXudxTuDtfXlh185nf7+mry7/be//L//z/DFxn7LWSSzkdAM+yVZ1JP1tS9+62lLx7QPUjWf2e21dZr/bwB39j/3R1t/+pf/vPH0yWRaJDGXQmgMd1x6Z/3R17//x4G5ftDz+a4z/TbeePLD852nJDzaG3c+22/vGPLzNdjB8cYf/nV784ngQykODHW3v7ry1f9/sPZZu92mh/pte+PJzsbTcHrgB7Jb3cW7jwd3PuutPDQr87iurLG98eT5X/51urN5RIa6faCup/FnpnO0/auN1/sj+XXbdjuDwZ2Dfnt31NUjX2Hqk127ePToyHirR9H2xpMsDmdBNpyKPJe+IjZtrdtzVx4c1z4Ms2ko0ypRlT9cf7j0f0HvoJ+j2XjjL9++/OGJfLqZxcH+KPL6i/uzrz7gyOJg+HIjHD7Nf/jRwmGeyyiRswgILcfz1urJZiu31l+mn6OuXy6v9bUyj2vm2JqAdzzBPIck+bhZuM+/SmfVqWEyV67veWdREgKqClmGRSGzHE7ND1YzDDN2MobhLIW6CnS3cwLHaeVllXHycW+zqlb0zh1wzq6e2Yf4ojkvX2gUQ5yU9tsZ+vnPZcIpYFtSVWEyLV/ZoH8N5l1SxQuMJ3uM03pTTPUKIc3kaDfPc5Bd9Fy6tqbXLJExAhAOgLq0THAYainECYYxGFy+Z81MIR7jSx9GFngGtFTQ394nmA1eitGOfClA5PBW0kQBeYpRjCFDjed6MomSMNYMx/K6d9Z/s3T/y8pnK1RS7qOleXz8klaxv6YwS3CYQtyCgQaO9r5y2QXkAfgF5AgEuR6GWpErADKKxHCUEwKLC8zrlEospQc/ykB1oStATInvw24GmdkEOTaYG3W+ZUvRjJHLc12odsFS+THq7HKIomZY3cGdx4mMN2e/5JNI5FwAz6rsv4bA6cZDSCg4fDof7Jlt7Jnc3IU0lvaybOsyYhBf/f0uh1mMfgSmKtqWEFzKYzVfjYHGZGpClJFxSAwFWRMs9PaYTHOZF1JTBCMCz6IgNiVINeRSTkLBi9MYNzUPs+VdMq/1NaydU+e4drvlP9Pk476sur0QVa5vBpNJaZp+NNf3vLMoFUXaNhY5JCkGgSyKZqIej5vBF733FFMfnj2HMLweudnjMX77HWSZXF+HpSXpOBAEzYC8KupCKra3cn8mJr4Y9NmpWSJn6P+o/LmA7D783UDeMeQJ2L9TjLbwmY+7BKmaafHYz+Ok1Vv2ekuaYZ1Ukpr91cedMb5GoD1Y82T/o/LEsfRf5cLPqvMcPh3n6ps4yrZHD5vHdS1uBdQf4dstfJ5ACNBvBlKDk6mVqZIN3SyXEfYYCzlT6Rxqt9Pur3/zH9vLa91n/yswfKq5BWYBjlWpO9C+Df0WF/DSFxPS+eqrr+7eudvptuvg6he7RvK0LZmjqHsHYaputpfWTK+9cO9xHX75agTJX8uJ/PUjeXf5IFzzQ+33lPXDwdX+gU7odT7/ut8/HFy9OVb+7SdjZtoFeyuqy+suff37f0z//rjg6rfa9xa/Xv/He93/cKT9LyNn9GTx3fGw8PCbaSC2x2Jjp7TlvrwjVxdMz+1K5kDOoHKOGpa7vPJlz1w7Jrga6uDq+hPTWbjzaCqW5fj/p+bZF1/D3ao++uHg6j0FWdXdzuJa1+4u3jscJHw4uPrN8zJK3c7g0a/+Q+dwsPSzLRj9EZAqK7/R1+7bdQbBMe2fb9PRn5XIdDh7S4Vzuwtf/bv/Y/Dw9xtvgqtdE+6ssoUquHpvn4rxhw30J21r5e8f3v3Mdd4Kln7z7PZ++709hXVm3u+QffH5g3ixfaBsHw6uPtxvXm/5zue//VBw9eF+0/Sj/Xa4/Svf1oL3tO+8SvZH8urgILj6Q6OuHvmIZP0Len/t6HirR9Gdz7+pg6v//BzjWCy4+aBN+r294Op52q8u2Vb7rfFpOu21L36b6I9/FInKi/1RdDi4uqimmmrYvZWHCytLv1r99YL7qbP1le9osXPpi9U1NI9rH3LLg+UlSBLwWh/Jbq3ba3ppFXNRfmWeXN/zzqJkCrgOZClOfUAEflNK/pXP9aZC30lzgD/23m8CX3T1FGDbqCiH4hqudrhynMDr16Vt/5vf7PFCN9bxpaLOrU0zmaZiPC4Igq4jJZ9UbirDdESGAKAL25UnO3gqIA9wGuFMA0OnljDA6FgKU5zOoI6kOshervzMIUw1aSpwNKi59huHOJ3gdgBTjmhJx5P9d+UhQFVp6GibYGeQUKA1s3E65rVNXM4tjbx3YtW1uH3YTTEOYNIMpwan0ZyI4jI7xDyhSYpBAYUKHw/SV3Wzu3S3VMcVbYSvp0bAgacYZxAL6XLgCURcHseffN2RczlLZYKG3V9bfPDFYDCwqpydYhOdCaY5kjcnWVRhhu0ZttcarOytM5uoTcv/O3ggFxePakzvtv8gCAFN0zraoNPbO+CrfJJMASuUGgOqHjW3Dkf2fhSaYQ/WPltbO/p5tAHaC3x3PFg9NEMqRzBWpcbkwmdy0C2fLskR+N4Spqia0xn0uvMe5ClMVbyu2elKDylA/758V569zqCKZthu13a7i8ff800EhGZYby3IkQl0s3wuZ1F6vePa1/y9FgF3TBUFqSoP8/dqht1ftRUPI5PAmPM4adty+aHabR+s42kOwymEseF2F9vdhUGPWebHz6UENVLNRA169+Taovxov1nex3fAut+qrvtgv00AqYLvto/1g5G88gF5Do+6/ZHfvSO777SvR9Fen4/wOUeMRKebLXTle/unbl/FMmAxQqtAJRMrC/lSVxrmW1sm03RPW+rk2N7BopD2EvcWwTCQKfjWvl2ZtYZjW2Zv6YFcqUbvp8xWH1G5AuUzr+36W9VtLs3d1bPLbr1QM7LyaScJZBnwAm6M97iue5wv7JnKRhOI+75tRtdwYVAzDMvrOHobXK6KWeXWjnbz7a2cICwtqZ6nUAU1lRjGJW8qhu16D+/pmYGE1Ifi9ec6WItwN8TpFIYIuAB3FekdNbMrv/EEt4fwigBdEHdb8H6/sSJVB9odudDFxRhnDN/SZx2HLg3Ulkc1jWgqsqZOdYOzhmUq9++aE7mbaX+LlN2MzkzozGs+VR4qBZ3UfJLDgYcqh3Qqh6kMjuFPbnAeqH2SE1/07KLnSkujAE0M7tmj4e+9Oig4hgkJEyy4VBm2PNr25Hv3SkuH+4voB2K8W2wIeeoItWuHa2se71cbbrXgbCOfq0rIkCYgRF0E9lz8e7VP23ZK+bMMplNU2EE+7ZU2gI9lCa7rHjf4yOGICu02WCa4zlyjV9Wg2y07Ns8xCLG47FiDKkagtPPTFMJQdjrNK714cA5JKnSNtNvKoH++k27PWwsmAw0Qc0gziBWpEl4a63kxnfGtQPGJuaCoPafTt+XRmnMaGH25SoGNcYsDb8kFCw7M470apzDzcSeAKQFqSa8F/Q/5sWt5zKLjJmt5NN0NHT1hFlXsys/gOmRxgblOo+A2ODcFhCkD1RVJ/GQ8HKYxVwxTd4X93hJy72hdlYcqwUzBZ4fN4yqXfiogPYY/ucF5IEpgYxuSFL5ahaUOGBfb8QRBVaSmgEY5kRDGOIvxDMl7rpB5fG78vQ1OCiFkkoo4QS6QKaDrRP/AaYWpw9pAblMYjpAL6PXgcHWfrIBxSKIMPQta7gGbt0LBNiAvIIyEP5Pv+Jwb8/gcFXS2V/X3zI2xuhLyZAJ5Vpoxrdb5eadLe/jxYxgN8dVrGI6uRz7thbME30DUVaAta86Sb3vjZHsbnz2DrS1IE4BLLT9WxQjgNsOnP0EcQ78vr3I5tJt3usKw7dG6hD+loGnnbgfW3loOhY/jHLMQpgzKT9QcYTKJ/ecvgn8LjES9Y4D7/shGVeodWCwg34RfCpkXh6yC/RqnCQ7H+JojHuM3PgwaWfqLB+nz4OnPqmcYv/+ddueuWhdPunQveoMbvoRXM2JrJJ7/z/DFkNo2dZa7xWMDeqe/JwceQwgojuFPbnAeyAoY+SAEmuYHfWjnqQ5ISxeeITyWhwlu7zIJZLl7lqlpVwTnx9/b4MTmMZd5WuQpCq7AsWEqhiqXu1AU+MsmS4RMi7cahyn5eYekOXx5D1f7cp/NW1dhsYP+TAyHOebX0ud8beffmyrWJwJWdZ4/Uim6roScJMAFaAQ0HfRzO+PSVKl1MYn36kjfvXPiO9QVsFUG5KJcJRfPEnyTUOUbS9cDUl2o6snGSRjiZCoBUFWP4x8+f/tsjxs5Tcu/origmItbjyjiOzt5XkjLVgjFdlthCmoqfmyVZ7b0OOQJxjEEHE6cylF7azUwFVRzyBIMY66aKYuH0ejpD7+Mv9+if+N9Lc8/WBGAgmJIWwOjgLyAXLzNhZNBMsVNH0YAeLzf+M0iLfNcZikqQlehIIBMQc8mh7PUGjQ4P9QzQpHldJAgW9DvyMGReu/zQILIIcshE6VSIgrIK5L5D/In31JVvs6T5EgpEpR41mo25zJO650Ndf2ilfjae6xXK2ySQZTgLMLCk0LA1liOZ2gb4FkHfrkTaLwEFaVUewsBWVHa4eRSLRQuMcxJ5ZA8sKNuIXQVljoQpbAzAYVC1wXzwqc7F+Vgy7lkSjm0KB5zfAOOIU0dU0FSAVy+lTGecxyHFduTJXvuweeMyrYpipT7ASDinCQtdXZ6GKHGKFMumUVUuV2jsigwCEHVrlal6D3u4urixC/wZH7IBpeMOurBsssRSMnJcrOrcSLTBDVdlm/8KlkCFxhzcZuxs5P/13+a+DPx4KG1uqYv9Klnk4/6OjRpLsBdBPoTfJ/KeBHufZKqCjyHNM8DMSHjp7/8y3/5v38Ofwi+kF7/9GXYQwx/pj/lmK3C5125/FG/cZ19PeUxLI37CzP3c9amitflzQhpcJHQOlH/dxtOVvwdeXBX77acEyu5BRQRzHSwW9CwVxyz9WGYkCRHjUlGQbmJaRNICNVUmiG+qYE+CeAPT2Way7//TN5dgFPYk6WKoUJWYJxhmIClC1VpHLaXj7Ytv1mHjW3483P4aRN//5U0+5dwVBEXLJdgGcQ1QTnrgUFRGkqhErkrFIVjIec6mKmz04dT9AziOkS/1PiCW2NT1dmSWQazGYbhcZWihYQsx6IoDc7KR3feokmCNXGxzLK5OJn3Rl/lfjRNMEJEhCAopTWMuWwSQkBlexfXBXWdZ9MAQ79Qb/mZP0X5IKcaJ1XsAxIqFxdgaelq8Q9fZMzFrUR9qjqe8lnAk0TyyklvaETXyRyrPFOkZ8A4gxQg5adVxCWXMuVFkUR5zsfj9EWx+/TZcPd1ygrT6blen7ETmwccigyTAKczDKhULNmap252zfa8W2RZP6NevtinHSp1AfOzNhKgGhgaGPnbYd4NGswPYmS6uUsLsCNiSp3SE/jmKCdarhEhp7iDRPZoX5AMqGiKQr1HaU5hYweGU/Cs8k/Xruz2joaGBYcohVkMhirnP8RGLA1j3MvPlPu8xABg67J3quwlU4O1PkxDGPmoKvLuAFQF9qNv8hw1hoRgpW82ZvPFofIeSz+EnUn5uqOHl6FRcBxHJMqwbcueJ7Wz9mUQlIxwQjCXJBNEgJxnjNUMFFGEmiFt/ZIPwm6NeVxXVFYoTqcQBsdVii4KDAJIUtA1adsX4ZWtuXBL6yKGyeTjnMx7Bv8hPucwxBcvIYrLZ5zHPFYUsOy9i2tzwFE9b1GUz0vp7fWWO458/AjuP5CO02wzt0hBrE5VZyGsP7INHVdXtG6HXXSGbSbkNMunfjyNt19uj3/4WcxSb3VtcPfX+CvXWO0Y5omVuAyTXdz0cVdDwwBbkXOlG9Rsz1sBFwFzBqa7rDNbIyexKxiormy70PZx3IyuBqc8MJKykMUsFM9fZQD5ifLrlJS5Y2eWkE3leaLtLtsDqjNJcsQmrPooPt2PejFQFei6MItx5KOpy+XuJYtaeymfbcnvf8btMXgmeBULbx19E0XoGKhrqDahh7cPSQabu7Lg8HBJNnngt9s83s+WlBKyvLQqP3h8kcF4DEkiWx64zkWUR6izeScTHE8k53t5mx/FYT7nPC//FGWPUem4w5kc4rjsB0SpKBKvlfeYUmmYoOtYFAfe8tumkGmq7Pb2+IfPD3V8ARdQRzTUzsoGl4c9Xt8cOl3WbdM5+R7PeLfgxIq1yMfReDIdb0+DXUv1Fh9+4T1aTVaE9CgD7fgD4tpny7FIYY9zOMKZj6McU092HdnWPhxZUfvP00zwQk6nBUGwdQ1Iy5GFI1VDWvQk25kGRh9WU4hTSJrR1eC0+hOzoZVzXqRKyEUYcbVUNHAeN3J1QNOdwo4AkUJYyAiFLqXApqjce1T5T/WjXgyqGr+wPYahDwDYsqRzqRrKvpdyGsC06sb686qCNExC6HvQcuFaVJCuPd5JgkLQyktPdE02WsnRVYWCZyEIzAoSpLLzYUOn4BDEpUZpaOeSB04J6BoxtHIxlBLz4kpkvzfm8SeuxAlsbZX28/r6BeVP1nWAAeD773FnR/b7e0W5zwNxjC9e4u5uaSRfX/dj9RR73vKm/tO5LAxVfEGSQBDAbAZ5BrQJlr7t0EFbgj6Q8Q79BTy68tmXfffe/S/+QVlxtswXCURz7Deqg+0YAx92c0xrrtcQp0xqfbnmya4mzQ9PernxKh+NiijMdRWXltRHngbMZqrUNaJJOqfneW/Rlc4qrAPACDabN9vglFs32qv40FOFZjlaArOQA0Dbo3OZx6rudhbbInTJL0DyhsbpBsAx5OdroCnyf/yIswgeXNUaph+qNnyVUXu8J1PMcnI8T+9thqnBah9UheyMmfDloHeIPvGCDfWKXyMqUN3FgkPFsSyvV/Z7Yx6/71ClLpSlKBeUP1l5tqU+wtms/M8sP8l3D+VU5zlI8ZGhlxfg++XftcZhb3ljHp8DJFXAskBKiCL0fcwL2VjHFz/M96o48jAoolgqFNstaluoa+RSTs0Vqtp6x3I6SDRm2QuD+yvu4+7yXe7hiGzDXOYxs6FVQDaC1wioo8lAY1JzZNuTXUse3cr3cuQKKLgMQpEXkguZJFJl+Ibt+c24POGey0BlsuvCLmtiWRuc/sDIXIA7HhPo2LlEf8rjSNRlMT7qQyZU0QzbQE9DO5HTaTHVQHewo4GnQrPaXkto1XI2mkJUHRbmxRW1BD5UbfhK74YcZzGGsSSC6wqa+iXUGD+RGRFnGKeoq6hQeWF1VFUGPReSDJ9tk1zIagRezvultFwAdR0JgTSHWQyzGDUG1yiSvzGPrznmz6lu0OAEC4MibRuLHJIUg7AZV5eC2l/67Fny09PQ0PF3v/PW1lSFIlPgck7Nq/x/tHuUt02u9vO1rrKoGXoE6dz7jWKBM5U7m+JZDtkiuTOA1WP8xrXHwA9EEEpKodum3RYduoSxi2B7btDgI8poxebNqURPG6Xy+fM8z0tlVIh5fcg1Qgx/Vn7qwuISrHflsi1bTd82aHAYdaXlTHCdFBYFBZWrXMQuzvDVCHdn6FnomHBhFZhrDm2dYZpjXkAhrsjRBkxDsHXpmQBaYx5fTRzJqDxSIzpJYDzG8RjabdB1MC72BFdl5e9mGc58GI6k44CmzqWwzplT3eCaAwnBmif5AhiPCYLKpKJgEZaT5ZhxVdesVhT0fUhTME1w3YvI2L/ZqkDlNY1TwcX+REfPIW1v3lNoBKRA64u9BQZKVT6FeApDiooruxoYx7Q/eL95vrfUVPn/GnZ6eD+RE16QCBKNnIBOiUpqCt0ADaVEQF1aDhznN45Tyd8ExDAFPYeqbK8q30fZnk/XSw0anGA8V2zepZauQWLmlkmSRJ4onml/VvpkxKS1Ltsdudh07IXvrahUJaOrYixNjaKrCCkxF4QLqaBQURK80mGDaQ7DKfoRmCpvm8CoPPV9ZjFOQzBVoTFgdA7FTZEKLfdtXnbae9pEKYx82JmAZ4GpgX7arI7578MFpBkkH1Ekyx0/SmRSEIHENcAx5eVW4L1lWuzhjMp3a0SPx/jtd+X/vX9fLi+VxupFwrblwwc4GuGLlxCE8Pix1LrNmtjg0OitOK7ri6uDPIfJFIMAX7wAqsiHD+RgcAtLpp11p8rxlOeFXOjTfsd89FBlCvb7J6iDQIAyqdUXewuMbK3Db3bgxXP8cROefQn/ri9Xj2l/6P1Oyos3q2V9n214/kz54w7sfiU7Gphzm8docKUNzrJyF5DdkZ/3YOkYvzEX0jKp52DBJSVoGEgQ6jOCM/Gff/CpGzQ4IRyHPl43uABNI5qKc47PejaZYP8Z/yXHVMiGu/syjjkqfuD6okGDT0dRgB/JMJa2mxsK0NKYP82GNYvxhw2c+KJnFz1XWhr9dJ/5yIf//idMMvjyjlwbQNuWl3ufwzv+cALjhCmMDDq4dNn1tG+XeSxVFbrdUtsb7ZZq35Ea0XECr1+Xzb75BpYuvKgCq7zHcQI7Q+AcsrRZYt6nzxJQ1fLvFob76sbesLyCjMcAMPVL6732Hjf4NNS8vnkOvS7zXNrrKkyZd3PNIAlwEsLUg54K+r6LWAW9IxcjnE1xVP6EjD/YvvIYp1Ew9UeZPyFBoBm2o6tqtVru32dGAoAgE+n85jEBqkrdwe5SOZO1Hiy96zc+3ANpKtue8FzqOFRV99SCE7HLHo+6fnV90Yy6Bp8CVSXdLqnrBWQ5UGWuE5f92USBceDytrouFQq2AYQg55BeYBhc7Z0LYmg7qCmgs8Z1fE5qgsxzmaZIkRIKF1yenaK0mCg4RgnOYjwRH/Upn1eWQyvLJQHBiCRITmceV15oCCO42xVtS6pnIXeUwMZ2eeF9IZc6cxsoVLYtmeYwDXHoo2PI093ngz0mIElFmGBcEEsllnH5FeNumff4ImtENziXAatI24asSoi9ZZCOA48f7100uNnmccXr68/ExBeDPjsRn2qAkyf4bxkkS/JeBxY/msf4nvaVx3i68bc//umfd4evVYUN1tbX+/1u61P3QERGqWuBwWQHgR5Tp7rugeEoVxkOBuzxutHtnr1np/bd1RfNqGvw6ajrBQDAieZsA12FxQ5Ow9Ie8MOLy1GsvXN+hEtd7LnSMZtXcS6ofYOzABVKdA2Vi/XSa4roW8Usxt0pUxRy6XzU1xGWJu73+dDHF9tKkJDP125+GsItM48/pUb0eYNS0HUwDTD08no/O3o+oxFsC/ICogh8HwzjIvioLucNKuA6kKUwCzBJII4kwm3JuNbUJt7+JisQ7/D66jrSk7MEZpDsVmRFNrTmyWN8016Y4gsrNeLZcDIeF8Pt7edPtp8/CZOwtbBaLk1vp1RWNai9AvIUIwRSwFxrKQJB1NS6rMI7e+uehyGTaSrG47IHDP181ajad9eMvQZniyyTw1GeZ+Rw1MOxelg5m+qLW2seL3UkQRj55X7e98CzLuJ3a+9cmsNSB7ouqqwZvOfTzwUMffBDaarcs8gFFycxqtFFJ7AzhZRfPh/1dYSqQNsSUYI7IygkpPNtmzUPc85RVO50/mGDuhAQpCTNQWfC1kAhjXnc4LDp3mpBUcBkCnl+kB09B6Suw8JCaRgPR5jlcnXl5prHVS8lCWQZhAEMh+C6e6WDGjS4zniX19fzFKqgphLDOHc3lJQ5F/5skr768clo49ls+DqJA2RsZeWru1//w+DOI6fdf0udBWsR7oY4nZbq5SjF6AwOCCoPw2g3397KCcLSkvrIoXUVLsdpcoMbXAMYBq4ts+Eo39hINxnMGfVQz6b64pb2myqXu5Bk8k+/4JBeXcbgBqdDkpMtn0xj0TL4wOU6oxdZd7qelTmBH/6Co0A2o+vCUPMwKxQ2x+BH8picyMsdIY15fJxeBnGMSQKOA4oCl3KEWFWFlYYJug5CyNp7zOcr1KGqstsFznHjBYQh9Lofyf9kDAzjWlYYrnoJVLXsmTiBKAZVbcbv5UDVoNstX0cUQpo1hxRngnd4fc/6pUmtK3oFZAkEPo4M6bwxj4UQaVHwJInCJAk5p4bd7S0t3n20/PBrr3dUodDA6MtVCmyMWxkkrFw0zyYaknNIUqFr5Jx6oEGD8wNTkDmYJIRzyTnw+QKb6tl0m3PgFQqOIXUGQVUS4coyBl97VXePSx81RplycVU+C4FBikkOmlvYmlQIvfhZaUwgSMq97taOrjrTfhZL15SaIrXzp4d8w8MMGzsyiKF4n0GTZDAOcDgFIOjZMHB5zwXtCpgmjXlcIY7xxUuYTGBpSboO2PZ125Mrn6o/g/EYCIH1zz7SvsrBlp1OU2G4wamxlwv98gX86c84HKFCZZPJ/wmoT7hdm5wrr68j3cf8Sx9GU2Urw3QV1t/aD3TbXnwg9YV2nuqG0e8veJ2e6bTfZ2brHVgsIN+EXwKYONj+dPOYMWx7lFTPTWnDbNzgukLTyGCB1RdzKZHVbKovmt5rcJ6qrtx4lQ+n6BnEdYiuNscQtwh1pv3uVK51ip4rXePcPbTz8DCPA/zuCWYFfLYsBy3QGdWUs2GmaMzjMwBmGY5GkCRyaRF6vbfKWV8w6srMWYZFIbMcxHyLV+VTRVVFzoFzEO9jO6uq0WIcYZKU7W37RlUY3qtozYA0KvXZIcthPIYgBNuCdvtoVEWdC+37GIYw3gXPazrspOBcJolI0mI6zQHA85htEyQKJXgmvL7vUdzB6MsVRNiGH2NIBrC2twYiIUTTdK/Vtw23XD90XW+32/oHFsOa8VUDo4A8wUgHS3xC3d0o4js7eV5Iy1YIxXZbYcp59UCDBuduHqvY7TAupJCQJIIxPL7c+h5/coMGh/XSc2BjTnMY+uiHYNqfxMd78SgEBAmkBbomcR05P5VDg30kKbweQZLi58uw2gPztAfaSQabu8A59lugq2B+2GA6noe59mYPp5AVoCrQ82ChDZceU92Yx0dmXgFhAFyURmOrdZmJu1VlZizycggHgTwRfdHxvLg1f+lwCJNxaf/fMGKk+tktC5RmSJ8dwhB++llmGTxalw8fXL+oiiuPPa6/Ufz9H2YA8Ou/c1ZXNcukhobndHpa144GmKSYARTVnvXmc+Iaelehsi51Ryll862EAngu0wxSFU95sLizk//Xf5r4M/HgobW6pi/0qWeTq3B+3KDBKVDHQcSpDCOeJOX1GbKRNbglOA825kKSkLNEyBbhhiJOzcd7CaZdaZLJvMAHS2ytLy+gHscNNHQERCkIjo5N25489Q47CfC7J9Cy4cs7cqkru6d1tNXe7CSDz5Zlz/tU5uTGPD47U0qI0kSsq0MLsVfFWlUv03V8qDIzTqsCjmEgk6S02+dZIGte3CSB8QQohXb7rWep+WmjGOIECL3e1Z7rSt1Slk8RBJCmIKR0XWi1ZJOKfIbIMhiPAUDadjmcjjGWVE32etDtgqo13TbXRlXIIJRTX+qmSBLcn+KUoKGhfm4Vm+va0YisAA4ik2kkQXItA0KQqAqaJzpfqivuRuBHMBMgTpQ5WdepzgsouJzORJ7LA6VQI7rehIE0uLaGDUVKMc34ZFJwvhfS9FEfcoMGNaIUnu9AmkHXBVM7vZfvXQjATJBcCkIKRsSp+XgvYcfke3npVuU9bgbJKSAl5JVfTCt32NP3YZzBLEJE2W/BWv80d6j9xpu7MJqiymTfg8XOlXunt9I8rus8Jwkk8fzVoS/KPH6nMnMcQ54B/bjRvpcL+vo1PHmCz57J335TWss3cpLXb1DVoChwvAtb22Db8Otfy9WVJpv6cuA48vEjuP+g4WSeE0kCW5vCMMTCkuz31UGP6Tp6HrPMC/SaZjlMJgJ43pmBrgvJT7yUgrmIdzNInsjvAXYdOAExcu059wMRhFJw/N3vPNNAy1YskzaegQY3ADV3d5IKqMgHGx9ygzkx8uFPv6ChyS/vyLXBlfOqNWjwiaj9xqMpAoJroqJcxRF+W83jpSWYTHA8kbXTOMuq9F1y+ZmrdWVm0wRDr313GISYZnIen/Z+LuhoVP5nnNzYN3ioUjdubsPWllyQZRfdpGzqSx+Hur6XoM75R0PWpabKbg96DS3zvKi9x0EguwWYJhn0SsP4wn5dSJFDlkbjrRdPiiLzs02t3RN6cdINofYe62AmEBeQz8l+nGViNuNTn4+nPAhFnKLn0gf3tG67YW9qcNNQV2JPUsFFM7wbzIUohY0daNvgfSGXOk1/NPj0VaiK1cpAp0gJULzQczpGZduSaQ7TELcnaGoQxLgzwTCBlR70PKldSYaKW2keV3Wby9Hx/fdYVCXedW0vrPpqZK7ueUcRcTKF0QjSBKAx/A7PtkOVure2IE2bLjmPHsaFBVxdgSQBXWuOr2/UZglFLMLt6fPkh+fcT7bH08Edylfzi1lmZjP+45N4ezvPcomEqLoC0FgODW4a6vrVYSSaGuwNGjS4RNSxWnEgWkzqOtFKS+fiFiVLE/f7fOjji21lHNDVPmQ5FlzqqlzuyOUuGGrjPb46qj9jUh/hbIazAPo9aLelpoFlSXo1OqTyjmKcwMtXuDvec3HP+10G7TYUBSaJ9H0wjMusNHZOqH3sigJxDGHYrH3n1cOuA70uhBGwD2d0G/peDL/RUJLMBUINVVty3NZg0FlYMD2P6RqhF6s/8zBLtkb+317lLwrkwFeqA+WTJ6ERoKo0THRdaGeQKjDXUsMFJInMcqky1A1iWtR1SFOJtMFNM4+r+tWWJaoT+Iue4w0aNLi+qP29SYJCUErR0IiuyVOvIWkBQx+mEZiq8CxQTx7MTEm5iBkaUopCQJrJLJNiPm4dQ4WlDuSF/ONzmQ1lmhNTB8sA14SuB45xRZ0vt77Mb1FgEErDBF2Ttn1V6h7X3tHJFLJMhuHJqkzbtnz4AP0ZTCYIIFdXbqB53OCClgcFLBsA318LvUa7LX/7TX3RdNhck1tpO+5vNY3/6uvV+w/aus40lV5wlWaxnaT/ZbP4aZvyjr7a0u+67tqScvIDDkWqDrR7cmUVH0Y409GYh3+k9qp5LaXtUdOkVEFNJU2+cYObNtOr+tV1TDUl0FRib9CgwZyo/b2TKWY5URm2vE+qNZ3kZMsns1B0LbAtUE5eBKFezaIC1V0sCjkLxUyXeYHzf3dnBru+3BwLP4J7i/j5mljpXV3buDGPq4oZWQZFIRWrYs29GhtY7bvT9fKfpzCt223gHIcj4BwGfflmtsF4DFEIVYXnpsJwg49CYqnTgarKY3Lydf2mVoA775MH20bPJYwReoF+pSyJZuOd0bOf0xdjCDldM9T7bWWxY3ltRaonJdesvcc2el1Y1MEwwKIf8ELHiRiP8yyTioKA6Ni02yFtjzYVqhvcVNT1q/NCxrFMUgmxZAoaBjaBElfC/KjeSxggRWqooKugKpLc3DdTcJzFgAhECkMRlo66hhcc0YCIlFKqSMRmCoCpw9oAkgymIb7ehbYt9UNRelxAkoo4Rc4lqcgsPqXWdCEwSDHKoU9RZfIURZbq1UzXkRBQKNg62MZxfpN3v6uqIKTkQipE2joMWtBzr3TSXkMSe5VfDgXLQsuSp4gfLngVdSwPPM9BgE9/giyTX30JK6tNheEGDS5JLRvP/G+TONnZKbpdbLfblF5c5u1svPPku//2y7NvUzVmn7fIbzrKvY7e9mrL9pRmADATHUQ0wGKgkfclNY3H+bffzUaj3HHYYKDev6+3Pdr40xrceMSx3HiV+zMBAK5D1pYZc5phf1Xey8hHBYltY8sSli4VemOLbMQZvBqRJOGkyNu66LVo27tobnlCCdMY0yShTTET6Lrw+6/k6xE+28adKXyzDkud69Ettokdl6x0wTElzH2mzhRwLKIw+HdfwMNlefXrsV+0eYxIEBVEDUAH0ABoHWHPKxZeIaDgkhI0DCQINRNmPYH3r4Us1zUupEKR7DGFHjAKznU3CWkiZAxMMSXLsoxhjLqDKCBPJVBx9r94qrtJrnJzQJ3ISF8SDmkqIHm/bFW5sTe/WCBTdTS0XEFAyQB4dUoqRjnbCQkBqbrEbjMFa134Evr/ePnf4Yc8fM/Dv1UfH3DbyRMuK9d//aTv3u0qvM1P6Y3Lkl9wkYe5mGWCFoLxgnEqxKXIf5P8LYLHWfo6jsMwfJgkCef84mVQLavXvZ8sF+IzIx8QD2xD2vS02wED1ZVdHUwG2uH7HGY29mei2Gc2pnCu3M4NGly9Q7G9vSkvGsPgqrwRfyaCEKkhDRV0JtUzYpepMkVRiDp3VM7Jd40oGREEIC1ImpXiCQHy7OSJUhwHAByQc12Rpq5czApccIgzDGMgUugKmDrVNSREzG9W1XeIU9RVVKhUrnYZx1raICovjs8ZNjUw+2Wz73+GaQhJdi7ynG0O857+wKDrkI57silj6nh3sZwID5fltTgIuHjzmFJqUMoR2xJcAKWOsK+5AbMcglDqGq4tM5XBeFoqjm2vnA3711kOG6/yJJW2hWqVVKtrZJ9RcJ67iULubKXFLjp6j5vaKLAUqqy0kJXfLSDnZ/6Lp7xbqIf6fd0ha5OpmsvJlMvp+2UDgINfFKRtuSDzcbALlLQZzapT0vS1dBJLZSCnqFX3OXf5577bW/LDUX7Iw/c8/Fs1f3UeFZPtSKLjoVI/6bt3uxJv8xN647LkzxI+3Y6SYSBiI42y0M/0Xn4p8jf+lrOC0+6vf/MfF7IvJsrOa+P535y/BNJfwnsmOlSeskiBKvUOLArJCRAKiiLVwzO3ZjYGCV98aesaKgoaOnGcplR1g1sBwyhXMNMg9drY4MZDVaDrQpLKKOLjqZyT75qiNJSCIezGbBJgHIvcACnwrOTJOaQ5+FJaF7uXxhm+GuHOriBF7ujSM5ilkxN56es77M7Qs9AxQVevtGVVS/tqF5OsNIA/MWf4DI6BzjSH+VPQtuU363sX12IiX3xwNUPiUUVXtY6q2oQocSJev06nM64oGMVya7uwLaJrpmngi5dZVS1NB4DXr9PKp6RGsfzxr1EQioWBYhpYFNJzqK7pjGGey+mMb+/k/qwoChlFYuIL11FMHUyTbO/klIJpkCzhz54l6RBWtX5u2X/dECRWoa9pjL4IM2ZT9aGhqsQPRJ5LISAIxd+exkHIWy6xbarp1LaoEODY5c5Xyz8LhabTKBYvX+WWgWcifxCI0a7u+q5BPBvp9lgSLzcNUhTy2fOEc3nvrp6k5d2EAIKGruGLlyljWMpvOlOjmxfAIxqO8789jcMN0Y48y6LaTLHG/ETyE4Kj3ZxzIKTsjX35bYsAgG1RQqD2+AVhKX8Q8upa7L9N2zro/3nkpwrGsawP2n2/ePEy9WdFFZymrK5orluOW5arZn8xjOgvo2keOStTGufZX36ISSmzqWv4y7OE0vJuivKp8h//NsNYrlQKUJpwp2yv1/LPgprZlacJDwI+8YVtUYWCbZHxlDOGtkWzTDx7nuS5XF3RklTOKb/v826H2jZRFHQdpXbDMoanm00fkt+fiLFPgjFJchlMcl9P7KmiVC9xOMoZO3ib88sfReKnnxN/VrRcYpofl991aMslrtN4Gs9IddPN7tJdC7s6aUeQCfmXAlICyoeCoufyToBiSPuIZyaOZRjxWchngZhMpWWSe3fUhtm4wW0DU5A5mBeSKZhlYjTKQQjHoararGk3ExqTfVdOAzGL5NAH0wJarYdxArZOFQVZqb8ctRAYlY4hZxFyAXFa/qX5nlryidBVWOyU1trOFGYSlu0L7Y2igFlUe4+5TqXOqHrCWLAogY1tnMXQdeVC+60E3SuI+nlnEeZclrrQaXOGCwFBSrICLR09C9lJDhQO8wwTgDg8sxxmRsGzkCC0HemY8kS1jHX19NHjWSZmM84FaBohBC8mrvCizWMJmpALRCksx7IdU2HKdMKfPo22d3LHUfxZqfq7bqkQOw794YdA18igxwDg6dOo8inZsxn/9tup7/PHj0zXIbNZMeizQY8ZemlybO/kL1+mw2E2mxW+X0ynvNdTWy66rvLyZVrfbTbjf/5rHM2E+mU/lcV3Qz/nRCw7Wsx+fBq7XjHoKe2OGoTSn5UmzWiY/emPM3+aex7t9tSFRSP2YOKLtifWllkYlPKPp3xh0fAD8a/fTQ0dz0T+0Sh/+TrvYdZe7vdU9mKX6kY+6LEgKL7/w6z2sKWp/PbbaU38pGnkx7/MXIeU8reMUC5NIzF9qYx241L+Hd5Sel1NHYSaO+Inkl/VyPZWXv/iaJTvy9/rlo/murT6p6hMWf7yZer7pXk5HOX7b7PbZfv9P4/8jJH9fK3hMPvxL7PhMAWAXk+bfOH0euUC6Rpsrbs8Tt1/+19Df0gfP6epjP/lf0xVBi2XaBp++61f/aJi28onyn/824wTCeC5NtnajNseHfQYpaX84ynPc+lP863NeDTMplPueuXXuz01z6Xr0ILL6TT//g8zfyYmXzhpKuaUfzjMV5ZYt8scR6l7Qwhoe7R+myedTR+Sf/Qaw8yd5nS4qU449yFo7ciqr5TpOK/e1N7bnF9+3y+qt5l5HnVd5aPyDwbs0UN1Zampvn6dUOf1hZHQ1KYzGjTYw2zG/UnW7dDH60a325jHNxM6EwsuRw6TmEpKBj1AKUp9YIILLeLaxHqffXK4MjCXEOUkzoHLM/CzmRqs9ktL6Y+/yCwBb/Wa9WeYwM+bsuDwYOnqcuSeOepa00GK/TYZtMuXOD8O8wxvKdg5O1aIeiwVHFuWsA15Ye9iNuM/PomTRA4WGFPpxcQVXrz3WEqZS5EB0OpPJRR0nVgmsUxaFGDoqKp4pK5a3aZOUyQEVBUNHU2j/IrgQtcJeeOToBR0be/zIidpKlS1/Er9ua4jJYAEiIKKpiiWKinRNIKcEEMHnQHZC3+iBHUNs6z8UUJQZXtSlZ/rRH+H5b/+nHN5hvLHUSmwqhpkZYn0mC7NWv637lzdDeBoJToCqEqm5jnwFGLOSNUPEurScyeVnxLQdSyfgx6Rn+wxOr4tPzfLORNFZP9uh/t/HvnnG7wKuAYEBNgUErF3NwbnIj+p5S/flODvyg+WUXYgfbvwJSVINcyqz+uWKkNCsP68qhuJR3tjPvn1N/JbJj0s/4dm0+nkJ4aqL3VT1aJjJBFRhbIvfzmK3vs255IfTiA/Q9KokfOBQ5FhkmGiol6laClpHPijrSJPNcPSDFszHaWisD4dX/EJzOOEv36dxqlcW2GOTRHLzawp2Nvg1oIp6DokSch0zJNEnolXsMHVhELA1sSUYeITmpJCyCiFjR2cRTAYwGJPmu9j0DtcGVhILATkXJ6FdVwpSiYaWmlnBoG8dtnveSEnQX3uIB3j9PfRGPRdkRY4i2Hoo2NIjV3lp8ZxiDlH18KeK9WTiFrzDBcc/vQCOEd9Ac6q+qfKoOeWdpyln1mu/p72cqhSSe0ZlkLOZqW26jiUC0gSmaSCcyBcJqXCT7g435F84d5jGQr+LE/9wHdDu1/0lHbb+e03Tk34EYTizh1NUfDhA6Py2tuUgGVTXSe//cYRvLwOQlEFMMjVFc22SFFIVcV2m9Vnb6ZBBj2WZaIKTuY7o0JV8d5d3bYqnxgpOxoQf/13Dg/je14iSYgPQqHqdx8RtPWWi4yh57E6X6heRxb7pOViEPIqEJQuLmi2TesNzzCQc/rwoZkXcnFBK7hsuYQQOBP5fb/Y3M5VIh+ulL/bB0ZV5UD+KjhZvClidPeuThD25dchXYBxG2OktN/VWq4Z73L27IVr0IXljrWmnkh+yySP1406sMGrAptr+d0qgVBRUNOIUum+RUsZ9FhRB0XP+P7bdJ13+v9Y+d/t//cEV1fyt5T0N9ZOrsmVO33ecvfl3yv7RMu7Ofanym9b5XshFNptliTi4UOTC1hdUeu64LX8CsVSfgUtmx6WPwhoJb9eFNK26BH5PY/9+u+cOjhZSJhT/iwTiwPmuoqioKoSTSOaWnZdu83eO5tOJ/9CxxYP1GBWLIz4LAIu0a7637YpL1SmHLzN+eVvt2r5Zb+r2Db9qPyqRrrdxnU8FzJMdnEzwImDLQNsVer+aOtP//Kfw+mot3Svv/rZwp1Hitc9NV/x/EgTvrUZcw6PHqjLC2x/w2veUYPbiXpFdW0ydAlj5UrY9MntQZSR52NWcHjkwHJX3hL/51WDa4gvV/nQx1dDJcnx8zXQ2NV9EYWAKAUhQWcnrqleWxOjEGcxxom42wXjjMxjhUpLF/XFGR8HHKpUUnuG84z/+CQGgMfrhqaRwQLjHBYXGCEIUFCKp2BvvtLmMcgc5JTzUZYWWWYIURg6MZb24gaSRHiesl/OByr+TEMn+qE2hi4UxazbHKm8V529Valwb+7W7fFDLfc+t0y4e0eHFNqSg8aURyboemtRBYvVN7RMUucL7bWvHGuHS0Md/l1DJ0uVbLXMpkHOUP7+4EB+902zPfnf/Mr+3apiVW/kT4THcsEiBHB0YS97uUNwZ6IrxHMK/VAG4DzyqyrZjwTTVPyQ/PvPUf+r0z54m+/p/+Pln7v/TZPcvVvezVtg0GaH5SfEqO92JvLXdl3dKYd77LD8tSfW0I/Kr6r4IfktkxzujU+U36D43tl0SvkdFUB1E2F0+Iflh5PKX38+v/yO1SSszrfBQObjKBBjzIAH6db4b7OfX289+6vIM6+zeLhl7T220G1BX6m4rc/2DLgowDYJoejYxLEbS6DBbUe9oqoMFKXKowt4kTcZyB9HksE4wHEAno01OzF8wkneYabZPMNzqhX8npWZwzSuQtVU6Rgfl5+gVIhkFG4hPbCigGPKJIFJQpNCRIlMUsnFGVA0mxqs9csu7pv6gQAAgABJREFU/eEVpgXcG1zpfpAS8qL8t0JP7KetIxEUBnEK4xkMfdL1wLOg5cInOswJwpzC1JpAbatnuRyP89pBWK94NS9JUUheyKIQRVWnHT9MO66p2O2UonuVT6soag8l3CzzeI4zj30ymMPXH2oz/92Ofi4MBgzaTqvPgVDm6ECPa88F3SehmVPm85X/2F/EgjCTyZRgGClAmQOSS2QhkcBkcfXlP0H/tx2vKofH2g5c5Pg5K/mve/9fuPwNPrghQR7JWZDt0l1Mfp6O/u15uj3VGPZX7q08/NXgzrpmOkeNZNRU1AjQsz0DRoK/+pWj69huN57/Bg3eWtOGu/LnnxNKoclA/ijGAX73BKch3l3ApY5sfVrN28NMs3GENsGrWeeJIpis1FDp7bOPDVUudwEBd2dsnIjhhE+jIs8Z1fBMZt80wYJjlEJx0xMcpIA8F9MAnm4pksKX93C1P9fpzBlqArUeOB7n3343q+MW25WVW/OShAGPwjwMitksb7eUX/3KWV5Q92PNdI0+Xjfq4GpKP67b33DzuD7zOPyfH20z/93e+ZzUj0/f/l8n/cVjZD5n+Y/9RYWiY2Ia4XQMYcHiCfAYDZSKIt/OALyi8s/f/7pKl7rXWP7r3v8XLv/NhgBeYFbHP5/UapVc8CJNp4HcToJXO7svf1EyZWH9i6X7X3aW7lpe9+hJMFANzAJyBiotVbJT9nZdpzrNBC9qLhOuabi2onlu4/Zv0ODommZoSJuZMR+SDF7vQprDg2VYaOMnlvo7zDQ7GsOaC1fTPDZ1uLOAK11p6Wd2T4KgEKkpoKlYV+Q5w1F9Cp7nD9okFBxD+gYKJHEBYVIkCXAuAfBMZp+mIZfAC5BXNbC6Zk6OEtCIUKg89RGJlFJwmWSwG5DFDD1L9tyzeebDnmEhIY4lF1KhWBQiDEqT2LJplsvtnVwIKAqYTPlol8cxt+xcSKyT8o6Ozyrx/u1YMzx8evhR3fKGm8cNzgyMQasFSQJZBmGIdd0kxwHXBV1vuqdBgyu6NWI2wzEAONBW5cnqkGAGdIJyJ/FfvM78mddqdb21z37z+4V7j0yn/b7VX7HAIaXeQT6F2KmuUz0aFVGYJzHPctnrsqpGemMENGhwFI5z4BVpeqPBu2g7+GiVrnRl25FwRmUhKEqTCcUQngO2dZZ5m6fjeW7wwf205nmeCpvljiE1RQG4WjEmhz3DWQ4br/IklbaF4Sx/+jQCgIcPTaqQly/TLIOJLySg09JUnU99zlg+6CntNquCqxVeqHVwdV2D6Uo9ZmMe31BQWv6pKnCOk4mcBeDY0OvB4mJjHjdocGWRQLQFzznkPVhxsatJc/6y0giEokKpqhg666qtVneh+3Dh3iOvt/SB1Z9Z0qsTj1Vp0NNuB3kh/ZnwZ1xySSka9D1VzRs0aPBmc0bLolxAXoAQ4hO9bQ1uHgwNlnu41EVd42d1T6ZAywYpsNemrn2WeZs1r/LYx3EACgXdaHTMT0LN8+wH0LbkoCWNywhwqP3DaSbTVNQldZFgHMu6ymCciNev06oYjS4k+jMRRkJKjKPyglLgAtSKBYYQSQkylRhLWp6JKMwrXpLrUZGkMY9vAYIQRiM5GMD6Z3J1BQyj6ZIGDa4mQun/LP8cwHSVjJfg3gLcVaQ375dVgm3VsLvGQtvMnYFY8dSF9/qN91b/qn61kG4daK3IT9qHHYcuDVTbInUhjcYz1qDBe1H7Xg4XO2zM4wbnDV3DhS5hFBf7ou3JM8zbrLlwEcizTRZkctCDdtPdn6ID1DzPBT74QllbhEthfKjXqNFuvr2V6zo+XjeYSjde5QCwtszCgNde4kGPGdaBFWnZ9OFDU9fJ6opqGnTQY0IAVbCqNQ1CyAPGk+uAxjy+0VA16HblcCg3tyBNpWmC6za90qDB1VWdIZvCaIojF1sOtCzwqirT+jyuXfz/2PvT5jaObM8fP5lZKwoFoLCRIAUukihqsUTZvrKjxzMe9//+Jzom4vrxPOpX1y/BPQ8mJibs2/fabku2ZUmWaJKiuBMEQRI7qlBL5i+IlGGYFGVqF6nzeaCogIq1ZFadym+ek+cwQlSmxEw9aSZFNs0n4iL1lP1l/urnu07X49VqIEtw+cH+2CthK8NDasJGrzGCIMhbNtZnEI8RXYW4RQ3jZa67lXWVd3VodWnARRC9tIDwbkgqDRoEIhET6QTXT0KSzkhANwABIh2HTPJ5MkX36zybBk3Yz38lhBDK9j/N2STJJsgTM07LvCFy5bCqgKqSKBLNZuR2RRQJ1+Wbpa5p0LGiThXqdfePEHGxf9he2RHKHld01zQSt4imgpNSBuqbnOwpcpTHpxlh2zA9vW+96vV9qcywuxHkrYYQohBFI5pGDAGiCtshBGkYNkX8rbpOmYtydzewbTWb1YpFPZtRsbIxgvwhT68FgCCIpNEhpV3FVPmZdDQxDAmTvW2rcA8ThtDoCBDk3AgfTsNryxT9hLHEvm2h6SS5OE4vFIUde8I+Mm+IXDmciO/L2nY7mltwowgmJw3bol0vCoMoDPf1c9wiMu2v46gffrAv3B1HVRRSHFGlwJb53k6NTUO9dKrRNaFnSKtF0ulebytYjR5B3k588Fqk1oGGA/k4JO1ehFqLVAVwW6QPO3l9r9OsVrpuS/BIM6xEZghir+M65aokzxM8Er8O9yFpU/QbI8hxkBl0pd/GD0QQgqoIXIH8+gRMBG0Pmi7pxGiCkIQJdkwop3csLGvVaspTysq+EriAMCIdnzQ60A1JIkYTtlCVZ7gI1ycbuzQdh+uTUTEHMf1kPF0tFwBEyoKXlSn6RTA0UUiLbEL4XV4NeD+/dBgJRSEdV+zsRs0WJyLKpJl2zvzVRoGpE0pYPq8CgKYRVYFEb8GwqoBhULPwW2eo9uk0XCiP3wEUBpb1eANBkLeSFqktkJ+64I6JaRPiAXQ7pOGJNhE0guDw/s1qZeHHf1TWHwaBly6MX/n4L1os+Rquc7CyMaWgKMQ0KK40RpBn4rDfBuXx68HzoVyFrSpJxLR8DvJpUsgIUxOndwAoLINbxsvMVn08oUjaHt1tkI0dAUDOFtRiTjxThFEYQaOzr9PiceXlrpd+hXfNodN9vPFmkXWPBYeYJqgQpXLg+7yfX7rdjmxb1XRGGKs3wtXldsZh+axyZlTv59WPcdb3EsuYl9dQbRjlMfIaMUwoFB5vIAjyVuKDtwdbAJCE99NiqAlVANGBpiC8DQ0NjH4Wa+k3Lj26v/Hwbn23ZNrJ13OF0m9crYfrGz6jMDlhOCn8giAIcpKIuHjsPQ4pJ8IyxRuMgD0AI6CrQAkYGmiKeCn+Xuk9jpuQ62Wi0F9X9RwuwA9Jp7svcXUVrJ73+Bl7CrrBvkjWNGIY5PVMnVRbJIqEpXNTg2eK6ZD1itsuUM4VJuizF2qWX9ggILpKKJXV0cWB/x2sNizzSAc+r1QCAMjlVFWj/bXEQZfr+00Igcf2f4yeoNdVlcQsRomoxplhUEaJptHBasODXuJ3bf4OBzenn8crkOUGgiBvv13u5ZSmsP+FdUmrCmUf3H4Wa+k33nh4d297LZ7KXLzx/z9z/loiM+SB90qvSvqN1zf8X35pGTrNZ1WUxwjyfJjmb2v2ZFIcbBNEprniAlIWtwyhsJem221TXCw+3sB2Popqi/y4QLyumMxFhTQ8U0zHi9crll/YTofYJjF0oilP+N/BasMyj3R1L/j3/6gBwH//byknrfVjUtx2lFF9P4BKWVWAZRwWt6jCSJhh+awqg6sVhTKFhKF24ZymKiSXU/EZQHn8LtFbgYzNgCAnhX5OaQ6cgVInu21oVGHbdRtR1dtZeri19KBZLVvJxPDE+dHzV3JnzvVqJr8qeez7vNmMmi3ebPNWmzNGDINSjKdGkOdFVchpXbOHPP9gTYVscl++xk2hvdRcMboKuorC+Eik77dSg7UKtF2iEqIqImbB8Us4v3i9YrcrNsvhVkWQgOs6C3xod0g/2VW7w1dW9z/x2jkTCGk0uUw9HYSivz14NMuAiWHiByJm7FubpM1+zQ9CnzCvPYTCGOUxgiDIibDOPR+yLOnUIrUtWHLrVfd2qfOo3KxW7HRm4urVwuRl23nlk1/NZjS34FZrXNVZLMZmrtlOijkOflARBEFeGqYmRjKPN7A1XifS97u5R1wfugGUG2osJvJZOP7KpRevV9xqi4XlcHMr1NVuTNDankY57VdHr9eDu3eavWrDSjL1m/i24srZc5bcGIxJ4VzpVx7WNYp1JVAeIwiCnAb6PuQIwgjCNjSa0e6u+ygSbjKfGx49N3r+WqYwztgrr/kUcfA8EQQiFidOSimOYH1jBEGQlz0iZxj8/GboBrBTJy0XMja4OrgBrbZFzx973O44fr1iGY3VcXk3iNZ2aK2uqSqt1jkJeaVOd2oiRrompeHIkYWsZLXhx0EoChsfN/blcYwdiknBEC+UxwiCIKfSRg+sQw6s1s64oefj5zKfjjrTVjLFWIyQV+7F1XWaH1KdNGQzajLBcB4aQRAEOTXIesWMkquT3Avg7iPodF9V9mkZjbW64VWq3Z2WshOkkgltaTXUFOhSPWCivN02GJh6PJ9V+8HVyaR6bWZfdieTqvQSyxQGcgVyfxtBeYwgCHJ6IEBlYmry+wnjwXXIbaOqjtiaMNPpyYw5eeAIAgSHSACnQBkwAi/0sZQz3K7Hw1BwDlaMGQZ1kvv/YmchCPLybSAlSq8uLqFwfK8dclqhRChUqAzIy9N9hICqPN6QyIzQbZe4XUapyDvgB3CfQhCCON4zKPNd19uQS8H+1YLwPKGqxA9EtRrIwkiaSoJekfMwEpXdaKscrK/7pYrX4ZrqcE2BehsopQEohqXFs1o+L5IJZsV++9paMTo+Jr3E9ICXGLMYoDxGEAQ5nTBgJlhy4wmWuudDTqh5K5OhgjH1CR5jDpEvuiEEClFU0Cm8kI6VM9ylkt9sBk5KuXrVdpIM8+siCPKqbCAFmdCI4RQc0itzFVO5qQJ7efpYoRDTH29IZEboWp34ATX05zmRzHfd6cLlMZGyOBNhtQ5OklWrwQ8/NgHgww/stKNW61GjxVttsbcXdFzBKDEVZhlseJxZSYVHrN2FSNBcTrv6furssDiQR/odrDyM8hhBEOTdJYSgSzouaatEV4TK+mY5iiAIojDwoxAY1YyYrTh5fTQgvgftBuyawlZBk8I4JH4XOiF0CYAhLFPE2fOa9yAUrivqTR4Ej3+hjBgGQb8xgiCvjpgOxRx4PtTbUNojTlycUJFv6NTUybtWJ/alY+riTIaPpCGmUYCX05gKg7gJYQidDm80hWnuy9q1Cuw0IKaLdJLoKkShsFQeRqTjkVoLGBGU7D+KnEMY9WRtL4y5X2G4WoO1TcEY+fA8cWLR0nJ3jwOA1mzxai1iDMLf55TWNZrNKKYOTpIqupopUJ/CUpm2vP1bHs3B1Fn9TO7Qc8XwiUJ5jCAI8s7QJZ0yWWmQPUpYDGwmfp0zDgKo1fxWo+o1QDecQtGyrEl+tgG7VWXTJe0zMKWKzL7AJn6TVFukGoKvCC0uHBscRWjPdz2uK9Y2A98XxaJ+7qwehkLTCOapRhDkleLExQdTUNqDpRIp7cIHUyfyLqSXrxMSbQ/FzAuRssR4mo9mRNIiL00e9ypLN5p8azsIPFEcURsufbBOuiH58AJMDAvbFFHAc1bYdMleXQ1DYiqhSiMA8ANotYWhP173268wvFflbi0yDSJCvd2KFhc7XpcDxFWNOlnD1IluMPlUxC0WRiIMlSjUwpCHofAD8EKxWA7WtpUwgn85F07mhKUzAJyMRnmMIAjyDhNC0CL1DmnqYMqgaAFciKDb2W2uP2rslGvdtuFkrFTGttJpMcyAuXy9S5t1qAABU9gcIp+4XeJGItKAaWDK5crHR640DkJgCvG6IuhJ4mwGM1QjCPKaMDQopIXnkx97GYA9/0TehfTyWSYxdeh5IKHp7m8oDFdTPxu6CkOOGHKEob283iH7h6VEtNqccGHFoe5BxwMhuCIEE+B3ie9FLPL9Dl30FEURjtJNGJEVZ1yQWl1YMSorDDeavN7iTY812kQIUBWgvbQfMsyKUTBNNlKghk5Mg8mn4nAV5XaHb+8EwHm91at3rXPHEpqCGadRHiMIgiC/R4ggihrV2vLcL/+5s74mmJIfuzA0eZGQIcYScUFGImjy2o6y1iS1MzClgf6CZ5QrjRtNHrPURJJlHJa0MUM1giDIc42te17KpksqddipE9cXtomt8nbh+mRzjwQR5GKBiKLdHYi6JG6RTjuqV4PNbbrU1MOQF5ROMROdOxczrYOKyQtovaF2AqomNNsBRSNOgn74gc0jsOJMUehgMDaC8hhBEAR5frqd1t72w43Fnzc3FjvNVjpfNOJJqqgEKCG6BpAEQYC2ReATbw+2NKIH0H2RM8rKxu0OFySy4ixuUfQbIwiCPB/SS9npQtsjTVeE0TvXAn4Auw3R9kjahkQMXqIH+DnFsMer1aC8Rzb21FprXwnvd1MLDFXYGocw3NuJOi1SHFUpBU0DIFBrct/nmYRgPZ+wHWeECEMnai+/esKmLZ80mizgNJkQ6ZTQVDANYhZ0fP5RHiMIgiAvmcbuzuy3/1l6dN/3vHS+OP0vfy6cvWI7j5N1EKIylrDBUCBTFdsbZDEkfpYUXuSMsrKxaTGmkLhFFEwBgiAIgjwvzQ7Mr4Hri6lRPpqFN55oTeaRXlwXldBuR5rgrFdCTAw74uoYaIzvVtyYSS+c1VKpfWXUiGC5AZzTqbOxi2NwZlSLW8qgN7g4olId6qsEPDGUCIYSwlBxzTDKYwRBEORl43ud3d2V2tJKeflRp9HKFCZGz18rnL2SKYz39/nVh6xrYLvQbsCeC00LEr0Q62cegjyub9wVdpxZ1v6IwdCpil8GBEFOI7Tn17VjkE+JbHJ/G3kVeAFsVfe/R44NhfSzfZh0FbJJiOuQjBNDJ39Y6EvWLpZ50bpdXqkEAJDLqapG+/ml9/bCR4/c+RXRVLQuoxFhhk4yNjc1kUpQk7Caw3ouYpq0GefgRqLuU88HO6kzEwyT2fHfXYdqk6ZPFIUwyi2dx3Wh0OdcM6wq4MSBMmJbx7pfBOUxgiDIO0SzWtn88YfGw/Wg28kUJi7e+P8Vzv3mNz4KDlwGV3Pgz3zG3qrjKILJSSPbmzVnFCsrIghyOlEVSMRA1+DqhLhYBNvEjFmvhIiD6z/eeFZsc79roggMlenKH3+PZO3ifSmeZJVK8O//UQOA//7fUk5a6+eXruyG1VrYavBAcwOVesxM2PTKuHhvQkyM0Jhu5LMqZeA4qswyremQz5LSHiyW2FIF8llwXllbWQZMDBPDIGeGFCcp8PuL8hhBEORdR4DgEAngvbSXjAAxYnEjdj5dmCic+53f+DAaGGkYZqD4otslXgjBMwxfejPubldEETAGpk6sGM5aIwhympFppS2AoTRkE6iNX9l3jYswfLzxrH+rq6Cr8q+o/E4Fwb5o5AJcd/930ySBzyuVIOIimVT9gK9vyETnWr0RbW8HXEBlN6QqC36tOWyadKSgR4R3hFKPyF64/xiM5khxiDgJFjOJk1IGHxLDgEwKCCOz68RvQxCJo4KzCCGMMaYIQshzT9mk4mDFiB2nhoHPJMpjBEGQdx4OkS+6IQQKUZLOmeEPJtilfZmqGdYf+o3jIjUF1ytifYX+0hJVFZ4h/4mccY8iMTlpmDqxbSwmgSDIaZfHFEzt8Qby9jPoGfYDWNsM5Lrf6l7w7/9R8zx+bcZmjPzyS0t+EjmHdEbzA1HZ44oeZRwWt6jCSNZh+axSqcJGVVndYQ+3WSpOYjFN1QV9sZpblFFVV1/8OAjKYwRBEAQe1yuGTghdAmAIK6nn7YJz/KrFGhhpMQwEamJHAFeIakCMwbGEbsTB63IAyKYU9BsjCPL2EHFwPU4JMMJ0FQwVNEXQlxFzSgnIsrIUI1jfYP/2fMIy7ppzCCMRhiIKBaWg61TppYaWy3yCEBqt/f3iFgtC0WjubwehkNuex4NAaCox9H61YToxbviBiMWYqpCkzX6twkAzaSU3TFK7JJ4iWgx0FdJJMHRCKX/WtB1hBK5PWp39jf2TGtTQxXMch1EwdGrHIOlyQwcFv8MojxEEQd5xQuI3SbVLGyH4itDiwrHBUcQzV8CQPuQ21DukoRFDFVhYAkGQE0wQiFo9UhVQGLU0krC4ZQgFvXOnqH+r9UjOz/oBtNqi3Yo67UBTSX5I7SeJdJIsjESrLXqK9He9b8WVs+esiMOZUT2VZPms2vuRUUryWZVzYArRNXqg5rCpiZEMWAZkk4QSGM2K53uuXJ9s7pLNPeL5+0ezjOd8PuU6557bOaI0wtzXKI8RBEHedaIo7LpdjzfCwFWpopm6pprPcRzpQ46TZFNU97+4R4dYy9qPvi8UhQDZHx9oGsUgQwRB3jb51GhGpg66otgxEdOEpqA2PkmInlvYdaPNzcAEoRv7wjUKhaqAbTPpE262on3dG4LXFW4n6nQ4N0j0+xrRjBJDl85kwihIV7CqEFVh4+MGAGTSasKmgyuH4ej4KYWBbQpdBcsgvZxYf/BcKQziJoQRcbvQdPfVtQw9CENodkizQ4JIECJU5TmfT8YIY2S/qXp3raMmQ3mMIAjyrtPlYrfLww5vNrlJxVgEyRew5kKze8k1n+J/lrUfd3cD21bzeW1y0nCSDPNkIgjyViF9hkRATIdEjCiojU+cPOYQBLyy7d/qNDczfGjY1DTWaQcJm05PmVShrbao1fe7VVHA0EnMZKkkjZl0eEiN97zHMriaKVAcUWU6Lrnq+Inbz/atZMIyuNx4+p6GBsPpfRm81wTGYCTzSlKdSx+y3MAnB+UxgiDIuwrnEPrd+l6tXnK9XU+0jGwMCvxFDkmBHX/Rcm/eGkydGAb6jhEEeesMpB9AFIGqga4Ce3mqwdCgkH68gbwMHSx4FEU+514IHCBivg/NZrSzA55Luj6vN7lBIkWPrBiIXwOkpU9Y5rzQNBK3iNYrQG3oNGmzwa8S61UY/k1JHrH9bN9KAsd09ioU4gZ0fai1CSGQSQi7940NObQ88EORsuDFq2f/6kNGUB4jCIK8yyMKHgiv2ShXH81+6/p1Z7yYyo68hvM6jvrhB7YMrjYNitmqEQR5p3Di4oOpxxvYGi9hIiMSQTcMWt2o1hEGEaHdbNK5BXduGZpNXTfYUM4cSka6wWybFfJaxtnfIJQUR1RZdYlSUBihFN7aqvtBBI2OYPRxqSoA8HzY2hN+QK5O8mIeq2ejPEYQBEFeAEqIxgj47erGgss6uzsrPCYS8TMkpYL6Shy5j+sbe7zdiijbV8gmeowRBHkn6XmPX5OY4XxfWYUR6CoxdWAMnjWz8ZvicGbpXigyCcPfviOMQrMZtTvc7YqgG3QbHdKlnZapqWoUASFAGbFMOjKij6b3D5Sw6fCQ+mse6ef3/b5OCBEq5RQgCEg3gH52sDCClrvfm6k4Vs9GeYwgCIK8oM1lxNLZbqf66M4jwwyVoVji/JB2KU2LFjFfiS9X5ggtlbqLix3DoB9+YJsFTG2NIAjyapFex24ACQsyCdBOzoj7cGbpXmkl0m4G/e+IrtG5Bbfd4Y6jhqFot7nr8spuaCeUyUmDWnSjyyglY2fouYKQ+bSeY53wm4URYSphTAFGVQCMf0Z5jCAIgrwCKAFdpSrlYafD4yw2lk1OF+3RITOVZEJ9ua4F3+f92X23yyOOk9wIgiCvCc+HchWCEHJJGE4TQ3uLLLD8OgQhMIXQXg1ozn+rPNxxo/UN3/O4FWdckFpdMAa6xvrV8vmv+aUJIVShTFOJYRAChFFNo06StSNq9rJDJ22acU5qD6pM2KawTaI0gHPwPF5vCS+g9TbENK6roOL6JJTHCIIgyAtCCDACVtzO5hL2RFz7IG+fHcnaZ5Ik+9LrFTebUX9233HUfFa1YtRxVOwFBEGQV03bg6WtfQV14Qw5kxOxtylqR34dGk0es1RVowAQ+Lxfedh1+S+/tKIIzp2LmZbSz66ccShAnFGw4syK0ekp0/VJKJR2JBKjmqaI7BBzkvTUZGCWd90JibZH/F4t7k5HlBvU80U2HmYTwtKxUjHKYwRBEOTF4Bz8kBPNcs6Mps4l/CLTMnYCcgmReUq94uMjfQIAYNu9mX5PBIHofeMVJ8kwTzWCIMjrIQhFrSViOjEfl6d6fed1XRFxoTDCueh2OaMgE2L1f293eLvDPU/oppAzphEXnic4F1EEjIKh738srBi14owQYejEjrNeful96WsaVNNoJkO9gNTbTDXASCi6Kqy4IExU26TpikRM6IrQT7JUljmlDYNQClEk3C5vdWC5DIzBlTNQSIOJ+c9RHiMIgiAvSMhFw4s6YBq5yVgh3jE2QhHERMIG5yn1io+P9AkAwPSUqes0P6RGEQwPqUkb6xsjCIKcflxXrG0GXlfELRL40XY5MAwyPWWqGuv/DkI4jprNkmxGjfUKLHU6fCdBKd3/XmgqyWf3pbAVZ4pCw0gwSkyTUAJ/WKG36ZJf1sheXRTTYTYhEuap8q/6Iew2hGnQWIw5SYFfVZTHCIIgyIsScdENRQCKYqXUuMWVDQ5cBf2ZShYfE10jmbTaW/2FfmMEQZBTgvQPN1tRvR5QCpmMqjDSzynd9fnubtjuRFGG0d5H53BaKU0jmYxi6LQfVWSZj/3b8nvhpJ4sEP6wQq/XhdIueF1ycQTOZCF28hNBUhAa5T4RnBM/In5INEE0nRoGpvNAeYwgCIK89dg2m54y5QZj5A9n+hEEQZCThfQPLzx0791raCr505+S8Rjt55SmlHbaQafDU0madJR8VrFitF9tWAZXH64zLNfZvvj3IuTQ6QKPiB0/Jf5VhXCLhSEFnytdzlSVmNp+0yEojxEEQZC3lMHKxjIczjSoXDT1hzP9CIIgyEseWytgx0Q6QXJJUJVnK+l0uOZwGIooFKoCts3kCppqnTcaUa0Wrq93DYN4bhSP/SbXVAUSNtVUYsdpwmaDWSeeUm34ZX0vhIAg3N/QT4t/VVchmxBCkIbPuiEdTkMuKWJYHhHlMYIgCPLWMljZGADOnYsVCrqTZKiNEQRBXj+mJkYy4Adir0EiDpbxzPZ8sOZwuxV12kHCpjIyaG7Bbba4nVAyaTZaUGMmHc6rxaKRz6oyuJpRmJ4yI74vUHWNYPTQi/amSYojqlDJ2qP93rw8xot5cOIYWY3yGEEQBHn7kHmqmy3ebPN6kwehUBUcCSEIgrzRsTUD2xTpBMkmoRuA+utY+7BnmPODf+t5vC+PwxC8rmi3o0Yt6i05fnwoVSEJmxqa6l2MGQbNZlUnpQyuFs5kMPb3paEqRLVJ0iOqAiqIQgYKadTGKI8RBEGQtxKZp7pa46rOYjE2c81O2kwGV6PHAEEQ5K3isGfY9w8KrYj3JHQEMkjb0InCGADELMYUYvd8yNIzzLnoe4yxbV81hgbDadLbQG2M8hhBEAR56YMk0m2yXYvyuLBS3HnuiseysrHX5VShpslGRlSZWwVBEAR5U7ger1aDtTLZ2VO7ETNUIELoCjCARos3e0kipGc47C3TZQxUlTBKeumyCNMfT25qGolbvTrDJjN0omuPqw33T3RUfmnkVchj6TQ2sNYxymMEQRDk5Q+eaGtdm9fVM2fFWCEas6gFz+XrlZWNzZ5XIW4RBRcbIwiCvGmq1eCHH5vza6Lk2R7oK5tQKULSIqmYaLVFrS5+8wxbPek1UGBpEEpBWvV+5WFs2zeFXE8uN7A1UB4jCIIgLxkf3D2yOURicXEtLYYZPEPmFlnxst2J2q0wjIRpUMticoClokVHEAR5RXa7l+uh3eGeFykKSSZVRSHdLmcUbJtFfF8VA/wW6iwEBBH4XKiURyERnDJKDJ1YvSzT0jOsqU+Tx8hbpJd668mxHVAeIwiCIK+ESARe5Hcjl1CTsQQhz7ByTFa8XFnxHi22rRj505+SxSHjQAVLBEEQ5OUicz2srHhbW13LYtdm7HicbZcDwyDTU2bX5z/82ASADz+wHUf98AM7NUzIQ7XThcmcOD/CkxaROZCDUPQ9w4frDyMIgvIYQRDknUMIHnA/5AGAQslx6ydKv3G1HjVb3OvyKBKMUVOnVgx9DgiCIM+GzCMN0Fv324thlr90fdHtcs6BKUTXqAxpdl1Rb/IggK4vdnZD19vfJx5/cq4H06BmQfcpebRHOq4YG6YjOWHqROZAPk1tSAloijBUoqugKkAICPSwIiiPEQRBkNeD9Bs3W5wxMTmuXzir2XGWy2HOUgRBkGdG5pEGgH6VePnL7l6wXQ78QMQsNZNRiiP7NnZtM/B9USzqCZv2Qm1JYUgbGtLyWVUGV8c4+/ADezC4WqEQ04ESapqqqgvKTqFwVJiwDJ6wSCJGCNm/5SDCJwtBeYwgCIK8nsFcKBpN3u7wVJKkUiyfVdFvjCAI8oeW03VFL/8zEVw0m5EsjOR1+XYlkHHO0occcfC6+za23eFeV3TDSNVov568ppFsRk3a1PU4AGTSSjLBAH5zIJuF34UCEQqqAlyAolLKAEgEcNoUsvQexzSwY+D5pFIXXgARB0MDgtHiCMpjBEEQBEEQBHmrkHE3AFAcUQM/mltwPU/kh9Qogo2NrtyHc+hXxZMVATxPuN3HCk+uGZYbhs6mp0wAsG2soverllAgESNtD+4tUdcX3YCkbaHg5C2C8hhBEAR5VcO7XhXNnd2w0YgIIbqqGDplOPhAEAT51T/selG3l1PacVRGodmMpIh1vahU6kYc4jFCQLQ7PAhE9KQYYEbB0Ckk9jf9AFptYcgFw79bM0wGKw8/BZVB0iJ+SGI6aIqgp9ebygjoKgQheVQSfgjFLHcsgZUUEJTHCIIgyKtCVtHcKvtMYUN53bY0J8kw0ymCIEjfP1wqdctbbsJmH35g6xqdW3ABYHrK7HpRecv1PJFKkFRKdRxVVcnw0G9ZG4aH1KT92KI6SRZxJv3JL1htOKbDmdz+cXJJbhlCYac8aVU3gHINGPBUMRxKCENlADiJi6A8RhAEQY6BBmZaZNNiRAPz+H+lKsS2mZOidpxihUwEQU7hqJRB3IRGk+y1qc9J+veTgNJLHHGhMKIqvdXCkWg2o3qT+77o+mJ3Nwz8/W1dGzimQhI201SuKkTTSCajGDpN2o81MAAk7d9qDsscXS/HzquQTQCAsAyhKac/oXMYibYnNCoMhcd1oVAMPkdQHiMIgiDHIy6c89G18/xKXDjH2V9W0fR9oSjENCiueUMQ5FRiaDCcJtUmnVvRFAYzMToy8L/SS+x1RdwiiTh1kqzdjuYW3CCAYlHXVa1Z01SVKL2ZxP4KYZlTmkdgxZmm0cE6w3K98SuKxJFZneUG9iyCoDxGEARBjqTnPR55uvdYVuB0Pd5uRZTtK2QTPcYIgpxwOAffJ75PpOe2T6cTVSrBZoVvb0O1wipV1YwxwblKgRDh+7zvJT58TFWFpE2TNh0b02XZYU2jgyuED+SU7vMSfcWHkVmdsccRBOUxgiAI8hKQFThLpe7iYscw6Icf2EeN8BAEQU4KYUhabdZq0zD8nTStVIJ//4/a7Jy7U+UtbgS2k0/ptspNhTDCms3fvMSOo/SDqwe9xHKlMeaURhAE5TGCIMipQvpJ2h3udoXb5RFH/wOCIKeEjsvX1/1kMigM80RCBIGIem7kZiuq1qJqLWw3Q6GxhMGzSUgnwDaFyoTf+1vpJe5XYJLu30Ev8TFzSiMIgqA8RhAEOTFIP0m7wx1HdRw1n1WtGHUcFVsGQZCTTqMe/Hy/IaBx4UImnRbVeuR19/VxEJHieEw11EbdD0CjjlYcVoZz3EmKw15iBEEQlMcIgiDvChEHzxNBsD8odJKKk2SYpxpBkNNNzGLj46bjqPWq2vSZq7BEnMQtahjisJcYQRAE5TGCIMi7gq7T/JAaRb+rxokgCHIKSCTVycnElSuJZELpzQD+rs5wq8V2EnS7TsptFMMIgqA8RhAEeVeR641dj4eh4BysGDMMOliNE0EQ5BQQM+mZM9qZM6ppUsbIgdzRmkrCiLcjwro4LYggCMpjBEGQdxW53rhU8pvNwEkpV6/aThL9xgiCnLpxpyLiVhS3uIJFjxAEQXmMIAiCHCAIheuKepMHweNfKCOGQdBvjCDI6YNS0DShaYKihTt1PauroDOiqpQpQAhO7yIojxEEQZBnx3XF2mbg+6JY1M+d1cNQaBrBPNUIgiDICUJlkIiBqdFYTFN1QRlGByAojxEEQZBnJwhFo8kjLvJZNeMwVT24GA9BEARB3nJ0FfI2t0xIxqmhU0o5ACpkBOUxgiAI8nwiORDVekQpOEmG8hhBEAQ5WcRUPuYESUvkEqplUAW9xwjKYwRBEOSYeF60teXF7U4QdBVNMXSiaZiIC0EQBDmpqEwkzciJCUtnmoKfMwTlMYIgCHJsqjX/9p1qqbw7MREbG1OKRd1xFIURVQEUyQiCIAiCICiPEQRBTj+EEEoZYwohj9O2qiokbeokGTYOgiAIgiAIymMEQZB3BUqZphm5nH35cubs2cxIIZnJmLaN2hhBEARBEATlMYIgyLsljzVNyziOde7s0IULaceJGQbWcEIQBAEZX8MUpqrE0IiuAqYpPCG9JlTKdYVHGmgawYrWCMpjBEEQ5PjyOKbpY4rKM5kRx3FUFbUxgiDIrxaSEVVXYzGSCoihgaJg9uMTACPCVMJQj4Qq4hZVcFYDQXmMIAiCHBMBgvMoiiIhcNiHIAjye3lMiaHTpE0IFaoidJw/PAmoTNimUOl+Bxo6VVFbICiPEQRBkGMSBq1Wc9713MqOnc3qjuMwhguPEQRBeqNSJiyD6yokY0ApmBpOI54EeawSJ8kivv8tYxTrLyAojxEEQZBjw3nX93c9t91p1z3Pi6II2wRBEERCCWiK0BQAHRvjxMAYYRhQjZwmQ4RNgCAIgiAIgiAIgiAojxEEQRAEQRAEQRAE5TGCIAiCIAiCIAiC4NpjBEHeBmq12srKSr1e7yX5UE3TtCwrHo9blmWaJlY/QhDk3bGBuq7btm1ZlmEYZg+0gQiCnFaCIHBdt1arVSqVdrsNAKZpOo5j27ZhGLquq6r6mpOYojxGEOTNs7Ky8re//e3u3bsAkEgkisXi5OTk+fPnJyYmzpw5g0NDBEHeHRuYzWYvXbo0MTFRKBRGRkbQBiIIcopxXXd9ff3OnTtffvnl8vIyAIyMjNy4cWN6enpoaCibzb7+Gh8ojxEEefPU6/W7d+/+4x//6MvjtbW1zc3NcrncaDQKhQJ6URAEeUdsYDabrVQqq6urhUJhcnISbSCCIKeYIAgajcba2toPP/xw//59KY9d161UKmNjYyMjI4VCIZ1Ox+NxaQBfg1RGeYwgyNuF67pra2t7e3tzc3M///zzBx98MDU1hV4UBEHeEZrN5uzs7OrqqmEYxWIRbSCCIO8U1Wr11q1bKysrw8PDo6Oj/YjCkZGR1+NJRnmMIMjbRdCj0WhIE9ntdkulUqFQGB8fn5qaGh0dTaVSlmW9/rUoCIIgr4Fuj52dHQDY3d3tdrvb29tjY2Pj4+PVanV0dDSXy1mWhQ2FIMipxO2xt7dXLpc3NzdXV1c3Nzf39vbGx8dzv/JKbSDKYwRB3l4GvShDQ0PT09NXrlyZmZmZmJh4/WtREARB3ogN3N7eXl5eHh4ezuVyV65c+eyzz1AeIwhyugmCoFqtdjqdcrm8vr6+vLw8NDSUTqcvXbr0qm3gm5HHnudtbW11Oh3XdROJBD4BCPKOMz8/32w2D/8+6EUplUq7Per1eqlUGhkZkYPFkzVMpJRqmlav19fX1wFgc3PTMAx8ABAEbeBTbGCr1Wo0GltbW/F4fHd3l1JaLpdt206lUo7jmKaJ1g9BkBNKo9FYX1/f2tryPG/w96iH53n1HrVabXV1NR6PVyoVSmmlUnEcJ5VK2bat6/ppkMe1Wm15ebndbmOSCQRBAGBra6tUKj19H7kmuVarzc3NFQqFc+fOvf/++yfOi6IoimVZq6ur33333bfffqtpGvrAEQR5ug2UXpRWq6Uoyvb29tzc3Pj4+KVLl65du3bjxo2TIo/R+iEI8kT75rru1tZWrVY7vg2cmpq6cePGzMzMxYsXT7w81nU9m80WCoV2u41mEUGQ48M5932/0Wh0Oh1CyOjo6Em8C8uyJiYmfN8vl8utVgu7FUGQP0R6UYIgUFU1DMNmsymEGBsbQ+uHIMi7YwOjKDJN03Xdzc1Ny7IOeJtPsDy2bfvixYujo6M3btzwfR/7G0EQALh9+3a5XN7a2nrKPqqqOo6TTCbj8fiVK1f+5//8n9evX8/lcifrTh3HuXHjxsWLFz3PC8MQux5BkOewgVevXv3000+vX7/uOA5aPwRBTi4yuFrawN3d3afsaZpmsViUq+ouXbok7Ylt2ydeHus9stksPg0IgvRpt9tPMXCmaTqOk8/n5WJjmZjh+vXrExMTJ+5OZeVS7HEEQY5jAxljqqqaphmPxzOZzKANvHz58sjICFo/BEFONLu7u6ZplkqlJyYjGLSBo6OjV65cGR0dlaEo58+ff0WKEjNXIwjytiN9DpcuXRobG5OjQwm2DIIgpxvpMR4dHT1//vzExATaQARB3lkbeOXKlffff390dFQmMnh1sTMojxEEeRuRHmPLshRFmZyc/C//5b9cuXJlaGgom806joP5ThEEOd1YlpXL5fL5fDabHR8fv3z58uTkJNpABEHeWRt46dKlixcvvoYYZJTHCIK8jUiP8eTkZDweLxaLV69eHR0dNQxD13VMd48gyKknl8v9+c9/vnTpUjqdzufzxWIxnU6jDUQQ5J21gblc7lWsNEZ5/HbRbrcrlUoURalUyrIsVVVfJJu3PJp8nmSpG9d1q9VqrVZrNpucc9M0U6nUiasTi7wLMMYMw7BtW1EU27ZzuZxM2S/lcTabPXPmDNZIR56OLA7huq7neTIc60UWOna73WazGQSBoii6rvfLEMqz1Gq1SqXSbrcBIJlMjo+Pp1Ip7ALkBZFRM7KS5/T09Keffjo9PW1ZViKRQI8xgiCnElkRXdM0Sqn0GKdSKdM0p6am3pQNRHn8JqlUKl999ZXneTMzMxMTE47jvIg8lkcDgH4l2Gq1euvWrbt3787Ozvq+XywWZ2ZmTlydWORdoL+2JB6PX7hw4bPPPrt06VI/uFqKE2wl5Om4rru+vr65uVkqlRKJxAvWg202m7/88kuj0ehP0Eh5LM9y586dL7/8cnl5GQCuXbv217/+FeUx8uLIqBlZyXN8fFy6ShRFUXtg+yAIcvqQC4nleE96jGdmZs6cOVMoFN6UDUR5/MaGcdVq9cGDB999910URUNDQ8PDw1EUPetxZCHEdrtdq9Xu3bv37bffxmKx69evy//tdrs7OzsrKytzc3PdbtfzvFwu57outj/ytpFKpWZmZmTBkgsXLnz66acnMSs18qaQHt3V1dXbt2+vrKy02+2xsbHntnXSb/zw4cNbt241Go1isSijcvrnajQaW1tbDx8+nJubA4BEItHpdLAXkBchmUxeu3ZtZGSkL4+xxsdJR47Qms3mzs4OY6xQKLxIXOhRR2s2m6VSqVaryVAX27ZTqdQLxs4gyOtEVdVEIjE2Nvbhhx/6vv/pp59KefwGYwZRHr8ZpF/322+/vXPnTjKZfO5hXBAE1Wp1eXn5zp07t2/f/uGHHyYmJvpHk4Yyn89PTk56npfP5+XcDLY/8rZRKBT+7d/+rd1u94OrsU2Q4yM9urdv3/7f//t/VyqVycnJsbGx5z6a9BvfunXr//2//yersw4PDx8o0yqrL8rtYrGII1HkBRkfH//rX//q+34/uBrb5KQjR2jz8/Nff/21YRj/9m//9iLdetTRSqXS3//+9zt37lSr1Xg8funSpWvXrr1g7AyCvGZ57DjOzMxMPp/nnPeDq9/gJaFSejnIWT3XdVutlu/7v7WvohiGIWv9yagAuUL4wYMH33777Q8//LC6ulooFMrlcqlUCsPQcRzbtnVd73swXNcNw5BzPjgs688LttttqY2/+eab+/fvr62tmaa5srIyMjJiGEan09F1PZPJFAoFz/Oka+6J8lh6s6WuppQqimKaZv9KjtpT07R4PE4p9TzP9/3+dcrf5S0PhosPrgwcHGse3n/wLPJ+5ZzC4XmE41/tYLsNMtjOlFLDMGQThWHoeR7n/MDxD+8/2L+HOdyPL94+h5+3o1rpqLt+27B7oCV5KQOy/lMkn15N0wzD4Jz3n5anvzVHPY2HrdlRb9nLfQ4Hj3/4raeU9v3G33zzzU8//eS6biwWq1Qqq6urg+c9jpXmnPf9xv/85z8fPHigqurY2NjIyIjjOEIIx3HkKik58yjf6Fwu98RmPHzlh9vhxfvr6e0zuP+gLVIU5YnW+yibc5S1PMr6Hee+Blvj8P5H3e/hfnx6+xz/6/z0b8HT34IXJ9UDLdhpGhNWKpWFhYUff/zxP//zP3O53H/9r//1Raz61tbW/Pz83bt3b968OTw83Gw2+2/lzs7O5uZmtVq1bTubzTYajSAIsBeQkwLrMdzjLbkklMcvc45wc3Pz4cOHe3t7/d/j8XihUBgZGemvW5MrhL/77rs7d+6srq42m03bth8+fKhpWjqdLhaLFy9elB946cHY3NxsNpuDH3UZfCWHFLVaTWrjn3/+eX19XY6Tbt26FQRBoVCQI5tUKpVMJnVdl9r7ifJY/tXm5qYcGNm2PTIy0r+So/ZMp9Pnz5/XNK1cLu/t7fWvU/4uB5SDQ6vBlYGtVqv/++H9B88i7xcA+r8McvyrHWy3QQbbWdO0QqEQj8cBoNVqlUol3/cPHP/w/oP9e5jD/fji7XP4eTuqlY66a+S0MvgUyac3nU4PDQ35vt9/Wp7+1hz1NB62Zke9ZS/3ORw8/uG3XtO0vt/4p59+qlQqiqKUy+XZ2VkAaDQa/fMex0r7vt/3Gz948KBarSYSia2tLbm9vb1948YN27Yty0r2kKL0qKicw1d+uB1evL+e3j6D+w/aong8/kTrfZTNOcpaHmX9jnNfg61xeP+j7vdwPz69fY7/dX76t+DpbwGCHH7qFhYWvvzyyx9++OHhw4eKojy3ZO1Hx/yf//N/FhYWoigaVBHyXZYnNU1zZGSkUChgFjcEQXn8tswRzs3Nzc7OyvTRkmQyOTY2Nj09reu6pmmqqkpBe/PmzbW1tUajIT/J8/Pzruum02nXdUdHR23b7nswFhcXDwxcxsfHVVV1XTeVSlUqldnZ2fv376+vr8uj1Wq1u3fvdrvdCxcuOI4ThmG1Wt3b2/M8T37m+25JqaWbzabneUtLS998883a2lp/qDE+Pu55XrFYNAzDsqz+rLw8/oMHD6TPZG9vT9f11dXVnZ2d/nXK36enp6empnK5nKqqnPO+h2dhYeGA/BvcP51OA8DW1tbt27flur5isSiEAID+FQ5immY6nb58+fLo6OjgSi3ZL4PHmZ6eloUxDvhDZPs/ePBgb29Pruc5LI/PnTunKEq32wWAtbW1fr/IIdTU1BTnfGxs7EBu23a73Wq1VlZW5HrIP2yfjY2N+/fvLywsrK6u1uv1J7aP3F+uflxdXf3hhx9kmxzVSvJp8X0fM5a/I8iIkgcPHjx69KjdbktnwtjYWLfb7Vsn+Y7Lp1q67uXbPZjLYGNjY2FhYWVlpf+2Sms2Pj5erVZHR0dzuZxhGAfespf7HMo1dQ8ePDhwtMHrt217eXl5YWFhdnZWJsoyDEPeZqvV4pwXi8VkMgkAe3t7T7TSUvDItziKovv379+5c2dubk4KTlVVNzY2fN9fXV3tdrtTU1OWZfm+L1tpd3cXAIaGhvoDX+mBbLfbnuctLi5+++23Kysr/XPJ9pmcnJR52h3HcV13c3PzwYMH8/Pzcrb0+P01GIs0eJaj9pdfn9nZ2WazmUwmn2i9+/0Vj8fT6fSBjBj9p2t7e1vTtLNnz8pg4MMrxI5zX4PfslqtNjs7Ozc3t7q66rruU+5XsrKy8oftc8DCP6XfpfUebB/TNMfGxuLxeBAE9Xr9wFsw+E1/kYSayMli8O0+HOFlWZZpmpTSQb/xDz/8MDs7W61Wc7nc4uJiNps9MKaST7XMgT+IzN+radpgdMzNmze3trYcx9nd3V1cXEwkEvF4XMYJyvGePLjMAHyc65fREIPX03/T+1dlGIaMa6jVanIkKZG/H666IseW7Xb7QEzK4f0Hr+eoGJPHWuWI6xz8Zslrk+12+CszeEeHIx9ltMhR/TLYv0dNih3uxxdvn8GRpGyNo/riqLtGUB6/FXOEd+7c6Yve/gd4eXm5Xq8nk8lYLCYHQ2s9+rFwzWZzdnZ2a2tLmsIbN24MrnxbXFw88AqtrKzU6/XV1dWZmZlSqbS4uHj4aDJPTDqdbrfbpVJpYWFB5uWSxx/0OczPz5fL5bW1tcXFxWq12g9UGxoaWl5enpycHB4enpiYGPSdzs7O3rx5UyakWV5eppRubW3V6/X+dcrf5eBSURTHcXzf73t4ZmdnD3xaBvefnJwEgNXV1bt37965cwcA5BgdAPpXOEgsFjtwX4P9Mnicbrd7/fr1QqFwwB/SarUePnz4448/VioV3/efGFwth91bW1sAsLS01O8XaVIvXbok2+1Abtvl5WWZvOf+/fvlcvkP2+f+/fvffvvtgwcPDkwfHN4/DEPZs/fu3Zufn39KK8mnZXt7GzOWvyP0n+eFhYVaraYoSjKZHB4e5pz3rZN8x+VTHYZh/+0ezGVw//79ubm5crncf1ulNRseHs7lcleuXPnss8+Gh4cPvGUv9zmUa+q+++67A0cbvP58Pr+9vV0qlfpjBXkXnU6nXC6rqnr9+nUZ7720tPREKy0HXvIt1jTt3r17Dx8+7L+A0mJvb2/LDOpyPrFcLi8sLNy+fVteAyGkH+gorffy8vLW1tbi4uKdO3fK5XL/XLJ9isXi0NDQhQsXbty4EQRBqVSan5+/ffu2PMvx+2swFmnwLEftL23+d999F4ah7M3D1lvS6XT6KR4H8TxPRnguLS1RSmu1mqZpU1NTT9zzD+9r8FvWbrdv3rx5586dUqnU6XSecr+zs7N7e3vys/X09pHs7Ox8/fXX//znP5/S79J6D7ZPLBZbXl5WVbU/iTz4Fgx+01EevzsMvt2HI7wmJibOnDmjadoBv3G1WpWC+auvvtre3j4wppK/y6d3kImJic8++yydTh+IjpH2bW5u7v/+3/+7tbV1/vx5IcT29rbUZtJpnMlkDuRKOOr64/H4geuRDF7V8PDwzMwMANy5c0eOgiTy98NVV+R7tLy83G63ByXu4f0Hr8f3fcuy0ul0oVAYjDHpj6WfeJ2D3yx5bbLdDn9lBu+oH/lYKpX29vba7bamaU/pl8H+PUoeH+7HF2+fwZGkbI2j+uKou0ZQHr8xpDf15s2bt27dWl1dJYQkk8n+irhut7u+vk4plU6/y5cvyyDqdDotzZwckKXT6eHh4aGhoWQy2e12pRdxcXFRCCH/UFoBOUNWqVRu374t56h83yeEyBlueTTDMEZGRsbHx4eGhhhjpVJpa2trc3PT87woivb29vpvo5xTl5ddq9WiKJIrwcIwLJVKS0tLnU5na2tL6tVisSh9s3KVy8bGBgDs7u7WajXbtmVF5TAMm81mpVKRv/u+r2ma53mXL1+Oomh2dvaXX36RHtrh4WFpgOQsWqvVmp+fl/vXarVCoVCv18vlcv8su7u7qVQqkUhI3/KgZ2lnZ8d1XTkvm8lk+ivZZL/cunVLmhXHcY6aT/V9f29vb2trq1wuy1btV54cHh52XXdlZaXb7VqWJf1XtVrNdV25irvdbm9ubna7XbkiUc59qqoqW0n6/+WfJ5PJ/srGbrf78OHDJ7bP7u6u9GDLoaqcWx3cPwiCmZkZ6YhrtVrb29uylWq1WqfTSSaTiUTCNM1+vWv551JFaJqG2SxPPf3neXNzU/o2k8kk5zyZTEp/b6lU2t3dDYJAPtWU0lgspmmaaZoyKmFtbe3777+/ffv22tqaEKJQKKTT6bDH+vr61tZWPB73ff/ixYu2bbfb7aPeVsuyms1mrVaTAqPb7XLO5cz3H85zy5n4n3766dtvv713717fOvXf7pWVlTAMs9ns+Pi4XLZk9AiCQK4Nlvc7MjJimqa0FX3PZK7HYAXj9fV1+RZns9lOpzNoK+QaXZkqSeZ0CMOw1WrJRpZ3vbOzI9+yvn/1p59+Wlpa2tjYkG90LpejlFar1Xq9fufOnZWVFZk57OLFi4qitFqt3d3d/tEG+0suopY2R/aXvCS5PnZ+fl4u0pFOpL4vpVarlUqlwf2lT1V6dJeWluSXolarSU/FoHWS/bW0tHTv3j3Z4PIg0hatra0tLS1Jh3MymZS1KJ84XpStdNR9yedwY2MjDMN6D8ZYpVKR1m9vb69vdQevX3p3f/rppyiKCCGZTEb2o3weDu8fhmGlUvnll18WFxfr9Xp//8P9DgCc83K5vLKy0m+fRqMh20fTtFgsJmckd3d3pcaOx+NRFM3MzLw9i+WQV+03lt906c84II+lveWcZzIZORCa7VGtVuXMnXx6G43GhQsXNE0bHx8ftHILCwsHzrixsRGLxUZHR+XjPTc315deURRtbW39+OOPclRgmma5XN7c3Nzb24vFYrquj46O9uXxoE1YWlp6ojxeXV3d3t6enJzsZ8OWMUH37t2TkkyGsx2Wx/V6vdFo9N+CwZosc3Nzh+Xf4f37MSatVusp8thxnANj0X6/7OzsLC4u3rx5U7ZPvV5/4tSebH85jSv767A8lmtzpDd7dna23y+D/TsYJyifCmkzZ2dn//GPfxyQx/V6vVqtXr58eXh4WA7b/rB9Bvfvt8+PP/4oW/6oviiVSsf8tiIoj18TpVLpiy+++Prrr3d2dhKJxMzMzHvvvddfEffzzz9//fXXDx8+lMPWfD4vK3oxxr788kspxgqFwueff/7+++/n8/lYLBZF0eLi4r179zqdzp///OdisdhPdSNnyB49erS7u/vzzz9LzTM1NRWLxW7duiUdyDL75UcffWTb9sbGRr1eX1xcfOKVSx/I6urq1taWqqrnzp27cuXKjRs3Wq3W3//+9wcPHkjHkXwhB6M4Dszb5fP5v/zlL8Visd1uz83Nffnll2tra9VqVbpMpStJ07SbN29ubGxMTU199tln/ZIVm5ubt27dknMBcv9Hjx59/vnnh88yNjb2v/7X/3rvvff6v8/Pz//973//+eef2+324uLiN998Qwjpr2ST/fKPf/yjVCrJSpKffPLJ2bNnHcf5wwVjg5UnNzY2/va3vy0sLNy6dUu2uZy9GxsbsyxreXn5iy++2NjYkL0pZ0bkFEOr1apUKktLS0EQfPzxx5cvX+6vbPz555+ld+5A+2xtbWWz2ZmZmXPnzslO39nZObB/uVzO5/NHZeUdHx//y1/+Yppmv951vV5fW1vTdT2ZTAZBgOuQ30EMwxgaGprpUa/X+zlO+3kKUqmUYRhnzpyJoqjdbm9vby8vL8uYlMnJyc8//3xiYqLdbj98+PDrr79eX19XFCWVSm1ubmaz2QOr6Qbf1uHh4V9++eXOnTu3bt2qVCpra2uEEDlM+cN5bjkT//XXX8/NzUVR1LdOIyMjcmTwt7/9bW1t7eHDh4yx9957b2xsbHl5udFoVKtVTdP6Nd6vXr1aKBRKpdLNmzflzh9//PHMzMzQ0JBcmdyvYFypVL788svz58+///77qVSqXC7LyYVBa3Du3Dk5qH3KlUv/6tLS0sLCguu6mUzm8uXLn332maZpfVsnhzVjY2Oe50lZe1R/ua771VdfSY+37C959tHRUSlWV1dXM5nMv/7rv8qxo+d51Wr17t27X3zxhVwGIveXPtXDPZVKpT755JP33nuvb51kf0n7Wa/X8/m8HMhKP8/t27elIdJ1fXp6+n/8j//x0UcfHTPP/OHnUH5ldnd35+bmhoeHJycn//Vf//XcuXOrq6t9qzt4/dKTXyqVCoXCpUuXPvnkE5k8XD4Ph/dvtVpfffXVvXv3XNe9cuVKf//D/S5jE9rtdj8Oa7B9zp8/37f2MpBh0BqjPH53/MYypk+G4x0Iru7LVzn8k8+P9BvLfeTTK3OwNZtNOXfz1VdfySytpVLpwBl3dnZqtdrExMTo6OjgsrjBoyWTyevXrxNC5NhpZ2dHVnWSxx98kqVNuHfv3uHg6rm5ue+//9627atXr37++ecXLlyQY7PFxUW5ZEaK88MBvfL3tbW1/lvQj7/45ptv5AzmgcjHA/tLn8r29vbS0pKMMTkquDqTyTxxLCrHsY8ePXr48KEc68pViod7ULaYvBHZXweCq/sLanK53NbW1sLCQr9fBvt3ME5QPhXSZi4sLBwIrpb3KwW2jH4/TvsM7i9jcB49erSwsCCv7ai+6C+uRB8yyuO3yGjOz8/LNDBysdb4+Pj09HQ8HpfpTD3P297edhzHMAzGmG3b09PT5XL5+++/l0ewbfvChQsffvhhPp+Pomh9fV3X9Ww2SwiZnp6enJzse3RbrZY0u/V6vdVqxWIxQohMySCVW79+7McffwwAQognaiE5Zzk3N7eysiL9Htls9tq1a5988omUx/JN++mnn6RyHh4eXltby2QyB3ILS2/k5OTkv/zLv0xPT7fbbcdxyuWynCGTK8TkOurh4WG5/G9sbOzs2bPnz5+X3mbpqo2iyHXdnZ2dSqUSi8X6YYqSWCxWKBSuXr36pz/96f333+//blnW999/Pz8/Lz9Ca2trm5ubg0Hm8z3kJMLo6KicaHh6vgpd123bPn/+/EcfffTxxx9fvHjxwYMHX3zxhUypKv1XyWRyZmZGBgLINW/tHplMZnd3V05DynVB8/Pzm5ubyWRSzmJMTEx4nienAznnx2kfTdNWVlZk+8j9c7ncgfYZfIo++uijTz75pJ9yVkoa13VlgH2hULh8+TK+s+8Oqqqapjk6Oiq15eG323Vd+c7KTKdyhkvmMoiiaHR09Pr163/605+mpqbk2725uSmnt+W6dzm7d9TbWiwW8/m8qqrlcrnVasnV9XLp1JUrV2T43+DgQMZfyNL/fbtaqVTkkjC5TFRWwyaELC4uPnr0yHGcYrE4OTnZbDbT6bRpmo1GQ9f1XC53/vz5GzduyNBZ6QbM5XJDQ0Nnz569cuWKVHSlUqkfsy3fYtu2Mz1isZj8XTbg5cuXb9y4MTo6Kvd8in9J+lelfyaVSl2+fPnTHtLJwxiT0Tc7Oztyz0wm07dajDFVVXO53KVLl2R/+b4vm6jZbErXt2maa2trmqYRQizLmpyclLHuUh7XarWlpSX5relbrbW1tcnJyb5/+4D1lic6c+aMjMxsNBoPHjyQ6akGs+NKw7WysiJ707KsTCZz7ty549QnP3xf5XL57t27y8vL0rcmFyePj49/9NFHFy9edBxn0OrK8bTs9FKpVK/Xs9msZVkyErLfv0fdr8yKPzo6evXqVdmDh/vdsiw5Du6LmcH2uXr1qnwC19bWfN/vr38+yhojp4nBPCYym32z2SwUCqlUqp83RK4OXV9f///Y+7Pftq6sTRymOM/zTJEUJZISZWuyIqcdB0aS6kJ1A/ZFo/u2L/sva6Av68ZVLxDkrdcpVzkuS7EtWdFAkTQncZ7nmfzw6cG7sX88Ik3L8pjzXAQOdXh49rTO3ms961lnZ2dSqdTlcimVSoPBoNVqiaq8VCp1Op1ut9vpdCqVylqtBo50Nps1Go1gxtGVjXO5XLPZLBQKo9EIvBu9Xo9zr0AgUKvVFovF7Xaj5lyn04EUAlycxG6Q+Cri3r/99ptAIIAQgEKhEAgEzWYzlUplMpler9dut30+n0qlEovFeDy4CHk8HjhBUPxqtVoImYIbIhQKNzc3hUKhWq3GqTKZTIpEIrvdTnJosecsl8vQLNjc3MSCGg6HRMcBugCEu+dwOIhiQjQaZe5FIXVG6+8ghVCr1QqFwkvDOdVqlbSIqHwjVEN0LhDBxl4OvQSmSTKZ7PV6Go0GfCKoLZDaMU+fPi2Xy06nE/YQUfRyuRwMBnu9Hryxq6urtVotFApN7x9cz+PxNjY2EGiBfhCeXCKRjEYjtVo9Pz+Px0bsOhaLjUYjLpfL8gTZ4/GnCOIJi0Qier2ey+XeuHEDrjhiznCum6R0KpFI5ufnVSqVx+Op1+vtdjuXy+EwHIvFjo6OTk5OENURi8X1C1yaYTIddGQGO4OlpaVvvvlmZ2cH6tb3799XKpWZTOa3336Lx+MKhWJhYUEsFo/FH+iorNVqxZOUy2WBQECi2ahmtrq6ajQa6/V6r9cbjUb5fB6nfaQxQwpl0tMiun7v3r2xjfj7gEKh8Pl8t2/f3tnZAXeU2d5vvvkGIalLnxme2tevXz958mRvb69UKslkMkSS4enMZrO0Z3d6//j9fqKgO/3JUTcYvcTn83d2dgQCQaVS6XQ68XgcucqQgmDX6e8HqM27sbFx7949pODSq5umZhHL8OjRo8ePH2Pr//3333/77bder9disYytbkQXk8nkGNeDXq0KhWJlZaXdbkciEZwbCS05l8splcoxaplQKARZl/AvYN8we3O5XCQSwemFx+P9z//5P7lcrkAgkMlkarX67OyM7Ab4fP6YjcXqgNkcjUbYjSETmGT00d8lBN0rxJcQX4WVBlnm3r17IFfv7OyMRqNoNFoulwnnBfnbtDWgrfFwOPzuu+8Q2YjFYuS3kAunVCrb7Xav1xsMBog5JBIJZPDm8/npT0tbs/n5eYlE0ul05Be4tO14u9G2663cNGPtKhQK5LcwS2/cuLG1tcW0umRmRiKRUCiUy+UINycSiWBziaoNzG8ZDIbvvvsO7ozBYNDpdNBLzHGf3j8ul4vP58/Nzf3pT3+SyWSE+cXi9wDmO91utz948GB9fZ3ohiA7FJlxWq3W6/UuLi7evXt3NBpdyuzr9/uIedZqNafT+fXXX8NxA/YHKhsHg0HYPWSHeTwesVh8cnJSr9dRIfb+/ftbW1vILJtyFkJ8FWRArMTl5eW7d+9CvTUWixE+EbkSx7mx9YtfNJvN8AIgf6RUKkEh4vz8fHNzczAYwLYTnRcEJCKRCGHB4PpqtXr//n1m3j7N1sHBFdwQbH3pvahcLh8bF5PJ9P3338PezmKR0A/g4p2dnRHGDTLUPB7PH/7wB51ON8Y0GQwGXq8X3wJXKJ1Ot9ttjC8Sg8GLBIMP70poQ4jFYlRDmN4/hLnDlDyk+4fD4Yzxs1ieIHs8/rTiM8oLwHWNfIZSqaTX67VarV6vR+YeCei1221EHS/Ng0XkBPUbY7HY2dkZXNrNZrNYLCJJGH5BeN1oH+HsgMcrFAoVCgXojiIIA+IiTonpdFqhUCClOZfLIct3LP6AuAqistjTNJtNp9MZjUZJNBs5gVqtVqFQQMsRGTLVahXeVghiTWkF4qJer5cZu0YWd6lUwoYJfoRKpdJut/G0iFooFArk5r1xy4u4ve0CdH4L3V7kdWNjeqmneTAYkGg2ehvJbMjYTKVSdJT7jf2TSCRmcYKQXiKPireFSqVKp9PvMltYfNbWSaVSmUwmp9PJXN2TLAMoZAsLCy6Xa2lpidBG6NWN6CK4D5NWq+gCFovFYDBgHoLJhpx5KPCNCRlaLBaHwwF5PHJ86nQ6uVwuFAohfohjj8/nQygDFWgRWyBR6DEbi/gh9pqxWCwQCMATn0wmx9YXvot/vG1vI0qTSqVyuRzahQgnia/iHAgdBMJ5oZckj8dD8IS2xjKZLBaLjfHlUL4F1i8WixFN5mKxGA6HaY2JKa4T2pqRLGulUqlQKJDFjRZlMhlElbFTR7ojbO+lURommO2ix4jL5SIh2WKxMK3u2MxsNBrgT4G/gKzmcDgMf8cY4CWZZdyn9w9SXbrd7tLSEhwQrG35/WAwGODUSt7pZJ8ABxY8dIRKgO2QQCCw2+02m+1SZl8+nwflYWFhQSaTYZoRsX3IX4M11mw2UdYEnGGRSASKhNls3tra2t7ehvNoSspYrVbzXwB67Dabzefzff3112traxqNJhAIgJUNEoff75fL5YQ7Q+zn8vIyuGk2m+38/FylUtVqtXq93mw2wfSRSqV4WdhsttFotLi4aDKZEOPt9/vFYpE8Ia4HM4Uu9I3dmtlsvnXr1t27d8nxmMPhPHz4kLkXlUgkY+MCNo3L5Xoju5juh8XFRXhdESiGysPy8vLi4uJXX31lNpsDgUC1WkVuc6PRcDgcsEKtVgv5cYlEAkNjMBjg5hgMBgjC83g8eBXBeNdoNDabzWg0er1ek8mEgWb2D2HuMI/HmEX37t3D8dhoNMLfh+A2nsfpdF7qLmTBHo8/QnzGbrfH43GoOsdiMbFYrFKpcDYG94/UkJzlnqTG3V//+ld4uwUCAfIxsMl4Yyzx0/S/Qsvx6OgIdUT6/T5iNei9ty0JiFp/yWQS1FAImaZSKS6Xm8lkTk9Pa7UafIRms9nhcJhMpo9SCRDZJsfHx3w+fzgcttvtRqMxdqh4H/3DgsWnCcQo0uk0xITJ50ajcW1tDRoN9OpOp9PxeLxYLPr9fpzHLBbLrVu3PB4PqUA7y++CNfP8+XO/349jklQq1el0iMmM+f4+C8C2PH/+/MWLF3CkwsVgMpkymQxdHG6m3cCFtYFXF28ZxMOVSqXJZEqlUvF4HDEopVLpvgAzcfqDvU263S7O7YSa+PsZdxYfF6gOgI2ZVCqFcwfRXVJi51J/DX3gXFlZsdvt7Xa7UChEo1FIRkErJBwOE73r67ISiK/q9Xq32726ugo/nUAgoC1tPp/f29sbDodjGRN6vf7uBfR6PRiOcPpD0BQSX/h8ZWXF4/FAthCLC8zzaDSKIpStVmuSTw27NafTuba2NolFcl1Aq0k/jO2o1Wr15gXo0/ulu3RUCkS70M+4FSjfRL6HtM7r9RqNRgQ/4vF4oVCoVCpj/TP9yTFzSP+srKxks9n32lfs8ZjFFQFfTr/f9/v9iUSiXC4jiwOL4fz8PBaLIR64eoExtxzzxU/XuAP7y2AwIDWFx+MhGQMuxs/I80rXAIxEInNzc3w+XyqVIj+nXq/DmfpWd0acKp1Ov3r1CplgiUQimUx2u13k/dZqNeLPs1qter2ergfwcdbbf8bECHkJzmP4U+n+QR4m4i2NRgPxZHa5sfgCAK5Ho9EIBAK0cGC5XFar1S6Xq9vtqlQqr9dbq9Wq1apUKoVTCZWHORxOJpPpdDqQaELdWuROT/pF5JXt7+8/efLk6OgIdYzEYrFcLvd6vY1G4/z8/I1s5E8KYCpVKpVnz57t7+9Ho9FmsymXy2EVsXmlSz3NAjCh4NU9Pz8ncSGZTNbv90lgvNPpTGL0fCwgSox/I+OdVBYgysAHBweFQgEVrRC9+RzHncWHBxheoOEg+xfrArrKcEg5nU6Px6O+ANTOp98TzBqZTIbg5+HhIShmvV4PVME3sj/e1lYgvgreh8FgUCqVOMzTTBDETi0Wy9ghTSqVOi4glUrBcNRqtRqNhmbkwXpotVqBQFAoFF6/fh0KhUjBgkqlgmqj0/tZIpGQfiafgyljtVrhFENuDrZDiUQCpCSoxoCpNElLn8ZYP4xxvMVisfkCUwIqJJOZSNmjWgr6uV6vZzIZWEt6FuHZWq0WavVDSaHb7YL5OMtogrlA+gf/+9F3tuzxmMUlQHbT4uJiIBA4Ojoi+Rh07U2kpkDkeZLmMO2RomvcIQPQ5/NptVoI07daLT6ff43W872CjouiBqBAINjY2HC73SaTSSAQVKvVs7MzaPG97fGYeM7wK+l0OhaLVSqV3377DZVLoZVqt9uR5vFGo/k+gFwRl8sll8uZflO73W4wGFCZ85///CfpH4fDQcrQRyKRK/QPCxafL7C6pVKpWq2GIFMkEiGxPpqnA/EtGM9Jd0O229OnT6GD7XQ6bTabxWIxm80WiyUSiXx2dNlisbi3t3d+fv7LL7/kcjmVSuXz+dxu9/Lyss/nu1qL4NIF4ykWiyEuFAwGUbevUqnk83m82sYiGB8YeE6Xy7W5uclUjVYqlYQrRCsDo7DT/Py82+12OByf6biz+FjzzeFwrK+vw/LgvziToHq2yWRaXl6+ceMGqVs7y51pHWMiao2tAgQIPzueIK3vHQqFUD/cYrGAZQnZvMFgcIUdVKvV2tvbg0VCkBkVWw4ODlBe1Ofz3bp1y+fzQUnho7SdzIp+v99utzudzpjTlt7hn5yctNttLpeLZBaLxYLqoSxPkD0efyGA3xqENJ1Op1Qq7XY7aGnQu4O2VrlcVqlU3377LeSpJ90NuV50jTuPx7OxsXH79m2sHGiWXiEvjgYzN+96r6dBosekBiDEG3d2dux2e7/fR9bEFYRw4H+FTjgcdYg1DQaDbDaLQprIUfT5fKi0+bFmCHZyl9ajk8lkYrEYW3+6fzY2NrRaLZfLhYjlFfqHBYt3wfvLV4fD22w2Ey1oQK/X+3w+VCrG6kaRD5vNlkqlHA4HHElEyxRF0bhc7s2bN2Uy2RSWLByLv/76a6lUQtHj1dVVr9cLAcXrWl/vYiffFvV6PZVKhUKhcDjc7/dNJpPD4dje3r5x4wZKNF1hg4jy0bCZxWIRNfMrlUo2m0Vp0Fqthqr1y8vLTqdzUp7wBziuqFQqtHdpaWnsr0TgTSQSkXGPx+MSicRoNK6urm5tbc3PzwuFwmq1OmPuNIvfM7AukOuLBRIOh0n1eGzn8L/gyqKu7/R70vWBX7x4kUgkBoOBxWJxuVw+nw8swuoFPpdeQjQVigC0vrfdbl9aWhKLxcVicW5uLpvNvjGMPAZS7eX09LRYLCYSCaPRCAXso6Mj5FSrVCqr1UorKXx4ICINPXOm5Njq6urc3BzNDC0UCnDRwpeHbBGocrBgj8dfDmjFaWTtl0oloneHkpWY+lardYpziNZrpe8M5UwUdrqGUWcou77x+imKpjOaTlIDkI7ovnuL6DrSY1uoMa3Uj7bGLnrPYDAsLCwwU1mgp5rNZjOZDN0/qNfSbDbL5fLV9MlZsHgXMG3RdQGxx6Wlpdu3b9MhXxDk1Go1Wa2wfjqdzuPxbG9vg6pHtExfvXpFnnM0Gk15Ttr+8Pl8hUJhNBoXFhbkcnk6nb4uRfd3t5Ozg1ajxc+hr949coJYDRS2O52OQCAgZPh+vw97O4s27IexqB6Ph+mkIOXBUOkUGX0QoYAOCCiy7Bpn8bbrwuPx1Gq1w8NDovaM4zFmWrlcJnV9oZk0CXTcOJFI8Hi8Gzdu3L9/f21tTaFQhEKhaDRKJ558+kBcNBaL/f3vf9/f3yf63tvb21artVar7e3tFYvFK9hG7IhwhoQlz+fzxWKRy+WS+vzQj4QS2MfqAVRtgJ45MxgzNzfH4/FoZqjNZnvw4MHW1pbRaKzVai9fvqzX62wghD0efzlAVlu5XCYbLyzjer0ukUjy+fwbNytEdRkpdpfGlrvdLtSn8vl8pVKZcsCGD69UKvH5/El3g15iIpGIRqPQkkWdXsg7D4fDcrlMcndRMRJKzteY4UCKc8RiMej+XXmrTepIFwqFRCIB0WaS/k1rwH4AQH0RFUELhUIul0O512QyaTKZ5HI5keZC9UI4Gpkjhdgd6haEw2G2jgiLDwCSz59KpZC/ajKZtFotAsjQjMFqRaIXsk/f9qSB6DEKtk/ZbKFuZK1WgyShTCZzOBzwpkH0ddJ36Vqa1Wp1igVGDiHiD7PcTaFQTLkbrXsPMgvur9Pp1Go1l8ut1+ukXgBKU9rtdrqw07W8j+LxOKz31e4AJXCi3NPr9fB2gMQOWM2zaMNeL/DOGgwG5+fn5XIZsbVcLoc0Y0jawkhCGEmtVjMDOGPj/i69xOJ3t1e+cMqIRCLUpC0UCiqVqlQqVSoVUns2cQGdTler1aYfj5EPHA6Hcag2mUyITm9ubuIEeI28Brq6CpMTRH+C3Qtycd+WAo2dJ1SmkH+rVqvX19fv3Lmj0Wji8fjx8fHVnh92FTzBSqVCeIIcDqdQKEDHm1Rn+GC0anCFUJMf53bYqM3NTavVKpVKIbVNavtXq9Xz8/NUKhUIBMAMRf/cvXsX/QOxNxbs8fjLAbLa4EeUy+U+n29hYcFisczNzeVyOcRecE4zmUxGo5Gpkoc4ADJPsOTo+AN8cmKxOJVKHR0doTDaFPU/XB8Oh6H6cGmKMh0fQAVOv9//008/pdNpt9vd7Xb3L5BKpa4rowM9gLajvYFAQCgUxmKxGev6TnlpkUg4PLI8Hq9er/N4vI8SMSb6hKRCJmrloYEWi4VkUaJ6odvtvn379vvrHxYsZgf83yqVilSAxJEDFZLD4TCZjSQyMKkA+DsCaxl1I7vdLtH/12q1Y3VusW2luTB0nWTEVOn1RVenLxaLb6x/S18/Nzc3RSUbv1WtVm/dutXpdFDxklQEFQqFwWDw8PBwrNq8Tqfb29t72/5BRAUCMAh9w/JnMhlivd99FEisJpPJoF7dx6LhGAwG5OP4/X5oaM/NzaE2mNvtrlQqjx49IgXJvv/++42NDbywSF0JVCgNh8NqtbrZbP7444/X1Ussfg/A+sKpbG5u7g9/+MP333+PUjqk9uy7V9MhPJdrFJehVwGzejn9CbGTAoHgbY/HU3Z9YKA0LnDlqun0ngrBFTz8LFXT35O7ZEznn24jxhH+AgwlCPPvo39YsMfjTxTIrS0UCpFIZDQalcvlWCxmsViEQmG9Xo/FYsPh0Gg0ut1uZMDKZLJut6tQKDQaDfx5qDXH5XIhxKVUKuEna7VauVwO2VPpdLpYLCaTyU6nIxQKO50OauIh6gjFP3iwcP1wOMS29dLavIgPLF0Ams+DweDo6KhUKkH5eX9/P5lMCgQCm822vLyMAipXy+iAXh/y2ZrNZjAY7Pf70M1C9T/8ulAoHI1GvV4PyoQImM/uGdVdQCAQ5HI5hNY1Go1KpZq9Mud1gW5vpVKJRCLIEsQjGQyGVCqFuD10yN+2f5CdzqxBzYLFuwP+7FardXJygoIT4XB4b28PzjucLhqNBur67Ozs3Lx5873qiCDGksvlkskkilvieIw6t4gTIg9WKBRaLBZkKNB1ksE9FovFN2/ebDabkObK5XI45Fer1UQi0el0pFIpj8drtVqwPyKRCBrOkJAg1eyr1apcLp8UlgQZBGLaxWKx2Wxms9nT01PkEOJ4nEwmB4OBXq83GAywwFKp9ApuR2huS6XSWq2Wy+Xa7XYsFoOtQxVTmUyG578aSR55dCqVCnJcpVIJBfk1Go1UKv0w+dU0EJkRCoUejwc1WiuVCsSooeD9+PFjvEC1Wi1pKepKNJvNubk5JFEfHx93u912ux0Oh2u1mlgsnpubI+M++3uHxe8NqLwdi8VarZZarfb5fCaTCbalVCqpVCqBQAC23SSeHamLzufzm82mSCRCIAR7uUqlkslkpFJpPB4/PT0F7+zSMyp9PWqbTeIJAoRhB9pFMpkMhUJOpxNRzXA4HAqFkskkOEHLF1AoFJduHacAoZ3BYECiqVhTOJP7/f5oNEorOb/VnorH44GRh6rjpVIJQt+NRgNKBNOrpr8PXKrz7/f74UCECBlUG+bm5lAKW3yBa+8fFuzx+BMFKsLJ5fLd3d3Xr18jY0QsFqPY93A4HI1Gbrf7hx9+2NnZ0ev18BhZrdalpSV4wREniUajYNf8t//237a2tnCaIrFHtVotFoulUqnL5dJqtdBwItl0dGwE10cikbW1NXC8p2ywECtGkbpoNJpKpfx+P8jVKPLudru9Xq/FYrmyrhWeDRliKpWKw+FEIpGDC+AZEFSHtlapVCIVBWeUuWf2J/braN3HqswJz6LFYllbW+v3+ycnJxgXkUiE/bpKpbpx48Yf//jHtbU1vV6PV9Es/QNlSLww2NXH4tpf+USjFYGRer1OZiOHwxmNRpA/WV9fdzgc768GOxguYPCicBGsE06ACA4jD/abb77Z2toaDAbI0h+rk5xMJlUq1fLy8p/+9Cen0/nTTz8dHR0RKw3PmtFoxA3j8Tjsj16vh3AgUamFSnY6nfZ6vUy1ZBrQIPV6vRwOJxgMhkIhv9+fyWRArpZKpU6n02w2a7VaWKerKZ/h6/gVun+0Wi3O5xg+iKOmL/BW+126GiqGGFGahYUFo9E4u27F+5gVQqEwnU5ns9lCoZDJZPx+P8jV9HxA6Bh1JaD4gLM0rler1ZeO++zvHRa/N2CGoDitVCq9deuW0+lEakkgEMD6AtvO5/NdGsPEuxv15AaDgUqlws4KkcZoNHp4eJhIJI6Pj/1+/5S6x/T1QqHQZDJNjzbDSVSpVHDbk5MT3NZutyNl5pdffgkGg7VabXFx8e7du9vb23q9fgqhZsqeRyKRkGgqdraZTGbGusezA3fGP6aXSn3f70pa5x97PLghpFLp8fFxNBrN5/NqtXpnZ8fr9SJg88b+Ydcaezz+QoBsPaRqQIKY7BchoanX62/cuHHr1i3k6dHqoEjlwvXwx8MjZbPZ4JtHSSTiy8dShN8RAUno1shkstXVVYVCkc/n6SRVVG9DEd1ut6vX61G5jn5yuVxeq9Wi0SiXyyVMM9Q7uXXrFuLGdEYHvPiFQoHD4TgvQPtK6SwRnKiXlpZ0Op3BYMB2qlKpiMXifD6P8lQWi2V9fV0sFsfj8VQqhVRtlF8ij4o7MGvT0Z5FZPHZbDZSP3n2ypyo6beysqLX62Uymd1up2POdHvpipqI+sKjiX0n/Zw8Hg/1RbrdrlQqVSgUqVQK7hK5XK7RaBYWFrYvAOVVoq9D+gfjaDAYNjc34VFG/6jVajiP4S4lyjSonzy2TTebzYuLi5hLCwsL0+v4sfgyMMu407OacEOQBk80WulVSbI64Sy32+0+n89sNoMLM2kVMJ+HaYUmAQwXnIT5fD5tnbC64XT79ttvcRbqdrs3btxot9vVapXJc5HL5Xa7XalUNptNsVjs9/vRdpVKtb6+rtPpGo0Gjpdzc3MCgUAqld68edNsNhNLS4Np5ejVhyqdOMtptVo+n0/uABtONtZWq1WhULRaLa1Wa7fbkedyaf8wrZDBYHA4HIPBYKx/YPcsFgsSf0QiUbPZ1Gg0eL9MsmaT3muoKk/aZbPZnE6n0WicvbIo2lWr1aCMRebh9Fk66Tmx0VepVPDnknHEq9bj8XzzzTfb29vz8/OYAwKBAONSLpeFQuGUce/3+7gSDuhL+wdv5+lWl8WXCkRHwRDMZDLD4TAajSoUim63m06n8/k85OJ2dnbW19fVarVQKIS1NBgMCAWD2ddut61WK2gOy8vLGxsbo9EIKvEvXrxQqVTQdkbdCoQTsS/SaDQSiQSVh3u9Hq4fDAarq6vlcnnK8Zisphs3bnS7XWi+oAQpthy5XE4gEDgcjrW1ta2tLa/XewVOEJ/PVyqVUqkUdjgYDHY6naOjo0QiMRwOW60WWHLlchnZzoh+9/v9GeOlNE8wHo8TnqBWq31jjeL3AbwrtVqty+UqFov9fj8cDqNjQbQJBoMIHUPTB08+HA7H+icej3e7XXgN0D8Q98W4i0QiNp7MHo8/73iLy+XicDg2m21Mtl4sFmu1WqvV6vF46Lq7eM1rtVpyPa602+16vR45bIgqYNODv8Latlotl8sF3x7OYGKx2Ol0VqvVYrGITSpK1ctkMjDrstnscDgk9ycboJWVFafT2e/3M5kMuSeAAzy8m2SrSpSiIatjMpnW1tYcDgc5gtLVSjOZDKnrS8eQl5eXi8XicDgEN9LhcPB4POzC8Tl2NgsLC7D4NpvN6/VOr1pMyriT+sCzV+ZE/N/hcEBW0WKxoOYws710RU06H+bmzZvM54R6odFo9Hg8m5ubUI5Bi6RSqdFotNvtRMia2T8YR4yCRCIh/QOiDlRbTSYTmSHoZ6bPGHOMw+GM/SKLLxWzjDs9qy0WC4m24a+I0S0sLJBZh9lIWyGDwYCpPn0VjD0P0wpNB54EQu60dcJxCEFas9mMQlBjlgegn4euUY9jEtaXTqfr9/u5XC4QCNRqNaVSaTAY5ufnO53O2O9ilU2ycuQYSdS2PR7PpXYVtEyJRAKHqdvtFgqFCwsL/X7/0v6ZZIWEQuFY/+D+eJhcLudyucrlskQigU0TiUSX3ufS4zGpKk/bWFQMnqWGPOLPaFer1cL+m8zD6bN0UnvFYvHKyordbicCDRhHgDmTyd3eOO5Ewg3aSJf2D/O9zLS6LL5su4p/B4PBaDQaCAT4fP5wOGy323K53Ol0bm5u/vDDD6urqxqNhsvlrqystNvtSCQCwTxEC7PZ7Nra2ubmptfrtdlsKpXKbrcTrQcU9VhfX9dqtZlM5tGjR6lUChyQnZ0drVZrMBjq9TqJPUL3Aeqe0/eosNJWqxUpHpBlxbomVdPX1tZgLa/ACYLWoMPhIDy4169fh0Ih1D22Wq2bm5uNRmN/fz8WiyFeenh4aDabZ5SuwOrzeDzLy8uFQoHwBDEuGxsbH2VvQ1u5w8NDRI+JAo5Wq93Y2NjZ2blz547L5RKLxXhIun8g6Iv+6fV6cFsQniDSwtnVxx6PP0uQvAgul2swGMgRiN7GKZXKMal3+PO0Wi25nlypUCiQw+ZwOLhcLmKh9H263a5SqaQ/R6gT8qr03ejP8aO4P4mBkKgvKjbTRfaUSiXxwTNfEnAHMK+hq5XibkqlEiKiiJnz+XybzUY/D1I1jEYjLWNAQ6lUTnENDgaDXq9Xq9WgIQn9QJKROMsuHHESs9lMPxXZYNHtpStq0nE2RIDHnhPhZRxlLRYL3TpyHxJ5YPYPrkQPC4XCsf7BDh4ubbqfmT5jzDHSLjbW8cVjlnGfvorhDtNqtWOz7lJrNn0VTHqeGQVU8CRyuXzMOl36JGOWh2k96Br1ZNWQtkOcn/4c/xi7m1qtnmTlyGYUgQ4c799oV0ejkdVqxfZ6Uv9MskLwXdL9Q98fShbkc8yBS+/D7Hk6YxnBEERNCTtglrEj7SJWi7Rr+iyd1F4AVh2phrO8s2Yf97Ht/lj/MN/LTKvL4su2q2S/FI/H6TmDmu3r6+urq6ukUoZIJIJGg1AoJNdDdwCxRK1Wi/1epVI5Ozsj94G8K2RHIDiH9DGJRCIUCu12OyQ8sXFSqVQSicTn8ymVSiIQSzP+6L0KMtGgmE3mMBE+BIlDKBR2u11cjyWJDRLNCcLqRi1fiIzCK4c9DI/H63a7BoMBEtzk/ti74vin1Wrlcjnh2pD7TGL8YfVhN2UwGLLZLF0lZBYeBzgj+Df6E8xHbNXo9i4sLIBtBCcgumh7e1un03E4nPX1dZxvx6ycRqMh40V+ZWdnZ2NjA8XkMCXG+gevCZfLtbm5SX4L/YORJf2Dmv/0rzPbhfnzIRW8v2DMvW3+PYsphzRaxh2gazDS3GDm9cwroXQHDxn9VzBV6M/BTxsOh2N3oz8H+4X5JAD9W8QYEZ7zpCvf9hq61fTz4CvM3pv+JEC73YYb9f/9v/8HzQaTyYTKnIgJv9XYMXtp0igw/zrpOWcZ60lX4p5cLvfSz6FdNOnXp7eLxe/BFk0a9+udt9Pv9u7zkGmdZnmSSdZj0tMyP6fXF/Nub2sJJz3P247X7FZoeosm9SEQiUR+/vnnx48f//Of/yyXy3a7/b/8l//yP/7H/9jc3FQoFLNU+Jtkzaa/C2axum/7zpp93Glcbbaz+LLtKgq8QbGPnjN0zXb6ZILkdmhi4Xr6SqFQCDnVVCqFICH+CvUmSAzCfwRyNWKSuBW0uJifIxrJfBLM20ajgRrmY3NYJpNB/5/sN+inQhk/tVpN1gJdeA8eNLVabbFYUNCoVqvl8/lardbr9bhcLrk/dG1AHka0WSQS9ft9KNTS95nkPw0EAv/2b//2+PHjFy9eQEj/22+//T//5/+g/Mf0d0q5XI5Go5VKhfSbWq1GqeSx9pLicMRmlstlMhYqlcrpdMIvRtsxqCTS6Y30r8BmknAO6R+SA4IbTukfsKzpX2e2a9I8ZMEej1n8joCK09Boefny5cOHD0OhkEAgWF9f/1//6399++23LpfrY5UhYcGCBYvPC9hAYyN4dHT0888/o46gRCLZ2Ni4d+/egwcPlpeX2Y5iwYLFB7ZLKNQSCAT+4z/+Y29v7+TkhM/nb25u3rt37/79+xApZMHiGsGSq1l8rkBN0ZcvX0KrNpVKIcdGr9drtdqPpa3KggULFp8jUNn14ODg0aNHR0dHJOoFVS1W2I8FCxYfyy4lk0mkYROd7dXV1fv379+7dw9KhCxYsMdjFiw4pIpgIpHI5XLNZlN5ARSIt1qts2irsmDBggULADX8S6VSPB5Pp9Mg8iG1D/mQrHIBCxYsPjCg+H1ycpJKpSKRSDAYrFQqENK/ceMGGzdmwR6PWbD4/4Cu24xicdAjXV5eHtMJZ8GCBQsWb9gNUNXakdsmFAoVCsXS0tKdO3fcbveMgmosWLBgcV1APeFnz5612+1Go1Gr1VCD3W63s+m1LNjjMQsW46DrNkMZVavVut1uq9U6phPOggULFiymA4LbqNZOVKNRdIQor7JgwYLFRzir8PnyC5hMJp1Oh5LRrG48i/cHVpqLxeeKTqcDAUmijCoUClEqgNVnZsGCBYu3AlRVW61WvV6HwxHqzajPPItaNQsWLFhcL2Kx2JMnT+LxOPkEVeUdDofBYGAzPliwx2MWLFiwYMGCBQsWLFj8LlCtVs/Pz2epcM6CBXs8ZsGCBQsWLFiwYMGCxReLt61wzoIFezz+yAAVDWv1uqi8zHvSpoE1Cp/+fECBeKTKCC7A0rxZsGDBggULFixYsPgswEpzXR29Xq9UKnE4HI1Gc11HIOY9UfMNxBJQStjj8ac8H6rVaqPR4HA4MplMqVRe49xgwYIFCxYsWLBgwYIFezz+FA9CrVYrl8uFw2GBQLC6uvruOsmtVqv0n5BIJOILkJpvkCWwWCzD4dDhcLAx5E8QtVrt7OwsHo8Xi0Uul2s0Gu12u1AoZDW0WXwZAD9iMBhwOBwej3ctzAgI7EGIHuJPsK6dTodI7olEIlYa6pOdD51Op91ugzLDjhQLFu9uA9+HlcavdDoddp1+FvMBI8Xn88VisUgkYnmI7PH48wAiuicnJ7u7uyqVymg0GgyGd7xnqVTa29uLx+O9Xs9qtRqNRo1GQ2q+PX/+nMPheDweqImyMeRPEPl8/smTJ7/++ms+nxcKhQsLC1999ZXRaNTpdGznsPgCAH4E8nHEYvG1MCNqtdrp6SmHw1lZWcF2DdY1n88T/WS9Xk/+yuJTmw/5fD6TydTrdXakWLC4Fhv4Pqw0fiWfz7Pr9LOYDxgpFLLS6/UsD5E9Hn8GG4JWqxWLxV5eYH9/3263w/P3LoftUql0fHz87NmzZDKpUCjEYjH2hSxYsGDx0YGIRC6XCwQCzWZTrVYbDAa5XP4uzAj4yIPB4N7enlAoNJvNqKzbarWSyWQwGEwmk61WSyAQuN1um83G1t39BN+G1Wo1Ho+/evUqm81yOJylpSUyjixYsJhl7xcOhw8ODiQSyTtaOdpK1+t1uVxuMBikUqlAIOj1eul0+uXLl+FwmMPhuFwutVotl8vZmOQnCDBGQ6EQHBmrq6scDkcqlbI8RPZ4/Kmbs/Pz85cvX/71r389PDwsl8tyuZyW1LsCEDd+9uzZL7/80mq1fD4f/VeDwfD999+73W6Qq7e2tubn5yUSCTsWnxr0ev3du3cdDgdNrmbL1rP4Mg5CpVIpEAg8evSo3W5vbm5KpVLwaa8M+Mj39vb+9re/6fX627dv4/N2u51KpY6Pjw8PD2u1Gkg0Ozs77Ch8auj3+41GIxaL/etf/woEAhgmMo4sWLCYZe93eHgYCARsNts7WjnaSlerVbfb7fP5jEajSCQqlUqvX79+8uTJ4eEhh8NZW1tzOp06nY6NSX6ys2Jvb4/D4TidTsSQjUajSqViO4c9Hn8c1Gq1VCpVLpdJ5gagUCgsFotCoeBwOPDA/fLLL/v7+5FIBJ+8ePFCKBRqNBq1Wk0yOsrlcjQarVQq5D7IA1Gr1eRudNx4b28vGAxCfysYDGo0mmq1arFY2u02EvCmH9pLpVK5XB7LLVGr1RqNBmdpeBbL5XIqlSKJLhaLhcPhkE/o9qrV6ks9i8x2ASqVyul04jRIP89wOJRcQCwWw3y3Wi3yLeZzEkPfarXK5XIul4PSFX1/GpPahecfDodT7sPsN0AikYyNJvN5mL3XbrfRuuFwCNlqoVDI5XInPeekWdFoNHK5XLlcbrVaXC6XPAMzd4jZbzTo+8CDw2zXu1zP4veDcrn86tWr3d3dvb09Ho9nMpnsdvuVj8d03Phf//rX8fGx2+0mZkEgECiVSq1Wq9FouFyuSqWSy+V8Pvuq+uTA4/HEYrFCoVAqlbCo7EixYDEL6L3f8fFxsViUSCTvyBmkrfRgMJDJZLSV5vP5crmcXaefwamMGimlUgk+KevFYI/HHxOpVOovf/nLwcEBydwAvF7vgwcPvF4vh8M5Ozv78ccfd3d3c7kc/hqNRv/v//2/z58/39nZ2djYIBkd+PzVq1fkPsgD2djYuH//Pg5CdNw4GAzi1HRycpJKpXZ3dzc2Nh48eNDr9X788cejo6Mpuce4z6tXr05OTkhuic/nW19f39nZwfEJR9NXr149fPjw7OyMtIvD4ZBP6Paur69f6llktgtYX1//3//7f2NJ08/T7XbtdrvNZjObzZVKZW9vL5lMkm8xn5O8PM7Pzw8ODh49egQ3BH1/pseU2S48f7fbnXIfZr8BVqt1bDSZz8PsPbqwk16vX1tb63a7JIec+ZyTZkUul/v555+hxyYUCn0+HwhX+Xx+yvgyQd8HyufMdr3L9Sx+V7bx4cOHT548yefzFoulVCrV6/UrH4/puPHx8TG0+gk0Gs3Ozo7dbt/c3KzX6wKBgOXrfpqQSCTz8/N8Pl+lUhUKBQ6HYzab4TRkwYLFFNB7v1QqdS18QNpKm0wmwmoUCAQajQYcXWxcDQbD6uqqRqNhJWw+QVgslgcPHoBKoNFoFhcXzWYzNocs2OPxhwOONLVaLZ/PP3/+HOQTiKaSmGE6nVapVJVKRaPRnJ+fR6PRdDpNTE+z2YxEInw+32QyYXPQ7/dTqdSzZ88ODg6Oj49pn5BIJKrX60ajcTAY6PX6SqWSyWQSF8Cxh8fj1ev1TqdTKBTkcnmhUOj1eqFQCMdjWMBqtYr4NolG+v3+vb29o6OjUChULBY5HI5Wqy0Wi4gELi8vWywWgUDQaDQQ/YbQF9olEAgODg6CwWC/38fpDp8PBoPV1VWz2QwDSnppf3//t99+I89DeqlWq62srAiFQr1eXyqV4vH48fHx7u5urVYjx+Nms3l0dIQsNXxLr9d3Oh2lUgnbTR+Pk8nkycnJs2fPgsGgQCAQi8VQf2GOYLvdptvVaDR2dna8Xu9gMOh0Ovl8PhQK7e3tnZyc4CuVSgW+W7/ff3BwsL+/HwgE0G/D4bDf7ycSie4F4GFVKBQky+758+doO917r1+/HqtQbbFYpFKpzWZrt9sYqXQ6jQ4h19Ozotlszs/P83g8tVqdy+VOTk729vZCoRCPx8vlciaTSSAQVCqVQCCQyWRarZZKpcrlcq1WSyaTwYYSJwiJch8dHT1+/Pj4+LhQKOAww2wXl8sl15+enu7u7sI7U6lULr2ePSR/SZjEnkB9dSQYD4fDVCr1/AJYQQKBIBgMymSydru9tLREuBiT+BHkKKXRaOAIx2pF3BjOMmRbzaKnSqsl12q1MTaKTCYzGAwKhWKs6jjNecGv8Hi8drvd6XQIV2gSI4PJqgDo64VC4Vjb0V7sg+knBJicEbp1k7gwtLOS5jpxuVwyXjAIdHtpDgi+WC6XL30q5m/N0m/tdhuEminv1lqtNsbJYjJTJnGg8Mz0/GQ+J+xeq9XCw1z79SzeN+hVNhwOBQKBSCQSi8WDwYCMDlaNQqHQ6/UKheKNo4nrZTIZphmp98FcZZNW69vyp6bbQMwriUTSbrfBnQFnsF6vazQaWMXhcPjGdTGjlY7FYhqNZjgculyuNzqtyC6uXC43Go0xqyiTycRiMWiA6MN3Wa1gD81ihZrN5piVI9vRMWtM23+1Wj1W24WeXRhW2lrSPTAL07Df7xPrLZVKr93Ktdvtcrn8tm/tSauDacOZYzSJbYpgT61WI31yXdezx+PPAIjpnZ2dPXnyBCefTqdjMpnkcjnOUfF4HD65s7OznZ2dfr+PGV8qlbAMJBKJ/QLgxpRKpWAw+PDhQ7/fPxwOl5eXyW/V6/VMJoPDVSwWu3v3rkQiwbwhRBf4+cAnxD0nJTbT0UhEvDudDniPOOaFQqHz83MShV5aWhq7FdqlUql4PN7S0lK9Xq9UKqVSCZ9nMhlMa/LaQC+dnp7yeDy6XaSX/vKXv2Sz2bt37/J4PPJbrVYrHo/n83m/3y+VSmUyGb6LbyF7rdFojEWikH+YSqXa7Tb65Gqezn6/X7/A2P3huz04ODg5OalWq4YLcDicbrcLvdyDgwNEUPv9/srKytzc3KWeWvQe3SI0nG4XRur4+PjPf/6z3+8f6z3MCr/f/9NPPxWLxY2NjVKpVCwW8czNZvPk5CSZTGo0GrFYbDAYhEJhPB6v1WonJyf4LbgDYHToKPfR0VEulxuNRjqdzmQyXdouoVBIrn/9+nWv1xuNRhaLRa/XX3o9ezz+8iIYTPaEUqm02+0bGxvfffddt9v9y1/+8vjx41QqRX8rEAgoFIqdnR3CxZjEjwDARFheXjaZTOFw+MmTJ3t7eyRuDE7K06dPCVcik8m8evUKTslbt26R6gC0WjLcgjQbZWFh4fvvv19eXh6rOk5zXsC8EIvFmUwml8sRrtAkRgaTVQHQ12u12rG2o70cDmfsCQEmZ4Rp1ZlcGPqoRnOdhEIhGS9s+Oj20hwQDodzenp6cHBw6VMxf2uWfqMLyWxtbRmNRrPZPPZu9fv9Y5wsJjNlEgcKjCp6fjKfE3YvmUymUqlIJHLt17N436BXWbfb1Wg0BoPBZDK1220yOlg1y8vLd+/e9Xq9bxxNXO9yuTY3N71eL+HcMVfZpNX6tvyp6TYQ88pqtWYymYODAzijcdwqlUr7+/uZTGZ5efmN6+JtrfTa2tqDBw8Gg8Gf//xnHJ6xrLC7o/vw7Oxsf38/HA6PWcWFhQWLxWK1WkkfvstqBXtoFivEtHLIv8Vei7bGtP3f2NgY41fSs4vD4YxZS7JTnZFpCHcGrLfD4bh2K0dbVLvdfvfu3a2trTfOh0mrg2nDmWM0iW1arVYbjYbf7yd9cl3Xs8fjzwAkuoiwXqlUMhqNN2/edDgcHA6nUCj4/f5MJtNut7PZbLFYlMlkOGm0L8DhcNRq9cbGBgKtarW60+nkcrlUKtXr9W7cuEG2dNC7zuVy6XQarnewJrRard1uhyHo9XoikchgMNjtdofDsby8bDabu93upW9oxFeRu3J6eiqRSMxm8/LyMsoI4cnT6XQ0GkWQRyKR4MxP0O12i8WiXC6/efOmUqksFouJRCIYDGazWcQtnU6nTCZDVedqtZrJZKLRaKPR8Hq9MFKkXcVisVQqHRwcwPep0+mIUxAupcFgIJFIDAYDecJYLAYPYi6XQ/ychH16vV6hUAiFQpFIpNFoKBSK5QtcSi8h+W8ajUZKjDj4AAA3WklEQVSpVI45I4fDIeKfw+EQngi9Xi8SidB7+XxeKpW6XC6tVotFCxnJWCyWSCQQm4WmLnwEzIO3Vqslo5xIJHq9Xrfb7fV69O9ijqXT6UKhwOPxJs2KFy9eDAYDuVyOO+O7+LdUKlUqlTabTavVVqtVkUiE3kNcHcwFuVwukUgajUYkEjk4OHj69GkkEpFIJA6HY2NjQ61Wk/HFWxDTrNfrkeuLxeLy8rLD4cDj5XK5WCzm9/uPj49JP7Ac1y8pbgzrsb+/H41Gq9UqnHT9fl8sFp+fn0Ohem5uLhAIQESanvl4HdL8CCimHhwc+P1+mh/R7/fb7TaYCJVKZXV1NZvNEkYDcYclk0mBQKDVasEHiUajBwcH5XIZZS2azSaJmp6dnYVCoVgsdnZ2dnBwkM/nSZQpFovxeLxSqWSxWBwOB6keX6lUXr169fjxY7yqc7mcSqVqNpvVarVUKlUqlVqtplAoxhgZw+EQLIyjoyOwKkBgwcJUKpUIR8hkMqfTSbRhwWGxWq2tVkskEvn9fnojgsjJ3NycRqPp9/vr6+tms5mOyYxZdaFQeGlsFv2wt7eXyWTEYnGxWFSpVORKur1LS0tarRYV5lqtFkJMlz4VIap4PB6DwQDGyli/KRSK6gVqtRrKHXc6HRLJkclktVqNng+vXr06PDwMhUK1Wg2plfR8oJkppO27u7uJRILD4ZhMplqtJhKJwuFwPp8nUWvmc4J9gDFKJBJkPiOwM+l6v99fLBaTyWQ4HEa4ZlI/sIfkDxA9xpvI7/c3Gg2NRmMymRwOx3A4DIfDmUyGrJpcLicUCgeDwRtHn8/n83g8nKMgg+JwOCQSCZNxRq/WbDZLZrXBYGi1WrPwpwjT8Ozs7OnTp48fP45Go2O6IWCcLS0tdbvddDpNnOCDC5RKpX6/L5VK3W43jCrNcXv16hWZpeQ4lEwmZ7HSarUa4ZOTkxO0F+93/AqeLRaLHR4egkyXTCYHg0H/ArBysVjMYrH4fD65XK5UKmlLdelqxXghPkmvJrVa3e/3Y7HYjFaIMOkymQyPx7Pb7Xq9HuH3MWuM8cpkMmaz2WAw6HQ6PCfGJZFIHB0dgZGHN4LL5RoLFzEZgjCkY/vt3d1dvJWEQuHXX38tkUiu0crRVgi9Vy6XrVar3W7vXKBWq4XDYYzU2Hzgcrk8Hg+0Ux6P53K5dDrdpTZcKBQmEgkwBNFvYCN2Oh0woRQKBXat8Xg8FosdHBzA80LYQzgv0O/KbrdLrj89PUVQ59J3Jc12ZI/HnzTa7XYmk4nFYul0ulQq9Xo9g8Hw3XffbW9v45AZCASi0WgqleLxeEqlUiKRuN3uubm5TCaD0Aeu//rrr1GNCTVvt7a2RCLR1tYWOC3VavX8/FwoFML4gnt8cnIik8nW1tZ4PN7+/n4qlSqVSmKx2GQyra2t3blzx+12q9XqWq12adS0Xq8HL1Cv1xHBvnXr1h//+MelpSUOhxMKhX766acXL17E43FcqVKpIH9Nk1h8Pt/29va9e/cMBkMulwMVnMRnnjx5wuPxEA1oNBqDwUCn01ksljt37szPz09pl9lsHiNCQ3/7q6++8ng8OB7DOKZSqXg8rlAoFhYWxGLxysqKXC4vlUqxWAzusVqttri4ePcCl57NEFu2Wq1LS0u426SxRnt9Ph85Zuv1+s3NTdhTqVTK4XBev3797//+78Vikc/nw0snkUhu377NPB4jQ/Krr74io3x0dATGy1guJeZYt9tdW1tTKBSXzgpYFplMtrCwIBKJiAMSrXO73T/88MPq6iqGSaVSgUOFGLJIJHK5XEqlcn5+njkrtre3//t//+82m42Mb6lUKhQK6XS63W53u11yvVqt3tzc3N7eXlhY4HA4kUjk+fPnmUwmmUySfmC3cV9S3BiZb5hRdrsdr65Go1EulwuFAjZSWq2Wy+WaTKZMJkPIuna7Hcw0rCZaMTUcDhsMBnqpgh9Rr9dx3OVwOEKhEN4ZwrbA2lxeXvZ6vSKRiMRAmIBd2t3dTafTtVptNBo5nU4SZUKV+EgkMpb5P3awPDk5MZlMLpcLGXqZTIasJpqR0e12f/75Z7AwoD4FFkm32200GtVqNRqN4tx+8+ZNSIyO9TAiYIuLi/Tzn5yclC6QSCTC4TAdk2Gu36WlJavV+u5ZgiTC9ve//z2RSFz6VOhbbO/4fP5Y16F/ENazWq04ooC1SMaRjqFhPgSDwXa7jetxyKTnA81MmfR21uv1LpcLMTdErZnPiXF/9uxZu91GLjTmc7VavbRduP7Vq1disRgeUnTvpH5gj8cfntM3Go1gf+C/JqsGCqawRdNHv9FohEIhRDiwWYduC3M06dU6Pz9PZjU+h9dpOn+K1DT58ccfsZ2DTaN1Q8A4W1paWltbI5w4wkPE9RsbGx6Px2QyicVimuPWbDbJLCUnt2g0OouV9ng8OPBc+uSI5T5//vzvf/+73+8vl8tSqXRpaQkdCCuHU2U6nfZ6vQgdTV+tGC/8Ir2abDZbo9E4PDyc0Qo1m00iVwYeYqvVQgx5zBpjvAjrEDI9ZFwikUgul4MzQi6Xuy8wFi56W4Ar2ul0rtHK0VaIOV4k6o7yimPzoVQqIaS0u7uLGcXlci+14Xq93mazLS0tkX7D53A/cbnclZWVwWAAAte//vWv09NTOEYJe6harYZCIWxT8a7k8Xjk+kKhoNPpcEZgvitptiN7PP6kAR8b8YUzD1Rer9doNBYKhcFgoFAoRqMRzj9CoRDX4EgDuiyygnu9Hvw3REcKfqlKpYI46mAwqFarYOcrlcrhcKhSqSQSCVyeqAPudrtx0J2kWYfowdnZWa1Wk8lkLpdrbW1ta2sLdkEul8disUwmg4yIs7MzhUIB3vVY67a2ttbW1gwGQ6lU4nK58NM3LxC7QLPZRITWYDD4fD4+n0+Ol5PaRTqHZESYzeZbt27hXA2vXqFQkMlk6C7E2/P5fKfTgWsQWdnwLEqlUscFcIJlRo95PJ5Go0GGM4IM6XQ6kUgIhUI8T7FY7Ha7aK/X64UPzGq1ggugUqnIA8Ov3Gq14LuFq4+ZHkNYA3fu3FlZWcErsN1u63Q6jCMzvm2+AJJemL0HFAqFcrksk8lIVE0gEKhUKpvNtrq6urW1ha1A7QLn5+fosVQqhV0jBHJCoVA4HG40GpgVNy5gt9vhfKlUKul0GrLAo9GIvv7S7sULZko/sPgcAY1TZL4JBAKfz7e0tIRQTKPRgJwB/N9IXROLxYgJwOK5XC5cj4AAVk2z2Wy32xKJxOVyETk6eO4REyiVSjiymkwmjUaD1YrFolar19fXNzc3LRYLEsAuZfoQ3/lvv/0GXonb7Xa5XHa7vdlsikSiYDCICDCfz1er1YuLizCtlzrLvF6v3W7ncDg43UEcER0ikUg8Ho9AIKhWq91uV6fTgeODHONCoRCNRuv1erFYrFQqUMi/tOqGSqVaX1+32WzkExAXSWAnHo8nk0kS66CtOuqg4neZFS+JVURY7NJRJtlocOwiVsDn8w0GA7qa8AjAtATnSCwWu1wu4jEcY+KIRCKPxwNiEUw3BC+YREdy/fz8vEajkclkSLpJpVLD4bBer6fT6W63q9frYeKYzy8UCnE6IsMEGY5arUaeU6lUNpvNaDTq9/sxdvPz84uLi5iE5XJZJBKBuROJRDBe3W4X14dCIZvNptPpbt68iZdjMBhMpVLn5+dj/cBWHP3woEefXt3ZbLbX64nF4uXlZdCpAoHAycnJ69evwZZC1SKZTJbL5YbDISqoI0SsUChwsrr0F7FakSgRjUZHo9H5+Xkul/P7/WSWTuJP4RWJSG8oFBIIBE6nc2try+VyIRWCeP1GoxFqEUOstNls9no9/O/W1tb29rbb7QbHrVwuV6tVBOtAM4SRwakmGAw2m03kT7lcLo/H43A44DvA8RhHIIvFMj8/r1ar6Tgtcyf58uXLw8PDfD4POvpXX31lMBgajQZO4LFYrNVqKZXKQCBgsViYPsex8Wq32wqFAquJXq18Ph/x6hmtEJfLJcdj2BMul2swGLC6YdjRG7BC4CA4HA7Et8i4QDCoXq/z+XydTre0tLSwsEAzq0nFBN0FmDxEGmQ7JxaLZ7FyNA+RmaZH9xvGF/3GHC/sGDEfkH1Jz4dut9vv9yuVysnJiVgsttlsBoPhUhcA9q4mkwnjOzc3F4vFKpUKKuYIhUK1Ws3j8cLh8Onp6dHRUbFYNBgMbrf71q1bUqk0Ho+Hw+FgMIgoC/HIkOtFItHt27cRjYNGRigUQryavFutVit7PP78gCwFkAN1Oh22UF6vVygU8vn8YrF4eno66btQ8jQYDJA1ev78OQwiFjxotNf1nPD3IOjhcDjG/GFkAYNifXJyIpVKNzc36TvQXjT4Mi0Wi91ux7maDv+iXTqdzuPxJBKJg4MDbJRnaRfubLPZXC7X+6vYTLcXVCuceJvN5vHxMaIx8/PzpL1isXhnZyeTyZyfnx8eHoIogtfY8+fP4aSc/osKhWLlAm9UFMSV8AXi/rPPCrhLtFqtwWBABAmjlkwmd3d3mb7MZDIZCoXw/HCy0OML10m73QaPXSAQ0NeXy+X9/X2YQkKuniIRweLzBdN67OzsLCwsKJXKfr8PWmy1WkVCxMLCQiAQIPMNMxDXY6uE4xnYB1wu1+12a7Vawo+AZA64OYiO9nq9+fl5rFaai3Hnzh0wBsPh8KQox+HhIVI8ZDIZWBU3b97Eodrlcu3u7j569KhUKmUyGbzCZTIZ3NhjLJKdnZ0ffvgBm1f8nEAgIDFkg8FAegabHki5cLncUql0enqK4zFRB2BaV7To9u3bOzs79NkPz18oFJClNhbroMfF/J+49GyGKuvw8UPscJLtNZvNDofDZrNBruy//tf/CocIljm4IXTULp1Ox2Ixh8NhNBrH3m6EjbK8vMzn86vVaiQSmZubI+NI/67L5bp9+3a73V5YWNBqtXw+Hzbq+Pg4Go0SZkEwGESGHtM9hzbu7OyQEnqgWSKXB88J/rnf7wfNSqPRLCws3L59++bNm9jfu1yu4+Njwpas1WrRaPT09LRWq8G6OhwOcLUwaru7u3Bh0P3AVhz9kKA5U8zVjaBuPp8vFovxeLxQKBweHpZKJeb8TKVSiNDu7e3h7aZUKuHRG/tFerXCB43M4bm5OcK/I/nJk2KnY2opS0tL33zzzdraGl6mLpcLL3qlUulwOJrNJgSWMpkMSK24fnt7G3lSkCO1WCygiCuVSovFgg1eJBL5+eefoWyCmKparbbZbEajkVhp7IhWV1dv3ryp1+snSdiMMVbofoDKLJb2aDQisXSJRHJpP9CrdTAYbG5u7u3tPXz4MBQKkdWEzLvFxcUZrRCCJfSsoFc3LEa5XCZWKBaLEetNj0ssFkulUnhrTNqLYpfr8XiWl5cLhcIUHiLedLdv38br441WjuYh4s04qd8wvrBCk3aDCwsLTqdTLpdPmQ8ajSabzTI1feh3n9PpvHR85+bmnE6nWCwmO2dwP2/fvu3z+SQSSSqV+u2339C3Wq1WJBIhTZJcbzabdy7A4XCy2Sx82eQkQo8Oezz+pCESifR6vcViAW0Vh4SDgwOklUNseWlpCQRjvV7P5XKnKDRAKxX+ktevX//666+hUIgQDAaDAQhpU0zV7EBuADxM8D9ptVoSBcVCQnU7JC0g0sj0WuFbiMHK5fJLq4wSDdhWq5XNZiH1PGO7eDyeRCIhucHkc0TdcThEiCkej7vd7l6vd3JyQuInCAsjfjKF4Ua3Bek6sCPdbjcejyORmL6Gy+UKhcJOpxMMBk9OTuDhEwgEaIJcLkc4940zZ5ZcXNEFmLMCwZzp38VzSiQSEsFgjjXxrRItX0RymONrMBgwshqNZjAYZLNZ+vpGoxEOh5GOjrdmpVK5NO+RxecO2nr0er1KpZLL5UQiUbfbBcFhfX0dBC2Q9huNBtlMYF7ZbLaFhQUkSrTbbbBLpFLpYDDAbG+1WoVCIZFIZDKZcrmMXQ70DuCipu2MRCKxWq3gJ6OY06T4DN7iRLxnZWVlY2MDuU9KpbLT6bx8+TKdTudyuWQymUgk4EQf2wYhF2NpaQn7Gy6XG41GE4lEJBKBncR/YaNUKhWEAAqFArY+4GuA9zHJukIxYWNjA2pk5PNqtQrGCvQFihcgERJ6XGgbPin6nU6nFQoFHPPg4Mjl8na7DdUGcE8MBgOpt4SdN54ZmW/oJTw/GEC1Wq1arTLlDBGP8ng8oMHDwYfCCmO2iFgbn8/XaDREIlGn0yGJJ4g2gKFD9wB9PEbcG3fA+HI4HKPRWKlUFAoF/ZzZbHasvRaLxe12+3w+Mivg4AClfzAYxGIxXA+KEJhKmN5IwJveDyzeN7BnmLS6I5EIWP3tdhvhCuwWoNvidDqXl5ehP6RUKqF3cHp6WiwWw+GwzWYrFApms5mWFx5brSqVCq/1RCKRvUCz2SwWi6CzYcfCVOUtFAqEvkuvZawy5IAsLCzgbY7Itk6nw+pGe7HVAVGCtlQymWw4HI5GI3DN8GCJRAJzFRsVRGthnOkdoMFgcDgcarX6Ug8aWD/gsGCPZDQamVYLJz3sFs7Ozsxm88rKCm2ULl2tFoul0+n84x//oFdTu92WSqXIH57FCiEFmuxCx1Y3dkS0FWJaY/BDK5UK7C2qJ4ztReldrsFgwHGAtFen06GeCGE4GgwGr9dLzvZvtHIgJE7i38GSo9/QokQicWkYCXtOkUjE5XJHoxHadel8KJVK4KWPEYvwRvP5fGAA4d1HgtXVarXVakWjUdhDvBNbrZbT6dzY2Pj666/n5+eFQqHRaIQOcb1ed7vdRqOx1+tls1lyPQSYMLL5fP6N70r2ePyJApE9vF9rtRrCaPF4PJ1OQ/PN7/dD+Hdra+v+/ftjfIxLAbW9p0+f+v1+VIM0Go03btzg8/nBYDCRSIzlpn5ecfWnT5+S+Oe7tAseKR6P9+jRo0wmQ7xWfD7/z3/+8+7uLly/Ozs7d+/eXVxcnJ59R28lMYJGoxElIpCwNHbWpSuvBgIB4vE1GAxbW1vVanVvb+96ucTMWTE/P//111/3er1Hjx59mFMo2oh/jO0PiNcWBCS8V97oI2DxBQCZey9fvpRIJGARr66ufv3114h5gg01y7yCqlw8Hn/27Nnx8TECO6RUxrs/JypGQOiVHNSJpj2Px5u/ANT+S6US6NzM4w3SQ+h6AcxP6HjCaDR6+fLl8fFxKBTK5/NjZTwmPS1UJJBDOMlSTdLtn30to+2weFBwSF/g+Pg4l8vR3BOEgpH9dXR09Le//Q0MKbpMyPRfpCmFszwhei8SiZD5ADtM994kjydaZzKZkD0Oy39p/QKk0kBJgcm1waxQqVQej4fL5er1+mQyWSwWcT36DeRPuiAWaxM+BUxa3WOjn7kAzl2yC5BVTK9B2ARQLaxW69jKpa8kOwF4VcjdkMWaTqcvVeWVyWRk/4PfghI1Tq2Li4t/+MMfcKiTSqVY+29UE8AKQiG94+Pjg4MD7EvHyhRdGXjObDYLJR1Q1sesFp/PJ73abDZJb9Os3dlXK8YUR9O3tULM1f1eTkRUe7FnOz8/39zcLJVKL168AK8H40Jm4xut3OyWfJb5gND68fHxkydPYrHYW80H8BEsFgsZX5p3Sd6zjUaj1Wrl83m0hW4v8plXV1eNRiMSufv9Pqgc5HpoZT98+HCsiNdnZ4LY6PH/H+AKomQOIgM4rtRqtWw2i9TWdrs9Pz8PaYFJd8M03d/ff/r06cHBAeIPyNBDUSikqzHTtD5x0Dq3L1++zOVyAoHAYrGgCMHV2gV/LYRhkcAAywLReVCbzGYzshx1Ot30PRn8arYLIDsXBDwEo+bm5kwmE6LQAoEAupGk8mqtVrNYLGazGfbdZDKl02mwR64FiBvv7u6SqtqYFTdu3Lhz506lUnn58uWnMMoIADJrBgLr6+ssw/BLcgt6vV7kutdqNQROSbGQSqUCdXfUKnzjaxvRwsFgUKlUwI84ODhArBXLFloM77iZo9Xgye8SbWpEMJRKJVw/iC9BdXbMgw5GBigkkz6hIw+pVOr4+PjFixehUKharYrFYoS+BQLBpXU7mcdgZs+n0+lsNkvr9qOuO4mDKRQKWDOECyb1OSIhEomk1WohvoGSqtBQRZzfYDAolUqcEJCbt7+//8svv4TDYfB61Gq1VquFa3/6xnFKNJsJRIaZ84HL5YrFYqh4TI8fYp9KLD+Ge+xETZfuY3JtyKwg/PZsNttqtXA93H/w4JBJjuxE7O8Rcx6bFSw+DCat7umjT48XPWOJTUDYecrcxsqSSqXg7ZO7IfDVaDT29vZQHgmw2Wyj0QhZGC6XK5VKQbUBqxtu6HQ6rdPp5ubmUKBuxgIQsHiFQgHSWQcHB9lsls/nS6VStVotFotR7uTKPczsE+Yap3t1UpnM2Vcr2SdfwQoxVzc5XiqVSkR9oZg9KepLq8/MYuWKxeLZ2RlCypDkKJVKUHMgh/xGo/FGK8cMRUya5zPuJ8/OziD9fX5+/lbzgTm+zE8w69A0zAr0MIm3Eyki/G/hAig5juvxIkNRFbJ8pBfAHuM9uTbY4/F7AdSnNzc3W61WIBD4+eefsVlEdbVGowG+7k8//cRM6B+Lw/z8888ogIb6yTdv3vzuu+/gNQyFQvv7+59j/yAnAWf+TCYjEolWV1fv378PQsjV2gUvnVar1ev10G98lw0Z/FvVavXWrVsoYAC/MhzMdBQaeoyvX78mlVftdvuDBw9QsXMwGMBDfI29Bx/kkydPSFVtzApUeHr16tUHE/GDtxg7v0tXAakZyKQeoUQ+ayu+DFgslgcPHtjtdlRSIcrDdDzN6XSSWoWz3JNml1QqFZ1OR0rgQiovHo9P3yh8mnYP+t6JRAK17kwmk1arRRWoK3BM8K7hcrljuv2xWAwxIqJ5iwzA6eoGpOo+yZSjt7BKpZLEe+nq6+Dm4LvgUlar1YcPH15vStik+SAUCmu1WjKZPDk5+RTU/tAPTN8f0baZ0R3A4ssGVtYkrgf2GDjaKRQKorIOiwfN6r29PZPJ5PP57t69CwGnNx6ECMctHA6PRiObzYZs+fe0Zt83kJySzWav0Qox936Tor7Ytd67dw+ZJpP2pZcyichRfCzSO4uVuy4uMVFH/+tf/+r3+3u93qc5H9BLyNMcy7tBJefPpfTx793uI2O23++TiBm4ahCvT6VSUHVDHh1SRlH25lLAKxYKhcCexcTd3t72er2tVisSiXS7XWQjXMsUhEcHEZKxHDZ6k0TX+73aQiXqf5DskslkUFlcX1+/crvw/Eaj0el0In8MrYANnSV+MnY3kjeC67HFxzuG1oAVCoVgekAzlsRztre3jUYjkoWuN98M3j5UnRGLxWOz4rp+Bd5Kuv4zPQfoSqTdbheiDnNzc/T1yN1yu91ra2sQ8+h0Oihtivj8Z1TPncV0ILtYqVSqVCqdTkequ0P9slwup1KpaDSK6AeUSKesbhCogsHg3t7er7/+CqmY1QugJtzh4SE0PD+X4zFdGRX63nNzczdu3FhaWrLb7UqlcjQaCYXCK3BMEGPJZDJOpzMajULTO5VKVatVUjlTr9ejGoLT6ZweaAIHB9ndWO/9fh/1VMc0WumYAyI2sEJbW1sbGxuxWOwf//jHNbrhUNH9+fPnzPmA7F8Y+Q9zPMbzYCc99tYg/cDcMWu1WqIhzFoMFoheIgJMyrCD46bT6ZDlC+V/jUYDNhxUqWAho9EotKkqlQoUqt44+aGDA45bq9VaWlpyOBwWi2VxcXF1dTUWi71RFvRTA+KKoVDoGq0Q9n4Wi8Xj8UCFK5/Pj0V9K5UKNoE3btzwer3TD9tQwTCZTM1ms9/vo/oJXo6o2uByuaDbP6OVu67ew3z45Zdf9vf3K5UKmQ8ej+fabfjVLCrZXWxsbDgcjkuPxwsLC5Ok49nj8acFUuOXaDVLJJLt7W2MOgSooOr27l6fcDiMBN1ryeek4wZEWZEooNIZWXS93+vNiX+XdsHDhERHJAWhFbCheOZbt275fL6r6V0j8oN/zP51ut/e99xD70Ha993vhqweuv4zrYtLVyItFosQY9/Y2Jh+Pb0u9Hr9lKqPLD4vYPVBZmN1dZVUd0dtDLoWokKh2NzchObHFAfQ6enpy5cvj46OIN/g8XhINfjIBT7H90Kz2Ryrvg59UVQLh7f0aoB9y+VyNEuTgJkhNsXNsbGxUalUUDs9nU7zeDyi0ep0OtfW1hB/Bl/myZMn+Xye1vhdW1ubUl3/XSxbIBAAMWFsPqTT6Q+ch4bnQZYg8x2KfiCvTtqiyuVymUzGugVZkPxkiURSLpdpUXfUN8FqVSqVYHy02+1wOPzLL79g60hXO4/H4ycnJ2azmS4OMmUPA46bXq93u93b29s3b940Go2j0Qh5p58X8vn8/v4+RByu1wpNt5lkt/zGtUyrESHXGufewWBQr9ehWb25ualWqz+8lYNKyOPHj3O5HJx3mA9QX7teG341iwpYLJb79+9vb2/T5GraQTwmlskejz9RIEqAbQqI+0ajcWtri7mKJgmTIGKMcBwqZJrNZoQFKpVKKpUKBAKFQiEQCLx48eKN5y5EADKZDA6KarV6UtyGjhugHKhOp3M4HAggBy4AHTmYb2RcTKp998ajF9FAhtonYukwIoiLvu15Ehk+SBrBnSuVClKOkX1HFMWZRN/pz4mvFAqFTCbD5XIhOKRWq0lmkVgsVqlUJpNJr9ej3nU6nUbdvHg8TvrtuuYYXaG00+mMzYqDg4NrKZ6EdiFelEgkSA1A1PHLZrO5XO74+Pj58+fVarXZbIJMOP16FFdAnT23282Sq78YwOhBRoGu7o6M/VAohJKbnU6HVDWcHj3O5/NQbYAAARQyFxcXoYQJjc13dAsyrRCpjq5QKCDCB3VQRAXpVX+19wISNOjq68gwzGazUFsgbJ0rHI+RgXx+fg7WTLPZRI1l+LnsdrvVan3jZo5+K0H/n8PhFIvFwWAAehusHES5SB17GECEHaxWa6lUepe2XOpcqFaryEmj5wMKE0DG/LoYOugBFPDEO5f8LuRVUaAuEAgIhcLhcNjr9cg7GhMS8wpb536/XyqVriaWxuKjHFnNZnOhUKhUKrRGMbQ2aWc3+GgQjUeuwRXsD/YkHA5naWmJfI4SOzqdDrmXqBUvlUptNtutW7dI6ThS7RzRSGZU7dJDSPIC9Cz1er1isfjs7OzdnfjI6GbuhdLptFKpFAqFrVYLbwSs1rcV52Pi/VkhOuoLKk25XE4kEhgROuo7/T5QI8LOUy6XQ08bu/FGo6HVaul0lQ9m5eizBqmqQ+YDUqyvK6hDkvxJ/WdSTdpgMGBWxGKxly9fdrtdlEnj8Xh0vWjwLKBvh+sbjQaUwz8v88Im1XBI5bfj4+NSqfT/Y+9Me9q42j4OXsELHhu8gs0YsA3cBUOoSVIqRCtFVaW2Uj9A1Q+QL1WpH4EXVdUXoEhNpYYk0JSAYwPGZrFNaux4S2xsHom/nqPzeJiJDfTO8ly/V9SZ2jNnueaccy1/k8mUSCRwvMGr4Hq93nA4PDo6WigUsAphJzqrq6v5fH5sbAyLnkqlEo1Gs9lsKpWCCDiEGdvRPcaC7NmzZ4VCAfkYcl5Z3m/A/xbSgPk75/WNr9Y+vNYu/CrIxMa0vCl1XHir2Pb4yveZy+USiQQiqHU6ndfrHRsbGxkZ4etS+ny+6elp/CJCVnK5nE6ny2azbeoetw+vUBqPx68wKq7QU0wDsFwuFwqFvr6+XC53eHgYj8eZftVbr4fN1ev1ExMTTqeTlowfDbBaeNHy6u7YTphMpmg0ylu565wx8+fr7SwK27dC+/v7z5496+vrGx8fr9VqzNMCXwE/629w4wf/D9N4vP6zHBwcxONxtVqNJlJQ5nxra8NV1VE78x6qf3vUofUQq4Jg/ut/pyAIUAFl78FoNBqLxQYGBoaGhnK53Orq6uPHj6PRKBRHRVFseUfz15dKpUePHkH2CX0kiiKqB5PReA+BN0+r1eIYC76+4+NjlFDGmx0H0IjXmJ6e9vl8crVO3ro9tlqtIyMjDoeDX+hrNJqenh4c0BweHqbTaUThOp3O0dFRjDpe7fz6T80/1zU3QpeuhSwWi8Ph0Gq1ME17e3uYrQMDA8x3erPhdde3QrxqMdSkoWbfaDTy+Tx8rVdbA/N29V1ZuXbe5vAqX//b0JJ9fX1M/xnf32w2l5aWbDYby3/O5XKhUGh6evr27du8XrT0+kQiEY/HkTspiuLS0lI7GkC0PX7H8L7Eg4MDdG2lUsEsQmmu8/Nzv98/Ozs7NTXl9XozmUwul2vRSUZmQigU8nq9U1NTsVisXq8jEeL8/Ly3txfCaGNjY9VqFbUNS6USDsyazabb7YZ3EalotVotk8mUy2Wn0ynnPcZJFSJmYTELhcLGxgb+E6oVENaDip1UzEAZvmIhTuYg99Ld3b2/v59Op588eWKz2bRarU6nGxsb6+/vPz4+hh8J34BqgXJVYdlZHa87XSgUms0mi7tux38id4KYTCZLpRLyaQcHBz0eD7LI4LVG7nStVkNGbiKRQDnuZrOJoMRSqYTuwMkZCnq132L83zg0KRaLsVgMBWYxKoxGI6pY48vR78yPhFxu5daT/i4bFUdHR8iyPjk5efr0qUajwemdVquFYi3aVvl6jL3BwUFUUKPI6o8GnHzjaMZsNjN1d/geX716BTkHpjperVZZDVLoY1ut1lKphNwnqXo8Ml1rtdre3t6TJ0+SySTT4pY7Gnvx4gUcO6gtLDe7Wf35er0OKwQx4Vqt9vDhw7/++iufz0tn/dXeC9hmo2Ap7hD/ioMkPsaEVftsqZKtsNRmnnDm9T09PYVKgpwyp9x9optwn7glQRAQscLEhJm/unIBDnk3Nzd1Oh1rt5saXdJasuwteXp6iuRDqHRi0XmdehwGgwHVQPr7+9VqdblcTiaTa2trzWbT7/fncjnoBaAaCPYtDoejWCxeen2pVHr48GEymWw2m0NDQwolfIj3AbPZDNlVFM87OTnZ2dl59OgR8nvT6fSff/4ZjUaLxaLVahVFkalgyCmKtTPXLi1s+fLlS61WCyuRzWZ1Op0oiqhgDAfM2dkZr2/scrnYVhCxMIhh7OnpEQTBbDaztzOsNKxENBrV6XSlUgknPqwOk9T+8Ne73e5Ln7cl4iyZTMLz0dPTYzAYsLHhZytf+flquz58g1qtvnErhAxk+DDh9S0UCrAq5XLZ4/F05PdmLX9wcABdZb1eD183ghNxXta+lbup0Q4bfnx83DIefv/995uKQ4T1djqdqI7x8uVLPBfKudtsNrzT19fX6/X6wMBAo9EwGo16vV7h+p2dnXg8XqvV0BEfSgmS/+/bY+zEoOKFclzPnj3LZDJYryAB3e/3f/vttwsLC5OTk1arFYvIFp1klUolCMLAwEAwGJycnOzq6nI6ncvLy3t7e6lUCiUB4Mo4OjpaXl6GfNze3h5yhm/duqVSqbDxZtqzZrNZ4byK3XlXV1cgENjY2Njd3cXhDTtT9/v9Y2NjoVBocnLSbDZ3dDLHV43Gb4XDYajG//zzz1tbW/CCzs7O+v1+m82WSqWWl5f39/e3trawHkXBHoPBoFB9Wqo7Den20dHRzz77LBKJXPoqUm4TRGvjF1Efm8kYMicA6kziBGt9fR2q9xBmWFpa0uv18Xic1dNaXV09OTlh0iDttBivnoe7CgaDX331ldFoXFlZQRVfv98PpXUEX62srBQKBeZHQtQA/z0Kv8uu6enpGRoa0mg0Fotlc3OTjQq8nh0ORzgcnpubi0QiCBPVaDQK1wuCMDo6euvWrS+//BKDnxZkHwfwjJ2cnLx48eL58+dM3b2rqyuTyeAFzNd7Pzo64mt1rqysbG1tDQ4Ozs3NffPNN8PDwy2zGLVDjUYj3ClvPdXG9dvb206nU61WX2qpMI8gvQb/D3wOqVRqfX292Wzu7Oyk0+lisWiz2S6d9Z1aEo/Hs7CwcH5+/ujRI9SeRS2cSqXSomkJrUj8cYWjCjyvgg6w8n0iLxr3iVcGIlYWFhZYZS/kg/X19WUymb///hu1yvP5vEqlYu12U6NLWksWSphms1mqCHqpLn2nPr1KpcJGBT8+a7VaNBrFSXEoFAoEAn6/X6PRZDKZS68/Ozvb2dmp1+tut7tF3ZR4D0EfofAS5tHm5ibUhhFczfRg3f/LDcaSSFcy1Wp1fX396OioWCzu7u7u7+/jpQkBS1QEwNqmv78ftVGYBdjc3Ozq6orH4zMzM8FgcGhoCFoSarV6ZWUlk8kwW3d2dhaNRtPpNFsfSu0PbxsnJyfn5+fl7pzP2uWVC3Byh11fuVxu0b+9Wiu53e5IJLKzs/NfsEK8XW1nHcXD6wsgx/vSajjtW7kbAXoTFotleXk5lUopjIfrw+shI+sQPaXT6dg7XRTF8AWCIJRKJYXrsaRE5A6up+3xBwB8iVDxUqvV2WzWZDKdnp7yZ2PBYHBxcXF6etpqtcLVXK1WmU4yvCJwnuDg32azwUtZKBSga+LxeCKRSCgUwva4UCighuHg4CA031AIPpPJ8DnrOOns6uqanZ3FRk4URdRNhVA+vKB6vZ6lmfH3MzExEQwGx8bGPB4PbHStVoPXFN8WDAZxnMbWZMxL3N3dXavVEG9pNpvZb7lcLpVKtb+/j/Ht8XhmZmb452IOFgaUVOW05vhMD4vFkk6nu7u77Xa71+v1+/0ej+cKvWm32ycmJvCwOp1uYmJCqpzcewFOFnt7ezG9rVZrKBS6e/euyWTCKZfdbq/VahaLBSdqqCSBTpHqAONsr1qtst9FzVjeX61SqRqNBoJaRVG8e/cucpO8Xm+j0Uin0+gpi8WCEYh+5OXEMFxb+tHn82FUAJ1OZzab+/v7W0YFxmE4HB4fH2frZoXr8evT09OTk5Od9gXxPoPTcWS/FwqFXC5XKBSwmMBexWazBQKBO3fuzM7Oulyuer0+MTGRv+D169fFYhELMmTc8erxzWZTr9cjHgFaxwaD4ZNPPkFucKPRQDVLxKRgQ/L69WuNRnN4eIiMX9gWo9GIpCa+agCbR9jGQ7c5m81iCZLP5w0Gw/AFLbMe81dOzxbxF9hRs2tglmdnZ/GkaKjT01OstERRVKlUiC5BZiP8k3q9Ht+grO4o5/VtR5lTavHg/8lms4eHh3BWsBqt7HugVlKtVre3t1UqFYKkEokE+m5kZIRtJxDVghxO9JeyDjDSF/mnBqhJC3lhFH7LZDI9PT1Go3FkZATjgWX5IrcFMTWCIHR3d8v9Vks/QgdVOioODg6wYahUKkhEmpubQ2AtMvfkrs/n8zhcDgQCbre7/coXxLVWoheHy5iAMALSI3W+9/FS5t/LcJmWy2XYBMxrjUaDQGtUJ+Xj0fhvk85WZhNarJDc/WMeDQ8Ph0KhXC4XjUbz+fzu7i7sDwQg7HZ7KBS6ffv25OSkyWTK5XIo2dBsNs8uSCaTKAAGk4txi3z4zc3NUqkEW2cwGBwOB2pfI6sT9kev1/t8vmazCSv95s0bdv1//vMfuefFqm9iYgIpx5iVCKBoycyfm5vD+rPN9pHO1v7+/vHxcZvN1qYV6u7uRkyc1Wo1GAxWq1W5F1q8vo1Go7e3F45Qp9PZvuIu9tLHx8fIKsfWNHABbxM6snI9PT04x2FWDms86V3Jva2gN4E0Pa1Wy959GA92ux3K9vl8vre3F4GoTqezxYYbDAbldx/uBy1ZKBRqtVosFmM9hRXC+fm5KIqIqMXqFJ8oXG+z2UZHR8fHx5XFcWl7/J6C8/XPP/+8Xq/z3n+z2ex2uwVBYIOY10nGygyFHwRBgLOC+VpxGIaIGrPZjH04hAEwBAVBMBgMcN+x6/nf7erqYp+j5psgCLyJZxVo79y503I/ZrPZZDIxzaqWu5I+F/xFgUCgWCw2m03sG1uqzA0PD//www+o1iN9LunhH8rkSL9H4dTKecGVT3n53kFsM/pFeiX/vGdnZxB8HxgYwDt1amqKtYPRaDSZTPV6PRgMYhUl1QFW/l2cvJpMJvYN6E1YCo/HEwwGUVAE550Ygfy44ntcuR/lRgX6C7FbUh+U3CiSaz3iQ/e6YAEkimI8Hi8UChg/MC8ul2tiYuLu3bsoqcDrJEMVHGVd+MUKxr/D4bBYLLu7u6gy1d/f7/V6WW4wwkPwf5nN5kgkIooijr21Wi1GGrbHCHatVCo2m423BrDSn376aTab3dnZ2djYQFoE7tzr9S4tLYVCIens83q9yH2S6tliaW632/1+P8K/sRyERwiXeTwellUoiiKOR7e3txOJRDqdPj8/R8AR5tFb1R3lvL54OmVlTrmtBX//o6OjOBVtWXjBek9PT6+urqZSKa1W63K5JicnEbKeTqdXV1dRjC2ZTA4PD0OnPRQKKegAo23xN//UfIQOjlF0Op3L5RJFkY0HZPlaLBbUbzs/P3c4HH6/H6NR7rek/ag8KkZGRr7++uuZmRn2Dur0euLfBme+OCyuVqsts17a+/xIw6heWlpqyXLEuh+zNRAIsLUT4pDxbThOks5WGCiMRqkVUl5DDg0N/fbbb4lEAm4MhE8LgiCK4r1792ZmZtxut0ajaZkdjUajZczLWWlRFBcWFiwWC3veN2/eHB4eulyuL774olwuX2ql5Z6XX5nEYjE2K9FKyNpFBr7X62VrnnbaR262tm+FEEeJjuvr6/P7/coB0lKvLx8D1WkkEX//LpdrZGREGn1wHSsH/4p07Mm1mzRiFFYL48Fut5+eniYSiY2NjWKxeHh4+Pz5c4vFwttwURQdDgfvRVd+97lcrvn5+a2tLdZTsJB2uz0cDk9NTSGoECvnjq6n7fEHBvaT7Z8tKR/qw9cq/SeLxXLp53BWXPptcp/z3m8c/LzV1SB3V7xPVdlPKFzQ/nPJgdBr+F5isRi06er1OuKQ2/efXKF32nleuQ0hIvCv8LvsqPvSb0DV8fb9Rcr92NGouML1xIcO8++pVKr+/n6Px4OXMTCZTHjBQ/e1RScZCy8giiKbpxj/Op0OsanShUJvby8WCvgc9R2gh4waV/hdfO50Om02G9uOtnhBfT7f6enp4OCg9H4WFxel0vTww2A5ItWzRe702NgYq0kLbUb4NBCyaLfbpdtjpOSw++ftA0t1k8v6U6vVyDFmfrA2lTkv9VxJ7//SdSSst8ViaTabrAJqOBwWRRELU/a53W7H44fDYUTdS9uNGbRarYaNDa9pyUfoOJ3OS8eDz+fD59i3oENtNhvak+935X5866iYmZnhR0Wn1xP/NphWGJDSWX9p77OYqb6+PkEQ/H7/wcGBz+dj1ozfHvNv+Uajgd34rVu38Ll0tmJhUKvVLrVCymtInU5XqVSk2ViiKM7Pz7Nx1TI72HZ0cHCwJQqjxUpjO4TcYP55MZexGOPHM+Lgurq6Ln1efmWCnALMSun2mI+kaKd95GYr+qsdK8TD3koKqyy+pgMezeFwIHDAbrd36nGR3r+0Gs6NWLmWFbVcu0kjRtHLGA/IZvf5fOxzvFksFguz4S6Xi8WftvPuw9vfbre39AjfX2hVtVrd0fUfBN3/hbpqBCHl5cuXzPcSi8UeP36cTCar1erMzMz9+/cXFxetVislfRHEvwdqK0hFF5DEbjQazWYz9kKNRoMFbvGvDD4CAlSrVRZOplKp2PfwYWb4HAfYCK5GhC1+l/+82Wyy8gG8NWj/fgBK3yFqg9ezZcEaLe3Afw9/kMffDx9cLU05Rokdo9HIPEhSotHo8vLygwcPNjY2II6wuLj4448/zszMXLMf5dpB2hr8ffJyX/gcwcZo4U7b7frjQfptyv3Y6ajo9Hri30NulvGzvtNZjODqdqyHdLa2cz9y8PfZcgSgMDtYMHObs0wqmSOds7wt4j+Xs07t3M9N9Vc7VohH+laS8urVq4ODgwcPHvz000/b29uIs/vuu+8ikQj2jR2NSeX753m3Vg7fAK0p6ahW7vd2bLh0PCu/3Tq9nrbHBPF/iMfjv/76K6KAUKoKruPFxcX79+9HIhFqIoIgPjL4hVQ0Gv3ll1/W1tZSqZTZbL59+/bi4uK9e/d4VVWCIAhCDn67eHh4GIvF1tbWVlZW/vnnn8HBwUgk8v3338/Ozn5YfkvifYCCq4l3Ayor/vHHH9Cgqlardrs9cgFluhIE8RHbPQSZ7+/vt+jqf0BVPQmCIN45qFCNnNvNzU1kHZ+cnMDvarPZrqxfQND2mCDexci7KAnA6iejJMDCwgItEAmC+FiBjvrGxkY2mz0+PkbUjN1uDwQCLTXqCYIgiLduj1+9epVKpdbW1p4+fZpKpcrlslarhYLJ1bKOCYK2x8Q7A7Vw+SBqlHJ1uVxXLspFEATxPgOZqGw2m0gkstksi5q5Wm1VgiAIsqiwqIjEQYXnUCh07969+fl5qj9P0PaY+JBALVy+RiuEQOmcjyCIjxVUfA0EAj09Pagiy9TIlTVLCIIgiBaY6MbIyAjkfLGSDIfDVH+euA5Umot4N6CgAq8vjUL8H1BdO4IgiI5AVc98Pn+pGrlcRVaCIAhCCmov5/N5Vi0ZK0lBENxuN4UiErQ9JgiCIAiCIAiCIIiro6ImIAiCIAiCIAiCIAjaHhMEQRAEQRAEQRAEbY8JgiAIgiAIgiAIgrbHBEEQBEEQBEEQBEHbY4IgCIIgCIIgCIKg7TFBEARBEARBEARB0PaYIAiCIAiCIAiCIGh7TBAEQRAEQRAEQRC0PSYIgiAIgiAIgiAI2h4TBEEQBEEQBEEQBG2PCYIgCIIgCIIgCIK2xwRBEARBEARBEARB22OCIAiCIAiCIAiCoO0xQRAEQRAEQRAEQdD2mCAIgiAIgiAIgiBoe0wQBEEQBEEQBEEQLfxPAAAA//9MCDU0Ze8d4QAAAABJRU5ErkJggg==)

The check for stationarity can be done via three different approaches:

- __visually__ : plot time series and check for trends or seasonality
- __basic statistics__ : split time series and compare the mean and variance of each partition
- __statistical test__ : Augmented Dickey Fuller test


```python
# A year has 52 weeks (52 weeks * 7 days per week) aporx.
rolling_window = 52
f, ax = plt.subplots(nrows=2, ncols=1, figsize=(15, 12))

sns.lineplot(x=df['date'], y=df['drainage_volume'], ax=ax[0], color='dodgerblue')
sns.lineplot(x=df['date'], y=df['drainage_volume'].rolling(rolling_window).mean(), ax=ax[0], color='black', label='rolling mean')
sns.lineplot(x=df['date'], y=df['drainage_volume'].rolling(rolling_window).std(), ax=ax[0], color='orange', label='rolling std')
ax[0].set_title('Depth to Groundwater: Non-stationary \nnon-constant mean & non-constant variance', fontsize=14)
ax[0].set_ylabel(ylabel='Drainage Volume', fontsize=14)
ax[0].set_xlim([date(2009, 1, 1), date(2020, 6, 30)])

sns.lineplot(x=df['date'], y=df['temperature'], ax=ax[1], color='dodgerblue')
sns.lineplot(x=df['date'], y=df['temperature'].rolling(rolling_window).mean(), ax=ax[1], color='black', label='rolling mean')
sns.lineplot(x=df['date'], y=df['temperature'].rolling(rolling_window).std(), ax=ax[1], color='orange', label='rolling std')
ax[1].set_title('Temperature: Non-stationary \nvariance is time-dependent (seasonality)', fontsize=14)
ax[1].set_ylabel(ylabel='Temperature', fontsize=14)
ax[1].set_xlim([date(2009, 1, 1), date(2020, 6, 30)])

plt.tight_layout()
plt.show()
```


​    
![png](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_30_0.png)
​    


In this visual check, we can see that the features don't have constant mean and std, but they are close to it.

#### Unit Root Test

Unit root is a characteristic of a time series that makes it non-stationary. And ADF test belong to the unit root test. Technically , a unit root is said to exist in a time series of value of alpha =1 in below equation.

$Y_{t} = \alpha Y_{t-1} + \beta X_{e} + \epsilon$ 

where Yt is value of the time series at time ‘t’ and Xe is an exogenous variable .

The presence of a unit root means the time series is non-stationary.

#### 2.3.1 Augmented Dicky-Fuller (ADF)

Augmented Dickey-Fuller (ADF) test is a type of statistical test called a unit root test. Unit roots are a cause for non-stationarity.

- Null Hypothesis (H0): Time series has a unit root. (Time series is not stationary).

- Alternate Hypothesis (H1): Time series has no unit root (Time series is stationary).

If the null hypothesis can be rejected, we can conclude that the time series is stationary.

There are two ways to rejects the null hypothesis:

On the one hand, the null hypothesis can be rejected if the p-value is below a set significance level. The defaults significance level is 5%

- **p-value > significance level (default: 0.05)**: Fail to reject the null hypothesis (H0), the data has a unit root and is non-stationary.

- **p-value <= significance level (default: 0.05)**: Reject the null hypothesis (H0), the data does not have a unit root and is stationary.

On the other hand, the null hypothesis can be rejects if the test statistic is less than the critical value.

- **ADF statistic > critical value**: Fail to reject the null hypothesis (H0), the data has a unit root and is non-stationary.
- **ADF statistic < critical value**: Reject the null hypothesis (H0), the data does not have a unit root and is stationary.


```python
# https://www.statsmodels.org/stable/generated/statsmodels.tsa.stattools.adfuller.html

from statsmodels.tsa.stattools import adfuller

result = adfuller(df['depth_to_groundwater'].values)
result
```




    (-2.8802016493166605,
     0.047699190920208856,
     7,
     592,
     {'1%': -3.441444394224128,
      '5%': -2.8664345376276454,
      '10%': -2.569376663737217},
     -734.3154255877616)




```python
# Thanks to https://www.kaggle.com/iamleonie for this function!
f, ax = plt.subplots(nrows=3, ncols=2, figsize=(15, 9))

def visualize_adfuller_results(series, title, ax):
    result = adfuller(series)
    significance_level = 0.05
    adf_stat = result[0]
    p_val = result[1]
    crit_val_1 = result[4]['1%']
    crit_val_5 = result[4]['5%']
    crit_val_10 = result[4]['10%']

    if (p_val < significance_level) & ((adf_stat < crit_val_1)):
        linecolor = 'forestgreen' 
    elif (p_val < significance_level) & (adf_stat < crit_val_5):
        linecolor = 'orange'
    elif (p_val < significance_level) & (adf_stat < crit_val_10):
        linecolor = 'red'
    else:
        linecolor = 'purple'
    sns.lineplot(x=df['date'], y=series, ax=ax, color=linecolor)
    ax.set_title(f'ADF Statistic {adf_stat:0.3f}, p-value: {p_val:0.3f}\nCritical Values 1%: {crit_val_1:0.3f}, 5%: {crit_val_5:0.3f}, 10%: {crit_val_10:0.3f}', fontsize=14)
    ax.set_ylabel(ylabel=title, fontsize=14)

visualize_adfuller_results(df['rainfall'].values, 'Rainfall', ax[0, 0])
visualize_adfuller_results(df['temperature'].values, 'Temperature', ax[1, 0])
visualize_adfuller_results(df['river_hydrometry'].values, 'River_Hydrometry', ax[0, 1])
visualize_adfuller_results(df['drainage_volume'].values, 'Drainage_Volume', ax[1, 1])
visualize_adfuller_results(df['depth_to_groundwater'].values, 'Depth_to_Groundwater', ax[2, 0])

f.delaxes(ax[2, 1])
plt.tight_layout()
plt.show()
```


​    ![output_36_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_36_0.png)

​    


If the data is not stationary but we want to use a model such as ARIMA (that requires this characteristic), the data has to be transformed.

The two most common methods to transform series into stationarity ones are:

- Transformation(변환) : e.g. log or square root to stabilize non-constant variance
- Differencing(차분) : subtracts the current value from the previous

#### 2.3.2 Transforming(변환)


```python
# Log Transform of absolute values
# (Log transoform of negative values will return NaN)
df['depth_to_groundwater_log'] = np.log(abs(df['depth_to_groundwater']))

f, ax = plt.subplots(nrows=1, ncols=2, figsize=(20, 6))
visualize_adfuller_results(df['depth_to_groundwater_log'], 'Transformed \n Depth to Groundwater', ax[0])

sns.distplot(df['depth_to_groundwater_log'], ax=ax[1])
```




    <Axes: xlabel='depth_to_groundwater_log', ylabel='Density'>




![output_39_1](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_39_1.png)  
    


#### 2.3.3. Differencing (차분)


```python
# First Order Differencing
ts_diff = np.diff(df['depth_to_groundwater'])
df['depth_to_groundwater_diff_1'] = np.append([0], ts_diff)

f, ax = plt.subplots(nrows=1, ncols=1, figsize=(15, 6))
visualize_adfuller_results(df['depth_to_groundwater_diff_1'], 'Differenced (1. Order) \n Depth to Groundwater', ax)
```


![output_41_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_41_0.png)    



## 3. Feature Engineering


```python
df['year'] = pd.DatetimeIndex(df['date']).year
df['month'] = pd.DatetimeIndex(df['date']).month
df['day'] = pd.DatetimeIndex(df['date']).day
df['day_of_year'] = pd.DatetimeIndex(df['date']).dayofyear
df['week_of_year'] = pd.DatetimeIndex(df['date']).weekofyear
df['quarter'] = pd.DatetimeIndex(df['date']).quarter
df['season'] = df['month'] % 12 // 3 + 1

df[['date', 'year', 'month', 'day', 'day_of_year', 'week_of_year', 'quarter', 'season']].head()
```





  <div id="df-62a83b8c-114f-40f2-afff-bb7355d87590">
    <div class="colab-df-container">
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
      <th>date</th>
      <th>year</th>
      <th>month</th>
      <th>day</th>
      <th>day_of_year</th>
      <th>week_of_year</th>
      <th>quarter</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2009-01-01</td>
      <td>2009</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2009-01-08</td>
      <td>2009</td>
      <td>1</td>
      <td>8</td>
      <td>8</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2009-01-15</td>
      <td>2009</td>
      <td>1</td>
      <td>15</td>
      <td>15</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2009-01-22</td>
      <td>2009</td>
      <td>1</td>
      <td>22</td>
      <td>22</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2009-01-29</td>
      <td>2009</td>
      <td>1</td>
      <td>29</td>
      <td>29</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-62a83b8c-114f-40f2-afff-bb7355d87590')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }
    
    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }
    
    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }
    
    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-62a83b8c-114f-40f2-afff-bb7355d87590 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';
    
        async function convertToInteractive(key) {
          const element = document.querySelector('#df-62a83b8c-114f-40f2-afff-bb7355d87590');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;
    
          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>




### 3.1 Encoding Cyclical Features

The new time features are cyclical. For example,the feature month cycles between 1 and 12 for every year. While the difference between each month increments by 1 during the year, between two years the month feature jumps from 12 (December) to 1 (January). This results in a -11 difference, which can confuse a lot of models.


```python
f, ax = plt.subplots(nrows=1, ncols=1, figsize=(20, 3))

sns.lineplot(x=df['date'], y=df['month'], color='dodgerblue')
ax.set_xlim([date(2009, 1, 1), date(2020, 6, 30)])
plt.show()
```

​    ![output_46_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_46_0.png)




```python
month_in_year = 12
df['month_sin'] = np.sin(2*np.pi*df['month']/month_in_year)
df['month_cos'] = np.cos(2*np.pi*df['month']/month_in_year)

f, ax = plt.subplots(nrows=1, ncols=1, figsize=(6, 6))

sns.scatterplot(x=df.month_sin, y=df.month_cos, color='dodgerblue')
plt.show()
```


​    ![output_47_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_47_0.png)

​    


### 3.2 TimeSeries Decomposition

Time series decomposition involves thinking of a series as a combination of level, trend, seasonality, and noise components.

These components are defined as follows:

- Level: The average value in the series.
- Trend: The increasing or decreasing value in the series.
- Seasonality: The repeating short-term cycle in the series.
- Noise: The random variation in the series.

Decomposition provides a useful abstract model for thinking about time series generally and for better understanding problems during time series analysis and forecasting.

All series have a level and noise. The trend and seasonality components are optional.

It is helpful to think of the components as combining either additively or multiplicatively:

- Additive : $y(t) = Level + Trend + Seasonality + Noise$
- Multiplicative : $y(t) = Level * Trend * Seasonality * Noise$

In this case we are going to use function seasonal_decompose() from the statsmodels library.


```python
from statsmodels.tsa.seasonal import seasonal_decompose

core_columns =  [
    'rainfall', 'temperature', 'drainage_volume', 
    'river_hydrometry', 'depth_to_groundwater'
]

for column in core_columns:
    decomp = seasonal_decompose(df[column], period=52, model='additive', extrapolate_trend='freq')
    df[f"{column}_trend"] = decomp.trend
    df[f"{column}_seasonal"] = decomp.seasonal
```


```python
df.columns
```




    Index(['date', 'depth_to_groundwater', 'temperature', 'drainage_volume',
           'river_hydrometry', 'rainfall', 'depth_to_groundwater_log',
           'depth_to_groundwater_diff_1', 'year', 'month', 'day', 'day_of_year',
           'week_of_year', 'quarter', 'season', 'month_sin', 'month_cos',
           'rainfall_trend', 'rainfall_seasonal', 'temperature_trend',
           'temperature_seasonal', 'drainage_volume_trend',
           'drainage_volume_seasonal', 'river_hydrometry_trend',
           'river_hydrometry_seasonal', 'depth_to_groundwater_trend',
           'depth_to_groundwater_seasonal'],
          dtype='object')




```python
fig, ax = plt.subplots(ncols=2, nrows=4, sharex=True, figsize=(16,8))

for i, column in enumerate(['temperature', 'depth_to_groundwater']):
    
    res = seasonal_decompose(df[column], period=52, model='additive', extrapolate_trend='freq')

    ax[0,i].set_title('Decomposition of {}'.format(column), fontsize=16)
    res.observed.plot(ax=ax[0,i], legend=False, color='dodgerblue')
    ax[0,i].set_ylabel('Observed', fontsize=14)

    res.trend.plot(ax=ax[1,i], legend=False, color='dodgerblue')
    ax[1,i].set_ylabel('Trend', fontsize=14)

    res.seasonal.plot(ax=ax[2,i], legend=False, color='dodgerblue')
    ax[2,i].set_ylabel('Seasonal', fontsize=14)
    
    res.resid.plot(ax=ax[3,i], legend=False, color='dodgerblue')
    ax[3,i].set_ylabel('Residual', fontsize=14)

plt.show()
```


​    ![output_52_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_52_0.png)



### 3.3 Lag

We want to calculate each variable with a shift() (lag) to compare the correlationwith the other variables.


```python
weeks_in_month = 4

for column in core_columns:
    df[f'{column}_seasonal_shift_b_2m'] = df[f'{column}_seasonal'].shift(-2 * weeks_in_month)
    df[f'{column}_seasonal_shift_b_1m'] = df[f'{column}_seasonal'].shift(-1 * weeks_in_month)
    df[f'{column}_seasonal_shift_1m'] = df[f'{column}_seasonal'].shift(1 * weeks_in_month)
    df[f'{column}_seasonal_shift_2m'] = df[f'{column}_seasonal'].shift(2 * weeks_in_month)
    df[f'{column}_seasonal_shift_3m'] = df[f'{column}_seasonal'].shift(3 * weeks_in_month)
```


```python
df.columns
```




    Index(['date', 'depth_to_groundwater', 'temperature', 'drainage_volume',
           'river_hydrometry', 'rainfall', 'depth_to_groundwater_log',
           'depth_to_groundwater_diff_1', 'year', 'month', 'day', 'day_of_year',
           'week_of_year', 'quarter', 'season', 'month_sin', 'month_cos',
           'rainfall_trend', 'rainfall_seasonal', 'temperature_trend',
           'temperature_seasonal', 'drainage_volume_trend',
           'drainage_volume_seasonal', 'river_hydrometry_trend',
           'river_hydrometry_seasonal', 'depth_to_groundwater_trend',
           'depth_to_groundwater_seasonal', 'rainfall_seasonal_shift_b_2m',
           'rainfall_seasonal_shift_b_1m', 'rainfall_seasonal_shift_1m',
           'rainfall_seasonal_shift_2m', 'rainfall_seasonal_shift_3m',
           'temperature_seasonal_shift_b_2m', 'temperature_seasonal_shift_b_1m',
           'temperature_seasonal_shift_1m', 'temperature_seasonal_shift_2m',
           'temperature_seasonal_shift_3m', 'drainage_volume_seasonal_shift_b_2m',
           'drainage_volume_seasonal_shift_b_1m',
           'drainage_volume_seasonal_shift_1m',
           'drainage_volume_seasonal_shift_2m',
           'drainage_volume_seasonal_shift_3m',
           'river_hydrometry_seasonal_shift_b_2m',
           'river_hydrometry_seasonal_shift_b_1m',
           'river_hydrometry_seasonal_shift_1m',
           'river_hydrometry_seasonal_shift_2m',
           'river_hydrometry_seasonal_shift_3m',
           'depth_to_groundwater_seasonal_shift_b_2m',
           'depth_to_groundwater_seasonal_shift_b_1m',
           'depth_to_groundwater_seasonal_shift_1m',
           'depth_to_groundwater_seasonal_shift_2m',
           'depth_to_groundwater_seasonal_shift_3m'],
          dtype='object')



## 4. Exploratory Data Analysis


```python
f, ax = plt.subplots(nrows=5, ncols=1, figsize=(15, 12))
f.suptitle('Seasonal Components of Features', fontsize=16)

for i, column in enumerate(core_columns):
    sns.lineplot(x=df['date'], y=df[column + '_seasonal'], ax=ax[i], color='dodgerblue', label='P25')
    ax[i].set_ylabel(ylabel=column, fontsize=14)
    ax[i].set_xlim([date(2017, 9, 30), date(2020, 6, 30)])
    
plt.tight_layout()
plt.show()
```


![output_58_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_58_0.png)    

    



```python
f, ax = plt.subplots(nrows=1, ncols=2, figsize=(16, 8))

corrmat = df[core_columns].corr()

sns.heatmap(corrmat, annot=True, vmin=-1, vmax=1, cmap='coolwarm', ax=ax[0])
ax[0].set_title('Correlation Matrix of Core Features', fontsize=16)

shifted_cols = [
    'depth_to_groundwater_seasonal',         
    'temperature_seasonal_shift_b_2m',
    'drainage_volume_seasonal_shift_2m', 
    'river_hydrometry_seasonal_shift_3m'
]
corrmat = df[shifted_cols].corr()

sns.heatmap(corrmat, annot=True, vmin=-1, vmax=1, cmap='coolwarm', ax=ax[1])
ax[1].set_title('Correlation Matrix of Lagged Features', fontsize=16)


plt.tight_layout()
plt.show()
```


​    ![output_59_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_59_0.png)

​    


### 4.1 Autocorrelation Analysis

ACF and PACF plots: After a time series has been stationarized by differencing, the next step in fitting an ARIMA model is to determine whether AR or MA terms are needed to correct any autocorrelation that remains in the differenced series. Of course, with software like Statgraphics, you could just try some different combinations of terms and see what works best. But there is a more systematic way to do this. By looking at the autocorrelation function (ACF) and partial autocorrelation (PACF) plots of the differenced series, you can tentatively identify the numbers of AR and/or MA terms that are needed.

- Autocorrelation Function (ACF): P = Periods to lag for eg: (if P= 3 then we will use the three previous periods of our time series in the autoregressive portion of the calculation) P helps adjust the line that is being fitted to forecast the series. P corresponds with MA parameter

- Partial Autocorrelation Function (PACF): D = In an ARIMA model we transform a time series into stationary one(series without trend or seasonality) using differencing. D refers to the number of differencing transformations required by the time series to get stationary. D corresponds with AR parameter.

Autocorrelation plots help in detecting seasonality.


```python
from pandas.plotting import autocorrelation_plot

autocorrelation_plot(df['depth_to_groundwater_diff_1'])
plt.show()
```


![output_62_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_62_0.png)    

    



```python
from statsmodels.graphics.tsaplots import plot_acf
from statsmodels.graphics.tsaplots import plot_pacf

f, ax = plt.subplots(nrows=2, ncols=1, figsize=(16, 8))

plot_acf(df['depth_to_groundwater_diff_1'], lags=100, ax=ax[0])
plot_pacf(df['depth_to_groundwater_diff_1'], lags=100, ax=ax[1])

plt.show()
```


​    ![output_63_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_63_0.png)

​    


## 5. Modeling

5. Modeling 🧩

Time series can be either univariate or multivariate:

- Univariate time series only has a single time-dependent variable.
- Multivariate time series have a multiple time-dependent variable.

But, first of all we are going to see how does cross-validation technic works in TimeSeries Analysis.


```python
from sklearn.model_selection import TimeSeriesSplit

N_SPLITS = 3

X = df['date']
y = df['depth_to_groundwater']

folds = TimeSeriesSplit(n_splits=N_SPLITS)
```


```python
f, ax = plt.subplots(nrows=N_SPLITS, ncols=2, figsize=(16, 9))

for i, (train_index, valid_index) in enumerate(folds.split(X)):
    X_train, X_valid = X[train_index], X[valid_index]
    y_train, y_valid = y[train_index], y[valid_index]

    sns.lineplot(
        x=X_train, 
        y=y_train, 
        ax=ax[i,0], 
        color='dodgerblue', 
        label='train'
    )
    sns.lineplot(
        x=X_train[len(X_train) - len(X_valid):(len(X_train) - len(X_valid) + len(X_valid))], 
        y=y_train[len(X_train) - len(X_valid):(len(X_train) - len(X_valid) + len(X_valid))], 
        ax=ax[i,1], 
        color='dodgerblue', 
        label='train'
    )

    for j in range(2):
        sns.lineplot(x= X_valid, y= y_valid, ax=ax[i, j], color='darkorange', label='validation')
    ax[i, 0].set_title(f"Rolling Window with Adjusting Training Size (Split {i+1})", fontsize=16)
    ax[i, 1].set_title(f"Rolling Window with Constant Training Size (Split {i+1})", fontsize=16)

for i in range(N_SPLITS):
    ax[i, 0].set_xlim([date(2009, 1, 1), date(2020, 6, 30)])
    ax[i, 1].set_xlim([date(2009, 1, 1), date(2020, 6, 30)])
    
plt.tight_layout()
plt.show()
```


​    ![output_67_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_67_0.png)



The idea with this plot is to understand which train and test set are we using to fit the model in each iteration.

### 5.1 Models for Univariate Time Series

5.1 Models for Univariate Time Series

First of all, we are going to analize univariate TimeSeries forecasting.

- Univariate time series: Only one variable is varying over time. For example, data collected from a sensor measuring the temperature of a room every second. Therefore, each second, you will only have a one-dimensional value, which is the temperature.


```python
train_size = int(0.85 * len(df))
test_size = len(df) - train_size

univariate_df = df[['date', 'depth_to_groundwater']].copy()
univariate_df.columns = ['ds', 'y']

train = univariate_df.iloc[:train_size, :]

x_train, y_train = pd.DataFrame(univariate_df.iloc[:train_size, 0]), pd.DataFrame(univariate_df.iloc[:train_size, 1])
x_valid, y_valid = pd.DataFrame(univariate_df.iloc[train_size:, 0]), pd.DataFrame(univariate_df.iloc[train_size:, 1])

print(len(train), len(x_valid))
```

    510 90


#### 5.1.1 Prophet

The first model (which also can handle multivariate problems) we are going to try is Facebook Prophet.

Prophet, or “Facebook Prophet,” is an open-source library for univariate (one variable) time series forecasting developed by Facebook.

Prophet implements what they refer to as an additive time series forecasting model, and the implementation supports trends, seasonality, and holidays.


```python
!pip install prophet --quiet
```


```python
from sklearn.metrics import mean_absolute_error, mean_squared_error
import math

from prophet import Prophet


# Train the model
model = Prophet()
model.fit(train)

# x_valid = model.make_future_dataframe(periods=test_size, freq='w')

# Predict on valid set
y_pred = model.predict(x_valid)

# Calcuate metrics
score_mae = mean_absolute_error(y_valid, y_pred.tail(test_size)['yhat'])
score_rmse = math.sqrt(mean_squared_error(y_valid, y_pred.tail(test_size)['yhat']))

print(Fore.GREEN + 'RMSE: {}'.format(score_rmse))
```

    INFO:prophet:Disabling weekly seasonality. Run prophet with weekly_seasonality=True to override this.
    INFO:prophet:Disabling daily seasonality. Run prophet with daily_seasonality=True to override this.
    DEBUG:cmdstanpy:input tempfile: /tmp/tmpiz72lvw3/dp_3lfk5.json
    DEBUG:cmdstanpy:input tempfile: /tmp/tmpiz72lvw3/11aw674z.json
    DEBUG:cmdstanpy:idx 0
    DEBUG:cmdstanpy:running CmdStan, num_threads: None
    DEBUG:cmdstanpy:CmdStan args: ['/usr/local/lib/python3.9/dist-packages/prophet/stan_model/prophet_model.bin', 'random', 'seed=12253', 'data', 'file=/tmp/tmpiz72lvw3/dp_3lfk5.json', 'init=/tmp/tmpiz72lvw3/11aw674z.json', 'output', 'file=/tmp/tmpiz72lvw3/prophet_modelhri1r5m1/prophet_model-20230423124655.csv', 'method=optimize', 'algorithm=lbfgs', 'iter=10000']
    12:46:55 - cmdstanpy - INFO - Chain [1] start processing
    INFO:cmdstanpy:Chain [1] start processing
    12:46:56 - cmdstanpy - INFO - Chain [1] done processing
    INFO:cmdstanpy:Chain [1] done processing


    [32mRMSE: 1.1737898100396633



```python
# Plot the forecast
f, ax = plt.subplots(1)
f.set_figheight(6)
f.set_figwidth(15)

model.plot(y_pred, ax=ax)
sns.lineplot(x=x_valid['ds'], y=y_valid['y'], ax=ax, color='orange', label='Ground truth') #navajowhite

ax.set_title(f'Prediction \n MAE: {score_mae:.2f}, RMSE: {score_rmse:.2f}', fontsize=14)
ax.set_xlabel(xlabel='Date', fontsize=14)
ax.set_ylabel(ylabel='Depth to Groundwater', fontsize=14)

plt.show()
```


​    ![output_76_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_76_0.png)
​    


#### 5.1.2 ARIMA

The second model that i want to try is ARIMA.

The Auto-Regressive Integrated Moving Average (ARIMA) model describes the autocorrelations in the data. The model assumes that the time-series is stationary. It consists of three main parts:

- Auto-Regressive (AR) filter (long term):

$y_t = c + \alpha_{1}y_{t-1} + ... + \alpha_{p}y_{t-p} + \epsilon_t = c+\sum_{i=1}^{p}\alpha_{i}y_{t-1}+\epsilon_t$
-> p

- Integration filter (stochastic trend)

-> d

- Moving Average (MA) filter (short term):

$y_t = c + \epsilon_t+\beta_1\epsilon_{t-1} + ... + \beta_{q}\epsilon_{t-q} = c + \epsilon_{t}+\sum_{i=1}^{q}\beta_i\epsilon_{t-i}$ 
-> q

ARIMA : $y_t = c+\alpha_{1}y_{t-1} + ... + \alpha_{p}y_{t-p}+\epsilon_t+\beta_{1}\epsilon_{t-1}+...+\beta_{q}\epsilon_{t-q}$

ARIMA( p, d, q)

- p: Lag order (reference PACF in Autocorrelation Analysis)
- d: Degree of differencing. (reference Differencing in Stationarity)
- q: Order of moving average (check out ACF in Autocorrelation Analysis)

Steps to analyze ARIMA
- Step 1 — Check stationarity: If a time series has a trend or seasonality component, it must be made stationary before we can use ARIMA to forecast. .
- Step 2 — Difference: If the time series is not stationary, it needs to be stationarized through differencing. Take the first difference, then check for stationarity. Take as many differences as it takes. Make sure you check seasonal differencing as well.
- Step 3 — Filter out a validation sample: This will be used to validate how accurate our model is. Use train test validation split to achieve this
- Step 4 — Select AR and MA terms: Use the ACF and PACF to decide whether to include an AR term(s), MA term(s), or both.
- Step 5 — Build the model: Build the model and set the number of periods to forecast to N (depends on your needs).
- Step 6 — Validate model: Compare the predicted values to the actuals in the validation sample.


```python
from statsmodels.tsa.arima.model import ARIMA

# Fit model
model = ARIMA(y_train, order=(1,1,1))
model_fit = model.fit()

# Prediction with ARIMA
y_pred = model_fit.forecast(90, alpha=0.05)

# Calcuate metrics
score_mae = mean_absolute_error(y_valid, y_pred)
score_rmse = math.sqrt(mean_squared_error(y_valid, y_pred))

print(Fore.GREEN + 'RMSE: {}'.format(score_rmse))
```

    [32mRMSE: 1.402615843711107



```python
y_pred.plot()
```




    <Axes: >




​    ![output_80_1](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_80_1.png)

​    



```python
model_fit.plot_diagnostics(figsize=(16,8))
plt.show()
```


​    ![output_81_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_81_0.png)

​    



```python
fig, ax = plt.subplots()

ax.plot(y_pred, label='Forecast')
ax.plot(y_valid, label='Ground truth')
ax.legend(loc='upper left')
ax.set_title("ARIMA Result")
```




    Text(0.5, 1.0, 'ARIMA Result')




​    ![output_82_1](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_82_1.png)

​    



```python
# f, ax = plt.subplots(1)
# f.set_figheight(6)
# f.set_figwidth(15)

# model_fit.plot_predict(start=1, end=599, ax=ax)
# sns.lineplot(x=x_valid.index, y=y_valid['y'], ax=ax, color='orange', label='Ground truth')

# ax.set_title(f'Prediction \n MAE: {score_mae:.2f}, RMSE: {score_rmse:.2f}', fontsize=14)
# ax.set_xlabel(xlabel='Date', fontsize=14)
# ax.set_ylabel(ylabel='Depth to Groundwater', fontsize=14)

# ax.set_ylim(-35, -18)
# plt.show()
```

#### 5.1.3 Auto-ARIMA


```python
!pip install pmdarima --quiet
```


```python
from statsmodels.tsa.arima_model import ARIMA
import pmdarima as pm

model = pm.auto_arima(y_train, start_p=1, start_q=1,
                      test='adf',       # use adftest to find optimal 'd'
                      max_p=3, max_q=3, # maximum p and q
                      m=1,              # frequency of series
                      d=None,           # let model determine 'd'
                      seasonal=False,   # No Seasonality
                      start_P=0, 
                      D=0, 
                      trace=True,
                      error_action='ignore',  
                      suppress_warnings=True, 
                      stepwise=True)

print(model.summary())
```

    Performing stepwise search to minimize aic
     ARIMA(1,1,1)(0,0,0)[0] intercept   : AIC=-631.136, Time=1.14 sec
     ARIMA(0,1,0)(0,0,0)[0] intercept   : AIC=-242.692, Time=0.17 sec
     ARIMA(1,1,0)(0,0,0)[0] intercept   : AIC=-574.047, Time=0.56 sec
     ARIMA(0,1,1)(0,0,0)[0] intercept   : AIC=-427.347, Time=0.37 sec
     ARIMA(0,1,0)(0,0,0)[0]             : AIC=-243.054, Time=0.14 sec
     ARIMA(2,1,1)(0,0,0)[0] intercept   : AIC=-629.209, Time=2.58 sec
     ARIMA(1,1,2)(0,0,0)[0] intercept   : AIC=-629.237, Time=2.10 sec
     ARIMA(0,1,2)(0,0,0)[0] intercept   : AIC=-492.779, Time=0.42 sec
     ARIMA(2,1,0)(0,0,0)[0] intercept   : AIC=-611.065, Time=0.25 sec
     ARIMA(2,1,2)(0,0,0)[0] intercept   : AIC=-628.351, Time=2.63 sec
     ARIMA(1,1,1)(0,0,0)[0]             : AIC=-632.995, Time=0.54 sec
     ARIMA(0,1,1)(0,0,0)[0]             : AIC=-428.258, Time=0.29 sec
     ARIMA(1,1,0)(0,0,0)[0]             : AIC=-575.735, Time=0.15 sec
     ARIMA(2,1,1)(0,0,0)[0]             : AIC=-631.069, Time=1.78 sec
     ARIMA(1,1,2)(0,0,0)[0]             : AIC=-631.097, Time=0.61 sec
     ARIMA(0,1,2)(0,0,0)[0]             : AIC=-494.001, Time=0.22 sec
     ARIMA(2,1,0)(0,0,0)[0]             : AIC=-612.866, Time=0.17 sec
     ARIMA(2,1,2)(0,0,0)[0]             : AIC=-630.210, Time=0.92 sec
    
    Best model:  ARIMA(1,1,1)(0,0,0)[0]          
    Total fit time: 15.143 seconds
                                   SARIMAX Results                                
    ==============================================================================
    Dep. Variable:                      y   No. Observations:                  510
    Model:               SARIMAX(1, 1, 1)   Log Likelihood                 319.497
    Date:                Sun, 23 Apr 2023   AIC                           -632.995
    Time:                        12:48:50   BIC                           -620.297
    Sample:                             0   HQIC                          -628.016
                                    - 510                                         
    Covariance Type:                  opg                                         
    ==============================================================================
                     coef    std err          z      P>|z|      [0.025      0.975]
    ------------------------------------------------------------------------------
    ar.L1          0.9196      0.021     43.766      0.000       0.878       0.961
    ma.L1         -0.4885      0.037    -13.357      0.000      -0.560      -0.417
    sigma2         0.0167      0.001     24.810      0.000       0.015       0.018
    ===================================================================================
    Ljung-Box (L1) (Q):                   0.02   Jarque-Bera (JB):               185.01
    Prob(Q):                              0.90   Prob(JB):                         0.00
    Heteroskedasticity (H):               1.17   Skew:                             0.22
    Prob(H) (two-sided):                  0.32   Kurtosis:                         5.92
    ===================================================================================
    
    Warnings:
    [1] Covariance matrix calculated using the outer product of gradients (complex-step).



```python
model.plot_diagnostics(figsize=(16,8))
plt.show()
```


​    ![output_87_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_87_0.png)

​    


As we saw in the previous Steps, AutoARIMA (auto_arima) validates that (1,1,1) is the best configuration for (p,d,q).

So how to interpret the plot diagnostics?

- Top left: The residual errors seem to fluctuate around a mean of zero and have a uniform variance between (-4, 4).

- Top Right: The density plot suggest normal distribution with mean zero.

- Bottom left: The most part of the blue dots are over the red line, so it seems that the distribution in very low skewed (not skewed for me).

- Bottom Right: The Correlogram, aka, ACF plot shows the residual errors are not autocorrelated.

#### 5.1.4 LSTM

We are going to use a multi-layered LSTM recurrent neural network to predict the last value of a sequence of values.

The following data pre-processing and feature engineering need to be done before construct the LSTM model.

- Create the dataset, ensure all data is float.
- Normalize the features.
- Split into training and test sets.
- Convert an array of values into a dataset matrix.
- Reshape into X=t and Y=t+1.
- Reshape input to be 3D (num_samples, num_timesteps, num_features).


```python
from sklearn.preprocessing import MinMaxScaler

data = univariate_df.filter(['y'])
#Convert the dataframe to a numpy array
dataset = data.values

scaler = MinMaxScaler(feature_range=(-1, 0))
scaled_data = scaler.fit_transform(dataset)

scaled_data[:10]
```




    array([[-0.81796644],
           [-0.79970385],
           [-0.7745311 ],
           [-0.74679171],
           [-0.73099704],
           [-0.71253702],
           [-0.7023692 ],
           [-0.68410661],
           [-0.66890424],
           [-0.65528134]])




```python
# Defines the rolling window
look_back = 52
# Split into train and test sets
train, test = scaled_data[:train_size-look_back,:], scaled_data[train_size-look_back:,:]

def create_dataset(dataset, look_back=1):
    X, Y = [], []
    for i in range(look_back, len(dataset)):
        a = dataset[i-look_back:i, 0]
        X.append(a)
        Y.append(dataset[i, 0])
    return np.array(X), np.array(Y)

x_train, y_train = create_dataset(train, look_back)
x_test, y_test = create_dataset(test, look_back)

# reshape input to be [samples, time steps, features]
x_train = np.reshape(x_train, (x_train.shape[0], 1, x_train.shape[1]))
x_test = np.reshape(x_test, (x_test.shape[0], 1, x_test.shape[1]))

print(len(x_train), len(x_test))
```

    406 90



```python
from keras.models import Sequential
from keras.layers import Dense, LSTM

#Build the LSTM model
model = Sequential()
model.add(LSTM(128, return_sequences=True, input_shape=(x_train.shape[1], x_train.shape[2])))
model.add(LSTM(64, return_sequences=False))
model.add(Dense(25))
model.add(Dense(1))

# Compile the model
model.compile(optimizer='adam', loss='mean_squared_error')

#Train the model
model.fit(x_train, y_train, batch_size=1, epochs=10, validation_data=(x_test, y_test))

model.summary()
```

    Epoch 1/10
    406/406 [==============================] - 7s 9ms/step - loss: 0.0085 - val_loss: 0.0024
    Epoch 2/10
    406/406 [==============================] - 3s 7ms/step - loss: 0.0020 - val_loss: 0.0029
    Epoch 3/10
    406/406 [==============================] - 2s 5ms/step - loss: 0.0027 - val_loss: 0.0032
    Epoch 4/10
    406/406 [==============================] - 2s 6ms/step - loss: 0.0011 - val_loss: 3.3838e-04
    Epoch 5/10
    406/406 [==============================] - 2s 5ms/step - loss: 0.0014 - val_loss: 3.9864e-04
    Epoch 6/10
    406/406 [==============================] - 2s 5ms/step - loss: 0.0016 - val_loss: 5.4464e-04
    Epoch 7/10
    406/406 [==============================] - 3s 7ms/step - loss: 0.0020 - val_loss: 3.9537e-04
    Epoch 8/10
    406/406 [==============================] - 3s 8ms/step - loss: 0.0010 - val_loss: 6.5017e-04
    Epoch 9/10
    406/406 [==============================] - 2s 5ms/step - loss: 8.9141e-04 - val_loss: 0.0036
    Epoch 10/10
    406/406 [==============================] - 2s 6ms/step - loss: 9.3876e-04 - val_loss: 2.3816e-04
    Model: "sequential_6"
    _________________________________________________________________
     Layer (type)                Output Shape              Param #   
    =================================================================
     lstm_10 (LSTM)              (None, 1, 128)            92672     
                                                                     
     lstm_11 (LSTM)              (None, 64)                49408     
                                                                     
     dense_10 (Dense)            (None, 25)                1625      
                                                                     
     dense_11 (Dense)            (None, 1)                 26        
                                                                     
    =================================================================
    Total params: 143,731
    Trainable params: 143,731
    Non-trainable params: 0
    _________________________________________________________________



```python
# Lets predict with the model
train_predict = model.predict(x_train)
test_predict = model.predict(x_test)

# invert predictions
train_predict = scaler.inverse_transform(train_predict)
y_train = scaler.inverse_transform([y_train])

test_predict = scaler.inverse_transform(test_predict)
y_test = scaler.inverse_transform([y_test])

# Get the root mean squared error (RMSE) and MAE
score_rmse = np.sqrt(mean_squared_error(y_test[0], test_predict[:,0]))
score_mae = mean_absolute_error(y_test[0], test_predict[:,0])
print(Fore.GREEN + 'RMSE: {}'.format(score_rmse))
```

    13/13 [==============================] - 1s 3ms/step
    3/3 [==============================] - 0s 6ms/step
    [32mRMSE: 0.2233293828264469



```python
fig, ax = plt.subplots()

ax.plot(test_predict[:,0], label='Forecast')
ax.plot(y_test[0], label='Ground truth')
ax.legend(loc='upper left')
ax.set_title("LSTM Result")
```




    Text(0.5, 1.0, 'LSTM Result')



​    ![output_95_1](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_95_1.png)

​    


### 5.2 Models for Multivariate Time Series

Finnally, we are going to analize multivariate TimeSeries forecasting.

Multivariate time series: Multiple variables are varying over time. For example, a tri-axial accelerometer. There are three accelerations, one for each axis (x,y,z) and they vary simultaneously over time.


```python
feature_columns = [
    'rainfall',
    'temperature',
    'drainage_volume',
    'river_hydrometry',
]
target_column = ['depth_to_groundwater']

train_size = int(0.85 * len(df))

multivariate_df = df[['date'] + target_column + feature_columns].copy()
multivariate_df.columns = ['ds', 'y'] + feature_columns

train = multivariate_df.iloc[:train_size, :]
x_train, y_train = pd.DataFrame(multivariate_df.iloc[:train_size, [0,2,3,4,5]]), pd.DataFrame(multivariate_df.iloc[:train_size, 1])
x_valid, y_valid = pd.DataFrame(multivariate_df.iloc[train_size:, [0,2,3,4,5]]), pd.DataFrame(multivariate_df.iloc[train_size:, 1])

train.head()
```





  <div id="df-ca56624d-c329-491f-b2e2-fea1433b3289">
    <div class="colab-df-container">
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
      <th>ds</th>
      <th>y</th>
      <th>rainfall</th>
      <th>temperature</th>
      <th>drainage_volume</th>
      <th>river_hydrometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2009-01-01</td>
      <td>-31.048571</td>
      <td>0.000000</td>
      <td>1.657143</td>
      <td>-28164.918857</td>
      <td>2.371429</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2009-01-08</td>
      <td>-30.784286</td>
      <td>0.285714</td>
      <td>4.571429</td>
      <td>-29755.789714</td>
      <td>2.314286</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2009-01-15</td>
      <td>-30.420000</td>
      <td>0.028571</td>
      <td>7.528571</td>
      <td>-25463.190857</td>
      <td>2.300000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2009-01-22</td>
      <td>-30.018571</td>
      <td>0.585714</td>
      <td>6.214286</td>
      <td>-23854.422857</td>
      <td>2.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2009-01-29</td>
      <td>-29.790000</td>
      <td>1.414286</td>
      <td>5.771429</td>
      <td>-25210.532571</td>
      <td>2.500000</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-ca56624d-c329-491f-b2e2-fea1433b3289')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }
    
    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }
    
    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }
    
    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-ca56624d-c329-491f-b2e2-fea1433b3289 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';
    
        async function convertToInteractive(key) {
          const element = document.querySelector('#df-ca56624d-c329-491f-b2e2-fea1433b3289');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;
    
          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>




#### 5.2.1 Multivariate Prophet


```python
from prophet import Prophet


# Train the model
model = Prophet()
model.add_regressor('rainfall')
model.add_regressor('temperature')
model.add_regressor('drainage_volume')
model.add_regressor('river_hydrometry')

# Fit the model with train set
model.fit(train)

# Predict on valid set
y_pred = model.predict(x_valid)

# Calcuate metrics
score_mae = mean_absolute_error(y_valid, y_pred['yhat'])
score_rmse = math.sqrt(mean_squared_error(y_valid, y_pred['yhat']))

print(Fore.GREEN + 'RMSE: {}'.format(score_rmse))
```

    INFO:prophet:Disabling weekly seasonality. Run prophet with weekly_seasonality=True to override this.
    INFO:prophet:Disabling daily seasonality. Run prophet with daily_seasonality=True to override this.
    DEBUG:cmdstanpy:input tempfile: /tmp/tmpiz72lvw3/kv3cqdzg.json
    DEBUG:cmdstanpy:input tempfile: /tmp/tmpiz72lvw3/u34z6wgv.json
    DEBUG:cmdstanpy:idx 0
    DEBUG:cmdstanpy:running CmdStan, num_threads: None
    DEBUG:cmdstanpy:CmdStan args: ['/usr/local/lib/python3.9/dist-packages/prophet/stan_model/prophet_model.bin', 'random', 'seed=422', 'data', 'file=/tmp/tmpiz72lvw3/kv3cqdzg.json', 'init=/tmp/tmpiz72lvw3/u34z6wgv.json', 'output', 'file=/tmp/tmpiz72lvw3/prophet_modelrmldq8w6/prophet_model-20230423125057.csv', 'method=optimize', 'algorithm=lbfgs', 'iter=10000']
    12:50:57 - cmdstanpy - INFO - Chain [1] start processing
    INFO:cmdstanpy:Chain [1] start processing
    12:50:58 - cmdstanpy - INFO - Chain [1] done processing
    INFO:cmdstanpy:Chain [1] done processing


    [32mRMSE: 1.0144259311109194



```python
# Plot the forecast
f, ax = plt.subplots(1)
f.set_figheight(6)
f.set_figwidth(15)

model.plot(y_pred, ax=ax)
sns.lineplot(x=x_valid['ds'], y=y_valid['y'], ax=ax, color='orange', label='Ground truth') #navajowhite

ax.set_title(f'Prediction \n MAE: {score_mae:.2f}, RMSE: {score_rmse:.2f}', fontsize=14)
ax.set_xlabel(xlabel='Date', fontsize=14)
ax.set_ylabel(ylabel='Depth to Groundwater', fontsize=14)

plt.show()
```


​    ![output_101_0](/images/2023-04-23-TimeSeries Analysis Guide(Acea Smart Water)/output_101_0.png)



## 6. Conclusion

The best results are taken from Univariate LSTM (with rolling window of 1 year) and multi-variate Prophet.

## 7. References

Here I am going to reference some useful links that I have used to build this notebook

Special reference for the helpful information and plots - https://www.kaggle.com/iamleonie/intro-to-time-series-forecasting

ARIMA - https://towardsdatascience.com/time-series-forecasting-arima-models-7f221e9eee06

Auto-ARIMA - https://www.machinelearningplus.com/time-series/arima-model-time-series-forecasting-python/

Keras LSTM - https://machinelearningmastery.com/time-series-prediction-lstm-recurrent-neural-networks-python-keras/

Prophet - https://towardsdatascience.com/time-series-prediction-using-prophet-in-python-35d65f626236

Special reference - https://www.kaggle.com/iamleonie/intro-to-time-series-forecasting/notebook#Models

Cyclical features - https://towardsdatascience.com/cyclical-features-encoding-its-about-time-ce23581845ca

ADF - https://medium.com/@cmukesh8688/why-is-augmented-dickey-fuller-test-adf-test-so-important-in-time-series-analysis-6fc97c6be2f0

ACF/PACF - https://towardsdatascience.com/significance-of-acf-and-pacf-plots-in-time-series-analysis-2fa11a5d10a8

LSTM - https://towardsdatascience.com/time-series-analysis-visualization-forecasting-with-lstm-77a905180eba
