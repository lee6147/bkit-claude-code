# AskUserQuestion Preview & Next Action UX Planning Document

> **Summary**: v2.1.69 AskUserQuestion preview 필드를 활용하여 PDCA 단계 완료 후 Executive Summary와 Next Action 선택 UX를 설계한다
>
> **Project**: bkit (Claude Code Plugin)
> **Version**: v1.5.9 (ENH-78)
> **Author**: Product Manager Agent
> **Date**: 2026-03-05
> **Status**: Draft

---

## 1. Overview

### 1.1 Purpose

Claude Code v2.1.69에서 AskUserQuestion의 options에 `preview` 필드가 추가되었다. 사용자가 선택지 위에 커서를 올리면 사이드바에 Markdown이 렌더링된다. 이 기능을 활용하여 PDCA 단계 완료 후 표시되는 Next Action 선택 UX를 개선한다.

현재 문제:
- `formatAskUserQuestion()` (lib/pdca/automation.js:243)은 `question`, `header`, `options`, `multiSelect`만 지원하며 `preview` 필드가 없다
- gap-detector-stop.js 등의 스크립트에서 AskUserQuestion을 직접 조립하여 preview 없이 제공한다
- 사용자는 선택지의 label과 description만 보고 결정해야 하며, 해당 선택이 다음 단계에서 어떤 일이 벌어지는지 미리 알 수 없다

해결 목표:
- Executive Summary (완료 보고) 직후 Next Action 선택지를 preview 필드와 함께 제시한다
- 각 선택지에 preview로 "선택 시 실행 내용 + 예상 결과"를 Markdown 형식으로 보여준다
- PDCA 단계(plan-plus 후, pdca plan 후, pdca report 후)별로 맥락에 맞는 선택지 세트를 제공한다

### 1.2 Background

ENH-78은 v2.1.69 Impact Analysis (docs/01-plan/features/claude-code-v2169-impact-analysis.plan.md)에서 원래 v1.6.0 Hard 항목으로 분류되었으나, v1.5.9 Executive Summary 기능과 시너지가 높아 v1.5.9로 통합되었다.

현재 AskUserQuestion 사용 현황:
- `gap-detector-stop.js`: match rate 기반으로 4종 선택지 조립 (preview 없음)
- `pdca-task-completed.js`: shouldAutoAdvance() 결과에 따라 단순 안내만 출력 (AskUserQuestion 미사용)
- Plan Plus SKILL.md Phase 1~4: AskUserQuestion을 순차적으로 사용하나 preview 미활용

### 1.3 Related Documents

- Impact Analysis: `docs/01-plan/features/claude-code-v2169-impact-analysis.plan.md` (Phase 3, ENH-78)
- 현재 구현: `lib/pdca/automation.js` (formatAskUserQuestion, line 243)
- Gap Detector 스크립트: `scripts/gap-detector-stop.js`
- Task Completed 훅: `scripts/pdca-task-completed.js`

---

## 2. Scope

### 2.1 In Scope

- [ ] `formatAskUserQuestion()` 확장: options 내 `preview` 필드 지원 추가
- [ ] PDCA 단계별 Next Action 선택지 세트 정의 (plan-plus 후, pdca plan 후, pdca report 후)
- [ ] `gap-detector-stop.js` 업데이트: preview 필드 포함한 선택지 조립
- [ ] `pdca-task-completed.js` 업데이트: plan/report 완료 시 AskUserQuestion 발행
- [ ] Executive Summary 연결 패턴 정의: 순차 출력 방식 (systemMessage → AskUserQuestion)
- [ ] 각 선택지 preview 내용 설계 (Markdown 형식 템플릿)

### 2.2 Out of Scope

- AskUserQuestion preview의 렌더링 엔진 수정 (CC 내부 기능)
- 신규 스크립트 파일 추가 (기존 파일 수정으로만 구현)
- HTTP hooks 방식으로의 전환 (command type 유지)
- multiSelect와 preview의 복합 사용 (첫 버전은 단일 선택만)
- 음성 입력(STT)과 preview 통합

---

## 3. Requirements

### 3.1 Functional Requirements

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-01 | `formatAskUserQuestion()`의 options 항목에 선택적 `preview` 필드를 지원한다 | Must | Pending |
| FR-02 | preview 필드 값은 Markdown 문자열이며, CC가 사이드바에 렌더링한다 | Must | Pending |
| FR-03 | plan-plus 완료 후 3종 Next Action 선택지(Design 진행 / Plan 수정 / 팀 리뷰 요청)를 preview와 함께 제시한다 | Must | Pending |
| FR-04 | pdca plan 완료 후 3종 Next Action 선택지(Design 진행 / Plan 수정 / 다음 기능)를 preview와 함께 제시한다 | Must | Pending |
| FR-05 | pdca report 완료 후 3종 Next Action 선택지(Archive / 추가 개선 / 다음 기능)를 preview와 함께 제시한다 | Must | Pending |
| FR-06 | gap-detector-stop.js의 기존 AskUserQuestion에 preview 필드를 추가한다 | Should | Pending |
| FR-07 | Executive Summary는 systemMessage로 먼저 출력하고, AskUserQuestion은 이어서 별도 발행한다 | Must | Pending |
| FR-08 | preview 내용은 "선택 시 실행 명령 + 예상 소요 시간 + 예상 결과물" 3요소를 포함한다 | Should | Pending |
| FR-09 | `buildNextActionQuestion()` 헬퍼 함수를 lib/pdca/automation.js에 추가하여 단계별 선택지 세트를 반환한다 | Must | Pending |
| FR-10 | preview 규격 불안정 대응: preview 필드 미지원 CC 버전에서도 graceful degradation으로 동작한다 | Must | Pending |

### 3.2 Non-Functional Requirements

| Category | Criteria | Measurement Method |
|----------|----------|-------------------|
| 호환성 | CC v2.1.68 이하에서도 preview 없이 정상 동작 | 수동 테스트 (v2.1.68 환경) |
| 코드 크기 | formatAskUserQuestion() 증가량 < 50 lines | 코드 리뷰 |
| 응답 지연 | preview 생성 연산이 hook 응답을 50ms 이상 지연하지 않음 | 로그 기반 측정 |
| 유지보수성 | 선택지 세트는 상수 또는 함수로 중앙화하여 SKILL.md 수정 없이 변경 가능 | 코드 구조 검토 |

---

## 4. UX 흐름도 설계

### 4.1 Executive Summary와 AskUserQuestion 연결 방식

**결론: 순차 출력 방식 (Sequential)**

```
[Stop Hook 실행]
      │
      ▼
[1] systemMessage에 Executive Summary 포함
      │  - 완료 단계명, feature, 주요 결과 요약
      │  - Markdown 형식 (헤더, 체크리스트)
      ▼
[2] CC가 systemMessage를 대화창에 출력
      │
      ▼
[3] userPrompt에 AskUserQuestion 포함
      │  - options에 preview 필드 포함
      ▼
[4] CC가 AskUserQuestion 다이얼로그 표시
      │  - 사용자가 선택지 위에 커서 → 사이드바에 preview 렌더링
      ▼
[5] 사용자 선택
      │
      ▼
[6] 선택된 액션 실행
```

**통합 방식을 택하지 않는 이유**:
- CC의 AskUserQuestion 다이얼로그는 question/options 표시에 최적화
- Executive Summary 같은 긴 Markdown은 systemMessage로 출력이 더 가독성 높음
- AskUserQuestion의 `header`는 간결한 제목용이며 전체 Summary 표시 공간이 아님

### 4.2 preview 필드 설계 원칙

preview에 포함할 3요소:
```
## {액션 이름}

**실행 명령**: `/pdca {command} {feature}`

**예상 소요**: {시간}

**결과물**:
- {결과물 1}
- {결과물 2}

**다음 단계 미리보기**:
{간략한 다음 단계 설명}
```

---

## 5. 시나리오별 UX 설계

### 시나리오 1: plan-plus 완료 후 Next Action

**발생 지점**: skills/plan-plus/SKILL.md Phase 5 (Plan Document Generation) 직후

**Executive Summary** (systemMessage):
```markdown
## Plan Plus 완료

**Feature**: {feature}
**문서 위치**: docs/01-plan/features/{feature}.plan.md

### 결과 요약
- 의도 탐색 (Phase 1): 핵심 목적 확정
- 대안 탐색 (Phase 2): {N}개 접근법 비교, {선택 접근법} 채택
- YAGNI 검토 (Phase 3): {K}개 항목 범위 제외
- 설계 검증 (Phase 4): 섹션별 승인 완료

### 선택지를 확인하고 다음 단계를 선택해주세요.
```

**AskUserQuestion 선택지**:

| Label | Description | Preview 내용 |
|-------|-------------|-------------|
| Design 진행 (권장) | 기술 설계 문서 작성 시작 | `/pdca design {feature}` 실행, 예상 시간 20~40분, 결과물: design.md (API 스펙, DB 스키마, 컴포넌트 구조) |
| Plan 수정 | 현재 Plan 문서 수정 | 현재 plan.md 재편집, 완료 후 다시 이 화면으로 복귀 가능 |
| 팀 리뷰 요청 | CTO Team에 Plan 검토 요청 | `/pdca team {feature}` 실행, CTO Lead가 Plan 검토 후 Design 단계로 진행 |

### 시나리오 2: pdca plan 완료 후 Next Action

**발생 지점**: pdca-task-completed.js에서 `[Plan] {feature}` Task 완료 감지 시

**Executive Summary** (systemMessage):
```markdown
## PDCA Plan 완료

**Feature**: {feature}
**문서**: docs/01-plan/features/{feature}.plan.md
**PDCA 상태**: plan → 다음 단계 선택 대기

### 현재까지 진행 상황
[Plan] ✅ → [Design] ⏳ → [Do] ⏳ → [Check] ⏳ → [Report] ⏳

### 다음 단계를 선택해주세요.
```

**AskUserQuestion 선택지**:

| Label | Description | Preview 내용 |
|-------|-------------|-------------|
| Design 진행 (권장) | Plan 기반으로 기술 설계 시작 | `/pdca design {feature}` 실행, 예상 시간 15~30분, 결과물: design.md, Plan 문서를 기반으로 API/DB/UI 설계 |
| Plan 수정 | Plan 문서에 변경 사항 반영 | docs/01-plan/features/{feature}.plan.md 직접 편집 안내, 주요 수정 포인트: 범위, 요구사항, 성공 기준 |
| 다른 기능 먼저 | 이 feature를 보류하고 다른 작업 | 현재 PDCA 상태는 "plan" 단계에 저장됨, `/pdca status`로 나중에 재개 가능 |

### 시나리오 3: pdca report 완료 후 Next Action

**발생 지점**: scripts/pdca-skill-stop.js 또는 report-generator 에이전트 Stop hook에서 report 완료 감지 시

**Executive Summary** (systemMessage):
```markdown
## PDCA Report 완료

**Feature**: {feature}
**최종 Match Rate**: {matchRate}%
**반복 횟수**: {iterCount}회
**보고서**: docs/04-report/features/{feature}.report.md

### PDCA 사이클 완료
[Plan] ✅ → [Design] ✅ → [Do] ✅ → [Check] ✅ → [Report] ✅

### 완료 축하드립니다! 다음 단계를 선택해주세요.
```

**AskUserQuestion 선택지**:

| Label | Description | Preview 내용 |
|-------|-------------|-------------|
| Archive (권장) | 완료된 PDCA 문서를 보관 | `/pdca archive {feature}` 실행, 4개 문서(plan/design/analysis/report)를 docs/archive/YYYY-MM/으로 이동, .pdca-status.json에서 정리 |
| 추가 개선 | 현재 feature에 개선 사항 반영 | `/pdca iterate {feature}` 실행, 현재 Match Rate {matchRate}% 기준으로 추가 갭 분석, 새 iteration 시작 |
| 다음 기능 | 새로운 feature PDCA 시작 | `/pdca plan {new-feature}` 또는 `/plan-plus {new-feature}` 실행, 현재 feature는 report 상태로 유지 |

---

## 6. 코드 설계

### 6.1 formatAskUserQuestion() 확장

**변경 전** (lib/pdca/automation.js:243):
```javascript
function formatAskUserQuestion(payload) {
  return {
    questions: [{
      question: payload.question || 'How would you like to proceed?',
      header: payload.header || 'Action',
      options: payload.options || [
        { label: 'Continue', description: 'Proceed with current task' },
        { label: 'Skip', description: 'Skip this step' }
      ],
      multiSelect: payload.multiSelect || false
    }]
  };
}
```

**변경 후** (preview 필드 지원 추가):
```javascript
function formatAskUserQuestion(payload) {
  const options = (payload.options || [
    { label: 'Continue', description: 'Proceed with current task' },
    { label: 'Skip', description: 'Skip this step' }
  ]).map(opt => {
    // preview 필드가 있으면 포함, 없으면 생략 (graceful degradation)
    const result = {
      label: opt.label,
      description: opt.description
    };
    if (opt.preview) {
      result.preview = opt.preview;
    }
    return result;
  });

  return {
    questions: [{
      question: payload.question || 'How would you like to proceed?',
      header: payload.header || 'Action',
      options,
      multiSelect: payload.multiSelect || false
    }]
  };
}
```

### 6.2 buildNextActionQuestion() 신규 헬퍼 함수

lib/pdca/automation.js에 추가:

```javascript
/**
 * v1.5.9 ENH-78: PDCA 단계별 Next Action AskUserQuestion 생성
 * @param {string} phase - 완료된 PDCA 단계 ('plan', 'plan-plus', 'report')
 * @param {string} feature - feature 이름
 * @param {Object} context - 추가 컨텍스트 (matchRate, iterCount 등)
 * @returns {Object} formatAskUserQuestion 호환 payload
 */
function buildNextActionQuestion(phase, feature, context = {}) {
  const { matchRate = 0, iterCount = 0 } = context;

  const questionSets = {
    'plan-plus': {
      question: `Plan Plus 완료. 다음 단계를 선택해주세요.`,
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
            '- `docs/02-design/features/' + feature + '.design.md`',
            '- API 엔드포인트 스펙',
            '- DB 스키마 및 데이터 모델',
            '- UI 컴포넌트 구조',
            '',
            '**다음 단계 미리보기**:',
            'Plan 문서를 기반으로 기술적 세부 사항을 설계합니다.',
            'design-validator 에이전트가 Plan 일관성을 검증합니다.'
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
            '- 성공 기준 재정의',
            '- YAGNI 검토 결과 반영',
            '',
            '**완료 후**: Plan Plus를 다시 실행하거나 바로 Design 단계로 진행 가능'
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
            '**팀 구성** (Enterprise 기준):',
            '- cto-lead (opus): 전체 검토 및 방향 결정',
            '- architect: 기술 아키텍처 검토',
            '- reviewer: 코드 품질 및 일관성',
            '',
            '**전제 조건**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 환경 변수 필요',
            '',
            '**예상 소요**: 10~20분 (팀 규모에 따라 다름)'
          ].join('\n')
        }
      ]
    },

    'plan': {
      question: `Plan 완료. 다음 단계를 선택해주세요.`,
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
            '**결과물**:',
            '- `docs/02-design/features/' + feature + '.design.md`',
            '- API 스펙, DB 스키마, UI 구조',
            '',
            '**현재 PDCA 상태**:',
            '[Plan] ✅ → **[Design]** ⏳ → [Do] ⏳ → [Check] ⏳'
          ].join('\n')
        },
        {
          label: 'Plan 수정',
          description: 'Plan 문서에 추가 변경 사항 반영',
          preview: [
            '## Plan 수정',
            '',
            `**대상 문서**: \`docs/01-plan/features/${feature}.plan.md\``,
            '',
            '**주요 수정 포인트**:',
            '- 기능 요구사항 (FR) 추가/수정',
            '- 범위 재정의',
            '- 리스크 및 완화 전략 보완',
            '',
            '**팁**: 큰 범위 변경이라면 `/plan-plus`로 재시작을 검토하세요'
          ].join('\n')
        },
        {
          label: '다른 기능 먼저',
          description: '이 feature를 보류하고 다른 작업',
          preview: [
            '## Feature 보류',
            '',
            `**현재 상태**: ${feature} → "plan" 단계에 저장됨`,
            '',
            '**나중에 재개하는 방법**:',
            '```',
            `/pdca status          # 현재 상태 확인`,
            `/pdca design ${feature}   # 저장된 지점에서 재개`,
            '```',
            '',
            '**주의**: 다른 feature를 시작하면 .bkit-memory.json의 activeFeature가 변경됩니다'
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
            '**이동되는 문서** (4개):',
            `- docs/01-plan/features/${feature}.plan.md`,
            `- docs/02-design/features/${feature}.design.md`,
            `- docs/03-analysis/${feature}.analysis.md`,
            `- docs/04-report/features/${feature}.report.md`,
            '',
            `**아카이브 위치**: \`docs/archive/YYYY-MM/${feature}/\``,
            '',
            '**팁**: `--summary` 옵션으로 메트릭을 .pdca-status.json에 보존 가능',
            `\`/pdca archive ${feature} --summary\``
          ].join('\n')
        },
        {
          label: '추가 개선',
          description: '현재 feature에 추가 개선 반영',
          preview: [
            '## 추가 개선 (새 Iteration)',
            '',
            `**현재 Match Rate**: ${matchRate}%`,
            `**완료된 반복**: ${iterCount}회`,
            '',
            `**실행 명령**: \`/pdca iterate ${feature}\``,
            '',
            '**이 선택이 적합한 경우**:',
            '- Report 이후 새로운 요구사항이 생긴 경우',
            '- Match Rate를 더 높이고 싶은 경우',
            '- 코드 품질 개선이 필요한 경우',
            '',
            '**주의**: 새 Iteration은 별도 [Act-N] Task로 기록됩니다'
          ].join('\n')
        },
        {
          label: '다음 기능',
          description: '새로운 feature PDCA 시작',
          preview: [
            '## 새 Feature 시작',
            '',
            '**시작 명령** (선택):',
            '```',
            '/plan-plus {new-feature}    # 브레인스토밍 포함 (권장)',
            '/pdca plan {new-feature}    # 표준 Plan',
            '```',
            '',
            `**현재 feature 상태**: ${feature}는 "report" 단계로 유지됩니다`,
            '나중에 `/pdca archive ' + feature + '`로 정리 가능합니다',
            '',
            '**팁**: 새 feature 이름을 미리 생각해두세요'
          ].join('\n')
        }
      ]
    }
  };

  return questionSets[phase] || questionSets['plan'];
}
```

### 6.3 gap-detector-stop.js preview 추가

기존 AskUserQuestion 옵션에 preview 추가 (matchRate >= threshold 케이스 예시):

```javascript
// 변경 전
options: [
  { label: 'Generate report (Recommended)', description: `Run /pdca-report ${feature || ''}` },
  { label: '/simplify code cleanup', description: 'Improve code quality then generate report' },
  { label: 'Continue improving', description: `Run /pdca-iterate ${feature || ''}` },
  { label: 'Later', description: 'Keep current state' }
]

// 변경 후
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
      '**결과물**: `docs/04-report/features/' + feature + '.report.md`',
      '- Plan, Design, 구현, 분석 통합 보고서',
      '- PDCA 사이클 완료 기록'
    ].join('\n')
  },
  // ... 나머지 옵션도 동일하게 preview 추가
]
```

### 6.4 pdca-task-completed.js AskUserQuestion 추가

`shouldAutoAdvance()`가 false인 경우 (manual/semi-auto 모드에서 plan 완료 시) AskUserQuestion 발행:

```javascript
// plan 완료 시 수동 진행 모드에서 Next Action 제시
if (!shouldAutoAdvance(detectedPhase)) {
  const { buildNextActionQuestion, formatAskUserQuestion } = require('../lib/common.js');
  const questionPayload = buildNextActionQuestion(detectedPhase, featureName, { matchRate });
  const userPrompt = emitUserPrompt(formatAskUserQuestion(questionPayload));

  const response = {
    systemMessage: [
      `## PDCA ${detectedPhase.toUpperCase()} 완료`,
      '',
      `**Feature**: ${featureName}`,
      `**다음 단계를 선택해주세요.**`
    ].join('\n'),
    userPrompt
  };
  console.log(JSON.stringify(response));
  process.exit(0);
}
```

---

## 7. Success Criteria

### 7.1 Definition of Done

- [ ] FR-01~FR-10 모두 구현 완료
- [ ] gap-detector-stop.js: 4종 AskUserQuestion에 preview 필드 추가
- [ ] pdca-task-completed.js: plan/report 완료 시 AskUserQuestion 발행
- [ ] formatAskUserQuestion() preview graceful degradation 동작 확인
- [ ] buildNextActionQuestion() 3종 phase (plan-plus, plan, report) 반환 확인
- [ ] 단위 테스트: buildNextActionQuestion() 반환값 구조 검증

### 7.2 Quality Criteria

- [ ] CC v2.1.68 이하 환경에서 preview 없이 정상 동작 (regression 없음)
- [ ] CC v2.1.69 이상 환경에서 preview 사이드바 렌더링 확인
- [ ] 각 preview Markdown이 사이드바에서 올바르게 렌더링됨
- [ ] 기존 gap-detector-stop.js 동작 변경 없음 (label/description 동일 유지)

---

## 8. Risks and Mitigation

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| AskUserQuestion preview 규격이 v2.1.69 이후 변경 | Medium | Medium | preview 필드를 optional로 유지, 규격 변경 시 함수 1곳만 수정 |
| preview 내용이 너무 길어 사이드바 UX 저하 | Low | Low | preview 최대 15줄 제한, 핵심 정보만 포함 |
| pdca-task-completed.js AskUserQuestion 발행이 중복 호출 | High | Low | 상태 플래그 또는 phase별 1회만 발행 조건 추가 |
| CC v2.1.68 이하에서 preview 필드로 에러 발생 | High | Low | 사전 테스트 후 graceful degradation 확인, FR-10 필수 구현 |
| gap-detector-stop.js preview 조립 코드 복잡도 증가 | Low | Medium | buildNextActionQuestion() 헬퍼로 중앙화, 직접 조립 최소화 |

---

## 9. Architecture Considerations

### 9.1 Project Level Selection

bkit은 기존 아키텍처를 유지한다. 본 ENH는 lib/pdca/automation.js와 scripts/ 파일만 수정한다.

### 9.2 Key Architectural Decisions

| Decision | Options | Selected | Rationale |
|----------|---------|----------|-----------|
| Executive Summary 연결 | 순차(systemMessage→AskUserQuestion) / 통합(header에 포함) | **순차** | 가독성 및 CC 출력 구조에 최적 |
| 선택지 세트 위치 | SKILL.md 내 하드코딩 / lib/ 함수 중앙화 | **lib/ 중앙화** | 유지보수성, SKILL.md 변경 최소화 |
| preview 폴백 | 에러 throw / 필드 생략 | **필드 생략** | graceful degradation, FR-10 요건 충족 |
| 함수 exports | 기존 module.exports 확장 / 별도 파일 | **기존 확장** | 의존성 최소화, common.js 통해 재노출 |

### 9.3 변경 파일 목록

```
lib/pdca/automation.js          # formatAskUserQuestion() 확장 + buildNextActionQuestion() 추가
scripts/gap-detector-stop.js    # options에 preview 필드 추가 (4종 케이스)
scripts/pdca-task-completed.js  # plan/report 완료 시 AskUserQuestion 발행 추가
lib/common.js                   # buildNextActionQuestion export 추가
```

---

## 10. Next Steps

1. [ ] CTO 팀 리뷰 및 Plan 승인
2. [ ] Design 단계: lib/pdca/automation.js 함수 시그니처 및 preview 구조 확정
3. [ ] 구현: formatAskUserQuestion() 확장 (FR-01, FR-02, FR-10)
4. [ ] 구현: buildNextActionQuestion() 헬퍼 함수 (FR-09, FR-03~05)
5. [ ] 구현: gap-detector-stop.js preview 추가 (FR-06)
6. [ ] 구현: pdca-task-completed.js AskUserQuestion 발행 (FR-07)
7. [ ] 테스트: CC v2.1.68 / v2.1.69 양 환경에서 동작 확인

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 0.1 | 2026-03-05 | Initial draft - ENH-78 UX design, 3 scenarios, code design | Product Manager Agent |
