

> Feature + flag

자신의 앱, 서비스, 기능에 어떤 깃발값이 꽂혀 있는지 보기 위한 것으로 이것을 통해 프로덕트의 기능을 on/off 처리하기 위해 존재하며 이를 통한 유연한 대응을 목적으로 가진 것이다.

어떤 기능에 현재 무슨 깃발이 꼽혀 있는지 알기 위한것으로 기능에 대한 on / off 처리를 통한 유연한 대응

- MSA, 클라우드 등의 발전과 맞물려 깃발을 꽂아 어떤 기능들을 고객들에게 서빙할 지를 파악하기 위함
- 최근 빠른 개발, 안정된 Serving을 위해 발전

최근의 동향(빠른 개발)에 맞춰 우리 서비스를 유연하게 서빙하기 위한 것이며, feture에 flag를 꽂을 수 있는 것


### Featureflag의 목적 및 효율성

- 기능 -고객 간 유연하고 빠른 대응 : MSA 특징과 맞물려, 프로덕트의 유연성/효율성을 증가시킬 수 있다.
- 고객을 알기 위한 수단
- on/off를 통해 앱의 가치를 높이는 방향으로 대용량 트래픽을 받기 위해서도 사용


### Featureflag의 요구 사항

- 대용량 트래픽을 처리할 수 있는 구조 : 고객 A, B 간 기능이 기하급수적으로 늘어나면 복잡도는 N^2 만큼 증가
- 각 프로덕트/서비스 간 독립적으로 플래그를 식별할 수 있는 아키텍처 : on/off로 인해 다른 기능이 에러가 나지 않도록
- 사용 용이성과 단순한 UI 및 기능성 (API/SDK) : Feature들을 변경하기 위한 UI 혹은 툴
- 높은 장애 내구성 (Fault Tolerant) : 장애가 나지 않도록


## FeatureFlag 레퍼런스 및 인터페이스

### OpenFeature

![[Pasted image 20250515014037.png]]

> https://openfeature.dev/docs/reference/intro

사용가능한 레퍼런스로 **OpenFeature**가 존재한다.

OpenFeature는 FeatureFlag의 표준화된 인터페이스를 실제로 구현한 오픈소스이다.
즉, FeatureFlagging을 위한 레퍼런스를 제공한다.

![[Pasted image 20250515014319.png]]


#### 구성요소
- Provider : OpenFeatureFlag의 구현체
- Evaluation : `client.get***` 에서처럼 key값 가져와 판단하는 것

![[Pasted image 20250515014529.png]]



## Remote Configuration 과의 차이

본질적으로 다른 곳에서 어던 수단을 통해 정보를 가져오고 활용한다는 것은 동일하다.
하지만 Remote Configuration은 앱/서비스의 실행, 배포, 빌드 시 영향을 미치는 Config를 중앙 저장소에서 가져오는 **목적성**이 강하다.

즉, _설정 값_ 을 원격에서 중앙 저자오하 시켜 사용하여 일원화시킨다.

결국, 다음과 같은 차이가 있다.
- 높은 대용량 시스템에 대한 고민 x
- 보안, 인증/인가에 대한 영역의 고민이 더 필요



## FeatureFlag 사용 결정하기

### 왜 필요한가?

- 고객이 충분한가
- 서비스의 고객트래픽이 충분한가
- B2C 서비스로서 고객을 알야할 가치가 충분한가
- 빠른 속도로 개발하고, 자주 많은 기능들을 배포하고 있는가
- 장애 시, Risky한 기능들이 있는가


#### 어떤 문제를 해결하는가?

> 서비스와 고객 간 기능을 서빙하기 위한 Buffer 생성

![[Pasted image 20250515174908.png]]

기능의 서빙을 한 단계 중간에서 받아주도록 하여 모든 상황에 유연한 대처가 가능하도록 만드는 것으로 이를 통해 큰 가치 창출 가능



[[FeatureFlag 실습 요구사항항]]


![[Pasted image 20250526211923.png]]

Featureflag Service: 우리가 만들 것이며 외부적으로 Flagd SDK를 사용해 외부 엔진 사용
Flagd : 외부에 존재하면 SDK를 통해 RPC 방식을 통한 호출

이렇게 호출된 Flagd는 Origin Flagd에서는 어떤 Feature에 대한 Flag (Key-value)에 대한 원본 소스 값을 가져올 수 있고 다시 이 소스들이 단순한 파일이 될 수도 있고 외부의 Endpoint가 될 수 있다.

Cache : SDK로 받은 Key에 대한 Value를 저장

