---
title: 파이토치를 이용한 데이터 가공 예시
date: 2021-03-23 20:58:12
categories:
  - machine_learning
tags:
  - machine_learning
---

```python

import torch
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline


# 데이터
x = np.random.uniform(-1.2, 2, size=(100)) #np.random.uniform은 Numpy에서 제공하는 균등분포 함수이다.
y = 0.1*x**3-0.1*x**2+0.1*np.random.uniform(size=(100)) # 노이즈 추가
plt.plot(x,y,'o') # 3차 함수지만 노이즈가 포함되어 있다.


# train/validation/test 분류
data = np.concatenate((x.reshape(-1,1),y.reshape(-1,1)), axis=-1) # X, Y 묶음, X: input, Y: target
print(data.shape) # 차원 0: 데이터 개수, 차원1: (x,y) --> (100,2)
np.random.shuffle(data) # 데이터 골고루 섞음

test_num = int(data.shape[0]*0.1) # data.shape[0] = 100 --> test num 100 * 10% --> 10개
valid_num = int(data.shape[0]*0.1) # data.shape[0] = 100 --> valid num 100 * 10% --> 10개
train_num = data.shape[0] - valid_num - test_num # train num --> 80개

train_data = data[:train_num] # 0~80개까지 (80,2)
valid_data = data[train_num:train_num+valid_num] # 80~90개까지 (10,2)
test_data = data[train_num+valid_num:] # 90~100개까지 (10,2)

# train_data[:,0] --> train_date 전체 데이터에서 0번째 차원 : x
# train_data[:,1] --> train_date 전체 데이터에서 1번째 차원 : y
plt.plot(train_data[:,0],train_data[:,1],'ro')
plt.plot(valid_data[:,0],valid_data[:,1],'ko')
plt.plot(test_data[:,0],test_data[:,1],'go')


# torch batch
# 이 구문 자체를 숙달해라!
# 데이터셋 클래스는 토치 안에 유틸, 유틸 안에 데이터, 데이터 안에 데이터셋이라는 클래스를 상속 받는다  
class dataset(torch.utils.data.Dataset):
    def __init__(self,data): # 클래스를 처음 접목시킬 때 들어오는 인자 데이터를 적어준다.
        self.data = data     # self.data로 다시 정의
        self.x = self.data[:,0].reshape(-1,1)
        self.y = self.data[:,1].reshape(-1,1)
     
    # idx : 전체 데이터에서 뽑아낼 데이터의 인덱스를 가르킴(ex.1,3,5,7,9번째 벡터를 뽑아내라)
    # if (100,2) --> batchsize = 3 --> idx (0, 55, 99)를 끄집어 내라. 인덱스 개수만큼 추출해라.
    # 전처리 과정에서 torch.Tensor 형태로 변환해줘야한다. 뉴럴 네트워크에서 input,output torch.Tensor 변환
    def __getitem__(self, idx): 
        return torch.Tensor(self.x[idx],torch.Tensor(np.array(self.y[idx])))
    
    # len 구문을 통해서 전체 데이터가 몇개 있는지 클래스에 명시
    def __len__(self):
        return len(self.data)
    
# dataset에 train_data를 넣어주면 train_daset이라는 토치안의 기능이 완성되는 것 
# batch_size만큼 출력하게 해주고 shuffle 해주고 torch로 자료형을 변화해주는 기능도 갖춰주게끔 함.
train_dataset = dataset(train_data) # train_data --> (80,2)

# 위의 dataset을 로딩할 수 있게끔 한번 더 과정을 거침
# batch_size = 32 --> __getitem__(self,idx) 속성의 인덱스를 무작위로 32개의
# array로 하나를 만들어줌. 0~80개 중에서 32개의 인덱스를 무작위로 뽑아줌
# x를 32개, y를 32개 뽑아준 뒤 torch 형태로 변환
# shuffle = True --> 매번 training 될 때마다 전체 데이터셋을 한번 더 뒤섞어주는 역할
# shuffle = False --> 고정된 상태의 데이터 출력
# drop_last= False --> batch_size = 32이기 때문에 32개씩 묶으면
# 32, 64, 96으로 넘어가기 때문에 전체 사이즈 80개이므로 16개 모자라게 된다.
# 이때 16개에 대한 배치를 수행할 것인지 말것인지의 여부를 결정하는 파라미터이다.
# False일 경우 마지막 배치를 그대로 수행(ex. 32개, 32개, 16개)
train_loader = torch.utils.data.DataLoader(train_dataset,batch_size=32,shuffle=True,drop_last=False)

cnt = 0
for X,Y in train_loader: #getimem --> X, Y
    print(X.shape) # (32,1)
    print(Y.shape) # (32,1)
    cnt += 1
    if cnt == 3:
        break
# OUTPUT
# torch.Size([32, 1])
# torch.Size([32, 1]) 
# torch.Size([32, 1]) 
# torch.Size([32, 1]) 
# torch.Size([16, 1]) 
# torch.Size([16, 1]) 

```