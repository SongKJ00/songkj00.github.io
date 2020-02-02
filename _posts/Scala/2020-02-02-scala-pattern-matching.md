---
layout: single
classes:
  - wide
title: Scala - Pattern Matching
comments: true
author_profile: true
tags:
  - scala



---

### 패턴 매칭을 이용한 함수적 분해(Functional Decomposition with Pattern Matching)

test 함수(특정 클래스의 인스턴스인지 확인 - java의 isInstance 함수)와 accessor(멤버에 대한 getter) 함수의 본래 목적은 생성 과정(contruction process)를 알아내기 위해서입니다.

결국, 아래 질문에 대한 답을 얻기 위해서입니다.

* 어떤 subclass로 만들었는가?
* 생성자의 파라미터들은 무엇이었는가?

scala에서는 이 질문들에 대한 답을 보다 쉬운 과정으로 얻어낼 수 있습니다.

### Case Classes

아래와 같은 클래스들이 있다고 가정해보겠습니다.

~~~scala
trait Expr
case class Number(n: Int) extends Expr
case class Sum(e1: Expr, e2: Expr) extends Expr
~~~

base가 되는 Expr이 있고, 이를 상속하는 Number와 Sum 클래스가 있습니다.

여기서 Number와 Sum 앞에 `case`라는 키워드가 붙어 있는데, 이 키워드를 붙이게 되면 그냥 class가 아닌 case class가 됩니다.

이렇게 만든 case class는 scala compiler가 아래와 같은 코드로 바꿉니다.

암시적으로 apply 메소드가 있는 companion object를 만들어냅니다.

~~~scala
object Number {
  def apply(n: Int) = new Number(n)
}
object Sum {
  def apply(e1: Expr, e2: Expr) = new Sum(e1, e2)
}
~~~

`Number(2)`

이와 같은 코드는 결국,

scala의 함수 호출 정책에 의해 `Number.apply(2)`로 바뀌고

미리 정의되어 있던 object Number의 apply 메소드가 호출되고, 새로운 인스턴스(`new Number(n)`)를 생성합니다.

즉, factory 함수가 만들어진 것입니다.

따라서 이제 `new Number(1)` 대신 `Number(1)` 이렇게만 써도 같은 결과를 만들어낼 수 있습니다.

이 case class들은 아래 pattern matching에 사용됩니다.

### Pattern Matching

scala에서 패턴 매칭은 C/Java의 `switch`를 클래스 계층(class hierarchies)에도 사용할 수 있게 합니다.

대신, scala에서는 `match`라는 키워드를 사용합니다.

그렇다면 아래와 같은 예제 코드를 보겠습니다.

~~~scala
def eval(e: Expr): Int = e match {
  case Number(n) => n
  case Sum(e1, e2) => eval(e1) + eval(e2)
}
~~~

여기서 `case Number(n) => n`이 의미하는 바는 eval 함수를 호출할 때 파라미터 e가 `Number` case class로 만들고 생성자의 파라미터가 하나이며 n 타입(Int)에 맞는 것이었다면 n을 return한다는 것입니다.

그리고, `case Sum(e1, e2) => eval(e1) + eval(e2)`가 의미하는 바는 파라미터 e가 `Sum` case class로 만들고 생성자의 파라미터가 두 개이며 e1, e2 타입(Expr)에 맞는 것이었다면 `eval(e1) + eval(e2)`의 결과를 return한다는 것입니다.

### Match Syntax

`match` 앞에는 선택자(selector)가 옵니다. 저희는 e라는 이름을 사용했습니다.

`match` 뒤에는 case들이 뒤따르고, 각 case들은 패턴 - pat과 expression - expr으로 구성되어 있습니다.(`pat => expr` 같은 구조임)

~~~scala
e match {
  case pat1 => expr1
  .
  .
  .
  case patn => exprn
}
~~~

여러 패턴들 중에 가장 먼저 match된 패턴의 expression을 평가(evaluate)하게 됩니다.

만약 선택자 e가 어느 패턴과도 match되지 않았다면, `MatchError` 예외가 발생합니다.

### Forms of Patterns

패턴들은 아래에 있는 것들의 조합으로 구성됩니다.

* 생성자(constructors)
  Ex) `Number`, `Sum`
* 변수(variables)
  Ex) `n`, `e1`, `e2`
* wildcard patterns _(underscore)
* 상수(constants)
  Ex) `1`, `true`

패턴을 작성할 때, 아래와 같은 규칙을 따라야 합니다.

* 변수를 나타낼 때는 소문자로 시작해야 합니다.
* 같은 변수 이름은 패턴 안에서 중복 사용될 수 없습니다.
  ex) `Sum(x, x)`는 허용되지 않습니다.
* 상수를 나타낼 때는 대문자로 시작해야 합니다. 그러나 미리 정의된 단어 `null`, `true`, `false` 같은 것들은 예외입니다.
* _(underscore)는 어느 타입이 오든지 상관하지 않는다는 것을 의미합니다.

### Example

예시를 들며 pattern matching이 어떻게 사용되는지 설명해보겠습니다.

evaluation을 진행하는 `eval` 함수가 아래와 같이 정의되어 있고  pattern matching을 사용합니다.

~~~scala
def eval(e) = e match {
  case Number(n) => n
  case Sum(e1, e2) => eval(e1) + eval(e2)
}
~~~

그렇다면 `eval(Sum(Number(1), Number(2)))`는 어떻게 수행될까요?

~~~scala
eval(Sum(Number(1), Number(2)))
~~~

`eval` 함수의 선택자 e가 `Sum(Number(1), Number(2))`로 바뀝니다.

&rarr;

~~~scala
Sum(Number(1), Number(2)) match {
  case Number(n) => n
  case Sum(e1, e2) => eval(e1) + eval(e2)
}
~~~

`Sum(e1, e2)` 패턴에 매칭되므로 해당 패턴의 expression이 평가됩니다. 여기서, e1, e2는 각각 Number(1), Number(2)로 바뀌게 됩니다.

&rarr;

~~~
eval(Number(1)) + eval(Number(2))
~~~

&rarr;

~~~scala
Number(1) match {
  case Number(n) => n
  case Sum(e1, e2) => eval(e1) + eval(e2)
} + eval(Number(2))
~~~

여기서도 마찬가지로 n이 1로 바뀝니다.

&rarr;

~~~scala
1 + eval(Number(2))
~~~

나머지 eval(Number(2))도 이제 패턴 매칭을 통해 2라는 결과가 나옵니다. 따라서 1+2가 수행되므로 3이 나오게 됩니다.

&rarr;

~~~scala
3
~~~

### Pattern Matching and Methods

**Example**에서 얘기한 eval 함수는 어느 클래스에도 속해있는 메소드가 아닌 함수였지만 기본 trait의 메소드로도 정의할 수 있습니다.

~~~scala
trait Expr {
  def eval: Int = this match {
    case Number(n) => n
    case Sum(e1, e2) => e1.eval + e2.eval
  }
}
~~~



### Pattern Matching이 필요한 이유 / Pattern Matching의 이점

그렇다면 pattern matching이 필요한 이유 / 이점이 무엇일까요?

자바와 비교해본다면 첫번째로, 추상(기본) 클래스에 새로운 메소드들이 생겨날 때마다, 모든 서브 클래스의 body에 이를 정의해줘야 하는 작업을 덜 수 있습니다.

scala에서는 기본 trait인 `Expr`에 있는 eval 메소드에 각 서브 클래스별 expression을 정의할 수 있습니다.

그러나 만약 자바에서 이 작업을 수행하려 했다면 Number 클래스의 body에 `eval` 메소드를 새롭게 구현해주고 마찬가지로 Sum 클래스의 body에도 `eval` 메소드를 구현해줘야 할 것입니다.

모든 서브 클래스들이 한 파일에 있다면 그나마 상황이 낫겠지만, 만약 여러 파일에 있다면 매번 그 파일들을 찾아가 일일이 `eval` 메소드를 정의해줘야 할 것입니다.

