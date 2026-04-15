# Claude Code 개발 지침

**notion-cms-project**는 Notion DB에서 큐레이션한 YouTube 채널의 인기 영상을 기간별 TOP 10으로 제공하는 단일 페이지 MVP 서비스입니다.

## Project Context

- PRD 문서: @docs/PRD.md
- 개발 로드맵: @docs/ROADMAP.md

## 핵심 기술 스택

- **Framework**: Next.js 15.5.3 (App Router + Turbopack)
- **Runtime**: React 19.1.0 + TypeScript 5
- **Styling**: TailwindCSS v4 + shadcn/ui (new-york style)
- **External API**: @notionhq/client + googleapis (YouTube Data API v3)
- **UI Components**: Radix UI + Lucide Icons
- **Development**: ESLint + Prettier + Husky + lint-staged

## 개발 가이드

- **개발 로드맵**: `@/docs/ROADMAP.md`
- **프로젝트 요구사항**: `@/docs/PRD.md`
- **프로젝트 구조**: `@/docs/guides/project-structure.md`
- **스타일링 가이드**: `@/docs/guides/styling-guide.md`
- **컴포넌트 패턴**: `@/docs/guides/component-patterns.md`
- **Next.js 15.5.3 전문 가이드**: `@/docs/guides/nextjs-15.md`

## 자주 사용하는 명령어

```bash
# 개발
npm run dev         # 개발 서버 실행 (Turbopack)
npm run build       # 프로덕션 빌드
npm run check-all   # 모든 검사 통합 실행 (권장)

# UI 컴포넌트
npx shadcn@latest add tabs    # Tabs 컴포넌트 추가 (기간 탭 전환)
npx shadcn@latest add dialog  # Dialog 컴포넌트 추가 (영상 임베드 재생)
npx shadcn@latest add card    # Card 컴포넌트 추가 (TOP 10 카드)
```

## 작업 완료 체크리스트

```bash
npm run check-all   # 모든 검사 통과 확인
npm run build       # 빌드 성공 확인
```

## 중요 제약사항

- **단일 페이지 서비스**: 추가 라우트(페이지) 생성 금지 — `/` 홈 페이지만 존재
- **탭별 분리 API 호출 금지**: 단일 수집 후 메모리 필터링으로 4탭 처리 (쿼터 4배 초과 방지)
- **any 타입 사용 금지**: `@/types/index.ts`에 정의된 타입 활용
- **ISR revalidate: 600**: 홈 페이지 10분 캐싱 유지 (YouTube API 쿼터 보호)
