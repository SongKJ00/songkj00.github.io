---
layout: single
classes:
  - wide
title: Scala - Functions and Data
comments: true
author_profile: true
tags:
  - scala



---

[이 강의](https://www.coursera.org/lecture/progfun1/lecture-2-5-functions-and-data-5mmJP)를 보고 정리한 글입니다.

함수가 data structure을 생성하고 캡슐화하는 것에 대해 정리해보겠습니다.

### Example - 유리수(Rational Numbers)

유리수 $$ x \over y$$는 두 개의 정수 분자(numerator) $$x$$ 와 분모(denominator) $$y$$ 로 표현됩니다.

두 개의 유리수의 덧셈을 구현한다면 아래와 같은 두 함수가 필요할 것입니다.

~~~scala
def addRationalNumerator(n1: Int, d1: Int, n2: Int, d2: Int): Int
def addRationalDenominator(n1: Int, d1: Int, n2: Int, d2: Int): Int
~~~

그러나 이 작업은 모든 분자와 분모를 관리해야 하기 때문에 귀찮은 작업이 될 수 있습니다.

그래서 분자와 분모를 하나의 data structure로 묶어 사용하는 것이 좋은 방법이 될 수 있습니다.

scala에서는, 이 작업을 *class*라는 것을 이용해서 구현할 수 있습니다.

~~~scala
class Rational(x: Int, y: Int) {
  def numer = x
  def denom = y
}
~~~

우리는 Rational이라는 새로운 타입을 얻었고, 생성자 'Rational'을 통해 이 타입의 element를 생성할 수 있습니다.

클래스 타입의 element를 객체(object)라고 합니다.

객체를 생성하려면 클래스의 생성자와 new 연산자를 함께 사용해야 합니다.

~~~scala
new Rational(1, 2)
~~~

이 코드로 분자가 1, 분모가 2인 Rational 클래스의 객체를 생성하게 됩니다.

아래 코드로는 Rational object를 생성해 x에 저장하고, 분자와 분모를 가지고 올 수 있습니다.

~~~scala
val x = new Rational(1, 2)
x.numer // res0: Int = 1
x.denom // res1: Int = 2
~~~

Rational 클래스는 두 개의 멤버 numer와 denom을 가지고 있고, Java와 똑같이 infix operator '.'을 통해 객체의 멤버에 접근할 수 있습니다.

### 유리수 연산 구현 - 덧셈

유리수끼리의 연산 중 덧셈을 구현해보겠습니다.

만약, 덧셈 함수를 클래스 내부가 아닌 외부에서 정의한다고 하면 아래와 같이 정의해야 할 것입니다.

~~~scala
def addRational(r: Rational, s: Rational): Rational = 
  new Rational(
    r.numer * s.denom + s.numer * r.denom,
    r.denom * s.denom)
~~~

또한, 유리수 값을 보려면 아래와 같이 분자와 분모를 슬래시('/')로 나눠주는 함수도 필요할 것입니다.

~~~scala
def makeString(r: Rational) = 
  r.numer + "/" + r.denom
~~~

$${1 \over 2} + {2 \over 3}$$ 수식의 결과는 아래와 같이 확인할 수 있습니다.

~~~scala
makeString(addRational(new Rational(1, 2), new Rational(2, 3))) // 7/6
~~~



하지만 우리는 이러한 함수들을 Rational 클래스 내부에 구현하여 함수들을 패키지화(package) 할 수 있습니다.

따라서, `Rational` 클래스는 `numer`와 `denom`뿐만 아니라 함수 `add`, `sub`, `mul`, `div`, `equal`, `toString` 도 가질 수 있습니다.

add 함수는 아래처럼 구현할 수 있습니다.

~~~scala
class Rational(x: Int, y: Int) {
  def numer = x
  def denom = y
  
  def add(that: Rational) = 
    new Rational(
      numer * that.denom + that.numer * denom,
      denom * that.denom)
  
  override def toString = numer + "/" + denom
}
~~~

아래 코드로 테스트해보면 `7/6`이 print되는 것을 확인할 수 있습니다.

~~~scala
val x = new Rational(1, 2)
val y = new Rational(2, 3)
println(x.add(y).toString)	// 7/6
~~~



### 유리수 추가 구현

1. 현재 유리수와 부호가 반대인 유리수를 만드는 메소드 `neg` 구현하기
   이 작업은 단순히 현재 객체의 numer 멤버에 부호만 반대로 하여 새로운 객체를 만들면 됩니다.
   아래 함수는 당연히 Rational 클래스 내부에 구현되어 있어야 합니다.

   ~~~scala
   def neg: Rational = new Rational(-numer, denom)
   ~~~

2. 뺄셈을 위한 메소드 `sub` 구현하기

   메소드 `sub`은 메소드 `add`의 구현에서 `+ `  기호만  `-`로 바꾸어 아래와 같이 구현할 수 있습니다.

   ~~~scala
    def sub(that: Rational) = 
       new Rational(
         numer * that.denom - that.numer * denom,
         denom * that.denom)
   ~~~

   그러나 우리는 위에 현재 유리수와 부호가 반대인 유리수를 만드는 메소드 `neg`를 구현하였으므로 메소드 `sub`를 아래와 같이 간단하게 구현할 수 있습니다.

   ~~~scala
   def sub(that: Rational) = add(that.neg)
   ~~~

   이렇게 구현하면 add에서 사용했던 식을 sub를 구현할 때 반복해서 적지 않아도 됩니다.
   이렇게 식을 반복해서 쓰지 않는 것을 **dry principle**이라고 합니다.