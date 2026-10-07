# Topic

Top-level values: the file is one value, not only a map - scalars, lists, fences, and the empty file.


## Case: empty-file-map

A completely empty file is an empty map.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
```

### json

```
{}
```

### ast

```
{
    "value": {
        "map": [],
        "block": true,
        "pos": {"l":0,"c":0}
    }
}
```


## Case: empty-inline-map

The AST can distinguish between an empty file from an empty inline map.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
{}
```

### json

```
{}
```

### ast

```
{"value":{"map":[],"pos":{"l":1,"c":1}}}
```


## Case: number

Top-level scalar and bare values: string, number, bool, null.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
42
```

### json

```
42
```

### ast

```
{"value":{"num":"42","pos":{"l":1,"c":1}}}
```


## Case: string

A top-level string value.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
"hello"
```

### json

```
"hello"
```

### ast

```
{
    "value": {
        "str": "hello",
        "pos": {"l":1,"c":1},
        "quote": 2
    }
}
```


## Case: bool

Top-level booleans and null.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
true
```

### json

```
true
```

### ast

```
{
    "value": {"scalar":"true","pos":{"l":1,"c":1}}
}
```


## Case: null

A top-level null value.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
null
```

### json

```
null
```

### ast

```
{
    "value": {"scalar":"null","pos":{"l":1,"c":1}}
}
```

## Case: inline-list

A top-level inline list.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```


### cnf

```
[1, 2]
```

### json

```
[1,2]
```

### ast

```
{
    "value": {
        "list": [{"value":{"num":"1","pos":{"l":1,"c":2}}}, {"value":{"num":"2","pos":{"l":1,"c":5}}}],
        "pos": {"l":1,"c":1}
    }
}
```


## Case: inline-map

A top-level inline map.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
{a: 1, b: 2}
```

### json

```
{"a":1,"b":2}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":2}}],
            "value": {"num":"1","pos":{"l":1,"c":5}}
        }, {
            "key": [{"str":"b","pos":{"l":1,"c":8}}],
            "value": {"num":"2","pos":{"l":1,"c":11}}
        }],
        "pos": {"l":1,"c":1}
    }
}
```


## Case: block-list

A top-level block list of strings.

```
to-json: "exact-value"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
- "one"
- "two"
```

### json

```
["one","two"]
```

### ast

```
{
    "value": {
        "list": [{
            "value": {
                "str": "one",
                "pos": {"l":1,"c":3},
                "quote": 2
            }
        }, {
            "value": {
                "str": "two",
                "pos": {"l":2,"c":3},
                "quote": 2
            }
        }],
        "dashed": true,
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: list-fences

Top-level list fences separate heterogeneous list elements.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
---
- "first list"
---
- "second list"
---
```

### json

```
[["first list"],["second list"]]
```

### ast

```
{
    "value": {
        "list": [{
            "value": {
                "list": [{
                    "value": {
                        "str": "first list",
                        "pos": {"l":2,"c":3},
                        "quote": 2
                    }
                }],
                "dashed": true,
                "block": true,
                "pos": {"l":2,"c":1}
            }
        }, {
            "value": {
                "list": [{
                    "value": {
                        "str": "second list",
                        "pos": {"l":4,"c":3},
                        "quote": 2
                    }
                }],
                "dashed": true,
                "block": true,
                "pos": {"l":4,"c":1}
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```

## Case: string-fence

A top-level raw fence string.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
'''
line one
line two
'''
```

### json

```
"line one\nline two\n"
```

### ast

```
{
    "value": {
        "str": "line one\nline two\n",
        "pos": {"l":1,"c":1},
        "quote": 3,
        "block": true
    }
}
```


## Case: trailing-content-error

Trailing content after a top-level value is rejected.

```
to-json: "error"
to-ast: "error"
error: "2:1: cannot parse content after top-level value: \"2\""
```

### cnf

```
1
2
```


## Case: trailing-content-long-error

A long trailing line is quoted up to five characters, then
elided.

```
to-json: "error"
to-ast: "error"
error: "2:1: cannot parse content after top-level value: \"hello...\""
```

### cnf

```
1
hello world
```


## Case: top-fence-indent-error

The file's value sits outside any INDENT: a top-level
fence indented past column zero has no form to parse as,
named at its opening line.

```
to-json: "error"
to-ast: "error"
error: "1:3: cannot indent a top-level value"
```

### cnf

```
  """
  a
  """
```


## Case: top-list-fence-indent-error

The same rule for a top-level list fence: the opening ---
indented past column zero rejects at its own start.

```
to-json: "error"
to-ast: "error"
error: "1:3: cannot indent a top-level value"
```

### cnf

```
  ---
  a: 1
  ---
```
