FROM nvidia/cuda:12.8.1-devel-ubuntu22.04

LABEL authors="jaret"

# Set noninteractive to avoid timezone prompts
ENV DEBIAN_FRONTEND=noninteractive

# ref https://en.wikipedia.org/wiki/CUDA
ENV TORCH_CUDA_ARCH_LIST="8.0 8.6 8.9 9.0 10.0 12.0"

# Install dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
    git \
    curl \
    build-essential \
    cmake \
    wget \
    python3.10 \
    python3-pip \
    python3-dev \
    python3-setuptools \
    python3-wheel \
    python3-venv \
    ffmpeg \
    tmux \
    htop \
    nvtop \
    python3-opencv \
    openssh-client \
    openssh-server \
    openssl \
    rsync \
    unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install nodejs
WORKDIR /tmp
RUN curl -sL https://deb.nodesource.com/setup_23.x -o nodesource_setup.sh && \
    bash nodesource_setup.sh && \
    apt-get update && \
    apt-get install -y nodejs && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Set aliases for python and pip
RUN ln -s /usr/bin/python3 /usr/bin/python

# install pytorch before cache bust to avoid redownloading pytorch
RUN pip install --pre --no-cache-dir torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128

WORKDIR /app/ai-toolkit

ARG CACHEBUST=1234
ARG GIT_COMMIT=local
RUN echo "Cache bust: ${CACHEBUST}"
LABEL org.opencontainers.image.revision="${GIT_COMMIT}"

# Copy dependency manifests first to leverage Docker layer caching
COPY requirements.txt ./

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt && \
    pip install --pre --no-cache-dir torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128 --force && \
    pip install setuptools==69.5.1 --no-cache-dir

# Pre-seed UI dependencies before copying the full source
COPY ui/package.json ui/package-lock.json ./ui/

WORKDIR /app/ai-toolkit/ui
RUN npm install

WORKDIR /app/ai-toolkit

# Copy source directories (bind-mounted data dirs like config/output are skipped)
COPY assets ./assets
COPY extensions ./extensions
COPY extensions_built_in ./extensions_built_in
COPY jobs ./jobs
COPY notebooks ./notebooks
COPY scripts ./scripts
COPY testing ./testing
COPY toolkit ./toolkit

# Copy project entrypoints and supporting files
COPY run.py run_modal.py flux_train_ui.py info.py version.py ./

# Copy UI application sources after installing dependencies
COPY ui ./ui

WORKDIR /app/ai-toolkit/ui
RUN npm run build && \
    npm run update_db

# Expose port (assuming the application runs on port 8675)
EXPOSE 8675

WORKDIR /

COPY docker/start.sh /start.sh
RUN chmod +x /start.sh

CMD ["/start.sh"]
