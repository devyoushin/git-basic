# Git Blame & Bisect 가이드

> **카테고리**: `docs/history/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git blame`: 파일의 각 줄을 **마지막으로 수정한 커밋/작성자** 추적.
`git bisect`: 이진 탐색(Binary Search)으로 **버그를 도입한 커밋** 자동 탐색.
두 도구 모두 "언제, 누가, 왜 이렇게 됐지?" 질문에 답하는 수사 도구.

---

## 2. git blame

### 2.1 기본 사용

```bash
# 파일 전체 blame
git blame src/auth.py

# 출력 형식:
# abc1234 (John Doe 2024-01-15 10:23:45 +0900  42)     return user.id
# ^커밋   ^작성자  ^날짜                        ^줄번호  ^내용
```

### 2.2 주요 옵션

```bash
# 특정 줄 범위만 (40~60줄)
git blame -L 40,60 src/auth.py

# 함수 단위로 (함수명으로 범위 자동 탐지)
git blame -L :get_user: src/auth.py

# 짧은 해시로 표시
git blame --abbrev=8 src/auth.py

# 이메일 표시
git blame -e src/auth.py

# 파일 이름 변경 이력 추적
git blame --follow src/auth.py

# 특정 커밋 시점의 blame
git blame <COMMIT_HASH> -- src/auth.py

# 공백 변경 무시
git blame -w src/auth.py
```

### 2.3 실전 활용

```bash
# 특정 줄 변경 이유 찾기
git blame src/config.py -L 25,25
# → 커밋 해시 abc1234 확인

git show abc1234
# → 커밋 메시지와 전체 변경사항 확인
# → PR 번호나 JIRA 티켓 번호로 컨텍스트 파악

# 작성자에게 직접 물어볼 때도 blame이 출발점
```

### 2.4 IDE 통합

```bash
# VS Code: GitLens 확장 → 각 줄에 hover 시 blame 정보 표시
# IntelliJ: VCS → Annotate → 파일에서 우클릭

# CLI에서 더 보기 좋게
git log --follow -p -L 40,60:src/auth.py
# 특정 줄의 변경 이력 전체를 패치와 함께 표시
```

---

## 3. git bisect

### 3.1 개념

```text
버그 발생 상황:
  커밋 A (정상) .... 커밋 Z (버그 있음)
  중간 어딘가에서 버그 도입

이진 탐색:
  A [good] ────────── Z [bad]
             ↓ 중간 M 테스트
  A ── M [good] ───── Z
                ↓ 중간 N 테스트
         M ─── N [bad] ─ Z
           ↓ 중간 P 테스트
         M ─ P [bad]       ← 버그 도입 커밋!

O(log n): 1000개 커밋도 10번만에 찾음
```

### 3.2 수동 bisect

```bash
# 1. bisect 시작
git bisect start

# 2. 현재 버전 (버그 있음) 표시
git bisect bad

# 3. 정상이었던 버전 표시
git bisect good v1.0.0        # 태그 사용
git bisect good abc1234        # 커밋 해시 사용
git bisect good HEAD~50        # 상대 참조

# 4. Git이 중간 커밋으로 자동 이동 → 테스트
# 테스트 결과에 따라:
git bisect good    # 이 커밋은 정상
git bisect bad     # 이 커밋은 버그 있음

# 5. 반복 (Git이 범위를 계속 절반으로 줄임)
# 최종적으로:
# abc1234 is the first bad commit

# 6. bisect 종료 (원래 브랜치로 복귀)
git bisect reset
```

### 3.3 자동 bisect (스크립트 활용)

```bash
# 테스트 스크립트 작성
# test.sh — exit 0: good, exit 1: bad
#!/bin/bash
npm test -- --grep "login should work"
# 또는
python -m pytest tests/test_auth.py::test_login -q
# 또는
curl -s http://localhost:3000/health | grep -q '"status":"ok"'

# 자동 실행
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
git bisect run ./test.sh
# Git이 자동으로 good/bad 판단하며 범위 좁힘

git bisect reset   # 완료 후 복귀
```

### 3.4 bisect 중 상태 확인

```bash
# 현재 bisect 진행 상황
git bisect log              # 지금까지 good/bad 기록
git bisect visualize        # gitk GUI로 시각화
git bisect visualize --oneline   # CLI 그래프

# 현재 커밋 건너뛰기 (테스트 불가한 커밋)
git bisect skip
git bisect skip abc1234     # 특정 커밋 건너뛰기
git bisect skip v1.0.1..v1.0.3  # 범위 건너뛰기
```

---

## 4. 자주 하는 실수 & 주의사항

### blame이 잘못된 커밋을 가리키는 경우

```bash
# 단순 포맷팅/리팩토링 커밋이 blame을 가로막는 경우
# -w로 공백 무시, 또는 --ignore-rev 사용
git blame --ignore-rev <FORMATTING_COMMIT> src/auth.py

# .git-blame-ignore-revs 파일로 팀 전체 설정
echo "abc1234def5678" >> .git-blame-ignore-revs
git config blame.ignoreRevsFile .git-blame-ignore-revs
git add .git-blame-ignore-revs
```

### bisect에서 테스트 신뢰성

```bash
# 테스트 스크립트가 빌드를 포함해야 하는 경우
#!/bin/bash
npm install --silent 2>/dev/null   # 의존성 재설치
npm run build 2>/dev/null          # 빌드
npm test 2>/dev/null               # 테스트
```

### bisect reset 잊지 않기

```bash
# bisect 후 반드시 reset
git bisect reset
# 안 하면 detached HEAD 상태로 남음
```

---

## 5. 실무 팁

```bash
# blame + log 조합으로 커밋 이유까지 파악
git blame -L 100,110 src/payment.py   # 줄 범위 blame
# → 커밋 해시 확인 후
git log --format="%H %s %b" abc1234   # 커밋 메시지 상세
git show abc1234                       # 전체 변경사항

# bisect으로 성능 저하 도입 커밋 찾기
#!/bin/bash
# benchmark.sh
TIME=$(curl -s -o /dev/null -w "%{time_total}" http://localhost:3000/api/users)
python3 -c "exit(0 if float('$TIME') < 0.5 else 1)"   # 0.5초 이상이면 bad

git bisect run ./benchmark.sh

# 특정 파일 변경 이력을 blame보다 상세히
git log -p --follow -- src/auth.py    # 전체 패치 이력
git log -L :get_user:src/auth.py      # 함수 변경 이력
```

---

## 참고 자료

- [git blame 공식 문서](https://git-scm.com/docs/git-blame)
- [git bisect 공식 문서](https://git-scm.com/docs/git-bisect)
- 관련 문서: `docs/history/log-and-search.md`
- 관련 문서: `docs/history/diff-guide.md`
