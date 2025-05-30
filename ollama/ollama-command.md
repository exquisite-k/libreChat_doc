## Ollama CLI 명령어 상세 설명

### *ollama* **`serve`**
```bash
ollama serve
```
* 기능: Ollama의 로컬 서버를 시작합니다.
* 설명: 이 명령은 일반적으로 백그라운드에서 Ollama의 HTTP API 서버를 실행시킵니다. 대부분의 경우 ollama run 같은 명령을 실행하면 자동으로 내부적으로 serve가 실행되므로 직접 사용할 일은 적지만, 서버를 명시적으로 구동하고자 할 때 유용합니다.
* 기본 포트: http://localhost:11434

---

### *ollama* **`create`**
```bash
ollama create <model-name> -f <Modelfile>

# 예시
ollama create llama2-custom -f Modelfile
```
* 기능: 사용자 정의 모델을 생성합니다.
* 설명: Modelfile을 기반으로 기존 모델을 확장하거나 커스터마이징할 수 있습니다. 시스템 메시지, 프롬프트 포맷, 추가 데이터 등을 설정 가능하며, 생성된 모델은 Ollama 로컬 모델 저장소에 등록됩니다.

---

### *ollama* **`show`**
```bash
ollama show <model-name>

# 예시
ollama show llama2-custom
```
* 기능: 특정 모델의 정보를 보여줍니다.
* 설명: 모델 이름, 기반 모델, 시스템 메시지, 템플릿, 생성 일시 등 메타데이터를 확인할 수 있습니다.

---

### *ollama* **`run`**
```bash
ollama run <model-name>

# 예시
ollama run llama2
```
* 기능: 지정한 모델을 실행합니다.
* 설명: 해당 모델을 인터랙티브 셸 또는 API를 통해 사용할 수 있도록 구동합니다. 이 명령어는 serve를 내부적으로 호출하며, 모델이 다운로드되어 있지 않다면 자동으로 다운로드합니다.

---

### *ollama* **`stop`**
```bash
ollama stop <model-name>
```
* 기능: 실행 중인 모델을 중지합니다.
* 설명: run 명령으로 활성화된 세션을 종료합니다. 실행 중인 리소스를 해제할 수 있습니다.

---

### *ollama* **`pull`**
```bash
ollama pull <model-name>

# 예시
ollama pull mistral
```
* 기능: Ollama 모델 레지스트리에서 모델을 다운로드합니다.
* 설명: 모델이 로컬에 없다면 사용 전에 반드시 pull이 필요합니다.

---

### *ollama* **`push`**
```bash
ollama push <model-name>

# 예시
ollama push mistral
```
* 기능: 로컬에서 만든 모델을 Ollama 레지스트리에 업로드합니다.
* 설명: Ollama 계정에 로그인되어 있어야 하며, 공개 또는 비공개 저장소에 배포할 수 있습니다.

---

### *ollama* **`list`**
```bash
ollama list
```
* 기능: 현재 로컬에 설치된 모델 목록을 출력합니다.
* 설명: 모델 이름, 크기, 생성일, 커스텀 여부 등을 확인할 수 있습니다.(Nama, ID, Size, Modified)

---

### *ollama* **`ps`**
```bash
ollama ps
```
* 기능: 현재 실행 중인 모델 세션들을 나열합니다.
* 설명: 여러 모델이 동시에 실행되고 있을 경우, 각각의 상태를 확인할 수 있습니다.

---

### *ollama* **`cp`**
```bash
ollama cp <source> <target>

# 예시
ollama cp llama2 llama2-backup
```
* 기능: 모델을 복사하여 새로운 이름으로 저장합니다.
* 설명: 기존 모델을 백업하거나 다른 이름의 변형을 만들고 싶을 때 유용합니다.

---

### *ollama* **`rm`**
```bash
ollama rm <model-name>

# 예시
ollama rm mistral
```
* 기능: 로컬에 저장된 모델을 삭제합니다.
* 설명: 더 이상 사용하지 않는 모델을 정리할 때 유용합니다. 저장 공간을 확보할 수 있습니다.

---

### *ollama* **`help`**
```bash
ollama help
ollama -h
ollama --help

# 서브 명령어의 도움말
# 예시
ollama help create
ollama help list
```
* 기능: Ollama CLI 명령어에 대한 도움말을 출력합니다.
* 설명: 각 명령어에 대한 사용법을 CLI에서 바로 확인할 수 있습니다.
> ---
> * `ollama` 만 입력해도 도움말이 출력됩니다.
> ---

---

### 🧩 추가 팁
> ---
> * 모든 명령어는 --help 옵션으로 더 많은 정보를 볼 수 있습니다.
> * Ollama는 백그라운드 실행에 최적화되어 있어 API 서버처럼 작동합니다.
> * ollama run은 모델을 즉시 실행하여 대화형 환경을 제공합니다.
> ---
