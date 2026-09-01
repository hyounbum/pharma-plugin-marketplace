# template-requirements.md

## 1. 기준 파일

반드시 원본 파일을 직접 읽고 템플릿 자산과 구조를 추출한다.

1. `품절 및 입고 양식.html`
   - 플러그인 내장 경로: `assets/품절 및 입고 양식.html`
   - 품절·입고 관련 8컬럼 표 구조
   - 품절/입고 화면 폭과 표 레이아웃 보정
   - base64 로고와 워터마크
2. `버드나무_제약사공지_수수료_의약품_연녹색배경.html`
   - 플러그인 내장 경로: `assets/버드나무_제약사공지_수수료_의약품_연녹색배경.html`
   - 수수료·의약품 정보 카드형 예시
   - 상단 빠른 이동 영역
   - 카드형 상세정보 CSS
   - base64 로고와 워터마크

새 템플릿을 비슷하게 다시 디자인하지 않는다. 두 파일의 실제 DOM, CSS 선언 순서, `!important` 덮어쓰기, 미디어 쿼리와 인쇄 CSS를 읽고 필요한 데이터 영역만 교체한다.

## 2. 유지해야 할 디자인 요소

### 2.1 전체 배경

최종 화면 배경은 기준 파일 후반의 연녹색 미세 조정 규칙을 유지한다.

```css
:root {
  --soft-green-950: #06351f;
  --soft-green-900: #0a4f2d;
  --soft-green-800: #166534;
  --soft-green-700: #2f9e44;
  --soft-green-600: #55b96a;
  --soft-green-500: #74c985;
  --soft-green-300: #b7e4c7;
  --soft-green-200: #d5f4dd;
  --soft-green-100: #eaf8ef;
  --soft-green-50: #f7fff9;
  --soft-bg: #edf9f1;
  --soft-card-line: rgba(22, 101, 52, 0.14);
  --soft-shadow: 0 14px 30px rgba(22, 101, 52, 0.10);
}

html,
body {
  background: var(--soft-bg);
}

body {
  background:
    radial-gradient(circle at 10% 5%, rgba(187, 247, 208, 0.40), transparent 31rem),
    radial-gradient(circle at 92% 8%, rgba(220, 252, 231, 0.58), transparent 34rem),
    linear-gradient(135deg, #fbfffc 0%, #f0fbf3 42%, #e6f6eb 100%);
}
```

기준 파일에는 앞선 녹색 테마 선언과 후반의 연녹색 덮어쓰기가 함께 있다. 선언 순서를 임의 정리하면 최종 화면이 달라질 수 있으므로, 원본 CSS를 통째로 보존하거나 브라우저 계산 스타일 비교 테스트를 통과해야 한다.

### 2.2 워터마크

- `body::before`의 base64 data URI 배경 이미지를 재사용한다.
- `position: fixed`, 전체 화면 inset, pointer-events 비활성 구조를 유지한다.
- 최종 연녹색 규칙의 opacity/filter를 유지한다.
  - `opacity: 0.055`
  - `filter: grayscale(1) brightness(2.95) opacity(0.60)`
- base64 원문: **[원본 자산 필요]**
- 두 기준 파일의 워터마크 data URI가 완전히 동일한지 해시 비교가 필요하다: **[확인 필요]**

### 2.3 최상위 문서 폭

공통 기준:

```css
.page,
.wrap {
  position: relative;
  z-index: 1;
  width: min(1400px, calc(100% - 48px));
  margin: 0 auto;
  padding: 42px 0 62px;
}
```

품절·입고 파일의 추가 보정:

```css
.wrap {
  width: min(1440px, calc(100% - 48px));
  padding-top: 38px;
}
```

혼합 문서에서 어떤 폭을 최종 기준으로 삼을지는 실제 정상 혼합 결과물로 **[확인 필요]**다. 현재 Instructions는 새 화면 CSS 작성을 금지하므로 기준 파일 중 선택한 구조의 값을 그대로 사용한다.

## 3. 제목과 머리말 규칙

### 3.1 주제목

최종 문서의 주제목은 항상:

`제약사 공지`

기준 파일의 `<title>`이나 `<h1>`에 있는 `품절 및 입고`, `버드나무 제약사 공지`를 최종 주제목으로 그대로 쓰지 않는다.

### 3.2 부제

부제는 실제 포함 섹션을 설명하는 기존 형식을 유지한다. 정확한 혼합 문서 부제 문구는 현재 Instructions에 고정되어 있지 않다: **[확인 필요]**

이미지에 없는 정보를 부제에 추가하지 않는다.

### 3.3 헤더 구조

```html
<header class="header">
  <div class="header-copy">
    <h1>제약사 공지</h1>
    <p class="sub">...</p>
  </div>
  <img class="notice-header-logo" alt="버드나무 로고" src="data:image/png;base64,...">
</header>
```

유지 항목:

- flex 정렬
- 헤더 내부 장식 pseudo element
- 둥근 모서리
- 연녹색 그라데이션
- 상단 녹색 테두리
- 그림자
- 로고는 제목 오른쪽 한 곳에만 배치

공통 헤더 기초값:

```css
header,
.header {
  position: relative;
  overflow: hidden;
  min-height: 132px;
  margin-bottom: 24px;
  padding: 30px 36px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 24px;
  border-radius: 28px;
}
```

연녹색 최종 덮어쓰기:

```css
header,
.header {
  background:
    linear-gradient(135deg, rgba(255, 255, 255, 0.92), rgba(234, 248, 239, 0.94));
  border: 1px solid rgba(85, 185, 106, 0.34);
  border-top: 8px solid rgba(85, 185, 106, 0.82);
  box-shadow: 0 14px 30px rgba(22, 101, 52, 0.10);
}
```

품절·입고 추가값:

```css
.header {
  min-height: 142px;
}
```

## 4. 로고 규칙

공통 기준 로고 CSS:

```css
.notice-header-logo {
  flex: 0 0 auto;
  width: clamp(300px, 34vw, 560px);
  height: clamp(76px, 9.5vw, 142px);
  object-fit: contain;
  object-position: center right;
  opacity: 1;
  margin-left: auto;
  filter: drop-shadow(0 8px 16px rgba(22, 101, 52, 0.10));
}
```

품절·입고 파일의 추가 화면 보정:

```css
.notice-header-logo {
  width: clamp(260px, 28vw, 460px);
  height: clamp(78px, 8vw, 126px);
  object-fit: contain;
  object-position: center right;
}
```

인쇄 시 품절·입고 파일에는 `width: 260px; height: auto;`가 있다.

두 기준 파일의 로고 base64 데이터와 원본 이미지 크기가 서로 다르게 보이므로, 통합 문서에서 사용할 정확한 logo data URI는 **[원본 자산 필요]** 및 **[확인 필요]**다. 임의로 새 로고를 만들거나 외부 URL로 교체하지 않는다.

## 5. 섹션과 제약사 구분

### 5.1 상위 섹션 순서

1. 품절 및 입고
2. 수수료
3. 의약품 정보

포함할 데이터가 없는 섹션의 표시 여부는 현재 지침상 명시되지 않았다: **[확인 필요]**. 빈 표나 빈 카드는 만들지 않는 방향이 현재 빈 셀 금지 원칙과 일치하지만, 실제 정상 예시로 확인해야 한다.

### 5.2 제약사 구분 방식

현재 기준 HTML은 제약사를 개별 상위 섹션으로 나누기보다 각 표 행 또는 카드의 회사명 필드로 표시한다.

- 표형: `회사` 컬럼
- 카드형: `.card-company`
- UI에서는 제약사 탭을 사용할 수 있으나 최종 HTML에서 제약사별 독립 탭 UI를 새로 만들지 않는다.
- 제약사별 별도 섹션으로 나누는지 여부는 현재 결과물에서 확인되지 않았다: **[확인 필요]**

## 6. 카드 구성

수수료·의약품 기준 파일의 카드 구조:

```html
<div class="table-wrap card-wrap">
  <div class="card-grid">
    <article class="info-card status-*">
      <div class="card-head">
        <div class="card-badge"><span class="badge status-*">...</span></div>
        <h3 class="card-title">제품명 및 함량</h3>
        <p class="card-company">제약사</p>
      </div>
      <div class="card-fields">
        <div class="card-field">
          <span class="card-label">항목명</span>
          <span class="card-value">값</span>
        </div>
      </div>
    </article>
  </div>
</div>
```

화면값:

- `.card-grid`: `repeat(auto-fit, minmax(320px, 1fr))`, gap 14px 또는 후반 덮어쓰기 15px
- `.info-card`: 흰 배경, 왼쪽 상태색 6~7px, radius 18px
- `.card-title`: 16px
- `.card-company`: 12.5px
- `.card-label`: 12px
- `.card-value`: 12.2px
- `.card-field`: label 104px + value flexible

상태색:

| 상태 클래스 | 색상 |
|---|---|
| `status-positive` | `#10b981` |
| `status-change` | `#3b82f6` |
| `status-caution` | `#f59e0b` |
| `status-danger` | `#ef4444` |
| `status-settlement` | `#8b5cf6` |
| `status-pending` | `#64748b` |

현재 Instructions상 단일 의약품의 상세정보가 많을 때 카드형을 사용할 수 있다. 수수료를 카드로 계속 렌더할지는 **[확인 필요]**다.

## 7. 표 구성

### 7.1 공통 표 스타일

```css
table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  overflow: hidden;
  background: #fff;
  border: 1px solid rgba(4, 120, 87, 0.58);
  border-radius: 16px;
}
```

화면 표 헤더:

```css
thead th {
  position: sticky;
  top: 0;
  z-index: 1;
  background: #047857;
  color: #fff;
  font-weight: 900;
  text-align: center;
  white-space: nowrap;
}
```

품절·입고 추가값:

- table min-width: 1180px
- table-layout: fixed
- table font-size: 12.2px
- thead: 11.7px, padding 8px 7px
- tbody: padding 8px 7px, line-height 1.35
- 회사명: 굵기 760
- 제품명 및 함량: 굵기 850
- 비고: 11.8px, 좌측 정렬

### 7.2 고정 컬럼

#### 품절 및 입고

`회사 | 품목명 및 함량 | 성분명 | 상태 | 입고일 | 품절일 | 규격 | 비고`

#### 수수료

`회사 | 제품명 및 함량 | 구분 | 기존 | 변동% | 적용 후 | 적용일 | 종료일 | 보험코드 | 비고`

#### 의약품 정보

`회사 | 제품명 및 함량 | 성분명 | 성분량 | 보험코드 | 약가 | 규격 | 포장단위 | 비고`

기본 컬럼 순서를 바꾸지 않는다. 이미지에 상세 필드가 추가로 존재하면 현재 지침이 허용한 최소 추가 컬럼 또는 상세 카드 필드로만 넣는다.

## 8. 글자 크기

### 8.1 화면 기준 파일 값

| 요소 | 확인된 값 |
|---|---|
| body | 13px, line-height 1.45 |
| h1 | `clamp(32px, 4.8vw, 54px)` |
| 부제 | 15px |
| 상위 섹션 제목 | 18px |
| note | 13px |
| 일반 표 | 12.5px |
| 카드 제목 | 16px |
| 카드 회사명 | 12.5px |
| 카드 라벨 | 12px |
| 카드 값 | 12.2px |

### 8.2 최종 PDF 최소값

기준 파일의 기존 작은 인쇄 글자값을 그대로 사용하지 않는다. 현재 GPT Instructions를 적용한다.

| 요소 | 최소값 |
|---|---|
| 문서 주제목 | 22pt |
| 상위 섹션 제목 | 16pt |
| PDF 본문 | 11.5pt |
| 표 헤더 | 11pt |
| 표 본문 | 10.5pt |

- 긴 표 때문에 전체 문서를 축소하지 않는다.
- 열 너비를 재배분하고 줄바꿈·다음 페이지를 허용한다.
- 제품명, 성분명, 비고는 `overflow-wrap: anywhere`와 자연스러운 줄바꿈을 유지한다.

## 9. 한글 폰트 처리

기준 파일의 폰트 스택:

```css
font-family:
  "Malgun Gothic",
  "Apple SD Gothic Neo",
  "Noto Sans KR",
  -apple-system,
  BlinkMacSystemFont,
  "Segoe UI",
  Arial,
  sans-serif;
```

요구사항:

- HTML은 `<meta charset="utf-8">`를 포함한다.
- 파일 저장은 UTF-8로 한다.
- PDF 생성용 Chromium 환경에 한글 glyph가 있는 폰트를 설치해야 한다.
- 외부 폰트 URL은 사용하지 않는다.
- 배포 가능한 실제 한글 폰트 파일과 라이선스: **[원본 자산 필요]**
- 폰트 파일을 HTML에 base64로 임베드하는지, 서버 시스템 폰트를 쓰는지는 **[확인 필요]**
- PDF 생성 후 한글 glyph 누락 검사를 수행한다.

## 10. 이미지 사용 규칙

허용:

- 기준 HTML에 포함된 base64 로고
- 기준 HTML에 포함된 base64 워터마크

금지:

- 외부 이미지 URL
- 새로 만든 로고
- 공고 원본 이미지를 최종 문서에 임의 삽입
- 이미지에 없는 아이콘 또는 장식 추가

원본 공고 이미지를 최종 보고서에 포함할지 여부는 현재 Instructions에 명시되지 않았으며 기존 기준 HTML에도 핵심 본문 이미지 삽입 구조가 확인되지 않았다. 따라서 포함하지 않는다.

## 11. 페이지 나눔 규칙

유지 또는 최소 보완:

```css
thead {
  display: table-header-group;
}

tr,
.info-card {
  break-inside: avoid;
  page-break-inside: avoid;
}
```

주의:

- 기준 파일에는 `header`, `.section` 전체에 `break-inside: avoid`가 있다.
- 긴 섹션 전체를 avoid 처리하면 빈 공간 또는 빈 페이지를 만들 수 있다.
- 현재 Instructions는 표가 다음 페이지로 이어져도 되며 빈 페이지를 금지한다.
- 따라서 긴 표 섹션은 페이지 분할을 허용하고, 헤더·개별 행·개별 카드 단위에서만 잘림을 방지하는 최소 인쇄 보완이 필요하다.
- 실제 변경 범위는 기준 파일과의 시각 비교 테스트로 제한한다.

## 12. A4 출력 규칙

최종 PDF:

- 용지: A4
- 방향: landscape
- CSS margin: 0
- Chromium margin: top/right/bottom/left 모두 0
- `printBackground: true`
- `displayHeaderFooter: false`
- `preferCSSPageSize: true`
- 배경 그래픽 포함
- 페이지 외곽의 불필요한 흰 여백 없음
- 빈 페이지 0
- 페이지 수 증가는 허용

권장 인쇄 핵심값:

```css
@media print {
  @page {
    size: A4 landscape;
    margin: 0;
  }

  html,
  body {
    margin: 0;
    padding: 0;
    min-height: 100%;
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }

  thead {
    display: table-header-group;
  }

  tr,
  .info-card {
    break-inside: avoid;
    page-break-inside: avoid;
  }
}
```

배경을 실제 종이 끝까지 채우는 방식은 Chromium의 무테 출력 한계와 PDF 렌더 방식에 따라 검증해야 한다. 현재 요구사항은 PDF 페이지 캔버스 내 무여백이며 실제 프린터의 물리적 무테 인쇄 보장은 범위 밖이다.

## 13. 파일명 규칙

기본 파일명:

- HTML: `제약사 공지.html`
- PDF: `제약사 공지.pdf`

현재 GPT 환경의 기본 경로:

- `/mnt/data/제약사 공지.html`
- `/mnt/data/제약사 공지.pdf`

MCP 플러그인 배포 환경의 실제 저장 경로·artifact URI 규칙은 **[확인 필요]**다. 사용자에게 보이는 파일명은 위 기본값을 유지한다. 사용자가 별도 파일명을 지정한 경우 이를 사용할 수 있다.

## 14. 인쇄용 CSS 충돌 처리

기준 파일에서 확인된 기존 인쇄값:

- `@page { size: A4 landscape; margin: 10mm; }`
- 표 본문 약 7.6~8.2px
- 카드 값 약 7.8px
- 상위 섹션 제목 약 12px

이 값은 현재 GPT Instructions의 무여백·최소 pt 기준과 충돌한다.

플러그인은 다음 순서로 처리한다.

1. 원본 화면 CSS와 DOM을 보존한다.
2. 원본 `@media print` 뒤에 현재 Instructions용 `print-overrides.css`를 내부 `<style>` 마지막에 삽입한다.
3. `@page margin: 0`과 최소 pt 값을 후순위로 적용한다.
4. PDF 생성 후 페이지 수, 빈 페이지, glyph, 승인 데이터 일치, 100% 가독성을 검사한다.
5. 글자를 줄이는 대신 페이지 수를 늘린다.

## 15. 현재 파일에서 가져와야 하는 자산

| 자산 | 출처 | 처리 |
|---|---|---|
| 전체 DOM 구조 | 두 기준 HTML | 직접 파싱·복제 |
| 전체 CSS 및 선언 순서 | 두 기준 HTML `<style>` | 원문 보존 |
| 로고 data URI | `.notice-header-logo[src]` | **[원본 자산 필요]**, SHA-256 고정 |
| 워터마크 data URI | `body::before background-image` | **[원본 자산 필요]**, SHA-256 고정 |
| 녹색 CSS 변수 | `:root` 블록 | 원문 보존 |
| 연녹색 최종 override | 파일 후반 CSS | 원문 보존 |
| 상태 배지 클래스 | 수수료·의약품 기준 파일 | 필요한 기존 클래스만 사용 |
| 표 열 너비/셀 클래스 | 품절·입고 기준 파일 | 고정 컬럼 렌더러에 재사용 |
| 빠른 이동 nav | 수수료·의약품 기준 파일 | 포함 섹션에 맞게 기존 구조만 데이터 교체 |
| 한글 폰트 파일 | 기준 HTML에는 파일 없음 | **[원본 자산 필요]** |
| 정상 PDF 인쇄값 | 현재 기준 파일과 Instructions가 충돌 | 정상 예시 PDF로 **[확인 필요]** |
