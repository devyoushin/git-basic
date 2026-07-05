# Git Diff 가이드

> **카테고리**: `docs/04-history/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git diff`는 두 Git 상태 사이의 변경사항을 비교.
어떤 두 시점을 비교하느냐에 따라 옵션이 달라지며,
`--staged`, `HEAD`, 커밋 해시, 브랜치명 등을 조합해서 사용.

---

## 2. diff 비교 대상 이해

```text
Working Tree ←→ Staging Area ←→ 커밋 히스토리

git diff              : Working Tree ↔ Staging Area
git diff --staged     : Staging Area ↔ 마지막 커밋 (HEAD)
git diff HEAD         : Working Tree ↔ 마지막 커밋 (전체)

git diff <HASH>       : Working Tree ↔ 특정 커밋
git diff <A>..<B>     : 커밋 A ↔ 커밋 B
git diff <BRANCH>     : 현재 브랜치 ↔ 다른 브랜치
```

---

## 3. 실습 / 명령어 예제

### 3.1 기본 diff

```bash
# 스테이징 전 변경사항 (가장 많이 사용)
git diff

# 스테이징된 변경사항 (커밋 직전 최종 확인)
git diff --staged
git diff --cached          # 동일

# 스테이징 전후 모두 (HEAD 대비 전체)
git diff HEAD
```

### 3.2 커밋 간 비교

```bash
# 두 커밋 비교
git diff abc1234 def5678

# HEAD 기준
git diff HEAD~1            # 직전 커밋 대비
git diff HEAD~3            # 3커밋 전 대비
git diff HEAD~3 HEAD       # HEAD~3 ↔ HEAD

# 특정 파일만 비교
git diff HEAD~2 -- src/auth.py
git diff abc1234 def5678 -- package.json
```

### 3.3 브랜치 비교

```bash
# 현재 브랜치 vs main
git diff main

# 두 브랜치 비교 (분기점 이후 변경사항)
git diff main..feature          # main ↔ feature 끝
git diff main...feature         # 분기 이후 feature 변경사항만

# 원격 vs 로컬
git diff origin/main main       # 로컬 main이 원격보다 뒤처진 경우
git fetch && git diff origin/main
```

### 3.4 파일 목록만 보기

```bash
# 변경된 파일 이름만 (stat 없이)
git diff --name-only
git diff --name-only HEAD~3 HEAD

# 변경 상태 포함 (A: added, M: modified, D: deleted)
git diff --name-status HEAD~1

# 통계 (삽입/삭제 줄 수)
git diff --stat
git diff --stat HEAD~5 HEAD
```

### 3.5 특정 파일/폴더만

```bash
# 특정 파일
git diff -- src/auth.py

# 특정 폴더
git diff -- src/

# 여러 파일
git diff -- src/auth.py src/user.py
```

### 3.6 단어/공백 무시

```bash
# 공백 변경 무시
git diff -w
git diff --ignore-all-space

# 줄 끝 공백만 무시
git diff --ignore-space-at-eol

# 단어 단위 diff (가독성 향상)
git diff --word-diff

# 색상 포함 단어 diff
git diff --word-diff=color
```

---

## 4. difftool 설정

### 4.1 VS Code를 difftool로 설정

```bash
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# 사용
git difftool HEAD~1 -- src/auth.py
```

### 4.2 기타 도구

```bash
# vimdiff
git config --global diff.tool vimdiff

# meld (Linux GUI)
git config --global diff.tool meld

# IntelliJ
git config --global diff.tool intellij
git config --global difftool.intellij.cmd 'idea diff $(cd $(dirname "$LOCAL") && pwd)/$(basename "$LOCAL") $(cd $(dirname "$REMOTE") && pwd)/$(basename "$REMOTE")'

# 프롬프트 없이 바로 열기
git config --global difftool.prompt false
```

---

## 5. 자주 하는 실수 & 주의사항

### staged vs HEAD 혼동

```bash
git add src/auth.py

git diff            # 아무것도 안 나옴 (스테이징 완료 상태)
git diff --staged   # 스테이징된 변경사항 확인 ← 이걸 써야
```

### 브랜치 비교 .. vs ...

```bash
# git diff A..B  : A 끝 ↔ B 끝 (단순 비교)
git diff main..feature
# A와 B 사이의 모든 변경사항

# git diff A...B : 분기점 이후 B의 변경만
git diff main...feature
# feature 브랜치가 생긴 이후에 feature에서만 변경된 것
# PR 리뷰 시 이 쪽이 더 유용
```

---

## 6. 실무 팁

```bash
# 커밋 전 최종 확인 루틴
git diff --staged          # 스테이징 확인
git diff --staged --stat   # 변경 파일 요약

# PR 전 변경 규모 파악
git diff origin/main --stat | tail -5

# 특정 기간 변경사항 확인
git diff HEAD@{1.week.ago} HEAD --stat

# 이진 파일 변경 여부만 확인
git diff --binary HEAD~1 | grep "^Binary"

# 외부 도구 없이 색상 강조 diff
git diff --color-words
git diff --color-words="[^[:space:]]"
```

---

## 참고 자료

- [git diff 공식 문서](https://git-scm.com/docs/git-diff)
- 관련 문서: `docs/04-history/log-and-search.md`
- 관련 문서: `docs/04-history/blame-and-bisect.md`
