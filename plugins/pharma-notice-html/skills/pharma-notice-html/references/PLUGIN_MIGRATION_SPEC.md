# PLUGIN_MIGRATION_SPEC.md

## 0. 문서 목적과 분석 범위

이 문서는 현재 GPT **제약사 공지 통합 HTML**의 동작을 Node.js·TypeScript 기반 MCP 플러그인으로 옮기기 위한 전환 명세다.

분석 근거는 다음으로 제한한다.

1. 현재 GPT의 이름, 설명, 전체 Instructions, 활성 능력
2. 지식 파일 `품절 및 입고 양식.html`
3. 지식 파일 `버드나무_제약사공지_수수료_의약품_연녹색배경.html`
4. 사용자가 이번 요청에서 재진술한 업무 흐름

다음은 확인되지 않았다.

- 현재 GPT의 대화 시작 문구 원문: **[확인 필요]**
- 실제 운영 대화 전체와 과거 정상·실패 산출물: **[확인 필요]**
- 이미지 판독에 사용된 모델명·버전·파라미터: **[확인 필요]**
- 운영 환경의 파일 보존 기간, 인증, 개인정보 처리 기준: **[확인 필요]**

이 문서는 현재 GPT의 업무 규칙을 보존하기 위한 명세이며, 외부 의약품 데이터 검색, 자동 약가 조회, 승인 없는 파일 생성 같은 새 기능을 포함하지 않는다.

---

# 1. 현재 GPT 기능 분석

| 기능명 | 기능 설명 | 입력값 | 출력값 | 실행 조건 | 예외 처리 | 사용자 승인 필요 여부 | 현재 지침에서 확인된 규칙 | 플러그인 구현 시 필요한 구성요소 |
|---|---|---|---|---|---|---|---|---|
| 다회 이미지 수집 | 사용자가 여러 차례 나눠 올린 이미지를 하나의 작업으로 누적한다. | 이미지 파일, 업로드 순서, 선택적 묶음 번호 | 이미지 목록, 총 이미지 수, 순번 | 사용자가 완료 의사를 밝히기 전 | 첫 묶음에는 계속 업로드하고 마지막에 `업로드 완료`라고 보내도록 안내 | 아니오 | 여러 차례 업로드를 같은 작업으로 취급, 업로드 순서대로 번호 부여 | 세션/작업 저장소, 파일 참조 저장, 이미지 인덱서 |
| 업로드 완료 게이트 | 완료 문구 전에는 분석을 확정하지 않는다. | `업로드 완료`, `사진 끝`, `전부 올림` 등 | 분석 가능 상태 | 완료 의사 명시 | 완료 전 분석 요청은 거절하고 업로드 상태 유지 | 완료 문구가 사실상 확인 역할 | 완료 전 분석 결과나 확인 목록 확정 금지 | 상태 머신, 완료 의도 판정, 명시적 플래그 |
| 이미지 중복·연속 페이지 처리 | 중복 이미지를 제거하고 한 공고의 여러 장을 통합한다. | 이미지 해시/유사도, 순서, 판독 내용 | 중복 표시, 통합 공고 | 분석 단계 | 충돌값은 임의 선택하지 않고 확인 목록에 표시 | 충돌 수정 시 필요 | 중복 이미지·중복 제품 행은 한 번만 등록, 이어지는 정보는 통합 | 해시/지각 해시, 그룹핑 로직, 충돌 검출기 |
| 이미지 판독 | 이미지에 보이는 내용만 추출한다. | 업로드 이미지 | 회사, 제품, 성분, 코드, 약가, 수수료, 날짜, 상태 등 | 업로드 완료 후 | 흐림/잘림은 판독 가능한 부분만 처리, 핵심값은 재업로드 요청 | 분석 자체는 아니오 | 외부 웹 검색으로 핵심 데이터 보완 금지, 추정 금지 | 비전 모델 호출 계층, 구조화 출력 검증 |
| 공고 유형 분류 | 판독 항목을 상위 섹션과 세부 상태로 분류한다. | 판독 텍스트 | 품절 및 입고 / 수수료 / 의약품 정보 | 이미지 분석 중 | 불명확하면 기타 또는 확인 필요 | 검토 단계에서 수정 가능 | 혼합 순서: `품절 및 입고` → `수수료` → `의약품 정보` | 분류 규칙, enum 매핑, 섹션 정렬기 |
| 누락값 표준화 | 빈 셀 없이 현재 지침의 표준 문구로 표시한다. | 누락·불명확 필드 | `확인 중`, `미정`, `해당 없음`, `공고 이미지 미표기`, `공고 이미지 판독불가` 등 | 판독 결과 정규화/렌더링 | 회사명 미표기는 `확인 중`, 성분명 미표기는 지정 문구 사용 | 수정 시 필요 | 이미지에 없는 값은 추정하지 않음, 문서 빈 셀 금지 | 원시 null과 표시값 분리, 필드별 fallback 정책 |
| 수수료 계산 | 기존, 변동%, 적용 후를 분리하고 계산한다. | 기존율, 변동률, 인상/인하 방향 | 적용 후 수수료율 | 숫자 계산이 가능한 경우 | 계산 불가면 원값 미표기/판독불가, 적용 후 `확인 중` | 결과 검토 시 필요 | 적용 후 = 기존 + 변동%, 인하는 차감 | 안전한 퍼센트 파서, 계산기, 원문 보존 |
| 분석 결과 확인 목록 | 파일 생성 전에 전체 판독 결과를 줄 목록으로 제시한다. | 구조화 초안 | 섹션별 수정 확인 목록 | 모든 업로드 완료 및 분석 종료 | 대량이면 총 이미지 수·공고 수·재업로드/수기 입력 번호를 먼저 표시 | 예 | 표가 아닌 줄 목록, 지정된 안내 문구 사용 | 초안 조회 도구, 검토 UI/텍스트 포매터 |
| 사용자 수정 통합 | 수정값·수기 입력을 기존 전체 초안에 병합한다. | 항목 ID 또는 원문 키, 수정 필드 | 새 revision의 전체 초안 | 분석 결과 검토 후 | 중복·충돌 재검사, 전체 목록을 처음부터 다시 제시 | 최종 승인 별도 필요 | 수기 입력값만 따로 보여주지 않음 | 낙관적 잠금, patch validator, 재통합기 |
| 항목 제외 | 사용자가 제외한 항목을 최종 결과에서 제거한다. | 항목 ID, 제외 여부 | 제외 상태가 반영된 초안 | 사용자가 명시적으로 제외 | 모델 임의 제외 금지 | 제외 동작에 사용자 의사 필요 | 제외 요청 항목은 결과물에서 제거 | 안정 itemId, 제외 플래그, 렌더 필터 |
| 그대로 승인 | 현재 전체 초안을 고정한다. | `그대로`, `수정 없음`, `이대로`, `바로 생성` 등 | 승인 revision, 승인 데이터 해시 | 최신 전체 목록 제시 후 | 재업로드/수기 입력/충돌 미해결이면 승인 금지 | 예 | 승인 전 파일 생성 금지 | 승인 도구, content hash, immutable snapshot |
| HTML 생성 | 기준 HTML을 복제하고 데이터만 교체한 단일 HTML을 만든다. | 승인 데이터, 기준 HTML, base64 자산 | `제약사 공지.html` | 승인 후 | 템플릿/자산 누락, 렌더 실패, 내용 불일치 시 오류 | 승인으로 충족 | `<!doctype html>`, 내부 `<style>`, 외부 CSS/JS/이미지 URL 금지, 제목 `제약사 공지` | 템플릿 파서, section renderer, 자산 해시 검증, 파일 저장 |
| PDF 생성 | 최종 HTML과 동일 내용으로 A4 무여백 PDF를 만든다. | 승인 HTML artifact | `제약사 공지.pdf` | HTML 생성 성공 후 | 한글 깨짐, 빈 페이지, 작은 글자, 기본 여백, 내용 불일치 시 실패/재생성 | 승인으로 충족 | 배경 그래픽 포함, 브라우저 머리글/바닥글 제거, 100% 가독성 확인 | Chromium/Playwright 계열 렌더러, 폰트 검증, PDF 검사기 |
| 재업로드·수기 입력 대체 | 핵심값 판독 불가 시 재업로드를 받고, 재실패 시 유형별 수기 양식을 제공한다. | 이미지 번호, 재업로드 이미지 또는 수기 행 | 보완된 전체 초안 | 판독 불가 이미지 발생 | 작업 전체를 중단하지 않음 | 보완 후 전체 초안 승인 필요 | 유형별 수기 입력 필드 순서가 지침에 고정됨 | 재분석 상태, 이미지 교체 참조, 수기 입력 파서 |
| 최종 전달 | HTML과 PDF 링크를 제공한다. | 생성된 artifact | 두 실제 다운로드 링크 | 두 파일 생성 성공 | 실패한 파일 종류 명시, 가짜 링크 금지 | 이미 승인됨 | 첫 문장과 링크 형식이 지침에 지정됨 | artifact 저장소, 서명 URL 또는 다운로드 리소스 |

## 1.1 확인된 상위 섹션 규칙

- 문서 주제목: `제약사 공지`
- 상위 섹션명은 다음 세 가지로 제한한다.
  1. `품절 및 입고`
  2. `수수료`
  3. `의약품 정보`
- 혼합 공고 순서도 위 순서를 따른다.
- 문서 내 `제품명` 표기는 `제품명 및 함량`으로 통일한다.

## 1.2 확인된 고정 컬럼

### 품절 및 입고

`회사 | 품목명 및 함량 | 성분명 | 상태 | 입고일 | 품절일 | 규격 | 비고`

### 수수료

`회사 | 제품명 및 함량 | 구분 | 기존 | 변동% | 적용 후 | 적용일 | 종료일 | 보험코드 | 비고`

### 의약품 정보

`회사 | 제품명 및 함량 | 성분명 | 성분량 | 보험코드 | 약가 | 규격 | 포장단위 | 비고`

## 1.3 현재 자료에서 발견된 충돌과 적용 우선순위

1. **PDF 생성 여부**
   - 사용자가 재진술한 흐름은 “필요한 경우 PDF”라고 표현한다.
   - 현재 GPT Instructions는 승인 후 HTML과 PDF를 함께 생성하고 다시 묻지 않도록 명시한다.
   - 전환 기준: **현재 Instructions를 우선하여 두 파일을 함께 생성**한다.

2. **기준 HTML 인쇄값과 현재 Instructions**
   - 두 기준 HTML의 기존 인쇄 CSS에는 `A4 landscape`, `margin: 10mm`, 약 7.6~8.2px 표 글자가 존재한다.
   - 현재 Instructions는 `margin: 0`, 표 본문 10.5pt 이상 등 더 큰 글자를 요구한다.
   - 전환 기준: **화면 디자인은 복제하되 인쇄 CSS는 현재 Instructions의 최소 글자·무여백 규칙으로 제한적으로 덮어쓴다.**

3. **기준 파일의 제목과 현재 Instructions**
   - 기준 파일에는 `품절 및 입고`, `버드나무 제약사 공지` 제목이 있다.
   - 현재 Instructions는 주제목을 항상 `제약사 공지`로 고정한다.
   - 전환 기준: **생성물 주제목은 `제약사 공지`**다.

4. **수수료 카드형 예시와 고정 컬럼 지침**
   - 수수료·의약품 기준 파일은 카드형 예시를 포함한다.
   - 현재 Instructions는 수수료의 고정 컬럼을 정의한다.
   - 전환 기준: 수수료는 고정 컬럼을 보존하는 구조가 우선이다. 카드형을 계속 쓸지 여부는 실제 정상 산출물로 **[확인 필요]**다.

---

# 2. 전체 업무 흐름

## 2.1 상태 정의

| 상태 코드 | 사용자 표시명 | 의미 |
|---|---|---|
| `IMAGE_UPLOADING` | 이미지 업로드 중 | 이미지를 누적하지만 분석하지 않음 |
| `UPLOAD_COMPLETE` | 업로드 완료 | 사용자 완료 의사가 확인되고 이미지 집합이 고정됨 |
| `IMAGE_ANALYSIS` | 이미지 분석 | 중복·연속 페이지 통합 및 구조화 판독 수행 |
| `REVIEW_DRAFT` | 분석 결과 검토 | 전체 확인 목록을 표시하고 수정/제외/승인을 기다림 |
| `USER_EDITING` | 사용자 수정 | 저장된 수정값을 통합하고 새 전체 목록을 생성 |
| `APPROVED` | 그대로 승인 | 특정 revision과 내용 해시가 승인됨 |
| `HTML_GENERATING` | HTML 생성 | 승인 데이터로 단일 HTML 생성 중 |
| `HTML_READY` | HTML 생성 완료 | 승인 HTML artifact가 생성됨 |
| `PDF_GENERATING` | PDF 생성 | HTML 기반 PDF 생성·검증 중 |
| `COMPLETED` | 완료 | HTML과 PDF artifact가 모두 준비됨 |
| `ERROR_REANALYSIS` | 오류 또는 재분석 | 재업로드, 수기 입력, 재분석 또는 렌더 오류 처리 상태 |

## 2.2 상태 전이표

| 현재 상태 | 허용되는 사용자 명령/행동 | 호출 도구 또는 서버 동작 | 다음 상태 | 금지·예외 |
|---|---|---|---|---|
| 이미지 업로드 중 | 이미지 추가, 묶음 번호 전달, `업로드 완료`/`사진 끝`/`전부 올림` | 이미지 저장·번호 부여. 완료 문구 시 업로드 플래그 고정 | 계속 업로드 시 동일 상태, 완료 시 `UPLOAD_COMPLETE` | `분석해줘`만 있고 완료 문구가 없으면 분석하지 않음 |
| 업로드 완료 | 분석 시작 | `analyze_notice_images` | `IMAGE_ANALYSIS` | 완료 후 새 이미지 추가 정책은 **[확인 필요]**. 재업로드는 오류 흐름에서 처리 |
| 이미지 분석 | 일반적으로 추가 명령 없음 | 중복 제거, 연속 페이지 통합, 판독, 분류, 누락값 처리 | 성공 시 `REVIEW_DRAFT`, 판독 문제 시 `ERROR_REANALYSIS` | 외부 웹 검색, 추정값 보완 금지 |
| 분석 결과 검토 | 필드 수정, 항목 제외, `그대로`, `수정 없음`, `이대로`, `바로 생성`, 재분석, HTML 미리보기 | `update_notice_items`, `exclude_notice_items`, `approve_notice`, `reset_notice_analysis`, `render_notice_preview` | 수정 시 `USER_EDITING`, 승인 시 `APPROVED`, 재분석 시 `ERROR_REANALYSIS` | 승인 전 `generate_notice_html/pdf` 금지 |
| 사용자 수정 | 저장, 추가 수정, 제외/복원, `그대로`, 재분석, 미리보기 | 수정 도구 후 전체 초안 재통합·재표시 | 수정 계속 시 동일 상태, 승인 시 `APPROVED` | 수정값만 부분 표시하지 않고 전체 목록 재제시 |
| 그대로 승인 | 승인 후 생성 | `approve_notice` 성공 후 `generate_notice_html` | `HTML_GENERATING` | 미해결 재업로드·수기 입력·충돌이 있으면 승인 실패 |
| HTML 생성 | 취소/수정 명령은 현재 규칙에 없음 | 템플릿 자산 로드, 데이터 삽입, 검증, 파일 저장 | 성공 시 `HTML_READY`, 실패 시 `ERROR_REANALYSIS` | 승인 hash와 다른 데이터 사용 금지 |
| PDF 생성 | PDF 생성 요청 또는 승인 후 자동 진행 | `generate_notice_pdf` | 성공 시 `COMPLETED`, 실패 시 `ERROR_REANALYSIS` | HTML 없이 PDF 단독 생성 금지, 내용 재가공 금지 |
| 오류 또는 재분석 | 선명한 이미지 재업로드, 유형별 수기 입력, `재분석` | `reset_notice_analysis` → `analyze_notice_images`, 또는 `update_notice_items` | 해결 시 `REVIEW_DRAFT`/`USER_EDITING`, 렌더 재시도 시 생성 상태 | 재업로드도 불가하면 수기 입력 양식 제공, 작업 전체 중단 금지 |

## 2.3 승인 불변식

- 승인 대상은 `noticeId + revision + approvedContentHash`의 조합이다.
- 승인 이후 데이터가 한 글자라도 바뀌면 기존 승인은 유효하지 않다.
- 제외된 항목은 승인 해시 계산에는 제외 상태와 함께 포함하고, 렌더 본문에서는 제거한다.
- HTML과 PDF는 같은 승인 해시와 같은 HTML artifact를 기준으로 생성한다.
- HTML/PDF 생성 도구는 승인 데이터를 수정할 수 없다.

## 2.4 누락·불명확 값 처리

데이터 계층에서는 `null`을 허용할 수 있으나, 최종 문서에서는 빈 셀을 만들지 않는다.

| 상황 | 최종 표시값 |
|---|---|
| 회사명 미표기 | `확인 중` |
| 성분명 미표기 | `공고 이미지 성분 미표기` |
| 이미지에 값이 없음 | 문맥에 따라 `미정`, `해당 없음`, `공고 이미지 미표기` |
| 글자가 있으나 판독 불가 | `공고 이미지 판독불가` |
| 수수료 계산 불가 | 적용 후 `확인 중` |

---

# 10. 구현 프로젝트 구조

```text
pharma-notice-plugin/
├─ package.json
├─ package-lock.json
├─ tsconfig.json
├─ eslint.config.js
├─ vitest.config.ts
├─ .env.example
├─ README.md
├─ skills/
│  └─ pharma-notice/
│     ├─ SKILL.md
│     └─ resources/
│        ├─ notice-schema.json
│        ├─ mcp-tools-spec.json
│        └─ template-requirements.md
├─ src/
│  ├─ server.ts
│  ├─ config/
│  │  ├─ env.ts
│  │  └─ constants.ts
│  ├─ mcp/
│  │  ├─ register-tools.ts
│  │  ├─ register-resources.ts
│  │  └─ result-envelope.ts
│  ├─ tools/
│  │  ├─ analyze-notice-images.ts
│  │  ├─ get-notice-draft.ts
│  │  ├─ update-notice-items.ts
│  │  ├─ exclude-notice-items.ts
│  │  ├─ approve-notice.ts
│  │  ├─ render-notice-preview.ts
│  │  ├─ generate-notice-html.ts
│  │  ├─ generate-notice-pdf.ts
│  │  └─ reset-notice-analysis.ts
│  ├─ domain/
│  │  ├─ notice-types.ts
│  │  ├─ notice-state-machine.ts
│  │  ├─ category-mapper.ts
│  │  ├─ duplicate-detector.ts
│  │  ├─ merge-continuation-pages.ts
│  │  ├─ conflict-detector.ts
│  │  ├─ missing-value-policy.ts
│  │  ├─ fee-calculator.ts
│  │  ├─ approval-hash.ts
│  │  └─ content-equivalence.ts
│  ├─ vision/
│  │  ├─ image-analyzer.ts
│  │  ├─ structured-output.ts
│  │  └─ prompts.ts
│  ├─ schemas/
│  │  ├─ notice-schema.json
│  │  ├─ tool-inputs.ts
│  │  ├─ tool-outputs.ts
│  │  └─ validators.ts
│  ├─ store/
│  │  ├─ notice-repository.ts
│  │  ├─ artifact-repository.ts
│  │  ├─ idempotency-repository.ts
│  │  └─ file-session-repository.ts
│  ├─ templates/
│  │  ├─ source/
│  │  │  ├─ 품절 및 입고 양식.html
│  │  │  └─ 버드나무_제약사공지_수수료_의약품_연녹색배경.html
│  │  ├─ extracted/
│  │  │  ├─ base.css
│  │  │  ├─ print-overrides.css
│  │  │  ├─ logo.data-uri.txt
│  │  │  └─ watermark.data-uri.txt
│  │  ├─ partials/
│  │  │  ├─ header.ts
│  │  │  ├─ stock-supply-table.ts
│  │  │  ├─ fee-table.ts
│  │  │  ├─ drug-table.ts
│  │  │  └─ drug-detail-card.ts
│  │  └─ render-template.ts
│  ├─ pdf/
│  │  ├─ pdf-generator.ts
│  │  ├─ print-settings.ts
│  │  ├─ glyph-check.ts
│  │  ├─ blank-page-check.ts
│  │  └─ readability-check.ts
│  ├─ ui/
│  │  ├─ notice-editor.html
│  │  ├─ notice-editor.ts
│  │  ├─ notice-editor.css
│  │  ├─ state-adapter.ts
│  │  └─ tool-client.ts
│  ├─ files/
│  │  ├─ temp-manager.ts
│  │  ├─ safe-file-name.ts
│  │  └─ cleanup-policy.ts
│  ├─ errors/
│  │  ├─ error-codes.ts
│  │  ├─ plugin-error.ts
│  │  └─ error-mapper.ts
│  └─ logging/
│     ├─ logger.ts
│     └─ redaction.ts
├─ public/
│  └─ notice-editor.html
├─ assets/
│  ├─ templates/
│  ├─ examples/
│  └─ fonts/
├─ test/
│  ├─ unit/
│  ├─ integration/
│  ├─ contract/
│  ├─ fixtures/
│  │  ├─ images/
│  │  ├─ drafts/
│  │  ├─ html/
│  │  └─ pdf/
│  └─ e2e/
└─ scripts/
   ├─ extract-template-assets.ts
   ├─ validate-template-hash.ts
   ├─ validate-json-schemas.ts
   └─ compare-approved-output.ts
```

## 10.1 패키지 구성

권장 범주이며 배포 시점 공식 패키지명과 버전은 잠금 파일로 고정한다.

- MCP 서버 SDK: `@modelcontextprotocol/sdk`
- 입력 검증: `zod` 및/또는 JSON Schema validator
- PDF 생성: Chromium 자동화 라이브러리
- 테스트: 단위·계약·E2E 테스트 러너
- 로깅: 구조화 JSON 로거
- 이미지 해시: SHA-256 및 필요 시 지각 해시 구현
- HTML 파싱: 기준 HTML 구조 검증용 DOM parser

## 10.2 환경 변수

```dotenv
PORT=3000
MCP_PATH=/mcp
ARTIFACT_DIR=/data/artifacts
TEMP_DIR=/data/tmp
TEMPLATE_DIR=/app/assets/templates
NOTICE_RETENTION_HOURS=[확인 필요]
LOG_LEVEL=info
LOG_REDACT_IMAGE_TEXT=true
KOREAN_FONT_DIR=[원본 자산 필요]
VISION_MODEL=[확인 필요]
VISION_MODEL_VERSION=[확인 필요]
MAX_IMAGE_COUNT=[확인 필요]
MAX_IMAGE_BYTES=[확인 필요]
```

## 10.3 로그 규칙

- 기록: requestId, noticeId, toolName, state 전이, revision, artifact hash, 오류 코드
- 기록 금지: 원본 이미지 바이너리, 전체 OCR 텍스트, 보험코드·제품정보 전체 덤프, 인증 토큰
- 사용자 수정 전후값의 전체 원문 저장 여부: **[확인 필요]**
- 로그 보존 기간과 접근 권한: **[확인 필요]**

## 10.4 오류 처리 원칙

- 오류는 `code`, `message`, `retryable`, `details` 구조로 반환한다.
- 실패한 도구만 명확히 표시하고 기존 성공 artifact를 삭제하지 않는다.
- PDF 실패 시 HTML 성공 상태는 유지한다.
- 승인 해시 불일치는 자동 수정하지 않고 `OUTPUT_CONTENT_MISMATCH`로 실패시킨다.
- 파일 생성 실패 시 가짜 URI를 반환하지 않는다.
- 재시도 가능한 오류도 idempotencyKey를 유지해 중복 artifact 생성을 막는다.
