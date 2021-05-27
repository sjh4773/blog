---
title: cat() 함수
date: 2021-03-28 12:36:58
categories:
    - Pytorch 공부
tags:
    - Pytorch 공부
---

# cat 함수란?

cat함수는 concatenate를 해주는 함수이고 concatenate하고자 하는 차원을 증가시킨다.
concatenate하고자하는 차원을 지정해주면 그 차원으로 두 tensor의 차원을 더한 값으로 차원이 변경된다.

# cat 함수의 시각화

![](/image/cat.JPG)

# Python 코드

```python
import torch

batch_size, N, K = 3, 10, 256

x = torch.rand(batch_size, N, K) # [M, N, K]
y = torch.rand(batch_size, N, K) # [M, N, K]

output1 = torch.cat([x,y], dim=1) #[M, N+N, K]
output2 = torch.cat([x,y], dim=2) #[M, N, K+K]

```