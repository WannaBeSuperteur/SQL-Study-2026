
## 목차

* [1. 기본 함수 설명](#1-기본-함수-설명)
* [2. 함수 사용 예시](#2-함수-사용-예시)
  * [2-1. `CAST`](#2-1-cast)
  * [2-2. `REGEXP_LIKE`](#2-2-regexplike)
  * [2-3. `REGEXP_REPLACE`](#2-3-regexpreplace)
  * [2-4. `REGEXP_INSTR`](#2-4-regexpinstr)
  * [2-5. `REGEXP_SUBSTR`](#2-5-regexpsubstr)
* [3. 참고 자료](#3-참고-자료)

## 1. 기본 함수 설명

* 함수 설명

| 함수               | 설명                                                                                                                                                 |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| `CAST`           | 데이터의 data type 을 변경하는 함수, ```CAST(value AS data_type_to_convert)```                                                                                |
| `CONVERT`        | 데이터의 형태 변환, ```CONVERT(data_type_to_convert, value)```<br>- 날짜 변환 등에서 ```CONVERT(data_type_to_convert, date, N)``` 식으로 하면 ```N```의 값에 따라 형태가 달라진다. |
| `REGEXP_LIKE`    | 정규식 패턴 검색 (WHERE 조건 용도)                                                                                                                            |
| `REGEXP_REPLACE` | 정규 표현식 기반으로 **다른 문자로 대체**                                                                                                                          |
| `REGEXP_INSTR`   | 정규 표현식과 **일치하는 부분의 위치** 반환                                                                                                                         |
| `REGEXP_SUBSTR`  | 정규 표현식과 **일치하는 부분의 문자열** 반환                                                                                                                        |
| `REGEXP_COUNT`   | 정규 표현식의 **패턴 등장 횟수** 반환                                                                                                                            |

### 1-1. `CAST`와 `CONVERT` 함수의 차이

* 사용법 (SQL 문법) 이 다르다는 것 이외에, CAST와 CONVERT 함수의 차이점은 다음과 같다.

| 구분           | RDBMS 지원                                         |
|--------------|--------------------------------------------------|
| `CAST` 함수    | **SQL 표준** (대부분의 RDBMS에서 지원)                     |
| `CONVERT` 함수 | **MySQL에서 사용하기 위한** 함수 (다른 RDBMS에서 지원되지 않을 수 있음) |

## 2. 함수 사용 예시

### 2-1. `CAST`

* 예시

```
SELECT mle_id,
       eval_performance_score,
       CAST(eval_performance_score AS varchar) as eval_perf_varchar,
       CAST(eval_performance_score AS int) as eval_perf_int
FROM probation_data
```

* 실행 결과

```
mle_id       |eval_performance_score|eval_perf_varchar|eval_perf_int|
-------------+----------------------+-----------------+-------------+
MLE-2023-0001|                 86.89|86.89            |           86|
MLE-2023-0001|                 88.44|88.44            |           88|
MLE-2023-0001|                 90.68|90.68            |           90|
MLE-2023-0002|                 70.68|70.68            |           70|
MLE-2023-0002|                 73.81|73.81            |           73|
MLE-2023-0002|                 76.44|76.44            |           76|
MLE-2023-0003|                 91.48|91.48            |           91|
MLE-2023-0003|                 91.27|91.27            |           91|
MLE-2023-0003|                 88.63|88.63            |           88|
MLE-2023-0004|                 61.54|61.54            |           61|
```

### 2-2. `REGEXP_LIKE`

* 예시
  * `skills_list` 컬럼에 `SQL`이 포함된 경우만 필터링

```
SELECT mle_id,
	   skills_list,
       eval_month_seq AS month,
       eval_performance_score AS performance,
       eval_competency_score AS competency,
       eval_attitude_score AS attitude
FROM probation_data
WHERE REGEXP_LIKE(skills_list, '.*SQL.*')
```

* 실행 결과

```
mle_id       |skills_list                                  |month|performance|competency|attitude|
-------------+---------------------------------------------+-----+-----------+----------+--------+
MLE-2023-0001|SQL;Docker;PyTorch                           |    1|      86.89|     88.66|   80.41|
MLE-2023-0001|SQL;Docker;PyTorch                           |    2|      88.44|      95.4|    87.3|
MLE-2023-0001|SQL;Docker;PyTorch                           |    3|      90.68|     97.58|    79.1|
MLE-2023-0002|Docker;Kubeflow;SQL;CUDA;JAX                 |    1|      70.68|     75.52|   93.45|
MLE-2023-0002|Docker;Kubeflow;SQL;CUDA;JAX                 |    2|      73.81|     77.22|   90.35|
MLE-2023-0002|Docker;Kubeflow;SQL;CUDA;JAX                 |    3|      76.44|     78.27|   91.27|
MLE-2023-0005|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |    1|      71.34|     100.0|   85.24|
MLE-2023-0005|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |    2|      66.74|     100.0|   94.89|
MLE-2023-0005|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |    3|      72.72|     100.0|   85.15|
MLE-2023-0006|FastAPI;SQL                                  |    1|      98.93|     67.86|   79.18|
MLE-2023-0006|FastAPI;SQL                                  |    2|      93.47|     66.16|   79.89|
MLE-2023-0006|FastAPI;SQL                                  |    3|      100.0|     74.27|   86.91|
MLE-2024-0011|Git;SQL;Kubeflow;CUDA;TensorFlow             |    1|      69.94|     70.11|   83.33|
MLE-2024-0011|Git;SQL;Kubeflow;CUDA;TensorFlow             |    2|      71.27|     74.57|   88.99|
MLE-2024-0011|Git;SQL;Kubeflow;CUDA;TensorFlow             |    3|      73.51|     78.34|   82.95|
MLE-2023-0013|SQL;Docker;FastAPI;Scikit-Learn              |    1|      92.06|     80.22|    56.7|
MLE-2023-0013|SQL;Docker;FastAPI;Scikit-Learn              |    2|      92.22|     83.99|   64.73|
MLE-2023-0013|SQL;Docker;FastAPI;Scikit-Learn              |    3|      91.35|     83.24|   64.44|
MLE-2024-0015|PyTorch;Git;SQL;Ray                          |    1|      71.17|     72.35|    61.1|
MLE-2024-0015|PyTorch;Git;SQL;Ray                          |    2|      78.99|     76.64|   63.16|
MLE-2024-0015|PyTorch;Git;SQL;Ray                          |    3|      78.48|     78.29|   67.77|
```

### 2-3. `REGEXP_REPLACE`

* 예시

```
SELECT mle_id,
	   last_commit_timestamp,
	   REGEXP_REPLACE(last_commit_timestamp,
	                  '[+]?[0-9]{2}:00|:00',
	                  '') as last_commit_date
FROM probation_data
WHERE REGEXP_LIKE(skills_list, '.*SQL.*')
```

* 실행 결과

```
mle_id       |last_commit_timestamp    |last_commit_date|
-------------+-------------------------+----------------+
MLE-2023-0001|2023-02-10 21:00:00+09:00|2023-02-10      |
MLE-2023-0001|2023-03-13 02:00:00+09:00|2023-03-13      |
MLE-2023-0001|2023-04-11 13:00:00+09:00|2023-04-11      |
MLE-2023-0002|2024-02-28 07:00:00+09:00|2024-02-28      |
MLE-2023-0002|2024-03-30 19:00:00+09:00|2024-03-30      |
MLE-2023-0002|2024-04-29 15:00:00+09:00|2024-04-29      |
MLE-2023-0005|2024-01-09 18:00:00+09:00|2024-01-09      |
MLE-2023-0005|2024-02-08 23:00:00+09:00|2024-02-08      |
MLE-2023-0005|2024-03-08 10:00:00+09:00|2024-03-08      |
```

### 2-4. `REGEXP_INSTR`

* 예시
  * ```SQL``` 이라는 skill이 ```sql_skills``` 의 몇 번째 글자에 등장하는지 확인 

```
SELECT mle_id,
       eval_month_seq AS month,
	   skills_list,
	   REGEXP_INSTR(skills_list, 'SQL') as sql_skills_idx
FROM probation_data
WHERE REGEXP_LIKE(skills_list, '.*SQL.*')
```

* 실행 결과

```
mle_id       |month|skills_list                                  |sql_skills_idx|
-------------+-----+---------------------------------------------+--------------+
MLE-2023-0001|    1|SQL;Docker;PyTorch                           |             1|
MLE-2023-0001|    2|SQL;Docker;PyTorch                           |             1|
MLE-2023-0001|    3|SQL;Docker;PyTorch                           |             1|
MLE-2023-0002|    1|Docker;Kubeflow;SQL;CUDA;JAX                 |            17|
MLE-2023-0002|    2|Docker;Kubeflow;SQL;CUDA;JAX                 |            17|
MLE-2023-0002|    3|Docker;Kubeflow;SQL;CUDA;JAX                 |            17|
MLE-2023-0005|    1|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |            22|
MLE-2023-0005|    2|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |            22|
MLE-2023-0005|    3|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |            22|
MLE-2023-0006|    1|FastAPI;SQL                                  |             9|
MLE-2023-0006|    2|FastAPI;SQL                                  |             9|
MLE-2023-0006|    3|FastAPI;SQL                                  |             9|
MLE-2024-0011|    1|Git;SQL;Kubeflow;CUDA;TensorFlow             |             5|
MLE-2024-0011|    2|Git;SQL;Kubeflow;CUDA;TensorFlow             |             5|
MLE-2024-0011|    3|Git;SQL;Kubeflow;CUDA;TensorFlow             |             5|
```

### 2-5. `REGEXP_SUBSTR`

* 예시
  * ```SQL``` 을 포함한 그 이전에 등장하는 ```skills``` 의 리스트 반환

```
SELECT mle_id,
       eval_month_seq AS month,
	   skills_list,
	   REGEXP_SUBSTR(skills_list, '.*SQL') as skills_until_sql
FROM probation_data
WHERE REGEXP_LIKE(skills_list, '.*SQL.*')
```

* 실행 결과

```
mle_id       |month|skills_list                                  |skills_until_sql           |
-------------+-----+---------------------------------------------+---------------------------+
MLE-2023-0001|    1|SQL;Docker;PyTorch                           |SQL                        |
MLE-2023-0001|    2|SQL;Docker;PyTorch                           |SQL                        |
MLE-2023-0001|    3|SQL;Docker;PyTorch                           |SQL                        |
MLE-2023-0002|    1|Docker;Kubeflow;SQL;CUDA;JAX                 |Docker;Kubeflow;SQL        |
MLE-2023-0002|    2|Docker;Kubeflow;SQL;CUDA;JAX                 |Docker;Kubeflow;SQL        |
MLE-2023-0002|    3|Docker;Kubeflow;SQL;CUDA;JAX                 |Docker;Kubeflow;SQL        |
MLE-2023-0005|    1|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |PyTorch;Kubeflow;Git;SQL   |
MLE-2023-0005|    2|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |PyTorch;Kubeflow;Git;SQL   |
MLE-2023-0005|    3|PyTorch;Kubeflow;Git;SQL;Python;TensorFlow   |PyTorch;Kubeflow;Git;SQL   |
MLE-2023-0006|    1|FastAPI;SQL                                  |FastAPI;SQL                |
MLE-2023-0006|    2|FastAPI;SQL                                  |FastAPI;SQL                |
MLE-2023-0006|    3|FastAPI;SQL                                  |FastAPI;SQL                |
MLE-2024-0011|    1|Git;SQL;Kubeflow;CUDA;TensorFlow             |Git;SQL                    |
MLE-2024-0011|    2|Git;SQL;Kubeflow;CUDA;TensorFlow             |Git;SQL                    |
MLE-2024-0011|    3|Git;SQL;Kubeflow;CUDA;TensorFlow             |Git;SQL                    |
```

## 3. 참고 자료

* [[ORACLE] 오라클 정규표현식 REGEXP 함수 사용법 - sanbon1112](https://blog.naver.com/abcd091/222315485958)
