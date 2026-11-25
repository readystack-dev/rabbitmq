# GENERATED L-009 hermetic Erlang build (--network=none, SBOM+SLSA-attestation-producing).
# Pinned bases from BASE_IMAGE_DIGESTS.lock; vendored erlang.mk deps/ + source-dist tarball + debs from Dockerfile.vendor.
FROM ubuntu:24.04@sha256:9c7f200c07e3d4a03e7249d2c16bf66a51120fb876aa2242cf31451412543289 AS otpbuild
ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8 LC_ALL=C.UTF-8
# dpkg-install the vendored build toolchain (gcc/g++/make/perl/libssl-dev/libncurses-dev/...) — no apt, fully offline.
RUN --mount=type=bind,source=vendored/build-debs,target=/tmp/bdebs \
    set --; for f in /tmp/bdebs/*.deb; do [ -e "$f" ] && set -- "$@" "$f"; done; \
    for pass in 1 2 3 4 5; do [ $# -eq 0 ] || dpkg -i --skip-same-version "$@" >/dev/null 2>&1 && break; done; \
    dpkg --configure -a && bad= && for d in "$@"; do p=$(dpkg-deb -f "$d" Package) v=$(dpkg-deb -f "$d" Version); \
      case "$(dpkg-query -W -f='${Version} ${db:Status-Abbrev}' "$p" 2>/dev/null)" in "$v ii"*) ;; *) bad="$bad $p=$v" ;; esac; done && \
    { [ -z "$bad" ] || { echo "vendored debs not installed at their vendored version:$bad" >&2; false; }; }
COPY vendored/otp-src/ /tmp/otp-src/
ARG SOURCE_DATE_EPOCH=1795161207
ENV SOURCE_DATE_EPOCH=${SOURCE_DATE_EPOCH}
# Erlang/OTP 28.5.0.8 from source into /usr/local, linked against this base's OpenSSL/ncurses/zlib.
RUN : "${SOURCE_DATE_EPOCH:?the OTP build needs SOURCE_DATE_EPOCH}" && \
    echo "8a696e15bcbe5d57197b1f4c03c92d415f9fd9af70f9222a05911c8a5463cba6  /tmp/otp-src/otp_src_28.5.0.8.tar.gz" | sha256sum -c - && \
    echo "2f87be1702583ecbeee82c0ad4d6353de96463cfa0fa6e7557e05f68d90da869  /tmp/otp-src/elixir-1.20.4.tar.gz" | sha256sum -c - && \
    mkdir -p /usr/src/otp && tar -xzf /tmp/otp-src/otp_src_28.5.0.8.tar.gz -C /usr/src/otp --strip-components 1 --no-same-owner && \
    cd /usr/src/otp && export ERL_TOP=/usr/src/otp && \
    ./configure --prefix=/usr/local --without-javac --without-wx --without-odbc --disable-sctp --enable-jit && \
    make -j"$(nproc)" && make install && \
    find /usr/local/lib/erlang -type d -name examples -exec rm -rf '{}' + && \
    cd / && rm -rf /usr/src/otp
# Elixir 1.20.4 from source into /opt/elixir: a BUILD tool only (the product bundles what it needs).
ENV ERL_COMPILER_OPTIONS=deterministic
RUN mkdir -p /usr/src/elixir && tar -xzf /tmp/otp-src/elixir-1.20.4.tar.gz -C /usr/src/elixir --strip-components 1 --no-same-owner && \
    make -C /usr/src/elixir && make -C /usr/src/elixir install PREFIX=/opt/elixir && \
    rm -rf /usr/src/elixir /tmp/otp-src
ENV PATH=/opt/elixir/bin:${PATH}
# Fail here, not at boot, if the crypto NIF did not build, or did not link the distro's libcrypto.
RUN erl -noshell -eval 'ok = crypto:start(), io:format("OTP ~s ~p~n", [erlang:system_info(otp_release), crypto:info_lib()]), halt().' && \
    ldd /usr/local/lib/erlang/lib/crypto-*/priv/lib/crypto.so | grep -qE 'libcrypto\.so\.3 => /(usr/)?lib/x86_64-linux-gnu/' && \
    elixir --version
FROM otpbuild AS erlbuild
WORKDIR /src
COPY vendored/rabbitmq-src/ .
# `make package-generic-unix` re-runs erlang.mk's source-dist, which calls git (commit date +
# `git ls-files`). The vendor stage rm'd .git (a nested repo can't be committed into the kit tree).
# Re-create a THROWAWAY local git here so those calls succeed — git init/add/commit are pure-local,
# no network, so --network=none holds. Fixes the Error-123 `touch: invalid date format` from an empty git log.
ARG SOURCE_DATE_EPOCH=1795161207
RUN git config --global --add safe.directory '*' && git init -q && git add -A -f && \
    GIT_AUTHOR_DATE="@${SOURCE_DATE_EPOCH} +0000" GIT_COMMITTER_DATE="@${SOURCE_DATE_EPOCH} +0000" \
    git -c user.email=build@readystack.dev -c user.name=readystack commit -q -m dist
# erlang.mk zips escripts with `7z a`, which stamps each entry with its file's mtime (the compile time): run
# it through kitforge-7z, which first sets the archived files' mtime to SOURCE_DATE_EPOCH.
COPY kitforge-7z /usr/local/kitforge/bin/7z
RUN chmod 0755 /usr/local/kitforge/bin/7z
ENV PATH=/usr/local/kitforge/bin:${PATH}
RUN make package-generic-unix PROJECT_VERSION=4.4.1
# rebar escriptize zips in memory with OTP's zip module, which stamps each entry with the build's clock time:
# kitforge-zip-sde sets every escript's and .ez's zip times to SOURCE_DATE_EPOCH once the release is in place.
COPY kitforge-zip-sde /usr/local/kitforge/kitforge-zip-sde
RUN mkdir -p /opt/rabbitmq && tar -xf PACKAGES/*generic-unix*.tar.* -C /opt/rabbitmq --strip-components 1 && \
    sed -i 's/^SYS_PREFIX=.*$/SYS_PREFIX=/' /opt/rabbitmq/sbin/rabbitmq-defaults && \
    find /opt/rabbitmq -name '*.gz' -exec sh -c 'gzip -dc "$0" | gzip -n -9 > "$0.det" && mv "$0.det" "$0"' {} ';' && \
    python3 /usr/local/kitforge/kitforge-zip-sde /opt/rabbitmq
RUN set -eu; find /usr/local/lib/erlang /opt/rabbitmq -type f | LC_ALL=C sort | while read -r f; do \
      [ "$(od -An -c -N4 "$f" | tr -d ' ')" = 177ELF ] || continue; \
      ldd "$f" 2>/dev/null | sed -n 's/^[[:space:]]*\([^[:space:]]*\) => \/.*/\1/p; s/^[[:space:]]*\(\/[^[:space:]]*\) (0x.*/\1/p; s/^[[:space:]]*\([^[:space:]]*\) => not found.*/absent \1/p'; \
    done | LC_ALL=C sort -u > /tmp/ldd.all; \
    grep '^absent ' /tmp/ldd.all | cut -d' ' -f2 | tr '\n' ' ' | sed 's/^./not in the release image either (not required): &/'; echo; \
    grep -v '^absent ' /tmp/ldd.all | while read -r lib; do case "$lib" in \
      /*) echo "$lib" ;; *) [ -n "$(find /usr/local/lib/erlang /opt/rabbitmq -name "$lib" -print -quit)" ] || echo "$lib" ;; esac; \
    done > /kitforge.needs && rm -f /tmp/ldd.all && \
    echo "the Erlang VM, its NIFs and the release load: $(tr '\n' ' ' < /kitforge.needs)"
# rs-probe: the HEALTHCHECK helper (standard library only, from the kit's rs-probe.go)
FROM golang:1.27.3-bookworm@sha256:972b4110ba0f47539a975c31e52c64765e1ff19f2d74469ae4a595c6676e80a5 AS build_rs_probe
WORKDIR /probe
COPY rs-probe.go main.go
COPY rs-probe.mod go.mod
ENV CGO_ENABLED=0 GOTOOLCHAIN=local GOFLAGS="-buildvcs=false" GOPROXY=off SOURCE_DATE_EPOCH=1795161207
RUN go build -trimpath -ldflags='-s -w' -o /rs-probe .
# the chiselled rootfs (vendored/chisel, cut by Dockerfile.vendor), the service user, the binaries
FROM ubuntu:24.04@sha256:9c7f200c07e3d4a03e7249d2c16bf66a51120fb876aa2242cf31451412543289 AS rootfs
ARG SOURCE_DATE_EPOCH=1795161207
ADD vendored/chisel/rootfs.tar /rootfs/
RUN printf 'rabbitmq:x:999:999::/var/lib/rabbitmq:/bin/sh\n' >> /rootfs/etc/passwd && printf 'rabbitmq:x:999:\n' >> /rootfs/etc/group && \
    mkdir -p /rootfs/var/lib/rabbitmq /rootfs/etc/rabbitmq /rootfs/etc/rabbitmq/conf.d /rootfs/var/log/rabbitmq && chown -R 999:999 /rootfs/var/lib/rabbitmq /rootfs/etc/rabbitmq /rootfs/etc/rabbitmq/conf.d /rootfs/var/log/rabbitmq
COPY --from=build_rs_probe /rs-probe /rootfs/usr/local/bin/rs-probe
COPY scaffold_0_enabled_plugins /rootfs/etc/rabbitmq/enabled_plugins
COPY scaffold_1_10-defaults.conf /rootfs/etc/rabbitmq/conf.d/10-defaults.conf
COPY --from=otpbuild /usr/local/lib/erlang /rootfs/usr/local/lib/erlang
COPY --from=otpbuild /usr/local/bin/ /rootfs/usr/local/bin/
COPY --from=erlbuild /opt/rabbitmq /rootfs/opt/rabbitmq
COPY --from=erlbuild /kitforge.needs /tmp/needs/rabbitmq
RUN chmod 755 /rootfs/usr/local/bin/rs-probe
RUN set -eu; ROOT=/rootfs; \
    chown -R 999:999 "$ROOT/var/lib/rabbitmq" "$ROOT/etc/rabbitmq" "$ROOT/var/log/rabbitmq" "$ROOT/opt/rabbitmq"; \
    chmod 0750 "$ROOT/var/lib/rabbitmq"
RUN --mount=type=bind,source=vendored/chisel,target=/chisel set -eu; \
    for n in /tmp/needs/*; do while read -r lib; do [ -n "$lib" ] || continue; \
      case "$lib" in /*) [ -e "/rootfs$lib" ] ;; *) [ -n "$(find /rootfs/usr/lib /rootfs/usr/lib64 -name "$lib" 2>/dev/null | head -1)" ] ;; esac || \
      { echo "$(basename "$n") needs $lib, which the chiselled rootfs does not have: add its slice to runtime_base.slices" >&2; exit 1; }; \
    done < "$n"; done; rm -rf /tmp/needs; \
    [ -e /rootfs/etc/os-release ] || { echo "no /etc/os-release: base-files_release-info is missing" >&2; exit 1; }; \
    [ "$(grep -c '^Package: ' /rootfs/var/lib/dpkg/status)" = "$(wc -l < /chisel/packages.tsv)" ] || \
      { echo "/var/lib/dpkg/status does not list every package of vendored/chisel/packages.tsv" >&2; exit 1; }; \
    rm -rf /rootfs/var/log/dpkg.log /rootfs/var/log/alternatives.log /rootfs/var/log/apt /rootfs/var/cache/ldconfig/aux-cache /rootfs/var/cache/fontconfig
FROM scratch
ARG SOURCE_DATE_EPOCH=1795161207
COPY --from=rootfs /rootfs/ /
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin RABBITMQ_HOME=/opt/rabbitmq RABBITMQ_DATA_DIR=/var/lib/rabbitmq HOME=/var/lib/rabbitmq PATH=/opt/rabbitmq/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin RABBITMQ_ENABLED_PLUGINS_FILE=/etc/rabbitmq/enabled_plugins RABBITMQ_LOGS=- LANG=C.UTF-8 LANGUAGE=C.UTF-8 LC_ALL=C.UTF-8
LABEL org.opencontainers.image.title="RabbitMQ"
LABEL org.opencontainers.image.version="4.4.1-CE-ubuntu24.04-r2"
LABEL org.opencontainers.image.licenses="MPL-2.0"
LABEL org.opencontainers.image.source="https://github.com/rabbitmq/rabbitmq-server"
LABEL org.opencontainers.image.description="RabbitMQ open-source AMQP 0-9-1 / AMQP 1.0 message broker (single-node CE, non-root, management + Prometheus enabled)"
LABEL org.opencontainers.image.vendor="ReadyStack.dev"
LABEL readystack.product="rabbitmq"
LABEL readystack.edition="CE"
LABEL readystack.build_method="source-rebuild-erlang"
LABEL org.opencontainers.image.revision="aa914c53d2f40f4be3d3a8738d0d5018ea1537e4"
HEALTHCHECK --interval=5s --timeout=120s --start-period=30s \
  CMD rabbitmq-diagnostics -q check_running || exit 1
USER rabbitmq
EXPOSE 5672 15672 15692
ENTRYPOINT ["rabbitmq-server"]
