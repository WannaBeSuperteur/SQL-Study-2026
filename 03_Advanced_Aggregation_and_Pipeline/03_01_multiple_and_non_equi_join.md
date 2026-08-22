
## 목차

* [1. `LEFT JOIN`](#1-left-join)
* [2. `FULL OUTER JOIN`](#2-full-outer-join)
* [3. 범위 조건 기반 조인 실전 예제](#3-범위-조건-기반-조인-실전-예제)
* [4. 레이블 데이터 결합 실전 예제](#4-레이블-데이터-결합-실전-예제)

## 1. `LEFT JOIN`

**LEFT JOIN** 은 **INNER JOIN** 과 마찬가지로 SQL에서 **교집합** 을 나타내는 역할을 한다.

* 예시

```
select distinct p.mle_id,
                p.primary_framework,
                m.name,
                m.birth_date,
                m.gender
from probation_data as p
left join mle_employee_personal_info as m
on p.mle_id = m.mle_id
```

* 실행 결과

```
mle_id       |primary_framework|name|birth_date|gender|
-------------+-----------------+----+----------+------+
MLE-2023-0001|PyTorch          |김지훈 |1988-03-03|남성    |
MLE-2023-0002|JAX              |윤시우 |1998-07-09|남성    |
MLE-2023-0003|PyTorch          |송도윤 |1995-03-31|남성    |
MLE-2023-0004|TensorFlow       |이서연 |1984-02-07|여성    |
MLE-2023-0005|TensorFlow       |윤현준 |1995-07-03|남성    |
MLE-2023-0006|None             |송지호 |1998-01-22|남성    |
MLE-2023-0007|None             |윤은서 |1995-03-21|여성    |
MLE-2023-0008|TensorFlow       |김하린 |2000-01-28|여성    |
MLE-2024-0009|None             |신준혁 |1988-03-26|남성    |
```

## 2. `FULL OUTER JOIN`

**FULL OUTER JOIN** 은 **양쪽 테이블의 데이터를 모두 가져오는** 형태의 JOIN 이다.

* MySQL에서는 **해당 구문을 지원하지 않는다.**
* MySQL에서 대신 사용하려면 다음과 같이 **LEFT JOIN, RIGHT JOIN 을 결합** 해야 한다.

```
select distinct coalesce(p.mle_id, m.mle_id) as mle_id,
       p.primary_framework,
       m.name,
       m.birth_date,
       m.gender
from probation_data as p
left join mle_employee_personal_info as m
on p.mle_id = m.mle_id

union

select coalesce(p.mle_id, m.mle_id) as mle_id,
       p.primary_framework,
       m.name,
       m.birth_date,
       m.gender
from probation_data as p
right join mle_employee_personal_info as m
on p.mle_id = m.mle_id;
```

* 실행 결과

```
mle_id       |primary_framework|name|birth_date|gender|
-------------+-----------------+----+----------+------+
MLE-2023-0001|PyTorch          |김지훈 |1988-03-03|남성    |
MLE-2023-0002|JAX              |윤시우 |1998-07-09|남성    |
MLE-2023-0003|PyTorch          |송도윤 |1995-03-31|남성    |
MLE-2023-0004|TensorFlow       |이서연 |1984-02-07|여성    |
MLE-2023-0005|TensorFlow       |윤현준 |1995-07-03|남성    |
MLE-2023-0006|None             |송지호 |1998-01-22|남성    |
MLE-2023-0007|None             |윤은서 |1995-03-21|여성    |
MLE-2023-0008|TensorFlow       |김하린 |2000-01-28|여성    |
MLE-2024-0009|None             |신준혁 |1988-03-26|남성    |
```

## 3. 범위 조건 기반 조인 실전 예제

* 목표
  * 각 직원들의 **평가 점수를 동적으로 mapping** 시킨다.
  * 성과, 역량, 태도 각각 **S(85-100), A(70-85), B(60-70), F(0-60)** 점수를 매겨서 등급화한다.

* 예제 SQL

```
with grade_tier as (
	select 'A' as grade, 85.0 as min_score, 100.1 as max_score union all
	select 'B' as grade, 70.0 as min_score, 85.0 as max_score union all
	select 'C' as grade, 60.0 as min_score, 70.0 as max_score union all
	select 'F' as grade, 0.0 as min_score, 60.0 as max_score
)
select e.mle_id,
       e.eval_month_seq,
       e.eval_performance_score as perf_score,
       gp.grade as perf_grade,
       e.eval_competency_score as comp_score,
       gc.grade as comp_grade,
       e.eval_attitude_score as atti_score,
       ga.grade as atti_grade
from probation_data as e
join grade_tier as gp
     on e.eval_performance_score >= gp.min_score
    and e.eval_performance_score < gp.max_score
join grade_tier as gc
     on e.eval_competency_score >= gc.min_score
    and e.eval_competency_score < gc.max_score
join grade_tier as ga
     on e.eval_attitude_score >= ga.min_score
    and e.eval_attitude_score < ga.max_score
order by substring(mle_id, 10, 4), eval_month_seq
```

* 실행 결과

```
mle_id       |eval_month_seq|perf_score|perf_grade|comp_score|comp_grade|atti_score|atti_grade|
-------------+--------------+----------+----------+----------+----------+----------+----------+
MLE-2023-0001|             1|     86.89|A         |     88.66|A         |     80.41|B         |
MLE-2023-0001|             2|     88.44|A         |      95.4|A         |      87.3|A         |
MLE-2023-0001|             3|     90.68|A         |     97.58|A         |      79.1|B         |
MLE-2023-0002|             1|     70.68|B         |     75.52|B         |     93.45|A         |
MLE-2023-0002|             2|     73.81|B         |     77.22|B         |     90.35|A         |
MLE-2023-0002|             3|     76.44|B         |     78.27|B         |     91.27|A         |
MLE-2023-0003|             1|     91.48|A         |     67.29|C         |     71.94|B         |
MLE-2023-0003|             2|     91.27|A         |     71.82|B         |     65.61|C         |
MLE-2023-0003|             3|     88.63|A         |     74.79|B         |      65.1|C         |
MLE-2023-0004|             1|     61.54|C         |     57.59|F         |     77.35|B         |
MLE-2023-0004|             2|      70.6|B         |      67.4|C         |      80.3|B         |
MLE-2023-0004|             2|      70.6|B         |      67.4|C         |      80.3|B         |
MLE-2023-0004|             3|     69.41|C         |     66.63|C         |     83.75|B         |
MLE-2023-0005|             1|     71.34|B         |     100.0|A         |     85.24|A         |
MLE-2023-0005|             2|     66.74|C         |     100.0|A         |     94.89|A         |
MLE-2023-0005|             3|     72.72|B         |     100.0|A         |     85.15|A         |
```

* 참고
  * ```UNION ALL``` 은 **```SELECT``` query 결과를 concatenate** 하기 위해 사용한다. 

## 4. 레이블 데이터 결합 실전 예제

* 목표
  * 수습 1~2개월차의 각종 행동지표를 이용하여, 3개월차 **최종 성과 점수** 및 **수습 통과/연장 여부** 를 예측한다.
  * ```mle_id``` 를 기준으로 feature 와 label 을 안전하게 합성한다.
  * 최종적으로 **알 수 없는 미래의 데이터가 섞이는 data leakage 를 방지** 한다.

* 참고
  * **feature 에 해당하는 부분** 을 ```SELECT``` 문을 이용하여 테이블처럼 만든다.
  * **label 에 해당하는 부분** 도 마찬가지로 테이블처럼 만든다.
  * 이 2개의 테이블을 서로 JOIN 한다.

* 예제 SQL

```
with first_month_performance as (
    select mle_id,
           code_commits_cnt as commits,
           prs_merged_cnt as merged_prs,
           eval_performance_score as perf,
           eval_competency_score as comp,
           eval_attitude_score as atti
    from sys.probation_data
    where eval_month_seq = 1
),

second_month_performance as (
    select mle_id,
           code_commits_cnt as commits,
           prs_merged_cnt as merged_prs,
           eval_performance_score as perf,
           eval_competency_score as comp,
           eval_attitude_score as atti
    from sys.probation_data
    where eval_month_seq = 2
),

final_passed as (
    select mle_id,
           probation_status
    from sys.probation_data
    where eval_month_seq = 3
)

select COALESCE(m1.mle_id, m2.mle_id, label.mle_id) as mle_id,
       m1.commits as commits_m1,
       m2.commits as commits_m2,
       m1.merged_prs as merged_prs_m1,
       m2.merged_prs as merged_prs_m2,
       m1.perf as perf_m1,
       m2.perf as perf_m2,
       m1.comp as comp_m1,
       m2.comp as comp_m2,
       m1.atti as atti_m1,
       m2.atti as atti_m2,
       label.probation_status as probation_status
from final_passed as label
join first_month_performance as m1 on label.mle_id = m1.mle_id
join second_month_performance as m2 on label.mle_id = m2.mle_id;
```

* 실행 결과

```
+---------------+------------+------------+---------------+---------------+---------+---------+---------+---------+---------+---------+------------------+
| mle_id        | commits_m1 | commits_m2 | merged_prs_m1 | merged_prs_m2 | perf_m1 | perf_m2 | comp_m1 | comp_m2 | atti_m1 | atti_m2 | probation_status |
+---------------+------------+------------+---------------+---------------+---------+---------+---------+---------+---------+---------+------------------+
| MLE-2023-0001 |         43 |         33 |            18 |             8 |   86.89 |   88.44 |   88.66 |    95.4 |   80.41 |    87.3 | Passed           |
| MLE-2023-0002 |         86 |         47 |            11 |            24 |   70.68 |   73.81 |   75.52 |   77.22 |   93.45 |   90.35 | Passed           |
| MLE-2023-0003 |         73 |         45 |             4 |            10 |   91.48 |   91.27 |   67.29 |   71.82 |   71.94 |   65.61 | Extended         |
| MLE-2023-0004 |         43 |         50 |             4 |            14 |   61.54 |    70.6 |   57.59 |    67.4 |   77.35 |    80.3 | Extended         |
| MLE-2023-0005 |         45 |         69 |             9 |            14 |   71.34 |   66.74 |     100 |     100 |   85.24 |   94.89 | Passed           |
| MLE-2023-0004 |         43 |         50 |             4 |            14 |   61.54 |    70.6 |   57.59 |    67.4 |   77.35 |    80.3 | Extended         |
| MLE-2023-0006 |         48 |         92 |            16 |             9 |   98.93 |   93.47 |   67.86 |   66.16 |   79.18 |   79.89 | Passed           |
| MLE-2023-0007 |         26 |         76 |            17 |            23 |   87.39 |   82.93 |   74.74 |   79.71 |   88.93 |   96.63 | Passed           |
| MLE-2023-0008 |         46 |         64 |            12 |            15 |   93.71 |   91.61 |   71.85 |    75.3 |   87.25 |   78.28 | Passed           |
| MLE-2024-0009 |         55 |         73 |             7 |            18 |   72.52 |   75.82 |   61.48 |   65.57 |    83.9 |   80.89 | Extended         |
| MLE-2023-0010 |         88 |         74 |            10 |            13 |   80.11 |   86.71 |   89.64 |   94.48 |   80.65 |    71.4 | Passed           |

...
```
