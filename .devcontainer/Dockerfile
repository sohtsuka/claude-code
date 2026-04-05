# node:24.14.1-trixie
FROM node@sha256:dcc3e56b82427ddc3b91ca2b18499450d670fc58251d944e5107d8ef2899f841

ARG TZ
ENV TZ="$TZ"

ARG CLAUDE_CODE_VERSION=latest

# Install basic development tools and iptables/ipset
RUN apt-get update && apt-get install -y --no-install-recommends \
  less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  iptables \
  ipset \
  iproute2 \
  dnsutils \
  aggregate \
  jq \
  nano \
  vim \
  zip \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Ensure default node user has access to /usr/local/share
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R node:node /usr/local/share

ARG USERNAME=node

# Persist bash history.
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/node/.claude && \
  chown -R node:node /workspace /home/node/.claude

WORKDIR /workspace

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Set up non-root user
USER node

# Install global packages
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=nano
ENV VISUAL=nano

# Default powerline10k theme
ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# Install Claude
RUN npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}


# --- Starting additional steps ---
ENV SDKMAN_DIR /home/node/.sdkman
ARG SDKMAN_JAVA_VERSION=25.0.2-tem
ARG SDKMAN_GRADLE_VERSION=9.4.1
ARG PNPM_VERSION=10.33

SHELL ["/bin/bash", "-c"]

# Install JDK & Gradle via sdkman
# Use saved installation script to fix sdkman version
# ```
# curl -s "https://get.sdkman.io" -o sdkman_install_stable.sh
# ```
# source: https://github.com/sdkman/sdkman-hooks/blob/master/app/views/install_stable.scala.txt
COPY --chown=node sdkman_install_stable.sh /tmp/sdkman_install_stable.sh
RUN bash /tmp/sdkman_install_stable.sh \
    && rm /tmp/sdkman_install_stable.sh
RUN source "$SDKMAN_DIR/bin/sdkman-init.sh" \
  && sdk install java ${SDKMAN_JAVA_VERSION} \
  && sdk install gradle ${SDKMAN_GRADLE_VERSION}

# Install pnpm
USER root
RUN corepack enable pnpm
USER node
ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0
RUN corepack use pnpm@${PNPM_VERSION}

# --- Finished additional steps ---

# Copy and set up firewall script
COPY init-firewall.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh && \
  echo "node ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/node-firewall && \
  chmod 0440 /etc/sudoers.d/node-firewall
USER node
