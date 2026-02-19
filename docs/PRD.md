# PRD: TIL 피드 웹사이트 (Notion CMS 기반)

> **버전**: 1.0.0
> **작성일**: 2026-02-19
> **상태**: 작성 중

---

## 1. 프로젝트 개요

### 목적
Notion 데이터베이스를 CMS로 활용하여 TIL(Today I Learned) 기록을 공개 피드 형태로 제공하는 웹사이트를 구축한다. 개발자가 매일 학습한 내용을 Notion에 작성하면 웹사이트에 자동으로 반영된다.

### 목표
- Notion API를 통해 TIL 데이터를 실시간으로 조회
- 날짜별 그룹핑 및 카테고리/태그 필터링으로 탐색 편의성 제공
- ISR(Incremental Static Regeneration)을 활용한 빠른 페이지 로딩
- 반응형 디자인으로 모바일/데스크탑 모두 지원

---

## 2. 기술 스택

| 분류 | 기술 | 버전 |
|------|------|------|
| Framework | Next.js (App Router) | 15.5.3 |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | v4 |
| UI Components | shadcn/ui (new-york style) | latest |
| CMS | Notion API | v1 |
| Notion SDK | @notionhq/client | latest |
| Notion 렌더러 | notion-to-md | latest |

---

## 3. Notion 데이터베이스 스키마

Notion 데이터베이스에 다음 속성을 정의한다.

| 속성명 | 타입 | 설명 | 옵션 |
|--------|------|------|------|
| `Title` | `title` | TIL 제목 | - |
| `Content` | `rich_text` | 본문 요약 (선택적) | - |
| `Category` | `select` | 분류 카테고리 | `Backend`, `Frontend`, `DevOps`, `CS` |
| `Tags` | `multi_select` | 세부 태그 | 자유 입력 |
| `Date` | `date` | 학습 날짜 | - |
| `Status` | `select` | 공개 여부 | `Published`, `Draft` |

### 스키마 예시 (JSON)
```json
{
  "Title": { "type": "title" },
  "Content": { "type": "rich_text" },
  "Category": {
    "type": "select",
    "options": ["Backend", "Frontend", "DevOps", "CS"]
  },
  "Tags": { "type": "multi_select" },
  "Date": { "type": "date" },
  "Status": {
    "type": "select",
    "options": ["Published", "Draft"]
  }
}
```

---

## 4. 주요 기능 명세

### 4.1 TIL 목록 조회
- `Status === "Published"` 인 항목만 노출
- `Date` 기준 내림차순 정렬
- 날짜별로 항목 그룹핑 (예: "2026-02-19 (3개)")
- ISR 적용: `revalidate: 60` (60초마다 재생성)

### 4.2 카테고리 / 태그 필터링
- 상단 필터 바에서 카테고리 선택 (단일 선택)
- 태그 클릭으로 필터링 (다중 선택 가능)
- 필터 초기화 버튼 제공
- URL 쿼리 파라미터로 필터 상태 유지 (`?category=Backend&tags=react,typescript`)

### 4.3 TIL 상세 페이지
- 카드 클릭 시 상세 페이지(`/til/[id]`)로 이동
- Notion 페이지 본문을 마크다운으로 렌더링
- 메타 정보 표시: 날짜, 카테고리, 태그
- 이전/다음 TIL 네비게이션

### 4.4 데이터 갱신
- ISR 방식으로 60초마다 자동 갱신
- On-Demand Revalidation 지원 (웹훅 엔드포인트)

---

## 5. 화면 구성 (Pages)

### 5.1 메인 피드 페이지 (`/`)

```
┌─────────────────────────────────────────┐
│  [로고/제목]           [깃허브 링크]       │
├─────────────────────────────────────────┤
│  카테고리: [전체] [Backend] [Frontend]... │
│  태그: [react] [typescript] [docker]...  │
├─────────────────────────────────────────┤
│  📅 2026-02-19 (2개)                    │
│  ┌──────────────────────────────────┐   │
│  │ 제목                              │   │
│  │ Backend · react, typescript      │   │
│  │ 간단한 요약 텍스트...              │   │
│  └──────────────────────────────────┘   │
│  ┌──────────────────────────────────┐   │
│  │ ...                               │   │
│  └──────────────────────────────────┘   │
│                                         │
│  📅 2026-02-18 (1개)                    │
│  ...                                    │
└─────────────────────────────────────────┘
```

### 5.2 상세 페이지 (`/til/[id]`)

```
┌─────────────────────────────────────────┐
│  [← 목록으로]                            │
├─────────────────────────────────────────┤
│  제목                                    │
│  2026-02-19 · Backend · react           │
├─────────────────────────────────────────┤
│                                         │
│  [Notion 본문 렌더링 영역]               │
│                                         │
├─────────────────────────────────────────┤
│  [← 이전 TIL]           [다음 TIL →]    │
└─────────────────────────────────────────┘
```

---

## 6. 파일 구조 및 아키텍처

```
src/
├── app/
│   ├── page.tsx                    # 메인 피드 페이지 (ISR)
│   ├── til/
│   │   └── [id]/
│   │       └── page.tsx            # TIL 상세 페이지 (ISR)
│   └── api/
│       └── revalidate/
│           └── route.ts            # On-Demand Revalidation 엔드포인트
├── components/
│   ├── til/
│   │   ├── TilCard.tsx             # TIL 카드 컴포넌트
│   │   ├── TilList.tsx             # TIL 목록 (날짜별 그룹핑)
│   │   ├── TilFilter.tsx           # 카테고리/태그 필터
│   │   └── TilContent.tsx          # 상세 페이지 본문 렌더러
│   └── layout/
│       ├── Header.tsx              # 공통 헤더
│       └── Footer.tsx              # 공통 푸터
├── lib/
│   └── notion.ts                   # Notion API 클라이언트 & 유틸
└── types/
    └── til.ts                      # TIL 타입 정의
```

### `lib/notion.ts` 역할
- Notion Client 초기화
- `getTilList()`: Published TIL 목록 조회 (필터 + 정렬)
- `getTilById(id)`: 단일 TIL 조회
- `getTilContent(id)`: 페이지 본문 마크다운 변환
- `getAllTilIds()`: 정적 경로 생성용 ID 목록

### `types/til.ts` 역할
```typescript
export type TilCategory = 'Backend' | 'Frontend' | 'DevOps' | 'CS';
export type TilStatus = 'Published' | 'Draft';

export interface Til {
  id: string;
  title: string;
  content: string;       // 요약 텍스트
  category: TilCategory;
  tags: string[];
  date: string;          // ISO 날짜 문자열
  status: TilStatus;
}

export interface TilDetail extends Til {
  body: string;          // 마크다운 변환된 본문
}
```

---

## 7. 환경변수 명세

`.env.local` 파일에 다음 변수를 설정한다.

```bash
# Notion API 설정
NOTION_API_KEY=secret_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NOTION_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# (선택) On-Demand Revalidation 보안 토큰
REVALIDATE_SECRET=your_secret_token
```

| 변수명 | 필수 | 설명 |
|--------|------|------|
| `NOTION_API_KEY` | ✅ | Notion Integration 시크릿 키 |
| `NOTION_DATABASE_ID` | ✅ | TIL 데이터베이스 ID |
| `REVALIDATE_SECRET` | 선택 | Revalidation 엔드포인트 보호용 토큰 |

### Notion 설정 방법
1. [Notion Developers](https://www.notion.so/my-integrations)에서 Integration 생성
2. 생성된 `Internal Integration Secret`을 `NOTION_API_KEY`에 설정
3. TIL 데이터베이스 페이지 → "Connections"에서 Integration 연결
4. 데이터베이스 URL에서 ID 추출: `notion.so/[workspace]/[DATABASE_ID]?v=...`

---

## 8. 구현 순서 (단계별 계획)

### Phase 1: 환경 설정
- [ ] Notion Integration 생성 및 데이터베이스 연결
- [ ] 환경변수 설정 (`.env.local`)
- [ ] `@notionhq/client`, `notion-to-md` 패키지 설치

### Phase 2: 데이터 레이어
- [ ] `types/til.ts` 타입 정의
- [ ] `lib/notion.ts` Notion API 클라이언트 구현
  - `getTilList()` 구현
  - `getTilById()` 구현
  - `getTilContent()` 구현

### Phase 3: 메인 피드 페이지
- [ ] `TilCard` 컴포넌트 구현
- [ ] `TilList` 컴포넌트 (날짜별 그룹핑) 구현
- [ ] `TilFilter` 컴포넌트 (카테고리/태그 필터) 구현
- [ ] `app/page.tsx` ISR 적용 (`revalidate: 60`)

### Phase 4: 상세 페이지
- [ ] `TilContent` 컴포넌트 (마크다운 렌더링) 구현
- [ ] `app/til/[id]/page.tsx` ISR 적용
- [ ] `generateStaticParams()` 구현

### Phase 5: 부가 기능
- [ ] On-Demand Revalidation API 라우트 구현
- [ ] Header / Footer 레이아웃 구성
- [ ] SEO 메타데이터 설정 (`generateMetadata`)

### Phase 6: 검증 및 배포
- [ ] 반응형 디자인 확인 (모바일/태블릿/데스크탑)
- [ ] Lighthouse 성능 점수 확인
- [ ] 빌드 성공 확인 (`npm run build`)
- [ ] 배포 (Vercel 권장)

---

## 9. 비기능 요구사항

### 보안
- 환경변수를 통한 API 키 관리 (클라이언트 노출 금지)
- On-Demand Revalidation 엔드포인트 토큰 인증

### 유지보수성
- TypeScript strict 모드 적용
- ESLint + Prettier 코드 품질 유지
- 컴포넌트 단위 분리로 재사용성 확보
