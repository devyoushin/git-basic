# PR(Pull Request) 워크플로우 가이드

> **카테고리**: `docs/03-collaboration/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

PR은 코드 병합 요청이면서 동시에 **팀 내 코드 리뷰, 지식 공유, 품질 게이트** 역할.
좋은 PR은 리뷰어가 컨텍스트 없이도 변경 이유와 방법을 이해할 수 있어야 함.
PR 크기가 작을수록 리뷰 품질이 높아지고 머지가 빨라짐.

---

## 2. PR 생성 전 체크리스트

```bash
# 1. 최신 main 반영
git fetch origin
git rebase origin/main                # 또는 merge

# 2. 커밋 정리 (WIP 커밋 squash)
git rebase -i HEAD~N                  # N개 커밋 interactive 편집

# 3. 테스트 통과 확인
npm test                              # 또는 pytest, go test 등

# 4. 불필요한 파일 확인
git diff origin/main --name-only      # 변경 파일 목록

# 5. 커밋 메시지 확인
git log origin/main..HEAD --oneline
```

---

## 3. PR 작성 가이드

### 3.1 제목

```text
✅ 좋은 PR 제목:
  feat: 카카오 소셜 로그인 추가 (#123)
  fix: 세션 만료 시 무한 리다이렉트 수정 (#456)
  refactor: 인증 로직 미들웨어로 분리

❌ 나쁜 PR 제목:
  fix bug
  WIP
  작업중
  Update files
```

### 3.2 본문 템플릿

```markdown
## 변경 사항
- 카카오 OAuth 2.0 연동
- 기존 이메일 로그인과 병행 지원
- 회원가입 없이 소셜 계정으로 자동 생성

## 변경 이유
사용자 피드백에서 소셜 로그인 요구 1위.
이메일 가입 이탈률 감소 목적.

## 테스트 방법
1. 로그인 페이지에서 '카카오로 시작하기' 클릭
2. 카카오 인증 후 메인 페이지로 리다이렉트 확인
3. 신규 유저 → DB에 kakao_id 저장 확인

## 스크린샷 (UI 변경 시)
[이미지 첨부]

## 관련 이슈
Closes #123
Refs #100

## 체크리스트
- [x] 테스트 추가/수정
- [x] 문서 업데이트
- [ ] Breaking change 없음
```

---

## 4. 리뷰 요청 & 진행

### 4.1 리뷰어 지정 원칙

```text
- 해당 코드 영역 담당자 (git blame으로 확인)
- 도메인 지식 보유자 (기술 검증)
- 팀 내 다양한 시니어리티

# blame으로 기존 담당자 확인
git blame src/auth/login.py | head -20
```

### 4.2 리뷰어 에티켓

```text
리뷰 요청자:
  ✓ PR 크기를 작게 유지 (200~400 줄이 이상적)
  ✓ 복잡한 부분에 직접 주석 달기
  ✓ WIP이면 Draft PR으로 생성
  ✓ 리뷰어 질문에 빠르게 응답

리뷰어:
  ✓ 24시간 내 초기 응답
  ✓ 코드 전체 맥락 이해 후 코멘트
  ✓ 제안과 필수 수정 구분 명확히
  ✓ 긍정적인 부분도 코멘트
```

### 4.3 리뷰 코멘트 예시

```text
# 필수 수정 (Blocker)
[MUST] 이 메서드에 try-catch 없으면 프로덕션에서 크래시 가능

# 제안 (Non-blocking)
[NIT] 변수명을 `user_id`보다 `requester_id`가 의도를 더 명확히 표현할 것 같아요

# 질문
이 로직이 동시 요청 시 Race condition이 발생할 수 있지 않을까요?

# 칭찬
이 부분 정말 깔끔하게 리팩토링됐네요!
```

---

## 5. 머지 전략

### 5.1 Squash Merge (권장 — feature → main)

```text
Before:
feature ── WIP: 절반만 구현 ── WIP: 계속 ── feat: 로그인 완성

After (main):
main ──── feat: 카카오 로그인 추가 (#123)
          (모든 WIP 커밋이 하나로 합쳐짐)
```

```bash
# GitHub UI: "Squash and merge" 선택
# CLI:
git merge --squash feature/login
git commit -m "feat: 카카오 로그인 추가 (#123)"
```

### 5.2 Rebase Merge (선형 히스토리 선호)

```bash
# GitHub UI: "Rebase and merge" 선택
# feature 브랜치의 각 커밋이 main 위로 replay됨
# 커밋이 정리되어 있어야 유용
```

### 5.3 Merge Commit (히스토리 보존)

```bash
# GitHub UI: "Create a merge commit" 선택
# feature 브랜치 히스토리 전체 보존
# release → main 등 중요한 머지에 사용
git merge --no-ff feature/login
```

---

## 6. PR 이후 정리

```bash
# 머지 후 로컬 브랜치 삭제
git switch main
git pull                               # 최신 main 가져오기
git branch -d feature/login            # 로컬 브랜치 삭제
git remote prune origin                # 원격 참조 정리

# 또는 한 번에
git fetch --prune && git branch -d feature/login
```

---

## 7. Draft PR 활용

```bash
# 작업 중 피드백을 먼저 받고 싶을 때
# GitHub UI: "Create draft pull request"

# CLI (gh 사용)
gh pr create --draft --title "feat: 로그인 구현 (WIP)"

# 준비 완료 시
gh pr ready
```

---

## 8. CI/CD 연동

```yaml
# .github/workflows/pr-check.yml
name: PR Check
on:
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test

      - name: Check lint
        run: npm run lint

      - name: Check types
        run: npm run type-check
```

---

## 9. 실무 팁

```bash
# gh CLI로 PR 관리
gh pr create --title "feat: 로그인" --body "$(cat pr-template.md)"
gh pr list --author "@me"
gh pr view 123
gh pr checkout 123              # PR 브랜치 로컬 체크아웃
gh pr merge 123 --squash       # CLI에서 머지

# PR 사이즈 체크 (변경 라인 수)
git diff origin/main --stat | tail -1
# 3 files changed, 127 insertions(+), 45 deletions(-)

# 큰 PR 분리 전략
git rebase -i origin/main      # 커밋 분리
git reset HEAD~N               # N개 커밋을 Working Tree로
# 파일 단위로 나눠서 여러 PR 생성
```

---

## 참고 자료

- [GitHub PR 가이드](https://docs.github.com/en/pull-requests)
- [gh CLI 문서](https://cli.github.com/manual/)
- 관련 문서: `docs/03-collaboration/branch-strategy.md`
- 관련 문서: `docs/03-collaboration/commit-convention.md`
