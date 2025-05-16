# 📘 LibreChat 커스터마이징 UI 프로젝트 명세서
## 🧭 프로젝트 목표
LibreChat의 백엔드는 그대로 유지하되,
디자인 커스터마이징이 자유롭고 재사용 가능한 UI 시스템을 별도 개발하는 것이 목적입니다.

* 기존 LibreChat UI(client 폴더)는 그대로 두고, 새롭게 custom-ui 폴더에서 Next.js 기반 UI를 개발
* 디자인 테마는 공통 스타일 파일(theme.css) 에 정의하여 다른 프로젝트에서도 재사용 가능하게 구성
* 추후 API Key가 연결되면 실제 챗봇 기능도 연동 가능


## 📁 프로젝트 구조
```
LibreChat/
├── client/              # 기존 LibreChat UI (보존)
├── server/              # 백엔드 API
├── custom-ui/           # 🔧 새로 개발할 커스터마이징 UI
│   ├── components/      # 공통 UI 컴포넌트
│   ├── pages/           # Next.js 페이지
│   ├── styles/
│   │   ├── theme-light.css
│   │   ├── theme-dark.css
│   │   └── index.css
│   ├── tailwind.config.js
│   └── ...ㅆㅆ
├── docker-compose.yml
└── ...
```


## 🧩 작업 방향성 요약
| 항목                | 설명                                               |
| ------------------ | -------------------------------------------------- |
| 백엔드 유지          | LibreChat 백엔드 그대로 사용 (`:3080` 포트)           |
| 프론트 분리 개발     | `custom-ui/` 폴더에 새 Next.js 앱 생성               |
| 디자인 토큰 중심 설계 | 색상, 여백, 폰트 등은 CSS 변수 or Tailwind 테마로 분리 |
| 컴포넌트 분리 구조   | 버튼, 입력창, 메시지 박스 등은 `/components`로 분리     |
| 실시간 반영         | `npm run dev` 로 개발용 서버에서 즉시 확인             |
| API 연결 (옵션)     | 백엔드 API는 REST 또는 WebSocket으로 후속 연동         |


## 🛠 필요한 작업 목록

### 1. 앱 생성

#### 1-1. Vite + React 앱 생성
``` bash
npm create vite@latest custom-ui -- --template react-ts
cd custom-ui
npm install
```

#### 1-2. Next.js 앱 생성
``` bash
npx create-next-app@latest custom-ui
```


### 2. TailwindCSS + 스타일 시스템 구성
* tailwind.config.js 설정
* styles/theme-light.css, theme-dark.css 생성
* 모든 색상, 여백, 폰트 등을 :root 변수로 관리


### 3. 기본 컴포넌트 제작
| 컴포넌트              | 설명                     |
| ------------------- | ------------------------ |
| `<ChatInput />`     | 메시지 입력창              |
| `<ChatBubble />`    | 사용자/모델 메시지 말풍선    |
| `<ChatLayout />`    | 전체 채팅 화면             |
| `<ModelSelector />` | (후속 연동 시) 모델 선택 UI |


### 4. 백엔드 API 인터페이스 구성 (선택)
``` ts
// lib/api.ts
export async function sendMessage(text: string) {
  const res = await fetch('http://localhost:3080/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message: text }),
  });
  return await res.json();
}
```


### 5. 디자인 스킨 교체 기능 (선택)
```tsx
<button onClick={() => setTheme('theme-light.css')}>Light</button>
<button onClick={() => setTheme('theme-dark.css')}>Dark</button>
```


# 📘 커스터마이징 UI 기능 명세서

## 1. 공통
- ✅ 기존 LibreChat 서버 유지
- ✅ 기존 프론트엔드(client)는 수정하지 않음
- ✅ `/custom-ui` 폴더에 새로운 프론트앱 작성
- ✅ API Key 없이 UI만 먼저 개발 가능

## 2. 스타일 시스템
- ✅ 색상/여백/폰트 등 CSS 변수로 통일 관리
- ✅ `theme-light.css`, `theme-dark.css` 등의 스킨 분리
- ✅ Tailwind config와 연동 가능
- ✅ 프로젝트 외부에 복사해도 스타일 재사용 가능

## 3. 컴포넌트 구조
- `ChatInput`: 메시지 입력창
- `ChatBubble`: 채팅 말풍선 (사용자/모델 구분)
- `ChatLayout`: 전체 레이아웃
- `ModelSelector`: 모델 드롭다운 (후속 추가)

## 4. API 연동 준비
- ✅ `sendMessage()` 함수에서 백엔드 API 연동
- ✅ Key가 없을 경우, 목업 응답으로 처리

## 5. 디자인 확장성
- ✅ 다양한 테마를 쉽게 갈아끼울 수 있는 구조
- ✅ shadcn/ui 또는 다른 Tailwind UI 프레임워크와 호환 가능


# 🚀 추가로 작업 예정인 부분
* 샘플 UI 컴포넌트 코딩
* 목업 메시지로 더미 채팅 테스트 UI 구성