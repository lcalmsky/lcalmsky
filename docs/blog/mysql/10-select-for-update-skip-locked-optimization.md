## `SELECT FOR UPDATE SKIP LOCKED` 최적화

* `SELECT FOR UPDATE` 문에 `SKIP LOCKED`를 사용하여 성능을 최적화 하는 방법
* `SKIP LOCKED`의 개념과 이를 사용하여 동시성을 향상시키는 방법 이해

### `SELECT FOR UPDATE`와 `SKIP LOCKED`의 이해

* `SELECT FOR UPDATE`는 트랜잭션 내에서 특정 행을 읽고 잠금을 설정하는 명령어
  * 다른 트랜잭션이 해당 행을 읽거나 수정하지 못하도록 함
* `SKIP LOCKED`는 `SELECT FOR UPDATE`문에 옵션으로 사용할 수 있는 기능
  * 잠금이 걸린 레코드는 건너뛰고 잠금이 걸리지 않은 레코드만 조회하고 잠금

### 최적화 전략

* 동시성 높은 환경에서 성능 개선에 유리(e.g. 선착순 예약, 주문 처리)
  * 잠금 대기시간 감소 및 처리량 증가
* NOWAIT 옵션과 함께 사용하여 데드락 방지
  * NOWAIT 옵션은 잠금이 걸린 경우 즉시 에러 반환
  * 이 설정을 사용하지 않으면 InnoDB Lock Wait Timeout 시간만큼 대기

### 실습

* 선착순 예약 시스템 구현을 통해 `SELECT FOR UPDATE SKIP LOCKED` 최적화 확인
* 사용자가 예약하면 예약되지 않은 행을 조회하고 잠금을 설정

#### 예약 테이블

```mysql
create table reservations
(
    id      int primary key auto_increment,
    user_id int not null,
    status  enum ('available', 'reserved', 'canceled') default 'available',
    created_at datetime default current_timestamp,
    updated_at datetime default current_timestamp on update current_timestamp,
    expires_at datetime
);
```

#### 예약 처리(기존 방식)

```mysql
select id
from reservations
where status = 'available'
order by id
limit 1
for
update;
```

```mysql
update reservations
set status = 'reserved', user_id = ?
where id = ?;
```

> 동시에 여러 요청이 발생할 시간이 매우 길게 소요됨

#### 예약 처리(최적화)

```mysql
select id
from reservations
where status = 'available'
order by id
limit 1
for
update
skip locked;
```

> `SKIP LOCKED`를 사용하여 잠금이 걸린 행은 건너뛰고 잠금이 걸리지 않은 행만 조회하여 성능 향상