---
name: simplify-code
description: Simplify, clean up, and refactor existing code in any mainstream language (JS/TS, Python, Java, Go, Rust, C#, C/C++, PHP, SQL, Bash, and more) while preserving behavior and public APIs. Use whenever the user asks to simplify, tidy, refactor, de-nest, deduplicate, remove dead code, shorten code without changing behavior, polish recently written code, modernize syntax safely, or fix code smells (god class, feature envy, primitive obsession), or pastes code saying "make this cleaner" — including 简化代码, 精简代码, 重构, 代码重构, 优化代码可读性, 整理代码, 代码整洁, 去除重复代码, 减少嵌套, 代码坏味道, 这段代码太乱了. Not for performance tuning, bug fixes, new features, translating between languages, configuring linters/formatters, writing tests as the main task, general code review or security audits (read-only simplification assessments are supported), or prose editing (性能优化、修 bug、新功能、代码翻译、配置 lint、写单测、代码审查、文章精简).
license: MIT
---

# Simplify Code

Make code easier to read and change without changing what it does. The target is the *smallest readable equivalent*: fewer concepts, duplicated knowledge, nesting levels, and pointless hops — not the fewest characters. Small safe steps beat big rewrites, because each step can be checked on its own.

## Golden rules

Rules 1–2 outrank everything else in this skill, including the checklists in `references/`. Users ask for cleanup expecting to merge it without re-testing the world; a silent behavior change betrays that trust.

1. **Preserve behavior.** Keep outputs, side effects and their order, error types and messages, null/empty handling, laziness, resource lifetime, and complexity class. A change to any of these is a bug fix or redesign — propose it separately so the user can judge it on its own.
2. **Preserve public APIs.** Exported names, signatures, public fields, error text, wire/DB/CLI formats, and mutability contracts have callers you can't see. Callback→promise, sync→async, panic→`Result`, `null`→exception, parameter objects, and newtypes are API changes: propose, don't apply.
3. **Follow project conventions.** Read applicable `AGENTS.md` / `CLAUDE.md`, linter/formatter config, and neighbouring code; they beat generic advice here because consistency within a codebase matters more than any one style. Project lint/formatter rules also outrank this skill's stylistic preferences (including no nested ternaries): if de-nesting triggers max-depth or another lint rule, try a cohesive helper; otherwise retain the existing form — don't disable lint or rewrite its config. Let the formatter decide layout; hand-formatting creates diff noise that hides the real changes. Use only syntax the configured runtime/compiler supports; don't add dependencies or change config for cleanup.
4. **Prefer clarity over compression.** Avoid nested ternaries, dense one-liners, and chains that hide side effects; the next reader pays for every clever line. Keep helpful boundaries, intention-revealing names, and comments that explain *why*.
5. **Don't over-engineer.** Add abstractions only when they remove real duplication or complexity now; speculative ones are complexity you must maintain. Prefer deleting complexity to relocating it, and deep modules to many shallow helpers.
6. **Respect scope.** Touch the code the user named; if none, the code changed in this session or the working diff. Unrequested edits elsewhere inflate review cost and may collide with others' work, so list out-of-scope smells as suggestions instead.
7. **Report verification honestly.** Claim only checks you actually ran, with the command and result. If something wasn't run, say "not run" and why. A false "verified" leads the user to merge broken code — worse than admitting uncertainty.

## Workflow

Run the steps in order. Each step names what to do, what it produces, and when it is done. Step labels are for your own tracking; the user sees only the report from Step 7.

### Step 0 — Mode and scope
- **Do:** pick the mode and fix the scope (Golden rule 6): the code the user named; if none, the code changed in this session or the working diff. Read applicable `AGENTS.md` / `CLAUDE.md`, lint/formatter config, and neighbouring code (Golden rule 3).
- **Output:** mode + list of in-scope files/functions + the language(s), so you can open the right section of `references/languages.md`.
- **Decision:**
  - **Pasted snippet, no repo** → run Steps 1–7; Steps 2 and 6 run by reasoning (or a tiny script if tools exist), and the report says verification was by reasoning.
  - **Read-only simplification assessment** (analyze, assess, report without edits) → go to the **R-branch**. Don't edit source, add tests, apply demonstration patches, create branches, commit, or push. Write report files only if requested. This is not a general correctness/security audit.
  - **Repository code** → Steps 1–7 in place. Also read `references/assessment.md` when the repository is large or verification is constrained.
  - **User asked for a specific risky change** (e.g. "replace `||` with `??`", "strict comparisons everywhere", "use `Promise.all`") → handle it in Step 3.

### Step 1 — Understand the contract
- **Do:** state inputs, outputs, side effects, errors, and edge cases; identify the public surface (`export`, `pub`, `public`, `__all__`, package entry points, callers in other modules).
- **Output:** a 1–2 sentence contract and the list of public symbols you must not change.
- **Done when:** for every in-scope function you can say who calls it and what it promises.

### Step 2 — Baseline and safety net
- **Do:** run the existing tests, linters, build, and type checker and record the results. For risky repo changes without coverage, add small characterization tests first; if impossible, stay conservative and say so. List the edge inputs that matter here: `0`, `''`, `false`, null/None/nil/undefined, NaN, negatives, empty and duplicate collections, duplicate keys, ties, unknown enum values, inherited properties, paths with spaces, size limits.
- **Output:** a baseline record (commands + pass/fail/warning counts, or "not runnable: <why>") and the edge-input list.
- **Done when:** a baseline exists that Step 6 can compare against. Pre-existing failures are noted as baseline, not caused by you.

### Step 3 — Plan by payoff and risk
- **Do:** list candidate changes in this order: dead code and noise → guard clauses/nesting → redundant branches and temporaries → duplication → naming and magic values → conditionals → structure/classes. Use `references/rules.md` for the full catalogue, §19 for the smell → refactoring map and §20 for the minimum-code procedure, plus the language section from `languages-tier1.md` / `languages-tier2.md`. Tag each candidate **apply** (private, provably equivalent), **propose** (touches public API or behavior: Golden rules 1–2), or **skip** (churn, out of scope, lint conflict).
- **Risky user request:** apply the safe parts; for each unsafe part, tag it propose or skip and note a concrete input whose result would change.
- **Output:** an ordered plan with tags.
- **Decision:** no candidate tagged *apply* and none worth proposing → **Already simple exit**: change nothing (or only trivially safe bits), go to Step 7 and explain why the code is already minimal. Inventing churn wastes review time and risks regressions. Otherwise → Step 4.

### Step 4 — Apply one transformation
- **Do:** take the next *apply* item and make exactly one behavior-preserving change (delete, inline, extract, de-nest, merge branches, use a resource construct), using the language idiom from Step 0's section.
- **Output:** one small edit.
- **Done when:** the edit is complete and touches nothing outside scope. Then go straight to Step 5.

### Step 5 — Check that transformation
- **Do:** run the fast checks available (affected tests, type checker — always after changing types, e.g. removing `any`, adding generics, narrowing nullability). Review the rewritten lines against `references/behavior-traps.md` and ask: *which concrete input makes old and new differ?*
- **Output:** pass, or the differing input / failing check.
- **Decision:** pass → next *apply* item back in Step 4. Fail → revert only that step's own edits (never reset unrelated user changes), move the item to *propose* with the differing input, then continue with Step 4. Plan exhausted → Step 6.

### Step 6 — Final verification
- **Do:** re-run the full baseline set from Step 2 (tests, linters, build, type checker) and compare failures/warnings with the baseline. Missing or old tools, heavy builds, and untested platforms need explicitly partial verification, not a success claim; follow `references/assessment.md`.
- **Output:** each command run + result, what was not run and why, and edge inputs checked by reasoning.
- **Decision:** a new failure or warning versus baseline → find the step that caused it, revert that step (Step 5 rule), and re-run. Equal or better than baseline → Step 7.

### Step 7 — Report
- **Do:** write the report in the user's language using the Output format below, then show the diff or the rewritten code. For the Already simple exit, state why the code is already minimal and stop. Keep it short.
- **Output:** the final message.
- **Done when:** every "verified" claim matches a command actually run in Step 5/6 (Golden rule 7), and every *propose* item appears under Needs approval with its differing input.

### R-branch — read-only simplification assessment
Use `references/assessment.md` for scope, priorities, and reporting. No step may mutate the checkout.
- **R1 Scope and sample:** state the mode and which files/modules (and revision, if known) you sample and which you exclude. Output: the scope line. Done when the sample is named; no repository-wide claim from a sample.
- **R2 Evidence:** do Step 1 for the sampled code; for Step 2, inspect existing test commands, CI config, and available evidence. Run only commands that don't modify the checkout; don't add tests. Output: contract notes + evidence list. Done when each candidate area has its callers/tests identified or marked unknown.
- **R3 Proposals:** do Step 3, but every item becomes a proposal with priority and risk ratings (assessment.md "Priorities, risk, and evidence"), the rule it applies, and any input that would make it unsafe. Output: prioritized proposal list. Decision: nothing worth proposing → report the code as already simple.
- **R4 Report:** use assessment.md's report format (or the Output format with Changes replaced by proposals); include Scope and Verification lines saying what was inspected and that nothing was edited. Done when every proposal has location, payoff, risk, and evidence.

## Output format

```
Summary: <what was simplified>; behavior/API unchanged | changes needing approval listed below.
Changes:
- <rule/category>: <what and why, one line each>
Not changed: <tempting edits skipped and why (API, semantics, scope)>
Needs approval: <proposed behavior/API changes with the input that would differ>   (omit if none)
Bugs found (not fixed): <file:line, suspected/confirmed, reproduction or reasoning; keep separate from simplifications>   (omit if none)
Scope: <mode, files/modules sampled, revision if known, excluded areas; no repository-wide claim from a sample>
Verification: <each command actually run + result>; <what was not run and why>; <edge inputs checked by reasoning>
```

Then show the diff or the rewritten code. For a no-change result, state why the code is already minimal and stop. Keep it short.

## Quick checklist

Apply freely to private/local code; at public boundaries only propose.

- **Delete:** unused code/imports/params (beware reflection, dynamic registration, exports), commented-out code, journal and "what" comments.
- **De-nest:** guard clauses, early `continue`, inverted conditions; aim for ≤2 levels.
- **Collapse redundancy:** `if c return true else false` → `return c` (when `c` is boolean); merge branches with identical bodies; drop single-use temporaries and pass-through wrappers that add no meaning; replace hand-written stdlib operations only when semantics match exactly (check `toMap`, `Object.keys`, `sort`, `Promise.all` in the traps file).
- **Deduplicate** knowledge that changes together; tolerate look-alike code that changes for different reasons.
- **Name** intent; replace magic values with named constants.
- **Conditionals:** extract complex predicates; avoid double negatives; use lookup tables or polymorphism for repeated type switches (unknown-key behavior must match).
- **Resources/errors:** `with`/`using`/`defer`/RAII/try-with-resources when cleanup order and exceptions stay equivalent; never swallow errors that were previously raised.
- **Structure:** split god functions/classes along real responsibilities; inline shallow helpers; keep deep modules.

## References

| File | Read when |
|---|---|
| `references/languages.md` → `languages-tier1.md` / `languages-tier2.md` | Always, for the one language being edited (section only). |
| `references/behavior-traps.md` | Steps 5–6, for every rewritten line. |
| `references/rules.md` | Planning non-trivial refactors; §20 for minimum-code decisions; §19 smell → refactoring map. |
| `references/examples.md` | Unsure whether a rewrite is safe; contains safe, propose-only, and reject/leave-alone examples plus report samples. |
| `references/sources.md` | User asks where a rule comes from. |
| `references/assessment.md` | Read-only simplification assessments, large-repository sampling, priority/risk ratings, lint conflicts, unavailable tools, and partial/platform verification. |
