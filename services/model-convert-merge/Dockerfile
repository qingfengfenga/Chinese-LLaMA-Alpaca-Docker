ARG UBUNTU_VERSION=22.04

FROM ubuntu:$UBUNTU_VERSION as build

RUN apt-get update && \
    apt-get install -y build-essential python3 python3-pip git curl python-is-python3

COPY requirements.txt requirements.txt

RUN pip install --upgrade pip setuptools wheel \
    && pip install -r requirements.txt

WORKDIR /app

COPY . .

RUN curl -o convert_llama_weights_to_hf.py https://raw.githubusercontent.com/huggingface/transformers/main/src/transformers/models/llama/convert_llama_weights_to_hf.py && \
    curl -o merge_llama_with_chinese_lora.py https://raw.githubusercontent.com/ymcui/Chinese-LLaMA-Alpaca/main/scripts/merge_llama_with_chinese_lora.py && \
    curl -o convert.py https://raw.githubusercontent.com/ggerganov/llama.cpp/master/convert.py

CMD python ${CLI_ARGS}