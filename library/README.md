# Quiz Forge library

The source material Quiz Forge reads: the final lesson notes of the Claude Certified Associate – Foundations (CCAO-F) course, by the same author.

- `ccao-f/v1/`: a frozen snapshot of 14 lessons (L0.1–L3.3), taken on 2026-10-07 before Module 1. Each file is a lesson's notes with the music credits removed. `MANIFEST.json` lists each lesson's title, module, objectives, size and SHA-256.
- `ccao-f/v1-docs/` (from Lesson 3.4): the same 14 lessons rendered as PDFs (3–5 pages each, same text), and `L3.2-slide.png`, one 1920x1080 slide from the CCAO-F Lesson 3.2 video. `MANIFEST.json` lists pages, size and SHA-256. Lesson 3.4 ships it as `quiz-forge-documents.zip`.
- A snapshot is never edited. When more lessons are added, they go into a new snapshot, `v2/`.

Every lesson and lab in this course uses `ccao-f/v1`, so your results match the videos.
