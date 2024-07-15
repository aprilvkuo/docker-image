# syntax=docker/dockerfile:1

# Use native architecture of the build node for model downloading
FROM --platform=$BUILDPLATFORM python:3.10 AS model

# Install client for Huggingface Hub
RUN pip install huggingface_hub

# Set default model information
ARG MODEL_REPO="Qwen/Qwen2-7B-Instruct-GGUF"
ARG MODEL_REVISION="main"
ARG MODEL_FILE="qwen2-7b-instruct-q5_k_m.gguf"

# Download model from Huggingface Hub
RUN huggingface-cli download --revision ${MODEL_REVISION} --local-dir /model --local-dir-use-symlinks False ${MODEL_REPO} ${MODEL_FILE}
RUN mv /model/${MODEL_FILE} /model/model.gguf

# Build image for the target platform
FROM --platform=$TARGETPLATFORM nvidia/cuda:12.3.2-runtime-ubuntu22.04

# Set working directory
WORKDIR /usr/src/app

# Install runtime dependencies
RUN apt update
RUN apt install -y --no-install-recommends build-essential gcc python3 python3-pip
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install app dependencies
COPY requirements.txt requirements.txt
RUN pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cu123
RUN pip install -r requirements.txt

# Copy model file
COPY --from=model /model/model.gguf model.gguf

# Bundle app source
COPY . .

# Expose ports
EXPOSE 80

# Provide default environment variables
ENV MODEL_PATH="model.gguf"
ENV MODEL_N_GPU_LAYERS="-1"
ENV MODEL_USE_MMAP="True"
ENV MODEL_USE_MLOCK="False"
ENV MODEL_N_CTX="0"
ENV MODEL_N_BATCH="32"
ENV MODEL_OFFLOAD_KQV="True"
ENV MODEL_FLASH_ATTN="True"
ENV MODEL_VERBOSE="False"
ENV SAMPLING_TEMPERATURE="0.0"
ENV SAMPLING_TOP_P="0.5"
ENV SAMPLING_SEED="0"
ENV SAMPLING_MAX_TOKENS="4096"

# Specify entrypoint and default parameters
ENTRYPOINT ["waitress-serve"]
CMD ["--threads", "1", "--host", "0.0.0.0", "--port", "80", "translator.server:app"]

