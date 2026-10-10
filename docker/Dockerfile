ARG VERSION=0.47.0
FROM python:3.14-slim-trixie AS pr-agent

# tools
RUN apt-get update && \
  DEBIAN_FRONTEND=noninteractive \
  apt-get install -y --no-install-recommends \
    git curl ca-certificates && \
  rm -rf /var/lib/apt/lists/*

# Require litellm package update to pull new model definitions
ENV LITELLM_LOCAL_MODEL_COST_MAP=True
ENV UV_COMPILE_BYTECODE=1

WORKDIR /app

# Install pr-agent from source
COPY pr-agent .
#RUN --mount=from=ghcr.io/astral-sh/uv:0.12,source=/uv,target=/bin/uv \
#    uv sync --frozen --python python3 --no-managed-python --no-dev --no-editable --extra github --extra litellm
#ENV PATH="/app/.venv/bin:$PATH"
RUN --mount=from=ghcr.io/astral-sh/uv:0.12,source=/uv,target=/bin/uv \
    uv pip install --system ".[github,litellm]"

# TODO: Use the following line to install a specific version of pr-agent.
#COPY pr-agent/pyproject.toml pr-agent/uv.lock ./
#RUN --mount=from=ghcr.io/astral-sh/uv:0.12,source=/uv,target=/bin/uv \
#    uv pip install --system "pr-agent[github,litellm]==${VERSION}"

ENTRYPOINT ["pr-agent"]
CMD ["--help"]