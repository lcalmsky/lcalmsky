## `INSERT` 최적화

`INSERT` 문을 최적화하는 방법과 그 중 ``LOAD DATA`` 문에 대해 이해하는 것이 중요합니다.

### `INSERT` 처리 비용

* 연결 비용(Connecting): MySQL 서버와의 연결 비용(3)
* 쿼리 전송(Sending Query to Server): 서버로 쿼리를 보내는 비용(2)
* 쿼리 파싱(ParsingQuery): 서버에서 쿼리를 분석하는 비용(2)
* 행 삽입(Inserting Row): 데이터 행을 삽입하는 비용(1 * size of row)
* 인덱스 삽입(Inserting Indexes): 각 인덱스에 대한 삽입 비용(1 * number of indexes)
* 연결 정료(Closing): 연결을 종료하는 비용(1)

### 최적화 방법

* 대량 삽입(`Bulk Insert`): 여러 번의 네트워크 통신을 줄이고 한 번에 많은 데이터를 전송
  * `INSERT` 문 개선: 단일 `INSERT` 문에 여러 값(MULTIPLE VALUES) 삽입으로 처리 속도 개선
  * `LOAD DATA` 활용: 파일에서 직접 데이터를 불러오는 기능으로 처리 속도가 훨씬 빠름
* 일관성 검사 지연(Consistency Check Delay): 일관성 검사(외래키나 유니크키 등)를 미루어 처리 속도 향상, 검증된 데이터에만 사용 가능
* 기본 값을 넣을 땐 `INSERT`시 생략: `INSERT` 문 파싱 속도를 더 빠르게 할 수 있음

### `LOAD DATA`란

* 스토리지 엔진에서 지원해주는 `Bulk Insert` 기능
* `INSERT` 문보다 처리 속도가 20배 더 빠름
* `INSERT` 문과 달리 데이터가 들어있는 File을 전달

### `LOAD DATA` 사용시 주의사항

* `auto commit` 하지 않도록 변경
  * `INSERT` 할 때마다 `redo log`가 `flush`됨
* 병렬 처리 고려
  * `LOAD DATA` 문은 단일 스레드, 단일 트랜잭션으로 처리
  * 트랜잭션이 처리되는 동안 `undo log`를 지울 수 없는 문제 발생
  * 이를 방지하기 위해 하나의 큰 파일보다 여러 작은 파일을 병렬로 처리할 수 있음
  * 트랜잭션이 완료되는 시점이 각각 달라 데이터 가시성으로 인한 문제가 발생하지 않는지 고려해야함
* 데이터 파일을 전송하는 경우 보안 설정 필요
  * `LOAD DATA` 문은 MySQL 서버가 파일로 가지고 있어야 실행할 수 있음
  * 클라이언트 쪽에서 파일을 가지고 있는 경우 `LOAD DATA` LOCAL을 실행해야함
  * 파일을 전송하기 위해서는 서버/클라이언트 모두 `local_infile=ON` 설정을 활성화 해야함

### `LOAD DATA` 최적화

* 파일 안의 데이터들을 PK 순으로 정렬
  * 2배 정도 빨라짐
  * PK는 클러스터링 인덱스라 값이 유사한 경우 데이터들이 서로 인접한 곳에 저장됨
  * 디스크 linear IO를 통해 삽입하여 효율적
* `UNIQUE KEY Constraint`를 비활성화
  * 일관성이 깨질 위험이 있다면 사용하면 안 됨
* `FOREIGN KEY Constraint`를 비활성화
  * 일관성이 깨질 위험이 있다면 사용하면 안 됨
* `auto_increment_lock_mode=2`로 설정

### 실습

#### local_infile 설정 조회

```mysql
SHOW GLOBAL VARIABLES LIKE 'local_infile';
```

> * 동적으로 설정 불가
> * MySQL 설정 추가 후 재부팅 필요

#### LOAD DATA 사용

```mysql
LOAD DATA LOCAL INFILE '{file_name}'
INTO TABLE {table_name}
FIELDS TERMINATED BY '{delimiter}'
ENCLOSED BY '{suffix}'
LINES TERMINATED BY '\\n';
```