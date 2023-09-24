# Ultralytics YOLO 🚀, AGPL-3.0 license
# Builds ultralytics/ultralytics:latest image on DockerHub https://hub.docker.com/r/ultralytics/ultralytics
# Image is CUDA-optimized for YOLOv8 single/multi-GPU training and inference

# Start FROM PyTorch image https://hub.docker.com/r/pytorch/pytorch or nvcr.io/nvidia/pytorch:23.03-py3
#FROM pytorch/pytorch:2.0.1-cuda11.7-cudnn8-runtime
FROM nvidia/cuda:11.1.1-cudnn8-devel-ubuntu18.04

ENV DEBIAN_FRONTEND noninteractive

RUN apt update -y && apt upgrade -y && \
    apt-get install -y wget build-essential checkinstall  libreadline-gplv2-dev  libncursesw5-dev  libssl-dev  libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev zlib1g-dev lzma l\
iblzma-dev libbz2-dev && \
    cd /usr/src && \
    wget https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tgz && \
    tar xzf Python-3.8.10.tgz && \
    cd Python-3.8.10 && \
    ./configure --enable-optimizations && \
    make install

RUN apt-get install ffmpeg libsm6 libxext6  -y
#RUN apt-get install git gcc build-essential qtbase5-dev -y

# Downloads to user config dir
ADD https://ultralytics.com/assets/Arial.ttf https://ultralytics.com/assets/Arial.Unicode.ttf /root/.config/Ultralytics/

# Install linux packages
# g++ required to build 'tflite_support' and 'lap' packages, libusb-1.0-0 required for 'tflite_support' package
RUN apt update \
    && apt install --no-install-recommends -y gcc git zip curl htop libgl1-mesa-glx libglib2.0-0 libpython3-dev gnupg g++ libusb-1.0-0
# RUN alias python=python3

#WORKDIR /usr/src/ultralytics
RUN groupadd -r algorithm && useradd -m --no-log-init -r -g algorithm algorithm
RUN mkdir -p /opt/algorithm /input /output \
    && chown algorithm:algorithm /opt/algorithm /input /output

USER algorithm
WORKDIR /opt/algorithm

ENV PATH="/home/algorithm/.local/bin:${PATH}"

RUN wget https://bootstrap.pypa.io/pip/3.6/get-pip.py && \
        python3 get-pip.py --user && \
        rm get-pip.py

RUN pip install torch==1.10.1+cu111 torchvision==0.11.2+cu111 torchaudio==0.10.1 -f https://download.pytorch.org/whl/cu111/torch_stable.html

RUN pip install --no-cache nvidia-tensorrt --index-url https://pypi.ngc.nvidia.com

# Copy contents
# COPY . /usr/src/ultralytics  # git permission issues inside container
RUN git clone https://github.com/ultralytics/ultralytics -b main /home/algorithm/ultralytics
ADD https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8n.pt /home/algorithm/ultralytics/

# Install pip packages
RUN python3 -m pip install --upgrade pip wheel
RUN pip install --no-cache -e /home/algorithm/ultralytics thop albumentations comet pycocotools pytest-cov

RUN pip install protobuf==3.20.*

# Run exports to AutoInstall packages
#RUN yolo export model=tmp/yolov8n.pt format=edgetpu imgsz=32
#RUN yolo export model=tmp/yolov8n.pt format=ncnn imgsz=32
# Requires <= Python 3.10, bug with paddlepaddle==2.5.0 https://github.com/PaddlePaddle/X2Paddle/issues/991
RUN pip install --no-cache paddlepaddle==2.4.2 x2paddle
# Fix error: `np.bool` was a deprecated alias for the builtin `bool` segmentation error in Tests
RUN pip install --no-cache numpy==1.23.5
# Remove exported models
RUN rm -rf tmp

# Set environment variables
ENV OMP_NUM_THREADS=1
# Avoid DDP error "MKL_THREADING_LAYER=INTEL is incompatible with libgomp.so.1 library" https://github.com/pytorch/pytorch/issues/37377
ENV MKL_THREADING_LAYER=GNU

RUN python3 -V
RUN python3 -c "import torch;print(torch.cuda.is_available())"

RUN python3 -m pip install --user pyqt5

COPY --chown=algorithm:algorithm requirements.txt /opt/algorithm/
RUN python3 -m pip install --user -rrequirements.txt

COPY --chown=algorithm:algorithm process.py /opt/algorithm/
COPY --chown=algorithm:algorithm weights/best_all.pt /opt/algorithm/weights/best.pt

ENTRYPOINT python3 -m process $0 $@
