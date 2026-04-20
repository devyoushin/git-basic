# Cherry-pick 가이드

> **카테고리**: `docs/workflow/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git cherry-pick`은 다른 브랜치의 특정 커밋만 골라서 현재 브랜치에 적용하는 명령어.
브랜치 전체를 merge하지 않고 특정 버그 수정 커밋만 가져올 때 자주 사용.
rebase와 마찬가지로 **새 커밋 해시가 생성됨** (내용은 같지만 부모가 다름).

---

## 2. 원리 / 개념 설명

### 2.1 cherry-pick 동작

```text
Before:
  A - B - C  (HEAD → main)

  A - B - D - E - F  (feature)
  # E 커밋만 main에 가져오고 싶음

git cherry-pick E:
  A - B - C - E'  (HEAD → main)
  # E'는 E와 내용은 같지만 부모가 C이고 해시가 다름

  A - B - D - E - F  (feature)
  # feature는 그대로
```

---

## 3. 실습 / 명령어 예제

### 3.1 기본 cherry-pick

```bash
# 특정 커밋 하나 가져오기
git cherry-pick <COMMIT_HASH>

# 커밋 해시 확인 방법
git log --oneline feature   # feature 브랜치의 커밋 목록

# 여러 커밋 한 번에
git cherry-pick <HASH_1> <HASH_2> <HASH_3>

# 범위로 가져오기 (A 제외, B까지)
git cherry-pick <HASH_A>..<HASH_B>

# 범위로 가져오기 (A 포함)
git cherry-pick <HASH_A>^..<HASH_B>
```

### 3.2 cherry-pick 옵션

```bash
# 커밋은 만들지 않고 스테이징까지만 (직접 커밋 메시지 작성)
git cherry-pick -n <COMMIT_HASH>
git cherry-pick --no-commit <COMMIT_HASH>

# 원본 커밋 정보 메시지에 포함
git cherry-pick -x <COMMIT_HASH>
# 커밋 메시지에 "(cherry picked from commit abc1234)" 자동 추가

# 작성자 정보 유지 (기본은 cherry-pick 실행자가 커밋 작성자)
git cherry-pick --signoff <COMMIT_HASH>
```

### 3.3 충돌 처리

```bash
git cherry-pick <COMMIT_HASH>
# CONFLICT 발생 시

git status   # 충돌 파일 확인
# 파일 열어서 수동 해결 후

git add <RESOLVED_FILE>
git cherry-pick --continue

# 중단하고 원래 상태로
git cherry-pick --abort

# 이 커밋은 건너뛰고 다음 커밋 계속
git cherry-pick --skip
```

### 3.4 실전 시나리오: hotfix를 release 브랜치에도 적용

```bash
# main에서 버그 수정
git switch main
git commit -m "fix: 결제 버그 수정"
# 이 커밋 해시 확인
git log --oneline -1   # 예: abc1234

# release 브랜치에도 동일한 수정 적용
git switch release/v1.2
git cherry-pick abc1234 -x   # -x: 원본 커밋 참조 포함
```

---

## 4. 자주 하는 실수 & 주의사항

### cherry-pick 남발 시 중복 커밋 문제

**증상**: 나중에 merge 시 "already applied" 충돌 또는 동일 변경이 두 번 적용됨.

```text
main:    A - B - C - C'  (cherry-pick으로 C를 C'로 가져옴)
feature: A - B - C - D

git merge feature 시 C와 C'가 충돌할 수 있음
```

**해결**: cherry-pick보다 rebase나 merge가 적합한지 먼저 검토.

### 연속 커밋의 순서 의존성

**상황**: E가 D에 의존하는데 E만 cherry-pick 하면 충돌 발생.
```bash
# D와 E를 함께 cherry-pick
git cherry-pick <D_HASH> <E_HASH>
```

---

## 5. 실무 팁

```bash
# cherry-pick 전에 내용 미리 확인
git show <COMMIT_HASH>
git show <COMMIT_HASH> --stat   # 변경된 파일 목록만

# 여러 브랜치에 동시 적용이 필요하다면 스크립트화
for branch in release/v1.2 release/v1.3; do
  git switch $branch
  git cherry-pick <COMMIT_HASH> -x
done
```

**언제 cherry-pick을 쓰는가**:
- 버그 수정을 여러 릴리즈 브랜치에 동시 적용
- feature 브랜치에서 특정 커밋만 먼저 main에 반영
- 실수로 잘못된 브랜치에 커밋한 것을 올바른 브랜치로 이동

---

## 참고 자료

- [git cherry-pick 공식 문서](https://git-scm.com/docs/git-cherry-pick)
- 관련 문서: `docs/workflow/rebase-guide.md`
- 관련 문서: `docs/troubleshooting/undo-mistakes.md`
