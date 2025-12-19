# Simple Page Server

이 프로젝트는 간단한 행사 페이지를 호스팅하기 위한 서버입니다. Nginx를 기반으로 하며, 단일 서버에서 여러 행사의 페이지(예: `/nscf2025`, `/my-event` 등)를 독립적으로 운영할 수 있도록 구성되어 있습니다.

`content.json` 파일만 수정하면 페이지의 내용(제목, 발표자료, 링크, 영상 등)을 동적으로 변경할 수 있는 구조입니다.

## 📂 프로젝트 구조 (Project Structure)

```text
.
├── docker-compose.yml       # 도커 실행 설정
├── nginx/
│   ├── nginx.conf           # Nginx 서버 설정 (라우팅, 인증, WebDAV 등)
│   └── htpasswd             # 관리자 페이지 접근을 위한 인증 파일
└── public/                  # 정적 파일들이 위치하는 루트 디렉토리
    └── nscf2025/            # [예시] "nscf2025" 행사 전용 폴더
        ├── index.html       # 해당 행사의 진입 페이지
        ├── page.html        # 해당 행사의 안내 페이지
        ├── admin.html       # 해당 행사의 관리자 페이지
        ├── survey.html      # 해당 행사의 설문조사 페이지
        ├── content.json     # 해당 행사의 데이터
        └── answers/         # 해당 행사의 설문조사 응답이 저장되는 곳
```

## 🚀 실행 방법 (Running)

Docker Compose를 사용하여 서버를 실행합니다.

```bash
docker-compose up -d --build
```

- **기본 접속**: `http://localhost/`
- **예시 행사 접속**: `http://localhost/nscf2025/`
- **관리자 페이지**: `http://localhost/nscf2025/admin.html` (ID/PW 설정 필요)

## ✨ 새로운 행사 페이지 추가하기 (Extension)

새로운 행사(예: `devfest2024`)를 추가하려면 `public` 폴더 안에 새 디렉토리를 만들고 템플릿 파일들을 복사하면 됩니다.

### 1단계: 폴더 생성 및 파일 복사
터미널에서 다음 명령어를 실행하거나 파일 탐색기에서 작업하세요.

```bash
# 1. 새 행사 폴더 생성
mkdir public/devfest2024

# 2. 기본 템플릿 파일 복사
cp public/*.html public/content.json public/devfest2024/

# 3. 설문조사 응답 저장 폴더 생성 (필수)
mkdir -p public/devfest2024/answers
```

### 2단계: 내용 수정
`public/devfest2024/content.json` 파일을 열어 해당 행사에 맞는 내용으로 수정합니다.
혹은 브라우저에서 `/devfest2024/admin.html`로 접속하여 GUI 환경에서 수정 후 "서버에 저장하기"를 누르면 됩니다.

**주의**: `answers/` 폴더가 없으면 설문조사 제출이나 엑셀 다운로드가 작동하지 않습니다.

## 🛠 유지보수 및 개발 (Maintenance)

### 1. HTML/JS 템플릿 수정
모든 페이지(`admin.html`, `page.html` 등)는 **상대 경로(Relative Path)**를 사용하도록 개발되었습니다.
- 예: `fetch('/answers/')` 대신 `fetch('answers/')` 사용.
- 이렇게 해야 하위 폴더(예: `/nscf2025/`)에서도 정상적으로 동작합니다.
- 공통 템플릿(`public/*.html`)을 수정한 경우, 개별 행사 폴더(`public/nscf2025/*.html`)에도 덮어씌워야 변경사항이 적용됩니다.

### 2. Nginx 설정
`nginx/nginx.conf`는 정규표현식(Regex)을 사용하여 모든 하위 폴더에 대해 규칙을 적용합니다.
- `location ~ /admin\.html$`: 어느 폴더에 있든 `admin.html`은 인증을 요구합니다.
- `location ~ /content\.json$`: `content.json` 파일의 수정(PUT)은 인증된 사용자만 가능합니다.
- `location ~ /answers/`: 설문조사 응답 폴더는 누구나 쓰기(PUT)가 가능합니다(설문 제출용).

### 3. 관리자 비밀번호 변경
`nginx/htpasswd` 파일을 생성하거나 수정하여 관리자 아이디와 비밀번호를 관리합니다.

```bash
# htpasswd 파일 생성 (기존 파일 덮어쓰기)
htpasswd -cB nginx/htpasswd myuser

# 사용자 추가 또는 비번 변경 (기존 파일 유지)
htpasswd -B nginx/htpasswd anotheruser
```
(htpasswd 유틸리티가 없다면 온라인 생성기를 사용하거나 도커 내부에서 실행하세요.)