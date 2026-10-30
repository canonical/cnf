# Topic

Maps: pairs, keys and paths, the inline and block forms, merging and conflicts, and the writer's folding and layout advice.


## Case: simple-keys

The simplest pairs: a string, a number, and the bare keywords.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
name: "now"
timeout: 30
enabled: true
verbose: false
nothing: null
```

### json

```
{"name":"now","timeout":30,"enabled":true,"verbose":false,"nothing":null}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"name","pos":{"l":1,"c":1}}],
            "value": {
                "str": "now",
                "pos": {"l":1,"c":7},
                "quote": 2
            }
        }, {
            "key": [{"str":"timeout","pos":{"l":2,"c":1}}],
            "value": {"num":"30","pos":{"l":2,"c":10}}
        }, {
            "key": [{"str":"enabled","pos":{"l":3,"c":1}}],
            "value": {"scalar":"true","pos":{"l":3,"c":10}}
        }, {
            "key": [{"str":"verbose","pos":{"l":4,"c":1}}],
            "value": {"scalar":"false","pos":{"l":4,"c":10}}
        }, {
            "key": [{"str":"nothing","pos":{"l":5,"c":1}}],
            "value": {"scalar":"null","pos":{"l":5,"c":10}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```



## Case: ordered-keys

Key order is preserved in both directions, however scrambled. All values are bare scalars, whose form is fully determined - only the order can vary.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
zebra: 1
apple: 2
mango: 3
banana: true
cherry: null
date: 4
elderberry: 5
fig: 6
```

### json

```
{"zebra":1,"apple":2,"mango":3,"banana":true,"cherry":null,"date":4,"elderberry":5,"fig":6}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"zebra","pos":{"l":1,"c":1}}],
            "value": {"num":"1","pos":{"l":1,"c":8}}
        }, {
            "key": [{"str":"apple","pos":{"l":2,"c":1}}],
            "value": {"num":"2","pos":{"l":2,"c":8}}
        }, {
            "key": [{"str":"mango","pos":{"l":3,"c":1}}],
            "value": {"num":"3","pos":{"l":3,"c":8}}
        }, {
            "key": [{"str":"banana","pos":{"l":4,"c":1}}],
            "value": {"scalar":"true","pos":{"l":4,"c":9}}
        }, {
            "key": [{"str":"cherry","pos":{"l":5,"c":1}}],
            "value": {"scalar":"null","pos":{"l":5,"c":9}}
        }, {
            "key": [{"str":"date","pos":{"l":6,"c":1}}],
            "value": {"num":"4","pos":{"l":6,"c":7}}
        }, {
            "key": [{"str":"elderberry","pos":{"l":7,"c":1}}],
            "value": {"num":"5","pos":{"l":7,"c":13}}
        }, {
            "key": [{"str":"fig","pos":{"l":8,"c":1}}],
            "value": {"num":"6","pos":{"l":8,"c":6}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```



## Case: quoted-keys

Quoted keys and dotted paths fold into nested maps; either quote family spells a key.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
"double": 1
'single': 2
"line1\nline2": 3
'line1\nline1': 4
"space double": 5
'space single': 6
'path'."with space": 7
```

### json

```
{"double":1,"single":2,"line1\nline2":3,"line1\\nline1":4,"space double":5,"space single":6,"path":{"with space":7}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{
                "str": "double",
                "pos": {"l":1,"c":1},
                "quote": 2
            }],
            "value": {"num":"1","pos":{"l":1,"c":11}}
        }, {
            "key": [{
                "str": "single",
                "pos": {"l":2,"c":1},
                "quote": 1
            }],
            "value": {"num":"2","pos":{"l":2,"c":11}}
        }, {
            "key": [{
                "str": "line1\nline2",
                "pos": {"l":3,"c":1},
                "quote": 2
            }],
            "value": {"num":"3","pos":{"l":3,"c":17}}
        }, {
            "key": [{
                "str": "line1\\nline1",
                "pos": {"l":4,"c":1},
                "quote": 1
            }],
            "value": {"num":"4","pos":{"l":4,"c":17}}
        }, {
            "key": [{
                "str": "space double",
                "pos": {"l":5,"c":1},
                "quote": 2
            }],
            "value": {"num":"5","pos":{"l":5,"c":17}}
        }, {
            "key": [{
                "str": "space single",
                "pos": {"l":6,"c":1},
                "quote": 1
            }],
            "value": {"num":"6","pos":{"l":6,"c":17}}
        }, {
            "key": [{
                "str": "path",
                "pos": {"l":7,"c":1},
                "quote": 1
            }, {
                "str": "with space",
                "pos": {"l":7,"c":8},
                "quote": 2
            }],
            "value": {"num":"7","pos":{"l":7,"c":22}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```




## Case: dotted-keys

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a.b: 1
a.c.d: 2
a.c."e f": 3
```

### json

```
{"a":{"b":1,"c":{"d":2,"e f":3}}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}, {"str":"b","pos":{"l":1,"c":3}}],
            "value": {"num":"1","pos":{"l":1,"c":6}}
        }, {
            "key": [{"str":"a","pos":{"l":2,"c":1}}, {"str":"c","pos":{"l":2,"c":3}}, {"str":"d","pos":{"l":2,"c":5}}],
            "value": {"num":"2","pos":{"l":2,"c":8}}
        }, {
            "key": [{"str":"a","pos":{"l":3,"c":1}}, {"str":"c","pos":{"l":3,"c":3}}, {
                "str": "e f",
                "pos": {"l":3,"c":5},
                "quote": 2
            }],
            "value": {"num":"3","pos":{"l":3,"c":12}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```




## Case: key-conflict-error

Redefining a key is a conflict: scalar-over-scalar, same value, map-over-scalar, scalar-over-map.
The conflict is in the parse; the AST is never built.

```
to-json: "error"
to-ast: "error"
error: "2:1: cannot redefine previous key `a`"
```

### cnf

```
a: 1
a: 2
```


## Case: inline-map-expand-error

A map cannot be expanded more than once: two inline maps under the same key are an error, empty or not, even though their own keys do not conflict. The conflict is in the parse; the AST is never built.

```
to-json: "error"
to-ast: "error"
error: "2:1: cannot expand map key `a` more than once"
```

### cnf

```
a: {b: 1}
a: {c: 2}
```

```
a: {}
a: {}
```



## Case: json-key-conflict-error

Duplicate keys in JSON are rejected in the interpretation. This is a JSON-side error,
but an implementation that supports translation must not ignore it.

```
from-json: "error"
error: "cannot redefine key \"a\""
```

### json

```
{"a":1,"a":2}
```


## Case: bare-value-error

A bare word is never a value: unquoted text after the colon is a syntax error.

```
to-json: "error"
to-ast: "error"
error: "1:4: cannot parse unquoted value \"bare\""
```

### cnf

```
a: bare
```


## Case: stray-bracket-value-error

A value starting outside every scanned class - a stray
closing bracket, say - has no token to read: the error
names the empty token's start.

```
to-json: "error"
to-ast: "error"
error: "1:4: cannot parse value at \")\""
```

### cnf

```
a: )
```


## Case: no-value-error

A key-value pair needs an explicit value.

```
to-json: "error"
to-ast: "error"
error: {regexp: "^2:5: cannot omit the value for (`b`|`b\\.c`)$"}
```

### cnf

```
a:
    b:
```

```
a:
    b.c:
``` 


## Case: lone-bare-error

A lone bare without the colon and value is an error.

```
to-json: "error"
to-ast: "error"
error: {regexp: "^1:1: cannot parse unquoted value (\"a\"|\"a\\.b\")$"}
```

### cnf

```
a
```

```
a.b
```


## Case: spaced-key-error

Spaces are not allowed between key segments, or trailing after the key.

```
to-json: "error"
to-ast: "error"
error: "1:${space:c}: cannot parse space in path"
vars:
    space: " "
```

### cnf

```
a .b: 1
```

```
a. b: 1
```

```
a.b : 1
```


## Case: empty-key-error-a

```
to-json: "error"
to-ast: "error"
error: "1:3: cannot parse empty key segment"
```

### cnf

```
a..b: 1
```


## Case: empty-key-error-b

```
to-json: "error"
to-ast: "error"
error: "1:1: cannot parse empty key segment"
```

### cnf

```
.b: 1
```


## Case: empty-key-error-c

```
to-json: "error"
to-ast: "error"
error: "1:3: cannot parse empty key segment"
```

### cnf

```
a.: 1
```


## Case: empty-key-error-d

```
to-json: "error"
to-ast: "error"
error: "2:7: cannot parse empty key segment"
```

### cnf

```
z:
    a..b:
```


## Case: empty-key-error-e

```
to-json: "error"
to-ast: "error"
error: "2:5: cannot parse empty key segment"
```

### cnf

```
z:
    .b:
```


## Case: empty-key-error-f

```
to-json: "error"
to-ast: "error"
error: "2:7: cannot parse empty key segment"
```

### cnf

```
z:
    a.:
```


## Case: inline-maps

Inline maps: bare, quoted, and mixed dotted keys.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
inline: {a: 1, b: 2}
quoted: {"c d": 2}
path: {ab."c d": 3}
```

### json

```
{"inline":{"a":1,"b":2},"quoted":{"c d":2},"path":{"ab":{"c d":3}}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"inline","pos":{"l":1,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"a","pos":{"l":1,"c":10}}],
                    "value": {"num":"1","pos":{"l":1,"c":13}}
                }, {
                    "key": [{"str":"b","pos":{"l":1,"c":16}}],
                    "value": {"num":"2","pos":{"l":1,"c":19}}
                }],
                "pos": {"l":1,"c":9}
            }
        }, {
            "key": [{"str":"quoted","pos":{"l":2,"c":1}}],
            "value": {
                "map": [{
                    "key": [{
                        "str": "c d",
                        "pos": {"l":2,"c":10},
                        "quote": 2
                    }],
                    "value": {"num":"2","pos":{"l":2,"c":17}}
                }],
                "pos": {"l":2,"c":9}
            }
        }, {
            "key": [{"str":"path","pos":{"l":3,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"ab","pos":{"l":3,"c":8}}, {
                        "str": "c d",
                        "pos": {"l":3,"c":11},
                        "quote": 2
                    }],
                    "value": {"num":"3","pos":{"l":3,"c":18}}
                }],
                "pos": {"l":3,"c":7}
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```




## Case: inline-empty-map

```
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
m: {}
```

### json

```
{"m":{}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"m","pos":{"l":1,"c":1}}],
            "value": {"map":[],"pos":{"l":1,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: inline-empty-key-error

```
to-json: "error"
to-ast: "error"
error: "1:7: cannot parse empty key segment"
```

### cnf

```
a: {b..c: 1}
```


## Case: inline-spaced-key-error

```
to-json: "error"
to-ast: "error"
error: "1:6: cannot parse space in path"
```

### cnf

```
a: {b 1}
```


## Case: inline-map-separator-error

```
to-json: "error"
to-ast: "error"
error: "1:6: cannot find key-value separator ':'"
```

### cnf

```
a: {b}
```


## Case: inline-map-unclosed-error

```
to-json: "error"
to-ast: "error"
error: "1:4: cannot find closing '}'"
```

### cnf

```
a: {b: 1
```


## Case: inline-map-separator-comma-error

```
to-json: "error"
to-ast: "error"
error: "1:10: cannot parse \"2}\" inside map"
```

### cnf

```
a: {b: 1 2}
```



## Case: inline-map-in-block-error

An empty map has no block form: a block map needs at least one
pair, so a bare {} on its own line under a key is a syntax error.
The empty map spells inline on the pair's line, always.

```
to-json: "error"
to-ast: "error"
error: "2:5: cannot parse key \"{\""
```

### cnf

```
m:
    {}
```


## Case: inline-value-in-block-error

An inline value has no block form: a block map needs at least
one pair, so a bare scalar on its own line under a key is a
syntax error. The value spells inline on the pair's line,
always.

```
to-json: "error"
to-ast: "error"
error: "2:6: cannot find key-value separator ':'"
```

### cnf

```
m:
    1
```


## Case: block-maps

A block map nested under a dotted path.

```
to-json: "exact-value"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a.b:
    c: 1
    d: 2
    e.f: 3
```

### json

```
{"a":{"b":{"c":1,"d":2,"e":{"f":3}}}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}, {"str":"b","pos":{"l":1,"c":3}}],
            "value": {
                "map": [{
                    "key": [{"str":"c","pos":{"l":2,"c":5}}],
                    "value": {"num":"1","pos":{"l":2,"c":8}}
                }, {
                    "key": [{"str":"d","pos":{"l":3,"c":5}}],
                    "value": {"num":"2","pos":{"l":3,"c":8}}
                }, {
                    "key": [{"str":"e","pos":{"l":4,"c":5}}, {"str":"f","pos":{"l":4,"c":7}}],
                    "value": {"num":"3","pos":{"l":4,"c":10}}
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


## Case: block-with-long-keys

Long keys should disable inlining by default. The pretty printer may
customize it, but it shouldn't inline that unless asked to.

TODO: Change from-json to "advised-text" when pretty printing is done.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa:
    xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx: 1
    yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy: 2
```

### json

```
{"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa":{"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx":1,"yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy":2}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{
                "str": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
                "pos": {"l":1,"c":1}
            }],
            "value": {
                "map": [{
                    "key": [{
                        "str": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
                        "pos": {"l":2,"c":5}
                    }],
                    "value": {"num":"1","pos":{"l":2,"c":42}}
                }, {
                    "key": [{
                        "str": "yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy",
                        "pos": {"l":3,"c":5}
                    }],
                    "value": {"num":"2","pos":{"l":3,"c":42}}
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


## Case: block-indents

Maps nested down and up through several levels: each block closes at its own prefix, and the pairs resume at the parent's.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b:
        c:
            d: 1
            e: 2
        f: 3
    g:
        h: 4
        i: 5
    j: 6
k:
    l: 7
```

### json

```
{"a":{"b":{"c":{"d":1,"e":2},"f":3},"g":{"h":4,"i":5},"j":6},"k":{"l":7}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"b","pos":{"l":2,"c":5}}],
                    "value": {
                        "map": [{
                            "key": [{"str":"c","pos":{"l":3,"c":9}}],
                            "value": {
                                "map": [{
                                    "key": [{"str":"d","pos":{"l":4,"c":13}}],
                                    "value": {"num":"1","pos":{"l":4,"c":16}}
                                }, {
                                    "key": [{"str":"e","pos":{"l":5,"c":13}}],
                                    "value": {"num":"2","pos":{"l":5,"c":16}}
                                }],
                                "block": true,
                                "pos": {"l":4,"c":13}
                            },
                            "indent": "    "
                        }, {
                            "key": [{"str":"f","pos":{"l":6,"c":9}}],
                            "value": {"num":"3","pos":{"l":6,"c":12}}
                        }],
                        "block": true,
                        "pos": {"l":3,"c":9}
                    },
                    "indent": "    "
                }, {
                    "key": [{"str":"g","pos":{"l":7,"c":5}}],
                    "value": {
                        "map": [{
                            "key": [{"str":"h","pos":{"l":8,"c":9}}],
                            "value": {"num":"4","pos":{"l":8,"c":12}}
                        }, {
                            "key": [{"str":"i","pos":{"l":9,"c":9}}],
                            "value": {"num":"5","pos":{"l":9,"c":12}}
                        }],
                        "block": true,
                        "pos": {"l":8,"c":9}
                    },
                    "indent": "    "
                }, {
                    "key": [{"str":"j","pos":{"l":10,"c":5}}],
                    "value": {"num":"6","pos":{"l":10,"c":8}}
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"k","pos":{"l":11,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"l","pos":{"l":12,"c":5}}],
                    "value": {"num":"7","pos":{"l":12,"c":8}}
                }],
                "block": true,
                "pos": {"l":12,"c":5}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: block-indents-siblings

Two sibling pairs of one map may open their blocks at
different indents: each block answers its own pair, the
grammar holding only that, and a shared map has no say. The
round trip carries each pair's own step: +4 under `a`, +8
under `b`, back to the top for `c`. The from-json direction
does not run: json carries no layout, so it cannot speak
about indents.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    x: 1
b:
        y: 2
c: 3
```

### json

```
{"a":{"x":1},"b":{"y":2},"c":3}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"x","pos":{"l":2,"c":5}}],
                    "value": {"num":"1","pos":{"l":2,"c":8}}
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"b","pos":{"l":3,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"y","pos":{"l":4,"c":9}}],
                    "value": {"num":"2","pos":{"l":4,"c":12}}
                }],
                "block": true,
                "pos": {"l":4,"c":9}
            },
            "indent": "        "
        }, {
            "key": [{"str":"c","pos":{"l":5,"c":1}}],
            "value": {"num":"3","pos":{"l":5,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: block-indent-spacing

The ast must be able to represent the indentation with precision.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
 b:
   c:
       d:
            e: 1
       f: 2
   g: 3
 h: 4
i: 5
```

### json

```
{"a":{"b":{"c":{"d":{"e":1},"f":2},"g":3},"h":4},"i":5}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"b","pos":{"l":2,"c":2}}],
                    "value": {
                        "map": [{
                            "key": [{"str":"c","pos":{"l":3,"c":4}}],
                            "value": {
                                "map": [{
                                    "key": [{"str":"d","pos":{"l":4,"c":8}}],
                                    "value": {
                                        "map": [{
                                            "key": [{"str":"e","pos":{"l":5,"c":13}}],
                                            "value": {"num":"1","pos":{"l":5,"c":16}}
                                        }],
                                        "block": true,
                                        "pos": {"l":5,"c":13}
                                    },
                                    "indent": "     "
                                }, {
                                    "key": [{"str":"f","pos":{"l":6,"c":8}}],
                                    "value": {"num":"2","pos":{"l":6,"c":11}}
                                }],
                                "block": true,
                                "pos": {"l":4,"c":8}
                            },
                            "indent": "    "
                        }, {
                            "key": [{"str":"g","pos":{"l":7,"c":4}}],
                            "value": {"num":"3","pos":{"l":7,"c":7}}
                        }],
                        "block": true,
                        "pos": {"l":3,"c":4}
                    },
                    "indent": "  "
                }, {
                    "key": [{"str":"h","pos":{"l":8,"c":2}}],
                    "value": {"num":"4","pos":{"l":8,"c":5}}
                }],
                "block": true,
                "pos": {"l":2,"c":2}
            },
            "indent": " "
        }, {
            "key": [{"str":"i","pos":{"l":9,"c":1}}],
            "value": {"num":"5","pos":{"l":9,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: block-indents-error

A deeper prefix than the block's is an indentation mismatch.

```
to-json: "error"
to-ast: "error"
error: "3:7: cannot mix indentation prefixes"
```

### cnf

```
a:
    b: 1
      c: 2
```


## Case: block-unindented-error

A block value must indent under its key: a fence, list fence, or
list at the same level as the key cannot open a block, so the key
is left without a value.

```
to-json: "error"
to-ast: "error"
error: "1:1: cannot omit the value for `a`"
```

### cnf

```
a:
"""
"""
```

```
a:
'''
'''
```

```
a:
---
1
---
```

```
a:
- 1
```


## Case: merge-maps-inline

Dotted paths merge with inline maps into one tree. The merge happens in the
interpretation: the AST keeps the three pairs as written.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a.b: 1
a: {c: 2}
a.d: {e.f: 3}
```

### json

```
{"a":{"b":1,"c":2,"d":{"e":{"f":3}}}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}, {"str":"b","pos":{"l":1,"c":3}}],
            "value": {"num":"1","pos":{"l":1,"c":6}}
        }, {
            "key": [{"str":"a","pos":{"l":2,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"c","pos":{"l":2,"c":5}}],
                    "value": {"num":"2","pos":{"l":2,"c":8}}
                }],
                "pos": {"l":2,"c":4}
            }
        }, {
            "key": [{"str":"a","pos":{"l":3,"c":1}}, {"str":"d","pos":{"l":3,"c":3}}],
            "value": {
                "map": [{
                    "key": [{"str":"e","pos":{"l":3,"c":7}}, {"str":"f","pos":{"l":3,"c":9}}],
                    "value": {"num":"3","pos":{"l":3,"c":12}}
                }],
                "pos": {"l":3,"c":6}
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: merge-maps-block

A dotted path merges into an already-open block map: the block.c pair folds back into the block opened above it.
The merge happens in the interpretation, while the AST keeps the two pairs as written.

```
from-json: "exact-value"
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b: 1
    c: 2
a.d: 3
```

### json

```
{"a":{"b":1,"c":2,"d":3}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"b","pos":{"l":2,"c":5}}],
                    "value": {"num":"1","pos":{"l":2,"c":8}}
                }, {
                    "key": [{"str":"c","pos":{"l":3,"c":5}}],
                    "value": {"num":"2","pos":{"l":3,"c":8}}
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"a","pos":{"l":4,"c":1}}, {"str":"d","pos":{"l":4,"c":3}}],
            "value": {"num":"3","pos":{"l":4,"c":6}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: merge-maps-apart

```
to-json: "exact-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b:
        c: 1
x: 9
a.b.d: 2
```

### json

```
{"a":{"b":{"c":1,"d":2}},"x":9}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"b","pos":{"l":2,"c":5}}],
                    "value": {
                        "map": [{
                            "key": [{"str":"c","pos":{"l":3,"c":9}}],
                            "value": {"num":"1","pos":{"l":3,"c":12}}
                        }],
                        "block": true,
                        "pos": {"l":3,"c":9}
                    },
                    "indent": "    "
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"x","pos":{"l":4,"c":1}}],
            "value": {"num":"9","pos":{"l":4,"c":4}}
        }, {
            "key": [{"str":"a","pos":{"l":5,"c":1}}, {"str":"b","pos":{"l":5,"c":3}}, {"str":"d","pos":{"l":5,"c":5}}],
            "value": {"num":"2","pos":{"l":5,"c":8}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: merge-maps-nested-path

A key path merges into an inline map nested at any depth, whether the path
comes from a later line or from the same inline map. The merge happens in the
interpretation: the AST keeps the pairs as written.

```
to-json: "advised-text"
from-ast: "advised-text"
```

### cnf

```
a: {b: {c: 1}}
a.b.d: 2
```

```
a: {b: {c: 1}, b.d: 2}
```

### json

```
{"a":{"b":{"c":1,"d":2}}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"b","pos":{"l":1,"c":5}}],
                    "value": {
                        "map": [{
                            "key": [{"str":"c","pos":{"l":1,"c":9}}],
                            "value": {"num":"1","pos":{"l":1,"c":12}}
                        }],
                        "pos": {"l":1,"c":8}
                    }
                }],
                "pos": {"l":1,"c":4}
            }
        }, {
            "key": [{"str":"a","pos":{"l":2,"c":1}}, {"str":"b","pos":{"l":2,"c":3}}, {"str":"d","pos":{"l":2,"c":5}}],
            "value": {"num":"2","pos":{"l":2,"c":8}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: inline-map-block-layout-error

An inline map renders its pairs on the one line: a pair item
claiming a block layout inside it is a malformed AST, and the
render rejects it instead of silently normalizing the form.

```
from-ast: "error"
error: "cannot render a block value inside an inline map"
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
            "value": {"map":[],"block":true,"pos":{"l":1,"c":11}}
        }],
        "pos": {"l":1,"c":1}
    }
}
```


## Case: empty-block-map-inline

A block map with zero items has no block form: the block
opens on the lines under the pair, and with none there the
map renders as the inline empty map instead, the only
spelling the grammar gives an empty map.

```
from-ast: "advised-text"
```

### cnf

```
a: {}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"map":[],"pos":{"l":1,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```
