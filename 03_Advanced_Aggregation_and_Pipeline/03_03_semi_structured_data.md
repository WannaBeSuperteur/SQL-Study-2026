
## 목차

* [1. 반정형 데이터 (Semi-structured Data)](#1-반정형-데이터-semi-structured-data)
* [2. 반정형 데이터 파싱 함수](#2-반정형-데이터-파싱-함수)
  * [2-1. `UNNEST` 함수](#2-1-unnest-함수)
  * [2-2. `EXPLODE` 함수](#2-2-explode-함수)
* [3. 실전 예제](#3-실전-예제)

## 1. 반정형 데이터 (Semi-structured Data)

**반정형 데이터 (Semi-structured Data)** 는 **데이터의 형식 및 구조는 존재하지만, 그 구조가 바뀔 수 있는** 형태의 데이터이다.

* 대표적으로 XML, HTML, JSON 등이 있다.

## 2. 반정형 데이터 파싱 함수

SQL에서 반정형 데이터를 파싱하기 위한 함수는 다음과 같다.

| 함수                       | 설명                                    |
|--------------------------|---------------------------------------|
| `UNNEST`                 | 입력받은 `array`을 행들의 집합 (row list) 으로 변환 |
| `EXPLODE` (`JSON_TABLE`) | 배열 형태의 데이터로 묶인 것을 행으로 펼친다.            |

### 2-1. `UNNEST` 함수

`UNNEST` 함수는 **array를 입력받고, 해당 array의 원소들을 row list로 반환** 한다.

* MySQL에는 존재하지 않고, PostgreSQL에만 존재한다.

| 예시                                                  | 실행 결과                                           |
|-----------------------------------------------------|-------------------------------------------------|
| `SELECT unnest(ARRAY[2004, 8, 31])`                 | **unnest**<br>2004<br>8<br>31                   |
| `SELECT unnest(ARRAY[[2003, 9, 1], [2004, 8, 31]])` | **unnest**<br>2003<br>9<br>1<br>2004<br>8<br>31 |

### 2-2. `EXPLODE` 함수

`EXPLODE` (또는 `JSON_TABLE`) 은 **배열 형태의 데이터의 각 원소를 행으로 펼치는** 함수이다.

* MySQL 에서는 `EXPLODE` 대신 `JSON_TABLE` 함수를 이용하여 **JSON 형태의 데이터를 파싱** 할 수 있다.
* `JSON_TABLE` 함수의 구분은 다음과 같다.

```sql
SELECT ...
FROM {original_table}
CROSS JOIN JSON_TABLE(
    {original_table}.json_column
    '$'
    COLUMNS(
        column1_name {data_type} path '$.json_path'
        ...
        columnN_name {data_type} path '$.json_path'
    )
) AS {json_table}
```

* 예시

```sql
select p.mle_id,
       p.activity_json,
       jt.*
from probation_data as p
cross join JSON_TABLE(
    p.activity_json,
    '$'
    columns (
        github_star INTEGER path '$.github_stars',
        paper_cite INTEGER path '$.paper_citations',
        one_on_one BOOLEAN path '$.weekly_1on1_attended'
    )
) as jt;
```

| SQL 구문                                      | 설명                                                                                                    |
|---------------------------------------------|-------------------------------------------------------------------------------------------------------|
| `cross join JSON_TABLE(...`                 | - `cross join` 을 통해 두 테이블의 **모든 행을 조합 (카티션 곱)** 계산<br>- `JSON_TABLE`의 첫 인자는 **JSON 데이터/문자열을 추출할 컬럼명** |
| `$`                                         | json을 추출할 path ('$' 의 경우 json의 **최상위 root**)                                                          |
| `columns`                                   | json으로부터 추출할 컬럼 리스트                                                                                   |
| `github_star INTEGER path '$.github_stars'` | JSON 내부의 `github_stars` 로부터 데이터를 가져와서 `github_star` 컬럼을 추가하여 저장                                       |

* 실행 결과
  * 아래와 같이 **카티션 곱 형태의 테이블** 이 된다. 

```
mle_id       |activity_json                                                                                                       |github_star|paper_cite|one_on_one|
-------------+--------------------------------------------------------------------------------------------------------------------+-----------+----------+----------+
MLE-2023-0001|{"github_stars": 301, "paper_citations": 35, "certifications": ["AWS-ML"], "weekly_1on1_attended": true}            |        301|        35|         1|
MLE-2023-0001|{"github_stars": 194, "paper_citations": 12, "certifications": ["AWS-ML", "CKAD"], "weekly_1on1_attended": false}   |        194|        12|         0|
MLE-2023-0001|{"github_stars": 40, "paper_citations": 70, "certifications": ["AWS-ML", "CKAD"], "weekly_1on1_attended": false}    |         40|        70|         0|
MLE-2023-0002|{"github_stars": 350, "paper_citations": 41, "certifications": ["AWS-ML"], "weekly_1on1_attended": true}            |        350|        41|         1|
MLE-2023-0002|{"github_stars": 161, "paper_citations": 27, "certifications": ["GCP-Data-Engineer"], "weekly_1on1_attended": false}|        161|        27|         0|
MLE-2023-0002|{"github_stars": 275, "paper_citations": 33, "certifications": [], "weekly_1on1_attended": false}                   |        275|        33|         0|
MLE-2023-0003|{"github_stars": 134, "paper_citations": 64, "certifications": ["CKAD"], "weekly_1on1_attended": true}              |        134|        64|         1|
MLE-2023-0003|{"github_stars": 191, "paper_citations": 20, "certifications": [], "weekly_1on1_attended": true}                    |        191|        20|         1|
MLE-2023-0003|{"github_stars": 29, "paper_citations": 30, "certifications": [], "weekly_1on1_attended": true}                     |         29|        30|         1|
MLE-2023-0004|{"github_stars": 36, "paper_citations": 80, "certifications": ["AWS-ML"], "weekly_1on1_attended": true}             |         36|        80|         1|
MLE-2023-0004|{"github_stars": 342, "paper_citations": 62, "certifications": ["CKAD"], "weekly_1on1_attended": true}              |        342|        62|         1|
MLE-2023-0004|{"github_stars": 97, "paper_citations": 12, "certifications": ["AWS-ML"], "weekly_1on1_attended": false}            |         97|        12|         0|
```

## 3. 실전 예제

* 목표
  * 위 예제를 기반으로, json의 `certification` 항목의 자격증 리스트를 추출한다.
  * 이때, **모든 각 직원에 대해, 해당 직원의 자격증 1개 당 하나의 row** 로 하는 테이블을 반환한다.

* SQL 구문

```sql
select p.mle_id,
       jt.*
from probation_data as p
cross join JSON_TABLE(
    p.activity_json,
    '$.certifications[*]'
    columns (
        certificate VARCHAR(50) path '$'
    )
) as jt;
```

| SQL 구문                                       | 설명                                                                           |
|----------------------------------------------|------------------------------------------------------------------------------|
| `'$.certifications[*]'`                      | JSON의 root 바로 아래에 있는 `certifications` 배열의 모든 원소를 참조                          |
| `columns (certificate VARCHAR(50) path '$')` | 해당 `certifications[*]` 배열의 각 값을 참조하기 위해 `$` 를 사용하고, 이를 `certificate` 컬럼으로 저장 |

* 실행 결과

```
mle_id       |certificate      |
-------------+-----------------+
MLE-2023-0001|AWS-ML           |
MLE-2023-0001|AWS-ML           |
MLE-2023-0001|CKAD             |
MLE-2023-0001|AWS-ML           |
MLE-2023-0001|CKAD             |
MLE-2023-0002|AWS-ML           |
MLE-2023-0002|GCP-Data-Engineer|
MLE-2023-0003|CKAD             |
MLE-2023-0004|AWS-ML           |
MLE-2023-0004|CKAD             |
MLE-2023-0004|AWS-ML           |
MLE-2023-0005|GCP-Data-Engineer|
MLE-2023-0005|GCP-Data-Engineer|
MLE-2023-0005|AWS-ML           |
MLE-2023-0004|CKAD             |
MLE-2023-0006|GCP-Data-Engineer|
MLE-2023-0006|CKAD             |
MLE-2023-0007|AWS-ML           |
MLE-2023-0007|AWS-ML           |
MLE-2023-0007|CKAD             |
```