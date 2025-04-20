# Docker Introduction

### Docker

컨테이너라는 패키지로 소프트웨어를 제공해주는 툴로 서로 격리된 환경에서 데이터 파이프라인과 데이터 베이스를 운영할 수 있다.

#### Docker Image
설정된 환경의 스냅샷으로, 동일한 이미지를 다양한 환경에서 구동가능하게 하여 재현성을 보장

#### 데이터 엔지니어가 Docker 를 사용해야하는 이유

- 재현 가능성을 제공
- 복잡한 파이프라인의 통합 테스트 가능
- Ci/CD를 통한 소프트웨어 공학 실현
- 데이터 파이프라인의 의존성 제공
- 다양한 환경의 실험 제공

##### 효과
- Test를 위한 로컬 세팅
- 통합 테스트, CI/CD
- 배치 작업
- Spark 사용
- 서버리스 (AWS Lambda)

해당 강의에서는 Docker를 통한 간단한 데이터 파이프라인 구성을 목적으로 Docker를 소개합니다.

### Pipeline Overview

![[Pasted image 20250323200503.png]]



### Docker의 기본 사용법

#### 기본 작성 방법

###### 도커를 통해 파이썬의 pandas를 사용한다고 가정 
```
FROM python:3.9
RUN pip install pandas
ENTRYPOINT [ "bash" ]
```

```
docker build -t test:pandas .
```

###### 로컬의 파일을 포함해서 Docker 이미지 생성


```
FROM python:3.11.9-slim
RUN pip install pandas
WORKDIR /app
COPY pipeline.py pipeline.py
ENTRYPOINT [ "bash" ]
```


```
docker build -t test:pandas
docker run -it test:pandas
```

> [!Docker 터미널 실행하는 방법]
> `docker -it`는 Docker 컨테이너를 상호작용 모드로 실행하기 위한 옵션입니다.
> - i : 표준 입력(sddin)을 열어 컨테이너와 상호작용할 수 있도록 하는 것
> - t : 가상 터미널을 할다앻 터미널 세션을 에뮬레이트 하는 것
>  `docker run -it <Image Name>`

###### 컨테이너에 매개변수 전달

```
FROM python:3.11.9-slim
RUN pip install pandas
WORKDIR /app 
COPY pipeline.py pipeline.py
ENTRYPOINT [ "python", "pipeline.py" ]
```

```
import sys

import pandas

print(sys.argv)

day = sys.argv[1] # '2020-01-01'

# some fancy stuff with pandas

print(f'job finished successfully for day = {day}')
```

build후 다음과 같은 명령어로 처리
```
docker run -it test:pandas 2025-01-18 ['pipeline.py', '2025-01-18']

>> job finished successfully for day = 2025-01-18
```




# NY 택시 데이터 Postgres에 수집

### Docker사용한 Postgres 명령어

##### 사전작업

###### 1. pgcli
```
pip install pgcli
```
######  2. PostgreSQL 실행
```
pgcli -h localhost -p 5432 -u root -d ny_taxi
``` 
###### 3. Jupyter Notebook 실행

###### 4. 택시 데이터 다운로드

**Dataset:**
- [https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- [https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf](https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf)


# pgAdmin과 Postgres 연결

##### while루프를 사용해 CSV를 청크 단위로 삽입

```
from time import time
while True:
	t_start = time()
	df = next(df_iter) df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
	df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
	df.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')
	print('.', end='')
	
	t_end = time()
	
	print('inserted another chunk..., took: %.3f seconds' % (t_end - t_start))
```

##### pgAdmin 실행

도커 명령을 통해 다음과 같이 실행합니다.

```
docker run -it \ -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \ -e PGADMIN_DEFAULT_PASSWORD="root" \ -p 8080:80 \ dpage/pgadmin4
```


![[Pasted image 20250323203819.png]]
![[Pasted image 20250323203826.png]]
![[Pasted image 20250323203831.png]]
![[Pasted image 20250323203840.png]]


# 수집 스크립트 도커화


### 주피터 노트북을 파이썬 스크립트로 변환

`nbconvert` 서브 명령어를 사용해서 Notebook을 파이썬 스크립트로 변환할 수 있다.

```
jupyter nbconvert --to=script upload-data.ipynb

[NbConvertApp] Converting notebook upload-data.ipynb to script [NbConvertApp] Writing 3466 bytes to upload-data.py
```

### Dockerizing 데이터 처리

- 데이터 Ingestion을 위해 pandas라이브러리를 사용해 로컬의 PostgreSQL 데이터베이스에 데이터를 넣는 과정
- 데이터 베이스 연결을 위한 username, password, host, port 등의 파라미터 설정
- OS 라이브러리를 통해 CSV파일 다운로드

```
#!/usr/bin/env python
# coding: utf-8

import os
import argparse

from time import time

import pandas as pd
from sqlalchemy import create_engine


def main(params):
    user = params.user
    password = params.password
    host = params.host 
    port = params.port 
    db = params.db
    table_name = params.table_name
    url = params.url
    
    # the backup files are gzipped, and it's important to keep the correct extension
    # for pandas to be able to open the file
    if url.endswith('.csv.gz'):
        csv_name = 'output.csv.gz'
    else:
        csv_name = 'output.csv'

    os.system(f"wget {url} -O {csv_name}")

    engine = create_engine(f'postgresql://{user}:{password}@{host}:{port}/{db}')

    df_iter = pd.read_csv(csv_name, iterator=True, chunksize=100000)

    df = next(df_iter)

    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

    df.head(n=0).to_sql(name=table_name, con=engine, if_exists='replace')

    df.to_sql(name=table_name, con=engine, if_exists='append')


    while True: 

        try:
            t_start = time()
            
            df = next(df_iter)

            df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
            df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

            df.to_sql(name=table_name, con=engine, if_exists='append')

            t_end = time()

            print('inserted another chunk, took %.3f second' % (t_end - t_start))

        except StopIteration:
            print("Finished ingesting data into the postgres database")
            break

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Ingest CSV data to Postgres')

    parser.add_argument('--user', required=True, help='user name for postgres')
    parser.add_argument('--password', required=True, help='password for postgres')
    parser.add_argument('--host', required=True, help='host for postgres')
    parser.add_argument('--port', required=True, help='port for postgres')
    parser.add_argument('--db', required=True, help='database name for postgres')
    parser.add_argument('--table_name', required=True, help='name of the table where we will write the results to')
    parser.add_argument('--url', required=True, help='url of the csv file')

    args = parser.parse_args()

    main(args)
```

위 파이썬 스크립트를 실행하기 위해 다음의 Dockering Injsetion Script 작성 가능


```
FROM python:3.11.9-slim

RUN apt-get update && apt-get install -y wget
RUN pip install pandas sqlalchemy psycopg2-binary

WORKDIR /app

COPY ingest_data.py ingest_data.py

ENTRYPOINT [ "python", "ingest_data.py" ]
```

이후 다음 스크립트를 실행 시켜 처리


```
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}	
```

도커를 통해 이미지 저장 전에 HTTP 서버를 이용해 데이터를 다운로드 받을 수 있도록 처리


### Docker Compose이용한 서비스 구성

#### Docker Compose

> 여러 컨테이너의 구성을 한번에 정리하고 관리, 실행

블로그 : https://data-newbie.tistory.com/930#deployments
설치 과정 : https://docs.docker.com/compose/install/

#### Postgres 데이터베이스 및 PgAdmin설정

```
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=ny_taxi
    volumes:
      - ./ny_taxi_postgres_data:/var/lib/postgresql/data:rw
    ports:
      - "5432:5432"    
  pgadmin:
    image: dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=root
    ports:
      - "8080:80"
```

- Postgres의 포트 : 5432로 설정 후 컨테이너에서도 같은 포트로 매핑
- pgAdmin을 설정 시, env 설정 및 포트 8080 -> 80으로 매핑
- `docker compose up` 을 통해 실행
- `docker compose down`을 통해 컨테이너 중지

##### Docker Compose 장점

- 네트워크 수동 생성 필요 x
- Detached mode로 실행하여 터미널을 다시 사용할 수 있는 장점
- `docker compose down` 으로 여러 터미널 창을 보지 않고 한 번에 종료 가능
- 로컬 세팅, 통합 테스트에 유용

[pgadmin에서의 테스트 쿼리 실행]

![[Pasted image 20250323211429.png]]


# GCP

> Google에서 제공하는 클라우드 컴퓨팅 서비스로 다양한 컴퓨팅, 스토리지, 애플리케이션 개발을 위한 다양한 호스팅 서비스가 제공됩니다.

![[Pasted image 20250323204757.png]]


# Terraform

> HashiCorp의 오픈소스 도구로 코드를 사용해서 클라우드 인프라를 만들고 관리하는 도구입니다. 마치 레고 블록으로 건물을 짓듯이, Terraform 코드를 사용해서 서버, 데이터베이스, 네트워크 같은 클라우드 자원들을 정의하고 만들 수 있습니다.

##### 특징
- 인프라 수명 주기 관리
- 버전 제어 커밋
- 스택 기반 배포에 매우 유용하며 AWS, GCP, Azure, K8S와 같은 클라우드 공급자와 함께 사용할 수 있습니다.
- 배포 전반에 걸쳐 리소스 변경 사항을 추적하기 위한 상태 기반 접근 방식
##### 사용하는 이유
- 인프라 추적의 단순화
- 더 쉬운 협업
- 재현성
- 리소스 제거 보장


#### 테라폼 작동 방식
Terraform은 코드형 인프라(IaC) 도구로, 인프라를 선언적 구성 파일로 정의하고 프로비저닝 및 관리

**Terraform Core**
•	Terraform의 핵심 요소로, RPC(원격 프로시저 호출)를 통해 플러그인과 통신.
주요 역할
•	구성 파일 읽기 및 종속성 그래프 생성.
•	리소스 상태 관리(`terraform.tfstate` 파일).
•	계획 실행(`terraform plan`) 및 적용(`terraform apply`).

**Terraform Plugin**
•	외부 바이너리로 구현된 플러그인으로, 특정 클라우드 제공자와 통신.
주요 역할
•	API 호출을 통해 리소스 생성/수정/삭제(CRUD) 처리.
•	인증 및 서비스 매핑.

**State Management**
•	Terraform은 리소스의 상태를 `.tfstate` 파일에 저장하여 인프라와 코드 간의 동기화를 유지.
•	상태 파일은 리소스 간 종속성을 추적하며, 성능을 최적화.

#####  Declarations
1.	`terraform`
	•	기본 설정을 정의하며, 인프라를 프로비저닝하는 데 필요한 정보를 설정합니다.
	•	주요 속성:
	•	`required_version`: Terraform 버전 요구사항.
	•	`backend`: 상태 파일 저장 위치 지정(예: `local`, `remote`).
	•	`required_providers`: 사용해야 할 클라우드 제공자 설정.
2.	`provider`
	•	Terraform이 관리할 리소스 유형과 데이터 소스를 추가하는 모듈.
	•	예: AWS, Google Cloud, Azure 등.
3.	`resource`
	•	인프라의 구성 요소를 정의하는 블록.
	•	예: `aws_instance`, `google_storage_bucket`.
4.	`variable` & `locals`
	•	`variable`: 런타임 시 동적으로 값을 전달하기 위한 입력 변수.
	•	`locals`: 고정된 값이나 계산된 값을 저장하는 로컬 변수.

##### 파일 구성
1.	`main.tf`: Terraform의 메인 구성 파일로, 리소스와 제공자를 정의.
2.	`variables.tf`: 변수 선언 및 기본값 설정.
3.	`resources.tf`: 리소스 블록을 분리하여 작성(선택 사항).
4.	`output.tf`: 생성된 리소스의 출력값을 정의(선택 사항).
5.	`.tfstate`: 현재 인프라 상태를 저장하는 파일.


##### Terraform 작동 흐름
1.	Write: 선언적 구성 파일 작성(`main.tf`, `variables.tf` 등).
2.	Plan: 현재 상태와 원하는 상태를 비교하여 변경 사항을 계획(`terraform plan`).
3.	Apply: 계획된 변경 사항을 실제로 적용(`terraform apply`).
4.	Destroy: 기존 인프라를 삭제(`terraform destroy`).


##### 주요 명령어

1.	`terraform init`: Terraform 프로젝트를 초기화하고 필요한 플러그인을 다운로드합니다.
2.	`terraform plan`: 변경 사항을 미리 확인하여 어떤 리소스가 생성, 수정, 삭제될지 계획을 세웁니다.
3.	`terraform apply`: Terraform 코드를 실제로 적용하여 인프라를 생성하거나 수정합니다.
4.	`terraform destroy`: Terraform으로 생성된 모든 리소스를 삭제합니다.



### Terraform 단일 파일 배포, 변수 파일을 사용한 배포

> GCP의 기한이 사라져서 제가 접속이 안되어서... 여기를 다시 못했습니다..


