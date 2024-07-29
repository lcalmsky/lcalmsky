## `InnoDB Deadlock Detection` 최적화

* innoDB 데드락 감지 기능을 비활성화하여 MySQL 성능 최적화

### 데드락 감지의 이해

* `select ... for update`, `insert`, `update`, `delete` 등 `Lock`을 요구하는 연산은 데드락 감지 스레드를 통해 데드락을 유발하는지 주기적으로 검사
* 많은 락을 요구하는 서비스 스레드가 많아지면 데드락 감지가 지연되면서 락을 사용하는 스레드 전체에 지연 발생 가능
* 데드락 감지 스레드는 데드락 감지 작업을 수행하는 동안 서비스 스레드들이 소유한 락의 상태를 변경하지 못하게 하기 위해 락 테이블에 락을 거는데 데드락 감지 작업이 끝나지 않으면 락을 가진 스레드들은 락을 반납하지 못해 성능 이슈 유발

### 최적화 전략

* `innodb_deadlock_detect` 변수를 `OFF`로 설정하여 데드락 감지 기능 비활성화(락을 요구하는 작업이 많은 경우)
  * 비활성화하면 데드락이 발생할 수 있으므로 innodb_lock_wait_timeout 변수를 적절히 설정하여 락 대기 시간을 제한(기본 값: 50s)
  * 환경에 따른 타임아웃 값 예시
    * 온라인 트랜잭션 처리 환경: 낮은 값이 유리
    * 데이터 웨어하우스: 높은 값이 유리

#### 락을 기다리는 트랜잭션의 유무 확인하는 방법
* `Performance Schema`
  * `Wait Event` 테이블: 락 테이블이나 I/O 작업을 통해서 대기하고 있는 스레드에 대한 정보 확인 가능
  * 락 테이블: 잠금 점유 여부, 잠금 가디리고있는 정보 확인 가능
* `InnoDB status`
  * innoDB 내부 상태를 통해 잠금을 기다리는 트랜잭션 확인 가능

### 실습

#### performance schema 확인

```mysql
SHOW VARIABLES LIKE 'performance_schema'
```

> 기본 값이 `ON`

#### 데드락 감지 비활성화

```mysql
SET GLOBAL innodb_deadlock_detect = OFF;
```

#### 데드락 감지 비활성화 확인

```mysql
SHOW VARIABLES LIKE 'innodb_deadlock_detect'
```