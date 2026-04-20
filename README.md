# git-basic

Git 기초부터 고급 기능까지 Claude와 함께 체계적으로 정리한 개인 지식 베이스.
명령어 암기보다 **원리 이해** 중심, **실무 맥락** 중심으로 작성.

---

## 빠른 시작

### 이 레포에서 Claude 사용하기

Claude Code(`claude` CLI) 또는 Claude.ai에서 이 디렉토리를 열면
`CLAUDE.md`를 자동으로 읽어 Git 전문가 모드로 응답합니다.

```bash
cd git-basic
claude   # Claude Code CLI 실행
```

### 커스텀 슬래시 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/git-rescue "실수 내용"` | 긴급 복구 방법 즉시 안내 |
| `/explain rebase -i` | 명령어/개념 상세 설명 |
| `/new-doc advanced git-bisect` | 새 문서 스캐폴딩 |
| `/search-kb cherry-pick` | 지식 베이스 검색 |

---

## 학습 로드맵

### Phase 1: 기초 (basics/)
```
setup.md          → git config, SSH 키, alias 설정
clone-and-init.md → init, clone, .git 구조 이해
staging-and-commit.md → 3-zone 모델, blob/tree/commit 원리
branch.md         → 브랜치 = 포인터, HEAD, detached HEAD
remote.md         → fetch vs pull, 원격 추적 브랜치
```

### Phase 2: 워크플로우 (workflow/)
```
merge-guide.md    → Fast-forward / 3-way merge / squash
rebase-guide.md   → rebase 원리, interactive rebase, 주의사항
stash-guide.md    → stash 스택, pop vs apply
cherry-pick-guide.md → 특정 커밋만 가져오기
```

### Phase 3: 히스토리 탐색 (history/)
```
log-and-search.md → log 옵션, pickaxe(-S), 함수 이력(-L)
diff-guide.md     → staged vs HEAD, 브랜치 비교
blame-and-bisect.md → 변경 책임자 추적, 버그 커밋 이진 탐색
reset-and-revert.md → soft/mixed/hard 차이, 공유 브랜치에서 revert
```

### Phase 4: 고급 기능 (advanced/)
```
reflog-guide.md   → HEAD 이동 기록, 90일 복구 창구
git-hooks-guide.md → pre-commit, commit-msg, pre-push 자동화
submodule-guide.md → 외부 레포 포함, 커밋 고정
worktree-guide.md → 여러 브랜치 동시 작업
```

### Phase 5: 협업 (collaboration/)
```
commit-convention.md → Conventional Commits, 커밋 메시지 원칙
branch-strategy.md → Git Flow / GitHub Flow / Trunk-based 비교
pr-workflow.md    → PR 작성, 리뷰 에티켓, 머지 전략
```

### Phase 6: 트러블슈팅 (troubleshooting/)
```
undo-mistakes.md   → 실수 유형별 복구 레시피
conflict-resolution.md → 충돌 해결, mergetool 설정
large-file-issues.md → Git LFS, 히스토리에서 파일 제거
```

---

## 빠른 참조

| 상황 | 파일 |
|------|------|
| 매일 쓰는 명령어 | `cheatsheets/daily-commands.md` |
| 긴급 복구 | `cheatsheets/emergency.md` |
| 브랜치 전략 비교 | `cheatsheets/branch-strategy.md` |
| 커밋 메시지 작성 | `templates/commit-message.md` |
| .gitignore 설정 | `templates/gitignore-collection.md` |

---

## 디렉토리 구조

```
git-basic/
├── CLAUDE.md                  # Claude 프로젝트 설정 (자동 로드)
├── README.md
├── docs/
│   ├── basics/                # Git 기초 (5개)
│   ├── workflow/              # 워크플로우 (4개)
│   ├── history/               # 히스토리 탐색 (4개)
│   ├── advanced/              # 고급 기능 (4개)
│   ├── collaboration/         # 협업 (3개)
│   └── troubleshooting/       # 트러블슈팅 (3개)
├── cheatsheets/               # 빠른 참조 (3개)
├── templates/                 # 재사용 템플릿
│   ├── commit-message.md
│   ├── gitignore-collection.md
│   └── git-hooks/             # Hook 스크립트
├── agents/                    # Claude 전문 에이전트 (3개)
├── rules/                     # 문서 작성 규칙 (2개)
└── .claude/
    ├── settings.json
    └── commands/              # 슬래시 커맨드 (4개)
```

---

## Claude 에이전트

| 에이전트 | 역할 |
|----------|------|
| `agents/git-tutor.md` | Git 개념, 내부 구조, 원리 설명 |
| `agents/workflow-advisor.md` | 브랜치 전략, rebase, hooks, 협업 |
| `agents/rescue-agent.md` | 긴급 복구, reflog, 실수 대응 |
