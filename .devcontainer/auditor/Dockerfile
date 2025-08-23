# syntax=docker/dockerfile:1.10.0
# check=error=true
# 
# AUDITOR TRG DevContainer Dockerfile
# This Dockerfile creates a specialized development environment for smart contract auditing
# with focused tooling, Docker-in-Docker support, and comprehensive security analysis tools.
# 
# Key features:
# - Multi-stage build for Echidna binary
# - Specialized audit tools (slither, mythril, crytic-compile)
# - Foundry framework for testing and interaction
# - Hardhat for development workflows
# - Docker-in-Docker support for containerized tools

# Pull latest Echidna prebuilt image from Crytic
# Echidna is a fuzzing tool for Ethereum smart contracts
FROM --platform=linux/amd64 ghcr.io/crytic/echidna/echidna AS echidna

# Base image: Debian 12 (Bookworm) with VS Code DevContainer support
# This provides a stable, development-focused base for auditing work
FROM mcr.microsoft.com/vscode/devcontainers/base:bookworm

# Switch to root user temporarily for system package installation
USER root

# Install essential system packages for development and auditing
# These packages provide the foundation for all development tools
RUN apt-get update -y && apt-get install -y \
    zsh python3-pip python3-venv python3-dev libpython3-dev build-essential vim curl git sudo pkg-config \
    --no-install-recommends

# The base container usually has a "vscode" user. If not, create one here.
# This ensures consistent user setup across different base images
RUN echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Switch to vscode user for security (drop privileges)
# This ensures all subsequent operations run as non-root user
USER vscode
WORKDIR /home/vscode
ENV HOME=/home/vscode

# Set needed paths for Python, pnpm, and other tools
# Configure environment variables for tool access and package management
ENV USR_LOCAL_BIN=/usr/local/bin
ENV LOCAL_BIN=${HOME}/.local/bin
ENV PNPM_HOME=${HOME}/.local/share/pnpm
ENV PATH=${PATH}:${USR_LOCAL_BIN}:${LOCAL_BIN}:${PNPM_HOME}

# Configure pip to allow system packages in container environment
# This is necessary for some tools that need system-level packages
ENV PIP_BREAK_SYSTEM_PACKAGES=1

# Install uv package manager - modern Python package manager
# Faster and more reliable than pip for tool installation
RUN python3 -m pip install --no-cache-dir --upgrade uv

# Set the default shell to zsh for better development experience
ENV SHELL=/usr/bin/zsh

# Running everything under zsh for consistency and features
SHELL ["/usr/bin/zsh", "-ic"]

# Install Go programming language through asdf version manager
# asdf provides consistent version management across different tools
# Go is required for various Web3 tools and Foundry framework
RUN git clone https://github.com/asdf-vm/asdf.git $HOME/.asdf --branch v0.15.0  && \
    echo '. $HOME/.asdf/asdf.sh' >> $HOME/.zshrc && \
    echo 'fpath=(${ASDF_DIR}/completions $fpath)' >> $HOME/.zshrc && \
    echo 'autoload -Uz compinit && compinit' >> $HOME/.zshrc && \
    . $HOME/.asdf/asdf.sh && \
    asdf plugin add golang && \
    asdf install golang latest && \
    asdf global golang latest

# Install Rust programming language
# Required for various Web3 security tools and Foundry framework
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y && source $HOME/.cargo/env

# Switch to root user temporarily for Node.js installation
USER root

# Install Node.js, npm, yarn, and pnpm through devcontainer features
# These are essential for JavaScript/TypeScript Web3 development and Hardhat
RUN curl -o- https://raw.githubusercontent.com/devcontainers/features/main/src/node/install.sh | bash
RUN chown -R vscode:vscode ${HOME}/.npm

# Switch back to vscode user for security
USER vscode 

# Install Foundry framework for Ethereum development and testing
# Foundry provides Forge (testing), Cast (interaction), and Anvil (local blockchain)
# Essential for smart contract development and testing during audits
RUN curl -L https://foundry.paradigm.xyz | zsh
RUN foundryup

# Install Python-based security analysis tools for auditing
# These tools provide comprehensive smart contract security analysis
# Focused on core auditing tools: slither, mythril, crytic-compile
RUN uv tool install slither-analyzer && \
    uv tool install mythril && \
    uv tool install crytic-compile 

# Install Hardhat and Solhint for Ethereum development
# Hardhat is a popular development environment, Solhint provides linting
RUN pnpm install -g hardhat solhint

# Copy prebuilt Echidna binary from echidna stage to final image
# This provides the prebuilt Echidna tool without rebuilding
COPY --chown=vscode:vscode --from=echidna /usr/local/bin/echidna ${HOME}/.local/bin/echidna
RUN chmod 755 ${HOME}/.local/bin/echidna

# Switch to non-root user for final setup
USER vscode

# Set up user environment with Foundry path
# Ensure Foundry tools are available in the user's shell
RUN echo 'export PATH="/usr/local/foundry/bin:$PATH"' >> /home/vscode/.zshrc

# Switch to root for system cleanup
USER root

# Clean up package cache and temporary files
# This reduces image size and improves security
RUN apt-get autoremove -y && apt-get clean -y

# Final switch to vscode user for development
USER vscode