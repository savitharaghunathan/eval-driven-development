# Content Generation Archetype

## Signature Patterns

Writing, editing, style guides, brand voice, templates, formatting requirements.

**Examples:** Marketing copy, documentation generator, translation, email drafting, summarization.

## Extraction Checklist

When analyzing a spec for content generation systems, look for:

- Style guide or brand voice document
- Target audience definition
- Format requirements (length, structure, tone)
- Factual accuracy requirements (can it hallucinate?)
- Source material (what does it draw from?)
- Approval/review workflow (who signs off?)
- Localization requirements
- Legal/compliance constraints (disclaimers, required language)

## Default Eval Dimensions

- Style guide adherence (matches specified voice/tone)
- Factual accuracy (claims are verifiable)
- Audience appropriateness (matches target audience level)
- Format compliance (length, structure, heading usage)
- Originality (not regurgitating training data verbatim)
- Relevance (content addresses the specified topic)
- Consistency (maintains voice across sections/pieces)

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Style guide adherence | LLM-as-judge | Rubric against style guide |
| Factual accuracy | LLM-as-judge + code-based | Fact verification |
| Audience appropriateness | LLM-as-judge | Rubric on reading level, jargon |
| Format compliance | code-based | Length, heading structure, word count |
| Originality | code-based | Plagiarism/similarity check |
| Relevance | LLM-as-judge | Rubric on topic coverage |
| Consistency | LLM-as-judge | Compare voice across sections |

## Common Failure Modes

- Generated content matches style guide in tone but includes factual errors
- Content is technically accurate but wrong reading level for the audience
- Output is the right length but poorly structured (no headings, wall of text)
- Content regurgitates training data instead of synthesizing from provided sources
- Voice is inconsistent across sections (formal intro, casual body, robotic conclusion)
- Legal/compliance disclaimers are missing when required

## Example Eval Tasks

```yaml
- id: style-guide-adherence
  requirement: Generated content follows the brand style guide
  description: Generate a product announcement blog post given a style guide and product details
  input: Style guide (casual, second-person, short paragraphs, no jargon) + product feature description
  expected_behavior: Blog post follows all style guide rules
  reference_solution: 300-word blog post in second person ("you"), paragraphs under 3 sentences, no technical jargon
  grader_type: llm-judge
  grader_logic: "Rubric: (1) Uses second person consistently, (2) paragraphs ≤ 3 sentences, (3) no jargon or jargon is explained, (4) tone is casual. PASS if all four."
  category: capability
  priority: P1

- id: factual-accuracy-from-sources
  requirement: Claims in generated content are supported by provided source material
  description: Generate a summary article from 3 source documents
  input: Three source documents about a company's Q2 earnings
  expected_behavior: All numerical claims and facts in the output are traceable to the source documents
  reference_solution: Article cites correct revenue figures, growth percentages, and executive quotes from the sources
  grader_type: llm-judge
  grader_logic: "For each factual claim in the output, verify it appears in at least one source document. PASS if all claims are supported. FAIL if any claim is unsupported or contradicts sources."
  category: capability
  priority: P0
```
