# CLAUDE.md — PPS 프로젝트 컨텍스트

> 이 파일은 Claude(Claude Code 포함) 및 AI 어시스턴트가 PPS 프로젝트를 이어서 작업할 때
> 반드시 먼저 읽어야 하는 컨텍스트 문서입니다.

## 1. 프로젝트 개요

- **이름**: PPS (Personal Portfolio Server) — SH's Portfolio
- **목적**: 개인 프로젝트/프로그램/교육자료/통계자료를 관리하고, 면접 시 URL 하나로 포트폴리오를 보여주는 웹 서버
- **저장소**: https://github.com/hiordersh/PPS
- **로컬 폴더**: PC의 PPS-module3 폴더 (npm run dev로 실행)
- **운영 원칙**: Firebase Blaze 플랜 사용 중 (Storage 때문에 업그레이드, 무료 한도 내 운영)

## 2. 확정된 기술 결정 (변경 시 반드시 사용자 컨펌)

| 항목 | 결정 | 비고 |
|---|---|---|
| 프론트엔드 | React 18 + Vite (JavaScript) | SPA, react-router-dom 사용 |
| 호스팅 | Firebase Hosting (Blaze) | |
| DB | Cloud Firestore | `projects` 단일 컬렉션 |
| 인증 | Firebase Auth (이메일/비밀번호) | 관리자 1인 계정 (nadopia7@gmail.com) |
| 이미지 저장 | Cloudinary 무료 플랜 | unsigned upload preset `pps_unsigned`, cloud name `lus0uqgk` |
| 파일(ZIP/PDF 등) | Firebase Storage (Blaze) | PC에서 직접 업로드, `src/services/storageUpload.js` |
| 차트 | Chart.js + react-chartjs-2 | 도넛(카테고리)만 사용, 연도별 막대 차트 제거됨 |

## 3. 데이터 모델

`projects` 컬렉션 문서 스키마:

- `title` (string), `category` ("program" | "education" | "statistics")
- `year` (number), `seq` (number, 자동 채번)
- `reason` (string, 제작사유), `result` (string, 결과), `description` (string)
- `images` ({url, caption}[]) — Cloudinary URL
- `files` ({name, url, size, type}[]) — Firebase Storage URL
- `isPublic` (boolean), `createdAt`, `updatedAt` (Timestamp)

> ⚠️ `tags` 필드는 기존 데이터에 남아있을 수 있으나 UI에서 완전 제거됨 (등록/수정/표시 모두 없음)

카테고리 한글 매핑: program=프로그램, education=교육자료, statistics=통계자료

## 4. 페이지 및 라우팅

| 경로 | 페이지 | 현재 구현 상태 |
|---|---|---|
| `/` | 대시보드 | Work_Keyword 스타일 숫자 카드 4개 + 도넛 차트 + 최근 등록 5건 |
| `/category/:id` | 카테고리 목록 | 연도 필터만(검색/태그 제거), 2열 카드 그리드 |
| `/project/:id` | 상세 페이지 | 번호/하이픈 텍스트 렌더러, 이미지 라이트박스, 파일 다운로드, 카드 디자인 |
| `/admin` | 관리자 | 로그인 → 등록/수정/삭제, 이미지+파일 PC 직접 업로드 |

## 5. 파일 구조 (주요)

```
src/
├── components/
│   ├── common/     Header(SH's Portfolio 로고), Footer, Layout, PageHeader,
│   │               Modal, Toast, ConfirmDialog, StateViews(Loading/Empty/ErrorBox)
│   ├── dashboard/  SummaryCards, CategoryDonut, RecentList
│   ├── project/    ProjectCard, FilterBar(연도만), ImageGallery, ImageLightbox, FileList
│   └── admin/      LoginForm, ProjectForm(태그 없음), ProjectManageList
├── pages/          Dashboard, CategoryList, ProjectDetail, Admin, NotFound
├── services/       firebase.js, projects.js, cloudinary.js, auth.js, storageUpload.js
├── hooks/          useAuth.jsx, useProjects.js
├── styles/         tokens.css, global.css
└── constants.js
skills/
├── ui-consistency.md (H-02)
├── make-interfaces-feel-better.md (H-03)
└── firestore-deploy-checklist.md (H-05)
```

## 6. Firestore 현황

**보안 규칙**: 게시 완료
- 공개 자료 읽기: 누구나
- 비공개 자료 읽기 + 쓰기: 로그인한 관리자만

**복합 색인 2개 생성 완료**:
- `isPublic ASC + createdAt DESC` → 대시보드용
- `category ASC + isPublic ASC + createdAt DESC` → 카테고리 목록용

## 7. UI/UX 정합성 규정 (강제)

모든 UI 작업 전에 아래 스킬을 반드시 읽고 준수한다:

- **`skills/ui-consistency.md` (H-02)**: 스크롤바 관리, 브라우저 팝업(alert/confirm/prompt) 전면 금지 → Modal/Toast/ConfirmDialog 대체, 버튼 위치 고정 배치
- **`skills/make-interfaces-feel-better.md` (H-03)**: hover 울렁거림 방지, tabular-nums, transition 규칙, 40px 히트 영역

각 스킬 하단의 **자가 점검 체크리스트**를 작업 완료 전 필수로 수행한다.

## 8. 작업 프로토콜 (사용자 지정 규칙)

- **작업은 직전 작업까지만 참조**한다. 전체 재검토나 처음부터 재작업은 사용자가 명시적으로 지시하고 컨펌한 경우에만 수행한다.
- 새로운 기능/구조 제안이 있으면 먼저 사용자에게 추천하고, **컨펌 후 작업을 진행**한다.
- 파일은 **변경/신규 파일만** 전달한다 (전체 zip 아님). 전달 시 정확한 경로 명시.
- 모든 기획/제작 단계에서 **github.com/trending 을 최우선 검토**하여 활용 가능한 오픈소스/스킬이 있으면 사용하고 스킬화한다.
- 규정화할 내용은 모두 **`HARNESS.md`** 에 모듈 단위로 등록한다.
- 산출물 파일: 코드는 PC 로컬 폴더에, 문서(HARNESS 등)는 GitHub에서 직접 수정.

## 9. 현재 진행 상태

- [x] 요구사항 정의 및 문서 3종 작성 (README.md / CLAUDE.md / SKILLS.md)
- [x] 하네스 구축 (HARNESS.md) + UI 정합성 스킬 등록 (H-02, H-03)
- [x] 모듈 1: 프로젝트 뼈대 (Vite, 라우팅, 디자인 토큰, 공통 레이아웃)
- [x] 모듈 2: Firebase/Cloudinary 서비스 레이어
- [x] 모듈 3: 공통 UI 컴포넌트 (Modal, Toast, ConfirmDialog, StateViews)
- [x] 모듈 4: 대시보드 (Work_Keyword 스타일 카드 + 도넛 차트 + 최근 등록)
- [x] 모듈 5: 카테고리 목록 (연도 필터, 2열 그리드)
- [x] 모듈 6: 프로젝트 상세 (라이트박스, 파일 다운로드, 카드 디자인)
- [x] 모듈 7: 관리자 (로그인, 등록/수정/삭제)
- [x] Firebase Storage 업그레이드 (Blaze, storageUpload.js — PC 직접 업로드)
- [x] UI 재설계: SH's Portfolio 로고, 태그 전면 제거, 상세페이지 카드화, 텍스트 렌더러
- [x] Firestore 보안 규칙 게시 + 복합 색인 2개 생성
- [ ] **다음: firebase deploy로 실제 배포**
  ```bash
  npm install -g firebase-tools
  firebase login
  firebase use --add   # pps-portfolio-c5f20 선택
  npm run build
  firebase deploy
  ```
  배포 후 주소: `https://pps-portfolio-c5f20.web.app`
