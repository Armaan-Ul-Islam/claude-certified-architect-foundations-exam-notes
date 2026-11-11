# CCA-F Study Vault

Obsidian vault of study notes for the Claude Certified Architect – Foundations (CCA-F) exam. Start at `Home.md`.

## Layout
- `Concepts/`: one topic per note.
- `Domains/`: the 5 exam domains. Each task statement is a heading listing the notes that cover it.
- `Scenarios/`: the 6 exam scenarios, plus `Exam-Scenarios.md` on telling them apart.
- `Reference/`, `Resources/`: technology reference; exam guide, courses and links.
- `assets/`: every image and PDF, including the official exam guide.
- `Study-Tracker.base`, `Exam-Map.canvas`, `Exam-Tips.md`: views built from the notes.

## Conventions
- File names use dashes, never spaces.
- Link with `[[wikilinks]]`. Link a task as `[[Domain-N-…#N.M <task statement>|Task N.M]]` and a scenario as `[[Scenario-N-…|Scenario N]]`.
- A new concept note gets properties `type: concept`, `domain`, `tasks`, `tags`, `aliases`, `confident: false`, and a line under each of its task headings in the domain note.
- An exam tip is a `> [!tip] Exam tip` callout followed by a `^exam-tip` line, embedded in `Exam-Tips.md`.
- Obsidian is often open. After editing files in `.obsidian/`, reload it with "Reload app without saving".
