# Git 대용량 파일 문제 해결

> **카테고리**: `docs/troubleshooting/`
> **관련 에이전트**: `agents/workflow-advisor.md`

---

## 1. 개요

Git은 소스코드 버전 관리에 최적화되어 있으며, 대용량 바이너리 파일에는 취약.
한 번이라도 커밋하면 히스토리에 영원히 남아 레포 크기를 키움.
**예방이 최선**, 발생 후 제거는 force push 필요.

---

## 2. 문제 증상

```text
❌ git push 속도가 매우 느림
❌ git clone이 수 GB 소요
❌ error: RPC failed; HTTP 413 curl 22 (push 거절)
❌ remote: fatal: pack exceeds maximum allowed size
❌ 작은 변경인데 .git 디렉토리가 GB 단위
```

---

## 3. 예방 — .gitignore 설정

### 3.1 즉시 .gitignore에 추가해야 할 파일 유형

```gitignore
# 빌드 결과물
*.jar
*.war
*.ear
*.class
*.o
*.a
dist/
build/
target/

# 미디어 파일
*.mp4
*.mov
*.avi
*.mp3
*.wav
*.psd
*.ai
*.sketch

# 데이터 파일
*.csv      # 대용량이면
*.sql
*.dump
*.db
*.sqlite3

# 패키지
node_modules/
vendor/
.venv/
__pycache__/

# 환경 파일
.env
*.pem
*.key
*.p12
*.pfx

# IDE
.idea/
.vscode/
*.iml
```

### 3.2 커밋 전 파일 크기 확인

```bash
# 스테이징 영역의 큰 파일 찾기
git diff --cached --stat | sort -k1 -rn | head -20

# 100MB 이상 파일 찾기 (커밋 전)
find . -not -path './.git/*' -size +100M
```

---

## 4. 진단 — 레포 크기 문제 분석

### 4.1 레포 크기 확인

```bash
# .git 디렉토리 크기
du -sh .git

# pack 파일 크기
du -sh .git/objects/pack/

# 참조별 디스크 사용량
git count-objects -vH
```

### 4.2 큰 객체 찾기

```bash
# 히스토리에서 큰 파일 TOP 10
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | sed -n 's/^blob //p' \
  | sort -rn -k2 \
  | head -10 \
  | awk '{printf "%s MB\t%s\n", $2/1024/1024, $3}'

# 간단한 방법 (BFG 사용 시)
bfg --no-blob-protection --strip-blobs-bigger-than 10M --dry-run
```

---

## 5. Git LFS (Large File Storage)

### 5.1 설치 및 초기 설정

```bash
# macOS
brew install git-lfs

# 활성화
git lfs install

# 추적할 파일 패턴 등록
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "*.zip"

# .gitattributes 커밋 (필수!)
git add .gitattributes
git commit -m "chore: Git LFS 설정"
```

### 5.2 LFS 사용 확인

```bash
# LFS 추적 파일 목록
git lfs ls-files

# LFS 상태
git lfs status

# LFS 포인터 내용 확인 (실제 파일 아닌 포인터 텍스트)
git lfs pointer --file <FILE>
```

### 5.3 LFS 파일 관리

```bash
# LFS 파일 pull (clone 후 실제 파일 다운로드)
git lfs pull

# 특정 파일만
git lfs pull --include="*.psd"

# LFS 없이 clone (포인터 파일만)
GIT_LFS_SKIP_SMUDGE=1 git clone <URL>

# LFS 마이그레이션 (기존 파일을 LFS로 전환)
git lfs migrate import --include="*.mp4" --everything
git push origin --force --all
```

---

## 6. 히스토리에서 파일 완전 제거

> ⚠️ CRITICAL: force push 필요. 팀원 전체 re-clone 또는 fetch + reset 필요.

### 6.1 BFG Repo Cleaner (권장)

```bash
# 설치
brew install bfg

# 큰 파일 제거 (50MB 이상)
bfg --strip-blobs-bigger-than 50M

# 특정 파일 제거
bfg --delete-files large-dataset.csv

# 폴더 제거
bfg --delete-folders node_modules

# 적용
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force origin --all
```

### 6.2 git filter-repo (공식 권장 도구)

```bash
# 설치
pip install git-filter-repo
# 또는
brew install git-filter-repo

# 특정 파일 제거
git filter-repo --path <FILE> --invert-paths

# 특정 확장자 전체 제거
git filter-repo --path-glob "*.mp4" --invert-paths

# 특정 폴더 제거
git filter-repo --path data/ --invert-paths

# 적용 후 force push
git push origin --force --all
git push origin --force --tags
```

### 6.3 팀원 작업 (force push 후)

```bash
# 팀원이 실행해야 할 명령어
git fetch --all
git reset --hard origin/main   # 또는 해당 브랜치

# 또는 re-clone (가장 깔끔)
cd ..
rm -rf <REPO_DIR>
git clone <URL>
```

---

## 7. 레포 최적화

```bash
# pack 파일 정리 (레포 크기 줄이기)
git gc --aggressive --prune=now

# 불필요한 참조 정리
git remote prune origin

# 레포 상태 진단
git fsck

# 최적화 후 크기 비교
du -sh .git
```

---

## 8. 실무 팁

```bash
# push 전 대용량 파일 자동 차단 (pre-commit hook)
# .git/hooks/pre-commit 에 추가
MAX_SIZE=$((10 * 1024 * 1024))  # 10MB
STAGED=$(git diff --cached --name-only)
for FILE in $STAGED; do
    SIZE=$(stat -f%z "$FILE" 2>/dev/null || stat -c%s "$FILE")
    if [ "$SIZE" -gt "$MAX_SIZE" ]; then
        echo "❌ $FILE is too large ($(($SIZE/1024/1024))MB). Use Git LFS."
        exit 1
    fi
done
```

```bash
# GitHub의 파일 크기 제한
# - 경고: 50MB 이상
# - 거절: 100MB 이상 (LFS 없이)
# - LFS: 최대 2GB per file (유료 플랜에 따라 다름)
```

---

## 참고 자료

- [Git LFS 공식 문서](https://git-lfs.github.com/)
- [BFG Repo Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
- [git-filter-repo](https://github.com/newren/git-filter-repo)
- 관련 문서: `templates/gitignore-collection.md`
