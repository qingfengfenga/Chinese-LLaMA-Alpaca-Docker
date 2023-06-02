# 文档

## 1、初始化目录

复制一份配置文件`.env-template`到`.env`

原始目录结构

```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
└── tokenizer.model

1 directory, 5 files
```

运行初始化容器，创建对应文件夹
```
$ docker-compose --profile init-dir up
[+] Building 0.0s (0/0)
[+] Running 1/0
 ✔ Container init-dir  Created                                                                                              0.0s 
Attaching to init-dir
init-dir exited with code 0
```

完成后的目录结构如下
```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
├── 7B-HF
├── Chinese-Alpaca-Plus
│   └── 7B
├── LoRA
└── tokenizer.model

5 directories, 5 file
```

## 2、把原始模型权重转换为HF格式
> 基于WSL2的Docker高负载下会出现闪退等问题，可以重启WSL和Docker后删除转换容器，重新运行docker-compose命令。

```
$ docker-compose --profile models-convert up
[+] Building 0.0s (0/0)
[+] Running 1/0
 ✔ Container models-convert  Created                                                                                                                                                            0.0s 
Attaching to models-convert
models-convert  | Fetching all parameters from the checkpoint at /app/models/llama/7B/7B.
models-convert  | Loading the checkpoint in a Llama model.
Loading checkpoint shards: 100%|██████████| 33/33 [01:36<00:00,  2.92s/it]
models-convert  | Saving in the Transformers format.
models-convert  | Saving a LlamaTokenizerFast to /app/models/llama/7B/7B-HF.
models-convert exited with code 0
```

此时的文件结构

```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
├── 7B-HF
│   ├── config.json
│   ├── generation_config.json
│   ├── pytorch_model-00001-of-00002.bin
│   ├── pytorch_model-00002-of-00002.bin
│   ├── pytorch_model.bin.index.json
│   ├── special_tokens_map.json
│   ├── tokenizer.json
│   ├── tokenizer.model
│   └── tokenizer_config.json
├── Chinese-Alpaca-Plus
│   └── 7B
├── LoRA
└── tokenizer.model

5 directories, 14 files
```

## 3、放入对应的LoRA文件

把对应的LoRA放入LoRA文件夹，这是演示为Chinese-Alpaca-Plus，需要两个LoRA，并且有顺序要求

放入LoRA后的目录结构
```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
├── 7B-HF
│   ├── config.json
│   ├── generation_config.json
│   ├── pytorch_model-00001-of-00002.bin
│   ├── pytorch_model-00002-of-00002.bin
│   ├── pytorch_model.bin.index.json
│   ├── special_tokens_map.json
│   ├── tokenizer.json
│   ├── tokenizer.model
│   └── tokenizer_config.json
├── Chinese-Alpaca-Plus
│   └── 7B
├── LoRA
│   ├── chinese-alpaca-plus-lora-7b
│   │   ├── README.md
│   │   ├── adapter_config.json
│   │   ├── adapter_model.bin
│   │   ├── special_tokens_map.json
│   │   ├── tokenizer.model
│   │   └── tokenizer_config.json
│   └── chinese-llama-plus-lora-7b
│       ├── README.md
│       ├── adapter_config.json
│       ├── adapter_model.bin
│       ├── special_tokens_map.json
│       ├── tokenizer.model
│       └── tokenizer_config.json
└── tokenizer.model

7 directories, 26 files
```

## 4、合并模型权重
运行合并脚本
```
$ docker-compose --profile models-merge up
[+] Building 0.0s (0/0)
[+] Running 1/0
 ✔ Container models-merge  Recreated                                                                                                                                                            0.1s 
Attaching to models-merge
models-merge  | Base model: /app/models/llama/7B/7B-HF
models-merge  | LoRA model(s) ['/app/models/llama/7B/LoRA/chinese-llama-plus-lora-7b', '/app/models/llama/7B/LoRA/chinese-alpaca-plus-lora-7b']:
Loading checkpoint shards: 100%|██████████| 2/2 [01:21<00:00, 40.89s/it]
models-merge  | Peft version: 0.4.0.dev0
models-merge  | Loading LoRA for 7B model
models-merge  | Loading LoRA /app/models/llama/7B/LoRA/chinese-llama-plus-lora-7b...
models-merge  | base_model vocab size: 32000
models-merge  | tokenizer vocab size: 49953
models-merge  | Extended vocabulary size to 49953
models-merge  | Loading LoRA weights
models-merge  | Merging with merge_and_unload...
models-merge  | Loading LoRA /app/models/llama/7B/LoRA/chinese-alpaca-plus-lora-7b...
models-merge  | base_model vocab size: 49953
models-merge  | tokenizer vocab size: 49954
models-merge  | Extended vocabulary size to 49954
models-merge  | Loading LoRA weights
models-merge  | Merging with merge_and_unload...
models-merge  | Saving to pth format...
models-merge  | Saving shard 1 of 1 into /app/models/llama/7B/Chinese-Alpaca-Plus/7B/consolidated.00.pth
models-merge exited with code 0
```
合并LoRA后的目录结构
```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
├── 7B-HF
│   ├── config.json
│   ├── generation_config.json
│   ├── pytorch_model-00001-of-00002.bin
│   ├── pytorch_model-00002-of-00002.bin
│   ├── pytorch_model.bin.index.json
│   ├── special_tokens_map.json
│   ├── tokenizer.json
│   ├── tokenizer.model
│   └── tokenizer_config.json
├── Chinese-Alpaca-Plus
│   └── 7B
│       ├── consolidated.00.pth
│       ├── params.json
│       ├── special_tokens_map.json
│       ├── tokenizer.model
│       └── tokenizer_config.json
├── LoRA
│   ├── chinese-alpaca-plus-lora-7b
│   │   ├── README.md
│   │   ├── adapter_config.json
│   │   ├── adapter_model.bin
│   │   ├── special_tokens_map.json
│   │   ├── tokenizer.model
│   │   └── tokenizer_config.json
│   └── chinese-llama-plus-lora-7b
│       ├── README.md
│       ├── adapter_config.json
│       ├── adapter_model.bin
│       ├── special_tokens_map.json
│       ├── tokenizer.model
│       └── tokenizer_config.json
└── tokenizer.model

7 directories, 31 files
```

*合并完成，准备量化*

## 5、初始化量化目录结构

```
$ docker-compose --profile init-dir-quantize up
[+] Building 0.0s (0/0)
[+] Running 1/1
 ✔ Container init-dir-quantize  Recreated                                                                                                                                                       0.1s 
Attaching to init-dir-quantize
init-dir-quantize exited with code 0
```

完成后的目录结构
```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
├── 7B-HF
│   ├── config.json
│   ├── generation_config.json
│   ├── pytorch_model-00001-of-00002.bin
│   ├── pytorch_model-00002-of-00002.bin
│   ├── pytorch_model.bin.index.json
│   ├── special_tokens_map.json
│   ├── tokenizer.json
│   ├── tokenizer.model
│   └── tokenizer_config.json
├── Chinese-Alpaca-Plus
│   ├── 7B
│   │   ├── consolidated.00.pth
│   │   ├── params.json
│   │   ├── special_tokens_map.json
│   │   └── tokenizer_config.json
│   └── tokenizer.model
├── LoRA
│   ├── chinese-alpaca-plus-lora-7b
│   │   ├── README.md
│   │   ├── adapter_config.json
│   │   ├── adapter_model.bin
│   │   ├── special_tokens_map.json
│   │   ├── tokenizer.model
│   │   └── tokenizer_config.json
│   └── chinese-llama-plus-lora-7b
│       ├── README.md
│       ├── adapter_config.json
│       ├── adapter_model.bin
│       ├── special_tokens_map.json
│       ├── tokenizer.model
│       └── tokenizer_config.json
└── tokenizer.model

7 directories, 31 files
```

## 6、把模型转换为GGML格式

将上述模型权重转换为 ggml 的 FP16 格式，并生成一个带有 path 的文件。.pthzh-models/7B/ggml-model-f16.bin

```
$ docker-compose --profile llama-cpp-models-convert up
[+] Building 0.0s (0/0)
[+] Running 1/0
 ✔ Container llama-cpp-models-convert  Recreated                                                                                                                                                0.1s 
Attaching to llama-cpp-models-convert
llama-cpp-models-convert  | Loading model file /app/models/llama/7B/Chinese-Alpaca-Plus/7B/consolidated.00.pth
llama-cpp-models-convert  | Loading vocab file /app/models/llama/7B/Chinese-Alpaca-Plus/tokenizer.model
llama-cpp-models-convert  | Writing vocab...
llama-cpp-models-convert  | [  1/291] Writing tensor tok_embeddings.weight                  | size  49954 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [  2/291] Writing tensor norm.weight                            | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [  3/291] Writing tensor output.weight                          | size  49954 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [  4/291] Writing tensor layers.0.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [  5/291] Writing tensor layers.0.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [  6/291] Writing tensor layers.0.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [  7/291] Writing tensor layers.0.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [  8/291] Writing tensor layers.0.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [  9/291] Writing tensor layers.0.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 10/291] Writing tensor layers.0.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 11/291] Writing tensor layers.0.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 12/291] Writing tensor layers.0.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 13/291] Writing tensor layers.1.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 14/291] Writing tensor layers.1.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 15/291] Writing tensor layers.1.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 16/291] Writing tensor layers.1.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 17/291] Writing tensor layers.1.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 18/291] Writing tensor layers.1.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 19/291] Writing tensor layers.1.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 20/291] Writing tensor layers.1.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 21/291] Writing tensor layers.1.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 22/291] Writing tensor layers.2.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 23/291] Writing tensor layers.2.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 24/291] Writing tensor layers.2.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 25/291] Writing tensor layers.2.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 26/291] Writing tensor layers.2.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 27/291] Writing tensor layers.2.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 28/291] Writing tensor layers.2.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 29/291] Writing tensor layers.2.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 30/291] Writing tensor layers.2.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 31/291] Writing tensor layers.3.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 32/291] Writing tensor layers.3.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 33/291] Writing tensor layers.3.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 34/291] Writing tensor layers.3.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 35/291] Writing tensor layers.3.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 36/291] Writing tensor layers.3.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 37/291] Writing tensor layers.3.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 38/291] Writing tensor layers.3.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 39/291] Writing tensor layers.3.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 40/291] Writing tensor layers.4.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 41/291] Writing tensor layers.4.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 42/291] Writing tensor layers.4.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 43/291] Writing tensor layers.4.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 44/291] Writing tensor layers.4.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 45/291] Writing tensor layers.4.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 46/291] Writing tensor layers.4.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 47/291] Writing tensor layers.4.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 48/291] Writing tensor layers.4.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 49/291] Writing tensor layers.5.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 50/291] Writing tensor layers.5.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 51/291] Writing tensor layers.5.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 52/291] Writing tensor layers.5.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 53/291] Writing tensor layers.5.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 54/291] Writing tensor layers.5.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 55/291] Writing tensor layers.5.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 56/291] Writing tensor layers.5.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 57/291] Writing tensor layers.5.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 58/291] Writing tensor layers.6.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 59/291] Writing tensor layers.6.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 60/291] Writing tensor layers.6.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 61/291] Writing tensor layers.6.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 62/291] Writing tensor layers.6.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 63/291] Writing tensor layers.6.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 64/291] Writing tensor layers.6.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 65/291] Writing tensor layers.6.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 66/291] Writing tensor layers.6.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 67/291] Writing tensor layers.7.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 68/291] Writing tensor layers.7.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 69/291] Writing tensor layers.7.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 70/291] Writing tensor layers.7.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 71/291] Writing tensor layers.7.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 72/291] Writing tensor layers.7.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 73/291] Writing tensor layers.7.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 74/291] Writing tensor layers.7.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 75/291] Writing tensor layers.7.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 76/291] Writing tensor layers.8.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 77/291] Writing tensor layers.8.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 78/291] Writing tensor layers.8.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 79/291] Writing tensor layers.8.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 80/291] Writing tensor layers.8.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 81/291] Writing tensor layers.8.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 82/291] Writing tensor layers.8.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 83/291] Writing tensor layers.8.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 84/291] Writing tensor layers.8.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 85/291] Writing tensor layers.9.attention.wq.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 86/291] Writing tensor layers.9.attention.wk.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 87/291] Writing tensor layers.9.attention.wv.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 88/291] Writing tensor layers.9.attention.wo.weight           | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 89/291] Writing tensor layers.9.attention_norm.weight         | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 90/291] Writing tensor layers.9.feed_forward.w1.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 91/291] Writing tensor layers.9.feed_forward.w2.weight        | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 92/291] Writing tensor layers.9.feed_forward.w3.weight        | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 93/291] Writing tensor layers.9.ffn_norm.weight               | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 94/291] Writing tensor layers.10.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 95/291] Writing tensor layers.10.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 96/291] Writing tensor layers.10.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 97/291] Writing tensor layers.10.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [ 98/291] Writing tensor layers.10.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [ 99/291] Writing tensor layers.10.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [100/291] Writing tensor layers.10.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [101/291] Writing tensor layers.10.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [102/291] Writing tensor layers.10.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [103/291] Writing tensor layers.11.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [104/291] Writing tensor layers.11.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [105/291] Writing tensor layers.11.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [106/291] Writing tensor layers.11.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [107/291] Writing tensor layers.11.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [108/291] Writing tensor layers.11.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [109/291] Writing tensor layers.11.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [110/291] Writing tensor layers.11.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [111/291] Writing tensor layers.11.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [112/291] Writing tensor layers.12.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [113/291] Writing tensor layers.12.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [114/291] Writing tensor layers.12.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [115/291] Writing tensor layers.12.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [116/291] Writing tensor layers.12.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [117/291] Writing tensor layers.12.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [118/291] Writing tensor layers.12.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [119/291] Writing tensor layers.12.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [120/291] Writing tensor layers.12.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [121/291] Writing tensor layers.13.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [122/291] Writing tensor layers.13.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [123/291] Writing tensor layers.13.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [124/291] Writing tensor layers.13.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [125/291] Writing tensor layers.13.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [126/291] Writing tensor layers.13.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [127/291] Writing tensor layers.13.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [128/291] Writing tensor layers.13.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [129/291] Writing tensor layers.13.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [130/291] Writing tensor layers.14.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [131/291] Writing tensor layers.14.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [132/291] Writing tensor layers.14.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [133/291] Writing tensor layers.14.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [134/291] Writing tensor layers.14.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [135/291] Writing tensor layers.14.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [136/291] Writing tensor layers.14.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [137/291] Writing tensor layers.14.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [138/291] Writing tensor layers.14.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [139/291] Writing tensor layers.15.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [140/291] Writing tensor layers.15.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [141/291] Writing tensor layers.15.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [142/291] Writing tensor layers.15.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [143/291] Writing tensor layers.15.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [144/291] Writing tensor layers.15.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [145/291] Writing tensor layers.15.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [146/291] Writing tensor layers.15.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [147/291] Writing tensor layers.15.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [148/291] Writing tensor layers.16.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [149/291] Writing tensor layers.16.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [150/291] Writing tensor layers.16.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [151/291] Writing tensor layers.16.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [152/291] Writing tensor layers.16.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [153/291] Writing tensor layers.16.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [154/291] Writing tensor layers.16.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [155/291] Writing tensor layers.16.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [156/291] Writing tensor layers.16.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [157/291] Writing tensor layers.17.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [158/291] Writing tensor layers.17.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [159/291] Writing tensor layers.17.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [160/291] Writing tensor layers.17.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [161/291] Writing tensor layers.17.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [162/291] Writing tensor layers.17.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [163/291] Writing tensor layers.17.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [164/291] Writing tensor layers.17.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [165/291] Writing tensor layers.17.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [166/291] Writing tensor layers.18.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [167/291] Writing tensor layers.18.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [168/291] Writing tensor layers.18.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [169/291] Writing tensor layers.18.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [170/291] Writing tensor layers.18.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [171/291] Writing tensor layers.18.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [172/291] Writing tensor layers.18.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [173/291] Writing tensor layers.18.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [174/291] Writing tensor layers.18.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [175/291] Writing tensor layers.19.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [176/291] Writing tensor layers.19.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [177/291] Writing tensor layers.19.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [178/291] Writing tensor layers.19.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [179/291] Writing tensor layers.19.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [180/291] Writing tensor layers.19.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [181/291] Writing tensor layers.19.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [182/291] Writing tensor layers.19.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [183/291] Writing tensor layers.19.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [184/291] Writing tensor layers.20.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [185/291] Writing tensor layers.20.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [186/291] Writing tensor layers.20.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [187/291] Writing tensor layers.20.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [188/291] Writing tensor layers.20.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [189/291] Writing tensor layers.20.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [190/291] Writing tensor layers.20.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [191/291] Writing tensor layers.20.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [192/291] Writing tensor layers.20.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [193/291] Writing tensor layers.21.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [194/291] Writing tensor layers.21.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [195/291] Writing tensor layers.21.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [196/291] Writing tensor layers.21.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [197/291] Writing tensor layers.21.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [198/291] Writing tensor layers.21.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [199/291] Writing tensor layers.21.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [200/291] Writing tensor layers.21.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [201/291] Writing tensor layers.21.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [202/291] Writing tensor layers.22.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [203/291] Writing tensor layers.22.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [204/291] Writing tensor layers.22.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [205/291] Writing tensor layers.22.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [206/291] Writing tensor layers.22.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [207/291] Writing tensor layers.22.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [208/291] Writing tensor layers.22.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [209/291] Writing tensor layers.22.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [210/291] Writing tensor layers.22.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [211/291] Writing tensor layers.23.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [212/291] Writing tensor layers.23.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [213/291] Writing tensor layers.23.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [214/291] Writing tensor layers.23.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [215/291] Writing tensor layers.23.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [216/291] Writing tensor layers.23.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [217/291] Writing tensor layers.23.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [218/291] Writing tensor layers.23.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [219/291] Writing tensor layers.23.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [220/291] Writing tensor layers.24.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [221/291] Writing tensor layers.24.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [222/291] Writing tensor layers.24.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [223/291] Writing tensor layers.24.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [224/291] Writing tensor layers.24.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [225/291] Writing tensor layers.24.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [226/291] Writing tensor layers.24.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [227/291] Writing tensor layers.24.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [228/291] Writing tensor layers.24.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [229/291] Writing tensor layers.25.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [230/291] Writing tensor layers.25.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [231/291] Writing tensor layers.25.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [232/291] Writing tensor layers.25.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [233/291] Writing tensor layers.25.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [234/291] Writing tensor layers.25.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [235/291] Writing tensor layers.25.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [236/291] Writing tensor layers.25.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [237/291] Writing tensor layers.25.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [238/291] Writing tensor layers.26.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [239/291] Writing tensor layers.26.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [240/291] Writing tensor layers.26.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [241/291] Writing tensor layers.26.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [242/291] Writing tensor layers.26.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [243/291] Writing tensor layers.26.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [244/291] Writing tensor layers.26.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [245/291] Writing tensor layers.26.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [246/291] Writing tensor layers.26.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [247/291] Writing tensor layers.27.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [248/291] Writing tensor layers.27.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [249/291] Writing tensor layers.27.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [250/291] Writing tensor layers.27.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [251/291] Writing tensor layers.27.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [252/291] Writing tensor layers.27.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [253/291] Writing tensor layers.27.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [254/291] Writing tensor layers.27.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [255/291] Writing tensor layers.27.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [256/291] Writing tensor layers.28.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [257/291] Writing tensor layers.28.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [258/291] Writing tensor layers.28.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [259/291] Writing tensor layers.28.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [260/291] Writing tensor layers.28.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [261/291] Writing tensor layers.28.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [262/291] Writing tensor layers.28.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [263/291] Writing tensor layers.28.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [271/291] Writing tensor layers.29.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [272/291] Writing tensor layers.29.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [273/291] Writing tensor layers.29.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [274/291] Writing tensor layers.30.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [275/291] Writing tensor layers.30.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [276/291] Writing tensor layers.30.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [277/291] Writing tensor layers.30.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [278/291] Writing tensor layers.30.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [279/291] Writing tensor layers.30.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [280/291] Writing tensor layers.30.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [281/291] Writing tensor layers.30.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [282/291] Writing tensor layers.30.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [283/291] Writing tensor layers.31.attention.wq.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [284/291] Writing tensor layers.31.attention.wk.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [285/291] Writing tensor layers.31.attention.wv.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [286/291] Writing tensor layers.31.attention.wo.weight          | size   4096 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [287/291] Writing tensor layers.31.attention_norm.weight        | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | [288/291] Writing tensor layers.31.feed_forward.w1.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [289/291] Writing tensor layers.31.feed_forward.w2.weight       | size   4096 x  11008  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [290/291] Writing tensor layers.31.feed_forward.w3.weight       | size  11008 x   4096  | type UnquantizedDataType(name='F16')
llama-cpp-models-convert  | [291/291] Writing tensor layers.31.ffn_norm.weight              | size   4096           | type UnquantizedDataType(name='F32')
llama-cpp-models-convert  | Wrote /app/models/llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-f16.bin
llama-cpp-models-convert exited with code 0
```

转换后的目录格式
```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
├── 7B-HF
│   ├── config.json
│   ├── generation_config.json
│   ├── pytorch_model-00001-of-00002.bin
│   ├── pytorch_model-00002-of-00002.bin
│   ├── pytorch_model.bin.index.json
│   ├── special_tokens_map.json
│   ├── tokenizer.json
│   ├── tokenizer.model
│   └── tokenizer_config.json
├── Chinese-Alpaca-Plus
│   ├── 7B
│   │   ├── consolidated.00.pth
│   │   ├── ggml-model-f16.bin
│   │   ├── params.json
│   │   ├── special_tokens_map.json
│   │   └── tokenizer_config.json
│   └── tokenizer.model
├── LoRA
│   ├── chinese-alpaca-plus-lora-7b
│   │   ├── README.md
│   │   ├── adapter_config.json
│   │   ├── adapter_model.bin
│   │   ├── special_tokens_map.json
│   │   ├── tokenizer.model
│   │   └── tokenizer_config.json
│   └── chinese-llama-plus-lora-7b
│       ├── README.md
│       ├── adapter_config.json
│       ├── adapter_model.bin
│       ├── special_tokens_map.json
│       ├── tokenizer.model
│       └── tokenizer_config.json
└── tokenizer.model

7 directories, 32 files
```


## 7、量化模型
> 这里建议重新build镜像后运行，已经编译好的镜像内的二进制文件在不同环境或不同架构可能会有兼容性问题。

```
$ docker-compose --profile llama-cpp-quantize up --build
[+] Building 40.2s (11/11) FINISHED
 => [llama-cpp-quantize internal] load build definition from Dockerfile                                                                                                                         0.0s
 => => transferring dockerfile: 450B                                                                                                                                                            0.0s 
 => [llama-cpp-quantize internal] load .dockerignore                                                                                                                                            0.0s 
 => => transferring context: 2B                                                                                                                                                                 0.0s 
 => [llama-cpp-quantize internal] load metadata for docker.io/library/ubuntu:22.04                                                                                                              7.0s 
 => [llama-cpp-quantize auth] library/ubuntu:pull token for registry-1.docker.io                                                                                                                0.0s
 => [llama-cpp-quantize 1/6] FROM docker.io/library/ubuntu:22.04@sha256:ac58ff7fe25edc58bdf0067ca99df00014dbd032e2246d30a722fa348fd799a5                                                        0.0s
 => CACHED [llama-cpp-quantize 2/6] RUN apt-get update &&     apt-get install -y build-essential python3 python3-pip git language-pack-zh-hans language-pack-zh-hant                            0.0s 
 => CACHED [llama-cpp-quantize 3/6] RUN git clone https://github.com/ggerganov/llama.cpp --depth=1 /app                                                                                         0.0s 
 => CACHED [llama-cpp-quantize 4/6] WORKDIR /app                                                                                                                                                0.0s 
 => [llama-cpp-quantize 5/6] RUN pip install --upgrade pip setuptools wheel     && pip install -r requirements.txt                                                                             15.0s 
 => [llama-cpp-quantize 6/6] RUN make                                                                                                                                                          15.8s
 => [llama-cpp-quantize] exporting to image                                                                                                                                                     2.2s
 => => exporting layers                                                                                                                                                                         2.2s
 => => writing image sha256:218ff2ae8356bd59fa8d1d84ac001e40c646fdf26cb10e6527596c28047166c5                                                                                                    0.0s
 => => naming to docker.io/library/llama.cpp:full-zh                                                                                                                                            0.0s
[+] Running 1/0
 ✔ Container llama-cpp-quantize  Created                                                                                                                                                        0.1s 
Attaching to llama-cpp-quantize
llama-cpp-quantize  | main: build = 1 (ffb06a3)
llama-cpp-quantize  | main: quantizing '/app/models/llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-f16.bin' to '/app/models/llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-q4_0.bin' as q4_0
llama-cpp-quantize  | llama.cpp: loading model from /app/models/llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-f16.bin
llama-cpp-quantize  | llama.cpp: saving model to /app/models/llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-q4_0.bin
llama-cpp-quantize  | [   1/ 291]                tok_embeddings.weight -     4096 x 49954, type =    f16, quantizing .. size =   390.27 MB ->   109.76 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [   2/ 291]                          norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [   3/ 291]                        output.weight -     4096 x 49954, type =    f16, quantizing .. size =   390.27 MB ->   109.76 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [   4/ 291]         layers.0.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.014 0.022 0.035 0.052 0.075 0.099 0.119 0.128 0.119 0.098 0.075 0.053 0.035 0.022 0.019
llama-cpp-quantize  | [   5/ 291]         layers.0.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.013 0.022 0.035 0.052 0.074 0.098 0.120 0.130 0.121 0.099 0.074 0.052 0.035 0.022 0.018
llama-cpp-quantize  | [   6/ 291]         layers.0.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.024 0.037 0.055 0.075 0.096 0.114 0.124 0.114 0.096 0.075 0.055 0.037 0.024 0.020
llama-cpp-quantize  | [   7/ 291]         layers.0.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.013 0.021 0.034 0.051 0.074 0.099 0.122 0.132 0.122 0.099 0.074 0.051 0.034 0.021 0.018
llama-cpp-quantize  | [   8/ 291]       layers.0.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [   9/ 291]      layers.0.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  10/ 291]      layers.0.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  11/ 291]      layers.0.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  12/ 291]             layers.0.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  13/ 291]         layers.1.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.076 0.097 0.112 0.118 0.112 0.097 0.076 0.056 0.038 0.025 0.020
llama-cpp-quantize  | [  14/ 291]         layers.1.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.077 0.097 0.113 0.119 0.113 0.097 0.076 0.056 0.038 0.025 0.020
llama-cpp-quantize  | [  15/ 291]         layers.1.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.024 0.037 0.055 0.076 0.097 0.115 0.122 0.115 0.097 0.076 0.055 0.037 0.024 0.020
llama-cpp-quantize  | [  16/ 291]         layers.1.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.013 0.021 0.034 0.051 0.073 0.099 0.123 0.134 0.123 0.099 0.073 0.051 0.034 0.021 0.018
llama-cpp-quantize  | [  17/ 291]       layers.1.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  18/ 291]      layers.1.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  19/ 291]      layers.1.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  20/ 291]      layers.1.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  21/ 291]             layers.1.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  22/ 291]         layers.2.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.118 0.112 0.097 0.077 0.056 0.038 0.025 0.021
llama-cpp-quantize  | [  23/ 291]         layers.2.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.076 0.097 0.113 0.120 0.113 0.097 0.076 0.056 0.038 0.025 0.020
llama-cpp-quantize  | [  24/ 291]         layers.2.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.076 0.096 0.112 0.119 0.112 0.096 0.076 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  25/ 291]         layers.2.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.077 0.097 0.113 0.118 0.112 0.097 0.077 0.056 0.038 0.025 0.020
llama-cpp-quantize  | [  26/ 291]       layers.2.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  27/ 291]      layers.2.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  28/ 291]      layers.2.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  29/ 291]      layers.2.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  30/ 291]             layers.2.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  31/ 291]         layers.3.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.076 0.097 0.112 0.118 0.112 0.097 0.076 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  32/ 291]         layers.3.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.076 0.097 0.112 0.119 0.112 0.097 0.076 0.056 0.039 0.025 0.020
llama-cpp-quantize  | [  33/ 291]         layers.3.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.076 0.096 0.111 0.118 0.112 0.096 0.076 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  34/ 291]         layers.3.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  35/ 291]       layers.3.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  36/ 291]      layers.3.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  37/ 291]      layers.3.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  38/ 291]      layers.3.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  39/ 291]             layers.3.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  40/ 291]         layers.4.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.118 0.112 0.096 0.076 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  41/ 291]         layers.4.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.076 0.097 0.112 0.118 0.112 0.097 0.076 0.056 0.038 0.025 0.021
llama-cpp-quantize  | [  42/ 291]         layers.4.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.111 0.096 0.076 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  43/ 291]         layers.4.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  44/ 291]       layers.4.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  45/ 291]      layers.4.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  46/ 291]      layers.4.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  47/ 291]      layers.4.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  48/ 291]             layers.4.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  49/ 291]         layers.5.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  50/ 291]         layers.5.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  51/ 291]         layers.5.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  52/ 291]         layers.5.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  53/ 291]       layers.5.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  54/ 291]      layers.5.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  55/ 291]      layers.5.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  56/ 291]      layers.5.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  57/ 291]             layers.5.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  58/ 291]         layers.6.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  59/ 291]         layers.6.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  60/ 291]         layers.6.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  61/ 291]         layers.6.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  62/ 291]       layers.6.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  63/ 291]      layers.6.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  64/ 291]      layers.6.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  65/ 291]      layers.6.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  66/ 291]             layers.6.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  67/ 291]         layers.7.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  68/ 291]         layers.7.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.112 0.117 0.112 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  69/ 291]         layers.7.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.076 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  70/ 291]         layers.7.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  71/ 291]       layers.7.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  72/ 291]      layers.7.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  73/ 291]      layers.7.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  74/ 291]      layers.7.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  75/ 291]             layers.7.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  76/ 291]         layers.8.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  77/ 291]         layers.8.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  78/ 291]         layers.8.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.076 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  79/ 291]         layers.8.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  80/ 291]       layers.8.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  81/ 291]      layers.8.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  82/ 291]      layers.8.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  83/ 291]      layers.8.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  84/ 291]             layers.8.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  85/ 291]         layers.9.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  86/ 291]         layers.9.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.112 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  87/ 291]         layers.9.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.015 0.025 0.039 0.056 0.076 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  88/ 291]         layers.9.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  89/ 291]       layers.9.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  90/ 291]      layers.9.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  91/ 291]      layers.9.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  92/ 291]      layers.9.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  93/ 291]             layers.9.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  94/ 291]        layers.10.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  95/ 291]        layers.10.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  96/ 291]        layers.10.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.112 0.097 0.076 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [  97/ 291]        layers.10.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [  98/ 291]      layers.10.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [  99/ 291]     layers.10.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 100/ 291]     layers.10.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 101/ 291]     layers.10.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 102/ 291]            layers.10.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 103/ 291]        layers.11.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 104/ 291]        layers.11.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.112 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 105/ 291]        layers.11.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.076 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 106/ 291]        layers.11.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.110 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 107/ 291]      layers.11.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 108/ 291]     layers.11.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 109/ 291]     layers.11.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.077 0.097 0.112 0.118 0.112 0.097 0.077 0.056 0.038 0.025 0.021
llama-cpp-quantize  | [ 110/ 291]     layers.11.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 111/ 291]            layers.11.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 112/ 291]        layers.12.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 113/ 291]        layers.12.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 114/ 291]        layers.12.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 115/ 291]        layers.12.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 116/ 291]      layers.12.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 117/ 291]     layers.12.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 118/ 291]     layers.12.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 119/ 291]     layers.12.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 120/ 291]            layers.12.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 121/ 291]        layers.13.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 122/ 291]        layers.13.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 123/ 291]        layers.13.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.076 0.097 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 124/ 291]        layers.13.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.026 0.039 0.057 0.077 0.096 0.110 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 125/ 291]      layers.13.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 126/ 291]     layers.13.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 127/ 291]     layers.13.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 128/ 291]     layers.13.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 129/ 291]            layers.13.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 130/ 291]        layers.14.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 131/ 291]        layers.14.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 132/ 291]        layers.14.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 133/ 291]        layers.14.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 134/ 291]      layers.14.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 135/ 291]     layers.14.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 136/ 291]     layers.14.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 137/ 291]     layers.14.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 138/ 291]            layers.14.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 139/ 291]        layers.15.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 140/ 291]        layers.15.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 141/ 291]        layers.15.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 142/ 291]        layers.15.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.026 0.021
llama-cpp-quantize  | [ 143/ 291]      layers.15.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 144/ 291]     layers.15.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 145/ 291]     layers.15.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 146/ 291]     layers.15.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 147/ 291]            layers.15.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 148/ 291]        layers.16.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 149/ 291]        layers.16.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 150/ 291]        layers.16.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 151/ 291]        layers.16.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 152/ 291]      layers.16.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 153/ 291]     layers.16.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 154/ 291]     layers.16.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 155/ 291]     layers.16.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 156/ 291]            layers.16.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 157/ 291]        layers.17.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 158/ 291]        layers.17.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 159/ 291]        layers.17.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 160/ 291]        layers.17.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 161/ 291]      layers.17.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 162/ 291]     layers.17.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 163/ 291]     layers.17.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 164/ 291]     layers.17.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 165/ 291]            layers.17.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 166/ 291]        layers.18.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 167/ 291]        layers.18.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 168/ 291]        layers.18.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.076 0.096 0.111 0.118 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 169/ 291]        layers.18.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 170/ 291]      layers.18.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 171/ 291]     layers.18.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 172/ 291]     layers.18.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 173/ 291]     layers.18.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 174/ 291]            layers.18.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 175/ 291]        layers.19.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 176/ 291]        layers.19.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 177/ 291]        layers.19.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 178/ 291]        layers.19.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 179/ 291]      layers.19.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 180/ 291]     layers.19.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 181/ 291]     layers.19.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 182/ 291]     layers.19.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 183/ 291]            layers.19.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 184/ 291]        layers.20.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 185/ 291]        layers.20.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 186/ 291]        layers.20.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 187/ 291]        layers.20.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 188/ 291]      layers.20.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 189/ 291]     layers.20.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 190/ 291]     layers.20.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 191/ 291]     layers.20.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 192/ 291]            layers.20.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 193/ 291]        layers.21.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 194/ 291]        layers.21.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 195/ 291]        layers.21.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 196/ 291]        layers.21.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.110 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 197/ 291]      layers.21.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 198/ 291]     layers.21.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 199/ 291]     layers.21.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 200/ 291]     layers.21.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 201/ 291]            layers.21.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 202/ 291]        layers.22.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 203/ 291]        layers.22.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 204/ 291]        layers.22.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.112 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 205/ 291]        layers.22.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 206/ 291]      layers.22.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 207/ 291]     layers.22.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 208/ 291]     layers.22.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 209/ 291]     layers.22.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 210/ 291]            layers.22.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 211/ 291]        layers.23.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 212/ 291]        layers.23.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 213/ 291]        layers.23.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 214/ 291]        layers.23.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 215/ 291]      layers.23.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 216/ 291]     layers.23.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 217/ 291]     layers.23.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 218/ 291]     layers.23.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 219/ 291]            layers.23.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 220/ 291]        layers.24.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 221/ 291]        layers.24.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 222/ 291]        layers.24.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 223/ 291]        layers.24.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.026 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 224/ 291]      layers.24.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 225/ 291]     layers.24.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 226/ 291]     layers.24.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 227/ 291]     layers.24.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 228/ 291]            layers.24.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 229/ 291]        layers.25.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 230/ 291]        layers.25.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 231/ 291]        layers.25.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 232/ 291]        layers.25.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 233/ 291]      layers.25.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 234/ 291]     layers.25.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 235/ 291]     layers.25.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 236/ 291]     layers.25.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 237/ 291]            layers.25.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 238/ 291]        layers.26.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 239/ 291]        layers.26.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 240/ 291]        layers.26.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 241/ 291]        layers.26.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 242/ 291]      layers.26.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 243/ 291]     layers.26.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 244/ 291]     layers.26.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 245/ 291]     layers.26.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 246/ 291]            layers.26.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 247/ 291]        layers.27.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 248/ 291]        layers.27.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 249/ 291]        layers.27.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 250/ 291]        layers.27.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 251/ 291]      layers.27.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 252/ 291]     layers.27.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 253/ 291]     layers.27.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 254/ 291]     layers.27.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 255/ 291]            layers.27.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 256/ 291]        layers.28.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 257/ 291]        layers.28.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 258/ 291]        layers.28.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 259/ 291]        layers.28.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 260/ 291]      layers.28.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 261/ 291]     layers.28.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 262/ 291]     layers.28.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 263/ 291]     layers.28.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 264/ 291]            layers.28.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 265/ 291]        layers.29.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 266/ 291]        layers.29.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 267/ 291]        layers.29.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 268/ 291]        layers.29.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 269/ 291]      layers.29.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 270/ 291]     layers.29.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 271/ 291]     layers.29.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.077 0.097 0.112 0.118 0.112 0.097 0.077 0.056 0.038 0.025 0.021
llama-cpp-quantize  | [ 272/ 291]     layers.29.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 273/ 291]            layers.29.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 274/ 291]        layers.30.attention.wq.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.117 0.111 0.096 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 275/ 291]        layers.30.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 276/ 291]        layers.30.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.112 0.118 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 277/ 291]        layers.30.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 278/ 291]      layers.30.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 279/ 291]     layers.30.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 280/ 291]     layers.30.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.024 0.038 0.055 0.076 0.097 0.114 0.120 0.114 0.097 0.076 0.055 0.038 0.024 0.020
llama-cpp-quantize  | [ 281/ 291]     layers.30.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 282/ 291]            layers.30.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 284/ 291]        layers.31.attention.wk.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.039 0.056 0.077 0.097 0.112 0.117 0.112 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 285/ 291]        layers.31.attention.wv.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.036 0.015 0.025 0.038 0.056 0.076 0.097 0.112 0.118 0.112 0.097 0.076 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 286/ 291]        layers.31.attention.wo.weight -     4096 x  4096, type =    f16, quantizing .. size =    32.00 MB ->     9.00 MB | hist: 0.037 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.116 0.111 0.097 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  | [ 287/ 291]      layers.31.attention_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | [ 288/ 291]     layers.31.feed_forward.w1.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.057 0.077 0.096 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 289/ 291]     layers.31.feed_forward.w2.weight -    11008 x  4096, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.015 0.024 0.037 0.054 0.075 0.098 0.116 0.123 0.116 0.098 0.075 0.054 0.037 0.024 0.020
llama-cpp-quantize  | [ 290/ 291]     layers.31.feed_forward.w3.weight -     4096 x 11008, type =    f16, quantizing .. size =    86.00 MB ->    24.19 MB | hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.097 0.111 0.117 0.111 0.097 0.077 0.056 0.039 0.025 0.021
llama-cpp-quantize  | [ 291/ 291]            layers.31.ffn_norm.weight -             4096, type =    f32, size =    0.016 MB
llama-cpp-quantize  | llama_model_quantize_internal: model size  = 13133.55 MB
llama-cpp-quantize  | llama_model_quantize_internal: quant size  =  3694.54 MB
llama-cpp-quantize  | llama_model_quantize_internal: hist: 0.036 0.016 0.025 0.039 0.056 0.077 0.096 0.111 0.117 0.111 0.096 0.077 0.057 0.039 0.025 0.021
llama-cpp-quantize  |
llama-cpp-quantize  | main: quantize time = 136252.48 ms
llama-cpp-quantize  | main:    total time = 136252.48 ms
llama-cpp-quantize exited with code 0
```

量化后的目录结构

```
./models/llama/7B
├── 7B
│   ├── checklist.chk
│   ├── consolidated.00.pth
│   ├── params.json
│   └── tokenizer_checklist.chk
├── 7B-HF
│   ├── config.json
│   ├── generation_config.json
│   ├── pytorch_model-00001-of-00002.bin
│   ├── pytorch_model-00002-of-00002.bin
│   ├── pytorch_model.bin.index.json
│   ├── special_tokens_map.json
│   ├── tokenizer.json
│   ├── tokenizer.model
│   └── tokenizer_config.json
├── Chinese-Alpaca-Plus
│   ├── 7B
│   │   ├── consolidated.00.pth
│   │   ├── ggml-model-f16.bin
│   │   ├── ggml-model-q4_0.bin
│   │   ├── params.json
│   │   ├── special_tokens_map.json
│   │   └── tokenizer_config.json
│   └── tokenizer.model
├── LoRA
│   ├── chinese-alpaca-plus-lora-7b
│   │   ├── README.md
│   │   ├── adapter_config.json
│   │   ├── adapter_model.bin
│   │   ├── special_tokens_map.json
│   │   ├── tokenizer.model
│   │   └── tokenizer_config.json
│   └── chinese-llama-plus-lora-7b
│       ├── README.md
│       ├── adapter_config.json
│       ├── adapter_model.bin
│       ├── special_tokens_map.json
│       ├── tokenizer.model
│       └── tokenizer_config.json
└── tokenizer.model

7 directories, 33 files
```

## 8、使用llama.cpp加载量化后的模型
> 由于docker-compose无法实现启动后直接进入终端，这里使用Docker run的方式启动，使用docker-compose启动后进入容器，手动运行启动命令也是可以的。

```
$ docker run -it -v ./models:/app/models -e LC_ALL=zh_CN.utf8 llama.cpp:full-zh ./main -m /app/models/llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-q4_0.bin --color -f prompts/alpaca.txt -ins -c 2048 --temp 0.2 -n 256 --repeat_penalty 1.1

main: build = 1 (ffb06a3)
main: seed  = 1685673792
llama.cpp: loading model from /app/models/llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-q4_0.bin
llama_model_load_internal: format     = ggjt v3 (latest)
llama_model_load_internal: n_vocab    = 49954
llama_model_load_internal: n_ctx      = 2048
llama_model_load_internal: n_embd     = 4096
llama_model_load_internal: n_mult     = 256
llama_model_load_internal: n_head     = 32
llama_model_load_internal: n_layer    = 32
llama_model_load_internal: n_rot      = 128
llama_model_load_internal: ftype      = 2 (mostly Q4_0)
llama_model_load_internal: n_ff       = 11008
llama_model_load_internal: n_parts    = 1
llama_model_load_internal: model size = 7B
llama_model_load_internal: ggml ctx size =    0.07 MB
llama_model_load_internal: mem required  = 5486.61 MB (+ 1026.00 MB per state)
.
llama_init_from_file: kv self size  = 1024.00 MB

system_info: n_threads = 4 / 8 | AVX = 1 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | FMA = 1 | NEON = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | VSX = 0 | 
main: interactive mode on.
Reverse prompt: '### Instruction:

'
sampling: repeat_last_n = 64, repeat_penalty = 1.100000, presence_penalty = 0.000000, frequency_penalty = 0.000000, top_k = 40, tfs_z = 1.000000, top_p = 0.950000, typical_p = 1.000000, temp = 0.200000, mirostat = 0, mirostat_lr = 0.100000, mirostat_ent = 5.000000
generate: n_ctx = 2048, n_batch = 512, n_predict = 256, n_keep = 21


== Running in interactive mode. ==
 - Press Ctrl+C to interject at any time.
 - Press Return to return control to LLaMa.
 - To return control without starting a new line, end your input with '/'.
 - If you want to submit another line, end your input with '\'.

 Below is an instruction that describes a task. Write a response that appropriately completes the request.
> 你好
你好！有什么我可以帮助你的吗？
>
llama_print_timings:        load time = 73490.86 ms
llama_print_timings:      sample time =     6.72 ms /    11 runs   (    0.61 ms per token)
llama_print_timings: prompt eval time =  5442.10 ms /    42 tokens (  129.57 ms per token)
llama_print_timings:        eval time =  2138.00 ms /    11 runs   (  194.36 ms per token)
llama_print_timings:       total time = 185983.34 ms
```