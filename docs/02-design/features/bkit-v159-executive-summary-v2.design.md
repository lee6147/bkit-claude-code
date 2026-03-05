# bkit v1.5.9 Executive Summary 재설계서 (v2)

> **Summary**: Plan/Design 25 FR 전수 조사 결과 기반, hooks 아키텍처 재설계
>
> **Project**: bkit (Vibecoding Kit)
> **Version**: 1.5.9-v2
> **Author**: CTO Team
> **Date**: 2026-03-05
> **Status**: Draft
> **Base Plans**:
> - `docs/01-plan/features/bkit-v159-executive-summary.plan.md` (15 FR)
> - `docs/01-plan/features/bkit-v159-askuserquestion-preview-ux.plan.md` (10 FR)
> **Previous Design**: `docs/02-design/features/bkit-v159-executive-summary.design.md` (v1)

---

## Executive Summary

| 관점 | 내용 |
|------|------|
| **Problem** | v1.5.9 Executive Summary를 "구현 완료"로 선언했으나, 핵심 기능인 CLI 출력이 미구현. 문서 내 삽입만 구현되고 스킬 완료 시 응답에 Executive Summary를 출력하는 메커니즘이 누락됨 |
| **Solution** | SKILL.md YAML frontmatter hooks + pdca-skill-stop.js 확장으로 Executive Summary CLI 출력 구현. bkit Feature Usage와 동일한 패턴(hooks 기반 프롬프트 주입)도 병행 |
| **Function/UX Effect** | /pdca plan, /plan-plus, /pdca report 완료 시 4관점 요약이 CLI에 즉시 출력되고, AskUserQuestion preview로 다음 단계 미리보기 제공 |
| **Core Value** | PDCA 문서 완료 = 행동 가능한 의사결정 도구. 문서 내용과 CLI 출력이 통합되어 개발 흐름 끊김 없는 연속 워크플로우 |

---

## 1. 전수 조사: Plan vs 구현 GAP 분석

### 1.1 Plan 1 — Executive Summary (15 FR)

| FR | 요구사항 | 구현 상태 | GAP |
|----|----------|-----------|-----|
| FR-01 | plan-plus.template.md Executive Summary 섹션 추가 | **구현 완료** | - |
| FR-02 | plan.template.md Executive Summary 섹션 추가 | **구현 완료** | - |
| FR-03 | report.template.md Executive Summary 섹션 추가 | **구현 완료** | - |
| FR-04 | skills/pdca/SKILL.md plan 액션에 Executive Summary 지시 | **구현 완료** (L83) | - |
| FR-05 | skills/plan-plus/SKILL.md Phase 5에 Executive Summary 지시 | **구현 완료** (L181) | - |
| FR-06 | agents/report-generator.md Executive Summary 지시 | **구현 완료** | - |
| FR-07 | lib/pdca/executive-summary.js 신규 모듈 | **구현 완료** (197줄) | **GAP-A**: summary 필드가 항상 null. 문서에서 추출하는 로직 없음 |
| FR-08 | lib/pdca/index.js re-export | **구현 완료** (65 exports) | - |
| FR-09 | lib/common.js 브릿지 통합 | **구현 완료** (199 exports) | - |
| FR-10 | formatAskUserQuestion preview 슬롯 추가 | **구현 완료** (L244-272) | - |
| FR-11 | Next Action 옵션 세트 (_getNextActions) | **구현 완료** | **GAP-B**: 영어 레이블 (설계서는 한국어 지정) |
| FR-12 | pdca-task-completed.js ENH-74/75 | **구현 완료** | - |
| FR-13 | team-idle-handler.js ENH-74/75 | **구현 완료** | - |
| FR-14 | 누락 export 복구 | **확인 필요** | 설계서 기준 5개 누락 함수 복구 여부 미확인 |
| FR-15 | 주석 수 정정 | **확인 필요** | 설계서 기준 "17 exports" → "24 exports" 정정 여부 |

### 1.2 Plan 2 — AskUserQuestion Preview UX (10 FR)

| FR | 요구사항 | 구현 상태 | GAP |
|----|----------|-----------|-----|
| P2-FR-01 | formatAskUserQuestion() preview 필드 지원 | **구현 완료** (L244-272) | - |
| P2-FR-02 | preview 존재 시만 포함 (graceful degradation) | **구현 완료** | - |
| P2-FR-03 | plan-plus 완료 시 Next Action 3종 옵션 | **구현 완료** (buildNextActionQuestion) | **GAP-B**: 영어 레이블 |
| P2-FR-04 | plan 완료 시 Next Action 3종 옵션 | **구현 완료** (buildNextActionQuestion) | **GAP-B**: 영어 레이블 |
| P2-FR-05 | report 완료 시 Next Action 3종 옵션 | **구현 완료** (buildNextActionQuestion) | **GAP-B**: 영어 레이블 |
| P2-FR-06 | gap-detector-stop.js preview 추가 | **구현 완료** | - |
| P2-FR-07 | Executive Summary systemMessage + AskUserQuestion 순차 출력 | **미구현** | **GAP-C**: 핵심 미구현 |
| P2-FR-08 | preview 3요소 (명령/소요시간/결과물) | **구현 완료** | - |
| P2-FR-09 | buildNextActionQuestion() 헬퍼 함수 | **구현 완료** (L281-429) | **GAP-B**: 영어 레이블 |
| P2-FR-10 | graceful degradation (CC v2.1.68 이하) | **구현 완료** | - |

### 1.3 GAP 요약

| GAP ID | 심각도 | 설명 |
|--------|--------|------|
| **GAP-A** | Medium | executive-summary.js의 summary 필드가 항상 null — 설계 의도대로 "AI가 채움"이나, CLI 출력 시 내용이 없음 |
| **GAP-B** | Low | buildNextActionQuestion() + _getNextActions() 레이블이 영어. 설계서는 한국어 지정 ("Design 진행 (권장)", "Plan 수정", "팀 리뷰 요청") |
| **GAP-C** | **Critical** | P2-FR-07 핵심 미구현: Executive Summary를 systemMessage로 출력하고 AskUserQuestion을 이어서 발행하는 흐름이 없음 |
| **GAP-D** | High | plan-plus 완료 시 Stop hook 미연결: unified-stop.js SKILL_HANDLERS에 'plan-plus' 없음 |
| **GAP-E** | High | pdca-skill-stop.js가 Executive Summary 출력 없이 단순 guidance + AskUserQuestion만 출력 |
| **GAP-F** | Medium | SKILL.md body에 "CLI 응답에도 Executive Summary 출력" 지시 없음 (문서 작성 지시만 있음) |

---

## 2. 구현 방식 분석: hooks 아키텍처

### 2.1 bkit Feature Usage 구현 방식 (참조 패턴)

```
SessionStart hook (hooks.json)
    → session-start.js (L729-790)
    → additionalContext에 프롬프트 지시 주입:
      "Rule: Include the following format at the end of every response..."
    → Claude가 매 응답에서 지시를 따라 Feature Usage 출력
```

**핵심**: 코드가 자동 생성하는 것이 아니라, **hooks가 프롬프트 지시를 주입** → AI가 지시를 따르는 구조.

### 2.2 현재 PDCA Stop Hook 아키텍처

```
hooks.json "Stop" event
    → unified-stop.js
    → getActiveSkill() (skill-post.js에서 setActiveSkill)
    → SKILL_HANDLERS['pdca'] → pdca-skill-stop.js
    → guidance + AskUserQuestion 출력 (Executive Summary 없음)
```

**문제점**:
1. `pdca-skill-stop.js`가 Executive Summary 내용을 출력하지 않음
2. `plan-plus`가 SKILL_HANDLERS에 등록되지 않아 Stop 시 handler 없음
3. SKILL.md YAML frontmatter hooks가 `# hooks:` 주석으로 비활성 (GitHub #9354 workaround)

### 2.3 제안: SKILL.md YAML hooks + 기존 아키텍처 통합

CC v2.1.69에서 Plugin Stop/SessionEnd hook 발화 수정됨.
SKILL.md YAML frontmatter hooks 사용 가능성 재평가 필요.

**그러나** 현실적 제약:
- bkit 전체 11개 스킬이 `# hooks:` 주석으로 통일 (일관성)
- hooks.json → unified-stop.js 아키텍처가 이미 안정적
- SKILL.md hooks + hooks.json hooks 동시 발화 시 중복 실행 위험

**결론**: 현재 아키텍처(hooks.json → unified-stop.js) 내에서 Executive Summary 출력 추가가 가장 안전하고 효과적.

---

## 3. 설계: Executive Summary CLI 출력 구현

### 3.1 접근법 선택

| 접근법 | 장점 | 단점 | 선택 |
|--------|------|------|------|
| A. SKILL.md body 프롬프트 지시 | Feature Usage와 동일 패턴, 코드 변경 최소 | AI 지시 미준수 가능성 | **보조** |
| B. pdca-skill-stop.js 확장 | 기존 아키텍처 활용, 안정적 | Executive Summary 내용을 hook에서 생성해야 함 | **주력** |
| C. SKILL.md YAML hooks | 아키텍처적으로 올바름 | #9354 workaround 해제 필요, 중복 실행 위험 | **향후** |

**선택: B(주력) + A(보조)**

- **B**: `pdca-skill-stop.js`에서 Executive Summary 포맷 출력 + AskUserQuestion 발행
- **A**: SKILL.md body에 "응답 시 Executive Summary도 함께 출력" 프롬프트 지시 추가

### 3.2 구현 1: pdca-skill-stop.js 확장 (GAP-C, GAP-E 해소)

**현재 흐름** (L375-400):
```javascript
const response = {
  decision: 'allow',
  guidance: guidance || null,
  userPrompt: userPrompt,
  systemMessage: guidance ? (...) : null
};
```

**변경 후** — plan/report 액션에 Executive Summary 포함:
```javascript
// v1.5.9: Executive Summary CLI 출력 (P2-FR-07)
const {
  // 기존 imports...
  buildNextActionQuestion,
  formatAskUserQuestion,
  generateExecutiveSummary,
  formatExecutiveSummary,
} = require('../lib/common.js');

// ... (기존 코드) ...

// plan 또는 report 완료 시 Executive Summary 출력
if (action === 'plan' || action === 'report') {
  const summary = generateExecutiveSummary(feature, action);
  const summaryText = formatExecutiveSummary(summary, 'full');

  // AskUserQuestion with preview (buildNextActionQuestion)
  const pdcaStatus = getPdcaStatusFull();
  const featureData = pdcaStatus?.features?.[feature] || {};
  const questionPayload = buildNextActionQuestion(action, feature, {
    matchRate: featureData.matchRate || 0,
    iterCount: featureData.iterationCount || 0
  });
  const formatted = formatAskUserQuestion(questionPayload);

  const response = {
    decision: 'allow',
    hookEventName: 'Skill:pdca:Stop',
    systemMessage: [
      `## Executive Summary`,
      '',
      summaryText,
      '',
      `---`,
      '',
      guidance
    ].join('\n'),
    userPrompt: JSON.stringify(formatted)
  };
  console.log(JSON.stringify(response));
  process.exit(0);
}
```

**핵심 변경점**:
- `systemMessage`에 Executive Summary 텍스트 포함
- `userPrompt`에 AskUserQuestion (preview 포함) 포함
- P2-FR-07 "순차 출력" 패턴 구현: systemMessage(Executive Summary) → userPrompt(AskUserQuestion)

### 3.3 구현 2: plan-plus Stop Handler 추가 (GAP-D 해소)

**unified-stop.js SKILL_HANDLERS 추가**:
```javascript
const SKILL_HANDLERS = {
  'pdca': './pdca-skill-stop.js',
  'plan-plus': './plan-plus-stop.js',  // v1.5.9: 신규 추가
  // ... 나머지 유지 ...
};
```

**scripts/plan-plus-stop.js** 신규 생성:
```javascript
#!/usr/bin/env node
/**
 * plan-plus-stop.js - Plan Plus Skill Stop Hook (v1.5.9)
 *
 * Plan Plus 완료 시:
 * 1. Executive Summary를 systemMessage로 출력
 * 2. AskUserQuestion으로 Next Action 제시 (preview 포함)
 */

const {
  readStdinSync,
  debugLog,
  getPdcaStatusFull,
  extractFeatureFromContext,
  buildNextActionQuestion,
  formatAskUserQuestion,
  generateExecutiveSummary,
  formatExecutiveSummary,
  outputAllow,
} = require('../lib/common.js');

debugLog('Skill:plan-plus:Stop', 'Hook started');

let input;
try {
  input = readStdinSync();
} catch (e) {
  debugLog('Skill:plan-plus:Stop', 'stdin read failed', { error: e.message });
  process.exit(0);
}

const inputText = typeof input === 'string' ? input : JSON.stringify(input);
const currentStatus = getPdcaStatusFull();
const feature = extractFeatureFromContext({ agentOutput: inputText, currentStatus });

if (!feature) {
  outputAllow('Plan Plus completed.', 'Skill:plan-plus:Stop');
  process.exit(0);
}

// Executive Summary 생성
const summary = generateExecutiveSummary(feature, 'plan-plus');
const summaryText = formatExecutiveSummary(summary, 'full');

// AskUserQuestion (preview 포함)
const questionPayload = buildNextActionQuestion('plan-plus', feature);
const formatted = formatAskUserQuestion(questionPayload);

const response = {
  decision: 'allow',
  hookEventName: 'Skill:plan-plus:Stop',
  systemMessage: [
    `## Plan Plus 완료: ${feature}`,
    '',
    summaryText,
    '',
    `---`,
    '',
    `Plan 문서가 생성되었습니다.`,
    `다음 단계를 선택해주세요.`
  ].join('\n'),
  userPrompt: JSON.stringify(formatted)
};

console.log(JSON.stringify(response));
process.exit(0);
```

### 3.4 구현 3: SKILL.md body 프롬프트 지시 추가 (GAP-F 해소, 보조 메커니즘)

Feature Usage와 동일한 패턴: SKILL.md body에 AI 지시를 추가하여 응답에 Executive Summary 포함 유도.

**skills/pdca/SKILL.md plan 액션 변경** (L83 뒤에 추가):
```markdown
6. Write `## Executive Summary` at document top with 4-perspective table (Problem/Solution/Function UX Effect/Core Value), each 1-2 sentences
7. **MANDATORY**: After completing the document, include Executive Summary in your response as well. Format:
   ```
   ## Executive Summary
   | 관점 | 내용 |
   |------|------|
   | **Problem** | {핵심 문제} |
   | **Solution** | {해결 방안} |
   | **Function/UX Effect** | {기능/UX 효과} |
   | **Core Value** | {핵심 가치} |
   ```
```

**skills/pdca/SKILL.md report 액션 변경** (L144 뒤에 추가):
```markdown
5. **MANDATORY**: After completing the report, include Executive Summary in your response. Same 4-perspective table format as plan.
```

**skills/plan-plus/SKILL.md Phase 5 변경** (L181 뒤에 추가):
```markdown
- **Executive Summary Response** -- MANDATORY: After generating the Plan document, also output the Executive Summary table in your response (not just in the document). This enables the user to immediately see the summary without opening the file.
```

### 3.5 구현 4: 한국어 레이블 적용 (GAP-B 해소)

**lib/pdca/executive-summary.js `_getNextActions()`** 변경:

```javascript
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
```

**lib/pdca/automation.js `buildNextActionQuestion()`** 변경:

```javascript
'plan-plus': {
  question: 'Plan Plus 완료. 다음 단계를 선택해주세요.',
  // ...options labels:
  // 'Design 진행 (권장)', 'Plan 수정', '팀 리뷰 요청'
},
'plan': {
  question: 'Plan 완료. 다음 단계를 선택해주세요.',
  // 'Design 진행 (권장)', 'Plan 수정', '다른 기능 먼저'
},
'report': {
  question: `Report 완료 (Match Rate: ${matchRate}%). 다음 단계를 선택해주세요.`,
  // 'Archive (권장)', '추가 개선', '다음 기능'
}
```

---

## 4. 변경 파일 총괄

| # | 파일 | 변경 유형 | GAP | 설명 |
|---|------|-----------|-----|------|
| 1 | `scripts/pdca-skill-stop.js` | Edit | GAP-C, GAP-E | Executive Summary + AskUserQuestion 출력 추가 |
| 2 | `scripts/plan-plus-stop.js` | **New** | GAP-D | Plan Plus Stop handler 신규 생성 |
| 3 | `scripts/unified-stop.js` | Edit | GAP-D | SKILL_HANDLERS에 'plan-plus' 추가 |
| 4 | `skills/pdca/SKILL.md` | Edit | GAP-F | plan/report body에 "응답에도 출력" 프롬프트 지시 추가 |
| 5 | `skills/plan-plus/SKILL.md` | Edit | GAP-F | Phase 5에 "응답에도 출력" 프롬프트 지시 추가 |
| 6 | `lib/pdca/executive-summary.js` | Edit | GAP-B | _getNextActions() 한국어 레이블 |
| 7 | `lib/pdca/automation.js` | Edit | GAP-B | buildNextActionQuestion() 한국어 레이블 |

### 4.1 기존 v1 설계서 대비 추가/변경

| v1 설계서 파일 | v2 변경 사항 |
|----------------|-------------|
| pdca-skill-stop.js (미언급) | **신규 추가**: Executive Summary 출력 로직 |
| plan-plus-stop.js (미존재) | **신규 추가**: Plan Plus 전용 Stop handler |
| unified-stop.js (미언급) | **신규 추가**: SKILL_HANDLERS plan-plus 등록 |
| SKILL.md body 지시 (미언급) | **신규 추가**: "응답에도 출력" 프롬프트 지시 |

---

## 5. Executive Summary CLI 출력 흐름 (최종)

### 5.1 /pdca plan 완료 시

```
[1] 사용자가 /pdca plan {feature} 실행
[2] Claude가 SKILL.md 지시 따라 plan.template.md 기반 문서 작성
    → 문서 내 ## Executive Summary 4관점 테이블 포함
[3] Claude가 SKILL.md body 지시 따라 응답에도 Executive Summary 출력
[4] PostToolUse(Skill) → skill-post.js → setActiveSkill('pdca')
[5] Stop → unified-stop.js → pdca-skill-stop.js
    → action='plan' 감지
    → systemMessage: Executive Summary 텍스트
    → userPrompt: AskUserQuestion (3종 옵션 + preview)
[6] CC가 systemMessage 출력 후 AskUserQuestion 다이얼로그 표시
[7] 사용자가 선택 → 해당 액션 실행
```

### 5.2 /plan-plus 완료 시

```
[1] 사용자가 /plan-plus {feature} 실행
[2] Claude가 Phase 0-5 진행, Plan 문서 생성
    → 문서 내 ## Executive Summary 포함
[3] Claude가 SKILL.md body 지시 따라 응답에도 Executive Summary 출력
[4] PostToolUse(Skill) → skill-post.js → setActiveSkill('plan-plus')
[5] Stop → unified-stop.js → plan-plus-stop.js (신규)
    → systemMessage: Executive Summary 텍스트
    → userPrompt: AskUserQuestion (3종 옵션 + preview)
[6] CC가 순차 출력
```

### 5.3 /pdca report 완료 시

```
[1] 사용자가 /pdca report {feature} 실행 (또는 report-generator Agent 완료)
[2] report-generator가 report.template.md 기반 보고서 작성
    → 문서 내 ## Executive Summary + ### 1.3 Value Delivered 포함
[3] Stop → unified-stop.js → pdca-skill-stop.js
    → action='report' 감지
    → systemMessage: Executive Summary (matchRate 포함)
    → userPrompt: AskUserQuestion (Archive/추가개선/다음기능)
```

### 5.4 이중 안전장치 설계 근거

| 계층 | 역할 | 실패 시 |
|------|------|---------|
| **SKILL.md body 지시** (A) | AI가 응답에 Executive Summary 직접 출력 | AI가 지시 미준수 가능 |
| **Stop hook 스크립트** (B) | 프로그래밍으로 systemMessage에 Executive Summary 포함 | hook 실행 실패 시 미출력 |

두 계층 중 하나만 작동해도 사용자는 Executive Summary를 볼 수 있음.

---

## 6. GAP-A 해소 전략: summary 필드 null 문제

### 6.1 현재 상태

`executive-summary.js`의 `generateExecutiveSummary()`가 반환하는 summary 객체:
```javascript
summary: {
  problem: null,      // 항상 null
  solution: null,
  functionUxEffect: null,
  coreValue: null
}
```

### 6.2 설계 의도

설계서 원문: "AI(스킬/에이전트)가 문서 내용 기반으로 채움"
→ 즉, 이 함수는 **구조만 제공**하고 내용은 AI가 채우는 의도.

### 6.3 해소 방안

**방안 A (현재 유지)**: summary 필드는 null로 유지. Stop hook의 `formatExecutiveSummary()`가 `-` 로 표시. 실제 내용은 AI가 SKILL.md body 지시를 따라 직접 출력.

**방안 B (향후 개선)**: Stop hook 스크립트에서 생성된 문서를 읽어 Executive Summary 섹션을 파싱하여 summary 필드를 채움.

**선택: A (현재 유지)** — 이유:
- Stop hook에서 문서 파싱은 timeout 위험 (5000ms)
- SKILL.md body 지시로 AI가 직접 출력하는 것이 더 자연스럽고 정확
- hook의 systemMessage는 보조 역할 (구조적 안내)

---

## 7. 향후: SKILL.md YAML hooks 마이그레이션 계획

### 7.1 현재 제약

- GitHub #9354: `${CLAUDE_PLUGIN_ROOT}` 미확장 문제 (hooks.json에서는 해결됨)
- CC v2.1.69: Plugin Stop/SessionEnd hook 발화 수정됨
- CC v2.1.54: `${CLAUDE_PLUGIN_ROOT}` frontmatter 치환 수정됨

### 7.2 마이그레이션 조건

CC v2.1.69+에서 SKILL.md hooks 안정성이 확인되면:

```yaml
# skills/pdca/SKILL.md
hooks:
  - event: Stop
    command: "node ${CLAUDE_PLUGIN_ROOT}/scripts/pdca-skill-stop.js"
    timeout: 10000
```

```yaml
# skills/plan-plus/SKILL.md
hooks:
  - event: Stop
    command: "node ${CLAUDE_PLUGIN_ROOT}/scripts/plan-plus-stop.js"
    timeout: 10000
```

**마이그레이션 시 필요 작업**:
1. hooks.json에서 해당 스킬의 Stop handler 제거 (중복 실행 방지)
2. unified-stop.js SKILL_HANDLERS에서 해당 항목 제거
3. SKILL.md `# hooks:` 주석 해제 및 활성화
4. 테스트: SKILL.md hooks만으로 정상 동작 확인

### 7.3 v1.5.9에서의 결정

v1.5.9에서는 **기존 hooks.json 아키텍처 유지**. 이유:
- 11개 스킬 전체의 `# hooks:` 주석 해제는 대규모 변경
- 테스트 범위가 크게 확대됨
- Executive Summary 출력 기능 추가에 집중

SKILL.md YAML hooks 마이그레이션은 **v1.6.0 이후** CC 안정성 확인 후 진행.

---

## 8. 구현 순서

| 순서 | 작업 | 파일 | GAP |
|------|------|------|-----|
| 1 | executive-summary.js 한국어 레이블 | lib/pdca/executive-summary.js | GAP-B |
| 2 | buildNextActionQuestion 한국어 레이블 | lib/pdca/automation.js | GAP-B |
| 3 | plan-plus-stop.js 신규 생성 | scripts/plan-plus-stop.js | GAP-D |
| 4 | unified-stop.js에 plan-plus 등록 | scripts/unified-stop.js | GAP-D |
| 5 | pdca-skill-stop.js Executive Summary 출력 | scripts/pdca-skill-stop.js | GAP-C, GAP-E |
| 6 | SKILL.md body 프롬프트 지시 추가 | skills/pdca/SKILL.md, skills/plan-plus/SKILL.md | GAP-F |
| 7 | Gap Analysis 실행 | - | 검증 |

---

## 9. bkit Feature Usage vs Executive Summary 비교

| 항목 | bkit Feature Usage | Executive Summary |
|------|-------------------|-------------------|
| **구현 방식** | SessionStart hook → 프롬프트 지시 주입 | Stop hook → systemMessage + SKILL.md body 프롬프트 지시 |
| **트리거 시점** | 세션 시작 시 1회 주입 → 매 응답 적용 | 스킬 완료 시 (plan/plan-plus/report) |
| **출력 주체** | AI가 지시 따라 직접 출력 | hook 스크립트(systemMessage) + AI(body 지시) 이중 |
| **hook 사용 여부** | SessionStart hook (hooks.json) | Stop hook (hooks.json → unified-stop.js) |
| **프롬프트 vs 코드** | 순수 프롬프트 지시 (코드 없음) | 코드(hook 스크립트) + 프롬프트(SKILL.md body) 혼합 |

**공통점**: 둘 다 **hooks를 활용한 프롬프트 주입** 패턴. 차이는 주입 시점과 지속성.

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 0.1 | 2026-03-05 | v1 설계서 (CTO Team 8 agents) | CTO Team |
| 0.2 | 2026-03-05 | v2 재설계: 전수 조사 + hooks 아키텍처 재설계 | CTO Team |
