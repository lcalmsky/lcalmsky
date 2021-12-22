## Overview

터미널 명령어 중 텍스트 처리에 관련된 명령어를 정리합니다. 다양한 옵션이 존재하지만 실무에서 사용하는 것들 위주로 정리하였습니다.

## sort

### Description

파일의 내용을 정렬하여 화면에 출력합니다.

### Options

* 위치 지정 옵션
  * -k: key를 기준으로 정렬
  * -t: 필드 구분자로 데이터 컬럼을 나눠줌(기본 값은 공백)
* 정렬 기준 옵션
  * -f: 대소문자 무시
  * -g: 일반 숫자 정렬
  * -n: 숫자 정렬
  * -r: 내림차순 정렬
  * -u: 정렬 후 행이 같을 경우 중복 제거

-g와 -n의 차이는 [이곳](https://stackoverflow.com/questions/1255782/whats-the-difference-between-general-numeric-sort-and-numeric-sort-options)에 잘 나와있습니다.

요약하자면 -g는 숫자를 부동 소수점으로 비교하므로 느리고 반올림하는 과정에서 오류가 발생할 수 있습니다. 

### Examples

`sort.txt` 파일을 정렬해보겠습니다.

<details>
<summary>sort.txt 파일 전체 보기</summary>

```text
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
27c50484-850d-4477-9893-1678b993c1b8;William;14
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
```

</details>

#### `sort sort.txt`

```shell
> sort sort.txt
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
27c50484-850d-4477-9893-1678b993c1b8;William;14
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
```

아무 옵션을 주지 않았기 때문에 가장 앞 글자 기준으로 정렬되었습니다.

#### `sort -t ";" -k 3 sort.txt`

```shell
> sort -t ";" -k 3 sort.txt
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
27c50484-850d-4477-9893-1678b993c1b8;William;14
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
```

세미콜론(;)을 기준으로 컬럼을 나눠 3번째 컬럼(나이)을 기준으로 정렬하였는데 문자열 정렬 옵션으로 동작하였습니다.

#### `sort -t ";" -k 3 -n sort.txt`

```shell
> sort -t ";" -k 3 -n sort.txt
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
27c50484-850d-4477-9893-1678b993c1b8;William;14
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
```

위의 예제에 -n 옵션을 추가하여 나이를 정렬하는데 숫자 정렬이 된 것을 확인할 수 있습니다.

#### `sort -t ";" -k 3 -n -r sort.txt`

```shell
> sort -t ";" -k 3 -n -r sort.txt
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
27c50484-850d-4477-9893-1678b993c1b8;William;14
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
```

위의 예제에서 -r 옵션을 추가하여 역정렬 된 것을 확인할 수 있습니다.

#### --debug 

```shell
> sort -t ";" -k 3 -n -r --debug sort.txt
Memory to be used for sorting: 8589934592
Number of CPUs: 8
Using collate rules of ko_KR.UTF-8 locale
Decimal Point: <.>
Thousands separator: <,>
Positive sign: <+>
Negative sign: <->
sort_method=mergesort
; k1=<11>, k2=<16>; s1=<9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16>, s2=<1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11>; cmp1=-1
; k1=<11>, k2=<11>; s1=<1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11>, s2=<e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11>; cmp1=0
; cmp2=52
; k1=<11>, k2=<16>; s1=<9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16>, s2=<e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11>; cmp1=-1
; k1=<7>, k2=<11>; s1=<1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11>, s2=<c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7>; cmp1=-1
; k1=<7>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7>; cmp1=-1
; k1=<17>, k2=<9>; s1=<1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9>, s2=<e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17>; cmp1=1
; k1=<13>, k2=<14>; s1=<27c50484-850d-4477-9893-1678b993c1b8;William;14>, s2=<23493703-dfa7-433f-abd3-dda0b8ded865;David;13>; cmp1=-1
; k1=<17>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17>; cmp1=1
; k1=<9>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9>; cmp1=-1
; k1=<9>, k2=<7>; s1=<122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7>, s2=<1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9>; cmp1=1
; k1=<16>, k2=<14>; s1=<27c50484-850d-4477-9893-1678b993c1b8;William;14>, s2=<9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16>; cmp1=1
; k1=<11>, k2=<14>; s1=<27c50484-850d-4477-9893-1678b993c1b8;William;14>, s2=<e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11>; cmp1=-1
; k1=<11>, k2=<13>; s1=<23493703-dfa7-433f-abd3-dda0b8ded865;David;13>, s2=<e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11>; cmp1=-1
; k1=<16>, k2=<17>; s1=<e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17>, s2=<9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16>; cmp1=-1
; k1=<10>, k2=<16>; s1=<9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16>, s2=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>; cmp1=-1
; k1=<14>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<27c50484-850d-4477-9893-1678b993c1b8;William;14>; cmp1=1
; k1=<13>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<23493703-dfa7-433f-abd3-dda0b8ded865;David;13>; cmp1=1
; k1=<11>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11>; cmp1=1
; k1=<11>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11>; cmp1=1
; k1=<7>, k2=<10>; s1=<afa9061f-cc05-4470-bc28-e77b9a58b517;James;10>, s2=<c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7>; cmp1=-1
; k1=<7>, k2=<9>; s1=<1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9>, s2=<c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7>; cmp1=-1
; k1=<7>, k2=<7>; s1=<c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7>, s2=<122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7>; cmp1=0
; cmp2=-50
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
27c50484-850d-4477-9893-1678b993c1b8;William;14
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
```

--debug 옵션을 사용하면 정렬이 어떻게 동작하였는지 확인할 수 있습니다.

#### 컬럼 지정

```shell
> sort -t ";" -k 2 sort.txt
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
27c50484-850d-4477-9893-1678b993c1b8;William;14
```

-k 2 이런식으로 옵션을 지정하게 되면 두 번째 컬럼 뒷부분을 다 포함하여 정렬됩니다.

```shell
> sort -t ";" -k 2,2 sort.txt
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
27c50484-850d-4477-9893-1678b993c1b8;William;14
```

-k 2,2 는 2번째 컬럼에서 2번째 컬럼까지만 비교하라는 뜻으로 예제 적절하지 않아서 바로 위의 예제 결과와 동일하네요 😅

#### 정렬 우선순위 지정

```shell
> sort -t ";" -k 3,3 -k 2,2 sort.txt
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
27c50484-850d-4477-9893-1678b993c1b8;William;14
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
```

-k 옵션을 여러 개 사용하여 정렬 우선순위를 지정할 수 있습니다.

위의 예시는 나이(문자열)로 정렬한 뒤, 나이가 같으면 이름으로 정렬하고있음을 확인할 수 있습니다.

## uniq

### Description

파일의 내용을 중복된 내용을 제거하고 출력하는 명령어 입니다.

### Options

* -d: 중복된 내용만 출력
* -u: 고유한 내용만 출력
* -i: 대소문자 무시

### Examples

sort.txt 파일을 수정하여 중복된 값을 가지도록 수정하여 uniq.txt 파일을 생성하였습니다.

<details>
<summary>uniq.txt 파일 전체 보기</summary>

```text
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
27c50484-850d-4477-9893-1678b993c1b8;William;14
27c50484-850d-4477-9893-1678b993c1b8;William;14
27c50484-850d-4477-9893-1678b993c1b8;William;14
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
```

</details>

#### `uniq uniq.txt`

```shell
> uniq uniq.txt
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
27c50484-850d-4477-9893-1678b993c1b8;William;14
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7%
```

uniq 명령어를 사용했을 때 기본 옵션(중복제거)으로 동작하였습니다.

#### `uniq -d uniq.txt`

```shell
> uniq -d uniq.txt
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
27c50484-850d-4477-9893-1678b993c1b8;William;14
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
```

-d 옵션을 사용하여 중복된 내용만 출력하였습니다.

#### nl과 같이 사용하기

원래 파일의 길이와 중복 제거한 길이, 중복만 포함한 길이를 각각 출력해보겠습니다.

```shell
> cat uniq.txt | nl
     1	afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
     2	afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
     3	afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
     4	122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
     5	1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
     6	1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
     7	1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
     8	e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
     9	27c50484-850d-4477-9893-1678b993c1b8;William;14
    10	27c50484-850d-4477-9893-1678b993c1b8;William;14
    11	27c50484-850d-4477-9893-1678b993c1b8;William;14
    12	23493703-dfa7-433f-abd3-dda0b8ded865;David;13
    13	9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
    14	9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
    15	9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
    16	1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
    17	e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
    18	e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
    19	e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
    20	c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7%
> uniq uniq.txt| nl
     1	afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
     2	122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
     3	1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
     4	e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
     5	27c50484-850d-4477-9893-1678b993c1b8;William;14
     6	23493703-dfa7-433f-abd3-dda0b8ded865;David;13
     7	9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
     8	1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
     9	e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
    10	c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7%
> uniq -d uniq.txt | nl
     1	afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
     2	1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
     3	27c50484-850d-4477-9893-1678b993c1b8;William;14
     4	9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
     5	e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
```

#### 떨어져있는 중복 데이터

기존 파일에 아래 두 줄을 추가해보겠습니다.

```text
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
9878141a-7dbb-42ce-a198-1a60b97a5fee;richard;16
```

다시 명령어를 실행해보면
```shell
 ~/git-repo/lcalmsky/resources/file/ [feature/2+*] uniq uniq.txt
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
27c50484-850d-4477-9893-1678b993c1b8;William;14
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
9878141a-7dbb-42ce-a198-1a60b97a5fee;richard;16
```

방금 추가한 Richard와 richard가 중복제거되지 않음을 확인할 수 있습니다.

이는 연속된 데이터에 대해서만 중복체크를 하기 때문인데요, 먼저 대소문자를 구별하지 않는 옵션을 사용하여 마지막 richard가 제거되는지 확인해보겠습니다.

#### -i

```shell
 ~/git-repo/lcalmsky/resources/file/ [feature/2+*] uniq -i uniq.txt
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
27c50484-850d-4477-9893-1678b993c1b8;William;14
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
```

richard가 제거된 것을 확인할 수 있습니다.

그럼 연속되지 않은 중복데이터를 제거하는 방법을 알아보겠습니다.

#### uniq with sort

```shell
 ~/git-repo/lcalmsky/resources/file/ [feature/2+*] sort -t ";" -k 2,2 -f uniq.txt | uniq -i
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
27c50484-850d-4477-9893-1678b993c1b8;William;14
```

sort 옵션을 이용해 이름에 해당하는 컬럼 기준으로 대소문자를 무시한 채 정렬하도록 하였고, 정렬된 이후 uniq 옵션을 이용해 역시나 대소문자를 무시하고 중복제거하도록 하여 정상적인 결과를 얻은 것을 확인할 수 있습니다.

반대로 중복된 데이터만 출력해보면

```shell
 ~/git-repo/lcalmsky/resources/file/ [feature/2+*] sort -t ";" -k 2,2 -f uniq.txt | uniq -d
afa9061f-cc05-4470-bc28-e77b9a58b517;James;10
1a72c571-994f-4e2d-bfc0-26927aaa3164;John;9
9878141a-7dbb-42ce-a198-1a60b97a5fee;Richard;16
e3f8a602-b62e-4115-bbd6-0702cdb88afc;Thomas;11
27c50484-850d-4477-9893-1678b993c1b8;William;14
```

역시 정상적으로 동작하는 것을 확인할 수 있습니다.

마지막으로 중복되지 않은 값만 확인해보면,

```shell
 ~/git-repo/lcalmsky/resources/file/ [feature/2+*] sort -t ";" -k 2,2 -f uniq.txt | uniq -u
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
9878141a-7dbb-42ce-a198-1a60b97a5fee;richard;16
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
```

이렇게 중복이 존재하는 행은 아예 삭제된 것을 확인할 수 있고, 여기에 대소문자 무시 옵션까지 추가하면,

```shell
 ~/git-repo/lcalmsky/resources/file/ [feature/2+*] sort -t ";" -k 2,2 -f uniq.txt | uniq -u -i
c10c3715-ee97-42e8-b906-6d50558378f4;Charles;7
23493703-dfa7-433f-abd3-dda0b8ded865;David;13
1ac46560-3642-42bb-81af-ab0d882a7563;Joseph;11
e794eda6-b5c6-4ea8-8f2a-7cb877d4d5d6;Michael;17
122edbb8-bcc0-46d8-8f0c-3fa7b54369e9;Robert;7
```

richard 마저 제거된 것을 확인할 수 있습니다.

---

지금까지 파일 내용을 정렬하고 중복 값을 다루는 방법에 대해 알아보았는데요, 다음 포스팅에서는 텍스트를 자르고 대체하는 방법에 대해 다뤄보겠습니다.