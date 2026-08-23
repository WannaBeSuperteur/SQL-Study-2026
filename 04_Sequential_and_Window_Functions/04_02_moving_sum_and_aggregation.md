
## 목차

* [1. 함수 설명](#1-함수-설명)
  * [1-1. `OVER (PARTITION BY ... ORDER BY ...)`](#1-1-over-partition-by--order-by-)
  * [1-2. `OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN ...)`](#1-2-over-partition-by--order-by--rows-between-)
* [2. 기본  `OVER (PARTITION BY ... ORDER BY ...)` 예제](#2-기본-over-partition-by--order-by--예제) 
* [3. Rolling Feature 생성 예제](#3-rolling-feature-생성-예제)

## 1. 함수 설명

### 1-1. `OVER (PARTITION BY ... ORDER BY ...)`

* 참고: [`PARTITION BY` 함수](../02_Aggregate_and_Feature_Engineering/02_03_set_and_remove_duplicate.md#2-1-partition-by-함수)

`OVER (PARTITION BY column_1 ORDER BY column_2)` 는 다음과 같은 의미를 갖는다.

* `column_1` 을 기준으로 row 들의 파티션을 나눈다.
  * `RANK()` 등 순위 함수를 사용할 때, **각 파티션 별로 순위가 매겨진다.** 
* `column_2` 를 기준으로 row 들이 정렬된다.

### 1-2. `OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN ...)`

`OVER (PARTITION BY column_1 ORDER BY column_2 ROWS BETWEEN ...)` 는 **집계 함수를 이용하여, 현재 행과 전후 행을 포함하여 집계** 한다.

* 기본 구문

```sql
집계함수 OVER (
    PARTITION BY column_1
    ORDER BY column_2
    ROWS BETWEEN ... AND ...
)
```

* `ROWS BETWEEN` 에 정의된 키워드에 따라, **현재 row의 앞뒤 row를 포함하여 집계** 한다.

| 키워드           | `ROWS BETWEEN` 과 함께 사용 시 예시                         | 설명                                               |
|---------------|-----------------------------------------------------|--------------------------------------------------|
| `N PRECEDING` | `ROWS BETWEEN UNBOUNDED PRECEIDING AND 1 PRECEDING` | 현재 행의 `N`개 행만큼 앞의 행 (`N` = `UNBOUNDED` 이면 첫 행)   |
| `N FOLLOWING` | `ROWS BETWEEN 1 FOLLOWING AND UNBOUNDED FOLLOWING`  | 현재 행의 `N`개 행만큼 뒤쪽 행 (`N` = `UNBOUNDED` 이면 맨 끝 행) |
| `CURRENT ROW` | `ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING`          | 현재 행                                             |

## 2. 기본 `OVER (PARTITION BY ... ORDER BY ...)` 예제

## 3. Rolling Feature 생성 예제

