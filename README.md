# Notion CMS YouTube TOP 10 큐레이션

Notion을 CMS로 활용하여 큐레이션한 YouTube 채널들의 인기 영상을 **기간별 TOP 10**으로 제공하는 웹 서비스입니다. 운영자는 코드 배포 없이 Notion DB에서 채널만 추가/제거하면 되고, 나머지 수집·검증·복구는 런타임이 99.9% 자동화합니다.

## 주요 특징

- **기간별 TOP 10**: 최신(1h) / 일간 / 주간 / 월간 탭
- **Notion CMS**: 비개발자도 Notion에서 채널 관리
- **사이트 내 임베드 재생**: YouTube iframe을 Dialog로 노출 (외부 이탈 없음)
- **99.9% 자동화**: `uploads_playlist_id` 자동 보정, 무효 채널 자동 비활성화, 쿼터 초과 자동 백오프
- **ISR 캐싱**: 10분 주기 자동 재생성 (YouTube API 일일 쿼터 10,000 units 내 안정 운영)

## 기술 스택

- **Framework**: Next.js 15 (App Router + Turbopack), React 19, TypeScript 5.6+
- **Styling/UI**: TailwindCSS v4, shadcn/ui (Dialog, Card, Tabs), Lucide React
- **API**: `@notionhq/client`, `googleapis` (YouTube Data API v3)
- **Deploy**: Vercel

## 시작하기

```bash
# 의존성 설치
npm install

# 환경변수 설정 (.env.local)
NOTION_API_KEY=secret_xxx
NOTION_DATABASE_ID=xxxx
YOUTUBE_API_KEY=xxxx

# 개발 서버 실행
npm run dev
```

브라우저에서 [http://localhost:3000](http://localhost:3000) 을 엽니다.

## 문서

- **📋 PRD**: [`docs/PRD.md`](./docs/PRD.md) — 기능 명세, 데이터 모델, 자동화 정책
- **🧩 컴포넌트 패턴**: [`docs/guides/component-patterns.md`](./docs/guides/component-patterns.md)
- **🎨 스타일링 가이드**: [`docs/guides/styling-guide.md`](./docs/guides/styling-guide.md)
- **⚡ Next.js 15 가이드**: [`docs/guides/nextjs-15.md`](./docs/guides/nextjs-15.md)

## Notion Channels DB 스키마

| 필드 | 타입 | 설명 |
|---|---|---|
| channel_name | text | YouTube 채널 표시 이름 |
| channel_id | text | YouTube 채널 ID (UC...) |
| uploads_playlist_id | text | 업로드 재생목록 ID (공란 시 자동 보정) |
| category | text | 채널 분류 |
| active | checkbox | 수집 대상 여부 (무효 채널은 자동 false 전환) |
| error_reason | text | 자동 비활성화 사유 (자동 기록) |

## 배포

[Vercel](https://vercel.com) 에 연결하고 환경변수 3개(`NOTION_API_KEY`, `NOTION_DATABASE_ID`, `YOUTUBE_API_KEY`)를 설정하면 됩니다.

## 라이선스

MIT
