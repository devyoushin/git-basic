# Git 명령어 및 예제 규칙 (Git Conventions)

---

## 1. 플레이스홀더 형식

| 플레이스홀더 | 의미 |
|-------------|------|
| `<BRANCH_NAME>` | 브랜치 이름 |
| `<COMMIT_HASH>` | 커밋 해시 (full 또는 short) |
| `<REMOTE>` | 원격 이름 (보통 origin) |
| `<FILE>` | 파일 경로 |
| `<TAG>` | 태그 이름 |
| `<MESSAGE>` | 커밋 메시지 |

## 2. 위험 명령어 대체 권장

| 위험한 패턴 | 권장 대체 |
|------------|----------|
| `git push --force` | `git push --force-with-lease` |
| `git reset --hard` | `git reset --soft` 또는 `git stash` 우선 검토 |
| `git clean -fd` | `git clean -n` 으로 먼저 확인 |
| `git rebase <shared-branch>` | 공유 브랜치에서는 merge 우선 검토 |

## 3. 자주 쓰는 명령어 조합 패턴

```bash
# 현재 상태 한눈에 파악 (항상 이것부터)
git status
git log --oneline --graph --decorate -10

# 안전한 작업 흐름
git fetch origin          # 원격 변경사항 가져오기
git diff origin/main      # 차이 확인
git merge origin/main     # 병합

# 실수 복구 시작점 (항상 이것부터)
git reflog | head -20
git status
```

## 4. 브랜치 명명 컨벤션 (문서 예제 내 사용)

```
main          # 메인 브랜치 (기본)
develop       # 개발 브랜치 (Git Flow)
feature/...   # 기능 브랜치
fix/...       # 버그 수정 브랜치
hotfix/...    # 긴급 수정 브랜치
release/...   # 릴리스 브랜치
```

## 5. 다이어그램 표기법

```text
A - B - C           # 선형 커밋 히스토리
        |
      (main)        # 브랜치 포인터
        |
      (HEAD)        # 현재 HEAD 위치

A - B - C           # 브랜치 분기
         \
          D - E     # feature 브랜치

(origin/main)       # 원격 추적 브랜치
```
