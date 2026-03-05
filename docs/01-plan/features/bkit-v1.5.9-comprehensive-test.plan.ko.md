# bkit v1.5.9 종합 테스트 계획서

> **요약**: bkit v1.5.9 종합 테스트 -- 유닛, 통합, E2E, UX, Hook, 회귀 및 14개 추가 카테고리를 아우르는 전체 커버리지 테스트 계획. 전체 199개 export, 27개 스킬, 16개 에이전트, 10개 훅 이벤트, 16개 템플릿, 45개 스크립트, 8개 언어 i18n 포함
>
> **프로젝트**: bkit-claude-code
> **버전**: 1.5.9
> **작성자**: CTO 팀 (qa-strategist, code-analyzer, gap-detector, product-manager, qa-monitor)
> **작성일**: 2026-03-05
> **상태**: 초안
> **이전 테스트**: v1.5.8 (920 TC 계획, 100%)
> **환경**: CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1, Claude Code v2.1.69+
> **브랜치**: feature/bkit-v1.5.9-executive-summary
> **총 TC**: 594 (v1.5.9 집중 161 + 전체 커버리지 309 + 철학 & 컨텍스트 엔지니어링 124)

---

## 요약 (Executive Summary)

| 관점 | 내용 |
|------|------|
| **문제** | v1.5.9에서 Executive Summary 모듈, AskUserQuestion Preview UX, ENH-74~81 훅 개선이 신규 도입됨. 추가로 모든 bkit 하위 시스템(199 export, 27 스킬, 16 에이전트, 45 스크립트, 16 템플릿, 4 출력 스타일, 8개 언어)에 대한 종합 전체 커버리지 QA 필요. 이전 계획에는 철학 및 컨텍스트 엔지니어링 검증이 부재 |
| **해결** | 5명의 전문 에이전트가 31개 테스트 카테고리, 594개 테스트 케이스를 다각도로 검증: v1.5.9 집중(161 TC) + 전체 커버리지(309 TC) + 철학 & 컨텍스트 엔지니어링(124 TC) |
| **기능/UX 효과** | 모든 공개 API, 모든 에이전트/스킬/템플릿/스크립트 존재 여부, 8개 언어 i18n 정확도, 3단계 PDCA 워크플로 E2E, 메모리 시스템 격리, 3가지 핵심 철학, 8개 컨텍스트 엔지니어링 FR, 6계층 훅 시스템, 4가지 동적 주입 패턴, AI 네이티브 원칙을 검증 |
| **핵심 가치** | 100% 기능 인벤토리 커버리지와 철학 수준의 검증을 달성하여, bkit이 올바르게 작동할 뿐 아니라 핵심 사명(자동화 우선, 추측 금지, 문서=코드)을 199개 전체 export에 걸쳐 구현하고 있음을 보장 |

---

## 1. 배경

### 1.1 테스트 필요성

bkit v1.5.9는 세 가지 주요 변경 범주를 도입함: (1) 3개 함수를 export하는 새로운 Executive Summary 모듈, (2) PDCA 단계 전환 시 풍부한 Markdown 미리보기를 제공하는 AskUserQuestion Preview UX, (3) agent_id/agent_type 일급 추출 및 continue:false 팀원 수명주기 제어를 포함한 ENH-74~81 CC v2.1.69 호환성 개선. 이러한 변경은 103개 파일에 +3,187/-5,967줄 변경량으로 종합 검증이 필요함.

### 1.2 이전 테스트 결과 (v1.5.8)

| 지표 | 값 |
|------|:--:|
| 총 TC | 920 (계획) |
| 통과율 | 100% |
| common.js export 수 | 184 |
| 스킬 | 27 |
| 에이전트 | 16 |
| 훅 이벤트 | 10 (13개 항목) |

### 1.3 v1.5.9 구성요소 인벤토리

| 구성요소 | 수량 | 위치 | v1.5.8 대비 변경 |
|----------|:----:|------|:----------------:|
| 스킬 | 27 (코어 22 + bkend 5) | `skills/*/SKILL.md` | 0 (2개 수정: pdca, bkit-rules) |
| 에이전트 | 16 | `agents/*.md` | 0 |
| 훅 이벤트 | 10 | `hooks/hooks.json` | 0 (InstructionsLoaded 제거) |
| 훅 항목 | 13 | `hooks/hooks.json` | 0 |
| 스크립트 | 45 | `scripts/*.js` | 0 (7개 수정) |
| 라이브러리 모듈 | 39 | `lib/**/*.js` | **+1** (lib/pdca/executive-summary.js) |
| 라이브러리 export (common.js) | **199** | `lib/` -> `lib/common.js` | **+15** (184에서) |
| 템플릿 | 16 | `templates/` | 0 (3개 수정: plan, plan-plus, report) |
| 출력 스타일 | 4 | `output-styles/` | 0 |
| 설정 파일 | 2 | `plugin.json`, `bkit.config.json` | 버전 변경만 |

### 1.4 v1.5.9 주요 변경 파일

| # | 파일 | 변경 유형 | 변경 줄 수 | 카테고리 |
|---|------|:---------:|:----------:|----------|
| 1 | `lib/pdca/executive-summary.js` | **신규** | +197 | Executive Summary 모듈 |
| 2 | `lib/pdca/index.js` | 수정 | +5 | executive-summary 3개 신규 export |
| 3 | `lib/common.js` | 수정 | +3 | executive-summary 3개 재export |
| 4 | `lib/pdca/automation.js` | 수정 | +191 | buildNextActionQuestion(), formatAskUserQuestion 미리보기 |
| 5 | `scripts/gap-detector-stop.js` | 수정 | ~+200 | 4개 matchRate 구간별 미리보기 |
| 6 | `scripts/pdca-task-completed.js` | 수정 | ~+40 | ENH-74/75, buildNextActionQuestion |
| 7 | `scripts/team-idle-handler.js` | 수정 | ~+10 | ENH-74/75 |
| 8 | `scripts/subagent-start-handler.js` | 수정 | ~+5 | ENH-74 agent_id |
| 9 | `scripts/subagent-stop-handler.js` | 수정 | ~+5 | ENH-74 agent_id/agent_type |
| 10 | `scripts/unified-stop.js` | 수정 | ~+10 | ENH-74 agent_id |
| 11 | `templates/plan.template.md` | 수정 | +12 | Executive Summary 섹션 |
| 12 | `templates/plan-plus.template.md` | 수정 | +12 | Executive Summary 섹션 |
| 13 | `templates/report.template.md` | 수정 | +10 | 전달 가치(Value Delivered) 테이블 |
| 14 | `skills/pdca/SKILL.md` | 수정 | +5 | Executive Summary 생성 지침 |
| 15 | `skills/bkit-rules/SKILL.md` | 수정 | +5 | v1.5.9 규칙 |
| 16 | `hooks/hooks.json` | 수정 | -6 | InstructionsLoaded 훅 제거 |

### 1.5 CTO 팀 구성

| 역할 | 에이전트 | 모델 | 담당 |
|------|----------|:----:|------|
| **CTO 리드** | cto-lead | opus | 전체 조율, 품질 게이트 |
| 코드 분석가 | code-analyzer | opus | 유닛 테스트: 함수 수준 I/O, 엣지 케이스 |
| QA 전략가 | qa-strategist | sonnet | 테스트 전략, 매트릭스 설계, 통합 |
| 갭 감지기 | gap-detector | opus | E2E 테스트: 설계-구현 갭 커버리지 |
| 프로덕트 매니저 | product-manager | sonnet | UX 테스트: AskUserQuestion, Executive Summary 출력 |
| QA 모니터 | qa-monitor | sonnet | 훅 테스트: 스크립트 실행, JSON 출력 |

---

## 2. 테스트 전략 개요

### 2.1 테스트 접근 방식

| 카테고리 | 초점 | 담당 에이전트 | TC 범위 |
|----------|------|--------------|:-------:|
| 유닛 테스트 | 함수 수준 입출력 검증 | code-analyzer | TC-U001~U044 |
| 통합 테스트 | 모듈 간 연결, export 체인 | gap-detector | TC-I001~I020 |
| E2E 테스트 | 전체 PDCA 워크플로 시나리오 | gap-detector | TC-E001~E015 |
| UX 테스트 | AskUserQuestion 미리보기, Executive Summary 표시 | product-manager | TC-X001~X020 |
| 훅 테스트 | 스크립트 실행, JSON 출력, 훅 이벤트 흐름 | qa-monitor | TC-H001~H025 |
| 회귀 테스트 | v1.5.8 기준 기능 보존 | qa-strategist | TC-R001~R015 |
| 엣지 케이스 | 오류 처리, 경계 조건, 성능 | code-analyzer | TC-EC001~EC010 |
| 성능 테스트 | 대규모 프로젝트, 다중 기능 시나리오 | qa-monitor | TC-P001~P005 |
| 핵심 철학 | 3가지 철학 (자동화 우선, 추측 금지, 문서=코드) | product-manager | TC-PH001~PH017 |
| 컨텍스트 엔지니어링 | 8개 FR (계층, import, fork, 훅, 권한, 태스크, 압축, 메모리) | code-analyzer | TC-CE001~CE031 |
| 6계층 훅 시스템 | 훅 계층 순서 및 통합 | qa-monitor | TC-HL001~HL007 |
| 동적 컨텍스트 주입 | 4가지 패턴 (태스크 크기, 의도, 모호성, 일치율) | product-manager | TC-DI001~DI018 |
| AI 네이티브 원칙 | 핵심 역량, 레벨 여정, 언어 티어, 에이전트 모델 | gap-detector | TC-AN001~AN018 |
| PDCA 방법론 | 파이프라인 관계, 자동 적용, Check-Act 루프, 아카이브 | gap-detector | TC-PM001~PM015 |
| 오케스트레이션 패턴 | leader/council/swarm/watchdog 패턴 선택 | qa-strategist | TC-OP001~OP008 |
| 출력 스타일 컨텍스트 | 레벨 맞춤 출력 스타일 주입 | qa-monitor | TC-OC001~OC005 |
| 응답 리포트 규칙 | 기능 사용 리포트 구조 및 정확성 | qa-monitor | TC-RR001~RR005 |

### 2.2 TC 요약

| 우선순위 | TC 수 (v1.5.9) | TC 수 (전체 커버리지) | TC 수 (철학 & CE) | 합계 |
|:--------:|:--------------:|:--------------------:|:-----------------:|:----:|
| P0 (필수) | 113 | 155 | 69 | 337 |
| P1 (권장) | 37 | 130 | 48 | 215 |
| P2 (선택) | 11 | 24 | 7 | 42 |
| **총합** | **161** | **309** | **124** | **594** |

### 2.3 전체 커버리지 카테고리 (섹션 14-27) + 철학 & CE (섹션 29-37)

| 카테고리 | 접두사 | TC 수 | 담당 에이전트 |
|----------|--------|:-----:|--------------|
| 코어 시스템: 플랫폼 & 캐시 | TC-CS | 16 | code-analyzer |
| 코어 시스템: I/O, 디버그, 설정, 파일, 경로 | TC-CS | 17 | code-analyzer |
| 인텐트 시스템: 언어 & 트리거 | TC-IN | 26 | product-manager |
| 태스크 시스템 | TC-TS | 21 | code-analyzer |
| 팀 시스템 | TC-TM | 27 | code-analyzer |
| PDCA 전체 시스템 | TC-PD | 22 | code-analyzer |
| 스킬 시스템 | TC-SK | 27 | gap-detector |
| 에이전트 시스템 | TC-AG | 19 | gap-detector |
| 설정 시스템 | TC-CF | 12 | gap-detector |
| 템플릿 시스템 | TC-TP | 26 | gap-detector |
| 출력 스타일 | TC-OS | 6 | qa-monitor |
| PDCA 레벨 E2E 워크플로 | TC-LV | 12 | product-manager |
| i18n (8개 언어) | TC-L10N | 16 | product-manager |
| 메모리 시스템 | TC-MEM | 11 | product-manager |
| 스크립트 시스템 | TC-SC | 32 | qa-monitor |

**철학 & 컨텍스트 엔지니어링 카테고리 (섹션 29-37)**:

| 카테고리 | 접두사 | TC 수 | 담당 에이전트 |
|----------|--------|:-----:|--------------|
| 핵심 철학 (3가지 철학) | TC-PH | 17 | product-manager |
| 컨텍스트 엔지니어링 (8개 FR) | TC-CE | 31 | code-analyzer |
| 6계층 훅 시스템 | TC-HL | 7 | qa-monitor |
| 동적 컨텍스트 주입 (4가지 패턴) | TC-DI | 18 | product-manager |
| AI 네이티브 원칙 | TC-AN | 18 | gap-detector |
| PDCA 방법론 | TC-PM | 15 | gap-detector |
| 오케스트레이션 패턴 | TC-OP | 8 | qa-strategist |
| 출력 스타일 컨텍스트 계층 | TC-OC | 5 | qa-monitor |
| 응답 리포트 규칙 | TC-RR | 5 | qa-monitor |

---

## 3. 테스트 범위 매트릭스

### 3.1 v1.5.9 신규 기능

| 기능 | 변경 파일 | TC 수 | 우선순위 |
|------|:---------:|:-----:|:--------:|
| Executive Summary 모듈 | executive-summary.js, index.js, common.js | 16 | P0 |
| AskUserQuestion Preview UX | automation.js, gap-detector-stop.js, pdca-task-completed.js | 20 | P0 |
| ENH-74: agent_id/agent_type | 5개 스크립트 | 12 | P0 |
| ENH-75: continue:false | pdca-task-completed.js, team-idle-handler.js | 8 | P0 |
| 템플릿 업데이트 | plan, plan-plus, report 템플릿 | 6 | P1 |
| 스킬 업데이트 | pdca, bkit-rules SKILL.md | 4 | P1 |
| 훅 설정 | hooks.json (InstructionsLoaded 제거) | 3 | P0 |

### 3.2 기존 기능 (전체 커버리지 -- 섹션 14-27)

| 기능 | 구성요소 수 | TC 수 | 섹션 | 우선순위 |
|------|:----------:|:-----:|:----:|:--------:|
| 코어 시스템 (플랫폼, 캐시, I/O, 디버그, 설정, 파일, 경로) | 49 export | 46 | 14 | P0/P1 |
| 인텐트 시스템 (언어, 트리거, 모호성) | 19 export | 32 | 15 | P0/P1 |
| 태스크 시스템 (분류, 컨텍스트, 생성, 추적) | 26 export | 21 | 16 | P0/P1 |
| 팀 시스템 (조율, 전략, 오케스트레이터, 통신, 큐, CTO, 상태) | 40 export | 27 | 17 | P0/P1 |
| PDCA 전체 시스템 (티어, 레벨, 단계, 상태) | 54 export | 22 | 18 | P0/P1 |
| 스킬 시스템 | 27개 스킬 | 27 | 19 | P0/P1 |
| 에이전트 시스템 | 16개 에이전트 | 19 | 20 | P0 |
| 설정 시스템 | 3개 설정 파일 | 12 | 21 | P0/P1 |
| 템플릿 시스템 | 26개 템플릿 | 26 | 22 | P0/P1/P2 |
| 출력 스타일 | 4개 스타일 | 6 | 23 | P0/P1 |
| PDCA 레벨 E2E | 3개 레벨 | 12 | 24 | P0/P1/P2 |
| i18n (8개 언어) | 8개 언어 | 16 | 25 | P0/P1 |
| 메모리 시스템 | 3개 시스템 | 11 | 26 | P0/P1 |
| 스크립트 시스템 | 45개 스크립트 | 32 | 27 | P0/P1/P2 |
| **소계** | | **309** | 14-27 | |

### 3.3 회귀 (섹션 9)

| 기능 | 구성요소 수 | TC 수 | 우선순위 |
|------|:----------:|:-----:|:--------:|
| PDCA 전체 사이클 | 6개 단계 | 10 | P0 |
| common.js 브릿지 | 199 export | 5 | P0 |
| 훅 시스템 | 10개 이벤트 | 5 | P1/P2 |

---

## 4. 유닛 테스트 케이스 (code-analyzer)

### 4.1 TC-U: Executive Summary 모듈 (executive-summary.js)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-U001 | ExecSummary | generateExecutiveSummary가 올바른 구조를 반환 | `('my-feature', 'plan')` | type='executive-summary', feature, phase, summary(null 4개), nextActions(3개), metadata를 포함한 객체 | P0 | -- |
| TC-U002 | ExecSummary | plan-plus 단계에서 generateExecutiveSummary | `('feat', 'plan-plus')` | nextActions: ['설계 시작', '계획 수정', '팀 리뷰 요청'] | P0 | -- |
| TC-U003 | ExecSummary | report 단계에서 generateExecutiveSummary | `('feat', 'report')` | nextActions: ['아카이브', '추가 개선', '다음 기능'] | P0 | -- |
| TC-U004 | ExecSummary | check 단계(>=90)에서 generateExecutiveSummary | `('feat', 'check', {matchRate:95})` | nextActions[0].label = '보고서 생성' | P0 | -- |
| TC-U005 | ExecSummary | check 단계(<90)에서 generateExecutiveSummary | `('feat', 'check', {matchRate:75})` | nextActions[0].label = '자동 개선' | P0 | -- |
| TC-U006 | ExecSummary | 알 수 없는 단계는 plan으로 폴백 | `('feat', 'unknown')` | nextActions가 plan 세트(3개)와 일치 | P0 | -- |
| TC-U007 | ExecSummary | metadata에 generatedAt ISO 문자열 포함 | `('feat', 'plan')` | metadata.generatedAt가 ISO 8601 패턴과 일치 | P0 | -- |
| TC-U008 | ExecSummary | context.matchRate가 featureData를 오버라이드 | `('feat', 'check', {matchRate:50})` | metadata.matchRate === 50 | P0 | -- |
| TC-U009 | ExecSummary | formatExecutiveSummary null 입력 시 빈 문자열 반환 | `(null)` | '' | P0 | -- |
| TC-U010 | ExecSummary | 전체 형식에 4가지 관점 모두 포함 | 유효한 summary 객체 | 출력에 [문제], [해결], [기능 & UX 효과], [핵심 가치] 포함 | P0 | -- |
| TC-U011 | ExecSummary | 간략 형식이 더 짧음 | 유효한 summary, 'compact' | '-- EXEC SUMMARY:' 헤더 포함, [문제] 괄호 없음 | P0 | -- |
| TC-U012 | ExecSummary | 액션이 있을 때 다음 액션 섹션 포함 | nextActions가 있는 summary | '다음 액션'과 번호 항목 포함 | P0 | -- |
| TC-U013 | ExecSummary | null summary 필드에 '-' 표시 | problem/solution이 null인 summary | 각 줄에 '-' 플레이스홀더 표시 | P0 | -- |
| TC-U014 | ExecSummary | generateBatchSummary 빈 배열에 null 반환 | `([])` | null | P0 | -- |
| TC-U015 | ExecSummary | generateBatchSummary 비배열에 null 반환 | `('string')` | null | P0 | -- |
| TC-U016 | ExecSummary | generateBatchSummary가 features 배열이 있는 batch-summary 반환 | `(['feat-a', 'feat-b'])` | type='batch-summary', features.length===2, generatedAt 존재 | P0 | -- |

### 4.2 TC-U: AskUserQuestion 미리보기 (automation.js)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-U017 | 미리보기 | formatAskUserQuestion이 preview 필드를 전달 | preview가 있는 옵션 | result.questions[0].options[0].preview 존재 | P0 | -- |
| TC-U018 | 미리보기 | preview 미제공 시 생략 | preview 없는 옵션 | result.questions[0].options[0]에 'preview' 키 없음 | P0 | -- |
| TC-U019 | 미리보기 | buildNextActionQuestion plan-plus에 3개 옵션 | `('plan-plus', 'feat')` | result.options.length === 3 | P0 | -- |
| TC-U020 | 미리보기 | plan-plus 옵션에 preview 문자열 존재 | `('plan-plus', 'feat')` | 3개 옵션 모두 비어있지 않은 preview 문자열 보유 | P0 | -- |
| TC-U021 | 미리보기 | plan에 preview가 있는 3개 옵션 | `('plan', 'feat')` | 3개 옵션, 모두 preview 포함 | P0 | -- |
| TC-U022 | 미리보기 | report에서 질문에 matchRate 포함 | `('report', 'feat', {matchRate:95})` | 질문에 '95%' 포함 | P0 | -- |
| TC-U023 | 미리보기 | report preview에 기능명 포함 | `('report', 'my-feat')` | 최소 하나의 preview에 'my-feat' 포함 | P0 | -- |
| TC-U024 | 미리보기 | 알 수 없는 단계는 plan으로 폴백 | `('unknown', 'feat')` | plan 응답과 동일한 구조 | P0 | -- |
| TC-U025 | 미리보기 | report preview에 iterCount 포함 | `('report', 'feat', {matchRate:90, iterCount:3})` | preview에 iterCount '3' 포함 | P0 | -- |
| TC-U026 | 미리보기 | 옵션 미제공 시 기본 옵션 | `({question:'test'})` | 2개 기본 옵션: 계속, 건너뛰기 | P1 | -- |

### 4.3 TC-U: detectPdcaFromTaskSubject (automation.js)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-U027 | 감지 | [Plan] 접두사 감지 | `'[Plan] user-auth'` | {phase:'plan', feature:'user-auth'} | P0 | -- |
| TC-U028 | 감지 | [Design] 접두사 감지 | `'[Design] user-auth'` | {phase:'design', feature:'user-auth'} | P0 | -- |
| TC-U029 | 감지 | [Act-3] 접두사 감지 | `'[Act-3] user-auth'` | {phase:'act', feature:'user-auth'} | P0 | -- |
| TC-U030 | 감지 | [Report] 접두사 감지 | `'[Report] user-auth'` | {phase:'report', feature:'user-auth'} | P0 | -- |
| TC-U031 | 감지 | null 입력 처리 | `null` | null | P0 | -- |
| TC-U032 | 감지 | 비PDCA 주제 처리 | `'Fix login bug'` | null | P0 | -- |

### 4.4 TC-U: _getNextActions (executive-summary.js 내부)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-U033 | 다음액션 | plan-plus nextActions에 /pdca design 명령 포함 | `('plan-plus', 'feat')` | 첫 번째 액션 명령에 '/pdca design feat' 포함 | P0 | -- |
| TC-U034 | 다음액션 | plan nextActions에 /plan-plus 명령 포함 | `('plan', 'feat')` | 두 번째 액션 명령에 '/plan-plus feat' 포함 | P0 | -- |
| TC-U035 | 다음액션 | report nextActions에 /pdca archive 명령 포함 | `('report', 'feat')` | 첫 번째 액션 명령에 '/pdca archive feat' 포함 | P0 | -- |
| TC-U036 | 다음액션 | check >=90일 때 첫 번째가 보고서 생성 | `('check', 'feat', {matchRate:92})` | 첫 번째 액션 label = '보고서 생성' | P0 | -- |
| TC-U037 | 다음액션 | check <90일 때 첫 번째가 자동 개선 | `('check', 'feat', {matchRate:80})` | 첫 번째 액션 label = '자동 개선' | P0 | -- |

### 4.5 TC-U: 자동화 함수

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-U038 | 자동화 | emitUserPrompt 전체 필드 | `{message:'msg', feature:'f', phase:'p', suggestions:['s1']}` | 출력 문자열에 모든 필드 포함 | P1 | -- |
| TC-U039 | 자동화 | emitUserPrompt 빈 옵션 | `({})` | 빈 문자열 '' 반환 | P1 | -- |
| TC-U040 | 자동화 | shouldAutoAdvance 반자동 check 단계에서 true | 반자동 설정, phase='check' | true | P1 | -- |
| TC-U041 | 자동화 | shouldAutoAdvance 수동 모드에서 false | 수동 설정, 임의 단계 | false | P1 | -- |
| TC-U042 | 자동화 | generateBatchTrigger 2개 미만 기능 시 null | `(['feat1'], 'plan')` | null | P1 | -- |
| TC-U043 | 자동화 | generateBatchTrigger 2개 이상 기능 시 batch 반환 | `(['f1','f2'], 'plan')` | type='batch', features.length===2 | P1 | -- |
| TC-U044 | 자동화 | getNextPdcaActionAfterCompletion plan -> design | `('plan', 'feat')` | {nextPhase:'design', command:'/pdca design feat'} | P1 | -- |

---

## 5. 통합 테스트 케이스 (gap-detector)

### 5.1 TC-I: 모듈 Export 체인

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-I001 | Export | executive-summary.js가 3개 함수 export | require 모듈 | generateExecutiveSummary, formatExecutiveSummary, generateBatchSummary 모두 typeof function | P0 | -- |
| TC-I002 | Export | pdca/index.js가 executive-summary 3개 함수 모두 재export | require pdca | 3개 함수 모두 접근 가능 | P0 | -- |
| TC-I003 | Export | common.js가 executive-summary 3개 함수 모두 재export | require common | generateExecutiveSummary, formatExecutiveSummary, generateBatchSummary 접근 가능 | P0 | -- |
| TC-I004 | Export | common.js 총 export 수가 199 | `Object.keys(require('./lib/common')).length` | 199 | P0 | -- |
| TC-I005 | Export | pdca/index.js 총 export 수 내역 | require pdca | Tier:8 + Level:7 + Phase:9 + Status:24 + Automation:14 + ExecSummary:3 = 65 | P0 | -- |
| TC-I006 | Export | automation.js가 14개 함수 export | require automation | buildNextActionQuestion, detectPdcaFromTaskSubject 포함 14개 named export | P0 | -- |

### 5.2 TC-I: 모듈 간 통합

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-I007 | 통합 | generateExecutiveSummary가 status 모듈의 getPdcaStatusFull 호출 | Mock status | 크래시 없이 유효한 summary 반환 | P0 | -- |
| TC-I008 | 통합 | generateExecutiveSummary가 core 모듈의 debugLog 호출 | Mock core | 'ExecutiveSummary' 태그로 debugLog 호출됨 | P0 | -- |
| TC-I009 | 통합 | buildNextActionQuestion 출력이 formatAskUserQuestion과 호환 | 체인 호출 | formatAskUserQuestion(buildNextActionQuestion(...))이 유효한 {questions:[...]} 반환 | P0 | -- |
| TC-I010 | 통합 | pdca-task-completed.js가 common.js에서 buildNextActionQuestion import | require 확인 | import 오류 없음 | P0 | -- |
| TC-I011 | 통합 | pdca-task-completed.js가 common.js에서 formatAskUserQuestion import | require 확인 | import 오류 없음 | P0 | -- |
| TC-I012 | 통합 | pdca-task-completed.js가 common.js에서 detectPdcaFromTaskSubject import | require 확인 | import 오류 없음 | P0 | -- |
| TC-I013 | 통합 | gap-detector-stop.js가 preview 필드로 emitUserPrompt 사용 | 코드 경로 추적 | preview를 포함한 questions로 emitUserPrompt 호출 | P1 | -- |
| TC-I014 | 통합 | team-idle-handler.js가 team 모듈의 handleTeammateIdle 사용 | 모듈 로드 | 순환 의존성 없음, 깨끗한 import | P1 | -- |

### 5.3 TC-I: 템플릿 통합

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-I015 | 템플릿 | plan.template.md에 Executive Summary 섹션 포함 | 파일 읽기 | '## Executive Summary'와 4관점 테이블 포함 | P0 | -- |
| TC-I016 | 템플릿 | plan.template.md Executive Summary 테이블에 문제/해결/기능-UX/핵심가치 | 파일 읽기 | 4개 관점 행 모두 존재 | P0 | -- |
| TC-I017 | 템플릿 | report.template.md에 전달 가치 테이블 포함 | 파일 읽기 | '### 1.3 Value Delivered'와 4관점 테이블 포함 | P0 | -- |
| TC-I018 | 템플릿 | plan-plus.template.md에 Executive Summary 섹션 포함 | 파일 읽기 | '## Executive Summary' 섹션 포함 | P1 | -- |
| TC-I019 | 템플릿 | 3개 템플릿 모두 일관된 관점명 사용 | 3개 파일 읽기 | Problem, Solution, Function/UX Effect, Core Value 모두 포함 | P1 | -- |
| TC-I020 | 템플릿 | report.template.md 전달 가치에 지표 한정자 포함 | 파일 읽기 | 기능/UX 효과 행에 'metrics' 또는 'actual' 언급 | P1 | -- |

---

## 6. E2E 테스트 시나리오 (gap-detector)

### 6.1 TC-E: Executive Summary가 포함된 PDCA 전체 사이클

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-E001 | E2E | Plan 단계에서 Executive Summary 섹션 생성 | `/pdca plan test-feature` | Plan 문서에 Executive Summary 테이블 포함 | P0 | -- |
| TC-E002 | E2E | Plan-Plus 단계에서 Executive Summary 생성 | `/plan-plus test-feature` | Plan-plus 문서에 Executive Summary 포함 | P0 | -- |
| TC-E003 | E2E | Report 단계에서 전달 가치 테이블 생성 | `/pdca report test-feature` | 보고서에 전달 가치 4관점 테이블 포함 | P0 | -- |
| TC-E004 | E2E | Plan 완료 시 3개 옵션의 AskUserQuestion 트리거 | [Plan] 태스크 완료 | AskUserQuestion에 '설계 시작', '계획 수정', '다른 기능 우선' 표시 | P0 | -- |
| TC-E005 | E2E | Report 완료 시 3개 옵션의 AskUserQuestion 트리거 | [Report] 태스크 완료 | AskUserQuestion에 '아카이브', '추가 개선', '다음 기능' 표시 | P0 | -- |
| TC-E006 | E2E | Check 단계 >=90%에서 보고서 옵션의 AskUserQuestion 트리거 | gap-detector 95%에서 중지 | AskUserQuestion에 '보고서 생성 (권장)' 표시 | P0 | -- |
| TC-E007 | E2E | Check 단계 <70%에서 강력 권장의 AskUserQuestion 트리거 | gap-detector 50%에서 중지 | AskUserQuestion에 '자동 개선 (강력 권장)' 표시 | P0 | -- |
| TC-E008 | E2E | 전체 PDCA 사이클에서 기능 컨텍스트 보존 | 동일 기능에 모든 단계 실행 | 모든 단계에서 기능명 일관 | P1 | -- |

### 6.2 TC-E: 팀 모드 통합

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-E009 | E2E-팀 | 팀 모드에서 agent_id를 포함한 서브에이전트 생성 | 팀원 생성 | SubagentStart 훅이 agent_id 수신 | P0 | -- |
| TC-E010 | E2E-팀 | 팀 모드 중지 시 agent_id 기록 | 팀원 중지 | SubagentStop 훅 출력에 agentId 필드 포함 | P0 | -- |
| TC-E011 | E2E-팀 | report 단계의 TaskCompleted에서 continue:false 설정 | [Report] 태스크 완료 | 응답 JSON에 continue: false | P0 | -- |
| TC-E012 | E2E-팀 | 다음 태스크 없는 TeammateIdle에서 continue:false 설정 | 팀원 유휴, 대기 없음 | 응답 JSON에 continue: false | P0 | -- |
| TC-E013 | E2E-팀 | 다음 태스크 있는 TeammateIdle에서 continue:false 미설정 | 팀원 유휴, 태스크 대기 중 | 응답 JSON에 continue: undefined | P0 | -- |
| TC-E014 | E2E-팀 | 다중 활성 기능에 대한 Batch summary 작동 | 3개 활성 기능 | generateBatchSummary가 3개 기능 항목 반환 | P1 | -- |
| TC-E015 | E2E-팀 | CTO 팀 오케스트레이션에서 새 Executive Summary 인식 | 팀 PDCA 세션 | 팀이 plan/report에서 Executive Summary 올바르게 참조 | P1 | -- |

---

## 7. UX 테스트 케이스 (product-manager)

### 7.1 TC-X: AskUserQuestion 미리보기 콘텐츠 품질

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-X001 | UX-미리보기 | plan-plus '설계 시작' preview에 명령, 소요시간, 출력 목록 포함 | buildNextActionQuestion('plan-plus', 'feat') | preview에 `/pdca design feat`, '20-40분', 파일 경로 포함 | P0 | -- |
| TC-X002 | UX-미리보기 | plan-plus '계획 수정' preview에 대상 파일 경로 포함 | 동일 | preview에 `docs/01-plan/features/feat.plan.md` 포함 | P0 | -- |
| TC-X003 | UX-미리보기 | plan-plus '팀 리뷰 요청' preview에 Agent Teams 환경변수 언급 | 동일 | preview에 'CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS' 포함 | P0 | -- |
| TC-X004 | UX-미리보기 | plan '설계 시작' preview에 PDCA 상태 바 포함 | buildNextActionQuestion('plan', 'feat') | preview에 '[Plan] OK -> **[Design]**' 포함 | P0 | -- |
| TC-X005 | UX-미리보기 | plan '다른 기능 우선' preview에 재개 명령 표시 | 동일 | preview에 '/pdca status'와 '/pdca design feat' 포함 | P0 | -- |
| TC-X006 | UX-미리보기 | report '아카이브' preview에 4가지 문서 유형 나열 | buildNextActionQuestion('report', 'feat', {matchRate:95}) | preview에 'plan.md, design.md, analysis.md, report.md' 포함 | P0 | -- |
| TC-X007 | UX-미리보기 | report '아카이브' preview에 --summary 팁 언급 | 동일 | preview에 '--summary' 포함 | P0 | -- |
| TC-X008 | UX-미리보기 | report '추가 개선' preview에 현재 일치율 표시 | 동일 | preview에 '95%' 포함 | P0 | -- |
| TC-X009 | UX-미리보기 | report '다음 기능' preview에 시작 명령 표시 | 동일 | preview에 '/plan-plus'와 '/pdca plan' 포함 | P0 | -- |

### 7.2 TC-X: gap-detector-stop 미리보기 콘텐츠

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-X010 | UX-GapStop | >=임계값 구간: preview가 있는 4개 옵션 | matchRate=95, threshold=90 | 4개 옵션: 보고서, /simplify, 계속, 나중에 -- 모두 preview 포함 | P0 | -- |
| TC-X011 | UX-GapStop | maxIter 구간: preview가 있는 3개 옵션 | matchRate=70, iterCount=5 | 3개 옵션: 수동 수정, 현재 완료, 설계 업데이트 -- 모두 preview 포함 | P0 | -- |
| TC-X012 | UX-GapStop | 70-89% 구간: preview가 있는 3개 옵션 | matchRate=80, iterCount=2 | 3개 옵션: 자동 개선, 수동 수정, 현재 완료 -- 모두 preview 포함 | P0 | -- |
| TC-X013 | UX-GapStop | <70% 구간: preview가 있는 3개 옵션 | matchRate=40, iterCount=0 | 3개 옵션: 자동 개선(강력 권장), 전체 설계 업데이트, 수동 수정 -- 모두 preview 포함 | P0 | -- |
| TC-X014 | UX-GapStop | Preview Markdown 유효성 (깨진 링크 없음) | 4개 구간 모두 | 모든 preview 문자열이 유효한 Markdown으로 파싱 | P1 | -- |

### 7.3 TC-X: Executive Summary 표시 품질

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-X015 | UX-ExecSum | 전체 형식에 명확한 섹션 헤더 | formatExecutiveSummary(summary, 'full') | EXECUTIVE SUMMARY 헤더, 4개 괄호 섹션 포함 | P0 | -- |
| TC-X016 | UX-ExecSum | 간략 형식이 7줄 이하 | formatExecutiveSummary(summary, 'compact') | 줄 수 <= 7 | P1 | -- |
| TC-X017 | UX-ExecSum | 전체 형식 다음 액션 번호가 [1]부터 시작 | 3개 액션이 있는 summary | 줄에 [1], [2], [3] 포함 | P1 | -- |
| TC-X018 | UX-ExecSum | 전체 형식 헤더에 단계 포함 | phase='report'인 summary | 두 번째 줄에 '(report)' 포함 | P1 | -- |
| TC-X019 | UX-ExecSum | 간략 형식에 날짜 포함 | generatedAt가 있는 summary | 첫 번째 줄에 날짜 부분 포함 | P1 | -- |
| TC-X020 | UX-ExecSum | formatExecutiveSummary 출력에 이모지 없음 | 임의 summary | 출력 문자열에 이모지 문자 없음 | P2 | -- |

---

## 8. 훅 테스트 케이스 (qa-monitor)

### 8.1 TC-H: hooks.json 설정

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-H001 | 훅설정 | hooks.json에 정확히 10개 훅 이벤트 유형 | hooks.json 파싱 | Object.keys(hooks.hooks).length === 10 | P0 | -- |
| TC-H002 | 훅설정 | hooks.json에 총 13개 훅 항목 | hooks.json 파싱 | 모든 이벤트 배열의 합 === 13 | P0 | -- |
| TC-H003 | 훅설정 | InstructionsLoaded 훅이 존재하지 않음 | hooks.json 파싱 | hooks.hooks에 'InstructionsLoaded' 키 없음 | P0 | -- |
| TC-H004 | 훅설정 | 10개 이벤트 유형 정확히 일치 | hooks.json 파싱 | SessionStart, PreToolUse, PostToolUse, Stop, UserPromptSubmit, PreCompact, TaskCompleted, SubagentStart, SubagentStop, TeammateIdle 정확한 집합 | P0 | -- |
| TC-H005 | 훅설정 | 모든 훅 명령에 node ${CLAUDE_PLUGIN_ROOT}/ 접두사 사용 | hooks.json 파싱 | 모든 command 필드가 'node ${CLAUDE_PLUGIN_ROOT}/'로 시작 | P0 | -- |
| TC-H006 | 훅설정 | 모든 훅 유형이 'command' (http 없음) | hooks.json 파싱 | 모든 type 필드 === 'command' | P0 | -- |

### 8.2 TC-H: pdca-task-completed.js 출력

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-H007 | 태스크완료 | 비PDCA 태스크가 'Non-PDCA task completed' 출력 | `{task_subject:'Fix bug'}` | outputAllow가 포함된 JSON, 크래시 없음 | P0 | -- |
| TC-H008 | 태스크완료 | [Plan] 태스크 자동 전진 시 systemMessage 출력 | `{task_subject:'[Plan] my-feat'}`, 반자동 | 'auto-advance'를 포함한 systemMessage가 있는 JSON | P0 | -- |
| TC-H009 | 태스크완료 | [Report] 태스크 자동 전진 시 continue:false 설정 | `{task_subject:'[Report] feat'}`, 자동 전진 트리거 | JSON에 continue: false | P0 | -- |
| TC-H010 | 태스크완료 | [Plan] 태스크 수동 모드에서 AskUserQuestion 출력 | `{task_subject:'[Plan] feat'}`, 수동 모드 | formatAskUserQuestion 출력이 포함된 userPrompt가 있는 JSON | P0 | -- |
| TC-H011 | 태스크완료 | [Report] 태스크 수동 모드에서 AskUserQuestion 출력 | `{task_subject:'[Report] feat'}`, 수동 모드 | 3개 옵션(아카이브, 개선, 다음)이 포함된 userPrompt가 있는 JSON | P0 | -- |
| TC-H012 | 태스크완료 | hookContext에서 agent_id 추출 | `{task_subject:'[Plan] feat', agent_id:'qa-1'}` | hookSpecificOutput.agentId === 'qa-1' | P0 | -- |
| TC-H013 | 태스크완료 | hookContext에서 agent_type 추출 | `{task_subject:'[Plan] feat', agent_type:'task'}` | hookSpecificOutput.agentType === 'task' | P0 | -- |
| TC-H014 | 태스크완료 | 잘못된 JSON 입력 우아하게 처리 | 유효하지 않은 stdin | 크래시 없이 'TaskCompleted processed.' 출력 | P0 | -- |

### 8.3 TC-H: team-idle-handler.js 출력

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-H015 | 팀원유휴 | 팀 모드 없을 때 허용 메시지 출력 | AGENT_TEAMS 환경변수 없음 | 'TeammateIdle processed (no team mode).' 출력 | P0 | -- |
| TC-H016 | 팀원유휴 | 다음 태스크 있는 유휴: continue가 undefined | `{agent_id:'qa'}`, 대기 태스크 존재 | JSON에 continue: undefined (또는 부재) | P0 | -- |
| TC-H017 | 팀원유휴 | 다음 태스크 없는 유휴: continue가 false | `{agent_id:'qa'}`, 대기 태스크 없음 | JSON에 continue: false | P0 | -- |
| TC-H018 | 팀원유휴 | agent_id가 hookSpecificOutput에 추출 | `{agent_id:'code-analyzer'}` | hookSpecificOutput.agentId === 'code-analyzer' | P0 | -- |
| TC-H019 | 팀원유휴 | agent_type이 hookSpecificOutput에 추출 | `{agent_id:'qa', agent_type:'task'}` | hookSpecificOutput.agentType === 'task' | P0 | -- |

### 8.4 TC-H: subagent-start/stop-handler.js 출력

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-H020 | SubagentStart | agent_id가 일급 필드로 추출 | `{agent_id:'gap-1'}` | hookSpecificOutput.agentId === 'gap-1' | P0 | -- |
| TC-H021 | SubagentStart | agent_name 없을 때 agent_id로 폴백 | `{agent_id:'gap-1'}` (agent_name 없음) | agentName === 'gap-1' | P0 | -- |
| TC-H022 | SubagentStart | agent_type 기본값 'agent' | `{agent_id:'x'}` (agent_type 없음) | agentType === 'agent' | P1 | -- |
| TC-H023 | SubagentStop | 컨텍스트에서 agent_id와 agent_type 추출 | `{agent_id:'qa-1', agent_type:'task'}` | hookSpecificOutput에 agentId='qa-1', agentType='task' | P0 | -- |
| TC-H024 | SubagentStop | transcript_path를 통한 성공 감지 | `{transcript_path:'/tmp/t.json'}` | status === 'completed' | P1 | -- |
| TC-H025 | SubagentStop | exit_code를 통한 실패 감지 | `{exit_code: 1}` | status === 'failed' | P1 | -- |

### 8.5 TC-H: unified-stop.js ENH-74

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-H026 | UnifiedStop | agent_id가 에이전트 감지 소스로 사용 | `{agent_id:'gap-detector'}` | detectActiveAgent가 'gap-detector' 반환 | P0 | -- |
| TC-H027 | UnifiedStop | agent_id/agent_type이 디버그 컨텍스트에 기록 | `{agent_id:'x', agent_type:'task'}` | agentId와 agentType 필드로 debugLog 호출 | P1 | -- |

### 8.6 TC-H: gap-detector-stop.js 출력 구조

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-H028 | GapStop | 출력 JSON에 decision='allow' | 임의 입력 | response.decision === 'allow' | P0 | -- |
| TC-H029 | GapStop | 출력 JSON에 matchRate가 있는 analysisResult | '95%' 일치 입력 | response.analysisResult.matchRate === 95 | P0 | -- |
| TC-H030 | GapStop | 출력 JSON에 AskUserQuestion이 있는 userPrompt | 임의 matchRate | response.userPrompt가 null이 아닌 객체/문자열 | P0 | -- |
| TC-H031 | GapStop | 출력 JSON에 AskUserQuestion 의무화가 있는 systemMessage | 임의 입력 | response.systemMessage에 'MANDATORY'와 'AskUserQuestion' 포함 | P0 | -- |
| TC-H032 | GapStop | 4개 matchRate 구간별 각기 다른 안내 텍스트 생성 | 95, maxIter, 80, 40 | 각 구간이 서로 다른 안내 접두사 | P1 | -- |

---

## 9. 회귀 테스트 케이스 (qa-strategist)

### 9.1 TC-R: v1.5.8 기능 보존

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-R001 | 회귀 | PDCA plan 단계에서 plan.md 파일 생성 | `/pdca plan test` | docs/01-plan/features/test.plan.md에 파일 생성 | P0 | -- |
| TC-R002 | 회귀 | PDCA design 단계에서 plan 문서 필요 | `/pdca design test` (plan 없음) | 경고: plan 문서 누락 | P0 | -- |
| TC-R003 | 회귀 | PDCA status에서 올바른 단계 시각화 | `/pdca status` | 단계 마커가 있는 ASCII 진행 막대 | P0 | -- |
| TC-R004 | 회귀 | common.js 코어 모듈 export(49개) 무결 | require common.js | 49개 코어 export(platform, cache, IO, debug, config, file, paths) 모두 존재 | P0 | -- |
| TC-R005 | 회귀 | common.js 인텐트 모듈 export(19개) 무결 | require common.js | 19개 인텐트 export 모두 존재 | P0 | -- |
| TC-R006 | 회귀 | common.js 태스크 모듈 export(26개) 무결 | require common.js | 26개 태스크 export 모두 존재 | P0 | -- |
| TC-R007 | 회귀 | common.js 팀 모듈 export(40개) 무결 | require common.js | 40개 팀 export 모두 존재 | P0 | -- |
| TC-R008 | 회귀 | 경로 레지스트리 (STATE_PATHS, LEGACY_PATHS, CONFIG_PATHS) 정상 작동 | require common.js | STATE_PATHS.pdcaStatus()가 유효한 경로 반환 | P0 | -- |
| TC-R009 | 회귀 | 에이전트 frontmatter 모델 스펙 변경 없음 | 16개 에이전트 .md 파일 읽기 | 7 opus, 7 sonnet, 2 haiku (v1.5.8과 동일한 분포) | P1 | -- |
| TC-R010 | 회귀 | 27개 스킬 모두 유효한 SKILL.md 보유 | skills/*/SKILL.md 확인 | 27개 모두 존재하고 description 필드 포함 | P1 | -- |

### 9.2 TC-R: 훅 시스템 회귀

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-R011 | 회귀 | SessionStart 훅이 한 번만 실행 (once:true) | 세션 초기화 | session-start.js가 정확히 한 번 실행 | P1 | -- |
| TC-R012 | 회귀 | PreToolUse Write|Edit 매처 작동 | Write 도구 호출 | pre-write.js 실행 | P1 | -- |
| TC-R013 | 회귀 | PostToolUse Skill 매처 작동 | Skill 도구 호출 | skill-post.js 실행 | P1 | -- |
| TC-R014 | 회귀 | PreCompact auto|manual 매처 작동 | 압축 트리거 | context-compaction.js 실행 | P2 | -- |
| TC-R015 | 회귀 | 메모리 시스템: bkit 메모리(.bkit/state/memory.json) vs CC 자동 메모리(~/.claude/projects/) 분리 | 경로 확인 | 서로 다른 파일시스템 위치, 충돌 없음 | P2 | -- |

---

## 10. 엣지 케이스 및 오류 처리 (code-analyzer)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-EC001 | 엣지 | 빈 기능명으로 generateExecutiveSummary | `('', 'plan')` | 빈 feature로 유효한 구조 반환 | P1 | -- |
| TC-EC002 | 엣지 | .bkit-memory.json 누락 시 generateExecutiveSummary | 상태 파일 없음 | null matchRate, 0 iterationCount로 summary 반환 | P1 | -- |
| TC-EC003 | 엣지 | nextActions 누락된 summary로 formatExecutiveSummary | nextActions 없는 summary | 크래시 없음, 다음 액션 섹션 없음 | P1 | -- |
| TC-EC004 | 엣지 | 100개 기능으로 generateBatchSummary | 100개 문자열 배열 | 100개 항목의 batch 반환, 크래시 없음 | P2 | -- |
| TC-EC005 | 엣지 | matchRate=0으로 buildNextActionQuestion | `('report', 'feat', {matchRate:0})` | 질문에 '0%' 포함 | P1 | -- |
| TC-EC006 | 엣지 | task_subject 누락된 pdca-task-completed.js | `{}` (task_subject 없음) | 'Non-PDCA task completed'로 종료 | P1 | -- |
| TC-EC007 | 엣지 | 잘못된 JSON stdin으로 team-idle-handler.js | 깨진 JSON 문자열 | 우아한 처리, 허용 메시지 출력 | P1 | -- |
| TC-EC008 | 엣지 | 정확한 임계값(90)에서 gap-detector-stop.js | '90%' 일치 입력 | >=임계값 구간(완료)에 진입 | P0 | -- |
| TC-EC009 | 엣지 | matchRate=100에서 gap-detector-stop.js | '100%' 일치 입력 | 완료 구간, 크래시 없음 | P2 | -- |
| TC-EC010 | 엣지 | 동시 기능 상태 업데이트 | 두 기능이 동시 업데이트 | .bkit-memory.json에 데이터 손상 없음 | P2 | -- |

---

## 11. 성능 테스트 케이스 (qa-monitor)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-P001 | 성능 | 50개 기능으로 generateBatchSummary 100ms 미만 완료 | 50개 기능명 | 실행 시간 < 100ms | P2 | -- |
| TC-P002 | 성능 | formatExecutiveSummary 전체 형식 10ms 미만 완료 | 유효한 summary 객체 | 실행 시간 < 10ms | P2 | -- |
| TC-P003 | 성능 | 훅 스크립트 콜드 스타트(pdca-task-completed.js) 500ms 미만 | node 실행 | 500ms 내에 process.exit 도달 | P2 | -- |
| TC-P004 | 성능 | common.js require 시간이 v1.5.8 기준 대비 변동 없음 | require('./lib/common') | require 시간이 v1.5.8의 20% 이내 | P2 | -- |
| TC-P005 | 성능 | gap-detector-stop.js가 대용량 입력(>100KB) 타임아웃 없이 처리 | 100KB stdin 입력 | 10초 타임아웃 내 완료 | P2 | -- |

---

## 12. 테스트 실행 계획

### 12.1 실행 우선순위 및 의존성

```
1단계 (P0 크리티컬 패스):
  TC-H001~H006 (훅 설정)              -- 모든 훅 테스트의 선행 조건
  TC-I001~I006 (Export 체인)           -- 모든 유닛 테스트의 선행 조건
  TC-U001~U016 (Executive Summary)    -- 아래와 병렬
  TC-U017~U025 (Preview UX)           -- 위와 병렬
  TC-U027~U037 (감지 + 액션)          -- 위와 병렬

2단계 (P0 통합):
  TC-H007~H032 (훅 스크립트 출력)      -- 1단계에 의존
  TC-I007~I020 (모듈 간 + 템플릿)      -- 1단계에 의존
  TC-E001~E013 (E2E 시나리오)          -- 1+2단계에 의존

3단계 (P1 품질):
  TC-X001~X020 (UX 품질)              -- 2단계에 의존
  TC-R001~R015 (회귀)                  -- UX와 병렬
  TC-U026, U038~U044 (P1 유닛)        -- 병렬

4단계 (P2 강화):
  TC-EC001~EC010 (엣지 케이스)          -- 2단계에 의존
  TC-P001~P005 (성능)                  -- 독립
  TC-E014~E015 (P1 E2E)               -- 2단계에 의존
```

### 12.2 에이전트 태스크 배정

| 에이전트 | 테스트 카테고리 | TC 수 | 예상 소요시간 |
|----------|----------------|:-----:|:------------:|
| code-analyzer | 유닛 테스트 (TC-U), 엣지 케이스 (TC-EC) | 54 | 2-3시간 |
| qa-monitor | 훅 테스트 (TC-H), 성능 (TC-P) | 37 | 2-3시간 |
| gap-detector | 통합 (TC-I), E2E (TC-E) | 35 | 3-4시간 |
| product-manager | UX 테스트 (TC-X) | 20 | 1-2시간 |
| qa-strategist | 회귀 (TC-R), 조율 | 15 | 1-2시간 |

### 12.3 환경 요구사항

| 요구사항 | 값 | 비고 |
|----------|---|------|
| Claude Code 버전 | v2.1.69+ | preview 필드 지원에 필요 |
| Agent Teams | CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 | 팀 모드 테스트에 필요 |
| Node.js | 18+ | 스크립트 실행용 |
| bkit 버전 | 1.5.9 | 테스트 대상 |
| 이전 버전 | 1.5.8 | 회귀 기준선 |

---

## 13. 성공 기준

### 13.1 통과율 목표 (섹션 4-11)

| 카테고리 | 목표 | 최소 |
|----------|:----:|:----:|
| P0 테스트 (113) | 100% | 100% |
| P1 테스트 (37) | 100% | 95% |
| P2 테스트 (11) | 95% | 90% |
| v1.5.9 소계 (161) | 100% | 97% |

### 13.2 품질 게이트

| 게이트 | 기준 | 차단 |
|--------|------|:----:|
| G1 | 모든 P0 유닛 테스트 통과 (TC-U001~U037) | 예 |
| G2 | 모든 P0 훅 테스트 통과 (TC-H001~H031) | 예 |
| G3 | 모든 P0 통합 테스트 통과 (TC-I001~I012) | 예 |
| G4 | 모든 P0 E2E 테스트 통과 (TC-E001~E013) | 예 |
| G5 | P0 엣지 케이스 TC-EC008 (임계값 경계) 통과 | 예 |
| G6 | common.js export 수 === 199 (TC-I004) | 예 |
| G7 | P0 카테고리에서 FAIL 0건 | 예 |
| G8 | 회귀 테스트 100% (TC-R001~R010) | 예 |

### 13.3 완료 기준

- 268개 P0 테스트 케이스 모두 실행 및 통과
- 167개 P1 테스트 케이스 모두 실행, >= 95% 통과율
- 35개 P2 테스트 케이스 모두 실행, >= 90% 통과율
- Critical 또는 High 심각도 버그 잔존 없음
- `docs/04-report/features/bkit-v1.5.9-comprehensive-test.report.md`에 테스트 보고서 생성

---

---

## 14. 코어 시스템 테스트 (code-analyzer)

### 14.1 TC-CS: 플랫폼 감지 (9개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CS001 | 플랫폼 | CC 환경에서 detectPlatform이 'claude-code' 반환 | CC 환경 | 'claude-code' | P0 | -- |
| TC-CS002 | 플랫폼 | BKIT_PLATFORM이 비어있지 않은 문자열 | require core | typeof string, length > 0 | P0 | -- |
| TC-CS003 | 플랫폼 | CC 환경에서 isClaudeCode가 boolean true 반환 | CC 환경 | true | P0 | -- |
| TC-CS004 | 플랫폼 | PLUGIN_ROOT가 유효한 절대 경로로 해석 | require core | '/'로 시작하고 'bkit' 포함 | P0 | -- |
| TC-CS005 | 플랫폼 | PROJECT_DIR가 유효한 디렉토리로 해석 | require core | 비어있지 않은 문자열, 경로 존재 | P0 | -- |
| TC-CS006 | 플랫폼 | BKIT_PROJECT_DIR가 올바르게 해석 | require core | 비어있지 않은 문자열 | P1 | -- |
| TC-CS007 | 플랫폼 | getPluginPath('scripts/foo.js')가 절대 경로 반환 | `('scripts/foo.js')` | PLUGIN_ROOT로 시작하는 경로 | P0 | -- |
| TC-CS008 | 플랫폼 | getProjectPath('docs/')가 프로젝트 상대 경로 반환 | `('docs/')` | PROJECT_DIR로 시작하는 경로 | P0 | -- |
| TC-CS009 | 플랫폼 | getTemplatePath('plan.template.md')가 유효한 템플릿 경로 반환 | `('plan.template.md')` | 파일시스템에 경로 존재 | P0 | -- |

### 14.2 TC-CS: 캐시 (7개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CS010 | 캐시 | set/get 왕복으로 동일 값 반환 | set('k','v'), get('k') | 'v' | P0 | -- |
| TC-CS011 | 캐시 | 없는 키에 get이 null 반환 | get('nonexistent') | null | P0 | -- |
| TC-CS012 | 캐시 | invalidate가 특정 키 제거 | set('k','v'), invalidate('k'), get('k') | null | P0 | -- |
| TC-CS013 | 캐시 | clear가 모든 키 제거 | 여러 개 set, clear() | 모든 get()이 null 반환 | P0 | -- |
| TC-CS014 | 캐시 | DEFAULT_TTL이 양수 | require core | typeof number, > 0 | P1 | -- |
| TC-CS015 | 캐시 | _cache가 일반 Object (Map 아님) | require core | typeof _cache === 'object', Map 아님 | P1 | -- |
| TC-CS016 | 캐시 | globalCache가 export된 객체 | require core | typeof globalCache === 'object' | P1 | -- |

### 14.3 TC-CS: I/O (9개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CS017 | IO | MAX_CONTEXT_LENGTH가 양의 정수 | require core | typeof number, > 0, 정수 | P0 | -- |
| TC-CS018 | IO | truncateContext가 MAX_CONTEXT_LENGTH에서 잘림 | MAX+100 문자의 문자열 | Result.length <= MAX_CONTEXT_LENGTH | P0 | -- |
| TC-CS019 | IO | truncateContext가 짧은 문자열은 변경 없이 반환 | 'hello' | 'hello' | P0 | -- |
| TC-CS020 | IO | parseHookInput이 유효한 JSON 파싱 | '{"key":"val"}' | {key:'val'} | P0 | -- |
| TC-CS021 | IO | outputAllow가 decision='allow'인 JSON 반환 | - | JSON.parse(stdout).decision === 'allow' | P0 | -- |
| TC-CS022 | IO | outputBlock이 decision='block'인 JSON 반환 | - | JSON.parse(stdout).decision === 'block' | P0 | -- |
| TC-CS023 | IO | outputEmpty가 빈 JSON 또는 최소 출력 반환 | - | 유효한 JSON 출력 | P1 | -- |
| TC-CS024 | IO | xmlSafeOutput이 < > & 문자 이스케이프 | '<script>alert("xss")</script>' | 출력에 리터럴 < 또는 > 없음 | P1 | -- |

### 14.4 TC-CS: 디버그, 설정, 파일, 경로

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CS025 | 디버그 | DEBUG_LOG_PATHS가 알려진 키를 가진 객체 | require core | 'hooks', 'intent' 등 키 보유 | P1 | -- |
| TC-CS026 | 디버그 | getDebugLogPath가 경로 문자열 반환 | `('hooks')` | 비어있지 않은 문자열 경로 | P1 | -- |
| TC-CS027 | 디버그 | debugLog가 유효한 입력에서 예외 미발생 | `('test', 'msg')` | 예외 없음 | P0 | -- |
| TC-CS028 | 설정 | loadConfig가 유효한 객체 반환 | loadConfig() | typeof object, 'pdca' 키 보유 | P0 | -- |
| TC-CS029 | 설정 | getConfig가 알려진 키의 값 반환 | `('pdca.matchRateThreshold')` | 90 | P0 | -- |
| TC-CS030 | 설정 | getConfig가 없는 키에 기본값 반환 | `('nonexistent', 42)` | 42 | P0 | -- |
| TC-CS031 | 설정 | getConfigArray가 공백 구분 문자열 반환 | `('fileDetection.excludePatterns')` | typeof string | P1 | -- |
| TC-CS032 | 설정 | getBkitConfig가 전체 설정 반환 | getBkitConfig() | pdca, triggers, team 키 보유 | P1 | -- |
| TC-CS033 | 설정 | safeJsonParse가 유효한 JSON에 파싱 객체 반환 | `('{"a":1}')` | {a:1} | P0 | -- |
| TC-CS034 | 설정 | safeJsonParse가 잘못된 JSON에 null 반환 | `('not json')` | null | P0 | -- |
| TC-CS035 | 파일 | TIER_EXTENSIONS가 티어 키를 가진 객체 | require core | 'tier1', 'tier2' 등 키 보유 | P1 | -- |
| TC-CS036 | 파일 | isSourceFile이 .js 파일에 true 반환 | `('app.js')` | true | P0 | -- |
| TC-CS037 | 파일 | isSourceFile이 .png 파일에 false 반환 | `('image.png')` | false | P0 | -- |
| TC-CS038 | 파일 | isCodeFile이 .ts에 true 반환 | `('main.ts')` | true | P1 | -- |
| TC-CS039 | 파일 | isUiFile이 .tsx/.vue/.svelte에 true 반환 | `('App.tsx')` | true | P1 | -- |
| TC-CS040 | 파일 | isEnvFile이 .env에 true 반환 | `('.env')` | true | P1 | -- |
| TC-CS041 | 경로 | STATE_PATHS에 pdcaStatus 함수 존재 | require core | typeof STATE_PATHS.pdcaStatus === 'function' | P0 | -- |
| TC-CS042 | 경로 | LEGACY_PATHS가 객체로 존재 | require core | typeof object | P1 | -- |
| TC-CS043 | 경로 | CONFIG_PATHS에 예상 키 존재 | require core | 알려진 설정 경로 키 보유 | P1 | -- |
| TC-CS044 | 경로 | getDocPaths가 plan/design/analysis/report가 있는 객체 반환 | getDocPaths('feat') | 4개 키 모두 보유, 각각 비어있지 않음 | P0 | -- |
| TC-CS045 | 경로 | findDoc이 존재하지 않는 기능에 null 반환 | findDoc('nonexistent-xyz', 'plan') | null | P1 | -- |
| TC-CS046 | 경로 | getArchivePath가 날짜 형식 포함 경로 반환 | getArchivePath('feat') | YYYY-MM 패턴 포함 | P1 | -- |

---

## 15. 인텐트 시스템 테스트 (product-manager)

### 15.1 TC-IN: 언어 감지 (6개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-IN001 | 언어 | SUPPORTED_LANGUAGES에 정확히 8개 항목 | require intent | ['en','ko','ja','zh','es','fr','de','it'] | P0 | -- |
| TC-IN002 | 언어 | detectLanguage('hello world')가 'en' 반환 | `('hello world')` | 'en' | P0 | -- |
| TC-IN003 | 언어 | detectLanguage('안녕하세요')가 'ko' 반환 | `('안녕하세요')` | 'ko' | P0 | -- |
| TC-IN004 | 언어 | detectLanguage('こんにちは')가 'ja' 반환 | `('こんにちは')` | 'ja' | P0 | -- |
| TC-IN005 | 언어 | detectLanguage('你好世界')가 'zh' 반환 | `('你好世界')` | 'zh' | P0 | -- |
| TC-IN006 | 언어 | detectLanguage('hola mundo')가 'es' 반환 | `('hola mundo')` | 'es' | P1 | -- |
| TC-IN007 | 언어 | detectLanguage('bonjour le monde')가 'fr' 반환 | `('bonjour le monde')` | 'fr' | P1 | -- |
| TC-IN008 | 언어 | detectLanguage('hallo welt')가 'de' 반환 | `('hallo welt')` | 'de' | P1 | -- |
| TC-IN009 | 언어 | detectLanguage('ciao mondo')가 'it' 반환 | `('ciao mondo')` | 'it' | P1 | -- |
| TC-IN010 | 언어 | detectLanguage(null)이 기본값 'en' 반환 | `(null)` | 'en' | P0 | -- |
| TC-IN011 | 언어 | AGENT_TRIGGER_PATTERNS에 6개 에이전트 키 | require intent | gap-detector, pdca-iterator, code-analyzer, report-generator, design-validator, qa-monitor 키 포함 | P0 | -- |
| TC-IN012 | 언어 | 각 에이전트 패턴에 8개 언어 키 | AGENT_TRIGGER_PATTERNS['gap-detector'] | en, ko, ja, zh, es, fr, de, it 보유 | P0 | -- |
| TC-IN013 | 언어 | SKILL_TRIGGER_PATTERNS에 예상 스킬 키 | require intent | 최소 4개 스킬 키 보유 | P1 | -- |
| TC-IN014 | 언어 | getAllPatterns가 결합 패턴 반환 | getAllPatterns() | 비어있지 않은 객체 | P1 | -- |
| TC-IN015 | 언어 | matchMultiLangPattern이 한국어 매칭 | `('검증해줘', gapPatterns)` | true | P0 | -- |
| TC-IN016 | 언어 | matchMultiLangPattern이 일본어 매칭 | `('確認して', gapPatterns)` | true | P1 | -- |

### 15.2 TC-IN: 트리거 매칭 (5개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-IN017 | 트리거 | NEW_FEATURE_PATTERNS에 8개 언어 키 | require intent | en, ko, ja, zh, es, fr, de, it가 있는 객체 | P0 | -- |
| TC-IN018 | 트리거 | matchImplicitAgentTrigger('verify this code')가 gap-detector 반환 | `('verify this code')` | {agent:'bkit:gap-detector', confidence:0.8} | P0 | -- |
| TC-IN019 | 트리거 | matchImplicitAgentTrigger('코드 리뷰 해줘')가 code-analyzer 반환 | `('코드 리뷰 해줘')` | {agent:'bkit:code-analyzer', confidence:0.8} | P0 | -- |
| TC-IN020 | 트리거 | matchImplicitAgentTrigger가 매칭 안 되는 입력에 null 반환 | `('random text xyz')` | null | P0 | -- |
| TC-IN021 | 트리거 | matchImplicitSkillTrigger('plan my feature')가 pdca 스킬 반환 | `('plan my feature')` | {skill:..., confidence:>=0.7} 또는 매치 객체 | P1 | -- |
| TC-IN022 | 트리거 | matchImplicitSkillTrigger가 매칭 안 되는 입력에 null | `('xyz123')` | null | P1 | -- |
| TC-IN023 | 트리거 | detectNewFeatureIntent('add new feature login')가 truthy 반환 | `('add new feature login')` | Truthy 결과 | P0 | -- |
| TC-IN024 | 트리거 | extractFeatureNameFromRequest('implement user-auth')가 이름 추출 | `('implement user-auth')` | 'user-auth' 또는 'user auth' 포함 | P1 | -- |

### 15.3 TC-IN: 모호성 분석 (8개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-IN025 | 모호성 | containsFilePath('fix src/app.js')가 true 반환 | `('fix src/app.js')` | true | P0 | -- |
| TC-IN026 | 모호성 | containsFilePath('make it better')가 false 반환 | `('make it better')` | false | P0 | -- |
| TC-IN027 | 모호성 | containsTechnicalTerms('optimize the SQL query')가 true 반환 | `('optimize the SQL query')` | true | P1 | -- |
| TC-IN028 | 모호성 | calculateAmbiguityScore가 0-1 float 반환 | `('vague request')` | score >= 0 && score <= 1 | P0 | -- |
| TC-IN029 | 모호성 | 구체적 요청의 calculateAmbiguityScore가 낮음 | `('Fix line 42 in src/app.js: change let to const')` | score < 0.3 | P0 | -- |
| TC-IN030 | 모호성 | 모호한 요청의 calculateAmbiguityScore가 높음 | `('make it better')` | score > 0.5 | P0 | -- |
| TC-IN031 | 모호성 | generateClarifyingQuestions가 배열 반환 | `('make it better')` | length >= 1인 배열 | P1 | -- |
| TC-IN032 | 모호성 | 모호한 텍스트의 hasMultipleInterpretations | `('fix the thing')` | true | P1 | -- |

---

## 16. 태스크 시스템 테스트 (code-analyzer)

### 16.1 TC-TS: 분류 (6개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TS001 | 분류 | CLASSIFICATION_THRESHOLDS가 크기 경계가 있는 객체 | require task | quickFix, minorChange, feature 키 보유 | P0 | -- |
| TC-TS002 | 분류 | classifyTask('fix typo')가 'quickFix' 반환 | `('fix typo')` | 'quickFix' 또는 동급 수준 | P0 | -- |
| TC-TS003 | 분류 | 500자 입력의 classifyTask가 'feature' 반환 | 500자 문자열 | 'feature' 분류 | P0 | -- |
| TC-TS004 | 분류 | 2000자 입력의 classifyTask가 'majorFeature' 반환 | 2000자 문자열 | 'majorFeature' 분류 | P0 | -- |
| TC-TS005 | 분류 | getPdcaLevel('quickFix')가 'none' 반환 | `('quickFix')` | 'none' 또는 동급 | P0 | -- |
| TC-TS006 | 분류 | getPdcaGuidance가 문자열 안내 반환 | `('feature')` | 비어있지 않은 문자열 | P1 | -- |
| TC-TS007 | 분류 | 5줄의 classifyTaskByLines가 적절한 수준 반환 | 5줄 문자열 | Minor 또는 quick 분류 | P1 | -- |

### 16.2 TC-TS: 컨텍스트 (7개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TS008 | 컨텍스트 | setActiveSkill/getActiveSkill 왕복 | setActiveSkill('pdca'), getActiveSkill() | 'pdca' | P0 | -- |
| TC-TS009 | 컨텍스트 | setActiveAgent/getActiveAgent 왕복 | setActiveAgent('gap-detector'), getActiveAgent() | 'gap-detector' | P0 | -- |
| TC-TS010 | 컨텍스트 | clearActiveContext가 스킬과 에이전트 모두 초기화 | 둘 다 설정 후, clearActiveContext() | getActiveSkill() null, getActiveAgent() null | P0 | -- |
| TC-TS011 | 컨텍스트 | 스킬 설정 시 hasActiveContext가 true | setActiveSkill('pdca') | true | P0 | -- |
| TC-TS012 | 컨텍스트 | 초기화 후 hasActiveContext가 false | clearActiveContext() | false | P0 | -- |
| TC-TS013 | 컨텍스트 | getActiveContext가 결합 객체 반환 | 스킬 + 에이전트 설정 | {skill:..., agent:...} | P1 | -- |

### 16.3 TC-TS: 생성기 (6개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TS014 | 생성기 | generatePdcaTaskSubject('plan', 'my-feat') | `('plan', 'my-feat')` | '[Plan] my-feat' | P0 | -- |
| TC-TS015 | 생성기 | generatePdcaTaskSubject('act', 'feat') | `('act', 'feat')` | '[Act' 포함 | P0 | -- |
| TC-TS016 | 생성기 | generatePdcaTaskDescription이 비어있지 않은 문자열 반환 | `('plan', 'feat')` | 비어있지 않은 문자열 | P1 | -- |
| TC-TS017 | 생성기 | createPdcaTaskChain이 태스크 객체 배열 반환 | `('my-feat')` | plan, design, do, check 항목이 있는 배열 | P1 | -- |
| TC-TS018 | 생성기 | getPdcaTaskMetadata가 phase/feature가 있는 객체 반환 | `('plan', 'feat')` | {phase:'plan', feature:'feat', ...} | P1 | -- |

### 16.4 TC-TS: 추적기 (7개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TS019 | 추적기 | savePdcaTaskId/getPdcaTaskId 왕복 | savePdcaTaskId('feat','plan','t1'), getPdcaTaskId('feat','plan') | 't1' | P0 | -- |
| TC-TS020 | 추적기 | getCurrentPdcaPhase가 현재 단계 문자열 반환 | 상태 업데이트 후 | 유효한 단계명 또는 null | P1 | -- |
| TC-TS021 | 추적기 | findPdcaStatus가 활성 기능 검색 | 기능 생성 후 | 일치하는 기능 상태 반환 | P1 | -- |

---

## 17. 팀 시스템 테스트 (code-analyzer)

### 17.1 TC-TM: 조율기 (5개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TM001 | 조율기 | isTeamModeAvailable이 boolean 반환 | 테스트 환경에서 호출 | typeof boolean | P0 | -- |
| TC-TM002 | 조율기 | getTeamConfig가 설정 객체 반환 | getTeamConfig() | enabled, maxTeammates, ctoAgent 키 보유 | P0 | -- |
| TC-TM003 | 조율기 | generateTeamStrategy('Dynamic')가 3팀원 계획 반환 | `('Dynamic')` | roles 배열 길이 <= 3 | P0 | -- |
| TC-TM004 | 조율기 | generateTeamStrategy('Enterprise')가 5팀원 계획 반환 | `('Enterprise')` | roles 배열 길이 <= 5 | P0 | -- |
| TC-TM005 | 조율기 | formatTeamStatus가 문자열 반환 | formatTeamStatus() | typeof string, 비어있지 않음 | P1 | -- |

### 17.2 TC-TM: 전략 (2개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TM006 | 전략 | TEAM_STRATEGIES에 Dynamic과 Enterprise 존재 | require team | 'Dynamic'과 'Enterprise' 키 보유 | P0 | -- |
| TC-TM007 | 전략 | getTeammateRoles('Dynamic')가 역할 배열 반환 | `('Dynamic')` | developer, qa 역할이 있는 배열 | P0 | -- |

### 17.3 TC-TM: 오케스트레이터 (6개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TM008 | 오케스트레이터 | PHASE_PATTERN_MAP에 모든 PDCA 단계 | require team | plan, design, do, check, act 키 보유 | P0 | -- |
| TC-TM009 | 오케스트레이터 | selectOrchestrationPattern('Dynamic','plan')이 'leader' 반환 | `('Dynamic','plan')` | 'leader' | P0 | -- |
| TC-TM010 | 오케스트레이터 | selectOrchestrationPattern('Enterprise','check')이 'council' 반환 | `('Enterprise','check')` | 'council' | P0 | -- |
| TC-TM011 | 오케스트레이터 | composeTeamForPhase가 팀원 배열 반환 | `('Enterprise','do','feat')` | 비어있지 않은 배열 | P1 | -- |
| TC-TM012 | 오케스트레이터 | generateSpawnTeamCommand가 문자열 명령 반환 | `([{role:'qa', agent:'qa-monitor'}])` | spawn/agent 참조가 포함된 문자열 | P1 | -- |
| TC-TM013 | 오케스트레이터 | shouldRecomposeTeam이 boolean 반환 | `('Enterprise','plan','design')` | typeof boolean | P1 | -- |

### 17.4 TC-TM: 통신 (6개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TM014 | 통신 | MESSAGE_TYPES가 알려진 유형이 있는 객체 | require team | 'directive', 'status', 'request' 등 보유 | P0 | -- |
| TC-TM015 | 통신 | createMessage에 4개 인자 필요 | `('cto','qa','directive','payload')` | 유효한 메시지 객체 | P0 | -- |
| TC-TM016 | 통신 | createMessage 3개 인자 시 에러 또는 null | `('cto','qa','directive')` | 에러 또는 null | P1 | -- |
| TC-TM017 | 통신 | createBroadcast가 모든 역할에 발송 | `('cto','status','update msg')` | 수신자가 있는 브로드캐스트 객체 | P1 | -- |
| TC-TM018 | 통신 | createPhaseTransitionNotice에 단계 정보 포함 | `('plan','design','feat')` | 'plan'과 'design' 포함 | P1 | -- |
| TC-TM019 | 통신 | createPlanDecision이 승인/거부 구조 반환 | `('approve','plan doc looks good')` | {decision:'approve', ...} | P1 | -- |

### 17.5 TC-TM: 태스크 큐, CTO 로직, 상태 기록기

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TM020 | 태스크큐 | createTeamTasks가 태스크 배열 반환 | `('feat', 'plan', ['qa','dev'])` | 태스크 객체 배열 | P0 | -- |
| TC-TM021 | 태스크큐 | isPhaseComplete가 boolean 반환 | `([{status:'done'},{status:'done'}])` | true | P0 | -- |
| TC-TM022 | 태스크큐 | findNextAvailableTask가 비어있을 때 null 반환 | `([])` | null | P1 | -- |
| TC-TM023 | CTO로직 | decidePdcaPhase가 유효한 단계 반환 | `({matchRate:95, phase:'check'})` | 'report' 또는 유효한 다음 단계 | P0 | -- |
| TC-TM024 | CTO로직 | evaluateCheckResults >=90에서 report 권장 | `({matchRate:92, criticalIssues:0})` | 권장에 'report' 포함 | P0 | -- |
| TC-TM025 | CTO로직 | evaluateCheckResults <70에서 재설계 권장 | `({matchRate:60})` | 권장에 'redesign' 또는 'design' 포함 | P0 | -- |
| TC-TM026 | 상태기록기 | initAgentState가 상태 파일 생성 | initAgentState() | 에러 없음, 상태 객체 반환 | P1 | -- |
| TC-TM027 | 상태기록기 | getAgentStatePath가 경로 반환 | getAgentStatePath() | 비어있지 않은 문자열 경로 | P1 | -- |

---

## 18. PDCA 전체 시스템 테스트 (code-analyzer)

### 18.1 TC-PD: 티어 시스템 (8개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PD001 | 티어 | getLanguageTier('.js')가 'tier1' 반환 | `('.js')` | 'tier1' | P0 | -- |
| TC-PD002 | 티어 | getLanguageTier('.go')가 'tier2' 반환 | `('.go')` | 'tier2' | P0 | -- |
| TC-PD003 | 티어 | getLanguageTier('.swift')가 'tier3' 반환 | `('.swift')` | 'tier3' | P1 | -- |
| TC-PD004 | 티어 | isTier1('.ts')가 true 반환 | `('.ts')` | true | P0 | -- |
| TC-PD005 | 티어 | isExperimentalTier가 boolean 반환 | `('.xyz')` | typeof boolean | P1 | -- |
| TC-PD006 | 티어 | getTierDescription이 4개 티어 모두에 문자열 반환 | 각 티어 | 각각에 비어있지 않은 문자열 | P1 | -- |

### 18.2 TC-PD: 레벨 시스템 (7개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PD007 | 레벨 | LEVEL_PHASE_MAP에 Starter/Dynamic/Enterprise | require pdca | 3개 키 모두 존재 | P0 | -- |
| TC-PD008 | 레벨 | detectLevel이 3개 레벨 중 하나 반환 | detectLevel() | 'Starter' 또는 'Dynamic' 또는 'Enterprise' | P0 | -- |
| TC-PD009 | 레벨 | canSkipPhase가 Starter+design에 true 반환 | `('Starter','design')` | true (Starter는 건너뛸 수 있음) | P0 | -- |
| TC-PD010 | 레벨 | getRequiredPhases('Enterprise')가 모든 단계 반환 | `('Enterprise')` | plan, design, do, check, act, report가 있는 배열 | P0 | -- |

### 18.3 TC-PD: 단계 시스템 (9개 export)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PD011 | 단계 | PDCA_PHASES에 6개 단계 | require pdca | plan, design, do, check, act, report가 있는 배열/객체 | P0 | -- |
| TC-PD012 | 단계 | getPhaseNumber('plan')이 1 반환 | `('plan')` | 1 | P0 | -- |
| TC-PD013 | 단계 | getPhaseName(1)이 'plan' 반환 | `(1)` | 'plan' | P0 | -- |
| TC-PD014 | 단계 | getNextPdcaPhase('plan')이 'design' 반환 | `('plan')` | 'design' | P0 | -- |
| TC-PD015 | 단계 | getPreviousPdcaPhase('design')이 'plan' 반환 | `('design')` | 'plan' | P1 | -- |
| TC-PD016 | 단계 | validatePdcaTransition이 항상 {valid:true} 반환 | 임의 전환 | {valid:true} | P1 | -- |
| TC-PD017 | 단계 | checkPhaseDeliverables가 숫자와 문자열 모두 수용 | `(1, 'feat')`와 `('plan', 'feat')` | 둘 다 유효한 결과 반환 | P1 | -- |

### 18.4 TC-PD: 상태 시스템 (24개 export, 선별)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PD018 | 상태 | getPdcaStatusPath가 .json으로 끝나는 경로 반환 | getPdcaStatusPath() | '.json' 또는 '.bkit-memory.json'으로 끝남 | P0 | -- |
| TC-PD019 | 상태 | readBkitMemory가 객체 또는 null 반환 | readBkitMemory() | typeof object 또는 null | P0 | -- |
| TC-PD020 | 상태 | writeBkitMemory/readBkitMemory 왕복 | 쓰기 후 읽기 | 동일 데이터 반환 | P0 | -- |
| TC-PD021 | 상태 | getArchivedFeatures가 배열 반환 | getArchivedFeatures() | 배열 (비어있을 수 있음) | P1 | -- |
| TC-PD022 | 상태 | enforceFeatureLimit max=50에서 크래시 없음 | enforceFeatureLimit(50) | 예외 없음 | P1 | -- |

---

## 19. 스킬 시스템 테스트 (gap-detector)

### 19.1 TC-SK: 27개 스킬 전체 존재 및 구조

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-SK001 | 스킬 | skills/pdca/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P0 | -- |
| TC-SK002 | 스킬 | skills/plan-plus/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P0 | -- |
| TC-SK003 | 스킬 | skills/starter/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P0 | -- |
| TC-SK004 | 스킬 | skills/dynamic/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P0 | -- |
| TC-SK005 | 스킬 | skills/enterprise/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P0 | -- |
| TC-SK006 | 스킬 | skills/development-pipeline/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P0 | -- |
| TC-SK007 | 스킬 | skills/code-review/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK008 | 스킬 | skills/zero-script-qa/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK009 | 스킬 | skills/bkit-rules/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P0 | -- |
| TC-SK010 | 스킬 | skills/bkit-templates/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK011 | 스킬 | skills/claude-code-learning/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK012 | 스킬 | skills/desktop-app/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK013 | 스킬 | skills/mobile-app/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK014 | 스킬 | skills/phase-1-schema/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK015 | 스킬 | skills/phase-2-convention/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK016 | 스킬 | skills/phase-3-mockup/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK017 | 스킬 | skills/phase-4-api/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK018 | 스킬 | skills/phase-5-design-system/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK019 | 스킬 | skills/phase-6-ui-integration/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK020 | 스킬 | skills/phase-7-seo-security/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK021 | 스킬 | skills/phase-8-review/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK022 | 스킬 | skills/phase-9-deployment/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK023 | 스킬 | skills/bkend-auth/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK024 | 스킬 | skills/bkend-cookbook/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK025 | 스킬 | skills/bkend-data/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK026 | 스킬 | skills/bkend-quickstart/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |
| TC-SK027 | 스킬 | skills/bkend-storage/SKILL.md 존재 | 파일 읽기 | 파일 존재 및 비어있지 않음 | P1 | -- |

### 19.2 TC-SK: 스킬 설명 유효성

27개 스킬 모두 `description: |` YAML 형식 사용 (CC v2.1.51 크래시 수정으로 안전). 위 존재 확인과 SKILL.md에 `#` 헤더 줄(Markdown 제목) 포함 여부로 검증.

---

## 20. 에이전트 시스템 테스트 (gap-detector)

### 20.1 TC-AG: 16개 에이전트 전체 존재 및 Frontmatter

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-AG001 | 에이전트 | agents/cto-lead.md 존재 및 유효한 frontmatter | 파일 읽기 | model: opus, mode: plan 또는 acceptEdits | P0 | -- |
| TC-AG002 | 에이전트 | agents/code-analyzer.md 존재 및 model: opus | 파일 읽기 | model: opus | P0 | -- |
| TC-AG003 | 에이전트 | agents/gap-detector.md 존재 및 model: opus | 파일 읽기 | model: opus | P0 | -- |
| TC-AG004 | 에이전트 | agents/pdca-iterator.md 존재 및 model: opus | 파일 읽기 | model: opus | P0 | -- |
| TC-AG005 | 에이전트 | agents/enterprise-expert.md 존재 및 model: opus | 파일 읽기 | model: opus | P0 | -- |
| TC-AG006 | 에이전트 | agents/frontend-architect.md 존재 및 model: opus | 파일 읽기 | model: opus | P0 | -- |
| TC-AG007 | 에이전트 | agents/security-architect.md 존재 및 model: opus | 파일 읽기 | model: opus | P0 | -- |
| TC-AG008 | 에이전트 | agents/bkend-expert.md 존재 및 model: sonnet | 파일 읽기 | model: sonnet | P0 | -- |
| TC-AG009 | 에이전트 | agents/design-validator.md 존재 및 model: sonnet | 파일 읽기 | model: sonnet | P0 | -- |
| TC-AG010 | 에이전트 | agents/product-manager.md 존재 및 model: sonnet | 파일 읽기 | model: sonnet | P0 | -- |
| TC-AG011 | 에이전트 | agents/qa-monitor.md 존재 및 model: sonnet | 파일 읽기 | model: sonnet | P0 | -- |
| TC-AG012 | 에이전트 | agents/qa-strategist.md 존재 및 model: sonnet | 파일 읽기 | model: sonnet | P0 | -- |
| TC-AG013 | 에이전트 | agents/report-generator.md 존재 및 model: sonnet | 파일 읽기 | model: sonnet | P0 | -- |
| TC-AG014 | 에이전트 | agents/infra-architect.md 존재 및 model: sonnet | 파일 읽기 | model: sonnet | P0 | -- |
| TC-AG015 | 에이전트 | agents/pipeline-guide.md 존재 및 model: haiku | 파일 읽기 | model: haiku | P0 | -- |
| TC-AG016 | 에이전트 | agents/starter-guide.md 존재 및 model: haiku | 파일 읽기 | model: haiku | P0 | -- |

### 20.2 TC-AG: 모델 분포 및 모드 검증

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-AG017 | 에이전트 | 총 모델 분포: 7 opus, 7 sonnet, 2 haiku | 16개 에이전트 모두 읽기 | 정확히 7/7/2 | P0 | -- |
| TC-AG018 | 에이전트 | 모드 분포: 9 acceptEdits, 7 plan | 16개 에이전트 모두 읽기 | 9 acceptEdits + 7 plan = 16 | P0 | -- |
| TC-AG019 | 에이전트 | 메모리 범위: 14 project, 2 user (starter-guide, pipeline-guide) | 16개 에이전트 모두 읽기 | 14 project + 2 user = 16 | P1 | -- |

---

## 21. 설정 시스템 테스트 (gap-detector)

### 21.1 TC-CF: 설정 파일

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CF001 | 설정 | bkit.config.json 존재 및 유효한 JSON | 읽기 + 파싱 | JSON 파싱 에러 없음 | P0 | -- |
| TC-CF002 | 설정 | bkit.config.json version이 '1.5.9'와 일치 | JSON 파싱 | version === '1.5.9' | P0 | -- |
| TC-CF003 | 설정 | bkit.config.json에 matchRateThreshold: 90인 pdca 섹션 | JSON 파싱 | pdca.matchRateThreshold === 90 | P0 | -- |
| TC-CF004 | 설정 | bkit.config.json pdca.automationLevel이 'semi-auto' | JSON 파싱 | pdca.automationLevel === 'semi-auto' | P0 | -- |
| TC-CF005 | 설정 | bkit.config.json pdca.maxIterations가 5 | JSON 파싱 | pdca.maxIterations === 5 | P0 | -- |
| TC-CF006 | 설정 | bkit.config.json team.enabled가 true | JSON 파싱 | team.enabled === true | P0 | -- |
| TC-CF007 | 설정 | bkit.config.json team.ctoAgent가 'cto-lead' | JSON 파싱 | team.ctoAgent === 'cto-lead' | P1 | -- |
| TC-CF008 | 설정 | bkit.config.json에 triggers 섹션 | JSON 파싱 | triggers.implicitEnabled, triggers.confidenceThreshold 보유 | P1 | -- |
| TC-CF009 | 설정 | plugin.json 존재 및 유효한 JSON | .claude-plugin/plugin.json 읽기 | 파싱 에러 없음 | P0 | -- |
| TC-CF010 | 설정 | plugin.json name이 'bkit'이고 version이 '1.5.9' | JSON 파싱 | name==='bkit', version==='1.5.9' | P0 | -- |
| TC-CF011 | 설정 | plugin.json에 outputStyles 필드 | JSON 파싱 | outputStyles === './output-styles/' | P0 | -- |
| TC-CF012 | 설정 | marketplace.json 존재 및 유효한 JSON | .claude-plugin/marketplace.json 읽기 | 파싱 에러 없음 | P1 | -- |

---

## 22. 템플릿 시스템 테스트 (gap-detector)

### 22.1 TC-TP: 코어 템플릿 (6개 파일)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TP001 | 템플릿 | plan.template.md 존재 및 ## 섹션 보유 | 파일 읽기 | '## ' 헤더 포함 | P0 | -- |
| TC-TP002 | 템플릿 | plan.template.md에 Executive Summary 섹션 | 파일 읽기 | '## Executive Summary' 포함 | P0 | -- |
| TC-TP003 | 템플릿 | design.template.md 존재 및 ## 섹션 보유 | 파일 읽기 | '## ' 헤더 포함 | P0 | -- |
| TC-TP004 | 템플릿 | analysis.template.md 존재 | 파일 읽기 | 파일 존재 | P0 | -- |
| TC-TP005 | 템플릿 | report.template.md 존재 및 Value Delivered 보유 | 파일 읽기 | 'Value Delivered' 포함 | P0 | -- |
| TC-TP006 | 템플릿 | do.template.md 존재 | 파일 읽기 | 파일 존재 | P0 | -- |
| TC-TP007 | 템플릿 | plan-plus.template.md에 Executive Summary | 파일 읽기 | '## Executive Summary' 포함 | P0 | -- |

### 22.2 TC-TP: 파이프라인 템플릿 (10개 파일)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TP008 | 템플릿 | pipeline/phase-1-schema.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP009 | 템플릿 | pipeline/phase-2-convention.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP010 | 템플릿 | pipeline/phase-3-mockup.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP011 | 템플릿 | pipeline/phase-4-api.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP012 | 템플릿 | pipeline/phase-5-design-system.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP013 | 템플릿 | pipeline/phase-6-ui.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP014 | 템플릿 | pipeline/phase-7-seo-security.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP015 | 템플릿 | pipeline/phase-8-review.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP016 | 템플릿 | pipeline/phase-9-deployment.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP017 | 템플릿 | pipeline/zero-script-qa.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |

### 22.3 TC-TP: 공유 템플릿 (4개 파일)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TP018 | 템플릿 | shared/api-patterns.md 존재 | 파일 읽기 | 파일 존재 | P2 | -- |
| TC-TP019 | 템플릿 | shared/bkend-patterns.md 존재 | 파일 읽기 | 파일 존재 | P2 | -- |
| TC-TP020 | 템플릿 | shared/error-handling-patterns.md 존재 | 파일 읽기 | 파일 존재 | P2 | -- |
| TC-TP021 | 템플릿 | shared/naming-conventions.md 존재 | 파일 읽기 | 파일 존재 | P2 | -- |

### 22.4 TC-TP: 기타 템플릿

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-TP022 | 템플릿 | CLAUDE.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP023 | 템플릿 | convention.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP024 | 템플릿 | schema.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP025 | 템플릿 | iteration-report.template.md 존재 | 파일 읽기 | 파일 존재 | P1 | -- |
| TC-TP026 | 템플릿 | _INDEX.template.md 존재 | 파일 읽기 | 파일 존재 | P2 | -- |

---

## 23. 출력 스타일 테스트 (qa-monitor)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-OS001 | 출력스타일 | output-styles/bkit-learning.md 존재 | 파일 읽기 | 파일 존재, 비어있지 않음 | P0 | -- |
| TC-OS002 | 출력스타일 | output-styles/bkit-pdca-guide.md 존재 | 파일 읽기 | 파일 존재, 비어있지 않음 | P0 | -- |
| TC-OS003 | 출력스타일 | output-styles/bkit-enterprise.md 존재 | 파일 읽기 | 파일 존재, 비어있지 않음 | P0 | -- |
| TC-OS004 | 출력스타일 | output-styles/bkit-pdca-enterprise.md 존재 | 파일 읽기 | 파일 존재, 비어있지 않음 | P0 | -- |
| TC-OS005 | 출력스타일 | 4개 스타일 모두 유효한 Markdown (깨진 구문 없음) | 파일 파싱 | Markdown 린트 에러 없음 | P1 | -- |
| TC-OS006 | 출력스타일 | plugin.json outputStyles가 올바른 디렉토리 지정 | plugin.json 확인 | outputStyles === './output-styles/' 및 디렉토리 존재 | P1 | -- |

---

## 24. PDCA 레벨 E2E 워크플로 테스트 (product-manager)

### 24.1 TC-LV: Starter 레벨

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-LV001 | 레벨-E2E | Starter 레벨에서 design 단계 건너뛰기 가능 | canSkipPhase('Starter','design') | true | P0 | -- |
| TC-LV002 | 레벨-E2E | Starter 레벨 getRequiredPhases가 부분 집합 반환 | getRequiredPhases('Starter') | Enterprise보다 적은 단계 | P0 | -- |
| TC-LV003 | 레벨-E2E | Starter 레벨에서 팀 모드 사용 불가 | generateTeamStrategy('Starter') | null 또는 에러 표시 | P0 | -- |
| TC-LV004 | 레벨-E2E | Starter 기본 에이전트가 starter-guide | 설정/레벨 확인 | starter-guide 제안됨 | P1 | -- |

### 24.2 TC-LV: Dynamic 레벨

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-LV005 | 레벨-E2E | Dynamic 레벨 최대 3팀원 | 팀 설정 Dynamic | maxTeammates <= 3 | P0 | -- |
| TC-LV006 | 레벨-E2E | Dynamic 오케스트레이션: plan=leader, do=swarm, check=council | 설정 확인 | orchestrationPatterns.Dynamic과 일치 | P0 | -- |
| TC-LV007 | 레벨-E2E | Dynamic getRequiredPhases에 do와 check 포함 | getRequiredPhases('Dynamic') | 'do'와 'check' 포함 | P1 | -- |

### 24.3 TC-LV: Enterprise 레벨

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-LV008 | 레벨-E2E | Enterprise 레벨 최대 5팀원 | 팀 설정 Enterprise | maxTeammates <= 5 | P0 | -- |
| TC-LV009 | 레벨-E2E | Enterprise 오케스트레이션: design=council, act=watchdog | 설정 확인 | orchestrationPatterns.Enterprise와 일치 | P0 | -- |
| TC-LV010 | 레벨-E2E | Enterprise getRequiredPhases가 6개 단계 모두 반환 | getRequiredPhases('Enterprise') | 배열 길이 >= 6 | P0 | -- |
| TC-LV011 | 레벨-E2E | Enterprise 기본 에이전트가 enterprise-expert | 설정/레벨 확인 | enterprise-expert 제안됨 | P1 | -- |
| TC-LV012 | 레벨-E2E | Enterprise가 9단계 파이프라인 지원 | 파이프라인 설정 | 9개 단계 모두 사용 가능 | P2 | -- |

---

## 25. i18n (8개 언어) 테스트 (product-manager)

### 25.1 TC-L10N: 언어 감지 정확도

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-L10N001 | i18n | 기술 텍스트 영어 감지 | `('Please implement the login feature')` | 'en' | P0 | -- |
| TC-L10N002 | i18n | 혼합 한글 한국어 감지 | `('로그인 기능을 구현해주세요')` | 'ko' | P0 | -- |
| TC-L10N003 | i18n | 한자+히라가나 일본어 감지 | `('ログイン機能を実装してください')` | 'ja' | P0 | -- |
| TC-L10N004 | i18n | 간체 중국어 감지 | `('请实现登录功能')` | 'zh' | P0 | -- |
| TC-L10N005 | i18n | 스페인어 감지 | `('Por favor implementar la funcion de inicio de sesion')` | 'es' | P1 | -- |
| TC-L10N006 | i18n | 프랑스어 감지 | `('Veuillez implementer la fonction de connexion')` | 'fr' | P1 | -- |
| TC-L10N007 | i18n | 독일어 감지 | `('Bitte implementieren Sie die Login-Funktion')` | 'de' | P1 | -- |
| TC-L10N008 | i18n | 이탈리아어 감지 | `('Per favore implementare la funzione di login')` | 'it' | P1 | -- |

### 25.2 TC-L10N: 언어별 에이전트 트리거 키워드

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-L10N009 | i18n | 한국어 '검증'이 gap-detector 트리거 | `('검증해줘')` | agent: 'bkit:gap-detector' | P0 | -- |
| TC-L10N010 | i18n | 일본어 '改善'이 pdca-iterator 트리거 | `('改善して')` | agent: 'bkit:pdca-iterator' | P0 | -- |
| TC-L10N011 | i18n | 중국어 '分析'이 code-analyzer 트리거 | `('分析代码')` | agent: 'bkit:code-analyzer' | P0 | -- |
| TC-L10N012 | i18n | 스페인어 'verificar'가 gap-detector 트리거 | `('verificar el codigo')` | agent: 'bkit:gap-detector' | P1 | -- |
| TC-L10N013 | i18n | 프랑스어 'rapport'가 report-generator 트리거 | `('generer un rapport')` | agent: 'bkit:report-generator' | P1 | -- |
| TC-L10N014 | i18n | 독일어 'prufen'이 gap-detector 트리거 | `('Code prufen')` | agent: 'bkit:gap-detector' | P1 | -- |

### 25.3 TC-L10N: 언어별 스킬 트리거 키워드

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-L10N015 | i18n | 한국어 '새 기능'이 NEW_FEATURE_PATTERNS 매칭 | matchMultiLangPattern('새 기능 추가', NEW_FEATURE_PATTERNS) | true | P0 | -- |
| TC-L10N016 | i18n | 일본어 '新機能'이 NEW_FEATURE_PATTERNS 매칭 | matchMultiLangPattern('新機能を作成', NEW_FEATURE_PATTERNS) | true | P1 | -- |

---

## 26. 메모리 시스템 테스트 (product-manager)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-MEM001 | 메모리 | readBkitMemory가 .bkit-memory.json 또는 docs/.bkit-memory.json에서 객체 반환 | readBkitMemory() | typeof object 또는 null | P0 | -- |
| TC-MEM002 | 메모리 | writeBkitMemory가 디스크에 데이터 영속 | writeBkitMemory({test:1}), readBkitMemory() | {test:1} 포함 | P0 | -- |
| TC-MEM003 | 메모리 | bkit 메모리 경로가 CC 자동 메모리 경로와 겹치지 않음 | STATE_PATHS 확인 | bkit 경로 != ~/.claude/projects/*/memory/MEMORY.md | P0 | -- |
| TC-MEM004 | 메모리 | 에이전트 메모리 경로 .claude/agent-memory/{agent}/MEMORY.md 구조 | ls .claude/agent-memory/ | 메모리가 있는 각 에이전트의 디렉토리 | P1 | -- |
| TC-MEM005 | 메모리 | bkit memory-store가 JSON 형식 사용 (Markdown 아님) | readBkitMemory() | 파싱된 JSON 반환, 원시 Markdown 아님 | P0 | -- |
| TC-MEM006 | 메모리 | LEGACY_PATHS 마이그레이션: .pdca-status.json -> docs/.bkit-memory.json | paths 모듈 확인 | 레거시 경로가 현재 경로와 다름 | P1 | -- |
| TC-MEM007 | 메모리 | 메모리 시스템이 누락 파일을 우아하게 처리 | 상태 파일 삭제 후 readBkitMemory() | null 또는 빈 객체 (크래시 없음) | P0 | -- |
| TC-MEM008 | 메모리 | 메모리 시스템이 손상된 JSON 처리 | 잘못된 JSON을 파일에 쓰기 후 readBkitMemory() | null 또는 빈 (크래시 없음) | P1 | -- |
| TC-MEM009 | 메모리 | 에이전트 메모리 범위: project 에이전트가 .claude/agent-memory/ 사용 | 에이전트 frontmatter 읽기 | 14개 에이전트가 project 범위 사용 | P1 | -- |
| TC-MEM010 | 메모리 | 에이전트 메모리 범위: user 에이전트가 ~/.claude/agent-memory/ 사용 | starter-guide, pipeline-guide 읽기 | 2개 에이전트가 user 범위 사용 | P1 | -- |
| TC-MEM011 | 메모리 | 3개 메모리 시스템의 경로 충돌 없음 | 3개 경로 모두 비교 | bkit-memory.json, agent-memory/MEMORY.md, CC 자동 메모리 모두 구별됨 | P0 | -- |

---

## 27. 스크립트 시스템 테스트 (qa-monitor)

### 27.1 TC-SC: 훅 스크립트 (hooks.json에 참조된 13개 스크립트)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-SC001 | 스크립트 | hooks/session-start.js 존재 및 require 에러 없음 | node -e "require('./hooks/session-start.js')" | require 에러 없음 | P0 | -- |
| TC-SC002 | 스크립트 | scripts/pre-write.js 존재 및 require 에러 없음 | require 확인 | require 에러 없음 | P0 | -- |
| TC-SC003 | 스크립트 | scripts/unified-bash-pre.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC004 | 스크립트 | scripts/unified-write-post.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC005 | 스크립트 | scripts/unified-bash-post.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC006 | 스크립트 | scripts/skill-post.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC007 | 스크립트 | scripts/unified-stop.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC008 | 스크립트 | scripts/user-prompt-handler.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC009 | 스크립트 | scripts/context-compaction.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC010 | 스크립트 | scripts/pdca-task-completed.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC011 | 스크립트 | scripts/subagent-start-handler.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC012 | 스크립트 | scripts/subagent-stop-handler.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC013 | 스크립트 | scripts/team-idle-handler.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |

### 27.2 TC-SC: 에이전트 Stop 스크립트 (에이전트별)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-SC014 | 스크립트 | scripts/gap-detector-stop.js 존재 | 파일 확인 | 파일 존재 | P0 | -- |
| TC-SC015 | 스크립트 | scripts/cto-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC016 | 스크립트 | scripts/iterator-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC017 | 스크립트 | scripts/code-review-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC018 | 스크립트 | scripts/learning-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC019 | 스크립트 | scripts/qa-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC020 | 스크립트 | scripts/analysis-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC021 | 스크립트 | scripts/team-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |

### 27.3 TC-SC: PDCA/파이프라인 단계 스크립트

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-SC022 | 스크립트 | scripts/pdca-post-write.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC023 | 스크립트 | scripts/pdca-skill-stop.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC024 | 스크립트 | scripts/phase-transition.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC025 | 스크립트 | scripts/phase1-schema-stop.js 존재 | 파일 확인 | 파일 존재 | P2 | -- |
| TC-SC026 | 스크립트 | scripts/phase2-convention-pre.js 존재 | 파일 확인 | 파일 존재 | P2 | -- |
| TC-SC027 | 스크립트 | scripts/phase2-convention-stop.js 존재 | 파일 확인 | 파일 존재 | P2 | -- |
| TC-SC028 | 스크립트 | scripts/phase3-mockup-stop.js ~ phase9-deploy-stop.js 모두 존재 | 파일 확인 | 7개 파일 모두 존재 | P2 | -- |

### 27.4 TC-SC: 유틸리티 스크립트

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-SC029 | 스크립트 | scripts/archive-feature.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC030 | 스크립트 | scripts/select-template.js 존재 | 파일 확인 | 파일 존재 | P1 | -- |
| TC-SC031 | 스크립트 | scripts/sync-folders.js 존재 | 파일 확인 | 파일 존재 | P2 | -- |
| TC-SC032 | 스크립트 | scripts/validate-plugin.js 존재 | 파일 확인 | 파일 존재 | P2 | -- |

---

## 28. 갱신된 테스트 실행 계획

### 28.1 전체 실행 우선순위

```
1단계 (P0 크리티컬 - v1.5.9 변경):
  TC-H001~H006, TC-I001~I006, TC-U001~U037   (기존 49 TC)

2단계 (P0 통합 + E2E):
  TC-H007~H032, TC-I007~I020, TC-E001~E013    (기존 46 TC)

3단계 (P0 전체 커버리지):
  TC-CS001~CS009 (플랫폼)             -- 9 TC
  TC-CS010~CS013 (캐시)               -- 4 TC
  TC-CS017~CS022 (I/O)               -- 6 TC
  TC-AG001~AG018 (에이전트)           -- 18 TC
  TC-CF001~CF011 (설정)              -- 11 TC
  TC-SK001~SK005, SK009 (코어 스킬)   -- 6 TC
  TC-PD001~PD002, PD007~PD012 (PDCA) -- 8 TC
  TC-TM001~TM004, TM006~TM010 (팀)   -- 9 TC
  TC-SC001~SC014 (스크립트)           -- 14 TC
  TC-LV001~LV003, LV005~LV006, LV008~LV010 (레벨) -- 8 TC
  TC-IN001~IN002, IN011~IN012, IN015, IN017~IN020, IN025~IN026, IN028~IN030 (인텐트) -- 13 TC
  TC-MEM001~MEM003, MEM005, MEM007, MEM011 (메모리) -- 6 TC
  TC-OS001~OS004 (출력 스타일)        -- 4 TC
  TC-TP001~TP007 (코어 템플릿)        -- 7 TC

4단계 (P1 품질):
  TC-X001~X020 (UX)                  -- 20 TC
  TC-R001~R015 (회귀)                -- 15 TC
  TC-CS014~CS016, CS023~CS046 (코어 P1) -- 22 TC
  TC-IN003~IN010, IN013~IN014, IN016, IN021~IN024, IN027, IN031~IN032 (인텐트 P1) -- 18 TC
  TC-TS006~TS007, TS013, TS016~TS018, TS020~TS021 (태스크 P1) -- 8 TC
  TC-TM005, TM011~TM019, TM022, TM026~TM027 (팀 P1) -- 12 TC
  TC-PD003, PD005~PD006, PD015~PD017, PD021~PD022 (PDCA P1) -- 8 TC
  TC-AG019 (에이전트 메모리 P1)       -- 1 TC
  TC-SK007~SK008, SK010~SK027 (스킬 P1) -- 20 TC
  TC-TP008~TP017, TP022~TP025 (템플릿 P1) -- 14 TC
  TC-SC015~SC024, SC029~SC030 (스크립트 P1) -- 12 TC
  TC-L10N005~L10N008, L10N012~L10N014, L10N016 (i18n P1) -- 8 TC
  TC-MEM004, MEM006, MEM008~MEM010 (메모리 P1) -- 5 TC
  TC-OS005~OS006, TC-LV004, LV007, LV011 (기타 P1) -- 5 TC

5단계 (P2 강화):
  TC-EC001~EC010 (엣지 케이스)        -- 10 TC
  TC-P001~P005 (성능)                -- 5 TC
  TC-E014~E015 (P1 E2E)             -- 2 TC
  TC-TP018~TP021, TP026 (템플릿 P2)  -- 5 TC
  TC-SC025~SC028, SC031~SC032 (스크립트 P2) -- 6 TC
  TC-LV012 (파이프라인 P2)           -- 1 TC
```

### 28.2 갱신된 에이전트 태스크 배정

| 에이전트 | 테스트 카테고리 | TC 수 | 예상 소요시간 |
|----------|----------------|:-----:|:------------:|
| code-analyzer | TC-U, TC-EC, TC-CS, TC-TS, TC-TM, TC-PD | 54 + 46 + 21 + 27 + 22 = **170** | 6-8시간 |
| qa-monitor | TC-H, TC-P, TC-OS, TC-SC | 37 + 6 + 32 = **75** | 3-4시간 |
| gap-detector | TC-I, TC-E, TC-SK, TC-AG, TC-CF, TC-TP | 35 + 27 + 19 + 12 + 26 = **119** | 5-6시간 |
| product-manager | TC-X, TC-IN, TC-LV, TC-L10N, TC-MEM | 20 + 26 + 12 + 16 + 11 = **85** | 4-5시간 |
| qa-strategist | TC-R, 조율, 최종 통합 | 15 + 조율 = **15+** | 2-3시간 |

### 28.3 갱신된 성공 기준

| 카테고리 | 목표 | 최소 |
|----------|:----:|:----:|
| P0 테스트 (268) | 100% | 100% |
| P1 테스트 (167) | 100% | 95% |
| P2 테스트 (35) | 95% | 90% |
| 전체 (470) | 100% | 97% |

---

## 29. 핵심 철학 테스트 (TC-PH001~PH017)

> **출처**: `bkit-system/philosophy/core-mission.md` -- 3가지 핵심 철학 (자동화 우선, 추측 금지, 문서=코드)

### 29.1 TC-PH: 자동화 우선 철학

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PH001 | 철학 | SessionStart 훅이 PDCA 상태 + 온보딩 메시지 자동 제공 | 새 세션 시작 | 사용자 명령 없이 PDCA 상태와 4개 옵션 표시 | P0 | -- |
| TC-PH002 | 철학 | 자연어("새 기능 만들어줘")가 PDCA 자동 적용 트리거 | "로그인 기능 만들어줘" (/pdca 명령 없이) | PDCA plan 제안 자동 트리거 | P0 | -- |
| TC-PH003 | 철학 | 레벨 자동 감지가 적절한 출력 스타일 제안 | services/ + infra/가 있는 프로젝트 | Enterprise 레벨 감지, bkit-enterprise 제안 | P1 | -- |
| TC-PH004 | 철학 | 대규모 기능 감지 시 CTO 팀 자동 제안 | Dynamic/Enterprise에서 >= 1000자 입력 | "Agent Teams로 PDCA 단계 병렬화 가능" 제안 | P1 | -- |
| TC-PH005 | 철학 | 에이전트 메모리가 사용자 개입 없이 작동 | 2개 세션에서 gap-detector 호출 | 두 번째 세션이 첫 번째 세션의 컨텍스트 참조 | P1 | -- |
| TC-PH006 | 철학 | 갭 분석 완료 시 다음 단계 자동 제안 | gap-detector가 matchRate=92%로 완료 | 보고서 생성 자동 제안 | P0 | -- |
| TC-PH007 | 철학 | matchRate에 따른 Check-Act 반복 루프 자동화 | matchRate=75% 후 gap-detector | pdca-iterator 자동 제안, 재검증 사이클 시작 | P0 | -- |

### 29.2 TC-PH: 추측 금지 철학

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PH008 | 철학 | Plan 문서 부재 시 Plan 우선 제안 트리거 | "인증 기능 구현해줘" (plan 없음) | "Plan 문서를 찾을 수 없습니다. 먼저 생성하시겠습니까?" | P0 | -- |
| TC-PH009 | 철학 | Design 문서 부재 시 Design 우선 제안 트리거 | "인증 코드 작성해줘" (design 없음) | "Design 문서를 찾을 수 없습니다. 먼저 생성하시겠습니까?" | P0 | -- |
| TC-PH010 | 철학 | 모호한 입력(점수 >= 50) 시 AskUserQuestion 트리거 | 구체적 명사/경로 없는 모호한 입력 | 명확화 질문이 있는 AskUserQuestion | P0 | -- |
| TC-PH011 | 철학 | MCP 도구명을 번호 목록이 아닌 정확한 이름으로 제공 | 출력의 bkend MCP 도구 참조 | 정확한 도구명 (예: "supabase_list_tables"), "도구 #3" 아님 | P1 | -- |
| TC-PH012 | 철학 | 불확실한 상황에서 문서-확인-후-질문 순서 따름 | 알 수 없는 기능에 대한 모호한 요청 | 먼저 문서 확인, 찾지 못하면 사용자에게 질문 | P1 | -- |

### 29.3 TC-PH: 문서=코드 철학

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PH013 | 철학 | 구현 전에 Design 문서 생성/참조 | 새 기능 요청 | 코드 작성 전에 Design 문서 생성/참조 | P0 | -- |
| TC-PH014 | 철학 | gap-detector가 설계-구현 동기화 검증 | Design 문서 + 구현 코드 | 구체적 불일치가 식별된 갭 목록 | P0 | -- |
| TC-PH015 | 철학 | 일치율이 설계-구현 정합성을 정확히 반영 | 10개 항목의 설계, 8개 구현 | matchRate ~80% (100%나 무작위 아님) | P0 | -- |
| TC-PH016 | 철학 | 보고서가 PDCA 사이클 교훈을 문서화 | 완료된 사이클 후 /pdca report | 보고서에 교훈, 개선 섹션 포함 | P1 | -- |
| TC-PH017 | 철학 | 아카이브가 4가지 문서 유형 모두 보존 | 보고서 후 /pdca archive | plan.md, design.md, analysis.md, report.md 모두 아카이브에 | P1 | -- |

---

## 30. 컨텍스트 엔지니어링 FR 테스트 (TC-CE001~CE031)

> **출처**: `bkit-system/philosophy/context-engineering.md` -- 8개 기능 요구사항 (FR-01~FR-08)

### 30.1 TC-CE: FR-01 다단계 컨텍스트 계층

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE001 | 컨텍스트 | 4단계 계층(L1 플러그인 -> L2 사용자 -> L3 프로젝트 -> L4 세션) 우선순위 | 4단계 모두에 설정 | L4 값이 L1/L2/L3보다 우선 | P0 | -- |
| TC-CE002 | 컨텍스트 | L4 세션이 L1 플러그인 설정을 올바르게 오버라이드 | L1: matchThreshold=80, L4: matchThreshold=95 | getHierarchicalConfig가 95 반환 | P0 | -- |
| TC-CE003 | 컨텍스트 | getContextHierarchy()가 4단계 병합 결과 반환 | 4개 설정 파일 모두 존재 | 모든 레벨이 병합된 객체 | P0 | -- |
| TC-CE004 | 컨텍스트 | getHierarchicalConfig(keyPath) 점 표기법 접근 | "pdca.matchThreshold" | 올바른 중첩 값 반환 | P1 | -- |
| TC-CE005 | 컨텍스트 | setSessionContext()가 L4 런타임 값 설정 | setSessionContext("level", "Enterprise") | 이후 getHierarchicalConfig가 "Enterprise" 반환 | P1 | -- |

### 30.2 TC-CE: FR-02 @import 지시문

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE006 | 컨텍스트 | SKILL.md @import가 외부 파일 내용 로드 | @import ./shared/api-patterns.md | api-patterns.md의 내용이 포함됨 | P0 | -- |
| TC-CE007 | 컨텍스트 | ${PLUGIN_ROOT} 변수가 올바르게 치환 | ${PLUGIN_ROOT}/templates/plan.template.md | 실제 플러그인 경로로 해석 | P0 | -- |
| TC-CE008 | 컨텍스트 | ${PROJECT} 변수가 현재 프로젝트 디렉토리로 해석 | ${PROJECT}/conventions.md | 실제 프로젝트 디렉토리로 해석 | P1 | -- |
| TC-CE009 | 컨텍스트 | 순환 의존성 감지 및 에러 반환 | A가 B를 import, B가 A를 import | 에러 반환 (무한 루프 아님) | P1 | -- |
| TC-CE010 | 컨텍스트 | TTL 기반 캐싱이 반복 import에 작동 | TTL 내에 동일 import 2회 호출 | 두 번째 호출이 캐시 결과 반환 | P2 | -- |

### 30.3 TC-CE: FR-03 컨텍스트 Fork 격리

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE011 | 컨텍스트 | forkContext()가 독립적 깊은 복제 생성 | forkContext('test-skill') | 반환된 컨텍스트가 부모와 분리 | P0 | -- |
| TC-CE012 | 컨텍스트 | mergeForkedContext()가 결과를 부모에 병합 (배열 중복 제거) | 배열 항목이 추가된 fork | 부모가 병합된, 중복 제거된 배열 수신 | P1 | -- |
| TC-CE013 | 컨텍스트 | discardFork()가 병합 없이 폐기 | discardFork(forkId) | 부모 컨텍스트 변경 없음 | P1 | -- |
| TC-CE014 | 컨텍스트 | gap-detector가 fork 모드로 작동 (context:fork, mergeResult:false) | gap-detector 에이전트 frontmatter | context: fork, mergeResult: false가 frontmatter에 존재 | P0 | -- |

### 30.4 TC-CE: FR-04 UserPromptSubmit 훅

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE015 | 컨텍스트 | user-prompt-handler.js가 기능 의도 감지 | "새로운 로그인 기능 만들어줘" | 출력에 기능 의도 감지됨 | P0 | -- |
| TC-CE016 | 컨텍스트 | 에이전트 트리거 감지가 8개 언어 모두에서 작동 | KO/EN/JA/ZH/ES/FR/DE/PT의 트리거 키워드 | 8개 언어 모두 올바르게 트리거 | P0 | -- |
| TC-CE017 | 컨텍스트 | 스킬 트리거 감지가 올바르게 작동 | 스킬 관련 키워드 | 스킬 트리거 감지됨 | P1 | -- |
| TC-CE018 | 컨텍스트 | 모호성 점수 계산이 0-1 범위 | 다양한 입력 텍스트 | 점수가 항상 0.0~1.0 사이 | P0 | -- |

### 30.5 TC-CE: FR-05 권한 계층

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE019 | 컨텍스트 | deny/ask/allow 3단계 권한 작동 | 3단계 모두 있는 권한 설정 | 각 단계가 올바르게 동작 | P0 | -- |
| TC-CE020 | 컨텍스트 | Glob 패턴 매칭 (예: Bash(rm -rf*)) 작동 | "Bash(rm -rf /tmp)"를 "Bash(rm -rf*)"에 대해 | 패턴 매칭, deny 적용 | P0 | -- |
| TC-CE021 | 컨텍스트 | deny 권한이 도구 실행 차단 (종료 코드 2) | Write 도구에 deny 설정 | 종료 코드 2로 도구 차단 | P0 | -- |

### 30.6 TC-CE: FR-06 태스크 의존성 체인

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE022 | 컨텍스트 | 태스크 분류 줄 기반 정확도 (10/50/200 임계값) | 5, 30, 150, 250줄 변경 | quick_fix, minor_change, feature, major_feature | P0 | -- |
| TC-CE023 | 컨텍스트 | major_feature가 blockedBy 자동 설정 | major_feature로 분류 | blockedBy 자동 채움 | P1 | -- |
| TC-CE024 | 컨텍스트 | createPdcaTaskChain()이 올바른 태스크 순서 생성 | 기능명 "auth" | Plan -> Design -> Do -> Check -> Act -> Report 체인 | P1 | -- |

### 30.7 TC-CE: FR-07 컨텍스트 압축 훅

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE025 | 컨텍스트 | PreCompact 훅이 PDCA 상태 스냅샷 저장 | PreCompact 이벤트 트리거 | docs/.pdca-snapshots/에 스냅샷 파일 생성 | P0 | -- |
| TC-CE026 | 컨텍스트 | 최근 10개 스냅샷만 유지 (자동 정리) | 12개 스냅샷 존재 | 가장 오래된 2개 삭제, 10개 남음 | P1 | -- |
| TC-CE027 | 컨텍스트 | context-compaction.js 출력에 충분한 복원 정보 포함 | 활성 PDCA가 있는 PreCompact | 출력에 feature, phase, matchRate, iterationCount 포함 | P1 | -- |

### 30.8 TC-CE: FR-08 MEMORY 변수 지원

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-CE028 | 컨텍스트 | setMemory/getMemory 키-값 저장/조회 | setMemory('key', 'value'), getMemory('key') | 'value' 반환 | P0 | -- |
| TC-CE029 | 컨텍스트 | deleteMemory가 특정 키 제거 | deleteMemory('key') | getMemory('key')가 null 반환 | P1 | -- |
| TC-CE030 | 컨텍스트 | clearMemory가 전체 저장소 초기화 | 3개 키 설정 후 clearMemory() | getAllMemory()가 빈 객체 반환 | P1 | -- |
| TC-CE031 | 컨텍스트 | 세션 간 영속성 .bkit-memory.json (또는 지정 경로) | 세션 1에서 setMemory, 세션 2에서 getMemory | 값이 세션 간 영속 | P0 | -- |

---

## 31. 6계층 훅 시스템 테스트 (TC-HL001~HL007)

> **출처**: `bkit-system/philosophy/context-engineering.md` -- 6계층 훅 시스템 (10개 이벤트)

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-HL001 | 훅계층 | 계층 1 (hooks.json 전역) - 10개 이벤트 올바르게 등록 | hooks.json 읽기 | 13개 항목에 걸친 10개 고유 이벤트 | P0 | -- |
| TC-HL002 | 훅계층 | 계층 2 (스킬 Frontmatter) - PDCA 스킬 agents 필드가 에이전트에 액션 라우팅 | /pdca analyze | agents.analyze를 통해 gap-detector 에이전트 호출 | P0 | -- |
| TC-HL003 | 훅계층 | 계층 3 (에이전트 Frontmatter) - gap-detector context:fork 설정 적용 | gap-detector 에이전트 frontmatter | context: fork가 frontmatter에 존재 | P0 | -- |
| TC-HL004 | 훅계층 | 계층 4 (설명 트리거) - 8개 언어 키워드 매칭 | 각 언어의 트리거 키워드 | 언어별 올바른 에이전트/스킬 트리거 | P0 | -- |
| TC-HL005 | 훅계층 | 계층 5 (스크립트) - 45개 스크립트 모두 require 에러 없이 로드 | 45개 스크립트 각각 require() | 에러 없음, 모든 모듈 로드 | P0 | -- |
| TC-HL006 | 훅계층 | 계층 6 (팀 오케스트레이션) - CTO 주도 단계 라우팅이 올바른 패턴 선택 | 단계 + 레벨 입력 | leader/council/swarm/watchdog 올바르게 선택 | P1 | -- |
| TC-HL007 | 훅계층 | 6계층이 올바른 순서로 컨텍스트 주입 | 모든 계층이 활성인 전체 세션 | 계층 1이 먼저, 계층 6이 마지막 실행 순서 | P1 | -- |

---

## 32. 동적 컨텍스트 주입 패턴 테스트 (TC-DI001~DI018)

> **출처**: `bkit-system/philosophy/context-engineering.md` -- 동적 컨텍스트 주입 패턴 (4가지 패턴)

### 32.1 TC-DI: 패턴 1 - 태스크 크기에서 PDCA 레벨로

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-DI001 | 동적 | <10줄 변경 -> quick_fix (PDCA 미적용) | 5줄 변경 | classification = quick_fix, PDCA 없음 | P0 | -- |
| TC-DI002 | 동적 | <50줄 변경 -> minor_change (PDCA 권장) | 30줄 변경 | classification = minor_change, PDCA 제안 | P0 | -- |
| TC-DI003 | 동적 | <200줄 변경 -> feature (PDCA 필수) | 150줄 변경 | classification = feature, PDCA 강제 | P0 | -- |
| TC-DI004 | 동적 | >=200줄 변경 -> major_feature (PDCA + 분할 권장) | 250줄 변경 | classification = major_feature, 분할 제안 | P0 | -- |

### 32.2 TC-DI: 패턴 2 - 사용자 의도에서 에이전트/스킬 자동 트리거로

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-DI005 | 동적 | 한국어 "맞아?"가 gap-detector 트리거 | 사용자 입력: "맞아?" | gap-detector 에이전트 트리거 | P0 | -- |
| TC-DI006 | 동적 | 한국어 "고쳐줘"가 pdca-iterator 트리거 | 사용자 입력: "고쳐줘" | pdca-iterator 에이전트 트리거 | P0 | -- |
| TC-DI007 | 동적 | 영어 "is this right?"가 gap-detector 트리거 | 사용자 입력: "is this right?" | gap-detector 에이전트 트리거 | P0 | -- |
| TC-DI008 | 동적 | 영어 "make it better"가 pdca-iterator 트리거 | 사용자 입력: "make it better" | pdca-iterator 에이전트 트리거 | P1 | -- |

### 32.3 TC-DI: 패턴 3 - 모호성 점수에서 명확화 질문으로

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-DI009 | 동적 | 구체적 명사 없음 -> +20 점수 가산 | 구체적 명사 없는 입력 | 점수 ~20 증가 | P0 | -- |
| TC-DI010 | 동적 | 미정의 범위 -> +20 점수 가산 | 범위 정의 없는 입력 | 점수 ~20 증가 | P1 | -- |
| TC-DI011 | 동적 | 파일 경로 포함 -> -30 점수 감산 | "src/auth/login.ts"가 있는 입력 | 점수 ~30 감소 | P1 | -- |
| TC-DI012 | 동적 | 기술 용어 포함 -> -20 점수 감산 | "OAuth2", "JWT"가 있는 입력 | 점수 ~20 감소 | P1 | -- |
| TC-DI013 | 동적 | 점수 >= 50 -> AskUserQuestion 트리거 | 매우 모호한 입력 | 명확화 옵션이 있는 AskUserQuestion | P0 | -- |
| TC-DI014 | 동적 | Magic Word (!hotfix) -> 점수 = 0 (문서화되었으나 미구현) | "!hotfix fix login bug" | magic word가 코드에 구현되지 않았음을 확인 | P2 | -- |

### 32.4 TC-DI: 패턴 4 - 일치율에서 Check-Act 반복으로

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-DI015 | 동적 | matchRate >= 90% -> report-generator 제안 | matchRate = 92% | 출력에 report-generator 제안 | P0 | -- |
| TC-DI016 | 동적 | matchRate 70-89% -> AskUserQuestion (수동/자동 선택) | matchRate = 80% | 수동/자동 옵션이 있는 AskUserQuestion | P0 | -- |
| TC-DI017 | 동적 | matchRate < 70% -> pdca-iterator 강력 권장 | matchRate = 55% | pdca-iterator 강력 권장 | P0 | -- |
| TC-DI018 | 동적 | 최대 5회 반복 제한 적용 | iterationCount = 5 | 수동 개입 제안, 추가 자동 반복 없음 | P1 | -- |

---

## 33. AI 네이티브 원칙 테스트 (TC-AN001~AN018)

> **출처**: `bkit-system/philosophy/ai-native-principles.md` -- 3가지 핵심 역량, 레벨 여정, 언어 티어, 에이전트 모델

### 33.1 TC-AN: 3가지 핵심 역량 지원

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-AN001 | AI네이티브 | 검증 능력 -> gap-detector가 AI 출력 검증 지원 | 설계 + 구현 | gap-detector가 불일치 식별 | P0 | -- |
| TC-AN002 | AI네이티브 | 방향 설정 -> 설계 우선 워크플로가 명확한 방향 제공 | 새 기능 요청 | 구현 전에 설계 문서 생성 | P0 | -- |
| TC-AN003 | AI네이티브 | 품질 기준 -> code-analyzer가 일관된 품질 기준 적용 | 리뷰용 코드 | 일관된 품질 검사 적용 | P0 | -- |
| TC-AN004 | AI네이티브 | bkit-rules 스킬이 품질 기준 자동 적용 | 임의 코드 변경 | 품질 규칙 자동 검사 | P1 | -- |

### 33.2 TC-AN: 레벨별 사용자 여정

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-AN005 | AI네이티브 | Starter -> SessionStart에 4개 옵션 표시 | 단순 프로젝트에서 새 세션 | 4개 옵션: 처음/학습/설정/업그레이드 | P0 | -- |
| TC-AN006 | AI네이티브 | Starter -> 간소화된 PDCA (단계 건너뛰기 가능) | Starter 레벨 프로젝트 | 단계 건너뛰기 가능 (모두 필수 아님) | P1 | -- |
| TC-AN007 | AI네이티브 | Dynamic -> 설정 자동 생성 지원 | Dynamic 프로젝트에서 /setup-claude-code | 설정 파일 자동 생성 | P1 | -- |
| TC-AN008 | AI네이티브 | Dynamic -> 5분 내 안내로 설계 문서 생성 | Dynamic 프로젝트에서 /pdca design | 명확한 구조의 템플릿 기반 설계 문서 | P1 | -- |
| TC-AN009 | AI네이티브 | Enterprise -> 플러그인 공유를 통한 팀 전체 표준화 | Enterprise 프로젝트 | 팀 표준화를 위해 플러그인 공유 가능 | P2 | -- |
| TC-AN010 | AI네이티브 | Enterprise -> /learn-claude-code 체계적 교육 제공 | Enterprise 프로젝트 | 체계적 교육 흐름 제공 | P2 | -- |

### 33.3 TC-AN: 언어 티어 시스템

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-AN011 | AI네이티브 | Tier 1 분류: Python, TS, JS, React/Next.js | detectTier("typescript") | "tier1" 또는 동급 반환 | P0 | -- |
| TC-AN012 | AI네이티브 | Tier 2 분류: Go, Rust, Dart, Vue, Astro, Flutter | detectTier("go") | "tier2" 또는 동급 반환 | P1 | -- |
| TC-AN013 | AI네이티브 | Tier 3 분류: Java, Kotlin, Swift, C/C++, Angular | detectTier("java") | "tier3" 또는 동급 반환 | P1 | -- |
| TC-AN014 | AI네이티브 | Tier 4 분류: PHP, Ruby, C#, Scala, Elixir | detectTier("php") | "tier4" 또는 동급 반환 | P1 | -- |
| TC-AN015 | AI네이티브 | Experimental 분류: Mojo, Zig, V | detectTier("mojo") | "experimental" 또는 동급 반환 | P2 | -- |

### 33.4 TC-AN: 에이전트 모델 선택 전략

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-AN016 | AI네이티브 | 7개 opus 에이전트가 전략적/복합 분석 역할에 배정 | 16개 에이전트 frontmatter 모두 읽기 | model: opus인 7개 에이전트 (cto-lead, code-analyzer, design-validator, gap-detector, enterprise-expert, infra-architect, security-architect) | P0 | -- |
| TC-AN017 | AI네이티브 | 7개 sonnet 에이전트가 실행/안내/반복 역할에 배정 | 16개 에이전트 frontmatter 모두 읽기 | model: sonnet인 7개 에이전트 (bkend-expert, pdca-iterator, pipeline-guide, starter-guide, product-manager, frontend-architect, qa-strategist) | P0 | -- |
| TC-AN018 | AI네이티브 | 2개 haiku 에이전트가 모니터링/문서 생성 역할에 배정 | 16개 에이전트 frontmatter 모두 읽기 | model: haiku인 2개 에이전트 (qa-monitor, report-generator) | P0 | -- |

---

## 34. PDCA 방법론 테스트 (TC-PM001~PM015)

> **출처**: `bkit-system/philosophy/pdca-methodology.md` -- PDCA-파이프라인 관계, 자동 적용, Check-Act 루프, Zero Script QA, 아카이브

### 34.1 TC-PM: PDCA-파이프라인 관계

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PM001 | PDCA | 각 파이프라인 단계가 내부 PDCA 사이클 실행 지원 | Phase 4 API 구현 | Phase 4 내에서 Plan -> Design -> Do -> Check -> Act | P0 | -- |
| TC-PM002 | PDCA | Starter 레벨 파이프라인 흐름: 1->2->3->6->9 | Starter 레벨 프로젝트 | Phase 4,5,7,8 건너뜀 | P0 | -- |
| TC-PM003 | PDCA | Dynamic 레벨 파이프라인 흐름: 1->2->3->4->5->6->7->9 | Dynamic 레벨 프로젝트 | Phase 8 선택사항 | P1 | -- |
| TC-PM004 | PDCA | Enterprise 레벨: 9개 단계 모두 사용 가능 | Enterprise 레벨 프로젝트 | Phase 1-9 모두 존재 | P1 | -- |

### 34.2 TC-PM: 자동 적용 규칙

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PM005 | PDCA | PreToolUse 훅이 Design 문서 존재 확인 | design 없이 기능 파일에 Write 도구 | 훅 출력에 설계 우선 제안 | P0 | -- |
| TC-PM006 | PDCA | PostToolUse 훅이 변경 파일 분석 후 갭 분석 제안 | 기능 파일에 Write 도구 완료 | "갭 분석을 실행할까요?" 제안 | P1 | -- |
| TC-PM007 | PDCA | Quick Fix (<10줄)가 PDCA 우회 | 5줄 수정 | PDCA 강제 없음 | P0 | -- |

### 34.3 TC-PM: Check-Act 반복 루프

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PM008 | PDCA | gap-detector Stop 훅이 matchRate에 따라 분기 | 3개 구간의 matchRate (<70, 70-89, >=90) | 구간별 서로 다른 액션 | P0 | -- |
| TC-PM009 | PDCA | pdca-iterator Stop 훅이 gap-detector 재실행 트리거 | 남은 갭이 있는 Iterator 완료 | gap-detector 재호출 | P1 | -- |
| TC-PM010 | PDCA | 최대 5회 반복 후 수동 전환 | iterationCount가 5에 도달 | 수동 개입 메시지, 추가 자동 반복 없음 | P1 | -- |

### 34.4 TC-PM: Zero Script QA

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PM011 | PDCA | zero-script-qa 스킬 로그 기반 테스트 방법론 유효성 | zero-script-qa SKILL.md 읽기 | 예제가 포함된 로그 기반 QA 방법론 문서화 | P1 | -- |
| TC-PM012 | PDCA | JSON 구조화 로그 패턴 참조 정확성 | zero-script-qa SKILL.md 읽기 | JSON 로그 패턴 올바르게 참조 | P2 | -- |

### 34.5 TC-PM: 아카이브 규칙

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-PM013 | PDCA | 일치율 >= 90%에서 아카이브 활성화 | matchRate = 92% | 아카이브 명령 사용 가능 | P0 | -- |
| TC-PM014 | PDCA | 보고서 생성 완료 시 아카이브 권장 트리거 | /pdca report 완료 | 아카이브 제안 | P1 | -- |
| TC-PM015 | PDCA | 아카이브가 4가지 문서 유형을 docs/archive/YYYY-MM/{feature}/로 이동 | /pdca archive auth-feature | 4개 문서가 날짜별 아카이브 폴더로 이동 | P0 | -- |

---

## 35. 오케스트레이션 패턴 테스트 (TC-OP001~OP008)

> **출처**: `bkit-system/philosophy/context-engineering.md` -- CTO 주도 에이전트 팀 오케스트레이션 패턴

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-OP001 | 오케스트레이션 | Dynamic 레벨 Plan 단계 -> leader 패턴 선택 | Dynamic + Plan | pattern = "leader" | P0 | -- |
| TC-OP002 | 오케스트레이션 | Dynamic 레벨 Check 단계 -> council 패턴 선택 | Dynamic + Check | pattern = "council" | P0 | -- |
| TC-OP003 | 오케스트레이션 | Enterprise 레벨 Design 단계 -> council 패턴 선택 | Enterprise + Design | pattern = "council" | P0 | -- |
| TC-OP004 | 오케스트레이션 | Enterprise 레벨 Act 단계 -> watchdog 패턴 선택 | Enterprise + Act | pattern = "watchdog" | P1 | -- |
| TC-OP005 | 오케스트레이션 | leader 패턴 -> CTO 리드가 단독 결정 | leader 패턴 활성 | CTO 단독 결정, council 없음 | P1 | -- |
| TC-OP006 | 오케스트레이션 | council 패턴 -> 다수 에이전트 기여 | council 패턴 활성 | 다수 에이전트 관점 수집 | P1 | -- |
| TC-OP007 | 오케스트레이션 | swarm 패턴 -> 병렬 실행 | swarm 패턴 활성 | 태스크가 병렬 분배 | P1 | -- |
| TC-OP008 | 오케스트레이션 | watchdog 패턴 -> 모니터링과 개입 | watchdog 패턴 활성 | 지속적 모니터링, 임계값 시 개입 | P2 | -- |

---

## 36. 출력 스타일 컨텍스트 계층 테스트 (TC-OC001~OC005)

> **출처**: `bkit-system/philosophy/context-engineering.md` -- 컨텍스트 계층으로서의 출력 스타일

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-OC001 | 출력스타일 | bkit-learning 스타일에 학습 포인트, TODO(learner) 마커 포함 | bkit-learning 활성화 | 응답에 학습 설명과 TODO(learner) 포함 | P0 | -- |
| TC-OC002 | 출력스타일 | bkit-pdca-guide 스타일에 PDCA 상태 배지, 체크리스트 포함 | bkit-pdca-guide 활성화 | 응답에 단계 배지와 체크리스트 포함 | P0 | -- |
| TC-OC003 | 출력스타일 | bkit-enterprise 스타일에 트레이드오프 테이블, 비용 영향 분석 포함 | bkit-enterprise 활성화 | 응답에 트레이드오프 분석과 비용 추정 포함 | P0 | -- |
| TC-OC004 | 출력스타일 | 레벨 감지가 출력 스타일 자동 매칭 (Starter->learning, Dynamic->pdca-guide, Enterprise->enterprise) | 각 레벨 감지 | 대응하는 스타일 제안 | P0 | -- |
| TC-OC005 | 출력스타일 | plugin.json에 outputStyles 경로 선언 | plugin.json 읽기 | "outputStyles" 키가 경로와 함께 존재 | P1 | -- |

---

## 37. 응답 리포트 규칙 테스트 (TC-RR001~RR005)

> **출처**: `bkit-system/philosophy/context-engineering.md` -- 응답 리포트 규칙

| TC-ID | 카테고리 | 설명 | 입력 | 기대 결과 | 우선순위 | 상태 |
|-------|----------|------|------|-----------|:--------:|:----:|
| TC-RR001 | 리포트규칙 | 모든 응답이 bkit 기능 사용 리포트 구조로 끝남 | bkit 지원 임의 응답 | 사용/미사용/권장이 있는 리포트 섹션 존재 | P0 | -- |
| TC-RR002 | 리포트규칙 | "사용" 섹션에 실제 사용 기능 나열 | gap-detector + PDCA status를 사용한 응답 | "사용: gap-detector, PDCA status" 나열 | P1 | -- |
| TC-RR003 | 리포트규칙 | "미사용" 섹션에 미사용 기능과 이유 나열 | Agent Teams 미사용 응답 | "미사용: Agent Teams (단독 개발자)" 나열 | P1 | -- |
| TC-RR004 | 리포트규칙 | "권장" 섹션에 PDCA 단계 적절 제안 제공 | 현재 단계 = Check (matchRate < 90%) | 권장: /pdca iterate | P0 | -- |
| TC-RR005 | 리포트규칙 | PDCA 단계별 권장 매핑 정확도 | 6개 PDCA 단계 각각 | 단계별 올바른 다음 명령 권장 | P1 | -- |

---

## 38. 갱신된 테스트 실행 계획 (철학 & 컨텍스트 엔지니어링 포함)

### 38.1 전체 실행 우선순위 (갱신)

```
1단계 (P0 크리티컬 - v1.5.9 변경):
  TC-H001~H006, TC-I001~I006, TC-U001~U037   (기존 49 TC)

2단계 (P0 통합 + E2E):
  TC-H007~H032, TC-I007~I020, TC-E001~E013    (기존 46 TC)

3단계 (P0 전체 커버리지):
  TC-CS001~CS009, TC-CS010~CS013, TC-CS017~CS022 (19 TC)
  TC-AG001~AG018, TC-CF001~CF011 (29 TC)
  TC-SK001~SK005, SK009 (6 TC)
  TC-PD001~PD002, PD007~PD012 (8 TC)
  TC-TM001~TM004, TM006~TM010 (9 TC)
  TC-SC001~SC014 (14 TC)
  TC-LV001~LV003, LV005~LV006, LV008~LV010 (8 TC)
  TC-IN001~IN002, IN011~IN012, IN015, IN017~IN020, IN025~IN026, IN028~IN030 (13 TC)
  TC-MEM001~MEM003, MEM005, MEM007, MEM011 (6 TC)
  TC-OS001~OS004 (4 TC)
  TC-TP001~TP007 (7 TC)

4단계 (P0 철학 & 컨텍스트 엔지니어링):
  TC-PH001~PH003, PH006~PH010, PH013~PH015     -- 12 TC (핵심 철학 P0)
  TC-CE001~CE003, CE006~CE007, CE011, CE014~CE016, CE018~CE022, CE025, CE028, CE031 -- 16 TC (컨텍스트 엔지니어링 P0)
  TC-HL001~HL005                                  -- 5 TC (훅 계층 P0)
  TC-DI001~DI007, DI009, DI013, DI015~DI017      -- 12 TC (동적 컨텍스트 P0)
  TC-AN001~AN003, AN005, AN011, AN016~AN018       -- 8 TC (AI 네이티브 P0)
  TC-PM001~PM002, PM005, PM007~PM008, PM013, PM015 -- 7 TC (PDCA 방법론 P0)
  TC-OP001~OP003                                  -- 3 TC (오케스트레이션 P0)
  TC-OC001~OC004                                  -- 4 TC (출력 스타일 P0)
  TC-RR001, RR004                                 -- 2 TC (리포트 규칙 P0)

5단계 (P1 품질):
  TC-X001~X020 (UX)                  -- 20 TC
  TC-R001~R015 (회귀)                -- 15 TC
  TC-CS014~CS016, CS023~CS046 (코어 P1) -- 22 TC
  TC-IN003~IN010, IN013~IN014, IN016, IN021~IN024, IN027, IN031~IN032 (인텐트 P1) -- 18 TC
  TC-TS006~TS007, TS013, TS016~TS018, TS020~TS021 (태스크 P1) -- 8 TC
  TC-TM005, TM011~TM019, TM022, TM026~TM027 (팀 P1) -- 12 TC
  TC-PD003, PD005~PD006, PD015~PD017, PD021~PD022 (PDCA P1) -- 8 TC
  TC-AG019 (에이전트 메모리 P1)       -- 1 TC
  TC-SK007~SK008, SK010~SK027 (스킬 P1) -- 20 TC
  TC-TP008~TP017, TP022~TP025 (템플릿 P1) -- 14 TC
  TC-SC015~SC024, SC029~SC030 (스크립트 P1) -- 12 TC
  TC-L10N005~L10N008, L10N012~L10N014, L10N016 (i18n P1) -- 8 TC
  TC-MEM004, MEM006, MEM008~MEM010 (메모리 P1) -- 5 TC
  TC-OS005~OS006, TC-LV004, LV007, LV011 (기타 P1) -- 5 TC
  TC-PH004~PH005, PH011~PH012, PH016~PH017 (철학 P1) -- 6 TC
  TC-CE004~CE005, CE008~CE009, CE012~CE013, CE017, CE023~CE024, CE026~CE027, CE029~CE030 (CE P1) -- 13 TC
  TC-HL006~HL007, TC-DI008, DI010~DI012, DI018 (동적 P1) -- 7 TC
  TC-AN004, AN006~AN008, AN012~AN014 (AI 네이티브 P1) -- 7 TC
  TC-PM003~PM004, PM006, PM009~PM011, PM014 (PDCA P1) -- 7 TC
  TC-OP004~OP007 (오케스트레이션 P1)  -- 4 TC
  TC-OC005, TC-RR002~RR003, RR005 (출력/리포트 P1) -- 4 TC

6단계 (P2 강화):
  TC-EC001~EC010 (엣지 케이스)        -- 10 TC
  TC-P001~P005 (성능)                -- 5 TC
  TC-E014~E015 (P1 E2E)             -- 2 TC
  TC-TP018~TP021, TP026 (템플릿 P2)  -- 5 TC
  TC-SC025~SC028, SC031~SC032 (스크립트 P2) -- 6 TC
  TC-LV012 (파이프라인 P2)           -- 1 TC
  TC-CE010 (Import TTL P2)          -- 1 TC
  TC-DI014 (Magic Word P2)          -- 1 TC
  TC-AN009~AN010, AN015 (AI 네이티브 P2) -- 3 TC
  TC-PM012 (Zero Script P2)         -- 1 TC
  TC-OP008 (Watchdog P2)            -- 1 TC
```

### 38.2 갱신된 에이전트 태스크 배정

| 에이전트 | 테스트 카테고리 | TC 수 | 예상 소요시간 |
|----------|----------------|:-----:|:------------:|
| code-analyzer | TC-U, TC-EC, TC-CS, TC-TS, TC-TM, TC-PD, TC-CE(FR01-06), TC-HL | 170 + 38 = **208** | 8-10시간 |
| qa-monitor | TC-H, TC-P, TC-OS, TC-SC, TC-HL005, TC-OC, TC-RR | 75 + 10 = **85** | 4-5시간 |
| gap-detector | TC-I, TC-E, TC-SK, TC-AG, TC-CF, TC-TP, TC-PH(문서=코드), TC-PM | 119 + 20 = **139** | 6-7시간 |
| product-manager | TC-X, TC-IN, TC-LV, TC-L10N, TC-MEM, TC-PH(자동화/추측금지), TC-AN, TC-DI | 85 + 48 = **133** | 6-7시간 |
| qa-strategist | TC-R, TC-OP, 조율, 최종 통합 | 15 + 8 + 조율 = **23+** | 2-3시간 |

### 38.3 갱신된 성공 기준

| 카테고리 | 목표 | 최소 |
|----------|:----:|:----:|
| P0 테스트 (337) | 100% | 100% |
| P1 테스트 (215) | 100% | 95% |
| P2 테스트 (42) | 95% | 90% |
| 전체 (594) | 100% | 97% |

---

## 버전 이력

| 버전 | 날짜 | 변경사항 | 작성자 |
|------|------|---------|--------|
| 0.1 | 2026-03-05 | 초안: v1.5.9 변경 대상 161 TC | CTO 팀 |
| 0.2 | 2026-03-05 | 전체 커버리지 확장: +270 TC (14개 신규 카테고리), 총 431 TC | CTO 팀 |
| 0.3 | 2026-03-05 | 철학 & 컨텍스트 엔지니어링: +124 TC (9개 신규 카테고리: PH/CE/HL/DI/AN/PM/OP/OC/RR), 총 594 TC | CTO 리드 |
| 1.0 | 2026-03-05 | 한국어 번역 완성 | CTO 리드 |
