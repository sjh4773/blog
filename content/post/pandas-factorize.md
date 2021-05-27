---
title: pandas.factorize()
date: 2021-04-14 19:15:38
categories:
  - pandas 공부
tags:
  - pandas 공부
---

# pandas.factorize()
대부분의 머신러닝 알고리즘은 숫자형을 다루므로 카테고리를 텍스트에서 숫자로 바꿔줘야 한다.
이를 위해 각 카테고리를 다른 정숫값으로 매핑해주는 factorize() 함수를 사용합니다.

### pandas.factorize() and Series.factorize()

    Parameters:
    values : 1D sequence.
    sort : [bool, Default is False] Sort uniques and shuffle labels.
    na_sentinel : [ int, default -1] Missing Values to mark ‘not found’.
    
    Return: Numeric representation of array

### factorize() 방법의 작동 설명
```python
import numpy as np
import pandas as pd
from pandas.api.types import CategoricalDtype

labels, uniques = pd.factorize(['c','a','a','c','b','b'])

print("Numeric Representation : \n",labels)
print("Unique Values : \n", uniques)
```
#### 결과값

    Numeric Representation : 
     [0 1 1 0 2 2]
    Unique Values : 
     ['c' 'a' 'b']

### 정렬
```python
label_sort, unique_sort = pd.factorize(['c','d','d','b','a','c'], sort=True)

print("Numeric Representation :",label_sort)
print("Unique Values : ",unique_sort)
```
#### 결과값
    umeric Representation : [2 3 3 1 0 2]
    Unique Values :  ['a' 'b' 'c' 'd']

### 누락값 표시
누락값은 na_sentinel(default -1)로 코드에 표시된다. 누락값은 고유 항목에 포함되지 않는다.
```python
label_none, unique_none = pd.factorize(['b', None, 'd', 'c', None, 'a', ], 
                                              na_sentinel = -101)
  
print("\n\nNumeric Representation : \n", label_none)
print("Unique Values : \n", unique_none)
```
#### 결과값


    Numeric Representation : 
     [   0 -101    1    2 -101    3]
    Unique Values : 
     ['b' 'd' 'c' 'a']
### 범주내에 있지 않는 값은 -1 처리된다.
```python
a = pd.Categorical(['d','a', 'a', 'c','f'], categories =['a', 'b', 'c', 'd'])
  
label_cat, unique_cat = pd.factorize(a)
  
print("\n\nNumeric Representation : \n", label_cat)
print("Unique Values : \n", unique_cat)
```

#### 결과값
    Numeric Representation : 
     [ 0  1  1  2 -1]
    Unique Values : 
     ['d', 'a', 'c']
    Categories (4, object): ['d', 'c', 'b', 'a']