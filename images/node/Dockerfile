# ci-node: Node + Bun untuk proyek JS/TS (Next.js, Vite, Prisma, dsb).
ARG NODE_MAJOR=22
FROM docker.io/library/node:${NODE_MAJOR}-bookworm

RUN apt-get update -qq \
    && apt-get install -y -qq --no-install-recommends ca-certificates git openssl openssh-client rsync unzip \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g bun@1 >/dev/null 2>&1 && rm -rf /root/.npm \
    && git config --system --add safe.directory '*'

WORKDIR /workspace
