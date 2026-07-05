# Git 초기 설정 가이드

> **카테고리**: `docs/01-basics/`
> **관련 에이전트**: `agents/git-tutor.md`

---

## 1. 개요

Git을 처음 설치하고 사용하기 전 반드시 설정해야 하는 항목들.
이름/이메일 설정부터 SSH 키 등록, 유용한 alias까지 한 번 해두면 평생 쓰는 설정.

---

## 2. 원리 / 개념 설명

### 2.1 Git config 적용 범위 (우선순위 높은 순)

```text
1. Local  — 현재 레포지토리만  (.git/config)
2. Global — 현재 사용자 전체  (~/.gitconfig)
3. System — 시스템 전체       (/etc/gitconfig)
```

더 좁은 범위가 우선. Local 설정이 Global보다 우선 적용됨.

---

## 3. 실습 / 명령어 예제

### 3.1 필수 기본 설정

```bash
# 이름과 이메일 설정 (커밋에 기록됨)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# 기본 브랜치명을 main으로 설정
git config --global init.defaultBranch main

# 줄바꿈 처리 (macOS/Linux)
git config --global core.autocrlf input

# 줄바꿈 처리 (Windows)
git config --global core.autocrlf true

# 현재 설정 전체 확인
git config --list --global
```

### 3.2 기본 에디터 설정

```bash
# VS Code를 기본 에디터로 설정
git config --global core.editor "code --wait"

# Vim
git config --global core.editor vim

# Nano
git config --global core.editor nano
```

### 3.3 SSH 키 생성 및 GitHub 등록

```bash
# 1. SSH 키 생성 (Ed25519 권장)
ssh-keygen -t ed25519 -C "you@example.com"
# 저장 경로: ~/.ssh/id_ed25519 (기본값 Enter)
# 패스프레이즈 설정 권장

# 2. SSH 에이전트에 키 추가
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. 공개키 복사 (GitHub에 등록할 내용)
cat ~/.ssh/id_ed25519.pub
# → GitHub Settings > SSH and GPG keys > New SSH key 에 붙여넣기

# 4. 연결 테스트
ssh -T git@github.com
# 성공: "Hi username! You've successfully authenticated..."
```

### 3.4 유용한 alias 설정

```bash
# 짧은 로그 보기
git config --global alias.lg "log --oneline --graph --decorate --all"

# 현재 상태 짧게
git config --global alias.st "status -s"

# 마지막 커밋 수정 (스테이징된 변경사항 포함)
git config --global alias.amend "commit --amend --no-edit"

# 브랜치 목록 (최근 변경 순 정렬)
git config --global alias.br "branch --sort=-committerdate"

# 사용법: git lg, git st, git amend, git br
```

### 3.5 ~/.gitconfig 직접 편집

```ini
# ~/.gitconfig
[user]
    name = Your Name
    email = you@example.com

[core]
    editor = code --wait
    autocrlf = input

[init]
    defaultBranch = main

[pull]
    rebase = false   # pull 시 merge 방식 (또는 true면 rebase)

[alias]
    lg = log --oneline --graph --decorate --all
    st = status -s
    br = branch --sort=-committerdate
    amend = commit --amend --no-edit

[push]
    autoSetupRemote = true  # Git 2.37+: 첫 push 시 -u 자동 설정
```

### 3.6 특정 레포에만 다른 이메일 사용 (회사 vs 개인)

```bash
# 해당 레포 디렉토리 안에서
git config --local user.email "work@company.com"
git config --local user.name "Work Name"

# 확인
git config --local --list
```

---

## 4. 자주 하는 실수 & 주의사항

### 이메일 설정 없이 커밋한 경우

**증상**: GitHub에서 잔디(contribution)가 안 심어짐
**원인**: 커밋의 이메일이 GitHub 계정에 등록된 이메일과 불일치
**해결**:
```bash
# 이메일 확인
git log --format="%ae" | head -5

# 마지막 커밋의 이메일 수정 (push 전)
git commit --amend --reset-author --no-edit

# 여러 커밋 수정은 rebase -i 필요
```

### SSH vs HTTPS 선택

| 방식 | 장점 | 단점 |
|------|------|------|
| SSH | 비밀번호 없이 편리 | 초기 설정 필요 |
| HTTPS | 설정 간단 | 매번 인증 (credential 저장 가능) |

```bash
# 기존 레포를 HTTPS → SSH로 변경
git remote set-url origin git@github.com:<USER>/<REPO>.git

# 반대로
git remote set-url origin https://github.com/<USER>/<REPO>.git
```

---

## 5. 실무 팁

```bash
# 설정 파일 직접 열기
git config --global --edit

# 특정 설정값만 확인
git config --global user.email

# 설정 삭제
git config --global --unset alias.old-alias

# 어떤 설정 파일에서 값이 왔는지 확인
git config --list --show-origin
```

**팀 협업 시**: 팀 레포에 `.gitconfig-team` 파일을 두고 `git config --local include.path ../.gitconfig-team`으로 공통 설정 공유.

---

## 참고 자료

- [Git 공식 설정 문서](https://git-scm.com/book/ko/v2/Git맞춤-Git-설정하기)
- 관련 문서: `docs/01-basics/clone-and-init.md`
