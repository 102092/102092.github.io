---
author: Han
title: 기억해야할 Git 명령어
date: 2021-06-20
description: Git command for you..
tags: ['git', 'command']
ShowToc: true
---


# Rebase root commit

```git
git rebase -i --root
```

> 참고
- https://stackoverflow.com/questions/22992543/how-do-i-git-rebase-the-first-commit/23000315


# Update commit author

1. 수정하고 싶은 직전 커밋의 Hash 확인

2. git rebase

```git
git rebase -i -p 커밋hash
```

- 위 명령어 입력할 경우, 해당 해쉬 커밋 이후 부터 모든 커밋이 리베이스 대상이됨.

3. rebase 모드에서, 수정하고자하는 커밋 상태를 `e` 로 변경

4. author 수정

```git
git commit --amend --author="사용자명 <이메일>"
```

5. rebase --continue

```git
git rebase --continue
```


> 참고
- https://jojoldu.tistory.com/120