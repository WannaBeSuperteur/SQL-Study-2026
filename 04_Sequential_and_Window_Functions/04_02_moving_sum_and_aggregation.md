
## 목차

* [1. 함수 설명](#1-함수-설명)
  * [1-1. `OVER (PARTITION BY ... ORDER BY ...)`](#1-1-over-partition-by--order-by-)
  * [1-2. `OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN ...)`](#1-2-over-partition-by--order-by--rows-between-)
* [2. 기본  `OVER (PARTITION BY ... ORDER BY ...)` 예제](#2-기본-over-partition-by--order-by--예제) 
* [3. Rolling Feature 생성 예제](#3-rolling-feature-생성-예제)

## 1. 함수 설명

### 1-1. `OVER (PARTITION BY ... ORDER BY ...)`

* 참고: [`PARTITION BY` 함수](../02_Aggregate_and_Feature_Engineering/02_03_set_and_remove_duplicate.md#2-1-partition-by-함수)

`OVER (PARTITION BY column_1 ORDER BY column_2)` 는 다음과 같은 의미를 갖는다.

* `column_1` 을 기준으로 row 들의 파티션을 나눈다.
  * `RANK()` 등 순위 함수를 사용할 때, **각 파티션 별로 순위가 매겨진다.** 
* `column_2` 를 기준으로 row 들이 정렬된다.

### 1-2. `OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN ...)`

`OVER (PARTITION BY column_1 ORDER BY column_2 ROWS BETWEEN ...)` 는 **집계 함수를 이용하여, 현재 행과 전후 행을 포함하여 집계** 한다.

* 기본 구문

```sql
집계함수 OVER (
    PARTITION BY column_1
    ORDER BY column_2
    ROWS BETWEEN ... AND ...
)
```

* `ROWS BETWEEN` 에 정의된 키워드에 따라, **현재 row의 앞뒤 row를 포함하여 집계** 한다.

| 키워드           | `ROWS BETWEEN` 과 함께 사용 시 예시                         | 설명                                               |
|---------------|-----------------------------------------------------|--------------------------------------------------|
| `N PRECEDING` | `ROWS BETWEEN UNBOUNDED PRECEIDING AND 1 PRECEDING` | 현재 행의 `N`개 행만큼 앞의 행 (`N` = `UNBOUNDED` 이면 첫 행)   |
| `N FOLLOWING` | `ROWS BETWEEN 1 FOLLOWING AND UNBOUNDED FOLLOWING`  | 현재 행의 `N`개 행만큼 뒤쪽 행 (`N` = `UNBOUNDED` 이면 맨 끝 행) |
| `CURRENT ROW` | `ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING`          | 현재 행                                             |

## 2. 기본 `OVER (PARTITION BY ... ORDER BY ...)` 예제

* 목표
  * `PyTorch`를 주 프레임워크로 사용하는 머신러닝 엔지니어들을 `candidate_source` (입사 경로) 별 파티션을 나눈다.
  * 이렇게 나눈 각 파티션 별로 **성과, 역량, 태도 점수의 합산 점수 및 그 순위** 를 표시한다. (최상위 점수부터)

* 예제 SQL

```sql
with avg_probation_score as (
    select mle_id,
           avg(eval_performance_score) as performance,
           avg(eval_competency_score) as competency,
           avg(eval_attitude_score) as attitude,
           avg(eval_performance_score) + avg(eval_competency_score) + avg(eval_attitude_score) as total
    from probation_data
    group by mle_id
)
select distinct coalesce (aps.mle_id, pd.mle_id) as mle_id,
       round(aps.performance, 2) as performance,
       round(aps.competency, 2) as competency,
       round(aps.attitude, 2) as attitude,
       round(aps.total, 2) as total_score,
       pd.candidate_source,
       dense_rank() over (partition by pd.candidate_source order by aps.total desc) as ranking
from probation_data as pd
join avg_probation_score as aps on pd.mle_id = aps.mle_id
where pd.primary_framework = 'PyTorch'
```

* 실행 결과

```
mle_id       |performance|competency|attitude|total_score|candidate_source  |ranking|
-------------+-----------+----------+--------+-----------+------------------+-------+
MLE-2023-0289|      94.03|     100.0|   93.13|     287.16|Agency            |      1|
MLE-2024-0317|      91.51|     91.54|   93.09|     276.14|Agency            |      2|
MLE-2023-0173|      86.05|     96.96|   92.04|     275.05|Agency            |      3|

...

MLE-2023-0076|      77.12|     62.46|   78.86|     218.45|Agency            |     24|
MLE-2024-0075|      63.77|     74.21|   79.07|     217.05|Agency            |     25|
MLE-2023-0373|      79.62|     66.92|   62.39|     208.92|Agency            |     26|
MLE-2023-0143|      81.34|     97.72|   75.06|     254.13|Campus Recruitment|      1|
MLE-2024-0088|      94.13|     97.77|   61.61|     253.51|Campus Recruitment|      2|
MLE-2023-0333|       68.2|     95.77|   89.08|     253.05|Campus Recruitment|      3|

...
```

## 3. Rolling Feature 생성 예제

