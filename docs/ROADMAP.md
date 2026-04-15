# Notion CMS YouTube TOP 10 큐레이션 개발 로드맵

> 최종 업데이트: 2026-04-15 | PRD 버전: MVP v1.0

---

## 📋 개요

### 프로젝트 목표

Notion DB에서 큐레이션한 YouTube 채널들의 인기 영상을 기간별(최신 1h / 일간 / 주간 / 월간) TOP 10으로 제공하는 단일 페이지 MVP 서비스를 구축한다.

### 전체 일정 개요

| Phase | 내용 | 기간 |
|-------|------|------|
| Phase 0 | 프로젝트 셋업 | 0.5일 |
| Phase 1 | 백엔드 데이터 파이프라인 (F001~F003, F007, F008) | 1.5일 |
| Phase 2 | 프론트엔드 UI 구현 (F004~F006) | 1.5일 |
| Phase 3 | 자동화 파이프라인 강화 + 반응형 디자인 | 1일 |
| Phase 4 | 배포 및 검증 | 0.5일 |
| **합계** | | **5일** |

### 핵심 성공 지표

- YouTube Data API 일일 쿼터 10,000 units 이내 운용 (1회 수집 약 60 units × 144회/일 = 8,640 units)
- ISR revalidate: 600 으로 캐싱 → 시간당 실제 API 호출 6회 이하
- 무효 채널/비공개 영상/쿼터 초과 모든 예외 상황 100% 런타임 자동 처리 (운영자 개입 없음)
- 단일 페이지 서비스 (`/` 루트만 존재)

---

## 🔄 개발 워크플로우

각 Task는 아래 5단계 루프를 **반드시 준수**한다. API 연동·비즈니스 로직·폼 처리·인증이 포함된 Task는 Playwright MCP E2E 테스트 통과 전까지 완료로 간주하지 않는다.

```
1. 기능 구현         → 작업 파일 명세에 따라 코드 작성
2. 테스트 시나리오    → 정상(Happy) / 엣지 / 에러 3종 시나리오 문서화
3. Playwright MCP   → E2E 테스트 실행 (아래 도구 활용)
4. 실패 시 수정       → 원인 분석 → 코드 수정 → 재실행 (건너뛰기 금지)
5. 통과 확인 후 완료  → ROADMAP 체크박스 ✅ 표시 → 다음 Task 진행
```

### Playwright MCP 도구 활용

| 도구 | 활용 목적 |
|------|----------|
| `mcp__playwright__browser_navigate` | 페이지 진입 및 로드 확인 |
| `mcp__playwright__browser_snapshot` | DOM/UI 상태 스냅샷 검증 |
| `mcp__playwright__browser_click` | 버튼·카드·탭 클릭 재현 |
| `mcp__playwright__browser_fill_form` | 폼 입력 |
| `mcp__playwright__browser_wait_for` | 비동기 로딩 완료 대기 |
| `mcp__playwright__browser_network_requests` | YouTube/Notion API 요청 검증 |
| `mcp__playwright__browser_console_messages` | 콘솔 에러 감지 |
| `mcp__playwright__browser_take_screenshot` | 반응형/시각 회귀 검증 |

### 표준 E2E 플로우 (홈 페이지 기준)

```
1. browser_navigate  → http://localhost:3000 진입
2. browser_snapshot  → 초기 탭(최신 1h) 렌더링 상태 확인
3. browser_click     → 일간 / 주간 / 월간 탭 전환
4. browser_wait_for  → 데이터 로딩 완료 대기
5. browser_click     → 영상 카드 클릭 → Dialog 오픈 확인
6. browser_network_requests → Notion/YouTube API 호출 여부 + 상태코드 확인
7. browser_console_messages → 콘솔 에러 없음 확인
```

### 작업 완료 기준 (DoD)

- [ ] `npm run check-all` 통과 (타입/ESLint)
- [ ] Playwright MCP 3종 시나리오(정상/엣지/에러) 전부 통과
- [ ] 콘솔 에러 없음
- [ ] 반응형(모바일 375px / 태블릿 768px / 데스크탑 1440px) 스냅샷 확인

---

## 🎯 마일스톤 개요

| 마일스톤 | 목표 | 완료 조건 | 상태 |
|---------|------|----------|------|
| M1: 데이터 파이프라인 완성 | Notion → YouTube 3단계 체인 정상 동작 | `getActiveChannels` + `fetchAllPeriodVideos` 성공 응답 | ✅ 완료 |
| M2: 핵심 UI 완성 | TOP 10 카드 + 탭 전환 + Dialog 재생 | 4개 탭 전환 및 영상 클릭 시 iframe 재생 동작 | ✅ 완료 |
| M3: 자동화 파이프라인 완성 | 무인 운영 99.9% 달성 | F008 전체 항목 동작 검증 | 🔲 미완료 |
| M4: 프로덕션 배포 | Vercel 배포 완료 | 실제 도메인에서 서비스 정상 동작 | 🔲 미완료 |

---

## 📅 Phase별 상세 계획

### Phase 0: 프로젝트 셋업 (준비 단계)

> 목표: 개발 환경 구성 및 환경변수 설정 완료

**🤔 왜 이 순서인가?**
- 타입 정의(`Video`, `Channel`, `HomePageData`)가 Phase 1·2의 공통 계약이 됨 → 먼저 확정해야 백엔드/UI 병렬 개발 가능
- 환경변수(`NOTION_API_KEY`, `YOUTUBE_API_KEY`) 없이는 Phase 1의 API 호출 자체가 불가능
- zod 검증을 빌드 시점에 두어 런타임에서 환경변수 누락으로 인한 장애를 사전 차단

- [x] **환경변수 파일 구성** — `.env.local` 파일 생성 및 3개 키(`NOTION_API_KEY`, `NOTION_DATABASE_ID`, `YOUTUBE_API_KEY`) 설정 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - `.env.local.example` 파일 참조하여 실제 값 주입
  - Notion 통합(Integration)에 DB read + write 권한 부여 확인

- [x] **타입 정의** — `src/types/index.ts`에 `PeriodTab`, `Video`, `Channel`, `RankedVideos`, `HomePageData`, `QuotaExceededError` 인터페이스 정의 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - `any` 타입 사용 금지, 모든 필드 명확한 타입 부여
  - `publishedAt`: ISO 8601 string 타입 명시

- [x] **환경변수 검증 모듈** — `src/lib/env.ts`에 zod 스키마로 빌드 시점 환경변수 누락 검증 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴

**Phase 0 완료 기준:**
- `npm run dev` 실행 시 오류 없이 개발 서버 기동
- 3개 환경변수 모두 `.env.local`에 설정 완료
- `src/types/index.ts` 컴파일 오류 없음

---

### Phase 1: 백엔드 데이터 파이프라인 구현

> 목표: Notion → YouTube 3단계 체인 완성, 기간 필터링 로직 구현, ISR 캐싱 적용

**🤔 왜 이 순서인가?**
- **UI보다 데이터 파이프라인 먼저**: 서비스의 본질은 "YouTube TOP 10 큐레이션 데이터" 자체. 데이터 없이 UI를 만들면 나중에 실제 응답 구조와 충돌 발생
- **Notion → YouTube 순서**: YouTube API는 채널 목록이 있어야 호출 가능 → Notion이 데이터의 출발점
- **3단계 체인(channels → playlistItems → videos)은 분리 불가**: 각 단계의 출력이 다음 단계의 입력 → 순차 구현 필수
- **기간 필터링이 ISR 설정보다 먼저**: 필터 로직이 완성돼야 캐싱 단위가 정해짐 (4탭 단일 수집 전략의 전제)
- **ISR을 Phase 1 마지막에 배치**: 데이터 수집 로직이 안정화된 후 캐싱을 씌워야 디버깅 용이 (캐싱이 먼저면 코드 수정이 즉시 반영되지 않아 개발 속도 저하)

#### 1-1. Notion API 연동 (F001, F008-1, F008-2)

- [x] **Notion 클라이언트 구현** — `src/lib/notion.ts`에 `@notionhq/client` 싱글턴 생성 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - `getActiveChannels()`: `active=true` 채널 최대 20개 조회
  - `updateUploadsPlaylistId()`: `uploads_playlist_id` Notion write-back
  - `deactivateChannel()`: 무효 채널 `active=false` 자동 전환 + `error_reason` 기록
  - Notion v5 API (`dataSources.query`) 사용

- [x] **채널 데이터 매핑** — `PageObjectResponse` → `Channel` 타입 변환 헬퍼 구현 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - `rich_text`, `title`, `checkbox` 속성 타입별 추출 처리

#### 1-2. YouTube Data API 3단계 체인 (F002)

- [x] **YouTube 클라이언트 구현** — `src/lib/youtube.ts`에 `googleapis` 싱글턴 생성 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴

- [x] **1단계: channels.list 조회** — `fetchUploadsPlaylistId(channelId)` 구현 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - `part=contentDetails`, `id=channel_id` 파라미터 사용
  - 응답 `items` 비어있으면 `null` 반환 (무효 채널 판별)
  - `uploads_playlist_id` 이미 존재 시 호출 생략 (쿼터 절약)

- [x] **2단계: playlistItems.list 수집** — `collectVideoIds(playlistIds)` 구현 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - `part=contentDetails`, `maxResults=50`으로 채널당 최대 50개 영상 수집
  - `contentDetails.videoId` + `contentDetails.videoPublishedAt` 추출
  - **`publishedAfter` 파라미터 절대 사용 금지** (search.list 전용, 100 units/req)
  - 개별 채널 실패 시 스킵하고 전체 수집 계속 진행

- [x] **3단계: videos.list 배치 조회** — `fetchVideoDetails(videoItems)` 구현 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - `part=['snippet', 'statistics']`, 50 id/요청 배치 처리
  - 썸네일 우선순위: `maxres > high > medium > default`
  - `videos.list` 응답에 없는 videoId = 비공개/삭제 영상 → 자동 제외 (F008-4)
  - `viewCount` parseInt 변환 처리

#### 1-3. 기간 필터링 로직 (F003)

- [x] **서버 런타임 기간 필터링** — `buildHomePageData(allVideos)` 구현 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - `PERIOD_DURATION_MS`: 1h / 24h / 7d / 30d 밀리초 상수 정의
  - `videoPublishedAt`과 `Date.now()` 서버 런타임 비교
  - 단일 수집 후 메모리 필터링으로 4탭 처리 (탭별 API 호출 금지)
  - 조회수 내림차순 정렬 후 TOP 10 슬라이싱
  - `isInsufficient`: 필터 결과 10개 미만 여부 플래그

#### 1-4. ISR 캐싱 설정 (F007)

- [x] **ISR revalidate 설정** — `src/app/page.tsx` 상단에 `export const revalidate = 600` 선언 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - Next.js App Router 라우트 세그먼트 설정으로 10분 캐싱 적용

#### 1-5. 홈 페이지 서버 컴포넌트 오케스트레이터

- [x] **데이터 수집 오케스트레이터** — `src/app/page.tsx`에 `getHomePageData()` 함수 구현 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - F001: 채널 목록 조회
  - F008-1: `uploadsPlaylistId` 공란 채널 자동 조회 + write-back
  - F008-2: 무효 채널 자동 비활성화
  - F002~F005: 영상 수집 + 기간 필터링 + TOP 10 정렬
  - F008-5: `try-catch`로 오류 시 `null` 반환 → 마지막 ISR 캐시 서빙

**Phase 1 완료 기준:**
- `getActiveChannels()` 실행 시 Notion DB에서 채널 목록 정상 조회
- `fetchAllPeriodVideos()` 실행 시 4개 기간별 TOP 10 데이터 반환
- `npm run build` 성공 (타입 오류 없음)

---

### Phase 2: 프론트엔드 UI 구현

> 목표: shadcn/ui 기반 TOP 10 카드, 기간 탭 전환, Dialog 임베드 재생 완성

**🤔 왜 이 순서인가?**
- **shadcn/ui 설치가 모든 UI Task의 전제**: Tabs·Dialog·Card 컴포넌트가 없으면 후속 컴포넌트 구현 불가 → 가장 먼저 설치
- **VideoCard → VideoDialog → PeriodTabs 순서**: PeriodTabs가 하위에서 VideoCard·VideoDialog를 사용하는 합성 관계 → 상향식(bottom-up) 조립
  - VideoCard: 가장 작은 단위(리프), 의존성 없음 → 최우선
  - VideoDialog: 단독 동작 가능, VideoCard와 독립적 → 병렬 개발 가능
  - PeriodTabs: 위 두 컴포넌트를 조합 + 상태 관리 → 마지막
- **홈 페이지 최종 연결은 맨 끝**: Phase 1의 `getHomePageData()`와 Phase 2 컴포넌트가 모두 준비된 후에야 임시 구조를 정식 UI로 교체 가능
- **next/image 도메인 설정을 VideoCard와 같은 단위로 묶음**: 썸네일 렌더링 실패는 런타임에서만 발견되므로 컴포넌트 구현 시점에 함께 설정해야 테스트 루프가 짧아짐

#### 2-1. shadcn/ui 컴포넌트 설치

- [x] **UI 컴포넌트 설치** — 필요 컴포넌트 추가 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  ```bash
  npx shadcn@latest add tabs   # ✅ 설치 완료
  # dialog, card 는 이미 설치되어 있음
  ```

#### 2-2. VideoCard 컴포넌트 (F004)

- [x] **VideoCard 컴포넌트 구현** — `src/components/video/VideoCard.tsx` | 담당: 개발자, 예상공수: 1.5h, 우선순위: 🔴
  - ✅ Props: `video: Video`, `rank: number`, `onClick: () => void`
  - ✅ 순위 뱃지 (1위: 금, 2위: 은, 3위: 동 강조)
  - ✅ 썸네일 이미지 (`next/image` + YouTube 도메인 허용)
  - ✅ 영상 제목 2줄 말줄임 (`line-clamp-2`)
  - ✅ 채널명 + 조회수 (`1,234,567회` 포맷)
  - ✅ 호버 피드백 (`hover:shadow-md`, 썸네일 스케일 효과)

- [x] **next.config.ts 이미지 도메인 설정** — YouTube 썸네일 도메인 허용 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - ✅ `i.ytimg.com`, `img.youtube.com` remotePatterns 등록

#### 2-3. VideoDialog 컴포넌트 (F006)

- [x] **VideoDialog 컴포넌트 구현** — `src/components/video/VideoDialog.tsx` | 담당: 개발자, 예상공수: 1.5h, 우선순위: 🔴
  - ✅ shadcn/ui `Dialog` 기반 모달
  - ✅ `<iframe src="youtube.com/embed/{videoId}?autoplay=1">` 임베드
  - ✅ `useEffect`로 닫힐 때 iframe src 초기화 (백그라운드 재생 방지)
  - ✅ `aspect-video` 16:9 비율 유지
  - ✅ 영상 제목, 채널명 DialogHeader에 표시

#### 2-4. PeriodTabs 컴포넌트 (F005)

- [x] **PeriodTabs 클라이언트 컴포넌트 구현** — `src/components/video/PeriodTabs.tsx` | 담당: 개발자, 예상공수: 1.5h, 우선순위: 🔴
  - ✅ `'use client'` 지시어
  - ✅ shadcn/ui `Tabs`로 최신(1h) / 일간 / 주간 / 월간 탭 구현, 기본값 `latest`
  - ✅ 단일 `data` prop 메모리 필터링 (탭별 API 재호출 없음)
  - ✅ `useState`로 `selectedVideo` / `dialogOpen` 상태 관리
  - ✅ 데이터 부족 시 `isInsufficient` 안내 인라인 렌더링 (별도 컴포넌트 불필요)

#### 2-5. 홈 페이지 서버 컴포넌트 최종 연결

- [x] **홈 페이지 최종 연결** — `src/app/page.tsx` 임시 구조 제거 후 정식 컴포넌트로 교체 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - ✅ `<PeriodTabs data={data} />` 렌더링
  - ✅ `data === null` 시 `ErrorState` 컴포넌트 표시
  - ✅ `npm run check-all` 통과

**Phase 2 완료 기준:**
- [x] 4개 탭 클릭 시 해당 기간 TOP 10 목록으로 즉시 전환 (페이지 이동/리로드 없음)
- [x] 영상 카드 클릭 시 Dialog 모달 내 YouTube iframe 자동재생 (`autoplay=1`)
- [x] 모달 닫기 시 영상 재생 중단 (iframe src 초기화)
- [x] `npm run check-all` 통과 (타입/ESLint/Prettier)

---

### Phase 3: 자동화 파이프라인 강화 + 반응형 디자인

> 목표: F008 전체 완성, 모든 기기에서 최적화된 레이아웃 제공

**🤔 왜 이 순서인가?**
- **자동화 검증을 배포 전에 수행**: 무인 운영이 서비스의 핵심 가치 → 프로덕션 배포 후 장애 발견은 비용 큼. Preview 환경에서 먼저 검증
- **반응형 디자인을 UI 완성 직후에 배치**: Phase 2에서 데스크탑 기준으로 구현된 UI의 레이아웃을 모바일/태블릿에 맞춰 조정하는 "튜닝" 단계 → 컴포넌트가 완성된 직후가 적기
- **자동화 검증과 반응형 디자인은 병렬 가능**: 두 작업의 코드 영역이 겹치지 않음 (데이터 레이어 vs 스타일 레이어)
- **F008 번호 순서(3→4→5)로 검증**: ISR 재수집(F008-3)이 동작해야 비공개 영상 필터링(F008-4)을 실제 트리거에서 관찰 가능, quotaExceeded 폴백(F008-5)은 API 키 무효화 시뮬레이션 필요하므로 마지막

#### 3-1. 자동화 파이프라인 검증 및 강화 (F008)

- [ ] **F008-3: ISR 자동 재수집 검증** — 10분마다 파이프라인 자동 실행 동작 확인 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - `revalidate: 600` 설정으로 Next.js가 백그라운드 재생성 처리
  - Vercel Preview 환경에서 캐시 갱신 타이밍 확인

- [ ] **F008-4: 비공개/삭제 영상 자동 제외 검증** — `videos.list` 누락 videoId 필터링 동작 확인 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - `videoMap.get(id)` 미존재 시 자동 제외 로직 단위 테스트

- [ ] **F008-5: quotaExceeded 폴백 검증** — 403 오류 시 캐시 서빙 동작 확인 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - `try-catch`에서 `null` 반환 → ISR 마지막 캐시 서빙 흐름 확인
  - YouTube API 키 임시 무효화로 403 시나리오 시뮬레이션

- [ ] **error_reason 기록 검증** — 무효 채널 자동 비활성화 후 Notion DB에 `error_reason` 기록 확인 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🟡
  - 실제 잘못된 `channel_id`를 Notion DB에 추가 후 자동 비활성화 흐름 검증

#### 3-2. 반응형 디자인 적용

- [ ] **모바일 레이아웃 최적화** — VideoCard 모바일 최적화 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - 모바일(< 640px): 세로형 카드 레이아웃 (썸네일 상단, 정보 하단)
  - 태블릿(640px~): 가로형 카드 레이아웃 (썸네일 좌측, 정보 우측)
  - 데스크탑(1024px~): `max-w-4xl` 컨테이너 중앙 정렬

- [ ] **Dialog 모바일 최적화** — 모바일에서 Dialog 전체화면에 가깝게 표시 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🟡
  - `sm:max-w-3xl` Dialog 너비 설정
  - 모바일 세로 화면에서 16:9 iframe 비율 유지 확인

- [ ] **탭 스크롤 처리** — 좁은 화면에서 탭 가로 스크롤 또는 줄바꿈 처리 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🟡
  - `overflow-x-auto` 또는 `flex-wrap` 적용

**Phase 3 완료 기준:**
- 모바일(375px), 태블릿(768px), 데스크탑(1440px) 모든 해상도에서 레이아웃 정상
- F008 1~5 항목 모두 동작 확인

---

### Phase 4: Vercel 배포 및 검증

> 목표: 프로덕션 배포 완료 및 최종 검증

**🤔 왜 이 순서인가?**
- **배포를 가장 마지막에**: Phase 0~3까지의 모든 검증이 끝난 후에만 프로덕션 배포 → 롤백 비용과 사용자 영향 최소화
- **Vercel 선택 이유**: Next.js 15 App Router + ISR을 가장 잘 지원하며, 환경변수 관리 UI가 내장되어 운영자 수동 작업 최소화 목표와 부합
- **환경변수 설정 → 빌드 검증 → E2E 검증 → 쿼터 확인 순서**: 앞 단계가 실패하면 뒷 단계는 의미가 없음 (fail-fast). 특히 쿼터 확인은 실제 트래픽 발생 후에만 정확한 수치 확인 가능
- **프로덕션 E2E를 Playwright MCP로 재실행**: 로컬/Preview와 프로덕션의 환경변수·캐시 헤더·CDN 동작이 다를 수 있음 → 실제 도메인 기준 최종 검증

- [ ] **Vercel 프로젝트 연결** — GitHub 저장소 연동 및 Vercel 프로젝트 생성 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴

- [ ] **Vercel 환경변수 설정** — Production/Preview 환경변수 주입 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  ```
  NOTION_API_KEY=secret_...
  NOTION_DATABASE_ID=...
  YOUTUBE_API_KEY=AIza...
  ```

- [ ] **프로덕션 빌드 검증** — `npm run build` 성공 및 Vercel 빌드 로그 확인 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🔴
  - 타입 오류, ESLint 오류 없음
  - `next/image` 도메인 설정 확인

- [ ] **실제 서비스 동작 검증** — Vercel 프로덕션 URL에서 E2E 체크 | 담당: 개발자, 예상공수: 1h, 우선순위: 🔴
  - TOP 10 데이터 정상 표시 확인
  - 4개 탭 전환 정상 동작 확인
  - Dialog 영상 재생 확인
  - ISR 캐시 헤더(`x-vercel-cache: HIT/MISS`) 확인
  - YouTube API 쿼터 소모량 Google Cloud Console에서 확인

- [ ] **채널 수집 현황 확인** — Notion DB에서 자동화 파이프라인 결과 검증 | 담당: 개발자, 예상공수: 0.5h, 우선순위: 🟡
  - `uploads_playlist_id` write-back 성공 여부
  - 무효 채널 자동 비활성화 여부

**Phase 4 완료 기준:**
- Vercel 프로덕션 URL에서 서비스 정상 동작
- YouTube API 일일 쿼터 8,640 units 이내 소모 (144회 × 60 units)
- `npm run check-all` 통과

---

## 🔗 의존성 맵

```
Phase 0 (타입 정의)
    ↓
Phase 1-1 (Notion 클라이언트)  ←→  Phase 1-2 (YouTube 클라이언트)
    ↓                                   ↓
Phase 1-3 (기간 필터링 로직)
    ↓
Phase 1-4 (ISR 캐싱 설정)
    ↓
Phase 1-5 (오케스트레이터: getHomePageData)
    ↓
Phase 2-1 (shadcn/ui 컴포넌트 설치)
    ↓
Phase 2-2 (VideoCard) ─────────────┐
Phase 2-3 (VideoDialog) ───────────┤
Phase 2-4 (PeriodTabs) ────────────┤  (병렬 개발 가능)
Phase 2-5 (InsufficientNotice) ────┘
    ↓
Phase 2-6 (홈 페이지 최종 연결)
    ↓
Phase 3-1 (자동화 파이프라인 검증)
Phase 3-2 (반응형 디자인)  (병렬 진행 가능)
    ↓
Phase 4 (Vercel 배포)
```

### 주요 제약 사항

| 제약 | 이유 | 적용 위치 |
|------|------|----------|
| `publishedAfter` 파라미터 사용 금지 | `search.list` 전용 (100 units/req), 쿼터 즉시 초과 | F002, F003 |
| 탭별 분리 API 호출 금지 | 쿼터 4배 초과 위험 | F005 |
| 추가 라우트 생성 금지 | 단일 페이지 서비스 (`/` 만 존재) | 전체 |
| `any` 타입 사용 금지 | 타입 안전성 | 전체 |
| `revalidate: 600` 필수 | YouTube API 쿼터 보호 | F007 |
| 채널 최대 20개 | YouTube API 일일 쿼터 10,000 units 이내 운용 | F001 |

---

## ⚠️ 리스크 및 대응 방안

| 리스크 | 영향도 | 확률 | 대응 방안 |
|-------|-------|------|----------|
| YouTube API 쿼터 초과 (일 10,000 units) | 🔴 높음 | 중간 | ISR revalidate:600으로 시간당 6회 제한, 채널 수 20개 상한, quotaExceeded 시 캐시 폴백 (F008-5) |
| Notion API v5 `dataSources.query` 호환성 | 🟡 중간 | 낮음 | @notionhq/client 최신 버전 사용, API 응답 타입 검증 강화 |
| YouTube iframe 자동재생 차단 | 🟡 중간 | 높음 | `autoplay=1` + `muted=1` 조합으로 브라우저 정책 대응, 사용자 제스처(카드 클릭) 이후 재생이므로 정책 준수 |
| 최신(1h) 탭 데이터 부족 | 🟢 낮음 | 높음 | `isInsufficient` 플래그로 "현재 업로드 영상이 부족합니다" 안내 표시 (F003) |
| Notion write-back 실패 | 🟡 중간 | 낮음 | write-back 실패해도 수집은 계속 진행, 다음 revalidate 시 재시도 |
| `next/image` YouTube 썸네일 도메인 미허용 | 🟡 중간 | 낮음 | `next.config.ts` `remotePatterns`에 `i.ytimg.com`, `img.youtube.com` 사전 등록 |

---

## 🚫 범위 외 (Out of Scope)

다음 기능은 이번 MVP 로드맵에서 **제외**되며 이후 버전에서 검토한다.

| 제외 기능 | 이유 |
|----------|------|
| 운영자 채널 관리 UI | Notion에서 직접 관리 가능, MVP 불필요 |
| 사용자 인증 및 회원가입 | 공개 서비스로 인증 없음 |
| 채널 카테고리별 필터링 | MVP 범위 초과 |
| 영상 즐겨찾기 / 저장 기능 | MVP 범위 초과 |
| 실시간 자동 갱신 (폴링/웹소켓) | ISR 10분 캐싱으로 대체 |
| 다크 모드 / 테마 설정 | MVP 범위 초과 |
| 검색 기능 | MVP 범위 초과 |
| 추가 라우트 (예: `/channels`, `/about`) | 단일 페이지 서비스 원칙 |

---

## 📝 작업 완료 기준 (Definition of Done)

### 태스크 단위 DoD

- [ ] TypeScript 컴파일 오류 없음 (`tsc --noEmit`)
- [ ] ESLint 오류 없음 (`npm run lint`)
- [ ] `any` 타입 미사용
- [ ] 비즈니스 로직 핵심 부분 한국어 주석 포함

### Phase 단위 DoD

- [ ] 해당 Phase의 모든 태스크 체크박스 완료
- [ ] `npm run check-all` 통과
- [ ] `npm run build` 성공

### 전체 프로젝트 DoD

- [ ] Vercel 프로덕션 URL에서 서비스 정상 동작
- [ ] 4개 기간 탭 정상 전환
- [ ] 영상 카드 클릭 → Dialog iframe 재생 정상
- [ ] ISR 캐싱 동작 확인 (Vercel `x-vercel-cache` 헤더)
- [ ] YouTube API 일일 쿼터 10,000 units 이내 운용
- [ ] F008 자동화 파이프라인 1~5번 항목 모두 동작 확인
- [ ] 모바일/태블릿/데스크탑 반응형 레이아웃 정상

---

## 📊 현재 구현 상태 (2026-04-15 기준)

| 파일 | 상태 | 내용 |
|------|------|------|
| `src/types/index.ts` | ✅ 완료 | `PeriodTab`, `Video`, `Channel`, `RankedVideos`, `HomePageData`, `QuotaExceededError` |
| `src/lib/env.ts` | ✅ 완료 | zod 환경변수 검증 |
| `src/lib/notion.ts` | ✅ 완료 | `getActiveChannels`, `updateUploadsPlaylistId`, `deactivateChannel` |
| `src/lib/youtube.ts` | ✅ 완료 | `fetchAllPeriodVideos`, `fetchUploadsPlaylistId` (3단계 체인 + 기간 필터링) |
| `src/app/page.tsx` | 🔄 진행중 | 서버 컴포넌트 오케스트레이터 완성, UI는 임시 구조 (Phase 2에서 교체 예정) |
| `src/components/video/VideoCard.tsx` | 🔲 미착수 | Phase 2-2 예정 |
| `src/components/video/VideoDialog.tsx` | 🔲 미착수 | Phase 2-3 예정 |
| `src/components/video/PeriodTabs.tsx` | 🔲 미착수 | Phase 2-4 예정 |

> Phase 0, Phase 1 완료. Phase 2 (UI 구현)부터 시작 가능한 상태.
