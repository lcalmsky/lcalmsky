## Global User

git을 사용하기 위해서는 이름과 이메일 주소를 설정해야 합니다. 이 정보가 없으면 커밋을 생성할 수 없습니다.

global user란 로컬 git 전체에 설정되는 사용자 정보를 말합니다.

아래 명령어를 통해 설정할 수 있습니다.

```shell
> git config --global user.name "jaime"
> git config --global user.email "lcalmsky@gmail.com"
```

잘 설정되었는지 확인하기 위해서는 아래 명령어를 사용합니다.

```shell
> git config --list
user.name=Jungmin Lee
user.email=jaime.lee@jobis.co
```

## Repository별 Author 설정

특정 Repository에서 다른 작성자로 기여해야 하는 경우 해당 경로에서 --global 옵션을 제거하고 설정하면 됩니다.

```shell
> cd <path-to-repository>
> git config user.name "Jungmin Lee"
> git config user.email "jungmin.lee@company.co.kr"
```

회사에서 github을 사용하고 같은 계정에 이메일만 분리해놨을 경우, 또는 집에서 PC 또는 노트북에 반대로 설정해야 하는 경우 유용하게 사용할 수 있습니다.

이렇게 설정하지 않으면 개인 Repository에 회사 계정으로 commit이 남고, 반대로 회사 Repository에 개인 계정으로 commit이 남는 실수를 저지를 수 있습니다.

마찬가지로 명령어를 통해 잘 설정되었는지 다시 확인해보면,

```shell
> git config --list
user.name=jaime
user.email=lcalmsky@gmail.com
# 생략
user.name=Jungmin Lee
user.email=jungmin.lee@company.co.kr
```

이렇게 `user.name`과 `user.email`이 두 번 나타나는 것을 확인할 수 있습니다. 처음 나타난 것이 global 변수, 나중에 나타난 것이 해당 repository에만 적용되는 변수입니다.

git은 항상 설정 목록의 마지막에 있는 값을 사용합니다.

## 페어 프로그래밍을 위한 Multiple Author 설정

Git은 현재 단일 커밋에 대해 여러 작성자를 지원하지 않지만 극복할 수 있는 몇 가지 옵션이 있습니다.

### user.name에 여러 이름 사용하기

가장 쉬운 방법은 user.name 설정에 모든 이름을 추가하는 것입니다.

```shell
> git config user.name "Younghee, Cheolsu"
```

페어 프로그래밍을 하는 동안만 author 설정을 유지해야하고, 이후에는 자신의 계정에 맞게 이름을 변경해두어야 합니다.

### 커밋 메시지에 다른 작성자 언급

관련 커밋 메시지에 다른 작성자와 관련된 메시지를 추가하는 방법입니다. git 위키의 커밋 메시지 컨벤션 섹션에도 나와있듯이, 커밋 메시지 끝에 헤더와 같이 검색 가능하고 파싱하기 쉬운 형식으로 추가 정보를 포함하는 방법을 제공합니다.

아래 예시처럼 작성할 수 있습니다.

```shell
Add foo feature

Co-authored-by: Cheolsu <cheolsu@example.com>
```

이를 편리하게 도와주는 [git-interpret-trailers](https://git-scm.com/docs/git-interpret-trailers)라는 도구가 있습니다.

> 자세한 내용은 링크를 참고하세요 😃

---

사용자 지정 커밋 메시지 템플릿을 이용하는 방법도 있습니다. .git-commit-template과 같은 간단한 텍스트 파일을 만들고 다음 명령을 실행하면 됩니다.

```shell
> git config commit.template .git-commit-template
```

페어 프로그래밍이 끝나면 다음 명령을 실행하여 커밋 메시지 템플릿을 기본 값으로 되돌릴 수 있습니다.

```shell
> git config --unset commit.template
```

---

마지막으로 rebase 명령어를 사용하여 커밋 메시지를 한 번에 수정하면서 공동 작성자를 추가하는 방법이 있습니다. 원격 저장소로 push 전에만 사용 가능합니다.

```shell
# N은 수정해야 할 커밋 개수 입니다.
> git rebase -i HEAD~N
```

```shell
e 6513610 Modify code block
e 79bdf06 Upload images
p 7dd93d7 Modify typo

# Rebase 92d1c55..7dd93d7 onto 92d1c55 (3 commands)
# 
# Commands:          
# p, pick <commit> = use commit

# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified); use -c <commit> to reword the commit message
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
```

e를 사용하여 편집할 커밋을 설정하고 p를 사용하여 최종적으로 사용할 커밋을 설정합니다.

이후 아래 명령어를 입력하면 커밋을 수정할 수 있습니다.

```shell
> git commit --amend
```

```shell
Upload images: edited

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Tue May 10 23:44:35 2022 +0900
#
# interactive rebase in progress; onto 92d1c55
# Last commands done (2 commands done):
#    edit 6513610 Modify code block
#    edit 79bdf06 Upload images
# Next command to do (1 remaining command):
#    edit 7dd93d7 Modify typo
# You are currently editing a commit while rebasing branch 'master' on '92d1c55'.
#
# Changes to be committed:
#       new file:   resources/images/49-01.png
#       new file:   resources/images/49-02.png
#
```

뒤에 edited라는 문구를 추가하여 주었습니다.

이후 아래 명령어를 사용하여 편집할 다음 커밋으로 넘어갈 수 있습니다.

```shell
> git rebase --continue
```

수정할 커밋에 대해 위 작업들을 반복하고 나면

```shell
Successfully rebased and updated refs/heads/master.
```

이렇게 `rebase`가 완료되었다는 로그가 출력됩니다.

이후 현재 커밋도 수정이 필요할 경우 다시 `git commit --amend` 명렁어를 사용할 수 있습니다.

### 커밋의 작성자 및 커미터 필드 인코딩

git은 각 커밋에 대해 커미터(committer)와 작성자(author)라는 두 사람의 이름과 이메일을 저장합니다. 둘의 차이점은 작성자는 변경 사항을 작성한 사람이고 커미터는 저장소에 업로드한 사람이라는 것입니다.

이 내용은 아래 명령어로 확인할 수 있습니다.

```shell
> git log --format=fuller
```

```shell
commit 4670e9e5e40ce02cb8bd479133d9d760186c7b79 (HEAD -> feature/11, origin/feature/11)
Author:     Younghee <younghee@example.com>
AuthorDate: Sat May 14 14:36:39 2022 +0900
Commit:     Younghee <younghee@example.com>
CommitDate: Sat May 14 14:36:39 2022 +0900

    Implement foo function

```

일반적으로 변경 사항을 직접 커밋하면 위의 예시처럼 동일하게 작성자와 커미터가 설정됩니다.

아래 명령어를 사용하여 작성자만 수정해 줄 수 있습니다.

```shell
git commit --author="Cheolsu <cheolsu@example.com>"
```

이러한 방법은 페어 프로그래밍에서 참가자를 표시하기에 좋은 방법이긴 하지만 대부분의 환경에서 커미터와 작성자를 같이 표시하진 않기 때문에 앞에서 사용한 방법들이 더 유용하다고 할 수 있습니다.

