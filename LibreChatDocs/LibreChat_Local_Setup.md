# 🧰 LibreChat 설치 가이드(K)
Docker를 사용하여 로컬 환경에서 LibreChat을 설치하고 실행하는 전체 과정을 자세히 설명합니다.\
본 가이드는 Windows 환경에서 Git Bash를 사용하는 것을 기준으로 작성되었습니다.

> ---
>   **💡 사전 준비 사항**
>   *( 진행하기 전에 설치되어 있어야 합니다. )*
>   * [Git](https://git-scm.com/downloads)
>   * [Docker Desktop](https://www.docker.com/products/docker-desktop/)\
>   (Docker Engine이 실행 중이어야 합니다.)
> ---

## 📦 1. LibreChat 저장소 클론
```bash
git clone https://github.com/danny-avila/LibreChat.git
cd LibreChat
```
> ---
>   * git clone을 통하여 설치가 완료 된 후, `cd LibreChat` 명령어를 통하여 해당 폴더로 이동 후 진행합니다.
>   * 현재 `dev` 브랜치는 존재하지 않으므로 `main` 브랜치를 기준으로 작업합니다.
> ---



## ⚙️ 2. 환경 변수 파일 설정

### 2.1 `.env` 파일 템플릿 복사하기
```bash
cp .env.example .env
```
> ---
>   * `.env` 파일은 LibreChat 폴더에 존재하여야 합니다. (`LibreChat` > `.env`)
> ---

### 2.2 필수 환경 변수 수정하기
텍스트 편집기로 `.env` 파일을 열고 다음 값을 설정합니다.

\
***[확인]***
```.env
HOST=localhost                                    <- 사용할 호스트
PORT=3080                                         <- 포트 변호
MONGO_URI=mongodb://127.0.0.1:27017/LibreChat     <- 
DOMAIN_CLIENT=http://localhost:3080               <- 
DOMAIN_SERVER=http://localhost:3080               <- 
```
> ---
>   * `UID=1000`, `GID=1000`는 주석처리가 되어있어야 합니다.
>     * `UID (User ID)` : 운영체제에서 사용자 계정을 식별하는 숫자입니다.(사용자 계정 이름 대신 숫자로 표현)
>     * `GID (Group ID)` : 그룹을 식별하는 숫자입니다.(여러 사용자가 속한 그룹 번호의 개념)
>   * 활성화가 되어있으면 실행이 안됩니다.
> ---

\
***[제거(선택)]***
```.env
모델명_API_KEY=user_provided
모델명_KEY=user_provided
모델_MODELS=모델명
```
> ---
>   * 해당 키에 대한 값이 활성화 되어 있으면 해당 모델에 대한 리스트가 활성화 됩니다.
>   * 키가 존재 한다면 키 값이 적용해주면 됩니다.
>   * `예)` GOOGLE_KEY=키값이 들어갑니다 ,ASSISTANTS_API_KEY=user_provided, OPENAI_API_KEY=user_provided 등
>   * 활성화 할 모델을 선택할 수 있다.
>   * `예)` GOOGLE_MODELS=gemini-2.5-flash-preview-04-17 등
> ---

\
***[추가]***
```.env
REDIS_URL=redis://redis:6379                      <- 
MEILISEARCH_HOST=http://meilisearch:7700          <- 
VITE_APP_PORT=3080                                <- 
NEXTAUTH_ENABLED=false                            <- 로그인 기능 사용 여부
```
> ---
>   * 각각 frontend, backend를 위한 값이라 생각(확인해봐야함)
> ---

\
***[수정]***
```.env
ENDPOINTS=사용모델명,사용모델명                   <- 노출할 엔드포인트 설정
ALLOW_REGISTRATION=false                          <- 타인 계정 생성 여부
```
> ---
>   * ENDPOINTS는 openAI,assistants,azureOpenAI,google,gptPlugins,anthropic로 기본 셋팅되어있습니다.(주석처리)
>   * ALLOW_REGISTRATION는 다른 사람도 계정을 생성 유무를 설정할 수 있는 속성입니다.
>   * true = 허가, false = 불가
> ---


## 🐳 3. Docker Compose로 빌드 및 실행
LibreChat 저장소 루트 디렉토리에서 다음 명령어를 실행합니다.
```bash
docker compose up
docker compose up -d
docker compose up --build
```
* docker compose up
    - 빠르게 서비스를 실행하려고만 할 때, 코드나 환경 설정이 변경이 없는 경우 실행 할 때 사용
    - 변경된 게 없으면 빌드 과정은 생략되고 바로 실행
    - `--build` 없이 실행하면 Docker는 기존 이미지와 캐시만을 활용하여 실행
---

* docker compose up -d (--detach)
    - 백그라운드(데몬) 모드로 컨테이너 실행 (로그 표시X)
    - 명령어 실행 후에도 터미널을 계속 사용 가능
    - 서비스를 장시간 실행해야 할 때 사용
    (예: 웹 서버, API 서버 등)
---

* docker compose up --build
    - 컨테이너를 실행하기 전에 이미지를 강제로 다시 빌드
    - `--build`옵션을 주면 `docker-compose.yml`에 정의된 이미지가 이미 존재하더라도, 무조건 다시 빌드합니다.
    - 코드, Dockerfile, 환경 변수 등이 변경되었을 때 최신 상태를 반영하려면 필요합니다.
    - 캐시를 무시하고 새로 빌드하고 싶은 경우 사용
    - 이 명령은 포그라운드 모드로 실행되므로 로그가 터미널에 출력
---

```bash
docker compose up --build -d
```
* docker compose up --build -d
    - 컨테이너를 실행하기 전에 이미지를 강제로 다시 빌드하며, 백그라운드(데몬) 모드로 컨테이너 실행됩니다.
    - `--detach`와 `--build`를 동시에 사용됩니다.
    - 로그를 표시합니다.
---


|   명령어  |   빌드 여부   |   실행 방식   |   특징    |
|----------|--------------|-------------|-----------|
|   `docker compose up`   |   필요 시 자동    |   포그라운드 실행    |   개발 중 간편 실행용    |
|   `docker compose up -d`    |   필요 시 자동    |   백그라운드 실행   |   실서비스 운영 시 주로 사용    |
|   `docker compose up --build`   |   항상 빌드   |   포그라운드 실행    |   변경 사항 반영 확실히 보장   |
|   `docker compose up --build -d`   |   항상 빌드    |   백그라운드 실행    |   실무에서 많이 사용    |



### 참고사항:
* 위 명령은 이미지들을 빌드하고 `docker-compose.yml`에 정의된 서비스들을 실행합니다.
* 첫 실행 시 종속성 설치와 빌드로 인해 시간이 다소 걸릴 수 있습니다.
* [http://localhost:3080](http://localhost:3080) 해당 주소에서 웹 서비스를 확인 가능합니다.
* LibreChat의 고유 서비스만 이용 가능합니다. (등록 AI 모델 이용)



## 📂 4. 개발 환경 설치
`LibreChat` 저장소 루트 디렉토리에서 다음 명령어를 실행합니다.
[( LibreChat 개발환경 설치 가이드 )](https://www.librechat.ai/docs/development/get_started)

```bash
# LibreChat 종속성 설치
npm ci

# FrontEnd 빌드
npm run frontend

# BackEnd 빌드
npm run backend
```


## 🙌 5. 환경 실행
`LibreChat` 저장소 루트 디렉토리에서 환경 실행하는 방법입니다.
```bash
cd LibreChat
npm run frontend:dev
```

`LibreChat` 디렉토리의 `client` 디렉토리에서의 환경 실행하는 방법입니다.
```bash
cd LibreChat
cd client
npm run dev
```

> ---
>   * 서버 실행 후 생성되는 Local 주소로 접속하면 개발자 환경의 서비스 접속이 가능합니다.
>   * [http://localhost:3090](http://localhost:3090)
>   * 기본값으로 위의 주소로 생성되며, 해당 서버가 생성되어있다면 +1 되어 생성됩니다. (예 : 3091)
> ---


## 📌 6. 기타 AI Model 설치
* `LibreChat` 디렉토리의 `librechat.yaml` 파일 생성 후, 아래의 값으로 적용합니다.
* `librechat.example.yaml`파일을 복제하여 사용할 경우, 모든 내용 제거 후, 아래의 내용으로 적용 바랍니다.

> ---
>   * `docker compose up --build`
>   * `librechat.yaml`의 추가 및 수정이 끝났다면 다시 빌드하여 실행 해주세요.
>   * 다시 빌드하지 않으면 정상적으로 작동하지 않을 수 있습니다.
> ---

```librechat.yaml _ Top
version: 1.2.1
cache: true

interface:
  # customWelcome: "LibreChat에 오신 것을 환영합니다! 즐거운 시간 보내세요."
  # 개인정보취급방침 설정
  privacyPolicy:
    externalUrl: 'https://librechat.ai/privacy-policy'
    openNewTab: true

# 소셜 로그인 리스트
registration:
  socialLogins: ['github', 'google', 'discord', 'openid', 'facebook', 'apple']

# 도메인 허용 설정 (테스트용)
# actions:
#   allowedDomains:
#     - "swapi.dev"
#     - "librechat.ai"
#     - "google.com"

endpoints:
  custom:
    # 추가 될 AI Model 값이 들어갑니다.
    # 추가 될 AI Model 값이 들어갑니다.
    # 추가 될 AI Model 값이 들어갑니다.
```
> ---
>   * 들여쓰기에 신경써서 적용하시면 됩니다.
>   * 이후 추가 될 모델은 하단에 속성과 값에 맞추어 적용하면 실행됩니다.
>   * `Docker`로 LibreChat을 `실행`한다면 `baseURL`의 '`localhost`'를 '`host.docker.internal`'로 `변경`해서 사용하세요.
> ---
---

### `#`*`Ollama`*
> ---
>   * [Ollama 설치 가이드 : Docker 설치 방법을 참고](https://www.librechat.ai/blog/2024-03-02_ollama#install-ollama)
> ---
``` librechat.yaml
endpoints:
  custom:

    # Ollama
    - name: 'Ollama'
      apiKey: 'ollama'

      # Docker로 LibreChat을 실행한다면 'localhost'를 'host.docker.internal'로 변경해서 사용하세요.
      baseURL: "http://host.docker.internal:11434/v1/"
      
      models:
        default:
          [
            "llama2",
          ]
        fetch: true
      titleConvo: true
      titleModel: "current_model"
      summarize: false
      summaryModel: "current_model"
      forcePrompt: false
      modelDisplayLabel: 'Ollama'
```
---

### `#`*`Sample`*
``` librechat.yaml
endpoints:
  custom:

    # Sample
    - name: 'Sample'
```
---


