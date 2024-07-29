## `Prefix Index` 최적화

* `Prefix Index`를 이용한 성능 최적화 방법

### `Prefix Index`의 이해

* 칼럼의 전체 값 대신 일부 접두만을 이용하여 인덱스를 구축하는 방법
* `BLOB`, `TEXT`, 긴 길이의 `VARCHAR` 타입 컬럼 같이 인덱스 크기가 커서 전체 인덱싱이 어려운 경우 사용
* `Prefix Index`를 이용하면 인덱스 크기를 줄이고 성능을 향상시킬 수 있으나 정확도는 떨어질 수 있음
* `REDUNDANT` or `COMPACT` row 포맷인 경우 최대 767 바이트까지 사용 가능
  * MySQL 5.5부터는 `COMPACT`가 기본
* `DYNAMIC` or `COMPRESSED` row 포맷인 경우 최대 3072 바이트까지 사용 가능

### `Prefix Index` 최적화

* 카디널리티(cardinality)가 충분히 높아지는 접두사 선택
  * 접두가 길이가 적은 경우 인덱스 공간을 줄일 수 있으나 카디널리티가 충분히 높아지지 않아 인덱스 스캔이 효율적으로 동작하지 않을 수 있음
  * 접두사 길이를 실험적으로 증가/감소시켜가며 유의미한 카디널리티를 가지는 길이를 찾아 `Prefix Index`로 설계해야함

### 실습

* 카디널리티를 고려해 `Prefix Index` 설계

#### 시나리오

* 세계 도시 데이터를 가지고 **도시 이름**으로 검색하기 쉽게 인덱스 구축

#### 테이블 생성

```mysql
create table word_cities
(
    city       varchar(100),
    city_ascii varchar(100),
    lat        decimal(10, 8),
    lng        decimal(11, 8),
    country    varchar(100),
    iso2       varchar(2),
    iso3       varchar(3),
    admin_name varchar(100),
    capital    varchar(50),
    population bigint,
    id         int primary key
);
```

#### 중복 데이터 확인

```mysql
select count(*) as count, city
from world_cities
group by city
order by count desc
limit 10;
```

> city 컬럼에 해당하는 데이터의 전체 길이에 해당하는 중복 데이터 확인 가능

```mysql
select count(*) as count, left(city, 1) as prefix
from world_cities
group by prefix
order by count desc
limit 10;
```

> city 컬럼의 접두사의 길이가 1인 데이터의 중복 데이터 확인 가능  
> left 함수를 이용하여 접두사의 길이를 조절하여 실험적으로 카디널리티를 확인  
> 접두사의 길이를 증가시킬 수록 중복 데이터가 줄어드는데 큰 차이가 없는 구간이 발견되면 해당 길이로 설정

#### `Prefix Index` 생성

```mysql
create index idx_city_prefix on world_cities (city(8));
```