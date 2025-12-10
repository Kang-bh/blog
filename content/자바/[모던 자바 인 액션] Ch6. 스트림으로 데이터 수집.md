# 6.1 컬렉터란 무엇인가?


    ```
    // 명령형 프로그래밍
    Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();
    
    for (Transaction transaction : transactions) {
	    Currency currency = transaction.getCurrency();
	    List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
	    if (transctionsForCurrency == null) {
		    transactionsForCurrency = new ArrayList<>();
		    transactionsByCurrencies.put(currency, transactionsForCurrency);
	    }
	    transctionsForCurrency.add(transaction);
    }
    // Stream에 collector이용하는 함수형 프로그래밍
    Map<Currency, List<Transction>> transactionsByCurrencies = transactions.stream().collect(groupingBy(Transction::getCurrency);)
    ```

해당 예제는 명령형 프로그래밍에 비해 함수형 프로그래밍의 편리함을 보여준다.
함수형의 경우에는 '무엇'을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 획득할 지 신경쓸 필요가 없다. 

여기서 다수준으로 그룹화를 수행하는 경우에 명령형 프로그래밍은 다중 루프, 조건문의 등장으로 인해 가독성과 유지보수성이 크게 떨어지지만 함수형 프로그래밍은 간단하게 표현할 수 있다.


### 고급 리듀싱 기능을 수행하는 컬렉터

