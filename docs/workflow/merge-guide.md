# Merge 가이드

> **카테고리**: `docs/workflow/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git merge`는 두 브랜치의 변경사항을 하나로 합치는 명령어.
상황에 따라 Fast-forward merge, 3-way merge, Squash merge 세 가지 방식으로 동작.
어떤 방식을 쓰느냐에 따라 히스토리 모양이 달라지므로 팀 협의가 중요.

---

## 2. 원리 / 개념 설명

### 2.1 Fast-forward Merge

main이 feature보다 뒤처졌고, feature가 main에서 직선으로 이어진 경우.
merge 커밋 없이 main 포인터만 앞으로 이동.

```text
Before:
  A - B - C  (main)
            \
             D - E  (HEAD → feature)

After: git switch main && git merge feature
  A - B - C - D - E  (HEAD → main, feature)
  # merge 커밋 없음. 깔끔한 선형 히스토리.
```

### 2.2 3-way Merge (Merge Commit)

main과 feature 양쪽에 서로 다른 커밋이 있을 때. 공통 조상을 기준으로 합침.

```text
Before:
  A - B - C - F  (HEAD → main)
            \
             D - E  (feature)

After: git switch main && git merge feature
  A - B - C - F - M  (HEAD → main)
            \       /
             D - E  (feature)
  # M은 자동 생성된 merge 커밋 (부모가 2개)
```

### 2.3 Squash Merge

feature의 모든 커밋을 하나로 합쳐서 main에 추가. feature 히스토리는 main에 남지 않음.

```text
Before:
  A - B - C  (main)
            \
             D - E - F  (feature)  ← 커밋 3개

After: git merge --squash feature && git commit
  A - B - C - S  (main)
  # S = D+E+F가 하나로 합쳐진 커밋 (새 커밋 직접 작성)
  # feature 브랜치 히스토리는 main에 반영 안 됨
```

### 2.4 세 방식 비교

| 방식 | 히스토리 | 언제 사용 |
|------|----------|----------|
| Fast-forward | 선형, 깔끔 | 단순 작업, 혼자 쓸 때 |
| 3-way (merge commit) | 분기 보존 | 팀 협업, 작업 단위 추적 원할 때 |
| Squash | main만 깔끔 | PR merge 시 feature 커밋 정리 원할 때 |

---

## 3. 실습 / 명령어 예제

### 3.1 기본 merge

```bash
# feature 브랜치를 main에 merge
git switch main
git merge feature

# merge 커밋 메시지 직접 작성
git merge feature -m "feat: 로그인 기능 merge"

# Fast-forward를 금지하고 항상 merge 커밋 생성
git merge --no-ff feature

# Fast-forward만 허용 (3-way 상황이면 실패)
git merge --ff-only feature
```

### 3.2 Squash merge

```bash
git switch main
git merge --squash feature
# 변경사항이 스테이징 영역에 올라오고 커밋은 아직 안 됨
git commit -m "feat: feature 브랜치 작업 통합"
```

### 3.3 충돌(Conflict) 해결

```bash
git merge feature
# CONFLICT 발생 시:

git status   # 충돌 파일 확인 (both modified)

# 파일 열어서 수동 해결
# <<<<<<< HEAD
# 현재 브랜치(main) 내용
# =======
# 합치려는 브랜치(feature) 내용
# >>>>>>> feature

# 해결 후
git add <RESOLVED_FILE>
git merge --continue   # 또는 git commit

# merge 취소하고 원래 상태로
git merge --abort
```

### 3.4 특정 파일만 다른 브랜치에서 가져오기

```bash
# merge 말고 특정 파일만 가져올 때
git checkout feature -- <FILE>
# 또는
git restore --source=feature -- <FILE>
```

---

## 4. 자주 하는 실수 & 주의사항

### merge 방향 실수

> ⚠️ feature에 main을 merge해야 할 때, main에 feature를 merge해버리는 실수.

```bash
# 항상 병합 대상 브랜치(받는 쪽)에 위치해야 함
git switch main          # main이 "받는 쪽"
git merge feature        # feature를 main으로 merge

# feature를 main에 맞춰 업데이트하려면 (방향 반대)
git switch feature
git merge main           # main을 feature로 merge
```

### merge 커밋이 너무 많아 히스토리가 지저분해짐

```bash
# git log --graph로 현재 히스토리 확인
git log --oneline --graph --all

# 앞으로 rebase 방식으로 정리된 히스토리 유지
# docs/workflow/rebase-guide.md 참조
```

---

## 5. 실무 팁

```bash
# merge 전 차이 미리 확인
git diff main..feature
git log main..feature --oneline   # feature에만 있는 커밋 목록

# merge 후 이미 merge된 브랜치 정리
git branch --merged main | grep -v main | xargs git branch -d

# GitHub/GitLab PR merge 방식 설정 권장
# - Squash and merge: 깔끔한 main 히스토리 원할 때
# - Create a merge commit: feature 작업 단위 보존 원할 때
# - Rebase and merge: 선형 히스토리 원할 때
```

**팀 협약**: merge 전략을 팀 전체가 통일해야 히스토리가 일관성 있게 유지됨. `--no-ff` 강제 설정 또는 PR merge 버튼 설정으로 통일 가능.

---

## 참고 자료

- [Git merge 공식 문서](https://git-scm.com/docs/git-merge)
- 관련 문서: `docs/workflow/rebase-guide.md`
- 관련 문서: `docs/troubleshooting/conflict-resolution.md`
