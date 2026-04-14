# Notion CMS YouTube TOP 10 큐레이션 MVP PRD

## 핵심 정보

**목적**: Notion에서 큐레이션한 YouTube 채널들의 인기 영상을 기간별 TOP 10으로 제공하여, 사용자가 신뢰할 수 있는 채널 기반의 인기 영상을 한눈에 소비할 수 있게 한다.

**사용자**: YouTube 인기 영상을 큐레이션 기반으로 소비하려는 일반 사용자 (채널 관리는 운영자가 Notion에서 직접 수행)

---

## 사용자 여정

```
1. [홈 페이지] 진입
   ↓ 페이지 로드 시 기본 탭 "최신(1h)" 자동 선택

2. [홈 페이지] 기간 탭 선택
   ↓ 사용자가 최신(1h) / 일간 / 주간 / 월간 탭 중 하나 클릭

   [최신(1h) 탭] → 최근 1시간 이내 업로드된 영상 중 누적 조회수 TOP 10 표시 (10개 미만 시 전체 노출 + 안내)
   [일간 탭]   → 최근 24시간 이내 업로드된 영상 중 누적 조회수 TOP 10 표시
   [주간 탭]   → 최근 7일 이내 업로드된 영상 중 누적 조회수 TOP 10 표시
   [월간 탭]   → 최근 30일 이내 업로드된 영상 중 누적 조회수 TOP 10 표시
   ↓

3. [홈 페이지] TOP 10 랭킹 카드 목록 확인
   ↓ 사용자가 영상 카드 클릭

4. [사이트 내 임베드 재생(Dialog)] → shadcn/ui Dialog 모달 내 YouTube iframe(`youtube.com/embed/{videoId}?autoplay=1`)으로 사이트를 떠나지 않고 즉시 재생
```

---

## 기능 명세

### 1. MVP 핵심 기능

| ID | 기능명 | 설명 | MVP 필수 이유 | 관련 페이지 |
|----|--------|------|--------------|------------|
| **F001** | 채널 목록 조회 | Notion DB에서 active 상태인 채널 목록을 읽어온다. 최대 20개 채널 제한 (YouTube Data API 일일 쿼터 10,000 units 보호) | 큐레이션의 원천 데이터 소스이며, 서비스의 핵심 채널 기반 제공 | 홈 페이지 |
| **F002** | YouTube 영상 수집 | `channels.list(part=contentDetails)`로 채널별 업로드 재생목록 ID를 확보한 뒤, `playlistItems.list`로 영상 ID와 업로드 일시(`contentDetails.videoPublishedAt`)를 수집하고, `videos.list`로 메타데이터(제목, 썸네일, 채널명)와 누적 조회수를 배치(최대 50 id/요청) 조회하는 3단계 체인 | TOP 10 랭킹 구성을 위한 필수 데이터 수집 | 홈 페이지 |
| **F003** | 기간 필터링 | `playlistItems.list` 응답의 `contentDetails.videoPublishedAt`을 서버 런타임에서 현재 시각과 비교하여 기간 윈도우(최신=1h, 일간=24h, 주간=7d, 월간=30d) 내 업로드된 영상만 필터링한다. **`publishedAfter`는 `search.list` 전용 파라미터(100 units/req)이므로 사용하지 않음.** 데이터 부족(10개 미만) 시 해당 기간 영상 전체를 노출하고 "현재 업로드 영상이 부족합니다" 안내 표시 | 기간별 TOP 10 제공을 위한 핵심 로직 | 홈 페이지 |
| **F004** | TOP 10 랭킹 렌더링 | 필터링된 영상을 누적 조회수 내림차순으로 정렬하여 상위 10개를 카드 형태로 표시한다. 카드에는 순위, 썸네일, 제목, 채널명, 조회수가 포함된다 | 서비스의 핵심 가치인 TOP 10 큐레이션 결과물 제공 | 홈 페이지 |
| **F005** | 기간 탭 전환 | 최신(1h) / 일간 / 주간 / 월간 탭 UI를 제공하며, 탭 클릭 시 해당 기간의 TOP 10으로 즉시 전환된다. 기본값은 최신(1h) 탭. 단일 데이터 수집 후 메모리 필터링으로 4탭 모두 처리 (탭별 API 분리 호출 금지 — 쿼터 4배 초과 위험) | 사용자가 원하는 기간의 랭킹을 선택할 수 있는 핵심 인터랙션 | 홈 페이지 |
| **F006** | 사이트 내 임베드 재생 | 영상 카드 클릭 시 shadcn/ui Dialog 모달을 열고 `<iframe src="https://www.youtube.com/embed/{videoId}?autoplay=1">`로 사이트 내에서 즉시 재생한다. 외부 youtube.com 이동은 하지 않는다 | 사용자 이탈 최소화 및 연속 소비 경험 제공 | 홈 페이지 |

### 2. MVP 필수 지원 기능

| ID | 기능명 | 설명 | MVP 필수 이유 | 관련 페이지 |
|----|--------|------|--------------|------------|
| **F007** | ISR 캐싱 | revalidate: 600 (10분) 설정으로 YouTube API 응답을 캐싱하여 불필요한 API 쿼터 소모를 방지한다 | YouTube Data API 일일 쿼터 10,000 units 보호 및 응답 속도 향상 | 홈 페이지 |
| **F008** | 자동화 파이프라인 | 운영자 개입 없이 데이터 수집/검증/복구를 런타임이 자체 수행한다. (1) `uploads_playlist_id` 공란 시 `channels.list` 자동 조회 후 Notion DB에 write-back, (2) `channels.list` 결과 공백인 무효 channel_id는 `active=false`로 자동 전환하고 `error_reason` 자동 기록, (3) ISR `revalidate: 600`으로 10분마다 자동 재수집, (4) `videos.list` 응답에서 누락된 비공개/삭제 영상 ID 자동 제외, (5) 403 `quotaExceeded` 응답 시 마지막 성공 캐시를 그대로 서빙하고 다음 revalidate까지 자동 대기 | 1인 운영 기준 99.9% 무인 자동화를 위해 필수 | 홈 페이지 |

### 3. MVP 이후 기능 (제외)

- 운영자 채널 관리 UI (Notion에서 직접 관리)
- 사용자 인증 및 회원가입
- 채널 카테고리별 필터링
- 영상 즐겨찾기 / 저장 기능
- 실시간 자동 갱신 (폴링/웹소켓)
- 다크 모드 / 테마 설정
- 검색 기능

---

## 메뉴 구조

```
홈 페이지 내부 탭 내비게이션
├── 최신(1h) 탭 - F003, F004, F005 (최근 1시간 업로드 기준 TOP 10, ISR 10분 갱신)
├── 일간 탭     - F003, F004, F005 (최근 24시간 업로드 기준 TOP 10)
├── 주간 탭     - F003, F004, F005 (최근 7일 업로드 기준 TOP 10)
└── 월간 탭     - F003, F004, F005 (최근 30일 업로드 기준 TOP 10)
```

> 별도 헤더 메뉴 없음. 단일 페이지 서비스로 탭 전환만 제공.

---

## 페이지별 상세 기능

### 홈 페이지

> **구현 기능:** `F001`, `F002`, `F003`, `F004`, `F005`, `F006`, `F007`, `F008` | **인증:** 불필요 (공개 페이지)

| 항목 | 내용 |
|------|------|
| **역할** | 서비스의 유일한 페이지. Notion 큐레이션 채널 기반으로 기간별 YouTube TOP 10 영상을 표시하고 사이트 내에서 즉시 재생하는 랜딩 겸 메인 페이지 |
| **진입 경로** | 서비스 URL 직접 접근 (유일한 페이지이므로 별도 진입 조건 없음) |
| **사용자 행동** | 기간 탭(최신(1h)/일간/주간/월간)을 선택하고, TOP 10 영상 카드 목록을 확인하며, 원하는 영상 카드를 클릭하여 사이트 내 Dialog 모달에서 즉시 재생 |
| **주요 기능** | - 기간 탭 UI 표시 (최신(1h) / 일간 / 주간 / 월간), 기본값 최신(1h)<br>- Notion DB에서 active 채널 목록 조회 (F001)<br>- `channels.list` → `playlistItems.list` → `videos.list` 3단계 체인으로 영상/조회수 수집 (F002)<br>- `contentDetails.videoPublishedAt` 기반 서버 런타임 기간 필터링 (F003)<br>- 누적 조회수 내림차순 정렬 후 TOP 10 카드 렌더링 (F004)<br>- 탭 클릭 시 단일 데이터 메모리 필터링으로 즉시 전환 (F005)<br>- 영상 카드 클릭 시 shadcn/ui Dialog 모달 + YouTube iframe 임베드 재생 (F006)<br>- ISR revalidate: 600 캐싱 (F007)<br>- `uploads_playlist_id` 자동 보정, 무효 channel_id 자동 비활성화, 비공개/삭제 영상 자동 제외, 쿼터 초과 시 캐시 폴백 (F008) |
| **다음 이동** | 영상 카드 클릭 → 사이트 내 Dialog 모달 오픈(동일 페이지 유지), 모달 닫기 → 홈 페이지 TOP 10 목록으로 복귀, 탭 전환 → 동일 페이지 내 TOP 10 갱신 |

---

## 데이터 모델

### Channels (Notion DB)

| 필드 | 설명 | 타입/관계 |
|------|------|----------|
| id | Notion 페이지 고유 식별자 | UUID (Notion 자동 생성) |
| channel_name | YouTube 채널 표시 이름 | text |
| channel_id | YouTube 채널 ID (UC로 시작하는 식별자) | text |
| uploads_playlist_id | 채널 업로드 재생목록 ID (UU...) — 공란 시 런타임이 `channels.list`로 자동 조회 후 write-back | text |
| category | 채널 분류 카테고리 | text |
| active | 수집 대상 여부 플래그 (true인 채널만 조회, 무효 channel_id는 자동 false 전환) | boolean |
| error_reason | 자동 비활성화 사유 텍스트 (예: "channels.list empty response", "quotaExceeded") | text |

### Video (런타임 메모리 모델 - DB 저장 없음)

| 필드 | 설명 | 타입/관계 |
|------|------|----------|
| videoId | YouTube 영상 고유 ID (Dialog iframe src에 사용) | string |
| title | 영상 제목 | string |
| thumbnailUrl | 영상 썸네일 이미지 URL | string |
| channelName | 업로드 채널 이름 | string |
| viewCount | 누적 조회수 | number |
| publishedAt | 영상 업로드 일시 | ISO 8601 string |

---

## 기술 스택 (최신 버전)

### 프론트엔드 프레임워크

- **Next.js 15** (App Router + Turbopack) - React 풀스택 프레임워크, ISR 캐싱 지원
- **TypeScript 5.6+** - 타입 안전성 보장
- **React 19** - UI 라이브러리 (최신 동시성 기능)

### 스타일링 & UI

- **TailwindCSS v4** (설정 파일 없는 새로운 CSS 엔진) - 유틸리티 CSS 프레임워크
- **shadcn/ui** - 고품질 React 컴포넌트 라이브러리 (**Dialog**, Card, Tabs 컴포넌트 활용 — Dialog는 사이트 내 영상 임베드 재생 모달에 사용)
- **Lucide React** - 아이콘 라이브러리

### 외부 API 클라이언트

- **@notionhq/client** - Notion API 공식 클라이언트 (채널 목록 조회 및 write-back)
- **googleapis** - YouTube Data API v3 클라이언트 (channels.list, playlistItems.list, videos.list)

### 캐싱

- **ISR (Incremental Static Regeneration)** - revalidate: 600 (10분) 설정으로 API 쿼터 보호 및 응답 속도 최적화, 쿼터 초과 시 자동 폴백 스토리지로도 활용

### 배포 & 호스팅

- **Vercel** - Next.js 15 최적화 배포 플랫폼, 환경변수 관리

### 패키지 관리

- **npm** - 의존성 관리

---

## 환경변수

| 변수명 | 설명 |
|--------|------|
| `NOTION_API_KEY` | Notion 통합 API 시크릿 키 (DB read + write-back 권한 필요) |
| `NOTION_DATABASE_ID` | Channels DB의 Notion 데이터베이스 ID |
| `YOUTUBE_API_KEY` | YouTube Data API v3 키 |

---

## 구현 순서 (Claude Code 메타 프롬프트)

| 단계 | 작업 내용 |
|------|----------|
| 1단계 | 프로젝트 초기 설정 및 환경변수 구성 (.env.local, 타입 정의) |
| 2단계 | Notion API 연동 및 채널 목록 조회 + write-back 클라이언트 구현 (F001, F008-1) |
| 3단계 | YouTube Data API 3단계 체인 연동 (channels.list → playlistItems.list → videos.list) (F002) |
| 4단계 | 기간 필터링 로직 구현 (videoPublishedAt 서버 런타임 비교, F003) |
| 5단계 | TOP 10 카드 UI 컴포넌트 구현 (F004) |
| 6단계 | 탭 전환 + Dialog 임베드 재생 모달 구현 (F005, F006) |
| 7단계 | 자동화 파이프라인 강화 — 무효 채널 자동 비활성화, 비공개 영상 제외, quotaExceeded 폴백 (F008-2~5) + 반응형 디자인 적용 |
| 8단계 | Vercel 배포 및 환경변수 설정 |

---

## API 쿼터 관리 전략

- Notion DB 채널 수 상한: **최대 20개** (초과 시 active 채널 우선순위 조정)
- YouTube Data API 일일 쿼터 10,000 units 기준:

| API | 비용 | 1회 수집당 호출 수 |
|-----|------|--------------------|
| `channels.list` | 1 unit | 최초 1회만 (F008 자동 write-back으로 이후 생략) |
| `playlistItems.list` | 1 unit | 채널 20개 × 1 = 20회 |
| `videos.list` (50 id 배치) | 1 unit | 최대 20회 (채널당 영상 50개 기준) |
| **1회 수집 소계** | **≈ 60 units** | — |
| **일일 (revalidate 600 × 144회)** | **≈ 8,640 units** | 10,000 한도 내 |

- **`publishedAfter` 파라미터는 사용하지 않음** (`search.list` 전용, 100 units/req — 사용 시 즉시 쿼터 초과)
- 4개 탭 데이터는 **단일 수집 후 서버 런타임 메모리에서 `publishedAt` 기준으로 필터링**
- ISR 10분 캐싱으로 실제 API 호출을 시간당 6회로 제한

---

## 자동화 정책 (99.9% 무인 운영)

운영자는 Notion DB에 채널을 추가/제거하는 것 외에는 어떠한 수동 개입도 하지 않는다. 모든 예외 상황은 런타임이 자체 복구한다.

| 번호 | 자동화 항목 | 트리거 조건 | 자동 동작 |
|------|-------------|-------------|----------|
| 1 | `uploads_playlist_id` 자동 보정 | Notion 레코드의 `uploads_playlist_id`가 비어 있음 | 런타임이 `channels.list(part=contentDetails, id=channel_id)`로 조회하여 `contentDetails.relatedPlaylists.uploads` 값을 Notion DB에 write-back. 다음 수집부터는 이 값을 재사용하여 `channels.list` 호출 생략 |
| 2 | 무효 channel_id 자동 검증 | `channels.list` 응답 `items`가 비어있음 (삭제/차단/오타 채널) | 해당 Notion 레코드 `active=false` 자동 전환 + `error_reason`에 `"channels.list empty response"` 기록. 이후 수집 대상에서 자동 제외 |
| 3 | 데이터 수집 자동화 | ISR revalidate 주기 도달 (10분) | 운영자 개입 없이 전체 파이프라인(채널 조회 → 영상 수집 → 필터링 → 캐시 갱신) 자동 실행 |
| 4 | 비공개/삭제 영상 자동 필터링 | `playlistItems.list`에는 존재하지만 `videos.list` 응답에 해당 videoId가 없음 | 런타임 메모리에서 해당 ID를 자동 제외하고 TOP 10 랭킹 재계산. Notion 수정 불필요 |
| 5 | 쿼터 초과 자동 백오프 | YouTube Data API가 HTTP 403 `quotaExceeded` 반환 | 마지막 ISR 성공 캐시를 그대로 서빙, 새 수집 중단. `error_reason`에 `"quotaExceeded"` 기록. 다음 revalidate 시 재시도하여 자동 복구 |

### 자동화 경계 (사람 개입 영역)

- 신규 채널 추가/제거: 운영자가 Notion DB에서 직접 수행
- `YOUTUBE_API_KEY` / `NOTION_API_KEY` 만료 갱신: 운영자가 Vercel 환경변수에서 직접 교체
- 이 외 모든 수집/검증/복구/필터링 흐름은 100% 런타임 자동 수행
