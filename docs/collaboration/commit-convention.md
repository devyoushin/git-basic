# Commit Convention 가이드

> **카테고리**: `docs/collaboration/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

좋은 커밋 메시지는 **왜** 이 변경을 했는지 미래의 나와 팀원에게 설명.
`git log`가 문서가 되고, 디버깅 시 히스토리 검색이 가능해짐.
**Conventional Commits** 형식이 업계 표준으로 자리잡은 상태.

---

## 2. Conventional Commits 형식

### 2.1 기본 구조

```text
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### 2.2 타입(type) 목록

| 타입 | 용도 | 예시 |
|------|------|------|
| `feat` | 새 기능 추가 | `feat: 카카오 로그인 추가` |
| `fix` | 버그 수정 | `fix: 세션 만료 시 crash 수정` |
| `docs` | 문서 변경 | `docs: API 응답 형식 설명 추가` |
| `style` | 코드 스타일 (기능 무관) | `style: 세미콜론 정리` |
| `refactor` | 리팩토링 (기능/버그 무관) | `refactor: 인증 미들웨어 분리` |
| `test` | 테스트 추가/수정 | `test: 로그인 실패 케이스 추가` |
| `chore` | 빌드, 설정 변경 | `chore: ESLint 규칙 추가` |
| `ci` | CI/CD 설정 | `ci: GitHub Actions 배포 추가` |
| `build` | 빌드 시스템 변경 | `build: webpack 5 업그레이드` |
| `perf` | 성능 개선 | `perf: DB 쿼리 인덱스 추가` |
| `revert` | 커밋 되돌리기 | `revert: feat: 카카오 로그인 추가` |

### 2.3 실제 예시

```text
feat(auth): 카카오 소셜 로그인 추가

카카오 OAuth 2.0 연동.
기존 이메일 로그인과 병행 지원.

- KakaoLoginButton 컴포넌트 추가
- /api/auth/kakao 엔드포인트 구현
- 사용자 프로필 자동 생성 로직 포함

JIRA-1234
```

```text
fix(payment): 결제 금액 소수점 계산 오류 수정

원화 결제 시 0.1원 단위 반올림 누락으로
실제 청구 금액과 DB 저장 금액 불일치 발생.

Fixes #567
```

---

## 3. 작성 규칙

### 3.1 제목 (description) 규칙

```text
✅ 올바른 예:
  feat: 사용자 프로필 이미지 업로드 기능 추가
  fix: null 포인터 예외 처리 누락 수정
  refactor: 중복 유효성 검사 로직 통합

❌ 잘못된 예:
  fix: fixed bug              (구체적이지 않음)
  Update code                 (타입 없음)
  feat: 기능을 추가했습니다.   (마침표 금지, 과거형 금지)
  WIP                         (커밋 완성 후 정리)
```

```bash
# 규칙 요약
- 제목 72자 이내
- 타입 후 콜론 + 공백
- 동사 원형/현재형으로 시작 (추가, 수정, 제거...)
- 마침표 없음
- 명령문 형식: "Add feature" not "Added feature"
```

### 3.2 본문(body) 작성 시

```text
- 제목과 빈 줄로 구분
- 각 줄 72자 이내
- HOW보다 WHY 설명 (코드는 HOW를 보여주니까)
- 불릿 포인트 사용 가능
```

### 3.3 푸터(footer)

```text
# Breaking Change
BREAKING CHANGE: auth API 응답 형식 변경
  이전: { token: "..." }
  이후: { access_token: "...", refresh_token: "..." }

# 이슈 참조
Fixes #123          # 이슈 자동 닫기
Closes #456
Refs #789           # 참조만 (닫지 않음)

# 관련 커밋
Co-authored-by: Name <email>
```

---

## 4. Breaking Change

```bash
# 방법 1: 타입 뒤에 ! 추가
feat!: API 인증 방식 변경

# 방법 2: 푸터에 명시
feat(api): 인증 방식 JWT로 전환

기존 세션 기반에서 JWT로 전환.

BREAKING CHANGE: Authorization 헤더 형식 변경
  이전: Session <session_id>
  이후: Bearer <jwt_token>
```

---

## 5. Commit Hook으로 형식 자동 검증

```bash
# .git/hooks/commit-msg
#!/bin/sh
PATTERN="^(feat|fix|docs|style|refactor|test|chore|ci|build|perf|revert)(\(.+\))?(!)?: .{1,72}"

COMMIT_MSG=$(cat "$1")
if ! echo "$COMMIT_MSG" | grep -qE "$PATTERN"; then
    echo ""
    echo "❌ 커밋 메시지 형식 오류!"
    echo "   올바른 형식: <type>(<scope>): <description>"
    echo "   예시: feat(auth): 카카오 로그인 추가"
    echo ""
    exit 1
fi
```

```bash
# commitlint 사용 (Node.js 프로젝트)
npm install --save-dev @commitlint/config-conventional @commitlint/cli

# commitlint.config.js
module.exports = { extends: ['@commitlint/config-conventional'] }

# Husky와 연동
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit $1'
```

---

## 6. 커밋 단위 결정

```text
✅ 하나의 커밋 = 하나의 논리적 변경
  - 로그인 기능 추가 (전체)
  - 특정 버그 수정
  - 설정 파일 업데이트

❌ 피해야 할 커밋 단위
  - "이것저것 수정" (여러 목적 혼합)
  - 30개 파일이 한 커밋에 (너무 큼)
  - 오타 수정, 공백 변경 (너무 작음 — fixup으로 합치기)
```

```bash
# 큰 변경을 여러 커밋으로 나누기
git add -p              # 파일 내에서도 부분 선택
git add src/auth/       # 폴더 단위

# WIP 커밋들을 나중에 정리 (push 전)
git rebase -i HEAD~5    # 마지막 5개 커밋 정리
# squash, fixup으로 합치기
```

---

## 7. 실무 팁

```bash
# 커밋 메시지 템플릿 설정
git config --global commit.template ~/.gitmessage

# ~/.gitmessage 내용
# <type>(<scope>): <description>
#
# WHY: 왜 이 변경이 필요한가?
# HOW: 어떻게 구현했는가? (선택)
#
# Refs: #ISSUE_NUMBER
```

```bash
# 이전 커밋들의 타입 분포 확인
git log --oneline | grep -oE "^[a-f0-9]+ (feat|fix|docs|chore|refactor|test)" | \
  awk '{print $2}' | sort | uniq -c | sort -rn

# CHANGELOG 자동 생성 도구
npx conventional-changelog-cli -p conventional -i CHANGELOG.md -s
```

---

## 참고 자료

- [Conventional Commits 공식 사이트](https://www.conventionalcommits.org/)
- [Angular 커밋 컨벤션](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit)
- 템플릿: `templates/commit-message.md`
- Hook 예시: `templates/git-hooks/commit-msg`
