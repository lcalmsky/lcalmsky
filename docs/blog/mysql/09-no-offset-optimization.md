## `NoOffset` 최적화

* limit 절에 offset을 사용하지 않고 성능을 최적화하는 방법
* offset을 사용하면 성능이 저하되는 이유와 해결 방법 이해

### Offset 사용의 문제점

* `OFFSET`을 사용하면 `LIMIT`에 지정한 개수만큼의 행을 건너뛰고 결과를 반환
* 이 때 지정된 수만큼 건너뛰기 위해 모든 행을 읽어야 하므로 성능 저하 발생

```mysql
select *
from posts
order by id
limit 20 offset 10000;   
```

### NoOffset 최적화 전략

* OFFSET 없이 조회하는 방법: 마지막으로 처리된 레코드의 키를 기준으로 다음 데이터 조회

#### 배치 예시

* as-is: chunk를 처리하고 다음 chunk를 처리할 때 limit 절에 offset 사용
* to-be: 마지막으로 처리된 레코드의 키를 기준으로 다음 데이터 조회

#### 처음 chunk 읽기 작업

```mysql
select *
from my_data
where status = 'active'
order by id
limit 10000;
```

#### 다음 번 chunk 읽기 작업

```mysql
select *
from my_data
where status = 'active'
  and id > 10000 # 마지막으로 처리된 레코드의 id
order by id
limit 10000;
```