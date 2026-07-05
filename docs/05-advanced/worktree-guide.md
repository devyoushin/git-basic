# Git Worktree 가이드

> **카테고리**: `docs/05-advanced/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

`git worktree`는 **하나의 Git 레포지터리에서 여러 브랜치를 동시에 다른 디렉토리에서 작업** 가능하게 해주는 기능.
`git stash`나 임시 커밋 없이 작업 컨텍스트를 즉시 전환할 수 있음.
대형 프로젝트에서 빌드 시간이 길거나, 핫픽스 대응이 잦은 경우 특히 유용.

---

## 2. 원리 / 개념 설명

### 2.1 기존 방식 vs Worktree

```text
기존 방식 (stash 사용):
  feature 브랜치 작업 중 → stash → switch to main → hotfix 작업
  → switch back → stash pop → feature 계속
  ⚠️ 빌드 캐시 날아감, 컨텍스트 전환 비용

Worktree 방식:
  /project/main-repo       ← feature 브랜치 작업 중 (그대로 유지)
  /project/hotfix-worktree ← hotfix 브랜치를 별도 폴더에서 동시에 작업
  ✓ 각 디렉토리가 독립적인 체크아웃, 같은 .git 공유
```

### 2.2 구조

```text
.git/
  worktrees/
    hotfix-1.2.1/     ← 추가 worktree 메타데이터
    experiment/

/project/main-repo/   ← 메인 worktree (primary)
/project/hotfix/      ← 추가 worktree (linked)
/project/experiment/  ← 추가 worktree (linked)

모두 같은 .git 오브젝트 DB를 공유
같은 브랜치를 두 worktree에서 동시에 체크아웃 불가
```

---

## 3. 실습 / 명령어 예제

### 3.1 Worktree 추가

```bash
# 기존 브랜치를 새 경로에 체크아웃
git worktree add ../hotfix-branch hotfix/1.2.1

# 새 브랜치를 생성하면서 추가 (-b 옵션)
git worktree add -b hotfix/1.2.1 ../hotfix-work main

# 임시 작업 (detached HEAD)
git worktree add --detach ../review-pr abc1234

# 현재 디렉토리 구조
ls ../
# main-repo/
# hotfix-work/
```

### 3.2 Worktree 목록 확인

```bash
git worktree list
# /Users/sunny/project/main-repo  abc1234 [feature/new-ui]
# /Users/sunny/project/hotfix      def5678 [hotfix/1.2.1]

# 상세 정보
git worktree list --porcelain
```

### 3.3 Worktree에서 작업

```bash
# 메인 디렉토리는 그대로 작업 유지
# 새 터미널에서 hotfix worktree로 이동
cd ../hotfix-work

# 일반 git 명령 동일하게 사용
git status
git add .
git commit -m "hotfix: null 포인터 수정"
git push origin hotfix/1.2.1

# 작업 완료 후 메인으로 돌아가면 컨텍스트 그대로
cd ../main-repo
```

### 3.4 Worktree 제거

```bash
# 사용 완료 후 제거
git worktree remove ../hotfix-work

# 강제 제거 (변경사항 있어도)
git worktree remove --force ../hotfix-work

# 폴더 수동 삭제 후 레퍼런스 정리
rm -rf ../hotfix-work
git worktree prune               # 존재하지 않는 worktree 참조 정리
```

---

## 4. 실전 활용 시나리오

### 4.1 핫픽스 대응 (가장 일반적)

```bash
# 현재 feature 브랜치 작업 중
# 프로덕션 버그 발견!

# 1. stash 없이 hotfix worktree 생성
git worktree add -b hotfix/v1.2.1 /tmp/hotfix-v1.2.1 main

# 2. 별도 터미널에서 핫픽스 작업
cd /tmp/hotfix-v1.2.1
vim src/auth.py    # 버그 수정
git commit -m "hotfix: 인증 토큰 만료 처리 누락 수정"
git push origin hotfix/v1.2.1

# 3. 메인 작업은 계속
cd ~/project/main-repo
git status   # feature 작업 그대로

# 4. 완료 후 정리
git worktree remove /tmp/hotfix-v1.2.1
git branch -d hotfix/v1.2.1
```

### 4.2 PR 코드 리뷰

```bash
# 리뷰할 PR 브랜치를 worktree로
git worktree add ../review-pr123 origin/feature/new-auth

cd ../review-pr123
# 실제 실행해보면서 리뷰
npm install && npm run dev

# 리뷰 완료
cd ../main-repo
git worktree remove ../review-pr123
```

### 4.3 A/B 비교 빌드

```bash
# 두 버전을 동시에 빌드해서 성능 비교
git worktree add ../version-a v1.0.0
git worktree add ../version-b v2.0.0-beta

# 각각 빌드 후 벤치마크 실행
cd ../version-a && npm run build && npm run benchmark
cd ../version-b && npm run build && npm run benchmark
```

---

## 5. 자주 하는 실수 & 주의사항

### 같은 브랜치 두 곳에서 체크아웃 불가

```bash
# ❌ 오류 발생
git worktree add ../another-dir main   # main이 이미 체크아웃된 경우
# fatal: 'main' is already checked out

# ✅ 해결: 다른 브랜치 사용하거나 detached HEAD
git worktree add --detach ../another-dir main
```

### Worktree 경로 삭제 후 prune 필요

```bash
# 폴더를 rm으로 삭제한 경우
git worktree prune
# 없어진 worktree 참조 정리 → git worktree list 깔끔해짐
```

---

## 6. 실무 팁

```bash
# 모든 worktree에서 같은 명령 실행
git worktree list --porcelain | grep "worktree " | awk '{print $2}' | \
  xargs -I {} sh -c 'cd {} && git status'

# worktree 목록을 alias로 보기 좋게
git config --global alias.wt "worktree list"

# 임시 worktree를 /tmp에 생성 (정리가 편함)
git worktree add /tmp/git-review-$(date +%Y%m%d) <BRANCH>
```

---

## 참고 자료

- [git worktree 공식 문서](https://git-scm.com/docs/git-worktree)
- 관련 문서: `docs/05-advanced/submodule-guide.md`
