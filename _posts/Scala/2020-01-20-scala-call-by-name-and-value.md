---
layout: single
classes:
  - wide
title: Scala - call by name, call by value
comments: true
author_profile: true
tags:
  - c_lang
  - data_structure

---

```scala
def test(x: Int, y: Int) = x*x
```

이와 같은 function이 있을 때 **test(3+4, 8)** 형태로 call을 할 때 call by name과 call by value의 차이는 아래와 같습니다.

### Call by name

call by name은 expression을 그대로 가져갑니다.

그래서 test(3+4, 8)을 call by name 방법을 사용하게 되면 reduction step이 아래와 같아집니다.

1. test(3+4, 8)
2. (3+4) * (3+4)
3. 7 * (3+4)
4. 7 * 7
5. 49

### Call by value

call by value는 인자를 value로 받기 때문에 reduction step이 아래와 같아집니다.

1. test(3+4, 8)
2. test(7, 8)
3. 7*7
4. 49

test(3+4, 8) 같은 경우 call by name의 reduction step이 한 단계 더 많기 때문에 이 식에서는 call by value가 advantage가 있다고 할 수 있습니다.

그렇다면 무조건 call by value가 더 좋은 걸까요?

당연히, 항상 그렇지 않습니다.

test(7, 2*4) 식을 예로 들어 보겠습니다.

| Reduction step | Call by name | Call by value |
| :------------: | :----------: | :-----------: |
|       1        | test(7, 2*4) | test(7, 2*4)  |
|       2        |     7*7      |  test(7, 8)   |
|       3        |      49      |      7*7      |
|       4        |              |      49       |

이 경우에는 call by value의 reduction step이 한 단계 더 많습니다.

그러므로, call by name과 call by value 중 어떤 것이 더 빠르냐는 상황에 따라 다른 것 같습니다.