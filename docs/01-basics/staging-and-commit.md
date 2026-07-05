# 스테이징과 커밋 가이드

> **카테고리**: `docs/01-basics/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

Git의 가장 핵심적인 세 영역: 작업 트리(Working Tree), 스테이징 영역(Staging Area / Index), 저장소(Repository).
`git add`는 변경사항을 스테이징 영역에 올리고, `git commit`은 스테이징 영역을 영구 스냅샷으로 저장.
이 3단계 구조 덕분에 "무엇을 커밋할지" 세밀하게 제어 가능.

---

## 2. 원리 / 개념 설명

### 2.1 Git의 세 영역

```text
[Working Tree]  →  git add  →  [Staging Area]  →  git commit  →  [Repository]
  실제 파일                      Index/Cache                       .git/objects
  (수정/생성/삭제)               (커밋 예정 스냅샷)                (영구 저장된 커밋)

  ←── git restore <file> ──     ←── git restore --staged <file> ──
  ←──────────── git reset HEAD~1 (--mixed, 기본값) ─────────────
```

### 2.2 git add의 내부 동작

`git add`를 실행하면 Git은:
1. 파일의 현재 내용을 압축하여 **blob object** 생성 (`.git/objects/`)
2. **Index(스테이징 영역)** 에 해당 blob의 참조를 등록

→ 스테이징 영역은 "다음 커밋의 예고편"

### 2.3 git commit의 내부 동작

`git commit`을 실행하면:
1. 스테이징 영역의 파일 목록으로 **tree object** 생성
2. tree + 메타데이터(작성자, 시간, 이전 커밋)로 **commit object** 생성
3. 현재 브랜치 포인터(HEAD)가 새 커밋을 가리킴

```text
Before commit:
  HEAD → main → C2
  Index: [새로운 변경사항]

After git commit:
  HEAD → main → C3 (새 커밋)
                ↑
           tree(C3) → blob(파일들)
```

---

## 3. 실습 / 명령어 예제

### 3.1 기본 add & commit

```bash
# 특정 파일 스테이징
git add <FILE>

# 현재 디렉토리 전체 스테이징
git add .

# 수정/삭제된 파일만 (새 파일 제외)
git add -u

# 변경사항을 대화형으로 선택해서 스테이징 (매우 유용!)
git add -p
# y: 이 hunk 스테이징 / n: 스킵 / s: 더 작게 분할 / e: 직접 편집 / q: 종료

# 스테이징 상태 확인
git status
git diff --staged   # 스테이징된 변경사항 확인
git diff            # 스테이징 안 된 변경사항 확인
```

### 3.2 커밋

```bash
# 에디터로 메시지 작성
git commit

# 인라인 메시지
git commit -m "feat: 로그인 기능 추가"

# 스테이징 + 커밋 한 번에 (새 파일 제외)
git commit -am "fix: 버그 수정"

# 마지막 커밋 수정 (push 전에만!)
git commit --amend -m "수정된 메시지"

# 메시지 변경 없이 마지막 커밋에 파일 추가
git add <FILE>
git commit --amend --no-edit
```

### 3.3 스테이징 취소 (되돌리기)

```bash
# 스테이징 취소 (파일은 그대로, 인덱스만 취소)
git restore --staged <FILE>
git restore --staged .   # 전체

# 파일 변경사항 자체를 버리기 (마지막 커밋 상태로)
git restore <FILE>

# 구식 방법 (동일 효과)
git reset HEAD <FILE>    # 스테이징 취소
git checkout -- <FILE>   # 파일 변경 취소
```

### 3.4 add -p (Interactive Staging) 실전

하나의 파일에 여러 기능 변경이 섞였을 때 분리해서 커밋할 수 있는 핵심 기능:

```bash
git add -p

# 프롬프트 옵션:
# y - 이 변경사항(hunk) 스테이징
# n - 스테이징 안 함
# s - 더 작은 단위로 분할
# e - 직접 편집해서 일부만 스테이징
# q - 종료
# ? - 도움말

# 결과: 같은 파일에서 두 번의 커밋 가능!
git commit -m "feat: 기능 A 추가"
git add -p
git commit -m "fix: 기능 B 버그 수정"
```

---

## 4. 자주 하는 실수 & 주의사항

### git add . 으로 의도치 않은 파일 스테이징

**증상**: 커밋에 `.env`, `node_modules/`, 빌드 파일 등이 포함됨
**예방**:
```bash
# 커밋 전 반드시 확인
git status
git diff --staged

# .gitignore에 미리 추가
echo ".env" >> .gitignore
echo "node_modules/" >> .gitignore
```

### --amend를 push 후에 사용

> ⚠️ WARNING: `git commit --amend`는 커밋 해시를 바꿈. 이미 push한 커밋에 사용하면 force push가 필요하고 팀원 히스토리와 충돌 발생.

```bash
# push 전: amend 자유롭게 사용 가능
# push 후: 절대 혼자 쓰는 브랜치가 아니면 사용 금지
```

### 커밋 메시지에 오타

```bash
# 직전 커밋 메시지 수정 (push 전)
git commit --amend -m "올바른 메시지"

# push 후라면 → 그냥 두거나 팀과 협의 후 force push
```

---

## 5. 실무 팁

```bash
# 커밋 전 습관적으로 실행할 명령어 세트
git status          # 변경 파일 목록
git diff            # 스테이징 전 변경사항
git diff --staged   # 스테이징된 변경사항

# 특정 커밋의 변경 내용 보기
git show <COMMIT_HASH>
git show HEAD        # 직전 커밋

# 빈 커밋 (CI 재실행, 트리거용)
git commit --allow-empty -m "ci: trigger pipeline"
```

**팀 협업 시**: `git add -p`로 논리적 단위로 커밋을 나누면 코드 리뷰가 훨씬 쉬워짐. 하나의 커밋 = 하나의 변경 이유.

---

## 참고 자료

- [Git 스테이징 영역 설명 (공식)](https://git-scm.com/book/ko/v2/Git의-기초-수정하고-저장소에-저장하기)
- 관련 문서: `docs/01-basics/branch.md`
- 관련 문서: `docs/04-history/reset-and-revert.md`
