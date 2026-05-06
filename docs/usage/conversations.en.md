# Multi-turn conversations

A **conversation** is a persistent discussion thread — each question
takes the previous exchanges into account. Useful for refining a
topic, comparing answers, or chaining several related questions
without repeating yourself.

## Start a conversation

On the search page `/user`, tick "Multi-turn conversation" before
the first question. Subsequent questions appear in the same thread
and inherit the context.

You can also convert an existing search into a conversation via
the "Continue this discussion" button.

## Persistence

All conversations are **encrypted at rest** (Fernet, derived from
`SECRET_KEY`) and kept indefinitely under `/user/conversations`. You
can:

- **Rename** a conversation
- **Archive** (hide from the main list without deleting)
- **Delete** permanently (immediate purge)
- **Export** as Markdown or JSON

## Context window

The context handed to the LLM at each turn includes:

- The **last N turns** (configurable, default 5)
- The **source citations** from previous turns
- The **system prompt** of the organisation (if configured)

Beyond N, older turns are summarised automatically by an auxiliary
LLM to stay under the synthesis model's context limit.

## Best practices

- One conversation = one topic. Open a fresh thread for a different
  topic to avoid context pollution.
- Rephrase explicitly when the LLM drifts ("Go back to the original
  question about X").
- Use strict mode (see [RAG search](rag-search.md#strict-mode)) if
  your operational decisions depend on the answer.
