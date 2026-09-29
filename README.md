# pod-github-runner

The `github-runner` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships a GitHub Actions self-hosted runner that builds
CI jobs rootless inside a disposable box.

## What it provides

Installs the pinned `actions/runner` release (default `2.334.0`) under
`${HOME}/actions-runner` (`config.sh` + `run.sh`, run as uid 1000) plus the CI
toolchain — `jq`/`git`/`go`/`cosign`, `qemu-user-static` for aarch64 binfmt
cross-builds, and the rootless nested container stack (`podman`/`buildah`/`skopeo`
via `container-nesting`). The `.NET` runtime deps (`icu`/`krb5`/`openssl`/
`libunwind`/`lttng-ust`) are declared explicitly because the runner's
`installdependencies.sh` has no Arch branch.

| Property | Value |
|---|---|
| Service | `github-runner` (`~/actions-runner/run.sh`, `restart: always`, uid 1000) |
| Requires | `layer-supervisord`, `layer-container-nesting` |
| Volume | `state` at `~/actions-runner` |
| Env | `RUNNER_WORK_DIR=~/actions-runner/_work`, `RUNNER_GROUP=Default` |
| env_accept | `RUNNER_ORG` — the org/user to register with |
| secret_accept | `RUNNER_TOKEN` — the registration token (credential-store backed) |

## Registration

Registration is token-guarded. A deploy with `RUNNER_TOKEN` empty is a clean
no-op (so a check bed brings the image up without registering); with a token,
the `post_enable` hook runs `config.sh --unattended`, and `pre_remove`
deregisters.

```bash
TOKEN=$(gh api -X POST /orgs/myorg/actions/runners/registration-token --jq .token)
charly config <box> -e RUNNER_ORG=myorg -e RUNNER_TOKEN="$TOKEN"
charly start <box>
```

## Layout

- `charly.yml` — the `github-runner:` candy entity plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:github-runner` — the candy properties, the
  packages, the token contract, the lifecycle hooks, and the org-runner routing
  playbook.
- `/charly-distros:container-nesting` — the rootless nested podman/buildah/skopeo
  stack the candy requires.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
