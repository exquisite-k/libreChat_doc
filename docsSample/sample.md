
## LibreChat 스타일 커스터마이징 관련 파일

### 1. 주요 스타일 파일
- **LibreChat/client/src/style.css**
  - 메인 스타일 시트 (2,681줄)
  - 기본 색상 변수, 테마 설정, 컴포넌트 스타일 등 정의
  - Light/Dark 모드 변수 정의
  - 애니메이션, 버튼 스타일, 마크다운 렌더링 스타일 등 포함

- **LibreChat/client/src/mobile.css**
  - 모바일 반응형 스타일 (374줄)
  - 모바일 화면에서의 네비게이션, 레이아웃, 애니메이션 정의
  - 미디어 쿼리를 통한 화면 크기별 스타일 조정

- **LibreChat/client/tailwind.config.cjs**
  - Tailwind CSS 설정 파일
  - 커스텀 색상, 폰트, 애니메이션 등의 테마 확장
  - 다크모드 설정 및 CSS 변수 매핑

### 2. 테마 관련 파일
- **LibreChat/client/src/hooks/ThemeContext.tsx**
  - 테마 상태 관리 및 컨텍스트 제공
  - 라이트/다크 모드 전환 로직
  - 시스템 테마 감지 및 적용 기능

- **LibreChat/client/src/utils/theme.ts**
  - 테마 유틸리티 함수
  - 폰트 사이즈 적용 및 초기 테마 설정 로직

### 3. 테마 선택기 컴포넌트
- **LibreChat/client/src/components/ui/ThemeSelector.tsx**
  - 테마 전환 UI 컴포넌트
  - 라이트/다크/시스템 테마 선택 기능
  - 단축키 지원 (Ctrl+Shift+T)

- **LibreChat/client/src/components/Nav/SettingsTabs/General/General.tsx**
  - 설정 메뉴 내 테마 선택 UI

### 4. 커스터마이징 포인트
- **색상 변수**: style.css의 `:root` 및 `.dark` 선택자 내 변수 수정
- **테마 확장**: tailwind.config.cjs의 `theme.extend` 섹션에 추가
- **모바일 스타일**: mobile.css 수정으로 반응형 디자인 조정
- **컴포넌트 스타일**: 개별 컴포넌트의 className 속성 참조

### 5. 스타일 커스터마이징 방법
1. **테마 색상 변경**: LibreChat/client/src/style.css의 CSS 변수 수정
2. **폰트 변경**: tailwind.config.cjs의 fontFamily 설정 수정
3. **레이아웃 조정**: 관련 컴포넌트의 Tailwind 클래스 수정
4. **다크모드 커스텀**: .dark 클래스 내부의 변수 값 수정

## LibreChat 레이아웃 구성 관련 파일

### 1. 주요 레이아웃 파일
- **LibreChat/client/src/routes/Layouts/Startup.tsx**
  - 애플리케이션 시작 시 초기 레이아웃
  - 로그인/회원가입 화면의 기본 레이아웃 정의
  - 인증 상태에 따른 리다이렉션 로직 포함

- **LibreChat/client/src/components/Auth/AuthLayout.tsx**
  - 인증 화면(로그인, 회원가입 등)의 레이아웃
  - 로고, 헤더, 에러 메시지, 테마 선택기 포함
  - 소셜 로그인 렌더링 처리

- **LibreChat/client/src/components/Chat/Presentation.tsx**
  - 채팅 화면의 메인 레이아웃 컴포넌트
  - 크기 조절 가능한 패널 레이아웃 처리
  - 사이드 패널과 메인 콘텐츠 영역 구성

### 2. 사이드 패널 및 네비게이션
- **LibreChat/client/src/components/SidePanel/SidePanelGroup.tsx**
  - 사이드 패널 그룹 컴포넌트
  - 크기 조절 및 접기/펼치기 기능
  - 레이아웃 상태 저장 처리

- **LibreChat/client/src/components/SidePanel/SidePanel.tsx**
  - 사이드 패널 컴포넌트
  - 내비게이션 메뉴와 컨트롤 포함
  - 반응형 디자인을 위한 화면 크기 감지

- **LibreChat/client/src/components/SidePanel/Nav.tsx**
  - 사이드 패널 내 네비게이션 메뉴
  - 아코디언 스타일 메뉴 구현
  - 축소 모드와 확장 모드 지원

- **LibreChat/client/src/components/Nav/Nav.tsx**
  - 메인 네비게이션 컴포넌트
  - 대화 목록, 북마크, 설정 등 포함
  - 모바일/데스크톱 레이아웃 전환 처리

### 3. 채팅 화면 레이아웃
- **LibreChat/client/src/components/Chat/ChatView.tsx**
  - 채팅 뷰 메인 컴포넌트
  - 헤더, 메시지 목록, 입력 폼 구성
  - 랜딩 페이지와 채팅 페이지 레이아웃 처리

- **LibreChat/client/src/components/Chat/Input/ChatForm.tsx**
  - 채팅 입력 폼 컴포넌트
  - 텍스트 입력, 파일 첨부, 버튼 등 배치
  - 반응형 디자인 및 RTL(오른쪽에서 왼쪽) 지원

- **LibreChat/client/src/components/Chat/Messages/Content/Container.tsx**
  - 메시지 콘텐츠 컨테이너
  - 메시지 레이아웃 및 스타일 정의
  - 첨부 파일 표시 처리

- **LibreChat/client/src/components/Chat/Messages/MessageParts.tsx**
  - 메시지 구성 요소 컴포넌트
  - 사용자/에이전트 메시지 스타일 구분
  - 메시지 편집 및 상호작용 UI

### 4. 레이아웃 토글 및 컨트롤
- **LibreChat/client/src/components/Chat/Menus/OpenSidebar.tsx**
  - 사이드바 열기 버튼 컴포넌트
  - 사이드바 가시성 토글 처리
  - 반응형 디자인 지원

- **LibreChat/client/src/components/Nav/NavToggle.tsx**
  - 네비게이션 토글 컴포넌트
  - 애니메이션 효과가 있는 토글 버튼
  - 왼쪽/오른쪽 사이드바 지원

### 5. 레이아웃 커스터마이징 방법
1. **전체 레이아웃 수정**: Presentation.tsx와 ChatView.tsx 파일을 수정하여 전체 레이아웃 구조 변경
2. **사이드바 수정**: SidePanel 관련 컴포넌트 수정으로 사이드바 디자인 및 동작 변경
3. **채팅 UI 수정**: Messages와 ChatForm 컴포넌트를 수정하여 채팅 인터페이스 변경
4. **반응형 디자인**: 미디어 쿼리와 useMediaQuery 훅을 활용한 반응형 레이아웃 조정