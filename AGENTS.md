# CCA-F Study Vault

Obsidian vault of study notes for the Claude Certified Architect – Foundations (CCA-F) exam. Start at `Home.md`.

## Layout
- `Concepts/`: one topic per note.
- `Domains/`: the 5 exam domains. Each task statement is a heading listing the notes that cover it.
- `Scenarios/`: the 6 exam scenarios, plus `Exam-Scenarios.md` on telling them apart.
- `Reference/`, `Resources/`: technology reference; exam guide, courses and links.
- `Reference/Claude-API-*.md`: one note per chapter of Anthropic's Building with the Claude API course, listed in the hub `Claude-API-Anthropic-Client-SDK.md`. Each starts with links to the previous chapter, the hub and the next. New course material goes in its chapter's note.
- `assets/`: every image, PDF and HTML diagram, including the official exam guide.
- `Study-Tracker.base`, `Exam-Map.canvas`, `Exam-Tips.md`: views built from the notes.

## Conventions
- File names use dashes, never spaces.
- Link with `[[wikilinks]]`. Link a task as `[[Domain-N-…#N.M <task statement>|Task N.M]]` and a scenario as `[[Scenario-N-…|Scenario N]]`.
- A new concept note gets properties `type: concept`, `domain`, `tasks`, `tags`, `aliases`, `confident: false`, and a line under each of its task headings in the domain note.
- An exam tip is a `> [!tip] Exam tip` callout followed by a `^exam-tip` line, embedded in `Exam-Tips.md`.
- A pasted image (a course slide or screenshot) is copied unedited into `assets/` as a PNG (convert WebP or JPG), named with dashes and a topic prefix (`Tool-Use-…`, `RAG-…`). Embed it with a relative Markdown image link, not `![[…]]`, whose alt text says what the image shows: `![What the slide shows](../assets/RAG-Vector-Database.png)`.
- An interactive HTML diagram gets a PNG snapshot with the same name, embedded like any image, with a link to the HTML under it. Obsidian can't run HTML inside a note, so the link opens it in the browser.
- Obsidian is often open. After editing files in `.obsidian/`, reload it with "Reload app without saving".
