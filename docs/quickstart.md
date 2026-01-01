# Quick Start

This guide will walk you through the process of using `duktr` to mine concepts from a list of texts.

## 1. Import the necessary classes

First, import the `ConceptMiner` and `GeminiProvider` classes (or `OpenAIProvider`, or any other LLM provider of your choice; for custom LLMs, refer to [Using a Custom LLM](./custom_llm.md)) from the `duktr` package:

```python
from duktr import ConceptMiner, GeminiProvider
```

## 2. Initialize the ConceptMiner

Next, initialize a `ConceptMiner` instance. You'll need to provide an LLM provider and configure the prompt that will be used to instruct the LLM.

There are two ways to configure the prompt:

**Option 1: Let `duktr` build the prompt for you (Recommended)**

The easiest way to get started is to provide a `task` description, the `concept` you want to extract, and a set of `rules` for the LLM to follow. `duktr` will then use this information to generate a high-quality prompt for you.

```python
# Initialize a miner that extracts "symptom" as the target concept from free-text records.
miner = ConceptMiner(
    llm=GeminiProvider(api_key="YOUR_API_KEY"),
    task="Extract the symptom(s) of the patient from their record.",
    concept="symptom",
    rules="""
    - Use short noun phrases (2–6 words)
    - Symptom(s) must be independent; no duplicates
    - Output in English
    """,
)

# You can inspect the generated prompt using:
print(miner.prompt)
```

**Option 2: Provide your own custom prompt**

If you need more control over the prompt, you can provide your own custom prompt string using the `prompt` parameter. Your prompt **must** include the following placeholders:

- `{text}`: This will be replaced with the input text.
- `{existing_list}`: This will be replaced with the list of existing concepts (catalog).

```python
from duktr.prompt import generate_prompt

# You can use the `generate_prompt` function to create a prompt template
# and then customize it to your needs.
my_prompt = generate_prompt(
    task="Extract the main topics from the following news article.",
    concept="topic",
    rules="- Topics should be 1-3 words long.\n- Use title case."
)

# Or you can write your own prompt from scratch.
my_prompt = """
Extract the main topics from the following news article.

Existing topics:
{existing_list}

Article:
{text}

You must output the extracted topics in a valid format exactly as follows:

["<topic 1>", "<topic 2>", ...]
"""

miner = ConceptMiner(
    llm=GeminiProvider(api_key="YOUR_API_KEY"),
    prompt=my_prompt,
)
```

**Important:** You can only use one of these options at a time. If you provide both a `prompt` and the `task`, `concept`, and `rules` parameters, `duktr` will use the `prompt` and ignore the other parameters.


## 3. Prepare your input texts

Create a list of texts that you want to mine for concepts.

```python
# Example inputs (note: the third record paraphrases the first).
texts = [
     "Patient reports frequent headaches and occasional dizziness.",
     "The individual is experiencing shortness of breath during exertion.",
     "The patient describes recurrent head pain along with vertigo." # paraphrased version of first text
]

# Note: duktr supports any iterable of strings as input, such as lists and Pandas Series.
# Pandas example 
import pandas as pd

df = pd.DataFrame({
    "text": texts
})

texts = df["text"]
```

## 4. Mine for concepts

Call the `mine()` method on your `ConceptMiner` instance to extract concepts from the texts.

```python
# Mine concepts per text.
per_text_concepts = miner.mine(texts)

# Pandas example
df["concepts"] = per_text_concepts
```

## 5. View the results

The `mine()` method returns a list of sets, where each set contains the concepts found in the corresponding text. The `ConceptMiner` instance also maintains a global catalog of all unique concepts found across all texts.

```python
print(per_text_concepts)  # concepts found in each record
print(miner.catalog)      # global catalog across all records
```

Expected output (using `GeminiProvider`'s default model):

```python
[{"Headache", "Dizziness"}, {"Shortness of breath"}, {"Headache", "Dizziness"}]
{"Headache", "Dizziness", "Shortness of breath"}
```
