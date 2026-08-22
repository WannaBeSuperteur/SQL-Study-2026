
## 목차

* [1. CTE (Common Table Expression)](#1-cte-common-table-expression)
  * [1-1. 서브쿼리 (Subquery)](#1-1-서브쿼리-subquery) 
* [2. `WITH` 문 사용 방법](#2-with-문-사용-방법)
* [3. `WITH` 문을 이용한 실전 예제](#3-with-문을-이용한-실전-예제)
* [4. 참고 문서](#4-참고-문서)

## 1. CTE (Common Table Expression)

SQL에서 **CTE (Common Table Expression)** 는 다음과 같은 목적을 위해 사용되는, `WITH` 문을 이용해서 나타내는 도구이다.

* 복잡한 쿼리의 단순화
* SQL 중간 결과의 재사용
* 쿼리 가독성 향상

### 1-1. 서브쿼리 (Subquery)

**서브쿼리 (Subquery)** 는 다른 SQL 쿼리 내에 포함된 쿼리로, 다음과 같은 역할을 한다.

* 쿼리의 중간 결과 생성
* 이렇게 생성된 중간 결과를 **주 쿼리 (main query)** 에 활용

서브쿼리가 될 수 있는 것은 `SELECT` `INSERT` `UPDATE` `DELETE` 등 대부분의 SQL 문에서 사용될 수 있으며, 보통 `()` 안에 서브쿼리를 넣는다.

## 2. `WITH` 문 사용 방법

CTE를 정의하기 위한 `WITH` 문의 사용법은 다음과 같다.

* ```WITH name AS``` 에서 CTE를 `name`으로 정의했고, 이를 아래쪽에서 `SELECT ... FROM name ...` 으로 활용하고 있다.

```
WITH name AS (
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT ...
FROM name
...
```

## 3. `WITH` 문을 이용한 실전 예제

성과, 역량, 태도의 월별 최고점 합산 ```217.5 / 300``` 점 미만인 수습사원을 탈락시킨다고 할 때, 수습 탈락 사원의 ```mle_id``` 및 성과, 역량, 태도 점수와 합산을 출력한다.

* SQL 문

```
with probation_failed_employees as (
    select mle_id,
           max(eval_performance_score) as final_perf,
           max(eval_competency_score) as final_comp,
           max(eval_attitude_score) as final_atti,
           max(eval_performance_score) + max(eval_competency_score) + max(eval_attitude_score) as final_score
    from probation_data
    group by mle_id
    having max(eval_performance_score) + max(eval_competency_score) + max(eval_attitude_score) < 217.5
)
select mle_id,
       final_perf,
       final_comp,
       final_atti,
       round(final_score, 2) as final_score_rounded
from probation_failed_employees
```

* 실행 결과

```
mle_id       |final_perf|final_comp|final_atti|final_score_rounded|
-------------+----------+----------+----------+-------------------+
MLE-2023-0016|     68.75|     74.75|     73.18|             216.68|
MLE-2024-0042|     88.04|     66.02|     62.54|              216.6|
MLE-2024-0054|     67.36|     75.67|     69.98|             213.01|
MLE-2024-0085|     64.72|     63.11|     85.11|             212.94|
MLE-2023-0119|     66.63|      68.8|     81.59|             217.02|
MLE-2023-0125|     62.43|     71.88|     74.16|             208.47|
MLE-2023-0152|     68.23|     72.53|     64.56|             205.32|
MLE-2023-0188|     65.07|     69.98|     79.27|             214.32|
MLE-2024-0192|     71.25|     77.87|     64.63|             213.75|
MLE-2023-0214|     74.31|     67.77|     65.08|             207.16|
MLE-2023-0217|     80.18|     69.65|     65.51|             215.34|
MLE-2023-0260|     68.78|     59.49|     77.72|             205.99|
MLE-2023-0285|      76.5|     60.08|     79.43|             216.01|
MLE-2024-0304|     68.24|     61.38|     69.36|             198.98|
MLE-2024-0334|     76.14|     64.38|     67.84|             208.36|
MLE-2023-0355|     69.89|     76.27|     66.51|             212.67|
MLE-2024-0357|     89.11|     63.54|     62.08|             214.73|
MLE-2024-0360|     73.55|     73.69|     65.59|             212.83|
MLE-2023-0398|     71.59|     67.64|     70.31|             209.54|
```

다음과 같이 `max(eval_performance_score) + max(eval_competency_score) + max(eval_attitude_score)` 부분을 **별도의 CTE로 만들 수 있다.**

* 해당 부분은 `probation_failed_employees` 에서 `final_score` 로 통합되었다.
* 추가된 별도의 CTE는 `probation_result_for_each_employee` 이다.

```
with probation_result_for_each_employee as (
    select mle_id,
           max(eval_performance_score) as final_perf,
           max(eval_competency_score) as final_comp,
           max(eval_attitude_score) as final_atti,
           max(eval_performance_score) + max(eval_competency_score) + max(eval_attitude_score) as final_score
    from probation_data
    group by mle_id
),
probation_failed_employees as (
    select mle_id,
           final_perf,
           final_comp,
           final_atti,
           final_score  # 기존 max(eval_performance_score) + max(eval_competency_score) + max(eval_attitude_score)
    from probation_result_for_each_employee
    where final_score < 217.5  # 기존 max(eval_performance_score) + max(eval_competency_score) + max(eval_attitude_score)
)
select mle_id,
       final_perf,
       final_comp,
       final_atti,
       round(final_score, 2) as final_score_rounded
from probation_failed_employees
```

## 4. 참고 문서

* [05. 서브쿼리와 CTE (Common Table Expressions) - Wikidocs 나만의 분석용 DuckDB 활용 with Python (Ver 1.0)](https://wikidocs.net/256340)
