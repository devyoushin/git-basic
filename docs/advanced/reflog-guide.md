# Git Reflog 가이드

> **카테고리**: `docs/advanced/`
> **관련 에이전트**: `agents/rescue-agent.md`

---

## 1. 개요

`git reflog`는 HEAD의 모든 이동 기록을 보여주는 "블랙박스".
reset --hard, rebase, 브랜치 삭제 등으로 "사라진 것처럼 보이는" 커밋을 복구하는 데 핵심.
기본적으로 90일간 보관되며, `git gc` 실행 전까지는 거의 모든 실수를 복구 가능.

---

## 2. 원리 / 개념 설명

### 2.1 reflog vs log 차이

```text
git log:
  현재 브랜치의 커밋 히스토리 (연결된 것만)

git reflog:
  HEAD가 이동한 모든 기록 (reset, checkout, rebase, merge 등 포함)
  연결이 끊긴 고아 커밋도 기록에 남음
```

### 2.2 reflog 저장 위치

```bash
# 로컬 파일로 저장됨
cat .git/logs/HEAD

# 브랜치별로도 별도 저장
cat .git/logs/refs/heads/main
```

### 2.3 reflog 참조 방법

```text
HEAD@{0}  ← 현재 HEAD
HEAD@{1}  ← 한 번 전 HEAD 위치
HEAD@{2}  ← 두 번 전
HEAD@{5 minutes ago}  ← 5분 전 상태
HEAD@{yesterday}      ← 어제 상태
```

---

## 3. 실습 / 명령어 예제

### 3.1 reflog 조회

```bash
# HEAD 이동 기록 전체
git reflog

# 특정 브랜치의 reflog
git reflog show main
git reflog show origin/main

# 날짜 포함해서 보기
git reflog --date=iso

# 최근 10개만
git reflog | head -10
```

### 3.2 실수 복구 시나리오

#### 시나리오 1: reset --hard로 커밋 날린 경우

```bash
# 실수
git reset --hard HEAD~3   # 커밋 3개가 사라진 것처럼 보임

# 복구
git reflog
# HEAD@{0}: reset: moving to HEAD~3
# HEAD@{1}: commit: feat: 세 번째 기능  ← 이걸 복구
# HEAD@{2}: commit: feat: 두 번째 기능
# HEAD@{3}: commit: feat: 첫 번째 기능

git reset --hard HEAD@{3}   # 또는 해시 직접 사용
```

#### 시나리오 2: 브랜치 삭제한 경우

```bash
# 실수
git branch -D feature   # 브랜치 삭제

# 복구 - reflog에서 마지막 커밋 찾기
git reflog | grep feature
# HEAD@{4}: checkout: moving from feature to main

git switch -c feature HEAD@{4}
# 또는
git branch feature HEAD@{4}
```

#### 시나리오 3: rebase 후 원래대로 돌아가기

```bash
# 실수: rebase 결과가 마음에 안 듦
git rebase -i main   # interactive rebase 실행

# 복구: ORIG_HEAD 활용 (rebase 직전 자동 저장)
git reset --hard ORIG_HEAD

# ORIG_HEAD가 없으면 reflog로
git reflog
# HEAD@{0}: rebase (finish): returning to refs/heads/feature
# HEAD@{1}: rebase (start): checkout main
# HEAD@{2}: commit: 내가 원하는 원래 상태  ← 이걸 복구
git reset --hard HEAD@{2}
```

#### 시나리오 4: Detached HEAD에서 커밋 후 브랜치 전환해서 잃어버린 경우

```bash
# 실수: Detached HEAD 상태에서 커밋 후 switch 해버림
git checkout abc1234   # Detached HEAD
# 작업 후
git commit -m "중요한 작업"   # 이 커밋이 고아가 됨
git switch main   # 커밋이 어디에도 없어진 것처럼 보임

# 복구
git reflog
# HEAD@{0}: checkout: moving from abc1234... to main
# HEAD@{1}: commit: 중요한 작업   ← 이 해시!
git branch recovered HEAD@{1}
```

### 3.3 특정 시점으로 복구

```bash
# 1시간 전 상태로
git reset --hard HEAD@{1 hour ago}

# 어제 오전 10시 상태로 (ISO 날짜)
git reset --hard 'HEAD@{2024-01-15 10:00:00}'

# n번 전 상태로
git checkout HEAD@{5}   # 5번 전 HEAD 위치를 Detached HEAD로 확인
```

---

## 4. 자주 하는 실수 & 주의사항

### reflog로도 복구 불가한 경우

> ⚠️ WARNING: 아래 상황에서는 복구 불가능.

```bash
# 1. commit한 적 없는 파일 삭제
git clean -fd   # untracked 파일 삭제 → 복구 불가

# 2. stash drop 후 gc 실행
git stash drop
git gc   # gc 실행 시 참조 없는 객체 정리됨

# 3. reflog 만료 후 (기본 90일)
git config gc.reflogExpire   # 만료 기간 확인
```

### reflog는 로컬 전용

reflog는 각 개발자의 로컬 레포에만 존재. 원격(GitHub)에는 없음.
팀원이 force push로 덮어쓴 커밋은 **그 팀원의 로컬 reflog**에서만 복구 가능.

---

## 5. 실무 팁

```bash
# 복구 전 항상 현재 상태 백업
git branch backup/before-recovery-$(date +%Y%m%d)

# reflog 만료 기간 늘리기 (기본 90일 → 180일)
git config --global gc.reflogExpire "180 days"
git config --global gc.reflogExpireUnreachable "90 days"

# 고아 커밋 전체 목록 확인 (gc 전)
git fsck --lost-found
# dangling commit 해시들 출력됨
git show <DANGLING_HASH>   # 내용 확인 후 복구 결정
```

**황금 규칙**: 뭔가 잘못됐다 싶으면 **`git reflog`부터** 실행. 거의 모든 실수가 여기서 복구됨.

---

## 참고 자료

- [git reflog 공식 문서](https://git-scm.com/docs/git-reflog)
- 관련 문서: `docs/history/reset-and-revert.md`
- 관련 문서: `docs/troubleshooting/undo-mistakes.md`
- 빠른 참조: `cheatsheets/emergency.md`
