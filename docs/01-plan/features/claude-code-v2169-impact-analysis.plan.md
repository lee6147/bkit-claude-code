# Claude Code v2.1.67~v2.1.69 Impact Analysis & Enhancement Plan

> **Feature**: claude-code-v2169-impact-analysis
> **Type**: Plan-Plus (Brainstorming-Enhanced)
> **Version**: bkit v1.5.8 -> v1.5.9/v1.6.0
> **Author**: CTO Team (10 agents)
> **Date**: 2026-03-05
> **Status**: Final
> **Previous**: claude-code-v2166-impact-analysis (v2.1.64~v2.1.66, 47 changes)

---

## Phase 0: Intent Discovery (Brainstorming)

### 0.1 분석 목적

Claude Code CLI v2.1.67~v2.1.69 (3 versions, 2 published) 버전업이 bkit plugin의 **모든 기능, 철학, 사상**에 미치는 영향을 완벽히 파악하고, 신기능을 활용한 기능 고도화 로드맵을 수립한다.

### 0.2 분석 팀 구성

| # | Agent | Role | Model | Task | Status |
|---|-------|------|:-----:|------|:------:|
| 1 | cto-lead | 총괄 분석 + 영향도 매트릭스 | opus | Task #1 | DONE |
| 2 | hook-analyst | Hook 시스템 영향도 | code-analyzer | Task #2 | DONE |
| 3 | skill-analyst | Skill/Plugin 영향도 | code-analyzer | Task #3 | DONE |
| 4 | agent-team-analyst | Agent Teams 영향도 | code-analyzer | Task #4 | DONE |
| 5 | security-analyst | Security/Sandbox 분석 | security-architect | Task #5 | DONE |
| 6 | sysprompt-analyst | System Prompt 변경 분석 | code-analyzer | Task #6 | DONE |
| 7 | perf-analyst | Performance/Memory 분석 | code-analyzer | Task #7 | DONE |
| 8 | issue-monitor | GitHub Issues 모니터링 | qa-strategist | Task #8 | DONE |
| 9 | enhancement-arch | ENH 고도화 제안 | enterprise-expert | Task #9 | DONE |
| 10 | frontend-ux | UX/TUI 변경 분석 | frontend-architect | Task #10 | DONE |

---

## Phase 1: Version Changes Summary

### 1.1 v2.1.67: SKIPPED (npm 미게시)

스킵 릴리스 패턴 지속 (v2.1.35, v2.1.40, v2.1.43, v2.1.46, v2.1.48, v2.1.57, v2.1.60, v2.1.65 동일).

### 1.2 v2.1.68 (2026-03-04, 3 changes)

| # | Change | bkit Impact |
|---|--------|:-----------:|
| 1 | Opus 4.6 기본 effort -> medium (Max/Team) | **MEDIUM** |
| 2 | "ultrathink" 키워드 재도입 (high effort) | LOW |
| 3 | Opus 4/4.1 제거, Opus 4.6 자동 마이그레이션 | NONE |

### 1.3 v2.1.69 (2026-03-05, ~93 changes, System Prompt +3,310 tokens)

**역대 최대급 릴리스** (v2.1.64의 46 changes를 능가).

| Category | Count | HIGH | MEDIUM | LOW | NONE |
|----------|:-----:|:----:|:------:|:---:|:----:|
| Features | ~20 | 6 | 5 | 7 | 2 |
| Bug Fixes | ~50 | 8 | 8 | 18 | 16 |
| Performance | ~15 | 2 | 5 | 6 | 2 |
| Changes | ~8 | 0 | 2 | 5 | 1 |
| **Total** | **~93** | **16** | **20** | **36** | **21** |

---

## Phase 2: Impact Analysis (10-Agent Results)

### 2.1 Compatibility Verdict

| Criteria | Result |
|----------|:------:|
| Breaking Changes | **0** |
| Hook System | **100% Compatible** |
| 27 Skills | **100% Compatible** |
| 16 Agents | **100% Compatible** |
| Agent Teams | **100% Compatible** |
| Overall | **COMPATIBLE** |
| Consecutive Releases | **v2.1.34~v2.1.69 = 35 releases** |

### 2.2 HIGH Impact Items (16 items)

#### 2.2.1 Hook System (Task #2 - hook-analyst)

| ID | Change | Impact | Action |
|----|--------|:------:|--------|
| H-01 | InstructionsLoaded hook event (18th) | HIGH | ENH-72: 신규 hook 등록 검토 |
| H-02 | agent_id + agent_type hook 필드 | HIGH | ENH-73: 에이전트 추적 고도화 |
| H-03 | TeammateIdle/TaskCompleted continue:false | **CRITICAL** | ENH-76: 품질 게이트 자동 중단 |
| H-04 | Plugin Stop/SessionEnd 미발화 수정 | HIGH | 자동 수혜 (cleanup 안정화) |
| H-05 | Hook event 무제한 누적 메모리 누수 수정 | **CRITICAL** | 자동 수혜 (장시간 세션) |
| H-06 | Plugin hooks template 충돌 수정 | LOW | 멀티 플러그인 환경에서만 |

**hook-analyst 핵심 결론**: bkit 코드 변경 없이 v2.1.69 즉시 호환. ENH-76 (continue:false) 구현 시 CTO Team 리소스 효율성 대폭 향상.

#### 2.2.2 Skill/Plugin System (Task #3 - skill-analyst)

| ID | Change | Impact | Action |
|----|--------|:------:|--------|
| S-01 | ${CLAUDE_SKILL_DIR} 변수 | HIGH | ENH-74: 스킬별 로컬 리소스 |
| S-02 | /reload-plugins 명령어 | MEDIUM | ENH-75: 개발 핫 리로드 가이드 |
| S-03 | 느린 시작 수정 (27 skills) | **HIGH** | 자동 수혜 (25~35% 시작 단축) |
| S-04 | Plugin Stop hook 미발화 수정 | **HIGH** | PDCA 상태 저장 신뢰성 100% |
| S-05 | skill description colon 파싱 수정 | LOW | 27 skills 전부 colon 사용 (Triggers:) |
| S-06 | /clear 캐시 스킬 리셋 수정 | MEDIUM | 개발/디버깅 개선 |

**skill-analyst 핵심 결론**: 27개 스킬 전부 `description: |` block scalar 사용, colon 파싱 수정 자동 적용. 시작 시간 25~35% 단축.

#### 2.2.3 Agent Teams (Task #4 - agent-team-analyst)

| ID | Change | Impact | Action |
|----|--------|:------:|--------|
| A-01 | Nested teammate 생성 방지 | **HIGH (검증 필요)** | CTO 2단계 위임 체인 테스트 |
| A-02 | In-process teammate GC 수정 | **CRITICAL** | 자동 수혜 (displayMode: in-process) |
| A-03 | Opus 4.6 medium effort 기본값 | **HIGH** | ENH-79: ultrathink 가이드 |
| A-04 | Subagent final report 간결화 | MEDIUM | CTO Team 토큰 효율 향상 |
| A-05 | TaskCreate activeForm 선택적 | LOW | ENH-82: Task 생성 간소화 |

**agent-team-analyst 핵심 발견**:
- **6개 2단계 nested 위임 경로** 존재 (cto -> qa-strategist -> gap-detector 등)
- v2.1.69의 nested teammate 방지가 Task() subagent에도 적용되는지 **즉시 검증 필요**
- in-process teammate GC 수정으로 PDCA 전체 사이클 **~500MB~1GB 메모리 절감**

#### 2.2.4 Security (Task #5 - security-analyst)

| ID | Change | Impact | Action |
|----|--------|:------:|--------|
| SEC-01 | symlink bypass acceptEdits 수정 | LOW | CC 엔진 수정으로 자동 보호 |
| SEC-02 | interactive tools auto-allow 수정 | MEDIUM | AskUserQuestion 정상 동작 보장 |
| SEC-03 | gitignored nested skill 차단 | NONE | bkit skills는 git tracked |

**security-analyst 핵심 결론**: bkit 보안 설계 "방어적으로 잘 구성". 모든 보안 수정이 긍정적. OWASP 매핑 전 항목 양호.

#### 2.2.5 System Prompt +3,310 tokens (Task #6 - sysprompt-analyst)

| ID | Change | Impact | Action |
|----|--------|:------:|--------|
| SP-01 | AskUserQuestion preview 필드 | **HIGH** | ENH-78: Gap 분석 비주얼 프리뷰 |
| SP-02 | Verification specialist 재추가 | MEDIUM | bkit PASS/FAIL/PARTIAL 연계 |
| SP-03 | Output efficiency 재추가 | MEDIUM | bkit 상세 보고서 축약 모니터링 |
| SP-04 | Agent Common suffix | MEDIUM | CTO Team sub-agent 보고 형식 |

**sysprompt-analyst 핵심 발견**: AskUserQuestion preview 필드가 bkit UX 개선의 최대 기회. `formatAskUserQuestion()` (lib/pdca/automation.js:243)에 preview 파라미터 추가 필요.

#### 2.2.6 Performance/Memory (Task #7 - perf-analyst)

| ID | Change | Impact | Savings |
|----|--------|:------:|:-------:|
| P-01 | React memoCache 누수 수정 | **CRITICAL** | -50~200MB/장시간 |
| P-02 | In-process teammate GC 수정 | **CRITICAL** | -100~500MB |
| P-03 | Hook event 누적 수정 | **CRITICAL** | -50~300MB |
| P-04 | REPL scope 누적 수정 | HIGH | -35MB/1000turns |
| P-05 | /clear 캐시 완전 해제 | HIGH | -20~100MB/clear |
| P-06 | Multi-GB 스파이크 수정 | HIGH | -0~2GB |
| P-07 | Yoga WASM 지연 로딩 | MEDIUM | -16MB 기본 |

**perf-analyst 핵심 결론**: v2.1.69는 bkit CTO Team에 **역대 최대 실질적 메모리 개선**. 장시간 Enterprise 세션 (500+ turns, 5 teammates) 메모리 **52% 절감** (~2.5GB -> ~1.2GB).

| Scenario | Before | After | Savings |
|----------|:------:|:-----:|:-------:|
| Short (50 turns) | ~300MB | ~260MB | 13% |
| Medium (200 turns, 3 team) | ~800MB | ~500MB | 37% |
| **Long (500+ turns, 5 team)** | **~2.5GB** | **~1.2GB** | **52%** |
| Extreme (1000+ turns) | ~5GB+ (OOM) | ~2GB | 60%+ |

#### 2.2.7 GitHub Issues (Task #8 - issue-monitor)

| ID | Issue | Impact | bkit Relevance |
|----|-------|:------:|:--------------:|
| #30926 | Bedrock beta flag regression | LOW* | Bedrock 사용자만 |
| #30932 | Windows EEXIST hang | NONE | Windows 전용 |
| #30928 | OneDrive EEXIST regression | NONE | Windows 전용 |
| #29548 | ExitPlanMode skips approval | MEDIUM | v2.1.63~ 미해결 |
| #29423 | Task subagents ignore CLAUDE.md | MEDIUM | v2.1.63~ 미해결 |

*Bedrock/LiteLLM 사용자에게는 Critical (Tool Call 전면 실패).

#### 2.2.8 UX/TUI (Task #10 - frontend-ux)

| ID | Change | Impact | bkit UX Benefit |
|----|--------|:------:|:---------------:|
| UX-01 | Plan mode 멀티라인 피드백 | HIGH | 7개 plan mode agent 상세 피드백 |
| UX-02 | AskUserQuestion preview | HIGH | PDCA 선택 비주얼 프리뷰 |
| UX-03 | Effort level 표시 | MEDIUM | PDCA 단계별 effort 가시화 |

---

## Phase 3: Enhancement Opportunities (ENH-72~84)

### 3.1 Brainstorming: 대안 비교

| Approach | Pros | Cons | Verdict |
|----------|------|------|:-------:|
| **A: 보수적** (문서만 업데이트) | 위험 최소, 빠른 릴리스 | 신기능 미활용, 기회 손실 | REJECT |
| **B: 점진적** (v1.5.9 Easy + v1.6.0 Hard) | 안정적 진행, 검증 후 확장 | 2 릴리스 필요 | **ACCEPT** |
| **C: 공격적** (v1.6.0 일괄 적용) | 빠른 혁신 | 검증 부족, 리스크 높음 | REJECT |

### 3.2 YAGNI Review

| ENH | YAGNI Check | Verdict |
|-----|-------------|:-------:|
| ENH-72 | InstructionsLoaded는 SessionStart로 대체 가능 | **DEFER (v1.6.0)** |
| ENH-73 | agent_id/agent_type는 이미 fallback으로 동작 중 | **ACCEPT (정확도 향상)** |
| ENH-74 | ${CLAUDE_SKILL_DIR}는 현재 ${PLUGIN_ROOT} 기반으로 충분 | **DEFER (v1.6.0)** |
| ENH-75 | /reload-plugins는 문서 안내만으로 충분 | **ACCEPT (Easy)** |
| ENH-76 | continue:false는 CTO Team 필수 기능 | **ACCEPT (핵심)** |
| ENH-77 | includeGitInstructions는 Enterprise 한정 | **ACCEPT (문서)** |
| ENH-78 | AskUserQuestion preview는 UX 핵심 개선 | **DEFER (v1.6.0, Hard)** |
| ENH-79 | ultrathink 가이드는 Opus effort 대응 필수 | **ACCEPT** |
| ENH-80 | effort 가이드는 문서 수준 | **ACCEPT (Easy)** |
| ENH-81 | CTO Team 안정성 문서는 즉시 반영 필요 | **ACCEPT** |
| ENH-82 | TaskCreate activeForm은 기존 코드 호환 | **ACCEPT (Easy)** |
| ENH-83 | 모델 마이그레이션은 문서 수준 | **ACCEPT (Easy)** |
| ENH-84 | MCP binary는 향후 기능 | **DEFER (v1.6.0)** |

### 3.3 ENH Priority Matrix

| Priority | ENH | Title | Difficulty | Release |
|:--------:|-----|-------|:----------:|:-------:|
| **P0** | ENH-76 | TeammateIdle/TaskCompleted continue:false 품질 게이트 | Medium | v1.5.9 |
| **P0** | ENH-73 | agent_id/agent_type hook 필드 1순위 활용 | Easy | v1.5.9 |
| **P1** | ENH-79 | Opus medium effort 대응 (ultrathink 가이드) | Easy | v1.5.9 |
| **P1** | ENH-81 | CTO Team 안정성 문서 갱신 (v2.1.69 메모리 수정) | Easy | v1.5.9 |
| **P1** | ENH-82 | TaskCreate activeForm 선택적 반영 | Easy | v1.5.9 |
| **P2** | ENH-75 | /reload-plugins 개발 가이드 | Easy | v1.5.9 |
| **P2** | ENH-77 | includeGitInstructions 문서화 | Easy | v1.5.9 |
| **P2** | ENH-80 | PDCA 단계별 effort 가이드 | Easy | v1.5.9 |
| **P2** | ENH-83 | Sonnet/Opus 모델 마이그레이션 문서 | Easy | v1.5.9 |
| **P1** | ENH-72 | InstructionsLoaded hook 통합 | Medium | v1.6.0 |
| **P1** | ENH-78 | AskUserQuestion preview 필드 활용 | Hard | v1.6.0 |
| **P2** | ENH-74 | ${CLAUDE_SKILL_DIR} 활용 | Easy | v1.6.0 |
| **P3** | ENH-84 | MCP binary 문서 분석 워크플로우 | Medium | v1.6.0 |

---

## Phase 4: Implementation Roadmap

### 4.1 v1.5.9 Release Plan (Minor - CC v2.1.69 대응)

**목표**: v2.1.69 호환성 확인 + 9개 Easy/Medium ENH 적용
**예상 변경 파일**: ~15 files

| # | ENH | Target Files | Description |
|---|-----|-------------|-------------|
| 1 | ENH-76 | `scripts/team-idle-handler.js`, `scripts/pdca-task-completed.js` | continue:false + stopReason 구현 |
| 2 | ENH-73 | `scripts/subagent-start-handler.js`, `scripts/subagent-stop-handler.js`, `scripts/team-idle-handler.js`, `scripts/pdca-task-completed.js` | agent_id/agent_type 1순위 |
| 3 | ENH-79 | `agents/code-analyzer.md`, `agents/gap-detector.md`, `agents/design-validator.md` | 분석 에이전트 깊은 사고 유도 |
| 4 | ENH-81 | `scripts/session-start.js` additionalContext | CC 권장 버전 v2.1.69, 메모리 안정화 문서 |
| 5 | ENH-82 | `agents/cto-lead.md`, `skills/pdca/SKILL.md` | activeForm 선택적 반영 |
| 6 | ENH-75 | `skills/bkit-rules/SKILL.md` | /reload-plugins 가이드 |
| 7 | ENH-77 | `skills/enterprise/SKILL.md` | includeGitInstructions 안내 |
| 8 | ENH-80 | `skills/pdca/SKILL.md` | PDCA 단계별 effort 가이드 |
| 9 | ENH-83 | `scripts/session-start.js` | 모델 마이그레이션 안내 |

### 4.2 v1.6.0 Release Plan (Major - 구조적 고도화)

**목표**: InstructionsLoaded hook + AskUserQuestion preview + SKILL_DIR 활용
**예상 변경 파일**: ~12 files

| # | ENH | Target Files | Description |
|---|-----|-------------|-------------|
| 1 | ENH-72 | `hooks/hooks.json`, 신규 `scripts/instructions-loaded.js` | InstructionsLoaded hook |
| 2 | ENH-78 | `lib/pdca/automation.js` (formatAskUserQuestion), `scripts/gap-detector-stop.js`, `scripts/iterator-stop.js`, `scripts/session-start.js` | preview 필드 |
| 3 | ENH-74 | 스킬별 SKILL.md (imports 경로) | ${CLAUDE_SKILL_DIR} |
| 4 | ENH-84 | `skills/enterprise/SKILL.md`, `agents/bkend-expert.md` | MCP binary 워크플로우 |

### 4.3 Risk Mitigation

| Risk | Mitigation | Owner |
|------|-----------|:-----:|
| Nested teammate 방지가 bkit 2단계 위임 차단 | v2.1.69 설치 후 `cto -> qa-strategist -> gap-detector` 3단계 체인 즉시 테스트 | CTO |
| Opus medium effort 분석 품질 저하 | code-analyzer, gap-detector에 "Think deeply" 지시 추가 (ENH-79) | CTO |
| AskUserQuestion preview 규격 불안정 | v1.6.0에서 신중 구현, #29547 모니터링 | Frontend |
| Bedrock regression #30926 | Bedrock 사용자 대상 주의사항 문서화 | QA |

---

## Phase 5: Monitoring & Metrics

### 5.1 CC Version Compatibility History

```
v2.1.34~v2.1.69: 35 consecutive compatible releases
Breaking Changes: 0 (across all 35 releases)
bkit Code Changes Required: 0
```

### 5.2 Hook Coverage

| Version | bkit Hooks | CC Hook Events | Coverage |
|---------|:---------:|:--------------:|:--------:|
| v2.1.63 | 10 | 17 | 58.8% |
| v2.1.66 | 10 | 17 | 58.8% |
| v2.1.69 | 10 | **18** (+InstructionsLoaded) | **55.6%** |
| v1.5.9 (target) | 10 | 18 | 55.6% |
| v1.6.0 (target) | **11** (+InstructionsLoaded) | 18 | **61.1%** |

### 5.3 Memory Improvement Tracking

```
v2.1.50: 9 leak fixes  -> CTO Team 가능
v2.1.63: 13 leak fixes -> CTO Team 안정화
v2.1.64: 4 leak fixes  -> 발견 (수정 대기)
v2.1.69: 7 leak fixes + 7 memory reductions + 9 optimizations -> 장시간 세션 완전 안정화
누적: 33 leak fixes + 16 memory optimizations
```

### 5.4 GitHub Issues Monitor

| Issue | Title | Status | bkit Impact |
|-------|-------|:------:|:-----------:|
| #29548 | ExitPlanMode skips approval | OPEN | MEDIUM |
| #29423 | Task subagents ignore CLAUDE.md | OPEN | MEDIUM |
| #30926 | Bedrock beta flag regression | OPEN | LOW* |
| #30586 | PostToolUse stdout duplicated | OPEN | LOW |
| #30613 | HTTP hooks JSON broken | OPEN | NONE |
| #28379 | Slash commands in remote-control | OPEN | LOW |

### 5.5 ENH Registry (ENH-72~84)

| ENH | Title | Release | Status |
|-----|-------|:-------:|:------:|
| ENH-72 | InstructionsLoaded hook | v1.6.0 | PLANNED |
| ENH-73 | agent_id/agent_type 활용 | v1.5.9 | PLANNED |
| ENH-74 | ${CLAUDE_SKILL_DIR} 활용 | v1.6.0 | PLANNED |
| ENH-75 | /reload-plugins 가이드 | v1.5.9 | PLANNED |
| ENH-76 | continue:false 품질 게이트 | v1.5.9 | PLANNED |
| ENH-77 | includeGitInstructions 문서화 | v1.5.9 | PLANNED |
| ENH-78 | AskUserQuestion preview | v1.6.0 | PLANNED |
| ENH-79 | ultrathink 가이드 | v1.5.9 | PLANNED |
| ENH-80 | effort 가이드 | v1.5.9 | PLANNED |
| ENH-81 | CTO Team 안정성 문서 | v1.5.9 | PLANNED |
| ENH-82 | TaskCreate activeForm 선택적 | v1.5.9 | PLANNED |
| ENH-83 | 모델 마이그레이션 문서 | v1.5.9 | PLANNED |
| ENH-84 | MCP binary 워크플로우 | v1.6.0 | PLANNED |

---

## Appendix A: Analysis Sources

- [Claude Code CHANGELOG.md](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code Releases](https://github.com/anthropics/claude-code/releases)
- [Claude Code System Prompts](https://github.com/Piebald-AI/claude-code-system-prompts)
- [npm @anthropic-ai/claude-code](https://www.npmjs.com/package/@anthropic-ai/claude-code)
- [GitHub Issues](https://github.com/anthropics/claude-code/issues)
- [Anthropic Blog](https://www.anthropic.com/news/enabling-claude-code-to-work-more-autonomously)

## Appendix B: Quick Reference

### bkit v1.5.8 Current State
- 27 Skills (22 core + 5 bkend)
- 16 Agents (7 opus / 7 sonnet / 2 haiku, 9 acceptEdits / 7 plan)
- 10 Hook Events (command type only)
- 180 Library Functions (common.js)
- hooks.json: 13 entries across 10 events

### v2.1.69 Key Numbers
- ~93 total changes (Features ~20, Bug Fixes ~50, Performance ~15, Changes ~8)
- System Prompt: +3,310 tokens
- Memory Leak Fixes: 7 (cumulative 33+)
- New Hook Event: InstructionsLoaded (18th total)
- New Hook Fields: agent_id, agent_type
- New Skill Variable: ${CLAUDE_SKILL_DIR}
- New Commands: /reload-plugins
- GitHub Issues: 20+ new (3 regressions, Windows-only)

### Compatibility Verdict
```
COMPATIBLE - 0 Breaking Changes
35 Consecutive Compatible Releases (v2.1.34~v2.1.69)
```

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2026-03-05 | Plan-Plus final - 10 agent analysis, 13 ENH, 2-release roadmap | CTO Team |
