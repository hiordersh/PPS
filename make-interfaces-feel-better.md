# SKILL: make-interfaces-feel-better — 인터페이스 폴리싱

> 하네스 ID: **H-03** | 적용 범위: 모든 UI 컴포넌트 작성/리뷰 시 | 성격: **강제 규정**
> 출처: [jakubkrehel/make-interfaces-feel-better](https://github.com/jakubkrehel/make-interfaces-feel-better)
> (원문 "Details that make interfaces feel better" 기반 오픈소스 스킬을 PPS 맥락에 맞게 재구성)

## 목적

작은 디테일들이 쌓여 완성도 높은 인터페이스를 만든다.
특히 **레이아웃 울렁거림(layout shift) 방지**를 최우선 규정으로 한다.

---

## 규칙 1. 레이아웃 울렁거림 방지 (최우선)

hover, 데이터 변동, 상태 변화 시 요소의 크기·위치가 변해 주변이 밀리는 현상을 금지한다.

- **hover 시 외곽선 변동 금지**: `border` 추가/두께 변경은 박스 크기를 바꾼다.
  대신 크기에 영향 없는 속성만 사용:

```css
/* ❌ 금지: 크기가 변해 주변이 밀림 */
.card:hover { border: 2px solid var(--color-primary); }

/* ✅ 허용: 크기 불변 */
.card { border: 1px solid transparent; }               /* 자리 미리 예약 */
.card:hover { border-color: var(--color-primary); }
/* 또는 */
.card:hover { box-shadow: 0 0 0 2px var(--color-primary); }
.card:hover { outline: 2px solid var(--color-primary); outline-offset: -2px; }
```

- **변동하는 숫자는 고정폭 숫자 사용** (대시보드 카운트, 통계 수치):

```css
.stat-number { font-variant-numeric: tabular-nums; }
```

- **비동기 콘텐츠 자리 예약**: 이미지에 `width/height` 또는 `aspect-ratio` 지정,
  데이터 로딩 중에는 실제 콘텐츠와 같은 크기의 스켈레톤 표시.
- **hover 시 텍스트 굵기 변경 금지** (`font-weight` 변화는 폭을 바꿈). 색상/배경으로만 강조.

## 규칙 2. 트랜지션·애니메이션

- 인터랙션에는 keyframe이 아닌 **CSS transition** 사용 — 사용자가 중간에 의도를 바꿔도
  자연스럽게 복귀(중단 가능성)해야 한다.
- 트랜지션 대상 속성을 **명시적으로 지정**한다 (`transition: all` 금지):

```css
transition: background-color var(--transition-fast), box-shadow var(--transition-fast);
```

- 애니메이션은 GPU 합성 가능한 속성(`transform`, `opacity`, `filter`)만 사용.
  `width/height/top/left` 애니메이션 금지 (리플로우 유발).
- 버튼 클릭 피드백: `transform: scale(0.96)` — 이보다 작은 값은 과장되어 보이므로 금지.

## 규칙 3. 타이포그래피·터치 영역

- 제목: `text-wrap: balance` / 본문 문단: `text-wrap: pretty` (외톨이 단어 방지)
- 클릭 가능한 요소의 히트 영역은 **최소 40×40px** 확보.
  보이는 요소가 작으면 pseudo-element로 확장:

```css
.icon-button { position: relative; }
.icon-button::after {
  content: ""; position: absolute; inset: -8px; /* 히트 영역 확장 */
}
```

- 히트 영역끼리 겹치지 않게 한다.

## 규칙 4. 중첩 라운드 (concentric radius)

카드 안의 이미지/버튼 등 중첩 요소의 라운드가 어긋나면 어색해 보인다:

```
바깥 radius = 안쪽 radius + padding
```

```css
.card { border-radius: calc(var(--radius-md) + var(--space-unit)); padding: var(--space-unit); }
.card img { border-radius: var(--radius-md); }
```

---

## PPS 적용 포인트

| 위치 | 적용 규칙 |
|---|---|
| 대시보드 요약 카드 숫자 | tabular-nums (규칙 1) |
| 프로젝트 카드 hover | transparent border 예약 또는 box-shadow (규칙 1) |
| 카드 썸네일 이미지 | aspect-ratio 고정 + 스켈레톤 (규칙 1) |
| 목록 필터 변경 시 | 그리드 높이 유지, 로딩 중 스켈레톤 (규칙 1) |
| 모든 버튼 | scale(0.96) 클릭 피드백, 명시적 transition (규칙 2) |
| 태그 칩, 아이콘 버튼 | 40px 히트 영역 (규칙 3) |
| 카드 내 이미지 라운드 | concentric radius 공식 (규칙 4) |

## 자가 점검 체크리스트

- [ ] hover 시 주변 요소가 1px이라도 밀리는 곳이 없는가?
- [ ] 데이터 갱신 시 숫자/목록 영역이 출렁이지 않는가?
- [ ] `transition: all` 이 코드에 0건인가?
- [ ] 이미지에 크기(aspect-ratio) 지정이 모두 되어 있는가?
- [ ] 작은 아이콘 버튼의 히트 영역이 40px 이상인가?
