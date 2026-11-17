# PR expansion: 2026-11-18

## What the repository actually merges

Reviewed recent merged and closed upstream PRs:

- **Product direction beats code quality.** #3204 (`connectors on every plan`, 60 files) was closed. #3208 (`on a free plan, say connectors need a paid plan`) was merged on 2026-11-16 with the explicit statement “Connectors stay paid-only.” This means the earlier #3258 would be rejected regardless of passing CI.
- **Small, focused fixes merge.** #3214 (mobile audit exception), #3169 (hide a button), #3178 (test fix), #3151 (pill polish) are all tiny merged PRs.
- **Docs and copy merge.** #2421, #2304, #2584, #2311, #3011 are merged docs-only PRs.
- **Snippet changes have a track record.** #1708, #1050, #1672, #2119, #2924 are merged snippet fixes/features.
- **Design decisions matter.** #3263 (give ownerless notes to the signing-in account) was closed while #3265 (show them under “On this device”) was merged. A technically clean PR can still be closed if the product approach differs.

## Decision

Closed **#3258** with a comment citing #3204/#3208. Kept **#3246** open but added a comment documenting that connectors staying paid is an explicit product decision, not an oversight.

Corrected **#3287**: the original claim that the pill has no role/tabindex/label was wrong. `src/App.jsx:539-545` passes those props; the issue was narrowed to missing state announcements (`aria-live`). No PR was opened for the false premise.

## New PRs

| PR    | Issue | Scope                                                               | Files                            | Local verification                                                                              |
| ----- | ----- | ------------------------------------------------------------------- | -------------------------------- | ----------------------------------------------------------------------------------------------- |
| #3294 | #3284 | YAML issue forms + PR template                                      | `.github/` only                  | PyYAML parses all three YAML files; no app code changed                                         |
| #3295 | #3249 | Pricing copy “Mobile app” -> “Unlimited mobile Cloud” in 11 locales | `src/locales/*/translation.json` | `npm run i18n:check` passes; Prettier passes                                                    |
| #3296 | #3275 | Date/time/datetime/locale snippet variables                         | `src/utils/snippets.ts` + tests  | 37 snippet tests pass; 3 other snippet suites pass; `npm run typecheck` passes; Prettier passes |

All seven active PRs (#3255, #3256, #3257, #3259, #3294, #3295, #3296) report `MERGEABLE` on GitHub — no conflicts with each other or with upstream.

## CI caveat

Current upstream `main` fails the `npm audit` gate with 26 findings (12 high) in dev/build dependencies. This is pre-existing: a clean checkout of `1611a2904` fails the same audit. Upstream PR **#3297** (`ci: audit only the dependencies the app ships`) is already open and explains that the flagged picomatch ReDoS has no patched release, so no dependency bump can clear it.

Each affected PR has a comment pointing to #3297 and listing the local checks that passed. The other quality steps (format, typecheck, tests) passed before the audit step.

## PR description template

The new PRs use the structure added in #3294: Summary, Linked issue (`Fixes #`/`Refs #`), What changed, Test plan/evidence, Screenshots, Checklist. This matches GitHub’s guidance and the repository’s CONTRIBUTING requirements.

## Suggested next steps

1. Wait for #3297 to land so the new PRs get a clean CI signal.
2. After that, prioritize #3295 (copy) and #3294 (templates) as low-risk merges.
3. #3296 implements a safe subset of #3275; clipboard/selection/cursor variables remain for a follow-up with async design.
