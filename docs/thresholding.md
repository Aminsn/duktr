## Catalog thresholding (`threshold`)

`duktr` maintains a global **concept catalog** that grows as you mine more texts. For each new text, the LLM is prompted with:

- the text itself (`{text}`), and
- an “Existing list” of concepts (`{existing_list}`) so it can **reuse** existing concepts when appropriate and only **create new** concepts when necessary.

As the catalog grows, sending the entire catalog on every call can exceed context limits or become inefficient, especially since many LLMs exhibit reduced attention to information in the middle of very long prompts. The `threshold` parameter controls when (and how) `duktr` **partitions** the catalog to keep each LLM call bounded.

### What `threshold` means

`threshold` is an integer (must be **>= 2**) that acts as a **maximum catalog size per single LLM call**.

- If the current catalog size is **≤ `threshold`**:  
  `duktr` calls the LLM **once** with the full catalog.

- If the current catalog size is **> `threshold`**:  
  `duktr` switches to **progressive partitioning** (multi-call evaluation) to avoid passing an oversized catalog in a single prompt.

Default: `threshold=300`.

### How thresholding works when the catalog is large

When `len(catalog) > threshold`, `duktr` evaluates the text against the catalog in **chunks**:

1. **Partition the catalog** into subsets.
   - Each partition has size up to `threshold - 1` concepts (an implementation detail to keep each chunk strictly bounded).
2. Maintain a running set called **carried matches**.
   - This contains concepts that have already been matched in earlier partitions and should continue to be available in later calls.
3. For each partition:
   - Call the LLM with:  
     **`partition ∪ carried_matches`**
   - Take the LLM’s output `o_j` and:
     - **Carry forward only the concepts that were already in the input** for that call:  
       `carried_matches ← carried_matches ∪ (o_j ∩ input_set)`
     - Keep the most recent output as `last_output`.

4. **Early-stop heuristic** (performance optimization):
   - If the LLM returns a non-empty set and **all returned concepts are already contained in the input set**, `duktr` stops scanning further partitions for this text.
   - Intuition: the model has successfully described the text using existing concepts and did not introduce new ones, so further partitions are unlikely to help.

5. Final result for the text:
   - Start from the `last_output` (from the early-stopped partition or the last partition if no early stop occurred).
   - Ensure all carried matches are included:  
     `final = last_output ∪ carried_matches`

6. **Update the global catalog**:
   - The global catalog is updated after each text with any concepts in `final` (including newly created ones).

### Practical implications of `threshold`

Choosing `threshold` is a trade-off between **prompt/catalog size** and **number of LLM calls**:

- **Higher `threshold`**
  - Fewer partitions → fewer LLM calls per text
  - Larger prompts (more concepts in `{existing_list}`)
  - Higher risk of hitting context limits or missing relevant concepts due to prompt length

- **Lower `threshold`**
  - More partitions → more LLM calls per text
  - Smaller prompts (each call sees fewer concepts)
  - Better control over context size, lower risk of missing any existing relevant concepts but potentially higher total latency/cost

### When to adjust it

You may want to **decrease** `threshold` if:
- your LLM has a tight context window,
- performance issues due to LLM ignoring middle parts of long list of concepts

You may want to **increase** `threshold` if:
- your LLM comfortably handles larger inputs,
- you want fewer round-trips per text,
- your concept strings are short and the catalog remains manageable.

### Constraints and validation

- `threshold` must be an **integer >= 2**.
- When thresholding is active, partitions are sized up to **`threshold - 1`** concepts, and then **carried matches** are added on top. In typical use, carried matches are small (only concepts that actually matched the text).

### Example (conceptual)

```python
from duktr import ConceptMiner, OpenAIProvider

miner = ConceptMiner(
    llm = OpenAIProvider(api_key="<YOUR_API_KEY>"),
    prompt = "<some prompt>",
    threshold = 5,  # small threshold for demonstration
    initial_catalog = [
        "Concept A", "Concept B", "Concept C", "Concept D",
        "Concept E", "Concept F", "Concept G", "Concept H",
        "Concept I", "Concept J", "Concept K", "Concept L"
    ]
)
```

Assume:
- `threshold = 5`
- catalog size = 12 (so partitioning is used)

`duktr` will:
- split the 12 concepts into partitions of up to `4` (`threshold - 1`)
- call the LLM sequentially with each partition plus any carried matches
- stop early if the LLM successfully matches existing concepts without introducing new ones
- otherwise continue through all partitions and merge results

The key behavior: **`threshold` never filters or scores concepts**. It only controls **how much of the catalog is shown to the LLM at a time**, and how `duktr` progressively searches the catalog while keeping prompts bounded.
