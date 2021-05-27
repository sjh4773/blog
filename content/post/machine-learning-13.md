---
title: 베이지안 최적화 기법 이론 보충
date: 2021-04-29 00:08:32
categories:
  - machine_learning
tags:
  - machine_learning
---

# 베이지안 최적화 기법 이론 보충

    머신러닝 혹은 딥러닝 모델들을 학습 시킬 때 하이퍼 파라미터를 결정해야 하는 경우가 많이 있다.
    이때 일반적인 하이퍼파리미터 튜닝 방법은 일저한 간격을 정해서 하이퍼파라미터 후보군을 설정하는 Grid Search와
    일정 구간 내에서 랜덤하게 하이퍼 파리미터 후보군을 결정하는 Random Search가 활용된다.
    하지만 두 방법론 모두 후보군에 대해서 성능을 확인한 후 비교해야 하므로 비효율적이다.
    이러한 경우에 사용할 수 있는 것이 Bayesian Optimization이다.
    베이지안 최적화는 크게 두 가지 핵심 모듈이 존재한다.
    하이퍼파라미터 후보군과 모델의 성능 사이의 관계를 도출해주는 모델을 Surrogate Model, 모델의 성능이 지금보다 향상될 수 있는 가능성에 대해 평가하는 함수를 Acquisition Function이라고 한다.
    acquisition function 관점에서 착취와 탐험사이 균형 맞추기가 중요하다.

![](/image/bay1.PNG)


Surrogate Model로 Gaussian Process Regression이 있고 Acquisition Function으로 Maximum Expected Improvement가 있다.

![](/image/bay2.PNG)

    Gaussian Process Regression은 출력값 Y가 Gaussian Process를 따른다는 가정하에 Regression을 진행하는 모델로
    일반적인 Linear Regression과 다르게 출력값에 대해 구간추정이 가능한 모델이다. 

![](/image/bay3.PNG)


    Maximum Expected Improvement는 앞에서 도출한 출력값을 통해 현재 출력값 대비 향상에 대한 기댓값을 구한 후 가장 기댓값이 큰 하이퍼파라미터 후보군을 선택하는 과정이 된다. 선택된 하이퍼파라미터를 실제 모델에 적용하면 모델의 성능에 대해 알 수 있고
    이렇게 얻어진 데이터를 다시 Surrogate Model에 적용하고 Maximum Expected Improvement를 통해 최적의 하이퍼라미터에 대해 찾아갈 수 있다. 


### 베이지안 최적화 과정

    우선 일정량의 hyperparameter 세팅을 X변수로 하고 모델의 성능을 Y변수로 하여 데이터 셋을 만든다.
    그 다음에 이 데이터 셋에서 X와 Y의 관계를 가지고 사전 지식을 만든 후 새로운 데이터들(X변수, hyperparamter 조합)이 들어왔을 때
    일반화 성능이 우수하도록 하는 최적의 x를 찾아서 추가해주는 형식으로 진행된다


#### 장점

    1.하이퍼 파라미터를 적절하게 선택하는 문제는 실제 상황에서 매우 중요하다. 하지만 뾰족한 방법이 없어, 경험에 의해 적절히 선택하는 것이 보통이다.
      딥러닝 모델은 다른 모델에 비해 선택해야할 파라미터의 개수가 많이 때문에 경험에 의해 선택하기가 쉽지 않다.
      이러한 상황에서 베이지안 최적화 방법이 유용하게 사용될 수 있다.
    
    2. 탐색 시간적인 측면에서 상당한 효율을 가진다.

#### 단점

    다른 최적화 방법론들도 마찬가지로 hyperparameter가 증가하면 차원이 급격하게 늘어나며 이에 대해 성능을 예측하는 것이 점점 더 어려워진다.
