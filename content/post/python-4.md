---
title: tuple과 list의 차이
date: 2021-04-06 22:42:39
categories:
    - Python 공부
tags:
    - Python 공부
---
# 리스트(list)와 튜플(tuple)의 차이는 무엇인가?

### 공통점

1.리스트와 튜플은 둘 다 컨테이너로 일련의 객체를 저장하는데 사용한다.
둘 다 타입과 상관 없이 일련의 요소(element)를 갖을 수 있다. 두 타입 모두 요소의 순서를 관리한다. (세트(set)나 딕셔너리(dict)와 다르게 말이다.)
![](/image/tuple1.PNG)


2.list와 tuple 모두 인덱스를 통해 특정 요소에 접근할 수 있다.
![](/image/tuple2.PNG)


3.list와 tuple 모두 iterable하다. 즉, for문을 넣고 돌릴 수 있다.
![](/image/tuple3.PNG) 
 





### 차이점

리스트와 튜플의 기술적 차이점은 불변성에 있다. 리스트는 가변적(mutable, 변경 가능)이며 튜플은 불변적(immutable, 변경 불가)이다. 이 특징이 파이썬 언어에서 둘을 구분하는 유일한 차이점이다.
![](/image/tuple4.PNG) 
 
이 특징은 리스트와 튜플을 구분하는 유일한 기술적 차이점이지만 이 특징이 나타나는 부분은 여럿 존재한다. 예를 들면 리스트에는 .append() 메소드를 사용해서 새로운 요소를 추가할 수 있지만 튜플은 불가능하다.
![](/image/tuple5.PNG) 
 
튜플은 .append() 메소드가 필요하지 않다. 튜플은 수정할 수 없기 때문이다.

[ 참 고 ]
https://edykim.com/ko/post/python-list-vs.-tuple/
https://itholic.github.io/python-list-tuple/
