# LibreChat UI 패키지화 구조 제안

LibreChat의 프론트엔드를 독립적인 UI 라이브러리(`librechat-ui`)로 구성하여, 다른 React 프로젝트에서 손쉽게 재사용할 수 있도록 하는 구조입니다. 이 문서는 이를 위한 **단계별 알고리즘**, **설명**, **주의사항**을 포함하고 있습니다.

---

## 📦 목표
- LibreChat의 프론트엔드를 **재사용 가능한 React 컴포넌트 라이브러리로 분리**
- 다른 프로젝트에서 `import { ChatWindow } from 'librechat-ui'`처럼 쉽게 사용
- 상태/스타일/서비스를 모듈화하여 유지보수성과 커스터마이징 향상

---

## 📋 패키지 구성 알고리즘 (단계별)
### 🔧 Step 1. 원본 코드 클론 및 정리
**1.1** LibreChat 저장소 클론
```bash
git clone https://github.com/danny-avila/LibreChat.git
```

**1.2** LibreChat 저장소 클론
```bash
cp -r LibreChat/client librechat-ui
```


### 🎨 Step 2. 컴포넌트 단위로 UI 정리
#### 2.1 주요 UI 컴포넌트 폴더 구조 구성
```bash
librechat-ui/
├── src/
│   ├── components/
│   │   ├── ChatWindow.tsx
│   │   ├── MessageList.tsx
│   │   ├── ChatInput.tsx
│   │   └── Sidebar.tsx
│   ├── hooks/
│   │   └── useChat.ts
│   ├── context/
│   │   └── ChatProvider.tsx
│   ├── services/
│   │   └── chatService.ts
│   └── index.ts
```


#### 2.2 모든 컴포넌트를 index에서 export
```ts
// index.ts
export { ChatWindow } from './components/ChatWindow';
```


### ⚙️ Step 3. 상태/서비스 분리
#### 3.1 zustand 또는 context를 통해 상태 분리
- 예: 채팅 메시지, 사용자 상태

#### 3.2 API 호출 추상화
```ts
// services/chatService.ts
export const sendMessage = async (message: string) => {
  return await fetch('/api/chat', {
    method: 'POST',
    body: JSON.stringify({ message }),
  }).then(res => res.json());
};
```

**주의사항**
- 실제 백엔드 URL은 props로 주입 가능하도록 처리
- fetch, axios 등 클라이언트에 맞게 선택


### 🎨 Step 4. 테마와 스타일링 정리
#### 4.1 Tailwind 사용 시, 외부 앱과 충돌 방지
- `tailwind.config.js`에 prefix 설정 (예: `lc-`)

```js
module.exports = {
  prefix: 'lc-',
};
```

#### 4.2 classnames, CSS Modules 등으로 스타일 스코프 제한
**주의사항**
- 테마는 prop으로 제어 가능하도록 설계
- 예: `theme="dark"` or `theme="light"`


### 🧪 Step 5. 테스트 및 빌드
#### 5.1 독립적인 테스트 페이지 생성
```tsx
// App.tsx (테스트용)
import { ChatWindow } from './librechat-ui';

function App() {
  return <ChatWindow userId="123" apiBase="/api" theme="dark" />;
}
```

#### 5.2 빌드 설정 (Vite, Rollup, or tsup 권장)
- 예: `tsup.config.ts` 또는 `vite.config.ts`
```ts
// tsup.config.ts
export default {
  entry: ['src/index.ts'],
  format: ['esm', 'cjs'],
  dts: true,
  outDir: 'dist',
};
```
```bash
npm install --save-dev tsup
npx tsup
```


### 📦 Step 6. 다른 프로젝트에서 사용
#### 6.1 로컬 npm 링크 or 패키지 설치
```bash
npm link ../librechat-ui
```

#### 6.2 사용 예
```tsx
import { ChatWindow } from 'librechat-ui';

<ChatWindow
  userId="abc123"
  apiBase="https://your.api.server"
  theme="dark"
/>;
```



## ⚠️ 주의사항 요약
| 항목      | 설명                                                          |
| ------- | ----------------------------------------------------------- |
| 인증 연동   | 다른 앱의 인증 시스템과 연결할 수 있도록 `token` props 등으로 인증 전달             |
| 상태 충돌   | 다른 앱과의 context/provider 충돌을 피하기 위해 scope 관리 필요              |
| 스타일 충돌  | Tailwind prefix, CSS module 등으로 방지                          |
| API 유연성 | `/api/chat` 같은 경로를 고정하지 말고 `apiBase` 등으로 외부 주입 처리           |
| SSR     | SSR 대응 여부는 서버 환경에 따라 고려 (Next.js 등에서 `useEffect` 조건부 처리 필요) |



## ✅ 마무리
이 구조를 통해 LibreChat의 강력한 채팅 UI를 **자체 앱의 일부로 자유롭게 사용**할 수 있습니다.
라이브러리화된 `librechat-ui`는 확장성과 재사용성을 모두 갖추게 됩니다.



## 📚 관련 도구 추천
- 상태 관리: zustand, react-context
- 빌드 도구: tsup, rollup, vite
- 스타일: tailwindcss, CSS modules
- 테스팅: storybook, vitest, jest

