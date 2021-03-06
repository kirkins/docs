---
navhome: /docs/
sort: 3
title: Knit
---

# Knit

`[%knit p=(list (each @t hoon))]`: text string with interpolation.

### Produces

A list-shaped string (`tape`) of the items in `p`, which are
either constant `@t` multibyte UTF-8 strings (`cord`) or hoons
producing a `tape`.

### Syntax

Irregular: `"foobar"`.

Irregular: `"foo{(weld "moo" "baz")}bar"`.

### Examples

String:

```
~zod:dojo> "hello, world."
"hello, world."
```

String with interpolation:

```
~zod:dojo> =+(planet="world" "hello, {planet}.")
"hello, world."
```

String with interpolated prettyprinting:

```
~zod:dojo> =+(planet=%world "hello, {<planet>}.")
"hello, %world."
```
