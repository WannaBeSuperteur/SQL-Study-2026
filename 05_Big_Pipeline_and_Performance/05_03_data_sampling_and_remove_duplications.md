
## 목차

* [1. 함수 설명](#1-함수-설명)
  * [1-1. `TABLESAMPLE` 함수](#1-1-tablesample-함수) 
  * [1-2. `RANDOM` 함수](#1-2-random-함수)
  * [1-3. `ROW_NUMBER` 함수](#1-3-rownumber-함수)
* [2. 층화 추출 (Startified Sampling) 실습](#2-층화-추출-startified-sampling-실습)
* [3. 참고 문서](#3-참고-문서)

## 1. 함수 설명

| 함수            | 기본 구문                                   | 함수 설명                                                                               |
|---------------|-----------------------------------------|-------------------------------------------------------------------------------------|
| `TABLESAMPLE` | `TABLESAMPLE sampling_method (percent)` | 샘플링 메서드 `sampling_method` 를 사용하여 **`percent`% 만큼의 비율** 의 데이터 추출                     |
| `RANDOM`      | `RANDOM()` 또는 `RAND()`                  | 0부터 1 사이의 난수 값 (pseudo random) 생성                                                   |
| `ROW_NUMBER`  | `ROW_NUMBER() OVER (ORDER BY ...)`      | [(참고)](../04_Sequential_and_Window_Functions/04_01_ranking_and_quantile.md#1-함수-설명) |

### 1-1. `TABLESAMPLE` 함수

`TABLESAMPLE` 함수는 `sampling_method` 에 해당하는 샘플링 방법을 사용하여, 전체 데이터 중 일부를 추출하는 함수이다.

* `sampling_method` 에 들어가는 샘플링 방법
* **MySQL 에서는 지원하지 않는다.**
* 샘플링 방법

| 샘플링 방법 (`sampling_method`)<br>(PostgreSQL 9.5 기준) | 설명                                   |
|---------------------------------------------------|--------------------------------------|
| `system`                                          | **디스크의 데이터 페이지 단위** 로 해당 비율만큼 랜덤 샘플링 |
| `bernoulli`                                       | **개별 row** 단위로 해당 비율만큼 랜덤 샘플링        |

### 1-2. `RANDOM` 함수

`RANDOM` 함수는 0부터 1까지의 난수를 생성하는 함수이다.

* MySQL에서는 **`RAND()` 함수를 대신 사용** 한다.

| SQL 쿼리문                               | 실행 결과               | 실행 결과 범위          |
|---------------------------------------|---------------------|-------------------|
| `select rand()`                       | 0.29065022830911935 | 0.0 - 1.0 사이의 값   |
| `select rand() * 100`                 | 11.998317960648162  | 0.0 - 100.0 사이의 값 |
| `select cast(rand() * 100 as signed)` | 91                  | 0 - 100 사이의 정수 값  |

### 1-3. `ROW_NUMBER` 함수

`ROW_NUMBER` 함수는 `order by ...`로 지정된 정렬 기준에 따라 정렬했을 때의 **행 번호 (row number)** 를 출력하는 함수이다.

* 예시 SQL 구문

```sql
with distinct_mle_ids as (
	select distinct mle_id
	from probation_data
)
select row_number() over (order by mle_id) as row_num,
       mle_id
from distinct_mle_ids;
```

* 실행 결과

```
row_num|mle_id       |
-------+-------------+
      1|MLE-2023-0001|
      2|MLE-2023-0002|
      3|MLE-2023-0003|
      4|MLE-2023-0004|
      5|MLE-2023-0005|
      6|MLE-2023-0006|
      7|MLE-2023-0007|
      8|MLE-2023-0008|
      9|MLE-2023-0010|
     10|MLE-2023-0013|
```

## 2. 층화 추출 (Startified Sampling) 실습

## 3. 참고 문서

* [Tablesample In PostgreSQL 9.5 - EDB POSTGRESQL](https://www.enterprisedb.com/blog/tablesample-postgresql-95)
