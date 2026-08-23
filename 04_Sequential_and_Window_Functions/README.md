
**4. 시계열 및 윈도우 분석 (Window Functions)**

* [4-1. 순위 및 분위수 분할](04_01_ranking_and_quantile.md)
  * `ROW_NUMBER`, `DENSE_RANK`, `NTILE`을 활용한 그룹별 Top-K 추출 및 데이터 분할
* [4-2. 이동 합계 및 집계](04_02_moving_sum_and_aggregation.md)
  * `OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN)`을 이용한 Rolling Feature 생성
* [4-3. 시퀀스 데이터 및 지연 처리](04_03_sequential_data.md)
  * `LAG`, `LEAD` 함수를 활용한 이전/이후 시점 이벤트 추적 및 시계열 피처 구성

**[ SPECIAL PROJECT ]**

* 각 직원의 수습 평가 점수를 수능처럼 순위 매기기
* [프로젝트 문서](04_04_special_evaluation_cutoff.md)
