# The _cnf_ format - 1.0rc1

This document specifies **cnf**, a configuration file format
focused on human readability and correctness, and that has
universal support via _json_ 1-to-1 data roundtrip.

## WARNING

This specification is a _release candidate_, and is _about to become final!_

If you are here, it means you are an initial implementer or user of the format,
so please report issues timely since we are about to close it down.

## Contents

- [Overview](#overview)
  - [Filename extension](#filename-extension)
  - [Example](#example)
  - [Implementations](#implementations)
- [Background](#background)
  - [How is it different from yaml?](#how-is-it-different-from-yaml)
  - [How is it different from toml?](#how-is-it-different-from-toml)
  - [How is it different from json?](#how-is-it-different-from-json)
  - [How large is an implementation?](#how-large-is-an-implementation)
  - [Why a cnf-json bridge is recommended?](#why-a-cnf-json-bridge-is-recommended)
  - [Was AI used?](#was-ai-used)
- [Grammar](#grammar)
  - [Conformance](#conformance)
  - [Preface](#preface)
  - [Lexical](#lexical)
  - [Syntactic](#syntactic)
- [The format](#the-format)
  - [Top-level values](#top-level-values)
  - [Keys and paths](#keys-and-paths)
  - [Maps](#maps)
  - [Lists](#lists)
  - [Numbers](#numbers)
  - [Strings](#strings)
  - [Indentation](#indentation)
  - [Comments](#comments)
- [Authors](#Authors)
- [Appendix: Maintaining this document](#appendix-maintaining-this-document)

## Overview

### Filename extension

The filename suffix `.cnf` is used for files that hold content in this format.

### Example

```
key: "value"
nested.key: "another value"

map:
    inline: {a: 1, b: 2}

    block:
        a: 1
        b: 2

    block.c: 3

list:
    inline: [1, 2, 3]

    block.strings:
        - "item 1"
        - "item 2"

    block.maps:
        - label: "map 1"
        - label: "map 2"

    block.list-of-maps:
        ---
        list1.map1:
            a: 1
            b: 2
        list1.map2:
            c: 3
            d: 4
        ---
        list2.map1:
            a: 1
            b: 2
        ---

    block.list-of-lists:
        ---
        - "list 1 item 1"
        - "list 1 item 2"
        ---
        - "list 2 item 1
        ---

string:
    double: "line 1\nline 2\nline 3"

    # One line, no escaping, literal slash
    single: 'line 1\nline 1 still'

    # Three lines, with escaping support.
    fence.double.inline: """line 1\nLine 2\nline 3\n"""
    fence.double.block:
        """
        line 1\nline \
        2
        line 3
        """

    # One line, no escaping, literal slash
    fence.single.inline: '''line 1\nline 1 still'''
    fence.single.block:
        '''
        line 1\n
        line 2
        '''

numbers:
    int: 123
    float: 1.23
    hex: 0x123
    octal: 0o777
    binary: 0b011
    exponent: 1e-23
```

### Implementations

[Conformant](#conformance) implementations will be listed here:

- [go-cnf/cnfast](https://pkg.go.dev/github.com/canonical/go-cnf/cnfast) - Original AST parser and renderer built during development. 
- [go-cnf/cnfpretty](https://pkg.go.dev/github.com/canonical/go-cnf/cnfpretty) - Go pretty printer for cnfast. 
- [go-cnf/cnfjson](https://pkg.go.dev/github.com/canonical/go-cnf/cnfjson) - Go text-preserving bridge between cnf and json. 
- [go-cnf/cnfmap](https://pkg.go.dev/github.com/canonical/go-cnf/cnfmap) - Simple Marshaler and Unmarshaler for Go values. 
- [go-cnf/cnftest](https://pkg.go.dev/github.com/canonical/go-cnf/cnftest) - Go harness to run the conformance tests.


## Background

This section is non-normative: it outlines why _cnf_ looks the way it does. 

The specification itself follows below with the [grammar](#grammar) and [the format](#the-format).

### How is it different from yaml?

_Vastly,_ as becomes clear after a short walk through the yaml spec.

The author of _cnf_ wrote and maintained _go-yaml_ for over a decade, the most
popular yaml package for Go. The _cnf_ format implements what most people are
really looking for when they use yaml, without paying the price of what's left.
Then, even being a small specification, the _cnf_ format still manages to
improve the readability of the format in a few important areas.

### How is it different from toml?

Similar case to yaml: a short walk through the latest toml spec makes it clear
that it was designed towards flexibility and richness, unlike the approach
followed by cnf, which constrains for consistency, readability, correctness,
and json compatibility.

### How is it different from json?

Not different at all in the sense of the supported data, since _cnf_ was
intentionally designed to offer a 1-to-1 mapping with json so that it benefits
from universal support in languages and APIs from day one. The key changes are
all in the representation of the data, in support of human reading and typing.

### How large is an implementation?

As a data point, a single-file fully comformant Go implementation,
including proper line/column debugging information, takes ~1000 code lines.

### Why a cnf-json bridge is recommended?

Obviously, each implementation has the choice of implementing the spec
as desired, but there are clear advantages to a json translator instead
of a native encoder and decoder.

A proper native encoder and decoder in any language isn't just about reading
maps and ints. Most of the work is in making that format feel native to the
language: how to fit the typing system, how to handle defaults, how to
handle custom callbacks, and so on. Then, even if you do all of that, you'll
need to teach people about it, and you'll add weight to the application.
Meanwhile, every language on the planet has a proper json encoder and decoder
that the developers already know about, and there are good chances it's
already a dependency of the application, or will be.

### Was AI used?

Yes, the work was AI assisted, but not vibe-coded. It's a project
carefully designed and polished for the long run.

## Grammar

### Conformance

An implementation must reject inputs that do not fit the defined [grammar](#grammar),
but not every input that fits the grammar is valid, as detailed in the sections
below. For an implementation to be considered comformant, it must pass the
comformance tests provided in [`cases/`](cases/). The format for these test cases
is detailed in the [README.md](cases/README.md) in the same directory.

### Preface

The formal grammar, in ABNF (RFC 5234). All terminals are
sequences of Unicode code points (`%x` ranges denote code
points, not octets). The grammar operates over a token stream:
`INDENT` and `DEDENT` are synthesized by the tokenizer from
line prefixes - they are not present in the text. Both values
and line comments must sit strictly at the current consistent
prefix, which must be all tabs or all spaces for a given
top-level value; different top-level values may use divergent
characters since their prefixes are not shared.

Line comments follow the same requirements as structural text
in terms of indentation prefixes: they must sit strictly at
the level they are commenting on.

The final _nl_ of the text may be omitted.

### Lexical

```
NL          = LF / CR LF
PREFIX      = 1*SP / 1*HTAB      ; all spaces or all tabs
INDENT      = <a line with a deeper prefix than the current block>
DEDENT      = <a line with the prefix of a parent block>

nothing     = <nothing at all, no chars, no spacing, for emphasis>
pad         = *( SP / HTAB )
nl          = pad NL

comment     = "#" <rest of line>
trail       = ( SP / HTAB ) comment

bare        = 1*barechar
barechar    = ALPHA / DIGIT / "-" / "_"
squoted     = "'" *squotechar "'"
squotechar  = %x20-26 / %x28-2027 / %x202A-D7FF / %xE000-10FFFF
dquoted     = DQUOTE *( dquotechar / escaped ) DQUOTE
dquotechar  = %x20-21 / %x23-5B / %x5D-2027 / %x202A-D7FF / %xE000-10FFFF
escaped     = "\" DQUOTE / "\\" / "\n" / "\r" / "\t" / "\b" / "\f" /
              "\x" 2HEXDIG / "\u" 4HEXDIG / "\u{" 1*6HEXDIG "}"

lfence      = "---"
sfence      = "'''"
sfenced     = sfence *sfencechar sfence
sfencechar  = "'" / squotechar
dfence      = DQUOTE DQUOTE DQUOTE
dfenced     = dfence *( DQUOTE / dquotechar / escaped ) dfence
dfencechar  = DQUOTE / dquotechar / escaped

number      = numint / numfloat / numhex / numoct / numbin
numint      = [ "-" ] ( DIGIT / NONZERODIG 1*DIGIT )
numfloat    = numint [ "." 1*DIGIT ] [ "e" ["+" / "-"] 1*DIGIT ]
numhex      = "0x" 1*HEXDIG
numoct      = "0o" 1*OCTDIG
numbin      = "0b" 1*BINDIG
OCTDIG      = %x30-37
BINDIG      = %x30-31
NONZERODIG  = %x31-39
```

The character ranges exclude the C0 controls U+0000 through U+001F,
each family's own quote, the backslash (in the escaped family), the
line and paragraph separators U+2028 and U+2029, and the surrogate
range U+D800 to U+DFFF. The surrogate range isn't accepted even in
escaped form.


### Syntactic

```
file        = mapcomment [ topvalue mapcomment ]
topvalue    = mapblock / listblock / sfenceblock / dfenceblock / inline

mapcomment  = *( comment nl / nl )

inline      = scalar / map / list
scalar      = string / number / "true" / "false" / "null"
string      = dquoted / squoted / sfenced / dfenced
list        = "[" pad [ inline *( pad "," pad inline ) pad ] "]"
map         = "{" pad [ inlinepair *( pad "," inlinepair ) pad ] "}"
inlinepair  = path nothing ":" pad inline

key         = bare / dquoted / squoted
path        = key *( nothing "." nothing key )
mapitem     = path nothing ":" pad mapvalue [trail] nl
mapvalue    = inline nl / nl INDENT block nl DEDENT

block       = sfenceblock / dfenceblock / mapblock / listblock
sfenceblock = sfence nl *( *sfencechar nl ) sfence nl
dfenceblock = dfence nl *( *dfencechar [ "\" ] nl ) dfence nl
mapblock    = mapcomment 1*( mapitem mapcomment )
listblock   = listitems / listfence
listitems   = 1*( "-" pad ( inline / inlinepair ) [trail] nl ) ; Lookahead for ":"
listfence   = lfence nl 1*( ( inline / fenceblock ) nl lfence nl )
fenceblock  = sfenceblock / dfenceblock / mapblock / listitems /
              INDENT listfence DEDENT
```

## The format

### Top-level values

A cnf top-level is a single value. Usually it is a block map organized as
a sequence of key-value pairs:

```
name: "now"
timeout: 30
```

But it may also be any supported value. This is a top-level list containing maps:

```
---
one: 1
---
two: 2
---
```

An empty top-level value is an empty map, which is a special case
since empty blocks are not allowed in the syntax otherwise.


### Keys and paths

A key is either a bare word (`[-_a-zA-Z0-9]+`) or a single or
double quoted string, in the same form as a one-line value string.

For example:

```
a: 1
b.c: 2
```

Keys that are joined with `.` form a path through nested maps. The
example above is equivalent to the inline map `{a: 1, b: {c: 2}}`.

Keys may also be quoted with single quotes or double quotes, with
the same syntax and rules as string values:

```
label."rm -rf": "unsafe"
```


### Maps

There are two layouts for maps:

- **Inline:** the entire map is expressed in a single line.
- **Block:** each key-value pair of the map sits in a different line, and may itself hold inline values and blocks.

This example shows a top-level block map with inline map values:

```
inline: {a: 1, b: 2}
quoted: {"c d": 2}
paths: {ef."g h": 3}
```

Inner block maps are defined at an indent level which must be
formed by a consistent prefix of either all tabs or all spaces.

```
map.block:
    a: 1
    b: 2
```

It is okay to use a different character for different values
sitting at the top-level parent, because these do not share an
indentation prefix. That prevents visual indentation mistakes
that could cause actual data errors.

The key paths can refer to the same map multiple times as long
as the keys are different:

```
map1:
    a.b: 1
    a: {c: 2}

map2:
    a: {b: 1, c: 2}
```

The value of `map1` and `map2` is identical.

While this is flexible, there are constraints to ensure that the content
remains sane when changing it. In particular, a map key cannot be
defined multiple times, even if the value is the same:

```
invalid:
    a.b: 1
    a: {b: 1}
```

A map also cannot be _expanded_ more than once. As an example, this is
invalid even though the keys themselves do not conflict:

```
invalid:
  a: {b: 1}
  a: {c: 2}
```

The rationale for it being invalid follows from the previous constraint: it looks
like the `a` value is being defined more than once, so making it invalid improves
the clarity of the data.

This example is also invalid but for a more obvious reason:

```
invalid:
  a: 1
  a.b: 2
```

In this case the `invalid.a` key is being assigned two different values:
`1` and a map containing the `b` key.

### Lists

There are three layouts for lists:

- **Inline:** the entire list is expressed in a single line.
- **Dashed:** each element of the list is expressed in a single line.
- **Fenced:** the list can hold inline and block elements.

This example includes all three layouts:

```
inline.list: [1, 2, 3]

dashed.strings:
    - "item 1"
    - "item 2"

dashed.maps:
    - label: "map 1"
    - label: "map 2"

fenced.maps:
    ---
    list1.map1:
        a: 1
        b: 2
    list1.map2:
        c: 3
    ---
    list2.map1:
        a: 1
        b: 2
    ---
```

If deeper nesting is required, a list fence element may itself be a
list fence, nested at a deeper indent that is consistent across all
list elements.

This example shows lists inside lists inside a list:

```
fenced.nested.lists:
    ---
        ---
        - 1
        ---
        - 2
        - 3
        ---
    ---
        ---
        - 4
        ---
    ---
```

### Numbers

The representation for numbers has the classic formats one would expect:

```
int: 123
float: 1.23
hex: 0x123
octal: 0o77
binary: 0b11
exponent: 1e-23
```

A decimal integer may not carry a redundant leading zero - `066` is
a syntax error, while `0o066` is an octal literal.

Implementations should support at least IEEE754 double precision for
interoperability, and may use more precision or width when desired, but must
explicitly reject integer values they cannot represent. In particular, some
mainstream language libraries decode json numbers with precision or
width beyond that minimum, and an important feature of _cnf_ is
compatibility with those libraries.

### Strings

There are two layouts for strings:

- **Inline:** the entire string is expressed in a single line.
- **Fenced:** the string is broken down into a multi-line block.

Both layouts support two quoting families:

- **Double-quoted:** escapes are processed.
- **Single-quoted:** a backslash is a backslash.

The fenced layout can only be expressed with triple-quotes, single or double.

This example includes some of these variations:

```
double: "line one\nline two"
single: 'line one\nstill one'

fenced.double:
    """
    line one
    line two\
    """
```

The first value contains a newline; the second contains a
backslash followed by the n letter; the third value is exactly
the same as the first value since the last newline is escaped.

The full set of escape characters is listed in the following
table. Unknown escapes are a syntax error.

| Escape | Yields | Notes |
|---|---|---|
| `\\` | `\` |  |
| `\"` | `"` | required inside `"` strings |
| `\n` `\r` `\t` | newline, c-return, tab | |
| `\b` `\f` | backspace, form feed | |
| `\xNN` | U+0000 to U+00FF | two hex digits |
| `\uNNNN` | the unicode character, see below | four hex digits |
| `\u{N...NNNNNN}` | the unicode character, see below | one to six hex digits |

Both `\u` spellings must exclude the surrogate range U+D800 to U+DFFF
and be under U+10FFFF.

On write, the canonical form is to escape the C0 controls below U+0020 in
the `\xNN` form, escape the line separators U+2028/U+2029 in the `\uNNNN` form,
and to write literally everything else, including the printable Latin-1
range U+0020–U+00FF.

A string fence spreads a string across lines. The opening and closing
`"""` or `'''` sit alone on their line at a matching indentation prefix,
and the content inside the fences define the multi-line string, excluding
the matching prefix from the fences.

```
script:
    """
    #!/bin/sh
    echo "not today"
    """
```

The value is `#!/bin/sh\necho "not today"\n`.

When handling fenced string blocks:

- The fence content is strictly _inside the fences_, which means
  the newline after the opening fence is not part of the value
  while the one before the closing fence is.
- Thus, a fence with no lines inside represents the empty string `""`.
- To obtain a non-empty fence that does not end in a newline, the
  fence must be `"""` and escape the last newline inside it.
- A fence cannot contain a line that is exactly its own delimiter
  at its own indentation - that line would close it.

A single-line fence is the compact spelling, useful mainly in quote-heavy
strings which benefit from less escaping:

```
double.inline: """#!/bin/sh\necho "not today"\n"""
single.inline: '''line one\nstill one'''
```


### Indentation

The rules for indentation are:

- A block map key-value pair introduces indentation when its value is not inline,
  and a fenced list introduces indentation when nesting under another fenced list.
  These are the only two places in the grammar that introduce indentations.
- The indentation prefix is defined by the first character (including comments)
  on a non-blank line line that is neither a tab (0x09) nor a space (0x20).
- The indentation for a given value must be either all spaces, or all tabs, not
  mixed, and no other character; different top-level hierarchies may diverge in
  the choice of character because they don't share the prefix.
- The block closes when a non-blank line returns to a shallower prefix.
  A valid close must strictly return to the exact prefix of an ancestor
  block, otherwise it's a syntax error.
- When the grammar allows blank lines and those are outside of fenced string
  blocks, these are ignored even if they contain prefixes that are invalid
  otherwise due to depth or wrong indentation character.
- When a blank line is found inside a fenced string block:
  - The blank string must be formed by the correct character for the
    indentation context.
  - If the blank line is shallower than or exactly at the fenced string's indent,
    the respective fenced string line contains the newline only.
  - If the blank line goes deeper than the fenced string's indent, the content
    following the indentation is considered part of the fenced string's line.

These two examples are simple cases where the prefix is invalid:

```
invalid:
    map1:
        a: 1
            b: 2

    map2:
        a:
            b: 1
      c: 2
```

In `map1`, `b` failed to follow the current indent prefix.
In `map2`, `c` failed to dedent to the parent's prefix.

### Comments

A comment starts at a `#` and runs to the end of the line.

Two spellings exist:

- **Standalone comment:** a `#` is preceded only by blanks in its line.
- **Inline comment:** a `#` follows an inline value and separating blanks.

A standalone comment sits precisely at the indent of the block it belongs to
and its indentation prefix must conform to the general indentation norms.

```
# Standalone comment
timeout: 30 # Inline comment
service:
    # Standalone comment
    enabled: true
```

Comments follow strict placement contraints that have the intent of improving
clairty of the text as a whole, and facilitating semantic parsing of the comments.

Specifically:

- They follow the same general indentation rules.
- They are not allowed around fence openings and closings.


## Authors

Gustavo Niemeyer


## Appendix: Maintaining this document

Recommendations when performing changes to this spec:

- **Do not overspecify the obvious.** The grammar and behavior should be
  precise, but avoid sentences explaining something inherently clear
  as that creates confusion where there was none.

- **Prefer examples over prose.** A small annotated example often explains
  cases better than an extensive explanation. Reach for the example first,
  and keep the prose around it short.

- Always nest incorrect examples under an `invalid` key.
