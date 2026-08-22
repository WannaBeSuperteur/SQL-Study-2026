
## 목차

* [1. `LEFT JOIN`](#1-left-join)
* [2. `FULL OUTER JOIN`](#2-full-outer-join)
* [3. 범위 조건 기반 조인 실전 예제](#3-범위-조건-기반-조인-실전-예제)
* [4. 레이블 데이터 결합 실전 예제](#4-레이블-데이터-결합-실전-예제)

## 1. `LEFT JOIN`

**LEFT JOIN** 은 **INNER JOIN** 과 마찬가지로 SQL에서 **교집합** 을 나타내는 역할을 한다.

* 예시

```
select distinct p.mle_id,
                p.primary_framework,
                m.name,
                m.birth_date,
                m.gender
from probation_data as p
left join mle_employee_personal_info as m
on p.mle_id = m.mle_id
```

* 실행 결과

```
mle_id       |primary_framework|name|birth_date|gender|
-------------+-----------------+----+----------+------+
MLE-2023-0001|PyTorch          |김지훈 |1988-03-03|남성    |
MLE-2023-0002|JAX              |윤시우 |1998-07-09|남성    |
MLE-2023-0003|PyTorch          |송도윤 |1995-03-31|남성    |
MLE-2023-0004|TensorFlow       |이서연 |1984-02-07|여성    |
MLE-2023-0005|TensorFlow       |윤현준 |1995-07-03|남성    |
MLE-2023-0006|None             |송지호 |1998-01-22|남성    |
MLE-2023-0007|None             |윤은서 |1995-03-21|여성    |
MLE-2023-0008|TensorFlow       |김하린 |2000-01-28|여성    |
MLE-2024-0009|None             |신준혁 |1988-03-26|남성    |
```

## 2. `FULL OUTER JOIN`

**FULL OUTER JOIN** 은 **양쪽 테이블의 데이터를 모두 가져오는** 형태의 JOIN 이다.

* MySQL에서는 **해당 구문을 지원하지 않는다.**
* MySQL에서 대신 사용하려면 다음과 같이 **LEFT JOIN, RIGHT JOIN 을 결합** 해야 한다.

```
select distinct coalesce(p.mle_id, m.mle_id) as mle_id,
       p.primary_framework,
       m.name,
       m.birth_date,
       m.gender
from probation_data as p
left join mle_employee_personal_info as m
on p.mle_id = m.mle_id

union

select coalesce(p.mle_id, m.mle_id) as mle_id,
       p.primary_framework,
       m.name,
       m.birth_date,
       m.gender
from probation_data as p
right join mle_employee_personal_info as m
on p.mle_id = m.mle_id;
```

* 실행 결과

```
mle_id       |primary_framework|name|birth_date|gender|
-------------+-----------------+----+----------+------+
MLE-2023-0001|PyTorch          |김지훈 |1988-03-03|남성    |
MLE-2023-0002|JAX              |윤시우 |1998-07-09|남성    |
MLE-2023-0003|PyTorch          |송도윤 |1995-03-31|남성    |
MLE-2023-0004|TensorFlow       |이서연 |1984-02-07|여성    |
MLE-2023-0005|TensorFlow       |윤현준 |1995-07-03|남성    |
MLE-2023-0006|None             |송지호 |1998-01-22|남성    |
MLE-2023-0007|None             |윤은서 |1995-03-21|여성    |
MLE-2023-0008|TensorFlow       |김하린 |2000-01-28|여성    |
MLE-2024-0009|None             |신준혁 |1988-03-26|남성    |
```

## 3. 범위 조건 기반 조인 실전 예제

## 4. 레이블 데이터 결합 실전 예제

