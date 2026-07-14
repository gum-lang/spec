## Objectives

`gum` aims to be a minimal configuration file format that's easy to read due to
obvious semantics. It's is designed to map unambiguously to a hash table.

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

## Preliminaries

- `gum` is case-sensitive.
- "Whitespace" herein means tab (U+0009) or space (U+0020).
- "Newline" herein means LF (U+000A) or CRLF (U+000D U+000A).
- A `gum` document must be a valid UTF-8 encoded file.

## Comment

A hash symbol marks the start of a comment until the end of the line.

```ini
# This is a comment
key = "value"  # This is an inline comment
```

## Assignment

Value assignment happens in the form of key/value pairs.

Keys are on the left of the equals sign and values are on the right. Whitespace
is ignored around key names and values. The key, equals sign, and value must be
on the same line, though some values may be continued over multiple lines.

```ini
key = "value"
```

There can only be one value per line, and values must have one of 
the following types:

- [String](#string)
- [Number](#number)
- [Boolean](#boolean)
- [Null](#null)
- [Table](#table)
- [List](#list)

## Keys

A key may either be *bare*, a *dotted path*, or a *quoted string*.

**Bare keys** may only contain ASCII letters, ASCII digits, and underscores
(`A-Za-z0-9_`). A bare key must start with an ASCII letter or underscore — a
leading digit is not permitted. Bare keys are always interpreted as strings.

```ini
key = "value"
bare_key = "bear key"
name = "John"
_surname = "Coltrane"
```

Keys may not match data types of `true`, `false`, or `null`.

**Dotted path keys** are a sequence of keys joined with a dot. This allows for
grouping related properties together without needing inline objects.

```ini
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

Whitespace around dotted path segments is ignored.

```ini
fruit.name = "banana"              # this is best practice
fruit. color = "yellow"            # the same as fruit.color
fruit . flavor = "banana-flavored" # the same as fruit.flavor
```

As long as a key holds a table, you may still write to it and to keys
within it via dotted paths.

```ini
fruit.apple.smooth = true
fruit.orange = 2
```

**Quoted strings** may also be used as keys and ignore the above rules.

```ini
"fruit.flavor" = null # "fruit.flavor" is a key and does
                      # not create a table called "fruit"
"42" = "answer to life, the universe, and everything"
```

## String

A key may either be *quoted* or *multi-line*.

**Quoted strings** are surrounded by quotation marks (`"`). Any Unicode
character may be used except those that must be escaped: quotation mark,
backslash, and control characters other than tab (U+0000 to U+0008, U+000A to
U+001F, U+007F).

```ini
str1 = "Bears. Beets. Battlestar Gallactica."
str2 = "\"\"You miss 100% of the shots you don't take\"\n\t- Wayne Gretsky \"\n\t- Michael Scott"
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
newline immediately following the opening delimiter and immediately preceding the closing delimiter will be trimmed. Escape
sequences are processed within multi-line strings just as with quoted ones.

```ini
str1 = """This is a multiline string.
It can span multiple lines."""

str2 = """This has escapes: \n newline and \t tab."""
```

The above looks like this in JSON:

```json
"str1": "This is a multiline string.\nIt can span multiple lines."
"str2": "This has escapes: \n newline and \t tab."
```

The whitespace left of the leftmost segment of a multi-line string will also be trimmed.

```ini
screenplay = """
      "Did you ever hear the Tragedy of Darth Plagueis the Wise?"
      "No."
      "I thought not. It's not a story the Jedi would tell you. It's a Sith legend."
"""
```

The above looks like this in JSON:

```json
"screenplay": "\"Did you ever hear the Tragedy of Darth Plagueis the Wise?\"\n\"No.\"\n\"I thought not. It's not a story the Jedi would tell you. It's a Sith legend.\""
```

## Number

Numbers are integers or floating-point values. Negative numbers are prefixed
with a minus sign.

```ini
int1 = 42
int2 = 0
int3 = -17
```

```ini
flt1 = 3.1415
flt2 = -0.01
flt3 = 2.
flt4 = .2
```

## Boolean

Booleans are just the tokens you're used to. Always lowercase.

```ini
bool1 = true
bool2 = false
```

## Null

The null value is represented by the lowercase token `null`.

```ini
value = null
```

The inclusion of null is to allow specifying non-values, either for the purposes of documentation or to override assignments when merging potentially conflicting tables.

## Table

Tables are collections of assignments enclosed in curly braces. Elements are separated with newlines, commas, or both. Order does not matter.

```ini
tbl1 = { x = 1, y = "two" }

tbl2 = {
  x = 1
  y = "two"
}
```

Tables may contain comments:

```ini
tbl = {
  # This is a config value
  x = 1
  # This is another value
  y = "two"
}
```

Tables may be nested arbitrarily, and nesting may use dotted path notation:

```camel
tbl1 = {
  host = "localhost"
  port = 5432
  credentials = {
    user = "admin"
    password = null
  }
}

tbl2 = {
  host = "localhost"
  port = 5432
  credentials.user = "admin"
  credentials.password = null
}
```

Empty tables are allowed:

```ini
empty = {}
```

## List

Lists are collections of values enclosed in square brackets. Elements are separated with newlines, commas, or both. Order matters in that it is preserved during parsing.

```ini
integers = [1, 2, 3]
colors = [
  "red"
  "yellow"
  "green"
]
nested = [
  [1, 2],
  [3, 4, 5]
]
mixed = [1, "two", true, null, { x = 1 }]
```

Lists may contain comments:

```ini
arr = [
  1
  # comment
  2
]
```

Empty lists are allowed:

```ini
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
