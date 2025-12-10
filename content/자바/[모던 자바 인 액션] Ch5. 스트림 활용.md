스트림에서 데이터의 처리 방법은 주로 스트림 API가 관리하기에 편안한 데이터 처리 기법을 제공한다. 이 말은 즉, 스트림 API 내부적으로 다한 최적화(코드의 병렬 실행 등)이 이루어 질 수 있다는 것이다. 이러한 것은 순차적인 반복을 단일 스레드로만 구현하는 외부 반복에서 처리할 수 없다.

해당 장에서는 이러한 스트림API가 지원하는 다양한 연산을 배울 것이며 다음과 같다.

- 필터링, 슬라이싱, 매칭
- 검색, 매칭, 리듀싱
- 특정 범위의 숫자와 같은 숫자 스트림 사용
- 다중 소스로부터 스트림 생성
- 무한 스트림

# 5.1 필터링

스트림의 요소를 선택하는 방법, 즉 프레디케이트로 필터링 방법과 고유 요소만 필터링하는 방법에 대해 알 필요가 있다.

### 프레디케이트로 필터링
`Boolean`을 반환하는 프레디케이트를 인수로 받아서 이에 일치하는 모든 요소를 포함하는 스트림을 반환한다.


    ```
    List<Dish> vegetarianMenu = menu.stream()
								.filter(Dish::isVegetarian)
								.collect(toList());
    ```

해당 코드에서 `Dish::isVegetarian`이 프레디케이트로 `filter`함수는 이를 인수로 받아서 필터링한다.


### 고유 요소 필터링

스트림에서 만든 객체의 `hashCode`, `equals`로 결정되는 고유 여부에 의해 판단된 고유 요소로 이루어진 스트림을 반환하는 것이다.


    ```
    List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
    numbers.steram()
	    .filter(i -> i % 2 == 0)
	    .distinct()
	    .forEach(System.out::println)
    ```

해당 코드에서는 리스트의 모든 짝수를 선택 후 중복을 필터링하여 고유 오소만 남도록 한다.

# 5.2  스트림 슬라이싱

스트림의 요소를 선택, 스킵하는 방법을 의미한다.


### 프레디케이트를 이용한 슬라이싱

자바 9에서 스트림 요소를 효과적으로 선택하도록 `takeWhile`, `dropWhile` 의 두 가지 메서드를 지원한다.

해당 메서드들의 효과를 설명하기 위한 데이터 소스는 다음과 같다.
    ```
    List<Dish> specialMenu = Array.asList(
	    new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
	    new Dish("prawns", fasle, 300, Dish.Type.FISH),
	    new Dish("rice", true, 350, Dish.Type.OTHER),
	    new Dish("chicken", false, 400, Dish.Type.MEAT),
	    new Dish("french fries" true, 530, Dish.Type.OTHER)
    );
    ```
#### TakeWhile

위 데이터 소스에서 320 칼로리 이하의 요리를 선택하기 위해 `filter`를 사용한다고 가정하자.


    ```
    List<Dish> filteredMenu = specialMenu.stream()
				.filter(dish -> dish.getCaloreis() < 320)
				.collect(toList());
    ```

`filter` 연산을 이용하면 **전체 스트림을 반복**하며 각 요소에 프레디케이트를 적용하여 처리하게 된다. 이 경우 리스트가 정렬되어 있다는 사실을 이용해 320칼로리보다 크거나 같은 요리가 나온 경우 **반복 작업을 중단**할 수 있다.(filter가 하지 못하는 작업)

그렇지만 여기서 아주 많은 요소가 큰 스트림에서는 연산의 속도에 큰 차이를 줄 수 있다.

이런 경우 `takeWhile` 연산을 이용할 수 있다.


    ```
    List<Dish> slicedMenu1 = specialMenu.strema()
				.takeWhile(dish -> dish.getCalories() < 320)
				.collect(toList());
    ```

이를 이용해 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림을 슬라이스 할 수 있다.

#### DropWhile

위 에시와 반대로 320칼로리보다 큰 요소를 탐색시 `dropWhile`을 이용할 수 있따.


    ```
	List<Dish> slicedMenu2 = specialMenu.stream()
			.dropWhile(dish -. dish.getCaloreis() < 320)
			.collect(toList());
    ```

`dropWhile`은 `takeWhile`과 정반대의 작업을 수행하는데 `dropWhile`의 경우 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버린다. 프레디케이트가 거짓이 되면 그 지점에서 작업을 중단하고 남은 모든 요소를 반환하게 된다.


### 스트림 축소

주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 것으로 `limit` 메서드를 지원한다.


    ```
    List<Dish> dishes = specialMenu.stream()
		    .filter(dish -> dish.getCaloreis() > 300)
		    .limit(3)
		    .collect(toList());
    ```

해당 코드는 프레디케이트와 일치하는 처음 세 요소를 선택한 다음 즉시 결과를 반환하는 코드이다.

### 요소 건너뛰기

처음 n개 요소를 제외한 스트림을 반환하는 `skip` 메서드가 존재한다.

처음 n개를 가져오는`limit` 과 달리 n개 이후 스트림을 반환하기에 상호 보완적인 연산을 수행한다.



    ```
    List<Dish> dishes = menu.stream()
				    .filter(d -> d.getCaloreis() > 300)
				    .skip(2)
				    .collect(toList());
    ```



# 5.3 매핑

특정 객체에서 특정 데이터를 선택하는 작업으로 스트림 API의 `map`, `flatMap` 메서드가 존재한다.

### 스트림의 각 요소에 함수 적용

함수를 인수로 받는 방식으로 `map` 메서드를 지원한다.
인수로 제공된 함수의 경우 각 요소에 적용되며 함수에 적용한결과가 **새로운 요소**가 된다.


    ```
    List<String> dishNames = menu.stream()
				    .map(Dish::getName)
				    .map(Dish::length)
				    .collect(toList());
    ```


### 스트림 평면화

리스트에서 고유 문자로 이루어진 리스트를 반환하는 방법으로 `flatMap`의 메서드가 존재한다.

> [!distinct 이용]
> ```
> 	words.stream()
> 		.map(word -> word.split("")) 
> 		.distinct()
> 		.collect(toList());
> ```
> 위와 같은 방식으로 처리하게 되면 `map`으로 전달한 람다가 각 단어의 String[](문자열 배열)을 반환하는  문제가 존재한다. 즉, 원하는 결과는 `Stream<String>` 인 반면 `Stream<String[]>` 이 된다는 것이다.




    ```
    List<String> uniqueCharacters = words.stream()
		    .map(word -> word.split(""))
		    .flatMap(Arrays::stream) // 생성된 스트림 하나의 스트림으로 평면화
		    .distinct()
		    .collect(toList());
    ```

해당 과정에서 `flatMap`은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다. 즉, map(Arrays::stream)과 달리 `flatMap`은 하나의 평면화된 스트림을 반환하는 것이다.

`map -> flatMap : Stream<String[]> -> Stream<String>`
이렇게 반환되는 결과의 최종 형태는 `List<String>`이다.

결국 `flatMap`메서드는 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하의 스트림으로 연결한다.
# 5.3 검색과 매칭

특정 속성이 데이터 집합에 존재하는 여부를 처리하기 위해 사용된다.

### 프레디케이트가 적어도 한 요소와 일치하는지 확인

주어진 스트림에서 적어도 한 요소와 일치하는지 확인하기 위해 사용하는 경우로 `anyMatch` 메서드를 사용한다.


    ```
    if (menu.stream().anyMatch(Dish::isVegetarian)) {
	    System.out.println("The menu is (somewhat) vegetarian friendly!!);
    }
    ```

### 프레디케이트가 모든 요소와 일치하는지 검사

스트림의 모든 요소가 주어진 프레디케이트와 일치하는 지 검사하기 위해 사용하는 경우로 `allMatch` 메서드가 존재한다.


    ```
    boolean isHealthy = menu.stream()
		    .allMatch(dish -. dish.getCaloreis() < 1000);
    ```

#### Nonematch

`noneMatch` 의 경우 주어진 프레디케이트와 일치하는 요소가 없는지 확인한다.



    ```
    boolean isHealthy = menu.stream()
				    .noneMatch(d -> d.getCaloreis() >= 1000);
    ```

위의 세 메서드(`anyMatch`, `allMatch`, `noneMatch`)는 스트림 **쇼트서킷** 기법을 사용한다.

> [!쇼트서킷 평가]
    > 전체 스트림을 처리하지 않았더라도 결과를 반환할 수 있는 경우가 존재한다. `allMatch`, `noneMatch`, `findFirst`, 등의 연산은 모든 스트림의 요소를 처리하지 않고도 결과를 반환하는 쇼트서킷의상황에서도 작동한다.



### 요소 검색

현재 스트림에서 임의의 요소를 반환한다.
`findAny` 메서드를 주로 사용하며 다른 스트림 연산과 연결해서 사용가능하다.


    ```
    Optinal<Dish> dish = menu.stream()
			    .filter(Dish::isVegetarian)
			    .findAny();
    ```
해당 경우 내부적으로 단일 과정으로 실행할 수 있도록 최적화하는 즉, 쇼트 서킷을 이용해 결과를 찾는 즉시 실행을 종료한다.


### 첫 번째 요소 찾기

일부 스트림에는 이미 논리적인 순서가 정해져있을 수 잇다. 이 경우 첫 번째 요소를 찾기 위해 `findFirst`메서드를 사용한다.


    ```
    List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
    Optional<Integer> firstSquareDivisiableByThree = someNumbers.stream()
	    .map(n -> n * n)
	    .filter(n -> n % 3 == 0)
	    .findFirst(); // 9
    ```

# 5.5 리듀싱

스트림 요소를 조합해 더 복잡한 질의를 표현하기 위한 것으로 모든 스트림 요소를 처리해서 값으로 도출하는 연산이다. 이를 함수형 프로그래밍 언어 용어로 **fold**라고도 부른다.


### 요소의 합

다음 코드처럼 reduce를 이용해 요소의 합을 구할 수 있다.


    ```
    // for-each loop 이용
    int sum = 0;
    for (int x : numbers) {
		sum += x;
	}
	// reduce 이용
	int sum = numbes.stream().reduce(0, (a, b) -> a + b);
    ```

위와 같이 `reduce`를 이용해 반복된 패턴을 추상화하여 처리할 수 있다.

`reduce` 의 인수는 다음과 같다.

- 초깃값 : 0
- 두 요소를 조합해 새로운 값을 만드는 BinaryOperator<`T`>

### 최댓값과 최솟값

`reduce`를 활용해 최대, 최소 값을 찾을 수 있다.


    ```
    Optional<Integer> max = numbers.stream().reduce(Integer::max);
    Optional<Integer> min = numbers.stream().reduce(Integer::min);
    ```

> [!reduce 메서드의 장점과 병렬화]
    > 기존의 단계적 반복으로 합계를 구하는 것과 다르게 `reduce`를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 `reduce`를 실행할 수 있게 된다.
    > 예를 들어, 반복적인 합계에서는 `sum`변수를 공유해야 하기에 쉽게 병렬화하기 어렵다. 강제적으로 동기화시켜도 스레드간의 소모적인 경쟁으로 인해 그 이득은 상쇄된다.




