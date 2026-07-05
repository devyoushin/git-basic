# Git 긴급 복구 치트시트

> 뭔가 잘못됐을 때 즉시 참조. **항상 `git reflog`부터 시작.**

---

## 🚨 즉시 실행 — 상황 파악

```bash
git status          # 현재 상태 확인
git reflog          # HEAD 이동 기록 전체 (복구 시작점)
git log --oneline --graph --all  # 전체 브랜치 상태
```

---

## 1. reset --hard로 커밋 날린 경우

```bash
git reflog
# HEAD@{1}: commit: feat: 중요한 기능  ← 이 해시 확인

git reset --hard HEAD@{1}   # 또는 커밋 해시로
# ✅ 커밋은 90일간 reflog에 보관
```

---

## 2. 브랜치 삭제한 경우

```bash
git reflog | grep <BRANCH_NAME>
# HEAD@{3}: checkout: moving from <BRANCH_NAME> to main

git branch <BRANCH_NAME> HEAD@{3}   # 브랜치 복구
# ✅ 마지막 커밋 해시로 브랜치 재생성
```

---

## 3. rebase/merge 직전으로 돌아가기

```bash
# 방법 1: ORIG_HEAD 활용 (가장 빠름)
git reset --hard ORIG_HEAD

# 방법 2: reflog에서 찾기
git reflog
git reset --hard HEAD@{N}   # rebase/merge 시작 전 N 확인
```

---

## 4. 잘못된 커밋 취소 (push 전)

```bash
# 마지막 커밋만 취소 (변경사항 유지)
git reset --soft HEAD~1    # → Staging Area
git reset HEAD~1           # → Working Tree

# 마지막 커밋 + 변경사항 모두 삭제
git reset --hard HEAD~1    # ⚠️ 변경사항 사라짐
```

---

## 5. 잘못된 커밋 취소 (push 후, 공유 브랜치)

```bash
# revert로 새 커밋 생성 (force push 불필요)
git revert <COMMIT_HASH>
git push origin main

# 여러 커밋 revert
git revert HEAD~3..HEAD
```

---

## 6. 파일 하나만 과거 버전으로 복구

```bash
# 마지막 커밋 상태로
git restore <FILE>

# N번 전 커밋 상태로
git restore --source=HEAD~2 -- <FILE>

# 특정 커밋 시점으로
git restore --source=<COMMIT_HASH> -- <FILE>
```

---

## 7. Detached HEAD에서 커밋 잃어버린 경우

```bash
git reflog
# HEAD@{1}: commit: 중요한 작업  ← 해시 확인

git branch recovered HEAD@{1}   # 브랜치로 구출
git switch recovered
```

---

## 8. stash 날린 경우

```bash
# stash도 reflog에 남음
git fsck --lost-found
# dangling commit <HASH> 목록에서 찾기

git show <DANGLING_HASH>   # 내용 확인
git stash apply <DANGLING_HASH>   # 적용
```

---

## 9. 원격 브랜치 복구 (force push로 덮어쓴 경우)

```bash
# 팀원 로컬 reflog에서 복구
git reflog show origin/<BRANCH>
git push origin <COMMIT_HASH>:refs/heads/<BRANCH>
# 또는 팀원이 자신의 로컬에서 직접 push
```

---

## 10. 충돌 상태 탈출

```bash
# merge 충돌 → 취소
git merge --abort

# rebase 충돌 → 취소
git rebase --abort

# cherry-pick 충돌 → 취소
git cherry-pick --abort
```

---

## ⚠️ 복구 불가 케이스

| 상황 | 이유 |
|------|------|
| commit 안 한 파일 삭제 (`git clean -fd`) | Git 객체 없음 |
| `git stash drop` + `git gc` | GC가 고아 객체 정리 |
| reflog 만료 후 (기본 90일) | 참조 소실 |
| 스테이징 안 한 변경사항 + `reset --hard` | Working Tree는 Git 미관리 |

---

## 복구 전 안전망 설치

```bash
# 현재 상태 백업 브랜치 생성
git branch backup/$(date +%Y%m%d-%H%M)

# reflog 보존 기간 늘리기
git config --global gc.reflogExpire "180 days"
```

---

**황금 규칙**: 패닉하지 말고 `git reflog`부터.
관련 문서: `docs/05-advanced/reflog-guide.md`, `docs/06-troubleshooting/undo-mistakes.md`
