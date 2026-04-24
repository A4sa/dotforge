# ============================================================================
#  Dockerfile — dotforge workspace container
#
#  Author      : Abdul Sattar <abdul.linuxdev@gmail.com>
#  Repository  : https://github.com/A4sa/dotforge.git
#  Description : Builds a container with the full dotforge developer workspace
#                pre-installed — Vim (with plugins), tmux, shell aliases,
#                and embedded Linux toolchain dependencies.
#
#  WHY USE THIS?
#  -------------
#  - Reproducible workspace on any machine, any OS
#  - Onboard a new team member in 2 minutes (docker pull / docker run)
#  - Use on a remote build server and attach via VS Code or SSH
#  - Isolate project environments without touching the host system
#
#  BUILD:
#    cd dotforge/docker
#    ./docker.sh build
#
#  RUN (interactive workspace):
#    ./docker.sh run
#
#  The base image is detected from the host at build time by docker.sh.
#  This ARG receives the value: ubuntu:22.04 or ubuntu:24.04 etc.
# ============================================================================

ARG BASE_IMAGE=ubuntu:24.04
FROM ${BASE_IMAGE}

# ── Build-time metadata ───────────────────────────────────────────────────────
LABEL maintainer="Abdul Sattar <abdul.linuxdev@gmail.com>"
LABEL description="dotforge developer workspace — Vim + tmux + embedded tools"
LABEL repository="https://github.com/A4sa/dotforge"

# ── Prevent interactive prompts during apt installs ───────────────────────────
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Kolkata

# ── System packages ───────────────────────────────────────────────────────────
#
# WHY EACH PACKAGE:
#   vim-gtk3          — Vim with +clipboard and +python3 (needed by UltiSnips)
#   tmux              — terminal multiplexer (core of dotforge workflow)
#   git               — version control (fugitive plugin needs it)
#   curl wget         — plugin manager bootstrap + general downloads
#   fzf               — fuzzy finder (Ctrl+P in Vim, Ctrl+R in shell)
#   ripgrep           — fast content search (:Rg in Vim)
#   bat               — syntax-highlighted file preview (ff alias)
#   universal-ctags   — code navigation (Tagbar plugin needs this)
#   cscope            — symbol navigation (kernel/BSP code)
#   clang-format      — C code formatter (auto-format on save)
#   picocom minicom   — serial console (board bring-up)
#   xclip             — system clipboard bridge (tmux copy-mode)
#   build-essential   — gcc, make, libc headers (compile kernel modules)
#   python3 python3-pip — Python 3 (UltiSnips engine + embedded scripts)
#   gcc-aarch64-linux-gnu — ARM64 cross-compiler (BSP builds)
#   gcc-arm-linux-gnueabihf — ARM32 hard-float cross-compiler
#   adb               — Android Debug Bridge (embedded Android targets)
#   tree              — directory listing (project navigation)
#   jq                — JSON processor (parse board config files)
#   locales           — UTF-8 locale (Vim encoding needs this)

RUN apt-get update && apt-get install -y --no-install-recommends \
    # Editor and terminal
    vim-gtk3 \
    tmux \
    # Version control
    git \
    # Download tools
    curl \
    wget \
    # Search and navigation
    fzf \
    ripgrep \
    bat \
    universal-ctags \
    cscope \
    # Code formatting
    clang-format \
    # Serial console
    picocom \
    minicom \
    # Clipboard
    xclip \
    # Build tools
    build-essential \
    make \
    # Python
    python3 \
    python3-pip \
    python3-venv \
    # Cross-compilers
    gcc-aarch64-linux-gnu \
    binutils-aarch64-linux-gnu \
    gcc-arm-linux-gnueabihf \
    binutils-arm-linux-gnueabihf \
    # Embedded tools
    adb \
    # Utilities
    tree \
    jq \
    less \
    file \
    locales \
    sudo \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ── Locale — required for Vim UTF-8 encoding ─────────────────────────────────
RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8

# ── bat symlink (Ubuntu names it batcat) ─────────────────────────────────────
# WHY: bash_aliases uses 'bat' but Ubuntu installs it as 'batcat' to avoid
#      conflict with another package. This symlink makes 'bat' work.
RUN ln -sf /usr/bin/batcat /usr/local/bin/bat 2>/dev/null || true

# ── Create developer user ─────────────────────────────────────────────────────
# WHY NOT ROOT: Running as root inside a container is bad practice:
#   - Git refuses to work in repos owned by another user
#   - Vim plugin installs go to /root instead of the user's home
#   - Files created inside the container are owned by root on the host
#
# The user is named 'dev' with a configurable UID (default 1000).
# Pass --build-arg UID=$(id -u) to match your host user ID so that
# mounted volumes have correct permissions.

ARG USERNAME=dev
ARG UID=1000
ARG GID=1000

RUN groupadd --gid ${GID} ${USERNAME} && \
    useradd --uid ${UID} --gid ${GID} --shell /bin/bash \
            --create-home ${USERNAME} && \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# ── Switch to dev user ────────────────────────────────────────────────────────
USER ${USERNAME}
WORKDIR /home/${USERNAME}

# ── Copy dotforge into the container ─────────────────────────────────────────
# WHY COPY not VOLUME: The dotforge files are baked into the image so the
# container is self-contained. Your work files (source code, projects) are
# mounted as volumes at runtime — see docker.sh for the mount flags.
COPY --chown=${USERNAME}:${USERNAME} . /home/${USERNAME}/dotforge

# ── Install vim-plug ──────────────────────────────────────────────────────────
RUN curl -fLo /home/${USERNAME}/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

# ── Symlink dotforge configs into place ──────────────────────────────────────
# WHY SYMLINKS not COPY: Symlinks mean editing the file inside the container
# edits the dotforge source — useful when developing dotforge itself.
RUN mkdir -p /home/${USERNAME}/.vim && \
    # Vim config
    ln -sf /home/${USERNAME}/dotforge/vim/vimrc \
           /home/${USERNAME}/.vimrc && \
    ln -sf /home/${USERNAME}/dotforge/vim/plugin_config.vim \
           /home/${USERNAME}/.vim/plugin_config.vim && \
    ln -sf /home/${USERNAME}/dotforge/vim/key_mapping.vim \
           /home/${USERNAME}/.vim/key_mapping.vim && \
    # UltiSnips snippets
    mkdir -p /home/${USERNAME}/.vim/UltiSnips && \
    ln -sf /home/${USERNAME}/dotforge/vim/snippets/c.snippets \
           /home/${USERNAME}/.vim/UltiSnips/c.snippets && \
    ln -sf /home/${USERNAME}/dotforge/vim/snippets/python.snippets \
           /home/${USERNAME}/.vim/UltiSnips/python.snippets && \
    # tmux config
    ln -sf /home/${USERNAME}/dotforge/tmux/tmux.conf \
           /home/${USERNAME}/.tmux.conf && \
    # Persistent undo directory (needed by vimrc)
    mkdir -p /home/${USERNAME}/.vim/undodir

# ── Install Vim plugins (headless) ───────────────────────────────────────────
# WHY: Baking plugins into the image means the container starts instantly
#      without a plugin download step. The user gets a fully working Vim
#      from the first 'docker run'.
# NOTE: +qall exits Vim after PlugInstall completes.
RUN vim -E -s -u /home/${USERNAME}/.vimrc +PlugInstall +qall 2>/dev/null || true

# ── Append dotforge shell config to .bashrc ───────────────────────────────────
RUN echo "" >> /home/${USERNAME}/.bashrc && \
    echo "# dotforge — load all shell config" >> /home/${USERNAME}/.bashrc && \
    echo "source \$HOME/dotforge/shell/bashrc_append" >> /home/${USERNAME}/.bashrc

# ── Workspace mount point ─────────────────────────────────────────────────────
# WHY: /workspace is where you mount your project source code at runtime.
#   docker run -v /path/to/your/project:/workspace dotforge
# The directory is created here so it exists and is owned by the dev user.
RUN mkdir -p /home/${USERNAME}/workspace
WORKDIR /home/${USERNAME}/workspace

# ── Default command ───────────────────────────────────────────────────────────
# WHY TMUX: The container opens directly into a tmux session so you get
# the full dotforge workspace experience (splits, persistent sessions)
# rather than a bare shell.
CMD ["tmux", "new-session", "-s", "dotforge"]
