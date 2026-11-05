# Oh My Codex 멀티 에이전트 개발 운영 가이드

> 목적: 단일 Codex CLI 환경에서 Oh My Codex(OMX)를 사용해 문제 발견부터 요구사항 정의, 병렬 개발, 독립 검증, Problem Owner 인수 검증까지 일관된 흐름으로 운영한다.
>
> 작성일: 2026-11-05  
> 기본 실행 환경: Codex CLI + Oh My Codex  
> 권장 팀 구성: Main Codex 1개 + Worker 5개

---

## 1. 전체 운영 원칙

1. **문제부터 정의한다.** Problem Owner가 요청한 해결책을 곧바로 기능으로 옮기지 말고, 문제의 원인·영향·현재 업무 방식·기대 효과를 확인한다.
2. **인터뷰 결과를 Source of Truth로 삼는다.** 확인되지 않은 값은 임의로 채우지 않고 `TBD` 또는 `확인 필요`로 남긴다.
3. **계획과 실행을 분리한다.** Deep Interview와 PRD/Ralplan 단계에서는 요구사항과 개발 계획을 확정하고, 승인 후 Team Mode로 구현한다.
4. **병렬화는 독립성이 있는 작업에만 적용한다.** 공통 API 계약, 데이터 모델, 공유 타입 등 선행 결정이 필요한 항목은 먼저 합의한다.
5. **Worker별 소유권을 명시한다.** 담당 업무, 수정 가능 파일, 금지 파일, 인터페이스, 완료 조건을 지정해 충돌을 줄인다.
6. **개발자의 완료 보고와 검증 통과를 구분한다.** 코드 리뷰, 자동 테스트, 통합 검증, 실제 사용자 인수 검증을 별도로 수행한다.

## 2. 전체 프로세스

```text
Problem Owner
    |
    v
$deep-interview
    |  문제·사용자·업무흐름·비즈니스 가치·제약사항 인터뷰
    v
Interview Result (Source of Truth)
    |
    v
$ralplan
    |  PRD + KPI + Acceptance Criteria + DoD + Test Spec
    |  Architecture + API/Data Contract + Task Breakdown
    |  Dependencies + 5 Worker Allocation Plan
    v
Human Review / Plan Approval
    |
    v
$team 5:executor
    |-- FE-1: 공통 UI / Layout / Design System
    |-- FE-2: 페이지 / 사용자 플로우 / API 연동
    |-- BE-1: 데이터 모델 / Repository / 도메인 로직
    |-- BE-2: API / Service / 인증 / 외부 연동
    `-- QA-1: 테스트 설계 / 테스트 코드 / 통합 검증
    |
    v
Main Codex Integration
    |
    v
$code-review
    |
    v
$ultraqa
    |
    v
Problem Owner Acceptance
    |
    v
KPI / Outcome Review
```

### 명령의 역할

| 명령 | 사용하는 시점 | 역할 |
|---|---|---|
| `$deep-interview` | 문제 정의 | Problem Owner를 인터뷰해 문제, 현재 프로세스, 사용자, 기대 결과, 제약을 파악 |
| `$ralplan` | 인터뷰 이후 | 요구사항을 PRD·기술 계획·작업 분해·의존성·검증 기준으로 구체화 |
| `$team 5:executor` | 승인된 계획 이후 | Codex Worker 5개를 실제로 실행하고 작업을 병렬 분배 |
| `$ultragoal` | 선택 사항 | 장기간 지속되는 목표/마일스톤을 추적해야 할 때 사용. 병렬 Worker 생성 명령을 대체하지 않음 |
| `$code-review` | 구현/통합 이후 | 코드 변경의 결함, 보안, 설계, 유지보수성 및 병합 위험 검토 |
| `$ultraqa` | 구현/통합 이후 | 테스트 실행, 기능 동작 검증, 실패 원인 분석, 수정 및 재검증 |

> `$ultragoal`과 `$team`은 서로 대체재가 아니다. 실제 병렬 코딩 Worker가 필요한 경우 Team Mode를 명시적으로 실행한다. 프로젝트가 짧거나 마일스톤 추적이 불필요하면 `$ultragoal`은 생략할 수 있다.

## 3. Problem Owner 인터뷰 범위

`$deep-interview`는 질문을 한 번에 나열하지 않고, 한 번에 핵심 질문 하나씩 대화형으로 진행한다. 답변이 모호하면 사례·수치·빈도를 추가로 질문한다.

필수 인터뷰 영역:

1. **문제 정의:** 문제, 발생 배경, 근본 원인, 빈도, 영향 범위, 해결하지 않았을 때의 결과
2. **사용자 및 이해관계자:** 실제 사용자, 구매자/의사결정자, 운영 담당자, 승인자, 권한 관계
3. **현재 업무 프로세스:** As-Is 흐름, 수작업, 병목, 기존 도구, 예외 및 우회 절차
4. **비즈니스 요구사항:** 사업 목표, 기대 가치, 비용·시간 절감, 수익 영향, 우선순위
5. **기능 요구사항:** 사용 시나리오, 입력·처리·출력, 필수/선택 기능, 예외 처리
6. **데이터 요구사항:** 출처, 소유권, 스키마, 품질, 확보 가능성, 갱신 주기, 보관·삭제, 접근 권한
7. **UI/UX 요구사항:** 사용자 여정, 화면, 정보 구조, 접근성, 반응형, 오류·빈 상태
8. **보안·개인정보:** 인증, 인가, 민감정보, 암호화, 감사 로그, 외부 전송, 규제·컴플라이언스
9. **시스템·연동:** 기존 시스템, API, 외부 서비스, 기술/인프라 제약
10. **비기능 요구사항:** 응답시간, 처리량, 동시 사용자, 가용성, 복구, 확장성, 호환성
11. **AI·자동화(해당 시):** AI 적용 근거, 모델 선택, 정확도, 근거 제시, 환각 허용 범위, Human-in-the-loop, 실패 대체 경로, 지연·비용, 평가 데이터
12. **운영·성공 기준:** 운영 책임, 모니터링, 장애 대응, 배포, 예산, 일정, KPI, 성공 판정 및 출시 후 측정

인터뷰 종료 시 확인할 결과:

- Problem Statement 및 문제의 근본 원인
- Problem Owner / Stakeholder / 사용자 페르소나
- As-Is / To-Be 업무 흐름
- 기대 가치와 우선순위
- 확인된 사실, 가정, 미확정 사항의 구분
- 제약·리스크·추가 확인 질문

**인터뷰 원칙:** Problem Owner가 “AI 챗봇이 필요하다”처럼 해결책을 제안하더라도 바로 요구사항으로 확정하지 않는다. 왜 필요한지, 현재 어떤 업무가 막히는지, 어떤 결과를 바라는지 먼저 확인한다.

## 4. Ralplan 산출물: PRD와 실행 가능한 개발 계획

Ralplan에는 Deep Interview 결과 파일을 전달한다. 인터뷰를 다시 진행하거나 확인되지 않은 정보를 임의로 확정하지 않는다.

### 4.1 PRD

- Product Vision 및 문제 정의
- 목표 사용자와 주요 사용자 시나리오
- Business Requirements
- Functional Requirements
- Non-functional Requirements
- MVP / Out of Scope
- 기능 우선순위: P0 / P1 / P2
- 가정, 제약, 리스크 및 미확정 사항

### 4.2 KPI Specification

각 KPI에 다음 속성을 정의한다.

| 속성 | 설명 |
|---|---|
| KPI 이름 | 측정할 성과의 명확한 이름 |
| 정의/공식 | 분자·분모 및 계산식 포함 |
| Baseline | 현재 기준값. 근거가 없으면 TBD |
| Target | 목표값. 인터뷰 근거 또는 Problem Owner 승인 필요 |
| 측정 기간 | 일/주/월/분기 등 |
| 데이터 출처 | 이벤트, DB, 로그, 설문, 운영 자료 등 |
| 측정 주체 | 책임자 및 측정 방식 |
| 검증 방법 | 재현 가능한 검증 또는 리포트 |

KPI는 다음처럼 분리한다.

- **Business KPI:** 비용 절감, 업무시간 단축, 매출/전환, 생산성, 오류 비용 등 실제 문제 해결 성과
- **Product/Operational KPI:** 활성 사용자, 기능 사용률, 완료율, 응답 시간, 오류율, 가용성 등 제품 운영 성과
- **AI KPI(해당 시):** 평가 데이터셋 정확도, 근거 일치율, 답변 불가 처리율, 사람의 수정/승인 비율, 추론 비용, 지연시간 등

목표 예시를 제시할 때는 반드시 `예시/제안값`으로 표기하고 확정 KPI와 혼동하지 않는다.

### 4.3 Acceptance Criteria

기능별로 검증 가능한 조건을 작성한다. 가능하면 Given / When / Then을 사용한다.

```gherkin
Given [사전 조건]
When [사용자 또는 시스템의 행동]
Then [관찰 가능하고 검증 가능한 기대 결과]
```

각 기능은 정상 흐름, 잘못된 입력, 경계값, 권한 부족, 외부 서비스 실패 등 중요한 예외 조건을 포함한다.

### 4.4 Definition of Done (DoD)

**태스크 단위 DoD**

- 승인된 Acceptance Criteria 충족
- 코드 및 설정 변경 완료
- 단위 테스트 작성/통과
- API 또는 컴포넌트 계약 준수
- 에러 처리 및 로그 반영
- 문서/타입/스키마 갱신
- 담당 Worker의 결과 보고 완료

**프로젝트 단위 DoD**

- P0 기능 구현 및 통합 완료
- 필수 단위·통합·E2E 테스트 통과
- 중요 보안/권한 시나리오 검증
- 코드 리뷰의 Blocker/Critical 해결
- 회귀 테스트 통과
- 배포/실행 절차 검증
- 운영·사용자 문서 작성
- Problem Owner 인수 기준 충족 또는 미충족 항목 명시

### 4.5 테스트 명세

- 기능별 테스트 케이스와 테스트 데이터
- 정상/오류/경계값 시나리오
- Frontend 컴포넌트/상태 테스트
- Backend API/도메인 테스트
- 통합 및 E2E 사용자 여정
- 권한/보안 테스트
- 성능/안정성 테스트(요구되는 경우)
- AI 평가 시나리오(해당하는 경우)
- KPI 측정 및 검증 방법

### 4.6 기술 계획 및 태스크 분해

- 시스템 아키텍처와 모듈 경계
- DB 모델 및 데이터 흐름
- API 계약(Request/Response, Error Schema)
- 공통 타입/DTO
- 인증·권한 정책
- 외부 시스템 계약
- 환경 변수와 로컬 실행 방식
- Worker별 Task ID, 파일 소유권, 완료 조건
- 작업 의존성 그래프 및 병렬 실행 가능 구간
- 통합·배포·롤백 계획

## 5. Team Mode: 5 Worker 구성

Main Codex는 별도 Worker로 세지 않는다. Main Codex가 PM, Tech Lead, Orchestrator, 통합 담당을 맡고 Worker 5개가 구현 및 QA를 수행한다.

| Worker | 역할 | 주요 책임 | 예시 소유 범위 |
|---|---|---|---|
| FE-1 | Frontend Foundation | 디자인 시스템, 공통 컴포넌트, Layout, 공통 상태/테마 | `components/`, `layouts/`, 공통 스타일 |
| FE-2 | Frontend Features | 페이지, 사용자 플로우, 화면별 기능, API 연동 | `pages/`, `features/` |
| BE-1 | Backend Domain/Data | DB 모델, Repository, 도메인 규칙, 데이터 검증 | `models/`, `repositories/`, 도메인 모듈 |
| BE-2 | Backend API/Integration | API, Service, 인증/인가, 외부 연동, 에러 응답 | `api/`, `services/`, integration 모듈 |
| QA-1 | Quality Engineering | 테스트 전략, 테스트 데이터, 단위/통합/E2E, 결함 재현 및 회귀 | `tests/`, 테스트 설정 및 QA 문서 |

> 위 디렉터리는 예시다. 실제 저장소 구조를 확인한 뒤 Ralplan이 수정 파일 범위를 지정해야 한다. 각 Worker가 동일 파일을 동시에 편집하지 않도록 한다.

### Main Codex 책임

- PRD와 승인된 계획을 Source of Truth로 유지
- 공통 API/데이터 계약과 변경 승인
- Worker별 구체적인 Task 할당 및 우선순위 관리
- Worker 간 의존성 및 Blocker 조정
- 공통 파일의 단일 소유자 지정
- Worktree/Branch 및 통합 순서 관리
- 결과물과 테스트 증거 확인
- 충돌 해결 및 최종 통합
- 미완료/미검증 기능을 완료로 보고하지 않음

### Worker별 필수 작업 지시 정보

모든 Task는 다음 정보를 포함해야 한다.

1. Task ID 및 목표
2. 구체적인 작업 범위
3. 담당 Worker
4. 수정 허용 파일/디렉터리
5. 수정 금지 영역
6. 선행 작업 및 의존성
7. 다른 Worker와 합의할 인터페이스
8. Acceptance Criteria
9. 테스트 명령 및 기대 결과
10. 완료 보고 형식

### 병렬 실행 전 공통 계약

Main Codex가 가능하면 Team 실행 전에 다음을 확정한다.

- API 명세 및 Request/Response 스키마
- DB 모델과 데이터 소유권
- 공유 타입/DTO
- 인증·인가 방식
- 공통 에러 포맷
- 프로젝트 실행/테스트 명령
- 공통 설정 파일의 담당자

계약이 확정되지 않은 영역은 임시 Mock 또는 명시된 계약 초안을 사용할 수 있지만, 가정과 변경 책임자를 기록한다.

## 6. QA는 개발과 동시에 시작한다

QA-1은 개발이 끝날 때까지 대기하지 않는다.

**초기/병렬 단계**

- PRD와 Acceptance Criteria에서 테스트 시나리오 도출
- FE/BE 테스트 전략과 테스트 데이터 준비
- API 계약 기반 테스트 작성
- Mock 및 Fixture 준비
- 경계값, 권한, 실패 시나리오 설계
- 테스트 실행 환경과 자동화 설정 준비

**통합 단계**

- 실제 API와 화면 통합 테스트
- 주요 사용자 여정 E2E 실행
- FE/BE 간 계약 불일치 확인
- 재현 가능한 Bug Report 작성
- 수정 결과 재검증 및 회귀 테스트

QA의 독립성은 중요하다. QA가 직접 개발 코드를 대량 수정하기보다는 결함을 재현하고 담당 Worker에게 수정 요청을 배정한다. 작은 테스트 설정 수정 등은 승인된 범위에서 처리할 수 있다.

## 7. Code Review와 UltraQA의 구분

| 항목 | `$code-review` | `$ultraqa` |
|---|---|---|
| 핵심 질문 | 코드와 설계가 안전하고 올바른가? | 제품 기능이 실제로 동작하는가? |
| 중점 | 결함, 보안, 구조, 유지보수성, 회귀 위험 | 테스트 실행, 정상/오류 흐름, 통합, E2E, 재검증 |
| 산출물 | 발견 사항, 심각도, 파일/라인, 수정 권고 | 테스트 결과, 실패 로그, 재현 절차, 수정/재테스트 결과 |
| 시점 | 주요 구현/통합 후 | 실행 가능한 통합 환경 확보 후 |

예시:

- 인증 코드에서 권한 검사가 누락되었는지: Code Review
- 로그인 성공/실패 및 토큰 만료 후 동작: UltraQA
- 입력값을 검증하지 않아 인젝션 위험이 있는지: Code Review
- 잘못된 입력으로 API가 기대한 오류를 반환하는지: UltraQA

두 검증은 중복이 아니라 상호 보완이다.

## 8. 권장 실행 프롬프트

### 8.1 Deep Interview 종료 후 Ralplan 요청

```text
$ralplan .omx/specs/deep-interview-{slug}.md

첨부한 Deep Interview 결과를 비즈니스 요구사항의 Source of Truth로 사용해라.
인터뷰를 반복하지 말고, 확인된 사실과 미확정 사항을 구분해 PRD,
KPI Specification, Acceptance Criteria, DoD, Test Specification을 작성해라.

이번 프로젝트는 이후 $team 5:executor로 실행할 계획이다.
Main Codex가 PM/Tech Lead를 맡고, Worker는 FE-1, FE-2, BE-1,
BE-2, QA-1로 운영한다.
따라서 계획 단계에서 5개 Worker의 Task 분해, 파일 소유권,
API/데이터 계약, 의존성 그래프, 병렬 실행 구간, 통합 순서를 설계해라.

QA는 개발과 병렬로 테스트 설계를 시작하고, 통합 이후 실제 통합/E2E
테스트를 수행하도록 계획해라.
계획과 문서만 작성하고 아직 코드를 수정하지 마라.
```

### 8.2 승인 후 Team 실행 요청

```text
$team 5:executor

승인된 Ralplan을 기준으로 Codex Worker 5개를 실행해라.

- FE-1: 공통 컴포넌트, Layout, 디자인 시스템
- FE-2: 페이지, 사용자 플로우, 화면 기능 및 API 연동
- BE-1: DB 모델, Repository, 핵심 도메인 로직
- BE-2: API, Service, 인증/인가 및 외부 연동
- QA-1: 테스트 설계, 테스트 코드, 테스트 데이터 및 검증

각 Worker는 할당된 Task, 파일 소유권, Acceptance Criteria를 준수해라.
공통 계약을 임의로 변경하지 말고 변경이 필요하면 Main Codex에 먼저 보고해라.
독립 작업은 병렬로 수행하고 선행 작업이 필요한 일은 의존성에 맞춰 진행해라.
QA는 초기에 테스트 준비를 시작하고 통합 가능한 시점에 통합 테스트를 실행해라.
각 Worker는 변경 파일, 실행한 테스트, 결과, 미해결 이슈를 보고해라.
Main Codex는 작업 결과를 확인하고 통합 및 최종 검증을 담당해라.
```

> `5:executor`는 Worker 수를 요청하는 표현이지, FE 2명·BE 2명·QA 1명을 보장하는 전용 역할 문법은 아니다. 실행 직후 실제 Worker 생성 수, 역할, 태스크 할당 및 작업 공간을 확인한다. 필요한 역할이 제대로 배정되지 않았다면 Main Codex가 작업 배정을 명시적으로 수정한다.

### 8.3 최종 검증 요청

```text
$code-review

통합된 전체 변경사항을 검토해라. 보안, 권한, 데이터 무결성,
API 계약, 오류 처리, 아키텍처, 유지보수성 및 회귀 위험을 확인해라.
발견 사항은 심각도와 재현/수정 근거를 포함해 보고해라.
```

```text
$ultraqa

승인된 Acceptance Criteria를 기준으로 전체 기능을 검증해라.
단위/통합/E2E 및 주요 예외 시나리오를 실행하고 실패 로그와 재현 절차를 남겨라.
문제가 발견되면 담당 Worker 또는 Main Codex에 수정을 요청하고 재검증해라.
통과하지 않은 항목은 성공으로 표시하지 마라.
```

## 9. Problem Owner Acceptance

자동화된 테스트 통과는 실제 문제 해결의 증거와 동일하지 않다. 최종 단계에서 Problem Owner가 실제 업무 시나리오로 결과를 확인해야 한다.

- 최초 Problem Statement가 해결되었는가?
- 핵심 사용자 시나리오가 실제 업무에서 작동하는가?
- KPI를 측정할 데이터가 수집되는가?
- 목표 KPI가 충족되었는가, 아니면 기준값/기간이 부족한가?
- 운영 담당자가 시스템을 사용할 수 있는가?
- 남은 리스크와 미완료 범위가 명시되었는가?

인수 결과는 `Accepted`, `Accepted with Follow-ups`, `Rejected`로 기록하고, 미충족 Acceptance Criteria와 후속 작업을 연결한다.

## 10. 최종 권장 사항

- 일반적인 단일 프로젝트: `$deep-interview` → `$ralplan` → `$team 5:executor` → `$code-review` → `$ultraqa` → Problem Owner Acceptance
- 장기 프로젝트/여러 마일스톤: 위 흐름에 `$ultragoal`을 추가해 장기 목표와 진행 체크포인트를 관리
- Worker 수가 많다고 무조건 빠른 것은 아니다. 작업이 충분히 독립적이고 명확한 인터페이스가 있을 때 5개 병렬 구성을 사용한다.
- FE/BE 작업의 경계가 모호하거나 공유 파일을 자주 바꿔야 한다면 태스크 분해를 먼저 수정한다.
- QA의 테스트 결과, Code Review 결과, Problem Owner의 인수 결과를 별도 증거로 남긴다.
- 실제 실행 전에 사용 중인 OMX 버전의 문서에서 명령 및 옵션을 확인한다.

---

## 참고 링크

- Oh My Codex GitHub: https://github.com/Yeachan-Heo/oh-my-codex
- Codex CLI: https://github.com/openai/codex
