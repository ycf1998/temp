# MQ — jq-like Markdown Processor

**Usage**: Query, filter, transform Markdown files with jq-like syntax. Read-only by default; `-U` for in-place writes.

## Core Selectors

```bash
.h .h1..h6 .text .code .code_inline .link .image .list .blockquote .table .yaml .html .footnote .math
```

Selector calls: `.h(1)` `.h(2,3)` `.h(1..3)` `.code("rust")`

## Attribute Access & Update

```bash
.h.level / .h.depth       # Heading level (1-6)
.code.lang                 # Code language (r/w)
.code.value                # Code content (r/w)
.link.url                  # Link URL (r/w)
.list.checked              # Task checkbox (r/w)

mq 'select(.code.lang == "bash")' file.md              # Find bash code blocks
mq -U '.code.lang |= "bash"' file.md                   # Change all code lang
mq -U '.h.depth |= 2' file.md                          # Promote all headings
```

## Common Patterns

```bash
# Extract
mq '.code("rust")' file.md                             # Rust code blocks
mq '.link.url' file.md                                 # All URLs
mq 'select(contains("TODO"))' file.md                  # Lines with TODO
mq 'select(.h.level <= 2)' file.md                     # h1 and h2 only

# Transform
mq -F json '.h | to_text()' file.md                    # Headings → JSON array
mq -F html 'identity()' file.md                        # Markdown → HTML
mq -I html '.link.url' page.html                       # Extract links from HTML

# Multi-file
mq -A '.code' *.md                                     # All code blocks, aggregated
```

## Key Flags

| Flag | Purpose |
|------|---------|
| `-U` / `--update` | In-place edit (requires `|=` or `set_attr()`) |
| `-I` | Input format: markdown, html, json, yaml, csv, xml... |
| `-F` | Output format: markdown, json, html, text, table, grep, raw |
| `-A` / `--aggregate` | Treat all input files as one array |
| `-f` | Load query from file |
| `-C` / `--color-output` | Colorize output |
| `--stream` | Line-by-line for large files |

## Gotchas

- `to_text()` flattens AST → output loses structure. Use attribute access (`.code.lang`) instead.
- `gsub()` only on text nodes; combine with `-U` carefully.
- `|=` works on attributes, not on nodes directly: `.code.lang |= "x"` not `.code |= "x"`.
- No `|=` in JSON/HTML/csv mode — Markdown only.
