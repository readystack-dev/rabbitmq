# RabbitMQ 4.4.1 — ReadyStack release `4.4.1-CE-ubuntu24.04-r2`

Generated 2026-11-20 from this release's build records and strategy. Statements that come from a single record name it in brackets; the others come from this release's strategy and build log.

## Release

- **Image:** `readystack/rabbitmq:4.4.1-CE-ubuntu24.04-r2` (linux/amd64). Index digest `sha256:fb0ea7058a39144fa4919240049fe3c6b988641fb99911c25c2840b7a0e1c37e` [`docs/provenance/LOCKED_DIGEST.txt`].
- **Image manifest digest:** `sha256:8256f411db60bbd4b10803641bccf24e3df67ed73cb4133ef12565791f2314e3` [`TARGET_DIGEST.txt`].
- **Upstream:** RabbitMQ 4.4.1 built from source at `v4.4.1` of https://github.com/rabbitmq/rabbitmq-server — upstream notes: https://github.com/rabbitmq/rabbitmq-server/releases/tag/v4.4.1 [`docs/provenance/BUILD_RECIPE.json`].
- **Base image:** none (`FROM scratch`): a chiselled Ubuntu 24.04 runtime, the files Canonical's chisel cut from Ubuntu's own packages (`vendored/chisel/slices.txt`, `vendored/chisel/packages.tsv`); runtime user `rabbitmq`.
- **Build:** `source-rebuild-erlang`, compiled offline with `--network=none`.
- **Supersedes:** ReadyStack release `4.4.1-CE-ubuntu24.04-r1`.

## What changed in this release

- Upstream version unchanged (4.4.1); ReadyStack rebuild.
- RabbitMQ 4.4.1 is unchanged from the release this supersedes (4.4.1-CE-ubuntu24.04-r1); this release rebuilds it. Changes: the image is rebuilt on the current ubuntu:24.04 image (sha256:9c7f200c07e3…), which its maintainers updated after 4.4.1-CE-ubuntu24.04-r1 was built on sha256:2198e4c65076…. Settings, ports and data volumes are those of 4.4.1-CE-ubuntu24.04-r1: replace the container with this image on the same volumes.

## Security status at release (measured 2026-11-20)

- **Docker Scout:** 0 fixable critical/high vulnerabilities; runs as `rabbitmq` (non-root); SBOM + provenance attestations present (SPDX SBOM + SLSA v1) [`docs/provenance/SCOUT_VERDICT.json`].
- **OSV cross-check:** no fixable critical/high vulnerability beyond Docker Scout's findings.
- **Signature:** cosign signature verified by ReadyStack's release gate before publishing; check it yourself with `cosign verify --key https://readystack.dev/keys/cosign.pub readystack/rabbitmq:4.4.1-CE-ubuntu24.04-r2` (signatures are recorded in the public Sigstore transparency log).
- **ReadyStack release check at release:** passed (ReadyStack's own test, not a Docker Scout grade). Vulnerabilities are disclosed every day and the image never changes, so it is re-checked daily, beside Docker Scout's own dated grade, at https://readystack.dev/queue.
- **Known default credential (upstream behaviour, kept):** `guest` / `guest` — RabbitMQ's built-in default user; the image sets loopback_users.guest = false, so it is accepted from any address, including through the published ports. Change it before exposing the service.

## Reproducibility

Rebuilds offline from this archive to the identical image digest `sha256:8256f411db60bbd4b10803641bccf24e3df67ed73cb4133ef12565791f2314e3`: run `./BUILD_OFFLINE.sh`, then `./VERIFY_OFFLINE.sh` (add `--online` to confirm the Docker Hub tag holds the same digest). Before publishing, ReadyStack rebuilt the kit with `./BUILD_OFFLINE.sh` and compared the result with the published digest [`docs/provenance/REPRO_VERIFICATION.json`], then ran `./BUILD_OFFLINE.sh` and `./VERIFY_OFFLINE.sh` from this archive (without `--online`).

## Validation

- 3 of 3 use cases passed (3 authored) against the running image [`docs/validation/test-results.json`].
- The cases are `GET /metrics` (200), `GET /metrics/per-object` (200), `GET /` (200); the suite checks nothing else.

## Known issues

- **high** (upstream): Ships a known default credential (guest) [`RSD-rabbitmq-3c03ddde3f`]
- **low** (ReadyStack package): Docker Hub README: 'a replicated queue keeps the orders safe if a broker node fails' is not possible on a one-node image (RSD-rabbitmq-7c6086743d). [`RSD-rabbitmq-8e88d107e3`] (carried from `4.4.0-CE-ubuntu24.04-r5`, recorded there as `RSD-rabbitmq-468fad0515`)
- **low** (ReadyStack package): Docker Hub README: The problem named by the open RSD-rabbitmq-7c6086743d stands: the README still advertises MQTT and STOMP, which the image does not enable. [`RSD-rabbitmq-532a5bb148`] (carried from `4.4.0-CE-ubuntu24.04-r5`, recorded there as `RSD-rabbitmq-27c6925d6c`)
- **low** (ReadyStack package): Security report: Directly follows the known default credential and reads as 'no defaults exist'. [`RSD-rabbitmq-41c1b3fac7`] (carried from `4.3.6-CE-ubuntu24.04-r1`, recorded there as `RSD-rabbitmq-9192285099`)

## Defects fixed

- 25 problems found before release and fixed before it shipped (25 by review); each is listed in `docs/defects/defects.json`.

## Verify this archive

```
sha256sum -c CHECKSUMS.sha256
./BUILD_OFFLINE.sh && ./VERIFY_OFFLINE.sh --online
cosign verify --key https://readystack.dev/keys/cosign.pub readystack/rabbitmq:4.4.1-CE-ubuntu24.04-r2
```

ReadyStack records every signature in the public Sigstore transparency log (Rekor), so the command needs no extra flags.

