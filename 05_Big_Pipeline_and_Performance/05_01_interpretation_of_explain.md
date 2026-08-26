
## 목차

* [1. `EXPLAIN` 함수 설명](#1-explain-함수-설명)
  * [1-1. `EXPLAIN` 함수의 상세 해석 방법](#1-1-explain-함수의-상세-해석-방법)
* [2. 쿼리 병목 지점 확인 실습](#2-쿼리-병목-지점-확인-실습)
  * [2-1. 실행 계획 및 실제 실행 시간 분석](#2-1-실행-계획-및-실제-실행-시간-분석) 
  * [2-2. 테이블 입출력, scan 병목 확인](#2-2-테이블-입출력-scan-병목-확인) 
* [3. 조인 방식 이해](#3-조인-방식-이해)
  * [3-1. Hash Join](#3-1-hash-join)
  * [3-2. Nested Loop Join](#3-2-nested-loop-join)
  * [3-3. Merge Join](#3-3-merge-join)
* [4. 참고 자료](#4-참고-자료)

## 1. `EXPLAIN` 함수 설명

`EXPLAIN` 함수 (키워드) 는 **실행하려는 SQL 쿼리의 실행 계획** 을 표시하는 함수이다.

* 기본 구분
  * 다음과 같이 `EXPLAIN` 키워드를 실행하려는 SQL 쿼리 앞에 추가하면 된다.

```sql
EXPLAIN
SELECT ...
FROM ...
WHERE ...
```

* 예시

```sql
explain
select mle_id,
    round(avg(eval_performance_score), 2) as avg_performance,
    round(avg(eval_competency_score), 2) as avg_competency,
    round(avg(eval_attitude_score), 2) as avg_attitude,
    round(avg(eval_performance_score + eval_competency_score + eval_attitude_score), 2) as avg_total
from probation_data
group by mle_id
```

* 실행 결과

```
id|select_type|table         |partitions|type|possible_keys|key|key_len|ref|rows|filtered|Extra          |
--+-----------+--------------+----------+----+-------------+---+-------+---+----+--------+---------------+
 1|SIMPLE     |probation_data|          |ALL |             |   |       |   |1162|   100.0|Using temporary|
```

### 1-1. `EXPLAIN` 함수의 상세 해석 방법

* `EXPLAIN` 결과의 각 컬럼에 대한 설명

| 컬럼              | 설명                                 |
|-----------------|------------------------------------|
| `id`            | 실행 순서                              |
| `select_type`   | SELECT 문 유형                        |
| `table`         | 참조하는 테이블 이름                        |
| `type`          | 테이블 데이터의 JOIN, SELECT type         |
| `possible_keys` | 데이터 조회 시 인덱스 목록 (SQL 문 최적화 용)      |
| `key`           | 실제 사용되는 인덱스                        |
| `key_len`       | 실제 사용되는 인덱스의 길이                    |
| `ref`           | `reference` (테이블 JOIN 시 액세스 관련 조건) |
| `rows`          | 조사 대상 행 개수                         |
| `filtered`      | 데이터 유지/제거 비율                       |
| `Extra`         | SQL 문 수행 관련 추가 정보                  |

* `select_type` 설명

| `select_type` 구분  | 설명                                        |
|-------------------|-------------------------------------------|
| `SIMPLE`          | 단순 `SELECT` 문 (UNION, sub-query 없음)       |
| `PRIMARY`         | **sub-query 를 포함** 한 SELECT 문             |
| `UNION`           | **union** 또는 **union all** 을 포함한 SELECT 문 |
| `DEPENDENT_UNION` | `UNION` 과 동일 (단, 바깥쪽 쿼리에 의존성 있는 쿼리)       |

## 2. 쿼리 병목 지점 확인 실습

### 2-1. 실행 계획 및 실제 실행 시간 분석

실행 계획 및 실행 시간 분석을 위해서는 `EXPLAIN ANALYZE` 를 사용한다.

* SQL 쿼리

```sql
explain analyze
with raw_salary_krw as (
    select mle_id,
           cast(REGEXP_REPLACE(raw_salary_krw, '[^0-9]', '') as signed) as salary_num
    from probation_data
)
select
  distinct coalesce(rsk.mle_id, pd.mle_id) as mle_id,
  pd.department_team,
  rsk.salary_num
from probation_data as pd
join raw_salary_krw as rsk
  on pd.mle_id = rsk.mle_id 
where pd.department_team = 'LLM & Generative AI'
  and rsk.salary_num >= 77889900;
```

* 실행 결과

```
-> Table scan on <temporary>  (cost=1617..1637 rows=1350) (actual time=4.63..4.64 rows=67 loops=1)
    -> Temporary table with deduplication  (cost=1617..1617 rows=1350) (actual time=4.63..4.63 rows=67 loops=1)
        -> Inner hash join (probation_data.mle_id = pd.mle_id)  (cost=1482 rows=1350) (actual time=1.11..3.69 rows=603 loops=1)
            -> Filter: (cast(regexp_replace(probation_data.raw_salary_krw,'[^0-9]','') as signed) >= 77889900)  (cost=0.174 rows=116) (actual time=0.0362..2.33 rows=937 loops=1)
                -> Table scan on probation_data  (cost=0.174 rows=1162) (actual time=0.0061..1.02 rows=1201 loops=1)
            -> Hash
                -> Filter: (pd.department_team = 'LLM & Generative AI')  (cost=123 rows=116) (actual time=0.0398..0.96 rows=255 loops=1)
                    -> Table scan on pd  (cost=123 rows=1162) (actual time=0.0224..0.819 rows=1201 loops=1)
```

* 실행 결과 해석
  * 2개의 테이블을 조인할 때, `Inner hash join` 으로 조인한다.
  * `actual time` 을 통해 실제 실행 시간을 알 수 있다.
  * `rows` 를 통해 결과 row의 개수를 확인할 수 있다.

### 2-2. 테이블 입출력, scan 병목 확인

다음과 같이 **MySQL의 `sys` 스키마를 이용** 하여 행 수, 지연 시간 등을 확인할 수 있다.

```sql
SELECT 
  table_schema,
  table_name,
  rows_fetched,
  io_read_requests,
  io_read_latency / 1000000000 AS read_latency_ms
FROM sys.schema_table_statistics
WHERE table_name = 'probation_data';
```

* 실행 결과

```
table_schema|table_name    |rows_fetched|io_read_requests|read_latency_ms|
------------+--------------+------------+----------------+---------------+
sys         |probation_data|      114559|              33|  0.00000000585|
```

## 3. 조인 방식 이해

### 3-1. Hash Join

### 3-2. Nested Loop Join

### 3-3. Merge Join

## 4. 참고 자료

* [MySQL 실행계획(explain) 정리 - Lifealong](https://0soo.tistory.com/235)
