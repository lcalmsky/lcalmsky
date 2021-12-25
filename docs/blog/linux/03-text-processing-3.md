## Overview

터미널 명령어 중 텍스트 처리에 관련된 명령어를 정리합니다. 다양한 옵션이 존재하지만 실무에서 사용하는 것들 위주로 정리하였습니다.

## cut

### Description

파일 내용을 컬럼을 기준으로 잘라내 출력합니다.

### Options

* -b: byte 기준으로 잘라냄
* -c: character 기준으로 잘라냄
* -f: 필드 기준으로 잘라냄
* -d: delimiter 지정
* --complement: 선택을 반전시킴
* --output-delimiter=<>: 지정한 구분자와 함께 출력

### Examples

#### `-f`, `-d`

```shell
> wc -l /etc/passwd                   
     123 /etc/passwd
> wc -l /etc/passwd | cut -f 6 -d ' '
123
```

[이전 포스팅](https://jaime-note.tistory.com/178)에서 다뤘던 wc를 -l 옵션을 이용해 라인수만 나오도록 출력한 뒤 cut과 같이 사용하여 라인수만 획득하였습니다.

-f를 이용해 6번째 필드를 지정하였고 -d로 delimiter를 공백으로 지정하여 6번째 칸에 해당하는 123을 가져옵니다.

#### 여러 컬럼 지정

```shell
> tail /etc/passwd
_demod:*:275:275:Demo Daemon:/var/empty:/usr/bin/false
_rmd:*:277:277:Remote Management Daemon:/var/db/rmd:/usr/bin/false
_accessoryupdater:*:278:278:Accessory Update Daemon:/var/db/accessoryupdater:/usr/bin/false
_knowledgegraphd:*:279:279:Knowledge Graph Daemon:/var/db/knowledgegraphd:/usr/bin/false
_coreml:*:280:280:CoreML Services:/var/empty:/usr/bin/false
_sntpd:*:281:281:SNTP Server Daemon:/var/empty:/usr/bin/false
_trustd:*:282:282:trustd:/var/empty:/usr/bin/false
_darwindaemon:*:284:284:Darwin Daemon:/var/db/darwindaemon:/usr/bin/false
_notification_proxy:*:285:285:Notification Proxy:/var/empty:/usr/bin/false
_oahd:*:441:441:OAH Daemon:/var/empty:/usr/bin/false

> tail /etc/passwd | cut -d ":" -f 1,2 
_demod:*
_rmd:*
_accessoryupdater:*
_knowledgegraphd:*
_coreml:*
_sntpd:*
_trustd:*
_darwindaemon:*
_notification_proxy:*
_oahd:*
```

tail을 이용하여 파일의 마지막을 출력했고, 콜론(:)으로 파싱하여 첫 번째와 두 번째 컬럼의 값을 출력하기 위해 -f 옵션 뒤에 콤마로 구분된 두 개의 인자를 전달하였습니다.  

#### `-b`

특정 디렉토리의 권한만 출력하고 싶을 때는 바이트 기준으로 잘라주시면 됩니다.

```shell
> ls -al | cut -b 1-10
total 40
drwxr-xr-x
drwxr-xr-x
-rw-r--r--
drwxr-xr-x
-rw-r--r--
drwxr-xr-x
-rw-r--r--
drwxr-xr-x
drwxr-xr-x
drwxr-xr-x
```

1-10은 1byte에서 10byte까지를, -10도 역시 마찬가지로 1~10 byte를, 10-는 10byte 이후부터 끝까지를 잘라줍니다.

```shell
> ls -al | cut -b -10 
total 40
drwxr-xr-x
drwxr-xr-x
-rw-r--r--
drwxr-xr-x
-rw-r--r--
drwxr-xr-x
-rw-r--r--
drwxr-xr-x
drwxr-xr-x
drwxr-xr-x
```

```shell
> ls -al | cut -b 10-
x  10 jaime  staff   320 Dec 10 08:39 .
x  44 jaime  staff  1408 Dec  4 02:14 ..
-@  1 jaime  staff  6148 Aug  4 02:48 .DS_Store
x  15 jaime  staff   480 Dec 26 02:54 .git
-   1 jaime  staff    18 Aug  5 01:18 .gitignore
x  12 jaime  staff   384 Dec 26 03:09 .idea
-   1 jaime  staff  5597 Dec 10 08:39 README.md
x   4 jaime  staff   128 Aug  3 00:24 docs
x   4 jaime  staff   128 Jul 10 11:58 images
x   6 jaime  staff   192 Dec 18 03:46 resources
```

## tr

### Description

파일의 내용을 변환하여 출력합니다.

```shell
usage: tr [-Ccsu] string1 string2
       tr [-Ccu] -d string1
       tr [-Ccu] -s string1
       tr [-Ccu] -ds string1 string2
```

### Options

* -c, -C, --complement: 선택 반전
* -d, --delete: 인자에 해당하는 부분을 지워줌
* string1, string2, ...
  * char1-char2: char1부터 char2까지
  * [:alnum:]: 문자 + 숫자
  * [:alpha:]: 문자
  * [:blank:]: 공백
  * [:space:]: 공백 + newline
  * [:digit:]: 10진수
  * [:xdigit:]: 16진수
  * [:lower:]: 소문자
  * [:upper:]: 대문자

### Examples

#### `-d`

```shell
> tail /etc/passwd | tr -d ':' 
_demod*275275Demo Daemon/var/empty/usr/bin/false
_rmd*277277Remote Management Daemon/var/db/rmd/usr/bin/false
_accessoryupdater*278278Accessory Update Daemon/var/db/accessoryupdater/usr/bin/false
_knowledgegraphd*279279Knowledge Graph Daemon/var/db/knowledgegraphd/usr/bin/false
_coreml*280280CoreML Services/var/empty/usr/bin/false
_sntpd*281281SNTP Server Daemon/var/empty/usr/bin/false
_trustd*282282trustd/var/empty/usr/bin/false
_darwindaemon*284284Darwin Daemon/var/db/darwindaemon/usr/bin/false
_notification_proxy*285285Notification Proxy/var/empty/usr/bin/false
_oahd*441441OAH Daemon/var/empty/usr/bin/false
```

`-d` 옵션을 사용하여 콜론을 모두 삭제하였습니다.

#### `string1 string2`

```shell
> tail /etc/passwd | tr ':' '/'
_demod/*/275/275/Demo Daemon//var/empty//usr/bin/false
_rmd/*/277/277/Remote Management Daemon//var/db/rmd//usr/bin/false
_accessoryupdater/*/278/278/Accessory Update Daemon//var/db/accessoryupdater//usr/bin/false
_knowledgegraphd/*/279/279/Knowledge Graph Daemon//var/db/knowledgegraphd//usr/bin/false
_coreml/*/280/280/CoreML Services//var/empty//usr/bin/false
_sntpd/*/281/281/SNTP Server Daemon//var/empty//usr/bin/false
_trustd/*/282/282/trustd//var/empty//usr/bin/false
_darwindaemon/*/284/284/Darwin Daemon//var/db/darwindaemon//usr/bin/false
_notification_proxy/*/285/285/Notification Proxy//var/empty//usr/bin/false
_oahd/*/441/441/OAH Daemon//var/empty//usr/bin/false
```

tr 뒤에 두 개의 인자를 주어 앞 인자에 해당하는 문자열을 뒷 인자에 해당하는 문자열로 치환하였습니다.

#### `[:lower:] [:upper:]`

```shell
> tail /etc/passwd | tr '[:lower:]' '[:upper:]'
_DEMOD:*:275:275:DEMO DAEMON:/VAR/EMPTY:/USR/BIN/FALSE
_RMD:*:277:277:REMOTE MANAGEMENT DAEMON:/VAR/DB/RMD:/USR/BIN/FALSE
_ACCESSORYUPDATER:*:278:278:ACCESSORY UPDATE DAEMON:/VAR/DB/ACCESSORYUPDATER:/USR/BIN/FALSE
_KNOWLEDGEGRAPHD:*:279:279:KNOWLEDGE GRAPH DAEMON:/VAR/DB/KNOWLEDGEGRAPHD:/USR/BIN/FALSE
_COREML:*:280:280:COREML SERVICES:/VAR/EMPTY:/USR/BIN/FALSE
_SNTPD:*:281:281:SNTP SERVER DAEMON:/VAR/EMPTY:/USR/BIN/FALSE
_TRUSTD:*:282:282:TRUSTD:/VAR/EMPTY:/USR/BIN/FALSE
_DARWINDAEMON:*:284:284:DARWIN DAEMON:/VAR/DB/DARWINDAEMON:/USR/BIN/FALSE
_NOTIFICATION_PROXY:*:285:285:NOTIFICATION PROXY:/VAR/EMPTY:/USR/BIN/FALSE
_OAHD:*:441:441:OAH DAEMON:/VAR/EMPTY:/USR/BIN/FALSE
```

소문자를 대문자로 치환하였습니다.