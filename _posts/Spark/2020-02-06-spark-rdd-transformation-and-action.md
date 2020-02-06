---
layout: single
classes:
  - wide
title: Spark - RDD, Transformation and Action
comments: true
author_profile: true
tags:
  - scala



---

Transformation과 action은 모두 RDD의 operation이지만 동작 방법이 조금 다릅니다.

Transformation과 action에 대해 이해하기 전에 Scala 컬렉션의 transformer와 accessor에 대해 먼저 살펴보겠습니다.

### Transformers and Accessors

Transformer는 결과로서 새로운 컬렉션을 반환하는 operation입니다.

Transformer의 예로는 map, filter, flatMap, groupBy 등이 있습니다.

~~~scala
map(f: A => B): Traversable[B]
~~~

만약, 리스트의 map 함수를 사용하면 새로운 리스트를 얻게 되므로 map을 transformer라고 하는 이유입니다.



그에 반대로, Accessor는 결과로서 새로운 컬렉션이 아닌 하나의 값(single value)만을 반환하는 operation입니다.

예로는 reduce, fold, aggregate 등이 있습니다.

~~~scala
reduce(op: (A, A) => A): A
~~~

reduce 함수를 사용하면 파라미터로 넘겨진 op 함수를 통해 single value가 나오게 됩니다.



### Transformations and Actions

Transformation과 Action은 위에서 얘기한 Transformer와 Accessor와 비슷한 것들입니다. 그러나 매우 중요한 차이점이 있습니다.

Transformation은 transformer와 비슷합니다. 

transformer가 새로운 컬렉션을 결과로서 반환했다면, transformation은 새로운 RDD를 결과로서 반환합니다.

Action은 accessor와 비슷합니다.

accessor은 single value를 결과로서 반환했지만, action은 RDD를 기반으로 결과를 계산해 RDD가 아닌 값을 반환하거나 외부 저장 시스템(e.g., HDFS)에 저장합니다.

또한 이것이 제일 중요한 차이점인데 **Transformation은 lazy하기 때문에 transformation의 결과인 RDD는 즉시 계산되지 않지만 Action은 eager하기 때문에 결과가 즉시 계산된다는 점입니다.**

이것이 scala 컬렉션과 매우 큰 차이점입니다.

**즉, scala 컬렉션에서 map을 사용하면 바로 결과(새로운 컬렉션)가 계산되어 return 되겠지만 RDD에서의 map은 호출 즉시 return할 결과를 위한 계산이 이뤄지지 않는다는 뜻입니다.**

이 laziness와 eagerness가 중요한 이유는 결국 spark가 네트워크를 처리하는 것에 있어 영향을 미치기 때문입니다.

transformation은 lazy하고 action은 eager한 이 특징은 spark 작업을 수행할 때 그 밑단에서 일어나는 네트워크 통신을 상당히 줄여주게 됩니다.



### Example

아래와 같은 예제가 있을 때, 마지막 라인까지 끝나면 클러스터에서는 어떤 일이 생길까요?

sc는 SparkContext입니다.

~~~scala
val largeList: List[String] = ...
val wordsRdd = sc.parallelize(largeList)	// RDD[String]
val lengthsRdd = wordsRdd.map(_.length)		// RDD[Int]
~~~

**정답은 아무런 일도 일어나지 않습니다.**

RDD에서의 map은 transformation이기 때문에 실행이 연기되어 아무런 일도 일어나지 않습니다.

lengthsRdd는 결국 RDD의 레퍼런스이고 실제 참조하는 RDD는 아직 생성되지도 않았습니다.

그렇다면 클러스터에서 이 map이 완료되었다는 것을 어떻게 알 수 있을까요?

map이 수행되고, 그 결과를 반환받고 싶다면, action operation을 추가하면 됩니다.

~~~scala
val largeList: List[String] = ...
val wordsRdd = sc.parallelize(largeList)	// RDD[String]
val lengthsRdd = wordsRdd.map(_.length)		// RDD[Int]
val totalChars = lengthsRdd.reduce(_ + _) // Add action operation
~~~

RDD의 action operation 중 하나인 reduce를 이용하여 전체 문자열의 문자 갯수를 얻을 수 있게 되었습니다.

Spark 입문자들이 쉽게 실수할 수 있는 것이 바로 map이나 filter 같은 transformation operation들이 호출과 동시에 실행되고 결과를 반환받을 거라 생각한다는 것입니다.

실제로는 아무런 일도 일어나지 않는데 말이죠.

결국 action operation이 있어야 그제서야 결과를 반환받을 수 있습니다.



### Benefits of Laziness for Large-Scale Data

그렇다면 이런 lazy한 transformation을 사용해서 얻는 이점이 무엇일까요?

spark가 RDD를 계산하는 타이밍은 바로 해당 RDD에 대해 action operation이 사용되었을 때입니다.

이는 곧 매우 큰 규모의 데이터들을 처리할 때 이점이 될 수 있습니다.

RDD[String]에서 "ERROR"라는 문자열이 포함된 문장들 중에 맨 처음 10개의 문장들만 추출하려 하는 예제가 있습니다.

~~~scala
val lastYearsLogs: RDD[String] = ...
val firstLogsWithErrors = lastYearsLogs.filter(_.contains("ERROR")).take(10)
~~~

`filter`는 transformation이니 `take` action이 수행되기 전까지 연기됩니다.

위 예제에서, `filter`의 조건을 만족하는 10개의 문장을 찾게 되면 `filter` 작업은 바로 종료됩니다.

이는 transformation이 lazy하기 때문에 얻는 장점인데, 만약 lazy하지 않았다면 클러스터에 퍼져있는 모든 RDD를 뒤지면서 "ERROR" 문자열이 포함된 문자열을 모두 찾아 새로운 RDD를 만들어 내고, 그 중에서 맨 앞 10개를 골라냈을 것입니다.

그러나 생각해보면, 우리가 필요한 건 결국 맨 앞 10개이므로 그 뒤에 있는 것은 굳이 찾을 필요도, 메모리에 들고 있을 필요도 없습니다.

spark는 이러한 아이디어를 적용시키기 위해 transformation을 lazy하게 구현했고, action operation이 수행될 때 transformation이 수행되도록 설계했습니다.

이로 인해 컴퓨팅 시간을 보다 크게 줄일 수 있게 됩니다.



### Other Useful RDD Actions

RDD에만 있고 scala 컬렉션에는 없는 꽤 중요한 action operation들이 있습니다.

이는 분산된 데이터들을 다룰 때 유용합니다.

* `takeSample` - `takeSample(withRepl: Boolean, num: Int): Array[T]`
  * RDD로부터 랜덤적으로 n개의 데이터를 뽑아 array를 return합니다.
  * 리턴값이 array이므로 RDD처럼 분산되어 존재하는 게 아니라 하나의 머신에만 존재합니다.
* `takeOrdered` - `takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]`
  * RDD의 맨 처음 n 개의 데이터들을 정렬된 순서로 담아 array로 return합니다.
* `saveAsTextFile` - `saveAsTextFile(path: String): Unit`
  * RDD의 데이터들을 텍스트 파일로서 파일 시스템이나 HDFS에 저장합니다.
* `saveAsSequenceFile` - `saveAsSequenceFile(path: String): Unit`
  * RDD의 데이터들을 로컬 파일 시스템에 Hadoop SequenceFile로서 저장하거나 HDFS에 저장합니다.