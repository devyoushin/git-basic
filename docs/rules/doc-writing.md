# 문서 작성 규칙 (Doc Writing Rules)

---

## 1. 언어 및 문체

- **기본 언어**: 한국어
- **영어 병기**: 기술 용어 첫 등장 시 영어 원문 병기
  - 예: 스테이징 영역(Staging Area), 작업 트리(Working Tree), 헤드(HEAD)
- **문체**: 명사형 종결 (`~합니다` 대신 `~함`, `~필요`)
- **존댓말 금지**: 기술 문서 특성상 경어체 사용하지 않음

## 2. 코드 블록

- **언어 태그 필수**: 모든 코드 블록에 언어 지정
  ```bash   # git 명령어
  ```text   # 다이어그램, 출력 결과
  ```ini    # 설정 파일 (.gitconfig)
- **실행 가능한 코드**: 복붙해서 바로 실행 가능한 수준
- **플레이스홀더**: 교체 필요한 값은 `<BRANCH_NAME>`, `<COMMIT_HASH>` 형식

## 3. 섹션 구성 (5개 필수)

| 섹션 | 내용 | 생략 가능 |
|------|------|----------|
| 1. 개요 | 무엇인지, 언제 쓰는지 | 불가 |
| 2. 원리/개념 설명 | 내부 동작, Before/After 다이어그램 | 불가 |
| 3. 실습/명령어 예제 | 복붙 즉시 실행 가능 | 불가 |
| 4. 자주 하는 실수 & 주의사항 | 위험 경고 포함 | 불가 |
| 5. 실무 팁 | 혼자 vs 팀 협업 맥락 구분 | 가능 |

## 4. Before/After 다이어그램 규칙

브랜치 변화가 있는 모든 명령어는 반드시 시각화:

```text
# Before
A - B - C  (main)
        \
         D - E  (feature)

# After (git merge feature)
A - B - C - M  (main)
        \  /
         D - E  (feature)
```

- 현재 HEAD 위치: `(HEAD -> main)` 표시
- 원격 브랜치: `(origin/main)` 표시
- 새로 생성된 커밋: 별도 표시

## 5. 위험 명령어 경고 형식

```
> ⚠️ WARNING: 이 명령어는 되돌리기 어렵습니다. 실행 전 git status와 git log를 반드시 확인하세요.
```

위험 수준 분류:
- **LOW** — 언제든 취소 가능 (add, commit, branch 생성)
- **MEDIUM** — reflog로 복구 가능하나 주의 필요 (reset, rebase)
- **HIGH** — 팀 공유 레포에 영향, 복구 복잡 (force push, rebase on shared branch)
- **CRITICAL** — 복구 불가 가능성 있음 (clean -fd, stash drop)

## 6. 금지 사항

- 추측성 동작 설명 금지 (실제 테스트 기반만)
- 위험 명령어를 경고 없이 그냥 나열 금지
- `--force` 단독 사용 예제 금지 → `--force-with-lease` 권장
