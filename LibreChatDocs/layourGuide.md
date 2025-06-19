
# LibreChat 레이아웃 파일 참고

## LibreChat/client/

**LibreChat/client/index.html**
- 인덱스

**LibreChat/client/vite.config.ts**
- 파비콘

> ---

## LibreChat/client/**src**/
- main.jsx
- App.jsx

### components/**Artifacts**/
- 미확인

### components/**Audio**/
- TTS.tsx : 메세지 영역 [소리내어읽기], [TTS] 오브젝트
- Voices.tsx : 설정 모달 음성 선택

### components/**Auth**/
- AuthLayout.tsx : 로그인 페이지
- Footer.tsx : 푸터 & 개인정보 처리방침 버튼 & 서비스 이용약관 버튼
- Login.tsx : openID 로그인 & 회원가입 버튼
- LoginForm.tsx : 로그인 폼
- Registration.tsx : 회원가입 폼
- SocialButton.tsx : 소셜 로그인 버튼 wrap

### components/**Banners**/
- 미확인

### components/**Bookmarks**/
- BookmarkEditDialog.tsx : 우측 메뉴 북마크 탭 모달
- BookmarkForm.tsx : 우측 메뉴 북마크 탭 모달 폼
- 외 미확인(용도는 파악되는데 찾아보기에 시간부족)

### components/**Chat**/
- AddMultiConvo.tsx : 다중 응답 대화 추가 버튼
- ChatView.tsx : 채팅창 영역 & 채팅창 입력 영역
- ExportAndShareMenu.tsx : 내보내기/공유 버튼
- Footer.tsx : 개인정보 처리방침 버튼 & 서비스 이용약관 버튼 & 채팅창 푸터 영역
- Header.tsx : 채팅창 헤더 영역
- Landing.tsx : 새 채팅 화면 에이전트 설명 영역
- Presentation.tsx : 채팅창 메인 영역
- TemporaryChat.tsx : 임시 채팅 버튼

    #### components/Chat/**Input**
    - AudioRecorder.tsx : 채팅 입력창 음성 녹음 버튼
    - 미확인(채팅 입력창 관련 컴포넌트로 예상)

    #### components/Chat/**Menus**
    - 미확인

    #### components/Chat/**Messages**
    - Message.tsx : 질문자 메세지 영역
    - 미확인(메시지 영역 컨텐츠로 예상)

### components/**Conversations**/
- 

### components/**Endpoints**/
- 

### components/**Files**/
- 

### components/**Input**/
- 

### components/**Messages**/
- MessageContent.tsx : AI 메세지 박스(외부)
- ContentRender.tsx : AI 메세지 박스(내부)
- ScrollToBottom.tsx : 맨 아래로 버튼

    #### components/Messages/**Content**/
    - Messages/Content/CodeBlock.tsx : 코드 박스
    - Messages/Content/RunCode.tsx : 코드 실행 버튼
    - Messages/Content/Error.tsx : 오류 메세지 출력 텍스트

### components/**Nav**/
- AccountSettings.tsx : 좌측 하단 사용자 버튼
- MobileNav.tsx : 좌측 사이드 메뉴 토글 버튼(모바일) & 채팅 타이틀(모바일) & 새 채팅 버튼(모바일)
- Nav.tsx : 좌측 사이드메뉴
- NavToggle.tsx : 우측 사이드메뉴 토글 버튼
- NewChat.tsx : 좌측 사이드 메뉴 토글 버튼 & 새 채팅 버튼
- SearchBar.tsx : 메시지 검색 영역
- Settings.tsx : 설정 모달 창

    #### components/Nav/**Bookmarks**
    - 북마크 버튼 관련(미확인)
    #### components/Nav/**ExportConversation**
    - 내보내기 관련으로 보임(미확인)
    #### components/Nav/**SettingsTabs**
    - Chat/Chat.tsx : 설정 모달 채팅 탭 내용
    - 설정 모달 내부 탭 관련(미확인)


### components/**Plugins**/
- 

### components/**Prompts**/
- 

### components/**Share**/
- 

### components/**SidePanel**/
- 

### components/**svg**/
- 

### components/**Tools**/
- 

### components/**ui**/
- 

### components/**Web**/
- 

