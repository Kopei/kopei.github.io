# AGENTS.md

Repository-wide coding-agent guidance. Follow existing project documentation and repository-specific requirements.

## Simplicity and proportional defensive programming

Adapted from OpenAI's [Coding style](https://developers.openai.com/api/docs/guides/latest-model#coding-style) and [Favor leaner prompts](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6#favor-leaner-prompts).

- Deliver the complete requested behavior with the simplest readable implementation. Reuse existing logic, types and APIs; preserve caller-visible behavior. Avoid speculative abstractions, unnecessary wrappers, compatibility layers, dependencies and unrelated refactoring.
- Validate untrusted data at trust boundaries; rely on established types and invariants internally. Avoid redundant checks, blanket exception handling, speculative fallbacks and retries without a concrete failure mode. Surface unexpected failures rather than masking them.
- Reuse existing tests. Add only tests requested, required by repository gates or needed for a specific uncovered regression. Avoid duplicative cases and unnecessary test infrastructure.
- Keep required correctness, security, privacy, authorization, data integrity and release checks intact. Simplicity does not justify skipping applicable gates. Keep instructions concise and avoid repeating rules.
