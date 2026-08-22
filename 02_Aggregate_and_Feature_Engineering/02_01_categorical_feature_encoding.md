
## 목차

* [1. 기본 함수 설명](#1-기본-함수-설명)
* [2. 실전 예제](#2-실전-예제)
  * [2-1. categorical feature one-hot encoding 하기](#2-1-categorical-feature-one-hot-encoding-하기)
  * [2-2. Multi-label categorical feature 처리하기](#2-2-multi-label-categorical-feature-처리하기)

## 1. 기본 함수 설명

* 함수 설명

| 함수          | 설명                                                                                                                                                                         |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CASE WHEN` | ```CASE WHEN 조건1 THEN 값1 ...``` 과 같은 **if 조건문** 역할 [(참고)](../01_Data_Clean_and_Preprocessing/01_01_missing_data_and_exception.md#1-기본-함수-설명)<br>- 새로운 열 생성 또는 집계 함수 용도로 사용 |
| `PIVOT`     | 데이터의 **행과 열을 재정렬 (pivot table 형태)** 및 집계 함수를 활용하여 재집계 등 데이터를 처리하는 방법<br>- ```PIVOT``` 과 같은 전용 함수를 이용하는 것이 아닌, ```CASE WHEN``` ```SUM``` ```MAX``` 등을 조합하여 pivot 할 수 있다.    |

## 2. 실전 예제

### 2-1. categorical feature one-hot encoding 하기

* 예시
  * ```primary_framework``` 값에 따라 (```PyTorch```, ```TensorFlow```, ```Scikit-Learn```, ```JAX```) one-hot encoding 
  * ```candidate_source``` 값에 따라 (```Referral```, ```Direct Apply```, ```Agency```, ```Campus Recruitment```) one-hot encoding

```sql
select mle_id,
       primary_framework,
       case when primary_framework = 'PyTorch' then 1 else 0 end as pf_pytorch,
       case when primary_framework = 'TensorFlow' then 1 else 0 end as pf_tf,
       case when primary_framework = 'Scikit-Learn' then 1 else 0 end as pf_sklearn,
       case when primary_framework = 'JAX' then 1 else 0 end as pf_jax,
       candidate_source,
       case when candidate_source = 'Referral' then 1 else 0 end as cs_ref,
       case when candidate_source = 'Direct Apply' then 1 else 0 end as cs_direct,
       case when candidate_source = 'Agency' then 1 else 0 end as cs_agency,
       case when candidate_source = 'Campus Recruitment' then 1 else 0 end as cs_campus
from probation_data
```

* 실행 결과

```
mle_id       |primary_framework|pf_pytorch|pf_tf|pf_sklearn|pf_jax|candidate_source  |cs_ref|cs_direct|cs_agency|cs_campus|
-------------+-----------------+----------+-----+----------+------+------------------+------+---------+---------+---------+
MLE-2023-0001|PyTorch          |         1|    0|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0001|PyTorch          |         1|    0|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0001|PyTorch          |         1|    0|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0002|JAX              |         0|    0|         0|     1|Direct Apply      |     0|        1|        0|        0|
MLE-2023-0002|JAX              |         0|    0|         0|     1|Direct Apply      |     0|        1|        0|        0|
MLE-2023-0002|JAX              |         0|    0|         0|     1|Direct Apply      |     0|        1|        0|        0|
MLE-2023-0003|PyTorch          |         1|    0|         0|     0|Agency            |     0|        0|        1|        0|
MLE-2023-0003|PyTorch          |         1|    0|         0|     0|Agency            |     0|        0|        1|        0|
MLE-2023-0003|PyTorch          |         1|    0|         0|     0|Agency            |     0|        0|        1|        0|
MLE-2023-0004|TensorFlow       |         0|    1|         0|     0|Direct Apply      |     0|        1|        0|        0|
MLE-2023-0004|TensorFlow       |         0|    1|         0|     0|Direct Apply      |     0|        1|        0|        0|
MLE-2023-0004|TensorFlow       |         0|    1|         0|     0|Direct Apply      |     0|        1|        0|        0|
MLE-2023-0005|TensorFlow       |         0|    1|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0005|TensorFlow       |         0|    1|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0005|TensorFlow       |         0|    1|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0004|TensorFlow       |         0|    1|         0|     0|Direct Apply      |     0|        1|        0|        0|
MLE-2023-0006|None             |         0|    0|         0|     0|Agency            |     0|        0|        1|        0|
MLE-2023-0006|None             |         0|    0|         0|     0|Agency            |     0|        0|        1|        0|
MLE-2023-0006|None             |         0|    0|         0|     0|Agency            |     0|        0|        1|        0|

...

MLE-2023-0010|JAX              |         0|    0|         0|     1|Campus Recruitment|     0|        0|        0|        1|
MLE-2023-0010|JAX              |         0|    0|         0|     1|Campus Recruitment|     0|        0|        0|        1|
MLE-2023-0010|JAX              |         0|    0|         0|     1|Campus Recruitment|     0|        0|        0|        1|
MLE-2024-0011|TensorFlow       |         0|    1|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2024-0011|TensorFlow       |         0|    1|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2024-0011|TensorFlow       |         0|    1|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2024-0012|PyTorch          |         1|    0|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2024-0012|PyTorch          |         1|    0|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2024-0012|PyTorch          |         1|    0|         0|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0013|Scikit-Learn     |         0|    0|         1|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0013|Scikit-Learn     |         0|    0|         1|     0|Referral          |     1|        0|        0|        0|
MLE-2023-0013|Scikit-Learn     |         0|    0|         1|     0|Referral          |     1|        0|        0|        0|
```

### 2-2. Multi-label categorical feature 처리하기

* 예시

```sql
select mle_id,
       skills_list,
       case when skills_list LIKE '%Python%' then 1 else 0 end as skill_python,
       case when skills_list LIKE '%PyTorch%' then 1 else 0 end as skill_pytorch,
       case when skills_list LIKE '%FastAPI%' then 1 else 0 end as skill_fastapi,
       case when skills_list LIKE '%SQL%' then 1 else 0 end as skill_sql,
       case when skills_list LIKE '%Docker%' then 1 else 0 end as skill_docker,
       case when skills_list LIKE '%TensorFlow%' then 1 else 0 end as skill_tf,
       case when skills_list LIKE '%Kubeflow%' then 1 else 0 end as skill_kbf,
       case when skills_list LIKE '%JAX%' then 1 else 0 end as skill_jax
from probation_data
```

* 실행 결과

```
mle_id       |skills_list                                      |skill_python|skill_pytorch|skill_fastapi|skill_sql|skill_docker|skill_tf|skill_kbf|skill_jax|
-------------+-------------------------------------------------+------------+-------------+-------------+---------+------------+--------+---------+---------+
MLE-2023-0001|SQL;Docker;PyTorch                               |           0|            1|            0|        1|           1|       0|        0|        0|
MLE-2023-0001|SQL;Docker;PyTorch                               |           0|            1|            0|        1|           1|       0|        0|        0|
MLE-2023-0001|SQL;Docker;PyTorch                               |           0|            1|            0|        1|           1|       0|        0|        0|
MLE-2023-0002|Docker;Kubeflow;SQL;CUDA;JAX                     |           0|            0|            0|        1|           1|       0|        1|        1|
MLE-2023-0002|Docker;Kubeflow;SQL;CUDA;JAX                     |           0|            0|            0|        1|           1|       0|        1|        1|
MLE-2023-0002|Docker;Kubeflow;SQL;CUDA;JAX                     |           0|            0|            0|        1|           1|       0|        1|        1|
MLE-2023-0003|Ray;Git;Kubeflow;CUDA;Python;PyTorch             |           1|            1|            0|        0|           0|       0|        1|        0|
MLE-2023-0003|Ray;Git;Kubeflow;CUDA;Python;PyTorch             |           1|            1|            0|        0|           0|       0|        1|        0|
MLE-2023-0003|Ray;Git;Kubeflow;CUDA;Python;PyTorch             |           1|            1|            0|        0|           0|       0|        1|        0|
MLE-2023-0004|Ray;Spark;Git;Kubeflow;TensorFlow                |           0|            0|            0|        0|           0|       1|        1|        0|
MLE-2023-0004|Ray;Spark;Git;Kubeflow;TensorFlow                |           0|            0|            0|        0|           0|       1|        1|        0|
MLE-2023-0004|Ray;Spark;Git;Kubeflow;TensorFlow                |           0|            0|            0|        0|           0|       1|        1|        0|
MLE-2023-0005|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow       |           1|            1|            0|        1|           0|       1|        1|        0|
MLE-2023-0005|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow       |           1|            1|            0|        1|           0|       1|        1|        0|
MLE-2023-0005|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow       |           1|            1|            0|        1|           0|       1|        1|        0|
MLE-2023-0004|Ray;Spark;Git;Kubeflow;TensorFlow                |           0|            0|            0|        0|           0|       1|        1|        0|
MLE-2023-0006|FastAPI;SQL                                      |           0|            0|            1|        1|           0|       0|        0|        0|
MLE-2023-0006|FastAPI;SQL                                      |           0|            0|            1|        1|           0|       0|        0|        0|
MLE-2023-0006|FastAPI;SQL                                      |           0|            0|            1|        1|           0|       0|        0|        0|
```
