
## 목차

* [1. `EXPLAIN` 함수 설명](#1-explain-함수-설명)
  * [1-1. `EXPLAIN` 함수의 상세 해석 방법](#1-1-explain-함수의-상세-해석-방법)
* [2. 쿼리 병목 지점 확인 실습](#2-쿼리-병목-지점-확인-실습)
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

## 3. 조인 방식 이해

### 3-1. Hash Join

### 3-2. Nested Loop Join

### 3-3. Merge Join

## 4. 참고 자료

* [MySQL 실행계획(explain) 정리 - Lifealong](https://0soo.tistory.com/235)
