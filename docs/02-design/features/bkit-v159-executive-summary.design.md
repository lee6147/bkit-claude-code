# bkit v1.5.9 통합 상세 설계서

> **Summary**: Executive Summary 자동 생성 + AskUserQuestion Preview UX + v2.1.69 ENH 통합의 코드 레벨 상세 설계
>
> **Project**: bkit (Vibecoding Kit)
> **Version**: 1.5.9
> **Author**: CTO Team (8 agents)
> **Date**: 2026-03-05
> **Status**: Draft
> **Base Plans**:
> - `docs/01-plan/features/bkit-v159-executive-summary.plan.md` (15 FR)
> - `docs/01-plan/features/bkit-v159-askuserquestion-preview-ux.plan.md` (10 FR)

---

## Executive Summary

| 관점 | 내용 |
|------|------|
| **Problem** | PDCA 문서화 완료 후 사용자가 결과 컨텍스트 없이 Next Action을 선택해야 하며, AskUserQuestion에 preview 필드 미지원으로 선택지의 실행 결과를 미리 볼 수 없음 |
| **Solution** | 3종 템플릿에 Executive Summary 섹션 추가, `executive-summary.js` 신규 모듈, formatAskUserQuestion() preview 확장, buildNextActionQuestion() 헬퍼, ENH-74/75/78/79 통합 |
| **Function/UX Effect** | 문서 완료 시 즉시 4관점 요약 표시, Next Action 선택지에 preview로 실행 예상 결과 제공, CTO Team 품질 게이트(continue:false), 에이전트 출력 간결화 |
| **Core Value** | PDCA 사이클의 각 문서화 단계가 "행동 가능한 의사결정 도구"로 진화하여 AI-native 개발 생산성 극대화 |

---

## 1. 설계 범위

### 1.1 통합 요구사항 (25 FR)

**Plan 1 — Executive Summary (15 FR)**:
- FR-01~03: 템플릿 3종 Executive Summary 섹션 추가
- FR-04~06: 스킬/에이전트 지시 수정
- FR-07~09: executive-summary.js 신규 모듈 + 브릿지 통합
- FR-10~11: formatAskUserQuestion preview 슬롯 + Next Action 옵션 세트
- FR-12~13: ENH-74/75 스크립트 통합
- FR-14~15: 누락 export 복구 + 주석 정정

**Plan 2 — AskUserQuestion Preview UX (10 FR)**:
- P2-FR-01~02: formatAskUserQuestion() preview 필드 지원
- P2-FR-03~05: 단계별 Next Action 선택지 (plan-plus/plan/report)
- P2-FR-06: gap-detector-stop.js preview 추가
- P2-FR-07: Executive Summary → AskUserQuestion 순차 출력
- P2-FR-08: preview 3요소 (명령/소요시간/결과물)
- P2-FR-09: buildNextActionQuestion() 헬퍼 함수
- P2-FR-10: graceful degradation

### 1.2 변경 파일 총괄 (25개)

| # | 파일 | 변경 | 줄수 | FR |
|---|------|------|------|-----|
| 1 | `templates/plan-plus.template.md` | Edit | 226 | FR-01 |
| 2 | `templates/plan.template.md` | Edit | 218 | FR-02 |
| 3 | `templates/report.template.md` | Edit | 209 | FR-03 |
| 4 | `skills/pdca/SKILL.md` | Edit | 473 | FR-04 |
| 5 | `skills/plan-plus/SKILL.md` | Edit | 235 | FR-05 |
| 6 | `agents/report-generator.md` | Edit | 249 | FR-06 |
| 7 | `lib/pdca/executive-summary.js` | **New** | ~120 | FR-07 |
| 8 | `lib/pdca/index.js` | Edit | 80 | FR-08, FR-14 |
| 9 | `lib/common.js` | Edit | 292 | FR-09, FR-15 |
| 10 | `lib/pdca/automation.js` | Edit | 337 | FR-10, P2-FR-01~02, P2-FR-09 |
| 11 | `scripts/pdca-task-completed.js` | Edit | 156 | FR-12, P2-FR-04~05, P2-FR-07 |
| 12 | `scripts/team-idle-handler.js` | Edit | 86 | FR-13 |
| 13 | `scripts/gap-detector-stop.js` | Edit | 377 | P2-FR-06 |
| 14 | `scripts/subagent-start-handler.js` | Edit | 115 | ENH-74 |
| 15 | `scripts/subagent-stop-handler.js` | Edit | 82 | ENH-74 |
| 16 | `scripts/unified-stop.js` | Edit | 241 | ENH-74 |
| 17 | `agents/code-analyzer.md` | Edit | 364 | ENH-79 |
| 18 | `agents/gap-detector.md` | Edit | 330 | ENH-79 |
| 19 | `agents/design-validator.md` | Edit | 218 | ENH-79 |
| 20 | `skills/bkit-rules/SKILL.md` | Edit | 268 | ENH-76 |
| 21 | `hooks/session-start.js` | Edit | 798 | ENH-81 |
| 22 | `.claude-plugin/plugin.json` | Edit | 27 | version bump |
| 23 | `bkit.config.json` | Edit | 116 | version bump |
| 24 | `lib/pdca/status.js` | N/A | 763 | FR-14 참조 |
| 25 | `scripts/unified-bash-post.js` 등 | Edit | - | version refs |

---

## 2. 템플릿 상세 설계 (FR-01~03)

### 2.1 plan-plus.template.md (FR-01)

**현재 구조**: Header(L1-24) → §1 User Intent Discovery(L26) → ... → §10 Next Steps → Appendix
**삽입 위치**: L24(`---`) 뒤, L26(`## 1.`) 앞

**삽입 내용**:
```markdown
## Executive Summary

| 관점 | 내용 |
|------|------|
| **Problem** | {Phase 1에서 발견한 핵심 문제 1-2문장} |
| **Solution** | {Phase 2에서 선택한 접근법 요약 1-2문장} |
| **Function/UX Effect** | {Phase 3 YAGNI 통과 항목의 기대 효과} |
| **Core Value** | {비즈니스/사용자 핵심 가치 1-2문장} |

---
```

**설계 근거**:
- 번호 없는 `##` 사용으로 기존 §1~§10 번호 체계 불변
- Header `---` 뒤에 위치하여 문서 최상단에서 즉시 인지 가능
- 변수 패턴: `{...}` (기존 템플릿과 동일)

### 2.2 plan.template.md (FR-02)

**현재 구조**: Header(L1-23) → §1 Overview(L25) → ... → §8 Next Steps
**삽입 위치**: L23(`---`) 뒤, L25(`## 1.`) 앞

**삽입 내용**: plan-plus.template.md와 동일한 테이블 구조 (다만 Phase 참조 없이 일반적 설명)

```markdown
## Executive Summary

| 관점 | 내용 |
|------|------|
| **Problem** | {이 기능이 해결하는 핵심 문제} |
| **Solution** | {채택한 해결 방안 요약} |
| **Function/UX Effect** | {기능적/UX 기대 효과} |
| **Core Value** | {핵심 가치 제안} |

---
```

**영향 범위**: 8개 파일에서 참조 (최다 참조 템플릿) — skills/pdca/SKILL.md, agents/product-manager.md 등

### 2.3 report.template.md (FR-03)

**현재 구조**: `## 1. Summary`(L25) → §1.1 Project Overview(L27) → §1.2 Results Summary(L36) → `---`(L48)
**변경 방식**: `## 1. Summary` → `## Executive Summary` rename + §1.3 추가

**변경 전** (L25):
```markdown
## 1. Summary
```

**변경 후**:
```markdown
## Executive Summary

### 1.1 Project Overview
(기존 유지)

### 1.2 Results Summary
(기존 유지)

### 1.3 Value Delivered

| 관점 | 내용 |
|------|------|
| **Problem** | {해결한 핵심 문제} |
| **Solution** | {적용한 해결 방안} |
| **Function/UX Effect** | {실제 달성한 기능/UX 효과, 메트릭 포함} |
| **Core Value** | {검증된 핵심 가치} |
```

**설계 근거**:
- §1 Summary가 이미 존재하므로 rename이 가장 자연스러움
- §1.1, §1.2는 기존 그대로 유지하여 하위 호환
- §1.3에서 4관점 테이블 추가
- §2 이후 번호 불변
- 고유 변수: `{cycle_number}`, `{start_date}`, `{duration}` (기존 유지)

---

## 3. 스킬/에이전트 지시 설계 (FR-04~06)

### 3.1 skills/pdca/SKILL.md (FR-04)

**변경 위치**: plan 액션 (L78-88) 및 report 액션 (L137-145)

**plan 액션 변경** — L82(step 5) 뒤에 step 6 추가:
```markdown
6. Write `## Executive Summary` at document top (4 perspectives: Problem/Solution/Function UX Effect/Core Value, each 1-2 sentences)
```

**report 액션 변경** — L141(step 3) 뒤에 step 4 추가:
```markdown
4. Include `## Executive Summary` with `### 1.3 Value Delivered` (4 perspectives reflecting actual results)
```
기존 step 4,5 → step 5,6으로 재번호.

### 3.2 skills/plan-plus/SKILL.md (FR-05)

**변경 위치**: Phase 5 Additional sections (L176-180)

**L180 뒤에 추가**:
```markdown
- **Executive Summary** -- Auto-synthesize 4-perspective summary (Problem/Solution/Function UX Effect/Core Value) from Phases 1-4 results. Place at document top, before §1.
```

### 3.3 agents/report-generator.md (FR-06)

**변경 위치**: Feature Completion Report 섹션, Overview(L50-53) 뒤 (L54-55 사이)

**삽입 내용**:
```markdown
### Executive Summary (Required)

Generate a 4-perspective Executive Summary as `### 1.3 Value Delivered` inside `## Executive Summary`:

| Perspective | Content Guide |
|-------------|--------------|
| **Problem** | What core problem was solved? (1-2 sentences, specific) |
| **Solution** | How was it solved? (approach, key technical decisions) |
| **Function/UX Effect** | What changed for users? (measurable metrics preferred) |
| **Core Value** | Why does this matter? (business impact, user value) |

Each perspective MUST be concise (1-2 sentences max). Use specific metrics from the gap analysis when available.
```

**주의사항**: report-generator.md의 model이 `haiku`로 설정됨. haiku로도 4관점 요약은 충분히 생성 가능 (구조화된 지시이므로 고수준 추론 불필요).

---

## 4. executive-summary.js 모듈 설계 (FR-07)

### 4.1 파일 구조

```javascript
/**
 * Executive Summary Module
 * @module lib/pdca/executive-summary
 * @version 1.5.9
 */

const path = require('path');

// Lazy require
let _core = null;
function getCore() {
  if (!_core) { _core = require('../core'); }
  return _core;
}

let _status = null;
function getStatus() {
  if (!_status) { _status = require('./status'); }
  return _status;
}

let _phase = null;
function getPhase() {
  if (!_phase) { _phase = require('./phase'); }
  return _phase;
}
```

### 4.2 generateExecutiveSummary(feature, phase, context)

```javascript
/**
 * v1.5.9: Generate executive summary data structure
 * @param {string} feature - Feature name
 * @param {string} phase - Current PDCA phase ('plan'|'plan-plus'|'report')
 * @param {Object} [context] - Additional context
 * @param {number} [context.matchRate] - Gap analysis match rate
 * @param {number} [context.iterationCount] - Iteration count
 * @returns {Object} Executive summary data
 */
function generateExecutiveSummary(feature, phase, context = {}) {
  const { debugLog } = getCore();
  const { getPdcaStatusFull } = getStatus();

  const status = getPdcaStatusFull();
  const featureData = status?.features?.[feature] || {};

  const matchRate = context.matchRate ?? featureData.matchRate ?? null;
  const iterationCount = context.iterationCount ?? featureData.iterationCount ?? 0;

  debugLog('ExecutiveSummary', 'Generating summary', { feature, phase });

  return {
    type: 'executive-summary',
    feature,
    phase,
    summary: {
      problem: null,      // AI가 문서 기반으로 채움
      solution: null,
      functionUxEffect: null,
      coreValue: null
    },
    nextActions: _getNextActions(phase, feature, { matchRate }),
    metadata: {
      matchRate,
      iterationCount,
      generatedAt: new Date().toISOString()
    }
  };
}
```

**설계 근거**:
- `summary` 필드는 `null`로 초기화 — AI(스킬/에이전트)가 문서 내용 기반으로 채움
- `nextActions`는 `_getNextActions()` 내부 함수로 단계별 옵션 생성
- `metadata`에 matchRate, iterationCount 포함하여 report 시 정량 데이터 제공

### 4.3 _getNextActions(phase, feature, context) — 내부 함수

```javascript
/**
 * @private
 * @param {string} phase
 * @param {string} feature
 * @param {Object} context
 * @returns {Array<{label: string, command: string, description: string}>}
 */
function _getNextActions(phase, feature, context = {}) {
  const { matchRate = 0 } = context;

  const actionSets = {
    'plan-plus': [
      { label: 'Design 진행', command: `/pdca design ${feature}`, description: '기술 설계 문서 작성 시작' },
      { label: 'Plan 수정', command: `/plan-plus ${feature}`, description: '계획 문서 보완' },
      { label: '팀 리뷰 요청', command: `/pdca team ${feature}`, description: 'CTO Team으로 리뷰' }
    ],
    'plan': [
      { label: 'Design 진행', command: `/pdca design ${feature}`, description: 'Plan 기반으로 기술 설계 시작' },
      { label: 'Plan 수정', command: `/plan-plus ${feature}`, description: 'Plan 문서에 변경 사항 반영' },
      { label: '다른 기능 먼저', command: `/pdca status`, description: '이 feature를 보류' }
    ],
    'report': [
      { label: 'Archive', command: `/pdca archive ${feature}`, description: '완료 문서 아카이브' },
      { label: '추가 개선', command: `/pdca iterate ${feature}`, description: 'Act 단계로 이동' },
      { label: '다음 기능', command: `/pdca plan`, description: '새 기능 시작' }
    ],
    'check': matchRate >= 90
      ? [
          { label: 'Report 생성', command: `/pdca report ${feature}`, description: '완료 보고서 작성' },
          { label: '/simplify 코드 정리', command: `/simplify`, description: '코드 품질 개선 후 Report' },
          { label: '수동 수정', command: '', description: '직접 코드 수정 후 재분석' }
        ]
      : [
          { label: '자동 개선', command: `/pdca iterate ${feature}`, description: 'Act 반복으로 개선' },
          { label: 'Report 생성', command: `/pdca report ${feature}`, description: '현재 상태로 보고서' },
          { label: '수동 수정', command: '', description: '직접 코드 수정 후 재분석' }
        ]
  };

  return actionSets[phase] || actionSets['plan'];
}
```

### 4.4 formatExecutiveSummary(summary, format)

```javascript
/**
 * v1.5.9: Format executive summary for CLI output
 * @param {Object} summary - generateExecutiveSummary() 반환값
 * @param {'full'|'compact'} [format='full'] - Output format
 * @returns {string} Formatted string
 */
function formatExecutiveSummary(summary, format = 'full') {
  if (!summary) return '';

  const { feature, phase, metadata } = summary;
  const s = summary.summary || {};
  const date = metadata?.generatedAt?.split('T')[0] || '';

  if (format === 'compact') {
    return [
      `-- EXEC SUMMARY: ${feature} -- ${date} --`,
      `  PROBLEM    ${s.problem || '-'}`,
      `  SOLUTION   ${s.solution || '-'}`,
      `  EFFECT     ${s.functionUxEffect || '-'}`,
      `  VALUE      ${s.coreValue || '-'}`,
      `------------------------------------------`
    ].join('\n');
  }

  // full format
  const lines = [
    `EXECUTIVE SUMMARY`,
    `  ${feature}  (${phase})  ${date}`,
    '',
    `  [PROBLEM]`,
    `  ${s.problem || '-'}`,
    '',
    `  [SOLUTION]`,
    `  ${s.solution || '-'}`,
    '',
    `  [FUNCTION & UX EFFECT]`,
    `  ${s.functionUxEffect || '-'}`,
    '',
    `  [CORE VALUE]`,
    `  ${s.coreValue || '-'}`,
  ];

  if (summary.nextActions?.length) {
    lines.push('', `  NEXT ACTION`);
    summary.nextActions.forEach((a, i) => {
      lines.push(`  [${i + 1}] ${a.label}  ${a.command}`);
    });
  }

  return lines.join('\n');
}
```

### 4.5 generateBatchSummary(features)

```javascript
/**
 * v1.5.9: Generate batch summary for multiple features
 * @param {string[]} features - Feature names
 * @returns {Object} Batch summary with per-feature data
 */
function generateBatchSummary(features) {
  if (!Array.isArray(features) || features.length === 0) return null;

  const { debugLog } = getCore();
  const { getPdcaStatusFull } = getStatus();

  const status = getPdcaStatusFull();

  debugLog('ExecutiveSummary', 'Batch summary', { count: features.length });

  return {
    type: 'batch-summary',
    features: features.map(f => {
      const data = status?.features?.[f] || {};
      return {
        feature: f,
        phase: data.phase || 'unknown',
        matchRate: data.matchRate || null,
        iterationCount: data.iterationCount || 0
      };
    }),
    generatedAt: new Date().toISOString()
  };
}
```

### 4.6 module.exports (3개)

```javascript
module.exports = {
  generateExecutiveSummary,
  formatExecutiveSummary,
  generateBatchSummary,
};
```

---

## 5. 브릿지 통합 설계 (FR-08, FR-09, FR-14, FR-15)

### 5.1 lib/pdca/index.js (FR-08, FR-14)

**현재**: 5개 하위 모듈, 56개 export (주석은 54로 부정확)

**변경 1 — require 추가** (L11 뒤):
```javascript
const executiveSummary = require('./executive-summary');
```

**변경 2 — 누락 5개 함수 추가** (Status 섹션, L44 주석 수정):
```javascript
  // Status (24 exports)  // 기존 "17 exports" → 실제 24개
  // ... 기존 19개 유지 ...
  deleteFeatureFromStatus: status.deleteFeatureFromStatus,
  enforceFeatureLimit: status.enforceFeatureLimit,
  getArchivedFeatures: status.getArchivedFeatures,
  cleanupArchivedFeatures: status.cleanupArchivedFeatures,
  archiveFeatureToSummary: status.archiveFeatureToSummary,
```

**변경 3 — executive-summary 추가** (L78 뒤, `};` 앞):
```javascript
  // Executive Summary (3 exports)
  generateExecutiveSummary: executiveSummary.generateExecutiveSummary,
  formatExecutiveSummary: executiveSummary.formatExecutiveSummary,
  generateBatchSummary: executiveSummary.generateBatchSummary,
```

**변경 후 총 export**: 56 + 5(누락복구) + 3(신규) = **64개**

### 5.2 lib/common.js (FR-09, FR-15)

**현재**: 190개 export (주석은 186으로 부정확)

**변경 1 — Status 주석 수정** (L129):
```javascript
  // Status (24 exports)  // 기존 "17 exports" → 24
```

**변경 2 — Status 누락 5개 추가** (L148 뒤):
```javascript
  deleteFeatureFromStatus: pdca.deleteFeatureFromStatus,
  enforceFeatureLimit: pdca.enforceFeatureLimit,
  getArchivedFeatures: pdca.getArchivedFeatures,
  cleanupArchivedFeatures: pdca.cleanupArchivedFeatures,
  archiveFeatureToSummary: pdca.archiveFeatureToSummary,
```

**변경 3 — Automation 주석 수정** (L150):
```javascript
  // Automation (15 exports)  // 기존 "11 exports" → 실제 13 + buildNextActionQuestion + formatAskUserQuestion 변경 = 15
```

**변경 4 — buildNextActionQuestion 추가** (L163 뒤):
```javascript
  buildNextActionQuestion: pdca.buildNextActionQuestion,
```

**변경 5 — Executive Summary 섹션 추가** (Automation 뒤):
```javascript
  // Executive Summary (3 exports)
  generateExecutiveSummary: pdca.generateExecutiveSummary,
  formatExecutiveSummary: pdca.formatExecutiveSummary,
  generateBatchSummary: pdca.generateBatchSummary,
```

**변경 후 총 export**: 190 + 5(누락복구) + 1(buildNextActionQuestion) + 3(executive-summary) = **199개**

---

## 6. formatAskUserQuestion 확장 설계 (FR-10, P2-FR-01~02, P2-FR-10)

### 6.1 변경 위치

`lib/pdca/automation.js` L243-257

### 6.2 변경 후 코드

```javascript
/**
 * Format ask user question payload
 * v1.5.9: preview 필드 지원 추가 (ENH-78 graceful degradation)
 * @param {Object} payload
 * @returns {Object}
 */
function formatAskUserQuestion(payload) {
  const defaultOptions = [
    { label: 'Continue', description: 'Proceed with current task' },
    { label: 'Skip', description: 'Skip this step' }
  ];

  const options = (payload.options || defaultOptions).map(opt => {
    const result = {
      label: opt.label,
      description: opt.description
    };
    // v1.5.9: preview 필드 (CC v2.1.69+, 이전 버전에서는 무시됨)
    if (opt.preview) {
      result.preview = opt.preview;
    }
    return result;
  });

  return {
    questions: [
      {
        question: payload.question || 'How would you like to proceed?',
        header: payload.header || 'Action',
        options,
        multiSelect: payload.multiSelect || false
      }
    ]
  };
}
```

**설계 근거**:
- `opt.preview`가 존재할 때만 `result.preview` 추가 (graceful degradation, P2-FR-10)
- CC v2.1.68 이하에서는 preview 필드가 무시됨 (JSON에 포함되어도 에러 없음)
- 기존 호출자 100% 호환 (options에 preview가 없으면 동작 불변)

---

## 7. buildNextActionQuestion 설계 (FR-11, P2-FR-03~05, P2-FR-09)

### 7.1 추가 위치

`lib/pdca/automation.js` L257 뒤 (formatAskUserQuestion 뒤)

### 7.2 전체 코드

```javascript
/**
 * v1.5.9: PDCA 단계별 Next Action AskUserQuestion 생성
 * @param {string} phase - 완료된 PDCA 단계 ('plan'|'plan-plus'|'report'|'check')
 * @param {string} feature - Feature name
 * @param {Object} [context] - matchRate, iterCount 등
 * @returns {Object} formatAskUserQuestion 호환 payload
 */
function buildNextActionQuestion(phase, feature, context = {}) {
  const { matchRate = 0, iterCount = 0 } = context;

  const questionSets = {
    'plan-plus': {
      question: 'Plan Plus 완료. 다음 단계를 선택해주세요.',
      header: 'Next Action',
      options: [
        {
          label: 'Design 진행 (권장)',
          description: '기술 설계 문서 작성 시작',
          preview: [
            '## Design 진행',
            '',
            `**실행 명령**: \`/pdca design ${feature}\``,
            '',
            '**예상 소요**: 20~40분',
            '',
            '**결과물**:',
            `- \`docs/02-design/features/${feature}.design.md\``,
            '- API 엔드포인트 스펙',
            '- DB 스키마 및 데이터 모델',
            '- UI 컴포넌트 구조'
          ].join('\n')
        },
        {
          label: 'Plan 수정',
          description: '현재 Plan 문서에 변경 사항 반영',
          preview: [
            '## Plan 수정',
            '',
            `**대상 문서**: \`docs/01-plan/features/${feature}.plan.md\``,
            '',
            '**주요 수정 포인트**:',
            '- 범위 (In/Out Scope) 조정',
            '- 기능 요구사항 추가/제거',
            '- 성공 기준 재정의'
          ].join('\n')
        },
        {
          label: '팀 리뷰 요청',
          description: 'CTO Team에 Plan 검토 요청',
          preview: [
            '## CTO Team 리뷰',
            '',
            `**실행 명령**: \`/pdca team ${feature}\``,
            '',
            '**전제 조건**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`',
            '',
            '**예상 소요**: 10~20분'
          ].join('\n')
        }
      ]
    },

    'plan': {
      question: 'Plan 완료. 다음 단계를 선택해주세요.',
      header: 'Next Action',
      options: [
        {
          label: 'Design 진행 (권장)',
          description: 'Plan 기반으로 기술 설계 시작',
          preview: [
            '## Design 진행',
            '',
            `**실행 명령**: \`/pdca design ${feature}\``,
            '',
            '**예상 소요**: 15~30분',
            '',
            '**현재 PDCA 상태**:',
            '[Plan] OK -> **[Design]** -> [Do] -> [Check]'
          ].join('\n')
        },
        {
          label: 'Plan 수정',
          description: 'Plan 문서에 추가 변경 사항 반영',
          preview: [
            '## Plan 수정',
            '',
            `**대상**: \`docs/01-plan/features/${feature}.plan.md\``,
            '',
            '**팁**: 큰 범위 변경이면 `/plan-plus`로 재시작 검토'
          ].join('\n')
        },
        {
          label: '다른 기능 먼저',
          description: '이 feature를 보류하고 다른 작업',
          preview: [
            '## Feature 보류',
            '',
            `**현재 상태**: ${feature} -> "plan" 단계에 저장됨`,
            '',
            '**재개 방법**: `/pdca status` 후 `/pdca design ${feature}`'
          ].join('\n')
        }
      ]
    },

    'report': {
      question: `Report 완료 (Match Rate: ${matchRate}%). 다음 단계를 선택해주세요.`,
      header: 'PDCA 완료',
      options: [
        {
          label: 'Archive (권장)',
          description: '완료된 PDCA 문서를 아카이브',
          preview: [
            '## Archive',
            '',
            `**실행 명령**: \`/pdca archive ${feature}\``,
            '',
            '**이동 문서** (4개):',
            `- plan.md, design.md, analysis.md, report.md`,
            '',
            `**위치**: \`docs/archive/YYYY-MM/${feature}/\``,
            '',
            '**팁**: `--summary` 옵션으로 메트릭 보존 가능'
          ].join('\n')
        },
        {
          label: '추가 개선',
          description: '현재 feature에 추가 개선 반영',
          preview: [
            '## 추가 개선',
            '',
            `**현재 Match Rate**: ${matchRate}%`,
            `**완료된 반복**: ${iterCount}회`,
            '',
            `**실행 명령**: \`/pdca iterate ${feature}\``
          ].join('\n')
        },
        {
          label: '다음 기능',
          description: '새로운 feature PDCA 시작',
          preview: [
            '## 새 Feature 시작',
            '',
            '**시작 명령**:',
            '- `/plan-plus {new-feature}` (권장)',
            '- `/pdca plan {new-feature}`',
            '',
            `**현재 feature**: ${feature}는 "report" 단계로 유지`
          ].join('\n')
        }
      ]
    }
  };

  return questionSets[phase] || questionSets['plan'];
}
```

### 7.3 module.exports 수정

automation.js의 module.exports에 `buildNextActionQuestion` 추가:

```javascript
module.exports = {
  // ... 기존 13개 유지 ...
  buildNextActionQuestion,   // v1.5.9 추가
};
```

**변경 후 automation.js exports**: 13 → **14개**

---

## 8. 스크립트 변경 상세 설계

### 8.1 pdca-task-completed.js (FR-12, P2-FR-04~05, P2-FR-07)

**현재**: 156줄, AskUserQuestion 미사용, agent_id 미추출

#### 변경 1: agent_id/agent_type 추출 (ENH-74)

**삽입 위치**: L49-51 (taskSubject 추출) 뒤

```javascript
// v1.5.9: ENH-74 agent_id 1순위 추출
const agentId = hookContext.agent_id || null;
const agentType = hookContext.agent_type || null;
```

#### 변경 2: continue:false 품질 게이트 (ENH-75)

**삽입 위치**: L86의 response 객체 내부

```javascript
const response = {
  systemMessage: `...`,
  // v1.5.9: ENH-75 품질 게이트
  continue: (nextPhase === 'report' || nextPhase === 'completed') ? false : undefined,
  hookSpecificOutput: {
    hookEventName: 'TaskCompleted',
    agentId,     // v1.5.9: ENH-74
    agentType,   // v1.5.9: ENH-74
    feature: featureName,
    phase: detectedPhase,
    nextPhase,
  }
};
```

**continue:false 조건**: report 또는 completed 단계 도달 시 teammate 중지 (불필요한 추가 작업 방지)

#### 변경 3: AskUserQuestion 발행 (P2-FR-04~05, P2-FR-07)

**삽입 위치**: L152 (`outputAllow()`) 대체 (shouldAutoAdvance가 false일 때)

```javascript
// v1.5.9: 수동 모드에서 Next Action 제시
if (!shouldAutoAdvance(detectedPhase) && (detectedPhase === 'plan' || detectedPhase === 'report')) {
  const questionPayload = buildNextActionQuestion(detectedPhase, featureName, {
    matchRate: featureData?.matchRate || 0,
    iterCount: featureData?.iterationCount || 0
  });
  const formatted = formatAskUserQuestion(questionPayload);

  const response = {
    systemMessage: [
      `## PDCA ${detectedPhase.toUpperCase()} 완료`,
      '',
      `**Feature**: ${featureName}`,
      `**다음 단계를 선택해주세요.**`
    ].join('\n'),
    userPrompt: JSON.stringify(formatted)
  };
  console.log(JSON.stringify(response));
  process.exit(0);
}
```

**import 추가** (L15-23 영역):
```javascript
const {
  // ... 기존 imports ...
  formatAskUserQuestion,      // v1.5.9 추가
  buildNextActionQuestion,    // v1.5.9 추가
} = require('../lib/common.js');
```

#### 변경 4: 중복 패턴 제거

**현재**: L26-33의 `PDCA_TASK_PATTERNS`이 `automation.js:detectPdcaFromTaskSubject()` 내부 패턴과 완전 중복

**개선**: `detectPdcaFromTaskSubject()`를 import하여 사용

```javascript
const { detectPdcaFromTaskSubject } = require('../lib/common.js');
// L56-63의 직접 매칭 코드 제거, 대신:
const detected = detectPdcaFromTaskSubject(taskSubject);
```

### 8.2 team-idle-handler.js (FR-13)

**현재**: 86줄

#### 변경 1: agent_id/agent_type 명시적 분리 (ENH-74)

**위치**: L44 뒤

```javascript
// v1.5.9: ENH-74 agent_id 명시적 분리
const agentId = hookContext.agent_id || teammateId;
const agentType = hookContext.agent_type || 'unknown';
```

#### 변경 2: continue:false (ENH-75)

**위치**: L64 response 객체

```javascript
const response = {
  systemMessage: `Teammate ${teammateId} is idle`,
  // v1.5.9: ENH-75 nextTask 없을 때 teammate 중지
  continue: idleResult?.nextTask ? undefined : false,
  hookSpecificOutput: {
    hookEventName: 'TeammateIdle',
    teammateId,
    agentId,      // v1.5.9: ENH-74
    agentType,    // v1.5.9: ENH-74
    nextTask: idleResult?.nextTask || null,
  }
};
```

### 8.3 gap-detector-stop.js (P2-FR-06)

**현재**: 377줄, 4곳의 AskUserQuestion에 13개 option (label/description만)

#### 변경: 4곳 options에 preview 추가

**방식**: 기존 option 구조를 유지하면서 각 option에 preview 필드만 추가

**예시 — L141-145 (matchRate >= threshold 분기)**:

```javascript
options: [
  {
    label: 'Generate report (Recommended)',
    description: `Run /pdca-report ${feature || ''}`,
    preview: [
      '## 완료 보고서 생성',
      '',
      `**실행 명령**: \`/pdca report ${feature}\``,
      `**현재 Match Rate**: ${matchRate}%`,
      '',
      '**결과물**: report.md (Plan/Design/구현/분석 통합)'
    ].join('\n')
  },
  {
    label: '/simplify code cleanup',
    description: 'Improve code quality then generate report',
    preview: [
      '## /simplify 코드 정리',
      '',
      '**실행 명령**: `/simplify`',
      '',
      '코드 품질 개선 후 자동으로 Report 단계로 진행'
    ].join('\n')
  },
  {
    label: 'Continue improving',
    description: `Run /pdca-iterate ${feature || ''}`,
    preview: [
      '## 추가 개선',
      '',
      `**실행 명령**: \`/pdca iterate ${feature}\``,
      '',
      '새 iteration 시작 (Gap 재분석 포함)'
    ].join('\n')
  },
  {
    label: 'Later',
    description: 'Keep current state',
    preview: [
      '## 나중에',
      '',
      `현재 상태(${matchRate}%)가 .pdca-status.json에 저장됩니다`,
      '`/pdca status`로 나중에 확인 가능'
    ].join('\n')
  }
]
```

**나머지 3개 분기** (L173-176, L200-204, L227-231)도 동일 패턴으로 각 option에 preview 추가.

**잠재 버그 수정**: L137-149에서 `emitUserPrompt()`에 `{ questions: [...] }` 형태를 전달하는 시그니처 불일치 확인 필요. `formatAskUserQuestion()`을 사용하도록 리팩터링 검토.

### 8.4 subagent-start-handler.js (ENH-74)

**현재**: L49-52에서 agent_id를 name fallback으로만 사용

**변경**: agent_id를 별도 필드로 저장

```javascript
// v1.5.9: ENH-74 agent_id를 별도 1순위 필드로 저장
const agentId = hookContext.agent_id || null;
const agentName = hookContext.agent_name || agentId || hookContext.tool_input?.name || 'unknown';
```

response의 hookSpecificOutput에 `agentId` 추가:
```javascript
hookSpecificOutput: {
  hookEventName: 'SubagentStart',
  agentName,
  agentId,     // v1.5.9: ENH-74
  agentType,
  teamName,
}
```

### 8.5 subagent-stop-handler.js (ENH-74)

**현재**: agent_type 미사용 (gap)

**변경**:
```javascript
// v1.5.9: ENH-74 agent_id/agent_type 추출
const agentId = hookContext.agent_id || null;
const agentType = hookContext.agent_type || 'unknown';
const agentName = hookContext.agent_name || agentId || 'unknown';
```

`updateTeammateStatus`에 agentId 전달:
```javascript
teamModule.updateTeammateStatus(agentName, status, null, { agentId });
```

### 8.6 unified-stop.js (ENH-74)

**현재**: L98-122 `detectActiveAgent()`에서 agent_id 미참조

**변경 1**: L180-188 영역에서 agent_id/agent_type 추출

```javascript
// v1.5.9: ENH-74
const agentId = hookContext.agent_id || null;
const agentType = hookContext.agent_type || null;
```

**변경 2**: L193-196에서 agent handler에 전달

```javascript
const handlerResult = await executeHandler(handler, hookContext, { agentId, agentType });
```

---

## 9. ENH-79 Output Efficiency 설계

### 9.1 대상 에이전트 (3종)

동일한 지시를 3개 에이전트의 Role 섹션 직후에 삽입:

| 에이전트 | 삽입 위치 |
|----------|----------|
| agents/code-analyzer.md | L45-46 사이 |
| agents/gap-detector.md | L48-49 사이 |
| agents/design-validator.md | L47-48 사이 |

### 9.2 삽입 내용

```markdown
### Output Efficiency (v1.5.9)

- Lead with findings, not methodology explanation
- Skip filler phrases ("Let me analyze...", "I'll check...")
- Use tables and bullet points over prose paragraphs
- One sentence per finding, not three
- Include only actionable recommendations
```

---

## 10. ENH-76/81 및 기타 설계

### 10.1 skills/bkit-rules/SKILL.md (ENH-76)

**삽입 위치**: L268 뒤 (Section 8 Agent Memory Awareness 뒤)

```markdown
## 9. Plugin Hot Reload (v1.5.9)

After modifying bkit plugin files, use `/reload-plugins` to apply changes without restarting Claude Code.
- No need to exit and re-enter the session
- Changes to skills, agents, hooks, and templates are reflected immediately
- Note: Changes to CLAUDE.md require `/clear` to fully refresh
```

### 10.2 hooks/session-start.js (ENH-81)

**삽입 위치**: L612-613 사이 (CTO-Led Agent Teams Active 블록 내부)

```javascript
// v1.5.9: ENH-81 CTO Team 안정성 가이드
additionalContext += `\n### CTO Team Stability (v1.5.9)`;
additionalContext += `\n- CC v2.1.64+ resolved 4 memory leaks in Agent Teams`;
additionalContext += `\n- Long sessions (>2hr) benefit from periodic /clear`;
additionalContext += `\n- Use ctrl+f to bulk-stop background agents when done`;
```

**버전 범프**: 9곳의 `v1.5.8` → `v1.5.9` 변경 (L3, L568, L632, L638, L681, L707, L729, L785)

### 10.3 plugin.json 버전 범프

```json
"version": "1.5.9"
```

### 10.4 bkit.config.json 버전 범프

```json
"version": "1.5.9"
```

---

## 11. Executive Summary → AskUserQuestion 연결 흐름 (P2-FR-07)

### 11.1 순차 출력 방식

```
[PDCA Skill/Agent 완료]
      |
      v
[1] Stop Hook 또는 TaskCompleted Hook 실행
      |
      v
[2] systemMessage에 Executive Summary 포함
      |  - 완료 단계명, feature, 주요 결과 요약
      |  - Markdown 형식
      v
[3] CC가 systemMessage를 대화창에 출력
      |
      v
[4] userPrompt에 AskUserQuestion 포함
      |  - options에 preview 필드 포함
      v
[5] CC가 AskUserQuestion 다이얼로그 표시
      |  - 사용자가 선택지 위에 커서 -> 사이드바에 preview
      v
[6] 사용자 선택 -> 해당 액션 실행
```

### 11.2 통합 방식을 사용하지 않는 이유

- AskUserQuestion의 `header`는 간결한 제목용이며 전체 Summary 표시 공간이 아님
- Executive Summary 같은 긴 Markdown은 systemMessage로 출력이 가독성 높음
- preview는 개별 선택지의 상세 정보용으로, Summary와 역할이 다름

---

## 12. 구현 순서 및 의존성

### 12.1 Phase 1: 기반 (병렬 가능, 의존성 없음)

| 순서 | 작업 | FR/ENH |
|------|------|--------|
| 1-a | `lib/pdca/executive-summary.js` 신규 생성 | FR-07 |
| 1-b | `lib/pdca/automation.js` formatAskUserQuestion 확장 | FR-10, P2-FR-01~02 |
| 1-c | `lib/pdca/automation.js` buildNextActionQuestion 추가 | FR-11, P2-FR-09 |
| 1-d | 에이전트 3종 Output Efficiency 지시 | ENH-79 |
| 1-e | `skills/bkit-rules/SKILL.md` ENH-76 | ENH-76 |

### 12.2 Phase 2: 통합 (Phase 1 → Phase 2)

| 순서 | 작업 | FR/ENH |
|------|------|--------|
| 2-a | `lib/pdca/index.js` re-export + 누락 복구 | FR-08, FR-14 |
| 2-b | `lib/common.js` 브릿지 + 주석 정정 | FR-09, FR-15 |
| 2-c | 템플릿 3종 수정 | FR-01~03 |
| 2-d | 스킬/에이전트 지시 수정 | FR-04~06 |

### 12.3 Phase 3: 스크립트 (Phase 2 → Phase 3)

| 순서 | 작업 | FR/ENH |
|------|------|--------|
| 3-a | `pdca-task-completed.js` ENH-74/75 + AskUserQuestion | FR-12, P2-FR-04~05, P2-FR-07 |
| 3-b | `team-idle-handler.js` ENH-74/75 | FR-13 |
| 3-c | `gap-detector-stop.js` preview 추가 | P2-FR-06 |
| 3-d | `subagent-start/stop-handler.js` ENH-74 | ENH-74 |
| 3-e | `unified-stop.js` ENH-74 | ENH-74 |

### 12.4 Phase 4: 마무리

| 순서 | 작업 | FR/ENH |
|------|------|--------|
| 4-a | `hooks/session-start.js` ENH-81 + 버전 범프 | ENH-81 |
| 4-b | `plugin.json`, `bkit.config.json` 버전 범프 | version |
| 4-c | Gap Analysis | 검증 |

---

## 13. 발견된 기존 버그/불일치 (설계 중 발견)

| # | 문제 | 위치 | 심각도 | v1.5.9 수정 |
|---|------|------|--------|:----------:|
| B-01 | Status 5개 함수 3-layer 누락 | index.js, common.js | Medium | Yes (FR-14) |
| B-02 | Status 주석 "17 exports" (실제 24) | index.js L44, common.js L129 | Low | Yes (FR-15) |
| B-03 | Automation 주석 "11 exports" (실제 13) | common.js L150 | Low | Yes (FR-15) |
| B-04 | gap-detector-stop.js emitUserPrompt() 시그니처 불일치 | L137-149 | Medium | 검토 |
| B-05 | pdca-task-completed.js PDCA_TASK_PATTERNS 중복 | L26-33 | Low | Yes |
| B-06 | subagent-stop-handler.js agent_type 미사용 | 전체 | Low | Yes (ENH-74) |

---

## 14. 테스트 전략

### 14.1 단위 검증

| 대상 | 검증 항목 |
|------|----------|
| executive-summary.js | generateExecutiveSummary 반환값 구조, formatExecutiveSummary full/compact |
| automation.js | formatAskUserQuestion preview graceful degradation |
| automation.js | buildNextActionQuestion 4종 phase 반환값 |
| common.js | 199개 export 확인 |
| index.js | 64개 export 확인 |

### 14.2 통합 검증

| 시나리오 | 검증 항목 |
|----------|----------|
| /plan-plus 완료 | 문서에 Executive Summary 포함, AskUserQuestion 3종 옵션 |
| /pdca plan 완료 | 문서에 Executive Summary 포함, TaskCompleted에서 AskUserQuestion |
| /pdca report 완료 | 문서에 Executive Summary + 1.3 Value Delivered, AskUserQuestion |
| /pdca analyze 완료 | gap-detector-stop.js 4종 분기에서 preview 표시 |
| CC v2.1.68 | preview 없이 정상 동작 (graceful degradation) |
| CC v2.1.69+ | preview 사이드바 렌더링 |

### 14.3 호환성 검증

| 항목 | 기준 |
|------|------|
| 기존 190개 export | 하위 호환 유지 |
| Lazy require 패턴 | 순환 의존 없음 |
| Hook timeout | 모든 스크립트 < 5000ms |

---

## 15. 리스크 및 완화

| 리스크 | 영향 | 완화 |
|--------|------|------|
| preview 필드 CC 버전 비호환 | Low | optional 필드, graceful degradation |
| common.js 199개 export 로딩 시간 | Low | Lazy require로 실제 로딩 지연 없음 |
| report-generator (haiku) Executive Summary 품질 | Medium | 구조화된 테이블 지시로 복잡한 추론 불필요 |
| continue:false CTO Team 오작동 | High | 조건부 적용 (report/completed만), 수동 모드 비활성 |
| gap-detector-stop.js emitUserPrompt 시그니처 버그 | Medium | formatAskUserQuestion 사용으로 전환 검토 |
| session-start.js 798줄 + 추가 (한도 근접) | Low | 간결한 추가만 (4줄) |

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 0.1 | 2026-03-05 | CTO Team 8 agents 코드베이스 분석 기반 초안 | CTO Team |
