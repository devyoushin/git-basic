# git-basic

Git 기초부터 고급 기능까지 원리 중심, 실무 맥락 중심으로 정리한 개인 지식 베이스입니다.

## 어디서 시작할까

- 문서 지도: `docs/README.md`
- 운영/실습 자산: `ops/README.md`
- AI 작업 지침: `CLAUDE.md`, `AGENTS.md -> CLAUDE.md`

## 구조

| 경로 | 내용 |
|------|------|
| `docs/` | Git 기초, 워크플로우, 히스토리, 협업, 트러블슈팅, 치트시트, 에이전트, 규칙, 템플릿 |
| `ops/` | Git hook 등 실행 가능한 운영 자산 |
| `.claude/` | Claude Code 커맨드와 설정 |
| `CLAUDE.md` | Claude/Codex 공통 작업 지침 원본 |
| `AGENTS.md -> CLAUDE.md` | Codex/agent 작업 지침 링크 |

## 학습 흐름

1. `docs/01-basics/`에서 Git 저장소, stage, commit, branch, remote 기본기 학습
2. `docs/02-workflow/`에서 merge, rebase, stash, cherry-pick 학습
3. `docs/04-history/`에서 log, diff, blame, bisect, reset/revert 학습
4. `docs/03-collaboration/`, `docs/06-troubleshooting/`에서 협업과 복구 패턴 학습
5. `ops/git-hooks/`에서 hook 스크립트 예제 확인
