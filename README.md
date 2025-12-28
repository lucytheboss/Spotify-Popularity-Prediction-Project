# Spotify-Popularity-Prediction-Project

## 1. Project Overview
이 프로젝트는 **"아티스트의 이전 흥행 성적이 다음 곡의 성공을 보장하는가?"** 라는 질문에서 시작되었습니다. Spotify Web API를 활용하여 2023년부터 2025년(예정 포함)까지의 음원 데이터를 수집하고, 아티스트별 시계열 트렌드를 분석했습니다. 특히 아티스트 고유의 인지도(Artist Fixed Effect)를 제거한 후, 순수한 곡(Track) 간의 성과 관계를 파악하기 위해 통계적 기법과 머신러닝 회귀 모델을 적용했습니다.

**🎯 Key Objectives**
Spotify API를 활용한 대규모 음원 및 아티스트 메타 데이터 수집 (Web Scraping & API Handling)
- **Feature Engineering**: 시차 변수(Lag features) 생성 및 아티스트별 평균 중심화(Centering)를 통한 편향 제거
- **Statistical Analysis**: 상관관계 분석 (Pearson, Spearman) 및 OLS 회귀 분석
- **Predictive Modeling**: 이전 곡의 성과를 기반으로 신곡의 성과를 예측하는 선형 회귀 모델 구축 (RMSE 평가)

## 2. Tech Stack & Tools
Language: Python 3.14
**Data Collection**: spotipy (Spotify Web API Wrapper)
**Data Processing**: pandas, numpy
**Visualization**: seaborn, matplotlib
**Statistics & ML**: scikit-learn (LinearRegression, GroupKFold), statsmodels (OLS), scipy

## 3. Data Collection & Preprocessing
### 3.1 Data Acquisition
spotipy 라이브러리를 사용, 2023년~2025년 발매된 트랙 정보를 수집했습니다.
- Track Data: 발매일, 인기도(Popularity), 트랙 ID
- Artist Data: 팔로워 수, 장르, 아티스트 인기도

### 3.2 Preprocessing & Feature Engineering
데이터의 신뢰성을 높이기 위해 다음과 같은 전처리 과정을 거쳤습니다.
- 결측치 제거 및 타입 변환: 발매일(date) 포맷 통일 및 NaN 데이터 처리.
- **Lag Feature 생성**: prev_track_popularity 변수를 생성하여, 아티스트별로 바로 직전 발매된 곡의 인기도를 현재 데이터 행에 매핑했습니다.
- **Mean Centering** : 단순 인기도는 아티스트의 체급(유명세)에 따라 결정되는 경향이 있습니다. 이를 보정하기 위해 **(개별 곡 인기도 - 아티스트 평균 인기도)**를 계산하여 popularity_centered 변수를 만들었습니다. 이를 통해 아티스트의 명성을 배제하고, 곡 자체의 상대적 성과만을 비교 분석했습니다.

## 4. Exploratory Data Analysis (EDA)
데이터 시각화를 통해 변수 간의 관계를 확인했습니다.
- Correlation Analysis: 이전 곡 인기도(prev)와 현재 곡 인기도(current) 사이에 강한 양의 상관관계(r≈0.84)가 관찰되었습니다.
- Centered Analysis: 아티스트 효과를 제거한 후에도(r≈0.32), 이전 곡이 평소보다 잘 됐을 경우 다음 곡도 평소보다 잘 되는 경향("Momentum Effect")이 유의미하게 존재함을 확인했습니다.
<img width="1725" height="987" alt="image" src="https://github.com/user-attachments/assets/b00631c5-e675-4ed2-a840-8af51023bdba" />

## 5. Modeling & Evaluation
popularity_centered(상대적 인기도)를 예측하기 위해 선형 회귀 모델을 구축했습니다.

**Validation Strategy: GroupKFold (n=5)**
- 한 아티스트의 곡이 훈련셋과 테스트셋에 섞여 들어가는 데이터 누수(Leakage)를 방지하기 위해, 아티스트 단위로 데이터를 분리하여 검증했습니다.

**Baselines:**
- Baseline 0: 모든 예측값을 0(평균)으로 가정
- Baseline 1: 이전 곡의 성과가 그대로 유지된다고 가정

**Model Result:**
Lag Regression Model이 RMSE 10.08을 기록하며, Baseline(RMSE 11.5~11.8) 대비 약 13~15% 성능 향상을 보였습니다.

추가적으로 장르(Genre)와 팔로워 수(Followers) 변수를 투입했으나, 성능 향상은 미미했습니다(RMSE 10.12). 이는 '직전 곡의 성과'가 단기 예측에서 가장 강력한 변수임을 시사합니다.

6. Conclusion & Insights
- **Momentum Exists**: 아티스트의 기본 체급을 제외하더라도, 전작의 흥행은 차기작의 성과에 긍정적인 영향을 미칩니다.
- **Data Integrity**: 시계열 데이터 분석 시 아티스트별 그룹화와 시차(Lag) 데이터 처리가 모델 성능에 결정적인 역할을 함을 확인했습니다.
- **Limitations**: 스트리밍 시장의 외부 요인(마케팅, 틱톡 바이럴 등)을 반영하지 못한 점은 추후 연구 과제입니다.
