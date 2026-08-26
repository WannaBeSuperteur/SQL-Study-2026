
## 목차

* [1. 함수 설명](#1-함수-설명)
* [2. 대용량 테이블 스캔 비용 단축 실습](#2-대용량-테이블-스캔-비용-단축-실습)
  * [2-1. 불필요한 대용량 컬럼 호출하지 않기](#2-1-불필요한-대용량-컬럼-호출하지-않기)
  * [2-2. 인덱스 사용](#2-2-인덱스-사용)
* [3. 참고 자료](#3-참고-자료)

## 1. 함수 설명

| 함수             | 기본 구문                                                    | 함수 설명                                                                                                                                                                                                      |
|----------------|----------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `PARTITION BY` | `(집계 함수) OVER (PARTITION BY column_name [ORDER BY ...])` | 특정 기준을 이용하여 partition 으로 grouping [(참고)](../02_Aggregate_and_Feature_Engineering/02_03_set_and_remove_duplicate.md#2-1-partition-by-함수)<br>- 해당 함수 사용 시, **partition 개수 폭증으로 SQL 쿼리가 느려질 수 있음** 에 유의해야 한다. |
| `GROUP BY`     | `GROUP BY column_name`                                   | `column_name` 컬럼의 값을 기준으로 전체 row를 grouping                                                                                                                                                                 |

## 2. 대용량 테이블 스캔 비용 단축 실습

### 2-1. 불필요한 대용량 컬럼 호출하지 않기

* 최적화되지 않은 쿼리 (`select *`) 사용

```sql
select *
from probation_data
where department_team = 'LLM & Generative AI'
  and YEAR(hire_date) = 2024
  and MONTH(hire_date) in (4, 5, 6)
```

* 최적화된 쿼리 **(`mle_id` 등 필요한 컬럼만 호출)**

```sql
select mle_id,
       eval_performance_score as perf,
       eval_competency_score as comp,
       eval_attitude_score as atti
from probation_data
where department_team = 'LLM & Generative AI'
  and YEAR(hire_date) = 2024
  and MONTH(hire_date) in (4, 5, 6)
```

### 2-2. 인덱스 사용

* 최적화되지 않은 쿼리 (인덱스 미 사용)

```sql
select mle_id,
       eval_month_seq,
       eval_attitude_score as atti
from probation_data
where department_team = 'LLM & Generative AI'
  and YEAR(hire_date) = 2024
```

* 최적화된 쿼리 **(인덱스 사용)**
  * `CREATE INDEX ... ON table_name(column1, column2, ...)` 를 통해서 인덱스를 생성한다.
  * 인덱스 생성을 통해 **SELECT 연산의 성능이 향상** 된다.

```sql
create index attitude_score_index
on probation_data(department_team,
                  hire_date,
                  mle_id,
                  eval_month_seq,
                  eval_attitude_score)

select mle_id,
       eval_month_seq,
       eval_attitude_score as atti
from probation_data
where department_team = 'LLM & Generative AI'
  and hire_date >= '2024-01-01'
  and hire_date < '2025-01-01';
```

## 3. 참고 자료

* [04. 인덱스(Index) 생성 - 데이터베이스 설계 및 구축 - Wikidocs](https://wikidocs.net/328858)
* [05-1. Query 시 Partition Pruning - 데이터 엔지니어를 위한 ClickHouse - Wikidocs](https://wikidocs.net/353312)
