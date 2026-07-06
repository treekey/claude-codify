# claude-codify

**Spend one high-tier model session writing your project's constitution, so every cheaper session after it can govern itself.**

`codify` is a [Claude Code](https://claude.com/claude-code) skill that turns a strong model's judgment into durable, project-specific institution files — a lean `CLAUDE.md`, dispatch rules, decision rubrics, prompt templates, a maintenance protocol, and (optionally) hook-enforced gates. Weaker models then follow the written institution instead of improvising, and get measurably closer to high-tier behavior.

[繁體中文說明 → README.zh-TW.md](README.zh-TW.md)

## Why

One session with a top-tier model is worth more as *legislation* than as labor: externalize its judgment into files once, and every later session inherits it. But prose rules drift when cheaper models execute them, and mechanical enforcement everywhere is overkill — **the weight of the institution should match the risk of the project.** A personal script doesn't need a dispatch protocol and verification gates; a production service shouldn't rely on prose alone. So the skill is tiered.

## Tiers

| | Output | Token footprint | For |
|---|---|---|---|
| **lite** | One `CLAUDE.md` (≤80 lines), 6 embedded rubrics, no hooks | One small file per session | Personal scripts, prototypes |
| **standard** | Lean `CLAUDE.md` + `.claude/rules/` (dispatch, rubrics, templates, maintenance, letter) | Rules load per-stage, on demand | Team projects, CI, frequent delegation to cheaper models |
| **full** | standard + 1–3 hook-enforced gates (protected paths, verify gate) + adversarial review protocol | Same + per-turn hook latency | Production, payments, credentials — anywhere violations are expensive |

Tiers share the same core principles; they differ only in carrier and enforcement strength. Upgrading is additive (`lite → standard → full`) — no re-legislation.

## Usage

**First time on a project** — open Claude Code in the project directory and run:

```text
/codify
```

The skill then works on its own: it inspects the project (code layout, git history, existing `CLAUDE.md`, CI config), recommends a tier with a one-line reason, and asks you at most five questions. After you confirm, it writes the institution files listed in the next section, adversarially reviews its own output, and ends with a one-page summary of what was written and how to use it.

You don't need a top-tier model for this. The skill embeds a judgment snapshot and a fully worked exemplar distilled from one (`references/judgment.md`, `references/exemplar.md`), so `init` produces solid institutions on mid-tier models — legislating becomes imitation plus fill-in, not invention. A stronger model still helps when you have one.

Already know which tier you want? Skip the recommendation:

```text
/codify lite
/codify standard
/codify full
```

**Every few weeks, or after switching to a different model** — run:

```text
/codify audit
```

This checks the institution for drift — a bloated `CLAUDE.md`, rules pointing at files that no longer exist, unadjudicated pitfall logs — and applies incremental fixes only. Any model can run it.

**When the project outgrows its tier** (a solo script becomes a team project, a service goes to production):

```text
/codify upgrade standard
/codify upgrade full
```

This adds only the missing deliverables; everything already written stays untouched.

## What gets written into a project

Everything the skill produces is plain Markdown (plus, at full tier, two small Python hook scripts) owned by your project: commit it, edit it, or delete it like any other file. Nothing is tied to a specific model or to this skill — the institution keeps working even if you uninstall `codify` and only lose the ability to run `audit`/`upgrade` conveniently.

After `init`, your project looks like this:

```text
lite                      standard                        full = standard plus
─────────────             ─────────────────────────       ─────────────────────────
CLAUDE.md   (≤80 lines,   CLAUDE.md   (lean body +        .claude/hooks/
             everything                one-line index)      protect_paths.py
             embedded)    .claude/rules/                    verify_gate.py
                            diagnosis.md                  .claude/settings.json
                            dispatch.md                     (hook registration)
                            rubrics.md
                            prompt-templates.md
                            maintenance.md
                            letter.md
```

What each file is for:

| Deliverable | Contents |
|---|---|
| `CLAUDE.md` | The only always-loaded file: command cheatsheet (verified by actually running them), hard rules, coupled-change list ("touch A → must touch B"), a one-line index to the rest. Capped at 150 lines. |
| `diagnosis.md` | The top-3 token / focus / error hotspots found during init — each with evidence (file:line or git history) and the fix that was applied. |
| `dispatch.md` | Which model tier gets which work, the 3-element delegation contract (goal / acceptance / report format), escalation and de-escalation triggers, executor–verifier separation, and when a conclusion must survive adversarial review. |
| `rubrics.md` | Judgment turned into checklists: when to escalate, when work counts as done (incl. fail-then-pass evidence for bug fixes), when to stop digging, and the quality gate a verifier walks through. |
| `prompt-templates.md` | Fill-in-the-blank delegation prompts for the five common task shapes: search, implement, refactor, research, review. |
| `maintenance.md` | Who may edit which institution file, the pitfall log (sessions append proposals; `audit` adjudicates), and the degradation warning signs. |
| `letter.md` | The things nobody asked but every future session should know: real priorities, environment quirks, and how this institution is most likely to die. |
| `.claude/hooks/` (full only) | At most 3 fail-safe gate scripts for the hardest rules — e.g. block edits to generated code, block finishing with untested logic changes. Each is triggered once after install to prove it actually blocks. |

Two things worth knowing before you run it:

- **An existing `CLAUDE.md` is never silently discarded.** It is backed up first (or rewritten on a branch when the project is in git), and anything slated for deletion is shown to you as a list before it goes.
- **Output language follows the project**: an existing `CLAUDE.md` keeps its language; otherwise the institution is written in the language you converse in. Technical terms stay in English either way.

Every rule must pass one test: *could a Haiku-class model execute it unambiguously from the text alone?* If not, it becomes a checklist or an explicit escalation path.

## Design choices worth knowing

- **The strong model's judgment is preserved inside the skill.** Diagnosis heuristics, rule-writing transformations, edge-case playbooks, and known failure modes of weaker legislators are written down in `judgment.md`; a complete worked output lives in `exemplar.md`. When the top-tier model is gone, its judgment isn't.
- **Progressive disclosure** — the skill loads one reference per stage, never the whole rulebook. No per-turn protocol injection.
- **Sessions may not amend the rules that bind them.** Pitfalls are logged as proposals; `audit` adjudicates. This is the anti-corrosion mechanism.
- **Hooks are earned, not default.** A rule becomes a hook only when violation cost is high *and* the check is mechanizable. Hooks must be fail-safe (script errors never block) and must explain themselves when they block.
- **Verify gate checks that tests *passed*, not merely ran** — an improvement over regex-matching test commands.
- **Adversarial review** — three counter-lenses with pass conditions a weak model can execute (construct a concrete counterexample / weigh evidence against blast radius / beat a simpler alternative), majority survival. Reserved for major conclusions only; it is the most expensive mechanism in the institution.

## Install

Tell Claude Code:

> Install the codify skill from https://github.com/treekey/claude-codify following its INSTALL.md

or see [INSTALL.md](INSTALL.md) for the two-step manual copy. Installation adds no hooks and touches nothing outside `~/.claude/skills/codify/`.

## License

[MIT](LICENSE)
