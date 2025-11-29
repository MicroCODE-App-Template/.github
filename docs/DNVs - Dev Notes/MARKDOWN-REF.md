# Markdown Syntax Reference

Complete reference guide for Markdown syntax and formatting.

## Headings

| Syntax   | Level     | Example          | Output              |
| -------- | --------- | ---------------- | ------------------- |
| `#`      | Heading 1 | `# Title`        | <h1>Title</h1>      |
| `##`     | Heading 2 | `## Section`     | <h2>Section</h2>    |
| `###`    | Heading 3 | `### Subsection` | <h3>Subsection</h3> |
| `####`   | Heading 4 | `#### Detail`    | <h4>Detail</h4>     |
| `#####`  | Heading 5 | `##### Minor`    | <h5>Minor</h5>      |
| `######` | Heading 6 | `###### Small`   | <h6>Small</h6>      |

## Text Formatting

| Syntax       | Purpose       | Example       | Output      |
| ------------ | ------------- | ------------- | ----------- |
| `*text*`     | Italic        | `*italic*`    | _italic_    |
| `_text_`     | Italic (alt)  | `_italic_`    | _italic_    |
| `**text**`   | Bold          | `**bold**`    | **bold**    |
| `__text__`   | Bold (alt)    | `__bold__`    | **bold**    |
| `***text***` | Bold italic   | `***both***`  | **_both_**  |
| `~~text~~`   | Strikethrough | `~~deleted~~` | ~~deleted~~ |

## Code

| Syntax       | Purpose                  | Example          | Output              |
| ------------ | ------------------------ | ---------------- | ------------------- |
| `` `code` `` | Inline code              | `` `code` ``     | `code`              |
| ` ``` `      | Code block               | ` ``` `          | (code block)        |
| ` ```lang`   | Code block with language | ` ```javascript` | (highlighted block) |

## Lists

| Syntax  | Purpose              | Example      | Output     |
| ------- | -------------------- | ------------ | ---------- |
| `- `    | Unordered list       | `- item`     | • item     |
| `* `    | Unordered list (alt) | `* item`     | • item     |
| `1. `   | Ordered list         | `1. item`    | 1. item    |
| `- [ ]` | Unchecked task       | `- [ ] todo` | - [ ] todo |
| `- [x]` | Checked task         | `- [x] done` | - [x] done |

**Note:** Indent with 2 spaces for nested items.

## Links and Images

| Syntax        | Purpose         | Example            | Output        |
| ------------- | --------------- | ------------------ | ------------- |
| `[text](url)` | Link            | `[link](url)`      | [link](url)   |
| `[text][ref]` | Reference link  | `[link][1]`        | [link][1]     |
| `[ref]: url`  | Link definition | `[1]: https://...` | (defines ref) |
| `<url>`       | Autolink        | `<https://...>`    | <https://...> |
| `![alt](url)` | Image           | `![alt](img.jpg)`  | Image         |
| `![alt][ref]` | Reference image | `![alt][1]`        | Image         |

## Blockquotes

| Syntax | Purpose           | Example      | Output     |
| ------ | ----------------- | ------------ | ---------- |
| `> `   | Blockquote        | `> quote`    | > quote    |
| `> > ` | Nested blockquote | `> > nested` | > > nested |

## Horizontal Rules

| Syntax | Purpose               | Example | Output |
| ------ | --------------------- | ------- | ------ |
| `---`  | Horizontal rule       | `---`   | ---    |
| `***`  | Horizontal rule (alt) | `***`   | ---    |
| `===`  | Horizontal rule (alt) | `===`   | ---    |

## Tables

| Syntax     | Purpose          | Example     | Output           |
| ---------- | ---------------- | ----------- | ---------------- |
| `\|`       | Column separator | `\| col \|` | Table cell       |
| `\|-\|-\|` | Table separator  | `\|-\|-\|`  | Table header row |
| `:---:`    | Center align     | `:---:`     | Center column    |
| `:---`     | Left align       | `:---`      | Left column      |
| `---:`     | Right align      | `---:`      | Right column     |

**Example:**

```markdown
| Header 1 | Header 2 | Header 3 |
| :------- | :------: | -------: |
| Left     |  Center  |    Right |
| Data     |   Data   |     Data |
```

## Footnotes

| Syntax  | Purpose             | Example      | Output   |
| ------- | ------------------- | ------------ | -------- |
| `[^1]`  | Footnote reference  | `text[^1]`   | text[^1] |
| `[^1]:` | Footnote definition | `[^1]: note` | Footnote |

## Other Elements

| Syntax     | Purpose            | Example           | Output         |
| ---------- | ------------------ | ----------------- | -------------- |
| `<br>`     | Line break         | `line<br>break`   | Line<br>break  |
| `<!-- -->` | HTML comment       | `<!-- hidden -->` | (hidden)       |
| `\`        | Escape character   | `\*not italic\*`  | \*not italic\* |
| `&nbsp;`   | Non-breaking space | `word&nbsp;word`  | word word      |

## Common Extensions

| Syntax         | Purpose         | Example        | Notes           |
| -------------- | --------------- | -------------- | --------------- |
| `$formula$`    | Inline math     | `$E=mc^2$`     | LaTeX math      |
| `$$formula$$`  | Block math      | `$$\int_0^1$$` | LaTeX math      |
| `:emoji:`      | Emoji           | `:smile:`      | GitHub Flavored |
| ` ```mermaid ` | Mermaid diagram | ` ```mermaid ` | Diagram syntax  |

## Best Practices

- Use `#` for document title (one per document)
- Use `##` for major sections
- Use `###` and below for subsections
- Keep line length reasonable (80-100 chars)
- Use code blocks for multi-line code
- Use reference-style links for readability in source
- Escape special characters when needed: `\*`, `\[`, etc.

## Notes

- **Headings:** Use 1-6 `#` symbols for hierarchy
- **Emphasis:** `*` or `_` for italic, `**` or `__` for bold
- **Lists:** Indent with 2 spaces for nested items
- **Tables:** Align with `:---:` (center), `:---` (left), `---:` (right)
- **Code blocks:** Add language after opening ` ``` ` for syntax highlighting
- **Markdown flavors:** CommonMark, GitHub Flavored Markdown, etc. may have variations
