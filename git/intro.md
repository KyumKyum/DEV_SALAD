# Git Basics

### 0. intro
(TODO)

### 1. Git?
---

### 2. Metaphor
---

### 3. Commands

#### I. `git init`

#### II. `git add`

#### III. `git commit`

> 작업물들을 모아, 일종의 '사진'을 찍어 기록을 남기는 단계.

- 이제는, 추가한 파일들에 대해서 '기록'을 남겨야 한다.
- 일반적으로, 커밋 로그는 다음과 같은 방법으로 남긴다.

```bash
git commit -m "feat(login): implement bcrypt hash provider to save password with securacy"
```

-  이제 우리는 기록이 하나 생겼다. 언제든지 우리는 이 곳으로 다시 돌아올 수도 있고 (`git revert`/`git reset`), 작업을 했다는 것의 증거물이 될 수 있다. (`git log`/`git blame`).
- commit은 정말로 중요하다. (TODO)

---
