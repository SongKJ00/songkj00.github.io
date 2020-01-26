---
layout: single
classes:
  - wide
title: Scala - Currying이란
comments: true
author_profile: true
tags:
  - scala


---

[이 강의](https://www.coursera.org/lecture/progfun1/lecture-2-2-currying-fOuQ9)를 보고 정리한 글입니다.

### Currying의 정의

[Scala 공식 홈페이지 한글 번역판](https://docs.scala-lang.org/ko/tour/multiple-parameter-lists.html)에서는 

`메소드에는 파라미터 목록을 여럿 정의할 수 있다. 파라미터 목록의 수 보다 적은 파라미터로 메소드가 호출되면, 해당 함수는 누락된 파라미터 목록을 인수로 받는 새로운 함수를 만든다.`

라고 얘기하고 있습니다.



### Currying을 쓰기 전 해야 했던 작업들

아래와 같이 `Int => Int` 타입 함수 하나와, `Int` 타입 변수 두 개를 파라미터로 받는 sum이라는 함수가 있습니다.
이 sum 함수를 통해 a부터 b 사이의 수를 f 함수식으로 매핑했을 때, 매핑한 값들의 총합을 알 수 있습니다.

~~~scala
def sum(f: Int => Int, a: Int, b: Int) = {
  def loop(a: Int, acc: Int): Int =
    if (a > b) acc
    else loop(a+1, f(a) + acc)
  loop(a, 0)
}
~~~

sum 함수를 이용한 예는 아래와 같습니다.
sumInts로는 a부터 b 사이의 수를 다 더한 결과를, sumCubs로는 a부터 b 사이의 수를 세제곱하여 다 더한 결과를 얻을 수 있습니다.

~~~scala
def sumInts(a: Int, b: Int) = sum(x => x, a, b)	// a부터 b 사이의 수를 다 더함
def sumCubes(a: Int, b: Int) = sum(x => x*x*x, a, b)	// a부터 b 사이의 수를 세제곱하여 다 더함
~~~

sumInts와 sumCubes 모두 두 개의 파라미터(a, b)가 붙는 함수라는 게 명시되어 있습니다.

scala에서는 함수를 리턴하는 함수를 정의할 수 있어서 sumInts와 sumCubes를 정의할 때 뒤에 붙는 파라미터 리스트를 없앨 수도 있습니다.



### Higher-order function을 이용한 function을 return하는 function 만들기

함수 `sum`을 아래와 같이 바꿔보겠습니다.

~~~scala
def sum(f: Int => Int): (Int, Int) => Int = {
  def sumF(a: Int, b: Int): Int = 
    if (a > b) 0
    else f(a) + sumF(a+1, b)
  sumF
}
~~~

`sum`은 이제 function을 return하는 function이 되었고, return되는 function인 `sumF`는 함수 파라미터 f를 사용하고 a와 b 사이의 결과를 모두 더합니다.

이제 sum은 `(Int, Int) => Int` 타입의 함수를 반환하는 함수이므로 sumInts 함수와 sumCubes 함수를 아래와 같이 바꿔서 사용할 수 있습니다.

~~~scala
def sumInts = sum(x => x)
def sumCubes = sum(x => x*x*x)

sumInts(1, 10) + sumCubes(1, 10)
~~~



### Currying을 이용해보기

여기서 또, sumInts와 sumCubes 함수에서 공통적이지 않은 이름인 Ints와 Cubes를 생략할 수 있습니다.

sum 함수를 아래와 같이 변경해보겠습니다.

~~~scala
def sum(f: Int => Int)(a: Int, b: Int): Int = 
  if (a > b) 0 else f(a) + sum(f)(a+1, b)
~~~

sum 함수가 위와 같이 바뀌었다면, sumCubes 함수는 아래와 같이 바꿀 수 있습니다.

~~~scala
def cube(x: Int) = x*x*x
sum (cube) (1, 10)
~~~

* `sum (cube)`는 sum 함수에 cube를 적용하여 세제곱의 합을 구하는 함수를 return합니다.
* 결국, `sum (cube)`는 sumCubes 함수와 같습니다.
* 이렇게 생성된 함수는 다음 파라미터인 (1, 10)을 사용하게 됩니다.

sum을 호출할 때, `sum (cube)`로 호출하면서 파라미터 리스트 중 a와 b가 누락되었지만 `(Int, Int) => Int` 타입의 함수가 새로 만들어졌습니다.
이렇게 파라미터 리스트가 누락되었을 때 누락된 파라미터 리스트를 가지고 새로운 함수를 만드는 것을 **currying**이라고 합니다.



### Currying을 이용한 다른 예제 - product 함수 구현

product 함수(특정 범위 내에 있는 수들의 곱셈 구하기)

~~~scala
def product(f: Int => Int)(a: Int, b: Int): Int = 
  if (a > b) 1
  else f(a) * product(f)(a+1, b)
product(x => x*x)(3, 4)
~~~

위 코드로 3부터 4까지의 수를 제곱으로 매핑하여 곱한 결과를 알 수 있습니다.

그렇다면 여기서 sum과 product를 일반화해서 구현한다면 어떻게 할 수 있을까요?

둘을 일반화한다면 먼저, product 함수에서 고쳐야 할 부분이 있습니다.

첫 번째, sum과 product의 항등원을 구별해야 합니다.
위 product 함수 구현에서  a > b가 true이면 곱셉의 항등원인 1을 return했는데, sum인 경우는 덧셈의 항등원인 0을 return해야 합니다.

두 번째, 사칙연산 식을 구별해야 합니다.
위 product 함수 구현에서는 f(a)와 product(f)(a+1, b)를 곱하지만, sum인 경우는 더해야 하기 때문에 사칙연산 기호도 바뀌어야 합니다.

sum과 product를 일반화해줄 새로운 함수 mapReduce를 아래와 같이 정의해보겠습니다.



~~~scala
// combine: 사칙연산 식(sum이면 덧셈, product면 곱셈)
// zero: 항등원
def mapReduce(f: Int => Int, combine: (Int, Int) => Int, zero: Int)(a: Int, b: Int): Int = 
  if (a > b) zero
  else combine(f(a), mapReduce(f, combine, zero)(a+1, b))
~~~



위와 같은 함수가 갖춰져 있다면 product 함수는 아래와 같이 재정의할 수 있습니다.

~~~scala
def product(f: Int => Int)(a: Int, b: Int): Int = mapReduce(f, (x, y) => x*y, 1)(a, b)
~~~

 sum 함수는 아래와 같이 재정의할 수 있습니다.

~~~ scala
def sum(f: Int => Int)(a: Int, b: Int): Int = mapReduce(f, (x, y) => x+y, 0)(a, b)
~~~

각 함수 호출을 아래와 같이 호출할 수 있습니다.

~~~scala
product(x => x*x)(3, 4)	// 144((3*3) * (4*4))
sum(x => x*x)(3, 4)     // 25((3*3) + (4*4))
~~~



### Currying을 이용하여 얻는 이점

그렇다면, currying을 이용하여 얻는 이점은 무엇일까요?

변하지 않는 부분은 미리 파라미터로 넘겨두어 매번 여러 개의 파라미터를 넘길 일이 없을 것입니다.
위 예제에서 sum을 이렇게 바꿀 수 있을 것입니다.

~~~scala
def square(x: Int) = x*x
def sum(f: Int => Int): (Int, Int) => Int = mapReduce(f, (x, y) => x+y, 0)
def sumSquare = sum(square) // or sum(x => x*x)

sumSquare(3, 4)	// or sum(square)(3, 4)
~~~

항등원인 0(zero)과 사칙연산 식(combine)은 변하지 않는 파라미터이니 미리 mapReduce에 넘겨 (a, b) 파라미터 리스트만 갖고 있는 함수를 새롭게 생성합니다.
그러면 앞으로 호출할 때는 a, b에 해당하는 부분만 넘겨도 될 것입니다.

