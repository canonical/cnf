# Topic

Comments: standalone and trailing, and their attribution to the
pairs and maps around them.


## Case: top-empty

The top map can be empty and still have comments.

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
# top.head[0]
# top.head[1]

# top.head[3]
# top.head[4]

# top.head[6]
# top.head[7]
```

### json

```
{}
```

### ast

```
{
    "headcmt": {
        "cmt": ["# top.head[0]", "# top.head[1]", "", "# top.head[3]", "# top.head[4]", "", "# top.head[6]", "# top.head[7]"],
        "pos": {"l":1,"c":1}
    },
    "value": {
        "map": [],
        "block": true,
        "pos": {"l":0,"c":0}
    }
}
```

## Case: inline-comments

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a: 1 # a.in
b: 2  # b.in
c: 3   # c.in
d: 4	# d.in (tab!)
e: [5] # e.in
f: {f: 6} # f.in
g: " # x" # g.in
h: ' # x' # h.in
i: """ # x""" # i.in
j: ''' # x''' # j.in
k:
    ---
    7 # k.in
    ---
l:
    - 8 # l.in
m:
    - m: 9 # m.in
n:
    n: 10 # n.in
```


### json

```
{"a":1,"b":2,"c":3,"d":4,"e":[5],"f":{"f":6},"g":" # x","h":" # x","i":" # x","j":" # x","k":[7],"l":[8],"m":[{"m":9}],"n":{"n":10}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"num":"1","pos":{"l":1,"c":4}},
            "incmt": {"cmt":[" # a.in"],"pos":{"l":1,"c":5}}
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {"num":"2","pos":{"l":2,"c":4}},
            "incmt": {"cmt":["  # b.in"],"pos":{"l":2,"c":5}}
        }, {
            "key": [{"str":"c","pos":{"l":3,"c":1}}],
            "value": {"num":"3","pos":{"l":3,"c":4}},
            "incmt": {"cmt":["   # c.in"],"pos":{"l":3,"c":5}}
        }, {
            "key": [{"str":"d","pos":{"l":4,"c":1}}],
            "value": {"num":"4","pos":{"l":4,"c":4}},
            "incmt": {
                "cmt": ["\t# d.in (tab!)"],
                "pos": {"l":4,"c":5}
            }
        }, {
            "key": [{"str":"e","pos":{"l":5,"c":1}}],
            "value": {
                "list": [{"value":{"num":"5","pos":{"l":5,"c":5}}}],
                "pos": {"l":5,"c":4}
            },
            "incmt": {"cmt":[" # e.in"],"pos":{"l":5,"c":7}}
        }, {
            "key": [{"str":"f","pos":{"l":6,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"f","pos":{"l":6,"c":5}}],
                    "value": {"num":"6","pos":{"l":6,"c":8}}
                }],
                "pos": {"l":6,"c":4}
            },
            "incmt": {"cmt":[" # f.in"],"pos":{"l":6,"c":10}}
        }, {
            "key": [{"str":"g","pos":{"l":7,"c":1}}],
            "value": {
                "str": " # x",
                "pos": {"l":7,"c":4},
                "quote": 2
            },
            "incmt": {"cmt":[" # g.in"],"pos":{"l":7,"c":10}}
        }, {
            "key": [{"str":"h","pos":{"l":8,"c":1}}],
            "value": {
                "str": " # x",
                "pos": {"l":8,"c":4},
                "quote": 1
            },
            "incmt": {"cmt":[" # h.in"],"pos":{"l":8,"c":10}}
        }, {
            "key": [{"str":"i","pos":{"l":9,"c":1}}],
            "value": {
                "str": " # x",
                "pos": {"l":9,"c":4},
                "quote": 4
            },
            "incmt": {"cmt":[" # i.in"],"pos":{"l":9,"c":14}}
        }, {
            "key": [{"str":"j","pos":{"l":10,"c":1}}],
            "value": {
                "str": " # x",
                "pos": {"l":10,"c":4},
                "quote": 3
            },
            "incmt": {"cmt":[" # j.in"],"pos":{"l":10,"c":14}}
        }, {
            "key": [{"str":"k","pos":{"l":11,"c":1}}],
            "value": {
                "list": [{
                    "value": {"num":"7","pos":{"l":13,"c":5}},
                    "incmt": {"cmt":[" # k.in"],"pos":{"l":13,"c":6}}
                }],
                "block": true,
                "pos": {"l":12,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"l","pos":{"l":15,"c":1}}],
            "value": {
                "list": [{
                    "value": {"num":"8","pos":{"l":16,"c":7}},
                    "incmt": {"cmt":[" # l.in"],"pos":{"l":16,"c":8}}
                }],
                "dashed": true,
                "block": true,
                "pos": {"l":16,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"m","pos":{"l":17,"c":1}}],
            "value": {
                "list": [{
                    "key": [{"str":"m","pos":{"l":18,"c":7}}],
                    "value": {"num":"9","pos":{"l":18,"c":10}},
                    "incmt": {"cmt":[" # m.in"],"pos":{"l":18,"c":11}}
                }],
                "dashed": true,
                "block": true,
                "pos": {"l":18,"c":5}
            },
            "indent": "    "
        }, {
            "key": [{"str":"n","pos":{"l":19,"c":1}}],
            "value": {
                "map": [{
                    "key": [{"str":"n","pos":{"l":20,"c":5}}],
                    "value": {"num":"10","pos":{"l":20,"c":8}},
                    "incmt": {"cmt":[" # n.in"],"pos":{"l":20,"c":10}}
                }],
                "block": true,
                "pos": {"l":20,"c":5}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```




## Case: block-in

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b:
        c: 1 # c.in
        d: 2  # d.in
        e: 3   # e.in
```

### json

```
{"a":{"b":{"c":1,"d":2,"e":3}}}
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
                            "value": {"num":"1","pos":{"l":3,"c":12}},
                            "incmt": {"cmt":[" # c.in"],"pos":{"l":3,"c":13}}
                        }, {
                            "key": [{"str":"d","pos":{"l":4,"c":9}}],
                            "value": {"num":"2","pos":{"l":4,"c":12}},
                            "incmt": {"cmt":["  # d.in"],"pos":{"l":4,"c":13}}
                        }, {
                            "key": [{"str":"e","pos":{"l":5,"c":9}}],
                            "value": {"num":"3","pos":{"l":5,"c":12}},
                            "incmt": {"cmt":["   # e.in"],"pos":{"l":5,"c":13}}
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
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```





## Case: top-pre

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
# top.head[0]
# top.head[1]

# top.head[3]

# top.head[5]
# top.head[6]

# a.pre[0]
# a.pre[1]
a: 1
```

### json

```
{"a":1}
```

### ast

```
{
    "headcmt": {
        "cmt": ["# top.head[0]", "# top.head[1]", "", "# top.head[3]", "", "# top.head[5]", "# top.head[6]"],
        "pos": {"l":1,"c":1}
    },
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":11,"c":1}}],
            "precmt": {
                "cmt": ["# a.pre[0]","# a.pre[1]"],
                "pos": {"l":9,"c":1}
            },
            "value": {"num":"1","pos":{"l":11,"c":4}}
        }],
        "block": true,
        "pos": {"l":11,"c":1}
    }
}
```



## Case: top-post

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a: 1
# a.post[0]
# a.post[1]

# top.foot[0]

# top.foot[2]
# top.foot[3]

# top.foot[5]
# top.foot[6]
```

### json

```
{"a":1}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"num":"1","pos":{"l":1,"c":4}},
            "postcmt": {
                "cmt": ["# a.post[0]","# a.post[1]"],
                "pos": {"l":2,"c":1}
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    },
    "footcmt": {
        "cmt": ["# top.foot[0]", "", "# top.foot[2]", "# top.foot[3]", "", "# top.foot[5]", "# top.foot[6]"],
        "pos": {"l":5,"c":1}
    }
}
```




## Case: top-post-pre

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a: 1
# a.post[0]
# a.post[1]

# b.pre[0]

# b.pre[2]
# b.pre[3]

# b.pre[5]
# b.pre[6]
b: 2
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
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"num":"1","pos":{"l":1,"c":4}},
            "postcmt": {
                "cmt": ["# a.post[0]","# a.post[1]"],
                "pos": {"l":2,"c":1}
            }
        }, {
            "key": [{"str":"b","pos":{"l":12,"c":1}}],
            "precmt": {
                "cmt": ["# b.pre[0]", "", "# b.pre[2]", "# b.pre[3]", "", "# b.pre[5]", "# b.pre[6]"],
                "pos": {"l":5,"c":1}
            },
            "value": {"num":"2","pos":{"l":12,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```




## Case: block-pre

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b:
        # b.head[0]
        # b.head[1]

        # b.head[3]

        # b.head[5]
        # b.head[6]

        # c.pre[0]
        # c.pre[1]
        c: 1
```

### json

```
{"a":{"b":{"c":1}}}
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
                    "headcmt": {
                        "cmt": ["# b.head[0]", "# b.head[1]", "", "# b.head[3]", "", "# b.head[5]", "# b.head[6]"],
                        "pos": {"l":3,"c":9}
                    },
                    "value": {
                        "map": [{
                            "key": [{"str":"c","pos":{"l":13,"c":9}}],
                            "precmt": {
                                "cmt": ["# c.pre[0]","# c.pre[1]"],
                                "pos": {"l":11,"c":9}
                            },
                            "value": {"num":"1","pos":{"l":13,"c":12}}
                        }],
                        "block": true,
                        "pos": {"l":13,"c":9}
                    },
                    "indent": "    "
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




## Case: block-post

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b:
        c: 1
        # c.post[0]
        # c.post[1]

        # b.foot[0]

        # b.foot[2]
        # b.foot[3]

        # b.foot[5]
        # b.foot[6]
```

### json

```
{"a":{"b":{"c":1}}}
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
                            "value": {"num":"1","pos":{"l":3,"c":12}},
                            "postcmt": {
                                "cmt": ["# c.post[0]","# c.post[1]"],
                                "pos": {"l":4,"c":9}
                            }
                        }],
                        "block": true,
                        "pos": {"l":3,"c":9}
                    },
                    "footcmt": {
                        "cmt": ["# b.foot[0]", "", "# b.foot[2]", "# b.foot[3]", "", "# b.foot[5]", "# b.foot[6]"],
                        "pos": {"l":7,"c":9}
                    },
                    "indent": "    "
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





## Case: block-post-pre

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b:
        c: 1
        # c.post[0]
        # c.post[1]

        # d.pre[0]

        # d.pre[2]
        # d.pre[3]

        # d.pre[5]
        # d.pre[6]
        d: 2
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
                    "key": [{"str":"b","pos":{"l":2,"c":5}}],
                    "value": {
                        "map": [{
                            "key": [{"str":"c","pos":{"l":3,"c":9}}],
                            "value": {"num":"1","pos":{"l":3,"c":12}},
                            "postcmt": {
                                "cmt": ["# c.post[0]","# c.post[1]"],
                                "pos": {"l":4,"c":9}
                            }
                        }, {
                            "key": [{"str":"d","pos":{"l":14,"c":9}}],
                            "precmt": {
                                "cmt": ["# d.pre[0]", "", "# d.pre[2]", "# d.pre[3]", "", "# d.pre[5]", "# d.pre[6]"],
                                "pos": {"l":7,"c":9}
                            },
                            "value": {"num":"2","pos":{"l":14,"c":12}}
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
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: all-comments

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
# a.pre[0]
a:
    # a.head[0]

    b: 1 # b.in

    # a.foot[1]
# a.post[0]
```

### json

```
{"a":{"b":1}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":2,"c":1}}],
            "precmt": {"cmt":["# a.pre[0]"],"pos":{"l":1,"c":1}},
            "headcmt": {
                "cmt": ["# a.head[0]"],
                "pos": {"l":3,"c":5}
            },
            "value": {
                "map": [{
                    "key": [{"str":"b","pos":{"l":5,"c":5}}],
                    "value": {"num":"1","pos":{"l":5,"c":8}},
                    "incmt": {"cmt":[" # b.in"],"pos":{"l":5,"c":9}}
                }],
                "block": true,
                "pos": {"l":5,"c":5}
            },
            "footcmt": {
                "cmt": ["","# a.foot[1]"],
                "pos": {"l":6,"c":1}
            },
            "postcmt": {
                "cmt": ["# a.post[0]"],
                "pos": {"l":8,"c":1}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":2,"c":1}
    }
}
```

## Case: tie-break

Clarifies ambiguities when there are no spaces.

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
# a.pre[0]
# a.pre[1]
a:
   # b.pre[0]
   # b.pre[1]
   b: 1
   # c.pre[0]
   # c.pre[1]
   c: 2
   # c.post[0]
   # c.post[1]
# a.post[0]
# a.post[1]
```

### json

```
{"a":{"b":1,"c":2}}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":3,"c":1}}],
            "precmt": {
                "cmt": ["# a.pre[0]","# a.pre[1]"],
                "pos": {"l":1,"c":1}
            },
            "value": {
                "map": [{
                    "key": [{"str":"b","pos":{"l":6,"c":4}}],
                    "precmt": {
                        "cmt": ["# b.pre[0]","# b.pre[1]"],
                        "pos": {"l":4,"c":4}
                    },
                    "value": {"num":"1","pos":{"l":6,"c":7}}
                }, {
                    "key": [{"str":"c","pos":{"l":9,"c":4}}],
                    "precmt": {
                        "cmt": ["# c.pre[0]","# c.pre[1]"],
                        "pos": {"l":7,"c":4}
                    },
                    "value": {"num":"2","pos":{"l":9,"c":7}},
                    "postcmt": {
                        "cmt": ["# c.post[0]","# c.post[1]"],
                        "pos": {"l":10,"c":4}
                    }
                }],
                "block": true,
                "pos": {"l":6,"c":4}
            },
            "postcmt": {
                "cmt": ["# a.post[0]","# a.post[1]"],
                "pos": {"l":12,"c":1}
            },
            "indent": "   "
        }],
        "block": true,
        "pos": {"l":3,"c":1}
    }
}
```


## Case: block-dedenting

Comments dedent with their blocks: each closing block's post run
stops at the first blank, and the blank travels forward to the
next candidate - the parent's post or the next pair's pre.

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    b:
        c:
            d: 1
            # d.post[0]

            # c.foot[0]

            # c.foot[2]
        # c.post[0]

    # a.foot[1]

    # a.foot[3]
# a.post[0]

# top.foot[0]
```

### json

```
{"a":{"b":{"c":{"d":1}}}}
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
                                    "value": {"num":"1","pos":{"l":4,"c":16}},
                                    "postcmt": {
                                        "cmt": ["# d.post[0]"],
                                        "pos": {"l":5,"c":13}
                                    }
                                }],
                                "block": true,
                                "pos": {"l":4,"c":13}
                            },
                            "footcmt": {
                                "cmt": ["# c.foot[0]","","# c.foot[2]"],
                                "pos": {"l":7,"c":13}
                            },
                            "postcmt": {
                                "cmt": ["# c.post[0]"],
                                "pos": {"l":10,"c":9}
                            },
                            "indent": "    "
                        }],
                        "block": true,
                        "pos": {"l":3,"c":9}
                    },
                    "indent": "    "
                }],
                "block": true,
                "pos": {"l":2,"c":5}
            },
            "footcmt": {
                "cmt": ["","# a.foot[1]","","# a.foot[3]"],
                "pos": {"l":11,"c":1}
            },
            "postcmt": {
                "cmt": ["# a.post[0]"],
                "pos": {"l":15,"c":1}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    },
    "footcmt": {
        "cmt": ["# top.foot[0]"],
        "pos": {"l":17,"c":1}
    }
}
```




## Case: indent-error

A standalone comment sits at the prefix of the block it belongs
to, like a pair. A deeper comment opens a prefix no structural
line lands on, and a comment matching no enclosing block's
prefix is an error - the same class as a misindented pair.

```
to-json: "error"
to-ast: "error"
error: "${a:lc}: cannot mix indentation prefixes"
vars:
    a: "#"
```

### cnf

```
  # deeper before
a: 1
```

```
a: 1
  # deeper after
```

```
a:
      # deeper before
    b: 1
```

```
a:
    b: 1
      # deeper after
```

```
a:
  # shallower before
    b: 1
```

```
a:
    b: 1
  # shallower after
```



## Case: comments-on-fences-error

Inline comments are not recognized on a fence's opening or
closing line: both must be bare, and anything after them -
spaced or glued - is a syntax error. Inside the fence a # is
content.

```
to-json: "error"
to-ast: "error"
error: {regexp: "^2:(7|8|5): cannot (find key-value separator ':'|parse space in path|parse key .*|find closing (list )?fence)$"}
```

### cnf

```
a:
    ''' # trail after open
    x
    '''
```

```
a:
    '''# glued after open
    x
    '''
```

```
a:
    """ # trail after open
    x
    """
```

```
a:
    """# glued after open
    x
    """
```

```
l:
    --- # trail after open
    x
    ---
```

```
l:
    ---# glued after open
    x
    ---
```

```
a:
    '''
    x
    ''' # trail after close
```

```
a:
    '''
    x
    '''# glued after close
```

```
a:
    """
    x
    """ # trail after close
```

```
a:
    """
    x
    """# glued after close
```

```
l:
    ---
    x
    --- # trail after close
```

```
l:
    ---
    x
    ---# glued after close
```


## Case: fence-parent-comment-error

A fence must open at the very next line after its pair: a
comment between them has no home - not the fence's content,
which is the string, not the pair's line, which is above.

```
to-json: "error"
to-ast: "error"
error: "2:5: cannot comment between key and fence"
```

### cnf

```
a:
    # cmt
    '''
    x
    '''
```

```
a:
    # cmt
    """
    x
    """
```

```
l:
    # cmt
    ---
    x
    ---
```


## Case: comment-only-block-error

A comment cannot open a block on its own: a key with only a
comment under it decides nothing between a block map and a
block list, so the block never forms and the pair is left
without a value.

```
to-json: "error"
to-ast: "error"
error: "1:1: cannot omit the value for `a`"
```

### cnf

```
a:
    # comment
```


## Case: list-item-comments

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    # a.head[0]

    # a.head[2]

    # b.pre[0]
    # b.pre[1]
    - "b"
    # b.post[0]
    # b.post[1]

    # c.pre[0]

    # c.pre[2]
    # c.pre[3]
    - "c"
    # c.post[0]
    # c.post[1]

    # a.foot[0]

    # a.foot[2]
    # a.foot[3]
```

### json

```
{"a":["b","c"]}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "headcmt": {
                "cmt": ["# a.head[0]","","# a.head[2]"],
                "pos": {"l":2,"c":5}
            },
            "value": {
                "list": [{
                    "precmt": {
                        "cmt": ["# b.pre[0]","# b.pre[1]"],
                        "pos": {"l":6,"c":5}
                    },
                    "value": {"str":"b","pos":{"l":8,"c":7},"quote":2},
                    "postcmt": {
                        "cmt": ["# b.post[0]","# b.post[1]"],
                        "pos": {"l":9,"c":5}
                    }
                }, {
                    "precmt": {
                        "cmt": ["# c.pre[0]", "", "# c.pre[2]", "# c.pre[3]"],
                        "pos": {"l":12,"c":5}
                    },
                    "value": {"str":"c","pos":{"l":16,"c":7},"quote":2},
                    "postcmt": {
                        "cmt": ["# c.post[0]","# c.post[1]"],
                        "pos": {"l":17,"c":5}
                    }
                }],
                "dashed": true,
                "block": true,
                "pos": {"l":8,"c":5}
            },
            "footcmt": {
                "cmt": ["# a.foot[0]", "", "# a.foot[2]", "# a.foot[3]"],
                "pos": {"l":20,"c":5}
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: list-fence-comments

```
to-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
list:
    ---
    # a.pre[0]
    a:
        # a.head[0]

        b: 1 # b.in

        # a.foot[1]
    # a.post[0]
    ---
```

### json

```
{"list":[{"a":{"b":1}}]}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"list","pos":{"l":1,"c":1}}],
            "value": {
                "list": [{
                    "value": {
                        "map": [{
                            "key": [{"str":"a","pos":{"l":4,"c":5}}],
                            "precmt": {"cmt":["# a.pre[0]"],"pos":{"l":3,"c":5}},
                            "headcmt": {
                                "cmt": ["# a.head[0]"],
                                "pos": {"l":5,"c":9}
                            },
                            "value": {
                                "map": [{
                                    "key": [{"str":"b","pos":{"l":7,"c":9}}],
                                    "value": {"num":"1","pos":{"l":7,"c":12}},
                                    "incmt": {"cmt":[" # b.in"],"pos":{"l":7,"c":13}}
                                }],
                                "block": true,
                                "pos": {"l":7,"c":9}
                            },
                            "footcmt": {
                                "cmt": ["","# a.foot[1]"],
                                "pos": {"l":8,"c":1}
                            },
                            "postcmt": {
                                "cmt": ["# a.post[0]"],
                                "pos": {"l":10,"c":5}
                            },
                            "indent": "    "
                        }],
                        "block": true,
                        "pos": {"l":4,"c":5}
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


