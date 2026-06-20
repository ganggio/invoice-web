# invoice-web

Notion에 작성한 견적서를 클라이언트가 웹 링크로 확인하고 PDF로 다운로드할 수 있는 솔로 개발자·프리랜서용 서비스입니다.

발행자는 Notion 견적서 DB에 새 항목을 작성한 뒤 `https://invoice-web.app/q/[slug]` 형태의 공유 URL을 전달하고, 클라이언트는 별도 가입 없이 견적서를 확인하고 PDF로 보관할 수 있습니다.

## 기술 스택

- **Framework**: Next.js 15.5.3 (App Router + Turbopack)
- **Runtime**: React 19.1.0 + TypeScript 5
- **Styling**: TailwindCSS v4 + shadcn/ui (new-york)
- **Forms & Validation**: React Hook Form + Zod
- **Icons**: Lucide React
- **Notifications**: Sonner
- **Notion 연동**: `@notionhq/client` _(예정)_
- **PDF 생성**: `@react-pdf/renderer` _(예정)_
- **Hosting**: Vercel

## 개발 명령어

```bash
npm run dev         # 개발 서버 실행 (Turbopack)
npm run build       # 프로덕션 빌드
npm run start       # 프로덕션 서버 실행
npm run lint        # ESLint 검사
npm run typecheck   # TypeScript 타입 검사
npm run check-all   # typecheck + lint + format 통합 검사 (권장)
```

개발 서버 실행 후 [http://localhost:3000](http://localhost:3000)에서 확인할 수 있습니다.

## 환경변수

프로젝트 루트에 `.env.local` 파일을 생성하고 다음 값을 설정합니다.

```bash
NOTION_TOKEN=secret_xxxxxxxxxxxx          # Notion Internal Integration Token
NOTION_QUOTE_DB_ID=xxxxxxxxxxxxxxxxxxxx   # 견적서 메인 DB ID
NOTION_ITEM_DB_ID=xxxxxxxxxxxxxxxxxxxx    # 항목(Line Item) DB ID
```

Notion Integration 생성 및 DB 스키마 구성 방법은 `docs/PRD.md`의 데이터 모델 섹션을 참고하세요.

## 문서

- [`docs/PRD.md`](./docs/PRD.md) — 제품 요구사항 정의서 (MVP 범위·기능·데이터 모델)
- [`CLAUDE.md`](./CLAUDE.md) — Claude Code 개발 지침
- [`docs/guides/`](./docs/guides/) — Next.js·스타일·컴포넌트·폼 처리 가이드
