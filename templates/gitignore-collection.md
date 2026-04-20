# .gitignore 템플릿 모음

> 언어/프레임워크별 .gitignore 핵심 항목 모음.
> 전체 템플릿은 [github.com/github/gitignore](https://github.com/github/gitignore) 참조.

---

## 공통 (항상 포함)

```gitignore
# OS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
Thumbs.db
ehthumbs.db
Desktop.ini

# 에디터
.vscode/
.idea/
*.swp
*.swo
*~
.project
.classpath

# 환경 변수 / 시크릿
.env
.env.*
!.env.example
*.pem
*.key
*.p12
*.pfx
secrets/
credentials/

# 로그
*.log
logs/
npm-debug.log*
yarn-error.log
```

---

## Node.js / JavaScript

```gitignore
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnp
.pnp.js
.yarn/cache
.yarn/unplugged
.yarn/build-state.yml

# 빌드
dist/
build/
out/
.next/
.nuxt/
.vuepress/dist/

# 테스트
coverage/
.nyc_output/

# TypeScript
*.tsbuildinfo

# 패키지 매니저 (하나만 커밋)
# npm을 쓰면 yarn.lock 무시, 반대도 동일
# yarn.lock
# package-lock.json
```

---

## Python

```gitignore
# 바이트코드
__pycache__/
*.py[cod]
*$py.class
*.pyc

# 가상환경
venv/
.venv/
env/
.env/
ENV/
.python-version

# 빌드
*.egg-info/
dist/
build/
*.egg
.eggs/
wheels/

# 테스트
.pytest_cache/
.coverage
.coverage.*
htmlcov/
.tox/

# 타입 체킹
.mypy_cache/
.dmypy.json
.pytype/

# Jupyter
.ipynb_checkpoints/
*.ipynb  # 민감 데이터 포함 가능성

# pip
pip-log.txt
pip-delete-this-directory.txt
```

---

## Java / Kotlin / Spring

```gitignore
# 빌드
target/
build/
out/
*.class
*.jar
*.war
*.ear
*.nar

# Maven
.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

# Gradle
.gradle/
gradle-app.setting
!gradle-wrapper.jar
!**/src/main/**/build/
!**/src/test/**/build/

# IDE
*.iml
.idea/
.idea/workspace.xml
.idea/tasks.xml
.idea/shelf/

# Spring Boot
application-local.yml
application-local.properties
application-prod.yml
application-prod.properties
```

---

## Go

```gitignore
# 바이너리
*.exe
*.exe~
*.dll
*.so
*.dylib

# 테스트
*.test
*.out

# Go Modules
vendor/
go.sum  # 주의: 보통 커밋함 (재현 가능한 빌드)

# 빌드 결과
/bin/
/dist/
```

---

## Rust

```gitignore
# 빌드
/target/
Cargo.lock  # 라이브러리면 무시, 바이너리면 커밋

# 테스트 결과
**/*.rs.bk
```

---

## Docker / 인프라

```gitignore
# Docker
.docker/
docker-compose.override.yml
docker-compose.local.yml

# Kubernetes
*.kubeconfig
kubeconfig

# Terraform
.terraform/
*.tfstate
*.tfstate.*
*.tfvars         # 민감 변수
!example.tfvars

# Ansible
*.retry
vault_password_file
```

---

## 글로벌 .gitignore 설정

```bash
# 모든 레포에 공통 적용 (OS, 에디터 등)
git config --global core.excludesfile ~/.gitignore_global

# ~/.gitignore_global
.DS_Store
.vscode/
.idea/
*.swp
Thumbs.db
.env
```

---

## .gitignore 규칙 문법

```gitignore
# 주석
*.log          # 모든 .log 파일
!important.log # 예외 (important.log는 추적)
/TODO          # 루트의 TODO만 (하위 폴더 제외)
build/         # build 폴더 전체
doc/*.txt      # doc 폴더의 txt (하위 폴더 제외)
doc/**/*.txt   # doc 폴더의 모든 txt (하위 폴더 포함)
```

---

## 이미 커밋된 파일 gitignore 적용

```bash
# .gitignore 추가 후에도 추적되는 파일 제거
git rm --cached <FILE>           # 단일 파일
git rm --cached -r <FOLDER>/     # 폴더

git add .gitignore
git commit -m "chore: .gitignore 추가 및 캐시 제거"
```
