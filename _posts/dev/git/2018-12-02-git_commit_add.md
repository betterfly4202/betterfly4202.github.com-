---
title: "[Git] Add와 Commit"
categories: "git"
---

요즘 툴들이 워낙 강력해서 대부분 툴 안에서 관리하긴 하지만, 하나씩 직접 입력하면서 각 과정에서 어떤 이벤트가 일어나는지 눈으로 확인하는 것은 추후에 문제가 발생했을 때도 어느 지점에서 어떤 문제가 발생했는지 명확하게 구분할 수 있기 때문에 이러한 훈련 또는 습관도 필요할 것이다.

# Git add

나는 Intelli J를 사용하는데, 이 강력한 개발도구는 클래스 또는 어떤 파일을 생성해도 Git에 추가할 것인지 팝업창이 발생한다.

![intellij](/assets/images/study/dev/2018/12_intellij_in_git.png)

위와 같이 아주 친절하게 추가할지의 여부를 확인하고, 우리는 파일을 생성하는 즉시 Git의 Stage에 파일을 올리기도 한다.

하지만 이런 작업들이 어떤 과정에서 진행되는지 인식하지 않고 도구들에만 의존하다보면, 내가 개발을 하는지 도구가 만들어주는건지 주객이 전도되어 버리는 일이 발생할지도 모른다!

그래서 깃의 소소하지만 중요한 명령어 중 가장 기본적인 add / commit의 과정을 다루어 보려 한다.

![git_stage](/assets/images/study/dev/2018/12_git_stage_add.png)

Git의 work flow를 살펴보면, 로컬환경에서 개발된 작업(파일)은 <code> git add </code> 를 통해서 stage 상에 올라간 후 <code> git commit </code> 를 통해 최종적으로 Git의 리모트 저장소에 적재 된다.

우리가 작업한 소스코드 또는 파일들을 add하기 위해 흔히 사용하는 명령어는

<code>git add . </code> 또는 <code>git add [file name] </code> 이다.

가장 위험한 것이 전자인 <code>git add . </code> 인데, 이 방법은 새로운 이벤트가 있었던 모든 변경사항들을 스테이지상에 올려 버린다.

이 경우 정확히 어떤 내용들이 반영되 었는지 일일이 확인하기 번거로울 뿐 더러, 당연히 확인하려 들지도 않을 것이다.

이 과정에서 원치않은 변경사항이 반영될 수도, 인텔리제이의 경우 .iml 과 같은 환경파일 같은 것들이 올라가 버릴지도 모른다. 
이 파일을 받은 다른 사용자들은 큰 번거로움을 겪게 될 것이다.

그러면 우리는 안전하게 <code>git add [file name] </code> 명령어를 사용하는 것이 좋을 것이다.

그런데 이것은 매우 번거롭다. 파일이 1~2개 정도라면 큰 지장이 없겠지만 5개 이사만 되더라도, 하나씩 파일이름을 입력해준다는 것은 보통 번거로운 일이 아니다.

그래서 <code>git add -p</code> 명령어가 있다.

~~~
git add -p
~~~

를 입력하면 어떤일이 일어나는지 보자.

![git_add_p](/assets/images/study/dev/2018/12_git_add_p.png)

내용을 살펴보면, 어떤 작업에서 어떤 이벤트가 있었는지 상세히(?) 설명을 해준다.

그리고 하단에 *Stage this hunk?* 라는 메세지와 함께 옵션 값을 입력해주어야 한다.

>hunk란?<br/>'하나의 이벤트(변경사항) 단위' 정도로 이해하면 될 것 같다.

### git add -p options
~~~
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
~~~

여러 옵션이 있지만, 일반적으로 y,n,q 정도의 옵션이면 큰 무리 없이 진행할 수 있다.

---

# Git commit

Git Commit에도 <code>git add -p</code>와 비슷한 작업이 있는데
<code> git commit -v</code>가 바로 그 명령이다.

<code>git diff</code> 명령어는 변경사항에 대한 상세한 기록을 확인할 수 있다. 하지만 그대로 열람만 가능하다.

![git_diff](/assets/images/study/dev/2018/12_git_diff.png)

하지만 <code>git commit -v</code>를 통해서 변경사항 확인 및 commit 까지 동시에 처리할 수 있다.

![git_commit_v](/assets/images/study/dev/2018/12_git_commit_v_sample.png)

화면과 같이 최상단에 커밋 메세지를 입력해주고 저장을 해주면 변경사항들이 정상적으로 commit된다.

![git_commit_log](/assets/images/study/dev/2018/12_git_commit_log.png)

## 참고
- https://blog.outsider.ne.kr/1247
- http://bcho.tistory.com/773