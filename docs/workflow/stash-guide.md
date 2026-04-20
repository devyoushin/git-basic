# Stash 가이드

> **카테고리**: `docs/workflow/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git stash`는 작업 중인 변경사항을 임시로 저장해두고 작업 트리를 깨끗한 상태로 만드는 명령어.
브랜치 전환 전, 급한 hotfix 작업 전, 실험적 코드를 잠깐 치워둘 때 유용.
스태시는 스택(Stack) 구조 — 여러 개 쌓을 수 있고, 가장 최근 것이 `stash@{0}`.

---

## 2. 원리 / 개념 설명

### 2.1 stash 내부 구조

```text
git stash 실행 시:
  1. Working Tree 변경사항 → stash@{0}에 저장
  2. Staging Area 변경사항 → stash@{0}에 함께 저장
  3. Working Tree와 Staging Area를 HEAD 상태로 리셋

stash 스택:
  stash@{0}  ← 가장 최근 (push할 때마다 밀림)
  stash@{1}
  stash@{2}
```

### 2.2 pop vs apply 차이

```text
git stash pop:
  stash@{0} 내용을 Working Tree에 적용
  + stash 목록에서 제거 (사라짐)

git stash apply:
  stash@{0} 내용을 Working Tree에 적용
  + stash 목록에 그대로 남음 (여러 브랜치에 적용 가능)
```

---

## 3. 실습 / 명령어 예제

### 3.1 기본 사용법

```bash
# 현재 변경사항 임시 저장 (tracked 파일만)
git stash

# 메시지와 함께 저장 (나중에 구분하기 편함)
git stash push -m "로그인 기능 작업 중"

# 새 파일(untracked)도 함께 저장
git stash push -u
git stash push --include-untracked

# .gitignore 파일도 포함 (매우 드문 경우)
git stash push -a

# 스태시 목록 확인
git stash list
# stash@{0}: WIP on main: a1b2c "최근 커밋"
# stash@{1}: On feature: 로그인 기능 작업 중

# 스태시 내용 상세 확인
git stash show
git stash show -p   # diff 형태로 보기
git stash show stash@{1} -p   # 특정 스태시 내용
```

### 3.2 스태시 적용

```bash
# 가장 최근 stash 적용 + 목록에서 제거
git stash pop

# 가장 최근 stash 적용 (목록 유지)
git stash apply

# 특정 stash 적용
git stash pop stash@{2}
git stash apply stash@{2}

# 다른 브랜치에 적용하는 경우
git switch other-branch
git stash apply stash@{0}   # apply로 스태시 유지 후 적용
```

### 3.3 스태시 삭제

```bash
# 특정 stash 삭제
git stash drop stash@{0}

# 모든 stash 삭제
git stash clear
```

### 3.4 스태시로 브랜치 만들기

```bash
# stash 내용을 새 브랜치로 분리 (충돌 없이 안전하게)
git stash branch <NEW_BRANCH> stash@{0}
# → 스태시 저장 시점의 커밋에서 새 브랜치 생성 + stash 적용 + stash 제거
```

### 3.5 실전 시나리오: 급한 hotfix

```bash
# 1. 작업 중이던 내용 임시 저장
git stash push -m "feature 작업 임시 저장"

# 2. main으로 이동
git switch main

# 3. hotfix 브랜치 생성 및 수정
git switch -c hotfix/login-bug
# 수정 작업...
git commit -m "fix: 로그인 버그 수정"
git switch main
git merge hotfix/login-bug

# 4. 다시 원래 작업으로 복귀
git switch feature
git stash pop
```

---

## 4. 자주 하는 실수 & 주의사항

### stash drop 후 복구 불가

> ⚠️ WARNING: `git stash drop` 또는 `git stash clear` 후 gc가 실행되면 복구 불가. 사라지기 전에 반드시 내용 확인.

```bash
# 삭제 전 내용 확인
git stash show stash@{0} -p

# stash drop 직후 복구 시도 (gc 전이라면 가능)
git fsck --lost-found | grep commit
# dangling commit 해시로 복구 시도
git show <DANGLING_COMMIT_HASH>
git branch recovered-stash <DANGLING_COMMIT_HASH>
```

### 새 파일이 stash에 안 들어감

**원인**: `git stash`는 기본적으로 tracked 파일만 저장. 새 파일(untracked)은 제외.
```bash
# 새 파일도 함께 저장
git stash push -u
```

### stash pop 시 충돌

```bash
git stash pop
# CONFLICT 발생 시

git status   # 충돌 파일 확인
# 충돌 해결 후
git add <FILE>
git stash drop   # pop은 실패했지만 stash는 여전히 있음
# (충돌 시 자동 drop 안 됨)
```

---

## 5. 실무 팁

```bash
# 스태시가 많을 때 설명 달아두기 (필수 습관)
git stash push -m "JIRA-123 작업 중, API 연동 부분"

# 스태시 일부 내용만 적용 (interactive)
git checkout -p stash@{0}   # -p 옵션으로 hunk별 선택 적용

# 현재 스테이징된 것만 stash (작업 트리 변경은 유지)
git stash push --staged -m "staged만 저장"
```

**팀 협업 시**: 스태시는 개인 로컬에만 존재하므로 팀과 공유 불가. 작업 내용을 공유해야 한다면 `git commit --allow-empty`나 WIP 커밋 후 `push`가 나음.

---

## 참고 자료

- [git stash 공식 문서](https://git-scm.com/docs/git-stash)
- 관련 문서: `docs/workflow/cherry-pick-guide.md`
- 관련 문서: `docs/troubleshooting/undo-mistakes.md`
