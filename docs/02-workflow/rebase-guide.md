# Rebase 가이드

> **카테고리**: `docs/02-workflow/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

`git rebase`는 커밋들을 다른 베이스(base) 위로 "재배치"하는 명령어.
merge와 달리 히스토리를 선형(직선)으로 만들어 깔끔하게 유지.
단, **이미 push된 공유 브랜치에서의 rebase는 팀원 히스토리와 충돌을 일으키므로 주의**.

---

## 2. 원리 / 개념 설명

### 2.1 rebase 동작 원리

```text
Before: (feature가 B에서 갈라짐)
  A - B - C - D  (main)
      \
       E - F  (HEAD → feature)

git rebase main 실행:
  1. E, F 커밋을 임시 보관 (patch)
  2. feature 시작점을 main의 최신(D)으로 이동
  3. E, F를 D 위에 재적용 → E', F' (새 커밋 해시!)

After:
  A - B - C - D  (main)
                \
                 E' - F'  (HEAD → feature)
  # E와 E'는 내용은 같지만 해시가 다름!
```

### 2.2 rebase vs merge 비교

```text
merge 결과 (히스토리 보존):
  A - B - C - D - M  (main)
      \          /
       E - F  (feature)

rebase 결과 (선형 히스토리):
  A - B - C - D - E' - F'  (main에 fast-forward 가능)
```

| 항목 | merge | rebase |
|------|-------|--------|
| 히스토리 | 분기 보존 (사실 반영) | 선형으로 정리 (보기 좋음) |
| 커밋 해시 | 그대로 | 재생성됨 |
| 충돌 해결 | 한 번 (merge 시) | 커밋마다 발생 가능 |
| 공유 브랜치 | 안전 | ⚠️ 위험 |

### 2.3 Interactive Rebase (-i)

특정 범위의 커밋을 수정/삭제/합치기/순서 변경.

```text
git rebase -i HEAD~3
# HEAD에서 3개 커밋을 대화형으로 편집

에디터에서:
  pick a1b2c3d commit A
  pick b2c3d4e commit B
  pick c3d4e5f commit C

명령어:
  pick   = 그대로 사용
  reword = 메시지 수정
  edit   = 커밋 내용 수정 (중간에 멈춤)
  squash = 이전 커밋과 합치기 (메시지 합침)
  fixup  = 이전 커밋과 합치기 (메시지 버림)
  drop   = 커밋 삭제
  reorder= 줄 순서 바꿔서 커밋 순서 변경
```

---

## 3. 실습 / 명령어 예제

### 3.1 기본 rebase

```bash
# feature 브랜치를 main 최신으로 rebase
git switch feature
git rebase main

# rebase 후 main에 fast-forward merge
git switch main
git merge feature   # Fast-forward (merge 커밋 없음)
```

### 3.2 Interactive Rebase 실전

```bash
# 최근 3개 커밋 정리
git rebase -i HEAD~3

# 에디터에서:
# pick → squash: 이전 커밋과 합치기
# pick → reword: 메시지 수정
# pick → drop: 커밋 삭제

# 저장 후 닫으면 자동 적용
# 충돌 발생 시:
git status
# 해결 후
git add <FILE>
git rebase --continue

# 전체 취소
git rebase --abort
```

### 3.3 커밋 메시지 여러 개 한꺼번에 수정

```bash
git rebase -i HEAD~5   # 최근 5개 커밋

# 에디터에서 수정할 커밋을 pick → reword로 변경
# 저장 후 각 커밋마다 메시지 에디터가 열림
```

### 3.4 커밋 3개를 1개로 합치기 (Squash)

```bash
git rebase -i HEAD~3

# 에디터:
#   pick   a1b2c "feat: A 기능"
#   squash b2c3d "feat: A 기능 보완"
#   squash c3d4e "feat: A 기능 테스트"

# 저장 후 하나의 커밋 메시지 에디터 열림
# 최종 메시지 작성 후 저장
```

### 3.5 특정 커밋만 수정 (edit)

```bash
git rebase -i HEAD~3

# 수정할 커밋을 pick → edit으로 변경 후 저장
# 해당 커밋에서 rebase가 멈춤

# 파일 수정 후
git add <FILE>
git commit --amend   # 해당 커밋 수정
git rebase --continue   # 나머지 커밋 계속
```

### 3.6 pull --rebase

```bash
# 원격 변경사항을 rebase로 적용 (merge 커밋 없이)
git pull --rebase origin main

# 기본 pull 방식을 rebase로 설정
git config --global pull.rebase true
```

---

## 4. 자주 하는 실수 & 주의사항

### 공유 브랜치에서 rebase (가장 큰 실수)

> ⚠️ WARNING: `main`, `develop` 등 팀원이 공유하는 브랜치에서 rebase하면 커밋 해시가 바뀌어 팀원 로컬과 충돌.

```bash
# 금지: 공유 브랜치에서 rebase
git switch main
git rebase origin/main   # ← 이미 공유된 커밋이 있으면 위험

# 안전한 패턴: 내 feature 브랜치를 main 위로 rebase
git switch feature
git rebase main   # ← feature는 내 브랜치이므로 안전
```

### rebase 충돌이 커밋마다 발생

**원인**: 오래된 브랜치를 최신 main에 rebase 시 각 커밋이 개별로 적용되므로 충돌 반복.
```bash
# 방법 1: 충돌을 하나씩 해결 (rebase --continue 반복)
# 방법 2: squash로 먼저 합친 후 rebase
git rebase -i HEAD~N   # 먼저 squash로 1개로 합치기
git rebase main        # 그 다음 rebase (충돌 1회만)
```

### rebase 후 push 거부

```bash
# 로컬 커밋 해시가 바뀌어서 원격과 달라짐
git push origin feature
# ! [rejected] feature -> feature (non-fast-forward)

# 내 혼자 쓰는 브랜치라면 force-with-lease 사용
git push --force-with-lease origin feature
```

---

## 5. 실무 팁

```bash
# rebase 전 현재 상태 기록 (reflog로도 복구 가능하나 명시적 백업)
git branch backup/feature-before-rebase

# rebase 취소하고 이전 상태로
git rebase --abort   # 진행 중인 rebase 취소

# rebase 완료 후 잘못됐으면 reflog로 복구
git reflog
git reset --hard <이전_COMMIT_HASH>
```

**팀 규칙 제안**: PR 올리기 전에 `git rebase main`으로 최신 상태 유지 → reviewer가 최신 코드와의 차이만 보면 됨.

---

## 참고 자료

- [Git rebase 공식 문서](https://git-scm.com/docs/git-rebase)
- 관련 문서: `docs/02-workflow/merge-guide.md`
- 관련 문서: `docs/05-advanced/reflog-guide.md`
- 관련 문서: `docs/06-troubleshooting/undo-mistakes.md`
