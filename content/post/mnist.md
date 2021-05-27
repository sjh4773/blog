---
title: mnist 데이터
date: 2021-03-24 21:21:29
categories:
  - machine_learning
tags:
  - machine_learning
---

MNIST 데이터란 인공지능이 이미지를 보고 이것이 어떤 이미지인가를
맞춰보게 하는 인공지능 훈련을 위한 Dataset

MNIST 개요
- 손글씨 데이터
- 손으로 작성된 0~9까지의 숫자 이미지
- 이미지 + 레이블 --> 하나의 데이터셋

이미지 속성
- 차원
  (가로,세로,색조) --> (28,28,1)
  픽셀값: 0 ~ 255
  
레이블 정보
- 단순 스칼라
  (확률정보이용) -> One-hot encoding
  
해결전략

1. 이미지, 레이블 전처리

원본이미지 --> 전처리(flatten)

레이블 "3" --> "[0,0,0,1,0,0,0,0,0,0]" One-hot encoding

이미지 전처리
- 텐서 랭크 변환
  (28,28,1) -> (28*28) Flatten 1차원 벡터로 변형
- 이미지 픽셀 정규화
  (0~255) -> (0~1)
  
Torch 코드
- 랭크 변환
  X = X.view(-1,784) --> -1 : batch size 의미,
  이미지를 batch에 해당하는 개수만큼 묶음으로써 본다
  
- 픽셀 정규화
  X /= torch.max(X) --> 0 ~ 1 정규화
  
레이블 처리
- 원-핫 인코딩(One-hot encoding)
  확률 분포도로 변환작업
  
ex) MNIST 레이블
(0 ~ 9) 10개 숫자 -> 10개 사건분포

5 -> One-hot encoding
"[0,0,0,0,0,1,0,0,0,0]"
  
2. Neural network 모델 생성
3. Backpropagation 알고리즘 최적화
4. Test data 검증


프로세스

1. Neural network 초기 생성
   Layer 구성 (784 -> 256 -> 128 -> 10)
   Activation        'Relu' 'Relu' 'Softmax'
   
2. Loss함수(KL-divergence)
   확률분포간의 거리

+ Softmax(모든 벡터의 값 0~1)