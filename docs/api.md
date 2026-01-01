# API Reference

This page provides a detailed API reference for all public classes in the `duktr` package.

## `ConceptMiner`

High-level interface for concept discovery using LLMs.

### `__init__`

```python
def __init__(
    self,
    llm: LLMInput = None,
    prompt: Optional[str] = None,
    task: Optional[str] = None,
    concept: Optional[str] = None,
    rules: Optional[str] = None,
    threshold: int = DEFAULT_CATALOG_THRESHOLD,
    initial_catalog: Optional[Set[str]] = None
)
```

Initializes a `ConceptMiner` instance.

There are two ways to configure the prompt used for concept mining:

1.  **Provide `task`, `concept`, and `rules`**: `duktr` will automatically generate a prompt for you. This is the recommended approach for most use cases.
2.  **Provide a custom `prompt`**: You can provide your own prompt string. This is useful if you need more control over the prompt.

**Important**: You can only use one of these options at a time. If you provide both a `prompt` and the `task`, `concept`, and `rules` parameters, `duktr` will use the `prompt` and ignore the other parameters.

**Args:**

- **`llm`**: An instance of a class that inherits from `BaseLLMProvider` or a callable that takes a string prompt and returns a string.
- **`prompt`**: A string that contains the prompt to be used for concept mining. It must include the placeholders `{text}` and `{existing_list}`.
- **`task`**: A string that describes the task for the LLM. Used to generate a prompt if a custom `prompt` is not provided.
- **`concept`**: A string that specifies the concept to be mined. Used to generate a prompt if a custom `prompt` is not provided.
- **`rules`**: A string that contains the rules for the LLM to follow. Used to generate a prompt if a custom `prompt` is not provided.
- **`threshold`**: An integer that specifies the maximum number of concepts to be sent to the LLM in a single request.
- **`initial_catalog`**: A set of strings that contains the initial concepts to be used for mining.

### `mine`

```python
def mine(
    self,
    texts: Iterable[str],
    progress: bool = False
) -> List[Set[str]]
```

Discovers concepts from a list of texts.

**Args:**

- **`texts`**: An iterable of strings that contains the texts to be mined.
- **`progress`**: A boolean that indicates whether to show a progress bar.

**Returns:**

A list of sets of strings, where each set contains the concepts mined from the corresponding text.

---

## LLM Providers

`duktr` provides a flexible architecture that allows you to use different LLM providers. You can use one of the built-in providers or create your own (refer to [Using a Custom LLM](./custom_llm.md)).

### `GeminiProvider`

This class provides an interface to the Google Gemini API.

```python
class GeminiProvider(BaseLLMProvider):
    def __init__(
        self,
        api_key: str,
        model: str = DEFAULT_GEMINI_MODEL,
        temperature: float = DEFAULT_TEMPERATURE,
        timeout: int = DEFAULT_TIMEOUT_SECONDS
    )
```

**Args:**

- **`api_key`**: Your Google API key.
- **`model`**: The name of the Gemini model to use.
- **`temperature`**: The temperature to use for text generation.
- **`timeout`**: The timeout in seconds for the API request.

### `OpenAIProvider`

This class provides an interface to the OpenAI API.

```python
class OpenAIProvider(BaseLLMProvider):
    def __init__(
        self,
        api_key: str,
        model: str = DEFAULT_OPENAI_MODEL,
        timeout: int = DEFAULT_TIMEOUT_SECONDS
    )
```

**Args:**

- **`api_key`**: Your OpenAI API key.
- **`model`**: The name of the OpenAI model to use.
- **`timeout`**: The timeout in seconds for the API request.

---
