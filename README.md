# Securing a Local-First AI Assistant Against Prompt Injection

A writeup of the security architecture I designed for a local, tool-using voice
assistant — focused on the threat that tops the OWASP LLM Top 10: prompt
injection, and its persistent cousin, memory poisoning.

> This repo documents architecture and threat modeling. The implementation is
> private.

## The problem
A tool-using AI agent that can search the web, read/write a local knowledge
vault, and chain tools has a dangerous property: untrusted external text
(search results, file contents, on-screen text) flows into the same model that
decides which tools to call. That's the prompt-injection attack surface, and
autonomous writes make it persistent (the ZombieAgent / SpAIware memory-poisoning
class).

## Attack surface I modeled
- Web search results reaching the model as trusted context
- Vault poisoning — a malicious write persisting an injection across sessions
- Tool chaining through the dispatch layer

## Core principle: trust follows authorship, not location
The key design decision: **only OS-asserted structural facts** (process IDs,
window counts, booleans, app identity via verified PID mapping) may influence
the tool-calling path. **Every human-readable string** (window titles, filenames,
notification text, OCR output, DOM text) is treated as untrusted and routed
through a dual-LLM quarantine. Content can inform what the assistant *says*, but
never directly trigger what it *does*.

## Layered defenses
1. Regex pre-screen on tool results
2. Classifier sidecar for semantic attacks
3. Dual-LLM pattern — untrusted content handled by tool-less tangent agents
4. Vault provenance tagging (`source: web_search` in frontmatter) so derived
   content is treated with appropriate skepticism on future reads
5. Permission gates on high-impact writes and external calls

## Validation
Built an 80-probe injection eval fixture wired into CI so a regression in
injection resistance fails the build. Live perception-ladder testing held against
every injection attempt in the test set.

## References
OWASP LLM Top 10 · Dual LLM pattern · LlamaFirewall / LLM Guard · 2026
in-the-wild reporting (Forcepoint X-Labs, Google, Unit 42)
