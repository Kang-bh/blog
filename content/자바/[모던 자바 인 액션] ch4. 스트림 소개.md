> [!Collection의 중요성]
    > 모든 자바 애플리케이션은 Collection을 만들고 처리하는 과정이 포함된다. 이러한 Collection을 이용하여 데이터를 그룹화하고 처리하기에 대부분의 프로그래밍 작업에 사용된다. 


# 4.1 스트림이란 무엇인가?

자바8 API에 추가된 기능으로 다음의 특징을 가진다.

- 스트림을 사용하여 선언형으로 컬렉션 데이터를 처리할 수 있다. 즉, 데이터 컬렉션의 반복을 깔끔하게 처리하는 것이다.
- 멀티스레드 코드의 구현 없이 데이터를 투명하게 병렬로 처리 가능하다.



```
// 기존 자바 7
List<Dish> lowCaloricDishes = new ArrayList<>();
for (Dish dish: menu) {
	if (dish.getCalories() < 400) {
		lowCaloricDishes.add(dish);
	}
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
	public int compare(Dish dish1, Dish dish2) {
		return Integer.compare(dish1.getCalories(), dish2.getCalories());
	}

	List<String> lowCaloricDishesName = new ArrayList<>();
	for(Dish dish: lowCaloricDishes) {
	lowCaloricDishesName.add(dish.getName())};

})

// 자바 8 스트림 이용 코드
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloricDishesName = 
		menu.stream()
			.filter(d -> d.getCalories() < 400) // 400 칼로리 이하의 요리 선택
			.sorted(comparing(Dish::getCalories)) // 칼로리로 요리 정렬
			.map(Dish::getName) // 요리명 추출
			.collect(toList()); // 모든 요리명 리스트에 저장

// 멀티코어 아케틱처에서의 병렬 실행
List<String> lowCaloricDishesName =
		menu.parallelStream()
			.filter(d -> d.getCalories() < 400)
			.sorted(comparing(Dishes::getCalories))
			.map(Dish::getName)
			.collect(toList());
```


자바 7의 코드에서는 lowCaloricDishes 라는 _가비지 변수_ 즉, 중간 변수의 역할만 수행하는 변수가 존재한다.
그러나 자바 8에서는 라이브러리에서 세부 구현을 구현했기에 모두 처리한다.


이러한 스트림은 다음의 장점을 가진다.

- 선언형으로 코드를 구현 : 루프, 조건문 등의 제어 블록 없이 동작 수행 가능하기에 요구사항 변화에 빠른 대응 가능
- 고수준 빌딩 블록으로 이루어진 filter, sorted, map, collect와 같은 함수를 통해 복잡한 데이터 처리 파이프라인을 간단하게 표현할 수 있다.
- 특정 스레딩 모델에 제한되지 않기에 데이터 처리 과정의 병렬화 진행 시 스레드와 락을 걱정할 필요가 없다.
	![[Pasted image 20250512235116.png]]

이러한 스트림 API 특징을 키워드별로 요약하자면 다음과 같다.

- 선언형 : 더 간결하며 가독성의 향상
- 조립 가능 : 유연성의 존재
- 병렬화 : 성능의 최대화

# 4.2 스트림 시작

#### 스트림이란

> 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

- 연속된 요쇼 : 컬렉션과 같이 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스
- 소스 : 컬렉션, 배열 I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비하는 것
- 데이터 처리 연산 : 함수형 프로그래밍 언어에서 일반적으로 지원하는연산과 데이터베이스와 비슷한 연산을 지원
	- filter, map, reduce, find, match, sort


##### 주요 특징
- 파이프 라이닝 : 스트림 연산끼리 연결하여 파이프 라인 구성가능
	- 최적화 방법인 laziness, short-circuiting 존재
- 내부 반복 : 반복자를 이용한 명시적 반복의 컬렉션과 달리 내부 반복을 지원



	```
    import static java.util.stream.Collectors.toList;
    List<String> threeHighCaloricDishNames = 
		    menu.stream()
				.filter(dish -> dish.getCalories() > 300) // 1	
				.map(Dish::getName) // 2
				.limit(3) // 3
				.collect(toList()); // 4

	System.out.pringln(threeHighCaloricDishNames);
    ```

0. menu에 stream 메서드를 호출해서 스트림 획득
	- 데이터 소스 : 메뉴
1. filter : 고칼로리 요리 필터링
2. map : 요리명 추출
3. limit : 선착순 새 개만 획득
4. collect : 결과를 다른 리스트로 저장

위 연산에서 0번째를 통해 연속된 요소를 스트림에 제공한 후 filter, map, limit, collect로 이어지는 일련의 데이터 처리 연산을 적용한다.
해당하는 데이터 처리 연산에서 filter, map, limit 연산은 서로 파이프라인을 형성하도록 스트림을 반환한다.

collect가 호출되기 전까지는 메서드 호출이 저장되며 collect 호출 전까지는 menu에서 아무것도 선택되지 않고 출력 결과도 있지 않다.

진행하는 상세 작업 내용은 다음과 같다.

- filter : 람다를 인수로 받아 스트림에서 특정 요소를 제외한다.
- map : 람다를 이용해 한 요소를 다른 요소로 변환하거나 정보를 추출한다.
- limit : 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 축소시킨다.
- collect : 스트림을 다른 형식으로 변환한다.

# 4.3 스트림과 컬렉션

> 컬렉션과 스트림 모두 연속된 요소 형식의 값을 저자하는 자료구조의 인터페이스를 제공한다.

### 컬렉션과 스트림의 차이점

#### 언제
두 개념의 차이를 가장 두드러지게 보이는 부분은 **언제**이다.

##### 컬렉션
현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조
즉, 컬렉션의 모든 요소는 컬렉션에 추가되기 전에 계산된다. 요소가 추가, 삭제되는 연산의 수행 때마다 컬렉션의 모든 요소를 메로리에 저장하며 미리 계산된 상태여야한다.
##### 스트림
요청할 때에만 계산되는 고정된 자료구조로 스트림에 요소 추가, 삭제가 불가능하다.
즉, 사용자가 요청하는 값만 스트림에서 추출한다는 것으로 사용자 입장에서 해당 변화를 알 수 없다.
이러한 성능은 producer, comsumer의 관계를 형성한다.


> [!DVD와 인터넷 스트리밍]
    > 컬렉션은 DVD에 저장된 영화로 현재 자료구조에 포함된 모든 값을 계산한 다음 파일을 추가하는 경우이다.
    > 스트림은 인터넷 스트리밍으로 필요할 때마다 값을 계산하는 영화에 비유할 수 있다.

#### 단 한번의 탐색
##### 스트림
스트림은 반복자와 마찬가지로 단 한 번만 사용된다. 즉, 탐색된 스트림의 요소는 이미 소비되어 없어진 것이다. 다시 탐색하기 위해선 초기 데이터 소스에서 새로운 스트림의 생성이 필요하다.
그러나 여기서 테이터 소스가 I/O 채널인 경우 반복해서 생성할 수 없다.

##### 컬렉션
컬렉션은 반복 사용할 수 있는 데이터 소스여야 한다.

> [!스트림과 컬렉션의 철학적 접근]
    > 스트림 : 시간적으로 흩어진 값의 집합
    > 컬렉션 : 특정 시간에 모든 것이 존재하는 공간(메모리)에 흩어진 값
    여기서 for-each와 같은 반복자를 통해 곤간에 흩어진 요소에 접근할 수 있다.


#### 외부 반복과 내부 반복
##### 컬렉션
**외부반복** : 컬렉션 인터페이스를 사용하기 위해서는 for-each와 같은 반복자를 이용해서 직접 요소를 반복해야한다.

여기서의 외부적으로 반복한다는 의미는 **명시적**으로 컬렉션 항목을 하나씩 가져와 처리하는 것이다.

##### 스트림
**내부 반복** : 반복을 알아서 처리하고 결과 스트림을 어딘가에 저장해준다.
이러한 내부 반복을 사용하는 스트림은 함수에 어떤 작업을 수행할지만 지정하면 모든 것이 알아서 처리된다.


> 간단하게 차이를 보자면 다음과 같다.
> 장난감이 공, 인형, 책이 있을 때
> 외부 반복 : 공 담기 -> 인형 담기 -> 책 담기
> 내부 반복 : 모두담기


    ```
    // Collection with for-each
    List<String> names = new ArrayList<>();
    for (Dish dish: menu) { // 명시적 순차 반복
	    names.add(dish.getName());
    }
    
    // Collection with 내부적 반복자
	List<String> names = new ArrayList<>();
	Iterator<String> iterator = menu.iterator();
	while(iterator.hasNext()) { // 명시적 반복
		Dish dish = iterator.next();
		names.add(dish.getName());
	}
	
    // Stream
    List<String> names = menu.stream()
					    .map(Dish::getName)
					    .collect(toList());
    ```


###### 내부 반복 사용의 장점

- 작업의 투명한 병렬 처리 가능
- 최적화된 순서로 처리 가능

이러한 내부 반복의 장점을 스트림 라이브러리에서는 데이터 표현, 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다.

반면, for-each를 이용한 외부 반복은 병렬성을 스스로 관리해야한다. 즉, sychronizecd와의 싸움의 시작인 것이다.


![[Pasted image 20250513141914.png]]


### 4.4 스트림 연산

`java.util.stream.Stream` 인터페이스는 많은 연산을 정의한다. 이러한 스트림 인터페이스에서 제공하는 연산은 크게 두 가지로 분류 된다.

#### 중간 연산
연결할 수 있는 스트림 연산으로 위에서 `filter`, `map`, `limit`과 같은 연산들이다.
이런 중간 연산은 다른 스트림을 반환하는데 이를 연결하여 새로운 질의를 생성할 수 있다.

이러한 중간 연산은 다음의 중요 특징을 가진다.

> 단말 연산을 스트림 파이프라인에 실행하기까지 아무 연산도 수행하지 않는다.

이는 **lazy**라는 특징을 가지며 중간 연산을 합친 다음 합쳐진 중간 연산을 최종 연산으로 한 번에 처리하는 것을 의미한다.

이러한 특징은 다음의 코드에서 최적화 효과를 가질 수 있다.


    ```
    List<String> names =
	    menu.stream()
		    .filter(dish -> {
					System.out.println("filtering: " + dish.getName());
					return dish.getCalories() > 300;
			})
			.map(dish -. {
				System.out.println("mapping: " + dish.getName());
				return dish.getName();
			})
			.limit(3)
			.collect(toList());
		
	// result
	filtering:pork
	mapping:pork
	filtering:beef
	mapping:beef
	filtering:chicken
	mapping:chicken
    ```

1. Short-Circuit : 여러 데이터 중 처음 3개만 선택되게 한다.
2. loop fusion : filter, map과 같은 다른 연산의 병합


#### 최종 연산
스트림을 닫는 연산으로 `collect`와 같은 파이프라인의 실행을 마무리하는, 결과를 도출하는 것이다. 보통 이에 의해 `List`, `Integer`, `void` 등의 스트림이 아닌 결과가 반환된다.

`menu.stream.forEach(System.out::pringln)`
해당하는 코드는 void를 반환하는 스트림 최종 연산이다.


#### 스트림 이용 과정

이러한 연산들을 종합하여 스트림을 이용하는 과정은 다음과 같다.

1. 질의를 수행할 데이터 소스
2. 스트림 파이프라인을 구성할 중간 연산 연결
3. 파이프라인 실행 후 결과를 만들 최종 연산


### 마무리

해당 장의 중요내용은 다음과 같다.

- 스트림은 소스에서 추출된 연속 요소로, 데이터 처리 연산을 지원한다.
- 스트림은 내부 반복을 지원하며 filter, map, sorted 등의 연산을 반복을 추상화한다.
- 스트림에는 중간 연산과 최종 연산이 존재한다.
	- 중간 연산은 스트림을 반환하며 다른 연산과 연결된다.
	- 최종 연산은 스트림 파이프라인을 처리하여 스트림을 제외한 결과를 반환한다.
- 스트림의 요소 처리는 Lazy한 특징을 가진다.


