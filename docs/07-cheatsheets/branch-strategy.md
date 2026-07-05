# Git 브랜치 전략 치트시트

> Git Flow / GitHub Flow / Trunk-based 비교 및 브랜치 명령어 참조

---

## 전략 비교 한눈에 보기

| 항목 | Git Flow | GitHub Flow | Trunk-based |
|------|----------|-------------|-------------|
| 메인 브랜치 | main + develop | main | main (trunk) |
| 릴리즈 브랜치 | 있음 | 없음 | 없음 |
| 적합한 팀 | 릴리즈 주기 있는 팀 | 지속 배포 팀 | 고급 CI/CD 팀 |
| 복잡도 | 높음 | 낮음 | 낮음 |
| 배포 빈도 | 낮음 | 높음 | 매우 높음 |

---

## Git Flow

```
main ────────────────────────────────────── (production)
      │                              ↑
      └── develop ──────────────────/──── (통합 브랜치)
               │          │
               ├── feature/login     (기능 개발)
               ├── feature/payment
               └── release/1.2.0 ──→ main (릴리즈)
                        │
                   hotfix/1.2.1 ──→ main + develop
```

```bash
# 기능 시작
git switch develop
git switch -c feature/login

# 기능 완료
git switch develop
git merge --no-ff feature/login
git branch -d feature/login

# 릴리즈
git switch -c release/1.2.0 develop
# 버그 수정 후
git switch main && git merge --no-ff release/1.2.0
git tag v1.2.0
git switch develop && git merge --no-ff release/1.2.0

# 핫픽스
git switch -c hotfix/1.2.1 main
# 수정 후
git switch main && git merge --no-ff hotfix/1.2.1
git tag v1.2.1
git switch develop && git merge --no-ff hotfix/1.2.1
```

---

## GitHub Flow

```
main ──●──────────────────●── (항상 배포 가능)
       │                  ↑
       └── feature/login ─┘
           (PR → Review → Merge)
```

```bash
# 기능 시작
git switch -c feature/add-login main

# 작업 후 push + PR 생성
git push -u origin feature/add-login
# GitHub에서 PR 생성, 리뷰 요청

# 머지 후 브랜치 삭제
git switch main && git pull
git branch -d feature/add-login
git push origin --delete feature/add-login
```

---

## Trunk-based Development

```
main (trunk) ──●──●──●──●──●── (매일 여러 번 배포)
               │           Feature Flag로 미완성 기능 숨김
               └── short-lived branch (1~2일)
```

```bash
# 짧은 브랜치로 작업 (최대 1~2일)
git switch -c feat/add-button main

# 작은 커밋, 자주 push
git commit -m "feat: 버튼 컴포넌트 추가"
git push origin feat/add-button

# PR → 빠른 리뷰 → squash merge
# 브랜치 즉시 삭제
```

---

## 브랜치 명명 규칙

```bash
feature/<이슈번호>-<설명>    # feature/123-user-login
fix/<이슈번호>-<설명>         # fix/456-session-timeout
hotfix/<버전>-<설명>          # hotfix/1.2.1-null-check
release/<버전>                # release/1.2.0
chore/<설명>                   # chore/update-deps
```

---

## 브랜치 관리 명령어

```bash
# 목록 조회
git branch                         # 로컬
git branch -a                      # 로컬 + 원격
git branch --sort=-committerdate   # 최근 작업순 정렬

# 생성 / 전환
git switch -c <BRANCH>             # 생성 + 전환
git switch -c <BRANCH> origin/<BRANCH>  # 원격 기반 생성

# 삭제
git branch -d <BRANCH>             # 머지된 것만
git branch -D <BRANCH>             # 강제 삭제
git push origin --delete <BRANCH>  # 원격 삭제

# 원격 정리
git fetch --prune                  # 삭제된 원격 브랜치 로컬 참조 정리

# 브랜치 비교
git log main..feature --oneline    # feature에만 있는 커밋
git diff main...feature            # 분기 이후 변경사항
```

---

## 어떤 전략을 선택할까?

```
Q: 릴리즈 일정이 정해져 있나?
  → YES: Git Flow
  → NO: 다음 질문

Q: 배포가 자동화(CI/CD)되어 있나?
  → YES + 팀이 소규모: GitHub Flow
  → YES + 고급 테스트 자동화: Trunk-based
  → NO: GitHub Flow
```

---

관련 문서: `docs/03-collaboration/branch-strategy.md`
