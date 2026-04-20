# 브랜치 가이드

> **카테고리**: `docs/basics/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

Git 브랜치(Branch)는 특정 커밋을 가리키는 가벼운 포인터(pointer).
브랜치를 만들어도 파일을 복사하지 않고 포인터만 추가하므로 매우 빠르고 저렴.
HEAD는 "현재 작업 중인 브랜치(또는 커밋)"를 가리키는 특별한 포인터.

---

## 2. 원리 / 개념 설명

### 2.1 브랜치의 내부 구조

```text
# 브랜치는 단순히 커밋 해시를 담은 파일
$ cat .git/refs/heads/main
a1b2c3d4...   ← 최신 커밋 해시

# HEAD는 현재 브랜치를 가리키는 파일
$ cat .git/HEAD
ref: refs/heads/main   ← main 브랜치를 가리킴
```

### 2.2 브랜치 생성 & 전환 시각화

```text
# 초기 상태
A - B - C  (HEAD → main)

# git branch feature
A - B - C  (HEAD → main, feature)
            ↑ 포인터만 하나 더 추가됨 (파일 복사 없음!)

# git switch feature
A - B - C  (main, HEAD → feature)

# feature에서 커밋
A - B - C  (main)
            \
             D  (HEAD → feature)
```

### 2.3 HEAD 개념

```text
HEAD → main → C   : 일반 상태 (main 브랜치 체크아웃)
HEAD → C          : Detached HEAD (브랜치 없이 커밋 직접 체크아웃)
HEAD~1            : HEAD에서 한 칸 뒤 커밋
HEAD~3            : HEAD에서 세 칸 뒤 커밋
HEAD^             : 첫 번째 부모 (= HEAD~1)
HEAD^2            : 두 번째 부모 (merge 커밋에서 사용)
```

---

## 3. 실습 / 명령어 예제

### 3.1 브랜치 생성 & 전환

```bash
# 브랜치 목록 (로컬)
git branch

# 브랜치 목록 (원격 포함)
git branch -a

# 브랜치 생성
git branch <BRANCH_NAME>

# 브랜치 생성 + 즉시 전환 (권장)
git switch -c <BRANCH_NAME>
git checkout -b <BRANCH_NAME>   # 구식 방법 (동일 효과)

# 브랜치 전환
git switch <BRANCH_NAME>
git checkout <BRANCH_NAME>      # 구식 방법

# 특정 커밋에서 브랜치 생성
git switch -c <BRANCH_NAME> <COMMIT_HASH>

# 원격 브랜치를 로컬로 가져오기
git switch -c <BRANCH_NAME> origin/<BRANCH_NAME>
git checkout --track origin/<BRANCH_NAME>   # 동일 효과
```

### 3.2 브랜치 삭제 & 이름 변경

```bash
# 브랜치 삭제 (merge된 브랜치만)
git branch -d <BRANCH_NAME>

# 강제 삭제 (merge 안 된 브랜치도)
git branch -D <BRANCH_NAME>

# 원격 브랜치 삭제
git push origin --delete <BRANCH_NAME>

# 브랜치 이름 변경 (현재 브랜치)
git branch -m <NEW_NAME>

# 브랜치 이름 변경 (다른 브랜치)
git branch -m <OLD_NAME> <NEW_NAME>

# 원격에 새 이름으로 push + 구 브랜치 삭제
git push origin -u <NEW_NAME>
git push origin --delete <OLD_NAME>
```

### 3.3 Detached HEAD 상태

```bash
# 특정 커밋을 직접 체크아웃 → Detached HEAD
git checkout <COMMIT_HASH>
git checkout v1.2.0   # 태그도 가능

# Detached HEAD 상태에서 작업 후 저장하려면
git switch -c <NEW_BRANCH>   # 새 브랜치로 저장
# 안 하면 switch 시 커밋이 고아(orphan)가 됨

# 원래 브랜치로 돌아가기
git switch main
```

### 3.4 브랜치 정보 확인

```bash
# 각 브랜치의 최신 커밋 확인
git branch -v

# 특정 브랜치에서 merge된 브랜치 목록
git branch --merged main

# merge되지 않은 브랜치 목록 (삭제 전 확인용)
git branch --no-merged main

# 브랜치가 어느 커밋에서 갈라졌는지 확인
git merge-base main feature
```

---

## 4. 자주 하는 실수 & 주의사항

### 잘못된 브랜치에서 작업 시작

**상황**: main에서 바로 작업해버린 경우
```bash
# 아직 commit 전 (작업 트리에만 변경사항)
git switch -c <NEW_BRANCH>   # 변경사항이 새 브랜치로 그대로 이동

# 이미 commit 한 경우
git switch -c <NEW_BRANCH>   # 새 브랜치에서 커밋 유지
git switch main
git reset HEAD~1             # main에서 커밋 제거 (변경사항은 유지)
```

### Detached HEAD에서 커밋하고 브랜치 전환

> ⚠️ WARNING: Detached HEAD 상태에서 커밋 후 브랜치를 전환하면 해당 커밋은 고아(orphan) 상태가 됨. `git reflog`로 복구는 가능하나 주의 필요.

```bash
# Detached HEAD에서 작업 저장하려면 반드시
git switch -c new-branch-name
```

### 원격 브랜치 삭제 후 로컬 추적 정보 남아있음

```bash
# 원격에서 삭제된 브랜치 로컬 참조도 정리
git fetch --prune
git remote prune origin
```

---

## 5. 실무 팁

```bash
# 최근에 작업한 브랜치 순서로 목록 보기
git branch --sort=-committerdate

# 브랜치 간 파일 비교
git diff main..feature -- <FILE>

# 특정 브랜치의 파일을 현재 브랜치로 가져오기
git checkout <BRANCH_NAME> -- <FILE>

# 브랜치 없이 임시 저장 후 작업 → stash 참조
# docs/workflow/stash-guide.md
```

**팀 협업 시**: `main`, `develop` 등 공유 브랜치는 직접 커밋 금지. 항상 feature 브랜치 → PR → merge 흐름 유지. 브랜치 이름에 이슈 번호 포함 권장: `feature/123-login`.

---

## 참고 자료

- [Git 브랜치 공식 문서](https://git-scm.com/book/ko/v2/Git-브랜치-브랜치란-무엇인가)
- 관련 문서: `docs/workflow/merge-guide.md`
- 관련 문서: `docs/workflow/rebase-guide.md`
- 관련 문서: `docs/advanced/reflog-guide.md`
