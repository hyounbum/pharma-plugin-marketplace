# Pharma Notice HTML Codex Plugin

제약사 공지 이미지를 승인형 초안으로 구조화하고, 사용자의 명시적 승인 후 내장 기준 디자인을 이용해 HTML과 PDF를 생성하도록 안내하는 Codex 플러그인입니다.

> 사내 전용 배포본입니다. 외부 배포 및 무단 재배포를 금지합니다.

배포 및 관리 담당: `beodnamu_ITteam`

## 직원용 시작 안내

GitHub 계정 없이 ZIP으로 전달받은 직원은 다음 문서를 먼저 확인하세요.

- `START-HERE.txt`: 가장 짧은 설치 순서
- `직원용_설치_및_사용설명서.md`: 다운로드, 설치, 사용, 업데이트, 문제 해결 전체 안내

## 포함 항목

- `plugins/pharma-notice-html/.codex-plugin/plugin.json`: 플러그인 매니페스트
- `plugins/pharma-notice-html/skills/`: 공지 분석·검토·승인·생성 절차
- `plugins/pharma-notice-html/assets/`: HTML 생성에 사용하는 기준 디자인
- `.agents/plugins/marketplace.json`: 팀 배포용 마켓플레이스

## 설치

저장소를 내려받은 후 PowerShell에서 다음 명령을 실행합니다.

```powershell
codex plugin marketplace add <이-저장소의-절대경로>
codex plugin add pharma-notice-html@pharma-team
```

설치가 끝나면 새로운 Codex 작업을 열고 다음과 같이 시작합니다.

```text
@pharma-notice-html 제약사 공지 이미지를 올릴게요. 업로드 완료 후 분석해 주세요.
```

## 업데이트

1. 원본 파일을 수정합니다.
2. `.codex-plugin/plugin.json`의 `version`을 변경합니다.
3. 아래 검증 명령을 실행합니다.
4. 저장소를 배포하고 사용자가 플러그인을 다시 설치하게 합니다.

```powershell
codex plugin add pharma-notice-html@pharma-team
```

업데이트 후에는 새로운 Codex 작업에서 테스트합니다.

## 배포 전 변경 항목

- `.codex-plugin/plugin.json`의 `author.name`과 `developerName`
- 필요한 경우 `homepage`, `repository`, `license` 및 지원 연락처
- `assets/`에 포함된 로고와 HTML 디자인의 재배포 권한
- 사내 공지 이미지와 생성 결과물의 보관·공유 정책

## 중요 동작 원칙

- 업로드 완료 의사를 확인하기 전에는 분석하지 않습니다.
- 이미지와 사용자가 제공한 수정값 이외의 정보로 의약품 데이터를 추정하지 않습니다.
- 사용자의 명시적 승인 전에는 HTML과 PDF를 생성하지 않습니다.
- 생성 결과는 원본 공지와 대조해 최종 검수해야 합니다.

## 이용 범위

이 배포본에는 오픈소스 라이선스가 부여되지 않았으며 사내 업무용으로만 사용할 수 있습니다. 코드·문서·로고·HTML 디자인 자산을 외부에 공개하거나 재배포하지 마세요.
