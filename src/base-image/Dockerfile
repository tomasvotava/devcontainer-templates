FROM debian:trixie-slim

ENV DEBIAN_FRONTEND=noninteractive \
    PATH="/home/vscode/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

# ── System packages ───────────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --install-recommends \
        curl git vim make sudo build-essential ca-certificates zsh fzf wget procps less openssh-client gh locales \
    && rm -rf /var/lib/apt/lists/* \
    && locale-gen en_US.UTF-8

# ── Node.js v22 LTS via NodeSource ───────────────────────────────────────────
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# ── vscode user (UID/GID 1000) with passwordless sudo ────────────────────────
RUN groupadd --gid 1000 vscode \
    && useradd --uid 1000 --gid 1000 --shell /bin/bash --create-home vscode \
    && echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/vscode \
    && chmod 0440 /etc/sudoers.d/vscode


# ── Volume mount points ──────────────────────────────────────────
RUN mkdir /commandhistory \
    && touch /commandhistory/.bash_history \
    && chown -R vscode:vscode /commandhistory \
    && mkdir -p /home/vscode/.claude /home/vscode/.config/Google \
    && chown -R vscode:vscode /home/vscode/.claude /home/vscode/.config

# ── Switch to vscode for user-scoped installs ────────────────────────────────
USER vscode
WORKDIR /home/vscode
    
# zsh
RUN sh -c "$(curl -L https://github.com/deluan/zsh-in-docker/releases/download/v1.2.1/zsh-in-docker.sh)" -- \
    -t robbyrussell \
    -p git -p python -p node -p fzf \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions \
    -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
    -a 'export PATH="$HOME/.local/bin:$PATH"' \
    -x

ENV SHELL=/bin/zsh

# ── uv (standalone installer) ────────────────────────────────────────────────
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
RUN curl -fsSL https://claude.ai/install.sh | bash

# ── Python 3.12, 3.13, 3.14 via uv ──────────────────────────────────────────
RUN uv python install 3.12 3.13 3.14 \
    && ln -s $(which python3.14) /home/vscode/.local/bin/python

# ── Poetry via uv tool (uses uv-managed Python; no separate system python3 needed) ──
RUN uv tool install poetry
