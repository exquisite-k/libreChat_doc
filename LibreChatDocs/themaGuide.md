# 테마

## **1. 테마 관리 메커니즘**
- `ThemeContext.tsx`에서 테마 관련 로직을 관리합니다.
- 파일 위치 : LibreChat/client/**src**/**hooks**/**`ThemeContext.tsx`**
- rawSetTheme 함수에서 HTML의 root 요소(`document.documentElement`)에\
  'dark' 또는 'light' 클래스를 추가/제거하는 방식으로 테마를 적용합니다.

  ```js
  root.classList.remove(darkMode ? 'light' : 'dark');
  root.classList.add(darkMode ? 'dark' : 'light');
  ```

## **2. 테마 저장 방식**
- 사용자가 선택한 테마는 localStorage에 'color-theme' 키로 저장됩니다.

  ```js
  localStorage.setItem('color-theme', rawTheme);
  ```

## **3. 초기 로딩 시 테마 적용**
- `index.html`에는 페이지 로딩 시 테마를 적용하는 인라인 스크립트가 있습니다.
- 파일 위치 : LibreChat/client/**`index.html`**

  ```js
  const theme = localStorage.getItem('color-theme');
  // 테마에 따라 배경색을 설정
  ```

## **4. Tailwind 설정**
- `tailwind.config.cjs`에서 다크 모드는 클래스 기반으로 설정되어 있습니다.
- 파일 위치 : LibreChat/client/**`tailwind.config.cjs`**
- 이는 HTML 요소에 `'dark'` 클래스가 있을 때 다크 모드 스타일이 적용됨을 의미합니다.

  ```js
  darkMode: ['class'],
  ```

## **5. CSS 스타일 정의**
- `style.css`에서 `'.dark'` 클래스 선택자를 사용하여 다크 모드 시 적용될 CSS 변수들을 정의합니다.
- 파일 위치 : LibreChat/client/src/**`style.css`**

  ```css
  .dark {
    --text-primary: var(--gray-100);
    --surface-primary: var(--gray-900);
    /* 다른 다크 모드 변수들 */
  }
  ```

- 또한 일부 컴포넌트는 `.dark` 선택자를 사용하여 직접 스타일을 오버라이드합니다:

  ```css
  .dark .btn-neutral {
    background-color: transparent;
    color: rgba(255, 255, 240, var(--tw-text-opacity));
  }
  ```

> ---
> - 결론적으로, HTML의 root 요소(`<html>`)에 'dark' 클래스가 추가되면 다크 모드 스타일이 적용되고, 'light' 클래스가 추가되면 라이트 모드 스타일이 적용됩니다.
>
> - 이는 `Tailwind CSS`의 다크 모드 클래스 기반 구현 방식과 CSS 변수를 활용한 테마 시스템을 통해 구현되어 있습니다.
> ---



<!-- 수정 파일 -->
client/dist/index.html

