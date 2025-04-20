# CPU Scheduling

> OS의 멀티 프로그래밍의 기초가 되는 개념

> [!멀티 프로그래밍의 목적성]
    > CPU의 속도가 너무 빠르기에 유휴시간이 크다. 이를 Interliving을 이용한 시분할을 통해서 CPU 사용량을 높이기 위해서 사용한다.

Multi Thread와 Multi Process는 엄연히 다른 개념이지만 작동원리는 비슷하기에 해당 챕터에서는 이를 **Multi Processing**에 대해서 설명한다.

### 5.1 기본 개념

CPU의 처리속도는 컴퓨터가 발전할수록 빨라졌고 이는 프로세스를 빨리처리하게 되는데에 직결되면서 CPU의 활용을 100%로 활용하지 못하고 CPU의 유휴시간(Idle Time)을 증가시키게된 게기가 된다.

![[스크린샷 2025-04-12 오후 1.57.17.png]]
위 그림은 Cpu Burst 순간과 I/O Busrt의 순간들(**CPU-I/O Burst Cycle**)을 일련의 과정으로 표현한 과정으로 각 Burst는 다음 의미를 가진다..

- I/O Burst : I/O 장치의 작동을 기다리는 작업으로 Wating Queue에서 **Ready Queue**로 이동하게 되는 과정이다.
- CPU Burst : CPU가 프로세스를 처리하는 과정으로 Waiting Queue에서 **Running Queue**로 넘어가게 되는 과정이다.

CPU Core가 하나 있는 상황에서 하나의 작업(Process)가 끝나면 출력이 되고 다른 프로세스가 입력으로 되는 과정을 나타낸 것이다.

#### CPU 처리 과정
아래 그림과 같이 여러 프로세스가 메모리에 동시에 로드되지만 CPU를 선점한 프로세스가 실행이 된다. 이후 실행이 완료되면 다음 프로세스가 CPU를 선점하고 다른 프로세스는 ready상태에서 대기한다.
![[Pasted image 20250412140918.png]]
- P1 : CPU에서 실행
- P2, P3 : ready Queue에서 대기
- 여기서 특정 자원을 기다린다면 Waiting Queue에서 대기


> [!NOTE]
    > Content여기서 Waiting Queue에서 대기하는 과정은 메모리에 존재하는 것인가? 아닌가?


결국 CPU의 처리속도가 빨라지면서 CPU를 100% 활용하지 못하게 되었기에 이를 방지하기 위해서 CPU Scheduling의 개념이 도입된다.

### Preemptive vs Non-preemptive

선점형 커널, 비선점형 커널로도 불린다. 여기서의 선점형은 지금 활성화된, CPU를 사용중인 프로세스를 *쫓아낸다* 라고 생각해도 된다.
##### Non-Preemptive (비선점형 커널)
CPU를 사용중인 프로세스가 있다면 해당하는 프로세스가 종료될 때 까지 기다린다. 여기서의 종료는 실행완료 또는 wating 상태로 전환되는 결과 둘 중 하나이다.

##### Preemptive (선점형 커널)
CPU Scheduler가 우선순위가 더 높은 프로세스의 처리를 위해 현재 진행하고 있던 프로세스를 내쫓는 즉, 다른 프로세스가 CPU를 선점하는 방식이다.

CPU Scheduling 과정에서 판단이 일어나는 종류

![[Pasted image 20250412150129.png]]

1. Running -> Waiting : I/O 요청이나 자식 프로세스 종료를 기다리기 위해 `wait()` 호출 시 실행된다. 프로세스가 I/O 요청이나 자식 프로세스 종료를 기다리기 위해 대기 상태로 전환되는 경우이기에 CPU를 사용하지 않는 과정으로 넘어가기에 자연스럽게 CPU는 다른 프로세스를 사용한다. 그렇기에 Scheduling Decision이 필요없는 **Non-preemptive** 과정이다.
2. Running -> Ready : 인터럽트가 발생하여 실행 중인 프로세스를 중단하고 준비 상태로 전환될 때는 CPU 스케줄러가 다른 프로세스를 선택해야 한다. 이는 Preemptive 또는 Non-preemptive 방식으로 처리할 수 있다.
3. Waiting -> Ready :  I/O 작업이 완료되어 프로세스가 다시 실행 준비가 되었을 경우로 Waiting 상태에서 Reday 상태로 전환된 프로세스는 Ready 큐에 추가됩니다. 여기서 CPU 스케줄러는 우선순위를 평가하여 Preemptive 또는 Non-preemptive 방식으로 처리할 수 있다.
4. Running -> Terminate : 실행 중인 프로세스가 작업을 완료하고 종료될 경우이다. 이 때, CPU는 준비 큐에서 다음 프로세스를 선택하기에 Scheduling Decision이 필요 없는 **Non-preemptive** 과정이다.

#### Dispatcher

CPU 스케줄링의 중요한 구성 요소로, CPU 스케줄러가 선택한 프로세스를 실제로 CPU에서 실행할 수 있도록 제어권을 넘기는 역할을 수행하는 하나의 모듈이다.

> 즉, Context Switch를 해주는 모듈

###### Dispatcher 기능
- **Context Switching** : 이전 프로세스의 상태(레지스터, 프로그램 카운터 등)를 저장하고, 새로 선택된 프로세스의 상태를 복원 수행
- **User Mode Switching** : CPU를 커널 모드에서 사용자 모드로 전환하여 새 프로세스를 실행할 수 있도록 설정
- **프로그램 실행 재개** :CPU를 새 프로세스의 코드가 저장된 메모리 위치로 이동시켜 실행을 시작합니다. 즉, 중단된 프로그램이 있다면 해당 프로그램의 적절한 위치로 이동하는 것

> [!Scheduler vs Dispatcher]
    > - Scheduler : 어떤 Process를 처리할지 선택하는 것
    > - Dispatcher : 실제 Switch 과정을 수행


여기서 Dispatcher는 가능한 빨라야한다!
Contexxt Switch마다 사용되는 Dispatcher는 다른 프로세스의 Stop 상태와 다른 프로세스의 Start 사이에 `Dispatch Latency`가 발생한다. 
평소 CPU를 100%를 찍는 상황이 온다면 Dispatch를 수행하다가 100%를 찍는다. 그렇기에 `Dispatcher Latency`는 CPU 사용시간보다 짧아지도록 해야한다.

![[스크린샷 2025-04-12 오후 3.26.29.png]]

### 5.2 Scheduling Criteria

CPU 스케줄링에 대한 알고리즘은 엄청 다양하게 존재하고 특정 상황에 유리한 알고리즘들이 존재한다. 이러한 알고리즘들을 선택할 때에 다음 기준들을 고려해야한다.

- CPU Utillization : 최대한 CPU를 활용하도록 처리해야한다.
- Throughput : 단위시간 내에 Process의 처리량
- Turnaround Time : 프로세스가 CPU에 도달(실행 준비 상태)한 시간부터 종료되는 시간
- Waiting Time : Process가 Ready Queue에서 대기하는 시간으로 CPU를 할당받을 때까지 대기하는 시간이다.
- Response Time : 반응까지 오는 시간으로 주로 Interactive 시스템(e.g. 게임)에서 사용된다. UX적인 요소에 집중한 부분으로 볼 수 있다.


### 5.3 Scheduling Algorithms

CPU 스케줄링에서는 Ready Queue에 존재하는 프로세스 중 어떤 프로세스를 CPU에 할당할 것이냐에 대한 과정이 매우 중요하다. 해당하는 로직, 알고리즘을 통해 위의 기준들의 요건을 맞추며 성능을 향상시킬 수 있기 때문이다.

다양한 알고리즘은 다음과 같이 존재한다.
#### FCFS Scheduling

First-Come, First-Served로 먼저 들어온 프로세스를 먼저 처리하는 것이다. 이는 가장 단순한 알고리즘의 형태로 FIFO Queue를 통해 간단하게 구현할 수 있다.

![[스크린샷 2025-04-12 오후 3.46.44.png]]

위 문제는 다음의 조건을 가진다.
- 모든 프로세스는 *동일 시간(0 시간)에 도착*한다.
- CPU Burst Time은 밀리세컨 단위로 위 그림처럼 프로세스 마다 처리해야하는 시간이 주어져 있다.
- 동일 시간에 도착하였지만 Queue에는 P1, P2, P3에 쌓인다. (이는 ns와 같은 매우 작은 오차이기에 설명을 위해서도 무시되고 실제 구현에서도 무시되도록 처리되기도 한다.)

CPU가 처리하는 과정을 Gantt Chart로 표현하면 다음과 같다.
![[스크린샷 2025-04-12 오후 3.49.32.png]]
처리과정은 다음과 같다.
1. P1이 먼저 들어왔기에 FCFS 원칙에 의해 P1을 처리
2. P1의 처리 후 그 다음 순서인 P2 처리
3. P2가 Terminate된 후 P3를 Running State로 변환 및 CPU가 처리
4. P3의 Ternimate 상태 변환 후 처리 종료

위 과정에서 Waiting Time을 계산하면 다음과 같다.
- P1 = 0, P2 = 24, P3 = 27
- Total Waiting Time : 0 + 24 + 27 = 51
- Average Waiting Time : 53/3 = 17

여기서 우리는 Average Waiting Time에 집중하여 이를 최소화하는 방향으로 알고리즘을 처리해야한다.

###### Turnaround Time 계산
- P1 = 24, P2 = 27, P3 = 30
- Total Turnaround Time : 24 + 27 + 30 = 81
- Average Turnaround Time : 81/3 = 27

또 다른 상황으로 P2, P3, P1 순서로 Ready Queue에 쌓였다고 보자.
CPU가 처리하는 과정은 다음과 같다.
![[스크린샷 2025-04-12 오후 3.54.02.png]]

1. P2가 먼저 들어왔기에 FCFS 원칙에 의해 P2를 처리
2. P2의 처리 후 그 다음 순서인 P3 처리
3. P3가 Terminate된 후 P1을 Running State로 변환 및 CPU가 처리
4. P1가 Terminate 상태로 변환된 후 처리 종료

위 과정에서 Waiting Time을 게산하면 다음과 같다.
- P1 = 6,  P2 = 0, P3 = 3
- Total Waiting Time : 6 + 0 + 3 = 9
- Average Waiting Time : 9/3 = 3
###### Turnaround Time 계산
- P1 = 30, P2 = 3, P3 = 6
- Toal Turnaround Time : 30 + 3 + 6 = 39
- Average Turnaround Time : 39 / 3 = 13

##### 정리

FCFS 스케줄링을 따르면 Average Waiting Time이 최적이 될 수 없다.
프로세스의 CPU-burst time에 따라 달라지기 때문이다. 또한, FCFS 스케줄링은 진행중인 프로세스가 완료된 후에 다음 프로세스가 진행되기에 **Non-Preemptive**이다.

> [!Convoy Effect]
    > Content
#### Shortest-Job-First Scheduling

해당 하는 알고리즘은 가장 CPU burst 시간이 짧은 프로세스를 다음 순서로 할당하는 것이다. 그렇기에 **shortest-next-CPU-burst-first-scheduling**이라고 불리기도 한다.

프로세스의 CPU Burst 시간ㅇ니 동일하면 FCFS와 동일한 동작이다.

![[스크린샷 2025-04-12 오후 9.30.40.png]]

위 그림의 프로세스가 존재한다고 할 때, SJF는 다음과 같이 처리된다.

![[스크린샷 2025-04-12 오후 9.32.10.png]]

1. CPU Burst 시간이 가장 적은 P4가 먼저 처리된다.
2. CPU Burst 시간이 그 다음을 적은 P1이 처리된다.
3. P3가 처리된다.
4. P2가 처리된 후 종료된다.

##### Waiting Time
- P1 = 3, P2 = 16, P3 = 9, P4 = 0
- Total Waiting Time : 3 + 16 + 9 + 0 = 28
- Average Waiting Time : 28/4 = 7

##### Turnaround time:
- P1 = 9, P2 = 24, P3 = 16, P4 = 3
- Total Turnaround Time : 9 + 24 + 16 + 3 = 52
- Average Turnaround Time : 52/4 = 13

##### SJF의 실현가능성
SJF는 이론상 대기 시간을 최소화하는 최적의 알고리즘이다.
그렇지만, 프로세스의 CPU burst Time을 알 방법은 없다. 그렇기에 *근사화*하여 SJF 스케줄링을 처리한다.
다음 CPU Burst Time을 예측하고 예측된 CPU Burst 시간 중 가장 짧은 시간을 처리한다.

###### Next Cpu Burst Time 예측 방법
과거의 CPU 사용량을 알면 지수적 평균을 구할 수 있다. 즉, 이전 CPU Burst 시간의 길이를 기반으로, 최근 정보와 과거 정보를 가중치로 결합하여 예측을 수행하는 Exponential Average를 이용하여 CPU Burst Time 예측한다.
![[스크린샷 2025-04-12 오후 9.43.25.png]]
![[Pasted image 20250412214424.png]]

실제 해당 수식을 적용했을 때 근사화된 값을 구했을 때와 실제 CPU Burst를 비교해서 확인해보면 다음의 그래프를 확인할 수 있다.

![[Pasted image 20250412225940.png]]

가중치를 1/2로 했을 때로 time 2까지의 과정은 다음과 같다.
1. ταυ_2 : 1/2 * 10 + 1/2 * 6 = 8 [ταυ_1 + t1] 로 현재(Time1)에 대한 CPU Burst 시간에 대한 예측값이다.
2. ταυ_3 : 1/2 * 8 + 1/2 * 4 = 6 [ταυ_2 + t2] 로 현재(Time2)에 대한 CPU Burst 시간에 대한 예측값이다.

왜 Exponential이냐
###### 가중치의 중요성

- 알파값이 0인 경우 : ταυ_n+1 = ταυ_n이 된다. 이는 완전히 예측값만을 가지고 처리하는 것을 의미한다.
- 알파값이 1인 경우 : ταυ_n+1 = tn이 된다. 이는 완전히 최근의 CPU Burst만을 가지고 처리하는 것을 의미한다.

기본적인 SJF는 Non-Preemptive한 SJF로 이미 진행 중인 프로세스의 자리를 훔치지 않고 완료된 후에 Ready Queue에 있는 프로세스들 중에서 가장 CPU Burst 시간이 적은 프로세스를 처리한다.

그렇다면 위의 로직에서 Preemptive하게 처리가 가능할까?
#### SRTF Scheduling

새로운 프로세스가 Ready Queue에 들어오는 순간에 진행중인 프로세스가 존재할 수 있다.
해당 경우 새롭게 들어온 프로세스가 진행중인 프로세스보다 실행시간이 작다면? 이런 경우에 선점을 할 수 있다.

Shortest-Remaining-Time-First의 약자이며 SJF의 종류 중 하나로 이는 **Preemptive SJF Scheduling**이라고 한다. 즉, 진행 중인 프로세스를 밀어내고 본인이 처리하는 것이다.

해당 예시를 들어보면 다음의 프로세스 처리가 존재한다.
![[스크린샷 2025-04-12 오후 11.58.17.png]]

위 프로세스들을 SRTF로 처리하면 다음의 처리과정을 가진다.

1. **P1**이 0초에 도착하여 처리된다.
2. 1초가 되어 P1의 Burst Time은 7이 된다. 이 때, P2가 도착하고 P1의 Remaining Burst Time(7)과 P2의 Burst Time(4)을 비교할 때 P2가 더 적게 남았기에 **P2**가 CPU 자원을 선점하고 P1는 Ready Queue로 들어간다.
3. 2초가 되어 P2의 Burst Time은 3이 된다. 이 때, P3가 도착하고 P2의 Remaining Burst Time(3)과 P3의 Burst Time(9)를 비교할 때 P2가 더 적게 남았기에 **P2**가 CPU 자원을 선점하고 P3는 Ready Queue로 들어간다.
4. 3초가 되어 P2의 Burst Time은 2가 된다. 이 때, P4가 도착하고 P2의 Remaining Burst Time(2)과 P4의 Burst Time(5)를 비교할 때 P2가 더 적게 남았기에 **P2**가 CPU 자원을 선점하고 P4는 Ready Queue로 들어간다.
5. 이후 Ready Queue에 남아있는 P1, P3, P4를 Burst Time이 작은 것들을 순차적으로 처리한다.


Gantt chart 로 표현하면 다음과 같다.
![[스크린샷 2025-04-13 오전 2.26.35.png]]
##### waiting time
- Total Waiting Time : (10 - 1) + (1 - 1) + (17 - 2) + (5 - 3) =26
- Average Waiting Time : 26/4 = 6.5


위 처리를 SJF Scheduling으로 표현하면 다음과 같다.
![[Pasted image 20250413023556.png]]
##### waiting time
- Total Waiting Time 0 + (8 - 1) + (12 - 2) + (17 - 3)= 31
- Average Waiting Time : 31/4 = 7.75

SJF 스케줄링과 STFJ를 비교 했을 때 SRTF가 SJF보다 Average Wiating Time이 더 짧은 것을 확인할 수 있다. 결국 **선점**을 하느냐 마느냐에서 성능이 갈린 것이다.

#### RR Scheduling

Round-Robin Scheduling으로 시간 공유 시스템에서 가장 많이 사용되는 CPU Scheduling이다. 여기서의 Round-Robin은 Preemptive FCFS의 개념에서 Time Quantum(Time slice)의 개념을 첨가한 것으로 보면 된다.
즉, preemptive FCFS를 하되 시분할하여 time간격마다 처리하는 것이다.

여기서 주로 Time quantum은 하나의 시간단위로 10~100ms를 주로 설정한다.

이러한 RR 스케줄링을 구현하기 위해서는 Ready Queue 는 Circular Queue가 되어야한다.
![[Pasted image 20250413025411.png]]

위 사진처럼 계속해서 프로세스가 나가고 들어가는 구조가 되어야한ㄷ.

여기서 Scheduler는 Ready Queue를 순회하면서 `1 time quantum`마다 CPU를 각 프로세스에 할당하는 방식으로 진행된다.


![[Pasted image 20250413031438.png]]

위의 프로세스를 RR Scheduling을 통해 실행한다고 가정하고 Time Quantum을 4로 했을 때 다음과 같이 실행된다고 볼 수 있다.

1. P1이 4초 실행 후 Context Switch
2. P2가 실행되며 완료
3. P3가 실행
4. 이후 P1이 계속 Context Switch를 일으키며 수행

![[Pasted image 20250413034215.png]]
##### Waiting time
- P1 = 10 - 4 = 6, P2 = 4, P3 = 7
- Total Waiting Time : 6 + 4 + 7 = 17
- Average Waiting Time : 17/3 = 5.66

##### Time Quantum 설정의 중요성

![[Pasted image 20250413034517.png]]

RR Scheduling의 경우 time quantum의 크기에 따라 성능이 좌우된다.

- Time Quantum을 12라고 가정했을 때, Context Switch의 수는 0이다.
- Time Quantum을 6이라고 가정했을 대, Context Switch의 수는 1이다.
- Time Quantum을 1이라고 가정했을 대, Context Switch의 수는 9이다.

즉, Time Quantum을 줄일 수록 Context Switch는 자주 일어난다. 이것이 문제가 되는 이유는 **dispatch latency**의 문제이다. Dispatch Latency가 자주 발생할 수록 CPU의 사용률을 잡아먹고 단위 시간당 처리할 수 있는 프로세스의 수가 줄어들기 때문이다.

반면, Time Quantum을 크게할 수록 FCFS와 유사해져 짧은 작업이 오랫동안 대기하는 문제가 발생할 수 있고 실시간성을 반영하지 못하는 문제가 발생한다.

그렇기에 적절한 Time Quantum을 설정하는 것이 중요하다.


### Priority-based Scheduling

우선순위를 기반으로 각 프로세스에 고유한 우선순위를 부여하고 높은 우선순위를 가진 프로세스가 먼저 CPU를 할당 받는 구조이다. 해당 과정에서 우선순위가 동일하다면 FCFS라고 봐야한다.

![[Pasted image 20250413040513.png]]
위 프로세스를 기준으로 할 때, 다음의 과정이 진행된다.
1. 우선순위가 1인 P2가 우선적으로 처리된다.
2. 우선순위가 2인 P5가 실행되고 이후 P1, P3, P4가 순차적으로 실행된다.

해당하는 Average Waiting Time은 8.2이다.

이러한 우선순위 스케줄링은 선점형 또는 비선점형의 특징을 가진다.
- 선점형 특징 : 실행 중인 프로세스보다 높은 우선순위를 가진 프로세스가 도착 시 CPU를 선점한다. 즉, 중간에 들어오는 프로세스의 경우에 따라 달라진다.
- 비선점형 : 실행 중인 프로세스가 완료될 때 까지 다른 프로세스는 대기한다.

##### Starvation
기아 상태라고 불리는 이것은 우선순위 스케줄링에서의 단점으로 꼽히며 이는 낮은 우선순위를 가진 프로세스가 계속해서 Ready Queue에 들어오는 높은 우선순위 프로세스들에 의해 계속 대기 상태에 머무르는 것을 의미한다. 즉, 무한정 대기(indefinite blocking)의 상황을 의미한다.

이러한 것을 해결하기 위해서 **Aging**의 개념을 도입한다.
나이를 부여한다라는 느낌으로 오래 대기하는 프로세스의 경우 프로세스의 우선순위를 높이는 것이다. 그러나 이런 해결방법은 우선순위를 계속 조정하기에 오버헤드가 발생하기도 한다.

#### RR + Priority Scheduling
우선순위가 동일한 경우에는 다양한 방식으로 우선순위를 조정하여 처리할 수 있지만 이러한 경우 주로 RR을 적용하여 처리한다.
![[스크린샷 2025-04-13 오전 4.18.45.png]]

1. 우선순위가 제일 높은 P4가 실행된다.
2. P4가 처리된 후 우선순위가 같은 P2, P3는 RR 스케줄링을 통해 time quantum마다 프로세스들이 Context Switching을 하며 처리된다.
3. P2가 완료된 후 P3만 남았기에 P3의 처리를 완료한다.
4. P3가 처리된 후 우선순위가 같은 P1, P5는 RR 스케줄링을 통해 time quantum마다 프로세스들이 Context Switching을 하며 처리된다.
5. P1이 완료된 후 남은 P5를 처리한다.

위 과정은 다음의 Gantt Chart로 처리할 수 있다.

![[Pasted image 20250413042326.png]]

#### Multi-Level Queue Scheduling
프로세스들을 여러 개의 큐로 나누고 각 큐에 서로 다른 스케줄링 정책을 적용하는 것이다. 
![[스크린샷 2025-04-13 오전 4.27.10.png]]

해당 스케줄링은 프로세스들의 특징에 따라 큐를 분리하여 효율적으로 관리한다. 스마트폰을 예시로 들었을 때, 전화, 카톡, Network, Display 등이 이루어질 때 이 모든 프로세스들이 동일 priority를 가지지 않기에 위의 그림과 같이 나누어 처리한다.


프로세스의 경우 특성에 따라 여러 큐로 분리되는데 각 큐는 고유한 스케줄링 정책을 가지고 큐 간에 우선순위가 결정된다.
![[Pasted image 20250413043442.png]]
- Real-time Processes : 최상위 우선순위를 가지며, 즉각적인 처리를 수행한다.
- System Processes : 시스템 운영에 필요한 작업으로 높은 우선순위를 가진다.
- Interactive Processes : 사용자와 상호작용이 필요한 작업으로 중간의 우선순위를 가진다.
- Batch Processes : 긴 작업 시간이 필요한 작업으로 가장 낮은 우선순위를 가진다.

큐간의 스케줄링은 다음의 방식들로 처리된다.

1. Fixed-Priority Preemptive Scheduling : 위에서 정의한 각 큐의 절대적인 우선순위를 통해 높은 우선 순위 큐가 비어 있지 않으면 낮은 우선순위 큐의 프로세스가 실행되지 않고 우선순위가 높으면 CPU를 선점한다. 그 예로 Batch Queue에서의 프로세스가 실행 중인 경우 Interactive Queue에 새로운 프로세스가 도착하면 Interactive 프로세스가 우선 실행된다.
2. Time-Slice : CPU 시간을 큐 간에 분할하여 할당한다. 예를 들어, Interactive Queue의 경우 전체 CPU 시간의 80%를 받고 RR 알고리즘을 사용한다. 배치 처리의 경우 20%를 받고 FCFS 알고리즘을 사용한다.


#### Multi-Level Feedback Queue
Multi-Level Queue Scheduling의 확장 버전으로 프로세스가 실행 중에 큐 간 이동이 가능하도록 설계되었다. 이는 CPU Burst Time, I/O 요청의 여부와 같은 동적인 특성에 따라 프로세스의 위치(큐)를 변경하여 성능을 최적화한다.

![[스크린샷 2025-04-13 오전 4.41.43.png]]

위와 같은 경우 주로 CPU Burst Time이 짧은 것들은 주로 상위 큐에 CPU Burst Time이 긴 경우 처리량의 극대화를 위해 하위 큐에 넣어 처리한다.

##### 동작 원리
1. 초기 상태 : 모든 프로세스는 가장 높은 우선순위 큐에서 시작한다.
2. 큐 이동 조건 : CPU Burst Time이 길어 자원을 독점하는 경우 좀 더 낮은 우선수위 큐로 이동한다. 반대로 오랫동안 대기 상태에 있는 경우 높은 우선순위 큐로 이동한다.
3. 큐 간 스케줄링 : 높은 우선순위 큐가 비어 있지 않으면 낮은 우선순위 큐의 프로세스가 실행되지 않는다.

##### 주요 매개 변수
- 큐의 개수
- 각 큐의 스케줄링 알고리즘
- 프로세스의 업그레이드 조건 
- 프로세스 강등 조건
- 큐 진입 조건


위의 과정은 현대에서도 많이 사용되며 위에서 나온 Starvation의 방지, 효율적 자원관리, 다양한 환경에서의 적용이 가능한 처리가 가능하지만 구현의 복잡성, 오버헤드의 발생이 문제가 된다.

## 5.4 Thread Scheduling

현대에서는 멀티스레드 환경이라 Thread Scheduling을 주로 하며 이는 Kernel Threads들을 처리한다. 이러한 상황에서 운영체제는 다음 두 가지를 관리한다고 보면 된다.
- 프로세스 간 스케줄링 : 여러 프로세스 간의 CPU를 할당하는 작업
- 스레드 간 스케줄링 : 하나의 프로세스 내에서 실행되는 여러 스레드 간에 CPU 분배

##### Kernel-Level Threads vs User-Leve Threads
- Kernel-Level Threads : 프로세스의 개념이 아니며 운영체제에 의해 직접 관리되며 각 스레드는 커널에 의해 
- User-Level Threads : 운영체제의 관리가 아니기에 커널은 인지하지 못하며 사용자 라이브러리가 관리하여 프로세스 내부에서만 동작한다.

결국 OS는 CPU Scheduling을 Kernel-Level Threads만을 가지고 수행한다고 보면 된다.

### 5.5 Multiple Processor Scheduling
멀티프로세서 환경에서 CPU자원을 효율적으로 관리하기 위한 스케줄링 방식이다.
멀티 프로세서에서의 아키텍처는 4가지가 존재한다.

- Multi-Core CPU
- Multi-Threaded Cores (하나의 코어에서 여러 개의 스레드를 동시 실행 가능한 시스템)
- NUMA (Non-Uniform Memory Access) systems
- Heterogeneous Multi-Processing

#### Simultaneous Multithreading (SMT)
하나의 물리적 코어에서 여러 스레드를 동시에 실행할 수 있도록 설계된 기술로, 멀티코어 프로세서의 병렬성을 강화시킨다.
![[스크린샷 2025-04-13 오전 8.47.17.png]]


- Common Ready Queue : 모든 코어가 공유하는 하나의 Ready Queue만 존재한다.
- Per-Core Run Queues : 각 코어가 독립적으로 관리하는 Ready Queue가 존재한다.
#### Multi-Threaded Multi-Core System

멀티코어 프로세서와 멀티 스레드 기술을 결합해 성능을 극대화하는 시스템으로 다음의 구조를 가진다.

- Multi Processors : 하나의 물리적 칩에 여러 개의 독립적인 CPU코어를 포함하여 병렬처리를 지원한다. 각 코어는 자체적으로 명령을 실행하고, 캐시 메모리를 공유한다.
- Simultaneous MultiThreading : 하나의 물리적 코어에서 여러 스레드를 동시에 실행할 수 있도록 설계된 기술로 CPU 자원의 유휴 시간을 줄이고 병렬성을 증가시킨다.

해당하는 시스템의 스케줄링은 다음의 처리 방식을 따른다.

1. Load Balancing: 모든 코어가 균등하게 처리하도록 관리한다.
2. Processor Affinity : 프로세서 친화성이라고 불리면 특정 스레드가 동일한 코어에서 실행되도록 설정하여 캐시 활용도를 높인다. 이것은 강제성의 정도에 따라 Soft, Hard로 나뉜다.
3. Push/Pull Migration : Push Migration의 경우 과부하가 발생한 CPU에서 다른 CPU로 강제로 이동시키고 Pull Migration은 유휴 상태의 CPU가 다른 CPU에서 작업을 가져오는 것이다.
![[스크린샷 2025-04-13 오전 9.04.52.png]]

### 5.6 Real-Time CPU Scheduling

실시간 시스템에서의 CPU Scheduling으로 해당 시스템은 시간에 대한 엄격한 준수가 필수적이며, 특정 작업이 정해진 시간 내에 완료되지 않으면 시스템의 신뢰성과 안정성이 손상될 수 있다.
그러나 이러한 작업에도 다음 유형으로 나뉘어서 Trade-off를 취해간다.

- Soft Real-Time Tasks : 시간 제약을 준수하지 못해도 시스템 전체에 큰 영향을 끼치지 않는 것으로 즉, 반드시 해당 시간내에 처리를 하는 것이 아닌 것이다. 예를 들어, 전화기, 음성 RF 신호의 사운드 변환의 경우 중간중간 끊기는 경우가 있는데 이러한 경우가 해당한다.
- Hard Real-Time Tasks : 반드시 시간 제약을 준수해야하는 작업으로 실패 시에 심각한 결과를 초래한다. 그 예시로 항공기 제어 시스템의 경우 시간을 준수하지 못하면 추락사고가 일어날 수도 있다.


##### Rate-Monotonic Scheduling
주기적인 작업에 대해 고정된 우선순위를 부여하는 알고리즘이다. 여기서 작업의 주기가 짧을수록 높은 우선순위를 가지게 된다. 때떄 CPU의 활용도는 69.3으로 제한된다. 구현이 간단하며 예측이 가능하고 주기적 작업에 적합하지만 CPU의 활용도 제한과 주기가 긴 작업들은 장시간 대기하는 기아상태에 빠지게 된다.
![[스크린샷 2025-04-13 오전 9.15.19.png]]

##### Earlist Deadline First
Deadline을 두어 이를 기준으로 우선순위를 부여하는 알고리즘으로 deadline이 임박한 작업이 가장 높은 우선순위를 가진다. 해당 알고리즘은 CPU 활용도가 100%까지 사용가능하며 비주기적 작업에도 처리를 할 수 있다. 다만 Deadline 및 오버헤드 관리에 대한 비용이 많이 든다.
![[스크린샷 2025-04-13 오전 9.16.37.png]]
