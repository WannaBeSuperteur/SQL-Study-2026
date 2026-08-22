
**2. 집계 및 피처 엔지니어링 (Feature Engineering)**

* [2-1. 범주형 피처 인코딩](02_01_categorical_feature_encoding.md)
  * `CASE WHEN` 및 `PIVOT`을 이용한 One-Hot Encoding 형태의 원시 피처 생성
* [2-2. 집계 피처 생성](02_02_generate_aggregate_feature.md)
  * `GROUP BY`, `HAVING` 및 조건별 집계(`COUNT(CASE WHEN ...)`, `SUM(CASE WHEN...)`)
* [2-3. 집합 연산 및 중복 제거](02_03_set_and_remove_duplicate.md)
  * `UNION ALL`, `INTERSECT`, `EXCEPT`를 통한 통합 학습용 데이터셋 구성
  * 목적: **의도적으로 포함된 중복 행을 제거하고 유일한 레코드만 추출**
