# Using a Custom LLM

`duktr` is designed to be flexible and allow you to use any LLM provider you want. This means you can use any LLM that you can call from Python, including models from Hugging Face, your own fine-tuned models, or even models hosted on other platforms.

## Using a Hugging Face Model

Here's an example of how to use a Hugging Face model with `duktr`. In this example, we'll use the `Qwen/Qwen2.5-0.5B-Instruct` model, an open-source lightweight model. It is not well-suited to complex reasoning tasks such as concept mining and clustering, but it is included here for demonstration purposes.

### 1. Install the necessary libraries

First, you'll need to install the `transformers` library from Hugging Face and `torch`.

```bash
pip install transformers torch
```

### 2. Create a wrapper function

Next, create a Python function that takes a prompt string and uses the Hugging Face `pipeline` to generate text.

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM, pipeline

MODEL_ID = "Qwen/Qwen2.5-0.5B-Instruct"

tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
model = AutoModelForCausalLM.from_pretrained(MODEL_ID)
model.eval()

generator = pipeline("text-generation", model=model, tokenizer=tokenizer)

def huggingface_llm(user_text: str) -> str:
    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": user_text},
    ]
    prompt = tokenizer.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )

    out = generator(
        prompt,
        max_new_tokens=80,
        do_sample=True,
        temperature=0.01,
        top_p=0.9,
        return_full_text=False,
    )[0]["generated_text"]

    return out.strip()

```

### 3. Use your custom LLM with `duktr`

Now, you can use the `huggingface_llm` function and pass it to the `ConceptMiner`.

```python
from duktr import ConceptMiner

# Initialize a ConceptMiner with your custom LLM function
miner = ConceptMiner(
    llm=huggingface_llm,
    task="Extract the main topics from the following news article.",
    concept="topic",
    rules="- Topics should be 1-3 words long.\n- Use title case."
)

# Now you can use the miner as usual
texts = [
    "The new drug is expected to cure cancer.",
    "The government has announced new regulations for the tech sector.",
]

per_text_concepts = miner.mine(texts)

print(per_text_concepts)
print(miner.catalog)
```

By following this pattern, you can integrate any LLM into your `duktr` workflow. All you need is a Python function that can take a prompt and return the answer as a string.

**Note:** Not all models are suitable for concept mining tasks. Choose a model that can understand and generate relevant concepts for your specific use case. Less capable models may fail to follow even simple instructions and can raise errors, since `duktr` expects a valid JSON response as specified in the prompt.
