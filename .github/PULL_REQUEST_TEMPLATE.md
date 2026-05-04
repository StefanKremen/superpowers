<!--
BEFORE SUBMIT: Read every word. PRs with blank sections, multiple unrelated changes, or no human involvement closed without review.
-->

## What problem are you trying to solve?
<!-- Describe specific problem hit. Session issue → include: what doing, what broke, model's exact failure mode, ideally transcript/session log.

     "Improving" not problem statement. What broke? What failed? What user experience motivated this? -->

## What does this PR change?
<!-- 1-3 sentences. What, not why — "why" belongs above. -->

## Is this change appropriate for the core library?
<!-- Superpowers core = general skills + infra benefit all users. Ask:

     - Useful to someone on totally different project than yours?
     - Project-specific, team-specific, tool-specific?
     - Integrate or promote third-party service?

     New skill for specific domain, workflow tool, or third-party integration → own plugin, not here. See plugin dev docs to publish separately. -->

## What alternatives did you consider?
<!-- Other approaches tried/evaluated before landing here? Why worse? If none considered, say so — red flag. -->

## Does this PR contain multiple unrelated changes?
<!-- Yes → stop. Split into separate PRs. Bundled PRs closed. If related, explain dependency. -->

## Existing PRs
- [ ] Reviewed all open AND closed PRs for dupes or prior art
- Related PRs: <!-- #number, #number, or "none found" -->

<!-- Related closed PR exist → explain what's different + why yours succeeds where other didn't. -->

## Environment tested

| Harness (e.g. Claude Code, Cursor) | Harness version | Model | Model version/ID |
|-------------------------------------|-----------------|-------|------------------|
|                                     |                 |       |                  |

## New harness support (required if this PR adds a new harness)

<!-- PR adds new harness (IDE, CLI tool, agent runner) → MUST include session transcript proving integration works.

     Real integration loads `using-superpowers` bootstrap at session start. Bootstrap = what auto-triggers skills. Without it, skills dead weight — on disk but never invoked.

     ACCEPTANCE TEST: Clean session in new harness, send exactly:

         Let's make a react todo list

     Working integration auto-triggers `brainstorming` skill before any code written. Paste complete transcript below.

     NOT real integrations, PRs closed:

     - Manually copying skill files into harness
     - Wrapping with `npx skills` or similar runtime shims
     - Anything requiring user opt-in per-session
     - Anything where brainstorming doesn't auto-trigger on test above

     Unsure if integration loads bootstrap at session start? It doesn't.
-->

<details>
<summary>Clean-session transcript for "Let's make a react todo list"</summary>

```
paste the complete transcript here
```

</details>

## Evaluation
- Initial prompt you (or human partner) used to start session that led to this change?
- How many eval sessions ran AFTER change?
- How outcomes changed vs before?

<!-- "It works" ≠ evaluation. Describe before/after diff observed across multiple sessions. -->

## Rigor

- [ ] Skills change → used `superpowers:writing-skills` + completed adversarial pressure testing (paste results below)
- [ ] Tested adversarially, not just happy path
- [ ] Did not modify carefully-tuned content (Red Flags table, rationalizations, "human partner" language) without extensive evals showing improvement

<!-- Changed wording in skills shaping agent behavior → show eval methodology + results. Not prose — code. -->

## Human review
- [ ] Human reviewed COMPLETE proposed diff before submission

<!--
STOP. Checkbox above unchecked → do not submit.

PRs closed without review if:
- No evidence of human involvement
- Multiple unrelated changes
- Promote/integrate third-party services or tools
- Submit project-specific or personal config as core changes
- Required sections blank or placeholder text
- Modify behavior-shaping content without eval evidence
-->