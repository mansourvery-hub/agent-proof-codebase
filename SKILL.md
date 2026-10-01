---
name: agent-proof-codebase
version: 0.1.0
description: Build, audit, and work inside codebases whose deterministic gates (invariant specs, property/model/differential/crash-consistency tests, mutation testing, fail-to-pass bug-fix proof, ratchets, protected referee files, red-team corpus) make any accepted change high-signal even from weak or sloppy AI agents. Use whenever the user wants to harden a repo against low-quality or AI-generated code, design CI quality gates, make tests that can actually fail instead of checkbox tests, audit test-suite strength, set up mutation testing, write invariants or a SPEC, add performance budgets or ratchets, prepare a project for release with rigorous verification, stop agents from gaming tests, or asks for bulletproof, antifragile, or "premium quality" code. Also use when a repo already has SPEC.md, GATES.md, or .gates/ and you are asked to change it.
---

# Agent-proof codebase

Make acceptance mechanically expensive to fake, so whatever gets merged (from a human, a weak agent, or you)
is high-signal: it fixes a real defect, improves a measured property, or clears a proven edge case, with
machine-checked evidence attached.

This is not "write more tests". A million weak tests are worth less than a hundred that fail when the code is
wrong. For every gate ask: **what is the cheapest way to pass without being correct, and have I closed it?**

## Choose your mode

| If the user wants... | Mode | Start with |
|---|---|---|
| a gate system built or retrofitted for a repo, "make this bulletproof/release-ready" | **A. Build** | Workflow A below |
| to know how strong the existing tests/CI really are | **B. Audit** | Workflow B below |
| a feature, fix or refactor in a repo that already has SPEC.md / GATES.md / `.gates/` / quality-gate rules in AGENTS.md | **C. Contribute** | `references/12-contributor-protocol.md` |
| speed/memory/size improvements | **D. Optimize** | Workflow D below |
| to test whether the gates can be beaten | **E. Red-team** | `references/11-red-team-corpus.md` |

If a repo is gated and you were asked to change code, you are in mode C even if the request does not say so.
If unsure which mode applies, run `python3 scripts/assess_repo.py .` and decide from what exists.

## Conduct rules (always, all modes)

These exist because agents optimize the reward they can see. Keep them even when nobody is watching.

1. **Evidence over assertion.** Never write "fixed", "verified", "safe" or "all tests pass" without having observed it
   this session. Quote the command and the relevant output. Say "unverified" plainly when you could not check.
2. **Never edit the referee to get a pass.** Specs, gate configs, CI workflows, CODEOWNERS, ratchet baselines, lint/type/
   coverage/mutation config, red-team corpus, frozen fixtures, agent instruction files. If you think a gate is wrong,
   stop and propose a separate gate-change with evidence.
3. **Never weaken a check.** No deleted or loosened assertions, skips/xfail, new suppressions, widened tolerances,
   mocking the subject, hardcoded expected values, catch-all error handling, sleeps or retries to hide flakiness,
   fixed seeds to hide failures, deleted features to remove failing behavior.
4. **Fix the code, not the test,** unless the test is demonstrably wrong against the spec (then say so and get a human decision).
5. **Do not grade your own homework.** Use a separate context or subagent to review tests, and another to attack the
   result. Writing the gates and breaking the gates are different sessions.
6. **Ask only for decisions that are genuinely the user's:** product scope, spending money, removing user-facing
   features, changing persisted formats. Decide the rest and proceed.
7. **State what you chose not to do** and why. Silent omissions look like oversights.

## Workflow A: build or retrofit the gates

Read the referenced file when you reach each step; do not load them all up front.

1. **Orient and measure the true baseline.** Read docs, manifests, CI. Run `python3 scripts/assess_repo.py .` and the
   project's own quality commands. Record what passes, how long it takes, what flakes.
   See `references/02-assess-and-plan.md`.
2. **Subtract first.** Delete dead code, orphaned screens, unused dependencies, legacy features outside the product's
   scope. Fewer lines means fewer gates and fewer bugs. Record LOC and dependency count as the first ratchets.
3. **Tier the risk** (R0 loss/corruption/security, R1 core logic, R2 glue, R3 generated). Spend rigor where errors
   are catastrophic. Identify irreversible decisions (schemas, identifiers, formats, public APIs) and freeze them early.
4. **Write the plan** (`GATES_PLAN.md`): current maturity per tier, gap table with cost and leverage, phases, what you
   will not do. Get approval only for decisions that belong to the user.
5. **Protect the referee** before anything else depends on it: CODEOWNERS, `.gates/protected-paths.txt`, trusted-checkout
   CI, branch protection, PR template with a Class line, test-weakening tripwire.
   See `references/09-protect-the-referee.md`; templates in `assets/`.
6. **Write the spec.** 30 to 60 numbered, falsifiable invariants (INV-001...) with oracle, tier, and gates. Mine bug
   history and failure modes. Human-confirm T0/T1 invariants. See `references/03-spec-and-invariants.md`.
7. **Make the code testable.** Pure core, injected clock/random/storage/network, boundary rules enforced by tools,
   strict analysis. Retrofit behind characterization tests. See `references/04-testable-architecture.md`.
8. **Write tests that can fail.** Property, model-based, differential, round-trip, crash-consistency, fuzz for R0/R1.
   Tag each with its INV ID. Delete or fix tests that score 0 on the rubric. See `references/05-test-strategies.md`.
9. **Turn on strength gates.** Diff-scoped mutation testing; fail-to-pass for bug fixes; ratchets at today's values that
   only tighten. See `references/06-mutation-testing.md`, `07-evidence-contract.md`, `08-ratchets-and-budgets.md`.
10. **Tier the CI and shape the feedback** so a weak agent can converge: seeds, minimal counterexamples, exact
    reproduce commands, one local entry point (`assets/gates.sh.example`). See `references/10-ci-and-feedback.md`.
11. **Test the gates.** Build the red-team corpus, run a gate-breaker session in a fresh context, fix every escape,
    track recall. See `references/11-red-team-corpus.md`.
12. **Install the contributor rules** into the project's AGENTS.md/CLAUDE.md (`assets/AGENTS.snippet.md`) and write
    `GATES.md` (registry of gates, blind spots, owners, escapes log).
13. **Close the loop.** Every escape becomes a permanent gate. Review gate health quarterly; delete gates that stopped paying.

Finish with an honest summary: what was built, what each gate catches and cannot catch, what remains open, what the
user must do (store submission, real-device testing, credentials), and a clear readiness statement with reasoning.

## Workflow B: audit existing test and gate strength

1. Run `scripts/assess_repo.py` and read CI config; list every gate and what it can reject.
2. For each R0/R1 module, grade a sample of tests with the rubric in `references/05-test-strategies.md` (0 to 3).
3. Measure strength, not coverage: run mutation testing (or `scripts/manual_mutation_runner.py` with hand-picked
   mutants around invariants) on the risky modules. Report survivors by category.
4. Try the cheapest bypasses yourself in a scratch worktree (hardcode a visible test input, delete an assertion,
   add a suppression, swallow an error). Note which ones CI accepts.
5. Check the referee: can a normal PR edit gates, baselines, or CI? Is there a server-side authority?
6. Report: gates found, test-quality distribution, survivors, bypasses that worked, referee exposure, and a ranked
   fix list. Offer to proceed to Workflow A for the top items. Distinguish clearly what you measured from what you inferred.

## Workflow D: optimize without losing correctness

1. **Budgets first.** Write numbers for the metrics that matter (startup, operation latency at several data scales,
   peak memory, bytes per record, artifact size) in a `PERF.md` and add benchmarks with seeded synthetic datasets.
2. **Profile, do not guess.** Optimize the largest measured cost; leave cold code alone, because every clever line
   is a new bug candidate.
3. **Keep the simple implementation as an oracle.** Each optimization ships with a before/after benchmark beyond
   noise and a property/differential test proving old == new on random inputs.
4. **Look for algorithmic and structural wins first:** pushing filters and limits into the database instead of loading
   whole tables, compact integer keys instead of strings, integer timestamps, avoiding per-row parsing, lazy loading,
   removing unused dependencies and native code. Then micro-optimize hot loops.
5. **Ratchet the result** so it cannot regress. Details in `references/08-ratchets-and-budgets.md` and the perf class
   in `references/07-evidence-contract.md`.

## Quick reference

**Evidence by change class**

| Class | Proof CI must verify |
|---|---|
| bugfix | test fails on base code, passes on change (`scripts/fail_to_pass_check.sh`, use `EXPECT_PATTERN`) |
| perf | before/after beyond noise + differential test vs old implementation |
| refactor | no test files edited, mutation score not lower, equivalence evidence |
| feature | spec updated first, properties for new invariants |
| test-only | kills a named survivor or raises a ratchet |
| dependency | written reason, scan, size/startup delta |
| gate-change / spec-change | separate PR, human approval, red-team recall not reduced |

**Gate tiers:** T0 seconds (format, lint, types, banned APIs) | T1 minutes (tests, properties, traceability, referee checks)
| T2 tens of minutes (diff-scoped mutation, fail-to-pass, perf, migrations) | T3 nightly (full mutation, fuzz, simulation,
red-team recall, audits) | T4 release (real devices, upgrade-from-every-version).

**Tests that can fail:** property (laws, not examples), model-based (simple reference model in lockstep), differential
(independent oracle), round trip, crash-consistency (fail at every write boundary, reopen, check invariants), fuzz, simulation.

## Bundled scripts (Python 3 standard library or bash; each script documents its options in its header)

| Script | Purpose |
|---|---|
| `scripts/assess_repo.py` | Fast repo scan and suggested gate maturity (L0 to L5) |
| `scripts/protected_paths_check.py` | Fail when referee files change without human approval |
| `scripts/test_weakening_check.py` | Tripwire for removed assertions, new skips and suppressions |
| `scripts/fail_to_pass_check.sh` | Prove a bug-fix test fails on old code and passes on the fix |
| `scripts/ratchet_check.py` | Monotone metrics against a committed baseline (tighten only) |
| `scripts/spec_trace_check.py` | Every INV in SPEC.md has a test, and tests cite only real INVs |
| `scripts/banned_apis_check.py` | Keep clocks, randomness, I/O and UI imports out of the pure core |
| `scripts/manual_mutation_runner.py` | Hand-picked mutants when no mature mutation tool exists |
| `scripts/redteam_runner.py` | Run the red-team corpus against the gates and report recall |

These are building blocks. Copy the ones you need into the target repo's `scripts/` and protect that folder.
They do not replace the project's own linters, test runners or mutation tools (`references/13-ecosystem-map.md`).

## Assets (copy and adapt)

`assets/SPEC.template.md`, `GATES.template.md`, `PULL_REQUEST_TEMPLATE.md`, `CODEOWNERS.example`,
`protected-paths.example.txt`, `banned-apis.example.json`, `ratchet-baseline.example.json`,
`redteam-case.template.md`, `AGENTS.snippet.md`, `gates.sh.example`, `github-actions.gates.example.yml`.

## Reference index (read when you reach that step)

| File | Read when |
|---|---|
| `01-principles-and-threat-model.md` | You need the why, the gaming-pattern catalogue (G1 to G20), or must explain limits to the user |
| `02-assess-and-plan.md` | Orienting, tiering risk, planning phases, retrofitting legacy code, scaling to project size |
| `03-spec-and-invariants.md` | Writing or reviewing SPEC.md, mining invariants, traceability |
| `04-testable-architecture.md` | Carving the pure core, determinism seams, storage/migration design, enforcement |
| `05-test-strategies.md` | Choosing and writing property/model/differential/crash tests; grading existing tests |
| `06-mutation-testing.md` | Adopting mutation testing, triaging survivors, using it to drive agent test-writing |
| `07-evidence-contract.md` | Defining PR classes and the proof each needs; reviewing PRs |
| `08-ratchets-and-budgets.md` | Baselines, performance measurement method, synthetic datasets, size/dependency budgets |
| `09-protect-the-referee.md` | CODEOWNERS, protected paths, hermetic CI, agent boundaries, gate-change process |
| `10-ci-and-feedback.md` | Tiering CI, flakiness policy, writing failure output agents can act on, health metrics |
| `11-red-team-corpus.md` | Building the corpus, gate-breaker sessions, recall tracking |
| `12-contributor-protocol.md` | Working inside a gated repo (mode C); reporting honestly |
| `13-ecosystem-map.md` | Picking tools per language and platform |

## Honest limits (tell the user)

Gates verify conformance to the spec, not the truth of the spec; the spec stays human-owned. Agents will optimize
any visible metric, so use several independent signals and keep growing the red-team corpus. Over-tight gates stall
legitimate work, so start ratchets at today's values. Taste and product judgment are not gateable. Gate maintenance is
real work; budget for it and delete gates that stop paying. Where proofs of optimality are impossible (for example
empirically fitted models), prove invariants and measure calibration instead, and do not claim more than you verified.
