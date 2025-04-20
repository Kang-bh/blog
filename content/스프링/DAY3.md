
클래스

패키지 : 자바 코드가 위치할 디렉토리


리팩토링 : 기존의 기능을 그대로 두고 코드의 가독성을 높이거나 메서드를 이동시키는 것


클래스 : 데이터와 메서드가 하나의 모듈로 구성되어 있는 것으로 재사용하기 편하게 모듈을 제공


정보은닉 : 정보보안이 아닌 의도치 않은 코드 구현을 위해 다른 개발자에게 감추기 위한 것

게터 또는 세터만 존재해야한다. 둘 다 있따면 public으로 설정하는 것이 올바르다.

의존성 주입

Emp 클래스가 Dpt 의 기능을 사용한다 == 의존한다.


```
// 새로 생성시 네 개의 값 전달

public Emp(String employeeId, String employeeName, int totalYearsOfService, String departmentCode) {
	this.employeeId = employeeId;
	
	this.employeeName = employeeName;
	
	this.totalYearsOfService = totalYearsOfService;
	
	this.departmentCode = departmentCode;

}

```


Label
프로그램 코드 이해를 쉽게 할 수 있게 해준다.
```
FIND: for (var emp : empList) {

	String employeeId = emp.getEmployeeId();
	
	if ("E004".equals(employeeId)) {
	
		findEmp = emp.propfile();
		
		break FIND;

	}

}
```


중복된 데이터 생성



```
public class Dpt {

// 부수 정보를 키로 입력 받아 테이블에 저장 및 특정 키 가져올 수 있도록 하는 코드

private static HashMap<String, String> dptData = new HashMap<> (); // Field 영역 멤버변수no

public Dpt() {
	init();
}

public void init() {

dptData.put("D01", "개발1팀");

dptData.put("D02", "회계");

dptData.put("D03", "사장");

}

  

public String getDepartmentName(String departmentCode) {

return dptData.get(departmentCode);

}

  

}
```

문제 : 데이터가 계속 중복으로 입력된다.