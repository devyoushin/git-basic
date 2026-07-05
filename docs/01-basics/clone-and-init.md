# Git Init & Clone 가이드

> **카테고리**: `docs/01-basics/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

`git init`은 빈 디렉토리 또는 기존 프로젝트를 Git 레포지터리로 초기화.
`git clone`은 원격 레포지터리를 로컬에 복사.
둘 다 `.git/` 디렉토리를 만들며, 이 안에 Git의 모든 데이터가 저장됨.

---

## 2. 원리 / 개념 설명

### 2.1 .git 디렉토리 구조

```text
.git/
├── HEAD           ← 현재 브랜치 포인터 ("ref: refs/heads/main")
├── config         ← 레포별 설정
├── description    ← 레포 설명 (GitWeb용, 일반적으로 무시)
├── hooks/         ← Git Hook 스크립트
├── info/          ← exclude 파일 등
├── objects/       ← 모든 Git 오브젝트 (blob, tree, commit, tag)
│   ├── pack/      ← 압축된 오브젝트
│   └── info/
└── refs/
    ├── heads/     ← 로컬 브랜치 (main, feature/... 등)
    └── remotes/   ← 원격 브랜치 참조
        └── origin/
```

### 2.2 로컬 ↔ 원격 관계

```text
origin (GitHub)                    local
━━━━━━━━━━━━━━━━━━━━               ━━━━━━━━━━━━━━━━━━━━━━━━
main ──→ [A][B][C]    fetch/pull   origin/main ──→ [A][B][C]
                       ←────────   main        ──→ [A][B][C][D]
                       push        (D는 아직 push 안 됨)
```

---

## 3. 실습 / 명령어 예제

### 3.1 git init

```bash
# 새 프로젝트 시작
mkdir my-project && cd my-project
git init
# Initialized empty Git repository in .git/

# 기존 프로젝트에 Git 추가
cd existing-project
git init

# 브랜치 이름 지정 (기본값 main으로)
git init -b main
# 또는 설정으로
git config --global init.defaultBranch main

# Bare 레포지터리 (서버 용도, Working Tree 없음)
git init --bare my-repo.git
```

### 3.2 git clone

```bash
# 기본 clone
git clone https://github.com/user/repo.git

# 로컬 디렉토리 이름 지정
git clone https://github.com/user/repo.git my-project

# 특정 브랜치만 clone
git clone -b develop https://github.com/user/repo.git

# 최근 커밋만 (히스토리 없음, 빠른 clone)
git clone --depth 1 https://github.com/user/repo.git

# Submodule 포함
git clone --recurse-submodules https://github.com/user/repo.git

# SSH 방식 (인증 설정 필요)
git clone git@github.com:user/repo.git
```

### 3.3 원격 레포 연결 (init 후)

```bash
# 원격 추가
git remote add origin https://github.com/user/repo.git

# 원격 확인
git remote -v
# origin  https://github.com/user/repo.git (fetch)
# origin  https://github.com/user/repo.git (push)

# 첫 push + 추적 관계 설정
git push -u origin main

# 원격 URL 변경 (HTTPS → SSH)
git remote set-url origin git@github.com:user/repo.git

# 원격 제거/이름 변경
git remote remove origin
git remote rename origin upstream
```

### 3.4 Fork 워크플로우 설정

```bash
# 오픈소스 기여 시 일반적인 설정
git clone https://github.com/MY_USER/forked-repo.git
cd forked-repo

# 업스트림(원본) 레포 추가
git remote add upstream https://github.com/ORIGINAL/repo.git

# 확인
git remote -v
# origin    https://github.com/MY_USER/forked-repo.git (fetch)
# upstream  https://github.com/ORIGINAL/repo.git (fetch)

# 업스트림 최신 변경사항 반영
git fetch upstream
git rebase upstream/main
git push origin main
```

---

## 4. 자주 하는 실수 & 주의사항

### clone 후 바로 push 안 되는 경우

```bash
# 원인: 추적 브랜치 미설정 (드문 경우)
git push   # error: no upstream branch

# 해결
git push -u origin main   # -u로 추적 관계 설정
```

### init.defaultBranch 설정 안 된 경우

```bash
# Git 2.28 이전 기본값은 master
git config --global init.defaultBranch main   # 전역 설정
```

### HTTPS vs SSH 선택

```text
HTTPS: 설정 간단, 매번 토큰/비밀번호 필요 (credential manager 사용 시 저장 가능)
SSH:   키 등록 한 번으로 인증 불필요, 자동화/스크립트에 권장

SSH 설정:
  ssh-keygen -t ed25519 -C "email@example.com"
  cat ~/.ssh/id_ed25519.pub  → GitHub Settings > SSH Keys에 추가
```

---

## 5. 실무 팁

```bash
# clone 속도 개선 (대형 레포)
git clone --depth 1 <URL>                 # 최신 커밋만
git clone --filter=blob:none <URL>         # blob 없이 (partial clone)
git clone --filter=tree:0 <URL>            # 메타데이터만

# 나중에 전체 히스토리 가져오기
git fetch --unshallow

# 여러 레포 일괄 clone (스크립트)
cat repos.txt | xargs -I {} git clone git@github.com:company/{}.git

# .git 폴더 크기 확인
du -sh .git
git count-objects -vH
```

---

## 참고 자료

- [git init 공식 문서](https://git-scm.com/docs/git-init)
- [git clone 공식 문서](https://git-scm.com/docs/git-clone)
- 관련 문서: `docs/01-basics/remote.md`
- 관련 문서: `docs/01-basics/setup.md`
