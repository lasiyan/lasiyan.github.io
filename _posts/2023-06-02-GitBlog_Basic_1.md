---
title: GitHub 블로그 시작하기 - 1
date: 2023-06-02 14:05:00 +0900
categories: [etc., GitHub]
tag: [github-blog]
---

## 들어가기에 앞서 ...
---

Chirpy 테마 기반의 Github 블로그 개발을 진행하면서 문제가 발생하였던 부분을 기록합니다.

처음 시작은 Windows 10 환경에서 진행하였으나, nodejs 실행 이슈 때문에 결과적으로 Linux (Ubuntu) 환경으로 진행하였습니다.

만약 윈도우 환경에서 진행할 경우, 이 단계부터 진행하는 것을 추천드립니다.

### 참고 !

아래 과정을 진행하기 전 [블로그 시작하기 2](https://lasiyan.github.io/posts/GitBlog_Basic_2/)를 먼저 읽어보시면 좋습니다.

이유는 Repository 생성과 Clone은 결과적으로 나중에 다시 삭제할 것이기 때문입니다...

글을 완전히 따라하실 분들은 VSCode 및 Ruby 설치 부분만 진행해주세요.



## 개발 환경
---
> Remote
- NVIDIA Jetson AGX Orin (developer kit)
- Jetpack 5.0.1 (based on **Ubuntu 20.04** Linux for Tegra)

> Local
- IDE: Visual Studio Code 또는 GitHub가 포함된 기타 편집 툴


## 준비물
---
개발 환경에 기술된 바와 같이, 저는 윈도우 기반의 Desktop PC와 Linux 기반의 Embedded Board가 준비되어 있습니다.

하지만 대부분 1대의 PC로 개발을 진행할 것입니다.

따라서 이 경우 Linux 또는 Mac OS가 준비된 PC 또는 가상 환경이 필요합니다.


## GitHub 준비하기
---
GitHub 블로그는 GitHub Repository(Page)를 기반으로 동작합니다. 따라서 저장소를 만들기 위하여 GitHub 가입을 진행하고 Repository를 생성하여야 합니다.

여기서 주의할 점은 Repository 이름을 반드시 `사용자이름.github.io` 로 생성하여야 합니다.

![create-repository](/assets/img/post/2023-06-02-GitBlog_Basic_1/repo_create.png)

위 사진과 같이 Owner 값과 동일하여야 합니다.

나머지 값들(Description, Add a README file 등)은 모두 기본값으로 두고 **Create repository** 를 클릭합니다.


## VS Code 준비하기
---
VS Code를 사용하는 이유는 다음과 같습니다.

1. Github와 연동이 편리하다.
2. Markdown 관련 다양한 확장을 지원한다.
3. (필자의 경우) 원격 개발을 지원한다.
4. 다들 많이 쓰는 편집기라서 ...

서두에 적힌 것과 같이 VSCode가 아닌 다른 편집기를 사용하여도 무관합니다.

어떤 사람은 GitHub 웹 상에서 직접 작업 하는 사람도 있었고, 메모장으로 파일만 만들고 GitHub Desktop을 통해 Commit 하는 사람도 있었습니다.

중요한 것은 최소한 Git을 통해 소스를 불러오고(**Clone**), 수정 후 이를 적용하는(**Commit**) 개념만 알고 있으면 어떤 방법이든 익숙한 방법을 사용하시면 됩니다.

이 글에서는 리눅스 환경에서 VSCode를 기준으로 설명드리겠습니다.

먼저 원하는 경로에 생성해 두었던 Repository를 불러옵니다.

> 필자의 사용자 이름은 **`lasiyan`** 입니다. 향후 모든 명령어 중 해당 값이 포함된 부분은 본인에게 맞는 이름으로 변경하여야 합니다.

> 필자는 원격 환경에서 개발을 진행 중입니다. 보이는 아이콘 또는 메뉴가 약간 다를 수 있습니다.

![clone-repository](/assets/img/post/2023-06-02-GitBlog_Basic_1/repo_clone.png)

Repository 주소는 `https://github.com/lasiyan/lasiyan.github.io.git` 와 같은 형식입니다. 다만 VSCode 자체에 GitHub가 연동된 적이 있다면 `Clone from GitHub` 를 통해 간편하게 불러올 수 있습니다.

불러오기에 성공한 경우 UI가 아래와 같이 변경되며, 변경 사항이 있으면 Stage를 통해 Commit 할 수 있습니다.

![clone-repository-after](/assets/img/post/2023-06-02-GitBlog_Basic_1/repo_clone_after.png)

처음 사용하시는 유저라면 Commit 간 에러가 발생하는 경우가 있습니다.

에러 내역을 읽어보면 원인을 알 수 있는데, 대부분의 경우 Git 정보가 등록되어 있지 않기 때문입니다 (permission 관련 문제)

>  git config --list를 통해 등록된 사용자를 확인하고 git config를 통해 `user.email`, `user.name` 등을 등록하면 정상적으로 Commit이 진행됩니다. (--global 옵션은 자율입니다)

![clone-permission](/assets/img/post/2023-06-02-GitBlog_Basic_1/repo_permission.png)

저는 해당 PC의 경우 다중 Github 계정을 사용하고 있지 않기 때문에 `--global`로 관리하고 있습니다.


## Ruby 설치하기
---
현재 Repository는 비어있습니다. 우리는 여기에 Jekyll 기반 테마를 설치할 것입니다.

Jekyll 테마는 Ruby 언어를 기반으로 구현되어 있기 때문에 먼저 Ruby를 설치해야 합니다.

리눅스에서 Ruby는 rbenv를 통해 설치가 가능합니다.
(Mac OS도 동일합니다)

아래와 같은 명령어를 통해 설치가 가능합니다.  
Ruby 버전은 Chirpy 테마에 맞게 3.1.4 버전으로 진행하였습니다.

```shell
# 0. 만약 git이 설치되어 있지 않다면, git을 먼저 설치 후 진행
$ sudo apt-get install -y git

# 1. 홈 디렉토리의 .rbenv 폴더에 rbenv 저장소를 불러옴
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv

# 2. 환경변수에 rbenv 등록
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ exec $SHELL

# 3. rbenv build 플러그인 설치
$ git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build

# 4. ruby 설치 (버전 3.1.4)
$ rbenv install 3.1.4
$ rbenv global 3.1.4

# (optional) 만약 설치 실패 시 추가 dependencies 설치 후 4번부터 다시 진행
$ sudo apt-get install -y libreadline-dev zlib1g-dev

# 5. 설치 확인
$ ruby -v

# 6. Ruby 패키지 설치
$ gem install jekyll bundler
$ gem install webrick  # for Windows or older ruby version
```

이로서 Ruby 및 관련 패키지가 설치되었습니다. 
다음 글에서 테마 설치 및 초기 설정에 관하여 포스팅 하겠습니다.
