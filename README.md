# react2
# 윤상우 202130224

---

## 📘 Next.js 실습 프로젝트 정리

이 저장소는 2025년 Next.js 수업에서 진행한 실습 내용을 정리한 자료입니다.

---

### 11월 12일(12주차)
- 스트리밍 (Streaming)
  - 개념 요약
    - 스트리밍은 페이지 HTML을 한 번에 모두 렌더링하지 않고, 작은 조각 단위로 잘라 순차적으로 전송하는 방식입니다.
      - 초기 화면을 더 빨리 띄워서 사용자에게 콘텐츠를 빠르게 보여 줄 수 있음.
  - 전제 조건
    - `cacheComponents_config` 옵션이 활성화되어 있다고 가정.
    - Next.js 15 Canary 버전부터 본격 지원.
    - `latest`는 안정(stable) 버전, `canary`는 최신 개발(dev) 버전.
  - 특징
    - 서버 컴포넌트에서 `async/await`을 사용하면 Next.js는 해당 부분을 동적 렌더링(Server Rendering) 대상으로 판단함.
    - 요청이 들어올 때 서버에서 데이터를 가져와 그 결과를 기준으로 렌더링을 수행.
    - 데이터 응답이 느리면 전체 화면이 늦게 나올 수 있으므로, 이럴 때 스트리밍을 사용하면 부분적으로 먼저 보여 줄 수 있어 유리함.
  - 구현 방법
    - 방법1 - `loading.tsx` 파일 사용
      - 전체 페이지가 준비되는 동안 보여 줄 로딩 UI를 정의.
      - 예: `app/blog/page.tsx`를 스트리밍하려면 → `app/blog/loading.tsx` 파일 생성.
        ```tsx
        export default function Loading() {
          return <div>Loading...</div>
        } 
        ```
    - 방법2 - `<Suspense>` 사용
      - 페이지의 특정 영역만 스트리밍하고 싶을 때 사용.
      - `<Suspense>`로 감싸지 않은 부분은 바로 렌더링되고, 감싼 영역만 비동기로 처리됨.
      ```tsx
      import { Suspense } from 'react';
      import BlogListSection from '@/components/BlogListSection';
      
      export default function BlogPage() {
        return (
          <Suspense fallback={<div>Loading posts...</div>}>
            <BlogListSection />
          </Suspense>
        );
      }
      ```

- 의미 있는 로딩 상태 (Instant Loading State)
  - 개념
    - 로딩 상태는 데이터를 기다리는 동안 사용자에게 즉시(instant) 보여 주는 대체 UI를 의미함.
    - 폴더 안에 `loading.tsx`를 만들어 두면, 그 폴더의 모든 하위 페이지에 동일한 로딩 UI를 적용할 수 있음.
  - 디자인 팁
    - 사용자가 “지금 어떤 걸 불러오고 있는지” 직관적으로 이해할 수 있도록 **의미 있는 로딩 상태**를 설계하는 것이 좋음.
      - 예) 스켈레톤(Skeleton), 스피너(Spinner)
        - 단순한 로딩 아이콘보다는 실제 컨텐츠의 형태를 어느 정도 짐작할 수 있는 UI가 더 바람직함.

- 스켈레톤 vs 스피너
  - 스켈레톤 (Skeleton)
    - 최종 콘텐츠의 레이아웃이나 구조를 회색 블록 등으로 먼저 보여주는 방식.
  - 스피너 (Spinner)
    - 단순히 “로딩 중”임을 알려주는 회전 아이콘 형태의 표시.
  - 게시글 목록, 카드, 썸네일처럼 레이아웃이 중요한 화면에서는 스켈레톤이 사용자 경험 측면에서 더 좋을 때가 많음.

- 데이터 Fetch 패턴
  - 순차적 Fetch (Sequential Fetch)
    - 컴포넌트 트리 상에서 상위 → 하위 순서로 데이터를 가져올 때 발생.
    - 예: `Playlists`를 가져오려면 먼저 `Artist`의 `artistID`가 필요해서, `Artist` 데이터를 전부 받은 뒤에야 `Playlists` fetch를 시작할 수 있는 경우.
    - 이런 식으로 요청들 사이에 의존성이 존재함.
  - 병렬 Fetch (Parallel Fetch)
    - 하나의 경로 안에서 여러 데이터 요청을 동시에 진행하는 패턴.
    - 기본적으로 레이아웃(Layout)과 페이지(Page)는 병렬로 렌더링됨.
    - 한 컴포넌트 안에서 여러 `await`를 순서대로 쓰면 직렬 실행이 될 수 있으므로, 병렬 처리를 위해 `Promise.all`을 활용할 수 있음.
  ```tsx
  const [artist, albums] = await Promise.all([
    getArtist(username),
    getAlbums(username)
  ]);


---

### 11월 5일(11주차)
테크페어 준비로 인한 미출석

---
### 10월 29일(10주차)

- **Context Provider (컨텍스트 제공자)**
  - Props 없이도 전역 상태(theme, 언어 등)를 트리 전체에 공유할 수 있다.
  - Provider를 **Server Component**에서 감싸면, 하위 **Client Component**들이 동일한 Context를 사용할 수 있다.

```tsx
// app/layout.tsx (Server Component)
import ThemeProvider from "./theme-provider";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

```tsx
// theme-provider.tsx (Client Component)
"use client";
import { createContext, useState, useEffect } from "react";

export const ThemeContext = createContext({
  theme: "light",
  toggleTheme: () => {},
});

export default function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState("light");
  const toggleTheme = () => setTheme(prev => (prev === "light" ? "dark" : "light"));

  useEffect(() => {
    if (typeof window !== "undefined") {
      document.documentElement.dataset.theme = theme;
    }
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

- **CSS 적용 (Attribute Selector)**
  - `html[data-theme='light']` 형식으로 테마를 구분한다.
  - 클래스보다 충돌이 적고 전역 테마 관리에 적합하다.

```css
html[data-theme="light"] {
  background-color: #ffffff;
  color: #000000;
}

html[data-theme="dark"] {
  background-color: #000000;
  color: #ffffff;
}
```

---

- **useEffect Hook 설명**
  - HTML 문서 전체에 theme를 적용하는 전형적인 패턴이다.
  - `typeof window !== "undefined"` 조건을 넣어 **SSR 환경**에서도 안전하게 실행되도록 한다.

---

- **Provider 구성 시 주의**
  - ThemeProvider는 `<html>` 대신 `{children}`만 감싸야 한다.
  - Provider는 트리에서 **한 번만** 사용하는 것이 좋다 → 불필요한 렌더링 방지.
  - 이렇게 하면 Server Component의 정적 부분을 최적화하기 쉬워진다.

---

- **환경 변수 노출 방지**
  - JS 모듈은 server와 client가 공유될 수 있으므로 주의해야 한다.
  - 서버 전용 코드(`process.env`)는 client에서 접근하지 않도록 분리한다.

```ts
// lib/data.ts
export async function getData() {
  const res = await fetch("https://external-service.com/data", {
    headers: {
      Authorization: process.env.API_KEY,
    },
  });
  return res.json();
}
```

---

- **데이터 가져오기 (Fetching Data)**
  - 서버 컴포넌트에서 데이터를 가져오는 방법:
    1. `fetch` API 사용
    2. ORM 또는 데이터베이스 직접 접근
  - `fetch` 사용 시 컴포넌트를 **비동기 함수(async)** 로 선언해야 한다.

```tsx
// app/blog/page.tsx
export default async function Page() {
  const res = await fetch("https://api.vercel.app/blog");
  const posts = await res.json();

  return (
    <ul>
      {posts.map((post: any) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

---

### 10월 22일(9주차)
- Server 및 Client Component Interleaving
  - 개념
    - Interleaving: 여러 데이터 블록을 섞어서 전송 → 오류 발생 시 영향을 최소화하는 기술.
    - Next.js에서의 의미:
      - Server Component와 Client Component가 섞여서(interleaved) 동작하는 구조.
  - 작동 원리
    1. Server Component가 서버에서 렌더링되어 HTML로 변환됨.
    2. 이 HTML이 Client Component의 children 자리에 삽입됨.
    3. 클라이언트에서는 Client Component만 hydration(JS 연결) 진행.
    4. 서버 데이터는 이미 들어와 있으므로, 이벤트(버튼 클릭 등)는 클라이언트에서 처리.
```
// components/ServerContent.tsx
export default async function ServerContent() {
  const data = await fetch("https://jsonplaceholder.typicode.com/posts/1").then(r => r.json());
  return <div>{data.title}</div>;
}
```
```
// app/interleaved/page.tsx
import ClientLayout from "@/components/ClientLayout";
import ServerContent from "@/components/ServerContent";

export default function Page() {
  return (
    <ClientLayout>
      <ServerContent />
    </ClientLayout>
  );
}

```
- Context Provider (컨텍스트 제공자)
  - Provider Component를 Server Component(Layout 등)에서 감싸면,
앱 전체에서 모든 Client Component가 동일한 Context를 공유할 수 있음.
  - Props 없이도 전역적으로 theme, 언어, 설정 등을 전달 가능.
```
// app/layout.tsx (Server Component)
import ThemeProvider from './theme-provider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```
```
// theme-provider.tsx (Client Component)
import { createContext, useState } from "react";

export const ThemeContext = createContext({
  theme: "light",
  toggleTheme: () => {},
});

export default function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState("light");
  const toggleTheme = () => setTheme(theme === "light" ? "dark" : "light");

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <html data-theme={theme}>{children}</html>
    </ThemeContext.Provider>
  );
}
```
- CSS 적용 (Attribute Selector)
  - html[data-theme='light'] → 속성 선택자(Attribute Selector)
  - 클래스(.class) 대신 속성값을 기준으로 스타일 지정 가능.
  - 여러 class를 중첩하지 않아도 되므로 스타일 충돌을 줄임.
```
html[data-theme='light'] {
  background-color: white;
  color: black;
}

html[data-theme='dark'] {
  background-color: black;
  color: white;
}
```

---
# 10월 01일(6주차)
## generateStaticParams
- 어떤 동적 페이지들을 미리 만들지 알려주는 함수
- 예제:  
  ```tsx
  // [slug]/data.tsx
  export const posts = [
    { id: "1", title: "첫 번째 글", content: "내용 1" },
    { id: "2", title: "두 번째 글", content: "내용 2" }
  ];

  // [slug]/page.tsx
  import { posts } from "./data";

  interface BlogPageProps {
    params: { Promise<{slug: string}> };
  }

  // Next.js가 빌드 시 호출
  export async function generateStaticParams() {
    return posts.map(post => ({
      slug: post.id, // id === [slug]경로에 들어갈 값
    }));
  }

  export default function BlogPage({ params }: BlogPageProps) {
    const post = posts.find(p => p.id === params.slug);

    if (!post) return <p>글을 찾을 수 없습니다.</p>;

    return (
      <div>
        <h1>{post.title}</h1>
        <p>{post.content}</p>
      </div>
    );
  }

  ```
---


### 5주차 병결

---

### 9월 17일 (4주차)
- 중첩 라우트 만들기
  - 폴더 구조: Next.js App Router에서 /blog 경로를 만들려면 app/blog 폴더를 생성하고 page.tsx 파일을 추가.
  - page.tsx는 /blog URL로 접근할 때 보여줄 페이지.
- [slug] 동적 라우트 개념
  - /blog/[slug] 의 [slug]는 동적 세그먼트로, 데이터의 key(예: 글의 고유 식별자)를 의미.
  - 데이터에 반드시 slug 필드가 있어야 해당 경로에서 사용 가능.
  - /blog/nextjs 처럼 호출하면 slug 값이 nextjs로 전달됨.
- [slug] 코드 설명
  - export default async function Posts({ params }: { params: { slug: string } })
    - async: 함수 내에서 await 사용 가능.
    - params는 { slug: string } 형태의 객체로 전달.
  - const { slug } = await params;
    - params가 Promise일 수 있어 await로 실제 값을 꺼낸 뒤 구조 분해 할당.
- searchParams(검색 매개변수)
  - params vs searchParams
    - params: URL path 중 동적 세그먼트(/blog/[slug]) 값.
    - searchParams: ?key=value 형태의 쿼리스트링 값.
  - 사용 시점
    - 서버에서 데이터를 필터링해야 할 때 searchParams prop 사용.
    - 클라이언트 단에서만 필터링 시 useSearchParams() 사용.
    - 이벤트 핸들러에서 new URLSearchParams(window.location.search)로 직접 읽을 수도 있음.
---
### 9월 10일 (3주차)


## 📁 폴더
- **app/** → App Router (라우팅 기본)
- **pages/** → Pages Router
- **public/** → 정적 파일
- **src/** → 코드와 설정 분리용

## 🏗️ 라우팅
- `page.js` → 엔드포인트
- `layout.js` → 상태 유지 레이아웃
- `template.js` → 매번 새로 생성
- `(group)` → URL에 영향 없음
- `_folder` → 라우팅 제외

## ⚡ 기능
- **loading.js** → 스켈레톤 로딩
- **error.js** → 에러 경계
- **not-found.js** → 404 처리
- **metadata** → SEO 자동 삽입

## 🌐 OGP
- `<meta property="og:...">` → 링크 공유 미리보기

---


### 9월 3일 (2주차)
- my-app 프로젝트 생성
- my-pnpm 프로젝트 생성
- ESLint 설정 (Installation)
    - Strict (추천): Next.js 기본 ESLint 구성 + Core Web Vitals 규칙 세트 포함. 초보 개발자에게 권장.
    - Base: Next.js의 기본 ESLint 구성만 포함.
    - Strict/Base 선택 시: eslint, eslint-config-next 패키지가 자동 설치됨.
    - .eslintrc.json 파일 생성 후, next lint 실행하면 오류 자동 검출.
- 자동 생성되는 항목
    프로젝트 생성 시 자동으로 세팅되는 것들:
    - package.json (scripts 추가), public 디렉토리
    - 선택옵션
        - TypeScript: tsconfig.json 생성
        - ESLint: .eslintrc.json 대신 eslint.config.mjs 생성
        - Tailwind CSS
        - src 디렉토리 구조
        - App Router: app/layout.tsx, app/page.tsx
        - Turbopack
        - Import alias: tsconfig.json에 paths 생성

- .eslintrc.json vs eslint.config.mjs
    - .eslintrc.json
        - JSON 형식 → 주석/변수/조건문 불가능
        - 정적인 설정만 가능
    - eslint.config.mjs
        - JS 모듈(ESM) 방식 → 주석, 변수, 조건문, 동적 로딩 가능
        - 다른 설정 파일 import 가능 → 재사용 용이
        - 최신 ESLint 공식 권장 방식
- pnpm 설치
```
npm i -g pnpm@latest
```

### 8월 27일 (1주차)
- OT진행
- 기본 프로그램 설치 및 설정
- test1 프로젝트 생성 및 test코딩
