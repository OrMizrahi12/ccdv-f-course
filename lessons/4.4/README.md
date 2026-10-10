# Lesson 4.4: Getting knowledge into context

The lesson's files, exactly as they were typed on camera (commit `Knowledge into context: BM25 retrieval as cited search results; paste vs retrieve vs search, measured`, on top of Lesson 4.3).

| File | What it is |
|---|---|
| `quizforge/retrieve.py` | The retriever, plain Python with no API calls: `chunks()` makes one chunk per lesson section (Chapters and Sources skipped), each with a source such as `L3.2#the-idea` and a title; `search(library, query, k=3)` scores every chunk with BM25 (rare words count more, long chunks less) and keeps the top three; `as_results()` turns them into search result blocks with citations enabled |
| `experiments/knowledge.py` | Three questions with the lesson that answers each, three ways in: `paste_all` (the whole library in one user message; it counts the lessons the answer names), `retrieved` (the top three search results with the question), `agentic` (a `search_library` tool that returns search results, up to four turns). Each way prints what was cited or named, its requests, input tokens, cost and right lessons |

Run it from your `quiz-forge` project, with the environment active:

```bash
python -m experiments.knowledge   # paste all vs retrieved vs agentic, three questions
```

12 Claude Haiku 4.5 requests on camera: about 18 cents. On camera: paste all 162,261 input tokens and $0.1640, retrieved 7,349 and $0.0087, agentic 9,524 and $0.0120; each found the right lesson 3 of 3, and retrieval cited it down to the section. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
