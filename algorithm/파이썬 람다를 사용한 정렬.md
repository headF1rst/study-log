---
toc: true
title: "파이썬 람다를 사용한 정렬"
category:
  - Python
---
파이썬 정렬함수 sort가 제공하는 다양한 정렬 조건을 배워보도록 하겠다.

## 정렬함수 sort
sort 함수는 key와 reverse 매개변수를 갖고있다.

### reverse
- reverse = True : 내림차순
- reverse = False : 오름차순 (기본값)

### key
정렬을 목적으로 하는 함수를 값으로 넣으면 key 값을 기준으로 정렬이 수행된다.

key 값에는 lambda를 사용할 수 있다.

### lambda 식

**lambda** 매겨변수 : 표현식

```python
def sum(x, y):
    return x + y

sum(3, 4)

# 람다식
(lambda x, y: x + y)(3, 4)
```

### 예제
리스트를 정렬 key 사용

```python
a = [(1, 2), (4, 1), (2, 3), (4, 6), (5, 0)]

# 튜플의 두번째 인자값을 기준으로 정렬이 이루어진다.
sorted_a = sorted(a, key = lambda x : x[1])

# [(5, 0), (4, 1), (1, 2), (2, 3), (4, 6)]
```