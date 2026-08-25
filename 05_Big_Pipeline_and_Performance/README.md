
**5. 대용량 파이프라인 및 성능 최적화**

* [5-1. 실행 계획(EXPLAIN) 해석](05_01_interpretation_of_explain.md)
  * 쿼리 병목 지점 확인 및 조인 방식(Hash Join, Nested Loop 등) 이해
* [5-2. 파티셔닝 및 클러스터링](05_02_partitioning_and_clustering.md)
  * `PARTITION BY`, `CLUSTER BY`를 활용한 대용량 테이블 스캔 비용 단축
* [5-3. 데이터 샘플링 및 중복 제거](05_03_data_sampling_and_remove_duplications.md)
  * `TABLESAMPLE`, `RANDOM()`, `ROW_NUMBER()` 기반 층화 추출(Stratified Sampling) 기법
