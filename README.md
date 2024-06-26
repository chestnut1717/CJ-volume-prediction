# CJ_volume-prediction


## Description
- CJ 대한통운 미래기술 챌린지(AI / 빅데이터) 대회 <b>상품/물량 수요 예측</b> track 참가
- 시계열 분석과 머신러닝 모델(XGBoost) Two-track으로 분석해보고 XGBoost를 최종적으로 선택

## Environment
- Python == 3.8
- scikit-learn == 1.0.1

## Files
- CJ_Project.ipynb : EDA부터 모델까지 분석한 결과
- 개발_계획서_최종.pdf : 모델 구축 결과 및 제안서

## EDA
- 초기에는 시각화 등을 통해 데이터에 대한 전반적인 이해 하고자 함
- 고객사 코드(SHPR_CD)와 입력자(INS_ID)를 통해 고객사 및 고객사를 추측케 하는 단서 확인
- e-fullfillment 사업이 네이버 스마트스토어와 파트너쉽 관계라는 점을 뉴스기사를 통해 파악하여, 스마트스토어로 범위를 좁혀나가 고객사 특정
- 물동량이 비정상적으로 튀는 outlier들을 바탕으로, 이 날에 이벤트가 있다고 가정을 세우고 특정된 고객사의 이벤트와 outlier가 나타나는 날짜 일치 여부 확인
- 유의미한 분석이 가능한 상위 10개 고객사를 바탕으로 예측 시작

## Model - Time_Series
- 일별 물동량 예측인만큼 먼저 시계열 분석을 실시
- ACF, PACF 등 정상성 테스트 => 1을 제외한 모든 lag에서 신뢰구간 내 파묻혀있어 winsorize
- SARIMAX(외인성을 고려해서) 를 사용해서 예측 => 기대했던 정규성 미치지 않았고 잔차의 분산또한 일정하지 않고, p-value 또한 유의수준을 넘어섬 => 부적합 판단

## Model - XGBoost
- 크롤링 등으로 수집한 상위 10개 고객사 이벤트 날짜를 바탕으로 설명변수 생성(종류별 이벤트, 이벤트 영향력 변수 등)
- 네이버 검색 api를 통해 검색어 통계치도 설명변수를 사용하고자 했으나, 오히려 통계적수치, 정확도가 떨어지는 현상이 나타나 폐기
- 개별 회귀계수에 대한 t검정 실시해서 유효하다 판단되는 변수 사용해서 XGBoost 모델 튜닝
- XGBoost 모델 로 고객사별 평균 MAPE가 19~24정도가 나오는 것을 확인 


## Q&A
Q) 이 모델은 2/14일 이벤트 유무 변수를 가지고 2/14일의 물동량을 예측하는 구조이다. 이것은 큰 의미가 없지 않느냐?? 어디다 써먹을 수 있지?      
   
A) 사업을 하고 있는 기업의 관점에서 보면 달라진다   
- 우선, 이벤트는 <b>당일날 바로 열리는 것이 아니다.</b> 사전에 이벤트 기획, 광고 등 마케팅 활동 등 준비기간도 있을 뿐더러 당일날 바로 확정되는 것이 아닌, 최소 1주일 전에 확정된다(네이버 기준) 
- 크롤링한 이벤트는 거의 대부분 네이버 쇼핑에서 주관하고 있고(쇼핑라이브, 쇼핑위크 등), 네이버와 CJ대한통운이 e-fullfillment 사업을 함께 진행하고 있다.
- 네이버 측에서는 물동량 예측 및 재고관리를 위해 물동량이 갑자기 증가하는 이벤트 날짜를 사전에 공유할 것이다(최대의 수익을 올리기 위해)
- <b>재고관리의 관점</b>에서 이벤트 날짜를 사전에 아는 것은 매우 중요한데, 대한통운의 e-fullfillment 사업은 기존의 택배방식과는 달리 주문이 들어오면 집하과정을 거치지 않고 대한통운 hub에서 바로 주문자에게 배송하는 시스템이기 때문이다.
- 따라서 이벤트가 있을 경우에는 미리 물량을 비축하는 등의 작업이 필요하다 각 고객사별로 최대한 정확한 수요량을 예측해야 추가비용(반송비용, 창고관리비용)등의 부가적인 지출을 줄일수 있기 때문이다.
- 결론은 <b>당일 예측을 위한 모델이 아니다.</b> 사전에 전달받은 내부정보(이벤트 등)를 토대로 촤소 1주일 전에 1주일 후의 물동량을 예측하는 모델이며 재고관리 측면에서 올바르게 사용될 수 있을 거라 기대한다.
![image](https://user-images.githubusercontent.com/62554639/154227002-4bb864c4-4d12-4c00-bb05-827a4fdd0362.png) 출처 : 대한통운 homepage



## Feedback / Complement
- 시계열 분석은 처음이라서 아직 부족한 부분도 있었다. 특히 시계열 정상성 검정방법에는 ADF 뿐만 아니라 KPSS 등 여러 방법이 있다는 것을 배웠다.
- 또한 잔차진단을 하기 위해서는 정상성, 자기상관 테스트 뿐만 아니라 정규분포, 등분산성 또한 고려해야 한다는 점을 배웠다
- "이벤트 예측을 하는 것은 불가능하다. 따라서 이벤트 예측보다, 실제 이벤트가 있을 때 얼마나 물동량을 정확하게 예측할 수 있는지에 대해 초첨을 맞춰야 한다"는 피드백을 받았다.


## 참고자료
<a href="https://arxiv.org/abs/2011.10715">NAVAE CLOVA, <i>A Worrying Analysis of Probabilistic Time-series - Models for Sales Forecasting</i>(PMLR, 2020), 98-105</a>


