---
title: Feature Engineering
date: 2021-04-12 23:33:57
categories:
  - Feature Engineering
tags:
  - Feature Engineering
---

### 머신러닝 모델 성능 최적화

* Feature Engineering
  - 알고리즘이 보다 효율적으로 예측할 수 있도록 데이터 가공
  - 기존 feature들을 다양한 형태로 재가공
    
* 알고리즘 선택
  - 선형, SVM, 확률, 트리 기반의 다양한 알고리즘
  - Boosting 계열 알고리즘의 강세
  - 모델 혼합(Blending)m stacking

* Hyper parameter 튜닝
  - Feature engineering 된 데이터 세트로 최적화된 알고리즘 하이퍼 파라미터 튜닝
    


### Feature Engineering의 주요 기법

#### [기본 Feature Engineering]

* 스케일링(Scaling)
    - standardScaler, MinMaxScaler
    
* 변환(Transformation)
    - 로그 변환(Log Transformation)
    - Polynomial 피처 변환
    - PCA 변환
    
* 인코딩(Encoding)
    - 레이블 인코딩, 원핫 인코딩
    
* 결측치(Null값)/이상치(Outlier) 치환

* Skew 데이터 세트 보정
    - 오버/언더 Sampling
    

#### [업무적인 이해에 기반한 피처 재가공과 생산]
* 중요 피처들을 결합/재가공 작업을 통해 새롭고 의미 있는 피처들을 도출
* 업무와 연관성이 노은 새로운 피처들을 생산


### Feature Engineering을 위한 주요 접근 방식

#### [업무적으로 중요한 feature들의 재결합 및 재가공]
* 중요도가 높은 feature들에 대해서 재가공

* 주요 컬럼끼리 차 또는 합,비율 등을 재가공

* 주요 컬럼 및 가공 컬럼들에 대한 다양한 aggregation 적용

* 업무적으로 의미있는 컬럼 재생산

#### [최근 데이터, Active인 데이터 위주 필터링 후 데이터 가공]
* 시계열성 데이터의 경우 오래전 데이터 보다는 최근 데이터를 기반으로 별도의 재가공 적용

* 업무적으로 여전히 유효한 최근 데이터를 필터링하여 가공 
  

###### 아직 데이터로 표출되지 않은 중요 피처들을 Feature engineering을 통해 도출


###### 지나치게 비슷한 피처들을 너무 많이 도출하는 것은 과적합(Overfitting)을 가져 올 수 있음. 적절한 피처들을 선택하는 추가 작업이 필요할 수 있음.