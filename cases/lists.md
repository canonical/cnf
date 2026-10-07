# Topic

Inline lists, block lists, lists of key-value pairs, and list fences for multi-line elements.


## Case: inline-lists

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
inline: [[1, 2], [3, 4]]
```

### json

```
{"inline":[[1,2],[3,4]]}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"inline","pos":{"l":1,"c":1}}],
            "value": {
                "list": [{
                    "value": {
                        "list": [{"value":{"num":"1","pos":{"l":1,"c":11}}}, {"value":{"num":"2","pos":{"l":1,"c":14}}}],
                        "pos": {"l":1,"c":10}
                    }
                }, {
                    "value": {
                        "list": [{"value":{"num":"3","pos":{"l":1,"c":19}}}, {"value":{"num":"4","pos":{"l":1,"c":22}}}],
                        "pos": {"l":1,"c":18}
                    }
                }],
                "pos": {"l":1,"c":9}
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: inline-list-unclosed-error

An inline list that never closes on its line is a syntax
error, named where the list opened - so nested lists tell
apart: the error names the list that ran out of line.

```
to-json: "error"
to-ast: "error"
error: {regexp: "^1:${open:c}: cannot find closing '\\]'$"}
vars:
    open: {regexp: "(\\[1)"}
```

### cnf

```
a: [1, 2
```

```
a: [2, [1
```


## Case: inline-list-separator-error

A list element must be followed by a comma or the closing
bracket; anything else is named at its own first column.

```
to-json: "error"
to-ast: "error"
error: {regexp: "^1:${off:c}: cannot parse \"${off}\" inside list$"}
vars:
    off: {regexp: "(; 2\\]|2\\])"}
```

### cnf

```
a: [1 2]
```

```
a: [1; 2]
```


## Case: block-lists


```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
block:
    - "first item"
    - 2
```

### json

```
{"block":["first item",2]}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"block","pos":{"l":1,"c":1}}],
            "value": {
                "list": [{
                    "value": {
                        "str": "first item",
                        "pos": {"l":2,"c":7},
                        "quote": 2
                    }
                }, {"value":{"num":"2","pos":{"l":3,"c":7}}}],
                "dashed": true,
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


## Case: block-lists-no-value-error

A dashed item with nothing after the dash has no value.

```
to-json: "error"
to-ast: "error"
error: "2:5: cannot omit the value"
```

### cnf

```
block:
    -
```


## Case: block-lists-no-value-key-error

The inlinepair form of a dashed item needs its value after
the colon, and the error spells the key.

```
to-json: "error"
to-ast: "error"
error: "2:5: cannot omit the value for `a`"
```

### cnf

```
block:
    - a:
```


## Case: empty-inline-list

An empty list renders as inline [].

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
l: []
```

### json

```
{"l":[]}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"l","pos":{"l":1,"c":1}}],
            "value": {"list":[],"pos":{"l":1,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: list-of-maps

A block list of maps: each - item is a map with a label.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
block.maps:
    - label: "first map"
    - label: "second map"
```

### json

```
{"block":{"maps":[{"label":"first map"},{"label":"second map"}]}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"block","pos":{"l":1,"c":1}}, {"str":"maps","pos":{"l":1,"c":7}}],
            "value": {
                "list": [{
                    "key": [{"str":"label","pos":{"l":2,"c":7}}],
                    "value": {
                        "str": "first map",
                        "pos": {"l":2,"c":14},
                        "quote": 2
                    }
                }, {
                    "key": [{"str":"label","pos":{"l":3,"c":7}}],
                    "value": {
                        "str": "second map",
                        "pos": {"l":3,"c":14},
                        "quote": 2
                    }
                }],
                "dashed": true,
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


## Case: inline-list-of-maps

A short list of maps fits inline; a long one expands to block form.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
l: [{label: "first map"}, {label: "second map"}]
```

### json

```
{"l":[{"label":"first map"},{"label":"second map"}]}
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
                            "key": [{"str":"label","pos":{"l":1,"c":6}}],
                            "value": {
                                "str": "first map",
                                "pos": {"l":1,"c":13},
                                "quote": 2
                            }
                        }],
                        "pos": {"l":1,"c":5}
                    }
                }, {
                    "value": {
                        "map": [{
                            "key": [{"str":"label","pos":{"l":1,"c":28}}],
                            "value": {
                                "str": "second map",
                                "pos": {"l":1,"c":35},
                                "quote": 2
                            }
                        }],
                        "pos": {"l":1,"c":27}
                    }
                }],
                "pos": {"l":1,"c":4}
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: list-fences

List fences (---) separate heterogeneous list elements: lists and maps.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
block.nested.lists:
    ---
    - 1
    ---
    - 2
    - 3
    ---

block.nested.maps:
    ---
    a: 1
    ---
    b: 2
    c: 3
    ---
```

### json

```
{"block":{"nested":{"lists":[[1],[2,3]],"maps":[{"a":1},{"b":2,"c":3}]}}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"block","pos":{"l":1,"c":1}}, {"str":"nested","pos":{"l":1,"c":7}}, {"str":"lists","pos":{"l":1,"c":14}}],
            "value": {
                "list": [{
                    "value": {
                        "list": [{"value":{"num":"1","pos":{"l":3,"c":7}}}],
                        "dashed": true,
                        "block": true,
                        "pos": {"l":3,"c":5}
                    }
                }, {
                    "value": {
                        "list": [{"value":{"num":"2","pos":{"l":5,"c":7}}}, {"value":{"num":"3","pos":{"l":6,"c":7}}}],
                        "dashed": true,
                        "block": true,
                        "pos": {"l":5,"c":5}
                    }
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"block","pos":{"l":9,"c":1}}, {"str":"nested","pos":{"l":9,"c":7}}, {"str":"maps","pos":{"l":9,"c":14}}],
            "precmt": {"cmt":[""],"pos":{"l":8,"c":1}},
            "value": {
                "list": [{
                    "value": {
                        "map": [{
                            "key": [{"str":"a","pos":{"l":11,"c":5}}],
                            "value": {"num":"1","pos":{"l":11,"c":8}}
                        }],
                        "block": true,
                        "pos": {"l":11,"c":5}
                    }
                }, {
                    "value": {
                        "map": [{
                            "key": [{"str":"b","pos":{"l":13,"c":5}}],
                            "value": {"num":"2","pos":{"l":13,"c":8}}
                        }, {
                            "key": [{"str":"c","pos":{"l":14,"c":5}}],
                            "value": {"num":"3","pos":{"l":14,"c":8}}
                        }],
                        "block": true,
                        "pos": {"l":13,"c":5}
                    }
                }],
                "block": true,
                "pos": {"l":10,"c":5}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: list-fences-nested

Indented list fences support lists inside lists inside lists.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
block.nested.lists:
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

block.nested.maps:
    ---
        ---
        a: 1
        ---
        b: 2
        c: 3
        ---
    ---
        ---
        d: 4
        ---
    ---
```

### json

```
{"block":{"nested":{"lists":[[[1],[2,3]],[[4]]],"maps":[[{"a":1},{"b":2,"c":3}],[{"d":4}]]}}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"block","pos":{"l":1,"c":1}}, {"str":"nested","pos":{"l":1,"c":7}}, {"str":"lists","pos":{"l":1,"c":14}}],
            "value": {
                "list": [{
                    "value": {
                        "list": [{
                            "value": {
                                "list": [{"value":{"num":"1","pos":{"l":4,"c":11}}}],
                                "dashed": true,
                                "block": true,
                                "pos": {"l":4,"c":9}
                            }
                        }, {
                            "value": {
                                "list": [{"value":{"num":"2","pos":{"l":6,"c":11}}}, {"value":{"num":"3","pos":{"l":7,"c":11}}}],
                                "dashed": true,
                                "block": true,
                                "pos": {"l":6,"c":9}
                            }
                        }],
                        "block": true,
                        "pos": {"l":3,"c":9}
                    },
                    "indent": "    "
                }, {
                    "value": {
                        "list": [{
                            "value": {
                                "list": [{
                                    "value": {"num":"4","pos":{"l":11,"c":11}}
                                }],
                                "dashed": true,
                                "block": true,
                                "pos": {"l":11,"c":9}
                            }
                        }],
                        "block": true,
                        "pos": {"l":10,"c":9}
                    },
                    "indent": "    "
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"block","pos":{"l":15,"c":1}}, {"str":"nested","pos":{"l":15,"c":7}}, {"str":"maps","pos":{"l":15,"c":14}}],
            "precmt": {"cmt":[""],"pos":{"l":14,"c":1}},
            "value": {
                "list": [{
                    "value": {
                        "list": [{
                            "value": {
                                "map": [{
                                    "key": [{"str":"a","pos":{"l":18,"c":9}}],
                                    "value": {"num":"1","pos":{"l":18,"c":12}}
                                }],
                                "block": true,
                                "pos": {"l":18,"c":9}
                            }
                        }, {
                            "value": {
                                "map": [{
                                    "key": [{"str":"b","pos":{"l":20,"c":9}}],
                                    "value": {"num":"2","pos":{"l":20,"c":12}}
                                }, {
                                    "key": [{"str":"c","pos":{"l":21,"c":9}}],
                                    "value": {"num":"3","pos":{"l":21,"c":12}}
                                }],
                                "block": true,
                                "pos": {"l":20,"c":9}
                            }
                        }],
                        "block": true,
                        "pos": {"l":17,"c":9}
                    },
                    "indent": "    "
                }, {
                    "value": {
                        "list": [{
                            "value": {
                                "map": [{
                                    "key": [{"str":"d","pos":{"l":25,"c":9}}],
                                    "value": {"num":"4","pos":{"l":25,"c":12}}
                                }],
                                "block": true,
                                "pos": {"l":25,"c":9}
                            }
                        }],
                        "block": true,
                        "pos": {"l":24,"c":9}
                    },
                    "indent": "    "
                }],
                "block": true,
                "pos": {"l":16,"c":5}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: list-fences-nested-error

A list fence element that is a dashed list, a map, or an inline
item cannot sit at a deeper indent than the fence's delimiters:
only a nested list fence opens a deeper block inside a fence.

```
to-json: "error"
to-ast: "error"
error: "3:9: list element indent must align with fence"
```

### cnf

```
block.nested.lists:
    ---
        - 1
        - 2
    ---
```

### cnf

```
block.nested.maps:
    ---
        a: 1
        b: 2
    ---
```

### cnf

```
block.nested.inline:
    ---
        1
    ---
```


## Case: list-fenced-indent-error

A nested list fence's indent must be the same across its own
items: the opening `---` line sets the fence's indent, and every
delimiter and item line inside it must repeat that exact prefix.
A fence whose items sit at differing indents is a syntax error.

```
to-json: "error"
to-ast: "error"
error: "6:3: cannot mix indents inside list fence"
```

### cnf

```
---
    ---
    - 1
    ---
---
  ---
  - 2
  ---
---
```


## Case: list-fences-empty-error

A list fence element cannot be empty: the lines between two ---
delimiters hold one element, and no lines hold none.

```
to-json: "error"
to-ast: "error"
error: "${empty:lc}: cannot parse empty list fence element"
vars:
    empty: "---\n---"
```

### cnf

```
---
---
```

```
---
- 1
---
---
```


## Case: list-fences-empty-nested-error

An empty element inside a nested fence: the nested fence's own
empty pair, not the outer fence's delimiters.

```
to-json: "error"
to-ast: "error"
error: "2:5: cannot parse empty list fence element"
```

### cnf

```
---
    ---
    ---
---
```


## Case: list-fence-parent-blank-error

A list fence must open at the very next line after its
pair: a blank between them has no home - not the fence's
content, which is the list element, not the pair's line,
which is above.

```
to-json: "error"
to-ast: "error"
error: "2:1: cannot blank between key and fence"
```

### cnf

```
l:

    ---
    x
    ---
```


## Case: inline-list-block-layout-error

An inline list renders its elements on the one line: an element
item claiming a block layout inside it is a malformed AST, and
the render rejects it instead of silently normalizing the
form.

```
from-ast: "error"
error: "cannot render a block value inside an inline list"
```

### ast

```
{
    "value": {
        "list": [{
            "value": {"num":"1","pos":{"l":1,"c":2}}
        }, {
            "value": {"map":[],"block":true,"pos":{"l":1,"c":5}}
        }],
        "pos": {"l":1,"c":1}
    }
}
```


## Case: empty-dashed-list-inline

A dashed block list with zero items has no block form: the
dashed spelling opens on the lines under the pair, and with
none the list renders as the inline empty list instead, the
one spelling the grammar gives an empty list in this shape.

```
from-ast: "advised-text"
```

### cnf

```
a: []
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"list":[],"pos":{"l":1,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```
