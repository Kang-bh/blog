# 운영체제

## 운영체제란?

> **컴퓨터**시스템을 운영하는 소프트웨어

### 컴퓨터 종류

위 정의에서 컴퓨터라는 하드웨어는 어떤 종류가 있을까?
- 스마트폰
- 데스트탑
- 맥북
- 계산기

여기서 계산기와 같은 것은 컴퓨터가 아니다. 그 이유는 컴퓨터의 정의가 다음과 같기 때문이다.

> **정보**를 처리하는 기계

#### 정보

클로드 새년이 정보에 대해 수학적으로 정의를 시작

![[Pasted image 20250322200723.png]]

위 내용이 정보에 대한 정리로 Notation은 다음과 같다.
- x : 사건
- I(x) : 사건 x의 정보량
- P(x) : 사건이 발생한 확률

해당 과정에서 다음과 같이 정보를 정의할 수 있다.

> 불확실한 상황을 측정해서 수치적으로 표현한 것


- 정보의 최소 단위 : bit (binary digit)

##### 정보의 처리
정보의 상태변환으로 0에서 1, 1에서 0으로 변환하는 것이다. 이러한 것을 Boolean Algebra(NOT, AND, OR)을 통해 처리할 수 있다.


덧셈 : 반가산기, 전가산기 이용
뺄셈 : 2의 보수 표현법
곱셈과 나눗셈 : 덧셈과 뺄셈의 반복
실수 연산 : 부동 소수점 표현
함수 : GOTO

이외에도 삼각함수, 미분, 적분, flip-flop, data bus가 있는데 이는 논리회로관련 내용을 확인해보도록 하자..


Stored-program, 폰노이만 아키텍처 (ISA) : 명령어 집합으로 작동되는 아키텍처
1. Ram에서 프로그램 보관
2. CPU로 Fetch
3. CPU에서 Execute

##### 프로그램 
명령어들의 집합 (컴퓨터에게 일을 시키는)


컴퓨터 하드웨어를 관리하는 소프트웨어를 운영체제라고 한다.

운영체제는 결국 어플리케이션 프로그램, 컴퓨터 유저, 하드웨어 간의 매개로써의 역할

결국 컴퓨터 시스템은 4개의 구성 요소로 존재한다.

- 하드웨어
- 운영체제
- 어플리케이션
- 유저

![[Pasted image 20250323071103.png]]

위 그림과 같이 존재한다고 볼 수 있다.

- 범용적으로 정의된 용어는 없다.
운영체제의 common 정의
- 일반적으로 컴퓨터에서 항상 운영되는 하나의 프로그램 : 커널
커널에서 시스템 프로그램, 어플리케이션 프로그램을 제공해준다.



전통적인 컴퓨터 시스템 구성 요소
![[Pasted image 20250323071722.png]]

- CPU
- 버스를 통해 메모리와 연결되어 있는 여러 디바이스 컨트롤러 존재 
이를 OS가 제어


### Bootstrap PRogram

BootStraping하는 것
![[Pasted image 20250323071931.png]]

끝을 떙기면 쏙 들어가는 것과 같이 컴퓨터가 전원이 켜질 때 메모리에 운영체제를 로딩해주는 일을 하는 것이다.
디스크 -> 메모리로 이동

결국 운영체제를 메모리에 집어넣는 행위이다.

#### Interrupts

예를 들어 키보드에 A를 누르는 경우 이를 CPU에게 알려줘야한다. 이를 위해 Interrupt 방법을 이용한다.

즉, 하드웨어가 인터럽트를 일으키고 이를 시스템 버스를 통해 CPU에 신호를 전달하는 행위이다.


### 폰 노이만 아키텍처

instruction-execution cycle

컴퓨터에 내려지는 명령어 집합들이 존재하고 이를 통해 구성된 컴퓨터 프로그램을 메모리에 로딩하여 메모리에 있는 명령어들을 CPU가 하나씩 fetch하고 execute를 한다.

즉, fetch, execute cycle이다.
그러기 위해서 instructiion register를 통해 명령어를 저장하고 수행하도록 처리한다.



#### Storage System

용량, 엑세스 타임에 따라 계층구조가 존재한다.

- register
- cache
- main memory
- solid-state disk
- hard dis
- optical disk
- magnetic tapes
![[Pasted image 20250323072927.png]]


#### I/O Structure

![[Pasted image 20250323073028.png]]

> [!DMA]
    > Directed Memory Access라는 의미로 유튜브의 영상을 받아 화면에 보여주는 경우에는 CPU가 하는 일은 대역폭 조절 정도의 역할 뿐이다. 이런 경우에 보통 디바이스와 메모리 사이에서의 통신이 이루어지는데 이를 DMA라고 한다.
    

최종적인 구성

- CPU
- Processor
- Core
- MultiCore
- MultiProcessor

##### Symmetric multiprociessing
최근에는 Symmetric multiprocessing (SMP) 구조이다.

메모리 하나에 여러 CPU가 연결되어 있고 각각의 register와 cache를 구성한 것이다.

![[Pasted image 20250323073440.png]]

##### Multi-core design

CPU하나안에 여러 코어 존재하는 것 즉, 같은 프로레스 칩 내부에 여러 코어로 존재하는 것이다.
![[Pasted image 20250323073548.png]]

##### Multiprogramming

이전에는 메모리에 프로그램을 하나만 로딩했지만 최근에는 여러 개의 프로그램을 동시에 메모리에 올려두고 작업을 수행할 수 있도록 하는 것.
여러 프로세스를 메모리에 올려두어 CPU의 사용률을 증가시킬 수 있다. 유휴 시간 줄어들기, 좀 더 효율적 활용

![[Pasted image 20250323073634.png]]


##### Multitasking(MultiProcessing)

멀티 프로그래밍이 되는 것으로 인해 가능한 것으로 이를 통해 여러 프로세스를 빈번하게 바꿔주면 유저는 여러 일들이 동시에 처리되는 것 처럼 볼 수 있다.

이를 Concurrency, Pallelelism으로 이어진다. (이 차이는 이해할 피료가 존재)

이를 위해서 **CPU scheduling**이 필요

프로세스를 어떤 것을 다음에 처리할 지. 이 목표는 CPU사용률을 높이는 것.




#### Mode
다른 프로그램이 부적절하게 실행되는 것을 방지하기 위해 존재하는 것으로 다음 두 모드가 존재한다.
##### User Mode

##### Kernel Mode


![[Pasted image 20250323074239.png]]

이를 위해 유저가 직접적으로 하드웨어에 접근하지 못하기에 낫배드임.


#### Virtualization

멀티 프로레싱의 OS버전으로 하나의 HW에 여러 OS를 띄워보자는 것으로 나옴.

VMM(VIrtual Machine Manager)을 통해 관리한다.


![[Pasted image 20250323074511.png]]



최근에는 다음과 같은 컴퓨팅 환경들로 다양하게 존재한다.

- Traditional Computing
- Mobile Computing
- Client-Server Computing
- Peer-to-Peer Computing (Airdrop)
- Cloud Computing
- Real-Time Embbeded System

