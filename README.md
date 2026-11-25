# RabbitMQ: ReadyStack build kit

The offline build kit for the Docker image `readystack/rabbitmq:4.4.1-CE-ubuntu24.04-r2`. It holds what the image is built from (upstream source, base images, toolchain, pinned dependencies) and the documents, so the image can be rebuilt with no network access (`docker build --network=none`).

- Release: [4.4.1-CE-ubuntu24.04-r2](https://github.com/readystack-dev/rabbitmq/releases/tag/4.4.1-CE-ubuntu24.04-r2)
- Image: [`docker.io/readystack/rabbitmq:4.4.1-CE-ubuntu24.04-r2`](https://hub.docker.com/r/readystack/rabbitmq/tags), manifest digest `sha256:8256f411db60bbd4b10803641bccf24e3df67ed73cb4133ef12565791f2314e3`
- Product page: https://readystack.dev/products/rabbitmq
- Defects: this repository's Issues (defects recorded from 25 November 2026 on) and https://readystack.dev/defects.html?product=rabbitmq

RabbitMQ is distributed under its upstream licence (MPL-2.0); the kit contains upstream's source under that licence, and each vendored component under its own.

## Download

From the [4.4.1-CE-ubuntu24.04-r2 release](https://github.com/readystack-dev/rabbitmq/releases/tag/4.4.1-CE-ubuntu24.04-r2): `ReadyStack-rabbitmq-4.4.1-CE-ubuntu24.04-r2.tar.gz` (741.5 MB), SHA-256 `643009c4b0fc6482b69a6728fa2f6dd87058b5fcbeb21d762dd36e5e30e4e502`.

```sh
sha256sum --check --ignore-missing SHA256SUMS
```

## Verify and rebuild

The rebuild statement is signed with the ReadyStack release key. Check the signature before you rely on it:

```sh
cosign verify-blob --key https://readystack.dev/keys/cosign.pub \
  --bundle ReadyStack-rabbitmq-4.4.1-CE-ubuntu24.04-r2.rebuild.json.sigstore ReadyStack-rabbitmq-4.4.1-CE-ubuntu24.04-r2.rebuild.json
```

Unpack with `-p` so file modes are kept, then follow the kit's own README.md and docs/REBUILD.md:

```sh
tar -xpzf ReadyStack-rabbitmq-4.4.1-CE-ubuntu24.04-r2.tar.gz
cd 4.4.1-CE-ubuntu24.04-r2
less README.md docs/REBUILD.md
```

The build trail (`ReadyStack-rabbitmq-4.4.1-CE-ubuntu24.04-r2.trail.json`, signed the same way, and its Markdown copy) records how this release was built and checked.

## This repository

`recipe/` holds the kit's recipe files (Dockerfiles, build scripts, pins) as published at https://readystack.dev/products/rabbitmq/recipe. The complete kit is the release download. Each release is a tag here.
