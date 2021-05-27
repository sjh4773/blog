---
title: 파이토치로 만드는 신경망
date: 2021-03-22 20:58:12
categories:
  - machine_learning
tags:
  - machine_learning
---

```python
# PyTorch 필요 라이브러리 호출
import torch # pip install pytorch 라이브러리 설치
import torch.nn as nn # 인공신경망을 구성해주는 메소드들의 집합
import torch.nn.functional as F # 활성화 함수들을 간단하게 호출
import torch.optim as optim # 토치를 통한 최적화 기능

# Neural network 모델 생성
# 입력 --> 은닉 (3,5)
# 은닉 --> 출력 (5,2)

# nn 뉴럴 네트워크 안에 있는 기능 하나를 호출 
class fir_model(nn.Moudule): # nn.Moduel안에 있는 부모 클래스를 상속 받는다.
    def __init__(self):
        super(fir_model,self).__init__()
        # 가중치행렬 1 설정
        self.lin1 = nn.Linear(3,5) # Torch nn안에 있는 메소드를 그대로 가져온다. (3,5)매트릭스를 만든다.
        # 가중치행렬 2 설정
        self.lin2 = nn.Linear(5,2)
        
    # forward propagation
    def forward(self,x): # forward는 x를 받는다.
        x = self.lin1(x) # 위의 3*5 matrix에 입력신호를 넣어주고 그에 따른 결과를 x로 치환한다.
        x = F.relu(x)    # relu 활성화 함수 통과
        x = self.lin2(x)
        x = F.sigmoid(x)
        return x         # x는 마지막 두차원, y_hat(예측치)이다.

# Neural network 모델 정의 및 최적화 툴(optimizer 사용예시)
model = fir_model()
opt = optim.SGD(model.parameters(),lr=0.01) # Adam, RMSprop --> SGD를 발전시킨 모델
# model.parameters() : 학습을 할 파라미터, 가중치 행렬이 무엇일지 정의 해준다.
# learning rate : 최적화 툴은 0.01의 러닝 레이트를 갖는다.

# 결과 출력
print(model)

# bias : 입력신호에 부가적인 정보를 더함, 뉴럴네트워크가 더 잘 트레이닝 될 수 있게끔 함.
# fir_model(
#   (lin1): Linear(in_features=3, out_features=5, bias=True)
#    input_layer : 3, hidden_layer : 5
#   (lin2): Linear(in_features=5, out_features=2, bias=True)
#    hidden_layer : 5, ouput_layer : 2


# x와 y를 torch에 대한 언어로 바꿔줘야 한다.
# 이 과정을 실행시켜주지 않으면 제대로 실행되지 않음.
# torch는 torch끼리의 언어로써,tensor는 toch tensor로만 적용되어야함.
x = torch.Tensor(x) # 입력 데이터(3차원 벡터)
y = torch.Tensor(y) # 출력 데이터(2차원 벡터)


# Backward propagation
# 최적화 단계 이 셀은 올바르게 실행되지 않는다. x,y는 실제로는 없지만 있다고 가정하고 코딩
criterion = nn.MSELoss() #criterion변수는 (MSELoss, (y_ - y)**2)

# optimizer안 모든 gradient --> 0 초기화,
# 이전에 학습 시켜놨던 gradient의 찌꺼기가 남아
# 올바르게 학습되지 않음 따라서 매번 경사하강법 반복마다 zero_grad() 사용
opt.zero_grad()
y_infer = model(x) # forward propagation
loss = criterion(y_infer,y) # criterion 직접 만들수도 있음
loss.backword()
opt.step() # 옵티마이저를 실행한다.
