# Topic

Numbers: the classic forms, base normalization, precision preservation, and the syntax errors.


## Case: spellings

All number forms: decimal, float, hex, octal, binary, exponent, negative.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
int: 123
float: 1.23
hex: 0x123
octal: 0o77
binary: 0b11
exponent: 1e-23
negative: -1.5
minus-zero: -0
```

### json

```
{"int":123,"float":1.23,"hex":291,"octal":63,"binary":3,"exponent":1e-23,"negative":-1.5,"minus-zero":-0}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"int","pos":{"l":1,"c":1}}],
            "value": {"num":"123","pos":{"l":1,"c":6}}
        }, {
            "key": [{"str":"float","pos":{"l":2,"c":1}}],
            "value": {"num":"1.23","pos":{"l":2,"c":8}}
        }, {
            "key": [{"str":"hex","pos":{"l":3,"c":1}}],
            "value": {"num":"0x123","pos":{"l":3,"c":6}}
        }, {
            "key": [{"str":"octal","pos":{"l":4,"c":1}}],
            "value": {"num":"0o77","pos":{"l":4,"c":8}}
        }, {
            "key": [{"str":"binary","pos":{"l":5,"c":1}}],
            "value": {"num":"0b11","pos":{"l":5,"c":9}}
        }, {
            "key": [{"str":"exponent","pos":{"l":6,"c":1}}],
            "value": {"num":"1e-23","pos":{"l":6,"c":11}}
        }, {
            "key": [{"str":"negative","pos":{"l":7,"c":1}}],
            "value": {"num":"-1.5","pos":{"l":7,"c":11}}
        }, {
            "key": [{"str":"minus-zero","pos":{"l":8,"c":1}}],
            "value": {"num":"-0","pos":{"l":8,"c":13}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```




## Case: base-normalization

Non-decimal bases normalize to decimal in JSON: 0x03, 0o044, 0b011.

```
to-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
hex: 0x03
octal: 0o044
binary: 0b011
```

### json

```
{"hex":3,"octal":36,"binary":3}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"hex","pos":{"l":1,"c":1}}],
            "value": {"num":"0x03","pos":{"l":1,"c":6}}
        }, {
            "key": [{"str":"octal","pos":{"l":2,"c":1}}],
            "value": {"num":"0o044","pos":{"l":2,"c":8}}
        }, {
            "key": [{"str":"binary","pos":{"l":3,"c":1}}],
            "value": {"num":"0b011","pos":{"l":3,"c":9}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```




## Case: spelling-preserved

Number forms JSON cannot hold pass through cnf verbatim: trailing zeros and big integers, in decimal and in the other bases.

```
to-json: "advised-text"
from-json: "exact-value"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a: 1.20
b: 1.2300e-5
c: 123456789012345678901234567890
d: 0x123456789abcdef0123456789abcdef
e: 0o777777777777777777777777777777
f: 0b1000000000000000000000000000000000000
```

### json

```
{"a":1.20,"b":1.2300e-5,"c":123456789012345678901234567890,"d":1512366075204170929049582354406559215,"e":1237940039285380274899124223,"f":68719476736}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"num":"1.20","pos":{"l":1,"c":4}}
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {"num":"1.2300e-5","pos":{"l":2,"c":4}}
        }, {
            "key": [{"str":"c","pos":{"l":3,"c":1}}],
            "value": {
                "num": "123456789012345678901234567890",
                "pos": {"l":3,"c":4}
            }
        }, {
            "key": [{"str":"d","pos":{"l":4,"c":1}}],
            "value": {
                "num": "0x123456789abcdef0123456789abcdef",
                "pos": {"l":4,"c":4}
            }
        }, {
            "key": [{"str":"e","pos":{"l":5,"c":1}}],
            "value": {
                "num": "0o777777777777777777777777777777",
                "pos": {"l":5,"c":4}
            }
        }, {
            "key": [{"str":"f","pos":{"l":6,"c":1}}],
            "value": {
                "num": "0b1000000000000000000000000000000000000",
                "pos": {"l":6,"c":4}
            }
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: exponent-normalized

JSON uppercase E exponent normalizes to lowercase e in cnf.

```
from-json: "advised-text"
to-ast: "advised-value"
from-ast: "advised-text"
```

### cnf

```
a: 1e5
b: 1e+5
```

### json

```
{"a":1E5,"b":1e+5}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"a","pos":{"l":1,"c":1}}],
            "value": {"num":"1e5","pos":{"l":1,"c":4}}
        }, {
            "key": [{"str":"b","pos":{"l":2,"c":1}}],
            "value": {"num":"1e+5","pos":{"l":2,"c":4}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```


## Case: leading-zero-error

A redundant leading zero is a syntax error; 0x02 is a hex literal, not one.

```
to-json: "error"
to-ast: "error"
error: "1:4: cannot parse number \"02\": redundant leading zero"
```

### cnf

```
a: 02
```


## Case: uppercase-exponent-error

The uppercase exponent marker is a syntax error; the grammar is lowercase-only.

```
to-json: "error"
to-ast: "error"
error: "1:4: cannot parse number \"1E5\": uppercase exponent"
```

### cnf

```
a: 1E5
```


## Case: bad-digits-error

Digit-prefixed garbage is not a number form: the grammar is closed, and a
bare word is never a value. Empty base digits are errors too.

```
to-json: "error"
to-ast: "error"
error: "1:4: cannot parse number \"${a}\""
vars:
    a: {regexp: ": (\\S+)"}
```

### cnf

```
a: 0A
```

```
a: 1a
```

```
a: 12z
```

```
a: 0x
```


## Case: hex-uppercase

Hex digits are case-insensitive in both places hex appears:
the numeral base and the \x and \u escapes. The render keeps
the recorded spelling verbatim, so the uppercase form round
trips.

```
to-json: "advised-text"
to-ast: "advised-value"
```

### cnf

```
hex: 0x2F
mixed: 0xAbCd
```

### json

```
{"hex":47,"mixed":43981}
```

### ast

```
{
    "value": {
        "map": [{
            "key": [{"str":"hex","pos":{"l":1,"c":1}}],
            "value": {"num":"0x2F","pos":{"l":1,"c":6}}
        }, {
            "key": [{"str":"mixed","pos":{"l":2,"c":1}}],
            "value": {"num":"0xAbCd","pos":{"l":2,"c":8}}
        }],
        "block": true,
        "pos": {"l":1,"c":1}
    }
}
```
