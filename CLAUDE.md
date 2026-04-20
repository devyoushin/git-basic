# CLAUDE.md — git-basic 지식 베이스

Git 기초부터 고급 기능까지 실무에서 바로 쓸 수 있도록 정리한 개인 참조 베이스.
명령어 암기보다 **왜 이렇게 동작하는지** 원리 이해 중심으로 작성.

## 프로젝트 설정

- **대상**: Git 전반 (기초 ~ 고급)
- **관점**: 실무 DevOps / 협업 워크플로우 중심
- **언어**: 한국어 (명령어/기술 용어는 영어 원문 병기)

---

## 프로젝트 구조

```
git-basic/
├── docs/                                  # 지식 문서 (카테고리별 분류)
│   ├── basics/              (5개)         # 초기 설정, 스테이징, 커밋, 브랜치, 리모트
│   ├── workflow/            (4개)         # merge, rebase, cherry-pick, stash
│   ├── history/             (4개)         # log, diff, blame, reset/revert
│   ├── advanced/            (4개)         # submodule, worktree, reflog, hooks
│   ├── collaboration/       (3개)         # PR 워크플로우, 코드리뷰, 컨벤션
│   └── troubleshooting/     (3개)         # 자주 발생하는 문제 해결
│
├── cheatsheets/                           # 한눈에 보는 명령어 참조
│   ├── daily-commands.md                  # 매일 쓰는 명령어
│   ├── branch-strategy.md                 # 브랜치 전략 요약
│   └── emergency.md                       # 긴급 상황 대처 (실수 복구)
│
├── templates/                             # 재사용 템플릿
│   ├── commit-message.md                  # 커밋 메시지 템플릿
│   ├── gitignore-collection.md            # 언어/프레임워크별 .gitignore
│   └── git-hooks/                         # 실용 Git Hook 스크립트
│       ├── pre-commit                     # 커밋 전 린트/테스트
│       └── commit-msg                     # 커밋 메시지 형식 검증
│
├── rules/                                 # Claude 작성 규칙
│   ├── doc-writing.md                     # 문서 스타일 가이드
│   └── git-conventions.md                 # Git 명령어/예제 규칙
│
├── agents/                                # Claude 전문 에이전트
│   ├── git-tutor.md                       # Git 개념 설명 전문가
│   ├── workflow-advisor.md                # 브랜치/협업 전략 전문가
│   └── rescue-agent.md                    # 실수 복구 전문가
│
└── .claude/
    ├── settings.json                      # 프로젝트 설정
    └── commands/                          # 커스텀 슬래시 커맨드
        ├── new-doc.md                     # /new-doc
        ├── git-rescue.md                  # /git-rescue
        ├── explain.md                     # /explain
        └── search-kb.md                   # /search-kb
```

---

## 커스텀 슬래시 커맨드

| 커맨드 | 사용법 | 설명 |
|--------|--------|------|
| `/new-doc` | `/new-doc advanced git-bisect` | 신규 문서 스캐폴딩 |
| `/git-rescue` | `/git-rescue "실수로 main에 push했어"` | 긴급 복구 방법 안내 |
| `/explain` | `/explain rebase -i` | 특정 명령어/개념 상세 설명 |
| `/search-kb` | `/search-kb cherry-pick` | 지식 베이스 검색 |

---

## 파일 네이밍 규칙

```
docs/{카테고리}/{주제}.md
```

- 카테고리: `basics`, `workflow`, `history`, `advanced`, `collaboration`, `troubleshooting`
- 주제: 소문자 영어, 하이픈 구분
- 예시: `docs/workflow/rebase-guide.md`, `docs/advanced/git-hooks-guide.md`

---

## 문서 작성 원칙

1. **원리 우선** — 명령어 나열보다 왜 동작하는지 내부 구조 설명
2. **Before/After** — 명령어 실행 전후 상태 변화를 항상 시각화
3. **위험도 표시** — 되돌리기 어려운 명령어는 반드시 ⚠️ 경고
4. **실무 맥락** — 혼자 쓰는 상황 vs 팀 협업 상황을 구분해서 설명
5. **한국어 기술 문서** — 명령어/옵션은 영어 원문 그대로

세부 규칙은 `rules/` 디렉토리를 참조.

---

## 카테고리별 문서 목록

### docs/basics/
| 파일 | 주제 |
|------|------|
| `setup.md` | 초기 설정, config, SSH 키 등록 |
| `staging-and-commit.md` | add, commit, 스테이징 영역 원리 |
| `branch.md` | 브랜치 생성/전환/삭제, HEAD 개념 |
| `remote.md` | remote, fetch, pull, push |
| `clone-and-init.md` | init, clone, 로컬/리모트 관계 |

### docs/workflow/
| 파일 | 주제 |
|------|------|
| `merge-guide.md` | Fast-forward, 3-way merge, 충돌 해결 |
| `rebase-guide.md` | rebase 원리, interactive rebase, 주의사항 |
| `cherry-pick-guide.md` | cherry-pick 활용, 충돌 처리 |
| `stash-guide.md` | stash 사용법, stash pop vs apply |

### docs/history/
| 파일 | 주제 |
|------|------|
| `log-and-search.md` | log 옵션, grep, pickaxe, 시각화 |
| `diff-guide.md` | diff, difftool, 스테이징 전후 비교 |
| `blame-and-bisect.md` | blame으로 변경자 추적, bisect로 버그 커밋 이진 탐색 |
| `reset-and-revert.md` | reset (soft/mixed/hard), revert 차이 |

### docs/advanced/
| 파일 | 주제 |
|------|------|
| `reflog-guide.md` | reflog로 삭제된 커밋 복구 |
| `git-hooks-guide.md` | pre-commit, commit-msg, pre-push 훅 |
| `submodule-guide.md` | submodule 추가/업데이트/제거 |
| `worktree-guide.md` | worktree로 여러 브랜치 동시 작업 |

### docs/collaboration/
| 파일 | 주제 |
|------|------|
| `pr-workflow.md` | PR 생성, 리뷰, 머지 전략 |
| `branch-strategy.md` | Git Flow, GitHub Flow, Trunk-based 비교 |
| `commit-convention.md` | Conventional Commits, 커밋 메시지 작성법 |

### docs/troubleshooting/
| 파일 | 주제 |
|------|------|
| `undo-mistakes.md` | 실수 복구 모음 (push 취소, 파일 복구 등) |
| `conflict-resolution.md` | 충돌 해결 전략, 도구 활용 |
| `large-file-issues.md` | 대용량 파일 문제, Git LFS |

---

## 추가 예정 주제 (백로그)

- `docs/advanced/git-bisect-guide.md` — 버그 커밋 이진 탐색 심화
- `docs/advanced/sparse-checkout.md` — 대형 레포 부분 체크아웃
- `docs/collaboration/monorepo-strategy.md` — 모노레포 Git 전략
- `docs/advanced/git-internals.md` — objects, refs, pack 파일 구조
