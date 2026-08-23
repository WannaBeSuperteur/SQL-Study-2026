## 목차

* [1. 목표](#1-목표)
* [2. SQL 쿼리문](#2-sql-쿼리문)
* [3. 실행 결과](#3-실행-결과)
  * [3-1. 전체 직원 평가 점수](#3-1-전체-직원-평가-점수)
  * [3-2. 등급 cutoff](#3-2-등급-cutoff)
  * [3-3. 원점수-표준점수 순위 변동](#3-3-원점수-표준점수-순위-변동)

## 1. 목표

* 모든 직원의 성과, 역량, 태도 및 이들의 합산 점수를 월별로 평균한다.
* 성과, 역량, 태도 점수를 `평균 100, 표준편차 20`인 정규분포로 하여 각각 표준점수를 구한다.
* 영역별 표준점수 및 그 합산을 기준으로 `상위 4%, 11%, 23%, 40%, 60%, 77%, 89%, 96%` 점수를 각각 cutoff 로 하여,
  * **성과, 역량, 태도 및 합산 점수** 에 대한 **1~8 등급컷** 을 산출한다.
  * 해당 결과로 **각 직원의 성과, 역량, 태도, 합산 점수에 대한 등급을 산출** 한다.
  * 각 직원의 성과, 역량, 태도 점수의 백분위를 구한다.
* 미리 정의된 **직원 분류 기준표** 에 따라 직원을 분류한 결과 column을 각 직원에 대해 추가한다.
* **직원 분류 기준표**의 각 기준에 속하는 직원의 인원수 비율 정보 및 해당 직원의 리스트를 각각 테이블 형태로 저장한다.
* 원점수 기반 순위와 표준점수 합산 기반 순위의 변동을 찾는다.

## 2. SQL 쿼리문

```sql
# 1. 모든 직원의 성과, 역량, 태도 및 이들의 합산 점수를 월별로 평균한다.
with avg_score as (
    select mle_id,
           avg(eval_performance_score) as avg_performance,
           avg(eval_competency_score) as avg_competency,
           avg(eval_attitude_score) as avg_attitude,
           avg(eval_performance_score) + avg(eval_competency_score) + avg(eval_attitude_score) as avg_total
	from probation_data
	group by mle_id
),
# 2. 성과, 역량, 태도 점수를 `평균 100, 표준편차 20`인 정규분포로 하여 각각 표준점수를 구한다.
mean_and_stds as (
    select avg(avg_performance) as entire_avg_perf,
           avg(avg_competency) as entire_avg_comp,
           avg(avg_attitude) as entire_avg_atti,
           std(avg_performance) as entire_std_perf,
           std(avg_competency) as entire_std_comp,
           std(avg_attitude) as entire_std_atti
    from avg_score
),
normalized_scores as (
    select a.mle_id,
           round(100.0 + 20.0 * (a.avg_performance - mas.entire_avg_perf) / mas.entire_std_perf, 0) as norm_perf,
           round(100.0 + 20.0 * (a.avg_competency - mas.entire_avg_comp) / mas.entire_std_comp, 0) as norm_comp,
           round(100.0 + 20.0 * (a.avg_attitude - mas.entire_avg_atti) / mas.entire_std_atti, 0) as norm_atti
    from avg_score as a
    join mean_and_stds as mas
),
normalized_scores_int as (
    select mle_id,
           cast(norm_perf as signed) as norm_perf,
           cast(norm_comp as signed) as norm_comp,
           cast(norm_atti as signed) as norm_atti,
           cast(norm_perf + norm_comp + norm_atti as signed) as norm_total
    from normalized_scores
),
# 3-3. 각 직원의 성과, 역량, 태도 점수의 백분위를 구한다.
normalized_scores_int_with_100 as (
    select mle_id,
           norm_perf,
           cast(100.0 * percent_rank() over (order by norm_perf) as signed) as pct_perf,
           norm_comp,
           cast(100.0 * percent_rank() over (order by norm_comp) as signed) as pct_comp,
           norm_atti,
           cast(100.0 * percent_rank() over (order by norm_atti) as signed) as pct_atti,
           norm_total,
           cast(100.0 * percent_rank() over (order by norm_total) as signed) as pct_total
    from normalized_scores_int
    order by norm_total desc
),
```

## 3. 실행 결과

### 3-1. 전체 직원 평가 점수

### 3-2. 등급 cutoff

### 3-3. 원점수-표준점수 순위 변동

