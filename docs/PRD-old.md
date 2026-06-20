# 📋 NotionQuote - Product Requirements Document (MVP)

> Notion에 작성한 견적서를 웹 링크와 PDF로 클라이언트에게 전달하는 솔로 프리랜서용 MVP 서비스

---

## 1. 📋 제품 개요 (Overview)

### 한 줄 설명
**NotionQuote**는 프리랜서/소규모 사업자가 이미 익숙한 Notion에 작성한 견적서를, 별도 가공 없이 **공유 URL**과 **PDF**로 클라이언트에게 즉시 전달할 수 있게 해주는 견적서 공유 웹 서비스입니다.

### 타겟 사용자
- **1차**: 디자이너/개발자/마케터 등 1인 프리랜서 (이미 Notion을 업무 도구로 사용 중)
- **2차**: 5인 이하 소규모 스튜디오/에이전시

### 해결하려는 문제
| 기존 방식의 문제 | 본 서비스의 해결책 |
|--------------|--------------|
| Notion 페이지를 그대로 공유하면 권한 설정이 번거롭고 디자인이 어수선함 | 견적서 전용 깔끔한 레이아웃으로 자동 렌더링 |
| Notion → 워드/한글 → PDF 변환 시 수작업 반복 | "PDF 다운로드" 버튼 한 번으로 동일 레이아웃 보장 |
| Excel/PDF 견적서 템플릿은 수정·재발행이 번거로움 | Notion에서 수정하면 공유 URL에 즉시 반영 |

### 차별점
- **Notion = 단일 소스**: 별도 입력 폼 없이 Notion DB만 관리하면 됨
- **로그인 불필요**: 클라이언트는 URL만 받으면 끝
- **웹 ↔ PDF 레이아웃 일치**: 화면에서 본 그대로 PDF로 다운로드

---

## 2. 🎯 목표 및 성공 지표 (Goals & Success Metrics)

### 비즈니스 목표 (정성)
- 프리랜서가 견적서 발송에 들이는 시간을 **5분 → 30초**로 단축
- Notion을 이미 쓰는 사용자의 워크플로우 안에 자연스럽게 녹아드는 도구 제공
- 4주 내 MVP를 출시하여 실제 사용자 피드백을 빠르게 수집

### MVP 성공 지표 (정량)
| 지표 | 목표값 | 측정 방법 |
|------|------|---------|
| 주간 활성 공유 URL | **20개** | `/q/[slug]` 페이지 고유 접근 카운트 |
| PDF 다운로드 전환율 | **공유 URL 진입의 50% 이상** | 다운로드 버튼 클릭 / 페이지 진입 |
| 견적서 페이지 LCP | **< 2.5초** | Vercel Analytics / Lighthouse |
| PDF 생성 응답 시간 | **< 5초** | API 라우트 처리 시간 로깅 |
| MVP 출시 일정 준수 | **4주 이내** | 마일스톤 산출물 완료 여부 |

---

## 3. 👥 페르소나 (Personas)

### 페르소나 A: 발행자(판매자)
- **이름**: 김프리 (30세, 프리랜스 UI/UX 디자이너)
- **도구 환경**: Notion(매일), Figma, Slack, 카카오톡
- **상황**: 매달 5~10건의 신규 견적서를 작성. 기존엔 Notion에서 작성 후 워드로 옮겨 PDF 변환 → 카카오톡 전달.
- **목표**: "내 Notion 견적서 DB를 그대로 클라이언트에게 보여주고 싶다. 디자인은 깔끔하게."
- **불만(Pain)**: PDF 변환 시 표 정렬이 깨지고 폰트가 바뀜. 수정할 때마다 같은 작업 반복.

### 페르소나 B: 수신자(클라이언트)
- **이름**: 이대표 (45세, 소규모 마케팅 회사 대표)
- **도구 환경**: 이메일, 카카오톡, 스마트폰 위주
- **상황**: 외주 디자이너로부터 견적서를 받아 내부 결재용으로 PDF 저장이 필요.
- **목표**: "링크 받으면 바로 모바일로 확인하고, PDF로 저장해서 회계팀에 전달하고 싶다."
- **불만(Pain)**: 모바일에서 Notion 공유 페이지는 로딩이 느리고 UI가 복잡함.

---

## 4. 🗺️ 핵심 사용자 시나리오 (User Scenarios)

### Happy Path
1. **김프리**는 Notion의 "견적서 DB"에 새 페이지를 만들고, 클라이언트명·항목·금액을 입력한다.
2. NotionQuote 관리 페이지(또는 환경설정)에서 해당 Notion 페이지 ID와 매핑된 **공유 slug**가 자동 생성된다.
3. 김프리는 `https://notionquote.app/q/abc123` 형태의 URL을 카카오톡으로 **이대표**에게 전달한다.
4. **이대표**는 모바일 브라우저로 URL에 접속, 깔끔한 견적서 레이아웃을 확인한다.
5. 이대표가 **"PDF 다운로드"** 버튼을 누르면 즉시 동일 레이아웃의 PDF가 다운로드된다.

### 엣지 케이스
| # | 시나리오 | 시스템 처리 |
|---|--------|---------|
| 1 | Notion 페이지에서 **필수 속성이 누락**됨 (예: 항목 빈 배열) | "견적서 정보가 불완전합니다. 발행자에게 문의해주세요." 안내 카드 + 누락 필드 목록 노출 |
| 2 | 존재하지 않는 **slug**로 접근 (`/q/xxxxx`) | `not-found.tsx`로 404 페이지 렌더링 (홈으로 돌아가기 CTA 포함) |
| 3 | Notion API **rate limit 초과** 또는 일시 장애 | 5초 stale-while-revalidate 캐시 응답 → 실패 시 fallback UI ("잠시 후 다시 시도") |
| 4 | **PDF 생성 실패** (서버 타임아웃, 폰트 로딩 실패 등) | 토스트 알림 + "다시 시도" 버튼 + 서버 로그 기록 |
| 5 | Notion 페이지가 **삭제됨/권한 회수됨** | 410 Gone 상태 + "이 견적서는 더 이상 유효하지 않습니다" 안내 |

---

## 5. ✨ 기능 요구사항 (Functional Requirements)

### 🟢 MUST HAVE (MVP)

#### F-01. Notion 데이터 연동
- **유저 스토리**: 발행자로서, 나는 Notion DB에 작성한 견적서가 자동으로 웹에 표시되기를 원한다. 왜냐하면 별도 입력 폼을 반복해서 채우고 싶지 않기 때문이다.
- **세부 요구**:
  - Notion Integration Token으로 인증
  - `databases.query` + `pages.retrieve`로 견적서 + 항목(Relation) 조회
  - 응답을 내부 도메인 타입 `Quote`로 변환하는 어댑터 레이어 구현

#### F-02. 공유 URL 기반 견적서 웹 뷰
- **유저 스토리**: 클라이언트로서, 나는 받은 링크로 견적서를 즉시 확인하고 싶다. 왜냐하면 로그인이나 앱 설치 없이 빠르게 내용을 보고 싶기 때문이다.
- **세부 요구**:
  - 라우트: `/q/[slug]` (App Router, Server Component 기본)
  - slug ↔ Notion Page ID 매핑은 환경변수 또는 단순 JSON 매핑 파일로 시작 (Phase 2에서 DB화)
  - 반응형 레이아웃 (모바일 우선)

#### F-03. PDF 다운로드
- **유저 스토리**: 클라이언트로서, 나는 동일 레이아웃 PDF를 다운로드받고 싶다. 왜냐하면 내부 결재에 첨부해야 하기 때문이다.
- **세부 요구**:
  - 다운로드 버튼 클릭 → API 라우트 호출 → PDF 스트림 반환
  - 파일명 규칙: `quote-{견적번호}-{발행일}.pdf`
  - 한글 폰트 임베드 (Pretendard 또는 Noto Sans KR)

#### F-04. 공유 URL 생성/관리 (발행자 측)
- **유저 스토리**: 발행자로서, 나는 견적서별 고유 공유 URL을 생성하고 싶다. 왜냐하면 클라이언트별로 다른 링크를 전달해야 하기 때문이다.
- **세부 요구**:
  - MVP: 환경변수에 `NOTION_PAGE_ID:slug` 쌍을 등록하면 자동 활성화
  - slug는 `nanoid(10)`으로 추측 불가능하게 생성

### 🟡 SHOULD HAVE (Phase 2)
- **F-05. 캐싱 레이어**: Notion API 응답을 5~10분간 Edge 캐시
- **F-06. 조회수/다운로드 카운트**: 발행자가 URL 활성 여부를 확인
- **F-07. 인쇄 친화 CSS**: 브라우저 인쇄 미리보기에서도 깔끔하게

### 🔴 WON'T HAVE (Out of Scope)
- 회원가입/로그인 시스템
- 결제 처리, 전자 서명
- 다국어(i18n) 자동 전환
- 이메일 자동 발송
- 견적서 버전 관리/히스토리
- 클라이언트의 견적 승인/반려 워크플로우

---

## 6. 🧱 정보 구조 / 데이터 모델 (Data Model)

### Notion DB 스키마 (메인 DB: "견적서")

| 속성명 | Notion 타입 | 필수 | 설명 |
|------|----------|----|----|
| `견적번호` | Title | ✅ | 예: `Q-2026-001` |
| `클라이언트명` | Text | ✅ | 수신자 회사/담당자명 |
| `클라이언트 이메일` | Email | ⬜ | 표시용 (발송 기능 없음) |
| `발행일` | Date | ✅ | |
| `유효기간` | Date | ✅ | |
| `발행자명` | Text | ✅ | 발행자/회사명 |
| `발행자 연락처` | Phone | ⬜ | |
| `항목` | Relation → "항목 DB" | ✅ | 1:N 관계 |
| `메모/비고` | Rich Text | ⬜ | 결제 조건 등 |
| `세율` | Number | ⬜ | 기본 10 (%) |
| `통화` | Select | ⬜ | KRW / USD (기본 KRW) |
| `slug` | Text | ✅ | 공유 URL용 식별자 |

### 항목 서브 스키마 (관계 DB: "항목")

| 속성명 | Notion 타입 | 필수 | 설명 |
|------|----------|----|----|
| `품목명` | Title | ✅ | |
| `설명` | Rich Text | ⬜ | 1~2줄 부연 설명 |
| `수량` | Number | ✅ | |
| `단가` | Number | ✅ | |
| `합계` | Formula | ✅ | `수량 * 단가` |

### TypeScript 도메인 타입

```ts
// src/lib/notion/types.ts

/** 견적서 라인 아이템 (Notion "항목 DB" 1행) */
export interface QuoteLineItem {
  id: string;
  name: string;          // 품목명
  description?: string;  // 부연 설명
  quantity: number;      // 수량
  unitPrice: number;     // 단가
  amount: number;        // 합계 (수량 * 단가)
}

/** 견적서 메타 + 라인 아이템 묶음 */
export interface Quote {
  id: string;            // Notion Page ID
  slug: string;          // 공유 URL slug
  quoteNumber: string;   // 견적번호 (예: Q-2026-001)
  issuer: {
    name: string;
    contact?: string;
  };
  client: {
    name: string;
    email?: string;
  };
  issuedAt: string;      // ISO Date
  validUntil: string;    // ISO Date
  items: QuoteLineItem[];
  taxRate: number;       // 기본 10 (%)
  currency: "KRW" | "USD";
  memo?: string;
  /** 파생 값 (서버에서 계산) */
  subtotal: number;
  tax: number;
  total: number;
}
```

> **가정**: Notion 측 `합계` Formula는 신뢰하되, `subtotal/tax/total`은 서버에서 재계산하여 정합성을 보장.

---

## 7. 🎨 UI/UX 요구사항

### 페이지 라우트 구조 (App Router)

```
src/app/
├── page.tsx                    # / (랜딩: 서비스 소개 + 사용법)
├── q/
│   └── [slug]/
│       ├── page.tsx            # 견적서 조회 페이지 (Server Component)
│       ├── loading.tsx         # 스켈레톤 UI
│       ├── not-found.tsx       # 잘못된 slug 처리
│       └── error.tsx           # Notion API 실패 처리
└── api/
    └── q/
        └── [slug]/
            └── pdf/
                └── route.ts    # PDF 다운로드 (GET)
```

### 견적서 뷰 핵심 섹션 (위 → 아래)

1. **헤더**: 발행자명/로고, 견적번호, 발행일, 유효기간
2. **클라이언트 정보 카드**: 수신자명, 이메일
3. **항목 테이블**: 품목 / 수량 / 단가 / 합계
4. **합계 영역**: 소계 → 세금 → **총액(강조)**
5. **메모/비고**: 결제 조건, 유효기간 안내
6. **액션 영역**: `[ PDF 다운로드 ]` 버튼 (sticky bottom on mobile)

### shadcn/ui 컴포넌트 매핑

| 영역 | 컴포넌트 |
|----|-------|
| 헤더 카드 | `Card`, `CardHeader`, `CardTitle`, `CardDescription` |
| 항목 테이블 | `Table`, `TableHeader`, `TableRow`, `TableCell` |
| 구분선 | `Separator` |
| 다운로드 버튼 | `Button` (variant: `default`, size: `lg`) |
| 에러 카드 | `Alert`, `AlertTitle`, `AlertDescription` |
| 로딩 | `Skeleton` |
| 토스트 | `Sonner` (shadcn 권장 토스트) |

### 반응형 브레이크포인트

| 브레이크포인트 | 대상 기기 | 레이아웃 변화 |
|-----------|-------|----------|
| `sm` (640px-) | 모바일 | 1열 카드, 항목 테이블은 카드형 리스트로 변환 |
| `md` (768px) | 태블릿 | 헤더 2열 그리드, 표 기본 형태 |
| `lg` (1024px+) | 데스크탑 | 최대 폭 880px 중앙 정렬 (A4 비율 고려) |

### PDF 다운로드 버튼 상태

```
idle    → "PDF 다운로드"          (Download 아이콘)
loading → "PDF 생성 중..."        (Spinner)
success → "다운로드 완료"          (Check, 2초 후 idle 복귀)
error   → "다시 시도"             (AlertCircle + 토스트로 사유 표시)
```

---

## 8. 🔌 외부 연동 (Integrations)

### Notion API

| 항목 | 값/방식 |
|----|-------|
| 인증 | Internal Integration Token, `NOTION_TOKEN` 환경변수 |
| 클라이언트 | `@notionhq/client` |
| 주요 엔드포인트 | `databases.query`, `pages.retrieve`, `blocks.children.list` (필요 시) |
| Rate Limit | 평균 3 req/sec → ISR(`revalidate: 300`) + 메모리 캐시(10분)로 흡수 |
| 실패 대응 | 1회 재시도(지수 백오프) → 실패 시 `error.tsx`로 fallback |

### PDF 생성 라이브러리 비교

| 라이브러리 | 장점 | 단점 | 적합도 |
|---------|----|----|------|
| **`@react-pdf/renderer`** | React로 PDF 트리 작성, 폰트 임베드 안정, 서버리스 친화 | 웹 UI와 별도 컴포넌트 트리 작성 필요 (코드 중복) | ⭐⭐⭐⭐ |
| `puppeteer-core` + `@sparticuz/chromium` | 웹 페이지를 그대로 PDF로 (레이아웃 일치 100%) | Vercel Lambda 용량 한계, cold start 길음, 비용 ↑ | ⭐⭐ |
| `html2pdf.js` / `jsPDF` | 클라이언트 사이드 처리 | 한글 폰트 처리·표 분할 품질 낮음, 크로스 브라우저 이슈 | ⭐ |

**최종 추천**: **`@react-pdf/renderer`**
- 이유: 서버리스 환경(Vercel)에서 안정적이고, 한글 폰트(Pretendard, Noto Sans KR) 임베드가 검증되어 있음. 웹 컴포넌트와 PDF 컴포넌트를 분리하는 비용은 발생하지만, **공통 도메인 타입(`Quote`)을 props로 받는 얇은 레이아웃 컴포넌트**로 추상화하면 유지보수 부담이 작음.

```ts
// 예시: PDF 라우트
// src/app/api/q/[slug]/pdf/route.ts
import { renderToStream } from "@react-pdf/renderer";
import { QuotePdf } from "@/components/pdf/quote-pdf";
import { getQuoteBySlug } from "@/lib/notion/quote.service";

export async function GET(
  _req: Request,
  { params }: { params: Promise<{ slug: string }> }
) {
  const { slug } = await params;
  const quote = await getQuoteBySlug(slug);
  if (!quote) return new Response("Not Found", { status: 404 });

  const stream = await renderToStream(<QuotePdf quote={quote} />);
  return new Response(stream as unknown as ReadableStream, {
    headers: {
      "Content-Type": "application/pdf",
      "Content-Disposition": `attachment; filename="quote-${quote.quoteNumber}.pdf"`,
    },
  });
}
```

---

## 9. ⚙️ 비기능 요구사항 (Non-functional Requirements)

| 카테고리 | 요구사항 |
|------|------|
| **성능** | 견적서 페이지 LCP < 2.5초, PDF 생성 < 5초 (P95) |
| **SEO/프라이버시** | 모든 `/q/[slug]` 페이지에 `<meta name="robots" content="noindex,nofollow">` |
| **보안** | slug는 `nanoid(10)`로 추측 불가, Notion Token은 서버 전용(`process.env`)에서만 접근 |
| **접근성** | WCAG AA, shadcn/ui 기본 a11y 준수, 모든 액션 버튼에 `aria-label` |
| **에러 핸들링** | App Router의 `not-found.tsx` / `error.tsx`로 일관된 폴백 UI 제공 |
| **로깅** | 서버 사이드 에러는 구조화된 로깅(예: `pino`), `console.log` 사용 금지 |
| **환경 변수 검증** | 부팅 시 `zod` 스키마로 `NOTION_TOKEN`, `NOTION_DB_ID` 검증 |

---

## 10. 🚀 마일스톤 / 로드맵 (Milestones)

### Week 1 — 기반 다지기
- [x] Next.js 15.5.3 + shadcn/ui 초기 세팅 (완료)
- [ ] Notion Integration 생성 + DB 스키마 확정
- [ ] `@notionhq/client` 연동 + `Quote` 도메인 타입 정의
- [ ] 환경 변수 검증 레이어 (`zod`)
- **산출물**: Notion → `Quote` 변환 단위 테스트 통과, 콘솔에서 견적서 1건 조회 성공

### Week 2 — 웹 뷰 구현
- [ ] `/q/[slug]` 페이지 라우트 + `loading.tsx` / `not-found.tsx` / `error.tsx`
- [ ] 견적서 레이아웃 컴포넌트 (Card / Table / Separator)
- [ ] 반응형 (sm/md/lg) 검수
- **산출물**: 모바일/데스크탑에서 실제 견적서 데이터로 렌더링 데모

### Week 3 — PDF 생성
- [ ] `@react-pdf/renderer` 도입 + 한글 폰트(Pretendard) 임베드
- [ ] `QuotePdf` 컴포넌트 (웹 레이아웃과 시각적 동일성 확보)
- [ ] `/api/q/[slug]/pdf` 라우트 구현
- [ ] 다운로드 버튼 상태 머신 (idle/loading/success/error)
- **산출물**: PDF 다운로드 E2E 성공, 견적서 1건 PDF 시각 검수 통과

### Week 4 — 마감 및 출시
- [ ] 캐싱(`revalidate`) + 에러 로깅
- [ ] 랜딩 페이지(`/`) 사용법 안내
- [ ] Lighthouse / 접근성 점검 (목표 점수 90+)
- [ ] Vercel 배포 + 도메인 연결
- [ ] 베타 사용자 3명에게 공유 후 피드백 수집
- **산출물**: 운영 환경 URL, 베타 피드백 노트, v0.1.0 태그

---

## 11. ❓ 열린 질문 & 가정 (Open Questions & Assumptions)

### 결정 필요 항목
1. **공유 URL 만료 정책**: 영구 유지 vs 발행일 + 90일 자동 비활성? (개인정보 보호 관점)
2. **PDF 폰트 라이선스**: Pretendard(OFL) vs Noto Sans KR(SIL OFL) — 임베드 시 라이선스 명시 위치는?
3. **다중 발행자 지원 시점**: MVP는 단일 발행자(고정 `NOTION_TOKEN`) 전제. Phase 2에서 OAuth로 전환 시점은?
4. **slug ↔ Notion Page ID 매핑 저장소**: MVP는 환경변수/JSON으로 충분하지만, 견적서가 50건을 넘기는 시점부터는 KV 또는 SQLite 필요.
5. **공유 URL 분석**: 단순 조회수만으로 충분한가? UTM 파라미터 지원 여부?

### 가정 (Assumptions)
- 발행자는 Notion 사용에 익숙하며, Integration 연결 가이드를 1회 따라할 수 있다.
- 클라이언트는 최신 브라우저(Chrome/Safari/Edge) + 한국어 환경을 사용한다.
- Vercel(또는 동급 PaaS) 무료/저가 플랜으로 MVP 트래픽(~월 1,000 페이지뷰)을 감당할 수 있다.
- 견적서 한 건의 항목 수는 일반적으로 50개 이하이다.

---

## 12. 📎 부록 (Appendix)

### 참고 링크
- [Notion API Docs](https://developers.notion.com/)
- [@react-pdf/renderer](https://react-pdf.org/)
- [Next.js 15 App Router](https://nextjs.org/docs/app)
- [shadcn/ui (new-york)](https://ui.shadcn.com/)
- [Pretendard 폰트](https://github.com/orioncactus/pretendard)

### 용어 정의
| 용어 | 정의 |
|----|----|
| **발행자(Issuer)** | 견적서를 작성·전달하는 프리랜서/사업자 |
| **수신자(Recipient)** | 견적서를 전달받는 클라이언트 |
| **slug** | 공유 URL 경로의 추측 불가능한 식별자 (예: `kT9_aB4xQz`) |
| **MVP** | Minimum Viable Product — 최소 기능 제품 |

### 유사 서비스 벤치마킹
| 서비스 | 장점 | 본 MVP와의 차이 |
|------|----|------------|
| **Bonsai** | 견적서 + 계약 + 청구서 통합 | 가입/결제 필요, 무거움 |
| **Notion 페이지 공유** | 가장 간편 | 디자인 통제 불가, PDF 품질 낮음 |
| **invoice.so** | 깔끔한 인보이스 빌더 | 직접 입력 필요 (Notion 연동 없음) |

### 관련 문서
- `@/CLAUDE.md` — 프로젝트 개발 지침
- `@/docs/PRD_PROMPT.md` — 본 PRD 작성에 사용된 메타 프롬프트
- `@/docs/guides/project-structure.md` — 폴더 구조 가이드
- `@/docs/guides/nextjs-15.md` — Next.js 15 전문 가이드

---

> 작성일: 2026-06-14
> Version: 0.1.0 (MVP Draft)
