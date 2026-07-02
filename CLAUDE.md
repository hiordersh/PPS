# CLAUDE.md — PPS 프로젝트 컨텍스트

> 이 파일은 Claude(Claude Code 포함) 및 AI 어시스턴트가 PPS 프로젝트를 이어서 작업할 때
> 반드시 먼저 읽어야 하는 컨텍스트 문서입니다.

## 1. 프로젝트 개요

- **이름**: PPS (Personal Portfolio Server)
- **목적**: 개인 프로젝트/프로그램/교육자료/통계자료를 관리하고, 면접 시 URL 하나로 포트폴리오를 보여주는 웹 서버
- **저장소**: https://github.com/hiordersh/PPS
- **운영 원칙**: **완전 무료** — 유료 플랜이 필요한 제안은 하지 말 것

## 2. 확정된 기술 결정 (변경 시 반드시 사용자 컨펌)

| 항목 | 결정 | 비고 |
|---|---|---|
| 프론트엔드 | React 18 + Vite (JavaScript) | SPA, react-router-dom 사용 |
| 호스팅 | Firebase Hosting (Spark 무료 플랜) | |
| DB | Cloud Firestore | `projects` 단일 컬렉션 |
| 인증 | Firebase Auth (이메일/비밀번호) | 관리자 1인 계정 |
| 이미지 저장 | Cloudinary 무료 플랜 | unsigned upload preset, 브라우저 직접 업로드 |
| 문서/압축파일 | GitHub Releases | ZIP/PDF/HTML, URL을 등록 폼에 수동 입력 |
| 차트 | Chart.js + react-chartjs-2 | 도넛(카테고리) + 막대(연도별) |
| ❌ 금지 | Firebase Storage | Blaze(유료) 플랜 필요하므로 사용 금지 |

## 3. 데이터 모델

`projects` 컬렉션 문서 스키마 (README.md의 스키마와 항상 동기화 유지):

- `title` (string), `category` ("program" | "education" | "statistics")
- `year` (number), `tags` (string[])
- `reason` (string, 제작사유), `result` (string, 결과), `description` (string, 마크다운)
- `images` ({url, caption}[]) — Cloudinary URL
- `files` ({name, url, size, type}[]) — GitHub Releases URL
- `isPublic` (boolean), `createdAt`, `updatedAt` (Timestamp)

카테고리 한글 매핑: program=프로그램, education=교육자료, statistics=통계자료

## 4. 페이지 및 라우팅

| 경로 | 페이지 | 요구사항 |
|---|---|---|
| `/` | 대시보드 | 요약 카드(총 건수/카테고리별) + 도넛 차트 + 연도별 막대 차트 + 최근 등록 5건 |
| `/category/:id` | 카테고리 목록 | 검색, 태그 필터, 연도 필터, 카드 그리드 |
| `/project/:id` | 상세 | 제작사유/결과/설명, 이미지 클릭 시 라이트박스 확대, 파일 다운로드 목록 |
| `/admin` | 관리자 | 로그인 필요. 등록/수정/삭제 폼, Cloudinary 이미지 업로드 |

## 5. 코딩 규칙

- 언어: UI 텍스트는 **한국어**, 코드 식별자·주석은 영어
- 컴포넌트: 함수형 + Hooks, 파일당 하나의 컴포넌트, PascalCase 파일명
- 스타일: CSS Modules 또는 일반 CSS, 모바일 퍼스트 반응형 (브레이크포인트: 768px, 1024px)
- Firestore 접근은 반드시 `src/services/projects.js` 를 통해서만 (컴포넌트에서 직접 쿼리 금지)
- 환경변수: `VITE_` 접두사, `.env`는 gitignore, `.env.example` 유지
- 비공개(`isPublic: false`) 데이터는 클라이언트 필터가 아닌 **Firestore 보안 규칙**으로 차단
- 커밋 메시지: `feat:`, `fix:`, `docs:`, `style:`, `refactor:` 접두사 사용

## 6. 무료 한도 주의사항

- Firestore 일 읽기 50K → 대시보드 집계는 문서 전체 조회 대신 가능하면 캐싱/집계 문서(`stats/summary`) 활용 고려
- Cloudinary 월 25 크레딧 → 업로드 시 자동 리사이즈(w_1600, q_auto) 변환 URL 사용
- GitHub Releases 파일당 2GB 제한 → 초과 시 분할 압축 안내

## 7. UI/UX 정합성 규정 (강제)

모든 UI 작업 전에 아래 스킬을 반드시 읽고 준수한다. 상세 규칙은 각 파일 참조:

- **`skills/ui-consistency.md` (H-02)**: 스크롤바 낭비 금지(세로 1개, 이중 스크롤 금지, scrollbar-gutter),
  브라우저 기본 팝업(alert/confirm/prompt) 전면 금지 → 커스텀 Modal/Toast/ConfirmDialog 대체,
  버튼 위치·레이아웃 고정 배치(주요 액션 우측, 취소는 그 왼쪽), 디자인 토큰 기반 통일성
- **`skills/make-interfaces-feel-better.md` (H-03)**: hover/데이터 변동 시 레이아웃 울렁거림 방지,
  tabular-nums, 중단 가능한 transition, 40px 히트 영역, concentric radius

각 스킬 하단의 **자가 점검 체크리스트**를 작업 완료 전 필수로 수행한다.

## 8. 작업 프로토콜 (사용자 지정 규칙)

- **작업은 직전 작업까지만 참조**한다. 전체 재검토나 처음부터 재작업은 사용자가 명시적으로 지시하고 컨펌한 경우에만 수행한다.
- 새로운 기능/구조 제안이 있으면 먼저 사용자에게 추천하고, **컨펌 후 작업을 진행**한다.
- 모든 기획/제작 단계에서 **github.com/trending 을 최우선 검토**하여 활용 가능한
  오픈소스/스킬이 있으면 사용하고, `skills/스킬명.md` 로 스킬화하여 저장소에 함께 커밋한다.
- 규정화할 내용은 모두 **`HARNESS.md`** 에 모듈 단위로 등록한다 (루프엔지니어링 사이클 준수).
- 산출물 파일은 사용자가 GitHub 저장소에 직접 업로드하여 관리한다.

## 9. 현재 진행 상태

- [x] 요구사항 정의 (PPS.txt)
- [x] 스택/저장소 전략 확정 (React+Vite, Cloudinary+GitHub Releases)
- [x] 문서 3종 작성 (README.md / CLAUDE.md / SKILLS.md)
- [x] 하네스 구축 (HARNESS.md) + UI 정합성 스킬 2종 등록 (H-02, H-03)
- [ ] 다음 단계: Vite 프로젝트 스캐폴딩 + Firebase 연동 코드 작성
