## `Index Dive` 최적화

* `Index Dive` 개념과 실행 계획 수립에서의 역할 이해 및 최적화

### `Index Dive`의 이해

* 다중 컬럼 인덱스를 활용하여 효율적으로 쿼리를 실행하는 최적화 기법
* `MySQL`이 실행 계획을 수립할 때 사용 가능한 인덱스를 평가하는 과정
  * 실제 인덱스 검색에 사용할 데이터의 일부를 가지고 스캔
  * 이 과정을 통해 수집한 정보를 바탕으로 사용할 인덱스 결정
* 사용할 수 있는 인덱스 중에서 최적의 인덱스를 결정하기 위해 사용됨

#### `Index Dive` 예시

* col_name IN (10, 20, 30)에는 총 3개의 범위가 존재
* col_name = 10 `Index Dive`
  * col_name = 10에서 사용할 수 있는 인덱스 결정
  * col_name = 10이라는 조건에 해당하는 범위에서 첫 번째 다이빙
  * 다이빙은 인덱스의 시작 부분 탐색
  * 두 번째 다이빙은 같은 값(10)에 대한 인덱스의 끝 부분 탐색
  * 두 다이빙을 통해 col_name = 10 조건을 만족하는 행의 수 추정
* 각 범위당 두 번의 `Index Dive`를 함

### `Index Dive` 최적화란?

* `Index Dive` 과정이 오래 걸리는 것을 방지하여 너무 많은 시간을 소요하지 않도록 하는 것

#### `MySQL` Optimizer의 실행 계획 우선순위

1. `Index Dive`를 이용한 예측
2. `Histogram`을 이용한 예측
3. 인덱스 통계를 이용한 예측

#### `Index Dive`의 비용이 커지는 경우

* `IN(List)`절의 `List`에 많은 요소가 있는 경우
* `IN(List)`절과 `OR` 절을 같이 여러 개 사용하는 경우
* 많은 범위를 다루는 경우

#### `Index Dive` 과정을 생략할 수 있는 경우

* `JOIN`이 없는 단일 테이블에 대한 쿼리
* 단일 인덱스에 대해 힌트를 준 경우
* `FULLTEXT` 인덱스가 아니면서 `Unique Constraint`가 아닌 경우
* 서브 쿼리가 없는 경우
* `DISTINCT`, `GROUP BY`, `ORDER BY`가 없는 경우
* `MySQL` 8.0 미만 버전에서는 `eq_range_index_dive_limit` 변수 조작을 통해서만 생략 가능

#### `Index Dive`를 못하게 만드는 시스템 변수

* `range_optimizer_max_mem_size`로 지정한 메모리 값보다 더 많은 `IN(List)`의 요소를 사용할 경우 실행 계획은 포기되고 인덱스는 사용되지 않음

### `Index Dive` 최적화

* `FORCE INDEX` 사용
  * 지정한 인덱스로 스캔하도록 만들어 `Index Dive`를 피함
  * 쿼리에 적합한 인덱스인지 검토 필요
* `eq_range_index_dive_limit` 변수 조정
  * 변수로 지정한 값보다 많은 비교를 해야하면 `Index Dive`를 수행하지 않고 인덱스 통계 정보만을 가지고 실행 계획 수립
  * 범위 비교가 아닌 동등 비교에만 해당

### 고려할 점

* Index Dive에 의해 수집되는 정보가 인덱스 통계에 비해 정확하므로 성능에 큰 영향을 주는 경우에 사용 권장

### 실습

#### `FORCE INDEX` 사용

```mysql
SELECT *
FROM table_name FORCE INDEX (index_name)
WHERE indexed_column IN (...)
```

#### `eq_range_index_dive_limit` 확인

```mysql
SHOW SESSION VARIABLES LIKE 'eq_range_index_dive_limit'
```

> 기본 값은 `200`

#### `performance_schema` 확인

* `performance_schema` 설정이 `ON`이어야 사용 가능

```mysql
SHOW VARIABLES LIKE 'performance_schema'
```

> 기본 값이 `ON`

```mysql
SELECT stages.EVENT_ID, statements.EVENT_ID, statements.END_EVENT_ID, statements.SQL_TEXT, stages.EVENT_NAME, stages.TIMER_WAIT
FROM performance_schema.events_stages_history_long AS stages
         JOIN performance_schema.events_statements_history_long AS statements
              ON (stages.EVENT_ID >= statements.EVENT_ID AND stages.EVENT_ID <= statements.END_EVENT_ID)
WHERE stages.EVENT_NAME LIKE '%statistics%'
  AND statements.SQL_TEXT LIKE '%FROM order%'
  AND statements.SQL_TEXT NOT LIKE '%SELECT stages.EVENT_ID%'
ORDER BY statements.EVENT_ID DESC;
```