# Process

> 실행중인 프로그램

- 운영체제 입장에서 작업의 단위를 프로세스라 부른다.
- 프로세스의 실행을 위해 다음 자원이 필요하다.
	- CPU
	- Memory
	- Files
	- I/O device

하드 디스크(스토리지)에 저장된 프로그램(명령어 집합)을 메모리에 로드하고 이를 CPU가 fetch하여 실행하는 것. 이 과정에서 메모리에 올라가 있는 것을 프로세스이다.

실행 중에 여러 외부 파일, I/O Device를 통해 리소스들을 가져올 수도 있음.


여러 부문으로 나뉜 프로세스는 다음과 같이 나뉜다.
- Text section : 명령어들의 코드
- Data section : 전역변수
- Heap section : 동적 메모리로 자바에서는 new
- Stack section : 함수 호출 시 해당 영역에 쌓인다. 주로 함수 Parameter, address, 지역 변수가 저장된다.

위 section을 그림으로 그리면 다음과 같이 되어있다.
![[Pasted image 20250328152044.png]]



0~max : 논리적인 주소로 logical한 주소. 0은 프로그램의 시작 부분이라고 볼 수 있다.

code -> data -> stack, hea
여기서 stack, heap영역의 용량이 부족하면 다른 메모리를 빌려 사용

?? stack이 위에서 쌓이는 이유


소스코드 -> out(프로그램) -> 메모리 로드 -> 다음과 같은 구조


OS는 위 프로세스를 관리하는 것