# API

## 엔드포인트

- [완성 생성](#완성-생성)
- [채팅 완성 생성](#채팅-완성-생성)
- [모델 생성](#모델-생성)
- [로컬 모델 목록](#로컬-모델-목록)
- [모델 정보 표시](#모델-정보-표시)
- [모델 복사](#모델-복사)
- [모델 삭제](#모델-삭제)
- [모델 다운로드](#모델-다운로드)
- [모델 업로드](#모델-업로드)
- [임베딩 생성](#임베딩-생성)
- [실행 중인 모델 목록](#실행-중인-모델-목록)
- [버전](#버전)

## 규칙

### 모델 이름

모델 이름은 `model:tag` 형식을 따르며, `model`은 `example/model`과 같은 선택적 네임스페이스를 가질 수 있습니다. 예시로는 `orca-mini:3b-q8_0`과 `llama3:70b`가 있습니다. 태그는 선택사항이며, 제공되지 않으면 기본값은 `latest`입니다. 태그는 특정 버전을 식별하는 데 사용됩니다.

### 지속 시간

모든 지속 시간은 나노초 단위로 반환됩니다.

### 스트리밍 응답

특정 엔드포인트는 JSON 객체로 응답을 스트리밍합니다. 이러한 엔드포인트에 `{"stream": false}`를 제공하여 스트리밍을 비활성화할 수 있습니다.

## 완성 생성

```
POST /api/generate
```

제공된 모델로 주어진 프롬프트에 대한 응답을 생성합니다. 이는 스트리밍 엔드포인트이므로 일련의 응답이 있을 것입니다. 최종 응답 객체에는 요청의 통계 및 추가 데이터가 포함됩니다.

### 매개변수

- `model`: (필수) [모델 이름](#모델-이름)
- `prompt`: 응답을 생성할 프롬프트
- `suffix`: 모델 응답 이후의 텍스트
- `images`: (선택사항) base64로 인코딩된 이미지 목록 (`llava`와 같은 멀티모달 모델용)
- `think`: (사고하는 모델용) 모델이 응답하기 전에 사고해야 하는가?

고급 매개변수 (선택사항):

- `format`: 응답을 반환할 형식. 형식은 `json` 또는 JSON 스키마가 될 수 있습니다
- `options`: `temperature`와 같은 [Modelfile](./modelfile.md#valid-parameters-and-values) 문서에 나열된 추가 모델 매개변수
- `system`: 시스템 메시지 (`Modelfile`에 정의된 것을 재정의)
- `template`: 사용할 프롬프트 템플릿 (`Modelfile`에 정의된 것을 재정의)
- `stream`: `false`이면 응답이 객체 스트림이 아닌 단일 응답 객체로 반환됩니다
- `raw`: `true`이면 프롬프트에 형식이 적용되지 않습니다. API에 대한 요청에서 완전한 템플릿 프롬프트를 지정하는 경우 `raw` 매개변수를 사용할 수 있습니다
- `keep_alive`: 요청 후 모델이 메모리에 로드된 상태로 유지되는 시간을 제어합니다 (기본값: `5m`)
- `context` (더 이상 사용되지 않음): `/generate`에 대한 이전 요청에서 반환된 컨텍스트 매개변수로, 짧은 대화 메모리를 유지하는 데 사용할 수 있습니다

#### 구조화된 출력

`format` 매개변수에 JSON 스키마를 제공하여 구조화된 출력이 지원됩니다. 모델은 스키마와 일치하는 응답을 생성합니다. 아래의 [구조화된 출력](#request-structured-outputs) 예제를 참조하세요.

#### JSON 모드

`format` 매개변수를 `json`으로 설정하여 JSON 모드를 활성화합니다. 이렇게 하면 응답이 유효한 JSON 객체로 구조화됩니다. 아래의 JSON 모드 [예제](#request-json-mode)를 참조하세요.

> [!IMPORTANT]
> `prompt`에서 모델에게 JSON을 사용하도록 지시하는 것이 중요합니다. 그렇지 않으면 모델이 많은 양의 공백을 생성할 수 있습니다. 

### 예제

#### 생성 요청 (스트리밍)

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?"
}'
```

##### 응답

JSON 객체의 스트림이 반환됩니다:

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T08:52:19.385406455-07:00",
  "response": "The",
  "done": false
}
```

스트림의 최종 응답에는 생성에 대한 추가 데이터도 포함됩니다:

- `total_duration`: 응답 생성에 소요된 시간
- `load_duration`: 모델 로딩에 소요된 시간(나노초)
- `prompt_eval_count`: 프롬프트의 토큰 수
- `prompt_eval_duration`: 프롬프트 평가에 소요된 시간(나노초)
- `eval_count`: 응답의 토큰 수
- `eval_duration`: 응답 생성에 소요된 시간(나노초)
- `context`: 이 응답에 사용된 대화의 인코딩으로, 다음 요청에서 대화 메모리를 유지하기 위해 전송할 수 있습니다
- `response`: 응답이 스트리밍된 경우 비어 있고, 스트리밍되지 않은 경우 전체 응답을 포함합니다

초당 토큰(token/s)으로 응답 생성 속도를 계산하려면 `eval_count` / `eval_duration` * `10^9`를 계산하세요.

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T19:22:45.499127Z",
  "response": "",
  "done": true,
  "context": [1, 2, 3],
  "total_duration": 10706818083,
  "load_duration": 6338219291,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 130079000,
  "eval_count": 259,
  "eval_duration": 4232710000
}
```

#### 요청 (스트리밍 없음)

##### 요청

스트리밍이 꺼져 있을 때 한 번의 응답으로 답변을 받을 수 있습니다.

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```

##### 응답

`stream`이 `false`로 설정되면 응답은 단일 JSON 객체가 됩니다:

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T19:22:45.499127Z",
  "response": "The sky is blue because it is the color of the sky.",
  "done": true,
  "context": [1, 2, 3],
  "total_duration": 5043500667,
  "load_duration": 5025959,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 325953000,
  "eval_count": 290,
  "eval_duration": 4709213000
}
```

#### 요청 (접미사 포함)

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "codellama:code",
  "prompt": "def compute_gcd(a, b):",
  "suffix": "    return result",
  "options": {
    "temperature": 0
  },
  "stream": false
}'
```

##### 응답

```json5
{
  "model": "codellama:code",
  "created_at": "2024-07-22T20:47:51.147561Z",
  "response": "\n  if a == 0:\n    return b\n  else:\n    return compute_gcd(b % a, a)\n\ndef compute_lcm(a, b):\n  result = (a * b) / compute_gcd(a, b)\n",
  "done": true,
  "done_reason": "stop",
  "context": [...],
  "total_duration": 1162761250,
  "load_duration": 6683708,
  "prompt_eval_count": 17,
  "prompt_eval_duration": 201222000,
  "eval_count": 63,
  "eval_duration": 953997000
}
```

#### 요청 (구조화된 출력)

##### 요청

```shell
curl -X POST http://localhost:11434/api/generate -H "Content-Type: application/json" -d '{
  "model": "llama3.1:8b",
  "prompt": "Ollama is 22 years old and is busy saving the world. Respond using JSON",
  "stream": false,
  "format": {
    "type": "object",
    "properties": {
      "age": {
        "type": "integer"
      },
      "available": {
        "type": "boolean"
      }
    },
    "required": [
      "age",
      "available"
    ]
  }
}'
```

##### 응답

```json
{
  "model": "llama3.1:8b",
  "created_at": "2024-12-06T00:48:09.983619Z",
  "response": "{\n  \"age\": 22,\n  \"available\": true\n}",
  "done": true,
  "done_reason": "stop",
  "context": [1, 2, 3],
  "total_duration": 1075509083,
  "load_duration": 567678166,
  "prompt_eval_count": 28,
  "prompt_eval_duration": 236000000,
  "eval_count": 16,
  "eval_duration": 269000000
}
```

#### 요청 (JSON 모드)

> [!IMPORTANT]
> `format`이 `json`으로 설정되면 출력은 항상 잘 형성된 JSON 객체가 됩니다. 모델에게 JSON으로 응답하도록 지시하는 것도 중요합니다.

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "What color is the sky at different times of the day? Respond using JSON",
  "format": "json",
  "stream": false
}'
```

##### 응답

```json
{
  "model": "llama3.2",
  "created_at": "2023-11-09T21:07:55.186497Z",
  "response": "{\n\"morning\": {\n\"color\": \"blue\"\n},\n\"noon\": {\n\"color\": \"blue-gray\"\n},\n\"afternoon\": {\n\"color\": \"warm gray\"\n},\n\"evening\": {\n\"color\": \"orange\"\n}\n}\n",
  "done": true,
  "context": [1, 2, 3],
  "total_duration": 4648158584,
  "load_duration": 4071084,
  "prompt_eval_count": 36,
  "prompt_eval_duration": 439038000,
  "eval_count": 180,
  "eval_duration": 4196918000
}
```

`response`의 값은 다음과 유사한 JSON을 포함하는 문자열이 됩니다:

```json
{
  "morning": {
    "color": "blue"
  },
  "noon": {
    "color": "blue-gray"
  },
  "afternoon": {
    "color": "warm gray"
  },
  "evening": {
    "color": "orange"
  }
}
```

#### 요청 (이미지 포함)

`llava`나 `bakllava`와 같은 멀티모달 모델에 이미지를 제출하려면 base64로 인코딩된 `images` 목록을 제공하세요:

#### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llava",
  "prompt":"What is in this picture?",
  "stream": false,
  "images": ["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"]
    }
  ]
}'
```

##### 응답

```json
{
  "model": "llava",
  "created_at": "2023-11-03T15:36:02.583064Z",
  "response": "A happy cartoon character, which is cute and cheerful.",
  "done": true,
  "context": [1, 2, 3],
  "total_duration": 2938432250,
  "load_duration": 2559292,
  "prompt_eval_count": 1,
  "prompt_eval_duration": 2195557000,
  "eval_count": 44,
  "eval_duration": 736432000
}
```

#### 요청 (Raw 모드)

경우에 따라 템플릿 시스템을 우회하고 전체 프롬프트를 제공할 수 있습니다. 이 경우 `raw` 매개변수를 사용하여 템플릿을 비활성화할 수 있습니다. 또한 raw 모드는 컨텍스트를 반환하지 않습니다.

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "[INST] why is the sky blue? [/INST]",
  "raw": true,
  "stream": false
}'
```

#### 요청 (재현 가능한 출력)

재현 가능한 출력을 위해 `seed`를 숫자로 설정하세요:

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "Why is the sky blue?",
  "options": {
    "seed": 123
  }
}'
```

##### 응답

```json
{
  "model": "mistral",
  "created_at": "2023-11-03T15:36:02.583064Z",
  "response": " The sky appears blue because of a phenomenon called Rayleigh scattering.",
  "done": true,
  "total_duration": 8493852375,
  "load_duration": 6589624375,
  "prompt_eval_count": 14,
  "prompt_eval_duration": 119039000,
  "eval_count": 110,
  "eval_duration": 1779061000
}
``` 

#### 생성 요청 (옵션 포함)

런타임에 Modelfile이 아닌 모델에 대한 사용자 정의 옵션을 설정하려면 `options` 매개변수를 사용할 수 있습니다. 이 예제는 사용 가능한 모든 옵션을 설정하지만, 개별적으로 설정하고 재정의하지 않으려는 것들은 생략할 수 있습니다.

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?",
  "stream": false,
  "options": {
    "num_keep": 5,
    "seed": 42,
    "num_predict": 100,
    "top_k": 20,
    "top_p": 0.9,
    "min_p": 0.0,
    "typical_p": 0.7,
    "repeat_last_n": 33,
    "temperature": 0.8,
    "repeat_penalty": 1.2,
    "presence_penalty": 1.5,
    "frequency_penalty": 1.0,
    "penalize_newline": true,
    "stop": ["\n", "user:"],
    "numa": false,
    "num_ctx": 1024,
    "num_batch": 2,
    "num_gpu": 1,
    "main_gpu": 0,
    "use_mmap": true,
    "num_thread": 8
  }
}'
```

##### 응답

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T19:22:45.499127Z",
  "response": "The sky is blue because it is the color of the sky.",
  "done": true,
  "context": [1, 2, 3],
  "total_duration": 4935886791,
  "load_duration": 534986708,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 107345000,
  "eval_count": 237,
  "eval_duration": 4289432000
}
```

#### 모델 로드

빈 프롬프트가 제공되면 모델이 메모리에 로드됩니다.

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2"
}'
```

##### 응답

단일 JSON 객체가 반환됩니다:

```json
{
  "model": "llama3.2",
  "created_at": "2023-12-18T19:52:07.071755Z",
  "response": "",
  "done": true
}
```

#### 모델 언로드

빈 프롬프트가 제공되고 `keep_alive` 매개변수가 `0`으로 설정되면 모델이 메모리에서 언로드됩니다.

##### 요청

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "keep_alive": 0
}'
```

##### 응답

단일 JSON 객체가 반환됩니다:

```json
{
  "model": "llama3.2",
  "created_at": "2024-09-12T03:54:03.516566Z",
  "response": "",
  "done": true,
  "done_reason": "unload"
}
```

## 채팅 완성 생성

```
POST /api/chat
```

제공된 모델로 채팅의 다음 메시지를 생성합니다. 이는 스트리밍 엔드포인트이므로 일련의 응답이 있을 것입니다. `"stream": false`를 사용하여 스트리밍을 비활성화할 수 있습니다. 최종 응답 객체에는 요청의 통계 및 추가 데이터가 포함됩니다.

### 매개변수

- `model`: (필수) [모델 이름](#모델-이름)
- `messages`: 채팅의 메시지로, 채팅 메모리를 유지하는 데 사용할 수 있습니다
- `tools`: 모델이 지원하는 경우 사용할 JSON 형식의 도구 목록
- `think`: (사고하는 모델용) 모델이 응답하기 전에 사고해야 하는가?

`message` 객체는 다음 필드를 가집니다:

- `role`: 메시지의 역할, `system`, `user`, `assistant`, 또는 `tool` 중 하나
- `content`: 메시지의 내용
- `thinking`: (사고하는 모델용) 모델의 사고 과정
- `images` (선택사항): 메시지에 포함할 이미지 목록 (`llava`와 같은 멀티모달 모델용)
- `tool_calls` (선택사항): 모델이 사용하려는 JSON 형식의 도구 목록

고급 매개변수 (선택사항):

- `format`: 응답을 반환할 형식. 형식은 `json` 또는 JSON 스키마가 될 수 있습니다.
- `options`: `temperature`와 같은 [Modelfile](./modelfile.md#valid-parameters-and-values) 문서에 나열된 추가 모델 매개변수
- `stream`: `false`이면 응답이 객체 스트림이 아닌 단일 응답 객체로 반환됩니다
- `keep_alive`: 요청 후 모델이 메모리에 로드된 상태로 유지되는 시간을 제어합니다 (기본값: `5m`)

### 구조화된 출력

`format` 매개변수에 JSON 스키마를 제공하여 구조화된 출력이 지원됩니다. 모델은 스키마와 일치하는 응답을 생성합니다. 아래의 [채팅 요청 (구조화된 출력)](#chat-request-structured-outputs) 예제를 참조하세요.

### 예제

#### 채팅 요청 (스트리밍)

##### 요청

스트리밍 응답으로 채팅 메시지를 보냅니다.

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {
      "role": "user",
      "content": "why is the sky blue?"
    }
  ]
}'
```

##### 응답

JSON 객체의 스트림이 반환됩니다:

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T08:52:19.385406455-07:00",
  "message": {
    "role": "assistant",
    "content": "The"
  },
  "done": false
}
```

최종 응답:

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T19:22:45.499127Z",
  "message": {
    "role": "assistant",
    "content": ""
  },
  "done": true,
  "total_duration": 4883583458,
  "load_duration": 1334875,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 342546000,
  "eval_count": 282,
  "eval_duration": 4535599000
}
```

#### 채팅 요청 (스트리밍 없음)

##### 요청

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {
      "role": "user",
      "content": "why is the sky blue?"
    }
  ],
  "stream": false
}'
```

##### 응답

```json
{
  "model": "llama3.2",
  "created_at": "2023-12-12T14:13:43.416799Z",
  "message": {
    "role": "assistant",
    "content": "Hello! How are you today?"
  },
  "done": true,
  "total_duration": 5191566416,
  "load_duration": 2154458,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 383809000,
  "eval_count": 298,
  "eval_duration": 4799921000
}
```

#### 채팅 요청 (구조화된 출력)

##### 요청

```shell
curl -X POST http://localhost:11434/api/chat -H "Content-Type: application/json" -d '{
  "model": "llama3.1",
  "messages": [{"role": "user", "content": "Ollama is 22 years old and busy saving the world. Return a JSON object with the age and availability."}],
  "stream": false,
  "format": {
    "type": "object",
    "properties": {
      "age": {
        "type": "integer"
      },
      "available": {
        "type": "boolean"
      }
    },
    "required": [
      "age",
      "available"
    ]
  },
  "options": {
    "temperature": 0
  }
}'
```

##### 응답

```json
{
  "model": "llama3.1",
  "created_at": "2024-12-06T00:46:58.265747Z",
  "message": { "role": "assistant", "content": "{\"age\": 22, \"available\": false}" },
  "done_reason": "stop",
  "done": true,
  "total_duration": 2254970291,
  "load_duration": 574751416,
  "prompt_eval_count": 34,
  "prompt_eval_duration": 1502000000,
  "eval_count": 12,
  "eval_duration": 175000000
}
```

#### 채팅 요청 (기록 포함)

대화 기록과 함께 채팅 메시지를 보냅니다. 멀티샷 또는 연쇄 사고 프롬프팅을 사용하여 대화를 시작하는 데도 동일한 접근 방식을 사용할 수 있습니다.

##### 요청

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {
      "role": "user",
      "content": "why is the sky blue?"
    },
    {
      "role": "assistant",
      "content": "due to rayleigh scattering."
    },
    {
      "role": "user",
      "content": "how is that different than mie scattering?"
    }
  ]
}'
```

##### 응답

JSON 객체의 스트림이 반환됩니다:

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T08:52:19.385406455-07:00",
  "message": {
    "role": "assistant",
    "content": "The"
  },
  "done": false
}
```

최종 응답:

```json
{
  "model": "llama3.2",
  "created_at": "2023-08-04T19:22:45.499127Z",
  "done": true,
  "total_duration": 8113331500,
  "load_duration": 6396458,
  "prompt_eval_count": 61,
  "prompt_eval_duration": 398801000,
  "eval_count": 468,
  "eval_duration": 7701267000
}
``` 

#### 채팅 요청 (이미지 포함)

##### 요청

이미지와 함께 채팅 메시지를 보냅니다. 이미지는 개별 이미지가 Base64로 인코딩된 배열로 제공되어야 합니다.

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llava",
  "messages": [
    {
      "role": "user",
      "content": "what is in this image?",
      "images": ["iVBORw0KGgoAAAANSUhEUgAAAG0AAABmCAYAAADBPx+VAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAA3VSURBVHgB7Z27r0zdG8fX743i1bi1ikMoFMQloXRpKFFIqI7LH4BEQ+NWIkjQuSWCRIEoULk0gsK1kCBI0IhrQVT7tz/7zZo888yz1r7MnDl7z5xvsjkzs2fP3uu71nNfa7lkAsm7d++Sffv2JbNmzUqcc8m0adOSzZs3Z+/XES4ZckAWJEGWPiCxjsQNLWmQsWjRIpMseaxcuTKpG/7HP27I8P79e7dq1ars/yL4/v27S0ejqwv+cUOGEGGpKHR37tzJCEpHV9tnT58+dXXCJDdECBE2Ojrqjh071hpNECjx4cMHVycM1Uhbv359B2F79+51586daxN/+pyRkRFXKyRDAqxEp4yMlDDzXG1NPnnyJKkThoK0VFd1ELZu3TrzXKxKfW7dMBQ6bcuWLW2v0VlHjx41z717927ba22U9APcw7Nnz1oGEPeL3m3p2mTAYYnFmMOMXybPPXv2bNIPpFZr1NHn4HMw0KRBjg9NuRw95s8PEcz/6DZELQd/09C9QGq5RsmSRybqkwHGjh07OsJSsYYm3ijPpyHzoiacg35MLdDSIS/O1yM778jOTwYUkKNHWUzUWaOsylE00MyI0fcnOwIdjvtNdW/HZwNLGg+sR1kMepSNJXmIwxBZiG8tDTpEZzKg0GItNsosY8USkxDhD0Rinuiko2gfL/RbiD2LZAjU9zKQJj8RDR0vJBR1/Phx9+PHj9Z7REF4nTZkxzX4LCXHrV271qXkBAPGfP/atWvu/PnzHe4C97F48eIsRLZ9+3a3f/9+87dwP1JxaF7/3r17ba+5l4EcaVo0lj3SBq5kGTJSQmLWMjgYNei2GPT1MuMqGTDEFHzeQSP2wi/jGnkmPJ/nhccs44jvDAxpVcxnq0F6eT8h4ni/iIWpR5lPyA6ETkNXoSukvpJAD3AsXLiwpZs49+fPn5ke4j10TqYvegSfn0OnafC+Tv9ooA/JPkgQysqQNBzagXY55nO/oa1F7qvIPWkRL12WRpMWUvpVDYmxAPehxWSe8ZEXL20sadYIozfmNch4QJPAfeJgW3rNsnzphBKNJM2KKODo1rVOMRYik5ETy3ix4qWNI81qAAirizgMIc+yhTytx0JWZuNI03qsrgWlGtwjoS9XwgUhWGyhUaRZZQNNIEwCiXD16tXcAHUs79co0vSD8rrJCIW98pzvxpAWyyo3HYwqS0+H0BjStClcZJT5coMm6D2LOF8TolGJtK9fvyZpyiC5ePFi9nc/oJU4eiEP0jVoAnHa9wyJycITMP78+eMeP37sXrx44d6+fdt6f82aNdkx1pg9e3Zb5W+RSRE+n+VjksQWifvVaTKFhn5O8my63K8Qabdv33b379/PiAP//vuvW7BggZszZ072/+TJk91YgkafPn166zXB1rQHFvouAWHq9z3SEevSUerqCn2/dDCeta2jxYbr69evk4MHDyY7d+7MjhMnTiTPnz9Pfv/+nfQT2ggpO2dMF8cghuoM7Ygj5iWCqRlGFml0QC/ftGmTmzt3rmsaKDsgBSPh0/8yPeLLBihLkOKJc0jp8H8vUzcxIA1k6QJ/c78tWEyj5P3o4u9+jywNPdJi5rAH9x0KHcl4Hg570eQp3+vHXGyrmEeigzQsQsjavXt38ujRo44LQuDDhw+TW7duRS1HGgMxhNXHgflaNTOsHyKvHK5Ijo2jbFjJBQK9YwFd6RVMzfgRBmEfP37suBBm/p49e1qjEP2mwTViNRo0VJWH1deMXcNK08uUjVUu7s/zRaL+oLNxz1bpANco4npUgX4G2eFbpDFyQoQxojBCpEGSytmOH8qrH5Q9vuzD6ofQylkCUmh8DBAr+q8JCyVNtWQIidKQE9wNtLSQnS4jDSsxNHogzFuQBw4cyM61UKVsjfr3ooBkPSqqQHesUPWVtzi9/vQi1T+rJj7WiTz4Pt/l3LxUkr5P2VYZaZ4URpsE+st/dujQoaBBYokbrz/8TJNQYLSonrPS9kUaSkPeZyj1AWSj+d+VBoy1pIWVNed8P0Pl/ee5HdGRhrHhR5GGN0r4LGZBaj8oFDJitBTJzIZgFcmU0Y8ytWMZMzJOaXUSrUs5RxKnrxmbb5YXO9VGUhtpXldhEUogFr3IzIsvlpmdosVcGVGXFWp2oU9kLFL3dEkSz6NHEY1sjSRdIuDFWEhd8KxFqsRi1uM/nz9/zpxnwlESONdg6dKlbsaMGS4EHFHtjFIDHwKOo46l4TxSuxgDzi+rE2jg+BaFruOX4HXa0Nnf1lwAPufZeF8/r6zD97WK2qFnGjBxTw5qNGPxT+5T/r7/7RawFC3j4vTp09koCxkeHjqbHJqArmH5UrFKKksnxrK7FuRIs8STfBZv+luugXZ2pR/pP9Ois4z+TiMzUUkUjD0iEi1fzX8GmXyuxUBRcaUfykV0YZnlJGKQpOiGB76x5GeWkWWJc3mOrK6S7xdND+W5N6XyaRgtWJFe13GkaZnKOsYqGdOVVVbGupsyA/l7emTLHi7vwTdirNEt0qxnzAvBFcnQF16xh/TMpUuXHDowhlA9vQVraQhkudRdzOnK+04ZSP3DUhVSP61YsaLtd/ks7ZgtPcXqPqEafHkdqa84X6aCeL7YWlv6edGFHb+ZFICPlljHhg0bKuk0CSvVznWsotRu433alNdFrqG45ejoaPCaUkWERpLXjzFL2Rpllp7PJU2a/v7Ab8N05/9t27Z16KUqoFGsxnI9EosS2niSYg9SpU6B4JgTrvVW1flt1sT+0ADIJU2maXzcUTraGCRaL1Wp9rUMk16PMom8QhruxzvZIegJjFU7LLCePfS8uaQdPny4jTTL0dbee5mYokQsXTIWNY46kuMbnt8Kmec+LGWtOVIl9cT1rCB0V8WqkjAsRwta93TbwNYoGKsUSChN44lgBNCoHLHzquYKrU6qZ8lolCIN0Rh6cP0Q3U6I6IXILYOQI513hJaSKAorFpuHXJNfVlpRtmYBk1Su1obZr5dnKAO+L10Hrj3WZW+E3qh6IszE37F6EB+68mGpvKm4eb9bFrlzrok7fvr0Kfv727dvWRmdVTJHw0qiiCUSZ6wCK+7XL/AcsgNyL74DQQ730sv78Su7+t/A36MdY0sW5o40ahslXr58aZ5HtZB8GH64m9EmMZ7FpYw4T6QnrZfgenrhFxaSiSGXtPnz57e9TkNZLvTjeqhr734CNtrK41L40sUQckmj1lGKQ0rC37x544r8eNXRpnVE3ZZY7zXo8NomiO0ZUCj2uHz58rbXoZ6gc0uA+F6ZeKS/jhRDUq8MKrTho9fEkihMmhxtBI1DxKFY9XLpVcSkfoi8JGnToZO5sU5aiDQIW716ddt7ZLYtMQlhECdBGXZZMWldY5BHm5xgAroWj4C0hbYkSc/jBmggIrXJWlZM6pSETsEPGqZOndr2uuuR5rF169a2HoHPdurUKZM4CO1WTPqaDaAd+GFGKdIQkxAn9RuEWcTRyN2KSUgiSgF5aWzPTeA/lN5rZubMmR2bE4SIC4nJoltgAV/dVefZm72AtctUCJU2CMJ327hxY9t7EHbkyJFseq+EJSY16RPo3Dkq1kkr7+q0bNmyDuLQcZBEPYmHVdOBiJyIlrRDq41YPWfXOxUysi5fvtyaj+2BpcnsUV/oSoEMOk2CQGlr4ckhBwaetBhjCwH0ZHtJROPJkyc7UjcYLDjmrH7ADTEBXFfOYmB0k9oYBOjJ8b4aOYSe7QkKcYhFlq3QYLQhSidNmtS2RATwy8YOM3EQJsUjKiaWZ+vZToUQgzhkHXudb/PW5YMHD9yZM2faPsMwoc7RciYJXbGuBqJ1UIGKKLv915jsvgtJxCZDubdXr165mzdvtr1Hz5LONA8jrUwKPqsmVesKa49S3Q4WxmRPUEYdTjgiUcfUwLx589ySJUva3oMkP6IYddq6HMS4o55xBJBUeRjzfa4Zdeg56QZ43LhxoyPo7Lf1kNt7oO8wWAbNwaYjIv5lhyS7kRf96dvm5Jah8vfvX3flyhX35cuX6HfzFHOToS1H4BenCaHvO8pr8iDuwoUL7tevX+b5ZdbBair0xkFIlFDlW4ZknEClsp/TzXyAKVOmmHWFVSbDNw1l1+4f90U6IY/q4V27dpnE9bJ+v87QEydjqx/UamVVPRG+mwkNTYN+9tjkwzEx+atCm/X9WvWtDtAb68Wy9LXa1UmvCDDIpPkyOQ5ZwSzJ4jMrvFcr0rSjOUh+GcT4LSg5ugkW1Io0/SCDQBojh0hPlaJdah+tkVYrnTZowP8iq1F1TgMBBauufyB33x1v+NWFYmT5KmppgHC+NkAgbmRkpD3yn9QIseXymoTQFGQmIOKTxiZIWpvAatenVqRVXf2nTrAWMsPnKrMZHz6bJq5jvce6QK8J1cQNgKxlJapMPdZSR64/UivS9NztpkVEdKcrs5alhhWP9NeqlfWopzhZScI6QxseegZRGeg5a8C3Re1Mfl1ScP36ddcUaMuv24iOJtz7sbUjTS4qBvKmstYJoUauiuD3k5qhyr7QdUHMeCgLa1Ear9NquemdXgmum4fvJ6w1lqsuDhNrg1qSpleJK7K3TF0Q2jSd94uSZ60kK1e3qyVpQK6PVWXp2/FC3mp6jBhKKOiY2h3gtUV64TWM6wDETRPLDfSakXmH3w8g9Jlug8ZtTt4kVF0kLUYYmCCtD/DrQ5YhMGbA9L3ucdjh0y8kOHW5gU/VEEmJTcL4Pz/f7mgoAbYkAAAAAElFTkSuQmCC"]
    }
  ]
}'
```

##### 응답

```json
{
  "model": "llava",
  "created_at": "2023-12-13T22:42:50.203334Z",
  "message": {
    "role": "assistant",
    "content": " The image features a cute, little pig with an angry facial expression. It's wearing a heart on its shirt and is waving in the air. This scene appears to be part of a drawing or sketching project.",
    "images": null
  },
  "done": true,
  "total_duration": 1668506709,
  "load_duration": 1986209,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 359682000,
  "eval_count": 83,
  "eval_duration": 1303285000
}
```

#### 채팅 요청 (재현 가능한 출력)

##### 요청

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {
      "role": "user",
      "content": "Hello!"
    }
  ],
  "options": {
    "seed": 101,
    "temperature": 0
  }
}'
```

##### 응답

```json
{
  "model": "llama3.2",
  "created_at": "2023-12-12T14:13:43.416799Z",
  "message": {
    "role": "assistant",
    "content": "Hello! How are you today?"
  },
  "done": true,
  "total_duration": 5191566416,
  "load_duration": 2154458,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 383809000,
  "eval_count": 298,
  "eval_duration": 4799921000
}
```

#### 채팅 요청 (도구 사용)

##### 요청

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {
      "role": "user",
      "content": "What is the weather today in Paris?"
    }
  ],
  "stream": false,
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_current_weather",
        "description": "Get the current weather for a location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The location to get the weather for, e.g. San Francisco, CA"
            },
            "format": {
              "type": "string",
              "description": "The format to return the weather in, e.g. 'celsius' or 'fahrenheit'",
              "enum": ["celsius", "fahrenheit"]
            }
          },
          "required": ["location", "format"]
        }
      }
    }
  ]
}'
```

##### 응답

```json
{
  "model": "llama3.2",
  "created_at": "2024-07-22T20:33:28.123648Z",
  "message": {
    "role": "assistant",
    "content": "",
    "tool_calls": [
      {
        "function": {
          "name": "get_current_weather",
          "arguments": {
            "format": "celsius",
            "location": "Paris, FR"
          }
        }
      }
    ]
  },
  "done_reason": "stop",
  "done": true,
  "total_duration": 885095291,
  "load_duration": 3753500,
  "prompt_eval_count": 122,
  "prompt_eval_duration": 328493000,
  "eval_count": 33,
  "eval_duration": 552222000
}
```

#### 모델 로드

messages 배열이 비어 있으면 모델이 메모리에 로드됩니다.

##### 요청

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": []
}'
```

##### 응답

```json
{
  "model": "llama3.2",
  "created_at":"2024-09-12T21:17:29.110811Z",
  "message": {
    "role": "assistant",
    "content": ""
  },
  "done_reason": "load",
  "done": true
}
```

#### 모델 언로드

messages 배열이 비어 있고 `keep_alive` 매개변수가 `0`으로 설정되면 모델이 메모리에서 언로드됩니다.

##### 요청

```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [],
  "keep_alive": 0
}'
```

##### 응답

단일 JSON 객체가 반환됩니다:

```json
{
  "model": "llama3.2",
  "created_at":"2024-09-12T21:33:17.547535Z",
  "message": {
    "role": "assistant",
    "content": ""
  },
  "done_reason": "unload",
  "done": true
}
``` 

## 모델 생성

```
POST /api/create
```

다음으로부터 모델을 생성합니다:
 * 다른 모델;
 * safetensors 디렉토리; 또는
 * GGUF 파일.

safetensors 디렉토리나 GGUF 파일에서 모델을 생성하는 경우, 각 파일에 대해 [블롭 생성](#create-a-blob)을 먼저 수행한 다음 `files` 필드에서 각 블롭과 연관된 파일 이름과 SHA256 다이제스트를 사용해야 합니다.

### 매개변수

- `model`: 생성할 모델의 이름
- `from`: (선택사항) 새 모델을 생성할 기존 모델의 이름
- `files`: (선택사항) 모델을 생성할 블롭의 SHA256 다이제스트에 대한 파일 이름 딕셔너리
- `adapters`: (선택사항) LORA 어댑터용 블롭의 SHA256 다이제스트에 대한 파일 이름 딕셔너리
- `template`: (선택사항) 모델의 프롬프트 템플릿
- `license`: (선택사항) 모델의 라이선스를 포함하는 문자열 또는 문자열 목록
- `system`: (선택사항) 모델의 시스템 프롬프트를 포함하는 문자열
- `parameters`: (선택사항) 모델 매개변수 딕셔너리 (매개변수 목록은 [Modelfile](./modelfile.md#valid-parameters-and-values) 참조)
- `messages`: (선택사항) 대화를 생성하는 데 사용되는 메시지 객체 목록
- `stream`: (선택사항) `false`이면 응답이 객체 스트림이 아닌 단일 응답 객체로 반환됩니다
- `quantize` (선택사항): 비양자화(예: float16) 모델을 양자화합니다

#### 양자화 유형

| 유형 | 권장 |
| --- | :-: |
| q4_K_M | * |
| q4_K_S | |
| q8_0 | * |

### 예제

#### 새 모델 생성

기존 모델에서 새 모델을 생성합니다.

##### 요청

```shell
curl http://localhost:11434/api/create -d '{
  "model": "mario",
  "from": "llama3.2",
  "system": "You are Mario from Super Mario Bros."
}'
```

##### 응답

JSON 객체의 스트림이 반환됩니다:

```json
{"status":"reading model metadata"}
{"status":"creating system layer"}
{"status":"using already created layer sha256:22f7f8ef5f4c791c1b03d7eb414399294764d7cc82c7e94aa81a1feb80a983a2"}
{"status":"using already created layer sha256:8c17c2ebb0ea011be9981cc3922db8ca8fa61e828c5d3f44cb6ae342bf80460b"}
{"status":"using already created layer sha256:7c23fb36d80141c4ab8cdbb61ee4790102ebd2bf7aeff414453177d4f2110e5d"}
{"status":"using already created layer sha256:2e0493f67d0c8c9c68a8aeacdf6a38a2151cb3c4c1d42accf296e19810527988"}
{"status":"using already created layer sha256:2759286baa875dc22de5394b4a925701b1896a7e3f8e53275c36f75a877a82c9"}
{"status":"writing layer sha256:df30045fe90f0d750db82a058109cecd6d4de9c90a3d75b19c09e5f64580bb42"}
{"status":"writing layer sha256:f18a68eb09bf925bb1b669490407c1b1251c5db98dc4d3d81f3088498ea55690"}
{"status":"writing manifest"}
{"status":"success"}
```

#### 모델 양자화

비양자화 모델을 양자화합니다.

##### 요청

```shell
curl http://localhost:11434/api/create -d '{
  "model": "llama3.2:quantized",
  "from": "llama3.2:3b-instruct-fp16",
  "quantize": "q4_K_M"
}'
```

##### 응답

JSON 객체의 스트림이 반환됩니다:

```json
{"status":"quantizing F16 model to Q4_K_M","digest":"0","total":6433687776,"completed":12302}
{"status":"quantizing F16 model to Q4_K_M","digest":"0","total":6433687776,"completed":6433687552}
{"status":"verifying conversion"}
{"status":"creating new layer sha256:fb7f4f211b89c6c4928ff4ddb73db9f9c0cfca3e000c3e40d6cf27ddc6ca72eb"}
{"status":"using existing layer sha256:966de95ca8a62200913e3f8bfbf84c8494536f1b94b49166851e76644e966396"}
{"status":"using existing layer sha256:fcc5a6bec9daf9b561a68827b67ab6088e1dba9d1fa2a50d7bbcc8384e0a265d"}
{"status":"using existing layer sha256:a70ff7e570d97baaf4e62ac6e6ad9975e04caa6d900d3742d37698494479e0cd"}
{"status":"using existing layer sha256:56bb8bd477a519ffa694fc449c2413c6f0e1d3b1c88fa7e3c9d88d3ae49d4dcb"}
{"status":"writing manifest"}
{"status":"success"}
```

#### GGUF에서 모델 생성

GGUF 파일에서 모델을 생성합니다. `files` 매개변수는 사용하려는 GGUF 파일의 파일 이름과 SHA256 다이제스트로 채워져야 합니다. 이 API를 호출하기 전에 [/api/blobs/:digest](#push-a-blob)를 사용하여 GGUF 파일을 서버에 푸시하세요.

##### 요청

```shell
curl http://localhost:11434/api/create -d '{
  "model": "my-gguf-model",
  "files": {
    "test.gguf": "sha256:432f310a77f4650a88d0fd59ecdd7cebed8d684bafea53cbff0473542964f0c3"
  }
}'
```

##### 응답

JSON 객체의 스트림이 반환됩니다:

```json
{"status":"parsing GGUF"}
{"status":"using existing layer sha256:432f310a77f4650a88d0fd59ecdd7cebed8d684bafea53cbff0473542964f0c3"}
{"status":"writing manifest"}
{"status":"success"}
```

#### Safetensors 디렉토리에서 모델 생성

`files` 매개변수는 파일 이름과 각 파일의 SHA256 다이제스트를 포함하는 safetensors 모델의 파일 딕셔너리를 포함해야 합니다. 이 API를 호출하기 전에 [/api/blobs/:digest](#push-a-blob)를 사용하여 각 파일을 서버에 먼저 푸시하세요. Ollama 서버가 재시작될 때까지 파일이 캐시에 남아 있습니다.

##### 요청

```shell
curl http://localhost:11434/api/create -d '{
  "model": "fred",
  "files": {
    "config.json": "sha256:dd3443e529fb2290423a0c65c2d633e67b419d273f170259e27297219828e389",
    "generation_config.json": "sha256:88effbb63300dbbc7390143fbbdd9d9fa50587b37e8bfd16c8c90d4970a74a36",
    "special_tokens_map.json": "sha256:b7455f0e8f00539108837bfa586c4fbf424e31f8717819a6798be74bef813d05",
    "tokenizer.json": "sha256:bbc1904d35169c542dffbe1f7589a5994ec7426d9e5b609d07bab876f32e97ab",
    "tokenizer_config.json": "sha256:24e8a6dc2547164b7002e3125f10b415105644fcf02bf9ad8b674c87b1eaaed6",
    "model.safetensors": "sha256:1ff795ff6a07e6a68085d206fb84417da2f083f68391c2843cd2b8ac6df8538f"
  }
}'
```

##### 응답

JSON 객체의 스트림이 반환됩니다:

```shell
{"status":"converting model"}
{"status":"creating new layer sha256:05ca5b813af4a53d2c2922933936e398958855c44ee534858fcfd830940618b6"}
{"status":"using autodetected template llama3-instruct"}
{"status":"using existing layer sha256:56bb8bd477a519ffa694fc449c2413c6f0e1d3b1c88fa7e3c9d88d3ae49d4dcb"}
{"status":"writing manifest"}
{"status":"success"}
```

## 블롭 존재 여부 확인

```shell
HEAD /api/blobs/:digest
```

모델 생성에 사용되는 파일 블롭(Binary Large Object)이 서버에 존재하는지 확인합니다. 이는 ollama.com이 아닌 Ollama 서버를 확인합니다.

### 쿼리 매개변수

- `digest`: 블롭의 SHA256 다이제스트

### 예제

#### 요청

```shell
curl -I http://localhost:11434/api/blobs/sha256:29fdb92e57cf0827ded04ae6461b5931d01fa595843f55d36f5b275a52087dd2
```

#### 응답

블롭이 존재하면 200 OK를, 존재하지 않으면 404 Not Found를 반환합니다.

## 블롭 푸시

```
POST /api/blobs/:digest
```

Ollama 서버에 파일을 푸시하여 "블롭"(Binary Large Object)을 생성합니다.

### 쿼리 매개변수

- `digest`: 파일의 예상 SHA256 다이제스트

### 예제

#### 요청

```shell
curl -T model.gguf -X POST http://localhost:11434/api/blobs/sha256:29fdb92e57cf0827ded04ae6461b5931d01fa595843f55d36f5b275a52087dd2
```

#### 응답

블롭이 성공적으로 생성되면 201 Created를, 사용된 다이제스트가 예상과 다르면 400 Bad Request를 반환합니다. 

## 로컬 모델 목록

```
GET /api/tags
```

로컬에서 사용 가능한 모델을 나열합니다.

### 예제

#### 요청

```shell
curl http://localhost:11434/api/tags
```

#### 응답

단일 JSON 객체가 반환됩니다.

```json
{
  "models": [
    {
      "name": "deepseek-r1:latest",
      "model": "deepseek-r1:latest",
      "modified_at": "2025-05-10T08:06:48.639712648-07:00",
      "size": 4683075271,
      "digest": "0a8c266910232fd3291e71e5ba1e058cc5af9d411192cf88b6d30e92b6e73163",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "qwen2",
        "families": [
          "qwen2"
        ],
        "parameter_size": "7.6B",
        "quantization_level": "Q4_K_M"
      }
    },
    {
      "name": "llama3.2:latest",
      "model": "llama3.2:latest",
      "modified_at": "2025-05-04T17:37:44.706015396-07:00",
      "size": 2019393189,
      "digest": "a80c4f17acd55265feec403c7aef86be0c25983ab279d83f3bcd3abbcb5b8b72",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "llama",
        "families": [
          "llama"
        ],
        "parameter_size": "3.2B",
        "quantization_level": "Q4_K_M"
      }
    }
  ]
}
```

## 모델 정보 표시

```
POST /api/show
```

세부 정보, 모델파일, 템플릿, 매개변수, 라이선스, 시스템 프롬프트를 포함한 모델에 대한 정보를 표시합니다.

### 매개변수

- `model`: 표시할 모델의 이름
- `verbose`: (선택사항) `true`로 설정하면 상세 응답 필드에 대한 전체 데이터를 반환합니다

### 예제

#### 요청

```shell
curl http://localhost:11434/api/show -d '{
  "model": "llava"
}'
```

#### 응답

```json5
{
  "modelfile": "# Modelfile generated by \"ollama show\"\n# To build a new Modelfile based on this one, replace the FROM line with:\n# FROM llava:latest\n\nFROM /Users/matt/.ollama/models/blobs/sha256:200765e1283640ffbd013184bf496e261032fa75b99498a9613be4e94d63ad52\nTEMPLATE \"\"\"{{ .System }}\nUSER: {{ .Prompt }}\nASSISTANT: \"\"\"\nPARAMETER num_ctx 4096\nPARAMETER stop \"\u003c/s\u003e\"\nPARAMETER stop \"USER:\"\nPARAMETER stop \"ASSISTANT:\"",
  "parameters": "num_keep                       24\nstop                           \"<|start_header_id|>\"\nstop                           \"<|end_header_id|>\"\nstop                           \"<|eot_id|>\"",
  "template": "{{ if .System }}<|start_header_id|>system<|end_header_id|>\n\n{{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>\n\n{{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>\n\n{{ .Response }}<|eot_id|>",
  "details": {
    "parent_model": "",
    "format": "gguf",
    "family": "llama",
    "families": [
      "llama"
    ],
    "parameter_size": "8.0B",
    "quantization_level": "Q4_0"
  },
  "model_info": {
    "general.architecture": "llama",
    "general.file_type": 2,
    "general.parameter_count": 8030261248,
    "general.quantization_version": 2,
    "llama.attention.head_count": 32,
    "llama.attention.head_count_kv": 8,
    "llama.attention.layer_norm_rms_epsilon": 0.00001,
    "llama.block_count": 32,
    "llama.context_length": 8192,
    "llama.embedding_length": 4096,
    "llama.feed_forward_length": 14336,
    "llama.rope.dimension_count": 128,
    "llama.rope.freq_base": 500000,
    "llama.vocab_size": 128256,
    "tokenizer.ggml.bos_token_id": 128000,
    "tokenizer.ggml.eos_token_id": 128009,
    "tokenizer.ggml.merges": [],            // `verbose=true`이면 채워집니다
    "tokenizer.ggml.model": "gpt2",
    "tokenizer.ggml.pre": "llama-bpe",
    "tokenizer.ggml.token_type": [],        // `verbose=true`이면 채워집니다
    "tokenizer.ggml.tokens": []             // `verbose=true`이면 채워집니다
  },
  "capabilities": [
    "completion",
    "vision"
  ],
}
```

## 모델 복사

```
POST /api/copy
```

모델을 복사합니다. 기존 모델에서 다른 이름으로 모델을 생성합니다.

### 예제

#### 요청

```shell
curl http://localhost:11434/api/copy -d '{
  "source": "llama3.2",
  "destination": "llama3-backup"
}'
```

#### 응답

성공하면 200 OK를, 원본 모델이 존재하지 않으면 404 Not Found를 반환합니다.

## 모델 삭제

```
DELETE /api/delete
```

모델과 해당 데이터를 삭제합니다.

### 매개변수

- `model`: 삭제할 모델 이름

### 예제

#### 요청

```shell
curl -X DELETE http://localhost:11434/api/delete -d '{
  "model": "llama3:13b"
}'
```

#### 응답

성공하면 200 OK를, 삭제할 모델이 존재하지 않으면 404 Not Found를 반환합니다.

## 모델 다운로드

```
POST /api/pull
```

ollama 라이브러리에서 모델을 다운로드합니다. 취소된 다운로드는 중단된 지점부터 재개되며, 여러 호출이 동일한 다운로드 진행률을 공유합니다.

### 매개변수

- `model`: 다운로드할 모델의 이름
- `insecure`: (선택사항) 라이브러리에 대한 안전하지 않은 연결을 허용합니다. 개발 중에 자체 라이브러리에서 다운로드하는 경우에만 사용하세요.
- `stream`: (선택사항) `false`이면 응답이 객체 스트림이 아닌 단일 응답 객체로 반환됩니다

### 예제

#### 요청

```shell
curl http://localhost:11434/api/pull -d '{
  "model": "llama3.2"
}'
```

#### 응답

`stream`이 지정되지 않거나 `true`로 설정된 경우 JSON 객체의 스트림이 반환됩니다:

첫 번째 객체는 매니페스트입니다:

```json
{
  "status": "pulling manifest"
}
```

그 다음 일련의 다운로드 응답이 있습니다. 다운로드가 완료될 때까지 `completed` 키가 포함되지 않을 수 있습니다. 다운로드할 파일 수는 매니페스트에 지정된 레이어 수에 따라 다릅니다.

```json
{
  "status": "downloading digestname",
  "digest": "digestname",
  "total": 2142590208,
  "completed": 241970
}
```

모든 파일이 다운로드된 후 최종 응답은 다음과 같습니다:

```json
{
    "status": "verifying sha256 digest"
}
{
    "status": "writing manifest"
}
{
    "status": "removing any unused layers"
}
{
    "status": "success"
}
```

`stream`이 false로 설정된 경우 응답은 단일 JSON 객체입니다:

```json
{
  "status": "success"
}
```

## 모델 업로드

```
POST /api/push
```

모델을 모델 라이브러리에 업로드합니다. ollama.ai에 등록하고 먼저 공개 키를 추가해야 합니다.

### 매개변수

- `model`: `<namespace>/<model>:<tag>` 형식으로 업로드할 모델의 이름
- `insecure`: (선택사항) 라이브러리에 대한 안전하지 않은 연결을 허용합니다. 개발 중에 라이브러리에 업로드하는 경우에만 사용하세요.
- `stream`: (선택사항) `false`이면 응답이 객체 스트림이 아닌 단일 응답 객체로 반환됩니다

### 예제

#### 요청

```shell
curl http://localhost:11434/api/push -d '{
  "model": "mattw/pygmalion:latest"
}'
```

#### 응답

`stream`이 지정되지 않거나 `true`로 설정된 경우 JSON 객체의 스트림이 반환됩니다:

```json
{ "status": "retrieving manifest" }
```

그 다음:

```json
{
  "status": "starting upload",
  "digest": "sha256:bc07c81de745696fdf5afca05e065818a8149fb0c77266fb584d9b2cba3711ab",
  "total": 1928429856
}
```

그 다음 일련의 업로드 응답이 있습니다:

```json
{
  "status": "starting upload",
  "digest": "sha256:bc07c81de745696fdf5afca05e065818a8149fb0c77266fb584d9b2cba3711ab",
  "total": 1928429856
}
```

마지막으로 업로드가 완료되면:

```json
{"status":"pushing manifest"}
{"status":"success"}
```

`stream`이 `false`로 설정된 경우 응답은 단일 JSON 객체입니다:

```json
{ "status": "success" }
```

## 임베딩 생성

```
POST /api/embed
```

모델에서 임베딩을 생성합니다

### 매개변수

- `model`: 임베딩을 생성할 모델의 이름
- `input`: 임베딩을 생성할 텍스트 또는 텍스트 목록

고급 매개변수:

- `truncate`: 컨텍스트 길이에 맞도록 각 입력의 끝을 자릅니다. `false`이고 컨텍스트 길이가 초과되면 오류를 반환합니다. 기본값은 `true`입니다
- `options`: `temperature`와 같은 [Modelfile](./modelfile.md#valid-parameters-and-values) 문서에 나열된 추가 모델 매개변수
- `keep_alive`: 요청 후 모델이 메모리에 로드된 상태로 유지되는 시간을 제어합니다 (기본값: `5m`)

### 예제

#### 요청

```shell
curl http://localhost:11434/api/embed -d '{
  "model": "all-minilm",
  "input": "Why is the sky blue?"
}'
```

#### 응답

```json
{
  "model": "all-minilm",
  "embeddings": [[
    0.010071029, -0.0017594862, 0.05007221, 0.04692972, 0.054916814,
    0.008599704, 0.105441414, -0.025878139, 0.12958129, 0.031952348
  ]],
  "total_duration": 14143917,
  "load_duration": 1019500,
  "prompt_eval_count": 8
}
```

#### 요청 (다중 입력)

```shell
curl http://localhost:11434/api/embed -d '{
  "model": "all-minilm",
  "input": ["Why is the sky blue?", "Why is the grass green?"]
}'
```

#### 응답

```json
{
  "model": "all-minilm",
  "embeddings": [[
    0.010071029, -0.0017594862, 0.05007221, 0.04692972, 0.054916814,
    0.008599704, 0.105441414, -0.025878139, 0.12958129, 0.031952348
  ],[
    -0.0098027075, 0.06042469, 0.025257962, -0.006364387, 0.07272725,
    0.017194884, 0.09032035, -0.051705178, 0.09951512, 0.09072481
  ]]
}
```

## 실행 중인 모델 목록
```
GET /api/ps
```

현재 메모리에 로드된 모델을 나열합니다.

#### 예제

### 요청

```shell
curl http://localhost:11434/api/ps
```

#### 응답

단일 JSON 객체가 반환됩니다.

```json
{
  "models": [
    {
      "name": "mistral:latest",
      "model": "mistral:latest",
      "size": 5137025024,
      "digest": "2ae6f6dd7a3dd734790bbbf58b8909a606e0e7e97e94b7604e0aa7ae4490e6d8",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "llama",
        "families": [
          "llama"
        ],
        "parameter_size": "7.2B",
        "quantization_level": "Q4_0"
      },
      "expires_at": "2024-06-04T14:38:31.83753-07:00",
      "size_vram": 5137025024
    }
  ]
}
```

## 임베딩 생성 (구버전)

> 참고: 이 엔드포인트는 `/api/embed`로 대체되었습니다

```
POST /api/embeddings
```

모델에서 임베딩을 생성합니다

### 매개변수

- `model`: 임베딩을 생성할 모델의 이름
- `prompt`: 임베딩을 생성할 텍스트

고급 매개변수:

- `options`: `temperature`와 같은 [Modelfile](./modelfile.md#valid-parameters-and-values) 문서에 나열된 추가 모델 매개변수
- `keep_alive`: 요청 후 모델이 메모리에 로드된 상태로 유지되는 시간을 제어합니다 (기본값: `5m`)

### 예제

#### 요청

```shell
curl http://localhost:11434/api/embeddings -d '{
  "model": "all-minilm",
  "prompt": "Here is an article about llamas..."
}'
```

#### 응답

```json
{
  "embedding": [
    0.5670403838157654, 0.009260174818336964, 0.23178744316101074, -0.2916173040866852, -0.8924556970596313,
    0.8785552978515625, -0.34576427936553955, 0.5742510557174683, -0.04222835972905159, -0.137906014919281
  ]
}
```

## 버전

```
GET /api/version
```

Ollama 버전을 조회합니다

### 예제

#### 요청

```shell
curl http://localhost:11434/api/version
```

#### 응답

```json
{
  "version": "0.5.1"
}
```