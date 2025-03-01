# Will work locally only after prior universal setup key download

FROM debian:bookworm-slim as builder

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y curl clang openssl libssl-dev gcc g++ \
    pkg-config build-essential libclang-dev && \
    rm -rf /var/lib/apt/lists/*

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN curl https://sh.rustup.rs -sSf | bash -s -- -y && \
    rustup install nightly-2023-08-21 && \
    rustup default nightly-2023-08-21

WORKDIR /usr/src/zksync
COPY . .

RUN cargo build --release

FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y curl libpq5 ca-certificates && rm -rf /var/lib/apt/lists/*

# copy VK required for proof wrapping
COPY prover/vk_setup_data_generator_server_fri/data/ /prover/vk_setup_data_generator_server_fri/data/

# copy universal setup key required for proof compression
COPY setup_2\^26.key /setup_2\^26.key

COPY --from=builder /usr/src/zksync/target/release/zksync_proof_fri_compressor /usr/bin/

ENTRYPOINT ["zksync_proof_fri_compressor"]
