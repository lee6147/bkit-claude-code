# bkit v1.5.9 Executive Summary Feature Planning Document

> **Summary**: PDCA 문서화 완료 후 4가지 관점(Problem, Solution, Function/UX Effect, Core Value)으로 Executive Summary를 자동 생성하여 사용자의 Next Action 의사결정을 지원하는 기능
>
> **Project**: bkit (Vibecoding Kit)
> **Version**: 1.5.9
> **Author**: CTO Team (6 agents)
> **Date**: 2026-03-05
> **Status**: Draft
> **Method**: Plan Plus (Brainstorming-Enhanced PDCA)

---

## Executive Summary

| 관점 | 내용 |
|------|------|
| **Problem** | PDCA 문서화(/plan-plus, /pdca plan, /pdca report) 완료 후 사용자가 결과 컨텍스트 없이 Next Action을 선택해야 하여 의사결정 품질이 낮고, 문서를 다시 스크롤하여 읽어야 하는 비효율 발생 |
| **Solution** | 3종 템플릿에 Executive Summary 섹션을 추가하고, PDCA 스킬/에이전트에 자동 생성 지시를 통합하며, `lib/pdca/executive-summary.js` 모듈로 프로그래밍 방식의 요약 생성 + AskUserQuestion preview 기반 Next Action UX 제공 |
| **Function/UX Effect** | 문서 완료 시 즉시 핵심 요약 표시 (스크롤 불필요), 의사결정 시간 -40% 예상, PDCA 단계별 맞춤 Next Action 옵션으로 워크플로우 연속성 향상, 8개 언어 자동 적용 |
| **Core Value** | "맥락 있는 의사결정" - PDCA 사이클의 각 문서화 단계가 단순 산출물이 아닌 행동 가능한 의사결정 도구로 진화하여, bkit의 핵심 가치인 AI-native 개발 생산성을 한 단계 높임 |

---

## 1. User Intent Discovery

### 1.1 Core Problem

PDCA 문서화(/plan-plus, /pdca plan, /pdca report) 완료 후 사용자가 **결과 컨텍스트 없이** 다음 단계를 선택해야 하는 문제:

- Plan 문서 작성 후: "이 계획을 진행할지, 수정할지" 판단 근거가 문서 내부에 분산
- Report 완료 후: "아카이브할지, 추가 개선할지" 결정에 필요한 핵심 지표가 즉시 보이지 않음
- 문서를 다시 스크롤하여 읽어야 하는 비효율 반복

### 1.2 Target Users

| User Type | Usage Context | Key Need |
|-----------|---------------|----------|
| bkit 개발자 | PDCA 사이클 실행 중 | 문서 완료 후 즉시 핵심 파악 + Next Action 선택 |
| CTO/PM | Plan 리뷰 시 | 4가지 관점으로 빠른 의사결정 |
| CTO Team Agent | 팀 모드에서 문서 참조 시 | 문서 상단에서 컨텍스트 즉시 파악 |

### 1.3 Success Criteria

- [ ] /plan-plus 완료 시 Executive Summary가 문서 상단에 자동 포함
- [ ] /pdca plan 완료 시 Executive Summary가 문서 상단에 자동 포함
- [ ] /pdca report 완료 시 Executive Summary가 문서에 포함 + CLI 출력
- [ ] 4가지 관점(Problem/Solution/Function/Core Value) 모두 1-2문장으로 요약
- [ ] Next Action 옵션이 PDCA 단계별로 맞춤 제공
- [ ] formatAskUserQuestion()에 preview 필드 슬롯 확보

### 1.4 Constraints

| Constraint | Details | Impact |
|------------|---------|--------|
| 기존 템플릿 섹션 번호 유지 | Executive Summary는 번호 없는 `##` 사용 | Low |
| automation.js 337행 한도 | 새 모듈 분리 필요 (executive-summary.js) | Medium |
| v2.1.69 preview 필드 안정성 | preview 슬롯만 확보, 실제 활용은 v1.6.0 | Low |
| common.js export 수 관리 | 188→191 (3개 추가, +1.6%) | Low |

---

## 2. Alternatives Explored

### 2.1 Approach A: 템플릿 + 출력 통합 — Selected

| Aspect | Details |
|--------|---------|
| **Summary** | 3종 템플릿에 `## Executive Summary` 섹션 추가 + PDCA 스킬 완료 시 CLI에도 즉시 출력 |
| **Pros** | 문서에도 남고 대화에서도 바로 보임, 문서 자체 완결성 유지, 버전 관리 용이 |
| **Cons** | 템플릿 3종 + 스킬 2종 + 에이전트 1종 수정 필요 |
| **Effort** | Medium (12개 파일 수정) |
| **Best For** | 문서 기반 워크플로우를 중시하면서도 즉각적 피드백이 필요한 경우 |

### 2.2 Approach B: CLI 출력 전용

| Aspect | Details |
|--------|---------|
| **Summary** | 템플릿 무변경, PDCA 스킬 실행 후 CLI 대화에서만 Executive Summary 표시 |
| **Pros** | 기존 템플릿 구조 완전 보존, 구현 범위 최소 |
| **Cons** | 문서에 남지 않아 CI/CD 연동 어려움, 나중에 문서를 열면 Summary 없음 |
| **Effort** | Low (5개 파일 수정) |
| **Best For** | 빠른 MVP, 템플릿 변경을 최소화하고 싶은 경우 |

### 2.3 Decision Rationale

**Selected**: Approach A (템플릿 + 출력 통합)
**Reason**: 사용자가 명확하게 "문서에도 남고 대화에서도 보이는" 방식을 선호. bkit의 PDCA 방법론에서 문서는 핵심 산출물이므로, Executive Summary가 문서 자체에 포함되어야 CTO Team 에이전트들도 문서 참조 시 즉시 컨텍스트를 파악할 수 있음.

---

## 3. YAGNI Review

### 3.1 Included (v1.5.9 Must-Have)

- [x] 템플릿 3종 Executive Summary 섹션 추가 (plan, plan-plus, report)
- [x] PDCA 스킬/에이전트 지시 수정 (pdca SKILL.md, plan-plus SKILL.md, report-generator.md)
- [x] `lib/pdca/executive-summary.js` 모듈 (generateExecutiveSummary, formatExecutiveSummary, generateBatchSummary)
- [x] Next Action UX (formatAskUserQuestion preview 슬롯 + PDCA 단계별 옵션)
- [x] common.js 브릿지 갱신 (3개 export 추가)
- [x] pdca/index.js 통합
- [x] v2.1.69 ENH 통합 (ENH-74 agent_id, ENH-75 continue:false, ENH-79 Output Efficiency)

### 3.2 Deferred (v1.6.0 Maybe)

| Feature | Reason for Deferral | Revisit When |
|---------|---------------------|--------------|
| AskUserQuestion preview 실제 렌더링 | v2.1.69 preview API 안정성 확인 필요 | CC v2.1.70+ |
| InstructionsLoaded Hook 활용 (ENH-72) | SessionStart로 대체 가능 | v1.6.0 |
| i18n Executive Summary | 8개 언어별 레이블 번역 | v1.6.0 |
| Executive Summary 히스토리 저장 | .bkit-memory.json에 Summary 이력 | v1.6.0 |

### 3.3 Removed (Won't Do)

| Feature | Reason for Removal |
|---------|-------------------|
| 별도 Executive Summary 파일 생성 | 문서 안에 섹션으로 포함하는 것이 더 효율적 |
| Executive Summary 에디터 (인터랙티브 수정) | 과도한 복잡성, YAGNI |
| PDF/HTML Executive Summary 내보내기 | bkit 스코프 밖 |

---

## 4. Scope

### 4.1 In Scope

- [x] `templates/plan-plus.template.md` — `## Executive Summary` 섹션 추가 (Header 아래, §1 앞)
- [x] `templates/plan.template.md` — `## Executive Summary` 섹션 추가 (Header 아래, §1 앞)
- [x] `templates/report.template.md` — `## 1. Summary` → `## Executive Summary` 구조 확장 (§1.3 Value Delivered 추가)
- [x] `skills/pdca/SKILL.md` — plan 액션에 Executive Summary 작성 지시 추가
- [x] `skills/plan-plus/SKILL.md` — Phase 5에 자동 합성 지시 추가
- [x] `agents/report-generator.md` — Executive Summary 4관점 작성 지시 추가
- [x] `lib/pdca/executive-summary.js` — 신규 모듈 생성 (3~5 exports)
- [x] `lib/pdca/index.js` — executive-summary 모듈 통합
- [x] `lib/common.js` — 브릿지 export 추가 (188→191)
- [x] `lib/pdca/automation.js` — formatAskUserQuestion() preview 슬롯 추가
- [x] `scripts/pdca-task-completed.js` — ENH-74 agent_id + ENH-75 continue:false
- [x] `scripts/team-idle-handler.js` — ENH-74 agent_id + ENH-75 continue:false

### 4.2 Out of Scope

- AskUserQuestion preview 실제 UI 렌더링 활용 (v1.6.0 DEFER)
- InstructionsLoaded Hook 통합 (v1.6.0 DEFER)
- i18n 레이블 번역 (v1.6.0 DEFER)
- Executive Summary 히스토리 저장 (v1.6.0 DEFER)

---

## 5. Requirements

### 5.1 Functional Requirements

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-01 | plan-plus.template.md에 `## Executive Summary` 섹션 추가 (4관점 테이블) | High | Pending |
| FR-02 | plan.template.md에 `## Executive Summary` 섹션 추가 (4관점 테이블) | High | Pending |
| FR-03 | report.template.md `## 1. Summary`를 `## Executive Summary`로 확장 (§1.3 Value Delivered 추가) | High | Pending |
| FR-04 | skills/pdca/SKILL.md plan 액션에 Executive Summary 작성 지시 추가 | High | Pending |
| FR-05 | skills/plan-plus/SKILL.md Phase 5에 자동 합성 지시 추가 (Phase 0-4 결과 기반) | High | Pending |
| FR-06 | agents/report-generator.md에 Executive Summary 4관점 작성 지시 추가 | High | Pending |
| FR-07 | `lib/pdca/executive-summary.js` 신규 모듈 생성 (generateExecutiveSummary, formatExecutiveSummary, generateBatchSummary) | High | Pending |
| FR-08 | `lib/pdca/index.js`에 executive-summary 모듈 re-export 통합 | Medium | Pending |
| FR-09 | `lib/common.js` 브릿지에 3개 export 추가 (188→191) | Medium | Pending |
| FR-10 | `formatAskUserQuestion()`에 preview optional 필드 슬롯 추가 | Medium | Pending |
| FR-11 | PDCA 단계별 Next Action 옵션 세트 정의 (plan 후, report 후 등) | Medium | Pending |
| FR-12 | `scripts/pdca-task-completed.js`에 ENH-74 agent_id 1순위 + ENH-75 continue:false 추가 | Medium | Pending |
| FR-13 | `scripts/team-idle-handler.js`에 ENH-74 agent_id 1순위 + ENH-75 continue:false 추가 | Medium | Pending |
| FR-14 | status.js 5개 누락 함수 브릿지 복구 (deleteFeatureFromStatus 등) | Low | Pending |
| FR-15 | common.js 주석 정정 ("11 exports" → "13 exports" 등) | Low | Pending |

### 5.2 Non-Functional Requirements

| Category | Criteria | Measurement Method |
|----------|----------|-------------------|
| Performance | executive-summary.js 함수 실행 < 50ms | console.time 측정 |
| Compatibility | CC v2.1.69+ 완전 호환 | 기존 TC + 신규 TC |
| Backward Compat | CC v2.1.34~v2.1.68에서도 preview 없이 정상 작동 | 이전 버전 테스트 |
| Code Quality | Lazy require 패턴 준수, JSDoc 완비 | Code review |

---

## 6. Success Criteria

### 6.1 Definition of Done

- [ ] 모든 FR (15개) 구현 완료
- [ ] /plan-plus 실행 시 문서에 Executive Summary 포함 확인
- [ ] /pdca plan 실행 시 문서에 Executive Summary 포함 확인
- [ ] /pdca report 실행 시 문서 + CLI 출력에 Executive Summary 포함 확인
- [ ] Next Action 옵션이 PDCA 단계별로 정확히 표시
- [ ] common.js export 수 정상 반영 (191개)
- [ ] Gap Analysis Match Rate >= 90%

### 6.2 Quality Criteria

- [ ] 기존 188개 export 하위 호환성 유지
- [ ] Lazy require 패턴 일관성 (순환 의존 없음)
- [ ] 3-layer re-export 패턴 준수 (executive-summary.js → index.js → common.js)

---

## 7. Risks and Mitigation

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| 기존 템플릿 사용자에게 혼란 | Medium | Low | 번호 없는 `##` 사용으로 기존 섹션 번호 불변 |
| preview 필드 CC 버전 비호환 | Low | Medium | optional 필드로 구현, 없어도 정상 작동 |
| common.js export 수 증가로 로딩 시간 영향 | Low | Low | Lazy require 패턴으로 실제 로딩 지연 없음 |
| report.template.md 구조 변경 충돌 | Medium | Low | `## 1. Summary` → `## Executive Summary` rename만, 하위 구조 유지 |
| CTO Team에서 continue:false 오작동 | High | Low | matchRate 기반 조건부 적용, 수동 모드에서는 비활성 |

---

## 8. Architecture Considerations

### 8.1 Project Level Selection

| Level | Characteristics | Selected |
|-------|-----------------|:--------:|
| **Starter** | bkit plugin 자체 (Node.js, 단일 패키지) | **Yes** |

### 8.2 Key Decisions

| Decision | Options | Selected | Rationale |
|----------|---------|----------|-----------|
| Executive Summary 모듈 위치 | automation.js 확장 vs 별도 파일 | **별도 파일** (executive-summary.js) | SRP 준수, automation.js 337행 한도, 기존 모듈 패턴 일관성 |
| 템플릿 삽입 위치 | 상단 (Header 아래) vs 하단 | **상단** | 리뷰어/에이전트가 가장 먼저 읽는 위치, 스크롤 불필요 |
| Executive Summary 포맷 | 테이블 vs 박스 vs 인라인 | **테이블** (기본) + **박스** (CLI) | 문서에는 테이블, CLI에는 PDCA 배지 통합 카드 |
| Next Action 구현 | emitUserPrompt 확장 vs 새 함수 | **formatAskUserQuestion 확장** | preview 슬롯으로 향후 확장성 확보 |

### 8.3 Component Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  Templates (3종)                                                 │
│  ├── plan.template.md          ← ## Executive Summary 추가      │
│  ├── plan-plus.template.md     ← ## Executive Summary 추가      │
│  └── report.template.md        ← ## 1.Summary → Executive Sum  │
├─────────────────────────────────────────────────────────────────┤
│  Skills/Agents (3종)                                             │
│  ├── skills/pdca/SKILL.md      ← plan 액션 지시 추가            │
│  ├── skills/plan-plus/SKILL.md ← Phase 5 합성 지시 추가         │
│  └── agents/report-generator.md ← 4관점 작성 지시 추가          │
├─────────────────────────────────────────────────────────────────┤
│  Library (4종)                                                   │
│  ├── lib/pdca/executive-summary.js  ← NEW (3 exports)          │
│  ├── lib/pdca/index.js              ← re-export 추가            │
│  ├── lib/common.js                  ← 브릿지 추가 (188→191)     │
│  └── lib/pdca/automation.js         ← preview 슬롯 추가         │
├─────────────────────────────────────────────────────────────────┤
│  Scripts (2종) - ENH Integration                                 │
│  ├── scripts/pdca-task-completed.js ← ENH-74/75                 │
│  └── scripts/team-idle-handler.js   ← ENH-74/75                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.4 Data Flow

```
[PDCA Skill 실행 완료]
    │
    ├── /plan-plus Phase 5 ──────────┐
    ├── /pdca plan ──────────────────┤
    └── /pdca report ────────────────┤
                                     ▼
                    ┌─────────────────────────────┐
                    │ Executive Summary 생성       │
                    │ (Template 섹션 + CLI 출력)   │
                    │                             │
                    │ Problem     ← Phase/Doc 기반 │
                    │ Solution    ← 선택된 접근법   │
                    │ Function/UX ← 성공 기준/메트릭│
                    │ Core Value  ← 비즈니스 가치   │
                    └──────────┬──────────────────┘
                               │
                               ▼
                    ┌─────────────────────────────┐
                    │ Next Action (AskUserQuestion) │
                    │                             │
                    │ plan 후: Design/수정/리뷰    │
                    │ report 후: Archive/개선/다음  │
                    │ (preview 슬롯 포함)          │
                    └─────────────────────────────┘
```

### 8.5 Executive Summary CLI 출력 디자인 (시안 3 채택)

```
╔═══════════════════════════════════════════════════════════════════════════╗
║  EXECUTIVE SUMMARY                                                       ║
╠═══════════════════════════════════════════════════════════════════════════╣
║  {feature}                                              {date}           ║
║  [Plan OK] → [Design ○] → [Do ○] → [Check ○] → [Act ○] → [Report ○]   ║
╚═══════════════════════════════════════════════════════════════════════════╝

  [PROBLEM]
  {핵심 문제 1-2문장}

  [SOLUTION]
  {해결 방안 1-2문장}

  [FUNCTION & UX EFFECT]
  {기능적/UX 기대 효과, 구체적 메트릭 포함}

  [CORE VALUE]
  {핵심 가치 제안 1-2문장}

╔═══════════════════════════════════════════════════════════════════════════╗
║  NEXT ACTION                                                             ║
╠═══════════════════════════════════════════════════════════════════════════╣
║  [1] Design 진행     /pdca design {feature}                              ║
║  [2] Plan 수정       /plan-plus {feature}                                ║
║  [3] 팀 리뷰 요청   /pdca team {feature}                                ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

간략 모드 (반복 출력/compact 시):
```
── EXEC SUMMARY: {feature} ─────────────────────────── {date} ──
  PROBLEM    {1문장}
  SOLUTION   {1문장}
  EFFECT     {1문장}
  VALUE      {1문장}
────────────────────────────────────────────────────────────────
  [1] Design  [2] 수정  [3] 리뷰  [4] 스킵
────────────────────────────────────────────────────────────────
```

---

## 9. v2.1.69 ENH Items Integration

### 9.1 v1.5.9 통합 대상 (Executive Summary 시너지)

| ENH | Title | 시너지 | 통합 포인트 |
|-----|-------|:------:|-----------|
| **ENH-74** | agent_id/agent_type hook 필드 | HIGH | pdca-task-completed.js, team-idle-handler.js에서 1순위 추출 |
| **ENH-75** | continue:false 지원 | CRITICAL | TaskCompleted에서 품질 게이트 실패 시 teammate 자동 중단 |
| **ENH-78** | AskUserQuestion preview | HIGH | formatAskUserQuestion()에 preview optional 슬롯 확보 |
| **ENH-79** | Output Efficiency | HIGH | Executive Summary 간결성 가이드, 에이전트 지시 추가 |

### 9.2 v1.5.9 단독 구현 (Executive Summary 무관)

| ENH | Title | 구현 내용 |
|-----|-------|----------|
| ENH-73 | ${CLAUDE_SKILL_DIR} 변수 | 스킬 내부 리소스 경로 활용 |
| ENH-76 | /reload-plugins 가이드 | 문서 업데이트 |
| ENH-77 | AUTO_MEMORY_PATH 가이드 | 문서 업데이트 |
| ENH-80 | Verification Specialist | 에이전트 effort 가이드 |
| ENH-81 | CTO Team 안정성 문서 | 메모리 최적화 가이드 |

### 9.3 구현 의존성 순서

```
Phase 1: 기반 (병렬 가능)
├── ENH-73, ENH-74, ENH-76, ENH-77, ENH-79, ENH-80
│
Phase 2: 핵심 (Phase 1 → Phase 2)
├── ENH-75 (← ENH-74 필요)
├── ENH-81
│
Phase 3: Executive Summary (Phase 2 → Phase 3)
├── FR-01~03: 템플릿 수정
├── FR-04~06: 스킬/에이전트 지시
├── FR-07~09: 모듈 생성 + 브릿지
├── FR-10~11: Next Action UX
├── FR-12~13: ENH-74/75 통합
└── FR-14~15: 기존 누락 수정
```

---

## 10. Implementation File Map

| 순서 | 파일 | 변경 유형 | FR | ENH |
|:----:|------|----------|:--:|:---:|
| 1 | `templates/plan-plus.template.md` | Edit | FR-01 | - |
| 2 | `templates/plan.template.md` | Edit | FR-02 | - |
| 3 | `templates/report.template.md` | Edit | FR-03 | - |
| 4 | `skills/pdca/SKILL.md` | Edit | FR-04 | - |
| 5 | `skills/plan-plus/SKILL.md` | Edit | FR-05 | - |
| 6 | `agents/report-generator.md` | Edit | FR-06 | ENH-79 |
| 7 | `lib/pdca/executive-summary.js` | **New** | FR-07 | - |
| 8 | `lib/pdca/index.js` | Edit | FR-08 | - |
| 9 | `lib/common.js` | Edit | FR-09, FR-15 | - |
| 10 | `lib/pdca/automation.js` | Edit | FR-10 | ENH-78 |
| 11 | `scripts/pdca-task-completed.js` | Edit | FR-12 | ENH-74, ENH-75 |
| 12 | `scripts/team-idle-handler.js` | Edit | FR-13 | ENH-74, ENH-75 |
| 13 | `scripts/subagent-start-handler.js` | Edit | - | ENH-74 |
| 14 | `scripts/subagent-stop-handler.js` | Edit | - | ENH-74 |
| 15 | `agents/code-analyzer.md` | Edit | - | ENH-79 |
| 16 | `agents/gap-detector.md` | Edit | - | ENH-79 |
| 17 | `agents/design-validator.md` | Edit | - | ENH-79 |
| 18 | `skills/bkit-rules/SKILL.md` | Edit | - | ENH-76 |
| 19 | `hooks/session-start.js` | Edit | - | ENH-81 |
| 20 | `.claude-plugin/plugin.json` | Edit | - | version bump |
| 21 | `bkit.config.json` | Edit | - | version bump |

**총 변경 파일: 21개** (신규 1개 + 수정 20개)

---

## 11. PDCA Phase-specific Next Action Options

### 11.1 After /plan-plus or /pdca plan

| # | Label | Description | Command |
|---|-------|-------------|---------|
| 1 | Design 진행 | 설계 문서 작성으로 이동 | `/pdca design {feature}` |
| 2 | Plan 수정 | 계획 문서 보완 | `/plan-plus {feature}` |
| 3 | 팀 리뷰 요청 | CTO Team으로 리뷰 | `/pdca team {feature}` |

### 11.2 After /pdca report

| # | Label | Description | Command |
|---|-------|-------------|---------|
| 1 | Archive | 완료 문서 아카이브 | `/pdca archive {feature}` |
| 2 | 추가 개선 | Act 단계로 이동 | `/pdca iterate {feature}` |
| 3 | 다음 기능 | 새 기능 시작 | `/pdca plan {next-feature}` |

### 11.3 After /pdca analyze (Check)

| # | Label | Description | Command |
|---|-------|-------------|---------|
| 1 | Report 생성 (>=90%) | 완료 보고서 작성 | `/pdca report {feature}` |
| 2 | 자동 개선 (<90%) | Act 반복 | `/pdca iterate {feature}` |
| 3 | 수동 수정 | 직접 코드 수정 후 재분석 | (manual) |

---

## 12. executive-summary.js Module Design

### 12.1 Exports

```javascript
module.exports = {
  generateExecutiveSummary,    // (feature, phase, context) → Object
  formatExecutiveSummary,      // (summary, format) → string
  generateBatchSummary,        // (features) → Object
};
```

### 12.2 Dependencies (Lazy Require)

```
executive-summary.js
  ├── getCore()   → lib/core (debugLog, getConfig)
  ├── getStatus() → lib/pdca/status (getPdcaStatusFull)
  └── getPhase()  → lib/pdca/phase (checkPhaseDeliverables)
```

### 12.3 Output Format

```javascript
{
  type: 'executive-summary',
  feature: 'my-feature',
  phase: 'plan',
  summary: {
    problem: 'string',
    solution: 'string',
    functionUxEffect: 'string',
    coreValue: 'string'
  },
  nextActions: [
    { label: 'Design 진행', command: '/pdca design my-feature' },
    { label: 'Plan 수정', command: '/plan-plus my-feature' },
    { label: '팀 리뷰', command: '/pdca team my-feature' }
  ],
  metadata: {
    matchRate: null,
    iterationCount: 0,
    generatedAt: '2026-03-05T...'
  }
}
```

---

## 13. Next Steps

1. [ ] 사용자 검토 및 승인
2. [ ] `/pdca design bkit-v159-executive-summary` — 상세 설계서 작성
3. [ ] `/pdca do bkit-v159-executive-summary` — 구현 시작
4. [ ] `/pdca analyze bkit-v159-executive-summary` — Gap Analysis

---

## Appendix: Brainstorming Log

> CTO Team 6 agents 브레인스토밍 결과 (2026-03-05)

| Phase | Agent | Question/Topic | Answer/Decision | Rationale |
|-------|-------|----------------|-----------------|-----------|
| Context | Product Manager | 템플릿 삽입 포인트 | Header 아래, §1 앞 (번호 없는 ##) | 기존 섹션 번호 불변, 리뷰어 최우선 시선 |
| Context | Code Analyzer #1 | Hook 트리거 | Stop hook primary, TaskCompleted secondary | Stop이 skill 완료 시점, 의미론적 최적 |
| Context | Code Analyzer #2 | 모듈 위치 | 별도 executive-summary.js | SRP, automation.js 337행 한도, 기존 패턴 일관성 |
| Context | Frontend Architect | UX 시안 | 시안 3 (PDCA 배지 통합) + 시안 4 (미니멀) 병용 | bkit-pdca-guide 일관성 + 속독 최적화 |
| Context | Enterprise Expert | ENH 시너지 | ENH-75 CRITICAL, ENH-74/78/79 HIGH | continue:false로 CTO Team 품질 게이트 |
| Context | Product Manager #2 | Next Action UX | formatAskUserQuestion preview 슬롯 확장 | v2.1.69 호환 + 하위 호환성 유지 |
| Intent | User | 접근 방식 | 템플릿 + 출력 통합 | 문서에도 남고 대화에서도 보이는 방식 |
| YAGNI | User | v1.5.9 스코프 | 4가지 모두 포함 | 계획 상세화 → 검토 → /pdca design |
| Alternatives | CTO Team | Approach A vs B | A (통합) 선택 | 문서 자체 완결성 + 즉각 피드백 |

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 0.1 | 2026-03-05 | Initial draft (Plan Plus with CTO Team 6 agents brainstorming) | CTO Team |
