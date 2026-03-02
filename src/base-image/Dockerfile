FROM debian:trixie-slim

ENV DEBIAN_FRONTEND=noninteractive \
    PATH="/home/vscode/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# ── System packages ───────────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
        curl git vim make sudo build-essential ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# ── Node.js v22 LTS via NodeSource ───────────────────────────────────────────
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# ── Claude Code CLI (global; runs as root so npm -g has write access) ─────────
RUN npm install -g @anthropic-ai/claude-code

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

# ── uv (standalone installer) ────────────────────────────────────────────────
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# ── Python 3.12, 3.13, 3.14 via uv ──────────────────────────────────────────
RUN uv python install 3.12 3.13 3.14 \
    && ln -s $(which python3.14) /home/vscode/.local/bin/python

# ── Poetry via uv tool (uses uv-managed Python; no separate system python3 needed) ──
RUN uv tool install poetry

# ── .bashrc: history persistence + PATH (for interactive shells) ─────────────
RUN printf '%s\n' \
        '' \
        '# Bash history persistence' \
        'export HISTFILE=/commandhistory/.bash_history' \
        'export PROMPT_COMMAND="history -a"' \
        '' \
        '# User local bin' \
        'export PATH="$HOME/.local/bin:$PATH"' \
    >> /home/vscode/.bashrc
