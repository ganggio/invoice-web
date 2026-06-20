# 📝 PRD 생성 메타 프롬프트

> 이 문서는 **Notion 기반 견적서 공유 웹 서비스 MVP**의 PRD(Product Requirements Document)를 작성하기 위한 메타 프롬프트입니다.
> Claude Code 또는 LLM에게 이 프롬프트를 전달하면, 일관된 형식과 충분한 깊이의 PRD 초안을 생성합니다.

---

## 🎯 메타 프롬프트 사용법

1. 아래 **"전체 프롬프트"** 섹션의 내용을 복사합니다.
2. Claude Code 채팅창 또는 prd-generator 에이전트에 붙여넣습니다.
3. 필요 시 `<프로젝트 컨텍스트>` 블록의 값을 수정합니다.
4. 생성된 PRD는 `docs/PRD.md`에 저장합니다.

---

## 📌 전체 프롬프트 (복사해서 사용)

````markdown
당신은 솔로 개발자와 소규모 팀을 위한 실용적인 PRD(Product Requirements Document)를 작성하는
시니어 프로덕트 매니저이자 풀스택 아키텍트입니다.

다음 <프로젝트 컨텍스트>를 기반으로 **Notion 기반 견적서 공유 웹 서비스의 MVP PRD**를
한국어로 작성해 주세요.

---

## <프로젝트 컨텍스트>

### 1. 제품 한 줄 정의
- 프리랜서/소규모 사업자가 **Notion에 작성한 견적서 내용을 클라이언트가 웹 링크로 확인하고
  PDF로 다운로드**할 수 있도록 만드는 MVP 서비스.

### 2. 핵심 사용자 흐름 (User Journey)
1. **판매자(Issuer)**: Notion 데이터베이스/페이지에 견적 항목을 작성한다.
2. **시스템**: Notion API를 통해 견적서 데이터를 가져와 웹 페이지로 렌더링한다.
3. **판매자**: 고유 공유 URL을 클라이언트에게 전달한다.
4. **클라이언트(Recipient)**: 링크 접속 → 견적서 확인 → "PDF 다운로드" 버튼 클릭.
5. **시스템**: 동일 레이아웃 그대로 PDF를 생성하여 다운로드 제공.

### 3. 기술 스택 (이미 결정됨)
- **Framework**: Next.js 15.5.3 (App Router + Turbopack)
- **Runtime**: React 19.1.0 + TypeScript 5
- **Styling**: TailwindCSS v4 + shadcn/ui (new-york 스타일)
- **Forms**: React Hook Form + Zod + Server Actions
- **UI**: Radix UI + Lucide Icons
- **외부 연동**: Notion API (@notionhq/client)
- **PDF 생성 후보**: @react-pdf/renderer, puppeteer, html2pdf.js 중 선택 (PRD에서 트레이드오프 비교)

### 4. 제약 사항
- **개발자**: 1인 (솔로 개발), 4주 내 MVP 출시 목표.
- **인증**: MVP 단계에서는 로그인 없음. URL slug 기반 단순 공유 (예: `/q/[slug]`).
- **저장소**: MVP는 DB 없이 Notion을 단일 소스로 사용. 필요 시 캐싱 레이어만 추가.
- **언어**: UI/문서 모두 **한국어** 우선, 향후 영어 확장 가능성 고려.
- **반응형**: 모바일/태블릿/데스크탑 모두 지원 필수.
- **접근성**: shadcn/ui 기본 a11y 규약 준수.

### 5. 명시적으로 MVP 범위에서 제외
- 사용자 회원가입/로그인 시스템
- 결제 처리, 전자 서명
- 다국어 자동 전환
- 이메일 자동 발송 (URL 공유만 지원)
- 견적서 버전 관리/히스토리

---

## <PRD 작성 지침>

### A. 문서 구조 (반드시 다음 순서로 작성)

1. **📋 제품 개요 (Overview)**
   - 한 줄 설명, 타겟 사용자, 해결하려는 문제, 차별점

2. **🎯 목표 및 성공 지표 (Goals & Success Metrics)**
   - 비즈니스 목표 (정성)
   - MVP 성공 지표 (정량, 예: "주간 활성 공유 URL 20개")

3. **👥 페르소나 (Personas)**
   - 판매자 페르소나 1명, 클라이언트 페르소나 1명 (이름/직업/시나리오 포함)

4. **🗺️ 핵심 사용자 시나리오 (User Scenarios)**
   - Happy Path를 3~5단계로 서술
   - 엣지 케이스 최소 3개 (Notion 데이터 누락, 잘못된 slug, PDF 생성 실패 등)

5. **✨ 기능 요구사항 (Functional Requirements)**
   - **MUST HAVE (MVP)**: Notion 연동, 견적서 웹 뷰, PDF 다운로드, 공유 URL
   - **SHOULD HAVE (Phase 2)**: 캐싱, 조회수 카운트
   - **WON'T HAVE (Out of scope)**: 위 "MVP 범위에서 제외" 항목 명시
   - 각 기능은 **유저 스토리 형식** ("~로서, 나는 ~을 원한다, 왜냐하면 ~")으로 기술

6. **🧱 정보 구조 / 데이터 모델 (Data Model)**
   - Notion 데이터베이스 스키마 예시 (속성명, 타입, 필수 여부)
     - 예: `견적번호 (Title)`, `클라이언트명 (Text)`, `발행일 (Date)`,
       `유효기간 (Date)`, `항목 (Relation)`, `총액 (Formula)` 등
   - 항목(Line Item) 서브 스키마: `품목명, 수량, 단가, 합계`
   - 웹/PDF 렌더링 시 사용하는 TypeScript 타입 정의 예시

7. **🎨 UI/UX 요구사항**
   - 페이지 라우트 구조 (App Router 기준):
     - `/` (랜딩 또는 사용 안내)
     - `/q/[slug]` (견적서 조회 페이지)
     - `/q/[slug]/pdf` 또는 PDF 다운로드 API 라우트
   - 견적서 뷰 핵심 섹션: 헤더(발행자/로고) → 클라이언트 정보 → 항목 테이블
     → 합계/세금 → 비고/유효기간 → 액션 버튼
   - shadcn/ui 컴포넌트 매핑 (Card, Table, Button, Separator 등)
   - 반응형 브레이크포인트 (sm/md/lg)
   - PDF 다운로드 버튼의 상태 (idle / loading / success / error)

8. **🔌 외부 연동 (Integrations)**
   - Notion API
     - 인증 방식 (Integration Token, 환경변수 `NOTION_TOKEN`)
     - 사용 엔드포인트 (`databases.query`, `pages.retrieve`)
     - Rate limit 대응 전략
   - PDF 생성 라이브러리 비교 (최소 3가지 옵션의 장단점 + 최종 추천)

9. **⚙️ 비기능 요구사항 (Non-functional Requirements)**
   - 성능: 견적서 페이지 LCP < 2.5초, PDF 생성 < 5초
   - SEO: 공유 URL은 기본적으로 `noindex` (개인정보 보호)
   - 보안: slug는 추측 불가능한 nanoid/uuid 사용
   - 접근성: WCAG AA 수준
   - 에러 핸들링: Notion 응답 실패 시 fallback UI

10. **🚀 마일스톤 / 로드맵 (Milestones)**
    - 4주 일정으로 주차별 분할 (Week 1~4)
    - 각 주차에 산출물(deliverable) 명시

11. **❓ 열린 질문 & 가정 (Open Questions & Assumptions)**
    - 결정이 필요한 항목 리스트 (예: "공유 URL 만료 정책?", "PDF 폰트 라이선스?")

12. **📎 부록 (Appendix)**
    - 참고 링크, 용어 정의, 유사 서비스 벤치마킹

### B. 작성 톤 & 형식

- **한국어**로 작성하되, 기술 용어와 라이브러리명은 원문 유지.
- 마크다운 헤딩(`##`, `###`)과 이모지를 활용해 가독성 확보.
- 표(Table)는 마크다운 표 문법으로 작성.
- 코드 블록은 ` ```ts `, ` ```tsx ` 등 언어 태그 명시.
- 추측이 필요한 부분은 **"가정: ~"** 으로 명시.
- 분량: 충분히 상세하되, 불필요한 반복 금지 (목표 분량 약 1,500~2,500 단어).

### C. 품질 체크리스트 (작성 후 자체 검증)

PRD 작성을 마친 후 다음을 자체 검증하고, 미흡한 항목은 보강하세요:

- [ ] 솔로 개발자가 4주 안에 구현 가능한 범위인가?
- [ ] MUST/SHOULD/WON'T 우선순위가 명확한가?
- [ ] Notion 데이터 모델이 실제 견적서 작성 흐름에 맞는가?
- [ ] PDF 생성 방식 선택 근거가 명확한가?
- [ ] 엣지 케이스와 에러 처리가 누락 없이 다뤄졌는가?
- [ ] CLAUDE.md의 기술 스택과 코딩 규칙에 부합하는가?
- [ ] 마일스톤이 측정 가능한 산출물로 표현되었는가?

---

## <출력 형식>

- 단일 마크다운 문서로 출력.
- 파일 저장 경로 제안: `docs/PRD.md`
- 첫 줄은 `# 📋 [제품명] - Product Requirements Document (MVP)` 형식.
- 마지막에 작성일자(`> 작성일: YYYY-MM-DD`)와 버전(`> Version: 0.1.0 (MVP Draft)`) 명시.

이제 위 지침에 따라 PRD를 작성해 주세요.
````

---

## 🧩 메타 프롬프트 구성 원리 (참고용)

이 메타 프롬프트는 다음과 같은 PE(Prompt Engineering) 패턴을 사용합니다:

| 패턴 | 적용 위치 | 효과 |
|------|-----------|------|
| **Role Assignment** | "당신은 시니어 PM이자 풀스택 아키텍트입니다" | 응답의 전문성과 톤 일관성 확보 |
| **Context Grounding** | `<프로젝트 컨텍스트>` 블록 | 환각 방지, 프로젝트 제약 반영 |
| **Explicit Scope Boundary** | "MVP 범위에서 제외" 명시 | 범위 확장(scope creep) 방지 |
| **Structured Output** | "문서 구조 (반드시 다음 순서로)" | 일관된 산출물 형식 보장 |
| **Self-Verification** | "품질 체크리스트 (작성 후 자체 검증)" | 완성도 자동 점검 |
| **Format Specification** | "출력 형식" 섹션 | 후속 자동화(파일 저장 등) 용이 |

---

## 🔄 변형 사용 시나리오

| 시나리오 | 수정 위치 |
|---------|----------|
| 견적서가 아닌 **청구서**로 변경 | `<프로젝트 컨텍스트>`의 "제품 한 줄 정의" |
| 4주가 아닌 **2주 MVP**로 단축 | "제약 사항" + "마일스톤" 섹션 |
| **다국어 지원**을 MVP에 포함 | "MVP 범위에서 제외" 항목 이동 |
| Notion 대신 **Google Sheets** 사용 | "외부 연동" 및 "데이터 모델" 섹션 |

---

> 작성일: 2026-06-14
> Version: 1.0.0
> 관련 문서: `@/CLAUDE.md`, `@/docs/guides/project-structure.md`
