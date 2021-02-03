# OECD 데이터로 바라본 자살률
## 구성원
- 김경한 : 영유아사망률을 주제로 한 EDA 전담
- 김준성 : 자살률을 주제로 한 EDA 전담
## 목적
- 데이터 처리를 위한 라이브러리인 pandas 실습한다.
- 데이터 시각화를 위한 라이브러리인 matplotlib, seaborn, plotly 실습한다.
- 자살률에 영향을 미치는 피처를 탐색하고 상관계수를 구한다.
## 데이터 출처
- 데이터 출처는 다음과 같다.
  - https://data.oecd.org/
  - https://ourworldindata.org/
- 국내외 자살률에 관한 뉴스를 참고하여 자살률에 영향을 미칠만한 피처를 선별하였다.
  - 성별, 나이, GDP, 학력수준, 빈곤지수 & etc
- 각 데이터 대상은 OECD 가입국이며 연 단위로 이루어져있다.
## 과정
- EDA는 일반적으로 다음과 같은 과정을 거쳤다.
  - 데이터를 불러온다.
  ```
  dfs=[]
  file_list = glob('./data/*.csv')
  for each_file in file_list:
      df = pd.read_csv(each_file)
      dfs.append(df)
  df = pd.concat(dfs)
  ```
  ```
  sc = pd.read_csv('./data/suicide-death-rates.csv')
  - 국가 & 연도를 기준으로 데이터를 취합하고 여러 메써드를 활용하여 데이터를 탐색한다.
    > - X축 : 피처
    > - Y축 : 자살률
  - 피봇테이블과 히트맵을 활용하여 전체적으로 상관계수를 탐색한다.
  ```
  sc_df = sc[['Code', 'Suicide rates']]
  sc_df = sc_df.groupby('Code').mean()
  sc_df.describe()
  ```
  ```
  plt.figure(figsize=(18, 10))
  plt.axhline(y=sc_df['Suicide rates'].mean())
  plt.annotate('KOR', xy=(4, 15), xytext=(3.4, 25),
              arrowprops=dict(facecolor='black'))
  sns.barplot(data=sc_df, x='Code', y='Suicide rates')
  ```
  is_2017 = sc_edu['Year'] == 2017
  sc_edu_df = sc_edu[is_2017]
  # is_25_34 = sc_edu_df.Subject.values == '25_34'
  # sc_edu_df = sc_edu_df[is_25_34]
  sc_edu_df = sc_edu_df[['Code', 'Edutry']]
  sc_edu_df = sc_edu_df.sort_values(by='Edutry', ascending=False)
  sc_edu_df.reset_index(inplace=True)
  plt.figure(figsize=(18, 9))
  sns.scatterplot(data=sc_edu_df, x='Code', y='Edutry')
  ```
  sc_attr = pd.merge(sc_gdp_df, sc_edu_df, how='inner', on='Code')
  sc_attr = pd.merge(sc_attr, sc_work_df, how='inner', on='Code')
  sc_attr = pd.merge(sc_attr, sc_dpr_df, how='inner', on='Code')
  sc_attr = pd.merge(sc_attr, sc_hp_df, how='inner', on='Code')
  sc_attr = pd.merge(sc_attr, sc_pg_df, how='outer', on='Code')
  sc_attr = sc_attr.drop(columns=['index_x', 'index_y', 'Year'])

  is_2017 = sc['Year'] == 2017
  sc_2017 = sc[is_2017]

  sc_attr = pd.merge(sc_2017, sc_attr, on='Code', how='outer')
  sc_attr = sc_attr.drop(columns=['Year'])

  sc_attr.columns
  sc_attr = sc_attr[['Code', 'Suicide rates', 'GDP', 'Edutry',
                     'Average annual hours worked', 'Depressive disorders',
                     'Life satisfaction', 'Poverty gap']]

  sc_attr.corr()
  ```
  show_cols = ['Suicide', 'GDP', 'Edutry', 'Work hours',
             'Depression', 'Happiness', 'Poverty gap']

  plt.figure(figsize=(18, 18))
  sns.set(font_scale=1)
  sns.heatmap(sc_attr.corr(), cbar=True, annot=True, square=True, fmt='.2f',
              annot_kws={'size': 15}, xticklabels=show_cols, yticklabels=show_cols)
  
  is_2017 = sc_edu['Year'] == 2017
  ```
  ```
  sns.pairplot(ko_tot, height=2)
  ```
- 참조
  - OECD 
![Screenshot 2021-02-03 at 14 46 31](https://user-images.githubusercontent.com/70704636/106704242-2084c400-662f-11eb-9571-46d50a0c7c1e.jpg)
  - 한국
![Screenshot 2021-02-03 at 14 46 51](https://user-images.githubusercontent.com/70704636/106704327-49a55480-662f-11eb-82ed-b33ed3773e77.jpg)
![Screenshot 2021-02-03 at 14 47 08](https://user-images.githubusercontent.com/70704636/106704347-5164f900-662f-11eb-89f9-c9635a9c8952.jpg)
## 결론
  - 한국의 자살률은 증가추세에 있다.
  - 대체적으로 연령대가 높을수록 자살률이 높다.
  - 대체적으로 남성이 여성보다 자살률이 높다.
  - 피처 별로 자살률과 상관성을 봤을 때 OECD 가입국의 평균과 한국은 꽤나 큰 차이점을 보인다.
    - OECD 가입국은 행복지수와 음의 상관관계임을 확인하였고, 그 외 피처에서는 별 다른 상관관계를 확인하지 못하였다.
      > - 자살률/행복지수 : -0.46
    - 한국은 노동시간과 음의 상관관계임을 확인하였고, GDP, 학력수준, 우울증과 양의 상관관계임을 확인하였다.
      > - 자살률/노동시간 : -0.84
      > - 자살률/GDP : 0.82
      > - 자살률/학력수준 : 0.73
      > - 자살률/우울증 : 0.57
  - 기존 상식과는 다르게 한국의 군자살률은 민간자살률보다 더 낮았다.
