
## 목차

* [1. 함수 설명](#1-함수-설명)
  * [1-1. `TABLESAMPLE` 함수](#1-1-tablesample-함수) 
  * [1-2. `RANDOM` 함수](#1-2-random-함수)
  * [1-3. `ROW_NUMBER` 함수](#1-3-rownumber-함수)
* [2. 층화 추출 (Stratified Sampling) 실습](#2-층화-추출-stratified-sampling-실습)
  * [2-1. 각 `primary_framework` 별 10명씩 추출](#2-1-각-primaryframework-별-10명씩-추출) 
  * [2-2. 각 `primary_framework` 별 15% 씩 추출](#2-2-각-primaryframework-별-15-씩-추출) 
* [3. 참고 문서](#3-참고-문서)

## 1. 함수 설명

| 함수            | 기본 구문                                   | 함수 설명                                                                               |
|---------------|-----------------------------------------|-------------------------------------------------------------------------------------|
| `TABLESAMPLE` | `TABLESAMPLE sampling_method (percent)` | 샘플링 메서드 `sampling_method` 를 사용하여 **`percent`% 만큼의 비율** 의 데이터 추출                     |
| `RANDOM`      | `RANDOM()` 또는 `RAND()`                  | 0부터 1 사이의 난수 값 (pseudo random) 생성                                                   |
| `ROW_NUMBER`  | `ROW_NUMBER() OVER (ORDER BY ...)`      | [(참고)](../04_Sequential_and_Window_Functions/04_01_ranking_and_quantile.md#1-함수-설명) |

### 1-1. `TABLESAMPLE` 함수

`TABLESAMPLE` 함수는 `sampling_method` 에 해당하는 샘플링 방법을 사용하여, 전체 데이터 중 일부를 추출하는 함수이다.

* `sampling_method` 에 들어가는 샘플링 방법
* **MySQL 에서는 지원하지 않는다.**
* 샘플링 방법

| 샘플링 방법 (`sampling_method`)<br>(PostgreSQL 9.5 기준) | 설명                                   |
|---------------------------------------------------|--------------------------------------|
| `system`                                          | **디스크의 데이터 페이지 단위** 로 해당 비율만큼 랜덤 샘플링 |
| `bernoulli`                                       | **개별 row** 단위로 해당 비율만큼 랜덤 샘플링        |

### 1-2. `RANDOM` 함수

`RANDOM` 함수는 0부터 1까지의 난수를 생성하는 함수이다.

* MySQL에서는 **`RAND()` 함수를 대신 사용** 한다.

| SQL 쿼리문                               | 실행 결과               | 실행 결과 범위          |
|---------------------------------------|---------------------|-------------------|
| `select rand()`                       | 0.29065022830911935 | 0.0 - 1.0 사이의 값   |
| `select rand() * 100`                 | 11.998317960648162  | 0.0 - 100.0 사이의 값 |
| `select cast(rand() * 100 as signed)` | 91                  | 0 - 100 사이의 정수 값  |

### 1-3. `ROW_NUMBER` 함수

`ROW_NUMBER` 함수는 `order by ...`로 지정된 정렬 기준에 따라 정렬했을 때의 **행 번호 (row number)** 를 출력하는 함수이다.

* 예시 SQL 구문

```sql
with distinct_mle_ids as (
	select distinct mle_id
	from probation_data
)
select row_number() over (order by mle_id) as row_num,
       mle_id
from distinct_mle_ids;
```

* 실행 결과

```
row_num|mle_id       |
-------+-------------+
      1|MLE-2023-0001|
      2|MLE-2023-0002|
      3|MLE-2023-0003|
      4|MLE-2023-0004|
      5|MLE-2023-0005|
      6|MLE-2023-0006|
      7|MLE-2023-0007|
      8|MLE-2023-0008|
      9|MLE-2023-0010|
     10|MLE-2023-0013|
```

## 2. 층화 추출 (Stratified Sampling) 실습

* 목표
  * `primary_framework` 기준으로, **각 프레임워크 별 10명씩, 그리고 15% 씩 row 를 층화 추출** 한다.
  * `ROW_NUMBER()`, `RAND()` 등의 함수를 사용한다.
  * `(mle_id, 월)` 조합이 아닌, **`mle_id` 만을 기준으로** 한다.

### 2-1. 각 `primary_framework` 별 10명씩 추출

* SQL 쿼리

```sql
with distinct_mle_ids as (
	select distinct mle_id,
	       primary_framework
	from probation_data
),
all_mles as (
    select mle_id,
           primary_framework,
           row_number() over (partition by primary_framework order by rand()) as row_num
    from distinct_mle_ids
)
select row_num,
       mle_id,
       primary_framework
from all_mles
where row_num <= 10 and primary_framework <> 'None';
```

* 실행 결과

```
row_num|mle_id       |primary_framework|
-------+-------------+-----------------+
      1|MLE-2023-0292|JAX              |
      2|MLE-2023-0318|JAX              |
      3|MLE-2023-0138|JAX              |
      4|MLE-2024-0082|JAX              |
      5|MLE-2023-0218|JAX              |
      6|MLE-2024-0254|JAX              |
      7|MLE-2024-0081|JAX              |
      8|MLE-2023-0307|JAX              |
      9|MLE-2024-0399|JAX              |
     10|MLE-2023-0029|JAX              |
      1|MLE-2024-0018|PyTorch          |
      2|MLE-2023-0203|PyTorch          |
      3|MLE-2023-0330|PyTorch          |
      4|MLE-2023-0349|PyTorch          |
      5|MLE-2023-0064|PyTorch          |
      6|MLE-2024-0104|PyTorch          |
      7|MLE-2023-0376|PyTorch          |
      8|MLE-2023-0269|PyTorch          |
      9|MLE-2023-0173|PyTorch          |
     10|MLE-2023-0383|PyTorch          |
      1|MLE-2024-0258|Scikit-Learn     |
      2|MLE-2023-0239|Scikit-Learn     |
      3|MLE-2023-0276|Scikit-Learn     |
      4|MLE-2024-0159|Scikit-Learn     |
      5|MLE-2023-0179|Scikit-Learn     |
      6|MLE-2023-0221|Scikit-Learn     |
      7|MLE-2024-0251|Scikit-Learn     |
      8|MLE-2024-0363|Scikit-Learn     |
      9|MLE-2024-0174|Scikit-Learn     |
     10|MLE-2023-0110|Scikit-Learn     |
      1|MLE-2024-0121|TensorFlow       |
      2|MLE-2024-0245|TensorFlow       |
      3|MLE-2024-0131|TensorFlow       |
      4|MLE-2024-0274|TensorFlow       |
      5|MLE-2023-0130|TensorFlow       |
      6|MLE-2023-0109|TensorFlow       |
      7|MLE-2023-0080|TensorFlow       |
      8|MLE-2023-0322|TensorFlow       |
      9|MLE-2023-0313|TensorFlow       |
     10|MLE-2024-0033|TensorFlow       |
```

### 2-2. 각 `primary_framework` 별 15% 씩 추출

* SQL 쿼리

```sql
with distinct_mle_ids as (
	select distinct mle_id,
	       primary_framework
	from probation_data
),
count_per_primary_framework as (
    select primary_framework,
           count(*) as cnt
    from distinct_mle_ids
    group by primary_framework
),
all_mles as (
    select mle_id,
           primary_framework,
           row_number() over (partition by primary_framework order by rand()) as row_num
    from distinct_mle_ids
)
select a.row_num,
       a.mle_id,
       a.primary_framework
from all_mles as a
join count_per_primary_framework as c
  on c.primary_framework = a.primary_framework
where a.row_num / c.cnt <= 0.15
  and a.primary_framework <> 'None'
order by primary_framework, a.row_num;
```

* 실행 결과

```
row_num|mle_id       |primary_framework|
-------+-------------+-----------------+
      1|MLE-2023-0294|JAX              |
      2|MLE-2023-0171|JAX              |
      3|MLE-2024-0223|JAX              |
      4|MLE-2024-0212|JAX              |
      5|MLE-2023-0010|JAX              |
      6|MLE-2023-0089|JAX              |
      7|MLE-2023-0382|JAX              |
      8|MLE-2023-0292|JAX              |
      9|MLE-2023-0222|JAX              |
     10|MLE-2024-0216|JAX              |
     11|MLE-2023-0218|JAX              |
     12|MLE-2023-0372|JAX              |
     13|MLE-2023-0002|JAX              |
     14|MLE-2023-0390|JAX              |
      1|MLE-2023-0394|PyTorch          |
      2|MLE-2023-0219|PyTorch          |
      3|MLE-2024-0317|PyTorch          |
      4|MLE-2024-0291|PyTorch          |
      5|MLE-2024-0012|PyTorch          |
      6|MLE-2024-0057|PyTorch          |
      7|MLE-2023-0066|PyTorch          |
      8|MLE-2023-0140|PyTorch          |
      9|MLE-2023-0373|PyTorch          |
     10|MLE-2023-0157|PyTorch          |
     11|MLE-2023-0184|PyTorch          |
      1|MLE-2024-0124|Scikit-Learn     |
      2|MLE-2024-0290|Scikit-Learn     |
      3|MLE-2024-0123|Scikit-Learn     |
      4|MLE-2024-0103|Scikit-Learn     |
      5|MLE-2024-0299|Scikit-Learn     |
      6|MLE-2023-0181|Scikit-Learn     |
      7|MLE-2024-0251|Scikit-Learn     |
      8|MLE-2023-0211|Scikit-Learn     |
      9|MLE-2024-0262|Scikit-Learn     |
     10|MLE-2024-0159|Scikit-Learn     |
     11|MLE-2024-0314|Scikit-Learn     |
     12|MLE-2024-0034|Scikit-Learn     |
     13|MLE-2023-0165|Scikit-Learn     |
     14|MLE-2023-0016|Scikit-Learn     |
      1|MLE-2023-0109|TensorFlow       |
      2|MLE-2024-0033|TensorFlow       |
      3|MLE-2024-0091|TensorFlow       |
      4|MLE-2024-0122|TensorFlow       |
      5|MLE-2023-0182|TensorFlow       |
      6|MLE-2024-0259|TensorFlow       |
      7|MLE-2023-0201|TensorFlow       |
      8|MLE-2023-0385|TensorFlow       |
      9|MLE-2024-0028|TensorFlow       |
```

* 참고

```sql
select * from count_per_primary_framework;
```

```
primary_framework|cnt|
-----------------+---+
PyTorch          | 78|
JAX              | 94|
TensorFlow       | 66|
None             | 64|
Scikit-Learn     | 98|
```

## 3. 참고 문서

* [Tablesample In PostgreSQL 9.5 - EDB POSTGRESQL](https://www.enterprisedb.com/blog/tablesample-postgresql-95)
