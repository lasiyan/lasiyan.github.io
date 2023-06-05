---
title: GitHub 블로그 시작하기 - 2
date: 2023-06-05 08:02:00 +0900
categories: [etc., GitHub]
tag: [github-blog]
---

## 테마 설치 전..
---

[Chirpy 가이드 페이지](https://chirpy.cotes.page/posts/getting-started/)를 보면 Starter 방식과 fork 방식을 안내하고 있습니다.

Starter 방식의 경우 가장 빠르고 쉽게 블로그를 생성할 수 있지만 향후 커스터마이징을 진행할 때 문제가 발생할 수 있다고 합니다.

결과적으로 저는 fork 방식으로 저장소를 생성하고 이를 빌드 후 새 저장소에 옮기는 방법을 사용했습니다.

해당 글은 Chirpy 6.0.1 버전 기준입니다.

## 테마 설치 1
---

먼저 기존 생성하였던 저장소(`사용자이름.github.io`)를 과감하게 삭제합니다.

그리고 [여기](https://github.com/cotes2020/jekyll-theme-chirpy/fork)서 저장소 fork를 진행합니다.

> 마찬가지로 이름은 `사용자이름.github.io`로 생성해주세요.  
나머지는 기본값으로 진행했습니다.


## 테마 설치 2
---
이 항목이 제가 윈도우에서 리눅스로 변경한 이유입니다.

먼저, 일부 블로그에서 설명하는 zip을 다운로드 또는 clone하여 저장하는 방식이 저에겐 제대로 동작하지 않았습니다.

`assets/js/dis/misc.min.js` 등 파일이 존재하지 않는다는 에러와 함께 `theme color toggle`, 글의 `Top of Content` 기능 등이 동작하지 않았습니다.

결과적으로 init이 진행하지 않았기 때문이었습니다.

> Build JavaScript files and export to assets/js/dist/, then make them tracked by Git.

가이드 페이지를 보면 init을 진행 시 assets 폴더에 javascript 파일을 복사한다고 되어 있습니다.

따라서 init을 진행하는 과정은 다음과 같습니다.

참고로 nodejs 설치가 필요합니다. 관련 내용은 [구글링](https://www.google.com/search?q=how+to+install+nodejs) 해보시면 많은 컨텐츠가 있습니다.

```shell
# fork한 저장소를 다시 불러온다 (사용자 이름 주의)
$ git clone https://github.com/lasiyan/lasiyan.github.io

# 로컬 저장소로 이동
$ cd lasiyan.github.io

# init 실행 (실행 경로 주의! 반드시 블로그 root 디렉토리에서 실행)
$ bash tools/init
```

init 과정이 해당 포스트를 작성한 이유입니다.

1. starter 방식 : 해당 과정이 필요 없음

2. zip 방식 : init 실행 시 github 관련 정보가 연동되지 않는 것처럼 보임.  
--no-gh 옵션을 추가해봤지만 정상적으로 동작되지 않았음

3. fork 방식
    1. 윈도우 환경 : NODE_ENV 환경변수 에러 발생

    2. 윈도우 + WSL(우부투 22.04) : NODE_ENV 환경변수 에러 발생

    3. 리눅스 환경 : 정상 동작

    4. 맥 환경 : 리눅스와 동일할 것으로 추정

결과적으로 리눅스 환경에서 성공하였기 때문이고,

윈도우 환경에서 저와 비슷한 에러가 발생하신 분들 중, 리눅스 환경을 사용할 수 없는 분이라면

해당 블로그의 소스를 download 하신 후 _posts 안의 내용물을 삭제하고 개인 블로그로 사용하셔도 됩니다.

## 테마 설치 3
---
이제 여느 블로그와 같이 설치를 진행하시면 됩니다.

Github 블로그에서 사용되는 jekyll 테마에는 모두 _config.yml 파일이 있습니다.

쉽게 말해서 티스토리나 네이버의 블로그 설정 페이지라고 보시면 됩니다.

처음부터 쭉 읽어보면서 원하는 대로 수정해도 되는데, 가이드 페이지에 기술된 주요 몇 가지 사항만 먼저 수정해 보겠습니다.

* url : 소스를 보면 cache 또는 sitemap관련해서 사용되는 것 같습니다.  
블로그 실제 url을 넣어주시면 됩니다. (`"https://lasiyan.github.io"`)

* timezone : Asia/Seoul

* lang : 보통 `en` 또는 `ko-KR` 을 사용할 겁니다. 저는 ko-KR 변경 시 폰트가 마음에 안들어서 en을 사용 중입니다.

* 나머지는 천천히 수정하셔도 괜찮습니다.


## 테마 설치 4
---
마지막으로 로컬 테스트 서버를 켜서 블로그 동작을 확인해 봅니다.

```shell
# 먼저 블로그 루트 디렉토리로 이동합니다 (init 했던 경로)

# bundle로 테마 관련 의존성 패키지를 설치합니다.
# bundle 명령어나 bundle install 이나 동일합니다.
# 이는 Gemfile 파일에 기술된 의존성 패키지들을 설치합니다.
$ bundle install

# 서버를 실행합니다 (저는 원격에서 개발하기 때문에 명시적으로 주소를 지정해 줍니다)
$ bundle exec jekyll serve
# bundle exec jekyll serve --host 192.168.0.116
```

http://127.0.0.1:4000 로 접속 시 정상적으로 테마가 로드되고, 기능(테마 색상 변경 등)이 동작되거나, 터미널에 특별한 에러가 발생하지 않으면 성공입니다.

## 배포하기
---

마지막으로 개발(테스트)한 페이지를 인터넷 상에서 볼 수 있도록 블로그를 배포해야 합니다.

블로그의 세팅 페이지에서 아래와 같이 설정되어 있는지 확인 해주세요.

![deploy-repository](/assets/img/post/2023-06-05-GitBlog_Basic_2/repo_deploy.png)

> 참고로 저는 테마 설치 3까지 진행하고, 해당 시점의 코드로 저장소를 새로 만들었습니다. 그래서 branch 값이 기본 값인 main 입니다

> 만약 fork 상태로 진행하셨다면 master일 것입니다.

*아래에서는 main을 기준으로 설명드리겠습니다. 자신의 branch에 맞게 수정해주세요*

init을 진행하면 .github 페이지에 pages-deploy.yml 파일이 있을 것입니다.

해당 파일에서 master 부분을 주석 처리 해주세요.
(만약 현재 branch가 master라면 main을 ..)

![deploy-pages](/assets/img/post/2023-06-05-GitBlog_Basic_2/pages_deploy.png)

모든 과정이 완료되면 Commit을 진행합니다.

![commit-repo](/assets/img/post/2023-06-05-GitBlog_Basic_2/repo_commit.png)

위 사진은 이 글이 Commit되는 예시입니다. 실제로는 처음 Commit이기 때문에 수 많은 파일이 있을 것입니다.

마지막으로 github 에서 Actions 탭을 확인합니다.

아래 사진처럼 commit을 진행할 때 마다 pages-deploy.yml 이 동작하여 테스트를 진행하는 것을 볼 수 있습니다.

참고로 이 과정이 끝나야 https://lasiyan.github.io 사이트에서 결과물을 확인할 수 있습니다.

![actions-repo](/assets/img/post/2023-06-05-GitBlog_Basic_2/repo_actions.png)

각 항목을 클릭해보면 진행 상황을 확인할 수 있고, 에러 발생 시 관련 로그를 체크할 수 있습니다.

다음 장에서는 댓글을 설정하는 방법에 대하여 포스팅 하겠습니다.