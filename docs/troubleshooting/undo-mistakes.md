# Git 실수 복구 가이드

> **카테고리**: `docs/troubleshooting/`
> **관련 에이전트**: `agents/rescue-agent.md`

---

## 1. 개요

Git 작업 중 발생하는 실수 유형별 복구 방법 모음.
복구 성공률은 실수를 발견한 시점과 종류에 달려 있음.
**황금 원칙: 발견하는 즉시 `git reflog` 실행, 패닉하지 않기.**

---

## 2. 복구 가능/불가 기준

```text
복구 가능:
  ✓ git commit 한 내용은 90일간 reflog에 보관
  ✓ git add한 파일은 Git 객체 데이터베이스에 있음
  ✓ stash도 reflog에 남음 (gc 전까지)

복구 불가:
  ✗ commit 하지 않고 삭제한 파일 (git clean -fd)
  ✗ commit 하지 않은 Working Tree 변경사항 (reset --hard)
  ✗ git gc 실행 후 참조 없는 객체
  ✗ reflog 만료(90일) 이후
```

---

## 3. 실수 유형별 복구

### 3.1 커밋 메시지 잘못 작성

```bash
# push 전 — amend로 수정
git commit --amend -m "올바른 메시지"

# push 후 (혼자 쓰는 브랜치) — amend + force push
git commit --amend -m "올바른 메시지"
git push --force-with-lease origin <BRANCH>

# push 후 (공유 브랜치) — revert는 불가, 그냥 두거나 팀원 동의 후 force push
```

---

### 3.2 파일을 잘못된 커밋에 포함

```bash
# push 전 — 마지막 커밋에서 파일 제거
git reset HEAD~1                       # 커밋 취소 (변경사항 유지)
git restore --staged <WRONG_FILE>      # 잘못된 파일 스테이징 해제
git checkout -- <WRONG_FILE>           # 변경사항 버리기
git add <CORRECT_FILES>
git commit -m "올바른 커밋"

# 여러 커밋 전 파일이면 — interactive rebase
git rebase -i HEAD~N                   # N번 전까지 편집
# 해당 커밋을 edit으로 변경
# 파일 수정 후
git add <FILES>
git commit --amend --no-edit
git rebase --continue
```

---

### 3.3 잘못된 브랜치에 커밋

```bash
# 상황: main에 직접 커밋해버린 경우

# 1. 올바른 브랜치로 커밋 이동
git branch feature/my-work   # 현재 커밋으로 새 브랜치 생성

# 2. main에서 커밋 제거
git switch main
git reset --hard HEAD~1      # 또는 HEAD~N (N개 커밋)

# 3. 올바른 브랜치로 이동
git switch feature/my-work

# push 후라면 — main 보호 규칙 확인 필요
git push --force-with-lease origin main   # ⚠️ 팀 공유 브랜치면 위험
```

---

### 3.4 reset --hard로 커밋 날린 경우

```bash
git reflog
# HEAD@{0}: reset: moving to HEAD~3
# HEAD@{1}: commit: feat: 세 번째 기능  ← 복구 목표
# HEAD@{2}: commit: feat: 두 번째 기능
# HEAD@{3}: commit: feat: 첫 번째 기능

git reset --hard HEAD@{3}   # 세 커밋 모두 복구
# 또는
git reset --hard <COMMIT_HASH>
```

---

### 3.5 브랜치 삭제한 경우

```bash
# 방법 1: reflog에서 마지막 커밋 찾기
git reflog | grep <BRANCH_NAME>
# HEAD@{4}: checkout: moving from <BRANCH_NAME> to main

git branch <BRANCH_NAME> HEAD@{4}

# 방법 2: 커밋 해시를 기억하면
git branch <BRANCH_NAME> <COMMIT_HASH>

# 원격에도 있으면
git push origin <BRANCH_NAME>
```

---

### 3.6 push 한 커밋 취소

```bash
# 혼자 쓰는 브랜치 — reset + force push
git reset --soft HEAD~1        # 변경사항 유지
git push --force-with-lease origin <BRANCH>

# 공유 브랜치 (main/develop) — revert (히스토리 보존)
git revert <COMMIT_HASH>
git push origin main

# 여러 커밋 revert
git revert HEAD~3..HEAD         # 각각 revert 커밋 생성
git revert --no-commit HEAD~3..HEAD && git commit -m "revert: 기능 A 제거"
```

---

### 3.7 git stash 날린 경우

```bash
# stash는 gc 전까지 dangling 객체로 존재
git fsck --lost-found
# dangling commit abc1234
# dangling commit def5678

# 내용 확인
git show abc1234

# 맞는 것 적용
git stash apply abc1234
# 또는
git branch recovered-stash abc1234
```

---

### 3.8 파일 하나를 이전 상태로 복구

```bash
# 마지막 커밋 상태로 (staged 변경사항도 취소)
git restore <FILE>

# 스테이징만 취소 (Working Tree 변경사항 유지)
git restore --staged <FILE>

# N커밋 전 상태로
git restore --source=HEAD~2 -- <FILE>

# 특정 커밋 시점으로
git restore --source=<COMMIT_HASH> -- <FILE>

# 특정 브랜치의 파일로 교체
git restore --source=main -- <FILE>
```

---

### 3.9 잘못된 merge/rebase 취소

```bash
# merge 취소 (진행 중)
git merge --abort

# merge 취소 (완료 후, push 전)
git reset --hard ORIG_HEAD   # merge 직전으로 복구

# rebase 취소 (진행 중)
git rebase --abort

# rebase 취소 (완료 후)
git reset --hard ORIG_HEAD
# 또는 reflog에서
git reflog
git reset --hard HEAD@{N}    # rebase 시작 전 N 확인
```

---

### 3.10 sensitive 파일을 push한 경우 (토큰, 패스워드)

```bash
# ⚠️ 즉시 토큰/패스워드 폐기 (무조건 먼저)

# 히스토리에서 파일 완전 제거
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch <SENSITIVE_FILE>" \
  --prune-empty --tag-name-filter cat -- --all

# 또는 BFG Repo Cleaner (더 빠름)
bfg --delete-files <SENSITIVE_FILE>
git push --force origin --all

# .gitignore에 추가
echo "<SENSITIVE_FILE>" >> .gitignore
git add .gitignore
git commit -m "chore: sensitive 파일 gitignore 추가"
```

---

## 4. 복구 전 체크리스트

```bash
# 1. 현재 상태 파악
git status
git log --oneline -10

# 2. 현재 위치 백업 (안전망)
git branch backup/$(date +%Y%m%d-%H%M)

# 3. reflog 확인
git reflog | head -20

# 4. 복구 커밋 내용 미리 확인
git show <TARGET_HASH>
```

---

## 5. 실무 팁

```bash
# reset 전 자동 백업 alias
git config --global alias.safe-reset '!git branch backup/$(date +%Y%m%d-%H%M) && git reset'

# 위험한 명령어 실행 전 습관
git stash                              # 현재 변경사항 임시 저장
git branch backup/before-experiment   # 현재 커밋 백업
```

---

## 참고 자료

- 관련 문서: `docs/advanced/reflog-guide.md`
- 관련 문서: `docs/history/reset-and-revert.md`
- 빠른 참조: `cheatsheets/emergency.md`
