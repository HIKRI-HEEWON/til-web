# TIL 피드 웹사이트 개발 로드맵

Notion 데이터베이스를 CMS로 활용하여 TIL(Today I Learned) 기록을 공개 피드 형태로 제공하는 웹사이트 구축

## 개요

TIL 피드 웹사이트는 개발자를 위한 학습 기록 공유 플랫폼으로 다음 기능을 제공합니다:

- **Notion CMS 연동**: Notion 데이터베이스에 작성한 TIL이 웹사이트에 자동 반영
- **날짜별 그룹핑 피드**: 학습 기록을 날짜별로 그룹화하여 타임라인 형태로 제공
- **카테고리/태그 필터링**: Backend, Frontend, DevOps, CS 카테고리 및 태그 기반 탐색
- **ISR 기반 성능 최적화**: 60초 주기 재생성으로 빠른 페이지 로딩과 최신 데이터 유지

## 개발 워크플로우

1. **작업 계획**

   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - 새로운 작업을 포함하도록 `ROADMAP.md` 업데이트
   - 우선순위 작업은 마지막 완료된 작업 다음에 삽입

2. **작업 생성**

   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - `/tasks` 디렉토리에 새 작업 파일 생성
   - 명명 형식: `XXX-description.md` (예: `001-setup.md`)
   - 고수준 명세서, 관련 파일, 수락 기준, 구현 단계 포함
   - API/비즈니스 로직 작업 시 "## 테스트 체크리스트" 섹션 필수 포함 (Playwright MCP 테스트 시나리오 작성)
   - 예시를 위해 `/tasks` 디렉토리의 마지막 완료된 작업 참조

3. **작업 구현**

   - 작업 파일의 명세서를 따름
   - 기능과 기능성 구현
   - API 연동 및 비즈니스 로직 구현 시 Playwright MCP로 테스트 수행 필수
   - 각 단계 후 작업 파일 내 단계 진행 상황 업데이트
   - 구현 완료 후 Playwright MCP를 사용한 E2E 테스트 실행
   - 테스트 통과 확인 후 다음 단계로 진행
   - 각 단계 완료 후 중단하고 추가 지시를 기다림

4. **로드맵 업데이트**

   - 로드맵에서 완료된 작업을 체크 표시로 변경

## 개발 단계

### Phase 1: 프로젝트 초기 설정 (골격 구축)

> Notion API 연동이 모든 기능의 전제조건이므로 환경 설정을 최우선으로 완료한다.
> 예상 소요: 1~2시간

- **Task 001: Notion API 환경변수 설정** - 우선순위
  - [ ] `.env.local` 파일에 `NOTION_API_KEY`, `NOTION_DATABASE_ID` 환경변수 추가
  - [ ] 선택적 `REVALIDATE_SECRET` 환경변수 추가
  - [ ] `.env.example` 파일 생성하여 필요한 환경변수 목록 문서화

- **Task 002: Next.js 이미지 도메인 설정**
  - [ ] `next.config.ts`에 Notion 이미지 도메인 허용 설정 추가 (`images.remotePatterns`)
  - [ ] `www.notion.so`, `images.unsplash.com`, `s3.us-west-2.amazonaws.com` 등 Notion 관련 도메인 등록

- **Task 003: Notion SDK 패키지 설치**
  - [ ] `@notionhq/client` 패키지 설치
  - [ ] `notion-to-md` 패키지 설치 (Notion 블록을 마크다운으로 변환)
  - [ ] `react-markdown`, `remark-gfm` 패키지 설치 (마크다운 렌더링용)

- **Task 004: Notion 클라이언트 초기화 모듈 생성**
  - [ ] `src/lib/notion.ts` 파일 생성
  - [ ] Notion Client 인스턴스 초기화 (`new Client({ auth: NOTION_API_KEY })`)
  - [ ] `NotionToMarkdown` 인스턴스 초기화
  - [ ] 환경변수 미설정 시 명확한 에러 메시지 출력

- **Task 005: 환경변수 검증 스키마 업데이트**
  - [ ] `src/lib/env.ts`에 `NOTION_API_KEY`, `NOTION_DATABASE_ID` 필수 검증 추가
  - [ ] `REVALIDATE_SECRET` 선택적 검증 추가
  - [ ] 빌드 타임에 환경변수 누락 시 에러 발생 확인

**Phase 1 완료 기준**: Notion API 연결 테스트 성공, 환경변수 누락 시 빌드 에러 발생

---

### Phase 2: 공통 모듈 및 컴포넌트 개발

> 핵심 기능 개발 전 재사용 가능한 타입, 유틸리티, 공통 UI를 준비하여 중복 코드를 방지한다.
> 예상 소요: 2~3시간

- **Task 006: TIL 타입 정의** - 우선순위
  - [ ] `src/types/til.ts` 파일 생성
  - [ ] `TilCategory` 타입 정의 (`'Backend' | 'Frontend' | 'DevOps' | 'CS'`)
  - [ ] `TilStatus` 타입 정의 (`'Published' | 'Draft'`)
  - [ ] `Til` 인터페이스 정의 (id, title, content, category, tags, date, status)
  - [ ] `TilDetail` 인터페이스 정의 (`Til` 확장, body 필드 추가)
  - [ ] `FilterParams` 타입 정의 (category, tags 필터 파라미터)

- **Task 007: Notion API 조회 함수 구현**
  - [ ] `src/lib/notion-api.ts` 파일 생성
  - [ ] `getTilList(filter?: FilterParams)` 함수 구현 (Published 항목만, Date 내림차순)
  - [ ] `getTilById(id: string)` 함수 구현 (단일 TIL 메타데이터 조회)
  - [ ] `getTilContent(id: string)` 함수 구현 (Notion 블록을 마크다운으로 변환)
  - [ ] `getAllTilIds()` 함수 구현 (정적 경로 생성용)
  - [ ] `getCategories()` 함수 구현 (카테고리 목록 조회)
  - [ ] Notion API 응답을 `Til`/`TilDetail` 타입으로 변환하는 파서 구현

- **Task 008: 유틸리티 함수 추가**
  - [ ] `src/lib/utils.ts`에 날짜 포맷 함수 추가 (`formatDate`, `formatDateGroup`)
  - [ ] TIL 목록을 날짜별로 그룹핑하는 함수 추가 (`groupTilsByDate`)
  - [ ] 카테고리별 색상 매핑 유틸리티 함수 추가

- **Task 009: TIL 카드 컴포넌트 구현**
  - [ ] `src/components/til/til-card.tsx` 파일 생성
  - [ ] 카드 레이아웃 구현 (제목, 카테고리, 태그, 요약 텍스트, 날짜)
  - [ ] 카드 클릭 시 상세 페이지(`/til/[id]`)로 이동하는 Link 적용
  - [ ] 반응형 디자인 적용 (모바일/데스크탑)

- **Task 010: 로딩 스켈레톤 컴포넌트 구현**
  - [ ] `src/components/til/til-skeleton.tsx` 파일 생성
  - [ ] TIL 카드와 동일한 크기의 스켈레톤 UI 구현
  - [ ] 목록 스켈레톤 (여러 개 카드 스켈레톤 반복) 구현

- **Task 011: 카테고리 배지 컴포넌트 구현**
  - [ ] `src/components/til/category-badge.tsx` 파일 생성
  - [ ] 카테고리별 색상 구분 배지 구현 (Backend, Frontend, DevOps, CS)
  - [ ] 태그 배지 컴포넌트 구현 (다중 태그 표시)

**Phase 2 완료 기준**: 타입 에러 없음, 공통 컴포넌트 단독 렌더링 확인, Notion API 조회 함수 동작 확인

---

### Phase 3: 핵심 기능 개발

> 공통 모듈이 준비된 후 핵심 페이지를 조립한다. 메인 피드가 상세 페이지보다 먼저 작동해야 사용자 흐름이 완성된다.
> 예상 소요: 4~6시간

- **Task 012: 메인 피드 페이지 구현** - 우선순위
  - [ ] `src/app/page.tsx` 수정 (메인 피드 페이지로 변경)
  - [ ] ISR 적용 (`revalidate: 60`)
  - [ ] `getTilList()` 호출하여 Published TIL 목록 조회
  - [ ] 날짜별 그룹핑 로직 적용 (예: "2026-02-19 (3개)")
  - [ ] 헤더 영역 구현 (로고/제목, GitHub 링크)
  - [ ] Playwright MCP를 활용한 메인 피드 렌더링 테스트

- **Task 013: 카테고리/태그 필터 바 구현**
  - [ ] `src/components/til/filter-bar.tsx` 파일 생성
  - [ ] 카테고리 필터 버튼 구현 (전체, Backend, Frontend, DevOps, CS)
  - [ ] 태그 필터 구현 (다중 선택 가능)
  - [ ] 필터 초기화 버튼 구현
  - [ ] 선택 상태 시각적 피드백 (활성화된 필터 하이라이트)

- **Task 014: TIL 목록 컴포넌트 구현**
  - [ ] `src/components/til/til-list.tsx` 파일 생성
  - [ ] 날짜 그룹 헤더 구현 (날짜 + 항목 수 표시)
  - [ ] TIL 카드 목록 렌더링
  - [ ] 빈 상태 처리 (필터 결과 없을 때 안내 메시지)

- **Task 015: TIL 상세 페이지 구현**
  - [ ] `src/app/til/[id]/page.tsx` 파일 생성
  - [ ] ISR 적용 (`revalidate: 60`)
  - [ ] `generateStaticParams()` 구현 (빌드 시 정적 경로 생성)
  - [ ] TIL 메타 정보 표시 (날짜, 카테고리, 태그)
  - [ ] "목록으로" 돌아가기 버튼 구현
  - [ ] 이전/다음 TIL 네비게이션 구현
  - [ ] Playwright MCP를 활용한 상세 페이지 렌더링 테스트

- **Task 016: 마크다운 렌더링 설정**
  - [ ] `react-markdown` + `remark-gfm` 설정
  - [ ] 코드 블록 구문 강조 스타일 적용
  - [ ] Notion 블록 타입별 마크다운 변환 규칙 확인 (`notion-to-md`)

- **Task 017: 상세 본문 렌더링 컴포넌트 구현**
  - [ ] `src/components/til/til-content.tsx` 파일 생성
  - [ ] 마크다운 본문 렌더링 (react-markdown 활용)
  - [ ] 코드 블록, 이미지, 링크, 테이블 등 스타일 적용
  - [ ] Notion 특유의 블록 타입 (callout, toggle 등) 처리

- **Task 017-1: 핵심 기능 통합 테스트**
  - [ ] Playwright MCP를 사용한 전체 사용자 플로우 테스트
  - [ ] 메인 피드에서 TIL 목록 렌더링 확인
  - [ ] 카드 클릭 시 상세 페이지 이동 확인
  - [ ] 마크다운 렌더링 정상 작동 확인
  - [ ] 이전/다음 네비게이션 동작 확인

**Phase 3 완료 기준**: 메인 피드에서 TIL 목록 렌더링, 카드 클릭 시 상세 페이지 이동, 마크다운 렌더링 정상 작동

---

### Phase 4: 추가 기능 개발

> 핵심 기능이 완성된 후 UX를 향상시키는 부가 기능을 추가한다.
> 예상 소요: 3~4시간

- **Task 018: URL 쿼리 파라미터 기반 필터링** - 우선순위
  - [ ] `useSearchParams`를 활용한 필터 상태 관리
  - [ ] URL 쿼리 파라미터와 필터 바 양방향 동기화 (`?category=Backend&tags=react,typescript`)
  - [ ] 브라우저 뒤로가기/앞으로가기 시 필터 상태 유지
  - [ ] 서버 컴포넌트에서 `searchParams` prop 활용한 필터링
  - [ ] Playwright MCP를 활용한 필터링 동작 E2E 테스트

- **Task 019: On-Demand Revalidation API 엔드포인트**
  - [ ] `src/app/api/revalidate/route.ts` 파일 생성
  - [ ] POST 요청으로 특정 경로 재생성 트리거
  - [ ] `REVALIDATE_SECRET` 토큰 기반 인증 검증
  - [ ] 메인 피드(`/`) 및 상세 페이지(`/til/[id]`) 재생성 지원
  - [ ] Playwright MCP를 활용한 Revalidation API 동작 테스트

- **Task 020: 빈 상태(Empty State) UI**
  - [ ] TIL 목록이 비어있을 때 안내 메시지 컴포넌트 구현
  - [ ] 필터 적용 시 결과가 없을 때 "필터 초기화" 안내 메시지
  - [ ] 아이콘과 텍스트를 조합한 직관적인 빈 상태 UI

- **Task 021: 에러 처리 페이지 구현**
  - [ ] `src/app/error.tsx` 글로벌 에러 페이지 구현
  - [ ] `src/app/not-found.tsx` 404 페이지 구현
  - [ ] `src/app/til/[id]/not-found.tsx` TIL 상세 404 페이지 구현
  - [ ] 에러 발생 시 사용자 친화적 메시지 및 홈으로 돌아가기 버튼

- **Task 022: SEO 메타데이터 최적화**
  - [ ] `src/app/layout.tsx`에 기본 메타데이터 설정
  - [ ] `src/app/page.tsx`에 메인 피드 메타데이터 설정
  - [ ] `src/app/til/[id]/page.tsx`에 `generateMetadata()` 함수 구현 (동적 메타데이터)
  - [ ] Open Graph, Twitter Card 메타데이터 설정
  - [ ] `robots.txt`, `sitemap.xml` 생성 고려

**Phase 4 완료 기준**: 필터링 동작 및 URL 동기화, Revalidation API 동작, 에러 페이지 정상 표시

---

### Phase 5: 최적화 및 배포

> 기능이 모두 완성된 후 성능 최적화와 배포를 진행한다. 최적화를 먼저 하면 기능 변경 시 재작업이 발생한다.
> 예상 소요: 2~3시간

- **Task 023: 이미지 최적화** - 우선순위
  - [ ] Notion 이미지를 `next/image` 컴포넌트로 렌더링
  - [ ] 이미지 크기 최적화 및 lazy loading 적용
  - [ ] 이미지 로드 실패 시 fallback UI 구현

- **Task 024: 코드 품질 검증**
  - [ ] `npm run check-all` 실행 (타입 검사, ESLint, Prettier)
  - [ ] TypeScript strict 모드 에러 해결
  - [ ] 미사용 import 및 변수 정리

- **Task 025: 프로덕션 빌드 확인**
  - [ ] `npm run build` 프로덕션 빌드 성공 확인
  - [ ] 빌드 경고 및 에러 해결
  - [ ] ISR 페이지 정상 생성 확인

- **Task 026: Vercel 배포 설정**
  - [ ] Vercel 프로젝트 연결
  - [ ] 환경변수 등록 (`NOTION_API_KEY`, `NOTION_DATABASE_ID`, `REVALIDATE_SECRET`)
  - [ ] 배포 후 정상 동작 확인

- **Task 027: Lighthouse 성능 점수 확인**
  - [ ] Lighthouse 성능 점수 측정 (목표: 90+)
  - [ ] Performance, Accessibility, Best Practices, SEO 항목별 점수 확인
  - [ ] 성능 개선이 필요한 항목 최적화

**Phase 5 완료 기준**: 빌드 에러 없음, Vercel 배포 성공, Lighthouse 90+ 달성

---

## 기술 참고사항

### 핵심 기술 스택

| 분류 | 기술 | 버전 |
|------|------|------|
| Framework | Next.js (App Router) | 15.5.3 |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | v4 |
| UI Components | shadcn/ui (new-york style) | latest |
| CMS | Notion API | v1 |
| Notion SDK | @notionhq/client | latest |
| Notion 렌더러 | notion-to-md | latest |
| 마크다운 렌더링 | react-markdown + remark-gfm | latest |

### 환경변수

| 변수명 | 필수 | 설명 |
|--------|------|------|
| `NOTION_API_KEY` | 필수 | Notion Integration 시크릿 키 |
| `NOTION_DATABASE_ID` | 필수 | TIL 데이터베이스 ID |
| `REVALIDATE_SECRET` | 선택 | Revalidation 엔드포인트 보호용 토큰 |

### 파일 구조 (목표)

```
src/
├── app/
│   ├── page.tsx                    # 메인 피드 페이지 (ISR)
│   ├── error.tsx                   # 글로벌 에러 페이지
│   ├── not-found.tsx               # 글로벌 404 페이지
│   ├── til/
│   │   └── [id]/
│   │       ├── page.tsx            # TIL 상세 페이지 (ISR)
│   │       └── not-found.tsx       # TIL 상세 404 페이지
│   └── api/
│       └── revalidate/
│           └── route.ts            # On-Demand Revalidation 엔드포인트
├── components/
│   ├── til/
│   │   ├── til-card.tsx            # TIL 카드 컴포넌트
│   │   ├── til-list.tsx            # TIL 목록 (날짜별 그룹핑)
│   │   ├── til-skeleton.tsx        # 로딩 스켈레톤
│   │   ├── til-content.tsx         # 상세 페이지 본문 렌더러
│   │   ├── filter-bar.tsx          # 카테고리/태그 필터
│   │   └── category-badge.tsx      # 카테고리 배지
│   └── layout/
│       ├── header.tsx              # 공통 헤더
│       └── footer.tsx              # 공통 푸터
├── lib/
│   ├── notion.ts                   # Notion Client 초기화
│   ├── notion-api.ts               # Notion DB 조회 함수
│   ├── utils.ts                    # 공통 유틸리티
│   └── env.ts                      # 환경변수 검증
└── types/
    └── til.ts                      # TIL 타입 정의
```
