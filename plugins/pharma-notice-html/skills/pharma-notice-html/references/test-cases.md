# test-cases.md

## 공통 판정 기준

- 승인 전 최종 HTML/PDF가 생성되면 실패
- 이미지에 없는 사실이 추가되면 실패
- 사용자 수정값이 누락되거나 다른 필드가 임의 변경되면 실패
- 제외 항목이 최종 결과에 남으면 실패
- HTML과 PDF의 승인 content hash가 다르면 실패
- 실제 파일 없이 성공 URI를 반환하면 실패
- 한글 깨짐, 빈 페이지, 기본 머리글/바닥글, 불필요한 PDF 외곽 여백은 실패

## 테스트 시나리오

| ID | 시나리오 | 입력 | 예상 동작 | 예상 출력 | 실패 조건 |
|---|---|---|---|---|---|
| TC-01 | 이미지 한 장 | 한 제약사, 한 제품의 선명한 공지 이미지 1장 → `업로드 완료` | 이미지 1번 부여, 완료 후 분석, 유형 분류, 초안 생성 | totalImages=1, sourceImageIndex=1인 항목, REVIEW_DRAFT | 완료 전 분석, 이미지에 없는 값 추가, 항목 누락 |
| TC-02 | 이미지 여러 장 | 서로 다른 공지 이미지 5장, 두 번에 나눠 업로드 → `업로드 완료` | 모든 이미지를 같은 notice에 누적, 순서대로 1~5 부여 후 분석 | totalImages=5, 전체 확인 목록 | 첫 묶음만 분석, 순번 누락·중복, 일부 이미지 미처리 |
| TC-03 | 업로드 완료 전 분석 요청 | 이미지 2장 후 `지금 분석해줘` | 분석 도구를 호출하지 않고 업로드 계속 안내 | IMAGE_UPLOADING 유지, 완료 문구 요청 | 초안 생성 또는 analyze_notice_images 호출 |
| TC-04 | 같은 이미지 중복 업로드 | 동일 파일을 2번 업로드 후 완료 | SHA-256 또는 동등 중복 판정, 한 번만 항목 생성 | duplicateImageIndexes에 2, 결과 항목 1개 | 같은 제품 행이 2개 생성, 원본 번호 추적 상실 |
| TC-05 | 글자가 흐린 이미지 | 회사명은 보이나 제품명·보험코드가 흐린 이미지 | 읽을 수 있는 회사명은 유지, 핵심 필드는 판독불가, 재업로드 요청 | reuploadRequiredImageIndexes 포함, 다른 판독값 유지 | 전체 분석 중단, 제품명·보험코드 추정 |
| TC-06 | 재업로드도 판독 실패 | TC-05 이미지의 재업로드가 여전히 흐림 | 해당 유형의 수기 입력 양식 제공 | manualInputRequiredImageIndexes 포함, 한 줄 입력 형식 안내 | 계속 같은 재업로드만 요구, 작업 전체 폐기 |
| TC-07 | 제약사 여러 곳 | A사 품절 공지, B사 수수료 공지, C사 신제품 공지 | 회사별 그룹, 상위 섹션 고정 순서로 정렬 | companies 3개, 섹션 순서 품절 및 입고→수수료→의약품 정보 | 회사 혼합, 업로드 순서나 섹션 순서 오류 |
| TC-08 | 제품명이 없는 공지 | 회사·상태만 있고 제품명이 보이지 않는 이미지 | productName=null, 최종 표시 정책에 따라 미표기/판독불가 | 수정 가능한 빈/확인 필요 필드와 sourceImageIndex | 유사 제품명 추정, 빈 셀 그대로 출력 |
| TC-09 | 날짜가 없는 공지 | 품절 상태와 제품만 있고 날짜 없음 | effectiveDate=null 또는 관련 날짜 필드 null, 최종 문서 표시값 적용 | 날짜 미표기 상태의 초안 | 오늘 날짜나 공고 작성일을 적용일로 추정 |
| TC-10 | 보험코드가 없는 공지 | 신제품명·성분·약가만 있고 보험코드 없음 | insuranceCode=null, 최종 표시 `공고 이미지 미표기` | 의약품 정보 초안 | 외부 DB에서 보험코드 조회 또는 임의 생성 |
| TC-11 | 수수료율 인상 | 기존 45%, 5% 인상, 적용일 명시 | previousRate=45%, changeRate=+5%, newRate=50% 계산 | 수수료 고정 필드와 계산 결과 | 45%의 5%인 47.25%로 계산하거나 적용 후 비움 |
| TC-12 | 수수료율 인하 | 기존 50%, 5% 인하 | 인하 방향을 차감하여 newRate=45% | 변동%와 적용 후 분리 | 55%로 계산, 인하 방향 누락 |
| TC-13 | 수수료 계산 불가 | 기존값 일부가 흐리고 `5% 인상`만 판독 | 기존은 판독불가, changeRate는 유지, newRate=`확인 중` 표시 | 경고가 있는 수수료 초안 | 임의 기존율 선택 또는 newRate 추정 |
| TC-14 | 품절과 입고가 함께 있는 공지 | 같은 회사에서 제품 A 품절, 제품 B 입고 예정 | 둘 다 `품절 및 입고` 상위 섹션, 상태별 하위 제목 가능 | 8컬럼 표에 두 항목 | 품절·입고를 잘못된 상위 섹션으로 분리 |
| TC-15 | 한 공고가 여러 장 | 1장은 제품 목록, 2장은 적용일·비고 | 연속 페이지로 병합, 같은 제품의 상세 필드 통합 | 제품별 단일 item, source image 연결 유지 | 페이지별 중복 item, 2장의 상세정보 누락 |
| TC-16 | 다른 이미지 값 충돌 | 같은 제품 보험코드가 이미지 1과 3에서 다름 | 임의 선택하지 않고 conflict 표시, 승인 차단 | conflicts에 필드·이미지 번호 표시 | 한 값을 자동 선택, 충돌 미표시 |
| TC-17 | 특정 항목 제외 | 초안 3개 중 item-2 `결과물에서 제외` 저장 | exclude_notice_items 호출, excluded=true | 편집 화면에는 제외 배지, 미리보기·HTML·PDF에는 item-2 없음 | item-2가 최종 문서에 남음, 다른 항목 삭제 |
| TC-18 | 사용자 수정 후 승인 | productName·price 수정 저장 → 전체 초안 재표시 → `그대로` | patch만 반영, revision 증가, 전체 초안 재조회, 승인 hash 고정 | 수정값이 포함된 APPROVED snapshot | 수정값만 부분 표시, 다른 필드 임의 변경, 이전 revision 승인 |
| TC-19 | 그대로 승인 | 분석 결과 변경 없이 `그대로` | 최신 draft hash로 approve_notice 호출, HTML→PDF 순차 생성 | HTML/PDF 두 artifact와 동일 content hash | 승인 도구 없이 생성, HTML만 만들고 종료 |
| TC-20 | 승인 전 PDF 생성 요청 | REVIEW_DRAFT에서 `PDF 생성` 클릭/요청 | PDF 도구 호출 금지, 승인 필요 상태 안내 | APPROVAL_REQUIRED 또는 버튼 비활성 | PDF 생성 성공, 임시 PDF를 최종 링크로 제공 |
| TC-21 | HTML 생성 실패 | 승인 후 기준 HTML 파일 읽기 권한 제거 | generate_notice_html 실패, PDF 호출하지 않음 | TEMPLATE_ASSET_MISSING 또는 HTML_RENDER_FAILED, 가짜 링크 없음 | 성공 URI 반환, PDF 생성 시도 |
| TC-22 | PDF 생성 실패 | HTML 성공 후 Chromium 비정상 종료 | HTML artifact 유지, PDF 실패만 명시 | HTML 다운로드 가능, PDF_RENDER_FAILED | HTML artifact도 삭제, PDF 가짜 링크 반환 |
| TC-23 | 한글 깨짐 | 회사명·성분명에 한글과 특수문자 포함, 서버 한글 폰트 제거 | glyph 검사 실패, PDF를 성공 처리하지 않음 | KOREAN_FONT_UNAVAILABLE 또는 koreanGlyphCheck=FAIL | 네모(□) 글자가 있는 PDF를 성공으로 반환 |
| TC-24 | 페이지 넘침 | 품절·입고 120행, 긴 제품명·비고 | 글자 최소값 유지, 표 헤더 반복, 여러 페이지 생성 | blankPageCount=0, readableAt100% 통과 | 한 페이지 축소, 표 글자 10.5pt 미만, 행 잘림 |
| TC-25 | 카드 페이지 경계 | 상세 의약품 카드 20개 | 카드 단위 break-inside avoid, 필요 시 다음 페이지 | 카드 내용 잘림 없음 | 카드가 중간에서 분리, 빈 페이지 발생 |
| TC-26 | 기본 PDF 여백 검사 | 승인된 짧은 공지 | A4 landscape, CSS/Chromium margin 0, header/footer off | marginMm=0, displayHeaderFooter=false | 10mm 여백 또는 URL·날짜·페이지번호 출력 |
| TC-27 | HTML과 PDF 내용 일치 | 승인 데이터의 제품명·코드·수수료·날짜를 샘플링 | PDF가 생성 HTML과 같은 승인 hash를 사용 | approvedContentMatch=true | HTML에만 수정값 반영, PDF에 이전값 존재 |
| TC-28 | 재분석 요청 | REVIEW_DRAFT에서 `재분석` 명시 | reset_notice_analysis 후 같은 이미지로 재분석, 승인 PENDING | 새 revision의 REVIEW_DRAFT | 기존 승인 유지, reset 없이 분석 덮어쓰기 |
| TC-29 | 수기 입력 통합 | 이미지 4 수수료 수기 행 제출 | 기존 전체 초안과 병합, 중복·충돌 재검사, 전체 목록 재제시 | 수기 item도 일반 수수료 항목으로 표시 | 수기 행만 별도 표시, 바로 승인/생성 |
| TC-30 | 같은 요청 중복 실행 | 동일 idempotencyKey로 HTML 생성 요청 2회 | 첫 artifact를 재사용 | artifactId와 sha256 동일, 파일 1개 | 파일 2개 생성, revision 중복 증가 |
| TC-31 | version conflict | revision 3 화면에서 저장하는 동안 서버가 revision 4가 됨 | 저장 거절 후 최신 초안 조회 | VERSION_CONFLICT, 자동 덮어쓰기 없음 | revision 4를 사용자 확인 없이 덮어씀 |
| TC-32 | 파일명 규칙 | 별도 지정 없이 승인 | 기본 한글 파일명 사용 | `제약사 공지.html`, `제약사 공지.pdf` | 영문 임의 파일명, 경로 노출, 확장자 오류 |

## 도구 계약 테스트

### analyze_notice_images

- `uploadComplete=false` 입력은 schema 또는 `UPLOAD_NOT_COMPLETE`로 실패해야 한다.
- sourceImageIndex는 1 이상이며 중복 인덱스를 거절하거나 명시적으로 정규화해야 한다.
- 같은 idempotencyKey에 다른 이미지 목록을 보내면 `IDEMPOTENCY_CONFLICT`여야 한다.

### update_notice_items

- 존재하지 않는 itemId는 `ITEM_NOT_FOUND`.
- `expectedRevision` 불일치는 `VERSION_CONFLICT`.
- 빈 patch는 schema validation 실패.
- 제출되지 않은 필드는 변경되지 않아야 한다.

### approve_notice

- 최신 content hash가 아니면 `APPROVAL_HASH_MISMATCH`.
- 재업로드 필요 이미지가 있으면 `UNRESOLVED_REQUIRED_INPUT`.
- 승인 성공 뒤 approvedRevision과 approvedContentHash는 변경 불가 snapshot이어야 한다.

### generate_notice_html

- `approvalStatus=PENDING`이면 `APPROVAL_REQUIRED`.
- 외부 CSS/JS/이미지 URL 수가 0이 아니면 validation 실패.
- 파일 첫 토큰이 doctype이 아니면 실패.
- excluded 항목이 존재하면 실패.

### generate_notice_pdf

- HTML artifact의 sha256 또는 contentHash 불일치는 `HTML_ARTIFACT_MISMATCH`.
- A4, landscape, margin 0, printBackground, header/footer off를 검증한다.
- 한글 glyph, 빈 페이지, 승인 데이터 일치 중 하나라도 실패하면 최종 성공으로 반환하지 않는다.

## 회귀 테스트 기준

기준 HTML과 생성 HTML의 비데이터 영역을 비교한다.

허용 차이:

- 문서 주제목을 현재 지침의 `제약사 공지`로 교체
- 포함 섹션과 실제 행/카드 데이터
- 현재 Instructions를 충족하기 위한 인쇄 전용 최소 override
- section 번호와 quick-nav 항목의 데이터 기반 변경

허용하지 않는 차이:

- 새 색상 팔레트
- 새 로고
- 외부 폰트/이미지 URL
- 헤더·카드·표의 임의 재디자인
- 화면용 문서 폭·배경·그림자 변경
- 승인 데이터 이외의 문구 추가
