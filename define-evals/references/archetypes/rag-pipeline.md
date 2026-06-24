# RAG Pipeline Archetype

## Signature Patterns

Retrieval, embedding, chunking, vector search, grounding, citation.

**Examples:** Knowledge base Q&A, document search, enterprise search, research assistant.

## Extraction Checklist

When analyzing a spec for RAG pipeline systems, look for:

- Corpus definition (what documents, what formats, how large?)
- Chunking strategy (how are documents split?)
- Embedding model and dimensions
- Retrieval method (vector search, hybrid, keyword)
- Re-ranking strategy (if any)
- Context assembly (how are retrieved chunks combined into a prompt?)
- Citation requirements (must the system attribute sources?)
- Freshness requirements (how current must the corpus be?)

## Default Eval Dimensions

- Retrieval precision/recall (right chunks retrieved)
- Groundedness (claims supported by retrieved sources)
- Hallucination boundary (declines when retrieval is empty or irrelevant)
- Source attribution accuracy (correct citations)
- Query understanding across phrasings (paraphrase robustness)
- Chunk relevance ranking (best chunks ranked highest)
- Retrieval latency
- Corpus coverage (what % of the corpus is reachable?)

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Retrieval precision/recall | code-based | Compare retrieved chunks to ground truth |
| Groundedness | LLM-as-judge | Check claims against sources |
| Hallucination boundary | LLM-as-judge | Check refusal on empty retrieval |
| Source attribution accuracy | code-based | Verify citations match |
| Query understanding | code-based | Same answer for paraphrases |
| Chunk relevance ranking | code-based | Rank correlation with ground truth |
| Retrieval latency | code-based | Measure time |
| Corpus coverage | code-based | Test reachability of known documents |

## Common Failure Modes

- System hallucinates when retrieval returns no relevant results (should refuse or caveat)
- Correct document retrieved but wrong chunk selected
- Citations point to sources that don't support the claim
- Paraphrased queries return completely different results
- Re-ranker degrades quality instead of improving it
- Chunking splits critical information across chunks, losing coherence

## Example Eval Tasks

```yaml
- id: hallucination-on-empty-retrieval
  requirement: System declines to answer when no relevant documents are found
  description: Ask a question with no matching documents in the corpus
  input: "What is our company's policy on bringing pets to the office?"
  expected_behavior: System says it cannot find relevant information rather than fabricating a policy
  reference_solution: "I couldn't find any documents about a pet policy. You may want to check with HR directly."
  grader_type: llm-judge
  grader_logic: "Rubric: Does the response acknowledge lack of source material? Does it avoid fabricating a policy? PASS if both, FAIL if it invents content."
  category: capability
  priority: P0

- id: citation-accuracy
  requirement: Citations match the claims they support
  description: Ask a factual question where the answer exists in the corpus
  input: "What is the maximum file upload size for our API?"
  expected_behavior: Response includes the correct limit and cites the correct source document
  reference_solution: "The maximum file upload size is 25MB (source: API Reference v3, section 4.2)"
  grader_type: code
  grader_logic: Extract cited source and section. Verify the cited document contains the claimed information. Check answer matches document content.
  category: capability
  priority: P1
```
