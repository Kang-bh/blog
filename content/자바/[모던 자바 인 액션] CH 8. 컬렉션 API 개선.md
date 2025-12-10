# 8.1 컬렉션 팩토리

자바 9로 업데이트가 되면서 작은 컬렉션 객체를 쉽게 만들 수 잇는 방법을 제공합니다.


    ```
    // 기존
    List<String> friends = new ArrayList<>();
	friends.add("Raphael");
	friends.add("Olivia");
	friends.add("Thibaut");
	
	// Arrays.asList() 팩토리 메서드 사용
	List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut")
    ```


#### UnsupportedOperationException 예외 발생

위의 방식처럼 코드를 간단하게 줄일 수 있지만 이런 경우에는 고정길이로 리스트를 생성하기에 새 요소를 추가하거나 삭제할 수 없다.

## 8.1.1 리스트 팩토리

`List.of` 팩토리 메서드를 통해 다음과 같이 간단하게 리스트를 생성할 수 있다.


    ```
    List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
    ```


해당하는 코드에 `friends.add("Kang")`을 수행하면 `java.lang.UnsupportedOperationException`이 발생하게 된다. 



## 8.1.2 집합 팩토리

다음의 방식으로 고정길이의 집합을 생성한다.


    ```
    Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
	```

중복된 요소로 Set을 생성한다면 `IllegalArgumentException`이 발생한다.

## 8.1.3 맵 팩토리

다음 두 가지 방식을 통해 고정 길이의 맵을 초기화할 수 있다.


    ```
    Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
    
    import static java.util.Map.enty;
    Map<String, Integer> ageOfFriends2 = Map.ofEnties(
	    entry("Raphael", 30),
	    entry("Olivia", 25),
	    entry("Thibaut", 26),
	)
    ```


# 8.2 리스트와 집합 처리

자바 8에서는 List, Set 인터페이스에 컬렉션 자체를 변경하는 다음의 메서드가 추가도니다.

- `removeIf` : 프레디케이트를 만족하는 요소를 제거한다. `List`나 `Set`을 구현하거나 그 구현을 상속받은 모든 클래스에서 이요가능하다.
- `replaceAll` : 리스트에서 사용하는 기능으로 `UnaryOperator` 함수를 이용해 요소를 변경한다.
- `sort` : `List`인터페이스에서 제공하는 기능으로 정렬처리를 수행한다.

### 8.2.1 removeIf 메서드

다음의 트랙잭션을 삭제하는 코드는 `ConcurrentModificationException`을 일으킨다. 그 이유는 `for-each` 루프가 `Iterator`객체를 사용하기 때문이다.


    ```
    for (Transaction transaction : transactions) {
	    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
		    transactions.remove(transaction);
	    }
    }
    // 실제 구현 코드
    for (Iterator<Tranasction> iterator = transactions.iterator(); iterator.hasNext();) {
    if (Chracter.isDigit(transaction.getReferenceCode().charAt(0))) {
	    transactions.remove(transction); // 반복하면서 별도의 두 객체를 통해 컬렉션을 변경
	    }
    }
    ```

위 방식은 다음의 방식을 컬렉션을 관리한다.
- `Iterator` 객체, `next()`, `hasNext()` 를 이용해 소스에 질의한다.
- `Collection` 객체 자체, `remove()`를 호출해 요소를 삭제한다.

이는 반복자의 상태가 컬렉션의 상태와 동기화되지 않음을 의미한다.
이러한 문제를 해결하기 위해 다음의 방식으로 해결할 수있으며 이는 자바 8의 `removeIf` 메서드와 호환이 되며 위에서 발생가능한 에러를 방지할 수 있다.


    ```
    for (Iterator<Tranasction> iterator = transactions.iterator(); iterator.hasNext();) {
    if (Chracter.isDigit(transaction.getReferenceCode().charAt(0))) {
		iterator.remove();
	    }
    }
    // removeIf 사용
    transactions.removeIf(transction -> Chracter.isDigit(transction.getReferenceCode().charAt(0)));
    ```


### 8.2.2 replaceAll 메서드

`replaceAll`메서드를 이용해 리스트의 각 요소를 새로운 요소로 변경할 수 있다


기존 컬렉션을 바꾸는 방법으로 다음과 같이 `ListIterator`객체를 이용하여 처리할 수 있다.


    ```
    for (ListIterator<Stirng> iterator = referenceCodes.listIterator(); iterator.hasNext(); ) {
    String code = iterator.next();
    iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
    }
    ```

이를 다음과 같이 `replaceAll`를 통해 다음과 같이 간단하게 구현할 수 있다.


    ```
	referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.subsring(1));
	```

# 8.3 Map 처리

### 8.3.1 forEach 메서드

맵에서 키와 값을 반복하면서 확인하며 다음과 같이 구현할 수 있다.


    ```
	ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
	```

### 8.3.2 정렬 메서드

다음 두 개의 유틸리티를통해 값 또는 키 기준으로 정렬할 수 있다.

- Entry.comparingByValue
- Entry.comparingByKey



```
Map<String, String> favoriteMoviews = Map.ofEntreis(
		entry("Raphael", "Star Wars"),
		entry("Cristina", "Matrix"),
		entry("Olivia", James Bond")
);

favoriteMovies
	.entrySet()
	.stream()
	.sorted(Entry.comparingByKey())
	.forEachOrderd(System.out::println);
```


> [!HashMap 성능]
    > 자바 9에서 HashMap의 내부 구조를 바꾸어 성능을 개선했다. 기존에는 키로 생성한 해시코드로 접근할 수 있는 버켓에 저장되어 키가 같은 많은 해시코드를 반환하는 경우에 O(n)의 시간이 걸리는 `LinkedList`로 버킷을 반환하여 성능이 저하되는 경우가 있었다.
    > 그러나 최근 버킷이 너무 커지는 경우 O(log(n))의 시간이 소요되는 정렬된 트리를 이용해 동적으로 치환해 충돌이 일어나는 요소 반환 성능을 개선했다.


### 8.3.3 getOrDefault 메서드

기존에 찾으려는 키가 존재하지 않는 경우 Null이 반환되기에 `NullPointerException` 에러 방지를 위해 요청 결과가 Null인지 확인하는 로직이 필요했다. `getOrDefault` 메서드의 등장은 이 문제를 쉽게처리할 수있다.



    ```
    Map<String, String> favoriteMoviews = Map.ofEntreis(
		entry("Raphael", "Star Wars"),
		entry("Cristina", "Matrix"),
		entry("Olivia", James Bond"));
	
	System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
	System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix")); // Matrix 출력
	```

### 8.3.4 계산 패턴

맵에 키가 존재하는지 여부에 따라 동작을 실행하고 결과를 저장하는 상황에 다음 세 가지 연산이 도움을 준다.

- `computeIfAbsent` : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추간하다.
- `computeIfPresent` : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
- `compute` : 제공된 키로 새 값을 계산하고 맵에 저장한다.



```
lines.forEach(line -> dataToHash.computeIfAbsent(line, this::calculateDigest));

frendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>())
			.add("Star Wars")
```


### 8.3.5 삭제 패턴

제공된 키에 해당하는 맵 항목을 제거하는 메서드로 