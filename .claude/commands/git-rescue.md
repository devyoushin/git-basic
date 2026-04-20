# /git-rescue — 긴급 복구 안내

## 사용법
```
/git-rescue "<상황 설명>"
```

**예시:**
```
/git-rescue "실수로 main에 직접 commit하고 push 해버렸어"
/git-rescue "git reset --hard 했는데 작업 내용이 다 날아갔어"
/git-rescue "wrong 브랜치에서 3개 커밋을 해버렸어"
/git-rescue "force push로 팀원 커밋이 덮어씌워졌어"
```

## 실행 내용

`agents/rescue-agent.md` 페르소나로 아래 순서로 응답:

### 출력 형식

```
## 🚨 상황 파악
[상황 요약 및 위험도: HIGH / MEDIUM / LOW]

## 📋 현재 상태 확인 먼저
\```bash
# 지금 상태를 확인하는 명령어 (실행 전 필수)
git status
git log --oneline -5
git reflog | head -10
\```

## 🛠️ 복구 방법

### Step 1. ...
\```bash
# 명령어
\```
> ⚠️ WARNING: [위험한 명령어가 있으면 경고]

### Step 2. ...

## ✅ 복구 확인
\```bash
# 복구 후 상태 확인
\```

## 🔒 재발 방지
- [예방 방법]
```

### 핵심 규칙

- **항상 상태 확인 먼저** — 복구 전 git status / reflog 확인 지시
- **위험 명령어 경고 필수** — reset --hard, push --force 등
- **되돌릴 수 없는 작업 명시** — 복구 불가능한 경우 솔직하게 안내
- **reflog 최우선 확인** — 대부분의 실수는 reflog로 복구 가능
