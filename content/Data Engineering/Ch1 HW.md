## Question 1. Docker 첫 실행 이해하기

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

이 이미지에서 `pip`의 버전은 무엇인가요?

- `24.3.1`
- 24.2.1
- 23.3.1
- 23.2.1

## Question 2. Docker 네트워킹과 docker-compose 이해하기

다음 `docker-compose.yaml`이 주어졌을 때, **pgadmin**이 postgres 데이터베이스에 연결하기 위해 사용해야 하는 `hostname`과 `port`는 무엇인가요?

```yaml
services:
  **db**:
    container_name: **postgres**
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - **'5433:5432'**
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    **container_name: pgadmin**
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    **ports:
      - "8080:80"**
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- **`postgres:5432`**
- `db:5432`

## Question 3. 운행 구간 수

**2019년 10월 1일(포함)부터 2019년 11월 1일(제외)**까지 기간 동안, **각각 다음과 같은 운행**이 몇 번 있었나요:

```sql
-- 1. 중복 체크
SELECT 
    COUNT(*) AS cnt
FROM green_tripdata
GROUP BY 
    "VendorID", "lpep_pickup_datetime",
    "lpep_dropoff_datetime", "store_and_fwd_flag",
    "RatecodeID", "PULocationID",
    "DOLocationID", "passenger_count",
    "trip_distance", "fare_amount",
    "extra", "mta_tax",
    "tip_amount", "tolls_amount",
    "ehail_fee", "improvement_surcharge", "total_amount",
    "payment_type", "trip_type", "congestion_surcharge"
HAVING
    COUNT(*) > 1;
-- 2. 운행수 확인
SELECT 
    SUM(CASE WHEN Trip_distance <= 1 THEN 1 END) AS "1. 1마일 이하",
    SUM(CASE WHEN Trip_distance > 1 AND Trip_distance <= 3 THEN 1 END) AS "2. 1마일 초과 3마일 이하",
    SUM(CASE WHEN Trip_distance > 3 AND Trip_distance <= 7 THEN 1 END) AS "3. 3마일 초과 7마일 이하",
    SUM(CASE WHEN Trip_distance > 7 AND Trip_distance <= 10 THEN 1 END) AS "4. 7마일 초과 10마일 이하",
    SUM(CASE WHEN Trip_distance > 10 THEN 1 END) AS "5. 10마일 초과"
FROM green_tripdata
WHERE lpep_pickup_datetime >= '2019-10-01' AND lpep_dropoff_datetime< '2019-11-01';

**-- "1. 1마일 이하"	"2. 1마일 초과 3마일 이하"	"3. 3마일 초과 7마일 이하"	"4. 7마일 초과 10마일 이하"	"5. 10마일 초과"
-- 104802	198924	109603	27678	35189**

```

1. 1마일 이하
2. 1마일 초과 3마일 이하
3. 3마일 초과 7마일 이하
4. 7마일 초과 10마일 이하
5. 10마일 초과

답:

- 104,802; 197,670; 110,612; 27,831; 35,281
- **`104,802; 198,924; 109,603; 27,678; 35,189`**
- 104,793; 201,407; 110,612; 27,831; 35,281
- 104,793; 202,661; 109,603; 27,678; 35,189
- 104,838; 199,013; 109,645; 27,688; 35,202
## Question 4. 일별 최장 운행

운행 거리가 가장 긴 승차일은 언제였나요? 계산 시 승차 시간을 기준으로 하세요.

```sql
SELECT DATE(lpep_pickup_datetime)
FROM green_tripdata
WHERE Trip_distance = (
	SELECT MAX(Trip_distance) FROM green_tripdata
)
-- 2019-10-31
```

팁: 각 날짜별로 가장 긴 거리의 한 운행만 고려하세요.

- 2019-10-11
- 2019-10-24
- 2019-10-26
- **`2019-10-31`**

## Question 5. 상위 3개 승차 구역

```sql
WITH trip AS (
	SELECT "PULocationID",
			SUM(total_amount) AS "total_amount"
	FROM green_tripdata
	WHERE DATE(lpep_pickup_datetime) = '2019-10-18'
	GROUP BY "PULocationID"
	ORDER BY "total_amount" DESC
)
SELECT zones."Zone" AS "Zone",
       trip."PULocationID" AS "PULocationID",
	   trip."total_amount" AS "total_amount"
FROM zones 
INNER JOIN trip ON trip."PULocationID" = zones."LocationID"
ORDER BY "total_amount" DESC
LIMIT 3;

-- "Zone"	"PULocationID"	"total_amount"
-- "East Harlem North"	74	18686.680000000084
-- "East Harlem South"	75	16797.260000000064
-- "Morningside Heights"	166	13029.790000000032
```

2019-10-18일에 `total_amount`가 13,000을 초과한 상위 승차 위치는 어디인가요?

날짜 필터링 시 `lpep_pickup_datetime`만 고려하세요.

- **`East Harlem North, East Harlem South, Morningside Heights`**
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

## Question 6. 최대 팁

```sql
WITH trip AS (
    SELECT "PULocationID",
           "DOLocationID",
           "tip_amount"
    FROM green_tripdata
    WHERE DATE(lpep_pickup_datetime) >= '2019-10-01' AND DATE(lpep_pickup_datetime) < '2019-11-01'
    ORDER BY "tip_amount" DESC
)
SELECT PUZone."Zone" AS "PULocation",
       DOZone."Zone" AS "DOLocation",
       trip."tip_amount" AS "tip_amount"
FROM trip
INNER JOIN zones AS PUZone ON trip."PULocationID" = PUZone."LocationID"
INNER JOIN zones AS DOZone ON trip."DOLocationID" = DOZone."LocationID"
WHERE PUZone."Zone" = 'East Harlem North'
ORDER BY "tip_amount" DESC
LIMIT 1;

-- "JFK Airport"
```

2019년 10월에 "East Harlem North" 구역에서 승차한 승객들 중에서 어느 하차 구역에서 가장 큰 팁이 있었나요?

_**주의**_: `trip`이 아닌 `tip`입니다. 구역의 ID가 아닌 이름이 필요합니다.

- Yorkville West
- **`JFK Airport`**
- East Harlem North
- East Harlem South

## Question 7. Terraform 워크플로우

다음 중 어떤 순서가 각각 다음 워크플로우를 설명하나요:

1. 공급자 플러그인 다운로드 및 백엔드 설정
2. 제안된 변경사항 생성 및 계획 자동 실행
3. terraform이 관리하는 모든 리소스 제거

답:

- terraform import, terraform apply -y, terraform destroy
- **teraform init,** terraform plan -auto-apply, terraform rm
- **terraform init**, terraform run -auto-approve, terraform destroy
- **`terraform init**, **terraform apply -auto-approve, terraform destroy**`
- terraform import, terraform apply -y, terraform rm