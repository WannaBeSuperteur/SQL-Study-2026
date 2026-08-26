
## 목차

* [1. `EXPLAIN` 함수 설명](#1-explain-함수-설명)
  * [1-1. `EXPLAIN` 함수의 상세 해석 방법](#1-1-explain-함수의-상세-해석-방법)
* [2. 쿼리 병목 지점 확인 실습](#2-쿼리-병목-지점-확인-실습)
  * [2-1. 실행 계획 및 실제 실행 시간 분석](#2-1-실행-계획-및-실제-실행-시간-분석) 
  * [2-2. 테이블 입출력, scan 병목 확인](#2-2-테이블-입출력-scan-병목-확인) 
  * [2-3. 지연을 유발하는 상위 쿼리 추적](#2-3-지연을-유발하는-상위-쿼리-추적) 
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

### 2-3. 지연을 유발하는 상위 쿼리 추적

다음과 같이 상위 쿼리를 추적할 수 있다.

```sql
select *
FROM sys.statement_analysis
ORDER BY total_latency desc;
```

* 실행 결과

```
query                                                            |db |full_scan|exec_count|err_count|warn_count|total_latency|max_latency|avg_latency|lock_latency|cpu_latency|rows_sent|rows_sent_avg|rows_examined|rows_examined_avg|rows_affected|rows_affected_avg|tmp_tables|tmp_disk_tables|rows_sorted|sort_merge_passes|max_controlled_memory|max_total_memory|digest|first_seen                |last_seen                 |
-----------------------------------------------------------------+---+---------+----------+---------+----------+-------------+-----------+-----------+------------+-----------+---------+-------------+-------------+-----------------+-------------+-----------------+----------+---------------+-----------+-----------------+---------------------+----------------+------+--------------------------+--------------------------+
SELECT * FROM `sys` . `stateme ... tal_latency` DESC LIMIT ?, ...|sys|*        |         9|        0|         0|95.49 ms     |63.46 ms   |10.61 ms   |87.00 us    |  0 ps     |      530|           59|         1060|              118|            0|                0|         0|              0|        530|                0|1.17 MiB             |2.14 MiB        | ...  |2026-08-26 09:44:52.173877|2026-08-26 12:26:01.950486|
SELECT * FROM `information_sch ... = ? AND `t` . `TABLE_NAME` = ?|   |         |         4|        0|         0|95.42 ms     |72.64 ms   |23.86 ms   |43.00 us    |  0 ps     |        4|            1|           12|                3|            0|                0|         0|              0|          0|                0|1.39 MiB             |1.92 MiB        | ...  |2026-08-23 20:12:08.769743|2026-08-26 08:33:52.857546|
SELECT `WORD` FROM `INFORMATIO ... `RESERVED` = ? ORDER BY `WORD`|   |         |         4|        0|         0|90.20 ms     |49.96 ms   |22.55 ms   |59.00 us    |  0 ps     |     1048|          262|         2096|              524|            0|                0|         4|              0|       1048|                0|1.19 MiB             |1.81 MiB        | ...  |2026-08-23 20:12:08.550737|2026-08-26 08:33:52.470233|
COMMIT                                                           |   |         |         3|        0|         0|9.71 ms      |4.99 ms    |3.24 ms    |  0 ps      |  0 ps     |        0|            0|            0|                0|            0|                0|         0|              0|          0|                0|18.05 KiB            |339.93 KiB      | ...  |2026-08-24 22:13:45.807469|2026-08-24 22:21:13.382301|
SELECT `table_schema` , TABLE_ ... RE TABLE_NAME = ? LIMIT ?, ...|sys|*        |         3|        0|         3|9.63 ms      |3.87 ms    |3.21 ms    |7.00 us     |  0 ps     |        3|            1|            3|                1|            0|                0|         9|              0|          3|                0|1.29 MiB             |1.86 MiB        | ...  |2026-08-26 09:38:25.797786|2026-08-26 09:38:28.685240|
SET `autocommit` = ?                                             |   |         |        34|        0|         0|9.17 ms      |653.00 us  |269.70 us  |  0 ps      |  0 ps     |        0|            0|            0|                0|            0|                0|         0|              0|          0|                0|18.05 KiB            |339.93 KiB      | ...  |2026-08-23 20:12:08.349402|2026-08-26 08:33:54.143983|
SELECT QUERY AS `digest_text`  ... tal_latency` DESC LIMIT ?, ...|sys|*        |         1|        0|         0|82.55 ms     |82.55 ms   |82.55 ms   |50.00 us    |  0 ps     |        0|            0|           57|               57|            0|                0|         0|              0|          0|                0|1.21 MiB             |2.17 MiB        | ...  |2026-08-26 09:44:06.174564|2026-08-26 09:44:06.174564|
SELECT QUERY AS `digest_text`  ... Y `total_latency` DESC LIMIT ?|sys|         |         1|        1|         0|8.86 ms      |8.86 ms    |8.86 ms    |38.00 us    |  0 ps     |        0|            0|            0|                0|            0|                0|         0|              0|          0|                0|1.25 MiB             |2.21 MiB        | ...  |2026-08-26 09:43:15.136612|2026-08-26 09:43:15.136612|
SELECT `TABLE_SCHEMA` , TABLE_ ...  ? ORDER BY TABLE_NAME LIMIT ?|sys|*        |        38|        0|         0|70.60 ms     |5.04 ms    |1.86 ms    |202.00 us   |  0 ps     |        0|            0|        13862|              365|            0|                0|         0|              0|          0|                0|1.92 MiB             |2.46 MiB        | ...  |2026-08-23 21:30:34.995107|2026-08-25 22:02:59.021854|
WITH `raw_salary_krw` AS ( SEL ... `salary_num` >= ? LIMIT ?, ...|sys|*        |         1|        0|        99|7.00 ms      |7.00 ms    |7.00 ms    |3.00 us     |  0 ps     |       67|           67|         2402|             2402|            0|                0|         1|              0|          0|                0|1.36 MiB             |1.70 MiB        | ...  |2026-08-26 09:27:17.135526|2026-08-26 09:27:17.135526|
SELECT * FROM `information_sch ... CHEMATA` WHERE SCHEMA_NAME = ?|   |         |         1|        0|         0|693.40 us    |693.40 us  |693.40 us  |4.00 us     |  0 ps     |        1|            1|            4|                4|            0|                0|         0|              0|          0|                0|1.10 MiB             |1.38 MiB        | ...  |2026-08-24 22:13:45.466692|2026-08-24 22:13:45.466692|
```

* 실행 결과 해석

| 컬럼                          | 설명                                               |
|-----------------------------|--------------------------------------------------|
| `query`                     | 실행한 쿼리 (normalized)                              |
| `db`                        | 쿼리를 실행한 데이터베이스 (DB) (DB가 없을 경우 `NULL`)           |
| `full_scan`                 | 전체 table scan 횟수                                 |
| `exec_count`                | 전체 쿼리 실행 횟수                                      |
| `error_count`               | 해당 쿼리 실행 시 발생한 전체 error 횟수                       |
| `warn_count`                | 해당 쿼리 실행 시 발생한 전체 warning 횟수                     |
| `total_latency`             | 해당 쿼리에 의한 **전체 wait latency** 의 합계               |
| `max_latency` `avg_latency` | 해당 쿼리에 의한 **wait latency 발생 건의 최대, 평균 latency**  |
| `lock_latency`              | 해당 쿼리에 의해 발생한 **lock 대기 시간의 합계**                 |
| `rows_sent`                 | 해당 쿼리에 의해 반환된 **전체 row 개수**                      |
| `rows_sent_avg`             | 해당 쿼리에 의해 반환된 **쿼리 실행 당 평균 row 개수**              |
| `rows_affected`             | 해당 쿼리에 의해 영향을 받은 **전체 row 개수**                   |
| `rows_affected_avg`         | 해당 쿼리에 의해 영향을 받은 **쿼리 실행 당 평균 row 개수**           |
| `max_total_memory`          | 해당 쿼리에 의한 **전체 memory 사용량 (byte)** 의 최댓값         |
| `first_seen`                | 해당 쿼리의 최초 발생 시각                                  |
| `last_seen`                 | 해당 쿼리의 마지막 발생 시각                                 |

## 3. 조인 방식 이해

| 조인 방식                       | 설명                                                                       |
|-----------------------------|--------------------------------------------------------------------------|
| 해시 조인 (Hash Join)           | **작은 테이블 (Build Input), 큰 테이블 (Probe Input)** 에 대해, 해시 함수를 이용하는 형태의 JOIN |
| 중첩 루프 조인 (Nested Loop Join) | 두 테이블 중 **작은 테이블에서 조건에 일치하는 row를 큰 테이블에서 반복 탐색**                         |
| 병합 조인 (Merge Join)          | 두 테이블을 **조인 컬럼 기준 오름차순 정렬** 후, 정렬된 조인 대상 키를 스캔하는 방식                      |

### 3-1. Hash Join

**해시 조인 (Hash Join)** 은 해시 함수를 이용하는 형태의 조인이다.

* 테이블 정의

| 작은 테이블              | 큰 테이블                |
|---------------------|----------------------|
| Build Input (빌드 입력) | Probe Input (프로브 입력) |

* 해시 조인 방법
  * Build Input 테이블에서 JOIN 되는 컬럼에 **해시 함수 적용**
  * 해당 해시 함수를 통해 **해시 키 (Hash Key)** 및 **해시 테이블 (Hash Table)** 생성
  * Probe Input 테이블에서 **해당 해시 테이블에서 같은 hash key를 찾아서 JOIN 실시**

* 해시 조인의 특징
  * Nested Loop, Merge Join에 비해 **조인 속도가 빠름**
  * 해시 함수 및 해시 테이블 사용
  * **동등 조건 (=)** 필요

### 3-2. Nested Loop Join

**중첩 루프 조인 (Nested Loop Join, NL Join)** 은 JOIN 되는 두 테이블 중 **작은 테이블의 레코드를 탐색하여 JOIN** 하는 것이다.

| 작은 테이블      | 큰 테이블       |
|-------------|-------------|
| Outer Table | Inner Table |

* 중첩 루프 조인 방법
  * Outer Table 에서 **JOIN 조건을 만족시키는 row** 를 찾음
  * 해당 row를 Inner Table 에서 반복적으로 탐색

* 중첩 루프 조인의 특징
  * JOIN 대상 컬럼에 대한 **인덱스 필수**
  * 두 테이블의 **모든 조합 비교 (random access, 즉 직접 접근 방식)**
  * **작은 크기의 데이터** 에 적합 (데이터를 모두 액세스해야 하므로 큰 데이터에서는 비효율적)

### 3-3. Merge Join

**Merge Join (병합 조인, Sort Merge Join)** 은 **조인 대상 컬럼 기준 오름차순 정렬** 후, 정렬된 key 기준으로 탐색하여 JOIN 결과를 생성한다.

* Merge Join 특징
  * Nested Loop Join의 **큰 데이터에서의 비효율성** 개선
    * **양쪽 테이블을 한번에 access** 한 후, 정렬을 통해 보다 빠르게 비교 및 탐색 가능
  * 정렬 작업 비용 발생
  * 대량의 데이터에 효율적
  * **동등 조건 (=)** 필요
  * **중복 데이터 제거** 필요 (이때, 중복 제거를 위한 **임시 테이블** 비용 발생)

## 4. 참고 자료

* [MySQL 실행계획(explain) 정리 - Lifealong](https://0soo.tistory.com/235)
* [30.4.3.35 The statement_analysis and x$statement_analysis Views - MySQL Official Document](https://dev.mysql.com/doc/refman/8.4/en/sys-statement-analysis.html)
* [[Database] JOIN 기법의 종류 (Nested Loops Join, Merge Join, Hash Join) - (ง •̀_•́)ง✧](https://ryean.tistory.com/73)
