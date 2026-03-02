# 스택별 프로젝트 생성 가이드

각 기술 스택별로 프로젝트를 생성하고 초기 설정하는 구체적인 명령어와 설정 파일을 담고 있다.

---

## React + Vite + TypeScript + Tailwind

사내 관리 도구에 가장 자주 쓰이는 조합.

### 생성 명령어

```bash
npm create vite@latest [프로젝트명] -- --template react-ts
cd [프로젝트명]
npm install

# Tailwind CSS
npm install -D tailwindcss @tailwindcss/vite

# 라우터
npm install react-router-dom

# 아이콘 (선택)
npm install lucide-react

# 상태관리 (필요시)
npm install zustand

# 날짜 처리 (필요시)
npm install date-fns
```

### Tailwind 설정

vite.config.ts에 Tailwind 플러그인 추가:

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

src/index.css에 Tailwind import:

```css
@import "tailwindcss";
```

### 폴더 구조 생성

```bash
mkdir -p src/components/{layout,ui,features}
mkdir -p src/pages
mkdir -p src/lib
mkdir -p src/hooks
mkdir -p src/services
mkdir -p src/stores
mkdir -p src/styles
```

### 기본 라우터 설정 (src/Router.tsx)

```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// 페이지 import (설계서 기반)
import Dashboard from './pages/Dashboard';
// ...

export default function Router() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        {/* 설계서의 페이지 구조에 따라 추가 */}
      </Routes>
    </BrowserRouter>
  );
}
```

---

## React + Vite + Firebase

위 React + Vite 기본 설정에 Firebase를 추가한 버전.

### 추가 설치

```bash
npm install firebase
```

### Firebase 설정 파일 (src/lib/firebase.ts)

```typescript
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
```

### .env.example

```bash
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
```

### Firebase Hosting 설정 (firebase.json)

```json
{
  "hosting": {
    "public": "dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      { "source": "**", "destination": "/index.html" }
    ]
  }
}
```

### 배포 명령어

```bash
npm run build
firebase deploy --only hosting
```

---

## Next.js + TypeScript + Tailwind

SEO가 필요하거나, 서버 사이드 기능이 필요할 때.

### 생성 명령어

```bash
npx create-next-app@latest [프로젝트명] \
  --typescript \
  --tailwind \
  --app \
  --src-dir \
  --import-alias "@/*"
cd [프로젝트명]

# 추가 패키지
npm install lucide-react    # 아이콘
npm install zustand          # 상태관리 (필요시)
```

### 폴더 구조 생성

```bash
mkdir -p src/components/{layout,ui,features}
mkdir -p src/lib
mkdir -p src/hooks
mkdir -p src/services
mkdir -p src/stores
```

### 페이지 구조 (App Router)

```
src/app/
├── layout.tsx              # 루트 레이아웃
├── page.tsx                # 메인 페이지
├── login/
│   └── page.tsx            # 로그인
├── dashboard/
│   └── page.tsx            # 대시보드
├── [entity]/
│   ├── page.tsx            # 목록
│   ├── [id]/
│   │   └── page.tsx        # 상세
│   └── new/
│       └── page.tsx        # 등록
└── settings/
    └── page.tsx            # 설정
```

### 루트 레이아웃 기본 (src/app/layout.tsx)

```typescript
import type { Metadata } from 'next';
import './globals.css';

export const metadata: Metadata = {
  title: '[프로젝트명]',
  description: '[설명]',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  );
}
```

---

## Next.js + Supabase

풀스택 + 관계형 DB가 필요할 때.

### 추가 설치

```bash
npm install @supabase/supabase-js
npm install @supabase/ssr       # 서버 사이드 인증용
```

### Supabase 설정 파일 (src/lib/supabase.ts)

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseKey);
```

### .env.example

```bash
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

---

## HTML + Tailwind CDN (정적 사이트)

가장 단순한 구성. 빌드 과정 없이 바로 사용.

### 기본 index.html

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[프로젝트명]</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <script src="js/main.js"></script>
</body>
</html>
```

### 폴더 구조

```bash
mkdir -p css js images
touch index.html css/styles.css js/main.js
```

---

## 공통: 설정 후 즉시 확인

어떤 스택이든 초기 설정 후 반드시 실행하여 확인:

```bash
# 개발 서버 실행
npm run dev

# 브라우저에서 확인
# - 기본 페이지가 표시되는가
# - Tailwind 클래스가 적용되는가
# - 콘솔에 에러가 없는가
```

에러가 있으면 즉시 수정하고, 정상 작동 확인 후 다음 단계로 넘어간다.
