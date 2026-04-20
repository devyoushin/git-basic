# /new-doc — 새 문서 스캐폴딩

## 사용법
```
/new-doc <카테고리> <주제>
```

**예시:**
```
/new-doc advanced git-bisect
/new-doc troubleshooting detached-head
/new-doc workflow squash-merge
```

## 실행 내용

1. `templates/` 의 문서 템플릿 로드
2. `rules/doc-writing.md` 규칙 적용
3. 카테고리에 따라 에이전트 페르소나 적용:
   - `basics` / `workflow` / `history` → `agents/git-tutor.md`
   - `advanced` → `agents/workflow-advisor.md`
   - `troubleshooting` → `agents/rescue-agent.md`
4. 다음 5개 섹션을 포함한 완성 문서 작성:
   - `## 1. 개요` — 무엇인지, 언제 쓰는지
   - `## 2. 원리 / 개념 설명` — 내부 동작, Before/After 시각화
   - `## 3. 실습 / 명령어 예제` — 복붙 즉시 실행 가능
   - `## 4. 자주 하는 실수 & 주의사항` — 위험 명령어 경고 포함
   - `## 5. 실무 팁` — 혼자 vs 팀 협업 맥락 구분
5. `CLAUDE.md`의 카테고리별 문서 목록에 새 파일 추가
