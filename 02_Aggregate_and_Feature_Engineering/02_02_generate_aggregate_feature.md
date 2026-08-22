
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

### 2-2. 조건별 집계(`COUNT(CASE WHEN ...)`, `SUM(CASE WHEN...)`)

