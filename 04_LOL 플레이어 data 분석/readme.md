
### 경희대학교 산업경영공학과 2015100915 김태호

# LOL 플레이어의 실력 분석

BeautifulSoup와 urllib, selenium을 사용하여 플레이어의 데이터가 있는 op.gg에서 데이터를 가져와 LOL플레이어의 최근 100경기의 데이터 분석한다.

### 사용방법
- 코드를 실행하고 아래에 "소환사명을 입력해주세요.: "가 나오면 소환사명을 입력해 주세요.(ex. 황토매트)


```python
username = input("소환사명을 입력해주세요.: ")
```

    소환사명을 입력해주세요.: 황토매트
    


```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
import pandas as pd
from tqdm import tqdm_notebook
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
from matplotlib import font_manager, rc
path = "c:/Windows/Fonts/malgun.ttf"
font_name = font_manager.FontProperties(fname=path).get_name()
rc('font', family=font_name)
import seaborn as sns
from selenium import webdriver
import urllib.request
import urllib.parse
import time
```


```python
url1 = "http://www.op.gg/summoner/userName=" #한글을 제외한 주소를 url1에 저장
#parse모듈을 사용해 한글부분을 유니코드로 치환
url2 = urllib.parse.quote_plus(str(username)) 
url = url1 + url2
driver = webdriver.Chrome('chromedriver.exe')
driver.get(url)
```

### op.gg에 서머너 이름 검색 후 최근전적부터 100번의 게임정보 가져오기


```python
xpath = """//*[@id="SummonerRefreshButton"]"""
driver.find_element_by_xpath(xpath).click() # 최근전적으로 갱신하기
time.sleep(10)
while True:
    try:
        xpath = """//*[@id="SummonerLayoutContent"]/div[2]/div[2]/div/div[2]/div[4]/a"""
        driver.find_element_by_xpath(xpath).click() # 전적 추가로 보기 버튼 누르기
        break
    except:
        continue
while True:
    try:
        xpath = """//*[@id="SummonerLayoutContent"]/div[2]/div[2]/div/div[2]/div[5]/a"""
        driver.find_element_by_xpath(xpath).click() # 전적 추가로 보기 버튼 누르기
        break
    except:
        continue
while True:
    try:
        xpath = """//*[@id="SummonerLayoutContent"]/div[2]/div[2]/div/div[2]/div[6]/a"""
        driver.find_element_by_xpath(xpath).click() # 전적 추가로 보기 버튼 누르기
        break
    except:
        continue
while True:
    try:
        xpath = """//*[@id="SummonerLayoutContent"]/div[2]/div[2]/div/div[2]/div[7]/a"""
        driver.find_element_by_xpath(xpath).click() # 전적 추가로 보기 버튼 누르기
        break 
    except:
        continue        
time.sleep(5)
```

html 모두 긁어오기


```python
page = driver.page_source
soup = BeautifulSoup(page, "html.parser")

# print(soup.prettify()) 를 통해 확인할 수 있다
```

## 랭크게임 티어와 랭킹알아보기


```python
import re #정규식
from IPython.display import Image
from IPython.core.display import HTML #이미지 출력
TierImage = soup.find('div','Medal').find('img','Image')
TierImage = TierImage['src']
TierImage = "https:" + TierImage
soup.find('div','Medal') #티어 이미지
try:
    soup.find('div','LadderRank') #래더 랭크
    LadderRank = soup.find('div','LadderRank').get_text()
    LadderRank = re.split(('\n|\t'), LadderRank)
    LadderRank = "".join(LadderRank)
except:
    LadderRank = '랭크기록이 없습니다.'
soup.find('div','SummonerRatingMedium') #랭크게임 정보
SummonerRatingMedium = soup.find('div','SummonerRatingMedium').get_text()
SummonerRatingMedium = re.split(('\n|\t'), SummonerRatingMedium)
SummonerRatingMedium = " ".join(SummonerRatingMedium)
SummonerRatinglist = SummonerRatingMedium.strip().split('/')
```


```python
print("===>> 소환사명 : ", username)
print("===>>", LadderRank)
print("===>>", SummonerRatinglist[0])
print("===>>", SummonerRatinglist[1].strip())
Image(url= TierImage)
```

    ===>> 소환사명 :  황토매트
    ===>> 래더 랭킹 1,738,557위 (상위 58%)
    ===>> 솔로랭크 Silver 3       8 LP            
    ===>> 146승 164패  승률 47%       직스의 칼날단
    




<img src="https://opgg-static.akamaized.net/images/medals/silver_3.png?image=q_auto&v=1"/>



## 모스트 챔피언 분석하기


```python
soup.find('div','MostChampionContent')

Champ = [] ; CS = [] ; KDARatio = [] ; tmp = [] ; K = [] ; D = [] ; A = [] ; WinRatio = [] ; PlayedGames = []
span = ['Kill','Death','Assist','KDA']
dict_tmp = {'ChampionName':Champ,'ChampionMinionKill':CS,'KDA':KDARatio,'Kill':K,'Death':D,'Assist':A, 
            'WinRatio':WinRatio,'Title':PlayedGames}

for x,y in dict_tmp.items():
    if x in span:
        tmp = soup.find('div','MostChampionContent').find_all('span',x)
    else:
        tmp = soup.find('div','MostChampionContent').find_all('div',x)
    for i in tmp:
        tmp = re.split(('\n|\t'),i.get_text())
        tmp = [n for n in tmp if n][0]
        y.append(tmp)
    y
```


```python
MostChampionContent_df = {'Champ':Champ, 'CS':CS, 'KDARatio':KDARatio, 'K':K,'D':D,'A':A
                          , 'WinRatio':WinRatio, 'PlayedGames':PlayedGames}
MCC_df = pd.DataFrame(MostChampionContent_df)
MCC_df.set_index('Champ',inplace=True)
print('가장 많이 플레이한 챔피언')
MCC_df
```

    가장 많이 플레이한 챔피언
    




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
      <th>CS</th>
      <th>KDARatio</th>
      <th>K</th>
      <th>D</th>
      <th>A</th>
      <th>WinRatio</th>
      <th>PlayedGames</th>
    </tr>
    <tr>
      <th>Champ</th>
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
      <th>세트</th>
      <td>CS 138.0 (5.2)</td>
      <td>2.41:1</td>
      <td>4.0</td>
      <td>4.1</td>
      <td>5.8</td>
      <td>58%</td>
      <td>48 게임</td>
    </tr>
    <tr>
      <th>바이</th>
      <td>CS 124.3 (4.8)</td>
      <td>2.30:1</td>
      <td>5.2</td>
      <td>5.1</td>
      <td>6.5</td>
      <td>50%</td>
      <td>38 게임</td>
    </tr>
    <tr>
      <th>미스 포츈</th>
      <td>CS 210.9 (8.2)</td>
      <td>3.89:1</td>
      <td>8.4</td>
      <td>3.9</td>
      <td>6.8</td>
      <td>78%</td>
      <td>23 게임</td>
    </tr>
    <tr>
      <th>직스</th>
      <td>CS 164.9 (5.7)</td>
      <td>2.41:1</td>
      <td>3.5</td>
      <td>4.6</td>
      <td>7.6</td>
      <td>38%</td>
      <td>21 게임</td>
    </tr>
    <tr>
      <th>진</th>
      <td>CS 213.2 (7.3)</td>
      <td>3.96:1</td>
      <td>6.3</td>
      <td>3.9</td>
      <td>9.3</td>
      <td>53%</td>
      <td>17 게임</td>
    </tr>
    <tr>
      <th>레오나</th>
      <td>CS 34.8 (1.3)</td>
      <td>3.91:1</td>
      <td>2.4</td>
      <td>3.3</td>
      <td>10.6</td>
      <td>75%</td>
      <td>16 게임</td>
    </tr>
    <tr>
      <th>아트록스</th>
      <td>CS 147.3 (4.8)</td>
      <td>1.48:1</td>
      <td>3.5</td>
      <td>6.3</td>
      <td>5.8</td>
      <td>43%</td>
      <td>14 게임</td>
    </tr>
  </tbody>
</table>
</div>




```python
KDARatio_fl = [] ; WinRatio_fl = [] ; PlayedGames_fl = []
for i in KDARatio:
    tmp = re.split((':1|%'),i)
    tmp = [n for n in tmp if n][0]
    KDARatio_fl.append(tmp)
for i in WinRatio:
    tmp = re.split((':1|%'),i)
    tmp = [n for n in tmp if n][0]
    WinRatio_fl.append(tmp)
for i in PlayedGames:
    tmp = re.split((' 게임'),i)
    tmp = [n for n in tmp if n][0]
    PlayedGames_fl.append(tmp)
CS_new = []
n = 0
while n < len(list(MCC_df.index)):
    sw = 0
    k = ''
    tmp = MCC_df['CS'][n]
    for i in tmp:
        if i == '(':
            sw = 1
        if sw == 1:
            k += i
    CS_new.append(k.strip('(|)'))
    n += 1

ForGragh = {'Champ':Champ,'CS':CS_new,'KDARatio':KDARatio_fl, 'WinRatio':WinRatio_fl, 'PlayedGames':PlayedGames_fl}
ForGragh = pd.DataFrame(ForGragh)
ForGragh['KDARatio'] = ForGragh['KDARatio'].astype(float)
ForGragh['WinRatio'] = ForGragh['WinRatio'].astype(float)
ForGragh['PlayedGames'] = ForGragh['PlayedGames'].astype(float)
ForGragh['CS'] = ForGragh['CS'].astype(float)
ForGragh = ForGragh.set_index('Champ').sort_values(by=['PlayedGames'], axis=0, ascending=False)
plt.figure()
plt.subplot(311)
ForGragh['KDARatio'].plot(kind='barh', grid=True, figsize=(15,15))
plt.xlabel('KDARatio')
plt.subplot(312)
ForGragh['WinRatio'].plot(kind='barh', grid=True, figsize=(15,15))
plt.xlabel('WinRatio')
plt.subplot(313)
ForGragh['CS'].plot(kind='barh', grid=True, figsize=(15,15))
plt.xlabel('CS')
plt.show()
```


![png](%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_files/%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_17_0.png)


## 최근 100경기 분석하기


```python
soup.find('div','GameItemList').find_all('div','GameItemWrap')
Champ = [] ; GameType = [] ; CS = [] ; KDARatio_ = [] ; tmp = [] ; K = [] ; D = [] ; A = [] 
CKRate_ = [] ; GameResult = [] 
span = ['Kill','Death','Assist','KDARatio','CS','Wards']
dict_tmp = {'ChampionName':Champ,'GameType':GameType,'CS':CS,'KDARatio':KDARatio_,'Kill':K,'Death':D,'Assist':A, 
            'CKRate' : CKRate_,'GameResult':GameResult}

for x,y in dict_tmp.items():
    if x in span:
        tmp = soup.find('div','RealContent').find_all('span',x)
    else:
        tmp = soup.find('div','RealContent').find_all('div',x)
    for i in tmp:
        tmp = re.split(('\n|\t'),i.get_text())
        tmp = [n for n in tmp if n][0]
        y.append(tmp)
tmp = []
for i in K:
    if i < '9999999':
        tmp.append(i)
K = tmp        
KDARatio_ = KDARatio_[1:]
K = K[1:]
D = D[1:]
A = A[1:]

Recent_Games = {'ChampionName':Champ,'GameType':GameType,'CS':CS,'KDARatio':KDARatio_,'K':K,
                          'D':D,'A':A, 'CKRate' : CKRate_,'GameResult':GameResult}
Recent_Games = pd.DataFrame(Recent_Games)
Recent_Games.set_index('ChampionName',inplace=True)
# 분석하기 쉽게 전처리
CS_min = []
n = 0
while n < len(list(Recent_Games.index)):
    sw = 0
    k = ''
    tmp = CS[n]
    for i in tmp:
        if i == '(':
            sw = 1
        if sw == 1:
            k += i
    CS_min.append(k.strip('(|)'))
    n += 1
KDARatio = []
for i in KDARatio_:
    tmp = re.split((':1'),i)
    KDARatio.append(tmp[0])
CKRate = []
for i in CKRate_:
    tmp = re.split(('%|킬관여 '),i)
    CKRate.append(tmp[1])

    Recent_Games2 = {'ChampionName':Champ,'GameType':GameType,'CS_min':CS_min,'KDARatio':KDARatio,'K':K,
                          'D':D,'A':A, 'CKRate' : CKRate,'GameResult':GameResult}
Recent_Games2 = pd.DataFrame(Recent_Games2)
Recent_Games2.set_index('ChampionName',inplace=True)

Recent_Games2['CS_min'] =Recent_Games2['CS_min'].astype(float) 
Recent_Games2['CKRate'] =Recent_Games2['CKRate'].astype(float) 
Recent_Games2['K'] =Recent_Games2['K'].astype(float) 
Recent_Games2['D'] =Recent_Games2['D'].astype(float) 
Recent_Games2['A'] =Recent_Games2['A'].astype(float) 
print('최근 100경기')
Recent_Games2.head()
```

    최근 100경기
    




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
      <th>GameType</th>
      <th>CS_min</th>
      <th>KDARatio</th>
      <th>K</th>
      <th>D</th>
      <th>A</th>
      <th>CKRate</th>
      <th>GameResult</th>
    </tr>
    <tr>
      <th>ChampionName</th>
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
      <th>아트록스</th>
      <td>솔랭</td>
      <td>3.3</td>
      <td>2.00</td>
      <td>7.0</td>
      <td>5.0</td>
      <td>3.0</td>
      <td>42.0</td>
      <td>패배</td>
    </tr>
    <tr>
      <th>바이</th>
      <td>솔랭</td>
      <td>3.9</td>
      <td>1.60</td>
      <td>3.0</td>
      <td>10.0</td>
      <td>13.0</td>
      <td>59.0</td>
      <td>패배</td>
    </tr>
    <tr>
      <th>진</th>
      <td>솔랭</td>
      <td>7.4</td>
      <td>7.67</td>
      <td>12.0</td>
      <td>3.0</td>
      <td>11.0</td>
      <td>49.0</td>
      <td>승리</td>
    </tr>
    <tr>
      <th>아트록스</th>
      <td>솔랭</td>
      <td>5.1</td>
      <td>2.20</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>8.0</td>
      <td>31.0</td>
      <td>승리</td>
    </tr>
    <tr>
      <th>아트록스</th>
      <td>솔랭</td>
      <td>4.9</td>
      <td>0.43</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>50.0</td>
      <td>패배</td>
    </tr>
  </tbody>
</table>
</div>



### 100경기 평균 기록


```python
Total_vic = list(Recent_Games2['GameResult']).count('승리')
Total_def = list(Recent_Games2['GameResult']).count('패배')
Total_wr = Total_vic/(Total_vic+Total_def)*100

Total_wr = round(Total_wr,2)
Total_KDA = round((Recent_Games2['K'].sum()+Recent_Games2['A'].sum())/Recent_Games2['D'].sum(),2)
Total_cs = round(Recent_Games2['CS_min'].mean(),2)
Total_CKRate = round(Recent_Games2['CKRate'].mean(),2)
print("===>> 최근 100경기 승률 : {}%\n===>> 평점 : {}점\n===>> 평균 분당cs : {}개\n===>> 평균 킬관여율 : {}%"
      .format(Total_wr,Total_KDA,Total_cs,Total_CKRate))
```

    ===>> 최근 100경기 승률 : 41.41%
    ===>> 평점 : 2.03점
    ===>> 평균 분당cs : 4.06개
    ===>> 평균 킬관여율 : 44.85%
    

### 챔피언별 기록


```python
grouped = Recent_Games2[['CS_min','K','D','A','CKRate']].groupby(level = 0) 
Recent_Games3 = grouped.mean()
Recent_Games3['KDA'] = (Recent_Games3['K']+Recent_Games3['A'])/Recent_Games3['D']
Recent_Games3 = Recent_Games3.applymap(lambda x : round(x,2)) 
Recent_Games3['PlayedGames'] = grouped.count()['CS_min']
```


```python
# GameResult 와 index를 key로 만든후 카운트 한다.
grouped2 = Recent_Games2[['K']].groupby([Recent_Games2.index,Recent_Games2['GameResult']]) 
grouped3 = grouped2.count()

WinRatio = []
for tmp in Recent_Games3.index:
    try:
        grouped_vic = grouped3.loc[(tmp,'승리'), 'K'] 
    except:
        grouped_vic = 0
    try:
        grouped_def = grouped3.loc[(tmp,'패배'), 'K'] 
    except:
        grouped_def = 0
    grouped_wr = grouped_vic/(grouped_vic+grouped_def)*100
    grouped_wr = round(grouped_wr,2)
    WinRatio.append(grouped_wr)
Recent_Games3['WinRatio'] = WinRatio
Recent_Games3 = Recent_Games3[Recent_Games3['PlayedGames'] >= 5].sort_values(by=['PlayedGames'], axis=0, ascending=False)
Recent_Games3
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
      <th>CS_min</th>
      <th>K</th>
      <th>D</th>
      <th>A</th>
      <th>CKRate</th>
      <th>KDA</th>
      <th>PlayedGames</th>
      <th>WinRatio</th>
    </tr>
    <tr>
      <th>ChampionName</th>
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
      <th>바이</th>
      <td>4.84</td>
      <td>5.50</td>
      <td>5.73</td>
      <td>6.77</td>
      <td>48.91</td>
      <td>2.14</td>
      <td>22</td>
      <td>50.00</td>
    </tr>
    <tr>
      <th>아트록스</th>
      <td>4.69</td>
      <td>3.33</td>
      <td>5.87</td>
      <td>5.40</td>
      <td>37.80</td>
      <td>1.49</td>
      <td>15</td>
      <td>42.86</td>
    </tr>
    <tr>
      <th>블리츠크랭크</th>
      <td>1.17</td>
      <td>2.75</td>
      <td>4.83</td>
      <td>11.25</td>
      <td>55.92</td>
      <td>2.90</td>
      <td>12</td>
      <td>50.00</td>
    </tr>
    <tr>
      <th>조이</th>
      <td>4.38</td>
      <td>4.00</td>
      <td>7.44</td>
      <td>5.00</td>
      <td>39.11</td>
      <td>1.21</td>
      <td>9</td>
      <td>11.11</td>
    </tr>
    <tr>
      <th>세트</th>
      <td>5.01</td>
      <td>5.88</td>
      <td>5.25</td>
      <td>8.00</td>
      <td>39.50</td>
      <td>2.64</td>
      <td>8</td>
      <td>50.00</td>
    </tr>
    <tr>
      <th>노틸러스</th>
      <td>1.33</td>
      <td>2.50</td>
      <td>5.50</td>
      <td>9.17</td>
      <td>48.33</td>
      <td>2.12</td>
      <td>6</td>
      <td>33.33</td>
    </tr>
    <tr>
      <th>미스 포츈</th>
      <td>7.78</td>
      <td>6.40</td>
      <td>5.40</td>
      <td>8.00</td>
      <td>51.20</td>
      <td>2.67</td>
      <td>5</td>
      <td>60.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure()
plt.subplot(311)
Recent_Games3['PlayedGames'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('PlayedGames')
plt.subplot(323)
Recent_Games3['WinRatio'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('WinRatio')
plt.subplot(324)
Recent_Games3['KDA'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('KDA')
plt.subplot(325)
Recent_Games3['CKRate'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('킬관여율')
plt.subplot(326)
Recent_Games3['CS_min'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('CS_min')
plt.show()
```


![png](%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_files/%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_25_0.png)


# 분석 결과

## 모스트챔피언 분석


```python
from IPython.core.interactiveshell import InteractiveShell 
InteractiveShell.ast_node_interactivity = "all"
print("===>> 소환사명 : ", username)
Image(url= TierImage)
print("===>>", LadderRank)
print("===>>", SummonerRatinglist[0])
print("===>>", SummonerRatinglist[1].strip())

print('가장 많이 플레이한 챔피언')
MCC_df
```

    ===>> 소환사명 :  황토매트
    




<img src="https://opgg-static.akamaized.net/images/medals/silver_3.png?image=q_auto&v=1"/>



    ===>> 래더 랭킹 1,738,557위 (상위 58%)
    ===>> 솔로랭크 Silver 3       8 LP            
    ===>> 146승 164패  승률 47%       직스의 칼날단
    가장 많이 플레이한 챔피언
    




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
      <th>CS</th>
      <th>KDARatio</th>
      <th>K</th>
      <th>D</th>
      <th>A</th>
      <th>WinRatio</th>
      <th>PlayedGames</th>
    </tr>
    <tr>
      <th>Champ</th>
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
      <th>세트</th>
      <td>CS 138.0 (5.2)</td>
      <td>2.41:1</td>
      <td>4.0</td>
      <td>4.1</td>
      <td>5.8</td>
      <td>58%</td>
      <td>48 게임</td>
    </tr>
    <tr>
      <th>바이</th>
      <td>CS 124.3 (4.8)</td>
      <td>2.30:1</td>
      <td>5.2</td>
      <td>5.1</td>
      <td>6.5</td>
      <td>50%</td>
      <td>38 게임</td>
    </tr>
    <tr>
      <th>미스 포츈</th>
      <td>CS 210.9 (8.2)</td>
      <td>3.89:1</td>
      <td>8.4</td>
      <td>3.9</td>
      <td>6.8</td>
      <td>78%</td>
      <td>23 게임</td>
    </tr>
    <tr>
      <th>직스</th>
      <td>CS 164.9 (5.7)</td>
      <td>2.41:1</td>
      <td>3.5</td>
      <td>4.6</td>
      <td>7.6</td>
      <td>38%</td>
      <td>21 게임</td>
    </tr>
    <tr>
      <th>진</th>
      <td>CS 213.2 (7.3)</td>
      <td>3.96:1</td>
      <td>6.3</td>
      <td>3.9</td>
      <td>9.3</td>
      <td>53%</td>
      <td>17 게임</td>
    </tr>
    <tr>
      <th>레오나</th>
      <td>CS 34.8 (1.3)</td>
      <td>3.91:1</td>
      <td>2.4</td>
      <td>3.3</td>
      <td>10.6</td>
      <td>75%</td>
      <td>16 게임</td>
    </tr>
    <tr>
      <th>아트록스</th>
      <td>CS 147.3 (4.8)</td>
      <td>1.48:1</td>
      <td>3.5</td>
      <td>6.3</td>
      <td>5.8</td>
      <td>43%</td>
      <td>14 게임</td>
    </tr>
  </tbody>
</table>
</div>



### 그래프


```python
InteractiveShell.ast_node_interactivity = "last"
plt.figure()
plt.subplot(311)
ForGragh['KDARatio'].plot(kind='barh', grid=True, figsize=(15,15))
plt.xlabel('KDARatio')
plt.subplot(312)
ForGragh['WinRatio'].plot(kind='barh', grid=True, figsize=(15,15))
plt.xlabel('WinRatio')
plt.subplot(313)
ForGragh['CS'].plot(kind='barh', grid=True, figsize=(15,15))
plt.xlabel('CS')
plt.show()
```


![png](%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_files/%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_30_0.png)

## 최근 100경기 분석(5판이상 플레이한 챔피언 기준)

```python
InteractiveShell.ast_node_interactivity = "all"
print("===>> 최근 100경기 분석(5판이상 플레이한 챔피언 기준)")
Recent_Games3
print("===>> 최근 100경기 승률 : {}%\n===>> 평점 : {}점\n===>> 평균 분당cs : {}개\n===>> 평균 킬관여율 : {}%"
      .format(Total_wr,Total_KDA,Total_cs,Total_CKRate))
```

    ===>> 최근 100경기 분석(5판이상 플레이한 챔피언 기준)
    




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
      <th>CS_min</th>
      <th>K</th>
      <th>D</th>
      <th>A</th>
      <th>CKRate</th>
      <th>KDA</th>
      <th>PlayedGames</th>
      <th>WinRatio</th>
    </tr>
    <tr>
      <th>ChampionName</th>
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
      <th>바이</th>
      <td>4.84</td>
      <td>5.50</td>
      <td>5.73</td>
      <td>6.77</td>
      <td>48.91</td>
      <td>2.14</td>
      <td>22</td>
      <td>50.00</td>
    </tr>
    <tr>
      <th>아트록스</th>
      <td>4.69</td>
      <td>3.33</td>
      <td>5.87</td>
      <td>5.40</td>
      <td>37.80</td>
      <td>1.49</td>
      <td>15</td>
      <td>42.86</td>
    </tr>
    <tr>
      <th>블리츠크랭크</th>
      <td>1.17</td>
      <td>2.75</td>
      <td>4.83</td>
      <td>11.25</td>
      <td>55.92</td>
      <td>2.90</td>
      <td>12</td>
      <td>50.00</td>
    </tr>
    <tr>
      <th>조이</th>
      <td>4.38</td>
      <td>4.00</td>
      <td>7.44</td>
      <td>5.00</td>
      <td>39.11</td>
      <td>1.21</td>
      <td>9</td>
      <td>11.11</td>
    </tr>
    <tr>
      <th>세트</th>
      <td>5.01</td>
      <td>5.88</td>
      <td>5.25</td>
      <td>8.00</td>
      <td>39.50</td>
      <td>2.64</td>
      <td>8</td>
      <td>50.00</td>
    </tr>
    <tr>
      <th>노틸러스</th>
      <td>1.33</td>
      <td>2.50</td>
      <td>5.50</td>
      <td>9.17</td>
      <td>48.33</td>
      <td>2.12</td>
      <td>6</td>
      <td>33.33</td>
    </tr>
    <tr>
      <th>미스 포츈</th>
      <td>7.78</td>
      <td>6.40</td>
      <td>5.40</td>
      <td>8.00</td>
      <td>51.20</td>
      <td>2.67</td>
      <td>5</td>
      <td>60.00</td>
    </tr>
  </tbody>
</table>
</div>



    ===>> 최근 100경기 승률 : 41.41%
    ===>> 평점 : 2.03점
    ===>> 평균 분당cs : 4.06개
    ===>> 평균 킬관여율 : 44.85%
    

## 그래프


```python
InteractiveShell.ast_node_interactivity = "last"
plt.figure()
plt.subplot(311)
Recent_Games3['PlayedGames'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('PlayedGames')
plt.subplot(323)
Recent_Games3['WinRatio'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('WinRatio')
plt.subplot(324)
Recent_Games3['KDA'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('KDA')
plt.subplot(325)
Recent_Games3['CKRate'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('킬관여율')
plt.subplot(326)
Recent_Games3['CS_min'].plot(kind='barh', grid=True, figsize=(15,20))
plt.ylabel('CS_min')
plt.show()
```


![png](%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_files/%EB%A6%AC%EA%B7%B8%EC%98%A4%EB%B8%8C%EB%A0%88%EC%A0%84%EB%93%9C%20%EB%AA%A8%EC%8A%A4%ED%8A%B8%20%EC%B1%94%ED%94%BC%EC%96%B8%20%EB%B0%8F%20%EC%B5%9C%EA%B7%BC%20100%EA%B2%BD%EA%B8%B0%20%EB%B6%84%EC%84%9D%ED%95%98%EA%B8%B0_33_0.png)

