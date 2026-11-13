# Node types

Thirteen. Every line is `<type> <id> key=value …`, and children are indented two spaces under
their container.

Every node also accepts `name="…"`, a human label with no effect on layout.

Every node but the canvas also accepts two bare flags, written last on the line:

- `hidden` — not drawn anywhere, export included. It still resolves, so `@badge.right`
  keeps working while `badge` is hidden. Hiding a container hides what is in it.
- `locked` — the person's pointer passes over it; you can still edit it, and add or remove
  the flag when asked.

## `frame` — a page

The node you almost always start with: a box with **its own coordinate space**. Inside it,
positions start at its top-left corner, and `%` measures the frame.

A document holds as many as you like: pages, slides or artboards. Frames nest, as an inset that clips.

| property     | type                 |                                      |
| ------------ | -------------------- | ------------------------------------ |
| `x`, `y`     | number or expression | where the page sits; both default 0  |
| `w`, `h`     | number or expression | the page size                        |
| `background` | paint or layers      | drawn behind everything on this page |
| `radius`     | see Corners          | rounds the background, and the clip  |
| `clip`       | flag                 | hide what overflows; off by default  |
| `skip`       | flag                 | left out of Play; top-level only     |
| `component`  | flag                 | other pages place copies of it       |
| `layout`     | component id         | the slide layout this page is on     |
| `slot`       | flag                 | in a layout: a part each page fills  |
| `hint`       | text                 | what an empty slot asks for          |

```vd
$gap = 24

frame page w=1280 h=720 background=paper
  rect band x=$gap y=$gap w=(100% - $gap * 2) h=320 fill=accent

frame notes x=1380 w=1280 h=720 background=paper
  text agenda x=$gap y=$gap w=600 h=48
    "What we agreed"
```

Lay pages out **left to right with a gap**, as above: overlapping pages draw over each
other.

## `canvas` — the surface

The outermost node, under the frames. **You almost never write it**: a document that
begins with frames gets one.

Write it only for a single-page document with shapes directly on it, where it is the page:

```vd
$gap = 24

canvas page w=1280 h=720 background=paper
  rect band x=$gap y=24 w=(100% - $gap * 2) h=260 fill=accent
```

| property     | type                 |                                    |
| ------------ | -------------------- | ---------------------------------- |
| `w`, `h`     | number or expression | optional; absent means unbounded   |
| `background` | paint or layers      | only meaningful when it has a size |

A canvas must be the **only** top-level node: put frames inside it, or leave it out.

## `rect` — a box

| property      | type                 | default   |
| ------------- | -------------------- | --------- |
| `x`, `y`      | number or expression | required  |
| `w`, `h`      | number or expression | required  |
| `fill`        | paint or layers      | `#CBD5E1` |
| `stroke`      | token or colour      | none      |
| `strokeWidth` | number or expression | 0         |
| `radius`      | see Corners          | 0         |
| `opacity`     | number or expression | 1         |

```vd
rect band x=$gap y=24 w=(100% - $gap * 2) h=260 fill=accent radius=16
```

## `ellipse` — an oval, or a circle

The ellipse inscribed in its box. A circle is `w` equal to `h`.

| property      | type                 | default   |
| ------------- | -------------------- | --------- |
| `x`, `y`      | number or expression | required  |
| `w`, `h`      | number or expression | required  |
| `fill`        | paint or layers      | `#CBD5E1` |
| `stroke`      | token or colour      | none      |
| `strokeWidth` | number or expression | 0         |
| `opacity`     | number or expression | 1         |

```vd
ellipse avatar.ring x=24 y=24 w=48 h=48 fill=none stroke=accent strokeWidth=2
```

## `vector` — any shape: icons, logos

An SVG path, drawn into a box.

| property      | type                               | default   |
| ------------- | ---------------------------------- | --------- |
| `x`, `y`      | number or expression               | required  |
| `w`, `h`      | number or expression               | required  |
| `d`           | SVG path data, quoted              | required  |
| `view`        | `"minX minY width height"`, quoted | `0 0 w h` |
| `fill`        | paint, layers or `none`            | `#CBD5E1` |
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

- **Over MCP, place an icon with `vd_add_icon`**, which writes this node; otherwise
  **copy the icon's `d` and `viewBox` verbatim** into `d` and `view`.
- The stroke is in page units and does not grow with `view`: an icon drawn with
  `strokeWidth=2` at 24 wants 4 at 48. A dot drawn as a tiny circle needs a stroke at
  least its diameter to read as solid.
- A line icon wants `fill=none` and a `stroke`. Without `fill=none` the shape is filled
  grey by default.
- Without `view`, the path draws at 1:1 inside its box. A divider is a `connector`.
- A path that cannot be read draws up to its first error and reports `bad-path`.

To turn a `rect` or `ellipse` into a free-form shape, replace its line with a `vector` of
the same id — same `x y w h`, `view="0 0 <w> <h>"`, and a `d` that traces it. The person
can do the same with **Convert to vector** (⇧⌘O) and then drag its points.

To make one shape of several — a badge from a ring and a dot, a ticket with a notch cut out —
`merge` them (`reference/edits.md`): `merge union badge.ring badge.dot` replaces them with
one `vector`. The person has the same as **Union**, **Subtract**, **Intersect** and
**Exclude** (⌥⇧U, ⌥⇧S, ⌥⇧I, ⌥⇧X).

## `image` — a picture

| property  | type                       | default  |
| --------- | -------------------------- | -------- |
| `x`, `y`  | number or expression       | required |
| `w`, `h`  | number or expression       | required |
| `src`     | a name from `asset`        | required |
| `fit`     | `cover`, `contain`, `fill` | `fill`   |
| `mask`    | the id of a shape          | none     |
| `radius`  | see Corners                | 0        |
| `opacity` | number or expression       | 1        |

```vd
asset hero = image/jpeg hash=sha256-1a77b0c3… bytes=412330

image cover.photo x=0 y=0 w=(100%) h=480 src=hero fit=cover
```

`src` names an entry in the document's asset registry — **never a path or a
URL**. `cover` fills the box and crops; `contain` fits the whole picture inside
it; `fill` stretches.

**You cannot invent an asset**: its line holds the hash and size of real bytes,
written by whatever added them (a paste, an import, `vd_add_asset`). A name with
no `asset` line reports `missing-asset` and draws a **slot**, a box labelled with
the node's `name` that the person drops a picture into: room for a picture you
cannot send.

### A picture in a shape: `mask`

`mask` names a `rect`, `ellipse` or `vector` on the same page, and the picture is cut to
its outline: a round avatar, a photo in a blob. The shape is **not drawn**: only its
outline counts (its `rotation`, corners, a vector's `d`, `view` and `rule`).

```vd
ellipse avatar.shape x=40 y=40 w=120 h=120
image avatar.photo x=20 y=30 w=180 h=140 src=face fit=cover mask=avatar.shape
```

- **The two boxes are the crop.** The picture keeps its own `x y w h`; move it and the
  shape stays put, so the part of the photo inside the circle changes. To move the whole
  thing, move both by the same amount.
- Where the shape sits in the layers does not matter.
- A shape with holes masks with its holes when it says `rule=evenodd`, which is what
  `merge` writes for a ring or a cut-out.
- The picture's corners, `rotation`, `opacity`, `blend` and `shadow` still apply; the
  shadow follows the cut.
- One shape may mask several pictures, hidden or not.
- Naming nothing is `unknown-reference`; a picture, frame, text or itself,
  `mask-not-a-shape`; a shape on another page, `mask-wrong-page`; one with no size,
  `mask-empty`. Each draws the picture **whole** and the shape as an ordinary shape.
- Inside a component, write the shape's own id: each instance's picture is cut by that
  instance's copy of the shape.
- `unset avatar.photo mask` lets go: the picture draws whole and the shape draws again.
  `delete avatar.shape` unsets `mask` on what it cut, in the same step.

The person has the same as **Mask with shape** and **Release mask** (⌥⌘M), and
double-clicks a masked picture to drag it inside its shape.

## `video` — a clip that plays

An `image` that plays: `x`, `y`, `w`, `h`, `src` (a `video/mp4` or `video/webm` asset),
`fit`, `mask`, `radius` and `opacity` as there, and the file's clock in milliseconds:

```vd
asset reel = video/mp4 hash=sha256-9b1e04c2… bytes=4100000 ms=12000

video hero.reel x=0 y=0 w=(100%) h=540 src=reel fit=cover start=2000 end=8000 poster=4000 muted loop
```

- `start`, `end` trim it; `poster` is the frame stills show (else `start`).
- A timed page plays it from its start, then holds the last frame, or `loop`s. `muted`
  silences it.
- Add one with `vd_add_asset`; a name with no clip is a slot.

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
| `slot`, `hint`     | flag, text                        | in a layout: words each page gives         |

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
`letterSpacing`, `link`, `field`. All literals: a run is not addressable, so nothing there can
be an expression.

`field=number` draws the page's number in place of the run's words, `field=count` how many
pages there are; `field=date` keeps the words written, and a PowerPoint export makes it a date
that updates. Put them in a layout's footer: `"Page "`, `"1" field=number`.

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

### Paragraphs: lists, levels, spacing

A `para` line opens a paragraph: its properties, then its words. Wrapped lines of a list item
line up under its words. Never type `•`, `1.` or leading spaces into the words.

```vd
text s2.points x=64 y=160 w=640 h=300 size=28
  para bullet "Revenue up 24% on Q2"
  para bullet level=2 marker=– "Teams accounts brought 40% of it"
  para number marker="a)" "First step"
  para number "Second step"
  para before=16 align=right "A plain paragraph"
```

`bullet` or `number` makes a list item, counted per `level` (1–9, each further in);
`marker` is a bullet's glyph or a number's pattern (`1.` `1)` `(1)` `a.` `A)` `i.` `I.`),
`start` the first number. A marker is drawn as its words are; `markerFill=primary` gives it
its own colour and `markerScale=0.8` its own size, as a multiple of the words'. `align` and `lineHeight` are the paragraph's own; `before` and
`after` are px of space around it. `indent` and `hang` (px: where its lines start, how far
the first hangs left) are rarely needed. Several runs go on lines indented under the `para`.
Once a body has `para` lines every paragraph is one (`para "Words"` for a plain one).

## `instance` — a copy of a component

Something drawn once and used on many pages — a footer, a legend, a border, a sidebar — is a
**component**: a top-level frame with the bare `component` flag. Put components in a row
**below the pages**, never on one; Play and every export of the pages leave them out.

In a deck (`kind slides`) a component is only ever a whole slide, the master or a layout
(below), at the slides' size. A footer or a card is never a component of its own there: it
goes on the master, on a layout, or on the slide it is drawn on.

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

### Slide layouts: `layout=` and `slot`

For a deck, give each kind of slide a **layout**: a component holding the title, the boxes
content goes in, and the master (logo, footer) as an instance. A page names it with
`layout=` and leaves out `w` and `h`: it is the layout's size, draws its background, and
gets every part of it. Name a layout's parts with the layout's id first, as a page's nodes
start with the page's; on the page they lose it (`columns.title` is `s3.title`), so a
page keeps its words when it moves to another layout. A layout's id is letters, digits and
`_`, never `-`: its parts point at each other (`@columns.left.right`), and a reference stops at
a `-`.

A part marked **`slot`** is one each page fills; `hint=` says with what. A text slot's words
in the layout are a sample, drawn on no page that has not said its own with `override`. A
slot that is a frame takes nodes in a `slot` block, and `100%` inside them is the slot's box:

```vd
frame columns x=0 y=1000 w=1280 h=720 background=surface component
  instance columns.chrome component=master x=0 y=0
  text columns.title x=64 y=56 w=(100% - 128) h=72 size=$h1 weight=700 slot hint=Title
    "Slide title"
  frame columns.left x=64 y=160 w=560 h=(100% - 240) slot hint="Text or chart"
  frame columns.right x=(@columns.left.right + 32) y=@columns.left.y w=@columns.left.w h=@columns.left.h slot

frame s3 x=0 y=0 layout=columns
  override title
    "Revenue vs churn"
  slot left
    chart s3.rev x=0 y=0 w=(100%) h=(100%) kind=column labels=Q1,Q2
      series s3.rev.a values=12,18
  slot right
    text s3.notes x=0 y=0 w=(100%) h=(100%)
      "Revenue up 24% on Q2"
```

- **Name parts by role** — `title`, `subtitle`, `body`, `left`, `right`, `picture`,
  `footer`, `number`, `date` — so PowerPoint gets real placeholders and pages carry over.
- **Nodes after the slot blocks** are the page's own, drawn above the layout.
- **`set s3 layout=content`** moves a page: overrides and slots carry by id, a slot
  with no match goes to the first empty one, and anything left stays on the page.
- **Never copy a layout's parts onto a page**; a slide that needs a different shape needs
  another layout.

### Speaker notes: `notes`

What the presenter says goes after the pages, never on them: `notes <page id>`, then a quoted
line per paragraph. Nothing draws it; PowerPoint export makes it the slide's notes.

```vd
frame s2 x=0 y=0 w=1280 h=720

notes s2
  "Lead with the 24%: it is the number people will repeat."
  "Churn is flat, not falling."
```

## `connector` — a line, or an arrow joining nodes

Placed by its ends, `x1`, `y1`, `x2`, `y2`, not a box. An end written as a reference
follows that node when it moves:

```vd
connector flow.a2b x1=@step1.right y1=@step1.cy x2=@step2.left y2=@step2.cy end=arrow
connector rule x1=40 y1=500 x2=520 y2=500 stroke=muted dash="4 4"
```

- `route=`: `straight` (default), `elbow` or `curve`. An end on a side leaves along it.
- `start=`, `end=`: `none` (default), `arrow`, `triangle`, `dot`, `bar`.
- A dark line 2 wide, or `stroke` and `strokeWidth`; also `dash`, `cap`, `opacity`,
  `blend`, `shadow`, `blur`.
- Its box is its ends' bounds, so a label is a `text` centred on `@flow.a2b.cx`.
- An end on a group or on another page is a warning.

## `chart` — numbers, drawn

Write the data and the runtime draws the bars, ticks and key; never draw a chart as rects.
It holds only `series` lines.

```vd
chart rev x=40 y=200 w=720 h=400 kind=column labels=Q1,Q2,Q3 format="$#,##0" grid
  series rev.a name="2025" values=12000,18000,24500 fill=muted
  series rev.b name="2026" values=15000,,40000 fill=accent
```

- `x`, `y`, `w`, `h`: the whole chart, key included. `kind`: `column` (default), `bar`,
  `line`, `area`, `pie`, `doughnut`, `scatter`, `km`; a pie draws its first series.
- `labels`: one per value, commas and no spaces; quote one with a space or comma:
  `labels="North America",Europe`.
- Bare flags: `stacked`, `percent` (stacked to 100%), `grid`, `numbers` (each value
  written on), `smooth` and `markers` (lines).
- `legend`: `none` `top` `bottom` `left` `right`, shown unasked for several series or a pie.
  `axis=none` hides the value axis.
- `format`: a spreadsheet code, `0.0`, `"$#,##0"`, `0%`, `#,##0,"k"`. Values are plain
  numbers: `0.12` with `format=0%`, never `12%`.
- `min`, `max`: the value axis' ends, else from the data with 0. `gap`: between categories,
  % of a bar (60). `hole`: a doughnut's, % (55). `radius`: bar corners.
- `scatter`: a dot at each pair, on two number axes fitted to the data (no zero needed).
  Each series gives `xvalues=` beside `values=`, one per value; no `labels`. `xmin`,
  `xmax`: the horizontal axis' ends.
- `km`: a Kaplan–Meier survival curve, 0–100% over time from 0, a + at each censored
  subject. Raw data, as Prism takes it: each series gives `xvalues=` (each subject's time)
  and `events=` (1 event, 0 censored), and the curve is worked out. Or a finished curve:
  `xvalues=` (step times), `values=` (survival in percent, 85 not 0.85) and `censored=`
  (tick times). Flags: `median` (dashed at 50%), `band` (95% CI) and `risk` (number at
  risk under the axis) — `band` and `risk` need `events`. `xmin`, `xmax` as a scatter.
- `font`, `size`, `fill`: its text. `stroke`: axis and grid lines. `opacity`.
- Exported to PowerPoint it stays a chart, its data editable there; drawn as rects it
  would be loose shapes.

## `series` — one row of a chart's data

Only in a `chart`: `name` (the key's word), `values` (one per label; an empty entry,
`12,,24`, is missing and drawn as nothing), `fill`, and `fills` (a colour per value: a
pie's slices, or `fills=,,,lime` to pick out one bar). In a scatter, `xvalues` place the
points: `series fit xvalues=152,160,171 values=51,58,70`. In a `km`, `events` or `censored`
go beside them: `series os.a xvalues=6,6,9,10 events=1,1,0,1`. Change one with
`set rev.b values=15,22,40`.

## `table` — a grid of text, laid out by the runtime

Write the rows and cells; the runtime works out every column, row and line. Never build
a table from rects and texts. A table holds only `row`s, a row only `cell`s.

```vd
table plans x=64 y=160 w=1152 h=240 cols=2,1,1,1 header band headerFill=primary headerText=#FFFFFF bandFill=wash lines=rows
  row plans.head
    cell "Plan"
    cell "Seats"
    cell "Price" align=right
    cell "Support"
  row plans.team
    cell "Team"
    cell "50"
    cell align=right
      "$199" weight=700
      " /mo" fill=muted
    cell "Phone, 24/7" fill=accent
  row plans.note
    cell "Prices exclude VAT" colspan=4 italic
```

- `x`, `y`, `w`, `h`: the box. Rows stack from its top, each as tall as its tallest cell
  (or its row's `h`); rows that need more than `h` are drawn below it and reported
  `table-overflow`, so raise `h`.
- `cols`: each column's share of `w`, as CSS `fr`: `cols=2,1,1`. Absent, equal.
- Flags: `header` (first row bold, `headerFill`, `headerText`), `band` (`bandFill` on every
  second body row, the first one first), `firstCol` (first column bold, `firstColFill`).
- `fill`: the body's background; `text`: the letters' colour. Both take tokens.
- `lines`: `all` (default), `rows`, `cols`, `outer`, `inner`, `none`, in `stroke` at
  `strokeWidth` (1).
- `padding`: inside each cell, one number or `"5 10"` (top/bottom, sides; the default) or
  `"t r b l"`.
- Text for every cell: `font`, `size` (16), `weight`, `lineHeight`, `italic`, `align`,
  `valign`. `opacity`.
- Exported to PowerPoint it stays a table, typed in and added to there; drawn as rects it
  would be loose shapes.

## `row` — one row of a table

Only in a `table`, and its id is required. `h`: the least it is drawn. `header` makes it
a header row too. `fill`, `text`, and the text defaults `font`, `size`, `weight`,
`lineHeight`, `italic`, `align`, `valign` for its cells. `border`, `borderTop`,
`borderRight`, `borderBottom` and `borderLeft` are each of its cells' edges (see `cell`):
`borderBottom="2 ink"` rules a line under the whole row. Add one with
`insert row plans.ent after plans.team` and its cells indented under it.

## `cell` — one cell of a row

Only in a `row`. Its words are a text node's: one run on the line (`cell "Plan"`), or
marked runs and `para` lines under it. Its id may be left out: it is then its row's id
and its column from 1, so `plans.team.3` is the third column of `plans.team`; patch it
with `set plans.team.3 content="$249"`. `colspan` and `rowspan` merge from this cell, and
what they cover is not written, so the next cell takes the next free column. `fill` is its
background and `text` its letters (a run's `fill` is still its letters). `padding`, and
`font`, `size`, `weight`, `lineHeight`, `italic`, `align`, `valign` for its words.
`@plans.team.3.top` is an error: a row or a cell has no box.

A cell's own edges go over the table's `lines`: `border` for all four, or `borderTop`,
`borderRight`, `borderBottom`, `borderLeft`. Each is `none`, a width (`2`), a colour
(`primary`), or both quoted with the width first (`"2 #1F2937"`); what is left out is the
table's `stroke` or `strokeWidth`. On a `row` they are every one of its cells'. Where two
cells share an edge a line beats `none`, and the heavier line beats the lighter.

## `group` — structure, and nothing else

```vd
group steps
  rect step.1 x=$gap y=0 w=200 h=120
  rect step.2 x=(@step.1.right + $gap) y=(prev.y) w=@step.1.w h=120
```

A group has **no `x`, `y`, `w` or `h`**: it is transparent to layout. Its children keep
the coordinate space of whatever contains the group, and `parent`, `prev`, `next` and `%`
see straight through it.

Group what belongs together; it breaks no expression.

## Corners: radius

`rect`, `image`, `video` and `frame` round their corners with `radius`. One number rounds all four:

```vd
rect card x=0 y=0 w=320 h=200 fill=#F1F5F9 radius=16
```

**Corners that differ take CSS's shorthand, in quotes**: top-left, then clockwise — top-right,
bottom-right, bottom-left. A tab rounded only on top:

```vd
rect tab x=0 y=0 w=320 h=48 fill=#2563EB radius="16 16 0 0"
```

As in CSS, two values are top-left and bottom-right, then top-right and bottom-left
(`radius="24 0"`), and three are top-left, then top-right and bottom-left, then bottom-right.
The shorthand holds plain numbers only.

**To bind a corner, or to change one corner of a bound radius**, write that corner on its
own: `radiusTL`, `radiusTR`, `radiusBR` and `radiusBL` each take a number or an expression,
and each overrides `radius` for its corner.

```vd
$r = 12

rect chip x=0 y=0 w=120 h=40 fill=#2563EB radius=$r radiusBL=0
```

- A radius too big for its box is drawn as CSS draws it: all four shrink together until
  they fit. `radius=999` on a short rect is a pill.
- On a `frame`, `radius` rounds the `background`, the shadow and, with `clip`, what it
  clips: a rounded photo frame is a frame with `background`, `radius` and `clip`.
- In a patch, `radius` sets every corner, as CSS's shorthand does: `set card radius=0`
  squares a card whatever its corners said, a part of an instance included, and
  `set card radius="16 16 0 0"` replaces `radius` with four corners. In one statement the
  later spelling wins. `set card radiusBL=0` changes that corner alone.
- `unset card radius` removes `radius` and leaves any corner of its own. To square every
  corner, write `set card radius=0`.

## Strokes

`frame`, `rect`, `ellipse`, `vector`, `image`, `video` and `text` take `stroke=` (token or colour)
and `strokeWidth=`. Give both: no width draws nothing.

```vd
frame card x=0 y=0 w=320 h=200 radius=16 clip stroke=line strokeWidth=1
rect dots x=400 y=0 w=120 h=40 fill=none stroke=accent strokeWidth=3 dash="0 8" cap=round
```

- `dash="6 4"`: dash, gap, repeating, quoted. `dash="0 8"` with `cap=round` is dotted.
- `cap=` (`butt`, `round`, `square`) ends each dash on a `rect`, `ellipse`, `frame`, `image` or
  `video`.
- `strokeAlign=` is `center` (default), `inside` or `outside`, for a closed outline
  (`stroke-align-open` on an open path: drawn centred). `text` has none. On a `vector`,
  dashes are not exact; an outline that crosses itself draws across its fill, so
  `merge union` it first. A page's border needs `inside`: the page edge cuts the rest.
- A `frame`'s stroke draws over its contents; an `image`'s follows its corners or `mask`;
  a `text`'s outlines the letters behind the fill.

## Rotation

`rect`, `ellipse`, `vector`, `image`, `video` and `text` take `rotation=` — degrees, clockwise,
about the centre of the box. `x y w h` stay the **unrotated** box, so `@card.right`
means its right edge as written. Frames and groups do not rotate. `rotation` is a number, so `rotation=$tilt` works.

```vd
rect sticker x=520 y=60 w=160 h=48 radius=24 fill=accent rotation=-8
text sticker.label x=520 y=72 w=160 h=24 align=center fill=#fff rotation=-8
  "New"
```

Give things that should turn together the same box centre, or the same rotation and
matching boxes, as above.

## Effects

`frame`, `rect`, `ellipse`, `vector`, `image`, `video` and `text` also take:

- `blend=` — how the node mixes with what is under it: `multiply`, `screen`, `overlay`,
  `darken`, `lighten`, `color-dodge`, `color-burn`, `hard-light`, `soft-light`,
  `difference`, `exclusion`, `hue`, `saturation`, `color`, `luminosity`. Omit it for
  normal. A node blends only with the document beneath it. Groups take no blend.
- `shadow=` — CSS `box-shadow` spelling, quoted: `x y blur spread colour`, several
  separated by commas, the first on top; `inset` first makes an inner shadow. The colour
  may be a token. A shadow follows the shape and never changes the box. Put one you reuse
  in a token.
- `blur=` — layer blur in px: the node and its shadows, a frame with all it holds.
- `backdrop=` — background blur, not on `image`, `video` or `text`: what lies under the node on
  its page, blurred, seen through it. Glass is this with a translucent fill and a light
  stroke.

```vd
$lift = "0 1 2 0 #0F172A14, 0 8 24 -4 #0F172A29"

image hero.photo x=0 y=0 w=(100%) h=480 src=city fit=cover
rect hero.tint x=0 y=0 w=(100%) h=480 fill=accent blend=multiply opacity=0.8
rect card x=64 y=520 w=320 h=200 radius=16 fill=#FFFFFF shadow=lift
```

## Ids

An id is how other nodes refer to one, so make it guessable: `poster.title`,
`step.1`, `legend.key`, `footer.legal`. Dots are ordinary characters and read as structure.

Ids must be unique across the document.
