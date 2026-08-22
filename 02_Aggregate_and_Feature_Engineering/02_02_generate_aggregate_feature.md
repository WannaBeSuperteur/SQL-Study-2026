
## 목차

* [1. 함수 기본 설명](#1-함수-기본-설명)
* [2. 실전 예제](#2-실전-예제)
  * [2-1. `GROUP BY`, `HAVING`](#2-1-group-by-having)
  * [2-2. 조건별 집계(`COUNT(CASE WHEN ...)`, `SUM(CASE WHEN...)`)](#2-2-조건별-집계countcase-when--sumcase-when)

## 1. 함수 기본 설명

| 함수                         | 설명                                                                                                                                     |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| ```GROUP BY```             | 특정 column의 값이 동일한 값끼리 집계 (```GROUP BY column```)                                                                                       |
| ```HAVING```               | ```GROUP BY column``` 를 이용하여 각 ```column``` 값끼리 집계한 결과가 ```조건``` 을 만족하는 것만 필터링 (```GROUP BY column HAVING 조건```)                       |
| ```COUNT(CASE WHEN ...)``` | ```COUNT``` + ```CASE WHEN``` [(참고)](../01_Data_Clean_and_Preprocessing/01_01_missing_data_and_exception.md#1-기본-함수-설명) → 조건에 맞는 값의 개수 |
| ```SUM(CASE WHEN ...)```   | ```SUM``` + ```CASE WHEN``` [(참고)](../01_Data_Clean_and_Preprocessing/01_01_missing_data_and_exception.md#1-기본-함수-설명) → 조건에 맞는 값들의 합   |

## 2. 실전 예제

### 2-1. `GROUP BY`, `HAVING`

**[ 예제 1 ]**

* ```candidate_source``` (채용 경로) 별 성과, 역량, 태도 평균 점수

```
select candidate_source,
       avg(eval_performance_score) as avg_perf,
       avg(eval_competency_score) as avg_comp,
       avg(eval_attitude_score) as avg_atti
from probation_data
group by candidate_source
```

```
candidate_source  |avg_perf         |avg_comp         |avg_atti         |
------------------+-----------------+-----------------+-----------------+
Referral          |81.43459999999996|81.26196666666667|77.62800000000001|
Direct Apply      |83.65398523985237|79.26276752767527| 78.1008487084871|
Agency            |83.52645061728391|80.51666666666664| 79.3697839506173|
Campus Recruitment|82.82464052287581|80.92915032679737|75.77816993464052|
```

**[ 예제 2 ]**

* ```candidate_source``` (채용 경로) 및 ```primary_framework``` (주 사용 프레임워크) 별 성과, 역량, 태도 평균 점수

```
select candidate_source,
       primary_framework,
       avg(eval_performance_score) as avg_perf,
       avg(eval_competency_score) as avg_comp,
       avg(eval_attitude_score) as avg_atti
from probation_data
group by candidate_source, primary_framework
```

```
candidate_source  |primary_framework|avg_perf         |avg_comp         |avg_atti         |
------------------+-----------------+-----------------+-----------------+-----------------+
Referral          |PyTorch          |80.05893939393938|82.19969696969699|75.09424242424241|
Direct Apply      |JAX              |82.39129629629628| 78.5174074074074|77.09444444444445|
Agency            |PyTorch          |83.24243589743588|81.30538461538463|79.83307692307694|
Direct Apply      |TensorFlow       |82.12129032258063|75.79451612903225|75.21838709677421|
Referral          |TensorFlow       |77.84079365079364|82.39111111111109| 79.4080952380952|
Agency            |None             | 85.0486274509804|78.59607843137255| 76.4441176470588|
Agency            |TensorFlow       |83.07979166666668|79.47791666666666|81.57708333333333|
Direct Apply      |None             |83.31877192982456|82.05508771929823|75.39157894736842|
Campus Recruitment|JAX              |83.89144444444447|84.27177777777779|75.50111111111109|
Referral          |Scikit-Learn     | 85.2655128205128|81.46756410256411|  78.479358974359|
Campus Recruitment|None             |82.37450000000003|77.43233333333335|75.83666666666664|
Campus Recruitment|Scikit-Learn     |79.94416666666666| 76.6546666666667|75.93000000000005|
Agency            |JAX              |82.20753623188403|85.34521739130436|78.29753623188404|
Direct Apply      |PyTorch          |86.56725490196077| 81.5821568627451|79.38745098039215|
Campus Recruitment|TensorFlow       |87.01228070175435|80.50684210526317|78.13578947368421|
Direct Apply      |Scikit-Learn     |83.47743589743592|77.60012820512819|81.08179487179486|
Referral          |None             |79.52458333333333|           77.835|80.27291666666667|
Campus Recruitment|PyTorch          |79.36641025641026|85.78846153846155| 72.6482051282051|
Agency            |Scikit-Learn     |84.25679487179487|77.35153846153845|80.40961538461539|
Referral          |JAX              |82.36550724637681|80.29362318840585|76.54391304347826|
```

**[ 예제 3 ]**

* ```candidate_source``` (채용 경로) 및 ```primary_framework``` (주 사용 프레임워크) 별 성과, 역량, 태도 평균 점수
* 단, 이번에는 **태도 평균 점수가 80점 이상** 인 케이스만 추출

```
select candidate_source,
       primary_framework,
       avg(eval_performance_score) as avg_perf,
       avg(eval_competency_score) as avg_comp,
       avg(eval_attitude_score) as avg_atti
from probation_data
group by candidate_source, primary_framework
having avg_atti >= 80
```

```
candidate_source|primary_framework|avg_perf         |avg_comp         |avg_atti         |
----------------+-----------------+-----------------+-----------------+-----------------+
Agency          |TensorFlow       |83.07979166666668|79.47791666666666|81.57708333333333|
Direct Apply    |Scikit-Learn     |83.47743589743592|77.60012820512819|81.08179487179486|
Referral        |None             |79.52458333333333|           77.835|80.27291666666667|
Agency          |Scikit-Learn     |84.25679487179487|77.35153846153845|80.40961538461539|
```

### 2-2. 조건별 집계(`COUNT(CASE WHEN ...)`, `SUM(CASE WHEN...)`)

**[ 예제 1 ]**

* 성과, 역량, 태도가 모두 90점 이상인 직원에 대한 전체 기록 개수
  * 조건을 만족시키면 **1 을 반환** 하도록 한다.

```
select COUNT(case when eval_performance_score >= 90
             and eval_competency_score >= 90
             and eval_attitude_score >= 90 then 1 END) as best_employee_count
from probation_data
```

```
best_employee_count|
-------------------+
                 12|
```

**[ 예제 2 ]**

* 성과, 역량, 태도가 모두 90점 이상인 직원의 각 기록 시점의 commit 개수 합산
  * 조건을 만족시키면 **```code_commits_cnt``` 열의 값을 반환** 하도록 한다.

```
select SUM(case when eval_performance_score >= 90
             and eval_competency_score >= 90
             and eval_attitude_score >= 90 then code_commits_cnt END) as best_employee_commits_sum
from probation_data
```

```
best_employee_commits_sum|
-------------------------+
                      650|
```

** [ 참고 ]