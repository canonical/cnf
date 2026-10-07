# Topic

Strings: the two families, the escape set and its range limits, quote choice, the closed-set error, and the fence forms - block and inline, raw and escaped, with the indentation rules inside.


## Case: families

The four string families, single, double, single fence, double fence.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
simple.double: "line one\nline two"
simple.single: 'line one\nstill one'
inline.double: """line one\nline two"""
inline.single: '''line one\nstill one'''
block.double:
    """
    line one
    line two
    """
block.single:
    '''
    line one\n
    line two
    '''
```

### json

```
{
    "simple":{
        "double":"line one\nline two",
        "single":"line one\\nstill one"
    },
    "inline":{
        "double":"line one\nline two",
        "single":"line one\\nstill one"
    },
    "block":{
        "double":"line one\nline two\n",
        "single":"line one\\n\nline two\n"
    }
}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"simple","pos":{"l":1,"c":1}}, {"str":"double","pos":{"l":1,"c":8}}],
            "value": {
                "str": "line one\nline two",
                "pos": {"l":1,"c":16},
                "quote": 2
            }
        }, {
            "key": [{"str":"simple","pos":{"l":2,"c":1}}, {"str":"single","pos":{"l":2,"c":8}}],
            "value": {
                "str": "line one\\nstill one",
                "pos": {"l":2,"c":16},
                "quote": 1
            }
        }, {
            "key": [{"str":"inline","pos":{"l":3,"c":1}}, {"str":"double","pos":{"l":3,"c":8}}],
            "value": {
                "str": "line one\nline two",
                "pos": {"l":3,"c":16},
                "quote": 4
            }
        }, {
            "key": [{"str":"inline","pos":{"l":4,"c":1}}, {"str":"single","pos":{"l":4,"c":8}}],
            "value": {
                "str": "line one\\nstill one",
                "pos": {"l":4,"c":16},
                "quote": 3
            }
        }, {
            "key": [{"str":"block","pos":{"l":5,"c":1}}, {"str":"double","pos":{"l":5,"c":7}}],
            "value": {
                "str": "line one\nline two\n",
                "pos": {"l":6,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }, {
            "key": [{"str":"block","pos":{"l":10,"c":1}}, {"str":"single","pos":{"l":10,"c":7}}],
            "value": {
                "str": "line one\\n\nline two\n",
                "pos": {"l":11,"c":5},
                "quote": 3,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: escape-set

The full escape set.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-value"
```

### cnf

```
a: "\\ \" \n \r \t \b \f \x41 \u0041 \u2713"
b: """\\ \" \n \r \t \b \f \x41 \u0041 \u2713"""
c:
    """
    \\ \" \n \r \t \b \f \x41 \u0041 \u2713\
    \"""\
    """
```

### json

```
{
    "a":"\\ \" \n \r \t \b \f A A ✓",
    "b":"\\ \" \n \r \t \b \f A A ✓",
    "c":"\\ \" \n \r \t \b \f A A ✓\"\"\""
}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "\\ \" \n \r \t \b \f A A ✓",
                "pos": {"l":1,"c":4},
                "quote": 2
            }
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {
                "str": "\\ \" \n \r \t \b \f A A ✓",
                "pos": {"l":2,"c":4},
                "quote": 4
            }
        }, {
            "key": [{"str":"c","pos":{"l":3,"c":1}}],
            "value": {
                "str": "\\ \" \n \r \t \b \f A A ✓\"\"\"",
                "pos": {"l":4,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: escape-limits

The escape range limits, both sides of each: U+001F/U+0020 (C0 boundary), U+00FF/U+0100 (\x ceiling), U+2027/U+2028/U+2029/U+202A (separators), U+FFFF/U+10000 (\u ceiling).

```
to-json: "exact-value"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-value"
vars:
    sa: "\u2027"
    sb: "\u2028"
    sc: "\u2029"
    sd: "\u202a"
    ua: "\uffff"
```

### cnf

```
c0a: "\x1f"
c0b: "\x20"
xa: "\xff"
xb: "\u0100"
sa: "\u2027"
sb: "\u2028"
sc: "\u2029"
sd: "\u202a"
ua: "\uffff"
```

### json

```
{
    "c0a":"\u001f",
    "c0b":" ",
    "xa":"ÿ",
    "xb":"Ā",
    "sa":"${sa}",
    "sb":"${sb}",
    "sc":"${sc}",
    "sd":"${sd}",
    "ua":"${ua}"
}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"c0a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "\u001f",
                "pos": {"l":1,"c":6},
                "quote": 2
            }
        }, {
            "key": [{"str":"c0b","pos":{"l":2,"c":1}}],
            "value": {"str":" ","pos":{"l":2,"c":6},"quote":2}
        }, {
            "key": [{"str":"xa","pos":{"l":3,"c":1}}],
            "value": {"str":"ÿ","pos":{"l":3,"c":5},"quote":2}
        }, {
            "key": [{"str":"xb","pos":{"l":4,"c":1}}],
            "value": {"str":"Ā","pos":{"l":4,"c":5},"quote":2}
        }, {
            "key": [{"str":"sa","pos":{"l":5,"c":1}}],
            "value": {
                "str": "${sa}",
                "pos": {"l":5,"c":5},
                "quote": 2
            }
        }, {
            "key": [{"str":"sb","pos":{"l":6,"c":1}}],
            "value": {
                "str": "\u2028",
                "pos": {"l":6,"c":5},
                "quote": 2
            }
        }, {
            "key": [{"str":"sc","pos":{"l":7,"c":1}}],
            "value": {
                "str": "\u2029",
                "pos": {"l":7,"c":5},
                "quote": 2
            }
        }, {
            "key": [{"str":"sd","pos":{"l":8,"c":1}}],
            "value": {
                "str": "${sd}",
                "pos": {"l":8,"c":5},
                "quote": 2
            }
        }, {
            "key": [{"str":"ua","pos":{"l":9,"c":1}}],
            "value": {
                "str": "${ua}",
                "pos": {"l":9,"c":5},
                "quote": 2
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: astral-ceiling

The \u ceiling from above: U+FFFF is the last \uNNNN-writable char but is written literally (only C0 controls and line separators are escaped); U+10000 (astral) takes the braced \u{...} escape, though the render still writes it literally.

```
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
vars:
    ua: "\uffff"
    ub: "\U00010000"
```

### cnf

```
ua: "${ua}"
ub: "${ub}"
```

### json

```
{
    "ua":"${ua}",
    "ub":"${ub}"
}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"ua","pos":{"l":1,"c":1}}],
            "value": {
                "str": "${ua}",
                "pos": {"l":1,"c":5},
                "quote": 2
            }
        }, {
            "key": [{"str":"ub","pos":{"l":2,"c":1}}],
            "value": {
                "str": "${ub}",
                "pos": {"l":2,"c":5},
                "quote": 2
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: quote-choice

Quote family choice: single quotes when the string has double quotes, double otherwise; with both, the fewer escapes wins and double takes ties.

```
to-json: "advised-text"
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a: "has 'single' quotes"
b: 'has "double" quotes'
c: "plain"
d: "both \" and '"
```

### json

```
{"a":"has 'single' quotes","b":"has \"double\" quotes","c":"plain","d":"both \" and '"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "has 'single' quotes",
                "pos": {"l":1,"c":4},
                "quote": 2
            }
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {
                "str": "has \"double\" quotes",
                "pos": {"l":2,"c":4},
                "quote": 1
            }
        }, {
            "key": [{"str":"c","pos":{"l":3,"c":1}}],
            "value": {
                "str": "plain",
                "pos": {"l":3,"c":4},
                "quote": 2
            }
        }, {
            "key": [{"str":"d","pos":{"l":4,"c":1}}],
            "value": {
                "str": "both \" and '",
                "pos": {"l":4,"c":4},
                "quote": 2
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: unclosed-quote-error

A quoted value that never closes on its line is a
syntax error, named at the quote token's start.

```
to-json: "error"
to-ast: "error"
error: {regexp: "^1:4: cannot find closing (quote|fence)$"}
```

### cnf

```
a: 'hi
```

```
a: "hi
```

```
a: """hi
```

```
a: '''hi
```


## Case: unknown-escape-error

An unknown escape is a syntax error. The error names the backslash
where the escape starts.

```
to-json: "error"
to-ast: "error"
error: {regexp: "^1:(7|9): cannot parse escape \\\\d$"}
```

### cnf

```
a: "c:\dir"
```

```
a: """c:\dir"""
```


## Case: unknown-escape-fence-block-error

A bad escape inside a block fence names its line and column:
the error points inside the content line, where the escape
starts.

```
to-json: "error"
to-ast: "error"
error: "3:36: cannot parse escape \\d"
```

### cnf

```
a:
    """
    a long content line with a bad \d escape
    """
```


## Case: fence-block

A fence block: # is content inside the fence, and the value ends with the newline before the closing fence.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
script:
    """
    #!/bin/sh
    echo "not today"
    """
```

### json

```
{"script":"#!/bin/sh\necho \"not today\"\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"script","pos":{"l":1,"c":1}}],
            "value": {
                "str": "#!/bin/sh\necho \"not today\"\n",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: empty-fence

A fence with no lines inside is the empty string.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    """
    """
```

### json

```
{"a":""}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: single-line-fence

Single-line fences are the compact form of the block form.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a: """Multiple\nlines.\n"""
b: '''Single\nline.'''
```

### json

```
{"a":"Multiple\nlines.\n","b":"Single\\nline."}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "Multiple\nlines.\n",
                "pos": {"l":1,"c":4},
                "quote": 4
            }
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {
                "str": "Single\\nline.",
                "pos": {"l":2,"c":4},
                "quote": 3
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: raw-fence

Multi-line raw-writable content uses `'''`.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
s:
    '''
    line o\ne\
    line two
    '''
```

### json

```
{"s":"line o\\ne\\\nline two\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"s","pos":{"l":1,"c":1}}],
            "value": {
                "str": "line o\\ne\\\nline two\n",
                "pos": {"l":2,"c":5},
                "quote": 3,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: inner-quote-fence

Prefer `'''` for raw-writable content: a single quote inside the fence is fine.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    '''
    it's
    me
    '''
b:
    """
    it"s
    me
    """
```

### json

```
{"a":"it's\nme\n","b":"it\"s\nme\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "it's\nme\n",
                "pos": {"l":2,"c":5},
                "quote": 3,
                "block": true
            },
            "indent": "    "
        }, {
            "key": [{"str":"b","pos":{"l":6,"c":1}}],
            "value": {
                "str": "it\"s\nme\n",
                "pos": {"l":7,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: raw-delimiter-fence

A content line that is exactly `'''` would close the raw fence early, so the escaped family is used.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
s:
    """
    before
    '''
    after
    """
```

### json

```
{"s":"before\n'''\nafter\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"s","pos":{"l":1,"c":1}}],
            "value": {
                "str": "before\n'''\nafter\n",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: trailing-whitespace-fence

Trailing whitespace after the text on a fenced line is content, not dropped: the escaped family can spell it with escapes, the raw family only literally.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    """
    text \t
    """
b:
    '''
    text 	
    '''
```

### json

```
{"a":"text \t\n","b":"text \t\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "text \t\n",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }, {
            "key": [{"str":"b","pos":{"l":5,"c":1}}],
            "value": {
                "str": "text \t\n",
                "pos": {"l":6,"c":5},
                "quote": 3,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: space-content-fence

Blank lines inside a fence keep their whitespace as content: 5 spaces in a 4-space fence is 1 space plus the newline.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a:
    """
    one
     
    """
```

### json

```
{"a":"one\n \n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "one\n \n",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: shallow-indent-fence

A blank line with a valid prefix shallower than the fence's contributes the newline only: the content never reached the fence.
The AST does not record the blank line's form, so from-ast compares the value.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-value"
```

### cnf

```
a:
    """
    one
  
    """
```

### json

```
{"a":"one\n\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "one\n\n",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: shallow-content-fence-error

A non-empty line at a prefix shallower than the fence's is a syntax error: the fence closes only at its own exact prefix.

```
to-json: "error"
to-ast: "error"
error: "4:3: cannot close fence at shallower prefix"
```

### cnf

```
a:
    """
    one
  two
    """
```


## Case: mixed-prefix-fence-error

A prefix mixing indent characters is a syntax error inside a fence, even on a blank line.

```
to-json: "error"
to-ast: "error"
error: "4:4: cannot mix indentation prefixes in fence"
```

### cnf

```
a:
    """
    one
  	
    """
```


## Case: fence-parent-blank-error

A fence must open at the very next line after its pair: a
blank between them has no home - not the fence's content,
which is the string, not the pair's line, which is above.

```
to-json: "error"
to-ast: "error"
error: "2:1: cannot blank between key and fence"
```

### cnf

```
a:

    '''
    x
    '''
```

```
a:

    """
    x
    """
```


## Case: trailing-single-fence-error

An inline fence closes at the first delimiter; a stray single quote after the close is a syntax error.

```
to-json: "error"
to-ast: "error"
error: "1:12: cannot parse trailing \"'\""
```

### cnf

```
a: '''it''''
```


## Case: trailing-double-fence-error

An inline fence closes at the first delimiter; a stray double quote after the close is a syntax error.

```
to-json: "error"
to-ast: "error"
error: "1:12: cannot parse trailing \"\\\"\""
```

### cnf

```
a: """it""""
```


## Case: trailing-escape-escaped

An inline fence with an escaped quote at the end is okay.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-value"
```

### cnf

```
a: """it\""""
```

### json

```
{"a":"it\""}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "it\"",
                "pos": {"l":1,"c":4},
                "quote": 4
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: escaped-newline-fence

A long line with a single quote forces the escaped fence with backslash continuation wrapping.

```
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-value"
```

### cnf

```
s:
    """
    it's a very long line of fence content that certainly ex\
    ceeds the sixty column inline width for sure
    """
```

### json

```
{"s":"it's a very long line of fence content that certainly exceeds the sixty column inline width for sure\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"s","pos":{"l":1,"c":1}}],
            "value": {
                "str": "it's a very long line of fence content that certainly exceeds the sixty column inline width for sure\n",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: escaped-newline-space-fence

A backslash continuation mid-fence eats the newline; the next line's excess indent is content, so the joined text carries a leading space.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-value"
```

### cnf

```
s:
    """
    Line one\nLine\
     two
    """
```

### json

```
{"s":"Line one\nLine two\n"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"s","pos":{"l":1,"c":1}}],
            "value": {
                "str": "Line one\nLine two\n",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: fence-continuation-into-close

The last fence line continues straight into the closing delimiter, so the value ends with no newline; the parse must record the exact lines it read so the render writes them back verbatim.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-value"
```

### cnf

```
s:
    """
    one \
    two\
    """
```

### json

```
{"s":"one two"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"s","pos":{"l":1,"c":1}}],
            "value": {
                "str": "one two",
                "pos": {"l":2,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: broken-inline-fence-error

A fence is a block value: it opens on its own line, indented
under its key. The inline form - the fence on the same line
as the key - is a syntax error, for both families.

```
to-json: "error"
to-ast: "error"
error: "1:4: cannot find closing fence"
```

### cnf

```
a: """
"""
```

```
a: '''
'''
```


## Case: unquoted-value-escalation

A bare word is never a value: the Unquoted family is
keys-only, and cnf text cannot hold it in value position. An
AST can - hand-built, or through a JSON decode - and the
render respells it under the value policy: double by default,
single when the content holds a double quote or a backslash
it spells raw, and double again when a single cannot hold
the content - an embedded single quote, a newline.

```
from-ast: "advised-text"
```

### cnf

```
a: "simple.word"
b: "can't"
c: 'has "dq"'
d: 'has \bs'
e: "both ' and \""
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "simple.word",
                "pos": {"l":1,"c":4},
                "quote": 2
            }
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {
                "str": "can't",
                "pos": {"l":2,"c":4},
                "quote": 2
            }
        }, {
            "key": [{"str":"c","pos":{"l":3,"c":1}}],
            "value": {
                "str": "has \"dq\"",
                "pos": {"l":3,"c":4},
                "quote": 1
            }
        }, {
            "key": [{"str":"d","pos":{"l":4,"c":1}}],
            "value": {
                "str": "has \\bs",
                "pos": {"l":4,"c":4},
                "quote": 1
            }
        }, {
            "key": [{"str":"e","pos":{"l":5,"c":1}}],
            "value": {
                "str": "both ' and \"",
                "pos": {"l":5,"c":4},
                "quote": 2
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: unquoted-multiline-fence

A fresh unquoted string in value position that holds an
inner newline opens as a fence block: one line per line of
content, instead of one escaped line hiding every break.
The trailing newlines do not count - only a break between
content lines does, so a value ending in several still
stays on the pair's line. The family follows
the fence rules: content ending without a newline needs the
escaped family with its marker, content ending in one takes
the raw family.

The parse never records an unquoted string in value
position, so this spelling serves the synthetic trees -
the JSON decode is the main producer.

```
from-json: "advised-text"
from-ast: "advised-text"
```

### cnf

```
trailing: "one\n\n"
inner:
    '''
    one
    two
    '''
cut:
    """
    once
    twice\
    """
```

### json

```
{
    "trailing": "one\n\n",
    "inner": "one\ntwo\n",
    "cut": "once\ntwice"
}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"trailing","pos":{"l":1,"c":1}}],
            "value": {
                "str": "one\n\n",
                "pos": {"l":1,"c":11},
                "quote": 2
            }
        }, {
            "key": [{"str":"inner","pos":{"l":2,"c":1}}],
            "value": {
                "str": "one\ntwo\n",
                "pos": {"l":3,"c":5},
                "quote": 3,
                "block": true
            },
            "indent": "    "
        }, {
            "key": [{"str":"cut","pos":{"l":7,"c":1}}],
            "value": {
                "str": "once\ntwice",
                "pos": {"l":8,"c":5},
                "quote": 4,
                "block": true
            },
            "indent": "    "
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: braced-escape

The braced \u{...} escape: one to six hex digits, any code
point up to U+10FFFF - the range beyond the four-digit
\uXXXX, the astral planes included. The value decodes to the
rune; the render spells the rune literally, the spelling the
astral-ceiling case pins.

```
to-json: "advised-text"
to-ast: "advised-text"
from-ast: "advised-value"
vars:
    ub: "\U00010000"
```

### cnf

```
a: "\u{41}"
b: "\u{10000}"
```

### json

```
{"a":"A","b":"${ub}"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"str":"A","pos":{"l":1,"c":4},"quote":2}
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {
                "str": "${ub}",
                "pos": {"l":2,"c":4},
                "quote": 2
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: braced-escape-error

A braced escape that holds no digit, more than six digits, a
non-hex digit, a value over U+10FFFF, a surrogate half, or no
closing brace is a syntax error, named at the backslash. The
bare \uNNNN form takes the same ceiling: a surrogate half is
not a character and has no UTF-8 spelling, so it refuses
too. A short bare form, or a bare \u that ends the line
outright, refuses the same way - the escape needs its four
digits on the line to be anything at all.

```
to-json: "error"
to-ast: "error"
error: "1:5: cannot parse \\u escape"
```

### cnf

```
a: "\u{}"
```

```
a: "\u{1234567}"
```

```
a: "\u{4g}"
```

```
a: "\u{110000}"
```

```
a: "\u{d800}"
```

```
a: "\u{41"
```

```
a: "\uD800"
```

```
a: "\u00"
```

```
a: "\u
```


## Case: unicode-fence-escape

A \u escape inside the inline fence form, including one at the
interior's very end - the last thing the close search scans
before the delimiter, where the whole escape must fit in the
scanned text.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-value"
vars:
    tick: "\u2713"
```

### cnf

```
a: """line \u{2713}"""
b: """\u2713 starts"""
```

### json

```
{"a":"line ${tick}","b":"${tick} starts"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "line ${tick}",
                "pos": {"l":1,"c":4},
                "quote": 4
            }
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {
                "str": "${tick} starts",
                "pos": {"l":2,"c":4},
                "quote": 4
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: escape-hex-uppercase

The hex digits of the \x and \u escapes take both cases, the
same rule as the numerals.

```
to-ast: "advised-text"
to-json: "advised-text"
vars:
    dcaron: "\u010f"
```

### cnf

```
a: "\x4A \u004A"
b: "\u{10F}"
```

### json

```
{"a":"J J","b":"${dcaron}"}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {
                "str": "J J",
                "pos": {"l":1,"c":4},
                "quote": 2
            }
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {"str":"${dcaron}","pos":{"l":2,"c":4},"quote":2}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```
