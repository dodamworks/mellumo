# 멜루모 오픈소스 준비 조사

기준일: 2026-11-05. 사용자 결정: Postiz 기능을 최대한 재사용하고 멜루모를 AGPL-3.0으로 공개한다. 코드 공개와 실제 서비스 출시를 구분한다. 아래 내용은 저장소 운영 준비이며 기존 제품 전체를 재시험하는 계획이 아니다.

## 1. 지금 공개 저장소에 준비한 것

| 항목 | 처리 |
|---|---|
| 코드와 출처 | Postiz v2.25.3 원본 Git 이력·저작자 유지, 출발 커밋 고정, upstream 원격 추가 |
| 라이선스 | 원본 LICENSE 유지, NOTICE에 변경 사실·날짜·Mellumo 출처 기록 |
| README | 제품 목적, 현재 기반 단계, 미구현 방향, 실행 문서와 라이선스 연결 |
| 기여 규칙 | Mellumo용 CONTRIBUTING, 이슈 양식, PR 양식, CODEOWNERS |
| 행동 규칙 | 기존 Contributor Covenant 본문 보존, Mellumo 연락 경로로 수정 |
| 보안 신고 | 실제 선택한 운영 이메일과 비공개 신고 안내, upstream 지원 약속과 분리 |
| 비밀정보 분리 | Git/Docker context에서 evaluation·실제 env·키·업로드·백업 제외 |
| 실행 재현 | 체크아웃 소스를 빌드하는 compose, 빈 키 example, 버전 고정된 Node/pnpm |
| CI 경계 | upstream 게시·외부 봇 workflow 보관, Mellumo의 가벼운 소스 검사만 활성화 |
| 업데이트 | upstream 갱신·Temporal 호환·백업 경계를 문서화 |

GitHub의 README·라이선스·기여/행동 규칙 안내는 [저장소 모범 사례](https://docs.github.com/en/repositories/creating-and-managing-repositories/best-practices-for-repositories), [시작 가이드](https://opensource.guide/starting-a-project/), [README 안내](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)를 참고했다. 템플릿 파일이 있다는 사실이 유지보수나 기능 안정성을 보증하지는 않는다.

## 2. AGPL을 실제로 운영하는 방법

원본 [AGPL-3.0 본문](../LICENSE)의 4·5·6·13절을 기준으로 준비한다. GNU 사이트가 조회 시 응답하지 않아 고정 upstream의 [동일 라이선스 원문](https://raw.githubusercontent.com/gitroomhq/postiz-app/a5affe62c74e39c1e7ac960d50d8418b807d532c/LICENSE)도 대조했다.

- 원본 저작권·라이선스·보증 부인 고지를 유지하고, 변경한 버전이라는 사실과 관련 날짜를 표시한다.
- 수정한 전체 covered work의 공개 조건을 AGPL로 유지한다. 신규 문서·코드를 더한다고 기존 코드의 저작권이 Mellumo로 바뀌는 것은 아니다.
- 수정 서비스를 네트워크로 제공할 때 실제 사용자에게 해당 실행 버전의 Corresponding Source를 무료로 받을 수 있는 접근을 눈에 띄게 제공한다. GitHub 저장소만 만들고 앱에서 찾을 방법이 없는 상태로 끝내지 않는다.
- 배포 커밋/태그와 소스·빌드/설치 지침을 연결한다. 미래 main 링크만으로 과거 배포 버전을 정확히 표현하지 않는다.
- 소스와 서버 비밀정보는 다르다. API key, 사용자 데이터, 운영 DB를 공개 저장소에 넣지 않는다. 실행에 필요한 설정 예시는 빈 값/설명으로 제공한다.
- 유료 호스팅도 가능하다. 코드의 자유와 호스팅 비용은 별개이며, ‘비상업적 사용만’ 같은 추가 제한을 AGPL 코드에 붙이지 않는다.

이번에는 소스 저장소 기반만 준비한다. 서비스 UI의 source/license 링크, 실제 배포 버전 표시와 호스팅 환경의 이행은 첫 서비스 배포 작업에서 연결해야 한다. 이를 이미 구현했다고 표시하지 않는다.

## 3. GitHub에서 설정할 것

| 설정 | 의도 |
|---|---|
| 공개 저장소와 설명 | 목적과 개발 단계가 분명하게 보이게 함 |
| Issues/PR와 templates | 재현 가능한 버그와 작업 목적을 받음 |
| Secret scanning / push protection | 지원되는 비밀 패턴 노출 감지 및 유입 방지 |
| Dependabot alerts/security updates | 알려진 의존성 취약점 확인과 업데이트 제안 |
| Private vulnerability reporting | 공개 이슈를 피하는 신고 경로 |
| 읽기 전용 Actions token | 필요하지 않은 저장소/패키지 쓰기 권한 축소 |
| main 보호 | force push와 삭제 방지. 초기 1인 운영에서 자기 자신이 승인해야 하는 불가능한 규칙은 피함 |

공개 저장소의 secret scanning은 무료 자동 검사 대상이지만, 모든 비밀을 찾는 보장은 아니다. Push protection도 지원 패턴과 설정에 따른다. [지원 패턴](https://docs.github.com/en/code-security/reference/secret-security/supported-secret-scanning-patterns) · [보안 빠른 시작](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository)

비공개 신고는 별도로 켜고 readback으로 확인해야 한다. [GitHub 설정 안내](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/configure-for-a-repository)

CI에서는 미검증 PR에 배포 키를 제공하지 않고 외부 action은 고정 commit을 우선한다. 원본의 ghcr.io/gitroomhq 게시나 Chrome 확장 배포를 Mellumo에서 그대로 실행하지 않는다. [Actions 보안](https://docs.github.com/en/actions/reference/security/secure-use) · [브랜치 보호](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)

실제 적용 여부는 저장소 생성 후 별도 영수증에 기록한다. 여기의 목록을 적용 완료로 해석하지 않는다.

## 4. 별도로 다뤄야 할 것

- **외부 구성요소:** Polotno·AI 모델/API·SNS 권한은 별도 조건. [외부 조건 목록](THIRD_PARTY.md).
- **브랜드:** Postiz 이름/로고의 원저작자 표기와 Mellumo의 제품 정체성을 구분. 대규모 문자열 치환으로 내부 프로토콜을 바꾸지 않음.
- **배포/출시:** 첫 Mellumo 이미지와 릴리스에는 대응 소스, 버전, 변경 내용, 설치/업데이트 지침을 함께 제공.
- **의존성 목록:** package/lock은 보존했으나 전체 dependency/asset 라이선스 감사나 SBOM 완성은 아직 아님. 배포 아티팩트와 함께 점진적으로 생성.
- **운영 정책:** 이슈 응답과 보안 유지보수는 실제 역량에 맞게 약속. 원본의 CNA·응답 시간·유료 지원을 승계한다고 주장하지 않음.
- **기여 수용:** 동일 AGPL 조건의 권리 있는 기여를 받고, 원본 Postiz CLA는 멜루모의 필수 조건으로 강제하지 않음. 별도 upstream PR은 그 프로젝트의 정책을 따름.

## 5. 실행 순서

지금은 코드 공개와 저장소 기반을 완성한다. 다음으로 기존 UI와 source/license 접근을 멜루모에 맞추고, 실제 목표 하나의 제작→실행→완료 보고를 연결한다. 사용 중 바꾼 경로에 필요한 확인만 한다. AGPL이라는 이유로 불필요한 승인 단계나 전면 재시험을 추가하지 않는다.

## 공개 파일 검사

공개 예정 스냅샷을 Gitleaks 8.30.1 기본 규칙으로 검사했다. 최초 23건은 모두 원본 `i18n.lock`의 번역 체크섬이었다. `.gitleaks.toml`에서 이 파일의 32자리 hex 체크섬 매핑 줄만 예외로 지정했다. 이 검사는 현재 공개 파일 기준이며 전체 Git 과거 이력의 비밀정보 부재나 모든 종류의 비밀정보 탐지를 보증하지 않는다. 로컬 평가 키·DB·실제 env는 공개 대상에서 제외한다.
