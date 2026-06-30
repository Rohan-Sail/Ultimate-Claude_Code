 Plan: Build a Security Testing Workflow Framework

 Context

 The working directory /home/rohan/newWorkflow_Claude is empty. The user provided a comprehensive security testing methodology spec (phased recon → auth testing → validation, with Triager and Learning agents) and wants it
 operationalized into a concrete, reusable framework they can actually run — not just a document of prose.

 Decisions confirmed:
 - Format: Both — Claude Code-native wiring (skills + subagent defs) plus a portable markdown copy of the framework.
 - Workbook depth: Auth-focused, extensible — fully flesh out Phase 2 (Authentication) per the spec; leave a structured skeleton for the other functionality classes to fill in as the engagement progresses.

 Outcome: a self-contained framework where a tester (or Claude) can start an engagement, drive it one attack-surface at a time through the loop, validate findings via a Triager, and feed lessons back into a Knowledge Base — usable
 both inside Claude Code and as plain markdown elsewhere.

 Directory structure

 /home/rohan/newWorkflow_Claude/
 ├── README.md                         # Overview + how to run (both modes)
 ├── CLAUDE.md                         # Project guidance so Claude Code loads the framework
 ├── docs/                             # Portable markdown framework
 │   ├── runbook.md                    # Master phased workflow + the loop + completion criteria
 │   ├── methodology.md                # Planning / Testing / Validation methodology per functionality
 │   ├── evidence.md                   # Evidence-collection standards (when to stop gathering)
 │   └── workbook/
 │       ├── README.md                 # Index + how to use + category status
 │       ├── 01-reconnaissance.md      # Recon activities + continuous-recon triggers
 │       ├── 02-authentication.md      # FULL test-case set (login/register/logout/pw reset/MFA/SSO/session/lockout/email verify)
 │       ├── 03-session-management.md  # Skeleton (extensible template)
 │       ├── 04-access-control.md      # Skeleton: IDOR, privilege escalation, forced browsing
 │       ├── 05-input-validation.md    # Skeleton: XSS, SQLi, injection, SSRF, XXE
 │       ├── 06-business-logic.md       # Skeleton: race conditions, workflow bypass
 │       └── _template.md              # Blank test-case template for new functionality classes
 ├── agents/                           # Portable agent specs (markdown)
 │   ├── triager.md                    # Triager: verdict outcomes, evaluation criteria
 │   └── learning.md                   # Learning agent: KB update rules
 ├── templates/                        # Reusable output templates
 │   ├── attack-surface-map.md
 │   ├── endpoint-inventory.md
 │   ├── finding-report.md             # Per-finding: repro, evidence, impact, severity, confidence
 │   ├── engagement-report.md          # Consolidated final report
 │   └── evidence-log.md
 ├── knowledge-base/                   # Grows over time
 │   ├── README.md                     # Organization + how to add entries
 │   ├── test-cases.md                 # Newly discovered test cases (Learning agent appends)
 │   ├── vulnerability-patterns.md     # Newly identified vuln patterns
 │   ├── false-positives.md            # Recorded FPs + indicators to avoid repeating
 │   └── lessons-learned.md
 ├── engagements/                      # Per-engagement working output dir
 │   └── .gitkeep
 └── .claude/                          # Claude Code-native wiring
     ├── engagements/                      # Per-engagement working output dir
     │   └── .gitkeep
     └── .claude/                          # Claude Code-native wiring
         ├── settings.json                 # Permissions for common testing tools (curl, ffuf, nmap, sqlmap, etc.)
         ├── skills/
         │   └── security-test/SKILL.md     # Invocable skill that loads runbook + drives the phased loop
         └── agents/
             ├── triager.md                # Claude Code subagent def
             └── learning.md               # Claude Code subagent def

     Key design decisions

     - Single-surface-at-a-time loop is encoded in docs/runbook.md and the security-test SKILL: recon → pick one functionality → plan → run workbook test cases → validate evidence → Triager review → update KB → expand recon if new
     surface → next functionality. Hard "do not advance until complete" gate per the spec.
     - Workbook format: each test case has ID | Title | Prerequisites | Steps | Expected (secure) behavior | Validation. Auth (02-authentication.md) gets ~40-60 concrete cases across all the auth components named in the spec; skeletons
     (03–06) use the same format with section headers and a few seed cases plus the _template.md.
     - Triager produces one of: Valid / Needs more evidence / Informational / Duplicate / Intended behavior / False positive — with explicit why and recommended next validation, matching the spec. Defined both as portable
     agents/triager.md and as .claude/agents/triager.md (Claude Code subagent with frontmatter).
     - Learning agent writes to knowledge-base/* (new test cases, vuln patterns, FPs, lessons) and is invoked at the end of each functionality per the completion criteria.
     - Evidence standards (docs/evidence.md) mirror the spec: stop gathering only when conclusively exploitable / conclusively not / more testing can't add confidence. Every finding template requires repro steps, request/response,
     impact (business + technical), severity, confidence.
     - .claude/settings.json adds an allowlist for read-only/common testing tools so the engagement doesn't stall on permission prompts; nothing destructive is pre-allowed.

     Files to create (representative, not exhaustive)

     - Top: README.md, CLAUDE.md
     - docs/runbook.md, docs/methodology.md, docs/evidence.md
     - docs/workbook/README.md, 01-reconnaissance.md, 02-authentication.md (the large one), 03–06 skeletons, _template.md
     - agents/triager.md, agents/learning.md
     - templates/* (5 files)
     - knowledge-base/* (README + 4 stub files)
     - .claude/settings.json
     - .claude/skills/security-test/SKILL.md
     - .claude/agents/triager.md, .claude/agents/learning.md
     - engagements/.gitkeep

     Patterns repeat across workbook skeletons and KB stubs — described once in _template.md / knowledge-base/README.md, referenced elsewhere.

     Verification

     After build:
     1. find /home/rohan/newWorkflow_Claude -type f | sort — confirm all intended files exist and tree matches the plan.
     2. Skim README.md → docs/runbook.md → docs/workbook/02-authentication.md → .claude/skills/security-test/SKILL.md — verify the loop is coherent and the skill actually references the runbook + workbook.
     3. Check .claude/agents/triager.md and .claude/agents/learning.md have valid subagent frontmatter (name, description, tools).
     4. Confirm engagements/ is empty-but-present (ready for an engagement's output).
     5. (Optional, in-session) invoke the security-test skill against a dummy local target to confirm it loads and begins the loop — only if the user authorizes a target.
