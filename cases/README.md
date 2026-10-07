# Case files

This directory is the test corpus for the cnf implementations,
next to the spec (`../README.md`) they exercise. Every `*.md`
file (except this README) is a topic file to be read by the
test harness.

## File layout

A topic file is one concept, holding one or more cases ordered
from most basic to most advanced:

```
# Topic

<topic summary - prose, ignored by the harness>


## Case: <case name>

<case summary - prose, ignored by the harness>

<directive block>

### cnf

<fenced block>

[<fenced block>]

...

### json

<fenced block>

[<fenced block>]

...
```

Two blank lines separate cases; one blank line separates every
header (`# Topic`, `## Case: <name>`, `### cnf`, `### json`,
`### ast`) from the first non-empty line after it. The case name
respects cnf's bare word convention and is unique within its
topic; the harnesses use it as the subtest name.

## Directives

The directive block is a fenced block of `key: "value"` pairs,
before any `###` section. The value is a cnf double-quoted
string.

### Fields

- `to-json` - the direction: translating each cnf block to
  json, verified against the first json block.
- `from-json` - the reverse direction: translating the json
  block to cnf, verified against the first cnf block.
- `to-ast` - parsing each cnf block into the AST form, verified
  against the first ast block.
- `from-ast` - decoding the ast block and rendering it as cnf,
  verified against the first cnf block.
- `error: "..."` - the expected message, shared by every error
  direction of the case: the literal value for exact
  comparison, or the `{regexp: "..."}` map value for a
  pattern matched against the message.
- `vars:` - a nested block of variables. See the
  Variables section for the value syntax and the reference
  semantics.
- `reformat:` - a nested block of reformat settings, one
  `name: value` pair per line. Reformatting is optional for
  comformance, so every case including it may be skipped.
- `parse-ast: true` - the ast blocks are spelled as cnf, not in
  the AST form, usually for readability in reformatting tests.
  The harness converts them to the AST at runtime before running
  the rest of the case as usual.

### Matching

The value of a direction names its matching operator:

- `"advised-text"` - it is recommended that the output
  matches the target block's text exactly (after trimming one
  trailing newline; the json target is also compacted).
  For outputs that are strong recommendations in terms of
  structure, besides the value. An implementation that cannot
  follow the exact representation is still conformant if the
  value matches exactly and the alternative text is simply a
  reordering or a correct different spelling of the same value.
- `"exact-value"` - same general semantics as `"advised-text"`,
  but with an additional upfront acknowledgement that the text
  is not stable, so value matching is recommended.
- `"advised-value"` - unlike the former operators, this is a
  looser recommendation of a valid interpretation for the data.
  It's typically used with `to-ast`, because the spec makes no
  attempt to prescribe rules about it. The ASTs in the
  conformance cases are used by some implementations in support
  of correctness.
- `"error"` - the implementation _must_ fail in those
  cases, and it is advised to use the error message provided in
  the `error` field: a literal message, or - anchored on both
  ends, `^` at the front and `$` at the back - a regular
  expression matched against the whole message. A pattern
  anchored on one end only is a broken case.

A directive may be skipped if absent or when the particular
implementation inherently cannot handle the operation. For example,
the `to-ast` operation is only possible if the implementation
chooses to support that specific representation, at least for
additional verification in test cases.

### Variables

The `vars:` field opens a nested block of variables, one
`name: value` pair per line where names are `[a-z]+`.

**Value syntax:** The value may be defined as:

- A quoted string is a literal representing its plain text.
- A single-key map `{regexp: "..."}` is a pattern, compiled
  and matched as a regexp. The actual text and position is
  of the variable is the match of the _first group,_ not
  the whole match.

The quoted strings support the `\n`, `\t`, `\"`, `\\`,
`\uXXXX` and `\UXXXXXXXX` escapes (four hex digits for the
BMP, eight for the astral planes), so a case can name a raw
rune without the file holding it.

**References:** Error messages and format code blocks may
reference variables:

- `${name}` - the var's text
- `${name:l}` - its 1-based line
- `${name:c}` - its 1-based rune column
- `${name:lc}` - the two joined "line:col"

The plain form `${name}` is the var's text directly and the
position forms represent the given information for the var
text.


## Blocks

`### cnf` and `### json` each hold zero or more fenced blocks:

```
[<fenced block>]

...
```

For `to-json`, every cnf block is verified against the first
json block; for `from-json`, every json block against the
first cnf block.

A section may exist only when a directive compares against it:
the cnf blocks are the `to-json` inputs (any kind) and the
`from-json` expectations (non-error only); the json blocks the
reverse. A section no directive reads is a case-file error.

## Conventions

- The json block may be pretty-printed; the harness must compact
  it before comparing against the implementation's output.
- Case summaries are one line, stating the behavior pinned.
- Invisible whitespace in cnf content (trailing spaces, mixed
  indent prefixes) is real test data - preserve it byte-exactly
  when editing.
- `TODO` notes in summaries mark open review questions and are
  ignored by the harnesses.
