# Git 브랜치 전략 가이드

> **카테고리**: `docs/collaboration/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

브랜치 전략은 팀의 배포 주기, 팀 규모, 자동화 수준에 따라 달라짐.
잘못된 전략 선택은 협업 마찰의 주요 원인.
현재 업계 트렌드: Git Flow → GitHub Flow → Trunk-based로 이동 중.

---

## 2. 전략별 비교

### 2.1 Git Flow

```text
main ─────────────────────────────────────────── (v1.0) ── (v1.1)
      │                                              ↑         ↑
develop ──────────────────────────────────────────/────────/──
          │          │             │
          ├── feature/A ──→ develop
          ├── feature/B ──→ develop
          └── release/1.0 ──→ main + develop
                                    │
                              hotfix/1.0.1 ──→ main + develop
```

**적합한 상황:**
- 릴리즈 일정이 정기적 (격주, 월 1회 등)
- QA 단계가 명확히 분리된 팀
- 앱스토어 배포처럼 롤백이 어려운 환경

**단점:**
- 브랜치가 많아 복잡
- develop ↔ main 동기화 오류 발생 가능
- 소규모 팀에 과도한 오버헤드

```bash
# Git Flow 주요 명령어
git switch -c feature/login develop      # 기능 브랜치 시작
git merge --no-ff feature/login          # feature → develop
git switch -c release/1.0 develop        # 릴리즈 브랜치
git merge --no-ff release/1.0 main       # 릴리즈 → main
git tag v1.0.0                           # 버전 태그
git merge --no-ff release/1.0 develop   # 릴리즈 → develop 역동기화
```

---

### 2.2 GitHub Flow

```text
main ──●────────────────────●────────────────────●──
       │                    ↑                    ↑
       ├── feature/login ───┘                    │
       └── feature/payment ──────────────────────┘
           (PR → Review → Merge → Deploy)
```

**적합한 상황:**
- 지속적 배포(CD) 환경
- main = 항상 배포 가능한 상태
- 소규모~중규모 팀

**특징:**
- main 브랜치 하나만 보호
- 모든 작업은 feature 브랜치에서
- PR이 코드 리뷰 + QA 역할

```bash
# GitHub Flow 기본 사이클
git switch -c feature/add-login main      # 시작
git push -u origin feature/add-login      # push
# PR 생성 → 리뷰 → Approve → Merge
git switch main && git pull               # 완료 후 동기화
git branch -d feature/add-login           # 로컬 정리
```

---

### 2.3 Trunk-based Development

```text
main ──●──●──●──●──●──●──●──●──  (여러 팀이 하루에도 여러 번 머지)
       ↑  ↑
       │  └── short-lived branch (1~2일)
       └── feature flag로 미완성 기능 숨김
```

**적합한 상황:**
- 성숙한 CI/CD 파이프라인 (테스트 자동화 필수)
- 대규모 팀 (Google, Facebook 방식)
- 배포 주기: 하루 수십 회

**핵심 기법:**
- Feature Flag: 코드는 병합하되 기능은 숨김
- Branch by Abstraction: 큰 리팩토링을 단계적으로
- 브랜치 수명: 최대 1~2일

```bash
# Feature Flag 예시
if feature_flags.is_enabled('new_checkout', user):
    return new_checkout_flow()
else:
    return old_checkout_flow()

# 단기 브랜치 사이클
git switch -c feat/button-text main   # 시작
# 작업 완료 → PR → 빠른 리뷰 → squash merge
# 브랜치 즉시 삭제
```

---

## 3. 전략 선택 가이드

```text
현재 팀 상황 평가:

1. 배포 주기가 얼마나 자주인가?
   - 주 1회 이하:  Git Flow 고려
   - 주 1회~매일:  GitHub Flow
   - 하루 여러 번: Trunk-based

2. CI/CD 자동화 수준은?
   - 없음/초기:    Git Flow (수동 QA 보완)
   - 부분 자동화:  GitHub Flow
   - 완전 자동화:  Trunk-based

3. 팀 규모는?
   - 1~5명:        GitHub Flow (Git Flow는 과함)
   - 5~20명:       GitHub Flow 또는 Trunk-based
   - 20명 이상:    Trunk-based 권장

4. 앱스토어/엄격한 릴리즈 프로세스?
   - YES: Git Flow
   - NO:  GitHub Flow 또는 Trunk-based
```

---

## 4. 브랜치 보호 규칙 설정

```text
main 브랜치 보호 (GitHub 설정):
  ✓ Require pull request reviews (최소 1명)
  ✓ Dismiss stale pull request approvals
  ✓ Require status checks to pass (CI)
  ✓ Require branches to be up to date
  ✓ Restrict who can push to matching branches
  ✓ Do not allow force pushes
```

---

## 5. 브랜치 네이밍 규칙

```bash
# 타입 접두사 + 이슈번호 + 설명
feature/<ISSUE>-<description>     # feature/123-user-login
fix/<ISSUE>-<description>          # fix/456-session-timeout
hotfix/<version>-<description>     # hotfix/1.2.1-null-pointer
release/<version>                  # release/1.2.0
chore/<description>                # chore/update-dependencies
experiment/<description>           # experiment/new-auth-flow

# 규칙
- 소문자 + 하이픈
- 이슈 번호 포함 (추적 가능)
- 설명은 구체적으로 (fix/bug ❌, fix/456-login-null-check ✓)
- 길어도 설명이 명확한 게 나음
```

---

## 6. Merge 전략 선택

```bash
# PR 머지 시 전략 선택
git merge --no-ff feature     # Merge commit (히스토리 보존, 분기 명확)
git merge --squash feature    # Squash (feature 커밋 하나로 압축)
git rebase feature            # Rebase (선형 히스토리)

# GitHub/GitLab 설정 권장
- feature → develop: Squash merge (WIP 커밋 정리)
- develop → main:    Merge commit (릴리즈 기록)
- hotfix → main:     Merge commit (긴급 수정 명확화)
```

---

## 7. 실무 팁

```bash
# 오래된 브랜치 정리
git branch --merged main | grep -v "main\|develop" | xargs git branch -d

# 원격에서 삭제된 브랜치 로컬 참조 정리
git fetch --prune

# 현재 브랜치 수명 확인
git for-each-ref --format='%(refname:short) %(creatordate:relative)' refs/heads \
  | sort -k2

# PR 없이 머지된 커밋 찾기 (Trunk-based 팀 감사용)
git log main --merges --first-parent --format='%H %s' | head -20
```

---

## 참고 자료

- [Atlassian Git Flow 가이드](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [GitHub Flow 공식 가이드](https://docs.github.com/en/get-started/quickstart/github-flow)
- [Trunk-based Development](https://trunkbaseddevelopment.com/)
- 빠른 참조: `cheatsheets/branch-strategy.md`
- 관련 문서: `docs/collaboration/pr-workflow.md`
