# Postiz · TryPost 상세 분석 — 멜루모 기반 선택

분석일: 2026-11-04 KST. 대상은 로컬에서 실행한 **Postiz v2.25.3 / TryPost v2.0.1**이다. 최신 `main` 전체를 평가한 결과가 아니다.

현재 추천은 **Postiz를 멜루모의 우선 기반으로 선택**하는 것이다. 사용자가 명확히 한 목표는 AI가 작업부터 완료 확인까지 수행하는 자동화이며, 사람 승인 중심 흐름을 기본으로 가정하지 않는다. 내장 agent·생성 도구·발행 workflow와 유지보수 생태계를 함께 평가했다. TryPost는 외부 agent용 도구 범위와 저장형 분석에서 여전히 강하다. 추천 근거와 추가 구현 범위는 [에이전트 중심 기반 선택](./agent-first-decision.md)을 우선한다. 실제 작업 완료율을 비교한 실험 결과는 아니다.

## 문서 구성

- [에이전트 중심 기반 선택](./agent-first-decision.md): 현재 요구에 따른 Postiz 추천과 반대 근거, 자동 완료에 필요한 연결.
- [Postiz 아키텍처](./postiz.md): 실행 구조, 데이터, 발행 워크플로, 세 가지 AI 경로, MCP, 운영·테스트.
- [TryPost 아키텍처](./trypost.md): 공유 Action 계층, 승인, 큐·복구, 분석 저장 구조, MCP, 운영·테스트.
- [멜루모 적용 및 통합 판단](./mellumo-integration.md): 재사용 경계, 합칠 때 생기는 문제, 검증 시나리오.
- [공식 문서 전체 목차와 조사 범위](./document-map.md): 343개 목차 항목의 분야별 목록과 본문 수집 여부.
- [로컬 MCP 도구 목록](./mcp-tool-inventory.json): 앞선 관찰에서 인증 정보를 제외한 도구 이름·입력 필드 목록.
- [근거 메타데이터](./sources.json): 고정 리비전, 이미지 digest, 문서 URL·수집 해시, 코드 근거 파일 해시.

## 범위와 근거 수준

공식 목차는 Postiz 177개, TryPost 166개 항목을 목록화했다. 합계 343개에는 문서 페이지 335개, OpenAPI 명세 링크 2개, GitHub·커뮤니티 등 외부 링크 6개가 포함된다. 이 중 구조·운영·기능 경계를 판단하는 핵심 39개 본문을 확보하고 관련 절을 대조했다. **목차의 문서를 전부 정독했거나 전체 코드를 줄 단위로 감사했다는 뜻은 아니다.** 저장소의 패키지·스키마·진입점·생성/발행 경로·인증/승인·AI/MCP·분석·운영/CI를 중심으로 읽었다.

근거를 다음처럼 구분한다.

| 구분 | 이번 보고서에서 뜻하는 것 |
|---|---|
| 코드 확인 | 고정 릴리스의 구현·설정·테스트 내용을 읽음 |
| 문서 확인 | 조사일의 공식 설명을 확인함. 릴리스보다 앞설 수 있음 |
| 로컬 관찰 | 앞선 동일 대화의 격리 실행·브라우저·MCP 테스트에서 실제 관찰함 |
| 해석 | 위 근거에서 도출한 멜루모 관점의 판단 |
| 미검증 | 실제 SNS/LLM 연결, 장애 주입, 부하, 전체 보안 감사 등 |

두 소스는 공식 릴리스 압축본이며 `.git` 체크아웃이 아니다. GitHub 공개 API에서 태그의 커밋을 다시 확인했다. TryPost 실행 이미지의 OCI revision은 이 커밋과 일치한다. Postiz 이미지는 digest로 고정했지만 OCI revision 정보가 없어, 실행 바이너리와 소스 태그의 정확한 일치는 확인하지 못했다. 따라서 코드 분석과 실행 관찰의 근거를 구분한다.

| 항목 | Postiz | TryPost |
|---|---|---|
| 태그 | v2.25.3 | v2.0.1 |
| 커밋 | `a5affe62c74e39c1e7ac960d50d8418b807d532c` | `b13cca4d834bc793ef895dba40f29ed050ec820d` |
| 해당 커밋 시각 UTC | 2026-10-24 13:27:20 | 2026-11-01 15:34:44 |
| 루트 라이선스 | AGPL-3.0 | AGPL-3.0 |

라이선스 때문에 제외하지 않는 사용자의 방향을 유지한다. 다만 루트 라이선스와 외부 SDK·모델·SNS API 이용 조건은 별도다. Postiz Polotno 편집기는 공식 문서상 상용 SDK 라이선스가 필요한 선택 기능이다. [P21] [T33] [Polotno 공식 설명](https://docs.postiz.com/self-host/configuration/polotno)

## 구조 비교

| 비교 기준 | Postiz | TryPost | 멜루모에 미치는 영향 |
|---|---|---|---|
| 기본 구조 | TypeScript 모노레포: Next.js + NestJS + Temporal orchestrator | PHP Laravel + Inertia/Vue 모듈형 모놀리스 | 합치면 TS·PHP와 두 비동기 실행 체계를 함께 유지 |
| 장시간 작업 | Temporal workflow/activity와 provider별 task queue | Horizon/Redis job + scheduler + DB 상태 | 둘 다 장애 처리가 있지만 운영 방식이 다름 |
| 내장 AI | Mastra 대화형 도구 에이전트, LangGraph 생성기, CopilotKit 연결 | 글쓰기·아이디어·축약·alt-text 중심의 Laravel AI | 내장 에이전트 경험은 Postiz가 더 풍부 |
| 외부 에이전트 제어 | 로컬 MCP 20개 도구 관찰 | 로컬 MCP 83개 도구 관찰 | 도구 수보다 수정·승인·성과 조회의 계약을 봐야 함 |
| 승인 | 공개 미리보기와 의견 수집 후 사람이 예약 | `pending_approval` + 사용자 권한 + 서버 분기 | 에이전트의 발행을 서버에서 제한하려면 TryPost가 유리 |
| 성과 분석 | provider API를 호출해 계정·게시물 지표 조회 | publication·일별 snapshot·수집 상태를 저장하고 조회 | 시계열 피드백 루프의 재료는 TryPost 쪽이 더 명시적 |
| SNS 범위 | 고정 코드에 35개 provider 등록 | 15개 플랫폼 enum, 네트워크 기준 13개 | Postiz가 더 넓음. 등록 수는 발행 성공 인증이 아님 |
| 게시물 모델 | 채널별 Post + group/batch + parentPost 스레드 | 채널별 독립 Post + post_group_id | 이름이 같아도 스레드·수정·그룹 의미가 다름 |
| 토큰 보관 | 검토한 Prisma 저장 경로에는 필드 암호화 변환이 보이지 않음 | Eloquent encrypted cast 지정 | Postiz는 별도 보호 설계 검토 필요. 전체 보안 점수는 아님 |
| 스키마 갱신 | Prisma `db push`, 릴리스 내 migration 디렉토리 미발견 | 129개 PHP migration, 기동 시 자동 적용 | 백업·업그레이드·롤백 경로를 다르게 준비 |
| 테스트 기반 | 해당 패턴의 테스트 파일 0개, Jest의 NX 참조 불일치 | `*Test.php` 728개, DB/browser CI 정의 | TryPost에 회귀 검증 재료가 더 많음. 통과 여부 미검증 |
| 로컬 구성 | 이번 compose 7개 서비스 | 이번 compose 3개 서비스 | 컨테이너 수와 실제 메모리·운영 난도는 별도 |

근거: [P01] [P02] [P05] [P08] [P11] [P15] [P16] [P17] [P18] [T05] [T06] [T09] [T12] [T18] [T20] [T25]. MCP 수는 앞선 로컬 `tools/list` 관찰값이며 설정과 버전에 따라 달라진다.

## 판단을 바꾼 세 가지 발견

### 1. 양쪽의 ‘승인’은 서로 다른 기능이다

Postiz는 미리보기 링크로 의견을 받고 작성자가 예약한다. 게시물 상태에 승인 대기 상태가 없고, 공식 문서도 발행 차단 상태가 아니라고 설명한다. Mastra 프롬프트의 ‘먼저 사용자 확인’은 모델 행동 지침이다. 서버에서 권한으로 발행을 막는 장치와 같지 않다. [P02] [P08] [Postiz 승인 문서](https://docs.postiz.com/general/approvals)

TryPost는 승인이 필요한 멤버가 예약/즉시발행을 요청하면 Action 계층에서 `pending_approval`로 저장한다. MCP도 사용자 권한을 따른다. **다만 관리자 REST API key와 직접 발행 권한의 MCP 연결은 승인 없이 발행할 수 있다.** 에이전트를 관리자 계정에 연결하면 승인 기능이 자동으로 안전장치가 되지 않는다. [T07] [T09] [T11] [TryPost MCP 권한](https://docs.trypost.it/ai/introduction)

### 2. TryPost는 아이디어 보드만 가져올 대상이 아니다

승인 규칙·채널별 게시물 모델·큐 잠금·성과 snapshot·권한·API/MCP가 묶여 있다. Postiz로 이식하려면 Vue 화면뿐 아니라 데이터와 상태 전이, 작업 처리, 테스트를 재구현해야 한다. ‘Postiz 본체 + TryPost 기능 몇 개’의 예상 비용을 낮게 잡으면 안 된다. [T07] [T09] [T18] [T26]

### 3. 둘 다 완성된 자율 SNS 운영자는 아니다

Postiz는 대화와 도구 실행, 콘텐츠 생성까지 갖추지만 확인한 기억 스키마는 브랜드 전략 전체를 표현하지 않는다. TryPost는 외부 에이전트가 운영하기 좋은 업무 도구가 있지만 내장 AI는 작업별 보조 기능이다. 목표→조사→전략→제작→승인→발행→성과→다음 실험을 지속적으로 조정하는 루프는 별도 조합이 필요하다. 댓글/DM 수집·응대도 이번 조사에서 완성된 공통 기능으로 확인하지 못했다. [P08] [P12] [T21]

## 문서와 코드가 어긋난 부분

- Postiz 공식 아키텍처는 Temporal을 설명하지만 developer guide에는 NX·Redis worker 구조가 남아 있다. 저장소 `CLAUDE.md`의 프런트엔드 Vite 표기도 현재 Next.js 실행과 다르다. 실행 지침은 package/scripts 및 현재 orchestrator를 우선해야 한다. [P01] [P04] [P20] [개발 문서](https://docs.postiz.com/contributing/developer-guide)
- Postiz 문서의 ‘34 platforms’와 고정 코드의 35개 provider 등록은 수와 단위가 다르다. TryPost는 15개 enum을 13개 network로 묶는다. 광고 숫자를 동일 단위의 비교값으로 쓰지 않았다. [P15] [T20]
- TryPost MCP 소개는 81개 도구라고 적지만 로컬은 83개였다. 실제 연결의 `tools/list`를 기준으로 사용해야 한다. [공식 소개](https://docs.trypost.it/ai/introduction)
- TryPost 9월 설계안의 물리적 테이블 구조는 이후 바뀌었다. 현재 Post 모델, 10월 merge/drop migration, `DEPLOYMENT.md`를 우선했다. `ANALYTIC.md`도 과거 목록임을 표시하고 있다. [T05] [T26] [T27] [T29]
- TryPost `composer.json`의 PHP `^8.2`만 보고 환경을 만들면 안 된다. 잠긴 의존성에는 더 높은 최소 버전이 있고, 로컬 생산 이미지의 실제 PHP는 8.5.12였다. [T01] [T02]

## 로컬 검증에서 확인한 것과 남은 것

앞선 실행에서 두 앱의 로그인, MCP 인증, 한국어 초안 생성→웹 편집→MCP 재조회가 동작했다. TryPost는 아이디어 생성/보드 확인도 했다. 무인증 API는 401, TryPost의 REST key를 MCP에 쓰면 403이었다. 실제 SNS 연결 없이 명시적 로컬 fixture 채널을 사용했다.

실제 SNS 발행·토큰 만료 복구·부분 발행 중 장애·실제 지표 수집·내장 AI 품질·성능 및 전체 테스트 suite는 검증하지 않았다. 이번 분석 단계에서 이를 새로 실행했다고 주장하지 않는다. Postiz의 AI 키 누락 시 Copilot 요청이 끝나지 않아 504가 발생하는 현상은 앞선 관찰과 코드가 일치했다. [P13]

멜루모의 다음 검증은 **목표를 받은 AI가 위임된 범위에서 제작·발행·결과 확인을 중간 수동 실행 없이 완수하는 한 사이클**이다. 승인 필요 여부는 사용자 정책에 따른다. 실행 후보와 성공 기준은 [현재 결정](./agent-first-decision.md)에 정리했다. 실제 계정 연결이나 외부 발행은 아직 실행하지 않았다.


## 코드 근거

각 링크는 분석한 릴리스의 고정 커밋을 가리킨다. 로컬 소스도 같은 경로에서 열 수 있다.

[P01]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/package.json
- [P01] [프로젝트 의존성과 실행 스크립트](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/package.json) · [로컬 파일](../../../package.json)
[P02]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/database/prisma/schema.prisma
- [P02] [데이터 모델과 상태](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/database/prisma/schema.prisma) · [로컬 파일](../../../libraries/nestjs-libraries/src/database/prisma/schema.prisma)
[P04]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/apps/orchestrator/src/app.module.ts
- [P04] [비동기 작업 구성](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/apps/orchestrator/src/app.module.ts) · [로컬 파일](../../../apps/orchestrator/src/app.module.ts)
[P05]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/temporal/temporal.module.ts
- [P05] [Temporal worker와 큐](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/temporal/temporal.module.ts) · [로컬 파일](../../../libraries/nestjs-libraries/src/temporal/temporal.module.ts)
[P08]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/chat/load.tools.service.ts
- [P08] [Mastra 에이전트와 기억](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/chat/load.tools.service.ts) · [로컬 파일](../../../libraries/nestjs-libraries/src/chat/load.tools.service.ts)
[P11]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/chat/tools/post.settings.tool.ts
- [P11] [MCP 설정 수정 범위](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/chat/tools/post.settings.tool.ts) · [로컬 파일](../../../libraries/nestjs-libraries/src/chat/tools/post.settings.tool.ts)
[P12]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/agent/agent.graph.service.ts
- [P12] [LangGraph 생성 파이프라인](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/agent/agent.graph.service.ts) · [로컬 파일](../../../libraries/nestjs-libraries/src/agent/agent.graph.service.ts)
[P13]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/apps/backend/src/api/routes/copilot.controller.ts
- [P13] [Copilot HTTP와 키 누락 처리](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/apps/backend/src/api/routes/copilot.controller.ts) · [로컬 파일](../../../apps/backend/src/api/routes/copilot.controller.ts)
[P15]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/integrations/integration.manager.ts
- [P15] [등록된 SNS 어댑터](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/integrations/integration.manager.ts) · [로컬 파일](../../../libraries/nestjs-libraries/src/integrations/integration.manager.ts)
[P16]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/database/prisma/integrations/integration.repository.ts
- [P16] [채널 토큰 저장 경로](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/database/prisma/integrations/integration.repository.ts) · [로컬 파일](../../../libraries/nestjs-libraries/src/database/prisma/integrations/integration.repository.ts)
[P17]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/database/prisma/prisma.service.ts
- [P17] [Prisma 클라이언트](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/libraries/nestjs-libraries/src/database/prisma/prisma.service.ts) · [로컬 파일](../../../libraries/nestjs-libraries/src/database/prisma/prisma.service.ts)
[P18]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/jest.config.ts
- [P18] [테스트 설정](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/jest.config.ts) · [로컬 파일](../../../jest.config.ts)
[P20]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/CLAUDE.md
- [P20] [개발 규칙과 워크플로 변경 규칙](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/CLAUDE.md) · [로컬 파일](../../../CLAUDE.md)
[P21]: https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/LICENSE
- [P21] [루트 라이선스](https://github.com/gitroomhq/postiz-app/blob/a5affe62c74e39c1e7ac960d50d8418b807d532c/LICENSE) · [로컬 파일](../../../LICENSE)
[T01]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/composer.json
- [T01] [PHP 의존성 선언](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/composer.json)
[T02]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/composer.lock
- [T02] [실제 잠긴 PHP 의존성](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/composer.lock)
[T05]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Models/Post.php
- [T05] [현재 게시물 모델](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Models/Post.php)
[T06]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Models/SocialAccount.php
- [T06] [채널과 암호화 캐스트](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Models/SocialAccount.php)
[T07]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Actions/Post/CreatePosts.php
- [T07] [다중 목적지 생성과 승인 분기](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Actions/Post/CreatePosts.php)
[T09]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Support/PostApproval.php
- [T09] [서버 승인 규칙](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Support/PostApproval.php)
[T11]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Http/Middleware/Api/LoadWorkspaceFromToken.php
- [T11] [워크스페이스에 고정된 토큰](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Http/Middleware/Api/LoadWorkspaceFromToken.php)
[T12]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Mcp/Servers/TryPostServer.php
- [T12] [MCP 도구 등록](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Mcp/Servers/TryPostServer.php)
[T18]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Actions/Analytics/ReadPublicationAnalytics.php
- [T18] [저장된 분석 지표 조회](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Actions/Analytics/ReadPublicationAnalytics.php)
[T20]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Enums/SocialAccount/Platform.php
- [T20] [플랫폼과 분석 지원 구분](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Enums/SocialAccount/Platform.php)
[T21]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Actions/Ai/AssistPostContent.php
- [T21] [내장 글쓰기 AI 호출](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/app/Actions/Ai/AssistPostContent.php)
[T25]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/.github/workflows/tests.yml
- [T25] [테스트 CI](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/.github/workflows/tests.yml)
[T26]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/DEPLOYMENT.md
- [T26] [게시물 모델 변경 배포 절차](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/DEPLOYMENT.md)
[T27]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/ANALYTIC.md
- [T27] [과거 분석 기능 목록과 주의](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/ANALYTIC.md)
[T29]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/docs/superpowers/specs/2026-09-24-independent-social-posts-design.md
- [T29] [채널별 독립 게시물 설계](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/docs/superpowers/specs/2026-09-24-independent-social-posts-design.md)
[T33]: https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/LICENSE.md
- [T33] [루트 라이선스](https://github.com/trypostit/trypost/blob/b13cca4d834bc793ef895dba40f29ed050ec820d/LICENSE.md)
