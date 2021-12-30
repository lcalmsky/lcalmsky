## Overview

터미널 명령어 중 텍스트 처리에 관련된 명령어를 정리합니다. 다양한 옵션이 존재하지만 실무에서 사용하는 것들 위주로 정리하였습니다.

## sed

### Description

**s**tream **ed**itor의 약자로 파일 내용을 출력할 때 사용하는 스트림의 내용을 수정하는 명령어 입니다.

### Options

* {RANGE}p: RANGE내의 라인을 출력
* {RANGE}d: RANGE내의 라인을 삭제
* /{PATTERN}/p: 패턴에 매칭되는 라인을 출력
* /{PATTERN}/d: 패턴에 매칭되는 라인을 삭제
* s /{REGEX}/{REPLACEMENT}: 정규표현식에 매칭되는 부분을 REPLACE로 대체

### Examples

#### `tail /etc/passwd | sed -n '1,5p'`

```shell
> tail /etc/passwd | sed -n '1,5p'
_demod:*:275:275:Demo Daemon:/var/empty:/usr/bin/false
_rmd:*:277:277:Remote Management Daemon:/var/db/rmd:/usr/bin/false
_accessoryupdater:*:278:278:Accessory Update Daemon:/var/db/accessoryupdater:/usr/bin/false
_knowledgegraphd:*:279:279:Knowledge Graph Daemon:/var/db/knowledgegraphd:/usr/bin/false
_coreml:*:280:280:CoreML Services:/var/empty:/usr/bin/false
```

`/etc/passwd` 파일의 마지막 10줄을 출력하는데 1~5 번째 라인만 출력합니다.

#### `tail /etc/passwd | sed '1,5d‘`

```shell
> tail /etc/passwd | sed '1,5d'   
_sntpd:*:281:281:SNTP Server Daemon:/var/empty:/usr/bin/false
_trustd:*:282:282:trustd:/var/empty:/usr/bin/false
_darwindaemon:*:284:284:Darwin Daemon:/var/db/darwindaemon:/usr/bin/false
_notification_proxy:*:285:285:Notification Proxy:/var/empty:/usr/bin/false
_oahd:*:441:441:OAH Daemon:/var/empty:/usr/bin/false
```

`/etc/passwd` 파일의 마지막 10줄을 출력하는데 1~5 번째 라인을 제외하고 출력합니다.

#### `tail /etc/passwd | sed -n '/Daemon/p'`

```shell
> tail /etc/passwd | sed -n '/Daemon/p'
_demod:*:275:275:Demo Daemon:/var/empty:/usr/bin/false
_rmd:*:277:277:Remote Management Daemon:/var/db/rmd:/usr/bin/false
_accessoryupdater:*:278:278:Accessory Update Daemon:/var/db/accessoryupdater:/usr/bin/false
_knowledgegraphd:*:279:279:Knowledge Graph Daemon:/var/db/knowledgegraphd:/usr/bin/false
_sntpd:*:281:281:SNTP Server Daemon:/var/empty:/usr/bin/false
_darwindaemon:*:284:284:Darwin Daemon:/var/db/darwindaemon:/usr/bin/false
_oahd:*:441:441:OAH Daemon:/var/empty:/usr/bin/false
```

`/etc/passwd` 파일의 마지막 10줄을 출력하는데 Daemon이 포함된 라인만 출력합니다.

#### `tail /etc/passwd | sed '/empty/d'`

```shell
> tail /etc/passwd | sed '/empty/d' 
_rmd:*:277:277:Remote Management Daemon:/var/db/rmd:/usr/bin/false
_accessoryupdater:*:278:278:Accessory Update Daemon:/var/db/accessoryupdater:/usr/bin/false
_knowledgegraphd:*:279:279:Knowledge Graph Daemon:/var/db/knowledgegraphd:/usr/bin/false
_darwindaemon:*:284:284:Darwin Daemon:/var/db/darwindaemon:/usr/bin/false
```

`/etc/passwd` 파일의 마지막 10줄을 출력하는데 empty가 포함된 라인을 제외하고 출력합니다.

#### `tail /etc/passwd | sed 's/:/=/'`

```shell
> tail /etc/passwd | sed 's/:/=/'
_demod=*:275:275:Demo Daemon:/var/empty:/usr/bin/false
_rmd=*:277:277:Remote Management Daemon:/var/db/rmd:/usr/bin/false
_accessoryupdater=*:278:278:Accessory Update Daemon:/var/db/accessoryupdater:/usr/bin/false
_knowledgegraphd=*:279:279:Knowledge Graph Daemon:/var/db/knowledgegraphd:/usr/bin/false
_coreml=*:280:280:CoreML Services:/var/empty:/usr/bin/false
_sntpd=*:281:281:SNTP Server Daemon:/var/empty:/usr/bin/false
_trustd=*:282:282:trustd:/var/empty:/usr/bin/false
_darwindaemon=*:284:284:Darwin Daemon:/var/db/darwindaemon:/usr/bin/false
_notification_proxy=*:285:285:Notification Proxy:/var/empty:/usr/bin/false
_oahd=*:441:441:OAH Daemon:/var/empty:/usr/bin/false
```

`/etc/passwd` 파일의 마지막 10줄을 출력하는데 콜론(:)을 등호(=)로 대체하여 출력합니다.

이렇게 했을 경우 첫 번째 콜론만 대체되는데 모든 콜론을 대체시키기 위해서는 마지막에 g 옵션을 추가해줘야 합니다.

#### `tail /etc/passwd | sed 's/:/=/g`

```shell
> tail /etc/passwd | sed 's/:/=/g'
_demod=*=275=275=Demo Daemon=/var/empty=/usr/bin/false
_rmd=*=277=277=Remote Management Daemon=/var/db/rmd=/usr/bin/false
_accessoryupdater=*=278=278=Accessory Update Daemon=/var/db/accessoryupdater=/usr/bin/false
_knowledgegraphd=*=279=279=Knowledge Graph Daemon=/var/db/knowledgegraphd=/usr/bin/false
_coreml=*=280=280=CoreML Services=/var/empty=/usr/bin/false
_sntpd=*=281=281=SNTP Server Daemon=/var/empty=/usr/bin/false
_trustd=*=282=282=trustd=/var/empty=/usr/bin/false
_darwindaemon=*=284=284=Darwin Daemon=/var/db/darwindaemon=/usr/bin/false
_notification_proxy=*=285=285=Notification Proxy=/var/empty=/usr/bin/false
_oahd=*=441=441=OAH Daemon=/var/empty=/usr/bin/false
```

위에서 설명한 것 처럼 모든 콜론이 등호로 대체되었습니다.

#### 복합 옵션 사용 `tail /etc/passwd | sed '1,5 s/:/=/g'`

```shell
> tail /etc/passwd | sed '1,5 s/:/=/g'
_demod=*=275=275=Demo Daemon=/var/empty=/usr/bin/false
_rmd=*=277=277=Remote Management Daemon=/var/db/rmd=/usr/bin/false
_accessoryupdater=*=278=278=Accessory Update Daemon=/var/db/accessoryupdater=/usr/bin/false
_knowledgegraphd=*=279=279=Knowledge Graph Daemon=/var/db/knowledgegraphd=/usr/bin/false
_coreml=*=280=280=CoreML Services=/var/empty=/usr/bin/false
_sntpd:*:281:281:SNTP Server Daemon:/var/empty:/usr/bin/false
_trustd:*:282:282:trustd:/var/empty:/usr/bin/false
_darwindaemon:*:284:284:Darwin Daemon:/var/db/darwindaemon:/usr/bin/false
_notification_proxy:*:285:285:Notification Proxy:/var/empty:/usr/bin/false
_oahd:*:441:441:OAH Daemon:/var/empty:/usr/bin/false
```

`/etc/passwd` 파일의 마지막 10줄을 출력하는데 1~5번째 줄만 콜론(:)을 등호(=)로 대체하여 출력합니다.

#### `tail /etc/passwd | sed -n '/Demo/,5p'`

```shell
> tail /etc/passwd | sed -n '/Demo/,5p'
_demod:*:275:275:Demo Daemon:/var/empty:/usr/bin/false
_rmd:*:277:277:Remote Management Daemon:/var/db/rmd:/usr/bin/false
_accessoryupdater:*:278:278:Accessory Update Daemon:/var/db/accessoryupdater:/usr/bin/false
_knowledgegraphd:*:279:279:Knowledge Graph Daemon:/var/db/knowledgegraphd:/usr/bin/false
_coreml:*:280:280:CoreML Services:/var/empty:/usr/bin/false
```

`/etc/passwd` 파일의 마지막 10줄을 출력하는데 Demo가 포함된 라인부터 5줄을 출력합니다.

`,+5`로 지정할 경우 상대적으로 검색된 대상부터 5줄을 추가로 출력해줍니다.

```shell
> tail /etc/passwd | sed -n '/Demo/,+5p'
_demod:*:275:275:Demo Daemon:/var/empty:/usr/bin/false
_rmd:*:277:277:Remote Management Daemon:/var/db/rmd:/usr/bin/false
_accessoryupdater:*:278:278:Accessory Update Daemon:/var/db/accessoryupdater:/usr/bin/false
_knowledgegraphd:*:279:279:Knowledge Graph Daemon:/var/db/knowledgegraphd:/usr/bin/false
_coreml:*:280:280:CoreML Services:/var/empty:/usr/bin/false
_sntpd:*:281:281:SNTP Server Daemon:/var/empty:/usr/bin/false
```

## awk

### Description

텍스트를 처리하는 스크립트 언어이고 '오크'라고 읽습니다.

스크립트이다 보니 옵션이 너무 방대하고 기본 적인 문법을 알아야 합니다.

```shell
awk options 'selection _creteria {action}' input-file
```

```shell
usage: awk [-F fs] [-v var=value] [-f progfile | 'prog'] [file ...]
```

자주 사용되는 옵션은 `-F`로 파일 구분자를 지정할 때 사용합니다.

awk를 사용할 때 내장되어있는 변수를 이용해야 하는데 주요 변수는 아래와 같습니다.

* $1, $2, ... $N: N번째 필드
* NR: Number of Records
* NF: Number of Fields
* FS: File Separator (기본 공백)
* RS: Record Separator (기본 개행문자)
* OFS: Output Field Separator
* ORS: Output Record Separator

### Examples

#### `wc /etc/passwd | awk '{ print $1 }`

```shell
 ~/git-repo/lcalmsky/ [feature/4+*] wc /etc/passwd
     123     329    7868 /etc/passwd
 ~/git-repo/lcalmsky/ [feature/4+*] wc /etc/passwd | awk '{ print $1 }'
123
```

wc를 이용해 파일의 라인 수, 단어 수, 바이트 수를 출력할 수 있는데 awk를 이용해 첫 번째 필드(공백 기준)만 출력하여 라인 수만 획득할 수 있습니다.

```shell
 ~/git-repo/lcalmsky/ [feature/4+*] wc /etc/passwd | awk '{ print $2 }'
329
 ~/git-repo/lcalmsky/ [feature/4+*] wc /etc/passwd | awk '{ print $3 }'
7868
```

마찬가지로 두 번째, 세 번째 필드의 값을 가져올 수도 있습니다.

#### `tail /etc/passwd | awk -F: '{ print $1 }'`

```shell
 ~/git-repo/lcalmsky/ [feature/4+*] tail /etc/passwd | awk -F: '{ print $1 }'
_demod
_rmd
_accessoryupdater
_knowledgegraphd
_coreml
_sntpd
_trustd
_darwindaemon
_notification_proxy
_oahd
```

/etc/passwd 파일의 마지막 10줄을 출력하면서 콜론 기준으로 첫 번째 필드를 출력할 수 있습니다.

#### tail /etc/passwd | awk -F: '/c/ { print $1 }'

```shell 
 ~/git-repo/lcalmsky/ [feature/4+*] tail /etc/passwd | awk -F: '/c/ { print $1 }'
_accessoryupdater
_coreml
_notification_proxy
```

/etc/passwd 파일의 마지막 10줄을 출력하면서 콜론 기준으로 첫 번째 필드를 가져오는데 'c'가 포함된 줄만 출력할 수 있습니다.

#### tail /etc/passwd | awk -F: '/c/ { print NR, $1 }'

```shell
 ~/git-repo/lcalmsky/ [feature/4+*] tail /etc/passwd | awk -F: '/c/ { print NR, $1 }'
3 _accessoryupdater
5 _coreml
9 _notification_proxy
```

내장 변수 NR과 같이 사용하여 Number of Record를 같이 출력할 수 있습니다.

NF를 이용하면 몇 개의 필드를 가지고 있는지 확인할 수 있습니다.

```shell
 ~/git-repo/lcalmsky/ [feature/4+*] tail /etc/passwd | awk -F: '/c/ { print NF, $1 }'
7 _accessoryupdater
7 _coreml
7 _notification_proxy
```

#### tail /etc/passwd | awk -F: '/c/ { print NR " -> " $1 }'

```shell
 ~/git-repo/lcalmsky/ [feature/4+*] tail /etc/passwd | awk -F: '/c/ { print NR " -> " $1 }'
3 -> _accessoryupdater
5 -> _coreml
9 -> _notification_proxy
```

print 뒤에 나오는 부분은 모두 출력되는 것이기 때문에 문자열을 얼마든지 추가할 수 있습니다.

```shell
 ~/git-repo/lcalmsky/ [feature/4+*] tail /etc/passwd | awk -F: '/c/ { print NR "("NF")" " -> " $1 }'
3(7) -> _accessoryupdater
5(7) -> _coreml
9(7) -> _notification_proxy
```