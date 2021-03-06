---
navhome: /docs/
next: true
sort: 13
title: =, "tiscom"
---

# `=, "tiscom"` 

`[%tscm p=hoon q=hoon]`: expose namespace

`p` evaluates to a noun with some namespace.  From within `q` you may access those 
names without a wing path (e.g., as in for some face `b` rather than `b.p`), or without 
invoking a core name when `p` is a core.  This is especially useful for calling arms 
from an imported library or for calling arms from an stdlib core repeatedly.

## Syntax

Regular: *2-fixed*.

## Examples

```
> (sum -7 --7)
-find.sum
[crash message]

> (sum:si -7 --7)
--0

> =,  si  (sum -7 --7)
--0
```
