# invoice-web MVP PRD

> Notion에 작성한 견적서를 클라이언트가 웹 링크로 확인하고 PDF로 다운로드할 수 있는 솔로 개발자/프리랜서용 서비스

---

## 핵심 정보

**목적**: 프리랜서/소규모 사업자가 Notion에서 작성한 견적서를 별도 작업 없이 깔끔한 웹 페이지와 PDF로 클라이언트에게 즉시 공유한다.

**사용자**: Notion으로 업무를 관리하는 1인 프리랜서·소규모 스튜디오(발행자)와, 링크를 받아 견적서를 확인·저장하는 클라이언트(수신자).

---

## 사용자 여정

```
1. [발행자] Notion 견적서 DB에 새 행(페이지) 작성
   - 견적번호, 클라이언트명, 항목, 단가, 수량, 유효기간 등 입력
   - slug 속성에 고유 식별자 입력 (또는 자동 생성)
   ↓

2. [발행자] 공유 URL 복사 → 클라이언트에게 전달
   - URL 형태: https://invoice-web.app/q/[slug]
   - 카카오톡·이메일·메신저 등으로 전달
   ↓

3. [클라이언트] 링크 접속 → 견적서 확인 페이지 진입
   ↓ slug 유효성 확인

   유효한 slug   → 견적서 확인 페이지 정상 렌더링
   유효하지 않음 → 404 페이지로 이동
   Notion 오류   → 에러 페이지로 이동
   ↓

4. [클라이언트] 견적서 내용 확인 후 선택
   - "PDF 다운로드" 버튼 클릭 → PDF 생성 → 파일 저장
   - 그냥 닫음 → 종료
   ↓

5. [완료] PDF 파일로 내부 결재 첨부 또는 보관
```

---

## 기능 명세

### 1. MVP 핵심 기능

| ID | 기능명 | 설명 | MVP 필수 이유 | 관련 페이지 |
|----|--------|------|-------------|------------|
| **F001** | Notion 견적서 데이터 연동 | Notion API로 견적서 DB를 조회하여 내부 Quote 타입으로 변환 | 서비스의 유일한 데이터 소스 — 이것 없이는 아무것도 보여줄 수 없음 | 견적서 확인 페이지 |
| **F002** | 공유 URL 기반 견적서 웹 뷰 | slug로 견적서를 식별하여 반응형 웹 레이아웃으로 렌더링 | 클라이언트가 서비스를 경험하는 유일한 진입점 | 견적서 확인 페이지 |
| **F003** | PDF 다운로드 | 견적서를 서버에서 PDF로 생성하여 파일 다운로드 제공 | 클라이언트의 핵심 니즈 — 내부 결재용 PDF 보관 | 견적서 확인 페이지 |

### 2. MVP 필수 지원 기능

| ID | 기능명 | 설명 | MVP 필수 이유 | 관련 페이지 |
|----|--------|------|-------------|------------|
| **F010** | 에러/예외 처리 UI | 잘못된 slug, Notion API 실패, 데이터 누락 등 각 상황별 안내 화면 | 클라이언트 경험 보호 — 흰 화면 대신 명확한 안내 필수 | 404 페이지, 에러 페이지 |
| **F011** | 랜딩/소개 페이지 | 서비스 소개 및 발행자를 위한 Notion 연동 설정 안내 | 발행자가 서비스를 이해하고 Notion 연동을 스스로 설정할 수 있어야 함 | 홈 페이지 |

### 3. MVP 이후 기능 (v2 후보)

- 조회수·다운로드 카운트 통계 (발행자 대시보드)
- 공유 URL 비밀번호 보호 또는 만료 기한 설정
- 발행자 로그인 및 다중 발행자 지원 (Notion OAuth)
- 견적서 승인/반려 워크플로우
- 이메일 자동 발송
- 다국어(i18n) 지원
- 조회수 UTM 파라미터 추적
- slug ↔ Notion Page ID KV 저장소 마이그레이션 (건수 50+ 시점)

---

## 메뉴 구조

```
invoice-web 내비게이션

공개 접근 (인증 불필요)
├── 홈
│   └── 기능: F011 (서비스 소개 및 Notion 연동 안내)
├── 견적서 확인
│   └── 기능: F001, F002, F003 (URL slug로 직접 접근, 메뉴에 노출되지 않음)
├── 404 페이지
│   └── 기능: F010 (잘못된 slug 또는 존재하지 않는 경로)
└── 에러 페이지
    └── 기능: F010 (Notion API 오류, 데이터 불완전 등)
```

> 이 서비스는 발행자용 관리 UI가 MVP 범위 밖입니다. 발행자는 Notion을 직접 편집하고, 클라이언트는 공유 URL로만 접근합니다.

---

## 페이지별 상세 기능

### 홈 페이지

> **구현 기능:** `F011` | **접근:** 공개 (누구나)

| 항목 | 내용 |
|------|------|
| **역할** | 서비스 소개 랜딩 페이지 — 발행자가 Notion 연동 방법을 이해하고 스스로 설정할 수 있도록 안내 |
| **진입 경로** | 루트 URL 직접 접속, 404/에러 페이지의 "홈으로" 버튼 클릭 |
| **사용자 행동** | 서비스 설명을 읽고, Notion DB 스키마 안내를 참고하여 Integration Token과 DB ID를 환경변수에 설정 |
| **주요 기능** | • 서비스 한 줄 설명 및 특장점 3가지 나열<br>• Notion Integration 생성 방법 단계별 안내<br>• Notion DB 필수 속성 스키마 명세 표 제공<br>• 공유 URL 형태(`/q/[slug]`) 예시 안내<br>• **"Notion 연동 가이드 보기"** 외부 링크 버튼 |
| **다음 이동** | 발행자 → Notion에서 slug 설정 후 공유 URL 생성 |

---

### 견적서 확인 페이지

> **구현 기능:** `F001`, `F002`, `F003` | **접근:** 공개 (URL slug 보유자)

| 항목 | 내용 |
|------|------|
| **역할** | 서비스의 핵심 페이지 — Notion 데이터를 렌더링하여 클라이언트가 견적서를 확인하고 PDF를 다운로드하는 공간 |
| **진입 경로** | 발행자로부터 공유받은 URL(`/q/[slug]`)로 직접 접속 |
| **사용자 행동** | 견적서 내용(항목·금액·유효기간 등)을 확인하고 "PDF 다운로드" 버튼을 클릭하여 파일 저장 |
| **주요 기능** | • **헤더 카드**: 발행자명, 견적번호, 발행일, 유효기간<br>• **클라이언트 정보 카드**: 수신자명, 이메일(선택)<br>• **항목 테이블**: 품목명 / 수량 / 단가 / 합계 (모바일에서는 카드 리스트로 전환)<br>• **합계 영역**: 소계 → 부가세(10%) → 총액 강조 표시<br>• **비고/메모 영역**: 결제 조건, 추가 안내 등<br>• **PDF 다운로드 버튼**: idle / loading / success / error 4가지 상태 관리<br>• 메타 태그 `noindex,nofollow`로 검색 엔진 색인 차단<br>• Notion 데이터 필드 누락 시 인라인 경고 표시 |
| **다음 이동** | PDF 다운로드 성공 → 파일 저장 완료 알림(2초 후 idle 복귀), 실패 → 에러 토스트 + "다시 시도" 버튼 |

---

### 404 페이지

> **구현 기능:** `F010` | **접근:** 잘못된 경로 접속 시 자동 진입

| 항목 | 내용 |
|------|------|
| **역할** | 존재하지 않는 slug 또는 삭제된 견적서 접근 시 명확한 안내 제공 |
| **진입 경로** | 유효하지 않은 `/q/[slug]` 접속, 존재하지 않는 경로 접속, Notion 페이지 삭제·권한 회수(410 처리) |
| **사용자 행동** | 안내 문구를 읽고 발행자에게 재문의하거나 홈으로 이동 |
| **주요 기능** | • "이 견적서를 찾을 수 없습니다" 안내 문구<br>• 발행자에게 URL 재확인 요청 안내<br>• **"홈으로 돌아가기"** 버튼 |
| **다음 이동** | "홈으로 돌아가기" → 홈 페이지 |

---

### 에러 페이지

> **구현 기능:** `F010` | **접근:** Notion API 오류 또는 서버 예외 발생 시 자동 진입

| 항목 | 내용 |
|------|------|
| **역할** | Notion API rate limit, 일시 장애, 데이터 파싱 오류 등 서버 에러 상황에서 사용자 경험 보호 |
| **진입 경로** | Notion API 호출 실패, 견적서 필수 필드 누락, 서버 타임아웃 |
| **사용자 행동** | 안내 문구 확인 후 재시도하거나 발행자에게 문의 |
| **주요 기능** | • 원인별 안내 메시지 (Notion 오류 / 데이터 불완전 / 일시 장애)<br>• **"다시 시도"** 버튼 (페이지 새로고침)<br>• **"홈으로 돌아가기"** 버튼<br>• 서버 측 구조화 로그 기록 (`pino`) |
| **다음 이동** | "다시 시도" → 동일 견적서 페이지 재요청, "홈으로 돌아가기" → 홈 페이지 |

---

## 데이터 모델

### Notion 견적서 DB ("견적서" 데이터베이스)

| 필드 | Notion 타입 | 필수 | 설명 |
|------|------------|------|------|
| 견적번호 | Title | 필수 | 예: `Q-2026-001` |
| 클라이언트명 | Text | 필수 | 수신자 회사명 또는 담당자명 |
| 클라이언트 이메일 | Email | 선택 | 표시용 (발송 기능 없음) |
| 발행일 | Date | 필수 | |
| 유효기간 | Date | 필수 | |
| 발행자명 | Text | 필수 | 발행자 또는 회사명 |
| 발행자 연락처 | Phone | 선택 | |
| 항목 | Relation | 필수 | "항목" DB와 1:N 관계 |
| 비고 | Rich Text | 선택 | 결제 조건, 추가 안내 등 |
| 세율 | Number | 선택 | 기본값 10 (%) |
| 통화 | Select | 선택 | KRW / USD (기본 KRW) |
| slug | Text | 필수 | 공유 URL 식별자 (`nanoid(10)`) |

### Notion 항목 DB ("항목" 데이터베이스)

| 필드 | Notion 타입 | 필수 | 설명 |
|------|------------|------|------|
| 품목명 | Title | 필수 | |
| 설명 | Rich Text | 선택 | 1~2줄 부연 설명 |
| 수량 | Number | 필수 | |
| 단가 | Number | 필수 | |
| 합계 | Formula | 자동 | `수량 * 단가` (서버에서 재계산하여 정합성 보장) |

### TypeScript 도메인 타입

```ts
// src/lib/notion/types.ts

/** 견적서 라인 아이템 (Notion "항목 DB" 1행) */
export interface QuoteLineItem {
  id: string;
  name: string;           // 품목명
  description?: string;   // 부연 설명
  quantity: number;       // 수량
  unitPrice: number;      // 단가
  amount: number;         // 합계 (수량 * 단가, 서버 재계산)
}

/** 견적서 전체 데이터 (웹 뷰 + PDF 공통 사용) */
export interface Quote {
  id: string;             // Notion Page ID
  slug: string;           // 공유 URL slug
  quoteNumber: string;    // 견적번호 (예: Q-2026-001)
  issuer: {
    name: string;
    contact?: string;
  };
  client: {
    name: string;
    email?: string;
  };
  issuedAt: string;       // ISO Date
  validUntil: string;     // ISO Date
  items: QuoteLineItem[];
  taxRate: number;        // 기본 10 (%)
  currency: 'KRW' | 'USD';
  memo?: string;
  // 서버에서 계산된 파생값
  subtotal: number;
  tax: number;
  total: number;
}
```

---

## 기술 스택

### 프론트엔드 프레임워크

- **Next.js 15.5.3** (App Router + Turbopack) — 서버 컴포넌트 기반 렌더링, ISR 캐싱
- **React 19.1.0** — 최신 동시성 기능 활용
- **TypeScript 5** — 엄격한 타입 안전성 (`any` 금지)

### 스타일링 & UI

- **TailwindCSS v4** — 설정 파일 없는 새로운 CSS 엔진
- **shadcn/ui (new-york 스타일)** — 견적서 레이아웃에 사용할 컴포넌트

  | 영역 | 컴포넌트 |
  |------|---------|
  | 견적서 헤더/클라이언트 카드 | `Card`, `CardHeader`, `CardTitle`, `CardDescription` |
  | 항목 테이블 | `Table`, `TableHeader`, `TableRow`, `TableCell` |
  | 구분선 | `Separator` |
  | 다운로드 버튼 | `Button` (variant: default, size: lg) |
  | 에러 안내 | `Alert`, `AlertTitle`, `AlertDescription` |
  | 로딩 스켈레톤 | `Skeleton` |
  | 다운로드 결과 알림 | `Sonner` (shadcn 권장 토스트) |

- **Lucide React** — 아이콘

### 폼 & 검증

- **React Hook Form 7.x** + **Zod** — 환경변수 스키마 검증 및 향후 폼 처리
- **Server Actions** — PDF 요청 등 서버 측 액션

### 외부 연동 — Notion API

- **`@notionhq/client`** — 공식 Notion SDK
- 인증: Internal Integration Token (`NOTION_TOKEN` 환경변수)
- 주요 엔드포인트: `databases.query`, `pages.retrieve`, `blocks.children.list`
- Rate Limit 대응: ISR `revalidate: 300` (5분) + 메모리 캐시 10분 + 1회 지수 백오프 재시도

### PDF 생성 라이브러리 비교 및 선택

| 라이브러리 | 장점 | 단점 | 추천도 |
|-----------|------|------|--------|
| **`@react-pdf/renderer`** | 서버리스 환경 안정, 한글 폰트(Pretendard/Noto Sans KR) 임베드 검증됨, Vercel 무료 플랜 호환 | 웹 컴포넌트와 별도 PDF 컴포넌트 트리 작성 필요 (코드 약간 중복) | ★★★★ **채택** |
| `puppeteer-core` + `@sparticuz/chromium` | 웹 페이지 그대로 PDF 변환 (레이아웃 100% 일치) | Vercel Lambda 50MB 용량 한계 근접, cold start 3~10초, 비용 증가 | ★★ |
| `html2pdf.js` / `jsPDF` | 클라이언트 사이드 처리로 서버 부하 없음 | 한글 폰트 처리 불안정, 표 분할 품질 낮음, 크로스 브라우저 이슈 | ★ |

**최종 선택: `@react-pdf/renderer`**

이유: Vercel 서버리스 환경에서 안정적으로 동작하고, Pretendard 폰트 임베드가 커뮤니티에서 검증되어 있습니다. 웹 컴포넌트와 별도 PDF 컴포넌트를 작성해야 하지만, 공통 `Quote` 타입을 props로 받는 얇은 레이아웃 컴포넌트(`<QuotePdf quote={quote} />`)로 추상화하면 유지보수 부담이 최소화됩니다.

```ts
// src/app/api/q/[slug]/pdf/route.ts
import { renderToStream } from '@react-pdf/renderer';
import { QuotePdf } from '@/components/pdf/quote-pdf';
import { getQuoteBySlug } from '@/lib/notion/quote.service';

export async function GET(
  _req: Request,
  { params }: { params: Promise<{ slug: string }> }
) {
  const { slug } = await params;
  const quote = await getQuoteBySlug(slug);
  if (!quote) return new Response('Not Found', { status: 404 });

  const stream = await renderToStream(<QuotePdf quote={quote} />);
  return new Response(stream as unknown as ReadableStream, {
    headers: {
      'Content-Type': 'application/pdf',
      // 파일명 규칙: 견적서_[클라이언트명]_[발행일].pdf
      'Content-Disposition': `attachment; filename*=UTF-8''${encodeURIComponent(
        `견적서_${quote.client.name}_${quote.issuedAt}`
      )}.pdf`,
    },
  });
}
```

### 배포 & 호스팅

- **Vercel** — Next.js 15 최적화 배포 플랫폼 (무료 플랜으로 MVP 트래픽 감당)

### 패키지 관리

- **npm** — 의존성 관리

---

## 라우트 구조 (App Router)

```
src/app/
├── page.tsx                     # / — 홈 페이지 (랜딩 + Notion 연동 안내)
├── q/
│   └── [slug]/
│       ├── page.tsx             # /q/[slug] — 견적서 확인 페이지 (Server Component)
│       ├── loading.tsx          # 스켈레톤 UI (Notion 데이터 로딩 중)
│       ├── not-found.tsx        # 잘못된 slug → 404 페이지
│       └── error.tsx            # Notion API 실패 → 에러 페이지
└── api/
    └── q/
        └── [slug]/
            └── pdf/
                └── route.ts     # GET /api/q/[slug]/pdf — PDF 다운로드
```

---

## 아키텍처 레이어

```
[Client Browser]
     ↓ HTTP
[Next.js App Router - Server Component]
     ↓
[Service Layer]   src/lib/notion/quote.service.ts
     ↓ Notion SDK
[Adapter Layer]   src/lib/notion/adapter.ts  (NotionPage → Quote 변환)
     ↓
[Notion API]      databases.query / pages.retrieve

[PDF Route]       /api/q/[slug]/pdf
     ↓
[QuotePdf Component]  @react-pdf/renderer
```

---

## 환경변수 및 설정

```bash
# .env.local
NOTION_TOKEN=secret_xxxxxxxxxxxx        # Notion Internal Integration Token
NOTION_QUOTE_DB_ID=xxxxxxxxxxxxxxxxxxxx  # 견적서 메인 DB ID
NOTION_ITEM_DB_ID=xxxxxxxxxxxxxxxxxxxx   # 항목(Line Item) DB ID (Relation 사용 시)
```

부팅 시 Zod 스키마로 환경변수를 검증합니다:

```ts
// src/lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NOTION_TOKEN: z.string().min(1, 'Notion Integration Token이 필요합니다'),
  NOTION_QUOTE_DB_ID: z.string().min(1, 'Notion 견적서 DB ID가 필요합니다'),
});

export const env = envSchema.parse(process.env);
```

---

## 엣지 케이스 및 에러 처리

| 상황 | 처리 방식 |
|------|---------|
| 존재하지 않는 slug 접속 | `not-found.tsx` — 404 안내 + "홈으로 돌아가기" |
| Notion 페이지 삭제·권한 회수 | `not-found.tsx` — "이 견적서는 더 이상 유효하지 않습니다" |
| Notion API rate limit 초과 | ISR 캐시 응답 우선 → 1회 재시도 → 실패 시 `error.tsx` |
| Notion API 일시 장애 | `error.tsx` — "잠시 후 다시 시도해 주세요" + "다시 시도" 버튼 |
| 견적서 필수 필드 누락 | `error.tsx` — "견적서 정보가 불완전합니다. 발행자에게 문의해주세요." + 누락 필드 목록 |
| PDF 생성 실패 (타임아웃 등) | 에러 토스트 + "다시 시도" 버튼 + 서버 로그(`pino`) 기록 |

### PDF 다운로드 버튼 상태

```
idle    → "PDF 다운로드"    (Download 아이콘)
loading → "PDF 생성 중..."  (Spinner, 버튼 비활성화)
success → "다운로드 완료"   (Check 아이콘, 2초 후 idle 복귀)
error   → "다시 시도"       (AlertCircle 아이콘 + Sonner 토스트로 원인 표시)
```

---

## 비기능 요구사항

| 카테고리 | 요구사항 |
|---------|---------|
| 성능 | 견적서 페이지 LCP < 2.5초 (ISR 캐시 활용), PDF 생성 < 5초 (P95) |
| SEO | 모든 `/q/[slug]` 페이지에 `<meta name="robots" content="noindex,nofollow">` |
| 보안 | slug는 `nanoid(10)`으로 추측 불가능하게 생성, Notion Token은 서버 전용 환경변수에서만 접근 |
| 접근성 | shadcn/ui 기본 a11y 준수 (WCAG AA), 모든 버튼에 `aria-label` |
| 로깅 | 서버 에러는 `pino`로 구조화 로깅, `console.log` 금지 |
| 반응형 | 모바일(320px~) / 태블릿(768px~) / 데스크탑(1024px~, 최대 폭 880px 중앙 정렬) |
| 폰트 | 웹: Pretendard (Next.js `next/font`), PDF: Pretendard TTF 파일 임베드 |

---

## 열린 질문

1. **slug 생성 방식**: 발행자가 Notion slug 속성에 직접 입력 vs 별도 유틸리티로 자동 생성 후 복사? MVP는 직접 입력 방식으로 시작.
2. **공유 URL 만료 정책**: 영구 유지 vs 발행일 기준 90일 자동 비활성? 개인정보 보호 관점에서 결정 필요.
3. **다중 발행자 지원 시점**: MVP는 단일 `NOTION_TOKEN`(단일 발행자) 전제. Phase 2에서 Notion OAuth 전환 검토.
4. **slug ↔ Notion Page ID 매핑**: MVP는 Notion DB slug 속성으로 직접 조회. 건수 50+ 시점부터 KV 스토어(Vercel KV 등) 검토.
5. **선택적 접근 보안**: 토큰 파라미터(`?token=xxx`) 또는 비밀번호 입력 페이지 — MVP에서 필요한지 결정 필요. (현재는 slug 자체가 보안 역할)

---

## 가정

- 발행자는 Notion 사용에 익숙하며, Integration 연결 가이드를 1회 따라할 수 있다.
- 클라이언트는 최신 브라우저(Chrome / Safari / Edge)를 사용하는 한국어 환경이다.
- Vercel 무료 플랜으로 MVP 트래픽(월 ~1,000 페이지뷰)을 충분히 감당할 수 있다.
- 견적서 1건당 항목 수는 일반적으로 50개 이하이다.
- Pretendard 폰트(OFL 라이선스)는 상업적 임베드가 허용된다.

---

> 작성일: 2026-06-14
> Version: 0.1.0 (MVP)
> 관련 문서: `@/CLAUDE.md`, `@/docs/guides/project-structure.md`, `@/docs/guides/nextjs-15.md`
