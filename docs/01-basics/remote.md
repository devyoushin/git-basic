# 원격 저장소 (Remote) 가이드

> **카테고리**: `docs/01-basics/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

원격 저장소(Remote Repository)는 GitHub, GitLab 등 서버에 올라간 Git 레포지토리.
`origin`은 clone할 때 자동으로 붙는 원격 이름이며, 여러 개의 원격을 동시에 등록 가능.
`fetch`는 원격 변경사항을 가져오기만 하고, `pull`은 가져오고 바로 merge(또는 rebase)까지 수행.

---

## 2. 원리 / 개념 설명

### 2.1 원격 추적 브랜치 (Remote Tracking Branch)

```text
로컬 저장소:
  main          ← 내가 직접 작업하는 브랜치
  origin/main   ← 원격 main의 마지막 동기화 상태 (읽기 전용)

git fetch 실행 후:
  원격의 최신 커밋 → origin/main 이동
  내 main은 그대로 (아직 merge 전)

git merge origin/main 실행 후:
  내 main이 origin/main을 따라잡음
```

### 2.2 fetch vs pull 차이

```text
git fetch:
  원격 → origin/main 업데이트
  내 main: 변화 없음 (안전)

git pull (= git fetch + git merge):
  원격 → origin/main 업데이트
  내 main: 자동 merge

git pull --rebase (= git fetch + git rebase):
  원격 → origin/main 업데이트
  내 main: rebase로 깔끔하게 적용
```

---

## 3. 실습 / 명령어 예제

### 3.1 원격 관리

```bash
# 원격 목록 확인
git remote -v

# 원격 추가
git remote add origin https://github.com/<USER>/<REPO>.git
git remote add upstream https://github.com/<ORIGINAL>/<REPO>.git  # fork 원본

# 원격 URL 변경 (HTTPS → SSH 등)
git remote set-url origin git@github.com:<USER>/<REPO>.git

# 원격 삭제
git remote remove <REMOTE>

# 원격 이름 변경
git remote rename origin upstream
```

### 3.2 fetch & pull

```bash
# 원격 변경사항 가져오기 (merge 안 함)
git fetch origin
git fetch --all     # 모든 원격

# 가져온 내용 확인 후 merge
git log origin/main --oneline -5   # 원격 내용 확인
git diff main origin/main          # 내 브랜치와 차이 확인
git merge origin/main              # 직접 merge

# fetch + merge 한 번에
git pull origin main

# fetch + rebase 한 번에 (깔끔한 히스토리 선호 시)
git pull --rebase origin main

# 원격에서 삭제된 브랜치 정리
git fetch --prune
```

### 3.3 push

```bash
# 현재 브랜치를 원격으로 push
git push origin main

# 처음 push 시 추적 관계 설정 (-u 또는 --set-upstream)
git push -u origin <BRANCH_NAME>
# 이후부터는 git push 만으로 가능

# 모든 브랜치 push
git push --all origin

# 태그 push
git push origin <TAG>
git push --tags       # 모든 태그

# 원격 브랜치 삭제
git push origin --delete <BRANCH_NAME>
```

### 3.4 Fork 레포 관리 (upstream 동기화)

```bash
# 1. upstream 등록 (fork 원본)
git remote add upstream https://github.com/<ORIGINAL>/<REPO>.git

# 2. upstream 변경사항 가져오기
git fetch upstream

# 3. 내 main을 upstream과 동기화
git switch main
git merge upstream/main   # 또는 rebase

# 4. 내 fork에 반영
git push origin main
```

---

## 4. 자주 하는 실수 & 주의사항

### git pull이 충돌 발생

**상황**: 원격과 로컬 모두에 변경사항이 있을 때
```bash
# 충돌 해결 흐름
git pull origin main   # 충돌 발생

# 충돌 파일 확인
git status
# 파일을 열어 <<<<<<, =======, >>>>>>> 마커 수동 해결 후

git add <RESOLVED_FILE>
git commit   # merge 커밋 완성
```

### push rejected (원격이 앞서 있음)

**증상**: `! [rejected] main -> main (non-fast-forward)`
**원인**: 원격에 내가 모르는 커밋이 있음
```bash
# 해결: 먼저 pull 후 push
git pull origin main   # 원격 내용 가져와서 merge
git push origin main   # 그 다음 push
```

### force push로 팀원 커밋 덮어쓰기

> ⚠️ WARNING: 공유 브랜치(main, develop)에서 `--force` push는 팀원 커밋 손실 위험.

```bash
# 절대 금지
git push --force origin main

# 대신 사용 (안전: 원격이 예상한 상태일 때만 push)
git push --force-with-lease origin main
```

---

## 5. 실무 팁

```bash
# 원격 브랜치 목록 최신화
git remote update

# 내 브랜치가 원격보다 몇 커밋 앞/뒤인지 확인
git status   # "Your branch is ahead of 'origin/main' by 2 commits"

# 원격 브랜치를 로컬로 바로 체크아웃
git switch -c feature origin/feature

# push 기본 동작 설정 (current: 같은 이름의 원격 브랜치로만 push)
git config --global push.default current
```

**팀 협업 시**: `git pull --rebase`를 기본으로 설정하면 불필요한 merge 커밋 없이 깔끔한 히스토리 유지 가능.

```bash
git config --global pull.rebase true
```

---

## 참고 자료

- [Git 원격 저장소 공식 문서](https://git-scm.com/book/ko/v2/Git의-기초-리모트-저장소)
- 관련 문서: `docs/02-workflow/merge-guide.md`
- 관련 문서: `docs/06-troubleshooting/conflict-resolution.md`
