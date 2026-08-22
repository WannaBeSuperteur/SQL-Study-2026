
## 목차

* [1. 기본 함수 설명](#1-기본-함수-설명)
* [2. 함수 사용 예시](#2-함수-사용-예시)
  * [2-1. `COALESCE`](#2-1-coalesce)
  * [2-2. `NULLIF`](#2-2-nullif)
  * [2-3. `CASE WHEN`](#2-3-case-when)

## 1. 기본 함수 설명

* 함수 설명

| 함수          | 설명                                                                                   |
|-------------|--------------------------------------------------------------------------------------|
| `COALESCE`  | 2개 이상의 컬럼 중 **NULL이 아닌 최초의 컬럼 값** 을 사용<br>- 단, 모든 컬럼이 모두 ```NULL```이면 ```NULL```을 적용 |
| `NULLIF`    | 인자 2개가 **서로 같으면 NULL, 그렇지 않으면 1번째 인자** 를 반환                                          |
| `CASE WHEN` | ```CASE WHEN 조건1 THEN 값1 ...``` 과 같은 **if 조건문** 역할<br>- 새로운 열 생성 또는 집계 함수 용도로 사용     |

## 2. 함수 사용 예시

### 2-1. `COALESCE`

* 예시
  * `mentor_id` 가 NULL 이면 `mle_id`, 즉 자기 자신을 `true_mentor_id`로 지정 

```
SELECT mle_id,
       mentor_id,
       COALESCE(mentor_id, mle_id) AS true_mentor_id
FROM probation_data
```

* 실행 결과
  * ```mentor_id``` 가 NULL 인 ```mle_id``` 에 대해서, **`true_mentor_id`가 자기 자신으로 지정** 된다. 

```
mle_id       |mentor_id|true_mentor_id|
-------------+---------+--------------+
MLE-2023-0001|MNT-120  |MNT-120       |
MLE-2023-0001|MNT-120  |MNT-120       |
MLE-2023-0001|MNT-120  |MNT-120       |
MLE-2023-0002|MNT-118  |MNT-118       |
MLE-2023-0002|MNT-118  |MNT-118       |
MLE-2023-0002|MNT-118  |MNT-118       |

...

MLE-2023-0007|MNT-107  |MNT-107       |
MLE-2023-0007|MNT-107  |MNT-107       |
MLE-2023-0007|MNT-107  |MNT-107       |
MLE-2023-0008|         |MLE-2023-0008 |
MLE-2023-0008|         |MLE-2023-0008 |
MLE-2023-0008|         |MLE-2023-0008 |
MLE-2024-0009|         |MLE-2024-0009 |
MLE-2024-0009|         |MLE-2024-0009 |
MLE-2024-0009|         |MLE-2024-0009 |
MLE-2023-0010|         |MLE-2023-0010 |
MLE-2023-0010|         |MLE-2023-0010 |
MLE-2023-0010|         |MLE-2023-0010 |
```

### 2-2. `NULLIF`

* 예시
  * 성과 평가 점수 (`eval_performance_score`) 와 역량 평가 점수 (`eval_competency_score`) 가 서로 같은지 확인

```
SELECT mle_id,
       eval_performance_score,
       eval_competency_score,
       NULLIF(eval_performance_score, eval_competency_score) AS check_perf_comp_same
FROM probation_data
```

* 실행 결과
  * 성과 평가와 역량 평가가 모두 100점인 행의 경우 데이터 없음
  * 이외의 경우 **성과 평가 점수** 를 사용

```
mle_id       |eval_performance_score|eval_competency_score|check_perf_comp_same|
-------------+----------------------+---------------------+--------------------+
MLE-2023-0001|                 86.89|                88.66|               86.89|
MLE-2023-0001|                 88.44|                 95.4|               88.44|
MLE-2023-0001|                 90.68|                97.58|               90.68|
MLE-2023-0002|                 70.68|                75.52|               70.68|
MLE-2023-0002|                 73.81|                77.22|               73.81|
MLE-2023-0002|                 76.44|                78.27|               76.44|

...

MLE-2024-0017|                  71.6|                86.38|                71.6|
MLE-2024-0017|                 71.94|                85.03|               71.94|
MLE-2024-0017|                 70.44|                89.66|               70.44|
MLE-2024-0018|                 100.0|                100.0|                    |
MLE-2024-0018|                 100.0|                100.0|                    |
MLE-2024-0018|                 100.0|                100.0|                    |
MLE-2023-0019|                 80.84|                89.24|               80.84|
MLE-2023-0019|                 79.34|                 93.0|               79.34|
MLE-2023-0019|                 83.41|                95.35|               83.41|
```

* 기타 코멘트
  * 2번째 인자가 ```1번째 인자가 갖는 오류 값``` 이면, 이에 해당할 때 NULL 처리하도록 하면 좋을 듯함

### 2-3. `CASE WHEN`

* 예시

```
SELECT mle_id,
       eval_performance_score,
       eval_competency_score,
       eval_attitude_score,
       CASE WHEN eval_attitude_score < 70 THEN '태도 불량'
       		WHEN eval_competency_score >= 90 AND eval_performance_score >= 90 THEN 'S급 직원'
            WHEN eval_performance_score >= 80 THEN '성과 우수'
            WHEN eval_competency_score BETWEEN 70 AND 80 THEN '역량 향상 기대'
            ELSE '특이사항 없음'
       END AS probation_comment
FROM probation_data
```

* 실행 결과

```
mle_id       |eval_performance_score|eval_competency_score|eval_attitude_score|probation_comment|
-------------+----------------------+---------------------+-------------------+-----------------+
MLE-2023-0001|                 86.89|                88.66|              80.41|성과 우수            |
MLE-2023-0001|                 88.44|                 95.4|               87.3|성과 우수            |
MLE-2023-0001|                 90.68|                97.58|               79.1|S급 직원            |
MLE-2023-0002|                 70.68|                75.52|              93.45|역량 향상 기대         |
MLE-2023-0002|                 73.81|                77.22|              90.35|역량 향상 기대         |
MLE-2023-0002|                 76.44|                78.27|              91.27|역량 향상 기대         |
MLE-2023-0003|                 91.48|                67.29|              71.94|성과 우수            |
MLE-2023-0003|                 91.27|                71.82|              65.61|태도 불량            |
MLE-2023-0003|                 88.63|                74.79|               65.1|태도 불량            |
MLE-2023-0004|                 61.54|                57.59|              77.35|특이사항 없음          |
MLE-2023-0004|                  70.6|                 67.4|               80.3|특이사항 없음          |
MLE-2023-0004|                 69.41|                66.63|              83.75|특이사항 없음          |
MLE-2023-0005|                 71.34|                100.0|              85.24|특이사항 없음          |
MLE-2023-0005|                 66.74|                100.0|              94.89|특이사항 없음          |
MLE-2023-0005|                 72.72|                100.0|              85.15|특이사항 없음          |
```