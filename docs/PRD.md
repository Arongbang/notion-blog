# 📋 PRD: 개인 개발 블로그 (Notion CMS)

> **문서 버전**: v1.0
> **작성일**: 2026-11-09
> **상태**: 초안 (MVP 범위 정의)

---

## 1. 프로젝트 개요

### 1.1 프로젝트명

**개인 개발 블로그** — Notion을 CMS로 활용한 개인 기술 블로그

### 1.2 목적

- 개발 과정에서 얻은 지식과 경험을 기록하고 공유하는 개인 기술 블로그를 운영한다.
- 글 작성·수정·발행을 익숙한 Notion 환경에서 처리하여 블로그 운영 부담을 최소화한다.

### 1.3 CMS로 Notion을 선택한 이유

| 항목             | 설명                                                                 |
| ---------------- | -------------------------------------------------------------------- |
| 작성 경험        | 별도 관리자 페이지 없이 Notion 에디터로 글 작성 (블록, 코드, 이미지) |
| 자동 반영        | Notion에서 `Status`를 `발행됨`으로 바꾸면 블로그에 자동 반영         |
| 구조화된 데이터  | Notion 데이터베이스 속성으로 카테고리·태그·발행일 관리               |
| 비용             | 별도 DB·CMS 서버 불필요 (Notion API 무료 사용)                       |
| 모바일 작성 지원 | Notion 모바일 앱으로 언제 어디서나 초안 작성                         |

### 1.4 대상 사용자

- **작성자(본인)**: Notion에서 글을 작성·관리하는 블로그 운영자
- **독자**: 기술 정보를 검색하거나 글을 읽으러 방문하는 개발자

---

## 2. 목표 및 성공 지표

### 2.1 목표

1. Notion 데이터베이스의 발행된 글이 블로그에 자동으로 노출된다.
2. 독자가 카테고리·검색으로 원하는 글을 빠르게 찾을 수 있다.
3. 모든 디바이스(모바일/태블릿/데스크톱)에서 읽기 편한 화면을 제공한다.

### 2.2 성공 지표 (KPI)

| 지표                              | 목표값                         |
| --------------------------------- | ------------------------------ |
| Notion 발행 → 블로그 반영 시간    | 최대 1시간 이내 (ISR 재검증)   |
| Lighthouse Performance 점수       | 90점 이상                      |
| Lighthouse Accessibility 점수     | 90점 이상                      |
| LCP (Largest Contentful Paint)    | 2.5초 이하                     |
| 반응형 지원 브레이크포인트        | 모바일 / 태블릿 / 데스크톱 3종 |

---

## 3. 기술 스택

| 구분       | 기술                                        | 비고                               |
| ---------- | ------------------------------------------- | ---------------------------------- |
| Framework  | Next.js 15 (App Router + Turbopack)         | Server Components 중심             |
| Language   | TypeScript 5                                | strict 모드                        |
| CMS        | Notion API (`@notionhq/client`)             | 서버 사이드에서만 호출             |
| Rendering  | Notion 블록 → React 렌더러                  | 직접 구현 또는 라이브러리 검토     |
| Styling    | Tailwind CSS v4 + shadcn/ui                 | new-york 스타일                    |
| Icons      | Lucide React                                |                                    |
| Theme      | next-themes                                 | 다크 모드 (스타터 기본 포함)       |
| Deployment | Vercel                                      | 환경 변수로 Notion 키 관리         |

---

## 4. Notion 데이터베이스 구조

### 4.1 속성(Property) 정의

| 속성명      | 타입           | 필수 | 설명                                 | 예시                     |
| ----------- | -------------- | ---- | ------------------------------------ | ------------------------ |
| `Title`     | `title`        | ✅   | 글 제목                              | Next.js 15 캐싱 정리     |
| `Category`  | `select`       | ✅   | 글의 대분류 (단일 선택)              | Frontend, Backend, DevOps |
| `Tags`      | `multi_select` | ⬜   | 세부 키워드 (다중 선택)              | React, Next.js, Cache    |
| `Published` | `date`         | ✅   | 발행일 (정렬 기준)                   | 2026-11-09               |
| `Status`    | `select`       | ✅   | 발행 상태 — `초안` / `발행됨`        | 발행됨                   |
| `Content`   | page content   | ✅   | 페이지 본문 (Notion 블록)            | 문단, 코드, 이미지 등    |

### 4.2 데이터 규칙

- 블로그에는 **`Status = 발행됨`** 인 글만 노출한다. `초안`은 절대 노출하지 않는다.
- 글 목록은 **`Published` 내림차순**(최신순)으로 정렬한다.
- 글 URL 식별자는 MVP에서는 **Notion 페이지 ID**를 사용한다. (Slug 속성은 향후 확장)

### 4.3 TypeScript 타입 (초안)

```typescript
// src/types/post.ts
export type PostStatus = '초안' | '발행됨';

export interface Post {
  id: string; // Notion 페이지 ID
  title: string;
  category: string;
  tags: string[];
  published: string; // ISO 날짜 문자열
  status: PostStatus;
}

export interface PostDetail extends Post {
  blocks: NotionBlock[]; // 본문 블록 목록
}
```

---

## 5. 주요 기능 요구사항

### F1. Notion 데이터베이스에서 글 목록 가져오기 (MVP)

- `@notionhq/client`의 `databases.query`로 발행된 글을 조회한다.
- 필터: `Status = 발행됨`, 정렬: `Published desc`
- 100개 초과 시 `start_cursor`로 페이지네이션 처리한다.
- Notion 응답을 내부 `Post` 타입으로 변환하는 매퍼 함수를 둔다.

### F2. 개별 글 상세 페이지 (MVP)

- `pages.retrieve`로 메타데이터, `blocks.children.list`로 본문 블록을 조회한다.
- 중첩 블록(토글, 리스트 하위 항목 등)은 재귀적으로 조회한다.
- **지원 블록 (MVP)**: 문단, 제목(H1~H3), 글머리/번호 목록, 코드, 인용, 콜아웃, 구분선, 이미지, 링크
- 존재하지 않거나 `초안` 상태인 글 접근 시 `notFound()`로 404 처리한다.

### F3. 카테고리별 필터링

- Notion 데이터베이스 스키마에서 `Category`의 선택지 목록을 가져온다.
- 카테고리 페이지에서 해당 카테고리 글만 표시한다.
- 홈 화면에 카테고리 필터(뱃지/탭) UI를 제공한다.

### F4. 검색 기능

- 제목·태그 기준 키워드 검색을 제공한다.
- 1차 구현: 서버에서 받아온 글 목록을 클라이언트에서 필터링 (글 수가 적은 개인 블로그 기준)
- 검색어는 URL 쿼리(`?q=`)로 관리하여 공유·새로고침 시 유지한다.

### F5. 반응형 디자인 (MVP)

- 모바일 우선(Mobile First) 설계
- 브레이크포인트: 모바일(< 768px) 1열 / 태블릿(≥ 768px) 2열 / 데스크톱(≥ 1024px) 3열 카드 그리드
- 상세 페이지 본문은 가독성을 위해 최대 너비(`max-w-3xl` 수준)를 제한한다.

---

## 6. 화면 구성

### 6.1 라우트 구조

| 화면     | 경로                         | 설명                           | 렌더링 전략 |
| -------- | ---------------------------- | ------------------------------ | ----------- |
| 홈       | `/`                          | 최근 글 목록 + 카테고리 필터   | ISR         |
| 글 상세  | `/posts/[id]`                | 개별 글 내용                   | SSG + ISR   |
| 카테고리 | `/category/[category]`       | 카테고리별 글 목록             | SSG + ISR   |
| 검색     | `/?q=키워드` (홈 내 기능)    | 키워드로 필터링된 글 목록      | 클라이언트  |

### 6.2 화면별 구성 요소

**홈 (`/`)**

- 헤더: 블로그 로고/이름, 네비게이션, 다크 모드 토글
- 검색 입력창
- 카테고리 필터 뱃지 목록
- 글 카드 그리드 (제목, 카테고리, 태그, 발행일)
- 푸터

**글 상세 (`/posts/[id]`)**

- 글 헤더: 제목, 카테고리, 태그, 발행일
- 본문: Notion 블록 렌더링 영역
- 하단: 목록으로 돌아가기 링크

**카테고리 (`/category/[category]`)**

- 카테고리명 및 글 개수
- 해당 카테고리 글 카드 그리드

---

## 7. 비기능 요구사항

### 7.1 성능

- Server Components에서 Notion API를 호출하고, 결과를 캐싱한다.
- `revalidate`(예: 3600초) 기반 ISR로 Notion API 호출 횟수를 줄인다.
- `generateStaticParams`로 글 상세·카테고리 페이지를 빌드 시 사전 생성한다.
- 이미지는 `next/image`를 사용한다.

### 7.2 Notion API 제약 대응

- **요청 제한**: 평균 초당 3회 → 캐싱/ISR로 호출 최소화
- **이미지 URL 만료**: Notion 업로드 파일 URL은 약 1시간 후 만료 → 재검증 주기를 1시간 이하로 설정 (향후 이미지 프록시 검토)
- **블록 페이지네이션**: 블록 100개 초과 시 커서 기반 반복 조회

### 7.3 보안

- `NOTION_API_KEY`는 서버 전용 환경 변수로 관리하며 클라이언트에 노출하지 않는다. (`NEXT_PUBLIC_` 접두사 금지)
- `.env.local`은 Git에 커밋하지 않는다.

### 7.4 SEO

- `generateMetadata`로 글별 title/description/Open Graph 메타데이터 생성
- `sitemap.ts`, `robots.ts` 제공 (MVP 이후 우선 작업)

### 7.5 접근성

- 시맨틱 HTML 태그(`header`, `main`, `article`, `nav`) 사용
- 키보드 탐색 및 충분한 색상 대비 확보

---

## 8. 환경 변수

```bash
# .env.local
NOTION_API_KEY=secret_xxxxxxxxxxxxxxxx      # Notion Integration 시크릿 키
NOTION_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxx # 블로그 글 데이터베이스 ID
```

Vercel 배포 시 동일한 키를 **Project Settings → Environment Variables**에 등록한다.

---

## 9. 예상 디렉토리 구조

```
src/
├── app/
│   ├── page.tsx                    # 홈 (최근 글 목록)
│   ├── posts/[id]/page.tsx         # 글 상세
│   └── category/[category]/page.tsx # 카테고리별 목록
├── components/
│   ├── post/                       # PostCard, PostList, PostHeader
│   ├── notion/                     # NotionRenderer, 블록별 컴포넌트
│   └── search/                     # SearchInput, CategoryFilter
├── lib/
│   └── notion.ts                   # Notion 클라이언트 및 조회 함수
└── types/
    └── post.ts                     # Post 관련 타입
```

---

## 10. MVP 범위

### ✅ 포함 (In Scope)

- Notion API 연동 (글 목록·상세 조회)
- 글 목록 페이지 및 상세 페이지
- 기본 스타일링 (Tailwind CSS + shadcn/ui)
- 반응형 디자인

### ⏭️ MVP 이후 (Out of Scope)

- 카테고리별 필터링 페이지 고도화
- 검색 기능 (서버 검색 / 전문 검색)
- 커스텀 Slug URL
- 댓글 (Giscus 등), 조회수
- RSS 피드, sitemap
- 목차(TOC) 자동 생성, 코드 하이라이팅 테마
- Notion Webhook 기반 On-demand Revalidation

> 💡 카테고리 필터링(F3)과 검색(F4)은 주요 기능이지만, MVP 이후 1순위로 구현한다.

---

## 11. 구현 단계

| 단계 | 작업                                   | 주요 산출물                                             |
| ---- | -------------------------------------- | ------------------------------------------------------- |
| 1    | Notion API 패키지 설치 및 환경 설정    | `@notionhq/client` 설치, `src/lib/notion.ts`, 타입 정의 |
| 2    | Notion 데이터베이스 생성 및 API 키 설정 | Integration 생성, DB 연결, `.env.local` 설정            |
| 3    | 글 목록 페이지 구현                    | 홈 화면, `PostCard`, `PostList` 컴포넌트                |
| 4    | 글 상세 페이지 구현                    | `/posts/[id]`, Notion 블록 렌더러, 404 처리             |
| 5    | 스타일링 및 최적화                     | 반응형 레이아웃, ISR 캐싱, 메타데이터, Lighthouse 점검  |

### 단계별 상세

**1단계 — 패키지 설치 및 환경 설정**

```bash
npm install @notionhq/client
```

- Notion 클라이언트 싱글턴 생성 (`src/lib/notion.ts`)
- `Post`, `PostDetail` 타입 정의

**2단계 — Notion 데이터베이스 및 API 키 설정**

1. [Notion Integrations](https://www.notion.so/my-integrations)에서 Internal Integration 생성
2. 4장 구조대로 데이터베이스 생성
3. 데이터베이스 페이지 → `연결(Connections)`에 Integration 추가
4. 데이터베이스 URL에서 ID 추출 후 `.env.local`에 등록

**3단계 — 글 목록 페이지**

- `getPublishedPosts()` 함수 구현 (필터·정렬·페이지네이션)
- 홈 화면에 카드 그리드 렌더링

**4단계 — 글 상세 페이지**

- `getPostById()`, `getPostBlocks()` 함수 구현
- 블록 타입별 렌더러 컴포넌트 구현
- `generateStaticParams`, `generateMetadata` 적용

**5단계 — 스타일링 및 최적화**

- 반응형 그리드 및 본문 타이포그래피 정리
- `revalidate` 설정, `next/image` 적용
- `npm run check-all`, `npm run build` 통과 확인 후 Vercel 배포

---

## 12. 리스크 및 대응

| 리스크                         | 영향                     | 대응 방안                                     |
| ------------------------------ | ------------------------ | --------------------------------------------- |
| Notion API 요청 제한 초과      | 페이지 로딩 실패         | ISR 캐싱, 빌드 시 사전 생성                   |
| Notion 이미지 URL 만료         | 이미지 깨짐              | 재검증 주기 ≤ 1시간, 향후 이미지 프록시/외부 저장 |
| 미지원 블록 타입               | 본문 일부 누락           | 미지원 블록은 무시하고 개발 환경에서 경고 로그 |
| Notion API 스키마/버전 변경    | 데이터 매핑 오류         | 매퍼 함수로 변환 로직 일원화, 타입 가드 적용   |
| API 키 노출                    | 워크스페이스 데이터 유출 | 서버 전용 환경 변수, Integration 권한 최소화  |
