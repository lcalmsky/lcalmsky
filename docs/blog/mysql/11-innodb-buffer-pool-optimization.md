## `Inno DB Buffer Pool` Optimization

* Buffer Pool의 설정 값을 변경하여 MySQL의 성능을 최적화하는 방법
* Buffer Pool의 역할과 관련 설정 변수들을 이용한 최적화 기법 이해

### InnoDB Buffer Pool의 이해

![](https://www.percona.com/blog/wp-content/uploads/2018/04/Untitled-Diagram-2.jpg)

* InnoDB Buffer Pool은 InnoDB 엔진의 데이터와 인덱스를 저장하는 메모리 영역
  * 인덱스 데이터를 캐싱하고 변경된 데이터(dirty pages)를 디스크에 flush
  * 디스크 I/O를 줄여 성능을 향상시키는 역할
* Buffer Pool은 Instance로 나눠져 있음
  * instance 단위로 flush list와 LRU list를 가짐
  * instance의 수가 많을 수록 성능 향상 가능(동시성 수준 증가)
* Chunk는 Buffer Pool의 기본 단위
  * Chunk 단위로 데이터를 읽고 쓰기
  * Chunk 단위로 LRU list에 추가
  * buffer pool의 크기를 조절할 때 chunk 단위로 조절하여 비용을 최소화하기 위해 설계됨
  * 