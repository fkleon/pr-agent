ARG VERSION=0.47.0
FROM python:3.14-slim-trixie AS pr-agent

# tools
RUN apt-get update && \
  DEBIAN_FRONTEND=noninteractive \
  apt-get install -y --no-install-recommends \
    git curl ca-certificates && \
  rm -rf /var/lib/apt/lists/*

# Install pr-agent from source
ARG VERSION
WORKDIR /app
RUN git init && \
  git remote add origin https://github.com/the-pr-agent/pr-agent.git && \
  git fetch --depth=1 origin "$VERSION" && \
  git checkout FETCH_HEAD

# Apply custom patches
COPY *.patch .
RUN git -c user.name=ci -c user.email=ci@localhost am *.patch

ENV UV_COMPILE_BYTECODE=1 \
    UV_NO_CACHE=1

RUN --mount=from=ghcr.io/astral-sh/uv:0.12,source=/uv,target=/bin/uv \
    uv --no-cache pip install --system ".[github,litellm]"

# TODO: Use the following line to install a specific version of pr-agent.
#COPY pr-agent/pyproject.toml pr-agent/uv.lock ./
#RUN --mount=from=ghcr.io/astral-sh/uv:0.12,source=/uv,target=/bin/uv \
#    uv pip install --system "pr-agent[github,litellm]==${VERSION}"

# Require litellm package update to pull new model definitions
ENV LITELLM_LOCAL_MODEL_COST_MAP=true

ENTRYPOINT ["pr-agent"]
CMD ["--help"]