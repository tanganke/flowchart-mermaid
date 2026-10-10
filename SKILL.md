---
name: flowchart-mermaid
description: Draw a flowchart, pipeline, dependency graph or decision tree in the chat as a Mermaid diagram. Use when the user asks for a flowchart, process diagram, structure overview, or "show me how X flows".
---

# Flowcharts with Mermaid

The chat renders a fenced code block tagged `mermaid` as a diagram (Mermaid 12, light and dark theme follow the app). Use it to show a process or structure that is clearer as a picture than as a list.

## When to draw

Draw when the content is a sequence with branches, a pipeline, a dependency graph, or a decision tree: a paper's argument flow, the `\input`/`\include` tree of the project, the compile pipeline, an experiment procedure, a rebuttal plan, a proof's lemma dependencies.

Do not draw a straight list of three steps, a table, or anything the user can read faster as a sentence. One diagram per answer unless the user asks for more; follow it with two or three sentences saying what to look at.

## Ground it in the project

Never invent structure. Before drawing the file tree, the section flow, or the compile chain, read the real files (`list_files`, `grep` for `\input`, `\include`, `\section`, `\cite`, `\ref`) and draw what is there. Take node labels from real names (section titles, file paths, macro names). If something is unclear, leave it out or mark it with a `?` label instead of guessing.

## Output format

- Write exactly one fenced block per diagram, opened with ```` ```mermaid ```` and **closed with a line of three backticks**. The chat draws the diagram only after the closing fence arrives; an unclosed fence stays plain code.
- Put nothing but Mermaid source inside the fence: no prose, no `%%{init: ...}%%` directive, no frontmatter.
- Never put the diagram inside a project file unless the user asks. If they want it in the paper, say that `.tex` does not render Mermaid and offer a TikZ version (`tikz` with `positioning`, `arrows.meta`) as a separate step.

## Syntax rules (the renderer runs in `strict` security mode)

- Start with `flowchart TD` (top-down, for processes and trees) or `flowchart LR` (left-right, for short pipelines). Use `flowchart`, not the older `graph`.
- Node ids are short ASCII: `A`, `build`, `s1`. Put the human text in the label: `build["Compile with latexmk"]`.
- **Always double-quote labels** that contain spaces, CJK text, punctuation, parentheses, backslashes, `%`, `#`, or `&`: `A["\input{sections/intro}"]` fails unquoted, so write `A["input: sections/intro.tex"]`. Prefer not to write a literal backslash at all.
- Line break inside a label: `<br/>`. Do not use other HTML.
- Shapes: `A[rect]`, `B(rounded)`, `C{decision}`, `D([start or end])`, `E[(store)]`, `F[[subroutine]]`. Use `{}` only for real yes/no decisions, and label both outgoing edges: `C -->|yes| D`, `C -->|no| E`.
- Edge text: `A -->|label| B`. Dotted for optional or indirect: `A -.-> B`. Thick for the main path: `A ==> B`.
- Group related nodes with `subgraph id["Title"]` ... `end`. Do not name a node or subgraph id `end`, `graph`, `subgraph`, `style`, or `class`.
- Labels follow the user's language. If the user writes Chinese, write Chinese labels (quoted).
- Do not use `click`, `href`, `callback`, `<script>`, `<a>`, or images: strict mode drops them.
- Do not set fill, stroke, or font colors with `style`/`classDef`. The theme handles dark mode and a hard-coded light fill becomes unreadable there. If one node must stand out (the failing step, the entry point), use a single shared `classDef` with a stroke width only, or a different shape.

## Size and legibility

- At most about 20 nodes and 3 levels of nesting. Above that, split into an overview plus one diagram per part, or collapse a group into one node and offer to expand it.
- Labels under about 40 characters. Move detail into the text after the diagram.
- Keep edges from crossing where you can: order nodes by flow, and prefer `TD` for tall trees and `LR` for wide pipelines.
- A long chain of more than 8 nodes in `LR` gets too wide to read: switch to `TD` or group it.

## Self-check before sending

1. First line is `flowchart TD` or `flowchart LR`.
2. Every label with special characters is in double quotes.
3. Every `subgraph` has its `end`; every decision node has labeled branches.
4. No node id collides with a reserved word, and no id is reused for two different nodes.
5. The fence is closed.

If the user reports that the diagram shows as code, it has a syntax error: fix the quoting or the reserved word and send the corrected block. Do not retry with a different diagram type, and do not describe a diagram you cannot see as if it rendered.

## Example

User: "Show me how this project compiles."

```mermaid
flowchart LR
  main["main.tex<br/>(root file)"] --> intro["sections/intro.tex"]
  main --> method["sections/method.tex"]
  main --> bib[("refs.bib")]
  intro --> fig1["figures/overview.pdf"]
  method --> fig2["figures/arch.pdf"]
  main --> latexmk{"latexmk succeeds?"}
  latexmk -->|yes| pdf(["main.pdf"])
  latexmk -->|no| log["read main.log, fix first error"]
  log --> latexmk
```

The root file pulls in two sections and the bibliography; the compile loops until the first error in the log is gone.
