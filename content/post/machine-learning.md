---
title: 회귀분석
date: 2021-03-24 21:09:29
categories:
  - machine_learning
tags:
  - machine_learning
---

```python


#Logistic regression example


# 라이브러리 호출
import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F

import pandas as pd # 데이터를 가공하기 위함
import numpy as np
import matplotlib.pyplot as plt # 그림을 그리기 위함
%matplotlib inline

#파일 업로드

data = pd.read_csv('./mba_admission.csv') # ./ --> 현재 경로를 나타낸다

data.columns # 데이터 속성을 확인함

data.shape # (40,4) --> 행 : 데이터 크기, 열 : gmat, gpa, work_experience --> 입력, admitted --> 결과

data.head()

# dataset 가공
class dataset(torch.utils.data.Dataset):
    def __init__(self,data):
        self.data = data
        self.data['gmat'] /= self.data['gmat'].max() # max 함수를 통해 정규화, 모든 데이터가 0~1로 정규화 된다.
        self.data['gpa'] /= self.data['gpa'].max()
        self.data['work_experience'] /= self.data['work_experience'].max()

    def __getitem__(self, idx):
        X = torch.Tensor(self.data[self.data.columns[:3]].values) # self.data.columns[:3] --> gmat, gpa, work_experience
        Y = torch.Tensor(self.data['admitted'])                   # .values --> pandas의 자체값만을 이용하겠다는 뜻 
        return X[idx,:], Y[idx]

    def __len__(self):
        return self.data.shape[0] # 전체 데이터의 개수 --> 40

train_dataset = dataset(data)
train_loader = torch.utils.data.DataLoader(train_dataset, batch_size=8, shuffle=True, drop_last=False)

# neural network 모델 생성
# 구조: 입력(3), 은닉(10), 출력(1)
# 활성화:        tanh      sigmoid
class admission_model(nn.Module):
    def __init__(self):
        super(admission_model,self).__init__()
        self.lin1 = nn.Linear(3, 10)
        self.lin2 = nn.Linear(10, 1)

    def forward(self,x): # 입력받은 데이터가 일련의 과정을 거치면 리턴되는 x는 y_hat
        x = self.lin1(x)
        x = F.tanh(x)
        x = self.lin2(x)
        x = F.sigmoid(x)
        return x

model = admission_model()
optimizer = optim.SGD(model.parameters(),lr=0.5)

# 학습단계
# 최적화를 할 때 반복횟수가 굉장히 중요하다. 어떤 데이터를 어떤 배치만큼 잘라내어 마지막 배치까지 돌아야 epoch가 됨
for ep in range(100): # 100번 epoch
    loss_buffer = [] #  각 epoch마다 loss를 확인하기 위함
    for X,Y in train_loader: # epoch마다 각각의 batch 단위를 학습 # 여기에서 X,Y는 위에서 정의한 class 안의 getitem의 X,Y에 대응되는 것
        optimizer.zero_grad()
        y_infer = model(X) # y_infer.shape == (8,1)
        Y = Y.view(-1,1) # Y.shape(8,) --> Y.view(-1,1)
        loss = -torch.mean(Y*torch.log(y_infer)+(1-Y)*torch.log(1-y_infer)) # cross entropy 거리함수 이용
        loss.backward()
        optimizer.step()
        loss_buffer.append(loss.item()) # 각 배치마다 loss_buffer의 빈 array에 loss 값 추가 

    if ep % 10 == 0:
        print('Epoch : {}, Loss: {}'.format(ep, np.mean(loss_buffer)))

        
# 결과 시각화
y_infer = model(torch.Tensor(data[data.columns[:3]].values))

plt.scatter(np.arange(data.shape[0]),data['admitted'],color='red',label='True result')
plt.scatter(np.arange(data.shape[0]),y_infer.detach().numpy(),color='blue',label='Predicted result') # detach() : 기존 Tensor에서 gradient 전파가 안되는 텐서 생성
plt.hlines(0.5,-10,60,label='Admission boundary')
plt.legend(loc='upper center', fontsize='10')
plt.xlim(-1,41)
plt.ylim(-0.1,1.5)
plt.xlabel('Volunteer')
plt.ylabel('Result')
plt.title('MBA admission result included prediction')



```