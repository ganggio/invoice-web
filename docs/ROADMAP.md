# invoice-web 개발 로드맵

> Notion에 작성한 견적서를 웹 링크로 공유하고 PDF로 다운로드할 수 있는 솔로 개발자·프리랜서용 서비스의 MVP 구현 로드맵

- 작성일: 2026-06-21
- 버전: 0.2.0 (MVP, 5-Phase 재구성)
- 관련 문서: `@/docs/PRD.md`, `@/CLAUDE.md`, `@/docs/guides/project-structure.md`, `@/docs/guides/nextjs-15.md`

---

## 개요

invoice-web은 Notion으로 업무를 관리하는 1인 프리랜서·소규모 스튜디오를 위한 견적서 공유 서비스로, 다음 핵심 가치를 제공합니다.

- **Notion을 단일 데이터 소스로 활용**: 발행자는 Notion DB만 편집하면 즉시 웹 공유 가능 (F001)
- **반응형 견적서 웹 뷰**: `/q/[slug]` URL로 모바일·데스크탑 어디서나 깔끔하게 확인 (F002)
- **PDF 다운로드**: 서버에서 Pretendard 폰트가 임베드된 PDF를 즉시 생성 (F003)
- **흰 화면 없는 에러 경험**: 잘못된 slug·Notion 오류 시 명확한 안내 화면 제공 (F010)
- **셀프 서비스 온보딩**: 홈 페이지에서 Notion 연동 가이드를 따라 발행자가 직접 설정 (F011)

---

## 개발 워크플로우

1. **작업 계획**
   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - 새로운 작업을 포함하도록 `ROADMAP.md` 업데이트
   - 우선순위 작업은 마지막 완료된 작업 다음에 삽입

2. **작업 생성**
   - `/tasks` 디렉토리에 새 작업 파일 생성 (명명 형식: `XXX-description.md`)
   - 명세, 관련 파일, 수락 기준, 구현 단계 포함
   - **API 연동·비즈니스 로직·UI 인터랙션을 포함한 모든 Task는 `## 테스트 체크리스트` 섹션을 반드시 포함하며, 다음 4개 카테고리의 Playwright MCP 시나리오를 작성해야 함**:
     1. 정상 시나리오 (Happy Path)
     2. 실패·예외 시나리오 (네트워크 오류, 잘못된 입력, 권한 없음 등)
     3. 엣지 케이스 (빈 데이터, 최대값, 경계값 등)
     4. 회귀 방지 (이전에 발견되었던 버그가 재발하지 않는지 확인)
   - 각 시나리오에는 사용할 Playwright MCP 도구를 반드시 명시 (`browser_navigate`, `browser_snapshot`, `browser_click`, `browser_fill_form`, `browser_type`, `browser_network_requests`, `browser_console_messages` 등)

3. **작업 구현**
   - 작업 파일의 명세를 따라 구현
   - **🚨 모든 구현 작업은 완료 직후 반드시 Playwright MCP로 테스트를 수행해야 함 (예외 없음)**
   - **표준 테스트 수행 절차**: dev 서버 기동 → `browser_navigate` → `browser_snapshot` → 액션 수행(`browser_click`/`browser_fill_form` 등) → `browser_network_requests` 로 API 호출 검증 → `browser_console_messages` 로 에러 확인 → 통과 시 체크박스 갱신, 실패 시 원인 분석 후 수정·재테스트
   - **테스트가 통과하지 않은 작업은 "완료"로 표시 금지**. 실패한 시나리오는 작업 파일에 실패 로그·스크린샷 경로를 남기고 수정 후 재실행
   - 각 단계 후 작업 파일의 체크리스트 갱신
   - 단계 완료 후 중단하고 추가 지시 대기

4. **로드맵 업데이트**
   - 완료된 작업과 Phase에 ✅ 표시
   - 진행 현황 요약 표 동기화
   - 모든 Playwright MCP 테스트 시나리오가 통과한 경우에만 ✅ 부여

---

## 진행 현황 요약

| Phase | 목표 | 예상 소요 | 완료 / 전체 | 상태 |
|-------|------|----------|-----------|------|
| Phase 1 | 프로젝트 초기 설정 | 1-2일 | 1 / 3 | 진행 중 |
| Phase 2 | 공통 모듈 개발 | 2-3일 | 0 / 7 | 대기 |
| Phase 3 | 핵심 기능 개발 (견적서 + PDF) | 3-4일 | 0 / 6 | 대기 |
| Phase 4 | 추가 기능 개발 (에러·홈·SEO·a11y) | 2-3일 | 0 / 5 | 대기 |
| Phase 5 | 최적화 및 배포 | 1-2일 | 0 / 3 | 대기 |
| **합계** | — | **9-14일** | **1 / 24** | — |

---

## 개발 단계

### Phase 1: 프로젝트 초기 설정

> **예상 소요 시간**: 1-2일
> **목표**: Next.js 스타터를 invoice-web 요구사항에 맞게 정리하고, 라우트 골격·환경변수·Notion 클라이언트 기반을 만들어 본격 개발의 토대를 만든다.
> **이유**: 견고한 기반(의존성·환경변수·라우트·Notion 클라이언트 인스턴스) 없이는 이후 모든 Task가 동작하지 않으므로 최우선 진행.

- **Task 001: 스타터 정리 및 의존성 설치** `[x]` — 우선순위
  - 관련 PRD: 기술 스택 전반
  - 핵심 산출물:
    - `package.json` (의존성 추가)
    - `src/app/page.tsx` (스타터 기본 화면 제거 및 빈 페이지로 교체)
    - `public/` (스타터 SVG 등 불필요 자산 정리)
  - 설치할 핵심 패키지:
    - 런타임: `@notionhq/client`, `@react-pdf/renderer`, `nanoid`, `pino`, `pino-pretty`, `zod`, `sonner`
    - 타입: `@types/node` 최신화 확인
    - shadcn/ui 추가: `button`, `card`, `table`, `separator`, `alert`, `skeleton`, `sonner` (`npx shadcn@latest add ...`)
  - 완료 조건:
    - `npm run dev` 실행 시 빈 홈 페이지가 정상 렌더링
    - `npm run check-all` 통과
    - shadcn/ui 컴포넌트 7종이 `src/components/ui/` 에 추가됨
    - **최소 시각 검증**: `browser_navigate('http://localhost:3000')` + `browser_snapshot` 으로 빈 홈 페이지가 렌더링되는지 1회 확인
  - 의존성: 없음
  - 비고: 인터랙티브 동작이 없는 설정 Task로 4개 카테고리 시나리오 작성은 면제 (최소 시각 검증만 수행)

- **Task 002: 환경변수 스키마·검증 및 폴더·라우트 골격 생성** `[ ]`
  - 관련 PRD: "환경변수 및 설정", "라우트 구조 (App Router)", "아키텍처 레이어"
  - 핵심 산출물:
    - `src/lib/env.ts` — Zod 기반 환경변수 검증 모듈
    - `.env.example` — 발행자 안내용 샘플 (값은 비워둠)
    - `.env.local` (gitignore 확인)
    - 빈 껍데기 라우트 파일:
      - `src/app/page.tsx` — 홈 placeholder
      - `src/app/q/[slug]/page.tsx` — 견적서 페이지 placeholder (Server Component)
      - `src/app/q/[slug]/loading.tsx` — 스켈레톤 placeholder
      - `src/app/q/[slug]/not-found.tsx` — 404 placeholder
      - `src/app/q/[slug]/error.tsx` — 에러 placeholder (Client Component)
      - `src/app/api/q/[slug]/pdf/route.ts` — PDF route placeholder
    - 디렉토리: `src/lib/notion/`, `src/components/quote/`, `src/components/pdf/`, `src/components/home/`
  - 검증 항목:
    - `NOTION_TOKEN` (필수, 1자 이상)
    - `NOTION_QUOTE_DB_ID` (필수, 1자 이상)
    - `NOTION_ITEM_DB_ID` (선택, Relation 사용 시)
  - 완료 조건:
    - 환경변수 누락 시 부팅 단계에서 사람이 읽을 수 있는 한국어 에러 메시지 노출
    - `env` 객체를 import 하면 타입이 추론됨 (`any` 금지)
    - 모든 라우트가 404 없이 빈 화면(또는 임시 텍스트)을 렌더링
    - 디렉토리 구조가 `docs/guides/project-structure.md` 와 정합
    - **최소 시각 검증**: `browser_navigate` 로 `/`, `/q/test-slug` 두 경로를 방문하고 `browser_snapshot` 으로 빈 화면이 그려지는지 확인, `.env.local` 누락 상태에서 발생하는 에러 메시지를 `browser_console_messages` 로 확인
  - 의존성: Task 001
  - 비고: 설정·라우트 골격 Task로 4개 카테고리 시나리오 작성은 면제 (최소 시각 검증만 수행)

- **Task 003: Notion 클라이언트 인스턴스 (`src/lib/notion/client.ts`)** `[ ]`
  - 관련 PRD: "외부 연동 — Notion API"
  - 핵심 산출물:
    - `src/lib/notion/client.ts` — `@notionhq/client` 의 `Client` 인스턴스를 `env.NOTION_TOKEN` 으로 초기화하여 export
  - 구현 사항:
    - 서버 전용 모듈임을 명시 (`import 'server-only'`)
    - 싱글톤 패턴으로 인스턴스 1회 생성
    - 한국어 JSDoc 주석으로 책임 명시
  - 완료 조건:
    - `getQuoteBySlug` 등 후속 Service 가 import 하여 사용 가능
    - 클라이언트 번들에 포함되지 않음(`server-only` 위반 시 빌드 실패 확인)
    - **최소 시각 검증**: 임시 디버그 라우트에서 `databases.retrieve` 1회 호출 → `browser_console_messages` 로 정상 응답 로깅 확인
  - 의존성: Task 002
  - 비고: 인프라 모듈 Task로 4개 카테고리 시나리오 작성은 면제 (최소 시각 검증만 수행)

#### Phase 1 완료 기준
- [ ] `npm run dev` 및 `npm run build` 가 무경고로 통과
- [ ] 모든 라우트 골격(`/`, `/q/[slug]`, `/api/q/[slug]/pdf`)이 응답함
- [ ] 환경변수 누락 시 명확한 한국어 에러 메시지 출력
- [ ] Notion 클라이언트 인스턴스가 서버에서만 사용 가능함이 검증됨
- [ ] 해당 Phase의 모든 Task가 Playwright MCP 테스트(최소 시각 검증 포함)를 통과함

---

### Phase 2: 공통 모듈 개발

> **예상 소요 시간**: 2-3일
> **목표**: 견적서 도메인 타입·Notion Adapter·Service·캐시·공통 UI 컴포넌트·공통 유틸(logger·통화 포맷)을 한 번에 만들어 이후 모든 기능이 재사용한다.
> **이유**: 모든 기능(웹 뷰·PDF·에러 페이지·홈)이 동일한 `Quote` 타입과 동일한 컴포넌트를 사용하므로, 공통 모듈을 먼저 만들면 중복 구현·타입 불일치·로깅 누락을 원천 차단할 수 있다.

- **Task 004: 도메인 타입 정의 (`src/lib/notion/types.ts`)** `[ ]` — 우선순위
  - 관련 PRD: F001, "데이터 모델"
  - 핵심 산출물:
    - `src/lib/notion/types.ts` — `Quote`, `QuoteLineItem`, `Currency` 타입 정의
  - 정의 항목:
    - `QuoteLineItem`: `id`, `name`, `description?`, `quantity`, `unitPrice`, `amount`
    - `Quote`: `id`, `slug`, `quoteNumber`, `issuer`, `client`, `issuedAt`, `validUntil`, `items`, `taxRate`, `currency`, `memo?`, 파생값(`subtotal`, `tax`, `total`)
    - 세율 기본값 상수 `DEFAULT_TAX_RATE = 10` (매직 넘버 금지)
    - 통화 기본값 상수 `DEFAULT_CURRENCY: Currency = 'KRW'`
  - 완료 조건:
    - `any` 타입 사용 없음
    - 모든 필드에 JSDoc 한국어 주석
    - **최소 시각 검증**: `tsc --noEmit` 통과 후 임시 페이지에 타입을 import 한 상태로 `browser_navigate` + `browser_snapshot` 1회 확인
  - 의존성: Task 002
  - 비고: 순수 타입 정의 Task로 4개 카테고리 시나리오 작성은 면제

- **Task 005: pino 로거 및 통화 포맷 유틸 (`src/lib/logger.ts`, `src/lib/format.ts`)** `[ ]`
  - 관련 PRD: "서버 측 구조화 로그 기록 (`pino`)", "console.log 금지", "통화 포맷 KRW/USD"
  - 핵심 산출물:
    - `src/lib/logger.ts` — pino 인스턴스 (`level: process.env.NODE_ENV === 'production' ? 'info' : 'debug'`, 개발 환경 `pino-pretty`)
    - `src/lib/format.ts` — `formatCurrency(amount, currency)` 함수 (`KRW`: `₩1,234,567`, `USD`: `$1,234.56`)
  - 구현 사항:
    - logger 는 서버 전용(`import 'server-only'`)
    - `formatCurrency` 는 웹/PDF 양쪽에서 재사용되므로 isomorphic
    - 매직 넘버 금지 — 소수 자릿수 상수(`KRW_DECIMALS = 0`, `USD_DECIMALS = 2`)로 분리
  - 완료 조건:
    - 모든 서버 코드에서 `console.log` 0건 (ESLint 규칙 추가 권장)
    - 단위 테스트로 통화별 포맷 검증
    - **최소 시각 검증**: 임시 디버그 페이지에서 KRW/USD 두 통화의 포맷 결과가 화면에 정확히 표시되는지 `browser_snapshot` 으로 확인, `browser_console_messages` 로 클라이언트 측 console 오염 0건 확인
  - 의존성: Task 004
  - 비고: 인프라 유틸 Task로 4개 카테고리 시나리오 작성은 면제 (최소 시각 검증만 수행)

- **Task 006: Notion Adapter 구현 (`src/lib/notion/adapter.ts`)** `[ ]`
  - 관련 PRD: F001, "Notion 견적서 DB", "Notion 항목 DB"
  - 핵심 산출물:
    - `src/lib/notion/adapter.ts` — Notion `PageObjectResponse` → `Quote` 변환 함수
    - `src/lib/notion/errors.ts` — `QuoteFieldMissingError`, `NotionApiError` 클래스 정의
    - `src/lib/notion/adapter.test.ts` — Notion fixture 기반 단위 테스트
  - 구현 사항:
    - `notionPageToQuote(page, items)` 함수
    - 필수 필드 누락 시 `QuoteFieldMissingError` 던지기 (어떤 필드가 비었는지 배열로 보존)
    - 합계 재계산: `subtotal = Σ(quantity × unitPrice)`, `tax = subtotal × taxRate / 100`, `total = subtotal + tax`
    - 세율 미설정 시 `DEFAULT_TAX_RATE` 적용
    - 통화 미설정 시 `DEFAULT_CURRENCY` 적용
  - 완료 조건:
    - 정상/누락/오타 케이스 단위 테스트 3종 이상 통과
    - 서버에서 합계를 항상 재계산 (Notion Formula 값 신뢰 금지)
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 정상 fixture를 사용하는 디버그 페이지(`/dev/adapter-check`) 에 접속 → `browser_navigate` → `browser_snapshot` 으로 `Quote.total` 값이 화면에 정확히 렌더링되는지 확인
    - **실패·예외**: 필수 필드(`quoteNumber`)가 빠진 fixture로 동일 페이지 접속 → `browser_snapshot` 으로 `QuoteFieldMissingError` 메시지·누락 필드 목록 노출 확인, `browser_console_messages` 로 에러 로깅 확인
    - **엣지 케이스**: 항목 0개 / 항목 100개 / `taxRate = 0` / `unitPrice = 0` fixture 4종에 대해 `browser_snapshot` 으로 `subtotal`, `tax`, `total` 의 경계값 계산을 검증
    - **회귀 방지**: Notion Formula 의 `total` 값이 의도적으로 잘못된 fixture 를 넣고도 화면에 표시되는 `total` 은 서버 재계산 값과 일치하는지 `browser_snapshot` 으로 확인 (Notion Formula 값 신뢰 금지 규칙 회귀 방지)
  - 의존성: Task 004, Task 005

- **Task 007: Quote Service 구현 (`src/lib/notion/quote.service.ts`)** `[ ]`
  - 관련 PRD: F001, "아키텍처 레이어", "엣지 케이스"
  - 핵심 산출물:
    - `src/lib/notion/quote.service.ts` — `getQuoteBySlug(slug)` 함수
  - 구현 사항:
    - `databases.query` 로 slug 필터 → Page 조회 → 항목 Relation 펼침
    - 견적서가 없으면 `null` 반환 (404 처리는 호출부에서)
    - Notion API 호출 실패 시 `NotionApiError` 던지기 (원본 에러 wrap)
    - 1회 지수 백오프 재시도 (500ms → 실패 시 throw)
    - `pino` 로 호출 성공·실패 구조화 로그 (`slug`, `latencyMs`, `errorCode`)
  - 완료 조건:
    - Service는 도메인 타입(`Quote | null`)만 반환 (Notion raw 타입 노출 금지)
    - 재시도 동작이 단위 테스트로 검증됨
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 실제 Notion 환경(또는 Mock 서버)에 유효한 slug 로 `browser_navigate('/q/{slug}')` → `browser_network_requests` 로 Notion API 호출이 1회만 발생하는지 확인, `browser_snapshot` 으로 견적서 렌더링 확인
    - **실패·예외**: `NOTION_TOKEN` 을 잘못된 값으로 설정 후 `browser_navigate` → `browser_console_messages` 로 `NotionApiError` 로그 1건 + 재시도 1회 확인, `error.tsx` 가 렌더링되는지 `browser_snapshot` 확인
    - **엣지 케이스**: 존재하지 않는 slug 로 `browser_navigate('/q/non-existent')` → 서비스가 `null` 반환 → `not-found.tsx` 가 노출되는지 `browser_snapshot` 으로 확인
    - **회귀 방지**: Notion 응답 지연을 인위적으로 늘려도 재시도가 1회로 제한되는지 `browser_network_requests` 로 동일 엔드포인트 호출 수 확인 (무한 재시도 방지)
  - 의존성: Task 003, Task 006

- **Task 008: ISR 및 메모리 캐시 적용 (`src/lib/notion/cache.ts`)** `[ ]`
  - 관련 PRD: "Rate Limit 대응" (ISR 300초 + 메모리 10분)
  - 핵심 산출물:
    - `src/app/q/[slug]/page.tsx` 에 `export const revalidate = 300` 적용 (5분 ISR)
    - `src/lib/notion/cache.ts` — 인메모리 LRU 캐시 (`MAX_CACHE_TTL_MS = 10 * 60 * 1000`, 최대 100건)
    - 캐시 키: `quote:${slug}`
  - 완료 조건:
    - 동일 slug 연속 요청 시 두 번째 호출은 캐시에서 응답 (로그로 확인)
    - 캐시 만료 시 Notion 재호출
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 같은 slug 페이지에 2회 연속 `browser_navigate` → `browser_network_requests` 로 두 번째 요청이 Notion API 를 호출하지 않는지 확인, `browser_console_messages` 로 `cache hit` 로그 확인
    - **실패·예외**: Notion API 호출이 실패한 경우 캐시에 실패 응답이 저장되지 않는지 확인 (다음 요청에서 재시도되는지 `browser_network_requests` 로 검증)
    - **엣지 케이스**: 캐시 TTL(10분) 만료 직후 요청 시 Notion 재호출이 1회 발생하는지 `browser_network_requests` 로 확인 (시간 가속용 디버그 토글 사용)
    - **회귀 방지**: 서로 다른 slug 100개 동시 요청 시 LRU 정책에 따라 가장 오래된 항목이 제거되는지 `browser_console_messages` 로깅으로 확인
  - 의존성: Task 007

- **Task 009: 공통 견적서 UI 컴포넌트 (`src/components/quote/`)** `[ ]`
  - 관련 PRD: F002, "견적서 확인 페이지" 주요 기능
  - 핵심 산출물:
    - `QuoteHeader.tsx` — 발행자명, 견적번호, 발행일, 유효기간 (shadcn `Card`)
    - `QuoteClientCard.tsx` — 수신자명, 이메일 (shadcn `Card`)
    - `QuoteItemsTable.tsx` — 데스크탑: `Table`, 모바일(<768px): 카드 리스트로 자동 전환
    - `QuoteTotals.tsx` — 소계 → 부가세(`taxRate`%) → 총액 강조 (shadcn `Separator`)
    - `QuoteMemo.tsx` — 비고/메모 영역 (선택 필드라 부재 시 미렌더)
    - `QuoteInlineWarning.tsx` — shadcn `Alert` 기반 선택 필드 누락 경고
  - 컴포넌트는 모두 `props: { quote: Quote }` 형태로 받음 (웹/PDF 공통 props 재사용 의도)
  - 완료 조건:
    - 더미 `Quote` 데이터로 임시 페이지(`/dev/quote-preview`) 에서 확인 가능
    - 모바일(320px), 태블릿(768px), 데스크탑(1024px, 최대 폭 880px) 3가지 뷰포트 검증
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 더미 Quote 데이터로 임시 페이지(`/dev/quote-preview`) 에 `browser_navigate` → `browser_snapshot` 으로 6개 컴포넌트(헤더/클라이언트/항목/합계/메모/경고) 렌더링 확인
    - **실패·예외**: `quote.memo` 가 `undefined` 인 fixture 로 접속 → `QuoteMemo` 컴포넌트가 렌더링되지 않는지 `browser_snapshot` 으로 확인
    - **엣지 케이스**: `browser_resize(320, 568)` / `browser_resize(768, 1024)` / `browser_resize(1024, 768)` 3종 뷰포트에서 각각 `browser_snapshot` → 모바일에서는 카드 리스트, 데스크탑에서는 `Table` 로 자동 전환되는지 확인
    - **회귀 방지**: 항목 0건인 fixture 에서도 `QuoteItemsTable` 이 빈 상태 안내를 표시하고 깨지지 않는지, 선택 필드 4개(이메일·연락처·메모·통화) 누락 시 경고가 1건으로 묶여 표시되는지 `browser_snapshot` 으로 확인
  - 의존성: Task 004, Task 005

- **Task 010: Pretendard 폰트 임베드 및 PDF 컴포넌트 (`src/components/pdf/quote-pdf.tsx`)** `[ ]`
  - 관련 PRD: F003, "PDF: Pretendard TTF 파일 임베드"
  - 핵심 산출물:
    - `public/fonts/Pretendard-Regular.ttf`, `public/fonts/Pretendard-Bold.ttf` 추가
    - `src/components/pdf/fonts.ts` — `Font.register` 호출
    - `src/components/pdf/quote-pdf.tsx` — `<QuotePdf quote={quote} />`
  - 구현 사항:
    - 페이지 사이즈 A4, 여백 40pt
    - 헤더(발행자/견적번호/발행일/유효기간) → 클라이언트 정보 → 항목 테이블 → 합계 → 메모 순
    - 항목 50건 이상 시 자동 페이지 분할
    - 통화 포맷은 Task 005 의 `formatCurrency` 재사용
  - 완료 조건:
    - 한글 텍스트가 깨지지 않고 정상 출력
    - 단일 페이지·다중 페이지 fixture 양쪽에서 PDF 정상 생성
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: PDF 미리보기 디버그 페이지(`/dev/pdf-preview`)에서 `browser_navigate` → `browser_snapshot` 으로 PDF 미리보기 iframe 렌더링 확인, 한글 텍스트가 깨지지 않는지 시각 검증
    - **실패·예외**: 폰트 파일이 누락된 상태에서 PDF 생성 시 `browser_console_messages` 로 `Font.register` 에러 로그가 출력되는지 확인
    - **엣지 케이스**: 항목 50건/100건 fixture 에서 페이지 자동 분할이 동작하는지 `browser_snapshot` 으로 확인, 통화 `KRW`/`USD` 각각의 포맷이 올바른지 확인
    - **회귀 방지**: 한글·영문 혼합 문자열 fixture 에서 글리프 누락(`□`) 없이 정상 렌더링되는지 `browser_snapshot` 으로 확인
  - 의존성: Task 004, Task 005, Task 009 (도메인 타입·통화 포맷 공유)

#### Phase 2 완료 기준
- [ ] `getQuoteBySlug('test-slug')` 가 도메인 `Quote` 객체를 반환
- [ ] Notion 4xx/5xx 시 재시도 1회 후 `NotionApiError` 발생
- [ ] 캐시 적중률이 로그로 확인 가능
- [ ] 더미 데이터로 모든 견적서 UI 컴포넌트가 3종 뷰포트에서 정상 렌더링
- [ ] PDF 컴포넌트에서 한글이 깨지지 않고 출력
- [ ] 모든 서버 코드에서 `console.log` 사용 0건
- [ ] 해당 Phase의 모든 Task가 Playwright MCP 테스트를 통과함

---

### Phase 3: 핵심 기능 개발

> **예상 소요 시간**: 3-4일
> **목표**: 견적서 확인 페이지(`/q/[slug]`)와 PDF 다운로드 기능을 완성하여 실제 사용자 여정(접속 → 견적서 확인 → PDF 저장)을 처음부터 끝까지 동작시킨다.
> **이유**: 견적서 공유 서비스의 가장 기본이 되는 두 기능(F001/F002/F003)을 한 Phase에 묶어, 사용자 핵심 가치를 가장 빠르게 검증한다.

- **Task 011: 견적서 페이지 Server Component 구현 (`/q/[slug]/page.tsx`)** `[ ]` — 우선순위
  - 관련 PRD: F001, F002
  - 핵심 산출물:
    - `src/app/q/[slug]/page.tsx` — async Server Component
    - `generateMetadata` — `<meta name="robots" content="noindex,nofollow">` 적용
  - 구현 사항:
    - `params` 는 `Promise<{ slug: string }>` (Next.js 15 async params)
    - `getQuoteBySlug(slug)` → `null` 이면 `notFound()` 호출
    - 정상 데이터 시 `QuoteHeader / QuoteClientCard / QuoteItemsTable / QuoteTotals / QuoteMemo / QuoteInlineWarning` 조합 렌더링
    - 최상위 컨테이너: `max-w-[880px] mx-auto` (PRD "최대 폭 880px")
    - 선택 필드 누락은 `QuoteInlineWarning` 으로 인라인 노출, 필수 필드 누락은 Adapter 에서 throw 되어 `error.tsx` 가 처리
  - 완료 조건:
    - 유효한 slug 접속 시 견적서 전체 렌더링
    - `view-source` 에서 `noindex,nofollow` 메타 확인
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 유효 slug 로 `browser_navigate('/q/{slug}')` → `browser_snapshot` 으로 5개 영역 렌더링 확인, `browser_evaluate` 로 `document.querySelector('meta[name="robots"]')?.getAttribute('content')` 가 `noindex,nofollow` 인지 확인
    - **실패·예외**: 존재하지 않는 slug 로 `browser_navigate` → 404 응답 및 `not-found.tsx` 가 노출되는지 `browser_snapshot` 으로 확인
    - **엣지 케이스**: slug 에 한글·특수문자(`/q/한글-slug`, `/q/test%20slug`)가 포함된 경우 정상 디코딩되어 조회되는지 `browser_snapshot` + `browser_network_requests` 로 확인, 선택 필드(`client.email`)가 비어 있는 fixture 에서 인라인 경고와 본문이 함께 렌더링되는지 확인
    - **회귀 방지**: 검색 엔진 크롤러를 가정한 User-Agent 헤더로 접근해도 `noindex,nofollow` 가 유지되는지 `browser_network_requests` 로 응답 HTML 검증
  - 의존성: Task 007, Task 008, Task 009

- **Task 012: 로딩 스켈레톤 (`/q/[slug]/loading.tsx`)** `[ ]`
  - 관련 PRD: "로딩 스켈레톤 (Notion 데이터 로딩 중)"
  - 핵심 산출물:
    - `src/app/q/[slug]/loading.tsx` — shadcn `Skeleton` 으로 헤더/항목 테이블/합계 영역 모사
  - 완료 조건:
    - 캐시 미적중 시 네트워크 throttling 에서 스켈레톤이 가시적으로 표시됨
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 네트워크 throttling(`Slow 3G`)을 활성화한 상태에서 `browser_navigate('/q/{slug}')` → 응답 도착 전 `browser_snapshot` 으로 스켈레톤 노출 확인
    - **실패·예외**: 응답이 도착하지 않은 채 타임아웃이 발생하는 시나리오에서 스켈레톤이 무한 표시되지 않고 `error.tsx` 로 전이되는지 `browser_snapshot` 으로 확인
    - **엣지 케이스**: 캐시 적중 시(즉시 응답) 스켈레톤이 깜빡임 없이 콘텐츠로 교체되는지 `browser_snapshot` 연속 캡처로 확인
    - **회귀 방지**: 스켈레톤이 실제 콘텐츠와 동일한 높이를 가져 CLS 가 발생하지 않는지 `browser_evaluate` 로 `LayoutShift` 측정값 0 확인
  - 의존성: Task 009

- **Task 013: PDF Route 구현 (`/api/q/[slug]/pdf/route.ts`)** `[ ]`
  - 관련 PRD: F003, PRD의 route.ts 예시 코드
  - 핵심 산출물:
    - `src/app/api/q/[slug]/pdf/route.ts` — GET 핸들러
  - 구현 사항:
    - `params` 는 `Promise<{ slug: string }>`
    - `getQuoteBySlug(slug)` 가 `null` 이면 `Response('Not Found', { status: 404 })`
    - `renderToStream(<QuotePdf quote={quote} />)` 으로 스트림 생성
    - `Content-Type: application/pdf`
    - `Content-Disposition: attachment; filename*=UTF-8''견적서_{client.name}_{issuedAt}.pdf` (RFC 5987 인코딩)
    - 실패 시 `pino` 로그 + 500 응답
  - 완료 조건:
    - `curl -O http://localhost:3000/api/q/{slug}/pdf` 로 파일 다운로드 성공
    - 파일명에 한글 포함 시 OS에서 정상 표시
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: `browser_navigate('/api/q/{slug}/pdf')` 후 `browser_network_requests` 로 응답 상태 200, `Content-Type: application/pdf`, `Content-Disposition` 헤더에 RFC 5987 인코딩된 한글 파일명이 포함되는지 확인
    - **실패·예외**: 존재하지 않는 slug 호출 시 응답 상태 404 가 반환되는지 `browser_network_requests` 로 확인, Notion API 실패 모의 시 500 응답과 `pino` 에러 로그가 남는지 `browser_console_messages` 로 확인
    - **엣지 케이스**: 항목 100건 fixture 에서 PDF 응답이 5초 이내에 완료되는지 `browser_network_requests` 의 `timing` 으로 검증
    - **회귀 방지**: `Content-Disposition` 의 한글 파일명이 macOS/Windows 다운로드 폴더에서 깨지지 않는지 헤더 raw 문자열을 `browser_network_requests` 로 검증
  - 의존성: Task 007, Task 010

- **Task 014: PDF 다운로드 버튼 컴포넌트 (`src/components/quote/QuoteDownloadButton.tsx`)** `[ ]`
  - 관련 PRD: F003, "PDF 다운로드 버튼 상태"
  - 핵심 산출물:
    - `QuoteDownloadButton.tsx` — Client Component (`'use client'`)
    - `Sonner` 토스트 연결 (`src/app/layout.tsx` 에 `<Toaster />` 추가)
  - 상태 머신:
    - `idle` → "PDF 다운로드" (Download 아이콘)
    - `loading` → "PDF 생성 중..." (Spinner, 버튼 비활성화)
    - `success` → "다운로드 완료" (Check 아이콘, 2초 후 `idle` 복귀, `SUCCESS_RESET_MS = 2000`)
    - `error` → "다시 시도" (AlertCircle 아이콘 + Sonner 토스트로 원인 표시)
  - 구현 사항:
    - `fetch('/api/q/[slug]/pdf')` → `blob` → `URL.createObjectURL` → `a.download` 트리거
    - 다운로드 도중 페이지 이탈 시 메모리 누수 방지 (`URL.revokeObjectURL`)
    - 모든 버튼에 `aria-label` (접근성)
  - 완료 조건:
    - 4가지 상태 전환이 시각적으로 모두 확인됨
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: `browser_navigate('/q/{slug}')` → `browser_click` 으로 다운로드 버튼 클릭 → `browser_snapshot` 으로 `loading` → `success` 상태 전환 확인, `browser_network_requests` 로 PDF 요청 완료 검증
    - **실패·예외**: PDF Route 가 500 을 반환하는 모의 상황에서 `browser_click` → `error` 상태 + Sonner 토스트 노출 확인 (`browser_snapshot` + `browser_console_messages`)
    - **엣지 케이스**: 다운로드 진행 중 `browser_navigate_back` 으로 페이지를 떠난 후 `browser_console_messages` 로 메모리 누수 경고가 없는지 확인 (`URL.revokeObjectURL` 호출 검증)
    - **회귀 방지**: `success` 상태가 정확히 2000ms(`SUCCESS_RESET_MS`) 후 `idle` 로 복귀하는지 `browser_wait_for` + `browser_snapshot` 으로 시점별 확인
  - 의존성: Task 013

- **Task 015: 견적서 페이지 통합 테스트 (Playwright MCP)** `[ ]`
  - 관련 PRD: F001, F002, 비기능 요구사항(LCP < 2.5초)
  - 핵심 산출물:
    - `tasks/015-quote-page-e2e.md` 의 `## 테스트 체크리스트`
  - 완료 조건:
    - 아래 4개 카테고리 시나리오 모두 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 유효 slug 접속 → `browser_navigate` → `browser_snapshot` 으로 헤더·항목·합계·메모 전체 렌더링 확인, `browser_evaluate` 로 항목 합계(`subtotal`, `tax`, `total`) 수치 정합성 검증
    - **실패·예외**: 존재하지 않는 slug 접속 시 `not-found.tsx` 노출 확인 (`browser_snapshot`), Notion API 5xx 모의 시 `error.tsx` 노출 확인 (`browser_snapshot` + `browser_console_messages`)
    - **엣지 케이스**: `browser_resize(375, 667)` 모바일 뷰포트에서 항목이 카드 리스트로 전환되는지, LCP 가 2500ms 미만인지 `browser_evaluate` 로 `PerformanceObserver` 측정값 확인
    - **회귀 방지**: `noindex,nofollow` 메타가 응답 HTML에 포함되는지 `browser_network_requests` 의 응답 본문에서 직접 검증 (크롤러 색인 방지 회귀)
  - 의존성: Task 011, Task 012

- **Task 016: PDF 다운로드 E2E 테스트 (Playwright MCP)** `[ ]`
  - 관련 PRD: F003, 비기능 요구사항(PDF 생성 < 5초 P95)
  - 핵심 산출물:
    - `tasks/016-pdf-download-e2e.md` 의 `## 테스트 체크리스트`
  - 완료 조건:
    - 아래 4개 카테고리 시나리오 모두 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: `browser_navigate('/q/{slug}')` → `browser_click` 으로 다운로드 → `browser_snapshot` 으로 `loading` 상태 노출 확인, 파일이 다운로드 폴더에 저장되고 파일명이 `견적서_{client.name}_{issuedAt}.pdf` 패턴인지 검증
    - **실패·예외**: 잘못된 slug 페이지에서 다운로드 시도 시 Sonner 토스트 에러 노출 확인 (`browser_snapshot`)
    - **엣지 케이스**: 다운로드 응답 시간이 5초 이하인지 `browser_network_requests` 의 `timing.duration` 으로 10회 측정 (P95 추적용 기록)
    - **회귀 방지**: KRW / USD 통화별로 다운로드된 PDF 의 통화 기호가 올바른지 PDF 텍스트 추출 후 검증 (이전에 발견된 통화 포맷 버그 회귀 방지)
  - 의존성: Task 014

#### Phase 3 완료 기준
- [ ] 유효 slug 접속 시 모든 영역이 반응형으로 정확히 렌더링
- [ ] LCP < 2.5초 (개발 환경 Lighthouse 측정)
- [ ] `noindex,nofollow` 메타 확인
- [ ] PDF에 한글이 깨지지 않고 표시됨
- [ ] 다운로드 버튼 4가지 상태가 정상 전환됨
- [ ] 평균 PDF 생성 시간 5초 미만 (개발 환경 기준)
- [ ] 해당 Phase의 모든 Task가 Playwright MCP 테스트를 통과함

---

### Phase 4: 추가 기능 개발

> **예상 소요 시간**: 2-3일
> **목표**: 핵심 기능을 보호하는 에러·예외 처리(F010), 발행자 셀프 온보딩(F011), SEO 메타데이터, 접근성(WCAG AA)을 완성한다.
> **이유**: 핵심 기능이 완성된 후 사용자 경험(흰 화면 방지, 발행자 온보딩, 검색 노출 정책, 접근성)을 보호·확장하는 부가 기능을 묶어 일관된 품질로 마무리한다.

- **Task 017: 404 페이지 (`/q/[slug]/not-found.tsx`)** `[ ]` — 우선순위
  - 관련 PRD: F010, "404 페이지"
  - 핵심 산출물:
    - `src/app/q/[slug]/not-found.tsx`
  - 구현 사항:
    - "이 견적서를 찾을 수 없습니다" 안내 문구
    - 발행자에게 URL 재확인 요청 문구
    - "홈으로 돌아가기" 버튼 (`/` 로 이동)
    - Notion 페이지 삭제·권한 회수 케이스도 동일 화면으로 처리 (410도 not-found로 흡수)
  - 완료 조건:
    - 존재하지 않는 slug 접속 시 자동 진입
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 존재하지 않는 slug 로 `browser_navigate('/q/non-existent')` → `browser_snapshot` 으로 404 안내 화면 노출 확인, "홈으로 돌아가기" 버튼 클릭 → `browser_click` → `browser_navigate` 결과로 `/` 로 이동 확인
    - **실패·예외**: Notion 페이지가 삭제된 케이스를 모의해 동일한 404 화면이 노출되는지 `browser_snapshot` 으로 확인 (410 도 not-found 로 흡수)
    - **엣지 케이스**: slug 가 빈 문자열·매우 긴 문자열(255자)인 경우에도 안정적으로 404 화면이 노출되는지 `browser_snapshot` 으로 확인
    - **회귀 방지**: 404 페이지에도 `noindex,nofollow` 메타가 적용되어 검색 엔진에 색인되지 않는지 `browser_evaluate` 로 검증
  - 의존성: Task 011

- **Task 018: 에러 페이지 (`/q/[slug]/error.tsx`)** `[ ]`
  - 관련 PRD: F010, "에러 페이지", "엣지 케이스" 표
  - 핵심 산출물:
    - `src/app/q/[slug]/error.tsx` — Client Component (`'use client'`)
  - 구현 사항:
    - `error.name` 기반 원인별 분기:
      - `NotionApiError` → "Notion 서비스에 일시적인 문제가 있습니다"
      - `QuoteFieldMissingError` → "견적서 정보가 불완전합니다. 발행자에게 문의해주세요." + 누락 필드 목록
      - 그 외 → "잠시 후 다시 시도해 주세요"
    - "다시 시도" 버튼 (`reset()` 호출)
    - "홈으로 돌아가기" 버튼
  - 완료 조건:
    - 의도적으로 `NOTION_TOKEN` 을 잘못 설정하여 에러 화면 노출 확인
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 잘못된 `NOTION_TOKEN` 환경에서 `browser_navigate('/q/{slug}')` → `browser_snapshot` 으로 `NotionApiError` 안내 메시지 노출 확인
    - **실패·예외**: `QuoteFieldMissingError` 가 발생하는 fixture 에서 누락 필드 목록이 화면에 표시되는지 `browser_snapshot` 으로 확인, "다시 시도" 클릭 (`browser_click`) → `reset()` 호출 후 동일 에러가 반복되는지 확인
    - **엣지 케이스**: 알 수 없는 에러(`new Error('unknown')`) 발생 시 기본 안내 문구("잠시 후 다시 시도해 주세요")가 노출되는지 `browser_snapshot` 으로 확인
    - **회귀 방지**: 에러 페이지에서 "홈으로 돌아가기" 클릭 시 정상적으로 `/` 로 이동하며 잔여 에러 상태가 없는지 `browser_navigate` + `browser_console_messages` 로 확인
  - 의존성: Task 006, Task 007

- **Task 019: 홈 페이지 UI 구현 (`src/app/page.tsx`)** `[ ]`
  - 관련 PRD: F011, "홈 페이지"
  - 핵심 산출물:
    - `src/app/page.tsx`
    - `src/components/home/Hero.tsx`, `Features.tsx`, `NotionSetupGuide.tsx`, `DbSchemaTable.tsx`
  - 구현 사항:
    - Hero: 서비스 한 줄 설명 + 특장점 3가지 (Notion 연동, 반응형 웹뷰, PDF 다운로드)
    - Notion Integration 생성 방법 단계별 안내 (1~5단계, 이미지 또는 아이콘과 함께)
    - Notion DB 필수 속성 스키마 명세 표 (PRD "데이터 모델" 표를 그대로 시각화)
    - 공유 URL 예시: `https://invoice-web.app/q/[slug]`
    - "Notion 연동 가이드 보기" 외부 링크 버튼 (target: `_blank`, `rel: noopener noreferrer`)
  - 완료 조건:
    - 모바일·태블릿·데스크탑 3종에서 가독성 확보
    - 접근성: `h1` 한 개, 시맨틱 마크업, 모든 버튼에 `aria-label`
    - Playwright MCP 테스트 시나리오 전부 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: `browser_navigate('/')` → `browser_snapshot` 으로 Hero / Features / NotionSetupGuide / DbSchemaTable 4개 섹션 렌더링 확인
    - **실패·예외**: 외부 가이드 링크 클릭 (`browser_click`) 시 새 탭(`browser_tabs`)으로 열리고 `rel=noopener noreferrer` 가 적용되어 있는지 `browser_evaluate` 로 검증
    - **엣지 케이스**: `browser_resize(320, 568)` / `browser_resize(768, 1024)` / `browser_resize(1280, 800)` 3종 뷰포트에서 가독성 깨짐 없이 `browser_snapshot` 통과
    - **회귀 방지**: `h1` 태그가 정확히 1개인지 `browser_evaluate` 로 `document.querySelectorAll('h1').length === 1` 검증 (SEO·접근성 회귀 방지)
  - 의존성: Task 002

- **Task 020: 홈 SEO 메타데이터·OG 태그 및 `/q/[slug]` noindex 재확인** `[ ]`
  - 관련 PRD: F011, 비기능 요구사항(SEO — 홈은 색인 허용, `/q/[slug]` 는 noindex)
  - 핵심 산출물:
    - `src/app/layout.tsx` 또는 `src/app/page.tsx` 의 `metadata`
    - `public/og-image.png` (1200×630)
  - 구현 사항:
    - `title`, `description`, `openGraph`, `twitter` 메타
    - 홈 페이지는 `robots: { index: true, follow: true }` 명시
    - `/q/[slug]` 페이지는 Task 011 에서 별도로 `noindex,nofollow` 처리되어 있음을 회귀 검증
  - 완료 조건:
    - 메타 태그 검사 도구로 OG 이미지 확인
    - **최소 시각 검증**: `browser_navigate('/')` → `browser_evaluate` 로 `<meta property="og:image">` 의 `content` 가 `/og-image.png` 를 가리키는지 확인, `<meta name="robots">` 의 값이 `index,follow` 인지 확인. `/q/{slug}` 도 별도로 `browser_evaluate` 로 `noindex,nofollow` 유지 재검증
  - 의존성: Task 011, Task 019
  - 비고: 메타데이터·정적 자산 Task로 4개 카테고리 시나리오 작성은 면제 (최소 시각 검증만 수행)

- **Task 021: 접근성 감사 (WCAG AA)** `[ ]`
  - 관련 PRD: "접근성 — shadcn/ui 기본 a11y 준수 (WCAG AA), 모든 버튼에 `aria-label`"
  - 핵심 산출물:
    - 접근성 체크리스트 (작업 파일 내)
  - 점검 항목:
    - 모든 버튼·인터랙티브 요소에 `aria-label`
    - 색상 대비 4.5:1 이상
    - 키보드 내비게이션(Tab/Enter/Space) 동작
    - 폼·테이블에 적절한 라벨·헤더
    - Playwright MCP axe-core 통합 또는 Lighthouse a11y 90점 이상
  - 완료 조건:
    - Lighthouse Accessibility 점수 90+ (홈, 견적서, 404, 에러 4개 페이지)
    - **시각 검증**: Lighthouse 결과 외에도 `browser_navigate` + `browser_press_key('Tab')` 반복으로 키보드 포커스 순회가 의미 있는 순서인지 `browser_snapshot` 으로 확인
  - 의존성: Task 015, Task 017, Task 018, Task 019
  - 비고: 감사·점수 측정 Task로 4개 카테고리 시나리오 작성은 면제 (Lighthouse·키보드 시각 검증으로 갈음)

#### Phase 4 완료 기준
- [ ] 잘못된 slug, Notion 실패, 필드 누락 3가지 케이스 모두 안내 화면 노출
- [ ] 발행자가 홈 페이지만 보고 Notion 연동을 끝까지 완료할 수 있음
- [ ] 홈 페이지 OG 미리보기가 정상 노출되고 `/q/[slug]` 는 `noindex,nofollow` 유지
- [ ] 4개 페이지(홈/견적서/404/에러)의 Lighthouse Accessibility 90+
- [ ] 해당 Phase의 모든 Task가 Playwright MCP 테스트를 통과함

---

### Phase 5: 최적화 및 배포

> **예상 소요 시간**: 1-2일
> **목표**: 성능 기준(LCP < 2.5s, PDF < 5s P95)을 달성하고 Vercel 프로덕션에 배포하여 사용자 여정 1~5단계 전체를 회귀 검증한다.
> **이유**: 기능이 모두 완성된 시점에 성능 최적화·반응형 검증·프로덕션 배포·E2E 회귀를 수행하면, 측정값이 실제 출시 환경 기준으로 의미를 가지며 최종 품질이 보장된다.

- **Task 022: 성능 최적화 (LCP < 2.5s, PDF < 5s P95)** `[ ]` — 우선순위
  - 관련 PRD: "성능 — 견적서 페이지 LCP < 2.5초, PDF 생성 < 5초 (P95)"
  - 핵심 산출물:
    - 성능 측정 리포트 (작업 파일 내)
  - 점검 항목:
    - `next/font` 로 Pretendard 웹폰트 최적화 (FOIT 방지)
    - 이미지 `next/image` 사용 확인
    - ISR `revalidate: 300` 동작 확인 (Task 008 검증)
    - PDF Route 응답 시간 10회 측정 평균/P95
    - Bundle Analyzer 로 불필요 의존성 제거
    - 반응형 검증: 모바일(320px), 태블릿(768px), 데스크탑(1024px, 최대 폭 880px)
  - 완료 조건:
    - 견적서 페이지 Lighthouse Performance 90+
    - PDF 평균 < 3초, P95 < 5초
    - **시각 검증**: `browser_navigate('/q/{slug}')` → `browser_evaluate` 로 `PerformanceObserver` 의 LCP 값이 2500ms 미만인지 확인, `browser_network_requests` 로 PDF Route 응답 10회 측정값을 기록, `browser_resize` 3종 뷰포트에서 레이아웃 깨짐 없이 `browser_snapshot` 통과
  - 의존성: Task 015, Task 016
  - 비고: 성능 측정 Task로 4개 카테고리 시나리오 작성은 면제 (Lighthouse·Performance API·반응형 측정으로 갈음)

- **Task 023: Vercel 배포 및 환경변수 설정** `[ ]`
  - 관련 PRD: "배포 & 호스팅 — Vercel"
  - 핵심 산출물:
    - Vercel 프로젝트 연결
    - 환경변수 등록: `NOTION_TOKEN`, `NOTION_QUOTE_DB_ID`, (선택) `NOTION_ITEM_DB_ID`
    - 커스텀 도메인 연결 (예: `invoice-web.app`)
  - 완료 조건:
    - 프로덕션 URL 에서 `/`, `/q/{실제-slug}`, PDF 다운로드 모두 정상 동작
    - Vercel Logs 에서 pino 로그 확인
    - **최소 시각 검증**: 프로덕션 URL 에 `browser_navigate` → `browser_snapshot` 으로 홈 페이지 정상 렌더링 확인 (실제 사용자 시나리오 검증은 Task 024 에서 수행)
  - 의존성: Task 022
  - 비고: 인프라 설정 Task로 4개 카테고리 시나리오 작성은 면제 (Task 024 에서 통합 검증)

- **Task 024: 전체 사용자 플로우 E2E 회귀 테스트 (Playwright MCP)** `[ ]`
  - 관련 PRD: 사용자 여정 1~5단계
  - 핵심 산출물:
    - `tasks/024-full-flow-e2e.md` 의 `## 테스트 체크리스트`
  - 완료 조건:
    - 아래 4개 카테고리 시나리오 모두 프로덕션 환경에서 통과
  - **Playwright MCP 테스트 시나리오**:
    - **정상 (Happy Path)**: 프로덕션 URL 에서 `browser_navigate('/')` → 가이드 확인(`browser_snapshot`) → `browser_navigate('/q/{실제-slug}')` → 견적서 렌더링 확인 → `browser_click` 으로 PDF 다운로드 성공 검증 (`browser_network_requests`)
    - **실패·예외**: 존재하지 않는 slug 접속 → `not-found.tsx` 노출 → "홈으로 돌아가기" 클릭 시 정상 이동 확인 (`browser_click` + `browser_snapshot`), 잘못된 토큰 모의 환경에서 `error.tsx` 노출 후 "다시 시도" 동작 확인
    - **엣지 케이스**: `browser_resize(375, 667)` 모바일 뷰포트로 전체 플로우(홈 → 견적서 → PDF) 재실행, KRW / USD 두 통화의 PDF 가 정상 다운로드되는지 `browser_network_requests` 로 검증
    - **회귀 방지**: M1~M5 마일스톤에서 식별된 모든 시나리오(noindex 메타, 캐시 적중, 한글 파일명, 인라인 경고 등)를 한 번씩 재실행하여 회귀 발생 여부를 `browser_snapshot` + `browser_console_messages` 로 확인
  - 의존성: Task 023

#### Phase 5 완료 기준
- [ ] Lighthouse Performance / Accessibility 모두 90+
- [ ] LCP < 2.5초, PDF 생성 P95 < 5초 측정값 기록
- [ ] 모바일·태블릿·데스크탑 3종 뷰포트에서 레이아웃 깨짐 없음
- [ ] Vercel 프로덕션에서 사용자 여정 1~5단계 완주
- [ ] 모든 E2E 시나리오 통과
- [ ] 해당 Phase의 모든 Task가 Playwright MCP 테스트를 통과함

---

## 마일스톤 & 의존성 다이어그램

```
Phase 1 (Task 001~003)            [프로젝트 초기 설정]
   │
   ▼
Phase 2 (Task 004~010)            [공통 모듈: 타입·Adapter·Service·캐시·UI·PDF 컴포넌트]
   │
   ▼
Phase 3 (Task 011~016)            [핵심 기능: 견적서 페이지 + PDF 다운로드]
   │
   ▼
Phase 4 (Task 017~021)            [추가 기능: 에러·홈·SEO·a11y]
   │
   ▼
Phase 5 (Task 022~024)            [최적화·배포·E2E 회귀]
```

### 핵심 마일스톤

| 마일스톤 | 도달 시점 | 검증 방법 |
|---------|----------|---------|
| M1: 개발 환경 준비 완료 | Phase 1 완료 | 모든 라우트 골격 응답 + 환경변수 검증 동작 |
| M2: 공통 모듈 사용 가능 | Phase 2 완료 | `getQuoteBySlug(slug)` 가 `Quote` 객체 반환, 더미 데이터로 UI/PDF 정상 렌더 |
| M3: 핵심 사용자 여정 동작 | Phase 3 완료 | `/q/[slug]` 렌더링 + 한글 PDF 다운로드 성공 + LCP < 2.5초 |
| M4: 흰 화면 없는 UX + 셀프 온보딩 | Phase 4 완료 | 모든 예외 케이스에서 안내 화면 노출, 홈 페이지만으로 발행자가 Notion 연동 완료 가능 |
| M5: 프로덕션 출시 | Phase 5 완료 | Vercel 배포 + 전체 E2E 통과 + 성능 기준 충족 |

---

## 위험 요소

PRD의 "열린 질문"·"엣지 케이스" 섹션을 기반으로 한 MVP 단계 잠재 리스크입니다.

| 위험 | 영향 | 대응 전략 |
|------|------|---------|
| Notion API Rate Limit 초과 (3 req/s) | 견적서 페이지 일시 장애 | ISR `revalidate: 300` + 메모리 캐시 10분 + 1회 지수 백오프 재시도 (Task 007, 008) |
| slug 충돌 또는 추측 가능성 | 다른 클라이언트의 견적서 노출 | `nanoid(10)` 으로 추측 불가능한 slug 생성 권장. MVP는 발행자 직접 입력이라 안내 강화 필요 (Task 019) |
| `@react-pdf/renderer` Vercel cold start | PDF 응답 지연 | Vercel Edge가 아닌 Node 런타임 명시, 폰트 파일을 `public/` 이 아닌 `src/` 정적 import 검토 (Task 010) |
| Pretendard 폰트 임베드 실패로 한글 깨짐 | PDF 사용 불가 | TTF 파일 검증, `Font.register` 호출 후 fixture로 한글 출력 확인 (Task 010) |
| 발행자가 필수 필드 누락 | 견적서 렌더링 실패 | Adapter에서 `QuoteFieldMissingError` 던지고 `error.tsx` 가 누락 필드 목록 표시 (Task 006, 018) |
| Notion 페이지 삭제 / 권한 회수 | 클라이언트 링크 깨짐 | `null` 반환 → `notFound()` 흡수, "더 이상 유효하지 않습니다" 안내 (Task 017) |
| 발행자가 단일이라는 가정 깨짐 | OAuth 전환 필요 | MVP 범위 외, v2 후보로 백로그 유지 |
| slug 50건+ 시점에 DB 조회 성능 저하 | 페이지 응답 지연 | KV 스토어(Vercel KV)로 slug ↔ Page ID 매핑 마이그레이션 (v2 후보) |
| 공유 URL 영구 노출의 개인정보 우려 | 법적 리스크 | 발행자가 Notion에서 slug 변경하면 즉시 무효화 가능함을 가이드에 명시. v2에서 만료/비밀번호 검토 |

---

## 테스트 전략 (Playwright MCP 기반)

invoice-web 프로젝트의 모든 구현 작업은 Playwright MCP 도구를 활용한 실측 검증을 거쳐야 합니다. 본 섹션은 어떤 Task 가 어떤 깊이의 테스트를 필요로 하는지, 표준 수행 절차는 무엇인지, 실패 시 어떻게 처리하는지를 명확히 정의합니다.

### 1. 테스트 의무 작성 대상 분류

| 분류 | Task 번호 | 필수 시나리오 |
|------|----------|--------------|
| **4개 카테고리 시나리오 필수** (API 연동·비즈니스 로직·UI 인터랙션) | 006, 007, 008, 009, 010, 011, 012, 013, 014, 015, 016, 017, 018, 019, 024 | 정상 / 실패·예외 / 엣지 케이스 / 회귀 방지 4종 모두 작성 |
| **최소 시각 검증 (`browser_navigate` + `browser_snapshot`) 만 수행** | 001, 002, 003, 004, 005, 020 | 단일 시각 검증 1회로 갈음 |
| **자체 측정 도구 + 최소 시각 검증** | 021 (Lighthouse a11y + 키보드 순회), 022 (Lighthouse Performance + LCP/네트워크·반응형 측정), 023 (프로덕션 첫 진입 확인) | Lighthouse·Performance API 결과 + `browser_snapshot` 1회 |

### 2. 표준 테스트 수행 절차 (8단계)

모든 Task 의 Playwright MCP 테스트는 다음 8단계 절차를 따릅니다.

1. **dev 서버 기동**: `npm run dev` 로 로컬 서버를 띄우고 정상 부팅을 확인
2. **`browser_navigate`**: 검증 대상 URL 로 이동 (예: `http://localhost:3000/q/{slug}`)
3. **`browser_snapshot`**: 페이지 초기 상태를 스냅샷으로 기록
4. **액션 수행**: `browser_click` / `browser_fill_form` / `browser_type` / `browser_press_key` / `browser_resize` 등으로 시나리오에 정의된 사용자 동작 실행
5. **`browser_network_requests`**: API 호출 발생 여부·상태 코드·응답 헤더·`timing.duration` 등을 검증
6. **`browser_console_messages`**: 콘솔 에러·경고가 없는지 확인 (특히 `console.log` 0건 규칙)
7. **시각·로직 검증**: `browser_evaluate` 또는 후속 `browser_snapshot` 으로 기대 상태 도달 여부 확인
8. **결과 기록**: 통과 시 작업 파일의 체크박스를 갱신, 실패 시 실패 로그·스크린샷 경로·재현 절차를 작업 파일에 추가

### 3. 테스트 실패 시 처리 정책

- **완료 표시 절대 금지**: 단 하나의 시나리오라도 실패하면 해당 Task 의 체크박스를 `✅` 로 갱신하지 않습니다.
- **원인 분석**: 실패 로그(`browser_console_messages`)·네트워크 응답(`browser_network_requests`)·스냅샷 차이를 종합 분석해 작업 파일에 한국어로 기록합니다.
- **수정 후 재테스트**: 코드를 수정한 뒤 8단계 절차를 처음부터 다시 수행합니다. 부분 실행 금지.
- **재발 방지**: 동일한 실패가 두 번 이상 발생한 경우, 해당 Task 의 "회귀 방지" 시나리오에 영구적인 검증 단계를 추가합니다.

### 4. 자주 사용할 Playwright MCP 도구 매트릭스

| 도구명 | 주요 용도 | 사용 예시 |
|--------|----------|----------|
| `browser_navigate` | 검증 대상 URL 진입 | `browser_navigate('http://localhost:3000/q/{slug}')` |
| `browser_navigate_back` | 페이지 이탈·뒤로가기 시나리오 검증 | 다운로드 진행 중 이탈 후 메모리 누수 확인 |
| `browser_snapshot` | 페이지 상태 시각 검증 | 컴포넌트 렌더링·반응형 레이아웃·에러 화면 확인 |
| `browser_click` | 버튼·링크 클릭 액션 | PDF 다운로드 버튼, 홈으로 돌아가기 버튼 |
| `browser_fill_form` | 다수 필드를 한 번에 채우기 | 향후 폼이 추가될 때 (현재 MVP 에는 폼 없음) |
| `browser_type` | 개별 입력 필드 텍스트 입력 | slug 검색 등 단일 입력 시나리오 |
| `browser_press_key` | 키보드 액션 (Tab·Enter·Space) | 접근성 키보드 내비게이션 검증 |
| `browser_resize` | 반응형 뷰포트 검증 | 320 / 375 / 768 / 1024 / 1280 등 |
| `browser_network_requests` | API 호출·헤더·응답시간 검증 | Notion API 호출 횟수, PDF Route 응답 시간 P95 |
| `browser_console_messages` | 콘솔 로그·에러 확인 | `console.log` 0건 규칙, 에러 로그 검증 |
| `browser_evaluate` | DOM·메타·Performance API 조회 | LCP 측정, `<meta name="robots">` 값 확인 |
| `browser_wait_for` | 비동기 상태 전이 대기 | `success` → `idle` 상태 전환 확인 |
| `browser_tabs` | 새 탭으로 열린 외부 링크 검증 | 홈 페이지의 Notion 가이드 외부 링크 |
| `browser_take_screenshot` | 실패 시 증거 캡처 | 실패 시 작업 파일에 첨부할 스크린샷 |

### 5. 운영 원칙

- 시나리오 작성 시 추상적인 표현("정상 동작 확인") 대신 사용할 도구와 검증 포인트를 구체적으로 명시합니다.
- 네트워크 의존이 큰 시나리오(예: Notion API 호출)는 Mock 서버 또는 fixture 기반 디버그 페이지를 활용해 결정론적으로 재현합니다.
- 모든 Phase 의 완료 기준에는 "해당 Phase 의 모든 Task 가 Playwright MCP 테스트를 통과함" 항목이 포함되어 있으므로, Phase 를 닫기 전 회귀 테스트(이전 Task 시나리오 재실행)를 1회 수행합니다.
