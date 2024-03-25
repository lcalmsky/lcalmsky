## Auto Increment Lock Mode 최적화

* `innodb_autoinc_lock_mode` 설정을 통한 `MySQL`의 `INSERT` 성능 최적 방법
* `innodb_autoinc_lock_mode=2` 설정과 이를 적용할 때 고려해야하는 상황에 대해 이해

### `innodb_autoinc_lock_mode` 설정에 따른 최적화

* Auto Increment 칼럼을 가진 테이블에 INSERT 작업시 동시성 수준 설정

### INSERT 유형

* 넣을 행의 수를 미리 알 수 있는지 여부로 결정
  * Simple inserts (예측 가능한 행 수): 일반 INSERT 문
  * Bulk Inserts (예측 불가능한 행 수): INSERT ... SELECT, LOAD DATA, REPLACE ... SELECT

### Auto Increment Lock Mode 종류

* `inno_autoinc_lock_mode = 0`: 모든 `INSERT`에 `Table Level AUTO-INC Lock` 사용
* `inno_autoinc_lock_mode = 1`: `Bulk INSERT`에서만 `Table Level AUTO-INC Lock` 사용
* `inno_autoinc_lock_mode = 2`: `Table Level AUTO-INC Lock` 미사용, 가벼운 `Mutex` 사용, MySQL 8.0에선 기본 값, 테이블 락을 사용하지 않아 동시성 수준이 올라가 성능의 이득을 볼 수 있음

### 주의할 점

* auto increment의 값이 연속적으로 증가함을 보장하지 않음
  * statement 기반 복제 방식을 사용할 경우 primary - replica간 컬럼 값이 일치하지 않는 경우가 발생할 수 있음
  *  row 기반의 복제를 이용해야함, MySQL 8.0에서는 row 기반 복제가 기본 값

### 실습

#### `innodb_autoinc_lock_mode` 확인

```mysql
SHOW GLOBAL VARIABLES LIKE 'innodb_autoinc_lock_mode';
```

> * 동적으로 설정 불가
> * MySQL 설정 추가 후 재부팅 필요

