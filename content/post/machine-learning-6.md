---
title: GAN 이론
date: 2021-04-04 23:25:57
categories:
  - machine_learning
tags:
  - machine_learning
---

# GAN이란?
GAN은 '생성적 적대 신경망'의 약자로 풀어서 쓰면, 생성자와 식별자가 서로 경쟁(Adversarial)하며 데이터를 생성(Generative)하는 모델(Network)을 뜻한다.

![](/image/GAN3.PNG)

![](/image/GAN5.PNG)

    잡음의 이미지로부터 무엇인가를 생성해낸 결과에 대해서 분류기라고 하는 Dis가 G(Z)를 가짜라고
    분류함 반면에 원래의 이미지를 받아왔을 때 원래 이미지에 대해서는 Positive하게 반응함
    원래 이미지에 대해서는 확률 높게 판단해주고 가짜 이미지에 대해서는 확신도를 아주 낮게 준다.

1. 잡음 이미지 Z
2. 생성신경망 통과 Generator
3. 생성 이미지 획득 G(Z)
4. 원 이미지 X
5. 분류기 통과 Discriminator(원래 이미지와 생성기로부터 만들어진 이미지를 동시에 입력받음)
6. 두 분류 결과 (1:생성이미지 D(G(Z)), 2:원래이미지 D(X))
1번에 대해서는 가짜, 2번에 대해서는 진짜로 분류하게끔 학습 목표를 제공해주어야 하는 것이 알고리즘의 총흐름도
7. 이전 6단계 결과에 대한 적대적 학습시행
![](/image/GAN1.PNG)
-> 생성이미지 출신 저하(생성이미지가 가짜다) & 원래이미지 출신 상승(원래이미지에 대한 확률을 높게 평가한다)
   
![](/image/GAN4.PNG)

Gradient descending
경사하강법을 이용하면 어떤 지점으로 수렴하게 되는 결과를 받아들임 
이 함수의 곡선을 가지고 Gradient ascending을 한다고 가정하면 바깥 영역에서 함수들이 굉장히 커지게 되며
그 끝을 알 수 없기 때문에 학습이 엉망으로 진행된다.

Gradient ascending
Gradient ascending을 사용하려면 함수가 기본적으로 위로 볼록한 형태가 되어야한다.
함수의 최고점(가장 볼록한 부분)에서 Discriminator가 진짜를 진짜로 분류하고 가짜를 가짜로 분류하게 된다.

![](/image/GAN2.PNG)

### 생성신경망을 어떻게 loss 함수를 적용해서 트레이닝 시킬 것인가?
generator 부분에서는 식이 +이다 이것을 보았을 때 Discriminator와 generator가 적대적으로 학습한다는 것을 알 수 있다.
generator가 잘학습하게 되면 log(1-D(G(Z)))라고 하는 부분이 줄어들게 된다.

Discriminator의 목적은 진짜를 진짜로 가짜를 가짜로 인식하게끔 하는게 목표
generator은 가짜를 진짜로 진짜를 가짜로 표현하게끔 표현하는 것이 목표