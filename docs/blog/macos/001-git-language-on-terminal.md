얼마 전 `zsh` 테마를 적용하고나니 `git` 명령어가 한글로 바뀌는 현상이 발생했습니다.

![](https://raw.gitusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-001-01.png)

(원래 맥은 기본으로 `apple git`을 사용하는데 `brew`나 다른 패키지 설치 툴을 이용해 `git`을 다시 설치하는 경우에도 이런 일이 발생할 수 있다고 합니다.)

개인적으로는 원어로 보는 것이 영어 표현에 익숙해 질 수 있기 때문에(영어를 잘 못하더라도) 중요하다고 생각하고, 구글링을 할 때도 영어로 되어있는 편이 검색하는데 더 유리하기 때문에 다시 영어로 출력되도록 바꾸기로 마음을 먹었습니다. 

그래서 이렇게 변하는 이유를 찾아보니 `git`설정에서 `gettext`로 `OS`에서 사용하는 언어를 가져온 뒤 직접 해당 언어를 사용하도록 설정을 업데이트하기 때문이라고 하는데요, 그렇다면 이 설정을 찾아서 바꿔주면 간단히 해결할 수 있겠죠?

먼저 현재 언어 설정을 확인해보면

```shell
❯ echo $LANG
ko_KR.UTF-8
```

역시 한글로 설정되어 있습니다.

저는 예전에 brew로 git을 설치했기 때문에 git을 설치할 때 옵션을 확인해보면,

```shell
brew edit git
```

```shell
.. 생략
  depends_on "gettext"
  depends_on "pcre2"
.. 생략
```

이렇게 `gettext`가 적용되어있는 것을 확인할 수 있습니다.

이 부분을 수정해서 재설치하는 방법이 있긴 한데 뭔가 그러긴 귀찮았고 깔끔하게 해결되지 않는 경우도 많이 보이더군요.

그래서 `zsh`의 설정을 변경하기로 했습니다. (어차피 전 `Terminal`, `iTerm2`, `IntelliJ IDEA`내 터미널에서 모두 `zsh`을 사용하기 때문에..😗)

방법은 아주 간단합니다.

`zsh` 설정을 열어서 `alias`를 하나 등록해주면 되는데 git이 입력될 때 언어 옵션을 같이 주는 방식입니다.

먼저 `.zshrc` 파일을 열어서

```shell
❯ vi ~/.zshrc
```

마지막 줄에 alias 명령어를 추가해줍니다.

```shell
alias git="LANG=en_US.UTF-8 git"
```

그리고 적용을 위해 아래 명령어를 입력합니다. (터미널을 재실행해도 됩니다)

```shell
❯ source ~/.zshrc
```

그리고 다시 `git`을 입력해보면...

![](https://raw.gitusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-macos-001-02.png)

정상적으로 영어로 출력되는 것을 확인할 수 있습니다. 🥳