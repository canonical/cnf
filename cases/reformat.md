# Topic

Reformat printing: the render-time rewriting of an AST's layout
under settings, the recorded text staying untouched.

Each knob's cases come in a series, -a through -c: the -a
case crosses the threshold and rewrites, the -b case sits at
the boundary and does nothing, the -c case does the
same at a nested position.

The rewrites are built on the following core ideas: 

**Inline** - The inline item layout.

**Depth** - The absolute level of nesting of the map, list, or path segment in consideration, where a key in the top map has depth 1.

**Nesting** - The levels of nesting inside a inline map, inline list, or path. An element outside of any inline map, list, or path, has nesting 0.

**Items** - The count of items inside a single nesting level of a single map or list.

**Upto/From** - The limits of an operation. Both limits are inclusive: with (1, 2, 3), upto 2 means (1, 2) and from 2 means (2, 3). The terms establish such limits via the concepts above.

**Width** - The relative number of runes required to represent the specific value described in the context; indents never count.

**Column** - The absolute position of the rune in the line; the last rune when used in a limit context.

**Fold** - Operation contracting multiple lines of an inline map, list, string, or path, into a single line. Folding a **map** increases the *nesting* of the items inside the map, without changing their *depth*. Folding a *path* increases the *nesting* of any keys appended to the path without affecting their *depth*.

**Unfold** - Operation expanding an inline map, list, string, or path, into multiple lines. Goes in the opposite direction of the *fold*.

**Example:**

```
a:
  b.c:
     d: {e: {f.g: {h: "i"}, f.j: ["k", "l", ["m", {n: "o"}]]}}
```

- "a": depth 1, map nesting 0, path nesting 1, items 1 (a)
- "b": depth 2, map nesting 0, path nesting 1, items 1 (c)
- "c": depth 3, map nesting 0, path nesting 2, items 1 (d)
- "d": depth 4, map nesting 0, path nesting 1, items 1 (e)
- "e": depth 5, map nesting 1, path nesting 1, items 1 (f)
- "f": depth 6, map nesting 2, path nesting 1, items 2 (g, j)
- "g": depth 7, map nesting 2, path nesting 2, items 1 (h)
- "h": depth 8, map nesting 3, path nesting 1, items 1 ("i")
- "i": depth 8, map nesting 3, path nesting 0, items 0 ()
- "j": depth 5, map nesting 1, path nesting 1, items 3 ("k", "l", list)
- "k": depth 6, map nesting 1, list nesting 1, items 0 ()
- "l": depth 6, map nesting 1, list nesting 1, items 0 ()
- "m": depth 7, map nesting 1, list nesting 2, items 0 ()
- "n": depth 8, map nesting 2, list nesting 2, items 1 ("o")
- "o": depth 8, map nesting 2, list nesting 2, items 0 ()


## Case: fold-path-upto-depth-c

A path folds into its key: the nesting levels below an item
join its key - the enclosing key prepended - while the key
would hold up to and including the depth budget's segments, a
segment already dotted on a line counting. The value at the
end of the path stays on the folded line, verbatim - an
inline map is a value, the fold never opens it. A path too
deep folds to the budget and its remainder nests, at a fresh
indent, one indent step under the folded key. The fold never splits a
dotted segment: that is the unfold operation.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 3
```

### cnf

```
a.b.c: 1
a.d: {"e": 2}
a.f.g:
    h: 3
a.i.j:
    k: 4
a.l:
    m.n: 5
```

### json

```
{"a":{"b":{"c":1},"d":{"e":2},"f":{"g":{"h":3}},"i":{"j":{"k":4}},"l":{"m":{"n":5}}}}
```

### ast

```
a:
    b:
        c: 1
    d: {"e": 2}
    f:
        g:
            h: 3
    i.j:
        k: 4
    l:
        m.n: 5
```


## Case: fold-path-from-depth-c

A path folds into its key past the depth threshold: the
nesting levels below an item join its key - the enclosing
key prepended - while the line extends past the threshold,
the from limit inclusive: a line ending before the
threshold is kept as is, the fold never extending it. The
value at the end of the path stays on the folded line,
verbatim - an inline map is a value, the fold never opens
it. A line ending before the threshold is kept whole: j.k
is kept, merging l would extend a line that does not reach
it.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-from-depth: 3
```

### cnf

```
a:
    b:
        c: 1
    d: {"e": {"f": 2}}
    g:
        h.i: 3
    j.k.l: 4
    m:
        n.o.p: 5
```

### json

```
{"a":{"b":{"c":1},"d":{"e":{"f":2}},"g":{"h":{"i":3}},"j":{"k":{"l":4}},"m":{"n":{"o":{"p":5}}}}}
```

### ast

```
a:
    b:
        c: 1
    d: {"e": {"f": 2}}
    g:
        h:
            i: 3
    j.k:
        l: 4
    m:
        n.o:
            p: 5
```


## Case: unfold-path-from-depth-c

A path unfolds past the depth threshold: the dotted
segments of a key past the threshold split off the key,
one nesting level per segment, the from limit inclusive -
the segments at depths up to and including the threshold
stay on the key, the fold's upto boundary mirrored. The
value at the end of the path rides on the unfolded key's
line, verbatim - an inline map is a value, the unfold
never opens it. A key line holding no segment past the
threshold stands whole.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 3
```

### cnf

```
a:
    b:
        c: 1
    d.e: {"f": {"g": 2}}
    h:
        i:
            j: 3
    k.l:
        m: 4
```

### json

```
{"a":{"b":{"c":1},"d":{"e":{"f":{"g":2}}},"h":{"i":{"j":3}},"k":{"l":{"m":4}}}}
```

### ast

```
a:
    b:
        c: 1
    d.e: {"f": {"g": 2}}
    h:
        i.j: 3
    k.l.m: 4
```


## Case: unfold-path-upto-depth-c

A path unfolds up to the depth limit: the dotted segments
at or before the limit split off the key, one nesting level
per segment, the limit inclusive - the segments past it stay
dotted on the innermost key, the deep tail kept as is. The
value at the end of the path stays on the innermost unfolded
key's line, verbatim - an inline map is a value, the unfold
never opens it. A key line holding no segment at or before
the limit is kept whole.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-upto-depth: 2
```

### cnf

```
a:
    b:
        c: 1
    d:
        e.f: 2
    g:
        h.i: 3
    j:
        k:
            l: 4
```

### json

```
{"a":{"b":{"c":1},"d":{"e":{"f":2}},"g":{"h":{"i":3}},"j":{"k":{"l":4}}}}
```

### ast

```
a:
    b.c: 1
    d.e.f: 2
    g:
        h.i: 3
    j.k:
        l: 4
```


## Case: unfold-path-multi-split

A key line splitting two or more segments opens one
nesting level per segment, in the line's own order: the
first splitting segment the outermost level, the last the
innermost, the value on the innermost key. The chain these
splits build is the unfold's fresh nesting - the levels
paper over no recorded layout, so the order is the only
thing the reader can see, and the segments must arrive
exactly as the line spelled them.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 1
```

### cnf

```
a:
    b:
        c:
            d: 1
x:
    y:
        z:
            w:
                q: 2
```

### json

```
{"a":{"b":{"c":{"d":1}}},"x":{"y":{"z":{"w":{"q":2}}}}}
```

### ast

```
a.b.c.d: 1
x.y.z.w.q: 2
```


## Case: fold-map-upto-items-a

A top-level block map up to the item limit folds into its
inline form: one line, comma-separated pairs. The limit is
inclusive, the same rule as the list's, the map's pairs for
the list's items.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-map-upto-items: 3
```

### cnf

```
{a: 1, b: 2, c: 3}
```

### json

```
{"a":1,"b":2,"c":3}
```

### ast

```
a: 1
b: 2
c: 3
```


## Case: fold-map-upto-items-b

The same block map over the item limit keeps its recorded
block form: the limit is inclusive, and three items are
over two, so the settings leave the AST as it was.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-map-upto-items: 2
```

### cnf

```
a: 1
b: 2
c: 3
```

### json

```
{"a":1,"b":2,"c":3}
```

### ast

```
a: 1
b: 2
c: 3
```


## Case: fold-map-upto-items-c

A nested block map up to the item limit folds into its inline
form, its ancestors kept: the fold is bottom-up - an
ancestor never folds while an element underneath remains
unfolded. A fold may nest inside a fold, the inner one's
items counting under the same limit, and an unfolded map
between two folding ones is decided on its own count.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-map-upto-items: 2
```

### cnf

```
a:
    b: {c: 1, d: {e: 2, f: 3}}
    g:
        h: 4
        i: 5
        j: 6
    k: {l: 7, m: 8}
    n: 9
```

### json

```
{"a":{"b":{"c":1,"d":{"e":2,"f":3}},"g":{"h":4,"i":5,"j":6},"k":{"l":7,"m":8},"n":9}}
```

### ast

```
a:
    b:
        c: 1
        d:
            e: 2
            f: 3
    g:
        h: 4
        i: 5
        j: 6
    k:
        l: 7
        m: 8
    n: 9
```




## Case: unfold-map-from-items-a

A top-level inline map past the item threshold unfolds to its
block form: one pair per line. The same threshold and the
same rule as the list's, the map's pairs for the list's items.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-map-from-items: 3
```

### cnf

```
a: 1
b: 2
c: 3
```

### json

```
{"a":1,"b":2,"c":3}
```

### ast

```
{a: 1, b: 2, c: 3}
```


## Case: unfold-map-from-items-b

The same map under the item threshold keeps its recorded inline
form: the threshold never triggers, so the settings leave the
AST as it was.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-map-from-items: 4
```

### cnf

```
{a: 1, b: 2, c: 3}
```

### json

```
{"a":1,"b":2,"c":3}
```

### ast

```
{a: 1, b: 2, c: 3}
```


## Case: unfold-map-from-items-c

An inline map past the item threshold unfolds to its block
form at a nested level too: the same rule under a map
key, the block indented under its pair. An inline map inside
an unfolded one keeps its inline form on its own count,
and one before the threshold is kept, the from limit inclusive.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-map-from-items: 3
```

### cnf

```
a:
    b:
        c: 1
        d: {e: 2, f: 3}
        g: 4
    h:
        i: 5
        j: 6
        k: 7
    l: {m: 8, n: 9}
    n: 10
```

### json

```
{"a":{"b":{"c":1,"d":{"e":2,"f":3},"g":4},"h":{"i":5,"j":6,"k":7},"l":{"m":8,"n":9},"n":10}}
```

### ast

```
a:
    b: {c: 1, d: {e: 2, f: 3}, g: 4}
    h: {i: 5, j: 6, k: 7}
    l: {m: 8, n: 9}
    n: 10
```


## Case: unfold-map-respects-indent

An unfolded map adopts the indent of the hierarchy it sits
in: the root map recorded two spaces for the block under
`keep`, and the levels the unfold opens under `svc` - the
map itself, and the inline map inside it - open at the same
two-space step instead of a fresh one. The reformat never
invents an indent the tree did not use.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-map-from-items: 2
```

### cnf

```
keep:
  x: 1
svc:
  name: "web"
  opts:
    a: 1
    b: 2
```

### json

```
{"keep":{"x":1},"svc":{"name":"web","opts":{"a":1,"b":2}}}
```

### ast

```
keep:
  x: 1
svc: {name: "web", opts: {a: 1, b: 2}}
```


## Case: unfold-list-from-items-a

A top-level inline list past the item threshold unfolds to its
block form: one element per line behind a dash. The AST
records the inline form; the settings override it for the
one render.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 3
```

### cnf

```
- 1
- 2
- 3
```

### json

```
[1,2,3]
```

### ast

```
[1, 2, 3]
```


## Case: unfold-list-from-items-b

The same list under the item threshold keeps its recorded inline
form: the threshold never triggers, so the settings leave the
AST as it was.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 4
```

### cnf

```
[1, 2, 3]
```

### json

```
[1,2,3]
```

### ast

```
[1, 2, 3]
```

## Case: unfold-list-from-items-c

An inline list past the item threshold unfolds to its block
form at a nested level too: the same rule under a map key,
the block indented under its pair. An inline list inside
an unfolded one keeps its inline form on its own count,
and one before the threshold is kept, the from limit inclusive.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 3
```

### cnf

```
a:
    b:
        - 1
        - [2, 3]
        - 4
    h:
        - 5
        - 6
        - 7
    l: [8, 9]
    n: 10
```

### json

```
{"a":{"b":[1,[2,3],4],"h":[5,6,7],"l":[8,9],"n":10}}
```

### ast

```
a:
    b: [1, [2, 3], 4]
    h: [5, 6, 7]
    l: [8, 9]
    n: 10
```


## Case: fold-list-upto-items-a

A top-level block list up to the item limit folds into its
inline form: one line, comma-separated. The limit is
inclusive - a list of exactly this many items folds too. The
AST records the block form; the settings override it for
the one render.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-list-upto-items: 3
```

### cnf

```
[1, 2, 3]
```

### json

```
[1,2,3]
```

### ast

```
- 1
- 2
- 3
```


## Case: fold-list-upto-items-b

The same block list over the item limit keeps its recorded
block form: the limit is inclusive, and three items are
over two, so the settings leave the AST as it was.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-list-upto-items: 2
```

### cnf

```
- 1
- 2
- 3
```

### json

```
[1,2,3]
```

### ast

```
- 1
- 2
- 3
```


## Case: fold-keyed-pair

A keyed dashed element folds into its inline form when its
value can be written inline: the element re-wraps as its
single-pair map, the key riding the wrapper - `- c: 1`
spells `{c: 1}`. The two are one value in two spellings, so
the fold keeps what a dash line wrote, and the inline list
writes it in the spelling the inline form holds.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-list-upto-items: 2
```

### cnf

```
k: [{c: 1}, {d: 2}]
```

### json

```
{"k":[{"c":1},{"d":2}]}
```

### ast

```
k:
    - c: 1
    - d: 2
```


## Case: unfold-list-ingest

A dashed element holding a single-pair inline map that the
unfold would open ingests the pair onto its key path instead:
the element becomes `- c: 1`, spelled on the dash line, no
demotion to the fence. The ingest respects the tree's own
indent - the whole opened structure sits at the step the
file uses.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 1
    unfold-map-from-items: 1
```

### cnf

```
a:
  b:
    - c: 1
d.e:
  f: 2
```

### json

```
{"a":{"b":[{"c":1}]},"d":{"e":{"f":2}}}
```

### ast

```
a: {b: [{c: 1}]}
d.e:
  f: 2
```


## Case: unfold-list-ingest-compose

An already-keyed element ingests by composition: the pair's
segments append to the element's own key path, the dash line
spelling the joined path in one piece.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 1
    unfold-map-from-items: 1
```

### cnf

```
k:
    - k.c: 1
```

### json

```
{"k":[{"k":{"c":1}}]}
```

### ast

```
k: [{k: {c: 1}}]
```


## Case: unfold-list-no-ingest-multi

A multi-pair element does not ingest: the pair is not single,
so the element opens as its block map and the list takes its
fence form, every element spelled as the map the grammar's
fence path reads back.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 1
    unfold-map-from-items: 1
```

### cnf

```
k:
    ---
    c: 1
    d: 2
    ---
```

### json

```
{"k":[{"c":1,"d":2}]}
```

### ast

```
k: [{c: 1, d: 2}]
```


## Case: unfold-list-ingest-chain

A chain of single-pair inline maps collapses onto the dash
line entirely: every link is a map the unfold would open, and
the innermost value stays inline, so the whole chain rides
the element's key path. `- c.d: 1` and `- k.c: 1` are one
ingest rule: the path composes, the innermost value lands
last.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 1
    unfold-map-from-items: 1
```

### cnf

```
k:
    - c.d: 1
```

### json

```
{"k":[{"c":{"d":1}}]}
```

### ast

```
k: [{c: {d: 1}}]
```


## Case: fold-keyed-block-value

A keyed element whose value cannot be written inline blocks
its ancestors' folds. The parse cannot spell a dash line
over a block, so this case arrives through the review form:
the element holds a block map, the list over the fold
threshold would fold but for the element, and it keeps its
fence form - the block held inside it, intact.

```
to-json: "advised-text"
from-ast: "advised-text"
```

### cnf

```
k:
    ---
    c:
        d: 1
    ---
```

### json

```
{"k":[{"c":{"d":1}}]}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"k","pos":{"l":1,"c":1}}],
            "value": {
                "list": [{
                    "value": {
                        "map": [{
                            "key": [{"str":"c","pos":{"l":3,"c":5}}],
                            "value": {
                                "map": [{
                                    "key": [{"str":"d","pos":{"l":4,"c":9}}],
                                    "value": {"num":"1","pos":{"l":4,"c":12}}
                                }],
                                "block": true,
                                "pos": {"l":4,"c":9}
                            },
                            "indent": "    "
                        }],
                        "block": true,
                        "pos": {"l":3,"c":5}
                    }
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: fence-keyed-element

A keyed element renders in the fence form without losing its
key: the dash spelling has no fence counterpart, so the
element spells as its pair - the map form, one value in two
spellings. A keyed element holding an inline map that the
unfold opens alongside it takes the fence together with the
plain keyed, the pair opening the opened block under the
pair line, and every key survives the round trip.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-list-from-items: 1
    unfold-map-from-items: 1
```

### cnf

```
k:
    ---
    c: 1
    ---
    d:
        e: 2
        f: 3
    ---
```

### json

```
{"k":[{"c":1},{"d":{"e":2,"f":3}}]}
```

### ast

```
k: [{c: 1}, {d: {e: 2, f: 3}}]
```


## Case: reindent-mixed

The reindent setting rewrites the indent every block level
opens at, the whole document one step: a tree with two
hierarchies at different recorded steps - two spaces under
`a`, four under `c` - renders every level one tab deep, the
variety gone by design, placement only.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    reindent: "\t"
```

### cnf

```
a:
	b: 1
c:
	d:
		e: 2
```

### json

```
{"a":{"b":1},"c":{"d":{"e":2}}}
```

### ast

```
a:
  b: 1
c:
    d:
        e: 2
```


## Case: reindent-fence-flat

A fence element's block content stays anchored at the fence's
own base through a reindent: the element has no line of its
own, its distance is zero, and the setting rewrites the steps
the levels recorded - never the alignment the fence already
holds. The element's pairs take the swap; nothing
double-steps the fence's own content.

```
to-json: "advised-text"
from-ast: "advised-text"
parse-ast: true
reformat:
    reindent: "\t"
```

### cnf

```
k:
	---
	c:
		d: 1
	---
top: 2
```

### json

```
{"k":[{"c":{"d":1}}],"top":2}
```

### ast

```
k:
    ---
    c:
        d: 1
    ---
top: 2
```


## Case: fold-list-upto-items-c

A block list up to the item limit folds into its inline
form at a nested level too: the same rule under a map
key, the folded value on the pair's line. A fence folds the
same way - it is a list's block form - a fold nesting
inside a fold, the inner one's items counting under the
same limit, and a fence over the limit is decided on its own
count.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-list-upto-items: 2
```

### cnf

```
a:
    b: [1, [2, 3]]
    c:
        ---
        [4, 5]
        ---
        - 6
        - 7
        - 8
        ---
    d: [9]
    e: [10, 11]
```

### json

```
{"a":{"b":[1,[2,3]],"c":[[4,5],[6,7,8]],"d":[9],"e":[10,11]}}
```

### ast

```
a:
    b:
        ---
        1
        ---
        - 2
        - 3
        ---
    c:
        ---
        - 4
        - 5
        ---
        - 6
        - 7
        - 8
        ---
    d:
        - 9
    e:
        - 10
        - 11
```


## Case: merge-unfold-together

A split level inserts into an existing map by its key: the
unfold splits a dotted line past its threshold, and the
level it opens merges into the block map sibling that
already holds the key - the split item's pairs joining the
existing map's, so the two never render as one duplicate.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 2
```

### cnf

```
a:
    b:
        c: 1
        d: 2
```

### json

```
{"a":{"b":{"c":1,"d":2}}}
```

### ast

```
a:
    b:
        c: 1
a.b.d: 2
```


## Case: merge-unfold-together-path-grounding

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 2
```

### cnf

```
a.b:
    c: 1
    d:
        e: 2
```

### json

```
{"a":{"b":{"c":1,"d":{"e":2}}}}
```

### ast

```
a.b:
    c: 1
    d:
        e: 2
```


## Case: merge-unfold-together-path

Same as before, but with a folded a.b key.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 2
```

### cnf

```
a.b:
    c: 1
    d:
        e: 2
```

### json

```
{"a":{"b":{"c":1,"d":{"e":2}}}}
```

### ast

```
a.b:
    c: 1
a.b.d.e: 2
```


## Case: merge-unfold-apart

The same split with a sibling between: the insert is keyed,
not positional, so a pair sitting between the two does not
bar the merge - the split level finds the existing map
wherever it sits, and the pair stays wherever it lands.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 2
```

### cnf

```
a:
    b:
        c: 1
        d: 2
x: 9
```

### json

```
{"a":{"b":{"c":1,"d":2}},"x":9}
```

### ast

```
a:
    b:
        c: 1
x: 9
a.b.d: 2
```


## Case: merge-unfold-fold-path

The path passes compose, the unfold running before the fold:
a key line past the unfold's threshold splits, one nesting
level per segment, and the fold then joins the levels within
its budget, the remainder nesting under the folded key. A
dotted line over the fold's budget is kept whole without the
split - the fold never splits a segment - so the split
re-joins what the fold alone leaves in place. A line at or
before the unfold's threshold is kept whole, the fold joining
it within its budget.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 3
    fold-path-upto-depth: 3
```

### cnf

```
a.b.c:
    d:
        e: 1
e.f.g: 2
```

### json

```
{"a":{"b":{"c":{"d":{"e":1}}}},"e":{"f":{"g":2}}}
```

### ast

```
a.b:
    c.d.e: 1
e:
    f.g: 2
```




## Case: merge-error

A merge that cannot resolve is a conflict: two lines reaching one
key with values the insert cannot join - a scalar against a map, or
two scalars at one key - refuse, the render failing rather than
emitting invalid cnf. The AST keeps both pairs as written; the
conflict is in the rewrite.

```
from-ast: "error"
parse-ast: true
reformat:
    unfold-path-from-depth: 2
error: "2:1: cannot merge with previous `a.b`: not a map"
```

### ast

```
a.b: 1
a.b.c: 2
```


## Case: merge-error-held

The other refusal: the incoming item cannot merge - its value
is not a block map, colliding with an existing key that holds
one. The split level's inner item inserts into the existing
map, and a scalar reaching a key the map already holds is
already held: the render fails rather than emit invalid cnf.

```
from-ast: "error"
parse-ast: true
reformat:
    unfold-path-from-depth: 2
error: "4:5: cannot merge with previous `c`: not a map"
```


### ast

```
a.b:
    c:
        d: 1
a.b.c: 2
```


## Case: unfold-path-and-map

The path unfold and the map unfold compose: every dotted key
splits into nesting levels, and every inline map opens into
its block form, the two passes over one tree. The lines
merge by key as the paths split - a.b.c and a.b join into one
a.b - and the maps the unfold opens hold their pairs one per
line, the inner inline maps opening on their own counts.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    unfold-path-from-depth: 1
    unfold-map-from-items: 1
```

### cnf

```
a:
    b:
        c:
            d: 1
            e:
                f: 2
            g:
                h: 3
        i: 4
        j:
            k: 5
            l: 6
```

### json

```
{"a":{"b":{"c":{"d":1,"e":{"f":2},"g":{"h":3}},"i":4,"j":{"k":5,"l":6}}}}
```

### ast

```
a.b.c: {d: 1, e.f: 2, g: {h: 3}}
a.b: {i: 4, j: {k: 5, l: 6}}
```


## Case: soft-wrap-string-from-width-a

A string whose line exceeds the hard width wraps at the last
space past the soft floor: the break lands on whitespace, the
word staying whole, the soft floor skipping the early spaces.
The wrapped string is rewritten as a fence - the wrap creates
lines, only the fence holds them - the wrapped lines its
recorded Text, the value untouched.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    soft-wrap-string-from-width: 3
    hard-wrap-string-from-width: 10
```

### cnf

```
a:
    """
    one two \
    three\
    """
```

### json

```
{"a":"one two three"}
```

### ast

```
a: "one two three"
```


## Case: soft-wrap-string-from-width-b

A string whose line sits exactly at the soft width is kept:
the limit is inclusive, the threshold never triggers, the
recorded form staying as it was.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    soft-wrap-string-from-width: 13
```

### cnf

```
a: "one two three"
```

### json

```
{"a":"one two three"}
```

### ast

```
a: "one two three"
```


## Case: soft-wrap-string-from-width-c

The same wrap at a nested level: the string under a map key
wraps the same way, its fence indented under its pair.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    soft-wrap-string-from-width: 3
    hard-wrap-string-from-width: 10
```

### cnf

```
x:
    a:
        """
        one two \
        three\
        """
```

### json

```
{"x":{"a":"one two three"}}
```

### ast

```
x:
    a: "one two three"
```


## Case: hard-wrap-string-from-width-a

A string whose line exceeds the hard width wraps wherever the
limit falls, mid-word if it must: the aggressive backstop, no
space needed.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    hard-wrap-string-from-width: 7
```

### cnf

```
a:
    """
    onetwot\
    hree\
    """
```

### json

```
{"a":"onetwothree"}
```

### ast

```
a: "onetwothree"
```


## Case: hard-wrap-after-soft

The two widths compose: the soft margin tried first, the hard
the backstop. A word longer than the soft margin breaks at
the soft width's last space when one exists, the remainder
past the hard width breaking mid-word.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    soft-wrap-string-from-width: 10
    hard-wrap-string-from-width: 6
```

### cnf

```
a:
    """
    onetwo\
    three\
    """
```

### json

```
{"a":"onetwothree"}
```

### ast

```
a: "onetwothree"
```


## Case: insert-routed-comments

A path fold can route a line under a key the rebuilt map
already holds: the `a.b.c: 3` line folds within the depth
range while the `a` holding a block stays whole outside
it, and the folded line lands inside `a`'s map. That is
the map1 shape of maps.md - paths expanding a block-spelled
map - and the route is legal cnf. The routed line's pre
and post comments must arrive with it: the runs belong to
the line, wherever the fold carries it.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-from-depth: 2
    ignore-comments: false
```

### cnf

```
a:
    x: 1

    # c.pre
    b.c: 3 # c.in
    # c.post
```

### json

```
{"a":{"x":1,"b":{"c":3}}}
```

### ast

```
a:
    x: 1

# c.pre
a.b.c: 3 # c.in
# c.post
```


## Case: fold-path-comments

A path fold absorbs single-pair levels onto one line, and
the comments those dissolved levels carry ride with them:
a run above the absorbed level's pair sat at the level's
own prefix, directly before the content the folded line
now spells, so it lands in the same position relative to
the folded line - above it, a pre run. The absorption
drops nothing here and refuses nothing: a four-deep chain
with a comment above its last pair folds all the way to
one line, the comment above.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 4
    ignore-comments: false
```

### cnf

```
# d.pre
a.b.c.d: 1
```

### json

```
{"a":{"b":{"c":{"d":1}}}}
```

### ast

```
a:
    b:
        c:
            # d.pre
            d: 1
```


## Case: fold-path-comments-mid

A comment describes the pair whose line it guards, and a
key extension moves that anchor to the final segment: the
line that spelled `c` now spells `c.d`, and a comment that
described `.c` would read as one describing `.d`. The fold
refuses that: an absorption that would extend a line past
a commented pair commits at the depth reached, the
comment-carrying level nesting under it whole - the same
nesting a depth-range refusal performs, and the one the
descent's own fold performs inside the nested level,
where the commented line faces the same rule on its own
key. A pre comment on the chain's second pair with the
budget past it folds only the first two segments and
freezes the commented remainder.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 4
    ignore-comments: false
```

### cnf

```
a.b:
    # c.pre
    c:
        d: 1
```

### json

```
{"a":{"b":{"c":{"d":1}}}}
```

### ast

```
a:
    b:
        # c.pre
        c:
            d: 1
```


## Case: fold-path-comments-mid-ignore

The same chain under ignore-comments folds through the
run-carrying level: the middle pair's pre run has no
position on a line that does not spell it, and the knob
drops it - the genuine no-spot loss of the path folds, and
the one the escape hatch exists for. The tail's own
comments, by contrast, ride - the same run above the same
folded line, wherever along the chain the tail sits.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 4
    ignore-comments: true
```

### cnf

```
a.b.c.d: 1
```

### json

```
{"a":{"b":{"c":{"d":1}}}}
```

### ast

```
a:
    b:
        # c.pre
        c:
            d: 1
```


## Case: fold-path-comments-inner

Head and foot runs describe the block they sit around, not
any one line of it: a fold that would dissolve the block -
promoting its pairs past the runs - strands both. The
anchor rule stops that dissolve: the line commits at the
depth reached, and the block stays whole under the folded
key, its runs adjacent to the content they describe, the
blank separation kept with them. The head run must sit
blank-separated above the first pair and the foot below the
last - the adjacent spelling attributes to the pairs
themselves, a different case - so the fold here is the
single-pair level above the block, stopping there and
dissolving none of its pairs.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 3
    ignore-comments: false
```

### cnf

```
a.b:
    # b.head

    x: 1
    y: 2

    # b.foot
```

### json

```
{"a":{"b":{"x":1,"y":2}}}
```

### ast

```
a:
    b:
        # b.head

        x: 1
        y: 2

        # b.foot
```


## Case: fold-path-comments-inner-ignore

The same block under ignore-comments dissolves freely: the
pairs promote past the head and foot runs, the lines
folding fully to the budget. The knob permits the loss, so
the runs drop - the block spelling of the comments is the
price of the fold.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 3
    ignore-comments: true
```

### cnf

```
a.b.x: 1
a.b.y: 2
```

### json

```
{"a":{"b":{"x":1,"y":2}}}
```

### ast

```
a:
    b:
        # b.head

        x: 1
        y: 2

        # b.foot
```


## Case: fold-path-comments-own-inner

A head run guards the whole block it sits in, and
absorbing the block's only pair vacates the block: the
run would describe a block that no longer holds the
content it commented. The fold stops before that
absorption, the line committing at the depth reached -
the block nesting whole under the folded key, its head
run adjacent to the pair it described, the front blank
riding the run as stored. The fold then proceeds inside
the nested block, folding the levels below: the run
sits around the content it described.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 3
    ignore-comments: false
```

### cnf

```
a:
    # a.head

    b.x: 1
```

### json

```
{"a":{"b":{"x":1}}}
```

### ast

```
a:
    # a.head

    b:
        x: 1
```


## Case: fold-path-comments-own-inner-ignore

The same shape under ignore-comments folds through the
vacated level: the head run has no home on the deeper
folded lines, and the knob drops it - the loss the
escape hatch exists for, the lines folding fully to the
budget.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-path-upto-depth: 3
    ignore-comments: true
```

### cnf

```
a.b.x: 1
```

### json

```
{"a":{"b":{"x":1}}}
```

### ast

```
a:
    # a.head

    b:
        x: 1
```


## Case: unfold-list-chain-comments

A dashed element holding a block map folds its single-pair
chain onto the dash line, so the line stays spellable - and
the chain's pairs hand over only their keys and values,
every comment run they carry dying on the way. A
run-carrying chain refuses the fold: the element keeps its
block, and the render demotes the list to the fence form,
whose element spells the head run around the content it
describes - the dash line's escape hatch, reached through
comments. The tree arrives through the review form: no cnf
spells a dash line over a commented block, the inline
source having no room for comments and the dashed spelling
no room for blocks.

```
from-ast: "advised-text"
reformat:
    ignore-comments: false
```

### cnf

```
l:
    ---
    a:
        # a.head

        b: 1
    ---
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"l","pos":{"l":1,"c":1}}],
            "value": {
                "list": [{
                    "value": {
                        "map": [{
                            "key": [{"str":"a","pos":{"l":2,"c":5}}],
                            "headcmt": {"cmt":["# a.head"],"pos":{"l":3,"c":9}},
                            "value": {
                                "map": [{
                                    "key": [{"str":"b","pos":{"l":4,"c":13}}],
                                    "value": {"num":"1","pos":{"l":4,"c":16}}
                                }],
                                "block": true,
                                "pos": {"l":4,"c":13}
                            },
                            "indent": "    "
                        }],
                        "block": true,
                        "pos": {"l":2,"c":5}
                    }
                }],
                "dashed": true,
                "block": true,
                "pos": {"l":1,"c":4}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: unfold-list-chain-comments-ignore

The same tree under ignore-comments folds the chain flat:
the dash line spells the keys of the levels it walked past,
and the head run - which the dash line has no position to
write - drops. The knob buys the dashed spelling at the
price of the run, the old behavior pinned.

```
from-ast: "advised-text"
reformat:
    ignore-comments: true
```

### cnf

```
l:
    - a.b: 1
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"l","pos":{"l":1,"c":1}}],
            "value": {
                "list": [{
                    "value": {
                        "map": [{
                            "key": [{"str":"a","pos":{"l":2,"c":5}}],
                            "headcmt": {"cmt":["# a.head"],"pos":{"l":3,"c":9}},
                            "value": {
                                "map": [{
                                    "key": [{"str":"b","pos":{"l":4,"c":13}}],
                                    "value": {"num":"1","pos":{"l":4,"c":16}}
                                }],
                                "block": true,
                                "pos": {"l":4,"c":13}
                            },
                            "indent": "    "
                        }],
                        "block": true,
                        "pos": {"l":2,"c":5}
                    }
                }],
                "dashed": true,
                "block": true,
                "pos": {"l":1,"c":4}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: fold-map-comments

The inline form of a map has no position for the comments
its pairs carry: the one-line spelling writes pairs only,
and the runs around them would die at render. A map whose
pairs hold any run therefore keeps its block form under
the fold threshold - the layout fold refuses, the same way
the path fold refuses a commented line. A three-pair map
with a pre run above its second pair and an in run on it
keeps its block layout under a threshold that would fold
it inline.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-map-upto-items: 3
    ignore-comments: false
```

### cnf

```
a: 1
# b.pre
b: 2 # b.in
c: 3
```

### json

```
{"a":1,"b":2,"c":3}
```

### ast

```
a: 1
# b.pre
b: 2 # b.in
c: 3
```


## Case: fold-map-comments-ignore

The same map under ignore-comments folds inline: the runs
have no position in the one-line spelling, and the knob
drops them - the pinned old behavior of the layout fold,
the one the escape hatch exists for.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-map-upto-items: 3
    ignore-comments: true
```

### cnf

```
{a: 1, b: 2, c: 3}
```

### json

```
{"a":1,"b":2,"c":3}
```

### ast

```
a: 1
# b.pre
b: 2 # b.in
c: 3
```


## Case: fold-list-comments

The same rule holds for a list's dashed form: an element's
runs have no position in the inline spelling, so a list
whose elements carry any run keeps its block form. A
three-element dashed list with a pre run above its second
element keeps its dashes under a threshold that would
fold it inline.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-list-upto-items: 3
    ignore-comments: false
```

### cnf

```
l:
    - 1
    # two.pre
    - 2
    - 3
```

### json

```
{"l":[1,2,3]}
```

### ast

```
l:
    - 1
    # two.pre
    - 2
    - 3
```


## Case: fold-list-comments-ignore

The same list under ignore-comments folds inline, the pre
run dropping - the dash spelling's price under the knob.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-list-upto-items: 3
    ignore-comments: true
```

### cnf

```
l: [1, 2, 3]
```

### json

```
{"l":[1,2,3]}
```

### ast

```
l:
    - 1
    # two.pre
    - 2
    - 3
```


## Case: fold-comments-enclosing

The guard sees past the immediate children: the head and
foot runs riding the pair whose block the fold would spell
inline belong to that same subtree, and the inline spelling
loses them with the children's own runs. A pair whose block
carries a head run keeps its block layout even under a
threshold that counts its two pairs as foldable - the head
run describes the block, and the one-line spelling of the
block drops it.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-map-upto-items: 3
    ignore-comments: false
```

### cnf

```
a:
    # a.head
    x: 1
    y: 2
```

### json

```
{"a":{"x":1,"y":2}}
```

### ast

```
a:
    # a.head
    x: 1
    y: 2
```


## Case: fold-comments-enclosing-ignore

The same pair under ignore-comments folds inline - the head
run dropping with the block spelling, the knob's price.

```
to-json: "exact-value"
from-ast: "advised-text"
parse-ast: true
reformat:
    fold-map-upto-items: 3
    ignore-comments: true
```

### cnf

```
{a: {x: 1, y: 2}}
```

### json

```
{"a":{"x":1,"y":2}}
```

### ast

```
a:
    # a.head
    x: 1
    y: 2
```
