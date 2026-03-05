# bkit v1.5.9 문서 동기화 계획서

> **요약**: bkit v1.5.9 릴리스 후 코드베이스 전반에 걸쳐 잔존하는 v1.5.8 참조를 v1.5.9로 일괄 동기화. 버전 번호, JSDoc 어노테이션, Feature Guidance, 아키텍처 수치(199 exports), CHANGELOG, README 등 80+ 파일 대상.
>
> **프로젝트**: bkit-claude-code
> **기능명**: bkit-v1.5.9-doc-sync
> **버전**: v1.5.9
> **작성일**: 2026-03-05
> **작성자**: CTO 리드
> **이전 작업**: bkit-v1.5.8-doc-sync (완료, 100%)
> **브랜치**: feature/bkit-v1.5.9-executive-summary

---

## 요약 (Executive Summary)

| 관점 | 내용 |
|------|------|
| **문제** | v1.5.9 기능(Executive Summary 모듈, AskUserQuestion Preview UX, ENH-74~81)이 코드에 반영되었으나, 45개 JS 파일의 `@version 1.5.8`, 10개 에이전트 문서의 Feature Guidance, README/CHANGELOG/marketplace.json 등 공개 문서, 7개 bkit-system 문서, 3개 가이드의 버전 번호와 수치(186→199 exports)가 v1.5.8 상태로 잔존 |
| **해결** | 7개 카테고리, 80+ 파일을 대상으로 체계적 문서 동기화 수행. JSDoc 일괄 치환, Feature Guidance v1.5.9 섹션 추가, 아키텍처 수치 정합성 검증(199 exports, 27 skills, 16 agents, 10 hooks, 47 scripts) |
| **기능/UX 효과** | 모든 공개 문서(README, CHANGELOG, marketplace)와 내부 문서(agents, bkit-system, guides)가 v1.5.9를 정확히 반영. 신규 사용자와 기여자가 일관된 버전 정보를 확인 가능 |
| **핵심 가치** | 코드-문서 정합성 100% 달성. "문서 = 코드" 철학 실현. v1.5.8 doc-sync 패턴을 재활용하여 효율적 동기화 |

---

## 1. 배경

### 1.1 동기화 필요성

bkit v1.5.9가 릴리스되었으나(커밋 300a3fa, b1bd59a), 코드베이스 전반에 v1.5.8 참조가 잔존:
- 45개 JS 파일에 `@version 1.5.8` JSDoc
- 10개 에이전트 문서에 "v1.5.8 Feature Guidance" 섹션
- README.md 버전 배지, CHANGELOG.md 미갱신
- marketplace.json 버전 `1.5.8`
- hooks.json description `v1.5.8`
- bkit-system 7개 문서의 수치/버전 참조
- CUSTOMIZATION-GUIDE.md, commands/bkit.md 등 가이드 문서

### 1.2 v1.5.9 핵심 변경 사항

| # | 변경 | 영향 |
|:-:|------|------|
| 1 | **Executive Summary 모듈** (lib/pdca/executive-summary.js) | 3개 함수 신규 export (+3) |
| 2 | **AskUserQuestion Preview UX** (lib/pdca/automation.js) | buildNextActionQuestion, formatAskUserQuestion에 preview 필드 |
| 3 | **ENH-74: agent_id/agent_type** | 5개 스크립트에 1순위 추출 로직 |
| 4 | **ENH-75: continue:false** | 팀원 수명주기 제어 |
| 5 | **InstructionsLoaded 훅 제거** | hooks.json -6줄 |
| 6 | **Export 증가** | common.js 184 → 199 (+15) |
| 7 | **plan-plus-stop.js** | 신규 스크립트 |
| 8 | **템플릿 수정** | plan, plan-plus, report 템플릿에 Executive Summary 섹션 |

### 1.3 수치 변경 요약

| 지표 | v1.5.8 | v1.5.9 | 변경 |
|------|:------:|:------:|:----:|
| common.js exports | 184 | 199 | +15 |
| 라이브러리 모듈 | 38 | 39 | +1 |
| 스크립트 | 45 | 47 | +2 |
| 훅 항목 | 13 | 13 | 0 (InstructionsLoaded 제거, 기존 유지) |
| 스킬 | 27 | 27 | 0 |
| 에이전트 | 16 | 16 | 0 |
| 템플릿 | 16 | 16 | 0 |

---

## 2. 전수 조사 결과

### 2.1 v1.5.8 참조 분류 (동기화 대상)

| 카테고리 | 파일 수 | 참조 수 | 조치 |
|----------|:-------:|:-------:|------|
| JSDoc `@version 1.5.8` | 45 | 45 | **UPDATE** → `@version 1.5.9` |
| 에이전트 Feature Guidance | 10 | 20 | **UPDATE** → v1.5.9 섹션 추가/교체 |
| 설정/버전 파일 | 2 | 3 | **UPDATE** → 버전 숫자 변경 |
| README.md | 1 | 2 | **UPDATE** → 배지, Features 수치 |
| CHANGELOG.md | 1 | - | **ADD** → [1.5.9] 엔트리 추가 |
| hooks.json | 1 | 1 | **UPDATE** → description `v1.5.9` |
| bkit-system 문서 | 7 | 15 | **UPDATE** → 버전, 수치, 이력 |
| CUSTOMIZATION-GUIDE.md | 1 | 3 | **UPDATE** → 인벤토리, 버전 |
| commands/bkit.md | 1 | 1 | **UPDATE** → 버전 참조 |
| docs/guides/ | 2 | 2 | **UPDATE** → 버전 참조 |
| **총 동기화 대상** | **71** | **92+** | |

### 2.2 동기화 제외 (SKIP)

| 카테고리 | 파일 수 | 사유 |
|----------|:-------:|------|
| v1.5.8 PDCA 문서 (docs/01-plan, 02-design, 03-analysis, 04-report) | 12+ | 역사적 기록, 변경 불가 |
| v1.5.9 PDCA 문서 | 10+ | 이미 v1.5.9 참조 |
| 에이전트 메모리 (.claude/agent-memory/) | 3 | .gitignore 대상, 자동 갱신 |
| 리서치/조사 문서 | 5+ | 시점 기록, 역사적 |

---

## 3. 상세 작업 계획

### 3.1 카테고리 A: JSDoc `@version` 일괄 치환 (45 파일)

**대상**: `@version 1.5.8` → `@version 1.5.9`

#### lib/ (36 파일)

| # | 파일 | 현재 | 변경 |
|:-:|------|:----:|:----:|
| 1 | lib/common.js | 1.5.8 | 1.5.9 |
| 2 | lib/permission-manager.js | 1.5.8 | 1.5.9 |
| 3 | lib/memory-store.js | 1.5.8 | 1.5.9 |
| 4 | lib/context-hierarchy.js | 1.5.8 | 1.5.9 |
| 5 | lib/context-fork.js | 1.5.8 | 1.5.9 |
| 6 | lib/import-resolver.js | 1.5.8 | 1.5.9 |
| 7 | lib/skill-orchestrator.js | 1.5.8 | 1.5.9 |
| 8 | lib/core/cache.js | 1.5.8 | 1.5.9 |
| 9 | lib/core/config.js | 1.5.8 | 1.5.9 |
| 10 | lib/core/debug.js | 1.5.8 | 1.5.9 |
| 11 | lib/core/file.js | 1.5.8 | 1.5.9 |
| 12 | lib/core/index.js | 1.5.8 | 1.5.9 |
| 13 | lib/core/io.js | 1.5.8 | 1.5.9 |
| 14 | lib/core/paths.js | 1.5.8 | 1.5.9 |
| 15 | lib/core/platform.js | 1.5.8 | 1.5.9 |
| 16 | lib/intent/ambiguity.js | 1.5.8 | 1.5.9 |
| 17 | lib/intent/index.js | 1.5.8 | 1.5.9 |
| 18 | lib/intent/language.js | 1.5.8 | 1.5.9 |
| 19 | lib/intent/trigger.js | 1.5.8 | 1.5.9 |
| 20 | lib/pdca/level.js | 1.5.8 | 1.5.9 |
| 21 | lib/pdca/phase.js | 1.5.8 | 1.5.9 |
| 22 | lib/pdca/status.js | 1.5.8 | 1.5.9 |
| 23 | lib/pdca/tier.js | 1.5.8 | 1.5.9 |
| 24 | lib/task/classification.js | 1.5.8 | 1.5.9 |
| 25 | lib/task/context.js | 1.5.8 | 1.5.9 |
| 26 | lib/task/creator.js | 1.5.8 | 1.5.9 |
| 27 | lib/task/index.js | 1.5.8 | 1.5.9 |
| 28 | lib/task/tracker.js | 1.5.8 | 1.5.9 |
| 29 | lib/team/cto-logic.js | 1.5.8 | 1.5.9 |
| 30 | lib/team/communication.js | 1.5.8 | 1.5.9 |
| 31 | lib/team/coordinator.js | 1.5.8 | 1.5.9 |
| 32 | lib/team/hooks.js | 1.5.8 | 1.5.9 |
| 33 | lib/team/index.js | 1.5.8 | 1.5.9 |
| 34 | lib/team/orchestrator.js | 1.5.8 | 1.5.9 |
| 35 | lib/team/state-writer.js | 1.5.8 | 1.5.9 |
| 36 | lib/team/strategy.js | 1.5.8 | 1.5.9 |
| 37 | lib/team/task-queue.js | 1.5.8 | 1.5.9 |

참고: lib/pdca/index.js, lib/pdca/automation.js, lib/pdca/executive-summary.js 는 이미 `@version 1.5.9`

#### scripts/ (8 파일)

| # | 파일 | 현재 | 변경 |
|:-:|------|:----:|:----:|
| 38 | scripts/phase9-deploy-stop.js | 1.5.8 | 1.5.9 |
| 39 | scripts/phase6-ui-stop.js | 1.5.8 | 1.5.9 |
| 40 | scripts/code-review-stop.js | 1.5.8 | 1.5.9 |
| 41 | scripts/phase5-design-stop.js | 1.5.8 | 1.5.9 |
| 42 | scripts/skill-post.js | 1.5.8 | 1.5.9 |
| 43 | scripts/context-compaction.js | 1.5.8 | 1.5.9 |
| 44 | scripts/learning-stop.js | 1.5.8 | 1.5.9 |
| 45 | scripts/user-prompt-handler.js | 1.5.8 | 1.5.9 |

참고: scripts/pdca-skill-stop.js, scripts/plan-plus-stop.js 는 이미 `@version 1.5.9`

**실행 방법**: `sed -i '' 's/@version 1.5.8/@version 1.5.9/g'` 일괄 치환

---

### 3.2 카테고리 B: 설정/버전 파일 (3 파일)

| # | 파일 | 위치 | 현재 | 변경 |
|:-:|------|------|:----:|:----:|
| 1 | .claude-plugin/marketplace.json | line 4, 37 | `"version": "1.5.8"` | `"version": "1.5.9"` |
| 2 | hooks/hooks.json | line 3 | `"bkit Vibecoding Kit v1.5.8"` | `"bkit Vibecoding Kit v1.5.9"` |

참고: bkit.config.json은 이미 `"version": "1.5.9"` (변경 불필요)

---

### 3.3 카테고리 C: 에이전트 Feature Guidance (10 파일)

**대상**: `agents/*.md`의 `## v1.5.8 Feature Guidance` 섹션

| # | 에이전트 | 조치 |
|:-:|----------|------|
| 1 | code-analyzer.md | v1.5.8 유지 + v1.5.9 섹션 추가 |
| 2 | design-validator.md | 동일 |
| 3 | enterprise-expert.md | 동일 |
| 4 | gap-detector.md | 동일 |
| 5 | infra-architect.md | 동일 |
| 6 | pdca-iterator.md | 동일 |
| 7 | pipeline-guide.md | 동일 |
| 8 | qa-monitor.md | 동일 |
| 9 | report-generator.md | 동일 |
| 10 | starter-guide.md | 동일 |

**추가할 v1.5.9 Feature Guidance 내용**:
```markdown
## v1.5.9 Feature Guidance

- **v1.5.9 Executive Summary**: New module (lib/pdca/executive-summary.js) with 3 exports (generateExecutiveSummary, formatExecutiveSummary, generateBatchSummary). Auto-generates 4-perspective summary (Problem/Solution/Function & UX Effect/Core Value) after PDCA document work.
- **v1.5.9 AskUserQuestion Preview UX**: Rich Markdown previews in PDCA phase transitions. buildNextActionQuestion() provides preview field with command examples, estimated time, and output paths.
- **v1.5.9 ENH-74~81**: agent_id/agent_type first-class extraction in hook scripts. continue:false support for teammate lifecycle control. 199 exports (was 184).
```

---

### 3.4 카테고리 D: 공개 문서 (3 파일)

#### README.md
| 항목 | 현재 | 변경 |
|------|------|------|
| 버전 배지 (line 5) | `Version-1.5.8-green` | `Version-1.5.9-green` |
| Features 목록 (line 63) | `190 exports` | 제거 또는 `199 exports`로 수정 |
| v1.5.9 Features | 없음 | Executive Summary, Preview UX, ENH-74~81 항목 추가 |

#### CHANGELOG.md
| 항목 | 조치 |
|------|------|
| [1.5.9] 엔트리 | `## [1.5.9] - 2026-03-05` 섹션 신규 추가 (Added/Changed/Fixed 구분) |

#### CUSTOMIZATION-GUIDE.md
| 항목 | 현재 | 변경 |
|------|------|------|
| Component Inventory (line 131) | `v1.5.8` | `v1.5.9` |
| 버전 요약 (line 201) | v1.5.8 설명 | v1.5.9 내용 추가 |
| Plugin Structure (line 733) | `v1.5.8` | `v1.5.9` |

---

### 3.5 카테고리 E: bkit-system 문서 (7 파일)

| # | 파일 | 변경 항목 |
|:-:|------|-----------|
| 1 | bkit-system/README.md | 버전 참조 v1.5.8→v1.5.9, exports 186→199, Trigger System 버전 |
| 2 | bkit-system/_GRAPH-INDEX.md | v1.5.9 섹션 추가, exports 186→199, common.js 버전 |
| 3 | bkit-system/philosophy/core-mission.md | Current Implementation v1.5.8→v1.5.9, exports 수치 |
| 4 | bkit-system/philosophy/context-engineering.md | 버전 참조, exports 수치 |
| 5 | bkit-system/components/agents/_agents-overview.md | 버전 v1.5.8→v1.5.9, exports 수치 |
| 6 | bkit-system/components/scripts/_scripts-overview.md | 버전, exports, 스크립트 수 45→47 |
| 7 | bkit-system/components/skills/_skills-overview.md | 버전 v1.5.8→v1.5.9 |

---

### 3.6 카테고리 F: 가이드/명령어 문서 (3 파일)

| # | 파일 | 변경 항목 |
|:-:|------|-----------|
| 1 | commands/bkit.md | Code Quality 버전 v1.5.8→v1.5.9 |
| 2 | docs/guides/remote-control-compatibility.md | bkit 버전 참조 |
| 3 | docs/guides/cto-team-memory-guide.md | 버전 참조 |

---

### 3.7 카테고리 G: 검증 (동기화 후)

| # | 검증 항목 | 방법 |
|:-:|-----------|------|
| 1 | `@version 1.5.8` 잔존 0건 확인 (JS) | `grep -r "@version 1.5.8" --include="*.js"` |
| 2 | marketplace.json 버전 일치 | `grep '"version"' .claude-plugin/marketplace.json` |
| 3 | hooks.json description 일치 | `grep 'description' hooks/hooks.json` |
| 4 | README.md 배지 = 1.5.9 | `grep 'Version-' README.md` |
| 5 | CHANGELOG.md에 [1.5.9] 존재 | `grep '\[1.5.9\]' CHANGELOG.md` |
| 6 | common.js exports = 199 | `node -e "console.log(Object.keys(require('./lib/common')).length)"` |
| 7 | 에이전트 10개 모두 v1.5.9 Guidance | `grep -l "v1.5.9 Feature Guidance" agents/*.md \| wc -l` |
| 8 | bkit-system 수치 일관성 (199) | `grep "199 exports" bkit-system/*.md bkit-system/**/*.md` |
| 9 | 이전 PDCA 문서 무변경 확인 | `git diff docs/01-plan/features/bkit-v1.5.8*` (변경 없어야 함) |

---

## 4. 실행 전략

### 4.1 CTO 팀 구성

| 역할 | 에이전트 | 담당 | 예상 파일 수 |
|------|----------|------|:-----------:|
| CTO 리드 | cto-lead | 전체 조율, 검증 | - |
| 코드 동기화 | code-analyzer | 카테고리 A (JSDoc 45파일) + B (설정 3파일) | 48 |
| 문서 동기화 | gap-detector | 카테고리 C (에이전트 10파일) + E (bkit-system 7파일) | 17 |
| 공개 문서 | report-generator | 카테고리 D (README, CHANGELOG, CUSTOMIZATION) + F (가이드 3파일) | 6 |

**총 3명 에이전트** (단순 치환 작업이므로 소규모 팀)

### 4.2 실행 순서

```
1단계: 카테고리 A (JSDoc 일괄 치환, 45 파일) — sed 일괄 실행
2단계: [병렬] 카테고리 B + C + D + E + F — 에이전트 분배
3단계: 카테고리 G (검증 9항목) — CTO 리드
4단계: 커밋 & 푸시
```

### 4.3 예상 변경량

| 지표 | 값 |
|------|---:|
| 변경 파일 수 | ~71 |
| 추가 줄 수 | ~150 (CHANGELOG + Feature Guidance) |
| 수정 줄 수 | ~100 (버전 치환) |
| 삭제 줄 수 | 0 |

---

## 5. 성공 기준

| # | 기준 | 측정 방법 |
|:-:|------|-----------|
| 1 | JS 파일에 `@version 1.5.8` 잔존 0건 | grep count = 0 |
| 2 | 모든 설정 파일 버전 = 1.5.9 | marketplace.json, hooks.json, bkit.config.json |
| 3 | README 배지 = 1.5.9 | 시각적 확인 |
| 4 | CHANGELOG에 [1.5.9] 섹션 존재 | grep 확인 |
| 5 | 에이전트 10개 v1.5.9 Feature Guidance | grep count = 10 |
| 6 | bkit-system exports 수치 = 199 | grep 확인 |
| 7 | 이전 PDCA 문서 무변경 | git diff 0줄 |
| 8 | `node -e "require('./lib/common')"` 정상 | 에러 없음 |

---

## 6. 위험 요소

| 위험 | 영향 | 완화 |
|------|------|------|
| v1.5.8 역사적 참조 오변경 | PDCA 기록 훼손 | SKIP 목록 엄격 적용, docs/01-plan~04-report 제외 |
| sed 치환 시 코드 로직 변경 | 런타임 오류 | `@version` 패턴만 치환, 코드 내 문자열 불변 |
| marketplace.json 포맷 깨짐 | 플러그인 설치 오류 | JSON 유효성 검증 (node -e "JSON.parse(...)") |

---

## 버전 이력

| 버전 | 일자 | 작성자 | 변경 내용 |
|------|------|--------|-----------|
| 1.0 | 2026-03-05 | CTO 리드 | 최초 작성 — 전수 조사 기반 71파일 동기화 계획 |
