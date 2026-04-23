# Sources

All sources were fetched and verified on April 22, 2026.

## Primary Sources (Full Content Verified)

1. [Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — Anthropic Engineering Blog, January 2026.
2. [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) — Hamel Husain, March 2024.
3. [LLM Evals: Everything You Need to Know](https://hamel.dev/blog/posts/evals-faq/) — Hamel Husain & Shreya Shankar, January 2026.
4. [EDDOps: Evaluation-Driven Development and Operations of LLM Agents](https://arxiv.org/abs/2411.13768) — Xia et al., arXiv, November 2024.
5. [An LLM-as-Judge Won't Save The Product](https://eugeneyan.com/writing/eval-process/) — Eugene Yan, April 2025.
6. [Eval-Driven Development with Claude Code](https://fireworks.ai/blog/eval-driven-development-with-claude-code) — Fireworks AI, 2025.
7. [A Pragmatic Guide to LLM Evals for Devs](https://newsletter.pragmaticengineer.com/p/evals) — Pragmatic Engineer, 2025.

## Agent Memory Evaluation

8. [Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers](https://arxiv.org/abs/2603.07670) — Pengfei Du, arXiv, March 2026. Survey of agent memory systems. Proposes four-layer metric stack (effectiveness, memory quality, efficiency, governance). Key findings: cross-session coherence is unsolved, nobody evaluates forgetting well, memory architecture often matters more than model choice.

## Non-Coding Agent Evaluation (Research Papers)

9. [tau-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains](https://arxiv.org/abs/2406.12045) — Yao, Shinn, Razavi, Narasimhan. Retail & airline customer service agent benchmark. Introduced pass^k metric.
10. [tau2-bench: Evaluating Conversational Agents in a Dual-Control Environment](https://arxiv.org/abs/2506.07982) — Telecom domain, dual-control where both agent and user use tools.
11. [Capability-Aligned Human-Centred Evaluation Framework for Conversational AI](https://arxiv.org/abs/2505.08253) — arXiv, May 2025. Evaluates coherence, accuracy, clarity, relevance, efficiency in dialogue.
12. [Survey on Evaluation of LLM-based Agents](https://arxiv.org/abs/2503.16416) — arXiv, March 2025. First comprehensive survey across all agent types and evaluation frameworks.

## Harness Engineering Evaluation

13. [Harness Engineering](https://martinfowler.com/articles/harness-engineering.html) — Birgitta Boeckeler, April 2026. Four-part taxonomy (guides, sensors, computational, inferential elements). Key gap: no equivalent of code coverage for harness quality.
14. [Inside the Scaffold: A Source-Code Taxonomy of Coding Agent Architectures](https://arxiv.org/abs/2604.03515) — Rombaut, April 2026. Analyzes 13 agents across 12 dimensions. Tool accuracy drops past 15-20 tools. Context compaction strategies diverge most across architectures.
15. [Meta-Harness: End-to-End Optimization of Model Harnesses](https://arxiv.org/abs/2603.28052) — Lee et al., March 2026. Harness optimizations transfer across models — infrastructure improvements yield gains regardless of which LLM you use.
16. [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Young (Anthropic), November 2025. Two critical failure modes: over-ambition (context exhaustion mid-task) and premature completion.
17. [Codified Context: Infrastructure for AI Agents in a Complex Codebase](https://arxiv.org/abs/2602.20478) — Vasilopoulos, February 2026. Hot/cold memory separation across 283 sessions. Tests domain-expert routing and compaction fidelity.
18. [Natural-Language Agent Harnesses](https://arxiv.org/abs/2603.25723) — March 2026. Scaffold differences dominate outcomes even under fixed base models.

## Secondary Sources (Search-Verified)

19. [Eval-Driven System Design](https://developers.openai.com/cookbook/examples/partners/eval_driven_system_design/receipt_inspection) — OpenAI Cookbook.
20. [Evaluation-Driven Development Workflows](https://www.databricks.com/dataaisummit/session/evaluation-driven-development-workflows-best-practices-and-real-world) — Data + AI Summit 2025.
21. [Selecting The Right AI Evals Tool](https://hamel.dev/blog/posts/eval-tools/) — Hamel Husain.
22. [AI Evals for Engineers & PMs](https://maven.com/parlance-labs/evals) — Husain & Shankar, Maven.
23. [Why AI Evals Are the Hottest New Skill](https://www.lennysnewsletter.com/p/why-ai-evals-are-the-hottest-new-skill) — Lenny's Newsletter.
24. [Awesome LLM Evaluation](https://alopatenko.github.io/LLMEvaluation/) — Curated eval resource directory.
25. [Claude Code Best Practices](https://code.claude.com/docs/en/best-practices) — Anthropic.
