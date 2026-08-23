
## 목차

* [1. 함수 설명](#1-함수-설명)
  * [1-1. `LAG` 함수](#1-1-lag-함수) 
  * [1-2. `LEAD` 함수](#1-2-lead-함수)
* [2. 실전 예제](#2-실전-예제)
  * [2-1. 이전/이후 시점 이벤트 추적](#2-1-이전이후-시점-이벤트-추적)
  * [2-2. 신규 시계열 feature 추가](#2-2-신규-시계열-feature-추가)

## 1. 함수 설명

`LAG` 함수와 `LEAD` 함수는 **특정 행을 기준으로 이전/다음 행의 값을 반환** 하는 함수이다.

### 1-1. `LAG` 함수

`LAG` 함수는 **직전 행의 값을 추출** 하는 함수이다.

* 구문

```sql
LAG(column_name) OVER (PARTITION BY ... ORDER BY ...)
```

### 1-2. `LEAD` 함수

`LEAD` 함수는 **직후 행의 값을 추출** 하는 함수이다.

* 구문

```sql
LEAD(column_name) OVER (PARTITION BY ... ORDER BY ...)
```

## 2. 실전 예제

### 2-1. 이전/이후 시점 이벤트 추적

* 목표
  * 모든 머신러닝 엔지니어에 대해, **2개월차 평가 점수의 1개월차 대비 향상도** 및 **3개월차 최종 평가 점수의 2개월차 대비 향상도** 를 기록한다.
  * 이때, `LAG`, `LEAD` 함수를 사용한다.

* SQL 구문

```sql
with probation_total_score as (
    select distinct mle_id,
           eval_month_seq,
           eval_performance_score + eval_competency_score + eval_attitude_score as total_score
    from probation_data
)
select coalesce(pd.mle_id, pts.mle_id) as mle_id,
       coalesce(pd.eval_month_seq, pts.eval_month_seq) as eval_month_seq,
       round(pts.total_score, 2) as total_score,
       round(lag(pts.total_score) over (partition by pts.mle_id order by pts.eval_month_seq), 2) as prev_total_score,
       round(pts.total_score - (lag(pts.total_score) over (partition by pts.mle_id order by pts.eval_month_seq)), 2) as improvement
from probation_data as pd
join probation_total_score as pts
   on pts.mle_id = pd.mle_id
  and pts.eval_month_seq = pd.eval_month_seq 
```

* 실행 결과

```
mle_id       |eval_month_seq|total_score|prev_total_score|improvement|
-------------+--------------+-----------+----------------+-----------+
MLE-2023-0001|             1|     255.96|                |           |
MLE-2023-0001|             2|     271.14|          255.96|      15.18|
MLE-2023-0001|             3|     267.36|          271.14|      -3.78|
MLE-2023-0002|             1|     239.65|                |           |
MLE-2023-0002|             2|     241.38|          239.65|       1.73|
MLE-2023-0002|             3|     245.98|          241.38|        4.6|
MLE-2023-0003|             1|     230.71|                |           |
MLE-2023-0003|             2|      228.7|          230.71|      -2.01|
MLE-2023-0003|             3|     228.52|           228.7|      -0.18|
```

### 2-2. 신규 시계열 feature 추가

* 목표
  * 모든 머신러닝 엔지니어들에 대해 기록된 수습 평가 점수 (성과, 역량, 태도 각각) 의 **월별 평균 (즉, 모든 직원의 평가 기록의 월별 평균)** 을 구한다.
  * 이 월별 평균의 **직전 월별 평균 대비 증가/감소량** 을 구한다.

* 예제 SQL

```sql
with monthly_avg_score as (
    select year(eval_date) as eval_year,
        month(eval_date) as eval_month,
        avg(eval_performance_score) as avg_performance,
        avg(eval_competency_score) as avg_competency,
        avg(eval_attitude_score) as avg_attitude
	from probation_data
	group by year(eval_date), month(eval_date)
	order by year(eval_date), month(eval_date)
)
select eval_year,
       eval_month,
       round(avg_performance, 2) as perf_cur_month,
       round(avg_competency, 2) as comp_cur_month,
       round(avg_attitude, 2) as atti_cur_month,
       round(avg_performance - lag(avg_performance) over (order by eval_year, eval_month), 2) as perf_improve,
       round(avg_competency - lag(avg_competency) over (order by eval_year, eval_month), 2) as comp_improve,
       round(avg_attitude - lag(avg_attitude) over (order by eval_year, eval_month), 2) as atti_improve
from monthly_avg_score
```

* 실행 결과

```
eval_year|eval_month|perf_cur_month|comp_cur_month|atti_cur_month|perf_improve|comp_improve|atti_improve|
---------+----------+--------------+--------------+--------------+------------+------------+------------+
     2023|         1|         87.56|         86.27|         62.27|            |            |            |
     2023|         2|         78.93|         78.72|         84.69|       -8.63|       -7.55|       22.42|
     2023|         3|         84.74|         80.53|         79.65|         5.8|        1.81|       -5.04|
     2023|         4|         83.91|         82.64|         76.63|       -0.82|        2.11|       -3.02|
     2023|         5|         85.26|         83.66|         76.71|        1.34|        1.02|        0.08|
     2023|         6|          83.1|         81.22|         78.15|       -2.15|       -2.43|        1.44|
     2023|         7|          84.1|         79.74|         78.76|        0.99|       -1.48|        0.61|
     2023|         8|         82.65|         78.55|         78.56|       -1.45|       -1.19|        -0.2|
     2023|         9|         82.26|          79.3|         77.16|       -0.39|        0.74|        -1.4|
     2023|        10|         81.39|         79.71|         75.95|       -0.87|        0.41|       -1.21|
     2023|        11|         80.73|          80.1|         75.93|       -0.67|        0.39|       -0.02|
     2023|        12|         81.33|         81.22|         77.97|        0.61|        1.12|        2.04|
     2024|         1|         83.18|         80.45|         78.33|        1.84|       -0.77|        0.36|
     2024|         2|         83.74|         81.24|         77.03|        0.56|        0.79|        -1.3|
     2024|         3|         83.46|         80.67|         77.37|       -0.28|       -0.57|        0.34|
     2024|         4|         82.12|         79.64|         78.55|       -1.34|       -1.03|        1.18|
     2024|         5|         82.04|         79.87|          77.6|       -0.08|        0.23|       -0.95|
     2024|         6|         82.55|          80.4|         78.25|        0.51|        0.53|        0.65|
     2024|         7|         82.92|         80.65|         77.45|        0.37|        0.25|       -0.79|
     2024|         8|         84.94|         80.39|         81.28|        2.02|       -0.26|        3.83|
```
