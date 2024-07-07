# Build stage
FROM node:latest

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Update and install required packages
RUN apt-get update && apt-get install -y \
  git \
  lld \
  sudo \
  nsis \
  llvm \
  curl \
  wget \
  file \
  unzip \
  mingw-w64 \
  apt-utils \
  libxdo-dev \
  libssl-dev \
  libudev-dev \
  librsvg2-dev \
  ca-certificates \
  build-essential \
  libwebkit2gtk-4.1-dev \
  libayatana-appindicator3-dev \
  && rm -rf /var/lib/apt/lists/*

# Install Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

# Add Rust to PATH
ENV PATH="/root/.cargo/bin:${PATH}"

# Install Windows Rust target
RUN rustup target add x86_64-pc-windows-msvc

# Install xwin
RUN cargo install xwin

# Configure pnpm
RUN npm i -g pnpm
RUN pnpm config set store-dir .pnpm-store


# Set the WORKDIR to your project directory
WORKDIR /project

# Create temp project
RUN pnpm create tauri-app . -y --beta -f -t react-ts -m pnpm
RUN sed '4s/.*/  "identifier": "com.vigen.temp",/' ./src-tauri/tauri.conf.json > temp.json && mv temp.json ./src-tauri/tauri.conf.json

# Ensure .cargo folder exists
RUN mkdir -p .cargo

# Set up the Rust toolchain to use the installed tools
RUN echo '[target.x86_64-pc-windows-msvc]\nlinker = "lld"\nrustflags = ["-Lnative=/root/.xwin/crt/lib/x86_64", "-Lnative=/root/.xwin/sdk/lib/um/x86_64", "-Lnative=/root/.xwin/sdk/lib/ucrt/x86_64"]' > .cargo/config.toml

# Set up xwin (adjust paths as necessary)
RUN mkdir -p /root/.xwin
RUN xwin --accept-license splat --output /root/.xwin || xwin --accept-license splat --output /root/.xwin --disable-symlinks 

# Install node packages
RUN --mount=type=cache,id=pnpm,target=/project/.pnpm-store pnpm install

# Build your project
RUN --mount=type=cache,id=cargo,target=/root/.cargo/registry pnpm tauri build --target x86_64-pc-windows-msvc --verbose --ci

# Remove unsed files
# RUN ls -la && find . -mindepth 1 -maxdepth 1 ! -name '.xwin-cache' ! -name '.cargo' -exec rm -rf {} +
