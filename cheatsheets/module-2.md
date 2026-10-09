# Module 2 on one page: Prompt Engineering & Output Handling

CCDV-F · two skills of the exam's Prompt and Context Engineering domain: Prompt Engineering (4.6%) and Output Handling (2.6%), about 4 of 53 items · checked against the Claude docs on 2026-10-08

| Lesson | Keep this |
|---|---|
| 2.1 Production prompts | Long input on top in its own tag, then context (who reads the output, and why), instructions as success criteria, and a format section that says what shape comes back. Say what to do and why, not shouted rules: dial back aggressive language. Measure with code checks over several runs. |
| 2.2 Where instructions live | Standing operator rules in the system prompt, once. This request and its data in the user turn. Text you didn't write (files, web pages, tool output) is data: a tool result or a tagged user block, never the system prompt. Tool rules in the tool's description. A rule that starts mid-session: a mid-conversation system message, which keeps the cache. Claude Code: CLAUDE.md for "always", a Skill for on-demand procedures. |
| 2.3 Examples | Start with zero and measure. When format or tone drifts, add 3–5 examples: relevant, diverse, each in `<example>` tags, about other content than the input. Claude copies what examples share, and every example's tokens ride along on every call. |
| 2.4 Structured output | Asking for JSON in the prompt is a request; structured outputs (`output_config.format`, `messages.parse()`) or a `strict: true` tool make the schema a constraint. Design the schema to leave no room: required fields, enums, fixed slots. It guarantees the shape, not the facts, and a refusal or `max_tokens` can still break it. |
| 2.5 Trust but verify | Check `stop_reason` first: a refusal is a normal response, `max_tokens` is a cut-off reply. Parse defensively, then check the rules the schema can't hold, in code, against the source. Send the reasons back and retry a limited number of times, then fail loudly. |
| 2.6 Iterating | Measure a baseline on fixed inputs with code checks, tally failures by kind, read the biggest kind, change one thing, measure again, keep or revert. A few runs show a direction, not a rate. Every measured prompt is a numbered file, never edited, with a changelog. Clean untrusted input before it goes inside its tags. |

## The four traps

| Trap | It sounds like | Why it's wrong |
|---|---|---|
| Say it louder | "Add IMPORTANT: YOU MUST…" "Reply with ONLY JSON." "Be consistent." | A plea adds no information: Claude still doesn't know who reads the output or why the format matters, and nothing enforces it. Say why, define the format, and let structure or code enforce it. |
| Make it bigger | "Switch to the most capable model." | A bigger model doesn't know your dashboard, your schema or your source. Fix the prompt and the checks first; change models when a measurement says so. |
| More of the same | "Add twenty more examples." | Examples of one kind teach one kind, and Claude copies what they share. Use 3–5 diverse examples, and count their tokens. |
| Take its word | "It's valid JSON, so it's right." "Ask Claude to double-check itself." "Retry until it parses." | Valid-looking isn't correct. Check the claims in code against the source, repair with a retry limit, and measure every change on the same inputs. |

Sources: platform.claude.com/docs (Prompt engineering overview, Prompting best practices, Mid-conversation system messages, Mitigate jailbreaks and prompt injections, Define tools, Increase output consistency, Structured outputs, Strict tool use, Reduce hallucinations, Handling stop reasons, Refusals, Define success criteria and build evaluations) and the CCDV-F exam guide. API details change: confirm them in the official docs.
