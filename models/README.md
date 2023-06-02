# 文件层级说明

原始llama模型权重，请按照以下目录格式放置，其他的目录会由脚本自动创建。

```
models/
├── README.md
└── llama
    └── 7B
        ├── 7B
        │   ├── checklist.chk
        │   ├── consolidated.00.pth
        │   ├── params.json
        │   └── tokenizer_checklist.chk
        └── tokenizer.model
```

## Models文件夹说明

- 原始模型权重
```
    llama/7B/7B
    llama/7B/tokenizer.model
```

- HF格式模型权重
```
    llama/7B/7B-HF
```

- Chinese-LLaMA-Alpaca LoRA
```
    llama/7B/LoRA
```

- 合并完成的全量模型权重
```
    llama/7B/Chinese-Alpaca-Plus/7B
    llama/7B/Chinese-Alpaca-Plus/tokenizer.model
```

- ggml FP16格式模型
```
    llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-f16.bin
```

- 量化后的模型
```
    llama/7B/Chinese-Alpaca-Plus/7B/ggml-model-q4_0.bin
```