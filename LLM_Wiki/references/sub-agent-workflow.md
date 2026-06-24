# Sub-Agent Workflow — Spec Hunter Web

## Primary Pattern: Review → Fix → Commit

This is the **mandatory flow before every commit**. Do not skip.

```
┌─────────────────────────────────────────────────────────────────┐
│  1. git add <files>                                              │
│                                                                  │
│  2. LOAD: .claude/agents/spec-hunter-web.md                     │
│     RUN: Code Reviewer persona                                   │
│     PROMPT: 'Review staged changes for bugs, mobile issues...'   │
│          │                                                       │
│          ▼                                                       │
│     OUTPUT: Structured issue list (or "✅ No issues found")      │
│          │                                                       │
│          ▼                                                       │
│  3. IF ISSUES: RUN Fix Agent persona                             │
│     PROMPT: 'Fix the following issues: [paste list]'             │
│          │                                                       │
│          ▼                                                       │
│  4. git diff → verify fixes                                      │
│  5. git add <fixed files>                                        │
│  6. git commit                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Review Agent Prompt (Full Template)

```bash
claude -p 'Review staged changes for:
1. Mobile responsiveness — does it work at 375px width? Touch targets ≥44px?
2. Dark theme — bg #121317, surface #1a1b1f, accent #2e96ff?
3. Icons — are all UI icons from lucide-react? Any emoji accidentally used?
4. Edge cases — empty state when no data? Loading state during API call? Error state on failure?
5. Permission handling — camera denied? microphone denied? Graceful fallback?
6. Gamification balance — points correct? level trigger right? streak logic?
7. Firestore safety — no direct secret exposure?
8. Test compliance — does it match TEST.md expected behavior?

List every issue with file path and line reference. Priority: crash > data loss > visual > edge case.
If no issues: output "✅ No issues found"'
```

## Cost Strategy

| Layer | Model | Work | Cost |
|-------|-------|------|------|
| Review | gpt-4o-mini / haiku | 80%: boilerplate, CRUD, UI review | Cheap |
| Fix | sonnet / claude-sonnet-4 | 20%: tricky bugs, edge cases | More |

## When to Sub-agent vs Single Agent

| Situation | Approach |
|-----------|----------|
| Routine feature (table view, form) | Review Agent first, Fix if needed |
| Complex feature (keyboard test, LCD, gamification) | Review Agent + Fix Agent mandatory |
| Simple fix (typo, styling) | Single agent: `claude -p "Fix: ..."` |
| Blocked on a bug | Review Agent for fresh perspective, then fix |

## Pre-Commit Verification (GSD Core Phase 7.6)

### 5-Step Pipeline (Before Every Commit)

1. **Static Security Scan** — no hardcoded secrets, no dangerous functions, no .env files
2. **Baseline Tests + Linting** — TypeScript check, test pass
3. **Self-Review Checklist** — no console.log, no commented code, mobile check, dark theme, Lucide icons
4. **Independent Reviewer Subagent** — fresh context, no shared bias:
   ```
   Goal: Review this git diff. Return ONLY JSON verdict.
   Context: {diff output} + {static scan results}
   Expected JSON:
   {
     "passed": true/false,
     "security_concerns": [],
     "logic_errors": [],
     "suggestions": [],
     "summary": "one sentence verdict"
   }
   FAIL-CLOSED: security_concerns or logic_errors non-empty → passed=false
   ```
5. **Auto-Fix Loop** (if reviewer flagged issues):
   - Spawn Fix Agent (NEVER the implementer, NEVER the reviewer)
   - Fix ONLY reported issues — no refactoring
   - Max 2 fix-and-reverify cycles
