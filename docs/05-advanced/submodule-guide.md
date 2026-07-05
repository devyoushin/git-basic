# Git Submodule 가이드

> **카테고리**: `docs/05-advanced/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

Submodule은 **다른 Git 레포지터리를 현재 레포 안에 포함**시키는 방법.
공유 라이브러리, 공통 설정, 외부 의존성을 별도 레포로 관리하면서
메인 레포에서 특정 커밋을 고정해서 사용할 때 유용.

---

## 2. 원리 / 개념 설명

### 2.1 Submodule이란

```text
main-repo/
├── .gitmodules          ← submodule 경로/URL 정보
├── src/
└── libs/
    └── shared-utils/    ← 다른 레포의 특정 커밋을 가리키는 포인터
        (실제 코드가 아닌 커밋 해시 참조)
```

### 2.2 .gitmodules 파일

```ini
[submodule "libs/shared-utils"]
    path = libs/shared-utils
    url = https://github.com/company/shared-utils.git
    branch = main
```

### 2.3 Submodule vs 다른 방법

| 방법 | 장점 | 단점 |
|------|------|------|
| Submodule | 커밋 고정, 독립적 버전 관리 | 복잡한 워크플로우 |
| npm/pip 패키지 | 간단한 의존성 관리 | 레포 수정 불편 |
| Monorepo | 한 곳에서 관리 | 레포 크기 증가 |
| git subtree | 히스토리 통합, 간단한 사용 | 업스트림 반영 어려움 |

---

## 3. 실습 / 명령어 예제

### 3.1 Submodule 추가

```bash
# 기본 추가
git submodule add https://github.com/company/shared-utils.git libs/shared-utils

# 특정 브랜치 추적
git submodule add -b main https://github.com/company/shared-utils.git libs/shared-utils

# 추가 후 커밋
git add .gitmodules libs/shared-utils
git commit -m "chore: shared-utils submodule 추가"
```

### 3.2 Submodule 포함 Clone

```bash
# 방법 1: clone 시 submodule 동시 초기화
git clone --recurse-submodules https://github.com/company/main-repo.git

# 방법 2: clone 후 초기화
git clone https://github.com/company/main-repo.git
cd main-repo
git submodule init      # .gitmodules 등록
git submodule update    # 실제 코드 가져오기

# 한 번에
git submodule update --init --recursive
```

### 3.3 Submodule 업데이트

```bash
# 특정 submodule을 최신 커밋으로 업데이트
cd libs/shared-utils
git pull origin main
cd ../..
git add libs/shared-utils
git commit -m "chore: shared-utils v2.1.0으로 업데이트"

# 모든 submodule 일괄 업데이트 (추적 브랜치 기준)
git submodule update --remote

# 특정 submodule만
git submodule update --remote libs/shared-utils
```

### 3.4 Submodule 상태 확인

```bash
# 상태 확인
git submodule status
# -abc1234 libs/shared-utils (v1.0.0)
# - prefix: 초기화 안 됨
# + prefix: 변경됨 (다른 커밋 체크아웃됨)
# 공백: 정상

# 요약
git submodule summary

# 포함 디렉토리
git submodule foreach 'git log --oneline -3'  # 각 submodule에서 명령 실행
```

### 3.5 Submodule에서 작업 후 반영

```bash
# submodule 내부에서 작업
cd libs/shared-utils
git switch -c feature/new-util
# 수정 작업
git commit -m "feat: 새 유틸 함수 추가"
git push origin feature/new-util

# 메인 레포에서 업데이트 반영
cd ../..
git add libs/shared-utils    # 새 커밋 해시 반영
git commit -m "chore: shared-utils 최신 커밋으로 업데이트"
```

### 3.6 Submodule 제거

```bash
# 1. .gitmodules에서 항목 제거
git submodule deinit libs/shared-utils

# 2. 추적에서 제거
git rm libs/shared-utils

# 3. 캐시 정리
rm -rf .git/modules/libs/shared-utils

# 4. 커밋
git commit -m "chore: shared-utils submodule 제거"
```

---

## 4. 자주 하는 실수 & 주의사항

### Submodule이 빈 폴더로 보이는 경우

```bash
# 원인: submodule 초기화 안 됨
git submodule update --init --recursive
```

### Detached HEAD 상태

```bash
# submodule은 기본적으로 detached HEAD 상태
# 특정 브랜치를 추적하려면
cd libs/shared-utils
git switch main             # 브랜치로 이동
git pull origin main

# 또는 .gitmodules에 branch 설정
git config -f .gitmodules submodule.libs/shared-utils.branch main
git submodule update --remote
```

### 팀원이 submodule 업데이트를 pull하지 않은 경우

```bash
# submodule 커밋이 맞지 않으면 빌드 실패
# 팀원이 실행해야 할 명령어
git pull
git submodule update --init --recursive   # pull 후 항상 실행
```

---

## 5. 실무 팁

```bash
# pull 후 자동으로 submodule 업데이트
git config --global submodule.recurse true
# 이후 git pull 하면 submodule도 자동 업데이트

# 모든 submodule에서 동시에 명령 실행
git submodule foreach 'git status'
git submodule foreach 'git pull origin main'
git submodule foreach --recursive 'git clean -fd'

# submodule URL 변경 (레포 이전 시)
git config --file .gitmodules submodule.libs/shared-utils.url \
  https://new-url/shared-utils.git
git submodule sync
git submodule update --remote
```

```bash
# .gitconfig에 유용한 alias
git config --global alias.spull '!git pull && git submodule update --init --recursive'
# git spull 로 pull + submodule 동시에
```

---

## 참고 자료

- [git submodule 공식 문서](https://git-scm.com/book/ko/v2/Git-도구-서브모듈)
- 관련 문서: `docs/05-advanced/worktree-guide.md`
