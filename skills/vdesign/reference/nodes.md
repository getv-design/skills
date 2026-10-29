# Node types

Nine. Every line is `<type> <id> key=value …`, and children are indented two spaces under
their container.

Every node also accepts `name="…"`, a human label with no effect on layout.

Every node but the canvas also accepts two bare flags, written last on the line:

- `hidden` — not drawn anywhere, export included. It still resolves, so `@badge.right`
  keeps working while `badge` is hidden. Hiding a container hides what is in it.
- `locked` — the person's pointer passes over it on the canvas. It changes nothing for you:
  you can still edit it, and you may add or remove the flag when asked.

## `frame` — a page

The node you almost always start with. A frame is a box with **its own coordinate space**:
everything inside it is positioned from its top-left corner, and `%` inside it measures the
frame rather than the document.

A document may hold as many as you like, side by side — that is how it holds several pages,
slides or artboards. Frames also nest, which is how you get a card with a boundary of its
own.

| property     | type                 |                                      |
| ------------ | -------------------- | ------------------------------------ |
| `x`, `y`     | number or expression | where the page sits; both default 0  |
| `w`, `h`     | number or expression | the page size                        |
| `background` | token or colour      | drawn behind everything on this page |
| `clip`       | flag                 | hide what overflows; off by default  |
| `skip`       | flag                 | left out of Play; top-level only     |
| `component`  | flag                 | other pages place copies of it       |

```vd
$gap = 24

frame page w=1280 h=720 background=paper
  rect hero.bg x=$gap y=$gap w=(100% - $gap * 2) h=320 fill=accent

frame notes x=1380 w=1280 h=720 background=paper
  text agenda x=$gap y=$gap w=600 h=48
    "What we agreed"
```

Lay several pages out **left to right with a gap between them**, as above: they must not
overlap, because nothing stops one page being drawn on top of another.

## `canvas` — the surface

The surface the frames sit on, and the outermost node. **You almost never write it.** A
document that begins with frames gets one automatically, which is why the example above has
no `canvas` line and no extra indent level.

Write it only for a single-page document with shapes directly on it, where it is the page:

```vd
$gap = 24

canvas page w=1280 h=720 background=paper
  rect hero.bg x=$gap y=24 w=(100% - $gap * 2) h=260 fill=accent
```

| property     | type                 |                                    |
| ------------ | -------------------- | ---------------------------------- |
| `w`, `h`     | number or expression | optional; absent means unbounded   |
| `background` | token or colour      | only meaningful when it has a size |

A canvas must be the **only** top-level node. Writing one and then a frame beside it is an
error; put the frames inside it, or leave it out entirely.

## `rect` — a box

| property      | type                 | default        |
| ------------- | -------------------- | -------------- |
| `x`, `y`      | number or expression | required       |
| `w`, `h`      | number or expression | required       |
| `fill`        | token or colour      | `#CBD5E1`      |
| `stroke`      | token or colour      | none           |
| `strokeWidth` | number or expression | 1 when stroked |
| `radius`      | number or expression | 0              |
| `opacity`     | number or expression | 1              |

```vd
rect hero.bg x=$gap y=24 w=(100% - $gap * 2) h=260 fill=accent radius=16
```

## `ellipse` — an oval, or a circle

The ellipse inscribed in its box. A circle is `w` equal to `h`.

| property      | type                 | default   |
| ------------- | -------------------- | --------- |
| `x`, `y`      | number or expression | required  |
| `w`, `h`      | number or expression | required  |
| `fill`        | token or colour      | `#CBD5E1` |
| `stroke`      | token or colour      | none      |
| `strokeWidth` | number or expression | 0         |
| `opacity`     | number or expression | 1         |

```vd
ellipse avatar.ring x=24 y=24 w=48 h=48 fill=none stroke=accent strokeWidth=2
```

## `vector` — any shape: icons, arrows, dividers, logos

An SVG path, drawn into a box.

| property      | type                               | default   |
| ------------- | ---------------------------------- | --------- |
| `x`, `y`      | number or expression               | required  |
| `w`, `h`      | number or expression               | required  |
| `d`           | SVG path data, quoted              | required  |
| `view`        | `"minX minY width height"`, quoted | `0 0 w h` |
| `fill`        | token, colour or `none`            | `#CBD5E1` |
| `stroke`      | token or colour                    | none      |
| `strokeWidth` | number or expression               | 0         |
| `opacity`     | number or expression               | 1         |
| `rule`        | `nonzero`, `evenodd`               | `nonzero` |
| `cap`         | `butt`, `round`, `square`          | `butt`    |
| `join`        | `miter`, `round`, `bevel`          | `miter`   |

```vd
vector nav.arrow x=320 y=20 w=24 h=24 view="0 0 24 24" d="M5 12h14M12 5l7 7-7 7" fill=none stroke=ink strokeWidth=2 cap=round join=round
```

`d` is exactly SVG's path syntax — `M L H V C S Q T A Z`, absolute or relative. `view`
is the path's own coordinate space, spelled the way SVG spells a `viewBox`, and it is
**stretched onto `x y w h`**: an icon drawn on a 24-unit grid keeps its numbers and is
sized by `w` and `h` alone.

- **Over MCP, place an icon with `vd_add_icon`** rather than writing one: it knows the
  Lucide set and writes this node for you. Otherwise, **copy the icon's `d` and its
  `viewBox` verbatim** into `d` and `view`.
- The stroke is in page units and does not grow with `view`: an icon drawn with
  `strokeWidth=2` at 24 wants 4 at 48. A dot drawn as a tiny circle needs a stroke at
  least its diameter to read as solid.
  Do not rescale the numbers; change `w` and `h`.
- A line icon wants `fill=none` and a `stroke`. Without `fill=none` the shape is filled
  grey by default.
- Without `view`, the path draws at 1:1 inside its box: `vector rule x=0 y=100 w=400 h=0
d="M0 0 H400" stroke=ink strokeWidth=1` is a divider.
- A path that cannot be read draws up to its first error and reports `bad-path`.

To turn a `rect` or `ellipse` into a free-form shape, replace its line with a `vector` of
the same id — same `x y w h`, `view="0 0 <w> <h>"`, and a `d` that traces it. The person
can do the same with **Convert to vector** (⇧⌘O) and then drag its points.

## `image` — a picture

| property  | type                       | default  |
| --------- | -------------------------- | -------- |
| `x`, `y`  | number or expression       | required |
| `w`, `h`  | number or expression       | required |
| `src`     | a name from `asset`        | required |
| `fit`     | `cover`, `contain`, `fill` | `fill`   |
| `radius`  | number or expression       | 0        |
| `opacity` | number or expression       | 1        |

```vd
asset hero = image/jpeg hash=sha256-1a77b0c3… bytes=412330

image cover.photo x=0 y=0 w=(100%) h=480 src=hero fit=cover
```

`src` names an entry in the document's asset registry — **never a path or a
URL**. `cover` fills the box and crops; `contain` fits the whole picture inside
it; `fill` stretches.

**You cannot invent an asset.** A registry line needs the content hash and byte
count of bytes that actually exist in the file, so `asset` lines are written by
whatever put the bytes there — a paste, an import, or an `add-image` tool. What
you can do is refer to one that is already declared, move it, resize it, or
change its `fit`. Referring to a name with no `asset` line reports
`missing-asset` and, in the editor, draws a **slot** — a box labelled with the
node's `name` that the person drops a picture into. That is how you leave room for
a picture you cannot send; see "Images" in the overview.

## `text` — a box of text

| property           | type                              | default                                    |
| ------------------ | --------------------------------- | ------------------------------------------ |
| `x`, `y`, `w`, `h` | number or expression              | required                                   |
| `font`             | registry name or CSS stack        | `system-ui, …`                             |
| `size`             | number or expression              | 16                                         |
| `weight`           | number or expression              | 400                                        |
| `lineHeight`       | number or expression              | 1.2 — a **multiple of size**, not a length |
| `letterSpacing`    | number or expression              | 0                                          |
| `align`            | `left` `center` `right` `justify` | `left`                                     |
| `valign`           | `top` `middle` `bottom`           | `top`                                      |
| `italic`           | bare flag                         | off                                        |
| `fill`             | token or colour                   | `#0F172A`                                  |
| `opacity`          | number or expression              | 1                                          |

**`h` is authored.** Nothing measures the glyphs for you, and text that does not fit is
drawn overflowing and reported as `text-overflow`. Estimate:

```text
characters per line ≈ w / (size * 0.5)
lines               = ceil(length / characters per line)
h                   = ceil(lines * size * lineHeight)
```

### Content

One quoted line per run, indented under the node:

```vd
text hero.title x=64 y=64 w=560 h=44 size=34 weight=700
  "An open substrate for the "
  "agent era" fill=highlight
```

A single unformatted run is just one quoted line. Split into more runs only where the
formatting actually changes — and mind the spaces, which belong to whichever run they
follow.

A run may carry `font`, `size`, `weight`, `italic`, `underline`, `strike`, `fill`,
`letterSpacing`, `link`. All literals: a run is not addressable, so nothing there can be an
expression.

`link` makes the run a link: `link="https://…"`, `link="mailto:…"`, or `link="#pricing"` to
jump to the frame `pricing` in the same document. Nothing else is followed — `http:`,
`javascript:` and file paths are refused and drawn as plain text. Only link to a URL the
person gave you; never invent one. A link is not underlined for you, so add `underline`
when it should look like one:

```vd
text cta.body x=64 y=480 w=400 h=24
  "Read the "
  "docs" link="https://example.com/docs" underline
  " or see "
  "pricing" link="#pricing" underline
```

## `instance` — a copy of a component

Something drawn once and used on many pages — a sidebar, a nav bar, a card — is a
**component**: a top-level frame with the bare `component` flag. Put components in a row
**below the pages**, never on one; Play and every export of the pages leave them out.

An **instance** places it. Its contents are the component's and are never written under it;
what it changes is said with `override` lines, each naming a part by its id **in the
component**:

```vd
frame sidebar x=0 y=1000 w=240 h=900 background=#f4f4f5 component
  rect nav.indicator x=8 y=@item.home.y w=(100% - 16) h=40 fill=accent radius=8
  text item.home x=24 y=80 w=(100% - 48) h=40 valign=middle
    "Home"
  text item.billing x=24 y=(prev.bottom + 4) w=@item.home.w h=40 valign=middle
    "Billing"

frame dashboard x=0 y=0 w=1440 h=900
  instance dash.nav component=sidebar x=0 y=0 h=(100%)

frame billing x=1540 y=0 w=1440 h=900
  instance bill.nav component=sidebar x=0 y=0 h=(100%)
    override nav.indicator y=@item.billing.y
    override item.billing weight=600
```

| property    | type                 |                                                 |
| ----------- | -------------------- | ----------------------------------------------- |
| `component` | id                   | the component frame it copies                   |
| `x`, `y`    | number or expression | where it sits in its page                       |
| `w`, `h`    | number or expression | optional; absent means the component's own size |

- **An instance is a frame** for layout: `100%` and `parent` inside it mean its box.
- **A part's id on the page** is the instance's id, a dot, and its id in the component:
  `bill.nav.item.billing`. Use it to reach a part from the page —
  `x=(@bill.nav.item.home.right + $gap)` — and to `set` it in a patch.
- **Inside a component, write references unprefixed** (`@item.home.y`). Each instance
  resolves them against its own copy, so the same line works on every page.
- **An override takes any property** the part has, and quoted lines under it replace a text
  part's words. `hidden=false` undoes a `hidden` the component sets. A part of a nested
  component is named through the nested one: `override logo.mark fill=accent`.
- **Change the component to change every page**; an override wins over it only for the
  property it names. Never copy a component's parts onto a page to vary them — override, or
  make a second component.

## `group` — structure, and nothing else

```vd
group cards
  rect card.1 x=$gap y=0 w=200 h=120
  rect card.2 x=(@card.1.right + $gap) y=(prev.y) w=@card.1.w h=120
```

A group has **no `x`, `y`, `w` or `h`**, and it is not a mistake that it does not. A frame
derived from its children would close a loop — a child's `w=(100%)` would read the group's
width, which would read the child's — so a group is transparent to layout. Its children keep
the coordinate space of whatever contains the group, and `parent`, `prev`, `next` and `%`
see straight through it.

Group things because they belong together, and because grouping breaks no expression.

## Rotation

`rect`, `ellipse`, `vector`, `image` and `text` take `rotation=` — degrees, clockwise,
about the centre of the box. `x y w h` stay the **unrotated** box, so `@card.right` still
means the right edge of the box as written; a rotated node is placed by its box and then
turned. Frames and groups do not rotate. `rotation` is a number, so `rotation=$tilt` works.

```vd
rect sticker x=520 y=60 w=160 h=48 radius=24 fill=accent rotation=-8
text sticker.label x=520 y=72 w=160 h=24 align=center fill=#fff rotation=-8
  "New"
```

Give things that should turn together the same box centre, or the same rotation and
matching boxes, as above.

## Effects: blend and shadow

`frame`, `rect`, `ellipse`, `vector`, `image` and `text` also take:

- `blend=` — how the node mixes with what is under it: `multiply`, `screen`, `overlay`,
  `darken`, `lighten`, `color-dodge`, `color-burn`, `hard-light`, `soft-light`,
  `difference`, `exclusion`, `hue`, `saturation`, `color`, `luminosity`. Omit it for
  normal. A node blends only with the document beneath it. Groups take no blend.
- `shadow=` — drop shadows in CSS `box-shadow` spelling, quoted: `x y blur spread colour`,
  several separated by commas, the first on top. The colour may be a token. `inset` is not
  supported. A shadow follows the shape — round for an ellipse, glyph-shaped for text — and
  never changes the box, so layout and `@` references ignore it. Put a shadow you reuse in
  a token.

```vd
$lift = "0 1 2 0 #0F172A14, 0 8 24 -4 #0F172A29"

image hero.photo x=0 y=0 w=(100%) h=480 src=city fit=cover
rect hero.tint x=0 y=0 w=(100%) h=480 fill=accent blend=multiply opacity=0.8
rect card x=64 y=520 w=320 h=200 radius=16 fill=#FFFFFF shadow=lift
```

## Ids

An id is how everything else refers to a node, so make it guessable: `hero.title`,
`card.1`, `nav.logo`, `footer.legal`. Dots are ordinary characters and read as structure.

Ids must be unique across the document.
