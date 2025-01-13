# vllm-cheat-sheet



# Install

<br><br>

## Ubuntu
```shell
uv venv myenv --python 3.12 --seed
source myenv/bin/activate

uv pip install vllm
```











<br><br>
<br><br>

# Models

<br><br>

## Supported Models
- https://docs.vllm.ai/en/latest/models/supported_models.html#supported-models
- vLLM supports generative and pooling models across various tasks. If a model supports more than one task, you can set the task via the --task argument. For each task, we list the model architectures that have been implemented in vLLM. Alongside each architecture, we include some popular models that use it.

<br><br>
<br><br>

## Load Model
```
from vllm import LLM

# For generative models (task=generate) only
llm = LLM(model=..., task="generate")  # Name or path of your model
output = llm.generate("Hello, my name is")
print(output)

# For pooling models (task={embed,classify,reward,score}) only
llm = LLM(model=..., task="embed")  # Name or path of your model
output = llm.encode("Hello, my name is")
print(output)
```

