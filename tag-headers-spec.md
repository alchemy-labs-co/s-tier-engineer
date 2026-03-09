**Tag Header**

Structure:

```
@key:value
@key:value
@key:value

content starts here
```

Example:

```
@type:idea
@topic:ai-interface
@tone:casual
@goal:brainstorm

I want a lightweight system that replaces markdown front matter when typing on a phone.
```

AI can parse this instantly.

Rules are simple:

`@` starts metadata
`:` separates key and value
one tag per line
blank line → actual text begins

No special closing marker needed.

---

## Multi-value Fields

Use `/` because it's thumb-easy.

```
@topic:ai/ui/product
@audience:founder/dev
```

---

## Optional Priority System

Numbers first. Also easy to type.

```
@p1
@energy:high
```

---

## Quick Context Blocks

If you want sections:

```
@task:design-system
@mode:analysis

> problem
markdown formatting sucks on mobile

> goal
create thumb friendly metadata syntax
```

`>` is easy to hit and visually clean.

---

## Ultra-Fast Version (Minimal)

If you want **very fast capture**:

```
@idea ai-input-format
@context mobile typing
@goal replace yaml front matter

text starts here
```

No colons needed.

---

## Why This Works

Parsing logic for an AI is trivial:

Anything beginning with `@` before the first blank line = metadata.

Example structured result:

```
type = idea
topic = ai-interface
tone = casual
goal = brainstorm
```

Then the remaining text becomes the body.

---

## Compared to YAML

YAML

```
---
type: idea
topic: ai
---
```

Problems:

auto-rendering
needs exact syntax
annoying characters on mobile

Tag Header:

```
@type:idea
@topic:ai
```

No breakage. No formatting.

---

## Thumb Efficiency

Right-handed iPhone reach favors:

center-right keys
first symbol page

So the fastest sequence becomes:

`@` → letters → `:` → letters → return

Which is about **4 thumb motions**.