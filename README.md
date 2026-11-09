# 📝 개인 개발 블로그

Notion을 CMS로 활용한 개인 기술 블로그입니다.
Notion 데이터베이스에서 글을 작성하고 상태를 `발행됨`으로 바꾸면 블로그에 자동으로 반영됩니다.

> 📋 상세 요구사항은 [`docs/PRD.md`](./docs/PRD.md)를 참고하세요.

## ✨ 주요 기능

- **Notion 연동**: Notion 데이터베이스에서 발행된 글 목록 조회
- **글 상세 페이지**: Notion 블록 기반 본문 렌더링
- **카테고리 필터링**: 카테고리별 글 목록
- **검색**: 제목·태그 기반 키워드 검색
- **반응형 디자인**: 모바일 / 태블릿 / 데스크톱 지원
- **다크 모드**: 시스템 설정 연동 테마 전환

## 🛠️ 기술 스택

| 구분       | 기술                                |
| ---------- | ----------------------------------- |
| Framework  | Next.js 15.5.3 (App Router + Turbopack) |
| Runtime    | React 19.1.0 + TypeScript 5         |
| CMS        | Notion API (`@notionhq/client`)     |
| Styling    | Tailwind CSS v4 + shadcn/ui         |
| Icons      | Lucide React                        |
| Deployment | Vercel                              |

## 🚀 시작하기

### 1. 의존성 설치

```bash
npm install
```

### 2. 환경 변수 설정

프로젝트 루트에 `.env.local` 파일을 만들고 아래 값을 입력합니다.

```bash
NOTION_API_KEY=secret_xxxxxxxxxxxxxxxx      # Notion Integration 시크릿 키
NOTION_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxx # 블로그 글 데이터베이스 ID
```

### 3. Notion 데이터베이스 준비

| 속성        | 타입           | 설명                   |
| ----------- | -------------- | ---------------------- |
| `Title`     | `title`        | 글 제목                |
| `Category`  | `select`       | 카테고리               |
| `Tags`      | `multi_select` | 태그                   |
| `Published` | `date`         | 발행일                 |
| `Status`    | `select`       | `초안` / `발행됨`      |

데이터베이스 페이지의 `연결(Connections)` 메뉴에서 생성한 Integration을 추가해야 API로 접근할 수 있습니다.

### 4. 개발 서버 실행

```bash
npm run dev
```

브라우저에서 [http://localhost:3000](http://localhost:3000)을 열어 확인합니다.

## 📜 스크립트

| 명령어              | 설명                                     |
| ------------------- | ---------------------------------------- |
| `npm run dev`       | 개발 서버 실행 (Turbopack)               |
| `npm run build`     | 프로덕션 빌드                            |
| `npm run start`     | 프로덕션 서버 실행                       |
| `npm run check-all` | 타입 검사 + 린트 + 포맷 검사 통합 실행   |
| `npm run lint:fix`  | ESLint 자동 수정                         |
| `npm run format`    | Prettier 포맷팅                          |

## 📁 프로젝트 구조

```
src/
├── app/          # App Router 페이지 및 레이아웃
├── components/   # UI 및 레이아웃 컴포넌트 (shadcn/ui 포함)
└── lib/          # 유틸리티, 환경 변수 검증
docs/
├── PRD.md        # 프로젝트 요구사항 문서
└── guides/       # 개발 가이드 (구조, 스타일링, 컴포넌트, 폼 등)
```

## 📚 개발 가이드

- [프로젝트 요구사항 (PRD)](./docs/PRD.md)
- [프로젝트 구조](./docs/guides/project-structure.md)
- [스타일링 가이드](./docs/guides/styling-guide.md)
- [컴포넌트 패턴](./docs/guides/component-patterns.md)
- [Next.js 15 가이드](./docs/guides/nextjs-15.md)
- [폼 처리 가이드](./docs/guides/forms-react-hook-form.md)

## 🌐 배포

[Vercel](https://vercel.com)에 저장소를 연결한 뒤 **Project Settings → Environment Variables**에 `NOTION_API_KEY`, `NOTION_DATABASE_ID`를 등록하면 배포됩니다.
