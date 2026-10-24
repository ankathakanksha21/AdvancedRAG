# Failure Handling in RAG

A RAG pipeline can break silently in seven ways. Here's how to handle each one.

## 1. Empty retrieval

**Problem:** Retrieval returns no chunks at all. The empty context goes straight to the LLM, and it hallucinates confidently.

<p align="center">
  <img src="images/empty-retrieval.png" alt="User query → vector search returns 0 results → generate response" width="450">
</p>

**Fix:** Use a fallback chain:

1. Rewrite the query and retry.
2. If that still returns nothing, fall back to BM25 keyword search.
3. If that also fails, decline gracefully. Don't let the LLM make things up.

<p align="center">
  <img src="images/empty-retrieval-fix.png" alt="Vector search returns 0 results → query rewrite → retry with keyword search → if still empty, generate a graceful degradation response" width="450">
</p>

## 2. Low-confidence retrieval

**Problem:** Top-k returns chunks, but they're the wrong ones. If they reach the LLM, the answer goes off-topic. The root cause is that nothing checks how relevant the results are.

<p align="center">
  <img src="images/low-confidence-retrieval.png" alt="User query → vector search returns low-confidence results → generate response" width="450">
</p>

**Fix:** Add a similarity score threshold. If the top result scores below it, don't trust those chunks. Rerank chunks before passing anything to the LLM.

<p align="center">
  <img src="images/low-confidence-retrieval-fix.png" alt="Low-confidence results → score threshold (max below 0.7) → rerank → only high-score chunks go to response generation" width="450">
</p>

## 3. LLM timeout

**Problem:** The model never responds. The API stalls, no timeout is set, and the request hangs while your user stares at a spinner.

<p align="center">
  <img src="images/unknown-error.png" alt="Chat app error: Error receiving response. An unknown error has occurred." width="450">
</p>

**Fix:** Set a hard timeout. 10 seconds is a good starting point. If it hits, fall back to a smaller, faster model instead of failing completely.

<p align="center">
  <img src="images/timeout-fix.png" alt="Vector search results → primary model; if the timeout hits, a smaller model answers instead" width="450">
</p>

## 4. Rate limit hit

**Problem:** Too many requests, and the API returns a 429. A naive RAG system forwards that error straight to the user.

<p align="center">
  <img src="images/rate-limit-error.png" alt="User query → vector search → generate response returns a confusing error message to the user" height="150">
  <img src="images/message-limit-error.png" alt="Chat app error: Message limit reached for current model. Please try again later or switch models." height="150">
</p>

**Fix:** Use exponential backoff: wait 2 seconds, then 4, then 8, with a maximum of 3 retries. Then handle success or failure gracefully.

<p align="center">
  <img src="images/rate-limit-fix.png" alt="LLM API call retried with backoff of 2s, 4s, then 8s, up to 3 retries, ending in success or a graceful degradation response" width="450">
</p>

## 5. Context too long

**Problem:** Retrieved chunks, conversation history, and the system prompt add up fast and exceed the model's context limit. The call fails with a 400.

**Fix:** Use adaptive prompting. Estimate the token count before the call. If you're over budget:

1. Trim to your top 3 chunks.
2. Summarize those chunks further.
3. Drop older conversation history.

<p align="center">
  <img src="images/context-trim-fix.png" alt="Token estimate → if over budget, adaptive trim summarizes the context → generate response" width="450">
</p>

Or route the request to a long-context model.

<p align="center">
  <img src="images/context-too-long-fix.png" alt="User query → vector search → token estimate → if over the token budget, route to a long-context model" width="450">
</p>

## 6. Cost runaway

**Problem:** A long conversation with large retrieved context means a huge token count on every call. No one notices until the bill arrives at the end of the month.

**Fix:** Estimate the cost before every LLM call and set two thresholds:

- **Yellow line:** Switch to a cheaper model.
- **Red line:** Reject the request.

Track cumulative spend per user, per session, and per day, and set circuit breakers so nothing runs unchecked.

<p align="center">
  <img src="images/cost-runaway-fix.png" alt="Estimated price → budget check: yellow switches to a cheaper model, red rejects the request" width="450">
</p>

## 7. Low-confidence answer

**Problem:** The LLM generates an answer that isn't grounded in what you retrieved. It sounds confident, it's wrong, and your user believes it.

**Fix:** After generation, score the answer against your retrieved chunks. If the grounding score is below your threshold, don't deliver it directly. Route it to a human review queue, or at least flag it with an explicit uncertainty marker before it reaches the user.

<p align="center">
  <img src="images/grounding-check-fix.png" alt="Generate response → grounding score → human review → verified reply or flagged" width="450">
</p>

## Where to start

You don't need all seven on day one, but you need to know where the holes are. Failure handling is about what you do when something breaks.
