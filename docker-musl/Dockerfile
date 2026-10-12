# Builds hbbs/hbbr as statically linked musl binaries and packages them into a
# FROM scratch image, matching the layout of rustdesk/rustdesk-server (classic).
#
# Why not `cross`: cross is for cross-ARCHITECTURE builds. We are on x86_64
# building for x86_64-unknown-linux-musl, so building natively on Alpine is
# simpler and avoids cross entirely. It also fixes the failure that blocked the
# cross approach -- cross's musl image ships no static OpenSSL, so
# `hbb_common -> native-tls -> openssl-sys` could not link. Alpine's
# openssl-libs-static provides exactly those archives.

FROM rust:alpine AS build

RUN apk add --no-cache \
      musl-dev \
      openssl-dev \
      openssl-libs-static \
      gcc g++ make cmake pkgconf perl \
      file

WORKDIR /src
COPY . .

# --locked so the build matches the committed Cargo.lock.
RUN cargo build --release --locked

# The runtime stage is FROM scratch, so a glibc-dynamic binary would fail at
# exec with "no such file or directory". Fail the build instead of shipping it.
RUN file target/release/hbbs | grep -q 'static' \
 && file target/release/hbbr | grep -q 'static' \
 || (echo "ERROR: hbbs/hbbr are not statically linked" && file target/release/hbbs target/release/hbbr && exit 1)

FROM scratch
COPY --from=build /src/target/release/hbbs /usr/bin/hbbs
COPY --from=build /src/target/release/hbbr /usr/bin/hbbr
WORKDIR /root
ENV HOME=/root
