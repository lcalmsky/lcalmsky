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
// 생략
user.name=Jungmin Lee
user.email=jungmin.lee@company.co.kr
```

이렇게 `user.name`과 `user.email`이 두 번 나타나는 것을 확인할 수 있습니다. 처음 나타난 것이 global 변수, 나중에 나타난 것이 해당 repository에만 적용되는 변수입니다.

git은 항상 설정 목록의 마지막에 있는 값을 사용합니다.

