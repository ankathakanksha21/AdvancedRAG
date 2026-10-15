# Query Transformation

Sometimes we don't fully understand what we're asking an AI, and we still expect perfect answers.

In a RAG system, everything starts with the query. If the query is vague or unclear, the whole pipeline carries that confusion forward and you get bad answers. Here are three ways to fix the query before retrieval.

## 1. Query decomposition

Use this when a single query needs information from multiple concepts. Instead of treating it as one big question:

1. Break it into smaller, focused sub-queries.
2. Run each sub-query through retrieval separately, which gives better document matches.
3. Combine the results into one final answer to the original question.

<p align="center">
  <img src="images/complex-query.png" alt="A complex query: I want an AI Engineer role, what should I learn, how long will it take, which tools to focus on, and how do I start getting job-ready projects?" width="450">
</p>

<p align="center">
  <img src="images/sub-queries.png" alt="The query split into four sub-queries, each sent to its own RAG step" width="400">
  <img src="images/combined-answer.png" alt="The four RAG answers combined into one final answer" width="400">
</p>

## 2. Query rewriting

Take a messy question and make it clear and precise:

- Remove the parts that don't help and only confuse retrieval.
- Add the keywords that match the right context.

The intent stays the same. It's expressed better, so retrieval finds the right context more easily.

<p align="center">
  <img src="images/query-rewriting.png" alt="'uh… something about fixing slow python code??' rewritten as 'How to optimize slow Python code performance'" width="450">
</p>

## 3. Query expansion

Create multiple related variations of the same query. This helps most when the input is vague, because the variations cover more ground.

For example, "need more open source NLP tools" expands into:

- free NLP libraries
- open source language processing platforms
- NLP software with open source code

<p align="center">
  <img src="images/query-expansion.png" alt="'need more open source NLP tools' expanded into three related queries" width="450">
</p>

Don't overdo it. Too many variations can drift from what the user meant, and the system starts pulling irrelevant results. More queries also mean more latency and cost, so expand only as much as you need.

## Beyond RAG

This problem shows up in almost every LLM-based application, not only RAG. When your RAG system gives bad answers, check the query before blaming the pipeline.
