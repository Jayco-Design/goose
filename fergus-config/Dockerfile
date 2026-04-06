# Fergus dev agent Dockerfile - extends the base goose image with GitHub CLI
# Build from repo root: docker build -f fergus-config/Dockerfile -t goose:fergus .

# Use the base goose build
FROM rust:1.82-bookworm AS builder

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential cmake pkg-config libssl-dev libdbus-1-dev libclang-dev \
    protobuf-compiler libprotobuf-dev ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build
COPY . .

ENV CARGO_REGISTRIES_CRATES_IO_PROTOCOL=sparse
RUN cargo build --release --package goose-server

# Runtime stage
FROM debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates libssl3 libdbus-1-3 libgomp1 libxcb1 \
    curl git jq \
    && rm -rf /var/lib/apt/lists/*

# Install GitHub CLI
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
    -o /usr/share/keyrings/githubcli-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
    > /etc/apt/sources.list.d/github-cli.list && \
    apt-get update && \
    apt-get install -y gh && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /build/target/release/goosed /usr/local/bin/goosed

RUN useradd -m -u 1000 -s /bin/bash goose && \
    mkdir -p /home/goose/.config/goose /home/goose/workspaces && \
    chown -R goose:goose /home/goose

ENV PATH="/usr/local/bin:${PATH}"
ENV HOME="/home/goose"

USER goose
WORKDIR /home/goose

ENTRYPOINT ["/usr/local/bin/goosed"]
CMD ["agent"]
