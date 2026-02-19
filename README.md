# TIL 피드 웹사이트

Notion 데이터베이스를 CMS로 활용한 TIL(Today I Learned) 피드 웹사이트입니다.
Notion에 학습 내용을 작성하면 웹사이트에 자동으로 반영됩니다.

## 기술 스택

- **Framework**: Next.js 15.5.3 (App Router)
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS v4 + shadcn/ui
- **CMS**: Notion API v1

## 시작하기

### 1. 환경변수 설정

`.env.local` 파일을 생성하고 다음 값을 입력합니다.

```bash
NOTION_API_KEY=secret_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NOTION_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Notion 설정 방법:
1. [Notion Developers](https://www.notion.so/my-integrations)에서 Integration 생성
2. TIL 데이터베이스 페이지 → "Connections"에서 Integration 연결
3. 데이터베이스 URL에서 ID 추출: `notion.so/[workspace]/[DATABASE_ID]?v=...`

### 2. 패키지 설치 및 실행

```bash
npm install
npm run dev
```

[http://localhost:3000](http://localhost:3000)에서 확인합니다.

## Notion 데이터베이스 스키마

| 속성명 | 타입 | 설명 |
|--------|------|------|
| `Title` | title | TIL 제목 |
| `Category` | select | `Backend` / `Frontend` / `DevOps` / `CS` |
| `Tags` | multi_select | 세부 태그 |
| `Date` | date | 학습 날짜 |
| `Status` | select | `Published` / `Draft` |

## 주요 명령어

```bash
npm run dev       # 개발 서버 실행
npm run build     # 프로덕션 빌드
npm run check-all # 린트 + 타입 검사
```

## 문서

- [PRD (제품 요구사항)](./docs/PRD.md)
