ARG asciidoctor_version=1.76.0
ARG pandoc_version=3.2.1.0
ARG lua_version=5.4

# https://github.com/asciidoctor/docker-asciidoctor
# 1.79.0
# FROM asciidoctor/docker-asciidoctor:1.76.0 as asciidoctor
FROM asciidoctor/docker-asciidoctor:1.79.0 as asciidoctor

# https://github.com/pandoc/dockerfiles
# 3.5.0.0
FROM pandoc/core:3.2.1.0 as pandoc

# https://github.com/astefanutti/decktape
FROM astefanutti/decktape:3.12.0 as decktape

FROM asciidoctor

LABEL maintainer='Loïc Péron <loic.peron@gmail.com>'

# Decktape #####################################################################

ENV TERM xterm-color

# Chromium, CA certificates, fonts
# https://git.alpinelinux.org/cgit/aports/log/community/chromium
RUN apk update && apk upgrade && \
    apk add --no-cache \
    ca-certificates \
    libstdc++ \
    chromium \
    font-noto-emoji \
    freetype \
    harfbuzz \
    nss \
    ttf-freefont \
    wqy-zenhei && \
    # /etc/fonts/conf.d/44-wqy-zenhei.conf overrides 'monospace' matching FreeMono.ttf in /etc/fonts/conf.d/69-unifont.conf
    mv /etc/fonts/conf.d/44-wqy-zenhei.conf /etc/fonts/conf.d/74-wqy-zenhei.conf && \
    rm -rf /var/cache/apk/*

# Node.js
COPY --from=decktape /usr/local/bin/node /usr/local/bin/

# DeckTape
COPY --from=decktape /decktape /decktape

# Pandoc Minimal ###############################################################

# Set and create user data directory
ENV XDG_DATA_HOME=/usr/local/share
WORKDIR $XDG_DATA_HOME/pandoc

# Set default working directory
WORKDIR /data

COPY --from=pandoc \
  /usr/local/bin/pandoc \
  /usr/local/bin/

# Add pandoc symlinks and install runtime dependencies
RUN ln -s /usr/local/bin/pandoc /usr/local/bin/pandoc-lua && \
    ln -s /usr/local/bin/pandoc /usr/local/bin/pandoc-server && \
    apk --no-cache add \
        gmp \
        libffi \
        lua$lua_version

# Pandoc Core ##################################################################
COPY --from=pandoc \
  /usr/local/bin/pandoc-crossref \
  /usr/local/bin/

# Additional packages frequently used during conversions
# NOTE: `libsrvg`, pandoc uses `rsvg-convert` for working with svg images.
# FIXME: Alpine 3.17 and later ships the binary in the rsvg-convert package.
RUN apk --no-cache add librsvg; \
    apk --no-cache add rsvg-convert || true

# https://github.com/natacom/asciidoc_devcontainer

# This Dockerfile adds a non-root 'vscode' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

COPY --chmod=0755 sh/* /usr/local/bin
