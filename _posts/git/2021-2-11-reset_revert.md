---
layout: post
title: git reset과 revert (+ checkout)
category: git
tags: [github]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.        
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 git 명령어 중, reset과 revert (+ checkout)에 대해서 알아보도록 하겠습니다. <br>

git 으로 버전관리를 하다보면 실수를 하거나 또 다른 이유로 이전 커밋으로 돌아가야할 때가 있습니다. 이때 사용할 수 있는 명령어가 **reset과 revert** 인데요, reset과 revert는 어떤 차이가 있을까요?   

<br>
<br>

# reset

reset 명령어는 이전의 커밋으로 돌아가는 동시에 현재부터 돌아갈 시점까지의 커밋이력은 모두 삭제하는 명령어입니다.

![스크린샷 2021-02-28 오후 12 03 05](https://user-images.githubusercontent.com/73867548/109406775-2d42d080-79bf-11eb-85ae-6cdab2446b52.jpg)

이렇게 우리가 4개의 commit을 했다고 했을때, commit 2를 reset하게 되면 commit 3, 4는 모두 삭제되는 것입니다.       
reset을 하면서 삭제된 커밋으로는 영영 돌아올 수 없으니 신중하게 사용해야 되겠네요 :)

```swift
1) git reset . : add 내용 삭제하기

2) git reset <commit번호> : <commit번호>버전으로 돌아가기, 커밋 이력만 삭제,
		            working directory는 유지 (현재 파일 내용)

3) git reset --hard <commit번호> : <commit번호>버전으로 돌아가기, 커밋 이력 삭제,
				working directory(작업한 파일 내용)도 이전 버전으로 돌아간다.

4) git reset --soft <commit번호> : <commit번호>버전으로 돌아가기, 커밋 이력 삭제,
				working directory(작업한 파일 내용)유지, add 상태 유지
```

<br>

### 1) git reset .
 현재 staging area에 add된 내용을 삭제합니다. 즉, 현재 commit버전으로 돌아온다?는 이야기인데, 이때 working directory는 삭제되지 않고 현상태가 유지됩니다.

 ![스크린샷 2021-02-28 오후 12 03 25](https://user-images.githubusercontent.com/73867548/109406845-9c202980-79bf-11eb-81ca-f3325c8958c6.jpg)

<br>

### 2) git reset <commit번호>
 <commit번호>버전으로 돌아가는 동시에 reset시점에서 돌아갈 시점 사이의 커밋 이력이 삭제됩니다. 하지만 reset 시점의 작업 내용, working directory는 유지됩니다. 아래의 1, 2, 3, 4 주석은 각각 첫번째, 두번째, 세번째, 네번째 커밋이라고 생각하면 될 것 같습니다.

 ![스크린샷 2021-02-28 오후 12 03 47](https://user-images.githubusercontent.com/73867548/109406872-d8ec2080-79bf-11eb-9f4b-b80fda867cca.jpg)

 commit 2로 돌아가더라도, 이렇게 4번까지 작업한 내용은 남아있게 된다는 말입니다. 잘 이해가 안된다면  아래의 --hard 명령어와 비교해봅시다.

<br>

### 3) git reset --hard <commit번호>
 'hard'라는 말처럼 강력하게 reset되는 명령어입니다. git reset과 같이 <commit번호>버전으로 돌아는 동시에 커밋 이력 삭제되지만, 기본 reset과는 다르게 working directory도 이전 버전으로 돌아가기 때문에 reset 시점에 작업했던 내용은 삭제됩니다.

 ![스크린샷 2021-02-28 오후 12 04 10](https://user-images.githubusercontent.com/73867548/109406914-2e283200-79c0-11eb-8236-d558d5af7222.jpg)

 commit 2로 돌아가는 동시에 작업내용도 commit 2의 시점으로 돌아가는 강력한 reset입니다.

<br>

### 4) git reset --soft <commit번호>
reset soft는 기본적인 reset과 비슷한데 reset 시점의 작업 내용이 자동으로 staging area에 add된다는 차이가 있습니다. reset soft를 해주고 git status를 확인해보면

![스크린샷 2021-02-28 오후 12 04 44](https://user-images.githubusercontent.com/73867548/109406951-73e4fa80-79c0-11eb-9c0a-391a6c4f232a.jpg)

이렇게 add가 되어있는 것을 확인할 수 있습니다.

 ![스크린샷 2021-02-28 오후 12 03 47](https://user-images.githubusercontent.com/73867548/109406872-d8ec2080-79bf-11eb-9f4b-b80fda867cca.jpg)

그리고 working directory 역시 삭제되지 않고 유지되어 있네요 :)

<br>
<br>

# revert
![스크린샷 2021-02-28 오후 12 05 00](https://user-images.githubusercontent.com/73867548/109406963-98d96d80-79c0-11eb-9773-f1e12442fdff.jpg)

git revert는 reset과 다르게 이전 커밋으로 돌아가더라도 커밋이력이 삭제되지 않습니다.

```swift
git revert <commit번호> : <commit번호>버전으로 돌아가기, 커밋 이력 유지,
			working directory는 유지 (현재 파일 내용)
```

<br>

### 1) git revert <commit번호>

revert는 기본적으로 reset 명령어와 비슷한 상태로 돌아가게 됩니다. reset과 같이 <commit번호>버전으로 돌아가는 동시에, revert하는 시점의 working directory(현재 파일  내용)는 유지됩니다. 하지만 reset과 달리 커밋 이력이 삭제되지 않습니다.

>revert를 하면 커밋이력이 삭제되지 않는 장점이 있지만,     
 커밋이력이 남아있기 때문에 같은 곳을 수정한다면 충돌이 발생할 수 있다는 점을 유의해야합니다!

 <br>
 <br>


# 🤔 reset과 revert는 어떻게 구분해서 써야할까요?

둘의 차이는 커밋 이력이 '삭제된다 / 안된다' 입니다. 그렇다면 '안전하게 커밋 이력이 삭제되지 않는 revert를 쓰면 되지 않을까?' 생각이 들기도 하는데요, 저는 개인적으로 충돌이 자주 일어나지 않는 reset이 단순해서 사용하기 더 편하다는 느낌을 받았습니다. 혹은 그냥 과거로 돌아가지말고 새로운 커밋을 만들어버리는게 편한 것도 같아요.

하지만 원격 저장소에 push를 했을 경우에는 상황이 달라지는 것 같습니다. 원격 저장소에 push한 후, 작업중 reset으로 돌아가게 되면 reset하기 전으로 작업을 다시 되돌리기 전에는 push를 할 수 없는 상태가 됩니다. 그렇기 때문에 원격 저장소에 있는 프로젝트의 과거로 돌아갈 때에는 revert로 돌아가야 안전하게 push가 가능합니다. reset으로 돌아가더라도 '강제로' push하는 방법이 있기는 하지만 강제로 하는 방법은 언제나 불가피한 경우말고는 피하는게 좋을 것 같네요.

좋은 의견이 있다면 댓글로 공유해보면 좋을 것 같아요 🤔

<br>
--------------
<br>

# checkout .

```swift
1) git checkout . : working directory에서 수정한 모든 파일을 현재 버전(수정전)으로 되돌리기

2) git checkout -- <file이름> : working directory에서 수정한 <file이름>을 현재 버전(수정전)으로 되돌리기

3) git checkout <폴더명> : working directory에서 수정한 폴더를 현재 버전(수정전)으로 되돌리기
```

checkout을 사용하면 수정한 내용을 현재의 커밋으로 초기화하는 효과를 볼 수 있습니다. 즉, staging area에 add할 것이 있는 상태에서 **add할 것이 없던 상태로**  돌아간다는 이야기입니다.
복잡하다면 아래의 예시를 봅시다!

![스크린샷 2021-02-28 오후 12 05 19](https://user-images.githubusercontent.com/73867548/109407119-ac390880-79c1-11eb-9e8c-7260d47de444.jpg)


커밋이 된 상태에서 22번 줄 '아무말 대잔치!'라는 주석을 추가하고 `git checkout .` 명령어를 입력한 것입니다. 이렇게 수정한 내용을 깔끔하게 커밋한 부분으로  초기화할 수 있습니다. <br>

*checkout은 branch를 불러올 때 사용하는 명령어로만 알고 있었는데 또 다른 뉘앙스로 유용한  기능이네요 :D*






<br>
<br>
