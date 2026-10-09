# Lesson 3.4: Images, PDFs, citations and files

The lesson's files, exactly as they were typed on camera (commit `Images, PDFs with citations, and the Files API, measured`, on top of Lesson 3.3). The documents they read are in `../../library/ccao-f/v1-docs/` (the student kit's `quiz-forge-documents.zip` holds the same files).

| File | What it is |
|---|---|
| `experiments/image.py` | A 1920x1080 slide as a base64 `image` block, placed before the question: Claude writes one practice question from it. Prints the expected image tokens (1,560, after Haiku 4.5 scales it to 1456x819) and the measured ones |
| `experiments/citations.py` | Lesson 3.2 as a `document` block, plain text and PDF, with citations enabled: each cited claim with its character range (checked against the notes) or its page |
| `quizforge/files.py` | `file_id(client, path)`: uploads a file once with the Files API, keeps `sha256 → file_id` in `runs/files.json`, and reuses the id while it still exists |
| `experiments/reuse.py` | The same PDF sent inline (base64) and by `file_id`: the bytes each request sends, and the input tokens of each |

Run them from your `quiz-forge` project, with the environment active and the documents unzipped:

```bash
python -m experiments.image
python -m experiments.citations
python -m experiments.reuse
python -m experiments.reuse
```

About 7 Claude Haiku 4.5 requests: under 10 cents. On camera: 1,560 image tokens expected, 1,564 measured; every character range matched its `cited_text`, and the PDF's claims were cited on pages 1 and 2; the PDF went as 173,491 bytes inline and 196 bytes by `file_id`, and both requests cost 10,811 input tokens. The student kit has the steps for Windows, macOS and Linux, and what the lesson printed.
