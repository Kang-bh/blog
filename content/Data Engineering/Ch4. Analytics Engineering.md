---
aliases:
  - "\bCh4. Analytics Engineering"
---


## ETL

Extract > Transform > Load
![[스크린샷 2025-03-02 오전 7.57.34.png]]
## ELT


Extract > Load > Transform
![[스크린샷 2025-03-02 오전 7.57.41.png]]


## Kimball's Dimension Modeling

비즈니스 사용자가 이해할 수 있는 데이터를 제공하면서 빠른 쿼리 성능의 제공

### Facts tabls
측정 지표 또는 비즈니스에 관한 데이터
비즈니스 프로세스에 해당되는 부분
하나의 동사로 작동

### Dimensions tables
비즈니스 맥락과 엔티티를 제공
하나의 명사로 작동


### Dimension Modeling의 구조
![[스크린샷 2025-03-02 오전 8.02.41.png]]

- stage Area
	- raw data 포함
	- 노출 불가

- Procesing Area
	- 모델에 적용단계
	- 효율성에 초점

- Presentation area
	- 최종적으로 보여지는 단계



# DBT

SQL, Python 이용한 분석 코드를 배포할 수 있는 것으로 Data Build Tool로 불린다.

다양한 소스에서 데이터를 가져와 처리


### DBT Core

데이터 변환을 처리해주는 오픈소스 프로젝트로 다음을 수행 가능
- dbt project 수행
- SQL, DB 로직 처리
- CLI Interface 제공


### DBT Cloud

SaaS application으로 dbt 개발, 관리 가능

- Web 기반의 제공
- 환경 제공
- 오케스트레이션, 로킹, 알람 등 처리

## DBT 사용 법

- 
