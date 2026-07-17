---
name: toon
description: >
  TOON (Token-Oriented Object Notation) format specifications skill, a compact, lossless encoding of JSON.
  TOON is JSON replacement for token-efficient structured data in LLM prompts.
  Use this to write data in TOON, encodes JSON to TOON, decodes TOON to JSON.
  Triggers on: .toon files, format conversion, "TOON", "token-oriented", "token efficiency", "compact prompt".
author: fityannugroho
license: MIT
---

# TOON

Compact JSON encoding for LLM prompts. `.toon` extension, spec v3.1, lossless round-trip. Root forms: object (`key: value`), array (`[N]:` or `[N]{fields}:`), or primitive (bare value).

## Objects

```
key: value
nested:
  child: 1
```
Colon + 1 space. Nested = default 2-space indent. Empty: `key:`

## Arrays

Format: `key[N<delim?>]{fields}:` — `[N]` is exact count, `{fields}` for tabular.

```
# Inline primitive array
scores[3]: 10,20,30

# Tabular — header declares fields, each row is one element
users[2]{id,name,role}:
  1,Alice,admin
  2,Bob,user

# List items — objects as items
contributors[2]:
  - name: Alice
    age: 30
  - name: Bob
    age: 25

# Mixed / non-uniform
mixed[3]:
  - string entry
  - 42
  - keyed: value

# Arrays of arrays
matrix[2]:
  - [2]: 1,2
  - [2]: 3,4

# Empty
tags: []
```

### Tabular first field in list item
When first field is tabular, header on hyphen line, rows at depth+2:
```
products[2]:
  - variants[2]{size,color}:
      M,red
      L,blue
    status: active
```

### Root array
```
[3]: x,y,z
```

## Delimiters

| Delimiter | Bracket | Notes |
|-----------|---------|-------|
| Comma | `[N]` | Default |
| Tab (HTAB) | `[N\t]` | Best token efficiency |
| Pipe | `[N|]` | Good when data has commas |

Declaration bracket → active delimiter, scoped to that array. Nested headers may override.

## Quoting Rules

String MUST be quoted when:
- Empty / leading or trailing whitespace
- Equals `true`, `false`, `null` (case-sensitive)
- Numeric-like: `42`, `-3.14`, `05`, `1e-6`
- Contains `:`, `"`, `\`, `[`, `]`, `{`, `}`, control chars
- Contains the relevant delimiter: active delimiter inside array scope, document delimiter (comma by default) for object `key: value` fields
- Starts with `-`

Valid escapes only: `\\`, `\"`, `\n`, `\r`, `\t`, `\uXXXX` (4 hex digits for control chars U+0000–U+001F). Others (e.g. `\x`) → error.

## Key Folding (v1.5+)

Collapses single-key chains into dotted paths:
```
a.b.c: 42     # instead of a: {b: {c: 42}}
```
`keyFolding: 'safe'`, `flattenDepth` (default Infinity). Segments must match `^[A-Za-z_][A-Za-z0-9_]*$`. Decoder: `expandPaths: 'safe'` for round-trip.

## Strict Mode (default ON)

- Array count/width must match `[N]` and field list
- Indentation: multiples of indentSize (default 2)
- No blank lines inside arrays, no tab indentation
- Delimiter consistency within scope

## Additional Resources

- **Syntax reference**: `reference/syntax.md` — grammar, quoting edge cases, strict mode checklist
- **Examples**: `examples/basic.toon`, `tabular.toon`, `advanced.toon`
- **Full spec**: https://github.com/toon-format/spec
