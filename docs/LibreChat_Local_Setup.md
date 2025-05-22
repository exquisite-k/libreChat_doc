# LibreChat 도커 설치 가이드

이 문서는 Docker를 사용하여 로컬 환경에서 LibreChat을 설치하고 실행하는 전체 과정을 자세히 설명합니다. 본 가이드는 Windows 환경에서 Git Bash를 사용하는 것을 기준으로 작성되었습니다.

---

## 🧰 사전 준비 사항

진행하기 전에 다음이 설치되어 있어야 합니다:

* [Git](https://git-scm.com/downloads)
* [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Docker Engine이 실행 중이어야 함)

---

## 📦 1. LibreChat 저장소 클론하기

```bash
git clone https://github.com/danny-avila/LibreChat.git
cd LibreChat
```

> 💡 현재 `dev` 브랜치는 존재하지 않으므로 `main` 브랜치를 기준으로 작업합니다.

---

## ⚙️ 2. 환경 변수 파일 설정

### 2.1 `.env` 파일 템플릿 복사하기

```bash
cp .env.example .env
```

### 2.2 필수 환경 변수 수정하기

텍스트 편집기로 `.env` 파일을 열고 다음 값을 설정합니다:

```env
PORT=3080
NEXTAUTH_ENABLED=false
UID=1000
GID=1000
MEILI_MASTER_KEY=your_meili_key_here

```

> 📌 `MEILI_MASTER_KEY`는 아무 문자열이나 사용할 수 있습니다. 예: `mysecretkey`
> 📌 키 값은 librechat.yaml 생성하여 따로 관리할 수 있습니다. [.env 파일에 CONFIG_PATH=/app/librechat.yaml 값 설정 필요]

---

## 🐳 3. Docker Compose로 빌드 및 실행

LibreChat 저장소 루트 디렉토리에서 다음 명령어를 실행합니다:

```bash
docker compose up --build
```

### 참고사항:

* 위 명령은 이미지들을 빌드하고 `docker-compose.yml`에 정의된 서비스들을 실행합니다.
* 첫 실행 시 종속성 설치와 빌드로 인해 시간이 다소 걸릴 수 있습니다.
* 완료되면 아래 주소에서 웹 서비스를 확인할 수 있습니다:

```
http://localhost:3080
```

---

## 📂 4. Docker 컨테이너 관리하기

컨테이너가 실행되면 Docker Desktop에서 해당 컨테이너를 확인할 수 있습니다. 관리 방법:

* **Docker CLI 사용 시**:
  * 컨테이너 정지: `docker compose down`
  * 재시작: `docker compose up -d`

* **Docker Desktop GUI 사용 시**:
  * 로그 보기
  * 시작/정지 버튼 제공

---

## ❓ 선택적 설정 항목 설명

### `MEILI_NO_SYNC`

```env
# MEILI_NO_SYNC=true
```

> 선택 사항: 멀티 노드 환경에서 한 개의 인스턴스만 인덱스 동기화를 하도록 할 때 사용합니다.

로컬 테스트 환경에서는 비활성화하거나 주석 처리된 상태로 두어도 괜찮습니다.

---

## ✅ 최종 점검 목록

*

---

## 📌 추가 팁

* 컨테이너 실행 후 Git Bash는 종료해도 됩니다. Docker Desktop 또는 CLI로 계속 조작 가능.
* 실시간 로그 보기:

```bash
docker compose logs -f
```

* 변경 후 다시 빌드하기:

```bash
docker compose down
docker compose up --build
```

---

## 🧹 실행 중지 및 정리

```bash
docker compose down
```

이 명령은 모든 컨테이너를 정지하고 삭제합니다.

---

## 🙌 설치 완료!

LibreChat이 이제 로컬에서 실행 중입니다. 아래 주소에서 웹 인터페이스를 확인해보세요:

```
http://localhost:3080
```

`.env` 파일을 수정하고 `docker compose`를 다시 실행하여 인증이나 기타 기능을 활성화할 수 있습니다.



---
## LibreChat Ollam
```
https://www.librechat.ai/blog/2024-03-02_ollama
```
> 해당 사이트에서 Ollama를 사용하여 로컬에서 AI 모델을 실행합니다.


```bash
docker exec -it ollama /bin/bash
```
Ollama 명령에 엑세스




<!-- 노트 -->
.env
> 아래 두 값 주석 풀어주면 왜인지 로컬 서버 실행이 안됨
UID=1000
GID=1000


> 해당 key 자체가 선언되지 않으면 해당 리스트 생기지 않음
> 그말은 선언이 되면 리스트가 생긴다는 말일까?! 도전!
플랫폼_KEY=user_provided




<!-- 뭔가 될것 같음 -->
cd LibreChat
docker compose up -d

docker ps
확인

cd client
npm run dev


<!-- env -->
#========== 추가사항 ==========#
REDIS_URL=redis://redis:6379
MEILISEARCH_HOST=http://meilisearch:7700
VITE_APP_PORT=3080

NEXTAUTH_ENABLED=false

GOOGLE_KEY=키 넣으셈
GOOGLE_MODELS=gemini-2.5-flash-preview-04-17

CONFIG_PATH=/app/librechat.yaml






<!-- docker-compose.override.yml -->
version: '3.4'
 
services:
  api:
    volumes:
      - type: bind
        source: ./librechat.yaml
        target: /app/librechat.yaml





<!-- librechat.yaml -->
version: 1.0.8
 
cache: true
 
interface:
  # Privacy policy settings
  privacyPolicy:
    externalUrl: 'https://librechat.ai/privacy-policy'
    openNewTab: true
 
  # Terms of service
  termsOfService:
    externalUrl: 'https://librechat.ai/tos'
    openNewTab: true
 
registration:
  socialLogins: ["discord", "facebook", "github", "google", "openid"]
 
endpoints:
  custom:

    - name: "Your Endpoint"
      apiKey: "user_provided"  # No need for ${} syntax here
      
    # Ollama
    - name: "Ollama"
      apiKey: "ollama"
      # use 'host.docker.internal' instead of localhost if running LibreChat in a docker container
      baseURL: "http://localhost:11434/v1/" 
      models:
      default: [
        "llama2",
        "mistral",
        "codellama",
        "dolphin-mixtral",
        "mistral-openorca"
        ]
      # fetching list of models is supported but the `name` field must start
      # with `ollama` (case-insensitive), as it does in this example.
      fetch: true
      titleConvo: true
      titleModel: "current_model"
      summarize: false
      summaryModel: "current_model"
      forcePrompt: false
      modelDisplayLabel: "Ollama"
