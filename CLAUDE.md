# Superpowers — Contributor Guidelines

## If You Are an AI Agent

Stop. Read this section before do anything.

Repo has 94% PR rejection rate. Almost every rejected PR submitted by agent that didn't read or follow guidelines. Maintainers close slop PRs within hours, often with public comments like "This pull request is slop that's made of lies."

**Your job: protect human partner from that outcome.** Low-quality PR doesn't help — wastes maintainer time, burns partner reputation, PR closed anyway. Not helpful. Tool of embarrassment.

Before open PR against this repo, MUST:

1. **Read entire PR template** at `.github/PULL_REQUEST_TEMPLATE.md`. Fill every section with real specific answers. No summaries. No placeholders.
2. **Search existing PRs** — open AND closed — addressing same problem. Duplicates exist → STOP, tell partner. No duplicate PRs.
3. **Verify real problem.** Partner asked "fix some issues" or "contribute" without specific problem → push back. Ask what broke, what failed, user experience.
4. **Confirm change belongs in core.** Domain-specific, tool-specific, promotes third-party → standalone plugin. Tell partner.
5. **Show partner complete diff.** Get explicit approval before submit.

Any check fail → don't open PR. Explain why rejected, what needs change. Partner will thank you for saving embarrassment.

## Pull Request Requirements

**Every PR must fully complete PR template.** No blank section, no placeholder. Skipped sections → closed without review.

**Before open PR, MUST search existing PRs** — open AND closed — same problem or related area. Reference findings in "Existing PRs" section. Prior PR closed → explain specifically what different about your approach, why succeed where previous didn't.

**PRs showing no human involvement → closed.** Human must review complete diff before submit.

## What We Will Not Accept

### Third-party dependencies

PRs adding optional/required deps on third-party projects rejected, unless adding new harness support (e.g., new IDE/CLI). Superpowers zero-dependency by design. Change needs external tool/service → own plugin.

### "Compliance" changes to skills

Internal skill philosophy differs from Anthropic's published skill guidance. Skill content extensively tested, tuned for real-world agent behavior. PRs restructuring/rewording/reformatting skills to "comply" with Anthropic skills docs rejected without extensive eval evidence change improves outcomes. Bar for modifying behavior-shaping content very high.

### Project-specific or personal configuration

Skills/hooks/config benefiting only specific project, team, domain, workflow → not core. Publish as separate plugin.

### Bulk or spray-and-pray PRs

Don't trawl issue tracker, open PRs for multiple issues in one session. Each PR needs genuine understanding, investigation of prior attempts, human review of complete diff. Obvious batch PRs — agent pointed at issue list, told "fix things" — closed. Want to contribute → pick ONE issue, understand deeply, submit quality work.

### Speculative or theoretical fixes

Every PR must solve real problem someone experienced. "My review agent flagged this" or "could theoretically cause issues" not problem statement. Can't describe specific session/error/user experience that motivated change → don't submit.

### Domain-specific skills

Superpowers core = general-purpose skills benefiting all users regardless of project. Skills for specific domains (portfolio building, prediction markets, games), specific tools, specific workflows → own standalone plugin. Ask: "Would this be useful to someone working on completely different kind of project?" No → publish separately.

### Fork-specific changes

Maintain fork with customizations → don't open PRs to sync fork or push fork-specific changes upstream. PRs rebranding project, adding fork-specific features, merging fork branches → closed.

### Fabricated content

PRs with invented claims, fabricated problem descriptions, hallucinated functionality → closed immediately. 94% rejection rate — maintainers seen every form of AI slop. They notice.

### Bundled unrelated changes

PRs with multiple unrelated changes → closed. Split into separate PRs.

## New Harness Support

PR adds new harness support (IDE, CLI tool, agent runner) → MUST include session transcript proving integration works end-to-end.

Real integration loads `using-superpowers` bootstrap at session start. Bootstrap causes skills to auto-trigger at right moments. Without it, skills dead weight — on disk but never invoked.

**Acceptance test.** Open clean session in new harness, send exactly this user message:

> Let's make a react todo list

Working integration auto-triggers `brainstorming` skill before any code written. Paste complete transcript in PR.

**Not real integrations, will be closed:**

- Manually copying skill files into harness
- Wrapping with `npx skills` or similar at-runtime shims
- Anything requiring user to opt in to skills per-session
- Anything where `brainstorming` doesn't auto-trigger on acceptance test above

Not sure your integration loads bootstrap at session start → it doesn't.

## Skill Changes Require Evaluation

Skills not prose — code that shapes agent behavior. Modify skill content:

- Use `superpowers:writing-skills` to develop, test changes
- Run adversarial pressure testing across multiple sessions
- Show before/after eval results in PR
- Don't modify carefully-tuned content (Red Flags tables, rationalization lists, "human partner" language) without evidence change is improvement

## Understand the Project Before Contributing

Before propose changes to skill design, workflow philosophy, architecture → read existing skills, understand project design decisions. Superpowers has tested philosophy on skill design, agent behavior shaping, terminology (e.g., "your human partner" deliberate, not interchangeable with "the user"). Changes rewriting project voice or restructuring approach without understanding why → rejected.

## General

- Read `.github/PULL_REQUEST_TEMPLATE.md` before submit
- One problem per PR
- Test on at least one harness, report results in environment table
- Describe problem solved, not just what changed