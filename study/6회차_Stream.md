# Stream

스트림은 자바 8에서 추가된 기술로 배열이나 컬렉션을 다루는데 강력한 기능을 제공한다. 컬렉션은 데이터의 보관을 목적으로 하지만, 스트림은 데이터를 꺼내 계산하는 용도로 사용한다. 데이터에 여러가지 작업을 하나의 스트림으로 간결하게 표현할 수 있다.

## 특징

- 내부 반복으로 작업을 처리한다
- 단 한번만 사용할 수 있다
- 원본의 데이터를 변경하지 않는다
- 필터-맵 기반의 API를 사용하여 지연 연산을 통해 성능을 최적화 한다
- 컬렉션의 `parallelStream()`를 사용한 병렬 처리를 지원한다.

스트림은 원본 데이터를 그대로 두고, 별도의 공간에서 작업하여 결과물을 만들어낸다.

이미 연산을 마쳐 결과로 반환된 스트림은 재사용 할 경우 에러가 발생한다. 결과물의 재사용이 필요한 경우 먼저 결과물을 다른 컬렉션 등에 보관 후 다시 스트림을 사용하면 된다.

for문, foreach문 처럼 반복 연산이 외부에 드러나지 않고 내부적으로 처리되어 **가독성**이 좋다.

## 구조

![https://www.tcpschool.com/lectures/img_java_stream_operation_principle.png](https://www.tcpschool.com/lectures/img_java_stream_operation_principle.png)

스트림은 작업할 데이터를 나타내는 스트림, 데이터를 가공하는 등 연산을 수행할 중개 연산(intermediate operations), 마지막으로 결과물을 내는 최종 연산(terminal operations)으로 구분된다.

중개 연산은 필터링, 정렬, 반복 연산 등 여러가지 작업을 조합하여 사용할 수 있다. 주의할 점으로는 최종 연산을 수행하지 않으면 중개 연산의 작업이 수행되지 않는다.

## 생성

```java
// 배열
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3); // index 1~(3-1) [b, c]
// 컬렉션 (Collection<E>)
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); // 병렬 처리 스트림
// 가변 매개변수
Stream<Integer> stream = Stream.of(1, 2, 3);
// 빈 스트림
Stream<String> emptyStream = Stream.empty();
// iterate
Stream<Integer> iteratedStream
	= Stream.iterate(30, n -> n + 2).limit(5); // (초기값, 반복 연산) [30, 32, 34, 36, 38]
// 기본 타입형
IntStream intStream = IntStream.range(1, 5); // (시작값, 종료값 제외) [1, 2, 3, 4]
LongStream longStream = LongStream.rangeClosed(1, 5); // (종료값 포함) [1, 2, 3, 4, 5]
// 스트림 연결
Stream<String> concat = Stream.concat(stream1, stream2);
```

## 중개 연산

- 필터링 : filter, distinct
- 변환 : map, flatMap
- 제한 : limit, skip
- 정렬 : sorted
- 연산 결과 확인 : peek

### 필터링

`distinct()`는 스트림의 **중복** 요소를 제거한 새로운 스트림을 반환한다. 내부적으로 `Object.equals()`를 사용한다. 스트림에 사용할 클래스를 직접 구현하는 경우, 해당 메서드를 구현해야 제대로 동작한다.

`filter()`는 주어진 **조건**에 맞는 요소들만으로 구성된 스트림을 반환한다.

```java
IntStream stream1 = IntStream.of(1, 1, 2, 3, 4, 5);
IntStream stream2 = IntStream.of(1, 1, 2, 3, 4, 5);

stream1.distinct(); // [1, 2, 3, 4, 5]
stream2.filter(n -> n % 2 != 0); // [1, 1, 3, 5]
```

### 변환

`map()`은 스트림의 요소들을 주어진 함수에 인수로 전달하여 그 결과값들으로 이루어진 스트림을 반환한다. 값이나 타입이 변할 수 있다.

`flatMap()`은 다차원 구조의 스트림을 한 단계 낮춘 스트림을 반환한다. 객체의 중첩구조에서 원하는 데이터를 가공할 수 있게 한다.

```java
Stream<String> stream = Stream.of("HTML", "CSS", "JAVA", "JAVASCRIPT");
stream.map(s -> s.length()); // [4, 3, 4, 10]

String[] arr = {"abc", "123"};
Arrays.stream(arr).flatMap(s -> Stream.of(s.split(""))); // [a, b, c, 1, 2, 3]
```

### 제한

`limit()`은 해당 스트림의 첫 번째 요소부터 주어진 개수만큼의 요소로 이루어진 스트림을 반환한다.

`skip()`은 해당 스트림의 첫 번째 요소부터 주어진 개수만큼의 요소를 제외한 나머지로 이루어진 스트림을 반환한다.

```java
IntStream stream1 = IntStream.range(0, 5);
IntStream stream2 = IntStream.range(0, 5);
IntStream stream3 = IntStream.range(0, 5);

stream2.limit(3); // [0, 1, 2]
stream1.skip(2); // [2, 3, 4]
stream3.skip(2).limit(2); // [2, 3]
```

### 정렬

`sorted()`는 주어진 비교자(Comparator)를 이용하여 정렬한다. 비교자를 전달하지 않으면 기본적으로 오름차순으로 정렬한다. 내림차순으로 정렬하려면 `Comparator.reverseOrder()`를 사용할 수 있다.

```java
IntStream stream1 = IntStream.of(2, 4, 5, 1, 3);
IntStream stream2 = IntStream.of(2, 4, 5, 1, 3);

stream1.sorted(); // [1, 2, 3, 4, 5]
stream2.sorted(Comparator.reverseOrder()); // [5, 4, 3, 2, 1]
```

### 연산 결과 확인

`peek()`은 스트림의 내용을 수정하지 않고 확인하는 용도로 사용한다. 반환 타입이 void인 메서드를 전달받아 사용하며, 주로 `System.out.println()`을 사용한다.

```java
IntStream stream = IntStream.of(2, 4, 5, 1, 3);
stream.peek(s -> System.out.println("origin : " + s))
    .skip(2)
    .peek(s -> System.out.println("skip : " + s))
    .limit(2)
    .peek(s -> System.out.println("limit : " + s))
    .sorted()
    .peek(s -> System.out.println("sorted : " + s))
    .forEach(n -> System.out.println(n));
/*
origin : 2
origin : 4
origin : 5
skip : 5
limit : 5
origin : 1
skip : 1
limit : 1
sorted : 1
1
sorted : 5
5
*/
```

위 결과에서 보이듯, 스트림은 각 중개 연산에서 모든 요소들이 수행된 후 다음으로 넘어가는 것이 아니라, 한 요소마다 스트림을 따라 모든 연산 후 다음 요소가 수행된다. 그러나 `sorted()`의 경우 현재 스트림의 모든 값을 확인할 수 있어야 하기 때문에 이전 작업이 모두 끝난 후 수행된다.

`limit()`에서 2개의 요소를 처리 후 남은 값은 아예 아무 연산도 수행되지 않음을 주의해야 한다. 이렇듯 필터(filter, distinct), 제한(limit, skip) 연산자의 순서를 잘 활용하면 내부적으로 불필요한 연산의 횟수를 줄일 수 있다.

## 최종 연산

스트림의 모든 중개 연산의 결과로 변환된 스트림을 사용해 결과를 표시한다. 지연되었던 실제 연산이 이 때 수행된다.

- 반복 : forEach
- 소모 : reduce
- 검색 : findFirst, findAny
- 검사 : anyMatch, allMatch, noneMatch
- 통계 : count, min, max
- 연산 : sum, average
- 수집 : collect

### 반복

`forEach()`는 반환 타입이 void인 메서드를 입력받아 수행한다. 보통 스트림의 모든 요소를 출력하는 용도로 사용한다.

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4);
stream.forEach(System.out::print); // 1234
```

### 소모

`reduce()`는 `BinaryOperator<T>` 타입의 메서드를 받아 첫 번째 요소와 두 번째 요소로 연산을 수행 후, 그 결과값과 세 번째 요소로 연산을 수행하는 식으로 반복한다.

`BinaryOperator<T>`는 같은 타입의 인자 두 개를 받아 같은 타입의 결과를 반환하는 인터페이스다.
`reduce()`는 총 3개의 파라미터를 받을 수 있다.

- accumulator : 각 요소를 처리하는 계산 로직. 중간 결과를 생성하여 계산을 계속 반복한다.
- identity : 계산을 위한 초기값으로, 스트림이 비어 계산을 하지 않더라도 이 값을 반환한다.
- combiner : 병렬 스트림에서 나눠 계산한 결과를 하나로 합치는 로직이다.

```java
// 1개 (accumulator)
Optional<T> reduce(BinaryOperator<T> accumulator);
// 2개 (identity)
T reduce(T identity, BinaryOperator<T> accumulator);
// 3개 (combiner)
<U> U reduce(U identity,
  BiFunction<U, ? super T, U> accumulator,
  BinaryOperator<U> combiner);
```

```java
// 1개
OptionalInt reduced = IntStream.range(1, 4) // [1, 2, 3]
  .reduce(Integer::sum); // 1 + 2 + 3 = 6
// 2개
int reducedTwoParams = IntStream.range(1, 4) // [1, 2, 3]
  .reduce(10, Integer::sum); // 10 + 1 + 2 + 3 = 16
// 3개
Integer reducedParallel = Arrays.asList(1, 2, 3)
  .parallelStream()
  .reduce(10, Integer::sum, Integer::sum); // (10 + 1) + (10 + 2) + (10 + 3) = 36
```

위에서 보듯 파라미터 3개를 사용하는 경우엔 동작 방식이 다르다. 각 요소 하나마다 하나의 쓰레드에 할당되어 초기값과 연산을 수행한다. 이 후 각 쓰레드의 연산 결과는 `combiner`로 하나로 합쳐 결과를 반환한다. 3개의 파라미터를 갖는 `reduce()`를 사용할 때 `parallelStream()` 대신 `stream()`을 사용할 경우 `combiner`의 로직은 수행되지 않는다.

만약 결과 값이 null 이라면, 널 포인터 에러가 발생한다.

### 검색

`findFirst(), findAny()`는 해당 스트림에서 첫 번째 요소를 참조하는 `Optional` 객체를 반환한다. 병렬 스트림에서는 `findAny()`를 사용해야만 올바른 결과값이 반환된다.

```java
IntStream stream = IntStream.of(4, 2, 7, 3, 5, 1, 6);
OptionalInt result = stream.sorted().findFirst(); // 1
```

만약 스트림이 비어있으면 괜찮지만, 결과 값이 null 이라면 널 포인터 에러가 발생한다.

### 검사

스트림의 요소 중 주어진 조건을 만족하는 값이 있는지 확인하여 boolean 값을 반환한다.

- anyMatch() : 하나라도 만족할 경우 true
- allMatch() : 모든 요소가 만족해야 true
- noneMatch() : 모든 요소가 만족하지 않아야 true

```java
IntStream stream = IntStream.of(30, 90, 70, 10);
boolean result = stream.anyMatch(n -> n > 80); // true
```

### 통계

`count()`는 스트림의 요소의 총 개수를 long타입으로 반환한다.

`max(), min()`은 스트림의 요소 중 가장 큰 값과 작은 값을 참조하는 `Optional` 객체를 반환한다. 만약 스트림이 비어있으면 괜찮지만, `max(), min()`의 결과 값이 null 이라면, 널 포인터 에러가 발생한다.

```java
IntStream stream1 = IntStream.of(30, 90, 70, 10);
IntStream stream2 = IntStream.of(30, 90, 70, 10);

long count = stream1.count(); // 4
OptionalInt max = stream2.max(); // 90
```

### 연산

기본 타입 스트림에는 스트림의 모든 요소의 합과 평균을 구할 수 있는 `sum(), average()` 메서드가 정의되어 있다. `average()`는 각 기본 타입으로 래핑된 `Optional` 객체를 반환한다.

```java
IntStream stream1 = IntStream.of(30, 90, 70, 10);
DoubleStream stream2 = DoubleStream.of(30.3, 90.9, 70.7, 10.1);

int sum = stream1.sum(); // 200
OptionalDouble average = stream2.average(); // 50.5
```

### 수집

`collect()`에인수로 전달되는 `Collectors` 객체의 구현 방법에 따라 스트림의 요소를 수집한다.

- 배열, 스트림으로 변환 : toArray, toCollection, toList, toSet, toMap
- 통계, 연산 : counting, maxBy, minBy, summingInt, averageingInt 등
- 소모 : reducing, joining
    - reducing : 최종 연산의 소모와 같은 기능
    - joining : delimiter, prefix, suffix(구분자, 접두사, 접미사)를 입력받아 문자열을 받환한다.
- 그룹화, 분할 : groupingBy, partitioningBy
    - groupingBy : 인자로 받은 값의 결과로 그룹핑한 Map을 반환한다. Map의 각 값은 List 타입이다.
    - partitioningBy : 인자로 받은 조건식의 결과로 구분한 Map을 반환한다. 키는 true, false이며, 값은 List 타입이다.

```java
// 변환
Stream<Integer> stream = Stream.of(1, 2, 3, 4);
List<Integer> list = stream.collect(Collectors.toList()); // [1, 2, 3, 4]
// 통계, 연산
Stream<Integer> stream = Stream.of(1, 2, 3, 4);
Optional<Integer> max = stream.collect(Collectors.maxBy()); // 4
// 소모
Stream<Integer> stream = Stream.of(1, 2, 3, 4);
String str = stream.collect(Collectors.joining()); // 1234
// 그룹화
Map<Integer, List<Product>> collectorMapOfLists =
 productList.stream()
  .collect(Collectors.groupingBy(Product::getAmount));
/*
{23=[Product{amount=23, name='potatoes'}, 
     Product{amount=23, name='bread'}], 
 13=[Product{amount=13, name='lemon'}, 
     Product{amount=13, name='sugar'}], 
 14=[Product{amount=14, name='orange'}]}
*/
```

같은 기능을 하는 최종 연산 `reduce(), min(), max()`의 경우 결과 값이 null 이면 널 포인터 에러를 반환하는 것에 비해 `collect()`는 에러 대신 `Optional` 객체를 반환한다. 널 포인터 에러에 좀 더 유연하게 대처할 수 있다.

만약 통계, 연산에 해당하는 결과값을 여러개 얻고 싶을 경우 `summarizingInt(), summarizingLong(), summarizingDouble()`을 사용할 수 있다. 결과값으로 받은 `IntSummaryStatistics` 객체에는 `getCount(), getSum(), getAverage(), getMin(), getMax()` 가 있어 원하는 값을 얻을 수 있다.

```java
IntSummaryStatistics statistics = 
 productList.stream()
  .collect(Collectors.summarizingInt(Product::getAmount));
// IntSummaryStatistics {count=5, sum=86, min=13, average=17.200000, max=23}
```

# 참고

[https://www.tcpschool.com/java/java_stream_concept](https://www.tcpschool.com/java/java_stream_concept)

[https://futurecreator.github.io/2018/08/26/java-8-streams/](https://futurecreator.github.io/2018/08/26/java-8-streams/)

[https://mangkyu.tistory.com/112](https://mangkyu.tistory.com/112)

번외 문제 - [https://mangkyu.tistory.com/116](https://mangkyu.tistory.com/116)