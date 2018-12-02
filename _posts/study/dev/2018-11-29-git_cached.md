---
title: "[Git] Add가 되지 않는 이슈"
categories: "GIT"
tags:
  - Git
---

최근 프로젝트 모듈을 분리하고, 디렉토리 구조를 바꾸면서 깃이 꼬여버렸다.

분리한 파일들이 깃에 add자체가 되지 않고, 로그에서도 특별한 상황을 확인하기 어려웠다.

파일을 옮겨서 다시 root 경로를 잡은 후 작업을 했지만 결과는 마찬가지였다.

add가 정상적으로 진행되지 않을 때, 확인해볼 것들이 있다.

~~~bash
  // 로컬 Branch 간 비교
  git diff <branch명> <다른 branch명> 

  // 로컬과 리모트Branch 간 비교
  git diff <branch명> origin/<branch명> 

  // Commit 간 비교
  git diff <commit hash> <commit hash> 

  // 마지막 커밋과 현재 수정사항 확인
  git diff HEAD

  // 현재 unstaged 된 수정사항 확인
  git diff
~~~

<code>git diff</code>를 통해 변경된 결과를 확인할 수 있다.

하지만 내 경우 새로 구성한 디렉토리 및 파일들이 정상적으로 git에 관리되지 않고 있었다.

정확히 어디서부터 꼬였는지 파악은 어려웠지만, 결과적으로 깃 캐시를 날려서 add 후 push까지 정상적으로 진행할 수 있었다.

### cached 제거

우선 cached 된 파일을 날려주자.

~~~bash
  git rm -r --cached .
~~~

>git rm => 원격 저장소와 로컬 저장소에 있는 파일을 삭제한다. <br/>
git rm --cached => 원격 저장소에 있는 파일을 삭제한다. 로컬 저장소에 있는 파일은 삭제하지 않는다.

아마도 리모트 저장소에 파일들이 잘못 적재되어 있던 모양이다.

위와 같이 cache를 날려주니, 정상적으로 add가 진행되었다!

--- 

이 밖에도 리모트 저장소가 아닌 로컬에서 unstaged된 파일을 깔꼼하게 정리하고 싶다면,
다음의 명령어를 날려주면된다.

~~~bash
  git clean -d -x -f
~~~
- -d : 디렉토리 포함
- -x : igrnoed 파일 포함
- -f : force

이제 로컬 git 환경이 어느정도 정리된 것 같다.

캐쉬를 날린 후 git add 해주어 최종적으로 push를 해주면 정상적으로 등록시킬 수 있다.

다음은 add와 commit에 대한 활용을 다루어 보아야겠다.

## 참고
- https://gracefullight.github.io/2016/12/28/git-add-%EC%95%88-%EB%90%98%EB%8A%94-%EA%B2%BD%EC%9A%B0-%ED%99%95%EC%9D%B8%ED%95%B4%EC%95%BC%EB%90%A0-%EA%B2%83/
- https://brunch.co.kr/@hopeless/9
- http://mygumi.tistory.com/103