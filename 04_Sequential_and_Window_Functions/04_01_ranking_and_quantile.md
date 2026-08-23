
## 목차

* [1. 함수 설명](#1-함수-설명)
* [2. 함수 실행 예시](#2-함수-실행-예시)
  * [2-1. `ROW_NUMBER`](#2-1-rownumber)
  * [2-2. `DENSE_RANK`](#2-2-denserank)
  * [2-3. `NTILE`](#2-3-ntile)

## 1. 함수 설명

| 함수           | 함수 형식                                     | 함수 설명                                                   |
|--------------|-------------------------------------------|---------------------------------------------------------|
| `ROW_NUMBER` | `SELECT ROW_NUMBER() OVER (ORDER BY ...)` | 정렬 순서 (`ORDER BY` 로 정렬 가능) 에 의한 **그룹 내 번호/순위** 함수       |
| `DENSE_RANK` | `SELECT DENSE_RANK() OVER (ORDER BY ...)` | **중복 값에 대해 동일 순위를 매기는** 함수 (중복 값 개수와 **관계없이 1씩** 순위 증가) |
| `NTILE`      | `SELECT NTILE(N) OVER (ORDER BY ...)`     | 전체 데이터를 **순위에 따라 `N`등분** 하여, 각 row 별로 속한 등분의 값을 반환하는 함수 |

## 2. 함수 실행 예시

### 2-1. `ROW_NUMBER`

**[ 1. 단순 예시 ]**

* 목표
  * 각 직원의 성과, 역량, 태도 합산 점수의 순위를 최상위권부터 보여준다.
  * 이때, **3개월 통합이 아닌, 각 월별 조합, 즉 ```(mle_id, eval_month_seq)``` 조합** 으로 보여준다.

* SQL 쿼리

```sql
with probation_score_sum as (
    select mle_id,
           eval_month_seq,
           eval_performance_score as performance,
           eval_competency_score as competency,
           eval_attitude_score as attitude,
           eval_performance_score + eval_competency_score + eval_attitude_score as score_sum
    from probation_data
)
select row_number() over (order by score_sum desc) as score_rank,
       mle_id,
       eval_month_seq,
       round(score_sum, 2) as score_sum  # 점수 합계가 245.970000003 과 같이 나오는 현상 보정
from probation_score_sum
```

* 실행 결과

```
score_rank|mle_id       |eval_month_seq|score_sum|
----------+-------------+--------------+---------+
         1|MLE-2023-0289|             3|   292.49|
         2|MLE-2023-0235|             2|   290.73|
         3|MLE-2023-0289|             1|   289.97|
         4|MLE-2024-0399|             3|   288.19|
         5|MLE-2024-0351|             1|   287.97|
         6|MLE-2023-0390|             3|   287.75|
         7|MLE-2023-0073|             3|   286.66|
         8|MLE-2024-0351|             3|   286.56|
         9|MLE-2024-0133|             3|   285.13|
        10|MLE-2024-0247|             3|   284.87|
        11|MLE-2023-0022|             3|   284.71|
        12|MLE-2024-0079|             2|   284.38|
        13|MLE-2024-0317|             2|   283.85|
        14|MLE-2024-0261|             3|   283.74|
        15|MLE-2023-0381|             3|   283.43|
```

**[ 2. 이름, 성별, 생년월일 같이 추출 ]**

* 목표
  * 위 **[ 1. 단순 예시 ]** 에서, 해당 직원의 이름과 성별, 생년월일을 같이 추출한다.

* SQL 쿼리

```sql
with probation_score_sum as (
    select mle_id,
           eval_month_seq,
           eval_performance_score as performance,
           eval_competency_score as competency,
           eval_attitude_score as attitude,
           eval_performance_score + eval_competency_score + eval_attitude_score as score_sum
    from probation_data
)
select row_number() over (order by pss.score_sum desc) as score_rank,
       pss.mle_id,
       mepi.name,
       mepi.gender,
       mepi.birth_date,
       pss.eval_month_seq,
       round(pss.score_sum, 2) as score_sum  # 점수 합계가 245.970000003 과 같이 나오는 현상 보정
from probation_score_sum as pss
left join mle_employee_personal_info as mepi
       on pss.mle_id = mepi.mle_id
```

* 실행 결과

```
score_rank|mle_id       |name|gender|birth_date|eval_month_seq|score_sum|
----------+-------------+----+------+----------+--------------+---------+
         1|MLE-2023-0289|박민재 |남성    |1999-07-13|             3|   292.49|
         2|MLE-2023-0235|임예원 |여성    |1995-12-06|             2|   290.73|
         3|MLE-2023-0289|박민재 |남성    |1999-07-13|             1|   289.97|
         4|MLE-2024-0399|황지원 |여성    |1991-12-01|             3|   288.19|
         5|MLE-2024-0351|전지호 |남성    |2000-03-27|             1|   287.97|
         6|MLE-2023-0390|한수아 |여성    |1982-02-20|             3|   287.75|
         7|MLE-2023-0073|조민주 |여성    |1990-12-30|             3|   286.66|
         8|MLE-2024-0351|전지호 |남성    |2000-03-27|             3|   286.56|
         9|MLE-2024-0133|한지아 |여성    |1997-08-21|             3|   285.13|
        10|MLE-2024-0247|홍채은 |여성    |2000-06-27|             3|   284.87|
        11|MLE-2023-0022|강현우 |남성    |1990-07-06|             3|   284.71|
        12|MLE-2024-0079|한서준 |남성    |1983-02-15|             2|   284.38|
        13|MLE-2024-0317|정소율 |여성    |1984-03-06|             2|   283.85|
        14|MLE-2024-0261|한서현 |여성    |1999-04-30|             3|   283.74|
        15|MLE-2023-0381|최다은 |여성    |1989-12-06|             3|   283.43|
```

### 2-2. `DENSE_RANK`

* 목표
  * 각 직원의 성과, 역량, 태도 합산 점수를 **가장 가까운 정수로 반올림한 점수** 의 순위를 최상위권부터 보여준다.
  * 이때, **3개월 통합이 아닌, 각 월별 조합, 즉 ```(mle_id, eval_month_seq)``` 조합** 으로 보여준다.

* SQL 쿼리

```
with probation_score_sum as (
    select mle_id,
           eval_month_seq,
           eval_performance_score as performance,
           eval_competency_score as competency,
           eval_attitude_score as attitude,
           eval_performance_score + eval_competency_score + eval_attitude_score as score_sum
    from probation_data
)
select dense_rank() over (order by round(score_sum, 0) desc) as dense_ranking,
       mle_id,
       eval_month_seq,
       round(score_sum, 2) as score_sum_r2,
       round(score_sum, 0) as score_sum_r0
from probation_score_sum
```

* 실행 결과

```
dense_ranking|mle_id       |eval_month_seq|score_sum_r2|score_sum_r0|
-------------+-------------+--------------+------------+------------+
            1|MLE-2023-0289|             3|      292.49|       292.0|
            2|MLE-2023-0235|             2|      290.73|       291.0|
            3|MLE-2023-0289|             1|      289.97|       290.0|
            4|MLE-2024-0351|             1|      287.97|       288.0|
            4|MLE-2023-0390|             3|      287.75|       288.0|
            4|MLE-2024-0399|             3|      288.19|       288.0|
            5|MLE-2023-0073|             3|      286.66|       287.0|
            5|MLE-2024-0351|             3|      286.56|       287.0|
            6|MLE-2023-0022|             3|      284.71|       285.0|
            6|MLE-2024-0133|             3|      285.13|       285.0|
            6|MLE-2024-0247|             3|      284.87|       285.0|
            7|MLE-2024-0079|             2|      284.38|       284.0|
            7|MLE-2024-0261|             3|      283.74|       284.0|
            7|MLE-2024-0317|             2|      283.85|       284.0|
```

### 2-3. `NTILE`

* 목표
  * 각 직원의 성과, 역량, 태도 합산 점수의 순위를 최상위권부터 보여주고, **상위 1%는 1, 상위 1-2%는 2, ... 하위 1%는 100** 과 같이 표시한다.
  * 이때, **각 직원의 점수를 3개월 통합 평균** 으로 보여준다.
  * 성과, 역량, 태도 합산 점수의 `상위 4%, 11%, 23%, 40%, 60%, 77%, 89%, 96%` 의 등급컷을 보여준다.

* SQL 쿼리 (`NTILE` 결과 추출)

```
with probation_score as (
    select mle_id,
           avg(eval_performance_score) + avg(eval_competency_score) + avg(eval_attitude_score) as score
    from probation_data
    group by mle_id
)
select mle_id,
	round(score, 2) as score,
	ntile(100) over (order by score desc) as top_percent
from probation_score
```

* 실행 결과 (`NTILE` 결과 추출)

```
mle_id       |score |top_percent|
-------------+------+-----------+
MLE-2023-0289|287.16|          1|
MLE-2023-0235|285.66|          1|
MLE-2024-0351|283.89|          1|
MLE-2023-0073|281.36|          1|
MLE-2023-0228|281.03|          2|
MLE-2023-0381|280.33|          2|
MLE-2024-0261|280.13|          2|
MLE-2023-0390|280.02|          2|
MLE-2024-0399|279.41|          3|
MLE-2024-0133|279.33|          3|

...

MLE-2023-0152|200.37|         99|
MLE-2023-0214|198.86|         99|
MLE-2023-0260|197.79|        100|
MLE-2023-0398|197.07|        100|
MLE-2024-0334|195.56|        100|
MLE-2024-0304|188.73|        100|
```

* SQL 쿼리 (등급컷 추출)

```sql
with probation_score as (
    select mle_id,
           avg(eval_performance_score) + avg(eval_competency_score) + avg(eval_attitude_score) as score
    from probation_data
    group by mle_id
),
probation_score_percent as (
	select mle_id,
		round(score, 2) as score,
		ntile(100) over (order by score desc) as top_percent
	from probation_score
),
percentage(cutoff_pct) as (
    values row(4), row(11), row(23), row(40), row(60), row(77), row(89), row(96)
)
select pct.cutoff_pct, min(psp.score) as cutoff_score
from probation_score_percent as psp
join percentage as pct
  on psp.top_percent <= pct.cutoff_pct
group by pct.cutoff_pct  # psp.score 의 최솟값을 probation_score_percent 에서 구하므로 GROUP BY 필요
order by pct.cutoff_pct;
```

* 실행 결과 (등급컷 추출)

```
cutoff_pct|cutoff_score|
----------+------------+
         4|      275.05|
        11|      266.63|
        23|      257.33|
        40|      246.83|
        60|      236.32|
        77|      225.16|
        89|      216.57|
        96|      207.29|
```
