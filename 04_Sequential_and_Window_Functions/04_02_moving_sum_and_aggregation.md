
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

* 목표
  * 모든 머신러닝 엔지니어들에 대해 기록된 수습 평가 점수 (성과, 역량, 태도 각각) 의 **월별 평균 (즉, 모든 직원의 평가 기록의 월별 평균)** 을 구한다.
  * 이 월별 평균을 이용하여 **해당월을 포함한 최근 6개월 간의 평균** (평가 실시 건수를 고려하지 않은 단순 산술 평균) 을 구한다.

* 예제 SQL

```
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
       avg(avg_performance) over (order by eval_year, eval_month rows between 5 preceding and current row) as perf_6m,
       avg(avg_competency) over (order by eval_year, eval_month rows between 5 preceding and current row) as comp_6m,
       avg(avg_attitude) over (order by eval_year, eval_month rows between 5 preceding and current row) as atti_6m
from monthly_avg_score
```

* 실행 결과

```
eval_year|eval_month|perf_cur_month|comp_cur_month|atti_cur_month|perf_6m          |comp_6m          |atti_6m          |
---------+----------+--------------+--------------+--------------+-----------------+-----------------+-----------------+
     2023|         1|         87.56|         86.27|         62.27|            87.56|            86.27|            62.27|
     2023|         2|         78.93|         78.72|         84.69|          83.2475|82.49527777777777|            73.48|
     2023|         3|         84.74|         80.53|         79.65|83.74421568627452|81.83933551198255|75.53784313725491|
     2023|         4|         83.91|         82.64|         76.63|83.78670072574485| 82.0389172184025|75.81101222307105|
     2023|         5|         85.26|         83.66|         76.71|84.08100615021613| 82.3630831418106|75.99154395567203|
     2023|         6|          83.1|         81.22|         78.15|83.91823571878741|82.17336837160015|76.35130946077692|
     2023|         7|          84.1|         79.74|         78.76|83.34119624510319|81.08578065230192|79.09968665375938|
     2023|         8|         82.65|         78.55|         78.56|83.96034439325135|81.05768805970933|78.07724220931493|
     2023|         9|         82.26|          79.3|         77.16|83.54806586738316|80.85241008506567| 77.6610917250248|
     2023|        10|         81.39|         79.71|         75.95|83.12780005595882|80.36420213447967|77.54714335632035|
     2023|        11|         80.73|          80.1|         75.93|82.37246739528494|79.77062114016668|77.41616922313379|
     2023|        12|         81.33|         81.22|         77.97|82.07726191583288|79.77034716756395|77.38632904048538|
     2024|         1|         83.18|         80.45|         78.33|81.92365853237423|  79.888458696386| 77.3141423236934|
     2024|         2|         83.74|         81.24|         77.03|82.10531767589275|  80.337161821386|77.06010239313784|
     2024|         3|         83.46|         80.67|         77.37|82.30396271759803|80.56689027365663|77.09551079597998|
     2024|         4|         82.12|         79.64|         78.55|82.42525719980516|80.55608892752842|77.52893542612792|
     2024|         5|         82.04|         79.87|          77.6|82.64434675759038| 80.5180198082419|77.80770800137874|
     2024|         6|         82.55|          80.4|         78.25|82.84698256297536|80.38039699293724|77.85370351389646|
     2024|         7|         82.92|         80.65|         77.45| 82.8040960083535|80.41330455596244|77.70824833182364|
     2024|         8|         84.94|         80.39|         81.28|83.00398316113127|80.27037920874022|78.41639937349031|
```
