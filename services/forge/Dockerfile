FROM alpine/git:2.36.2 as download

COPY clone.sh /clone.sh

RUN . /clone.sh BLIP https://github.com/salesforce/BLIP.git
RUN . /clone.sh huggingface_guess https://github.com/lllyasviel/huggingface_guess.git

FROM pytorch/pytorch:2.3.0-cuda12.1-cudnn8-runtime

ENV DEBIAN_FRONTEND=noninteractive PIP_PREFER_BINARY=1

RUN apt-get update && \
    # we need those
    apt-get install -y git \
    # extensions needs those
    ffmpeg libglfw3-dev libgles2-mesa-dev pkg-config libcairo2 libcairo2-dev build-essential

WORKDIR /
RUN git clone https://github.com/lllyasviel/stable-diffusion-webui-forge.git && \
    cd stable-diffusion-webui-forge && \
    pip install -r requirements_versions.txt

ENV ROOT=/stable-diffusion-webui-forge

COPY --from=download /repositories/ ${ROOT}/repositories/

WORKDIR ${ROOT}
ENV NVIDIA_VISIBLE_DEVICES=all
ENV CLI_ARGS=""
EXPOSE 7860
CMD python -u webui.py --listen --port 7860 ${CLI_ARGS}