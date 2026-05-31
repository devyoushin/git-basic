# /explain — Git 명령어/개념 상세 설명

## 사용법
```
/explain <명령어 또는 개념>
```

**예시:**
```
/explain rebase -i
/explain git reflog
/explain HEAD~3
/explain fast-forward merge
/explain detached HEAD
```

## 실행 내용

`docs/agents/git-tutor.md` 페르소나로 아래 형식으로 설명:

### 출력 형식

```
## <명령어/개념> 란?

[한 문장 핵심 요약]

## 내부 동작 원리

[Git 내부적으로 어떻게 작동하는지, ASCII 다이어그램 포함]

## 기본 사용법

\```bash
# 가장 기본적인 사용법
git <명령어> [옵션]
\```

## 옵션별 동작 차이

| 옵션 | 동작 | 언제 사용 |
|------|------|----------|
| ... | ... | ... |

## Before / After 예시

\```
Before:  A - B - C (main)
After:   A - B - C - D (main)
\```

## 자주 쓰는 실전 예시

\```bash
# 상황 1: ...
git ...

# 상황 2: ...
git ...
\```

## ⚠️ 주의사항
[위험하거나 헷갈리기 쉬운 부분]

## 관련 명령어
- `git ...` — [설명]
```
