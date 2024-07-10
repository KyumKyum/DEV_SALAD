# 커밋 합치기 & 커밋 나누기 (Commit Squash & Commit Split)

- `git rebase`는 나에게 있어서 정말 두려운 순간이다. 관리가 안된 커밋로그로 인해서 끝나지 않는 것 같은 중복된 conflict 해결, 그리고 여전히 일어나는 conflict. 어디서부터 꼬인지도 모르겠는 commit과 merge의 향연을 보다보면, 자연스럽게 머리가 아파온다.
- 해당 문제로 골머리를 썩히고 있던 중, 시니어 개발자 NoizBuster님이 좋은 자료를 공유해주셔서 지금까지 공부한 내용을 참고하여 내용을 정리해봤다! 🙂

---

### 0. 문제상황

- 일단, 양심고백 한 가지: 난 꽤 최근까지 rebase와 merge가 별 차이 없는 것인 줄 알았다. 
    - 커밋 로그가 안 생겨난다, Linear한 커밋 로그 기록이 만들어진다 등등...

- 그런데, `git rebase`시 내가 만들었던 모든 커밋 로그에 대해서 **여러번 동일한 내용의 conflict**가 발생하는 것을 몇 번 경험하였고, 해당 내용들이 이전에 내가 해당 브랜치에서 작업했던 내용과 커밋들이었다는 것을 발견하였다.
    - `git rebase`: 현재 브랜치의 모든 커밋을 새 베이스 브랜치 위로 하나씩 적용을 하는 것. (그래서 Linear History가 만들어진다는 것)
        - 즉, 각 커밋이 **개별적으로** 적용이 되기에, 만일 일정 시점의 커밋 로그에서 conflict가 발생한다면, 그 이후 모든 커밋 로그에 대해서 동일한 내용의 conflict가 발생하게 되는 것.

- 지난 해커톤때도 이런 문제로 시간을 너무 많이 잡아먹었고 ~~(해커톤에서는 허용되는 진리의 `git push -f`로 나름 수월하게 해결했다)~~, 지금 일하는 환경에서도 비슷한 문제를 겪어, 확실한 해결 방법이 필요하다고 생각을 하던 중, 다음과 같은 생각에 도달하게 된다.

> 잠깐만, 그러면 커밋 로그를 하나로 묶거나, 일부를 합쳐거나 지워서 로그의 개수를 줄이면 되잖아!


---
### 1. Interactive Rebase
- 현재 개발 공간에서 `git log --pretty=oneline`을 입력했을 때의 결과이다.

```bash
$ git log --pretty=oneline
3f48ae571e358dcf29a... implement basic core manager architecture, integrate with redis mq.
df7998685ad742cd979... rebase: merged with ...
ea461239d09bace8104... feat(...): implement ... feature to handle clustered payload
88e106710f4d88470d9... feat(cache): implemeted cache to handle clustered payload
c2bbb5a0f7da64f66b5... style(review): apply lint, change function name with more readablility
02160a54fee5353e1cd... fix(naming): change default exported name at root.controller.ts

```
- **엉망진창이다**. rebase 이후 조금 수정한 작업물을 그냥 "rebase 했음!"이라고 로그를 남겼고, 메세지만 다를 뿐 똑같은 내용의 커밋 메세지도 있다(왜?!) 그리고 단순 린트 적용도 있고, 작업 내용은 전체 결과물의 부분 작업물들이다.

- 이러한 엉망진창인 로그들을, 하나의 상자의 담아 정리하고 싶다. 해당 상자의 이름이 저 모든 커밋을 대표하는 새로운 커밋 로그가 될 것이다.

> 묶어버려!
```bash
# interactive rebase
git rebase -i HEAD~6 # Procees interactive rebase with 8 recent commits.
```
- 최근 6개의 커밋 기록을 대상으로 interactive rebase를 진행한다.

```bash
pick 02160a5 fix(naming): change default exported name at ...
pick c2bbb5a style(review): apply lint, change function name with more readablility
pick 88e1067 feat(cache): implemeted cache to handle clustered payload
pick ea46123 feat(...): implement ... feature to handle clustered payload
pick df79986 rebase: merged with ...
pick 3f48ae5 feat(core)!: implement basic core manager architecture, integrate with redis mq.

# Rebase c9600c4..3f48ae5 onto c9600c4 (6 commands)
#

```
- 시간 역순으로 (이전 -> 최근 (HEAD))으로 로그가 나온다.
- 여기서 주목할만할 Command는 다음과 같다.
1. `pick`: 해당 커밋을 살려둔다.
2. `reword `: 해당 커밋을 살려두고, 커밋 메세지를 수정한다.
3. `edit`: 해당 커밋을 살려두고, 커밋 메세지와 작업 내용 자체를 수정할 수 있다.
4. `squash`: 해당 커밋을 살려두고, 이전 커밋에 흡수시킨다.
5. `drop`: 해당 커밋을 지운다. 커밋의 변경사항 또한 사라진다. 필요없는 커밋을 날릴 때 사용.

> 이 외에도 명령어가 몇 가지가 더 있다! 나중에 더 정리해봐도 좋을 것 같다.

---
### 2. Commit Squash

- 나의 목표는 다음과 같다: 커밋 로그를 하나로 합치고 (squash), 남은 커밋 로그 하나를 이 작업을 대표하는 내용으로 바꾸기 (reword).


- 다음과 같이 수정한다.
```bash
reword 02160a5 fix(naming): change default exported name at ...
squash c2bbb5a style(review): apply lint, change function name with more readablility
squash 88e1067 feat(cache): implemeted cache to handle clustered payload
squash ea46123 feat(...): implement ... feature to handle clustered payload
squash df79986 rebase: merged with dev
squash 3f48ae5 feat(core)!: implement basic core manager architecture, integrate with redis mq.

```
- 저장한후 에디터를 나온다. 이제 다음 작업으로 넘어가게 된다. 나같은 경우는, 커밋 로그를 수정하는 `reword` 옵션이 있기에, 이를 수정하는 작업이 진행된다.

```bash
fix(naming): change default exported name at ...

requestEventService -> publishEventService

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.

```
- 다음과 같이 바꾼다.

```bash
rebase!: squash commit logs related with core manager development.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.

```
- 이후에는 squash되는 커밋들의 목록이 나오며, 해당 에디터를 나오면 interactive rebase가 완료된다.

```bash
# This is a combination of 6 commits.
# This is the 1st commit message:

rebase!: squash commit logs related with core manager development.

# This is the commit message #2:

style(review): apply lint, change function name with more readablility

# This is the commit message #3:
...
```

```bash
$ git logs --pretty=oneline
a0ce43fe9de8498acde... (HEAD -> feat/...) rebase!: squash commit logs related with core manager development. 
```
- 예쁘게 합쳐졌다!Git Interactive Rebase, Squash, Amend and Other Ways of Rewriting History

---
### 3. Commit Split
- 위에서 설명한 Commit Squash와 정반대의 일을 할 수 있는 `edit` command에 대해서 조금 더 설명할 것이 있다.

3. `edit`: 해당 커밋을 살려두고, 커밋 메세지와 작업 내용 자체를 수정할 수 있다.
    - 해당 커밋으로 HEAD가 옮겨지고, 작업 내용을 수정할 수 있다. `git commit --amend`와 같은 명령어로 커밋 메세지를 수정할 수 있고, `git rebase --continue`로 해당 리베이스 작업을 계속할 수 있다.

- `edit` 옵션을 사용하는 예시이다.

```bash
...
edit c2bbb5a style(review): apply lint, change function name with more readablility
pick 88e1067 feat(cache): implemeted cache to handle clustered payload
...
```
- `88e1067` 커밋을 살리고, `c2bbb5a`에 `edit`을 하여 수정을 한다. 저장하고 에디터를 나가면, squash와는 달리 `c2bbb5a`로 HEAD가 옮겨진 상태가 된다.
- 여기서 다음과 같은 작업을 진행한다.
```bash
$ git commit --amend -m 'style(review): apply lint' # 해당 커밋 메세지를 '린트 적용'에 대한 내용으로 바꾸고
$ git add app.spec.ts # 테스트 코드 하나 추가해서
$ git commit -m 'test(app): add simple app test code' # 테스트 코드 추가했다는 커밋 넣고
# Some Editing code
$ git add . 
$ git commit -m 'refactor(review): change function name with more readability' # 코드 수정 후 함수 이름을 바꿨다고 남긴다.
```
- 다음과 같은 작업을 진행했다.
    - 기존에 있는 커밋 내용을 (린트 + 함수 이름 수정)을 기능별로 나누고, 중간에 테스트 코드를 추가했다는 커밋을 넣었다.

- `git rebase --continue`로 해당 작업을 끝내고 리베이스를 계속 하다가, 작업을 끝내면 다음과 같은 커밋 로그가 만들어져있을 것이다.

```bash
...
cf127de refactor(review): change function name with more readability
37daea6 test(app): add simple app test code
c2bbb5a style(review): apply lint
88e1067 feat(cache): implemeted cache to handle clustered payload
...
```
- 이런 커밋 끼워팔기로 풍부한 커밋 수정이 가능하다!
---

### 4. Considerations
- Commit Squash, Commit Split 모두 커밋 로그 자체를 바꾸기 때문에, interactive rebase는 push하지 않은 작업물에 대해서 진행하는 것을 추천한다... 만약에 push한 내용에 대해서 이미 작업을 진행했다면, 마음 속의 -f는 잠시 집어 넣어두고, 다시 rebase를 진행하는 식으로 해서 잘 resolve해보자...!

---

### References
[Git Tools - Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)

[Git Rebase --Interactive 옵션 알아보기](https://wormwlrm.github.io/2020/09/03/Git-rebase-with-interactive-option.html)

[git squash - 여러개의 커밋로그를 하나로 묶기](https://meetup.nhncloud.com/posts/39)

[Git Interactive Rebase, Squash, Amend and Other Ways of Rewriting History](https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history)