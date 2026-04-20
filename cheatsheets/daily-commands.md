# Git 매일 쓰는 명령어 치트시트

> 자주 사용하는 명령어를 상황별로 정리. 암기보다 참조용.

---

## 📋 상태 확인 (항상 여기서 시작)

```bash
git status              # 변경사항 전체 확인
git status -s           # 짧게 보기
git log --oneline -10   # 최근 10개 커밋
git log --oneline --graph --decorate --all   # 전체 브랜치 그래프
git diff                # 스테이징 전 변경사항
git diff --staged       # 스테이징된 변경사항
```

---

## ➕ 스테이징 & 커밋

```bash
git add <FILE>          # 특정 파일 스테이징
git add .               # 전체 스테이징
git add -p              # 변경사항 일부만 선택적으로 스테이징

git commit -m "메시지"  # 커밋
git commit -am "메시지" # add + commit (새 파일 제외)
git commit --amend --no-edit  # 직전 커밋에 파일 추가 (메시지 유지)
git commit --amend -m "새 메시지"  # 직전 커밋 메시지 수정

git restore --staged <FILE>   # 스테이징 취소
git restore <FILE>            # 파일 변경사항 버리기 (마지막 커밋으로)
```

---

## 🌿 브랜치

```bash
git branch                    # 브랜치 목록
git branch -a                 # 원격 포함 전체 목록
git branch --sort=-committerdate  # 최근 작업 순 정렬

git switch <BRANCH>           # 브랜치 전환
git switch -c <BRANCH>        # 브랜치 생성 + 전환
git switch -c <BRANCH> origin/<BRANCH>  # 원격 브랜치 기반으로 생성

git branch -d <BRANCH>        # 브랜치 삭제 (merge된 것만)
git branch -D <BRANCH>        # 강제 삭제
git push origin --delete <BRANCH>  # 원격 브랜치 삭제
```

---

## 🔄 원격 연동

```bash
git remote -v               # 원격 목록
git fetch origin            # 원격 변경사항 가져오기 (merge 안 함)
git fetch --prune           # 원격에서 삭제된 브랜치 로컬 참조도 정리
git pull origin <BRANCH>    # fetch + merge
git pull --rebase           # fetch + rebase (깔끔한 히스토리)
git push origin <BRANCH>    # push
git push -u origin <BRANCH> # 처음 push (추적 관계 설정)
git push --force-with-lease # 안전한 force push (원격 변경 있으면 실패)
```

---

## 🔀 Merge & Rebase

```bash
git merge <BRANCH>          # 현재 브랜치에 <BRANCH>를 merge
git merge --no-ff <BRANCH>  # merge 커밋 항상 생성
git merge --squash <BRANCH> # 커밋 하나로 합쳐서 merge
git merge --abort           # merge 취소

git rebase main             # 현재 브랜치를 main 위로 rebase
git rebase -i HEAD~3        # 최근 3개 커밋 interactive 편집
git rebase --continue       # 충돌 해결 후 계속
git rebase --abort          # rebase 취소
```

---

## ⏪ 되돌리기

```bash
git reset --soft HEAD~1     # 직전 커밋 취소 (변경사항 → Staging)
git reset HEAD~1            # 직전 커밋 취소 (변경사항 → Working Tree)
git reset --hard HEAD~1     # 직전 커밋 취소 + 변경사항 삭제 ⚠️

git revert <COMMIT_HASH>    # 커밋 취소 (새 커밋 생성, push 후 권장)
git revert HEAD             # 직전 커밋 revert

git restore <FILE>          # 파일을 마지막 커밋 상태로
git restore --source=HEAD~2 -- <FILE>  # 2커밋 전 상태로 파일 복구

git reset --hard ORIG_HEAD  # merge/rebase 직전으로 복구
```

---

## 📦 Stash

```bash
git stash                   # 변경사항 임시 저장
git stash push -m "설명"    # 메시지 포함 저장
git stash push -u           # 새 파일(untracked)도 포함

git stash list              # 저장된 stash 목록
git stash pop               # 가장 최근 stash 적용 + 삭제
git stash apply stash@{1}   # 특정 stash 적용 (목록 유지)
git stash drop stash@{0}    # 특정 stash 삭제
git stash clear             # 모든 stash 삭제 ⚠️
```

---

## 🔍 검색 & 탐색

```bash
git log --oneline --all -S "함수명"   # 특정 코드가 추가/삭제된 커밋
git log --oneline --all --grep "JIRA-123"  # 커밋 메시지 검색
git log --follow -- <FILE>            # 파일 이름 변경 포함 이력
git blame <FILE>                      # 줄별 마지막 수정자/커밋

git show <COMMIT_HASH>                # 커밋 내용 확인
git diff main..feature                # 두 브랜치 차이
git diff HEAD~3 HEAD -- <FILE>        # 파일의 n커밋 전후 비교
```

---

## 🚑 긴급 복구

```bash
git reflog                  # HEAD 이동 기록 전체 (복구 시작점)
git reset --hard HEAD@{N}   # N번 전 상태로 복구
git branch recovered HEAD@{N}  # 특정 시점의 커밋으로 브랜치 생성

git cherry-pick <HASH>      # 다른 브랜치의 특정 커밋만 가져오기
git fsck --lost-found       # 고아 커밋 목록 확인
```

---

## 🏷️ 태그

```bash
git tag                     # 태그 목록
git tag v1.0.0              # Lightweight 태그
git tag -a v1.0.0 -m "Release v1.0.0"  # Annotated 태그 (권장)
git push origin v1.0.0      # 태그 push
git push --tags             # 모든 태그 push
git tag -d v1.0.0           # 로컬 태그 삭제
git push origin --delete v1.0.0  # 원격 태그 삭제
```

---

## ⚙️ 자주 쓰는 설정

```bash
# 유용한 alias
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.st "status -s"
git config --global alias.br "branch --sort=-committerdate"

# pull 기본 rebase로
git config --global pull.rebase true

# 자동 upstream 설정 (Git 2.37+)
git config --global push.autoSetupRemote true
```
