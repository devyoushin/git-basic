# Git 충돌(Conflict) 해결 가이드

> **카테고리**: `docs/troubleshooting/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

Git 충돌은 두 브랜치가 **같은 파일의 같은 위치를 다르게 수정**했을 때 발생.
Git이 자동 병합을 포기하고 사람이 직접 결정하도록 마커를 남김.
충돌은 정상적인 협업 과정이며, 당황할 필요 없음.

---

## 2. 충돌 발생 구조 이해

### 2.1 충돌 마커

```text
<<<<<<< HEAD (또는 ours)
현재 브랜치의 내용
=======
들어오는 브랜치의 내용
>>>>>>> feature/login (또는 theirs)
```

```text
<<<<<<< HEAD
    return user.find(id)
=======
    return User.objects.get(pk=id)
>>>>>>> feature/db-refactor
```

### 2.2 충돌 발생 상황

```text
Merge 충돌:    git merge <BRANCH>
Rebase 충돌:   git rebase <BRANCH>  (커밋마다 충돌 가능)
Cherry-pick:   git cherry-pick <HASH>
Stash apply:   git stash apply
Pull 충돌:     git pull (내부적으로 merge/rebase)
```

---

## 3. 충돌 해결 프로세스

### 3.1 충돌 파악

```bash
# 충돌 파일 목록 확인
git status
# both modified: src/user.py
# both modified: config/settings.json

# 충돌 위치 빠르게 확인
grep -r "<<<<<<" .   # 충돌 마커 있는 파일 찾기
```

### 3.2 충돌 해결 방법 선택

```bash
# 방법 1: 텍스트 에디터로 직접 편집
# 마커 포함 구간을 원하는 내용으로 수정 후 마커 전체 삭제

# 방법 2: ours/theirs 선택 (한쪽 통째로 채택)
git checkout --ours   <FILE>    # 현재 브랜치 내용 채택
git checkout --theirs <FILE>    # 들어오는 브랜치 내용 채택

# 방법 3: mergetool 사용 (GUI)
git mergetool                    # 기본 도구 실행
git mergetool --tool=vimdiff     # 특정 도구 지정
```

### 3.3 해결 완료 처리

```bash
# 해결한 파일 스테이징
git add <RESOLVED_FILE>

# 모든 충돌 해결 후 — merge
git merge --continue    # 또는 git commit

# 모든 충돌 해결 후 — rebase
git rebase --continue

# 해결을 포기하고 취소
git merge --abort
git rebase --abort
```

---

## 4. mergetool 설정

### 4.1 VS Code를 mergetool로 설정

```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

### 4.2 기타 도구 설정

```bash
# IntelliJ/WebStorm
git config --global merge.tool intellij
git config --global mergetool.intellij.cmd \
  'idea merge $(cd $(dirname "$LOCAL") && pwd)/$(basename "$LOCAL") \
  $(cd $(dirname "$REMOTE") && pwd)/$(basename "$REMOTE") \
  $(cd $(dirname "$BASE") && pwd)/$(basename "$BASE") \
  $(cd $(dirname "$MERGED") && pwd)/$(basename "$MERGED")'

# vimdiff (터미널)
git config --global merge.tool vimdiff
git config --global merge.conflictstyle diff3   # 공통 조상도 표시
```

### 4.3 diff3 충돌 스타일 (권장)

```bash
git config --global merge.conflictstyle diff3
```

```text
<<<<<<< HEAD
    현재 브랜치 내용
||||||| base
    두 브랜치 분기 전 원본 내용  ← diff3에서만 표시
=======
    들어오는 브랜치 내용
>>>>>>> feature
```

---

## 5. 충돌 예방 전략

### 5.1 자주 rebase/merge하기

```bash
# feature 브랜치에서 매일 main 반영
git fetch origin
git rebase origin/main

# 또는
git merge main
```

### 5.2 작은 단위로 PR

- 큰 PR은 충돌 가능성 높음
- 하나의 PR은 한 가지 목적만

### 5.3 파일 분리

- 설정 파일은 환경별로 분리 (`config.dev.json`, `config.prod.json`)
- 자동 생성 파일(`package-lock.json`, `yarn.lock`)은 특히 주의

---

## 6. 자주 발생하는 충돌 유형별 해결

### package-lock.json / yarn.lock 충돌

```bash
# 방법 1: 한쪽 채택 후 재생성
git checkout --theirs package-lock.json
npm install   # lock 파일 재생성

# 방법 2: npm의 경우 자동 병합 도구
npx npm-merge-driver install --global
```

### 마이그레이션 파일 충돌 (Django, Rails 등)

```bash
# 두 마이그레이션 파일 모두 유지 + 타임스탬프로 정렬
# ours 파일 유지
git checkout --ours migrations/0042_*.py
# 의존성(dependencies) 순서 수동 수정
```

### CHANGELOG / README 충돌

```bash
# 두 변경사항 모두 통합 (덮어쓰지 않도록)
# 직접 편집하여 두 내용 합치기
```

---

## 7. 실무 팁

```bash
# 충돌 해결 중 현재 상태 확인
git diff                          # 충돌 마커 포함 전체 diff
git diff --diff-filter=U          # 미해결 충돌 파일만

# 특정 커밋에서 파일 가져오기 (부분 채택)
git checkout <COMMIT_HASH> -- <FILE>

# 복잡한 충돌은 한쪽 채택 후 cherry-pick으로 재적용
git checkout --ours .
git add .
git merge --continue
# 이후 theirs의 변경사항을 cherry-pick으로 선택 적용

# 충돌 해결 내용을 팀원에게 설명하는 커밋 메시지
git commit -m "merge: feature/auth merge, auth.py 충돌 해결

두 브랜치 모두의 검증 로직 통합.
ours: JWT 검증, theirs: OAuth 검증 → 둘 다 유지하도록 분기 처리"
```

---

## 참고 자료

- [git merge 공식 문서](https://git-scm.com/docs/git-merge)
- 관련 문서: `docs/workflow/merge-guide.md`
- 관련 문서: `docs/workflow/rebase-guide.md`
