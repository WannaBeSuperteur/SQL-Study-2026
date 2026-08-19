## Prompt

머신러닝 엔지니어에게 어떤 SQL 관련 역량이 필요할지 나열하고, 해당 역량을 기르기 위한 SQL 학습 주제 15가지를 추천해 줘. (단, 대분류-소분류 형태로 소분류를 총 15가지로 한다.)

## Gemini Answer

머신러닝 엔지니어(MLE)에게 SQL은 단순 데이터 조회를 넘어 학습 데이터셋(Dataset) 구축, 데이터베이스 단계에서의 피처 엔지니어링(Feature Engineering), 대용량 데이터 전처리 최적화를 위한 핵심 도구입니다.

머신러닝 엔지니어에게 필요한 4가지 핵심 SQL 역량

학습 데이터셋 추출 및 정제: 복잡한 조건의 결측치 처리, 이상치 제거, 레이블링 데이터를 정확히 추출하는 역량

SQL 기반 피처 엔지니어링: 모델 입력으로 들어갈 집계 피처, 이동 평균, 시계열 시퀀스 데이터 등을 파이썬 메모리로 로드하기 전에 DB 레벨에서 미리 생성하는 역량

대용량 데이터 쿼리 성능 최적화: BigQuery, Snowflake, Spark SQL 등 분산 데이터웨어하우스 환경에서 메모리 초과 없이 효율적으로 데이터를 다루는 역량

파이프라인 연동 및 자동화: Airflow, Feature Store 등 ML 파이프라인 및 데이터 워크플로우에 SQL 쿼리를 모듈화하여 이식하는 역량

머신러닝 엔지니어를 위한 SQL 학습 주제 15선

1. 데이터 정제 및 기저 전처리

01) 결측치 및 예외 처리: COALESCE, NULLIF, CASE WHEN을 활용한 데이터 보정 및 기본값 대체

02) 형변환 및 문자열 파싱: CAST, CONVERT, REGEXP(정규표현식)을 이용한 텍스트 피처 추출

03) 날짜 및 시간 데이터 핸들링: DATE_TRUNC, INTERVAL, 타임존 변환을 통한 시계열 데이터 정렬

2. 집계 및 피처 엔지니어링 (Feature Engineering)

04) 범주형 피처 인코딩: CASE WHEN 및 PIVOT을 이용한 One-Hot Encoding 형태의 원시 피처 생성

05) 집계 피처 생성: GROUP BY, HAVING 및 조건별 집계(COUNTIF, SUM(CASE WHEN...))

06) 집합 연산 및 중복 제거: UNION ALL, INTERSECT, EXCEPT를 통한 통합 학습용 데이터셋 구성

3. 고급 결합 및 파이프라인 구조화

07) 다중 및 비등가 조인(Non-Equi Join): LEFT JOIN, FULL OUTER JOIN, 범위 조건 기반 조인을 활용한 레이블 데이터 결합

08) CTE와 가독성 최적화: WITH 절(Common Table Expression)을 활용한 복잡한 ML 데이터 추출 흐름 모듈화

09) 반정형 데이터 다루기: JSON 파싱 및 배열(Array) 데이터 풀기(UNNEST, EXPLODE)

4. 시계열 및 윈도우 분석 (Window Functions)

10) 순위 및 분위수 분할: ROW_NUMBER, DENSE_RANK, NTILE을 활용한 그룹별 Top-K 추출 및 데이터 분할

11) 이동 합계 및 집계: OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN)을 이용한 Rolling Feature 생성

12) 시퀀스 데이터 및 지연 처리: LAG, LEAD 함수를 활용한 이전/이후 시점 이벤트 추적 및 시계열 피처 구성

5. 대용량 파이프라인 및 성능 최적화

13) 실행 계획(EXPLAIN) 해석: 쿼리 병목 지점 확인 및 조인 방식(Hash Join, Nested Loop 등) 이해

14) 파티셔닝 및 클러스터링: PARTITION BY, CLUSTER BY를 활용한 대용량 테이블 스캔 비용 단축

15) 데이터 샘플링 및 중복 제거: TABLESAMPLE, RANDOM(), ROW_NUMBER() 기반 층화 추출(Stratified Sampling) 기법