# Development Guidelines

## Project Overview

- **목적**: Notion 데이터베이스를 CMS로 활용한 TIL(Today I Learned) 피드 웹사이트
- **기술 스택**: Next.js 15.5.3 (App Router + Turbopack), React 19, TypeScript 5, TailwindCSS v4, shadcn/ui (new-york), Notion API

## Current Project State

### 완료된 Phase

- **Phase 1 완료**: 환경변수, 이미지 도메인, SDK 설치, Notion 클라이언트 초기화, 환경변수 검증 스키마

### 현재 존재하는 파일 (실제 확인)

| 파일 | 상태 | 비고 |
|------|------|------|
| `src/lib/env.ts` | 존재 | NOTION_API_KEY, NOTION_DATABASE_ID 검증 포함 |
| `src/lib/notion.ts` | 존재 | Notion Client + NotionToMarkdown 초기화 |
| `src/lib/utils.ts` | 존재 | `cn()` 함수만 있음 (날짜 함수는 Task 008에서 추가) |
| `src/components/layout/header.tsx` | 존재 | **스타터 템플릿** — Phase 3에서 TIL용으로 교체 |
| `src/components/layout/footer.tsx` | 존재 | **스타터 템플릿** — Phase 3에서 TIL용으로 교체 |
| `src/app/page.tsx` | 존재 | **스타터 템플릿** — Phase 3 Task 012에서 TIL 피드로 교체 |

### 스타터 템플릿 잔여 파일 (단계적 교체 대상)

- `src/app/login/`, `src/app/signup/` — TIL 구현 완료 후 삭제
- `src/components/sections/` (hero, features, cta) — page.tsx 교체 시 삭제
- `src/components/navigation/` — header 교체 시 삭제
- `src/components/login-form.tsx`, `src/components/signup-form.tsx` — 삭제 예정

**규칙**: 위 파일들은 현재 건드리지 않음. 해당 Phase 작업 시 교체/삭제 처리.

### 아직 존재하지 않는 파일 (생성 예정)

- `src/types/til.ts` — Phase 2 Task 006
- `src/lib/notion-api.ts` — Phase 2 Task 007
- `src/components/til/` 전체 — Phase 2~3

## Project Architecture

### 디렉토리 구조 (목표)

```
src/
├── app/
│   ├── page.tsx                    # 메인 피드 (ISR revalidate: 60)
│   ├── error.tsx                   # 글로벌 에러 페이지
│   ├── not-found.tsx               # 404 페이지
│   ├── til/[id]/
│   │   ├── page.tsx                # TIL 상세 페이지 (ISR revalidate: 60)
│   │   └── not-found.tsx
│   └── api/revalidate/
│       └── route.ts                # On-Demand Revalidation 엔드포인트
├── components/
│   ├── til/                        # TIL 전용 컴포넌트
│   │   ├── til-card.tsx
│   │   ├── til-list.tsx
│   │   ├── til-skeleton.tsx
│   │   ├── til-content.tsx
│   │   ├── filter-bar.tsx
│   │   └── category-badge.tsx
│   ├── layout/                     # 레이아웃 컴포넌트
│   │   ├── header.tsx              # Phase 3에서 TIL용으로 교체
│   │   └── footer.tsx              # Phase 3에서 TIL용으로 교체
│   └── ui/                         # shadcn/ui 자동 생성 (수정 금지)
├── lib/
│   ├── notion.ts                   # Notion Client 초기화 전용
│   ├── notion-api.ts               # Notion DB 조회 함수 전용
│   ├── utils.ts                    # 공통 유틸리티 (cn, 날짜 포맷, 그룹핑)
│   └── env.ts                      # 환경변수 Zod 검증 스키마
└── types/
    └── til.ts                      # TIL 관련 타입 전용
```

### 주요 참조 문서

- `docs/PRD.md` — 기능 명세
- `docs/ROADMAP.md` — 개발 로드맵 (작업 완료 시 반드시 체크 업데이트)
- `docs/guides/` — 스타일링, 컴포넌트 패턴, Next.js 15, 폼 처리 가이드

## Code Standards

### 파일 명명 규칙

- 모든 컴포넌트 파일: **kebab-case** (`til-card.tsx`, `filter-bar.tsx`, `category-badge.tsx`)
- 타입 파일: kebab-case (`til.ts`)
- 유틸 파일: kebab-case (`notion-api.ts`, `env.ts`)
- **금지**: PascalCase 파일명 (`TilCard.tsx` → 사용 불가)

### TypeScript 규칙

- `any` 타입 사용 **금지** → 정확한 타입 또는 `unknown` 사용
- `TilCategory`, `TilStatus`, `Til`, `TilDetail`, `FilterParams` 타입은 `src/types/til.ts`에서만 정의
- 환경변수 타입은 `src/lib/env.ts`의 `Env` 타입 참조

### 컴포넌트 작성 규칙

- **기본: Server Component** — `async` 함수로 작성
- **`'use client'` 선언 조건**: `useState`, `useEffect`, `useSearchParams`, 이벤트 핸들러 사용 시에만
- Server Component에서 `useState`, `useEffect` 사용 **금지**
- Props 인터페이스는 컴포넌트 파일 상단에 인라인으로 정의

## Functionality Implementation Standards

### Notion API 접근 패턴

- Notion 클라이언트 인스턴스: `src/lib/notion.ts`에서 초기화, 다른 파일에서 import
- 모든 DB 조회 함수: `src/lib/notion-api.ts`에서만 작성
- 페이지 컴포넌트에서 직접 `fetch` 또는 Notion SDK 호출 **금지**
- 반드시 `getTilList()`, `getTilById()`, `getTilContent()`, `getAllTilIds()` 패턴 사용

### ISR 설정 규칙

- 메인 피드(`app/page.tsx`): `export const revalidate = 60`
- 상세 페이지(`app/til/[id]/page.tsx`): `export const revalidate = 60`
- 정적 경로 생성: `generateStaticParams()` 반드시 구현
- On-Demand Revalidation: `REVALIDATE_SECRET` 토큰으로 인증 후 `revalidatePath()` 호출

### 필터링 패턴

- 필터 상태: URL 쿼리 파라미터로 유지 (`?category=Backend&tags=react,typescript`)
- 서버 컴포넌트: `searchParams` prop으로 필터 파라미터 수신
- 클라이언트 필터 UI: `useSearchParams` + `useRouter` 사용 (`'use client'` 필요)
- 서버 측에서 `getTilList(filterParams)` 호출로 필터링 처리

### 날짜 그룹핑

- `src/lib/utils.ts`의 `groupTilsByDate()` 함수 사용 (Task 008 이후)
- 날짜 포맷: `formatDate()`, `formatDateGroup()` 함수 사용 (Task 008 이후)
- 그룹 헤더 표시 예시: "2026-02-19 (3개)"

## Framework/Plugin/Third-party Library Standards

### shadcn/ui 사용 규칙

- `src/components/ui/` 파일 직접 수정 **금지** (shadcn 자동 생성)
- 새 컴포넌트 추가: `npx shadcn@latest add [component-name]`
- style: `new-york`, tailwind v4 기준
- 기존 shadcn 컴포넌트 우선 사용, 없으면 커스텀 컴포넌트 작성

### 마크다운 렌더링

- `react-markdown` + `remark-gfm` 조합 사용
- Notion 블록 → 마크다운 변환: `notion-to-md` (`lib/notion-api.ts`에서 처리)
- 마크다운 렌더링 컴포넌트: `src/components/til/til-content.tsx`

### 이미지 처리

- Notion 이미지: `next/image` 컴포넌트 사용 **필수**
- `next.config.ts`에 Notion 도메인 `images.remotePatterns` 등록 완료
  - 등록된 도메인: `www.notion.so`, `*.notion.so`, `images.unsplash.com`, `s3.us-west-2.amazonaws.com`, `prod-files-secure.s3.us-west-2.amazonaws.com`
- `<img>` 태그 직접 사용 **금지**

### 스타일링 규칙

- TailwindCSS v4 클래스 사용 (CSS 변수 기반)
- 인라인 `style` prop 사용 **금지**
- 클래스 병합: `cn()` 함수 (`src/lib/utils.ts`) 사용
- CSS Module 사용 **금지**, 별도 CSS 파일 최소화

## Environment Variables

### 환경변수 관리 규칙

- 모든 환경변수 접근: `src/lib/env.ts`의 `env` 객체를 통해서만
- `process.env.XXX` 직접 접근 **금지** (`env.ts`, `notion.ts` 제외)
- Notion API Key는 서버 전용 (`NEXT_PUBLIC_` 접두사 사용 **금지**)
- 환경변수 추가 시 `src/lib/env.ts` Zod 스키마에 반드시 추가

### 필수 환경변수

| 변수명 | 필수 | 위치 |
|--------|------|------|
| `NOTION_API_KEY` | 필수 | `.env.local` |
| `NOTION_DATABASE_ID` | 필수 | `.env.local` |
| `REVALIDATE_SECRET` | 선택 | `.env.local` |

## Workflow Standards

### 작업 완료 체크리스트

1. `npm run check-all` 통과 확인 (typecheck + lint + format)
2. `npm run build` 빌드 성공 확인
3. `docs/ROADMAP.md`에서 완료된 태스크 체크 표시 업데이트
4. shrimp-task-manager `verify_task`로 완료 처리

### 파일 수정 시 동시 수정 필요 사항

| 수정 대상 | 동시 수정 파일 |
|-----------|----------------|
| 환경변수 추가 | `src/lib/env.ts` + `.env.example` |
| 새 TIL 타입 추가 | `src/types/til.ts` |
| Notion 도메인 추가 | `next.config.ts`의 `images.remotePatterns` |
| 새 Notion API 함수 추가 | `src/lib/notion-api.ts` |
| ROADMAP 태스크 완료 | `docs/ROADMAP.md` 체크 업데이트 |

### Notion API 에러 처리

- 환경변수 미설정 시 빌드 타임에 명확한 에러 발생하도록 `env.ts`에서 검증
- `getTilById()` 결과 없을 경우 `notFound()` 호출 (`next/navigation`)
- API 호출 실패 시 `error.tsx` 페이지로 폴백

## Key File Interaction Standards

### `src/lib/env.ts`

- Zod 스키마로 환경변수 검증
- 새 환경변수 추가 시 이 파일의 `envSchema`에 추가
- `NOTION_API_KEY`, `NOTION_DATABASE_ID` 필수 필드로 등록됨

### `src/lib/notion.ts`

- Notion Client 단일 인스턴스 초기화
- `NotionToMarkdown` 인스턴스 초기화
- 다른 파일에서 import하여 사용, 이 파일에 비즈니스 로직 추가 **금지**

### `src/lib/notion-api.ts` (Phase 2 생성 예정)

- 모든 Notion DB 조회 로직 집중
- `notion.ts`의 클라이언트 인스턴스 import하여 사용
- 반드시 `Til`/`TilDetail` 타입으로 변환하여 반환

### `src/types/til.ts` (Phase 2 생성 예정)

- `TilCategory`, `TilStatus`, `Til`, `TilDetail`, `FilterParams` 타입 정의
- 새 TIL 관련 타입은 모두 이 파일에 추가

## AI Decision-making Standards

### 컴포넌트 타입 결정 트리

```
데이터 fetch 필요? → YES → Server Component (async)
     ↓ NO
브라우저 API / 상태 / 이벤트 필요? → YES → Client Component ('use client')
     ↓ NO
Server Component (non-async)
```

### 스타일 적용 결정

- shadcn/ui에 원하는 컴포넌트 있음 → shadcn 컴포넌트 사용
- shadcn/ui에 없음 → TailwindCSS v4 클래스로 커스텀 컴포넌트 작성
- 절대 인라인 style 또는 별도 CSS 파일 사용 안 함

### Notion API 함수 위치 결정

- DB 조회, 파싱, 변환 로직 → `src/lib/notion-api.ts`
- 클라이언트 초기화만 → `src/lib/notion.ts`
- 날짜 포맷, 그룹핑 등 순수 유틸 → `src/lib/utils.ts`

### 스타터 템플릿 파일 처리 결정

- 해당 Phase 작업 대상이 아닌 스타터 파일 → 건드리지 않음
- 교체 대상 파일이 있는 Phase 작업 시 → 기존 파일 덮어쓰거나 삭제 후 새로 작성

## Prohibited Actions

- **`any` 타입 사용** — 타입 안전성 위반
- **Server Component에서 `useState`/`useEffect` 사용** — 런타임 에러
- **`process.env` 직접 접근** (`env.ts`, `notion.ts` 제외) — 환경변수 검증 우회
- **`NEXT_PUBLIC_` 접두사로 Notion API Key 노출** — 보안 취약점
- **`src/components/ui/` 파일 직접 수정** — shadcn 업데이트 시 덮어씌워짐
- **페이지 컴포넌트에서 직접 Notion SDK 호출** — 관심사 분리 위반
- **`<img>` 태그 사용** — Next.js 이미지 최적화 미적용
- **인라인 `style` prop 사용** — TailwindCSS 규칙 위반
- **`npm run check-all` 미통과 상태로 작업 완료 처리** — 코드 품질 기준 미달
- **PascalCase 파일명 사용** — 프로젝트 명명 규칙 위반
- **스타터 템플릿 잔여 파일을 해당 Phase 이전에 삭제** — 의존성 오류 가능
