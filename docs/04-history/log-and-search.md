# Git Log & 히스토리 검색 가이드

> **카테고리**: `docs/04-history/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git log`는 커밋 히스토리를 조회하는 명령어로, 옵션에 따라 출력 형태가 크게 달라짐.
특정 코드가 언제, 누가, 왜 바꿨는지 추적하는 능력은 실무에서 필수.
`--grep`, `-S`, `-G` 등의 옵션으로 커밋 내용을 강력하게 검색 가능.

---

## 2. 원리 / 개념 설명

### 2.1 log 출력 구성요소

```text
commit a1b2c3d4e5f6...        ← 커밋 해시
Author: Name <email>           ← 작성자
Date:   Mon Apr 21 10:00:00   ← 날짜

    feat: 로그인 기능 추가      ← 커밋 메시지

    - 세션 기반 인증 구현
    - 로그아웃 엔드포인트 추가
```

---

## 3. 실습 / 명령어 예제

### 3.1 기본 log 조회

```bash
# 기본 로그
git log

# 한 줄 요약 (가장 많이 사용)
git log --oneline

# 브랜치 그래프 포함 (협업 시 필수)
git log --oneline --graph --decorate --all

# 최근 N개만 보기
git log -5
git log --oneline -10

# alias로 등록해서 매일 쓰기
git config --global alias.lg "log --oneline --graph --decorate --all"
git lg
```

### 3.2 필터링 옵션

```bash
# 날짜 범위
git log --after="2024-01-01" --before="2024-12-31"
git log --since="1 week ago"
git log --until="yesterday"

# 특정 작성자
git log --author="John"
git log --author="john@company.com"

# 커밋 메시지 검색
git log --grep="feat"
git log --grep="JIRA-123"

# 대소문자 무시
git log --grep="login" -i
```

### 3.3 코드 내용으로 검색 (강력!)

```bash
# 특정 문자열이 추가/삭제된 커밋 찾기 (Pickaxe)
git log -S "getUserById"          # 이 함수가 처음 등장/사라진 커밋
git log -S "password" --all       # 전체 브랜치에서

# 정규식으로 검색
git log -G "def login.*:"         # 정규식 패턴이 변경된 커밋

# 특정 파일의 변경 이력
git log -- <FILE>
git log --follow -- <FILE>        # 파일 이름 변경 이력도 추적
git log --oneline -- src/auth.py

# 특정 함수의 변경 이력
git log -L :functionName:<FILE>
git log -L :getUserById:src/user.js
```

### 3.4 통계 및 상세 보기

```bash
# 변경된 파일 목록 포함
git log --stat

# 실제 diff 포함
git log -p
git log -p -- <FILE>   # 특정 파일만

# 변경 요약 (숫자)
git log --shortstat

# 특정 커밋 내용 보기
git show <COMMIT_HASH>
git show HEAD~2
git show HEAD~2 -- <FILE>   # 특정 파일만
```

### 3.5 브랜치 비교

```bash
# main에는 없고 feature에만 있는 커밋
git log main..feature --oneline

# 두 브랜치의 차이 전체
git log main...feature --oneline

# feature에 merge되지 않은 main 커밋
git log feature..main --oneline

# 원격과 로컬 차이
git log origin/main..main --oneline   # 로컬에만 있음 (push 안 된)
git log main..origin/main --oneline   # 원격에만 있음 (pull 안 된)
```

### 3.6 포맷 커스터마이징

```bash
# 커스텀 포맷
git log --format="%h %an %ar %s"
# %h: 짧은 해시, %an: 작성자, %ar: 상대 날짜, %s: 제목

# 색상 포함
git log --format="%C(yellow)%h%Creset %C(blue)%an%Creset %s"

# 날짜 포맷 변경
git log --date=format:"%Y-%m-%d %H:%M" --format="%ad %s"
```

---

## 4. 자주 하는 실수 & 주의사항

### `--all` 없이 log 확인

```bash
# 현재 브랜치만 보임
git log --oneline

# 모든 브랜치 보기 (권장)
git log --oneline --all
```

### 파일 이름 변경 후 이력 추적 실패

```bash
# 파일 이름이 바뀐 경우 --follow 필수
git log -- old-name.js          # 이름 바뀌기 전 이력만
git log --follow -- new-name.js  # 이름 변경 전후 이력 모두
```

---

## 5. 실무 팁

```bash
# 오늘 내가 한 작업 확인
git log --author="$(git config user.name)" --since="midnight" --oneline

# 특정 PR/브랜치의 커밋 요약 생성
git log main..feature --oneline --no-merges

# 가장 많이 커밋한 파일 (변경 빈도 높은 핫스팟)
git log --format= --name-only | sort | uniq -c | sort -rn | head -20

# 특정 기간 내 커밋 수 세기
git log --since="1 month ago" --oneline | wc -l
```

---

## 참고 자료

- [git log 공식 문서](https://git-scm.com/docs/git-log)
- 관련 문서: `docs/04-history/blame-and-bisect.md`
- 관련 문서: `docs/04-history/diff-guide.md`
