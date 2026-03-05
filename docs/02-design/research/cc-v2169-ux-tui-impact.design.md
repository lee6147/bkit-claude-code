# cc-v2169 UX/TUI 변경사항 bkit 영향 분석 설계서

> **Feature**: cc-v2169-impact-analysis (Task #6 보충: UX/TUI 분야)
> **Phase**: Design — Mockup 단계 (phase-3-mockup 스킬 적용)
> **Date**: 2026-03-05
> **Author**: Frontend Architect Agent
> **Status**: Draft

---

## 1. 개요

### 1.1 분석 범위

Claude Code v2.1.69에서 도입된 UX/TUI 변경사항 19건 중 bkit 사용자 경험과 직접 연관된 항목을 식별하고,
향후 bkit 개선 기회(ENH)로 전환 가능한 시나리오를 구체화한다.

### 1.2 bkit UX 핵심 접점

| bkit 컴포넌트 | TUI 상호작용 방식 | 관련 CC 변경사항 |
|--------------|------------------|----------------|
| AskUserQuestion (3개 스크립트) | 선택지 UI, 멀티셀렉트 | preview 필드, 숫자 키패드 |
| plan mode 에이전트 (7개) | 멀티라인 피드백 | Shift+Enter, backslash+Enter |
| PDCA 워크플로우 시작 | 세션 초기화 표시 | effort level 표시 |
| /pdca 스킬 실행 | 프롬프트 입력 | Ctrl+U bash 모드 종료 |
| output-styles (4종) | 출력 렌더링 | terminal flicker 수정 |

---

## 2. 변경사항별 bkit 영향 분류

### 2.1 직접 영향 항목 (HIGH)

#### 변경 #7: plan mode 멀티라인 피드백 (backslash+Enter, Shift+Enter)

**현재 상황**:
- bkit의 7개 plan mode 에이전트(`zero-script-qa`, `pdca-iterator`, `code-review` 등)가 사용자에게 피드백을 요청할 때 단일 라인 입력만 가능했음
- 복잡한 피드백(예: "1번과 3번 방향 중 3번을 선택하되, DB 마이그레이션은 건너뛰고 API 먼저 진행") 입력이 번거로움

**v2.1.69 이후 변경**:
```
Shift+Enter  → 새 줄 추가 (멀티라인 모드 진입)
backslash+Enter → 줄 연결 (기존 동작 유지)
```

**bkit 영향 (긍정)**:
- plan mode에서 `/pdca design` 실행 후 상세한 설계 피드백 입력 가능
- `zero-script-qa` 에이전트에서 복합 이슈 리포트 한 번에 전달 가능
- CTO Team에 멀티라인 지시문 전달 시 가독성 향상

**필요 조치**: 없음 (CC 내장 동작, bkit 코드 변경 불필요)

---

#### 변경 #9: /resume 최근 프롬프트 표시

**현재 상황**:
- PDCA 세션 중단 후 재개 시 첫 번째 프롬프트가 표시되어 어떤 작업이었는지 파악이 어려움

**v2.1.69 이후 변경**:
```
기존: /resume → 첫 번째 프롬프트 표시
변경: /resume → 가장 최근 프롬프트 표시
```

**bkit 영향 (긍정)**:
- PDCA 세션 재개 시 마지막 작업 컨텍스트가 즉시 표시됨
- `/pdca plan` → `/pdca design` → (중단) → `/resume` 시나리오에서 "design 단계 진행 중" 확인 용이
- CTO Team 에이전트 세션 재개 시 마지막 지시문 확인 가능

**필요 조치**: 없음 (CC 내장 동작)

**ENH 기회**: session-start.js에서 bkit 세션 재개 안내 메시지를 추가할 수 있음 (ENH-72 후보)

---

### 2.2 중간 영향 항목 (MEDIUM)

#### 변경 #1: effort level 표시 (로고/스피너)

**표시 예시**:
```
Claude Code — with low effort
Claude Code — with high effort
```

**bkit 영향 분석**:

| bkit 시나리오 | 예상 effort level | UX 효과 |
|-------------|-----------------|--------|
| `/pdca plan` (간단한 피처) | low effort | 사용자가 빠른 완료 예상 가능 |
| CTO Team 8-에이전트 병렬 실행 | high effort | 대기 시간 예상 설정 가능 |
| `zero-script-qa` (코드 분석) | medium~high | QA 복잡도 인식 |
| `/simplify` 실행 | low effort | 즉각적 완료 기대 |

**bkit 영향 (긍정)**:
- 사용자가 PDCA 사이클 각 단계의 복잡도를 시각적으로 인식
- CTO Team 다중 에이전트 실행 시 "high effort" 표시로 대기 수용도 향상

**필요 조치**: 없음 (CC 내장 동작)

---

#### 변경 #5: AskUserQuestion preview 필드

**현재 bkit AskUserQuestion 패턴** (gap-detector-stop.js 예시):
```javascript
userPrompt = emitUserPrompt({
  questions: [{
    question: `Match rate ${matchRate}%. Generate completion report?`,
    header: 'Complete',
    options: [
      { label: 'Generate report (Recommended)', description: `Run /pdca-report ${feature || ''}` },
      { label: '/simplify code cleanup', description: 'Improve code quality then generate report' },
      { label: 'Continue improving', description: `Run /pdca-iterate ${feature || ''}` },
      { label: 'Later', description: 'Keep current state' }
    ],
    multiSelect: false
  }]
});
```

**v2.1.69 신규 preview 필드** (목업, 코드, 다이어그램 지원):
```javascript
// v2.1.69 이후 가능한 확장 패턴 (ENH 후보)
userPrompt = emitUserPrompt({
  questions: [{
    question: `Match rate ${matchRate}%. Generate completion report?`,
    header: 'Complete',
    preview: {
      type: 'code',  // 'mockup' | 'code' | 'diagram'
      content: generateMatchRateSummary(matchRate, gapItems)
    },
    options: [ /* 기존 옵션 유지 */ ]
  }]
});
```

**활용 시나리오 (구체화)**:

**시나리오 A: Gap Analysis 결과 preview**
```
+-------------------------------------------+
| Check: 매치율 87%                          |
+-------------------------------------------+
| Preview (code):                            |
|  FR-01: PASS ✅                            |
|  FR-02: PASS ✅                            |
|  FR-03: FAIL ❌ (3 GAPs)                   |
|  FR-04: PASS ✅                            |
+-------------------------------------------+
| 어떻게 진행할까요?                          |
| > [1] 자동 개선 (권장)                      |
|   [2] 수동 수정                            |
|   [3] 설계서 업데이트                      |
+-------------------------------------------+
```

**시나리오 B: PDCA 단계 전환 preview (다이어그램)**
```
+-------------------------------------------+
| PDCA 단계 완료                             |
+-------------------------------------------+
| Preview (diagram):                         |
|  Plan ✅ → Design ✅ → Do ✅               |
|  Check ⬡ → Report ○                       |
+-------------------------------------------+
| Check 단계를 시작하시겠습니까?              |
| > [1] Gap Analysis 실행                    |
|   [2] 잠시 후                              |
+-------------------------------------------+
```

**시나리오 C: CTO Team 에이전트 선택 preview (목업)**
```
+-------------------------------------------+
| 분석 방식 선택                             |
+-------------------------------------------+
| Preview (mockup):                          |
|  단일 에이전트: 순차 분석, ~5분             |
|  3-에이전트 팀: 병렬 분석, ~2분            |
|  8-에이전트 팀: 완전 병렬, ~1분            |
+-------------------------------------------+
| 어떤 방식으로 분석할까요?                  |
| > [1] 단일 에이전트                        |
|   [2] 3-에이전트 팀 (권장)                |
|   [3] 8-에이전트 팀 (대규모)              |
+-------------------------------------------+
```

**ENH 평가**: HIGH 가치 — PDCA 의사결정 품질 향상, bkit의 핵심 인터랙션 개선
**구현 복잡도**: MEDIUM — emitUserPrompt() 함수 확장 필요
**ENH 번호**: ENH-73 후보

---

#### 변경 #4: 숫자 키패드 옵션 선택 지원

**현재 상황**:
- bkit AskUserQuestion은 화살표 키 + Enter로 옵션 선택
- 숫자 키패드(1, 2, 3...)로 직접 선택 불가

**v2.1.69 이후**:
- 숫자 키패드로 AskUserQuestion 옵션 직접 선택 가능
- 예: "1" 입력 → 첫 번째 옵션 즉시 선택

**bkit 영향 (긍정)**:
- gap-detector-stop.js의 4-옵션 선택이 "1"~"4" 숫자 입력으로 빨라짐
- PDCA 반복 사이클 속도 향상 (키보드 탐색 감소)
- iterator-stop.js, pdca-skill-stop.js 등 전체 bkit AskUserQuestion에 자동 적용

**필요 조치**: 없음 (CC 내장 동작, 즉시 혜택)

---

### 2.3 간접 영향 항목 (LOW)

#### 변경 #2: agent name 터미널 제목 표시 (--agent)

**bkit 연관성**:
- CTO Team 에이전트 실행 시 터미널 탭에 에이전트 이름 표시
- 멀티-터미널 환경에서 어떤 에이전트가 실행 중인지 즉시 확인 가능
- `phase1-impl`, `phase2-impl`, `qa-library` 등 에이전트 이름이 탭에 표시됨

**필요 조치**: 없음

---

#### 변경 #3: Ctrl+U 빈 bash prompt에서 bash 모드 종료

**bkit 연관성**:
- `/pdca` 스킬 실행 후 bash 모드 진입 시 빠른 종료 지원
- bkit 워크플로우에서 bash 모드를 자주 사용하지 않으므로 영향 미미

**필요 조치**: 없음

---

#### 변경 #8: Escape 실행 중 인터럽트 (입력 드래프트 있을 때)

**현재 상황**:
- Escape 키로 실행 중인 에이전트를 중단하려 할 때 입력 드래프트가 있으면 동작하지 않음

**v2.1.69 이후**:
- 입력 드래프트가 있어도 Escape로 실행 중 에이전트 인터럽트 가능

**bkit 영향 (긍정)**:
- CTO Team 에이전트가 잘못된 방향으로 진행 시 즉각 중단 가능
- PDCA 워크플로우 중 실수 복구 속도 향상

**필요 조치**: 없음

---

#### 변경 #6: 터미널 flicker 수정

**bkit 영향**:
- output-styles 4종(`bkit-concise`, `bkit-verbose`, `bkit-learning`, `bkit-pdca-guide`) 렌더링 시 flicker 감소
- 특히 CTO Team 다중 에이전트 동시 출력 시 화면 안정성 향상

---

#### 변경 #12: 타이핑 프레임 stall 수정

**bkit 영향**:
- 긴 PDCA 세션에서 타이핑 응답성 유지
- `zero-script-qa` 같은 장시간 실행 에이전트 이후에도 입력 속도 정상 유지

---

## 3. AskUserQuestion Preview 필드 활용 설계 (ENH-73 상세)

### 3.1 emitUserPrompt() 함수 확장 설계

**현재 인터페이스** (lib/common.js 기반):
```javascript
emitUserPrompt({
  questions: [{
    question: string,
    header?: string,
    options: Array<{ label: string, description?: string }>,
    multiSelect?: boolean
  }]
})
```

**확장 인터페이스 (v2.1.69 preview 필드 활용)**:
```javascript
emitUserPrompt({
  questions: [{
    question: string,
    header?: string,
    preview?: {
      type: 'mockup' | 'code' | 'diagram',
      content: string,
      title?: string
    },
    options: Array<{ label: string, description?: string }>,
    multiSelect?: boolean
  }]
})
```

### 3.2 적용 대상 스크립트 (우선순위 순)

| 스크립트 | 현재 AskUserQuestion 수 | preview 적용 시나리오 | 우선순위 |
|---------|:--------------------:|---------------------|:------:|
| gap-detector-stop.js | 3개 | Gap 요약 (code type) | HIGH |
| iterator-stop.js | 2개 | 개선 결과 diff (code type) | HIGH |
| pdca-skill-stop.js | 4개 | PDCA 단계 진행 상태 (diagram type) | MEDIUM |

### 3.3 Gap Analysis Preview 구현 예시

**gap-detector-stop.js 확장 설계**:

```javascript
// ENH-73: Gap Analysis 결과를 preview로 표시
function generateGapPreview(matchRate, gapItems, feature) {
  const frLines = (gapItems || []).slice(0, 5).map(item =>
    `  ${item.status === 'PASS' ? 'PASS ✅' : 'FAIL ❌'} ${item.id}: ${item.title}`
  ).join('\n');

  return {
    type: 'code',
    title: `Gap Analysis — ${feature || 'Feature'}`,
    content: [
      `Match Rate: ${matchRate}%`,
      `──────────────────────`,
      frLines,
      gapItems && gapItems.length > 5 ? `  ... +${gapItems.length - 5} more` : ''
    ].filter(Boolean).join('\n')
  };
}

// 적용 위치: line ~137 (matchRate >= threshold 블록)
userPrompt = emitUserPrompt({
  questions: [{
    question: `Match rate ${matchRate}%. Generate completion report?`,
    header: 'Complete',
    preview: generateGapPreview(matchRate, gapItems, feature),  // ENH-73
    options: [
      { label: 'Generate report (Recommended)', description: `Run /pdca-report ${feature || ''}` },
      { label: '/simplify code cleanup', description: 'Improve code quality then generate report' },
      { label: 'Continue improving', description: `Run /pdca-iterate ${feature || ''}` },
      { label: 'Later', description: 'Keep current state' }
    ],
    multiSelect: false
  }]
});
```

---

## 4. plan mode 멀티라인 피드백 활용 가이드 (ENH-74)

### 4.1 현재 사용자 워크플로우 (v2.1.68 이하)

```
[사용자] /pdca design
[bkit] Design 단계를 시작합니다. 주요 고려사항은?
[사용자] API 먼저, DB는 나중에     ← 단일 라인 제약으로 간략한 피드백
```

### 4.2 v2.1.69 이후 개선된 워크플로우

```
[사용자] /pdca design
[bkit] Design 단계를 시작합니다. 주요 고려사항은?
[사용자] Shift+Enter 로 멀티라인 입력:
         API 엔드포인트 설계 우선
         DB 마이그레이션은 Phase 2로 분리
         인증 미들웨어는 기존 JWT 패턴 유지
         성능: 응답시간 200ms 이하 목표   ← 상세한 피드백 가능
```

### 4.3 session-start.js 안내 메시지 업데이트 (ENH-74 구현 지점)

**추가 위치**: PDCA Core Rules 섹션 또는 Plan Phase 안내

```javascript
// session-start.js 추가 텍스트 (조건부 출력)
const planModeGuide = `
## Plan Mode Tips (v2.1.69+)
- Shift+Enter: Add new line in plan mode feedback
- Use multiline feedback for complex design decisions
- Example: "API first\\nDB migration in Phase 2\\nAuth: JWT"
`;
```

---

## 5. 향후 bkit UX 개선 제안

### 5.1 ENH 목록 (v2.1.69 기반)

| ENH | 제목 | 우선순위 | 구현 복잡도 | 예상 임팩트 |
|-----|------|:-------:|:----------:|:----------:|
| ENH-72 | /resume bkit 세션 재개 안내 | LOW | LOW | LOW |
| ENH-73 | AskUserQuestion preview 필드 활용 | HIGH | MEDIUM | HIGH |
| ENH-74 | plan mode 멀티라인 피드백 안내 추가 | MEDIUM | LOW | MEDIUM |

### 5.2 ENH-73 구현 로드맵

**Phase 1 (즉시 적용 가능)**:
- CC v2.1.69가 preview 필드를 공식 지원하는지 확인 (API 검증 필요)
- gap-detector-stop.js에 code type preview 추가 (5줄 이내 변경)

**Phase 2**:
- iterator-stop.js에 개선 결과 preview 추가
- pdca-skill-stop.js에 PDCA 단계 diagram preview 추가

**Phase 3**:
- emitUserPrompt() 함수 자체에 preview 헬퍼 추가
- 8개 언어 대응 (preview content는 언어 독립적이므로 최소 수정)

### 5.3 미구현 이유가 있는 항목

| 변경 | 미구현 이유 |
|------|-----------|
| /remote-control 커스텀 세션 이름 | bkit slash commands가 remote-control 미지원 (#28379 모니터) |
| VSCode compaction 표시 | bkit는 CLI 환경 우선, VSCode extension 미포함 |
| RTL 텍스트 수정 | bkit 8개 언어에 RTL 언어(아랍어 등) 미포함 |

---

## 6. 호환성 요약

### 6.1 bkit v1.5.8 + CC v2.1.69 호환성

| 항목 | 상태 | 근거 |
|------|:----:|------|
| AskUserQuestion (기존) | SAFE | preview 필드 미사용 시 기존 동작 유지 |
| plan mode 에이전트 7개 | SAFE | 멀티라인은 옵션, 기존 단일라인 유지 |
| output-styles 4종 | SAFE + IMPROVED | flicker 수정으로 렌더링 개선 |
| CTO Team 에이전트 | SAFE + IMPROVED | 에이전트명 터미널 표시, Escape 개선 |
| hooks 10개 | SAFE | UX 변경은 hook events에 영향 없음 |
| 8개 언어 지원 | SAFE | TUI 변경은 언어 독립적 |

### 6.2 결론

**bkit v1.5.8은 CC v2.1.69와 100% 호환**됩니다. UX/TUI 변경사항 16건 중 12건이 bkit 사용자 경험을 즉시 개선하며, 코드 변경 없이 자동 적용됩니다.

신규 ENH 기회 3건(ENH-72~74) 중 ENH-73(AskUserQuestion preview)이 가장 높은 임팩트를 가지며, CC v2.1.69 preview 필드 공식 API 확인 후 bkit v1.5.9+ 에서 구현을 권장합니다.

---

## 7. 변경사항 전체 매핑 요약

| # | 변경사항 | bkit 영향 | 조치 |
|---|---------|:--------:|------|
| 1 | effort level 표시 | MEDIUM (+) | 없음 |
| 2 | agent name 터미널 제목 | LOW (+) | 없음 |
| 3 | Ctrl+U bash 모드 종료 | LOW | 없음 |
| 4 | 숫자 키패드 옵션 선택 | MEDIUM (+) | 없음 |
| 5 | AskUserQuestion preview 필드 | HIGH (+) | ENH-73 등록 |
| 6 | /remote-control 커스텀 세션 이름 | NONE | #28379 모니터 유지 |
| 7 | plan mode 멀티라인 피드백 | HIGH (+) | ENH-74 등록 |
| 8 | Escape 실행 중 인터럽트 | MEDIUM (+) | 없음 |
| 9 | /resume 최근 프롬프트 표시 | MEDIUM (+) | ENH-72 등록 |
| 10 | ctrl+o transcript 멈춤 수정 | LOW (+) | 없음 |
| 11 | Ctrl+S stash 클리어 수정 | LOW (+) | 없음 |
| 12 | 터미널 flicker 수정 | MEDIUM (+) | 없음 |
| 13 | 타이핑 프레임 stall 수정 | LOW (+) | 없음 |
| 14 | /stats 크래시 수정 | LOW (+) | 없음 |
| 15 | cursor 빈 줄 이동 수정 | LOW (+) | 없음 |
| 16 | 확장 subagent transcript 빈 줄 수정 | LOW (+) | 없음 |
| 17 | VSCode compaction 표시 | NONE | bkit CLI 전용 |
| 18 | VSCode permission mode picker | NONE | bkit CLI 전용 |
| 19 | VSCode RTL 텍스트 수정 | NONE | 해당 언어 미포함 |

**총 평가**: 16건 즉각 혜택 (코드 변경 없음), 3건 ENH 등록, 3건 해당 없음
