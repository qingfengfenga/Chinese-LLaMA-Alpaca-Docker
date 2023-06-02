ARG UBUNTU_VERSION=22.04

FROM ubuntu:$UBUNTU_VERSION as build

RUN apt-get update && \
    apt-get install -y build-essential python3 python3-pip git language-pack-zh-hans language-pack-zh-hant

RUN git clone https://github.com/ggerganov/llama.cpp --depth=1 /app

WORKDIR /app

RUN pip install --upgrade pip setuptools wheel \
    && pip install -r requirements.txt

RUN make

CMD ${CLI_ARGS}
