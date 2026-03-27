# Initial Language Parsing — Summary of Findings

## Overview

This document captures the problems encountered and solutions applied while building the DocMonster Template Syntax VS Code extension. The core challenge is tokenizing DocMonster's custom `{{ }}` delimiters with embedded C# inside an HTML document, where C# curly braces `{ }` conflict with the template's `}} ` closing delimiter.

---

## Problem 1: `source.cs` Nested Block Scopes Swallow `}}`

### Symptom
Multi-line `{{% ... }}` code blocks broke syntax highlighting. The `}}` end delimiter was never matched, causing all subsequent HTML to lose coloring — the rest of the document stayed in C# scope.

### Root Cause
The built-in VS Code C# TextMate grammar (`source.cs`) contains numerous **begin/end pattern pairs** that create nested scopes. Once inside a nested scope, only that scope's `end` pattern is checked — the parent rule's `end` pattern (`}}`) is **never evaluated** until the nested scope closes.

Critical `source.cs` patterns that caused this:

| Pattern | Begin | End | Problem |
|---------|-------|-----|---------|
| `#block` | `\{` | `\}` | A C# `{` opens a block scope that consumes one `}` from `}}`, preventing the delimiter from matching as a two-character sequence |
| `#initializer-expression` | `\{` | `\}` | Same brace-swallowing behavior |
| `#assignment-expression` | `=(?!=\|>)` | `(?=[,);;\}])` | An `=` in `Script.Layout = "_layout.dmhtml";` opens a scope that looks for `;` or `}` — if the `;` is followed by content before `}}`, the scope may not close cleanly |
| `#if-statement` | `\b(if)\b\s*(?=\()` | `(?<=\})\|(?=;)` | Opens on `if(...)` and waits for `}` or `;`, potentially spanning across `}}` |
| `#parenthesized-expression` | `\(` | `\)` | Creates nested scope for parenthesized content |
| `#script-top-level` | (dispatches) | (various) | Top-level script context creates statement/declaration scopes |

The fundamental issue: **TextMate's begin/end scoping is greedy and hierarchical**. A child scope's `end` must match before the parent's `end` is ever tried. If a child scope opens (e.g., `#block` on `{`) and doesn't find its `end` (because it's consumed by `}}`), the parent `}}` end is never reached.

### Example of the failure
```html
{{% 
    var theme = Project.Settings.RenderTheme;    ← `=` opens #assignment-expression
    if(Topic.TopicState.IsPreview) { }}          ← `{` opens #block, `}` closes #block,
                                                    second `}` closes #assignment, 
                                                    but now `}}` is consumed piecewise
```

Even simpler cases failed:
```html
{{% Script.Layout = "_layout.dmhtml"; }}   ← `=` opens #assignment-expression scope
```
The assignment's end pattern `(?=[,);;\}])` would match at `;`, but the remaining content and `}}` would fall into further `source.cs` patterns.

### Solution: Custom C# Tokenization (`#csharp-code`)
Replaced `include: source.cs` with a custom `#csharp-code` repository entry that uses **only `match` patterns** (no begin/end) for all C# constructs except strings and comments:

- **Keywords**: single `match` regex for all C# keywords
- **Types**: `match` for built-in type names
- **Literals**: `match` for numbers, booleans, `null`
- **Operators**: `match` for all operator sequences
- **Punctuation**: `match` for `{`, `}`, `()[]`, `.`, `;`, `,`
- **Curly braces**: `\{` matched as simple `punctuation.curlybrace.open.cs`, and `\}(?!\})` matched as `punctuation.curlybrace.close.cs` (negative lookahead ensures a lone `}` not followed by another `}` is consumed, while `}}` remains available for the end delimiter)
- **Strings/comments**: bounded `begin/end` patterns that can never span past `}}`:
  - Regular strings: `"..."` with end at `"` or end-of-line
  - Verbatim strings: `@"..."` with end at `"` not followed by `"`
  - Interpolated strings: `$"..."` with end at `"` or end-of-line
  - Char literals: `'...'` with end at `'` or end-of-line
  - Line comments: simple `match` for `//.*$`
  - Block comments: `/* ... */` begin/end

This eliminates **all** nested block-level scoping, so the parent rule's `end` pattern (`}}`) is always checked at every character position.

---

## Problem 2: Injection Grammar Timing / First-Match Failure

### Symptom
The **first** multi-line code block in any file always broke, regardless of its content. Moving a different code block to the top position caused _that_ block to fail instead.

### Root Cause
The delimiter patterns were defined **only** in the injection grammar (`docmonster.injection.tmLanguage.json`). The root grammar (`docmonster.tmLanguage.json`) contained only `include: text.html.basic`. 

VS Code injection grammars use a lazy-loading mechanism — the injection may not be fully active when the tokenizer processes the first match position. This meant the very first code block was tokenized by `text.html.basic` (which has no DocMonster awareness), leaving `{{%` as unrecognized content that broke subsequent parsing. All further blocks worked because the injection was loaded by then.

### Solution: Patterns in Root Grammar
Moved all four delimiter patterns into the **root grammar's** `patterns` array, listed **before** `text.html.basic`:

```json
"patterns": [
    { "include": "#docmonster-code-block" },
    { "include": "#docmonster-raw-expression" },
    { "include": "#docmonster-encoded-expression" },
    { "include": "#docmonster-expression" },
    { "include": "text.html.basic" }
]
```

The injection grammar still exists but now references the root grammar's repository entries (`text.html.docmonster#docmonster-code-block`, etc.) so the patterns also apply inside HTML sub-scopes (attributes, script blocks) where the root patterns don't reach.

---

## Current Architecture

### File: `docmonster.tmLanguage.json` (root grammar)
- **Scope**: `text.html.docmonster`
- Contains all four delimiter rules with `#csharp-code` patterns
- Includes `text.html.basic` last for HTML tokenization
- Contains the full `#csharp-code` repository with safe match-only patterns

### File: `docmonster.injection.tmLanguage.json` (injection grammar)
- **Scope**: `docmonster.injection`
- **Selector**: `L:text.html.docmonster` (left/high priority)
- References root grammar's repository entries to inject into HTML sub-scopes
- No duplicate pattern definitions

### File: `package.json`
- Declares `embeddedLanguages` mapping on **both** grammars:
  - `meta.embedded.inline.csharp` → `csharp`
  - `meta.embedded.block.csharp` → `csharp`
- This enables bracket matching, comment toggling, and other editor behaviors to use C# rules inside delimiter regions

---

## Key Design Constraints

1. **No `source.cs` include inside delimiter regions** — its nested begin/end patterns are fundamentally incompatible with a `}}` end delimiter.
2. **Curly brace `}` needs negative lookahead** — `\}(?!\})` ensures a lone `}` is consumed as C# punctuation, while `}}` remains for the end delimiter to match.
3. **String patterns must be line-bounded** — regular and interpolated strings use `"|(?=$)` as their end to prevent runaway across lines if a quote is inside `}}`.
4. **DocMonster patterns must precede HTML** — listed before `text.html.basic` in root patterns for priority and reliable first-match behavior.
5. **Injection grammar still needed** — for delimiter recognition inside HTML attribute values and other nested HTML scopes where root-level patterns don't reach.

---

## Trade-offs

- **Less granular C# highlighting**: The custom `#csharp-code` patterns don't distinguish method names, class names, generic parameters, LINQ clauses, etc. with the same granularity as `source.cs`. Keywords, types, operators, strings, and numbers are highlighted, which covers the vast majority of template code.
- **No semantic C# features**: No IntelliSense, Roslyn diagnostics, or rename support inside template regions. This would require a language server with virtual document projection.
