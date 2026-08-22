
## 목차

* [1. 연산자 설명](#1-연산자-설명)
  * [1-1. `UNION ALL`](#1-1-union-all) 
  * [1-2. `INTERSECT`](#1-2-intersect)
  * [1-3. `EXCEPT`](#1-3-except)
* [2. 실전 예제: 통합 학습용 데이터셋 구성](#2-실전-예제-통합-학습용-데이터셋-구성)
  * [2-1. `PARTITION BY` 함수](#2-1-partition-by-함수)

## 1. 연산자 설명

### 1-1. `UNION ALL`

**UNION ALL** 함수는 UNION 을 할 때 **중복 행 제거 없이 모든 행을 반환** 하는 연산자이다.

* 예를 들어 다음 2개의 테이블을 UNION ALL 하면 가장 오른쪽 테이블이 된다.

| ```table_1```                       | ```table_2```                       | ```UNION ALL``` 결과                                          |
|-------------------------------------|-------------------------------------|-------------------------------------------------------------|
| **user_id**<br>1201<br>1202<br>1203 | **user_id**<br>1201<br>1204<br>1205 | **user_id**<br>1201<br>1202<br>1203<br>1201<br>1204<br>1205 |

* 기본 구문은 다음과 같다.

```sql
SELECT column_1, column_2, ... FROM table_1
UNION ALL
SELECT column_1, column_2, ... FROM table_2
```

### 1-2. `INTERSECT`

**INTERSECT** 는 **컬럼 개수와 데이터 유형이 동일한** 두 테이블에서, **서로 동일하게 존재하는 데이터 (교집합)** 를 반환하는 연산자이다.

* 기본 구문은 다음과 같다.

```sql
SELECT column_1, column_2, ... FROM table_1
INTERSECT
SELECT column_1, column_2, ... FROM table_2
```

### 1-3. `EXCEPT`

**EXCEPT** 는 **`table_1`에는 존재하지만 `table_2`에는 존재하지 않는 데이터 (차집합)** 를 구하는 연산자이다.

* 기본 구문은 다음과 같다.

```sql
SELECT column_1, column_2, ... FROM table_1
EXCEPT
SELECT column_1, column_2, ... FROM table_2
```

## 2. 실전 예제: 통합 학습용 데이터셋 구성

* 목표
  * 의도적으로 포함된 **중복 행을 제거하고 유일한 레코드만 추출** 
  * 각 사원 (약 400 명) 을 row 로 하는 학습 데이터셋 구축 **(수습 1개월, 2개월, 3개월차 통합)**

**1. ```PARTITION BY``` 함수 사용 예제**

* 예제 SQL

```sql
select distinct mle_id,
    department_team,
    prior_experience_years,
    skills_list,
    sum(code_commits_cnt) over (partition by mle_id) as total_commits,
    sum(prs_merged_cnt) over (partition by mle_id) as total_pr_merges,
    avg(eval_performance_score) over (partition by mle_id) as avg_perf,
    avg(eval_competency_score) over (partition by mle_id) as avg_comp,
    avg(eval_attitude_score) over (partition by mle_id) as avg_atti
from probation_data
```

* 실행 결과

```
mle_id       |department_team    |prior_experience_years|skills_list                                      |total_commits|total_pr_merges|avg_perf         |avg_comp          |avg_atti          |
-------------+-------------------+----------------------+-------------------------------------------------+-------------+---------------+-----------------+------------------+------------------+
MLE-2023-0001|Computer Vision    |                   8.3|SQL;Docker;PyTorch                               |          116|             46|            88.67|             93.88|             82.27|
MLE-2023-0002|Computer Vision    |                   4.9|Docker;Kubeflow;SQL;CUDA;JAX                     |          189|             60|73.64333333333333| 77.00333333333333|             91.69|
MLE-2023-0003|MLOps & Platform   |                   2.3|Ray;Git;Kubeflow;CUDA;Python;PyTorch             |          182|             29|            90.46| 71.30000000000001|             67.55|
MLE-2023-0004|MLOps & Platform   |                   7.5|Ray;Spark;Git;Kubeflow;TensorFlow                |          228|             53|          68.0375|            64.755|            80.425|
MLE-2023-0005|Search & RecSys    |                  10.4|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow       |          208|             32|70.26666666666667|             100.0| 88.42666666666666|
MLE-2023-0006|LLM & Generative AI|                   3.2|FastAPI;SQL                                      |          212|             42|97.46666666666665| 69.42999999999999| 81.99333333333333|
MLE-2023-0007|Core Data Science  |                  10.6|Kubeflow;FastAPI;Docker                          |          131|             59|85.42333333333333|             77.52| 94.04333333333334|
MLE-2023-0008|Computer Vision    |                   4.6|PyTorch;FastAPI;Ray;TensorFlow                   |          179|             51|             92.2| 76.99333333333333| 81.70666666666666|
MLE-2023-0010|Search & RecSys    |                   4.4|Docker;PyTorch;Python;JAX                        |          246|             47|85.52666666666666| 93.33666666666666| 76.01666666666667|
MLE-2023-0013|LLM & Generative AI|                      |SQL;Docker;FastAPI;Scikit-Learn                  |          227|             51|91.87666666666667| 82.48333333333333| 61.95666666666667|
MLE-2023-0016|LLM & Generative AI|                   3.3|Kubeflow;Python;Docker;PyTorch;Spark;Scikit-Learn|          176|             45|            65.89|             70.68|             70.72|
```

### 2-1. `PARTITION BY` 함수

* 참고: [GROUP BY 함수](02_02_generate_aggregate_feature.md#1-함수-기본-설명)

`PARTITION BY` 함수는 특정 기준을 이용하여 grouping 을 하는 함수이다.

* 기본 구문: `(집계 함수) OVER (PARTITION BY column_name [ORDER BY ...])`
* `GROUP BY` 와의 차이점은 **Grouping 후에도 각 레코드들이 집계되지 않고 상세 정보가 유지** 된다는 것이다.
  * 따라서 `GROUP BY` 와 같은 효과를 보려면 `SELECT DISTINCT` 식으로 해야 한다.

| 구분                  | `GROUP BY` 결과                      | `PARTITION BY` 결과                  |
|---------------------|------------------------------------|------------------------------------|
| 기본                  | ![image](../images/02_03_0001.PNG) | ![image](../images/02_03_0002.PNG) |
| ```DISTINCT``` 사용 시 |                                    | ![image](../images/02_03_0003.PNG) |
