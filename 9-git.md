## 0. 공부하게 된 계기
나 혼자 쓰는 프로젝트에서만 깃을 쓰다가 같이 쓰는 프로젝트에 커밋을 하다 보니 규칙을 잘 이해해야 한다고 생각이 들었고,  
규칙을 잘 이해하기 위해서는 헷갈리는 개념을 다시 정리해야 될 필요가 있다고 느꼈다.  
깃이 꼬이는 것을 방지하기 위해 기초 개념부터 다시 정리하기로 했다.  

## 1. git stage
깃에 상태에는 크기 tracked, untracked 상태로 나뉘고, tracked 상태는 다시 modified, unmodified, staged 상태로 나뉜다.  
untracked 상태는 git에서 형상 관리를 하지 않는 상태이다.  
track을 하고 싶으면 add를 하면 된다.  
modified 상태는 tracked 된 상태에서 파일이 변경되었다는 의미이다.  
즉, 형상관리에 들어간 파일이 커밋 이후에 새로운 변화가 생기면 modified 상태가 된다.  
반대로 unmodified 상태는 커밋 이후에 새로운 변화가 없는 상태를 말한다.  
stage 상태는 커밋 전 상태라고 생각하면 와닿을 것이다.  
형상관리에 들어가지 않았던 파일이나 들어갔지만 modified 상태의 파일을 대상으로 git add하면 되는 상태이다.

```
file -> (git add) -> stage -> (git commit) -> unmodified
```

위와 같은 그림이 내가 가장 이해할 수 있는 그림인 것 같다.


## 2. git stash
##### 임시 저장(tmp 디렉토리와 비슷). stack과 같은 구조를 가짐
https://www.lainyzine.com/ko/article/git-stash-usage-saving-changes-without-commit/ 이 주소에 더 자세히 나와있다.  
수정된 코드를 아직 커밋하지 않은 상태일 때, 임시 저장을 하지 않는다면 변경사항 전체를 하드 리셋하거나,  
저장소를 하나 더 클론 받아오거나 급하게 변경된 부분까지만 커밋하는 방법을 써야 됐다고 한다.  
이러한 문제를 해결하기 위해 나온 것이 git stash이다.  
커밋은 하지 않지만 변경된 부분을 저장하기 위해 존재한다.  

##### 간단한 사용법
git stash는 임시 저장을 하는 명령어로 git stash save의 약어이다.  
git stash -m 'message'로 저장할 수도 있고 stash를 보기 위해서는 git stash list를 하면 된다.  
git stash pop은 임시 저장된 내용을 꺼내오면서 삭제하는 명령어이다.  
git apply(꺼내옴) git drop(stash에서 삭제) 두 명령어가 합쳐진 것이라 생각하면 된다.  

##### 결국 merge와 비슷하다
임시 저장한 뒤에 다시 같은 파일에(같은 줄에) 어떤 작업을 하고, git stash pop을 하면 어떻게 될까?  
충돌이 난다.  
이를 해결하기 위해서 충돌을 하나하나 해결하는 merge와 비슷한 일을 해야 된다.  

##### 사용하는 이유
불필요한 commit을 남기지 않는 이유가 가장 큰 것 같다.

## 3. git merge vs git rebase
동작은 같지만 commit tree 관리에 차이가 생긴다.  
merge는 분기된 브랜치가 합쳐지는 graph가 나타난다.  
반면 rebase는 해당 위치를 기준으로 분기 위치를 다시 잡게 된다.  
자세한 차이는 https://brunch.co.kr/@anonymdevoo/7,  
https://dongminyoon.tistory.com/9  여기에 그림과 같이 잘 정리되어 있다.  

## 4. git resolve
만약 서로 다른 브랜치가 같은 파일, 같은 줄을 동시에 바꾸게 되어 merge 등의 행위를 못 하는 경우 '충돌'이 났다고 표현한다.  
그 충돌을 해결하는 것을 resolve한다고 한다.  
충돌을 해결하기 위해서는 하나를 제외하고 나머지 브랜치들의 내용을 포기해야 한다.  
이러한 과정을 강제로 병합시키지 않는 이상 손수 하나하나 찾아가며 바꾸는 것이 일반적으로 맞다.

## 5. cherry picker
https://cselabnotes.com/kr/2021/03/31/56/ 여기를 참고했다.  
커밋을 다른 브랜치에 잘못 하거나, 요구 사항이 바뀌어 필요 없는 커밋이 생기거나, 코드 의존성 때문에 다른 사람의 커밋 중 일부를 가져와야 하는 경우에 rebase나 cherry-pick의 방법을 사용할 수 있다.  
cherry-pick이 그렇게 권장되는 명령어는 아니지만 rebase의 번거로움(다른 브랜치에서 commit을 가져오고 싶다면 먼저 그 브랜치를 현재 브랜치로 merge한 후 rebase)를 감수하기 싫다면 사용하기 좋은 방법이라고 한다.  

##### 개념
git cherry-pick <commit hash>을 하면 해당 커밋의 내용을 현재 작업중인 브랜치로 가져온다.  
git cherry-pick <commit hash1>...<commit hash2>을 하면 해당 커밋들 사이에 있는 모든 커밋들을 가져온다.  
cherry-pick을 사용해도 당연히 충돌이 일어날 수 있다.  
충돌을 해결한 뒤에 git cherry-pick -continue를 사용하면 다시 cherry pick이 진행된다.  
git cherry-pick -abort를 사용하면 cherry pick을 하기 전 상태로 돌아간다.  
