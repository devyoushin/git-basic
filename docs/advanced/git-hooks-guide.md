# Git Hooks 가이드

> **카테고리**: `docs/advanced/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

Git Hooks는 특정 Git 이벤트 발생 시 자동으로 실행되는 스크립트.
커밋 전 린트, 커밋 메시지 형식 검증, push 전 테스트 실행 등 자동화에 사용.
`.git/hooks/` 디렉토리에 위치하며 실행 권한(`chmod +x`)이 있어야 동작.

---

## 2. 원리 / 개념 설명

### 2.1 주요 Hook 종류

```text
클라이언트 사이드 (로컬에서 실행):
  pre-commit      ← git commit 실행 직전 (exit 1이면 커밋 중단)
  prepare-commit-msg  ← 에디터 열기 전 메시지 초안 준비
  commit-msg      ← 커밋 메시지 작성 후 (형식 검증)
  post-commit     ← 커밋 완료 후 (알림 등)
  pre-push        ← push 직전 (테스트 실행)
  pre-rebase      ← rebase 전

서버 사이드 (원격 레포에서 실행):
  pre-receive     ← push 수신 전
  post-receive    ← push 수신 후
  update          ← 각 브랜치 업데이트 전
```

### 2.2 Hook 동작 방식

```text
git commit 실행
    │
    ▼
.git/hooks/pre-commit 스크립트 실행
    │
    ├── exit 0 → 커밋 진행
    └── exit 1 → 커밋 중단 (오류 메시지 출력)
```

---

## 3. 실습 / 명령어 예제

### 3.1 Hook 파일 생성 기본

```bash
# Hook 디렉토리 확인
ls .git/hooks/
# pre-commit.sample, commit-msg.sample 등 샘플 파일 있음

# pre-commit Hook 생성
touch .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

### 3.2 pre-commit: 커밋 전 린트 검사

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "🔍 린트 검사 중..."

# Python: flake8
if command -v flake8 >/dev/null 2>&1; then
    flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
    if [ $? -ne 0 ]; then
        echo "❌ 린트 오류 발생. 커밋 중단."
        exit 1
    fi
fi

# JavaScript/TypeScript: ESLint
if [ -f package.json ]; then
    npx eslint --ext .js,.ts src/
    if [ $? -ne 0 ]; then
        echo "❌ ESLint 오류 발생. 커밋 중단."
        exit 1
    fi
fi

echo "✅ 린트 통과!"
exit 0
```

### 3.3 commit-msg: 커밋 메시지 형식 검증

```bash
#!/bin/sh
# .git/hooks/commit-msg
# Conventional Commits 형식 강제

COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# 허용 타입
PATTERN="^(feat|fix|docs|style|refactor|test|chore|ci|build|perf|revert)(\(.+\))?: .{1,72}"

if ! echo "$COMMIT_MSG" | grep -qE "$PATTERN"; then
    echo ""
    echo "❌ 커밋 메시지 형식 오류!"
    echo ""
    echo "올바른 형식: <type>(<scope>): <description>"
    echo "예시:"
    echo "  feat: 로그인 기능 추가"
    echo "  fix(auth): 토큰 갱신 버그 수정"
    echo "  docs: README 업데이트"
    echo ""
    echo "허용 타입: feat, fix, docs, style, refactor, test, chore, ci, build, perf, revert"
    echo ""
    exit 1
fi

exit 0
```

### 3.4 pre-push: push 전 테스트 실행

```bash
#!/bin/sh
# .git/hooks/pre-push

echo "🧪 테스트 실행 중..."

# Python pytest
if [ -f pytest.ini ] || [ -f pyproject.toml ]; then
    python -m pytest --tb=short -q
    if [ $? -ne 0 ]; then
        echo "❌ 테스트 실패! push 중단."
        exit 1
    fi
fi

# Node.js
if [ -f package.json ]; then
    npm test
    if [ $? -ne 0 ]; then
        echo "❌ 테스트 실패! push 중단."
        exit 1
    fi
fi

echo "✅ 테스트 통과! push 진행."
exit 0
```

### 3.5 팀 전체에 Hook 공유 (핵심!)

`.git/hooks/`는 git에 포함되지 않아 팀원과 공유 불가. 해결 방법:

```bash
# 방법 1: 레포에 hooks/ 디렉토리 만들고 심링크
mkdir -p hooks
cp .git/hooks/pre-commit hooks/pre-commit
git add hooks/
git commit -m "chore: pre-commit hook 추가"

# 팀원이 설정
ln -sf ../../hooks/pre-commit .git/hooks/pre-commit
# 또는 setup 스크립트로 자동화
```

```bash
# setup.sh
#!/bin/sh
echo "Git Hooks 설정 중..."
for hook in hooks/*; do
    hook_name=$(basename $hook)
    ln -sf "../../$hook" ".git/hooks/$hook_name"
    chmod +x ".git/hooks/$hook_name"
    echo "  ✓ $hook_name"
done
echo "완료!"
```

```bash
# 방법 2: husky (Node.js 프로젝트)
npm install --save-dev husky
npx husky init

# .husky/pre-commit 파일 생성
echo "npm test" > .husky/pre-commit
```

```bash
# 방법 3: core.hooksPath 설정
git config core.hooksPath hooks   # hooks/ 디렉토리를 hook 경로로
```

---

## 4. 자주 하는 실수 & 주의사항

### Hook이 실행 안 됨

**원인**: 실행 권한 없음
```bash
chmod +x .git/hooks/pre-commit
ls -la .git/hooks/pre-commit   # -rwxr-xr-x 확인
```

### Hook 우회 (긴급 시)

```bash
# 커밋 메시지 형식 hook 우회
git commit --no-verify -m "WIP: 긴급 수정"
# ⚠️ 남용 금지. 정말 긴급한 경우에만 사용.
```

### 팀원 Hook 적용 강제 불가

Hook은 로컬 설정이므로 강제 적용 불가. CI/CD에서도 같은 검사를 실행하여 서버 사이드에서 보완.

---

## 5. 실무 팁

```bash
# 현재 활성화된 Hook 목록
ls -la .git/hooks/ | grep -v sample

# Hook 실행 로그 디버깅
GIT_TRACE=1 git commit -m "test"

# Hook에서 스테이징된 파일만 검사 (속도 개선)
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep ".py$")
if [ -n "$STAGED_FILES" ]; then
    flake8 $STAGED_FILES
fi
```

---

## 참고 자료

- [Git Hooks 공식 문서](https://git-scm.com/book/ko/v2/Git맞춤-Git-Hooks)
- [Husky (Node.js)](https://typicode.github.io/husky/)
- Hook 스크립트 모음: `templates/git-hooks/`
- 관련 문서: `docs/collaboration/commit-convention.md`
