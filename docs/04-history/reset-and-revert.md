# Reset & Revert 가이드

> **카테고리**: `docs/04-history/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git reset`은 현재 브랜치 포인터를 과거로 이동(히스토리 변경).
`git revert`는 특정 커밋을 취소하는 새 커밋을 추가(히스토리 보존).
**push된 커밋에는 reset 대신 revert를 사용** — reset은 팀원 히스토리와 충돌 발생.

---

## 2. 원리 / 개념 설명

### 2.1 git reset 세 가지 모드

```text
커밋 상태: A - B - C - D  (HEAD → main)
목표: D를 취소하고 C로 돌아가기
명령어: git reset HEAD~1 (또는 git reset <C_HASH>)

--soft:
  HEAD → C 로 이동
  D의 변경사항 → Staging Area에 남아있음
  Working Tree: 그대로

--mixed (기본값):
  HEAD → C 로 이동
  D의 변경사항 → Working Tree에 남아있음 (스테이징 취소)
  Working Tree: 그대로

--hard:
  HEAD → C 로 이동
  D의 변경사항 → 완전히 삭제
  Working Tree: C 상태로 리셋 (⚠️ 위험)
```

### 2.2 git revert 동작

```text
Before:
  A - B - C - D  (HEAD → main)

git revert D:
  A - B - C - D - D'  (HEAD → main)
  # D'는 D의 변경사항을 정반대로 되돌리는 새 커밋
  # D는 히스토리에 그대로 남음
  # 팀원 히스토리와 충돌 없음 ✓
```

### 2.3 reset vs revert 선택 기준

| 상황 | 권장 방법 |
|------|----------|
| push 전 (로컬만) | reset 자유롭게 사용 |
| push 후 (혼자 쓰는 브랜치) | reset + force push (--force-with-lease) |
| push 후 (팀 공유 브랜치) | **revert 사용** (히스토리 보존) |

---

## 3. 실습 / 명령어 예제

### 3.1 git reset

```bash
# 직전 커밋 취소 (변경사항은 Staging Area에 유지)
git reset --soft HEAD~1

# 직전 커밋 취소 (변경사항은 Working Tree에 유지, 기본값)
git reset HEAD~1
git reset --mixed HEAD~1   # 명시적

# 직전 커밋 취소 + 변경사항 완전 삭제
git reset --hard HEAD~1

# 특정 커밋으로 이동
git reset --hard <COMMIT_HASH>

# 특정 파일만 스테이징 취소 (커밋은 그대로)
git reset HEAD <FILE>
git restore --staged <FILE>   # 최신 방식

# 특정 파일을 특정 커밋 상태로 복구
git reset <COMMIT_HASH> -- <FILE>
```

### 3.2 git revert

```bash
# 특정 커밋 되돌리기 (새 커밋 생성)
git revert <COMMIT_HASH>

# 직전 커밋 되돌리기
git revert HEAD

# 커밋 없이 스테이징까지만 (직접 커밋 메시지 작성)
git revert -n <COMMIT_HASH>
git revert --no-commit <COMMIT_HASH>

# 여러 커밋 되돌리기
git revert HEAD~3..HEAD   # 최근 3개 커밋 각각 revert

# merge 커밋 되돌리기 (부모 지정 필요)
git revert -m 1 <MERGE_COMMIT_HASH>
# -m 1: 첫 번째 부모(main)를 기준으로 revert
```

### 3.3 실전 시나리오

```bash
# 시나리오 1: 커밋 3개를 하나로 정리 후 다시 커밋
git reset --soft HEAD~3   # 3개 커밋 취소 (변경사항은 스테이징에)
git commit -m "feat: 기능 A 완성"   # 하나로 커밋

# 시나리오 2: 잘못 커밋한 파일만 제거
git reset HEAD~1   # 커밋 취소 (변경사항은 Working Tree에)
git restore --staged <WRONG_FILE>   # 해당 파일만 스테이징 제거
git checkout -- <WRONG_FILE>        # 변경사항 버리기
git add <CORRECT_FILES>
git commit -m "수정된 커밋"

# 시나리오 3: push된 커밋 팀 공유 브랜치에서 되돌리기
git revert <COMMIT_HASH>   # 새 revert 커밋 생성
git push origin main        # push (force push 불필요)
```

---

## 4. 자주 하는 실수 & 주의사항

### reset --hard 후 복구

> ⚠️ WARNING: `git reset --hard`는 Working Tree 변경사항을 영구 삭제. commit된 내용은 reflog로 복구 가능하나 commit 안 된 변경사항은 복구 불가.

```bash
# reset --hard 후 커밋된 내용 복구 (reflog 활용)
git reflog
# 이전 커밋 해시 확인
git reset --hard <이전_COMMIT_HASH>

# 또는 새 브랜치로 복구
git branch recovered <이전_COMMIT_HASH>
```

### 공유 브랜치에서 reset 후 force push

> ⚠️ WARNING: 팀원이 이미 pull한 커밋을 reset + force push하면 팀원 로컬 히스토리와 충돌 발생.

```bash
# 공유 브랜치에서는 절대 금지
git reset HEAD~1
git push --force origin main   # ← 팀원 히스토리 파괴

# 대신 revert 사용
git revert HEAD
git push origin main
```

### revert 후 다시 기능 복구 시 cherry-pick 불가

**상황**: A 기능을 revert했다가 다시 되살리려 할 때 original commit을 cherry-pick하면 revert가 또 revert되어 원래대로.

```bash
# 올바른 방법: revert 커밋을 revert
git revert <REVERT_COMMIT_HASH>
```

---

## 5. 실무 팁

```bash
# reset 전 항상 현재 위치 기록
git log --oneline -5

# reset 후 잘못됐으면 ORIG_HEAD로 복구
git reset --hard ORIG_HEAD   # reset 이전 상태로 즉시 복구

# 특정 파일을 n번 전 커밋 상태로 복구
git restore --source=HEAD~2 -- <FILE>

# reset과 reflog 조합으로 어떤 실수든 복구 가능
git reflog   # 모든 HEAD 이동 기록
git reset --hard <원하는_COMMIT_HASH>
```

---

## 참고 자료

- [git reset 공식 문서](https://git-scm.com/docs/git-reset)
- [git revert 공식 문서](https://git-scm.com/docs/git-revert)
- 관련 문서: `docs/05-advanced/reflog-guide.md`
- 관련 문서: `docs/06-troubleshooting/undo-mistakes.md`
