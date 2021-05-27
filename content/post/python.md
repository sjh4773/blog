---
title: 카페 추천 시스템
date: 2021-03-28 00:02:27
categories:
    - Python 공부
tags:
    - Python 공부
---

Cosine similarity(유사도 측정)

-> 유사인 경우 1, 상극인 경우 -1
 (cosine 함수)
구하는 방식 -> 벡터내적
𝑐𝑜𝑠𝜃 = 𝐴 ∙ 𝐵 / |𝐴||𝐵|
(−1 ≤ 𝑐𝑜𝑠𝜃 ≤ 1)



```python
import numpy as np

def cosine_similarity(a,b):
    num = np.dot(a,b)
    den = np.sqrt(np.sum(a**2))*np.sqrt(np.sum(b**2))
    return num/(den+1e-15)

# 카페 평가 기준 : 분위기, 예술성, 친절함, 가성비, 위생, 맛
# 각 항목별 점수 범위 : 0 ~ 10
# 위의 평가 기준 순서대로 평가 

backkeumdang = np.array([8, 7, 8, 5, 8, 9]) # 기준 카페, (분위기, 예술성, 친절함, 가성비, 위생, 맛)

# 데이터 베이스 내 카페 특징 추출
star_bucks = np.array([6, 7, 6, 2, 6, 6])
parm_table = np.array([3, 3, 5, 1, 5, 5])
twosome_place = np.array([3, 2, 2, 2, 5, 5])
paul_bassett = np.array([3, 2, 5, 2, 5, 6])

# 백금당 vs [모든 카페] cosine 유사도
cafes = [star_bucks, parm_table, twosome_place, paul_bassett]
names = ['star_bucks','parm_table','twosome_place','paul_bassett']

scores = []
for cvec in cafes:
    sim = cosine_similarity(backkeumdang, cvec)
    scores.append(sim)

# 유사도 높은 순으로 정렬


idx = np.argsort(scores) # 작은 값부터 순서대로 데이터의 index를 반환해줌
recom= np.array(names)[idx] # index 값에 따른 카페명이 추출되고 유사도 결과가 오름차순으로 나온다.
print(np.flip(recom)) # 내림차순

# 결과 : 1위 스타벅스, 2위 팜테이블, 3위 폴바셋, 4위 투썸플레이스
```