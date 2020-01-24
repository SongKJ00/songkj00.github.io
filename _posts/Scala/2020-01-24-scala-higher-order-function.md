---
layout: single
classes:
  - wide
title: Scala - Higher order function이란
comments: true
author_profile: true
tags:
  - scala

---

[이 페이지](https://docs.scala-lang.org/tour/higher-order-functions.html)를 보고 공부한 내용을 정리하였습니다.

Higher order function은 파라미터로 다른 함수를 받거나 결과로 함수를 리턴하는 함수를 말합니다.

higher order function의 가장 흔한 예는 바로 **map**입니다.

~~~scala
val salaries = Seq(20000, 70000, 40000)
val doubleSalary = (x: Int) => x * 2
val newSalaries = salaries.map(doubleSalary) // List(40000, 140000, 80000)
~~~

`doubleSalary` 는 `Int`형 파라미터 `x`를 하나 받고, `x*2`를 return하는 함수입니다.
`=>` 왼쪽에 있는 tuple(여기서는 `(x: Int)`)은 파라미터 리스트이고, 오른쪽에 있는 것은 return할 값(value of the expression)입니다.
마지막 라인에서, `doubleSalary`의 내용이 `salaries` 리스트의 각 요소에 적용됩니다.

function을 익명으로(anonymous) 만들고 map의 파라미터로 직접 넘겨 위 코드를 더욱 간결하게 만들 수 있습니다.

~~~scala
val salaries = Seq(20000, 70000, 40000)
val newSalaries = salaries.map(x => x * 2) // List(40000, 140000, 80000)
~~~

이 예에서는 `x`가 `Int`라는 게 명시적으로 표시되지 않았습니다.
그 이유는 `map`을 사용하는 `salaries`의 요소가 `Int`형이기 때문에 컴파일러가 `x`는 `Int`라는 것을 예측할 수 있기 때문입니다.
(C++의 auto 키워드와 비슷한 것이지 않을까 싶습니다.)
위 코드를 더욱 이상적으로 바꿀 수 있습니다.

~~~scala
val salaries = Seq(20000, 70000, 40000)
val newSalaries = salaries.map(_ * 2)
~~~

 Scala 컴파일러가 이미 파라미터의 타입을 알기 때문에(여기서는 `Int`) function에서 `=>` 의 오른쪽에 해당하는 부분만 적어줘도 됩니다.
이 때는 파라미터 이름으로 `_` (underscore)를 사용해야 합니다.

### method를 function으로의 변환(Coercing methods into functions)

Scala 컴파일러는 method를 function으로 변형하기 때문에 higher-order function에 method를 파라미터로 넘기는 것이 가능합니다.

~~~scala
case class WeeklyWeatherForecast(temperatures: Seq[Double]) {

  private def convertCtoF(temp: Double) = temp * 1.8 + 32

  def forecastInFahrenheit: Seq[Double] = temperatures.map(convertCtoF) // <-- passing the method convertCtoF
}
~~~

이 예제에서 method `convertCtoF` 가 higher-order function인 `map` 의 파라미터로 넘겨집니다.
이것이 가능한 이유는 scala 컴파일러가 method `convertCtoF` 를 function `x => convertCtoF(x)`로 변형하기 때문입니다.

### function을 파라미터로 쓰는 function(Functions that accept functions)

higher-order function을 사용하는 이유는 반복되는 코드를 줄이기 위해서입니다.
Salaries를 다양한 방법으로 변형시키는 메소드들이 있다고 가정해보겠습니다.
higher-order function을 사용하지 않고 구현한다면, 아래와 같이 구현할 것입니다.

~~~scala
object SalaryRaiser {

  def smallPromotion(salaries: List[Double]): List[Double] =
    salaries.map(salary => salary * 1.1)

  def greatPromotion(salaries: List[Double]): List[Double] =
    salaries.map(salary => salary * math.log(salary))

  def hugePromotion(salaries: List[Double]): List[Double] =
    salaries.map(salary => salary * salary)
}
~~~

이 세 개의 method를 보면 단순히 곱하는 수(multiplication factor)만 다릅니다.

이 예제를 간단히 바꾼다면, 반복되는 코드를 higher-order function으로 구현하면 됩니다.

~~~scala
object SalaryRaiser {

  private def promotion(salaries: List[Double], promotionFunction: Double => Double): List[Double] =
    salaries.map(promotionFunction)

  def smallPromotion(salaries: List[Double]): List[Double] =
    promotion(salaries, salary => salary * 1.1)

  def greatPromotion(salaries: List[Double]): List[Double] =
    promotion(salaries, salary => salary * math.log(salary))

  def hugePromotion(salaries: List[Double]): List[Double] =
    promotion(salaries, salary => salary * salary)
}
~~~

새롭게 생긴 method `promotion` 은 salaries를 파라미터로 받고, `Double => Double` 형태의 function(파라미터와 return 타입 모두 Double)도 파라미터로 받습니다.
return은 salaries를 promotionFunction으로 map한 결과입니다.

method나 function은 일반적으로 어떤 동작이나 데이터의 변형을 표현합니다.
그러므로, 다른 함수에 기반을 둔 함수를 구현하는 것은 공통의 메카니즘을 만드는 데 유리합니다.



### function을 return하는 function(Functions that return functions)

function을 생성해야 하는 특별한 경우가 있습니다.
아래는 function을 return하는 method의 예입니다.

~~~scala
def urlBuilder(ssl: Boolean, domainName: String): (String, String) => String = {
  val schema = if (ssl) "https://" else "http://"
  (endpoint: String, query: String) => s"$schema$domainName/$endpoint?$query"
}

val domainName = "www.example.com"
def getURL = urlBuilder(ssl=true, domainName)
val endpoint = "users"
val query = "id=1"
val url = getURL(endpoint, query) // "https://www.example.com/users?id=1": String
~~~

urlBuilder의 return 타입은 `(String, String) => String` 입니다.
이 return 타입의 의미는 return되는 anonymous function은 두 개의 String 타입을 파라미터로 받고, String을 return한다는 것입니다.
이 예제에서는 return되는 anonymous function은 ``(endpoint: String, query: String) => s"https://www.example.com/$endpoint?$query"`` 입니다.