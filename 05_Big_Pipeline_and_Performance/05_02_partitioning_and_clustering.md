
## 목차

* [1. 함수 설명](#1-함수-설명)
  * [1-1. `PARTITION BY`](#1-1-partition-by) 
  * [1-2. `GROUP BY`](#1-2-group-by)
* [2. 대용량 테이블 스캔 비용 단축 실습](#2-대용량-테이블-스캔-비용-단축-실습)
  * [2-1. `PARTITION BY` 사용](#2-1-partition-by-사용)
  * [2-2. `GROUP BY` 사용](#2-2-group-by-사용)

## 1. 함수 설명

| 함수             | 기본 구문                                                    | 함수 설명                                                                                                                                   |
|----------------|----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `PARTITION BY` | `(집계 함수) OVER (PARTITION BY column_name [ORDER BY ...])` | 특정 기준을 이용하여 partition 으로 grouping [(참고)](../02_Aggregate_and_Feature_Engineering/02_03_set_and_remove_duplicate.md#2-1-partition-by-함수) |
| `GROUP BY`     | `GROUP BY column_name`                                   | `column_name` 컬럼의 값을 기준으로 전체 row를 grouping                                                                                              |

### 1-1. `PARTITION BY`

### 1-2. `GROUP BY`

## 2. 대용량 테이블 스캔 비용 단축 실습

### 2-1. `PARTITION BY` 사용

### 2-2. `GROUP BY` 사용
