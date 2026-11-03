# The expression language

Expressions are stored as written and resolved when the document is read. They form a
one-way graph: a value may read other values, and nothing may read back. A cycle is
an error, not solved.

## What an expression can contain

|                     |                                                                |
| ------------------- | -------------------------------------------------------------- |
| `12`, `-4`, `1.5`   | a number                                                       |
| `50%`               | a percentage of the containing box — see below                 |
| `$gap`              | a design token                                                 |
| `@hero.title.right` | another node's edge, by id                                     |
| `prev.bottom`       | a relative reference: `parent`, `prev`, `next`, `self`, `root` |
| `+ - * /`           | arithmetic, with the usual precedence                          |
| `( )`               | grouping                                                       |
| unary `-`           | `(-$gap)`                                                      |

There are **no functions** — no `min`, `max`, `clamp`, `abs`. If you need one, the value is
a literal or the layout needs rethinking.

There are **no units** either: a value is a number, a percentage, or an expression —
`h=480`, not `h=480px`; `w=(100%)`, not `w=100vw`. Units appear only inside a quoted
gradient, which is CSS spelling.

## Edges

Any reference names an edge of the box it points at:

| edge        | is                      |
| ----------- | ----------------------- |
| `x`, `left` | the left edge           |
| `y`, `top`  | the top edge            |
| `w`, `h`    | width, height           |
| `right`     | `x + w`                 |
| `bottom`    | `y + h`                 |
| `cx`, `cy`  | the centre on each axis |

```text
x=(@logo.right + $gap)          beside the logo
y=(@hero.title.bottom + 12)     under the headline
x=(parent.cx - self.w / 2)      centred horizontally
```

## Percentages

A bare percentage measures against the **containing box** — the nearest enclosing frame, or
the canvas when it has a size — on the axis the property lives on:

- `x` and `w` measure against the parent's width
- `y` and `h` measure against the parent's height
- anywhere else — `opacity=(50%)` — it is simply `0.5`

```text
w=(100% - $gap * 2)             full width inside a gutter
w=((100% - $gap * 4) / 3)       one of three columns with gaps between
```

Inside a group, a percentage measures the frame around the group. One on a frame, or on
anything directly on an unsized canvas, has no box and is `percent-without-parent`.

## Relative references

`parent`, `prev`, `next`, `self` and `root` need no `@` and take an edge the same way.

```text
y=(prev.y)                      line up with the previous sibling
y=(prev.bottom + $gap)          stack under it
w=(parent.w / 2)                half the container
```

`prev` and `next` are siblings **in layout order**, which sees through groups (a group
carries no geometry). A frame does carry geometry, so `parent` inside one is the frame.

Every reference is read in the coordinates of the node that wrote it. Inside a frame,
`parent.left` and `parent.top` are `0`, `parent.cx` is half the frame's width, and
`parent.right` is its width. A reference to a node
in another frame means the same point on the page, carried into this frame's coordinates.

## Which properties accept an expression

Every number a node takes, where its type takes it: `x`, `y`, `w`, `h` (a connector's `x1`,
`y1`, `x2`, `y2`), `rotation`, `strokeWidth`, `radius` and each corner, `opacity`, `blur`,
`backdrop`, and a text's `size`, `weight`, `lineHeight` and `letterSpacing`. A video's
`start`, `end` and `poster` are literals.

`fill`, `stroke`, `background` and `font` are **names, not numbers**. They take a token name
or a literal value; an expression there is reported as `expression-not-supported`.

Marks inside a text run — `weight=700` on one span — are always literals: a run has no
address for an expression to resolve against.

## Writing values

| you mean                          | you write             |
| --------------------------------- | --------------------- |
| the number 24                     | `x=24`                |
| the token `gap`                   | `x=$gap`              |
| an expression                     | `x=(@a.right + $gap)` |
| the string `accent`               | `fill=accent`         |
| a string with a space             | `name="Hero heading"` |
| a string starting with `$` or `(` | `fill="$literal"`     |
| true                              | `italic`              |
| false                             | omit it               |

Quoting follows JSON: `\"` and `\n` work.

## Tokens

```vd
$gap = 24
$accent = #5b5bd6
$display = 34
$label = "Section heading"
```

A bare numeric token is a number; quote it to keep it a string. A token may not hold an
expression — it is a value, not a formula.

## Paint: colours, gradients, layers

`fill`, `stroke`, a frame's `background` and a run's `fill` take a **paint**: a colour, a
token naming one, or a gradient in CSS spelling. Quote a gradient — it has spaces.

```vd
$accent = #2563EB
$glow = "linear-gradient(135deg, accent, #7C3AED 60%, transparent)"

rect hero.bg x=0 y=0 w=(100%) h=480 fill=glow
ellipse hero.orb x=600 y=40 w=240 h=240 fill="radial-gradient(circle at 30% 30%, #fff, accent)"
text hero.title x=64 y=120 w=560 h=64 size=48 weight=700 fill="linear-gradient(90deg, #F43F5E, #F59E0B)"
  "Ship faster"
```

- `linear-gradient(<angle>, <stops>)` — the angle in `deg` (0 points up, 90 right) or
  `to right`, `to bottom left`…; omitted, it runs top to bottom.
- `radial-gradient([circle|ellipse] [at X% Y%], <stops>)` — centred where asked, reaching
  the farthest corner.
- A stop is a colour or a token name, with an optional `%`. `transparent` fades the
  neighbouring colour out, as in CSS.
- A gradient spans the node's box; on text it sweeps across the whole box, not each run.
- Not supported: `conic-gradient`, `repeating-*`, sizes such as `closest-side`, and a token
  holding a gradient used as a stop of another gradient.
- **Layers.** A shape's `fill` and a frame's or canvas's `background` take a list, first on top, as a
  CSS background: `fill="linear-gradient(#0000, #000C), image(hero), #fff"`. A token may
  hold the whole list. `stroke` and a run's `fill` take one paint.
- `image(<asset>, cover|contain|fill)` is a picture layer, fitted as `image`'s `fit` is
  (default `cover`), cut by the corners and outline. One naming no asset is the warning
  `fill-asset-missing`; the other layers still draw.

## Fonts

```vd
font display = Inter
  fallback "system-ui, -apple-system, sans-serif"
  face fonts/Inter-Regular.woff2 weight=400
  face fonts/Inter-Bold.woff2 weight=700 style=italic
```

`font=display` on a node or a run names the entry. A name with no entry passes straight
through as a CSS font stack, so `font="Georgia, serif"` works with no registry at all.

`fallback` is what draws while a face loads, or if it never does.

Each `face` is one font file, with the `weight` (a number) and `style` (`normal` or
`italic`) it draws.

A face can instead be an asset, its file in the document like a picture's:
`face asset=fraunces-600 weight=600`, with `asset fraunces-600 = font/woff2 …` from
`vd_add_asset`. Only a font whose licence lets it travel can be added; the asset line's
`license=` says which (`OFL`, `Apache-2.0`, `UFL`, `MIT`). Any other font is named by family,
with a `fallback`, and no face.

## Assets

```vd
asset logo = image/png hash=sha256-9f2c4e8a… bytes=24567
asset hero = image/jpeg hash=sha256-1a77b0c3… bytes=412330 credit="Jane Doe / Unsplash" source="https://unsplash.com/photos/abc" license=Unsplash
```

One asset, one line, with its provenance. `hash` and `bytes` are required: they
name the picture's bytes further down the file.
`credit`, `source` and `license` are optional in the grammar but **not optional
for an image you found** — the bytes end up inside a file someone re-shares.

A `.vd` file is a **text head and a payload tail**, separated by a line reading
`--- payload base64 ---`. Everything above it is the design, and is all you ever
read or write. Everything below it is one line per asset: its hash, a space, and
its bytes as base64. You never write that half, and tools do not show it to you.
The registry's `bytes` and media type tell a reader what a document needs, and
how large, before any bytes arrive.
