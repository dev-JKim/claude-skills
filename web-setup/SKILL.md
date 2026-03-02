---
name: web-setup
description: 웹 프로젝트 초기 설정 스킬. 기획이 끝난 후 실제 코드를 작성하기 전에 기술 스택 선택, 프로젝트 생성, 폴더 구조 세팅, CLAUDE.md 작성까지 한 번에 처리합니다. 사용자가 "프로젝트 세팅", "초기 설정", "개발 환경 구축", "프로젝트 시작", "CLAUDE.md 만들어줘", "기술 스택 정하자", "세팅해줘" 등을 언급하면 반드시 이 스킬을 사용하세요. web-planning 스킬로 만든 설계서가 있으면 그 내용을 기반으로 자동 진행합니다.
---

# Web Setup — 프로젝트 초기 설정 스킬

이 스킬의 목적: 기획이 끝난 웹 프로젝트를 **바로 개발 시작할 수 있는 상태**로 만든다.
기획서(설계서)가 있으면 거기서 정보를 가져오고, 없으면 최소한의 질문으로 파악한다.

---

## 진행 순서

```
Step 1: 설계서 확인 + 기술 스택 확정
Step 2: 프로젝트 생성 + 폴더 구조 세팅
Step 3: 공통 설정 파일 생성
Step 4: CLAUDE.md 작성
Step 5: 초기 설정 검증
```

---

## Step 1: 설계서 확인 + 기술 스택 확정

### 1-1. 설계서 확인

먼저 프로젝트 루트에 설계서 파일이 있는지 확인한다:
- `PROJECT_DESIGN.md`, `설계서.md`, 또는 유사한 파일

있으면 → 설계서에서 기술 스택, 페이지 구조, DB 등을 추출하여 확인받는다.
없으면 → 아래 질문으로 최소 정보를 수집한다.

### 1-2. 기술 스택 결정

설계서가 없거나 기술 스택이 미정인 경우, 아래 기준으로 추천한다:

**프레임워크 선택 기준:**

| 프로젝트 유형 | 추천 스택 | 이유 |
|--------------|-----------|------|
| 사내 관리 도구 (데이터 CRUD) | React + Vite + TypeScript | 빠른 개발, SPA로 충분 |
| 사내 도구 + 인증/DB 내장 | React + Vite + Firebase | Auth, Firestore 즉시 사용 |
| SEO 필요한 공개 사이트 | Next.js + TypeScript | SSR/SSG 지원 |
| 단순 소개 페이지 (1~3페이지) | HTML + Tailwind CDN | 가장 가볍고 빠름 |
| 풀스택 + SQL DB 필요 | Next.js + Supabase | PostgreSQL + 인증 내장 |

**스타일링:**
- 기본 추천: Tailwind CSS (유틸리티 기반, 빠른 개발)
- 대안: CSS Modules (컴포넌트 격리 필요 시)

**상태 관리:**
- 소규모 (페이지 5개 이하): React 내장 useState/useContext
- 중규모 (페이지 10개+): Zustand (가볍고 단순)
- 서버 데이터 중심: TanStack Query (캐싱, 동기화)

**UI 컴포넌트 라이브러리 (선택):**
- shadcn/ui: 커스텀 자유도 높음, Tailwind 기반
- 없이 직접 만들기: 완전한 제어, 학습에 좋음

사용자에게 추천 이유를 설명하고 확정받는다.

### 1-3. 확정 사항 정리

스택이 결정되면 한 번에 정리하여 보여준다:

```
✅ 기술 스택 확정

- 프레임워크: React + Vite
- 언어: TypeScript
- 스타일링: Tailwind CSS
- 상태관리: Zustand
- DB: Firebase Firestore
- 인증: Firebase Auth
- 배포: Firebase Hosting
- 패키지 매니저: npm
```

---

## Step 2: 프로젝트 생성 + 폴더 구조 세팅

기술 스택이 확정되면 프로젝트를 생성한다.

→ 스택별 상세 생성 명령어와 폴더 구조는 `references/stack-setup.md` 참조

### 공통 폴더 구조 원칙

어떤 스택이든 아래 원칙을 따른다:

```
src/
├── components/          # 재사용 컴포넌트
│   ├── layout/          # Header, Sidebar, Footer 등 레이아웃
│   ├── ui/              # Button, Card, Input 등 기본 UI
│   └── features/        # 기능별 컴포넌트 (거래 관련, 신고서 관련 등)
├── pages/ (또는 app/)   # 페이지 컴포넌트
├── lib/                 # 유틸리티, 헬퍼 함수
│   ├── utils.ts         # 공통 유틸 (날짜 포맷, 숫자 포맷 등)
│   ├── constants.ts     # 상수 (상태값, 메뉴 항목 등)
│   └── types.ts         # TypeScript 타입 정의
├── hooks/               # 커스텀 훅 (useAuth, useTrades 등)
├── services/            # API 호출, DB 접근 로직
├── stores/              # 상태관리 (Zustand 사용 시)
└── styles/              # 글로벌 스타일
```

**폴더 구조 결정 시 확인:**
- 설계서의 페이지 목록을 기반으로 pages/ 하위 파일 목록을 미리 잡는다
- 설계서의 엔티티를 기반으로 services/ 하위 파일 목록을 미리 잡는다
- 빈 파일이라도 미리 생성해두면 이후 작업이 수월하다

---

## Step 3: 공통 설정 파일 생성

프로젝트 생성 후 반드시 세팅해야 할 공통 파일들:

### 3-1. 환경변수 (.env)

```bash
# .env.example (커밋용 — 실제 값 없이 키만)
VITE_API_URL=
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_PROJECT_ID=

# .env.local (실제 값 — .gitignore에 포함)
VITE_API_URL=https://...
VITE_FIREBASE_API_KEY=실제키값
```

반드시 `.env.local`이 `.gitignore`에 포함되어 있는지 확인한다.

### 3-2. .gitignore 확인

기본 생성된 .gitignore에 아래 항목이 있는지 점검:

```
node_modules/
dist/
.env.local
.env.*.local
*.log
.DS_Store
```

### 3-3. 기본 타입 정의 (TypeScript 사용 시)

설계서의 데이터 구조를 기반으로 `src/lib/types.ts`를 미리 작성한다:

```typescript
// 설계서 엔티티를 TypeScript 타입으로 변환
export interface Trade {
  id: string;
  date: string;
  type: 'import' | 'export';
  partner: string;
  amount: number;
  currency: string;
  status: 'pending' | 'completed' | 'cancelled';
  createdAt: Date;
  updatedAt: Date;
}
```

### 3-4. 기본 유틸리티 (src/lib/utils.ts)

자주 쓰이는 유틸을 미리 만들어둔다:

```typescript
// 날짜 포맷: 2024.01.15
export function formatDate(date: Date | string): string { ... }

// 숫자 포맷: 1,234,567
export function formatNumber(num: number): string { ... }

// 통화 포맷: $1,234.56 / ₩1,234
export function formatCurrency(num: number, currency: string): string { ... }
```

### 3-5. 상수 파일 (src/lib/constants.ts)

설계서에서 반복 사용되는 값을 상수로 정의:

```typescript
// 상태값
export const TRADE_STATUS = {
  PENDING: 'pending',
  COMPLETED: 'completed',
  CANCELLED: 'cancelled',
} as const;

// 메뉴 항목 (네비게이션용)
export const MENU_ITEMS = [
  { label: '대시보드', path: '/dashboard', icon: 'LayoutDashboard' },
  { label: '거래 관리', path: '/trades', icon: 'FileText' },
  ...
];
```

---

## Step 4: CLAUDE.md 작성

CLAUDE.md는 이후 모든 코딩 작업의 규칙서다.
설계서와 위에서 결정한 내용을 종합하여 작성한다.

### CLAUDE.md 템플릿

```markdown
# [프로젝트명]

## 프로젝트 개요
[설계서의 1-1 배경과 목적에서 가져온 한 줄 설명]

## 기술 스택
- 프레임워크: [확정된 프레임워크]
- 언어: [TypeScript / JavaScript]
- 스타일링: [Tailwind CSS 등]
- 상태관리: [Zustand / Context 등]
- DB: [Firestore / Supabase 등]
- 인증: [Firebase Auth / Supabase Auth 등]
- 배포: [Firebase Hosting / Vercel 등]

## 코딩 규칙
- 컴포넌트는 함수형(FC)으로만 작성
- 파일명: PascalCase (컴포넌트), camelCase (유틸/훅)
- UI 텍스트는 한국어
- 코드 주석은 한국어
- console.log는 개발 완료 후 제거
- any 타입 사용 금지, 반드시 타입 정의

## 폴더 구조
[Step 2에서 확정된 폴더 구조]

## 디자인 규칙
- 컬러: Primary [#코드], Success [#코드], Warning [#코드], Danger [#코드]
- 카드: rounded-lg shadow-sm p-6
- 버튼: rounded-md, hover 시 opacity 변화
- 테이블: 짝수행 배경 bg-gray-50, 헤더 bg-gray-100
- 입력 필드: border-gray-300, focus:ring-primary
- 간격: Tailwind 기본 spacing (p-4, gap-4, mb-6 등)

## 반응형 규칙
- sm: 640px / md: 768px / lg: 1024px / xl: 1280px
- 모바일에서 사이드바 → 햄버거 메뉴
- 모바일에서 테이블 → 가로 스크롤 또는 카드형
- 터치 대상 최소 44px

## 상태 표시 규칙
- 로딩: 스켈레톤 UI 사용 (스피너 아님)
- 빈 상태: 안내 문구 + 행동 유도 버튼
- 에러: 에러 메시지 + 다시 시도 버튼
- 성공: 토스트 알림 (3초 후 자동 사라짐)

## 보안 규칙
- API 키, 비밀번호는 환경변수(.env.local)에만 저장
- 모든 페이지는 인증 확인 후 접근
- 사용자 입력값은 반드시 검증
- XSS 방지: dangerouslySetInnerHTML 사용 금지

## 코드 품질 규칙
- 변수명/함수명: camelCase, 의미를 알 수 있는 이름 사용
  - ✅ tradeList, handleSubmit, isLoading
  - ❌ data1, fn, flag, temp
- 컴포넌트명: PascalCase, 역할이 드러나는 이름
  - ✅ TradeListTable, SearchFilter, DeleteConfirmModal
  - ❌ Table1, MyComponent, Comp
- boolean 변수: is/has/can/should 접두사
  - ✅ isLoading, hasPermission, canEdit
  - ❌ loading, permission, edit (명사와 혼동)
- 상수: UPPER_SNAKE_CASE
  - ✅ MAX_PAGE_SIZE, TRADE_STATUS
- 중복 코드 금지: 3회 이상 반복되면 함수/컴포넌트로 분리
- 한 함수는 한 가지 역할만, 50줄 이내 권장
- 한 파일은 300줄 이내 권장, 넘으면 분리 검토

## 한국어 문구 규칙
- 높임말 통일: "~합니다" 체 사용 (또는 프로젝트에서 정한 체)
- 버튼: 동사형 ("저장", "삭제", "등록", "검색")
- 성공 메시지: "[대상]이(가) [행동]되었습니다."
- 에러 메시지: "[대상]을(를) [행동]하지 못했습니다."
- 확인 메시지: "[대상]을(를) [행동]하시겠습니까?"
- 빈 상태: "[대상]이(가) 없습니다. [행동 유도]"
- 맞춤법 주의: "되" vs "돼" 구분, 띄어쓰기 준수
- 외래어 표기 통일: "데이터" (데이타❌), "로그인" (로그 인❌)

## 주석 규칙
- 한국어로 작성
- "왜" 이렇게 했는지 설명 (코드 자체가 "무엇"을 설명)
- 파일 상단: 이 파일의 역할 한 줄 설명
- 함수 상단: 복잡한 로직일 때만 설명 추가
- TODO 주석: // TODO: [내용] — 반드시 배포 전 해결

## 작업 규칙
- 한 번에 하나의 기능만 구현
- 새 컴포넌트는 기존 컴포넌트 스타일과 일관성 유지
- 하드코딩 금지, 반복되는 값은 constants.ts에 정의
- 매 기능 완료 후 사용자 확인 받기
```

### CLAUDE.md 작성 시 주의

- 설계서에서 정한 디자인 컬러, 레이아웃 패턴을 반드시 반영한다
- 설계서의 보안 체크리스트를 보안 규칙으로 변환한다
- 설계서의 에러 처리 방침을 상태 표시 규칙으로 변환한다
- 너무 길면 오히려 무시될 수 있으므로, 핵심만 간결하게 유지한다

---

## Step 5: 초기 설정 검증

모든 설정이 끝나면 아래를 확인한다:

### 검증 체크리스트

```
□ 개발 서버가 정상 실행되는가 (npm run dev)
□ 브라우저에서 기본 페이지가 표시되는가
□ Tailwind 클래스가 정상 적용되는가
□ TypeScript 에러가 없는가
□ .env.example이 존재하는가
□ .env.local이 .gitignore에 포함되었는가
□ CLAUDE.md가 프로젝트 루트에 있는가
□ 폴더 구조가 설계대로 생성되었는가
□ 기본 타입 정의(types.ts)가 생성되었는가
□ 라우터 설정이 페이지 구조와 일치하는가
```

모든 항목을 통과하면 사용자에게 알린다:

```
✅ 초기 설정 완료!

생성된 파일:
- CLAUDE.md (프로젝트 규칙서)
- src/lib/types.ts (데이터 타입)
- src/lib/constants.ts (상수)
- src/lib/utils.ts (유틸리티)
- .env.example (환경변수 템플릿)

다음 단계: 공통 레이아웃 (Header, Sidebar) 구현
```

---

## 주의사항

- 이 스킬은 설계서 기반으로 진행하는 것이 가장 효과적이다. web-planning 스킬을 먼저 거치는 것을 권장한다.
- 설계서가 없어도 진행 가능하지만, 최소한 핵심 기능과 페이지 구조는 확인받는다.
- 프로젝트 생성 명령어는 스택마다 다르므로, 반드시 `references/stack-setup.md`를 참조한다.
- CLAUDE.md는 한 번 만들고 끝이 아니라, 개발 진행하면서 필요 시 업데이트한다.
