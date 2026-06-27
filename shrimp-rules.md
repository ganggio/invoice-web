# Development Guidelines (invoice-web)

> AI Agent 전용 운영 규칙. 일반 개발 상식이 아닌 **invoice-web 프로젝트에서만 통용되는 규칙·금지사항·동기화 의무·결정 기준**을 정의한다.
> 본 문서를 위반하는 결과물은 즉시 폐기·재작업 대상이다.

---

## 프로젝트 개요

- **목적**: Notion 견적서 DB를 단일 데이터 소스로 사용하여 `/q/[slug]` URL로 클라이언트에게 공유하고, 서버에서 한글 폰트가 임베드된 PDF를 즉시 다운로드 제공한다.
- **사용자**: 1인 프리랜서·소규모 스튜디오(발행자) + 링크 수신 클라이언트(가입 불필요).
- **MVP 범위**: F001(Notion 연동), F002(웹 뷰), F003(PDF 다운로드), F010(에러 UI), F011(랜딩). 발행자용 관리 UI·인증·다중 발행자는 **MVP 범위 밖**.
- **기술 스택 고정값** (변경 금지):
  - Next.js `15.5.3` (App Router + Turbopack)
  - React `19.1.0`
  - TypeScript `5`
  - TailwindCSS `v4` + shadcn/ui `new-york`
  - 외부 SDK: `@notionhq/client`, `@react-pdf/renderer`
  - Logger: `pino` (+ 개발 환경 `pino-pretty`)
  - Forms: `react-hook-form` + `zod`
  - 호스팅: Vercel

---

## 현재 코드베이스 스냅샷 (2026-06-27 기준)

> **AI는 작업 시작 전 이 스냅샷과 실제 파일 상태를 반드시 대조한다. 불일치 시 실제 파일을 우선하고 이 섹션을 즉시 갱신한다.**

### 완료 상태

- **ROADMAP Task 001**(스타터 정리 및 의존성 설치) **완료**. 그 외 Task 002 ~ Task 024는 **모두 미완료**.
- `package.json` 의존성은 PRD 요구 패키지(`@notionhq/client`, `@react-pdf/renderer`, `nanoid`, `pino`, `pino-pretty`, `zod`, `sonner` 등)를 모두 포함한다. **`npm install`로 신규 패키지 추가 금지** (PRD 명시 외).
- `src/components/ui/` 에 shadcn 컴포넌트 18종이 이미 설치됨. 중복 추가 금지.

### 미동기화·플레이스홀더

- `src/lib/env.ts` 는 **스타터 잔재 스키마**(`NODE_ENV`, `VERCEL_URL`, `NEXT_PUBLIC_APP_URL`)를 그대로 두고 있다. **Task 002 진행 시 PRD 명세(`NOTION_TOKEN`, `NOTION_QUOTE_DB_ID`, `NOTION_ITEM_DB_ID?`)로 전면 교체**한다. 기존 키와 병합 금지.
- `src/app/page.tsx` 는 단순 텍스트 placeholder. Task 020(랜딩 페이지)에서 `src/components/home/*` 섹션으로 교체.
- `src/lib/notion/`, `src/components/quote/`, `src/components/pdf/`, `src/components/home/`, `tasks/`, `public/fonts/` 디렉토리는 **아직 존재하지 않는다.** 해당 Task에서 신규 생성한다.
- `src/app/q/[slug]/{page,loading,not-found,error}.tsx`, `src/app/api/q/[slug]/pdf/route.ts` 는 아직 없다. Task 002에서 골격을 만든다.

### 기 작성된 인프라

- `src/app/layout.tsx` — `<html lang="ko" suppressHydrationWarning>`, `ThemeProvider`, `Toaster` 마운트 완료. **수정 금지** (메타데이터·폰트 외).
- `src/components/providers/theme-provider.tsx` — `next-themes` 통합 완료. 추가 Provider 신설 시 같은 위치.
- `src/components/layout/container.tsx` — 공통 Container 존재. 페이지 골격에 재사용.
- `src/components/theme-toggle.tsx` — 다크모드 토글 완료.
- `src/lib/utils.ts` — `cn()` 헬퍼만 존재. 다른 유틸은 별도 파일로 분리(예: `src/lib/format.ts`, `src/lib/logger.ts`).

---

## 프로젝트 아키텍처

### 디렉토리 권한 매트릭스

| 경로                                | 책임                                                                                                                                            | 추가 금지 항목                     |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| `src/app/`                          | App Router 라우트 (`page.tsx`, `layout.tsx`, `loading.tsx`, `not-found.tsx`, `error.tsx`, `route.ts`)                                           | 비즈니스 로직·Notion SDK 직접 호출 |
| `src/app/api/q/[slug]/pdf/route.ts` | PDF 스트림 응답 전용 GET 핸들러                                                                                                                 | Notion adapter/service 외 로직     |
| `src/lib/notion/`                   | Notion 도메인 레이어 (`client.ts`, `adapter.ts`, `quote.service.ts`, `cache.ts`, `types.ts`, `errors.ts`)                                       | 클라이언트 컴포넌트가 import 금지  |
| `src/lib/logger.ts`                 | pino 인스턴스 (서버 전용)                                                                                                                       | 클라이언트에서 import 금지         |
| `src/lib/format.ts`                 | `formatCurrency` 등 isomorphic 유틸                                                                                                             | 서버 전용 API 호출                 |
| `src/lib/env.ts`                    | Zod 환경변수 검증                                                                                                                               | 런타임 분기 로직                   |
| `src/components/ui/`                | shadcn/ui 재사용 컴포넌트                                                                                                                       | 도메인 비즈니스 로직               |
| `src/components/quote/`             | 견적서 도메인 UI (`QuoteHeader`, `QuoteClientCard`, `QuoteItemsTable`, `QuoteTotals`, `QuoteMemo`, `QuoteInlineWarning`, `QuoteDownloadButton`) | shadcn 컴포넌트 자체 정의          |
| `src/components/pdf/`               | `@react-pdf/renderer` 트리 (`quote-pdf.tsx`, `fonts.ts`)                                                                                        | 웹 DOM 컴포넌트 혼용               |
| `src/components/home/`              | 랜딩 페이지 섹션 (`Hero`, `Features`, `NotionSetupGuide`, `DbSchemaTable`)                                                                      | `/q/[slug]` 전용 컴포넌트          |
| `src/components/layout/`            | `Container` 등 페이지 골격                                                                                                                      | 도메인 컴포넌트                    |
| `src/components/providers/`         | React Context 프로바이더 (`ThemeProvider`)                                                                                                      | 비-Provider 로직                   |
| `public/fonts/`                     | `Pretendard-Regular.ttf`, `Pretendard-Bold.ttf` (PDF 임베드용)                                                                                  | 웹폰트 외 다른 폰트                |
| `docs/`                             | `PRD.md`, `ROADMAP.md`, `guides/`                                                                                                               | 임시 메모·작업 로그                |
| `tasks/`                            | `XXX-description.md` 형식 작업 명세                                                                                                             | 형식 외 파일                       |

### 의무 라우트 골격

다음 라우트는 **반드시 모두 존재**해야 한다. 누락 시 ROADMAP Task 002 미완료로 간주.

```
src/app/
├── page.tsx                          # / (홈 — 색인 허용)
├── q/
│   └── [slug]/
│       ├── page.tsx                  # Server Component, generateMetadata에 noindex,nofollow 필수
│       ├── loading.tsx               # shadcn Skeleton
│       ├── not-found.tsx             # 404 (410 흡수)
│       └── error.tsx                 # 'use client' 강제
└── api/
    └── q/
        └── [slug]/
            └── pdf/
                └── route.ts          # GET only
```

### 레이어 호출 방향 (역방향 호출 금지)

```
Route (page.tsx / route.ts)
        ↓
Service (src/lib/notion/quote.service.ts)
        ↓
Adapter (src/lib/notion/adapter.ts)
        ↓
Client  (src/lib/notion/client.ts)  →  Notion SDK
```

- Route가 Adapter를 직접 호출하지 말 것. 반드시 Service를 경유한다.
- Adapter는 Notion raw 타입을 외부로 노출하지 않는다(반환 타입은 `Quote | null` 또는 throw).
- Service는 Notion SDK 타입을 외부에 노출하지 않는다.

---

## 코드 표준

### 언어·식별자 규칙

- 사용자 응답·주석·문서·커밋 메시지: **한국어**.
- 변수/함수/타입/파일명: **영어**.
- 파일명: `kebab-case.tsx` (예: `quote-header.tsx`). **`snake_case` 금지**.
- 컴포넌트 식별자: `PascalCase` (예: `QuoteHeader`).
- 상수: `SCREAMING_SNAKE_CASE` (예: `DEFAULT_TAX_RATE`).
- 들여쓰기: 스페이스 2칸. 세미콜론 미사용(prettier `semi: false`). 작은따옴표(`singleQuote: true`).

### 설정 파일 잠금값 (변경 시 사용자 명시 승인 필수)

| 파일                        | 잠금 키·값                                                                                                                                              |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.prettierrc`               | `semi: false`, `singleQuote: true`, `tabWidth: 2`, `useTabs: false`, `trailingComma: 'es5'`, `arrowParens: 'avoid'`, `printWidth: 80`, `endOfLine: lf`, `plugins: ['prettier-plugin-tailwindcss']` |
| `tsconfig.json`             | `strict: true`, `paths: { '@/*': ['./src/*'] }`, `moduleResolution: 'bundler'`, `jsx: 'preserve'`                                                       |
| `components.json`           | `style: 'new-york'`, `rsc: true`, `tailwind.baseColor: 'neutral'`, `tailwind.cssVariables: true`, `iconLibrary: 'lucide'`, alias 5종                    |
| `next.config.ts`            | `poweredByHeader: false`, `compress: true`, `images.formats: ['image/webp', 'image/avif']`, `experimental.optimizePackageImports: ['lucide-react']`, 보안 헤더 4종(`X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: origin-when-cross-origin`, `X-XSS-Protection: 1; mode=block`) |
| `eslint.config.mjs`         | `next/core-web-vitals` + `next/typescript` + `prettier` 확장 유지. 룰 비활성화 금지.                                                                    |
| `.husky/pre-commit`         | `lint-staged` 실행. 우회/제거 금지.                                                                                                                     |
| `src/app/layout.tsx`        | `<html lang="ko" suppressHydrationWarning>`, `ThemeProvider`(`attribute="class" defaultTheme="system" enableSystem disableTransitionOnChange`), `<Toaster />` |

위 키·값을 변경해야 한다고 판단되면 **변경하지 말고 사용자에게 사유와 영향 범위를 먼저 보고**한다.

### 금지 패턴 (위반 시 즉시 수정)

- `any` 타입 사용 금지. 불가피한 경우에도 `unknown` + 타입 가드를 사용한다.
- `console.log` / `console.error` / `console.warn` 사용 금지. **반드시 `src/lib/logger.ts`의 pino 인스턴스만 사용**.
- 매직 넘버 금지. 모든 수치는 상수로 분리한다.
  - 예시: `DEFAULT_TAX_RATE = 10`, `DEFAULT_CURRENCY = 'KRW'`, `KRW_DECIMALS = 0`, `USD_DECIMALS = 2`, `SUCCESS_RESET_MS = 2000`, `MAX_CACHE_TTL_MS = 10 * 60 * 1000`, `ISR_REVALIDATE_SECONDS = 300`.
- 인라인 스타일 (`style={{...}}`) 금지. Tailwind 유틸리티만 사용.
- 하드코딩 색상 (`bg-white`, `text-black`, `bg-gray-100` 등) 금지. **시맨틱 토큰 (`bg-background`, `text-foreground`, `text-muted-foreground`, `bg-primary`, `bg-destructive` 등)만 사용**.
- 상대 경로 import (`../../lib/...`) 금지. **`@/...` 경로 별칭만 사용**.
- `pages/` 디렉토리·`getServerSideProps`·`getStaticProps` 금지. App Router 전용.
- 불필요한 `'use client'` 금지. 상호작용/상태가 없으면 Server Component 유지.
- `tasks/` 또는 `docs/` 외 위치에 임의의 `.md` 파일 생성 금지 (사용자 명시적 요청 없는 한).

### Server / Client 경계

- **기본은 Server Component**. `'use client'`는 다음 조건에서만 허용:
  - React 19 훅(`useState`, `useEffect`, `useActionState`, `useFormStatus`) 사용
  - 이벤트 핸들러 직접 바인딩
  - `next-themes`, Sonner 등 클라이언트 전용 라이브러리 사용
- 서버 전용 모듈은 파일 최상단에 `import 'server-only'`를 **반드시 선언**한다. 대상: `src/lib/logger.ts`, `src/lib/notion/client.ts`, `src/lib/notion/quote.service.ts`, `src/lib/notion/adapter.ts`, `src/lib/notion/cache.ts`.
- `error.tsx`는 Next.js 규칙에 따라 `'use client'` 강제.

### Next.js 15 비동기 params

- `params`와 `searchParams`는 **반드시 `Promise<...>`로 선언**하고 `await`로 풀어 쓴다.

  ```ts
  // ✅ 필수
  export default async function Page({
    params,
  }: {
    params: Promise<{ slug: string }>
  }) {
    const { slug } = await params
  }
  ```

### Tailwind 클래스 결합

- 조건부/다중 클래스는 **반드시 `cn()` (`@/lib/utils`) 헬퍼를 사용**한다. 문자열 템플릿(`` `${a} ${b}` ``)으로 결합 금지.

---

## 기능 구현 표준

### 도메인 타입 (`src/lib/notion/types.ts`)

- 단일 소스 오브 트루스. 다음 타입은 **이 파일에서만 정의**하고, 컴포넌트·Service·PDF가 동일하게 import 한다.
  - `Currency = 'KRW' | 'USD'`
  - `QuoteLineItem` (`id`, `name`, `description?`, `quantity`, `unitPrice`, `amount`)
  - `Quote` (`id`, `slug`, `quoteNumber`, `issuer`, `client`, `issuedAt`, `validUntil`, `items`, `taxRate`, `currency`, `memo?`, `subtotal`, `tax`, `total`)
- 모든 필드에 한국어 JSDoc 주석. 영문 주석 금지.
- 같은 타입을 `src/components/pdf/` 또는 `src/components/quote/`에서 **재정의 금지**. 항상 import.

### 금액 계산 규칙

- `subtotal = Σ(quantity × unitPrice)`, `tax = subtotal × taxRate / 100`, `total = subtotal + tax`.
- **Notion Formula 필드의 `합계` 값은 신뢰하지 않는다.** Adapter가 서버에서 재계산하여 `Quote.subtotal`, `Quote.tax`, `Quote.total`을 채운다.
- 세율 미설정 시 `DEFAULT_TAX_RATE`(10) 적용. 통화 미설정 시 `DEFAULT_CURRENCY`('KRW') 적용.

### Notion API 호출 규칙

- 클라이언트 인스턴스는 `src/lib/notion/client.ts` 싱글톤만 사용 (`new Client()` 추가 생성 금지).
- 호출 실패 시 **반드시 1회 지수 백오프 재시도(500ms)** 후 `NotionApiError`로 wrap하여 throw. 무한 재시도 금지.
- `getQuoteBySlug(slug)`는 `Quote | null`만 반환. raw `PageObjectResponse` 노출 금지.
- 필수 필드 누락 시 `QuoteFieldMissingError`를 누락 필드 배열과 함께 throw → `error.tsx`가 처리.

### 캐싱 정책

- `/q/[slug]/page.tsx`에 `export const revalidate = ISR_REVALIDATE_SECONDS` (300초) 적용.
- `src/lib/notion/cache.ts`의 인메모리 LRU(`MAX_CACHE_TTL_MS = 10 * 60 * 1000`, 최대 100건)와 2단 구성.
- **실패 응답은 캐시에 저장 금지**. 다음 요청에서 재시도되어야 한다.
- 캐시 키: `quote:${slug}`.

### SEO·메타 정책

| 경로           | robots                    | OG/Twitter                                                    |
| -------------- | ------------------------- | ------------------------------------------------------------- |
| `/` (홈)       | `index,follow`            | OG 이미지(`/og-image.png`, 1200×630), title, description 필수 |
| `/q/[slug]`    | `noindex,nofollow` (필수) | OG 태그 추가 금지 (개인정보 노출 방지)                        |
| `404`, `error` | `noindex,nofollow`        | —                                                             |

- `/q/[slug]/page.tsx`의 `generateMetadata`에서 `robots: { index: false, follow: false }`를 **반드시** 반환한다.

### PDF 응답 규약

- 라우트: `GET /api/q/[slug]/pdf`. POST/PUT/DELETE 추가 금지.
- 응답 헤더:
  - `Content-Type: application/pdf`
  - `Content-Disposition: attachment; filename*=UTF-8''<RFC5987-encoded>`
- 파일명 패턴: `견적서_{client.name}_{issuedAt}.pdf` (반드시 `encodeURIComponent` 적용).
- 폰트: `public/fonts/Pretendard-Regular.ttf`, `public/fonts/Pretendard-Bold.ttf`를 `src/components/pdf/fonts.ts`에서 `Font.register`. 다른 한글 폰트로 대체 금지.
- 페이지 사이즈 A4, 여백 40pt. 항목 50건 초과 시 자동 분할.
- `formatCurrency` (Task 005)를 웹과 PDF 양쪽에서 동일하게 재사용. PDF에 별도 포맷 함수 작성 금지.

### PDF 다운로드 버튼 상태 머신

- 4단계 고정: `idle` → `loading` → `success` → `idle`(2초 후) 또는 `error`.
- `SUCCESS_RESET_MS = 2000` 상수 사용. 다른 값 금지.
- `error` 상태는 **Sonner 토스트로 원인 메시지를 반드시 노출**한다.
- `URL.createObjectURL` 사용 후 페이지 언마운트 시 `URL.revokeObjectURL` 호출 의무 (메모리 누수 방지).
- 모든 상태에서 `aria-label` 유지.

### 반응형 규약

- 견적서 페이지 최상위 컨테이너: `max-w-[880px] mx-auto`.
- 항목 테이블: 모바일(`<768px`)에서 카드 리스트로 자동 전환.
- 검증 의무 뷰포트 3종: `320×568`, `768×1024`, `1024×768`.

### 폼·검증

- 폼은 `react-hook-form` + `zod` + `@hookform/resolvers/zod` 조합만 사용.
- 서버 액션에서도 **동일한 Zod 스키마로 재검증**. 클라이언트만 검증 금지.

### 테마

- 색상은 `src/app/globals.css`의 OKLCH 변수 + Tailwind 시맨틱 토큰을 통해서만 사용.
- `next-themes`는 `src/components/providers/theme-provider.tsx`를 통해서만 통합. `attribute="class"` 고정.
- `<html lang="ko" suppressHydrationWarning>` 유지.

---

## 라이브러리·외부 의존성 사용 표준

### 아이콘

- `lucide-react`만 사용. 다른 아이콘 라이브러리 추가 금지. `optimizePackageImports`에 등록되어 있음(`next.config.ts`).

### 토스트

- `sonner`만 사용. `src/components/ui/sonner.tsx`의 `<Toaster />`가 `app/layout.tsx`에 마운트되어 있다. 추가 토스트 라이브러리 금지.

### shadcn 컴포넌트 추가

- 항상 CLI로 추가: `npx shadcn@latest add <name>`. 수동 작성 금지.
- 추가된 컴포넌트는 `src/components/ui/`에 위치. 다른 경로 이동 금지.
- 이미 설치된 컴포넌트: `alert`, `avatar`, `badge`, `button`, `card`, `checkbox`, `dialog`, `dropdown-menu`, `form`, `input`, `label`, `navigation-menu`, `progress`, `select`, `separator`, `sheet`, `skeleton`, `sonner`. 중복 추가 금지.

### Notion SDK

- `@notionhq/client`의 `Client`만 사용. `fetch`로 Notion REST를 직접 호출 금지(인증·페이지네이션 복잡도 회피).

### PDF 라이브러리

- `@react-pdf/renderer` 고정. `puppeteer`, `playwright-pdf`, `jspdf`, `html2pdf.js`로 대체 금지(Vercel Lambda 용량·한글 호환성 사유).

---

## 워크플로우 표준

### ROADMAP 기반 작업 진행

- 모든 구현 작업은 `docs/ROADMAP.md`의 Task 단위로 진행한다. ROADMAP에 없는 작업을 즉흥적으로 추가하지 않는다.
- 새 Task가 필요하면 ROADMAP에 먼저 추가하고 우선순위 위치(마지막 완료 Task 다음)에 삽입한다.
- Task 진행 시 다음 순서를 따른다.
  1. `tasks/XXX-description.md` 작업 파일 작성 (명세·관련 파일·수락 기준·구현 단계·테스트 체크리스트 포함)
  2. 구현
  3. **Playwright MCP 테스트 수행 (필수, 예외 없음)**
  4. 통과 시 ROADMAP의 해당 Task에 ✅ 표시
  5. 진행 현황 요약 표 동기화

### Playwright MCP 테스트 의무

- **API 연동 / 비즈니스 로직 / UI 인터랙션을 포함한 모든 Task**는 다음 4개 카테고리 시나리오를 작업 파일 `## 테스트 체크리스트`에 작성하고 통과시켜야 한다.
  1. 정상 시나리오 (Happy Path)
  2. 실패·예외 시나리오
  3. 엣지 케이스
  4. 회귀 방지
- 시나리오마다 사용할 도구 명시: `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_fill_form`, `browser_type`, `browser_resize`, `browser_press_key`, `browser_network_requests`, `browser_console_messages`, `browser_evaluate`, `browser_wait_for`, `browser_tabs`, `browser_navigate_back`.
- 표준 절차: `npm run dev` 기동 → `browser_navigate` → `browser_snapshot` → 액션 수행 → `browser_network_requests`로 API 검증 → `browser_console_messages`로 에러 확인 → 통과 시 체크박스 갱신, 실패 시 원인 분석·수정·재테스트.
- **테스트 미통과 Task를 "완료"로 표시 금지**. 실패한 경우 작업 파일에 실패 로그·스크린샷 경로를 남긴다.
- 4-카테고리 면제 대상(최소 시각 검증만 수행): 의존성 설치·환경변수·디렉토리 골격·순수 타입 정의·인프라 유틸·메타데이터·정적 자산·접근성 감사·성능 측정·인프라 배포 Task. ROADMAP의 해당 Task에 명시된 "비고: ... 면제" 문구를 신뢰한다.

### 코드 품질 게이트

- Task 완료 전 다음 명령이 **모두 통과**해야 한다.

  ```bash
  npm run check-all   # typecheck + lint + format:check
  npm run build       # Turbopack 프로덕션 빌드
  ```

- pre-commit 훅(`.husky/pre-commit`)에서 `lint-staged`가 자동 실행됨. 훅을 `--no-verify`로 우회 금지.
- 빌드 실패 또는 ESLint 에러를 남긴 채 커밋 금지.

### 커밋 메시지

- 한국어 + Conventional Commits 형식(`feat:`, `fix:`, `refactor:`, `docs:`, `chore:` 등) 사용.
- 본문에 변경 이유를 한국어로 1~2문장 기록.

### 메타 파일·디렉토리 보호

> 다음 경로는 **사용자가 명시적으로 요청한 경우에만** 수정·생성·삭제한다.

| 경로                         | 처리 원칙                                                                                                  |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `.claude/agents/**`          | 에이전트 정의 파일. 작업 중 자동 수정 금지.                                                                |
| `.claude/commands/**`        | 슬래시 커맨드 정의. 자동 수정 금지.                                                                        |
| `.claude/settings.local.json`| 사용자 로컬 허용 목록. AI가 자동 갱신 금지(권한 추가는 사용자가 결정).                                     |
| `.claude/hooks/**`           | hook 스크립트. 자동 수정 금지.                                                                             |
| `.husky/**`                  | git hook. 자동 수정 금지. `--no-verify`로 우회 금지.                                                       |
| `.vscode/**`                 | 에디터 설정. 자동 수정 금지.                                                                               |
| `.mcp.json`                  | MCP 서버 등록. 자동 수정 금지.                                                                             |
| `shrimp_data/**`             | shrimp-task-manager 자동 생성 영역. 수동 편집 금지. 필요 시 `mcp__shrimp-task-manager__*` 도구만 사용.     |
| `shrimp-rules.md`            | 본 문서. `mcp__shrimp-task-manager__init_project_rules` 호출 또는 사용자 명시 요청 시에만 갱신.            |
| `package-lock.json`          | `npm install` 결과로만 변경. 수동 편집 금지.                                                               |
| `public/og-image.png`, `public/fonts/**` | 자산 파일. 교체 시 PRD·동기화 표 확인.                                                         |

### shrimp-task-manager 운용

- `tasks/XXX-description.md` 작업 명세 파일은 **사람·AI가 함께 읽는 1차 산출물**이다. 반드시 직접 작성한다.
- `shrimp_data/tasks.json` 등 내부 상태는 `mcp__shrimp-task-manager__plan_task`, `split_tasks`, `update_task`, `execute_task`, `verify_task`, `list_tasks`, `query_task` 등으로만 조작한다.
- ROADMAP의 Task ID(예: `Task 002`)와 shrimp-task-manager 내부 task ID는 별개 식별자다. 혼동 시 ROADMAP을 우선한다.

---

## 핵심 파일 동기화 표준

다음 조합 중 하나를 수정할 때 나머지 항목도 **같은 PR에서 동기화**해야 한다.

### 동기화 규칙 표

| 트리거                                                             | 함께 갱신할 대상                                                                                                                                        |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `docs/PRD.md`의 데이터 모델 변경                                   | `src/lib/notion/types.ts`, `src/lib/notion/adapter.ts`, `src/components/quote/*`, `src/components/pdf/quote-pdf.tsx`, `docs/ROADMAP.md` (Task 영향분석) |
| `src/lib/notion/types.ts`의 `Quote` 또는 `QuoteLineItem` 필드 변경 | `src/components/quote/*` (모든 컴포넌트 props 재검토), `src/components/pdf/quote-pdf.tsx`, `src/lib/notion/adapter.ts` 매핑 로직, fixture               |
| 환경변수 추가/삭제                                                 | `src/lib/env.ts` Zod 스키마, `.env.example`, `README.md`의 환경변수 섹션, `docs/PRD.md` "환경변수 및 설정" 섹션                                         |
| 새 shadcn 컴포넌트 추가                                            | `package.json` (의존성), `src/components/ui/<name>.tsx` (자동), `docs/PRD.md` 컴포넌트 매핑 표(해당 시)                                                 |
| 새 라우트 추가                                                     | `docs/PRD.md` "라우트 구조" 섹션, `docs/ROADMAP.md` (Task 영향분석), `src/app/<route>/page.tsx`                                                         |
| ROADMAP Task 완료                                                  | `docs/ROADMAP.md`의 Task 체크박스 + 진행 현황 요약 표 + Phase 완료 기준 체크박스                                                                        |
| 폰트 파일 추가                                                     | `public/fonts/<file>.ttf`, `src/components/pdf/fonts.ts` (`Font.register` 호출), `docs/PRD.md` 폰트 섹션                                                |
| PDF 응답 헤더 변경                                                 | `src/app/api/q/[slug]/pdf/route.ts`, `docs/PRD.md` 예시 코드, `tasks/013-*.md` 테스트 체크리스트                                                        |
| 로깅 구조 변경 (`pino` 필드)                                       | `src/lib/logger.ts`, 모든 호출부 (`src/lib/notion/*`, `src/app/api/q/[slug]/pdf/route.ts`)                                                              |
| 캐시 정책 변경                                                     | `src/app/q/[slug]/page.tsx`의 `revalidate`, `src/lib/notion/cache.ts`의 TTL 상수, `docs/PRD.md` "Rate Limit 대응"                                       |
| `next.config.ts` 보안 헤더·이미지 포맷·번들 옵션 변경              | `next.config.ts`, `docs/PRD.md` "비기능 요구사항"(보안 섹션), `README.md`(노출 시)                                                                      |
| shadcn 컴포넌트 신규 설치                                          | CLI 실행(`npx shadcn@latest add ...`) 결과로 생기는 `src/components/ui/<name>.tsx` + `package.json` + 이 문서의 "이미 설치된 컴포넌트" 목록             |
| `.prettierrc` / `tsconfig.json` / `components.json` 잠금값 변경    | 변경 전 사용자 승인 필수 → 본 문서 "설정 파일 잠금값" 표 동기 갱신                                                                                      |

---

## AI 의사결정 표준

### 새 컴포넌트 위치 결정 트리

```
새 컴포넌트가 필요할 때:
├── shadcn/ui로 즉시 추가 가능한 재사용 부품인가?
│   └── YES → `npx shadcn@latest add <name>` → `src/components/ui/`
├── 견적서 도메인(Quote, QuoteLineItem)을 직접 props로 받는가?
│   ├── 웹 DOM 렌더링 → `src/components/quote/<PascalCase>.tsx`
│   └── PDF 렌더링 (@react-pdf/renderer) → `src/components/pdf/<kebab>.tsx`
├── 홈 랜딩 페이지 전용인가?
│   └── YES → `src/components/home/<PascalCase>.tsx`
├── 페이지 골격/컨테이너인가?
│   └── YES → `src/components/layout/<kebab>.tsx`
└── React Context Provider인가?
    └── YES → `src/components/providers/<kebab>.tsx`
```

### `'use client'` 사용 여부 결정

```
컴포넌트가 다음 중 하나에 해당하는가?
├── useState/useEffect/useReducer 등 React 훅 사용 → 'use client'
├── 이벤트 핸들러를 prop이 아닌 인라인으로 바인딩 → 'use client'
├── next-themes / sonner / Radix Portal 등 클라이언트 라이브러리 사용 → 'use client'
├── Next.js error.tsx 규약 → 'use client' (필수)
└── 위 어디에도 해당하지 않음 → Server Component 유지
```

### 에러 응답 분기 결정

```
요청 처리 중 발생한 상황:
├── slug에 해당하는 Notion 페이지 없음 / 권한 회수 / 삭제됨
│   └── `notFound()` 호출 → `not-found.tsx` (410도 흡수)
├── Notion API 호출 자체가 실패 (4xx/5xx/네트워크)
│   └── `NotionApiError` throw → `error.tsx`가 "Notion 서비스 일시 장애" 분기 노출
├── 필수 필드 누락
│   └── `QuoteFieldMissingError` throw (누락 필드 배열 포함) → `error.tsx`가 "정보가 불완전합니다" + 필드 목록 노출
└── 선택 필드(이메일·연락처·메모·통화) 누락
    └── throw 금지. `QuoteInlineWarning` 컴포넌트로 인라인 경고만 표시
```

### 매직 넘버를 만났을 때

```
숫자 리터럴이 코드에 등장할 때:
├── 단순 수학 식의 일부 (예: 배열 인덱스 0, 1) → 그대로 허용
├── 도메인 의미가 있는가 (세율, 통화 자릿수, 타임아웃, TTL 등)?
│   └── YES → 상수 분리 (관련 모듈의 최상단 또는 `src/lib/constants.ts`)
└── 픽셀/Tailwind 값 → Tailwind 클래스 또는 임의값 (`max-w-[880px]`)으로 표현
```

### 우선순위 충돌 시 판단

다음 순서로 우선한다.

1. **데이터 안전성 / 도메인 정합성** (서버 재계산, Zod 검증, 캐시 정책)
2. **사용자 경험** (에러 화면, 로딩, 접근성)
3. **성능 목표** (LCP < 2.5s, PDF < 5s P95)
4. **코드 가독성·재사용성**
5. **개발 편의성**

상위 항목을 희생해 하위 항목을 만족시키지 않는다.

### 서브에이전트·슬래시 커맨드 위임 매트릭스

> **선언적 작업일수록 전용 에이전트로 위임한다. 위임 가능한 작업을 메인 컨텍스트에서 직접 처리하지 않는다.**

| 상황                                                          | 우선 사용 도구                                                                                 |
| ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `src/app/` 라우트 골격·layout 추가/수정                       | `Agent(subagent_type='nextjs-app-developer')`                                                  |
| `src/components/quote/*`, `src/components/home/*` 등 정적 마크업·스타일링 | `Agent(subagent_type='ui-markup-specialist')`                                       |
| 코드 변경 직후 품질 검토                                      | `Agent(subagent_type='code-reviewer')` — 한국어 리뷰 보고                                      |
| `docs/ROADMAP.md` 재구성·신규 Phase 추가                      | `Agent(subagent_type='development-planner')`                                                   |
| 다파일/다경로 탐색이 3쿼리 이상 필요한 코드 위치 파악         | `Agent(subagent_type='Explore', breadth='medium')`                                             |
| 구현 전략·아키텍처 트레이드오프 설계                          | `Agent(subagent_type='Plan')`                                                                  |
| ROADMAP 진행 갱신                                             | `Skill('docs:update-roadmap')`                                                                 |
| 브라우저 E2E 검증 (Playwright MCP 의무 시나리오)              | `mcp__playwright__browser_*` 도구군                                                            |
| 라이브러리 공식 문서 조회 (Next.js / Notion SDK / react-pdf 등)| `mcp__context7__resolve-library-id` → `mcp__context7__query-docs` (웹 검색보다 우선)          |
| shadcn 컴포넌트 검색·추가                                     | `mcp__shadcn__search_items_in_registries` / `get_add_command_for_items` → CLI 실행             |
| Task 분해·실행 트래킹                                         | `mcp__shrimp-task-manager__plan_task` → `split_tasks` → `execute_task` → `verify_task`         |
| 커밋·브랜치·PR·머지                                           | `Skill('git:commit')`, `Skill('git:branch')`, `Skill('git:pr')`, `Skill('git:merge')`          |
| 백그라운드 자동화·주기 실행                                   | `Skill('loop')`, `Skill('schedule')`                                                           |

#### 위임 금지

- ❌ 단일 파일 단순 조회·grep을 위해 `general-purpose`·`Explore` 에이전트 호출 (직접 `Read`/`Bash`/`Grep` 사용).
- ❌ 동일 검색을 메인과 서브에이전트에서 **중복 수행**.
- ❌ 사용자가 보내지 않은 슬래시 커맨드(`/<name>`)를 임의로 `Skill`로 호출 (사용자 명시 또는 본 매트릭스에 적힌 경우만).

---

## 금지 사항 (Hard No)

### 절대 금지

- ❌ `pages/` 디렉토리 생성 또는 `getServerSideProps` / `getStaticProps` 사용.
- ❌ `console.log` / `console.error` / `console.warn` 추가 (서버·클라이언트 무관). **pino logger 또는 Sonner 토스트만 사용**.
- ❌ `any` 타입 사용. 불가피하면 `unknown` + 타입 가드.
- ❌ Notion Formula 필드의 `합계` 값을 그대로 표시. 항상 서버 재계산.
- ❌ `/q/[slug]` 페이지에 `robots: { index: true }` 또는 OG 이미지 추가.
- ❌ 홈 페이지에 `noindex,nofollow` 적용.
- ❌ `src/lib/notion/*`, `src/lib/logger.ts`를 클라이언트 컴포넌트에서 import. `import 'server-only'` 위반.
- ❌ Notion API 무한 재시도. **반드시 1회 지수 백오프**.
- ❌ PDF 라이브러리 교체(`puppeteer`, `jspdf`, `html2pdf.js` 등).
- ❌ 한글 폰트를 Pretendard 외 다른 폰트로 교체.
- ❌ 인라인 스타일(`style={{...}}`) 또는 하드코딩 색상 클래스(`bg-white`, `text-gray-900` 등) 사용.
- ❌ pre-commit 훅 우회(`git commit --no-verify`).
- ❌ ROADMAP에 없는 Task를 즉흥적으로 구현. (반드시 ROADMAP 먼저 갱신.)
- ❌ 4-카테고리 Playwright MCP 테스트를 건너뛰고 Task를 "완료" 표시.
- ❌ `shadcn/ui` 컴포넌트를 수동으로 작성하거나 다른 디렉토리에 배치.
- ❌ `src/lib/notion/types.ts` 외의 위치에서 `Quote` / `QuoteLineItem` 타입 재정의.
- ❌ 사용자가 명시적으로 요청하지 않은 신규 `.md` 파일 생성 (특히 `README`, `CHANGELOG`, `NOTES` 등).
- ❌ 이모지를 코드/주석/파일에 추가 (사용자가 명시적으로 요청한 경우 외).
- ❌ `npm install`로 PRD에 명시되지 않은 새 패키지 임의 추가 (예: 상태관리·스타일 라이브러리·다른 폼 라이브러리).

### 권장하지 않음

- ⚠️ 4단계 이상 중첩 디렉토리.
- ⚠️ 단일 파일 300줄 초과.
- ⚠️ 의미 없는 폴더명(`misc/`, `common/`, `shared/`, `utils/`).
- ⚠️ default export와 named export 혼재.

---

## 검증 체크리스트 (Task 완료 직전 반드시 확인)

```
□ npm run check-all 통과
□ npm run build 통과 (Turbopack)
□ Playwright MCP 4-카테고리 테스트 통과 (면제 Task 제외)
□ ROADMAP.md의 Task 체크박스 갱신
□ ROADMAP.md의 진행 현황 요약 표 동기화
□ 핵심 파일 동기화 표(위)에 해당하는 모든 파일 갱신
□ console.* 호출이 0건임을 grep으로 확인
□ any 타입 사용이 0건임을 확인
□ 새로 추가한 서버 전용 모듈에 `import 'server-only'` 존재 확인
□ 새로 추가한 상수가 매직 넘버 형태로 노출되지 않음
□ /q/[slug] 경로 작업 시 noindex,nofollow 메타가 유지됨을 browser_evaluate로 확인
□ 환경변수 변경 시 .env.example과 README.md가 동기화됨
□ 메타 파일·디렉토리 보호 표의 경로를 사용자 승인 없이 수정하지 않았음
□ 설정 파일 잠금값(.prettierrc / tsconfig.json / components.json / next.config.ts) 변경 없음
□ 현재 코드베이스 스냅샷 섹션과 실제 파일 상태가 일치함 (불일치 시 즉시 갱신)
```
