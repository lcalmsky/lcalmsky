## ORDER BY를 통한 최적화

인덱스의 유무에 따른 ORDER BY의 내부 동작 원리와 이를 이용한 최적화 기법에 대해 이해하는 게 중요합니다.

### ORDER의 이해

`ORDER BY` 절에 명시된 컬럼에 인덱스가 있으면 MySQL은 이미 정렬된 데이터를 사용하므로 추가적인 정렬 작업이 필요 없습니다.

인덱스가 없는 경우, `filesort` 방식으로 임시 테이블을 사용해 정렬 작업을 수행합니다.

`filesort`는 다음과 같은 상황에서 발생할 수 있습니다.

1. `ORDER BY` 절이 인덱스와 일치하지 않는 경우: `ORDER BY` 절에서 사용된 컬럼이 인덱스에 포함되어 있지 않거나, 인덱스의 순서와 일치하지 않는 경우 `filesort`가 발생할 수 있습니다.
1. 임시 테이블 사용: 정렬 작업을 위해 MySQL이 임시 테이블을 생성해야 하는 경우 `filesort`가 발생할 수 있습니다. 이 경우 정렬된 결과가 임시 파일에 기록되고, 이 파일을 읽어서 최종 결과를 생성합니다.
1. 정렬 버퍼 초과: MySQL이 정렬을 위한 버퍼 크기를 초과하는 경우에도 `filesort`가 발생할 수 있습니다.

`ORDER BY` 절이 인덱스를 사용하는지 `filesort`를 사용하는지 여부를 확인하기 위해 `EXPLAIN` 명령어를 사용할 수 있습니다.

* 실행 계획의 Extra 컬럼에 Using filesort가 있으면 `filesort` 사용
* 실행 계획의 key 컬럼에 인덱스가 있으면 인덱스 사용

### 최적화 방법

기본적으로 `ORDER BY` 절을 사용하는 경우 인덱스를 사용하도록 하는 것이 좋습니다.

특히 `LIMIT` 절과 함께 사용될 때 유의미한데, 인덱스가 있으면 MySQL은 필요한 만큼의 데이터만 읽고나서 처리를 중단하지만 인덱스가 없으면 전체 데이터를 정렬한 후 원하는 행의 수만큼 추출합니다.

`filesort`를 이용하고 있는 경우 최적화 할 수 있는 방법은 아래와 같습니다.

1. `sort_buffer_size` 튜닝
   - `filesort`는 임시 테이블에 기록하고 `Sort Buffer`를 통해 정렬을 수행
   - 정렬해야하는 데이터가 많다면 성능이 잘 나오지 않음(디스크 정렬)
   - `sort_buffer_size`를 증가시키면 디스크 정렬을 최소화할 수 있음
   - `sort_merge_passes` 변수를 통해 디스크 정렬 수행 여부를 알 수 있음, 정렬 작업 후 값이 증가한 경우 디스크 정렬을 수행했다는 
   - 윈도우 환경에선 동시에 정렬작업을 처리할 때 메모리 할당이 효율적으로 작동하지 않아 효과적이지 않을 수 있음
2. 정렬 동작방식 튜닝(`Single-Pass`에서 `Two-Pass`로)
   - `filesort는` 두 가지 방식으로 정렬 수행
   - `Single-Pass`는 `Sort Buffer`에 데이터를 모두 넣어서 정렬 수행
   - `Two-Pass`는 정렬하는 컬럼과 PK만 넣어서 정렬 수행 후 나머지 데이터와 병합
   - 일반적으로 `Single-Pass`가 성능이 좋으나 데이터의 크기가 큰 경우 `Two-Pass`가 우위
   - `Two-Pass`를 사용할 땐 `select`에 필요한 컬럼만 명시하는 게 성능에 도움이 됨 
   - `max_length_for_sort_data` 변수 값을 조절(낮춰서)하여 `Two-Pass` 방식으로 변경 가능
   - `max_length_for_sort_data` 값보다 데이터가 많으면 Two-Pass, 적으면 Single-Pass 방식으로 동작
   - MySQL 8.0.2 이전 버전에서만 동작, 이후 버전은 `Optimizer`가 알아서 수행
3. 문자열 정렬 튜닝
   - 문자열 컬럼의 값 전체를 사용해서 정렬하지 않고 `max_sort_length` 값 만큼만 잘라서 정렬하도록 수행
   - `max_sort_length` 값을 줄이면 정렬 결과가 빨라질 수 있으나 정렬 결과에 영향을 줄 수 있음

### 고려할 점

`ORDER BY` 절을 사용했을 때 정렬 결과가 항상 동일해야 한다면 `ORDER BY` 절에 고유한 값을 가진 컬럼을 포함시켜야 합니다.

만약 여러 컬럼을 명시하지 않고 카디널리티가 상대적으로 낮은 컬럼으로 정렬을 해야하는 경우, ORDER BY 절에 PK를 추가하여 매번 같은 결과를 얻을 수 있습니다.

ORDER BY절에 인덱스를 이용하고 있다면 PK를 추가하는 것은 성능에 영향을 주지 않습니다. 모든 보조 인덱스(secondary index)는 실제 테이블 레코드에 접근하기 위해 내부적으로 PK를 가지고있기 때문입니다.

### 실습

#### sort_buffer_size 확인

```mysql
SHOW VARIABLES LIKE 'sort_buffer_size';
```

#### sort_buffer_siez 변경

```mysql
SET SESSION sort_buffer_size = 2 * 262144;
```

> global 단위와 session 단위 변경이 모두 가능하지만 되도록이면 session 단위로 설정하는 게 좋습니다. global 단위로 변경하면 메모리 사용량이 전체적으로 증가할 수 있어 성능에 더 안 좋은 영향을 미칠 수 있습니다.    
> session: 데이터베이스에 연결하는 커넥션 단위로 설정  
> global: 전체 커넥션 단위로 설정  

#### sort_merge_passes 확인

```mysql
SHOW STATUS LIKE 'sort_merge_passes';
```

