# Grounded Generation

RAG has three steps, but most people only optimize two. That's why their systems still hallucinate.

<p align="center">
  <img src="images/rag-three-steps.png" alt="Step 1: retriever finds the correct chunk. Step 2: augmentation adds context and prompt to the LLM. Step 3: generation by the LLM" width="450">
</p>

If you're building with RAG, you probably obsess over retrieval and handle augmentation well. Generation gets left to the model, which handles it in its own hallucinating way.

## The problem

Sometimes the retriever does its job. You get the right chunk, the context is injected, everything looks perfect, and the model still hallucinates.

<p align="center">
  <img src="images/confused-robot.png" alt="A confused robot surrounded by question marks and crosses" width="450">
</p>

This happens in two ways:

- The model ignores your context and answers from what it already knows.
- The model partly uses your context but mixes it with its own knowledge, giving a confident answer that doesn't match your source.

The answer sounds so fluent that you don't think to check it against your documents.

<p align="center">
  <img src="images/fluent-answer.png" alt="A fluent, confident-looking LLM answer listing AI engineering courses" width="450">
</p>

You end up with answers that are generic, wrong, or not grounded in your data. Here are two ways to fix this.

## 1. Tell the model exactly how to behave

Give the model explicit instructions in the prompt:

```text
Only use the provided context.
Don't rely on your prior knowledge.
If the answer isn't in the context, say you don't know.
```

Don't suggest. Control it. If the model still struggles, upgrade to a better model.

## 2. Make the model cite its source

Use structured output with citations, so the model shows where each answer came from:

<p align="center">
  <img src="images/citation-output.png" alt="JSON output with an answer and a citation containing chunk_id, source and excerpt" height="120">
  <img src="images/dont-know-output.png" alt="JSON output with the answer 'I don't know.' and a null citation" height="120">
</p>

If the model can't point to a source, it should say "I don't know" instead of bluffing. This keeps answers grounded and traceable, so you can trust the system even in high-stakes areas like legal or healthcare.
