# CAMEL v0.1.0

camel is markup everyone likes.

## Objectives

CAMEL aims to be a minimal configuration file format that's easy to read due to
obvious semantics. CAMEL is designed to map unambiguously to a hash table. CAMEL
should be easy to parse into data structures in a wide variety of languages.

## Table of contents

- [Preliminaries](#preliminaries)
- [Comment](#comment)
- [Key/Value Pair](#keyvalue-pair)
- [Keys](#keys)
- [String](#string)
- [Number](#number)
- [Boolean](#boolean)
- [Null](#null)
- [Object](#object)
- [Array](#array)
- [Full Example](#full-example)
- [ABNF Grammar](#abnf-grammar)

## Preliminaries

- CAMEL is case-sensitive.
- "Whitespace" herein means tab (U+0009) or space (U+0020).
- "Newline" herein means LF (U+000A) or CRLF (U+000D U+000A).
- A CAMEL document must be a valid UTF-8 encoded Unicode document.

## Comment

A hash symbol marks the rest of the line as a comment, except when inside a
string.

```camel
# This is a comment
key = "value"  # This is an inline comment
another = "# This is a string"
```

## Key/Value Pair

The primary building block of a CAMEL document is the key/value pair.

Keys are on the left of the equals sign and values are on the right. Whitespace
is ignored around key names and values. The key, equals sign, and value must be
on the same line, though some values can be continued over multiple lines.

```camel
key = "value"
```

Values must have one of the following types.

- [String](#string)
- [Number](#number)
- [Boolean](#boolean)
- [Null](#null)
- [Object](#object)
- [Array](#array)

Unspecified values are invalid.

```camel
key =  # INVALID
```

There must be a newline (or EOF) after a key/value pair.

```
first = "Tom" last = "Preston-Werner"  # INVALID
```

## Keys

A key may be either bare or part of a dotted path. Quoted keys are not
supported.
A key may either be a bare piece of text, a dotted path, or a quoted string.

**Bare keys** may only contain ASCII letters, ASCII digits, and underscores
(`A-Za-z0-9_`). A bare key must start with an ASCII letter or underscore — a
leading digit is not permitted. Bare keys are always interpreted as strings.

```camel
key = "value"
bare_key = "bear key"
name = "John"
_surname = "Coltrane"
```

Note that bare keys may match the literal tokens `true`, `false`, or `null`;
these are still valid as keys when they appear on the left-hand side of an
equals sign, but doing so is discouraged.

```camel
true = "value"
null = null
```

**Dotted path keys** are a sequence of keys joined with a dot. This allows for
grouping related properties together without needing inline objects.

```camel
name = "Banana"
physical.color = "yellow"
physical.shape = "banana-shaped"
```

In JSON, that would give the following structure:

```json
{
  "name": "Banana",
  "physical": {
    "color": "yellow",
    "shape": "banana-shaped"
  }
}
```

Whitespace around dotted path segments is ignored, but avoiding extraneous whitespace is encouraged.

```camel
fruit.name = "banana"              # this is best practice
fruit. color = "yellow"            # same as fruit.color
fruit . flavor = "banana-flavored" # same as fruit.flavor
```

As long as a key holds a table, you may still write to it and to names
within it via dotted paths.

```camel
# This creates "fruit" as a dict and "fruit.apple" within it.
fruit.apple.smooth = true

# So then you can add keys to the "fruit" dict like so:
fruit.orange = 2
```

Once a key has been assigned a non-container value, it cannot be used
as part of a dotted path.

```
# INVALID
fruit.apple = 1
fruit.apple.smooth = true  # INVALID
```

## String

There are two ways to express strings: quoted and multi-line.

**Quoted strings** are surrounded by quotation marks (`"`). Any Unicode
character may be used except those that must be escaped: quotation mark,
backslash, and control characters other than tab (U+0000 to U+0008, U+000A to
U+001F, U+007F).

```camel
str = "I'm a string. \"You can quote me\". Name\tJos\u00E9\nLocation\tSF."
```

The following escape sequences are recognized:

```
\b         - backspace       (U+0008)
\t         - tab             (U+0009)
\n         - linefeed        (U+000A)
\f         - form feed       (U+000C)
\r         - carriage return (U+000D)
\"         - quote           (U+0022)
\\         - backslash       (U+005C)
\/         - forward slash   (U+002F)
\uXXXX     - unicode         (U+XXXX)
```

**Multi-line strings** are surrounded by three quotation marks (`"""`). A
newline immediately following the opening delimiter will be trimmed. Escape
sequences are processed within multi-line strings (same set as quoted strings).

```camel
str1 = """This is a multiline string.
It can span multiple lines."""

str2 = """This has escapes: \n newline and \t tab."""
```

The above produce the equivalent of:

```json
"str1": "This is a multiline string.\nIt can span multiple lines."
"str2": "This has escapes: \n newline and \t tab."
```

```camel
str = """She said "hello"."""    # Produces: She said "hello".
```

```
str = """triple quotes """ inside"""  # INVALID (""" terminates)
```

## Number

Numbers are integers or floating-point values. Negative numbers are prefixed
with a minus sign.

```camel
int1 = 42
int2 = 0
int3 = -17
```

Leading zeros are not allowed (except for the number `0` itself).

A **float** consists of an integer part followed by a fractional part.

```camel
# fractional
flt1 = 3.1415
flt2 = -0.01
```

The decimal point, if used, must be surrounded by at least one digit on each
side.

```
# INVALID FLOATS
invalid_float_1 = .7
invalid_float_2 = 7.
```

## Boolean

Booleans are just the tokens you're used to. Always lowercase.

```camel
bool1 = true
bool2 = false
```

## Null

The null value is represented by the lowercase token `null`.

```camel
value = null
```

The inclusion of null is to allow specifying non-values, either for the purposes of documentation or to override assignments when merging potentially conflicting table.

## Object

Objects (also called hash tables or dictionaries) are collections of key/value
pairs enclosed in curly braces. Objects come in two forms: inline and
multi-line.

**Inline objects** use commas to separate members:

```camel
obj = { x = 1, y = "two" }
```

**Multi-line objects** use newlines to separate members:

```camel
obj = {
  x = 1
  y = "two"
}
```

Objects may contain comments:

```camel
obj = {
  # This is a config value
  x = 1
  # This is another value
  y = "two"
}
```

Objects may be nested arbitrarily:

```camel
database = {
  host = "localhost"
  port = 5432
  credentials = {
    user = "admin"
    password = null
  }
}
```

Empty objects are allowed:

```camel
empty = {}
```

Keys and dotted paths within objects behave the same as at the top level.

```
# INVALID
obj = { x = 1, x = 2 }  # duplicate keys: last wins, but discouraged
```

## Array

Arrays are ordered values surrounded by square brackets. Arrays come in two
forms: inline and multi-line. Arrays can contain values of any type, and
different types may be mixed.

**Inline arrays** use commas to separate elements:

```camel
integers = [1, 2, 3]
colors = ["red", "yellow", "green"]
nested = [[1, 2], [3, 4, 5]]
mixed = [1, "two", true, null, { x = 1 }]
```

**Multi-line arrays** use newlines to separate elements. Each element is
followed by a newline (and optionally a comment):

```camel
integers2 = [
  1
  2
  3
]
```

Comments may appear inside arrays:

```camel
arr = [
  1
  # comment
  2
]
```

Empty arrays are allowed:

```camel
empty = []
```

## Full Example

The following is a complete CAMEL document demonstrating all features:

```camel
# camel config example
name = "camel-parser"
version = 1

author.name = "Alice"
author.email = "alice@example.com"

features = {
  strict = true
  logging = false
  max_retries = 3
  tags = ["dev", "prod"]
}

database = {
  host = "localhost"
  port = 5432
  credentials = {
    user = "admin"
    password = null
  }
}

description = """This is camel.
It is markup everyone likes."""
```

Parsing this document produces the following JSON structure:

```json
{
  "name": "camel-parser",
  "version": 1,
  "author": {
    "name": "Alice",
    "email": "alice@example.com"
  },
  "features": {
    "strict": true,
    "logging": false,
    "max_retries": 3,
    "tags": ["dev", "prod"]
  },
  "database": {
    "host": "localhost",
    "port": 5432,
    "credentials": {
      "user": "admin",
      "password": null
    }
  },
  "description": "This is camel.\nIt is markup everyone likes."
}
```

## ABNF Grammar

A formal description of CAMEL's syntax is available in the
[ABNF file](./camel.abnf).

```abnf
camel        = *(expression / comment-line)

expression   = *ws assignment *ws [comment] nl
comment-line = *ws comment nl

assignment   = path *ws "=" *ws value
path         = key *( *ws "." *ws key )

key          = (ALPHA / underscore) *( ALPHA / DIGIT / underscore )
value        = false / true / null / object / array / number / string
underscore   = %x5f ; _

false        = %x66.61.6c.73.65 ; false
true         = %x74.72.75.65    ; true
null         = %x6e.75.6c.6c    ; null

object           = inline-object / multiline-object
inline-object    = "{" *ws [ assignment *( *ws "," *ws assignment ) ] *ws "}"
multiline-object = "{" *ws nl *(ws / comment-line / nl) *multiline-obj-el "}"
multiline-obj-el = assignment multiline-sep *(ws / nl)

array            = inline-array / multiline-array
inline-array     = "[" *ws [ value *( *ws "," *ws value ) ] *ws "]"
multiline-array  = "[" *ws nl *(ws / comment-line / nl) *multiline-arr-el "]"
multiline-arr-el = value multiline-sep *(ws / nl)

string           = multiline-string / quoted-string

multiline-string = triple-quote [ nl ] ml-body triple-quote
triple-quote     = %x22.22.22 ; """
ml-body          = *( unescaped-ml / escape-seq / nl )
unescaped-ml     = %x20-5B / %x5D-10FFFF ; excludes \

quoted-string    = quotation-mark *char quotation-mark
quotation-mark   = %x22 ; "
char             = unescaped / escape-seq
unescaped        = %x20-21 / %x23-5B / %x5D-10FFFF
escape-seq       = escape ( %x22 / %x5C / %x2F / %x62 / %x66 / %x6E / %x72 / %x74 / %x75 4HEXDIG )
escape           = %x5C ; \

number           = [ minus ] dec-num
minus            = %x2D ; -
dec-num          = int [ frac ]
int              = zero / ( digit1-9 *( DIGIT / %x5F DIGIT ) )
frac             = decimal-point 1*( DIGIT / %x5F DIGIT )
zero             = %x30 ; 0
digit1-9         = %x31-39 ; [1-9]
decimal-point    = %x2E ; .

comment            = "#" *non-ascii-or-space
non-ascii-or-space = %x20-10FFFF
ws                 = *( %x20 / %x09 ) ; Space / Tab
nl                 = [ %x0D ] %x0A ; LF or CRLF
multiline-sep      = *ws [ comment ] nl

ALPHA            = %x41-5A / %x61-7A
DIGIT            = %x30-39
HEXDIG           = DIGIT / "A" / "B" / "C" / "D" / "E" / "F" / "a" / "b" / "c" / "d" / "e" / "f"
```
