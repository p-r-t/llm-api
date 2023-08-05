# LLM API

Generates a REST API to use LLaMa2 model via docker images that run on CPU, not GPU


# Usage

In order to run this API on a local machine, a running docker engine is needed.

run using docker:

create a `config.yaml` file with the configs described below and then run:

```
docker run -v $PWD/models/:/models:rw -v $PWD/config.yaml:/llm-api/config.yaml:ro -p 8000:8000 --ulimit memlock=16000000000 ghcr.io/p-r-t/llm-api
```

or use the `docker-compose.yaml` in this repo and run using compose:

```
docker compose up
```

When running for the first time, the app will download the model from huggingface based on the configurations in `setup_params` and name the local model file accordingly, on later runs it looks up the same local file and loads it into memory

## Llama on CPU - using llama.cpp

You can configure the model usage in a local `config.yaml` file, the configs, here is an example:

```
models_dir: /models
model_family: llama
setup_params:
  repo_id: user/repo_id
  filename: ggml-model-q4_0.bin
model_params:
  n_ctx: 512
  n_parts: -1
  n_gpu_layers: 0
  seed: -1
  use_mmap: True
  n_threads: 8
  n_batch: 2048
  last_n_tokens_size: 64
  lora_base: null
  lora_path: null
  low_vram: False
  tensor_split: null
  rope_freq_base: 10000.0
  rope_freq_scale: 1.0
  verbose: True
```

Fill `repo_id` and `filename` to a huggingface repo where a model is hosted, and let the application download it for you.

- `convert` refers to https://github.com/ggerganov/llama.cpp/blob/master/convert-unversioned-ggml-to-ggml.py set this to true when you need to use an older model which needs to be converted
- `migrate` refers to https://github.com/ggerganov/llama.cpp/blob/master/migrate-ggml-2023-03-30-pr613.py set this to true when you need to apply this script to an older model which needs to be migrated

the following example shows the different params you can sent to Llama generate and agenerate endpoints:

```
POST /generate

curl --location 'localhost:8000/generate' \
--header 'Content-Type: application/json' \
--data '{
    "prompt": "What is the capital of paris",
    "params": {
        "suffix": null or string,
        "max_tokens": 128,
        "temperature": 0.8,
        "top_p": 0.95,
        "logprobs": null or integer,
        "echo": False,
        "stop": ["\Q"],
        "frequency_penalty: 0.0,
        "presence_penalty": 0.0,
        "repeat_penalty": 1.1
        "top_k": 40,
    }
}'
```

# Credits

- [llama.cpp](https://github.com/ggerganov/llama.cpp) for making it possible to run Llama models on CPU. 
- [llama-cpp-python](https://github.com/abetlen/llama-cpp-python) for the python bindings lib for `llama.cpp`
- [GPTQ-for-LLaMa](https://github.com/qwopqwop200/GPTQ-for-LLaMa) for providing a GPTQ implementation for Llama based models.
