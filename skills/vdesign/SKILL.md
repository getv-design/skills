---
name: vdesign
description: Author and edit vDesign documents — posters, slides, hero sections, card grids, any laid-out visual document — in the .vd DSL. Use when asked to create, change, or read a vDesign document, a .vd file, or when a task involves designing a layout that should stay editable and relational rather than being flattened to fixed coordinates.
license: CC-BY-4.0 (https://creativecommons.org/licenses/by/4.0/). Credit vDesign.
---

# Authoring vDesign documents

A vDesign document is a tree of boxes whose geometry is **declarative**. A property holds
either a number or an expression, and expressions are stored as written and resolved on
read. `w=(100% - $gap * 2)` stays that formula in the file forever; change `$gap` and
everything bound to it moves.

That is the entire point of the format, and it is what you are being asked for. A document
full of literal coordinates is a worse version of a Figma file. A document of expressions
and tokens can be edited by changing one number.

## The output contract

Emit **only DSL lines**. No prose before or after, no explanation, no JSON. A fenced block
is tolerated but unnecessary.

## The grammar in one screen

```vd
version 4
kind design
title "Launch hero"
$gap = 24
$accent = #5b5bd6
$paper = #ffffff
$highlight = #ffd166

font display = Inter
  fallback "system-ui, sans-serif"

frame page w=1280 h=720 background=paper
  rect hero.bg x=$gap y=24 w=(100% - $gap * 2) h=260 fill=accent radius=16
  text hero.title x=(@hero.bg.left + 32) y=64 w=560 h=82 size=34 weight=700 fill=paper
    "An open substrate for the "
    "agent era" fill=highlight
  group cards
    rect card.1 x=$gap y=(@hero.bg.bottom + $gap) w=((100% - $gap * 4) / 3) h=200
```

- **The first lines are the header.** `version 4` and `kind design`. Write both. `design`
  is the free-form canvas; a document that does not say which it is leaves every reader to
  guess from its contents. The one other kind is a brand, `version 5` and `kind brand`, which
  `reference/brand.md` covers — make one only when asked for a brand.
- **A design in a brand** has a `brand ember@3` line under its title. Read the brand's brief
  first (`vd_read_brand`), name its roles in paint (`fill=primary`, `background=background`)
  and never write a colour out, and never write the brand line yourself: `vd_apply_brand` does.
- **Then `title "…"`** — a short human name for what you are making, two to five words
  ("Coffee shop landing", not "Document"). It is what the tab shows, so write it before the
  first page: the name should appear while the design is still arriving.
- **Two spaces of indent is one level.** Indentation is the only nesting.
- A line is `<type> <id> key=value key=value …`.
- A **quoted line** under a `text` node is one run of its content.
- A **boolean is a bare flag**: `italic`, never `italic=true`.
- **There are no comments.** `#` already starts a colour. Anything trailing a value is an
  error rather than something to ignore, so do not annotate the document.

### The one rule worth memorising

> A value is an **expression** if it starts with `$`, `@`, or `(`. Everything else is a
> literal.

So `x=$gap` and `x=@logo.right` need no parentheses, and everything else that is an
expression does: `y=(prev.bottom + 12)`, `w=(50%)`, `y=(parent.h / 2)`. A quoted value is
always a literal, which is how you write a string that starts with a sigil: `fill="$brand"`.

## Rules that decide whether the output is any good

**1. Write relationships, not coordinates.** This is the difference between a vDesign
document and a screenshot. Before writing a number, ask what it is derived from.

```text
✗ rect card.2 x=148 y=308 w=395 h=200
✓ rect card.2 x=(@card.1.right + $gap) y=(prev.y) w=@card.1.w h=200
```

**2. Put every repeated value in a token first.** A colour used twice, a spacing used
three times, a type scale — all tokens. `$gap` used eleven times is the compression, and it
is also the edit surface: "make it roomier" becomes one line.

**3. Only reference ids you have already written.** References resolve against the
document, and a consumer may be rendering your output as it streams. Write `logo` before
the headline that sits beside it.

**4. Give ids a role, not a number.** `hero.title`, `card.1`, `nav.logo` — an id should be
guessable by someone who has not read the file. Never `node1`, `rect5`.

**5. Text height is authored — estimate it.** Nothing measures glyphs for you. For a box
of width `w` at font size `size`:

```text
characters per line ≈ w / (size * 0.5)
lines               = ceil(length of the text / characters per line)
h                   = ceil(lines * size * 1.2)
```

Round up. Too small is reported as a `text-overflow` problem; too large just leaves space.

**6. `fill` takes a token name or a CSS colour, never an expression.** `fill=accent`,
`fill=#5b5bd6`. Sizes and positions bind; colours do not.

**7. A group has no geometry.** It holds children and nothing else — no `x`, `y`, `w`, `h`.
Its children keep the coordinate space of whatever contains the group, and `parent`, `prev`
and `next` see straight through it.

**8. A page is a `frame`, and more than one page is more than one frame.** Start the
document with a frame; you do not write a `canvas` line at all. Each frame is its own
coordinate space, so `x=$gap` means `$gap` from _that page's_ left edge and `100%` means
that page's width. `parent.cx` is that page's centre too, and `parent.left` is `0`, so the
same line centres something on every page. Lay several out left to right with a gap between
them — nothing stops one page being drawn over another, so do not overlap them.

```vd
$gap = 24

frame page w=1280 h=720 background=paper
  rect hero.bg x=$gap y=$gap w=(100% - $gap * 2) h=320 fill=accent

frame pricing x=1380 w=1280 h=720 background=paper
  text pricing.title x=$gap y=$gap w=600 h=48 size=34 weight=700
    "Pricing"
```

**9. Centre a label by giving it the box it sits in, never an offset.** A button's label,
a badge's number, a chip's text: same `x y w h` as the shape behind it, then
`align=center valign=middle`. The renderer centres the line inside the box, so it stays
centred at any size or font. A guessed `y=(@cta.top + 14)` is off by a few pixels and
drifts the moment the button or the type changes.

```vd
rect cta x=$gap y=400 w=180 h=48 fill=accent radius=10
text cta.label x=@cta.x y=@cta.y w=@cta.w h=@cta.h align=center valign=middle fill=paper
  "Get started"
```

**10. Draw a repeated part once, as a component.** A sidebar, nav bar or card that appears on
several pages is a frame marked `component`, in a row below the pages, and each page places
an `instance` of it. What one page shows differently — its selected menu item, its own
title — is an `override` line under that instance, never a copy of the parts:

```vd
frame nav x=0 y=900 w=1280 h=64 background=surface component
  text nav.title x=$gap y=0 w=400 h=64 valign=middle
    "Overview"

frame home x=0 y=0 w=1280 h=720
  instance home.nav component=nav x=0 y=0

frame pricing x=1380 y=0 w=1280 h=720
  instance pricing.nav component=nav x=0 y=0
    override nav.title
      "Pricing"
```

`reference/nodes.md` has the rest: a part's page id is the instance's id and its own
(`pricing.nav.nav.title`), and references inside a component are written unprefixed.

## Editing an existing document

When given a document and asked to change it, reply with **statements**, not a new
document. Five verbs, plus `title` to rename the document and `merge` to make one shape of
several:

```vd-patch
title "Pricing page"
set $gap = 40
set hero.title size=($display * 1.2) weight=800
set hero.sub content="A shorter subhead."
unset hero.sub opacity
insert rect card.4 after card.3 x=(@card.3.right + $gap) y=(prev.y) w=@card.1.w h=200
insert frame pricing x=(@page.right + 100) w=@page.w h=@page.h background=paper
move logo before hero.bg
delete hero.tagline
merge union badge.ring badge.dot as badge.mark
```

An `insert` with no placement lands on the canvas, which in a multi-page document is
_beside_ the pages rather than on one. Say `after <id>` or `in <frame>` whenever the new
node belongs to a page.

Copy is changed with `set … content=`, or with an indented body when the marks matter.
Never `delete` and `insert` a text node to reword it — it loses its place and every `@`
reference to it.

Prefer changing a **token** over changing the nodes that use it — that is usually what the
request actually means. "More breathing room" is `set $gap = 40`, not eleven `set`
statements.

See `reference/edits.md` for placement, and for what happens when you delete something
another node references.

Two flags come from the person more often than from you: `hidden` (not drawn, export
included) and `locked` (their pointer passes over it). Keep them as you find them. Hide with
`set card.2 hidden` only when asked to hide — when asked to remove something, `delete` it,
because a hidden node is still in the file for the next reader to wade through. `unset
card.2 hidden` shows it again.

A top-level frame is a slide when the document is played. For a backup or reference slide
that should stay in the deck but out of the talk, write `skip` on its frame (`set appendix
skip`) — don't hide it, which takes it off the canvas and out of every export too, and don't
delete it.

## Motion and video

How a page moves in time is a `motion` section after the last page: transitions between
slides, parts of a page appearing in order, buttons that go to another page — and, for a
video, words entering one by one, numbers counting up, strokes drawing on and a camera moving
across the page. The canvas at rest and every still export show each page as written, so write
the page as it looks when finished and let motion say how it gets there.

```vd
motion intro enter=fade
  intro.title enter=rise by=word
  intro.chart enter=fade step
  intro.cta enter=pop go=#pricing
```

Add it when the person is making something to present, click through or post as a video, and
keep a deck light. Never copy a page to animate between the copies. A web page taller than a
screen gets `motion <page> scroll`. A video can have one music track, `music <name>` under its
header, and `bpm=` to time lines in beats — from the person or the file, never a guess.
`reference/motion.md` has the rest; read `reference/video.md` before making a video (an ad, a
promo, a social clip), and check one with `vd_frames`.

## Icons

Never write an icon's path from memory: it is slow, and it comes out roughly right at best.
Icons come from the Lucide set, about 1,900 line icons, by name.

- **Over MCP**, find one with `vd_search_icon` (`"settings"`, `"notification"`) and place it
  with `vd_add_icon`, which writes the `vector` for you — the icon's path, Lucide's stroke,
  round caps. Its `x`, `y` and `size` take expressions, so place an icon by relationship like
  anything else: `x="(@cta.right - 32)"`.
- **In the app**, find one with `search_icons` and get its line with `use_icon`, before you
  write any DSL; copy that `vector` line into your reply as it comes back.
- **Where no icon tool is offered**, draw it as a `vector` (`reference/nodes.md`).
- **Lucide has no brand logos** — GitHub, X, YouTube. A logo is an image the person supplies:
  leave it as a slot (see Images).

A placed icon is an ordinary vector named `lucide:<name>`. Recolour it with
`set <id> stroke=…`; resize it with `w` and `h`, and scale `strokeWidth` with them — 2 at 24.

## Images

A picture is an `image` node that names an asset: `image hero.logo x=40 y=32 w=160 h=48
src=logo fit=contain`. The asset's registry line (`asset logo = image/png hash=…`) and its
bytes are **never yours to write** — hashes come from the app. You only ever write `src=`.

- **In the app**, images the person attached are offered by name at the top of the request
  ("Images you can put in the design"). Place one with `src=<that name>`; an edit uses
  `insert image <id> in <frame> …`. Size it to the image's aspect ratio, and prefer
  `fit=contain` for a logo.
- **Over MCP**, a picture at a public URL goes in with `vd_add_asset` first, then you place
  it. `vd_assets` lists what a document already has.
- **A picture you cannot send** — one the person gave you in chat, or has on their machine —
  becomes a **slot**. You cannot pass on an image you can see, and no tool reads file paths,
  so do not try: write the image anyway, with a `src` nothing supplies yet and a `name`
  saying what goes there:

  ```vd
  image nav.logo x=40 y=24 w=120 h=40 src=logo fit=contain name="Your logo"
  ```

  The canvas shows it as a labelled box the person drops the file into, whenever they like.
  Size and place it as if the picture were there, carry on with the design, and tell them
  where it is ("drop your logo onto the box in the top-left"). Every image with the same
  `src` fills together. `vd_assets` lists the slots still waiting.

A `src` naming nothing reports `missing-asset`. That is what a slot is, so do not "fix" it by
deleting the image; do fix a `src` you simply misspelled.

## Looking at your work, and exporting

**Over MCP (desktop and CLI), `vd_export` is how you see what you made.** `vd_read` is text, and a layout that
reads fine can still look wrong — text overflowing, two cards overlapping. After a build or
a large edit, export the frame as `png` with `save: false` and look at the picture that comes
back; fix what you see, then look again.

When the person asks for a file — "export it as a PDF", "give me a PNG of the cover" — call
`vd_export` with `save: true`. Formats: `png`, `jpg`, `webp`, `svg`, `pdf`, `ai` (Illustrator:
one artboard of the whole canvas, or `artboards: "frames"` for a file per frame), `html` and
`pptx` (PowerPoint: a slide per frame, with shapes and text still editable). In the app the
person chooses where it goes, and may cancel; if they do, do not try again unasked. Only
save when asked — a look is `save: false`.

**With no vDesign tools at all** — this skill loaded on its own, by an agent that can write
files — the design is a file. Write the document to `<title>.vd`, holding exactly the lines
the output contract describes, and give the person that file. It opens in the vDesign app,
at getv.design/app, and in the VS Code extension, where they can see it, keep editing it and
export it to any format above. You cannot look at it first, which is one more reason to
place everything by relationship and leave text room to breathe. Do not draw it some other
way to show them: a picture from anything but vDesign is not what the design looks like.

## Reference

Each file is named by its path in this skill. Over MCP, `vd_skill` returns any of them by that
path; in the flattened system prompt they follow below.

- `reference/grammar.md` — the expression language in full: edges, percentages, relative
  references, precedence; plus tokens, paint (colours, gradients, layers), fonts and the asset
  registry.
- `reference/nodes.md` — every node type and every property it accepts: `frame`,
  `canvas`, `rect`, `ellipse`, `vector`, `image`, `text`, `instance`, `group`.
- `reference/edits.md` — the mutation dialect in full.
- `reference/motion.md` — motion: page transitions, reveals, buttons, timing, the camera, beats
  and music.
- `reference/video.md` — videos for social media and websites: sizes, storyboards, recipes,
  cutting to the beat.
- `reference/brand.md` — brands: a `kind brand` document's roles, logos, rules and guides, how
  a patch changes one, how one is made, and how a design uses one.
- `examples/` — complete documents: `hero.vd`, `cards.vd`, `slide.vd`, `deck.vd` (three pages
  with motion), and `promo.vd`, a 9-second video storyboard.
