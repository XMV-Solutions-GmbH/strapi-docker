ARG NODE_VERSION
FROM node:${NODE_VERSION}

# vips-dev: required by sharp (image processing) on Alpine for ARM64.
# sharp ships pre-built ARM64 binaries via npm but falls back to compiling
# against libvips when none are found — vips-dev ensures that fallback works.
RUN apk add --no-cache \
    build-base \
    gcc \
    autoconf \
    automake \
    zlib-dev \
    libpng-dev \
    nasm \
    bash \
    vips-dev \
    python3 \
    make \
    g++

EXPOSE 1337
