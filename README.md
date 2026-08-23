# SQL-Study-2026

**[주의] csv 파일 2개 ([mle_employee_personal_info.csv](mle_employee_personal_info.csv), [mle_probation_evaluation_dataset.csv](mle_probation_evaluation_dataset.csv)) 는 Gemini 로 생성한 가상 (허구) 정보로, 실존 인물의 정보와 관련이 없습니다.**

학습 계획

* 핵심 요지
  * 머신러닝 엔지니어 / 데이터 사이언티스트로서 필요한 SQL 역량 향상
* Progress: ⬜ (TO-DO), 💨 (ING), ✅ (DONE)
* [Gemini 질문/답변](Gemini_Recommendation.md)

| Date                       | Plan                                                                                                  | Progress |
|----------------------------|-------------------------------------------------------------------------------------------------------|----------|
| 08.21 금 - 08.22 토 (2d)     | SQL (MySQL → SQLite) 설치 및 환경 설정                                                                       | ✅        |
| 08.21 금 - 08.22 토 (2d)     | 기본 SQL 실행 테스트 (csv 변환)                                                                                | ✅        |
| **08.22 토 (1d)**           | **1. 데이터 정제 및 기저 전처리**                                                                                | ✅        |
| 08.22 토 (1d)               | 1-1. 결측치 및 예외 처리<br> - `COALESCE`, `NULLIF`, `CASE WHEN`을 활용한 데이터 보정 및 기본값 대체                         | ✅        |
| 08.22 토 (1d)               | 1-2. 형변환 및 문자열 파싱<br> - `CAST`, `CONVERT`, `REGEXP`(정규표현식)을 이용한 텍스트 피처 추출                             | ✅        |
| 08.22 토 (1d)               | 1-3. 날짜 및 시간 데이터 핸들링<br> - `DATE_ADD`, `DATE_SUB`, `NOW`, `INTERVAL`, 타임존 변환을 통한 시계열 데이터 정렬           | ✅        |
| **08.22 토 (1d)**           | **2. 집계 및 피처 엔지니어링 (Feature Engineering)**                                                            | ✅        |
| 08.22 토 (1d)               | 2-1. 범주형 피처 인코딩<br> - `CASE WHEN` 및 `PIVOT`을 이용한 One-Hot Encoding 형태의 원시 피처 생성                        | ✅        |
| 08.22 토 (1d)               | 2-2. 집계 피처 생성<br> - `GROUP BY`, `HAVING` 및 조건별 집계(`COUNT(CASE WHEN ...)`, `SUM(CASE WHEN...)`)        | ✅        |
| 08.22 토 (1d)               | 2-3. 집합 연산 및 중복 제거<br> - `UNION ALL`, `INTERSECT`, `EXCEPT`를 통한 통합 학습용 데이터셋 구성                        | ✅        |
| **08.22 토 - 08.23 일 (2d)** | **3. 고급 결합 및 파이프라인 구조화**                                                                              | ✅        |
| 08.22 토 (1d)               | 3-1. 다중 및 비등가 조인(Non-Equi Join)<br> - `LEFT JOIN`, `FULL OUTER JOIN`, 범위 조건 기반 조인을 활용한 레이블 데이터 결합     | ✅        |
| 08.22 토 (1d)               | 3-2. CTE와 가독성 최적화<br> - `WITH` 절(Common Table Expression)을 활용한 복잡한 ML 데이터 추출 흐름 모듈화                   | ✅        |
| 08.23 일 (1d)               | 3-3. 반정형 데이터 다루기<br> - JSON 파싱 및 배열(Array) 데이터 풀기(`UNNEST`, `EXPLODE`)                                | ✅        |
| **08.23 일 (1d)**           | **4. 시계열 및 윈도우 분석 (Window Functions)**                                                                | 💨       |
| 08.23 일 (1d)               | 4-1. 순위 및 분위수 분할<br> - `ROW_NUMBER`, `DENSE_RANK`, `NTILE`을 활용한 그룹별 Top-K 추출 및 데이터 분할                 | ⬜        |
| 08.23 일 (1d)               | 4-2. 이동 합계 및 집계<br> - `OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN)`을 이용한 Rolling Feature 생성     | ⬜        |
| 08.23 일 (1d)               | 4-3. 시퀀스 데이터 및 지연 처리<br> - `LAG`, `LEAD` 함수를 활용한 이전/이후 시점 이벤트 추적 및 시계열 피처 구성                          | ⬜        |
| **08.24 월 - 08.25 화 (2d)** | **5. 대용량 파이프라인 및 성능 최적화**                                                                             | ⬜        |
| 08.24 월 (1d)               | 5-1. 실행 계획(EXPLAIN) 해석<br> - 쿼리 병목 지점 확인 및 조인 방식(Hash Join, Nested Loop 등) 이해                         | ⬜        |
| 08.25 화 (1d)               | 5-2. 파티셔닝 및 클러스터링<br> - `PARTITION BY`, `CLUSTER BY`를 활용한 대용량 테이블 스캔 비용 단축                            | ⬜        |
| 08.25 화 (1d)               | 5-3. 데이터 샘플링 및 중복 제거<br> - `TABLESAMPLE`, `RANDOM()`, `ROW_NUMBER()` 기반 층화 추출(Stratified Sampling) 기법 | ⬜        |

