FROM debian:trixie-slim

ENV DEBIAN_FRONTEND=noninteractive \
    PATH="/home/vscode/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# ── System packages ───────────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
        curl git vim make sudo build-essential ca-certificates zsh fzf wget procps less \
    && rm -rf /var/lib/apt/lists/*

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


RUN mkdir -p -m 755 /etc/apt/keyrings \
	&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
	&& cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& mkdir -p -m 755 /etc/apt/sources.list.d \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& apt-get update \
	&& apt-get install gh -y

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
