# Provenance — RabbitMQ (`4.4.1-CE-ubuntu24.04-r2`)

Every item below is copied from a record in this archive; the record is named in brackets.

## Source

- Upstream repository: https://github.com/rabbitmq/rabbitmq-server
- Revision built: `v4.4.1` (commit `aa914c53d2f40f4be3d3a8738d0d5018ea1537e4`)
- Source tree: `e8c52034fe5a90bae604a9185692261bfc80c4399f45cd764c63b273af1356c7` over 3727 files [`docs/provenance/SOURCE_TREE_HASH.json`]
- Community Edition only: enterprise code is removed or refused before the build (CE audit at acquire).

## Build

- Method: `source-rebuild-erlang`; base image `scratch + a chiselled Ubuntu 24.04 runtime (vendored/chisel/slices.txt, packages.tsv)`
- Compiled with `--network=none` from vendored source, dependencies and base images; BuildKit pinned to `moby/buildkit:v0.31.2` (shipped in `toolchain/`).
- SOURCE_DATE_EPOCH: `1795161207` [`SOURCE_DATE_EPOCH.txt`]
- Frozen inputs: [`docs/provenance/FREEZE_MANIFEST.json`], build recipe: [`docs/provenance/BUILD_RECIPE.json`].

## Output

- Published image: `readystack/rabbitmq:4.4.1-CE-ubuntu24.04-r2` — index digest `sha256:fb0ea7058a39144fa4919240049fe3c6b988641fb99911c25c2840b7a0e1c37e` [`docs/provenance/LOCKED_DIGEST.txt`]
- linux/amd64 image digest `sha256:8256f411db60bbd4b10803641bccf24e3df67ed73cb4133ef12565791f2314e3` — reproduced from this archive before publishing [`docs/provenance/REPRO_VERIFICATION.json`].

## Attestations and signature

- SBOM + SLSA provenance attached to the image: attached: an SPDX SBOM and SLSA v1 provenance [`docs/provenance/SCOUT_VERDICT.json`]
- cosign signature by the ReadyStack release key: verified before publishing. Check it: `cosign verify --key https://readystack.dev/keys/cosign.pub readystack/rabbitmq:4.4.1-CE-ubuntu24.04-r2`

## Pinned inputs, and why

- 30 pinned inputs, each with where it came from, why this one, how it was checked and when it was decided: `docs/PINS.md` [`docs/provenance/PINS.json`] (upstream source 2, base and build images 4, image digests 3, language runtimes and toolchains 2, dependency versions readystack sets 3, pins in the kit's dockerfiles 8, readystack's build engine 6, dependencies upstream chose 2).

## Build trail

- How this release passed through the ReadyStack harness, record by record (every chain, drive, stage, review round, defect and check, each naming the record it is copied from): `ReadyStack-rabbitmq-4.4.1-CE-ubuntu24.04-r2.trail.json`, published beside this archive and signed with the release key (`ReadyStack-rabbitmq-4.4.1-CE-ubuntu24.04-r2.trail.json.sigstore`), and readable at https://readystack.dev/products/rabbitmq/trail/4.4.1-CE-ubuntu24.04-r2.

