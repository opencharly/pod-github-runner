# AGENTS.md — pod-github-runner

Standalone candy repo for the `github-runner` candy — a GitHub Actions
self-hosted runner that builds CI jobs rootless inside a disposable box. The
candy lives in `charly.yml` at the repo root.

Canonical files:

- `charly.yml` — the `github-runner:` candy entity (description, `require`,
  `distro`, `volume`, `env`, `env_accept`, `secret_accept`, `var`, `hook`,
  `service`, `plan`) and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:github-runner` — the owning skill: candy properties, packages,
  the token contract, the lifecycle hooks, and the org-runner routing playbook.
  Load before editing, building, deploying, or troubleshooting this candy.
- `/charly-distros:container-nesting` — the rootless nested podman/buildah/skopeo
  stack the candy requires (`layer-container-nesting`).
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, ports).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs and
  `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, `env_accept` / `secret_accept`, lifecycle hooks).
- `/charly-build:secrets` — the credential store backing `RUNNER_TOKEN`.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert `config.sh` + `run.sh` presence, `config.sh --version` (proving the
  `.NET` deps resolved), `skopeo`/`buildah`/`cosign`/`go` versions, the aarch64
  qemu interpreter, rootless nested `podman`, the runner running as uid 1000, and
  the `newuidmap` capability.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `github-runner:` candy entity in `charly.yml`; the `skill:` entity in
  the same file is the owning skill's source — a candy change and its skill change
  land together.
- The runner version pin is the `RUNNER_VERSION` var; the registration logic is
  the token-guarded `post_enable` / `pre_remove` hooks. Keep them token-guarded —
  an empty `RUNNER_TOKEN` must stay a clean no-op for token-less beds.
- `podman`/`buildah`/`skopeo`/`crun`/`fuse-overlayfs` come from
  `layer-container-nesting`; do not redeclare them here (R3).
- The `skill:` entity is the source for `/charly-distros:github-runner`; never
  edit the generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
