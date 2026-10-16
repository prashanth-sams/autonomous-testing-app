# QA agent configs

Example configs for [autonomous-test-agent](https://github.com/prashanth-sams/autonomous-test-agent)
(v1.0.6), each written against the Test Playground (`index.html`). Each one
covers a single feature, and the comments in each file explain what it does and
why. Every example was run against the Playground; the budgets, verdicts and
fingerprints in the comments come from those runs.

To run one in CI, point `.github/workflows/qa-agent.yml` at it and use the
`mode` its header names:

```yaml
- uses: prashanth-sams/autonomous-test-agent@v1
  with:
    url: ${{ env.APP_URL }}
    mode: explore                      # or pr
    config: qa/agent-critical-journeys.yaml
```

## Suggested reading order

| File | Teaches | Mode |
| --- | --- | --- |
| [agent-minimal.yaml](agent-minimal.yaml) | The smallest config, and what the defaults do | explore |
| [agent-all-options.yaml](agent-all-options.yaml) | Every option at its default value, as a reference | either |
| [agent-explore-nightly.yaml](agent-explore-nightly.yaml) | A long exploratory run on a schedule; `limits` | explore |
| [agent-critical-journeys.yaml](agent-critical-journeys.yaml) | Scripted journeys, every step kind, budget per journey | either |
| [agent-pr-impact-map.yaml](agent-pr-impact-map.yaml) | Mapping changed files to journeys; globs; confidence | pr |
| [agent-pr-always-run-critical.yaml](agent-pr-always-run-critical.yaml) | A smoke test on every PR, whatever the diff | pr |
| [agent-pr-free-exploration.yaml](agent-pr-free-exploration.yaml) | Exploring the rest of the app with leftover steps | pr |
| [agent-auth-login-steps.yaml](agent-auth-login-steps.yaml) | Signing in with scripted steps before testing | either |
| [agent-auth-storage-state.yaml](agent-auth-storage-state.yaml) | Starting from a saved session ([auth/storage-state.json](auth/storage-state.json)) | either |
| [agent-business-rules.yaml](agent-business-rules.yaml) | `rules`: the four kinds of business invariant | either |
| [agent-safety.yaml](agent-safety.yaml) | Allowed hosts, prohibited and destructive actions, redaction | either |
| [agent-oracles.yaml](agent-oracles.yaml) | Turning oracles on and off; ignoring known noise | either |
| [agent-known-false-positives.yaml](agent-known-false-positives.yaml) | Dismissing findings by fingerprint | either |
| [agent-blocking-gate.yaml](agent-blocking-gate.yaml) | Failing the PR check on serious, reproduced defects | pr |
| [agent-mobile-viewport.yaml](agent-mobile-viewport.yaml) | `browser`: engine, headless, viewport size | either |
| [agent-evidence.yaml](agent-evidence.yaml) | `reporting`: video, trace, HAR; `target.build` | either |

`agent.yaml` and `agent2.yaml` predate these examples; the workflow runs
`agent2.yaml`.

## Rules that apply to every config

- **pr and explore select differently.** `explore` runs every critical journey
  and then explores the whole app. `pr` runs only what the diff maps to
  through `impactMap`, so a config with no `impactMap` tests nothing in `pr`
  mode unless `alwaysRunCriticalJourneys` or `freeExploration` is on.
- **Lists replace defaults; objects merge.** Setting `destructiveKeywords`,
  `ignorePatterns` or `redactKeys` drops the built-in entries, so repeat the
  ones you still want.
- **Nothing is templated.** `${VAR}` stays literal. Write the config in a
  workflow step to inject secrets.
- **Scripted values are not redacted.** A password in a critical journey
  script shows in `run.json` and in reproduction steps in the PR comment. Use
  test accounts.
- **The action installs Chromium only.** `firefox` and `webkit` work from the
  CLI after `npx playwright install`.
- **`planner.kind: ai` has no effect yet.** Only the heuristic planner is
  built.
