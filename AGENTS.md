# AGENTS.md — git-basic Codex 작업 지침

이 저장소는 Git 원리와 실무 워크플로우 지식 베이스입니다. Codex 작업 시 `CLAUDE.md`와 `docs/rules/`의 규칙을 동일하게 따릅니다.

## 공통 원칙

- 설명 문서는 `docs/` 아래에 둡니다.
- 실행 가능한 hook이나 스크립트는 `ops/` 아래에 둡니다.
- Git 명령은 위험도와 공유 브랜치 영향 여부를 명확히 표시합니다.
- 복구 문서는 reflog, reset, revert, restore의 차이를 분명히 설명합니다.

## Claude와의 싱크

- `CLAUDE.md`는 Claude용 상세 지침입니다.
- `AGENTS.md`는 Codex용 진입점입니다.
- 공통 규칙은 `docs/rules/`를 기준으로 유지합니다.

## 작업 체크리스트

- destructive Git 명령은 명시적 요청 없이는 사용하지 않습니다.
- 링크 검사와 `git diff --check`를 수행합니다.
- hook 파일을 변경하면 실행 권한을 확인합니다.
