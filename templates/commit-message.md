# 커밋 메시지 템플릿

> Git 커밋 메시지 작성 시 참조용 템플릿 모음.
> `git config --global commit.template ~/.gitmessage` 로 기본 템플릿 설정 가능.

---

## 기본 템플릿 (~/.gitmessage)

```text
# <type>(<scope>): <description>  ← 72자 이내, 현재형, 마침표 없음
#
# [body: 선택사항]
# 왜 이 변경을 했는지 설명 (HOW보다 WHY)
# 각 줄 72자 이내
#
# [footer: 선택사항]
# Breaking Changes, 이슈 참조
# BREAKING CHANGE: 변경 설명
# Fixes #123 / Closes #456 / Refs #789
#
# ─────────── 타입 목록 ───────────
# feat:     새 기능 추가
# fix:      버그 수정
# docs:     문서 변경
# style:    코드 스타일 (기능 무관)
# refactor: 리팩토링 (기능/버그 무관)
# test:     테스트 추가/수정
# chore:    빌드, 설정 변경
# ci:       CI/CD 설정
# build:    빌드 시스템 변경
# perf:     성능 개선
# revert:   커밋 되돌리기
```

---

## 상황별 템플릿

### feat: 새 기능

```text
feat(auth): 카카오 소셜 로그인 추가

카카오 OAuth 2.0 연동으로 소셜 로그인 지원.
기존 이메일 로그인과 병행 운영.

- KakaoLoginButton 컴포넌트 추가
- POST /api/auth/kakao 엔드포인트 구현
- 신규 유저 자동 가입 처리

Closes #123
```

### fix: 버그 수정

```text
fix(payment): 소수점 반올림 오류로 결제 금액 불일치 수정

원화 결제 시 Math.round 미적용으로 0.1원 단위 오차 발생.
DB 저장값과 실제 청구 금액 불일치로 정산 오류 야기.

Fixes #456
```

### refactor: 리팩토링

```text
refactor(auth): 인증 로직 미들웨어로 분리

각 라우터에 중복된 JWT 검증 로직을
auth.middleware.js로 통합.
기능 변경 없음, 코드 중복 제거 목적.
```

### chore: 빌드/설정 변경

```text
chore: ESLint 9.x 업그레이드 및 규칙 정리

flat config 형식으로 마이그레이션.
no-unused-vars, no-console 규칙 추가.
```

### docs: 문서 변경

```text
docs: API 인증 방식 설명 추가

Bearer 토큰 사용법과 갱신 흐름을
README에 추가. 신규 팀원 온보딩 가이드.
```

### breaking change

```text
feat(api)!: 인증 응답 형식 변경

JWT 기반으로 전환하면서 응답 구조 변경.

BREAKING CHANGE: /api/auth/login 응답 형식 변경
  이전: { "session_id": "..." }
  이후: { "access_token": "...", "refresh_token": "..." }

마이그레이션 가이드: docs/migration/v2.md

Closes #789
```

### revert: 되돌리기

```text
revert: feat(auth): 카카오 로그인 추가

카카오 SDK 보안 취약점 발견으로 임시 비활성화.
패치 후 재활성화 예정.

This reverts commit abc1234def5678.
Refs #890
```

---

## 커밋 메시지 검증 정규식

```bash
# 형식 검증 (commit-msg hook에 사용)
PATTERN="^(feat|fix|docs|style|refactor|test|chore|ci|build|perf|revert)(\(.+\))?(!)?: .{1,72}"

echo "feat: 로그인 추가" | grep -E "$PATTERN"   # ✅ 통과
echo "Update files"      | grep -E "$PATTERN"   # ❌ 실패
```

---

## 실용 단축 alias

```bash
# 커밋 타입별 alias
git config --global alias.feat '!f() { git commit -m "feat: $*"; }; f'
git config --global alias.fix  '!f() { git commit -m "fix: $*"; }; f'
git config --global alias.docs '!f() { git commit -m "docs: $*"; }; f'

# 사용
git feat "카카오 로그인 추가"
git fix "세션 만료 처리"
```
