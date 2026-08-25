## 목차

* [1. 목표](#1-목표)
* [2. SQL 쿼리문](#2-sql-쿼리문)
* [3. 실행 결과](#3-실행-결과)
  * [3-1. 전체 직원 평가 점수](#3-1-전체-직원-평가-점수)
  * [3-2. 등급 cutoff](#3-2-등급-cutoff)
  * [3-3. 각 분류 별 인원수 및 비율 정보](#3-3-각-분류-별-인원수-및-비율-정보)
  * [3-4. 각 분류 별 직원 리스트](#3-4-각-분류-별-직원-리스트)

## 1. 목표

* 모든 직원의 성과, 역량, 태도 및 이들의 합산 점수를 월별로 평균한다.
* 성과, 역량, 태도 점수를 `평균 100, 표준편차 20`인 정규분포로 하여 각각 표준점수를 구한다.
* 영역별 표준점수 및 그 합산을 기준으로 `상위 4%, 11%, 23%, 40%, 60%, 77%, 89%, 96%` 점수를 각각 cutoff 로 하여,
  * 각 직원의 성과, 역량, 태도 점수의 백분위를 구한다.
  * **성과, 역량, 태도 및 합산 점수** 에 대한 **1~8 등급컷** 을 산출한다.
  * 해당 결과로 **각 직원의 성과, 역량, 태도, 합산 점수에 대한 등급을 산출** 한다. 
* 미리 정의된 **직원 분류 기준표** 에 따라 직원을 분류한 결과 column을 각 직원에 대해 추가한다.
* **직원 분류 기준표**의 각 기준에 속하는 직원의 인원수 및 비율 정보 및 해당 직원의 리스트를 각각 테이블 형태로 저장한다.

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
# 3. 영역별 표준점수 및 그 합산을 기준으로 상위 4%, 11%, 23%, 40%, 60%, 77%, 89%, 96% 점수를 각각 cutoff 로 하여,
# 3-1. 각 직원의 성과, 역량, 태도 점수의 백분위를 구한다.
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
# 3-2. 성과, 역량, 태도 및 합산 점수 에 대한 1~8 등급컷 을 산출한다.
percentage(cutoff_pct) as (
    values row(4), row(11), row(23), row(40), row(60), row(77), row(89), row(96)
),
cutoffs as (
    select row_number() over (order by p.cutoff_pct) as class,
           p.cutoff_pct,
           min(case when 100 - nsih.pct_perf <= p.cutoff_pct then nsih.norm_perf end) as perf_cutoff,
           min(case when 100 - nsih.pct_comp <= p.cutoff_pct then nsih.norm_comp end) as comp_cutoff,
           min(case when 100 - nsih.pct_atti <= p.cutoff_pct then nsih.norm_atti end) as atti_cutoff,
           min(case when 100 - nsih.pct_total <= p.cutoff_pct then nsih.norm_total end) as total_cutoff
    from normalized_scores_int_with_100 as nsih
    join percentage as p
    group by p.cutoff_pct
    order by p.cutoff_pct
),
# 3-3. 해당 결과로 각 직원의 성과, 역량, 태도, 합산 점수에 대한 등급을 산출 한다.
normalized_scores_int_with_100_and_class as (
    select nsih.mle_id,
           any_value(nsih.norm_perf) as norm_perf,
           any_value(nsih.pct_perf) as pct_perf,
           1 + count(case when nsih.norm_perf < c.perf_cutoff then 1 end) as class_perf,
           any_value(nsih.norm_comp) as norm_comp,
           any_value(nsih.pct_comp) as pct_comp,
           1 + count(case when nsih.norm_comp < c.comp_cutoff then 1 end) as class_comp,
           any_value(nsih.norm_atti) as norm_atti,
           any_value(nsih.pct_atti) as pct_atti,
           1 + count(case when nsih.norm_atti < c.atti_cutoff then 1 end) as class_atti,
           any_value(nsih.norm_total) as norm_total,
           any_value(nsih.pct_total) as pct_total,
           1 + count(case when nsih.norm_total < c.total_cutoff then 1 end) as class_total
    from normalized_scores_int_with_100 as nsih
    join cutoffs as c
    group by nsih.mle_id
    order by norm_total desc
),
# 4. 미리 정의된 직원 분류 기준표에 따라 직원을 분류한 결과 column을 각 직원에 대해 추가한다.
normalized_scores_final as (
    select nsi.mle_id,
           any_value(nsi.norm_perf) as norm_perf,
           any_value(nsi.pct_perf) as pct_perf,
           any_value(nsi.class_perf) as class_perf,
           any_value(nsi.norm_comp) as norm_comp,
           any_value(nsi.pct_comp) as pct_comp,
           any_value(nsi.class_comp) as class_comp,
           any_value(nsi.norm_atti) as norm_atti,
           any_value(nsi.pct_atti) as pct_atti,
           any_value(nsi.class_atti) as class_atti,
           any_value(nsi.norm_total) as norm_total,
           any_value(nsi.pct_total) as pct_total,
           any_value(nsi.class_total) as class_total,
           coalesce(group_concat(c.class_name separator ' | '), 'no class') as class_names
    from normalized_scores_int_with_100_and_class as nsi
    left join mle_classification as c  # left_join 사용 시 mle_classification 과 매칭이 안 되는 mle_id 도 처리 가능
      on nsi.class_perf >= cast(substring(c.performance_range, 1, 1) as signed) and
         nsi.class_perf <= cast(substring(c.performance_range, 3, 1) as signed) and
         nsi.class_comp >= cast(substring(c.competency_range, 1, 1) as signed) and
         nsi.class_comp <= cast(substring(c.competency_range, 3, 1) as signed) and
         nsi.class_atti >= cast(substring(c.attitude_range, 1, 1) as signed) and
         nsi.class_atti <= cast(substring(c.attitude_range, 3, 1) as signed)
    group by nsi.mle_id
    order by norm_total desc
),
# 5. 직원 분류 기준표의 각 기준에 속하는 직원의 인원수 및 비율 정보 및 해당 직원의 리스트를 각각 테이블 형태로 저장한다.
# 5-1. 각 기준에 속하는 인원수 및 비율 정보
count_info as (
    select c.class_name as class_name,
           any_value(c.performance_range) as performance_range,
           any_value(c.competency_range) as competency_range,
           any_value(c.attitude_range) as attitude_range,
           count(case when nsi.class_names like concat('%', c.class_name, '%') then 1 end) as class_count
    from mle_classification as c
    join normalized_scores_final as nsi
    group by c.class_name
),
ratio_info as (
    select class_name,
           performance_range,
           competency_range,
           attitude_range,
           class_count,
           concat(round(100 * class_count / (select count(*) from normalized_scores_final), 2), '%') as class_pct
    from count_info
    order by performance_range, competency_range, attitude_range
),
# 5-2. 각 기준의 해당 직원의 리스트 (예시)
employee_list_of_classification as (
    select nsi.mle_id,
           nsi.class_perf as class_perf,
           nsi.class_comp as class_comp,
           nsi.class_atti as class_atti
    from normalized_scores_final as nsi
    join mle_classification as c
      on nsi.class_perf >= cast(substring(c.performance_range, 1, 1) as signed) and
         nsi.class_perf <= cast(substring(c.performance_range, 3, 1) as signed) and
         nsi.class_comp >= cast(substring(c.competency_range, 1, 1) as signed) and
         nsi.class_comp <= cast(substring(c.competency_range, 3, 1) as signed) and
         nsi.class_atti >= cast(substring(c.attitude_range, 1, 1) as signed) and
         nsi.class_atti <= cast(substring(c.attitude_range, 3, 1) as signed)
    where c.class_name = '성과는 높으나 태도 개선 필요군'
)
# 최종 출력
select *
from normalized_scores_final;
```

## 3. 실행 결과

### 3-1. 전체 직원 평가 점수

```sql
# 최종 출력
select *
from normalized_scores_final;
```

```
mle_id       |norm_perf|pct_perf|class_perf|norm_comp|pct_comp|class_comp|norm_atti|pct_atti|class_atti|norm_total|pct_total|class_total|class_names                           |
-------------+---------+--------+----------+---------+--------+----------+---------+--------+----------+----------+---------+-----------+--------------------------------------+
MLE-2023-0289|      122|      81|         3|      134|      98|         1|      130|      93|         2|       386|      100|          1|역량·태도 우수 성과 대기만성형 | 역량·태도 만점의 전천후 후보  |
MLE-2023-0235|      133|      98|         1|      131|      95|         2|      119|      78|         3|       383|      100|          1|성과 중심 솔로 플레이어                         |
MLE-2024-0351|      131|      94|         2|      119|      77|         3|      131|      94|         2|       381|       99|          1|실전형 열정가                               |
MLE-2023-0073|      126|      88|         3|      127|      89|         2|      123|      84|         3|       376|       99|          1|no class                              |
MLE-2023-0228|      126|      88|         3|      127|      89|         2|      121|      81|         3|       374|       99|          1|no class                              |
MLE-2023-0381|      130|      93|         2|      125|      86|         3|      118|      77|         3|       373|       98|          1|성과 중심 솔로 플레이어                         |
MLE-2024-0261|      125|      86|         3|      125|      86|         3|      123|      84|         3|       373|       98|          1|무난한 중원 유지자 (B급 실무자)                   |
MLE-2023-0390|      125|      86|         3|      131|      95|         2|      116|      73|         4|       372|       98|          1|no class                              |
MLE-2024-0133|      110|      65|         4|      129|      92|         2|      132|      95|         2|       371|       98|          1|역량·태도 만점의 전천후 후보 | 역량·태도 우수 성과 대기만성형  |
MLE-2024-0399|      132|      97|         1|      123|      81|         3|      116|      73|         4|       371|       98|          1|성과 중심 솔로 플레이어                         |
```

### 3-2. 등급 cutoff

```sql
# 최종 출력
select *
from cutoffs;
```

```
class|cutoff_pct|perf_cutoff|comp_cutoff|atti_cutoff|total_cutoff|
-----+----------+-----------+-----------+-----------+------------+
    1|         4|        132|        132|        133|         362|
    2|        11|        127|        127|        127|         348|
    3|        23|        119|        119|        118|         331|
    4|        40|        107|        108|        108|         311|
    5|        60|         94|         93|         95|         291|
    6|        77|         83|         83|         82|         270|
    7|        89|         73|         74|         72|         254|
    8|        96|         66|         66|         68|         236|
```

### 3-3. 각 분류 별 인원수 및 비율 정보

```sql
# 최종 출력
select *
from ratio_info;
```

```
class_name         |performance_range|competency_range|attitude_range|class_count|class_pct|
-------------------+-----------------+----------------+--------------+-----------+---------+
전설적인 슈퍼스타          |1-1              |1-1             |1-1           |          0|0.00%    |
핵심 인재 (S급 리더)      |1-2              |1-2             |1-2           |          0|0.00%    |
성과 중심 솔로 플레이어      |1-2              |1-3             |3-5           |          4|1.00%    |
조직 위험 요소 에이스       |1-2              |1-3             |8-9           |          1|0.25%    |
실전형 열정가            |1-2              |3-5             |1-2           |          3|0.75%    |
성과는 높으나 태도 개선 필요군  |1-3              |1-2             |6-9           |          4|1.00%    
성과 중심의 노력형 전문가     |1-3              |6-9             |1-2           |          3|0.75%    |
운·환경 의존형 고성과자      |1-3              |7-9             |7-9           |          5|1.25%    |
실무 중심의 안정적 기여 대상   |1-4              |4-6             |4-6           |         44|11.00%   
역량·태도 만점의 전천후 후보   |1-9              |1-2             |1-2           |          5|1.25%    
성실한 협업 서포터         |2-4              |5-7             |1-3           |         19|4.75%    |
모범적인 성장 잠재주        |3-4              |3-4             |1-2           |          3|0.75%    |
역량·태도 우수 성과 대기만성형  |3-5              |1-2             |1-2           |          3|0.75%    
무난한 중원 유지자 (B급 실무자)|3-5              |3-5             |3-5           |         51|12.75%   
역량 개발이 필요한 일반 직원   |3-5              |6-9             |3-5           |         30|7.50%    
오만한 기술/전문 지식가      |4-6              |1-2             |7-9           |          7|1.75%    |
팀 내 긍정적 분위기 메이커    |5-7              |3-5             |1-2           |         15|3.75%    |
아이디어 중심의 분석가       |6-8              |1-3             |6-8           |         10|2.50%    |
역량·성과 정체 직원        |6-8              |6-8             |3-5           |         26|6.50%    |
성과 부진 잠재 핵심 인재     |6-9              |1-2             |1-2           |          2|0.50%    |
소극적인 기술 전문가        |6-9              |1-3             |7-9           |          9|2.25%    |
열정적 고군분투형          |6-9              |3-5             |1-2           |          8|2.00%    |
태도 우수 초급/적응 필요 인재  |7-9              |7-9             |1-2           |          2|0.50%    
성과·역량 집중 개선 대상     |7-9              |7-9             |6-8           |         11|2.75%    |
재배치 및 집중 관리 대상     |8-9              |8-9             |8-9           |          1|0.25%    |
```

### 3-4. 각 분류 별 직원 리스트

* 분류 이름: `성과는 높으나 태도 개선 필요군`

```sql
employee_list_of_classification as (
    select nsi.mle_id,

...

    where c.class_name = '성과는 높으나 태도 개선 필요군'
)
# 최종 출력
select *
from employee_list_of_classification;
```

```
mle_id       |class_perf|class_comp|class_atti|
-------------+----------+----------+----------+
MLE-2024-0380|         1|         2|         6|
MLE-2023-0361|         2|         2|         7|
MLE-2024-0359|         2|         2|         8|
MLE-2024-0088|         3|         2|         8|
```
