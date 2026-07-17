# TOON Syntax Reference

## Header Grammar (ABNF)
```
bracket-seg   = "[" 1*DIGIT [ delimsym ] "]"
delimsym      = HTAB / "|"
fields-seg    = "{" fieldname *( delim fieldname ) "}"
delim         = delimsym / ","
fieldname     = key

header        = [ key ] bracket-seg *SP [ fields-seg *SP ] ":"
key           = unquoted-key / quoted-key
unquoted-key  = ( ALPHA / "_" ) *( ALPHA / DIGIT / "_" / "." )
quoted-key    = DQUOTE *quoted-char DQUOTE
```

## Delimiter Scoping Rules
- **Document delimiter**: comma (default), used for all object field values (`key: value`)
- **Active delimiter**: declared by nearest array header. Scoped to that array only.
- Nested headers may change the active delimiter for their scope.
- Object field values inside an array scope still use document delimiter.

## Quoting Edge Cases
| Case | Rule |
|------|------|
| String with leading/trailing space | MUST quote |
| String = true/false/null | MUST quote (case-sensitive) |
| Numeric-like (`42`, `-3.14`, `05`, `1e-6`) | MUST quote |
| Contains `:`, `"`, `\`, `[`, `]`, `{`, `}` | MUST quote |
| Contains control chars (U+0000-U+001F) | MUST quote |
| Contains active delimiter (comma/tab/pipe) | MUST quote |
| String equals or starts with `-` | MUST quote |
| Empty string | MUST quote (`""`) |

### Escape Sequences
| Escape | Character |
|--------|-----------|
| `\\` | Backslash (U+005C) |
| `\"` | Double quote (U+0022) |
| `\n` | Newline (U+000A) |
| `\r` | Carriage return (U+000D) |
| `\t` | Tab (U+0009) |
| `\uXXXX` | Any control char U+0000–U+001F (4 hex digits) |

## Key Folding Rules
- Mode: `"off"` | `"safe"` (default: off)
- Only folds chains where each segment is an `IdentifierSegment`: `^[A-Za-z_][A-Za-z0-9_]*$`
- Chain stops at arrays, non-single-key objects, or leaf values
- `flattenDepth`: max segments to fold (default Infinity)
- Collision avoidance: folded key must not equal existing sibling literal key

## Path Expansion (Decoder)
- Mode: `"off"` | `"safe"` (default: off)
- Split dotted key at `.`, expand only if ALL segments are IdentifierSegments
- Deep merge semantics for overlapping paths
- Strict mode: conflict = error. Non-strict: last-write-wins

## Strict Mode Checklist
- [ ] Array count match: decoded elements == `[N]`
- [ ] Tabular row width: field count == header field count
- [ ] Indentation: multiples of indentSize (default 2)
- [ ] No blank lines inside arrays
- [ ] No tab characters for indentation
- [ ] Delimiter consistency within same array scope
- [ ] Valid escape sequences only
- [ ] Missing colon after key = error
