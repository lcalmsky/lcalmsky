`warp`는 `macOS`에서 `terminal`과 `iterm2`를 대체하기 위한 강력한 터미널 용 `IDE` 입니다.

사실 갈수록 터미널을 사용할 일이 많이 없어지긴 하지만, 아직도 `git` 명령어나 `ssh` 접속을 터미널을 이용하는 경우가 많이 있습니다.

전 알게되자마자 다운받아 사용한지 이제 10일정도 되었는데, 간단한 기능 소개와 함께 사용 후기를 남겨보려고 합니다.

## 설치 방법

### brew

```shell
brew install warp
```

### 직접 다운로드

https://app.warp.dev/get_warp

## 첫인상

> 이게 IDE야 터미널이야?

UI가 정말 이쁘고 기능이 다양합니다.

기존 맥 기본 `terminal`을 사용하시던 분들은 프로파일 기능 등 부족한 부분을 대신하기 위해 `iterm2`을 많이 사용하시는데요, 보통 `oh-my-zsh`을 설치하거나 추가적인 설정이 필요합니다.

하지만 `warp`는 기본이 `zsh`이고 테마도 이쁜 걸로 적용되어 있고, 설정 파일 수정이 아닌 설정 UI를 통해 수정 가능합니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-01.png)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-02.png)

## 기능

### AI Command Search (⌃ + `)

웬만한 기능을 검색 형태로 찾을 수 있습니다.

파일을 복사한다든지, 열려있는 포트를 찾는다는지 등을 검색하면 해당 기능을 수행할 수 있는 `command`를 추천해줍니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-03.png)
![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-04.png)
![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-05.png)
![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-06.png)

### Command Palette (⌘ + p)

제공하는 기능과 해당 기능의 단축키를 한 눈에 볼 수 있습니다. IntelliJ의 Action 검색(⌘ + ⇧ + A)과 비슷하다고 생각하시면 됩니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-07.png)

### Search Command History (⌃ + R)

히스토리 검색 기능을 UI로 제공합니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-08.png)

### Search Workflows

워크플로우 검색 기능을 제공합니다.

워크플로우는 어떤 업무를 하기 위한 명령어들을 모아놓은 것을 말합니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-09.png)

### Workflows

바로 위에서 검색기능을 먼저 설명하였는데, 워크플로우 사용이 매우 간편합니다.

AES256을 이용해 RSA private 키를 생성하는 워크플로우를 실행하면 아래 처럼 명령어를 생성해주고 편리하게 파라미터를 입력할 수 있게 해줍니다. 

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-10.png)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-11.png)

### ETC

이 밖에도 git, gradle, aws 등 자주 사용하는 명령어들에 대한 힌트가 제공되고 힌트가 여러 개일 경우 선택할 수 있는 UI가 제공됩니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-002-12.png)

## 단점

사용을 위해선 가입 및 로그인이 필요합니다.

사용해보면서 느낀 점이, 무료 툴 느낌이 전혀 나질 않습니다. 그 말인 노예로 만든 뒤 유료화를 할 것 같은 느낌이 강력하게 듭니다.

이것을 제외하고는 아직까지는 딱히 단점을 찾지 못했습니다.

---

저는 git CLI를 제외하고는 터미널 명령어에 익숙하지 않은 편이라 아주 편리하게 잘 사용하고 있습니다.

유료만 되지 않는다면(?) 정말 최고의 터미널 앱이 될 거 같은데 향후 행보가 기대되네요!