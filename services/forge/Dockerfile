# Multi-stage build for Stable Diffusion WebUI Forge
# Stage 1: Download and prepare required repositories
FROM alpine/git:2.36.2 as download

COPY clone.sh /clone.sh

RUN . /clone.sh stable-diffusion-webui-assets https://github.com/AUTOMATIC1111/stable-diffusion-webui-assets.git 6f7db241d2f8ba7457bac5ca9753331f0c266917
RUN . /clone.sh stable-diffusion-stability-ai https://github.com/Stability-AI/stablediffusion.git cf1d67a6fd5ea1aa600c4df58e5b47da45f6bdbf \
    && rm -rf assets data/**/*.png data/**/*.jpg data/**/*.gif
RUN . /clone.sh stable-diffusion-webui-assets https://github.com/AUTOMATIC1111/stable-diffusion-webui-assets 6f7db241d2f8ba7457bac5ca9753331f0c266917
RUN . /clone.sh BLIP https://github.com/salesforce/BLIP.git 48211a1594f1321b00f14c9f7a5b4813144b2fb9
RUN . /clone.sh k-diffusion https://github.com/crowsonkb/k-diffusion.git ab527a9a6d347f364e3d185ba6d714e22d80cb3c
RUN . /clone.sh clip-interrogator https://github.com/pharmapsychotic/clip-interrogator 2cf03aaf6e704197fd0dae7c7f96aa59cf1b11c9
RUN . /clone.sh generative-models https://github.com/Stability-AI/generative-models 45c443b316737a4ab6e40413d7794a7f5657c19f
RUN . /clone.sh huggingface_guess https://github.com/lllyasviel/huggingface_guess.git 70942022b6bcd17d941c1b4172804175758618e2

# Stage 2: Build the main application image with PyTorch and CUDA support
FROM pytorch/pytorch:2.3.1-cuda12.1-cudnn8-runtime

# Set environment variables for non-interactive installation and binary pip packages
ENV DEBIAN_FRONTEND=noninteractive PIP_PREFER_BINARY=1

# Install system dependencies required for Forge and its extensions
RUN apt-get update && \
    # Core dependencies
    apt-get install -y git \
    # Dependencies for extensions (video processing, graphics, rendering)
    ffmpeg libglfw3-dev libgles2-mesa-dev pkg-config libcairo2 libcairo2-dev build-essential

# Clone the Stable Diffusion WebUI Forge repository and install Python dependencies
WORKDIR /
RUN git clone https://github.com/lllyasviel/stable-diffusion-webui-forge.git && \
    cd stable-diffusion-webui-forge && \
    pip install -r requirements_versions.txt

# Update joblib to latest version for better parallel processing
RUN pip install -U joblib

# Set the root directory for the application
ENV ROOT=/stable-diffusion-webui-forge

# Copy downloaded repositories from the first stage
COPY --from=download /repositories/ ${ROOT}/repositories/

# Set working directory to the Forge root
WORKDIR ${ROOT}

# Make all NVIDIA GPUs visible to the container
ENV NVIDIA_VISIBLE_DEVICES=all

# Allow CLI arguments to be passed to the WebUI
ENV CLI_ARGS=""

# Expose the default WebUI port
EXPOSE 7860

# Start the WebUI with listening enabled on all interfaces
CMD python -u webui.py --listen --port 7860 ${CLI_ARGS}