---
name: task-plan
description: Plan and create project tasks as inline Markdown for Abyss Tasks. Use when the user asks for a project work plan, roadmap, stages, milestones, tasks with IDs and dependencies, or tasks that must appear in Abyss Tasks. Handles 🆔 task IDs, ⛔ blocked-by links, task types, categories, priorities, dates, and acceptance criteria. Triggers include "план работ", "задачи для проекта", "поставь задачи", "roadmap", "milestones", "Abyss Tasks", "зависимости задач". Use this instead of task-inline when the tasks belong to a project note. For quick GTD capture in daily notes use task-inline; for metadata task notes in base/tasks use task-note.
---

# Project Task Planning for Abyss Tasks

A project task plan is a set of inline Markdown tasks in the project note body. Abyss Tasks reads them directly, so the project note is the single source of truth: no separate task notes are needed.

Use this skill when the user wants a work plan for a project and the result must be visible in Abyss Tasks: kanban, timeline, IDs, dependencies, blocked states.

## Where tasks live

- Project note: `projects/<Title>.md`, with `tags: project/...` in frontmatter.
- Tasks: under `# Tasks` in the project body, grouped by stage headings.
- Keep one task per line. Put details and acceptance criteria as plain text below the list, not as extra checkboxes.

Abyss Tasks recognizes the project note because its tags are under `#project` (`project/single`, `project/longform`, etc.). It parses the inline checkboxes in the body.

## Task line format

```markdown
- [ ] #task/TYPE #category/CATEGORY Description 🆔 ID ⛔ DEP1, DEP2 🔺 📅 YYYY-MM-DD
```

Include only the parts the task needs. A minimal task is:

```markdown
- [ ] #task/one-off #category/computer_science Собрать требования 🆔 PRJ-1
```

### IDs

`🆔` is the task ID marker. Abyss Tasks accepts IDs matching `[A-Za-z0-9_-]+`.

- Use ASCII only. `ЕКП-4` looks fine to a human but the plugin will not parse a Cyrillic ID; `EKP-4` will.
- Pick a short project prefix and number sequentially: `EKP-4`, `EKP-5`, ...
- Check existing tasks before adding new ones. IDs are used globally by the plugin, and duplicates make dependencies ambiguous.
- Never invent an ID for an existing task; read the current project note first.

### Dependencies

`⛔` means "this task cannot start before the referenced task ends".

```markdown
- [ ] ... Разметить книгу для RAG 🆔 EKP-5 ⛔ EKP-4
- [ ] ... Собрать сценарий n8n 🆔 EKP-10 ⛔ EKP-6, EKP-9
```

- Reference IDs, not titles.
- Multiple blockers are comma-separated after one `⛔`.
- A referenced ID must exist. A missing ID shows as an unavailable dependency in Abyss Tasks.
- Add dependencies after the blocker lines exist, so you can verify every ID.

### Task types and categories

Task types come from the vault's GTD statuses in `periodic/statuses/`:

- `#task/next_action` — the task to start with.
- `#task/one-off` — a single action.
- `#task/multistep` — a task that has subtasks.
- Other types: `inbox`, `waiting_for`, `regular`, `idea`, `reference`, `someday`.

Categories come from `base/categories/`. Convert the file name to a tag: `computer science.md` → `#category/computer_science`. Never invent a category; if none fits, omit the tag or ask.

### Priorities and dates

Priority markers trail the description, before any date suffix:

`🔺` highest, `⏫` high, `🔼` medium, `🔽` low, `⏬` lowest. No marker means normal priority.

Date markers:

- `📅` due
- `⏳` scheduled
- `🛫` start
- `➕` created
- `✅` completion
- `❌` cancelled
- `⏰` time
- `⏱️` duration
- `🏁` on-completion

Only add dates the user gave. Do not invent deadlines or priorities.

### Stages

Abyss Tasks has no separate milestone entity. Use headings to group tasks into stages:

```markdown
## 📍 Книга готова к RAG
- [ ] ...
- [ ] ...

## 📍 Прототип работает
- [ ] ...
```

The emoji is a visual marker; project views group tasks by heading.

## Workflow

1. Read the project note. Inspect the existing `# Tasks` section and collect used IDs.
2. Define the result of the project and 2–4 stages. A stage is a checkpoint, not a task.
3. Choose an ASCII prefix and number the tasks. Start with `#task/next_action` for the first actionable task; use `#task/one-off` for the rest.
4. Write task lines with type, category, ID, and a clear result in the description. Add dependencies after all blockers have IDs.
5. Add acceptance criteria as text under the task list. One line per ID, for example:
   ```markdown
   ## Критерии готовности
   - **EKP-4** — текст книги и задания собраны в файлы.
   ```
6. Verify with the Obsidian CLI:
   ```bash
   obsidian tasks todo format=json file="<Project>"
   ```
   Then open the project in Abyss Tasks and check that dependencies resolve and blocked tasks show as blocked.
7. Do not create `base/tasks` notes for the same plan unless the user explicitly asks. Abyss Tasks ignores `#task/default` by default, so the two systems would not see each other.

## Example

```markdown
# Tasks

## 📍 Подготовка

- [ ] #task/next_action #category/computer_science Собрать требования 🆔 PRJ-1
- [ ] #task/one-off #category/computer_science Согласовать объём 🆔 PRJ-2 ⛔ PRJ-1

## 📍 Реализация

- [ ] #task/one-off #category/computer_science Собрать прототип 🆔 PRJ-3 ⛔ PRJ-2

## Критерии готовности

- **PRJ-1** — требования собраны.
- **PRJ-2** — объём согласован.
- **PRJ-3** — прототип работает.
```

## Pitfalls

- **Cyrillic ID.** `ЕКП-4` is not parsed; use `EKP-4`.
- **Duplicate ID.** Two tasks with the same `🆔` make dependencies ambiguous.
- **Dependency on a missing ID.** The plugin shows the task as blocked by an unavailable task.
- **`#task/default` in an inline task.** That tag belongs to metadata task notes; the plugin's ignore query excludes it.
- **Mixing inline tasks and task notes.** Pick one system for a project. This skill uses inline tasks.
- **Acceptance criteria as checkboxes.** `- [ ]` creates another task; use plain bullets.
- **Invented dates or priorities.** Add them only when the user gave them.

## Optional dependency check

The plugin UI is the source of truth. For a quick programmatic check:

```bash
obsidian eval code="(function(){const p=app.plugins.plugins['abyss-tasks'];return p.queries.listNodes().filter(x=>(x.target.ref?.filePath||'').includes('<Project>')).map(x=>({title:x.node.title,id:x.node.dependencyId,deps:x.node.dependsOn}));})()"
```

This uses plugin internals. If it breaks after a plugin update, fall back to `obsidian tasks` and the Abyss Tasks project view.
