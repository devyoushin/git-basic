# /search-kb — 지식 베이스 검색

## 사용법
```
/search-kb <키워드>
```

**예시:**
```
/search-kb cherry-pick
/search-kb 충돌 해결
/search-kb rebase 취소
/search-kb force push
```

## 실행 내용

`docs/` 및 `docs/cheatsheets/` 전체를 대상으로 검색 후 관련 내용 요약 반환.

### 출력 형식

```
## 검색 결과: "<키워드>"

### 1. docs/<카테고리>/<파일>.md — ## <섹션명>
<관련 내용 2~3문장 요약>
\```bash
# 핵심 명령어
\```

### 2. docs/cheatsheets/<파일>.md
<요약>

---
빠른 참조: `docs/cheatsheets/daily-commands.md`
```
