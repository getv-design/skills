# Editing a document

Given a document and a request to change it, reply with **statements**, not a new document.
An edit is usually a handful of lines against hundreds, and each statement is applied as an
operation — so it is undoable as one step and nothing else in the document is disturbed.

Statements are never indented, except for the body of an `insert`.

## `set`

```vd-patch
set $gap = 40                              a token
set poster.title size=34 weight=700          one or more properties
set step.2 w=(@step.1.w * 2)               an expression, same rules as anywhere
```

`radius` is CSS's shorthand here as in CSS: writing it sets every corner, so `set badge
radius=0` squares a badge with corners of its own, and `set badge radius="16 16 0 0"` gives it
four. `set badge radiusTL=0` changes one corner and leaves the rest. When one statement
writes both, the later wins: `set badge radius=8 radiusTL=0` rounds three corners.

**Reach for the token first.** If four steps are laid out with `$gap`, "more breathing
room" is `set $gap = 40` — one line, and the relationships survive. Writing four `set`
statements with new coordinates destroys exactly what the document was for.

### Text

`content` is the one thing a node line does not carry, so `set` reaches it two ways:

```vd-patch
set poster.title content="A new headline"
```

```vd-patch
set poster.title
  "Open tooling for the "
  "agent era" fill=highlight
```

The first is for plain copy and is what you want almost always. The indented body is for
when the runs and their marks matter — the same shape a `text` node declares in a document.
Use one or the other, never both in a statement.

Rewording is always `set`. Never `delete` and `insert` to change copy: the node loses its
place in the document and every `@` reference to it breaks.

A list is rewritten whole the same way, one `para` line per item; to add, drop or reorder
an item, or turn a box into a list, send all its paragraphs:

```vd-patch
set s2.points
  para bullet "Revenue up 24% on Q2"
  para bullet level=2 "Teams accounts brought 40% of it"
  para bullet "Churn held at 2%"
```

`content="…"` writes plain words and drops the list.

A picture is put in a shape by naming the shape, and the crop is where the picture's own
box sits under it (`reference/nodes.md`, "A picture in a shape"):

```vd-patch
set avatar.photo mask=avatar.shape
set avatar.photo x=10 y=20
```

A `set` writes exactly what it says, so moving or resizing a masked picture with `set`
**crops** it: the shape stays where it is. To move both, set both by the same amount (or
put them in a group and move that).

## `unset`

Removes a property so its default comes back.

```vd-patch
unset poster.sub opacity
unset step.1 stroke strokeWidth dash strokeAlign
unset avatar.photo mask
```

For a token, use `delete $name` — a token is a thing, not a property.

`unset badge radius` takes away `radius` only: a corner written on its own (`radiusTL` and
the rest) stays. `set badge radius=0` squares every corner, and `unset badge radiusTL
radiusTR radiusBR radiusBL` hands every corner back to `radius`.

## `insert`

```vd-patch
insert rect step.4 after step.3 x=(@step.3.right + $gap) y=(prev.y) w=@step.1.w h=200
insert text caption before footer x=$gap y=0 w=400 h=20
insert group steps in page
  rect inner x=0 y=0 w=10 h=10
  text label x=0 y=12 w=40 h=10
    "A caption"
```

Placement comes before the properties:

|                |                                                                                                                             |
| -------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `after <id>`   | next to that node, following it                                                                                             |
| `before <id>`  | next to that node, ahead of it                                                                                              |
| `in <id>`      | appended inside a frame, canvas or group                                                                                    |
| `in <id> at 0` | at a specific position inside it                                                                                            |
| _omitted_      | appended to the canvas — **name the frame** in a multi-page document, or the node lands beside the pages rather than on one |

Children and text runs go indented underneath, exactly as in a document.

## `move`

Takes the same placements.

```vd-patch
move logo before band
move step.3 in steps at 0
```

## `delete`

```vd-patch
delete poster.tagline
delete $unused
```

In a brand, `delete` and a statement's kind remove one of its parts: `delete role muted`,
`delete rule mix`, `delete guide ui`, `delete logo mark`, and `delete review role.muted` for one
line of what is under review.

```vd-patch
delete rule mix
delete review role.muted
```

## `title`

Renames the document. It is the same line a whole document opens with.

```vd-patch
title "Jazz night poster"
```

## `kind`

Makes a design a deck, or a deck a design; only the header changes. Not for a brand.

```vd-patch
kind slides
```

## `notes`

Replaces one page's speaker notes with the quoted lines under it, a paragraph each; a bare
`notes <page>` removes them.

```vd-patch
notes s2
  "Lead with the 24%."
  "Then churn."
```

## `motion`

Replaces one page's whole motion block with the lines under it; a bare `motion <page>`
clears it. The format is in the motion reference.

```vd-patch
motion intro enter=fade
  intro.title enter=rise
  intro.chart enter=fade step
```

## `replace`

Writes one brand statement whole, as a brand document writes it: a `role`, `logo` or `rule`
line, or a `guide`, `review` or `brand` block with its lines under it. It stands in for the one
with the same id, which keeps its place, or is added last when there is none. `replace
review` replaces the whole list. `replace brand ember@4` moves the version and keeps what it
does not restate: the `id`, the summary unless a new one is given, and the history — a
`change` line under it is added, or replaces the one for its version. A role, logo or rule
takes no lines under it. The brand reference has the statements.

```vd-patch
replace role primary light=flame dark=#F06A3F
replace guide voice
  "Direct, warm, never cute."
  banned "seamless"
replace brand ember@4
  change 4 "Darker flame for contrast"
```

In a design that uses a brand, `replace theme dark` sets dark values a line at a time, and
`delete theme dark $primary` removes one; `delete theme dark` removes the block.

```vd-patch
replace theme dark
  $primary = #F06A3F
```

## `merge`

Replaces two or more shapes with one `vector`: what Union, Subtract, Intersect and Exclude
do in a design tool. Name the verb, then the shapes, in any order; `as` names the result.

```vd-patch
merge union badge.ring badge.dot as badge.mark
merge subtract ticket.bg ticket.notch
merge intersect lens.left lens.right as lens
merge exclude ring.outer ring.inner
```

| verb        | keeps                                                     |
| ----------- | --------------------------------------------------------- |
| `union`     | everything any of them covers                             |
| `subtract`  | the **bottom-most** shape, minus everything above it      |
| `intersect` | only where all of them overlap                            |
| `exclude`   | where an odd number of them overlap: the overlaps cut out |

The result is the bottom-most shape: it keeps that shape's place in the layer order, its
`name`, `fill`, `stroke`, `opacity`, `blend`, `shadow` and flags, and its id unless `as`
gives another. Its `d` is the merged outline in a fresh `view`, its box fitted to it, with
no `rotation`. Rotation and corners are baked into the path; arcs stay curves.

- Inputs are `rect`, `ellipse` and `vector`. **Text is refused** (`merge-not-a-shape`):
  there is no outline-text yet. So are images, frames and groups; name the shapes inside a
  group instead.
- Merge works on **filled areas**. A shape with `fill=none`, drawn by its stroke alone, is
  refused (`merge-stroke-only`); give it a fill first, or leave it out.
- A **hidden** shape is refused (`merge-hidden`); `unset` its `hidden` first. A **locked**
  one merges, and the result is locked if any input was.
- All of them must be on **one page or frame**. Shapes in different groups on that page
  are fine, and a group the merge leaves empty is removed; shapes on two pages are refused
  (`merge-across-frames`). Parts of an instance are refused (`merge-in-instance`): merge
  the shapes in the component, and every instance follows.
- `as` names a new id: one already used by a node that is not an input is refused
  (`duplicate-id`), and so is a property written where the id belongs (`missing-id`). A
  motion line on the bottom-most shape follows the result to its new id.
- A result with nothing in it, such as `intersect` of shapes that do not touch, is refused
  (`merge-empty`) and nothing is removed. If the geometry itself fails, the line is refused
  as `merge-failed` and nothing changes; the rest of the patch still lands.
- It is destructive. A `$token` or `@` reference on an input's `x`, `y`, `w`, `h`,
  `rotation` or corners is gone, written into the path. A node that read a shape the merge
  removed keeps where it was, as a number. Undo restores the inputs.
- Merge only what should become one shape for good. To keep a ring editable as two
  circles, leave it as two circles.

## Tables

A table's rows and cells are nodes (`reference/nodes.md`), edited with the same verbs. A cell
written without an id is its row's id and its column from 1, so the third cell of
`plans.team` is `plans.team.3`:

```vd-patch
set plans.team.3 content="$249"
set plans.team.3 fill=#FFE9A8 weight=700
insert row plans.edu after plans.team
  cell "Education"
  cell "Up to 500"
  cell "Free" align=right
delete plans.solo
set plans.head borderBottom="2 ink"
```

**A column is one cell in each row**, and the table's `cols` says its share. To add one, set
`cols` and insert a cell into every row at its place, counting from 0; widen a cell that
spans the table:

```vd-patch
set plans cols=2,1,1,1
insert cell plans.head.sup in plans.head at 2
  "Support"
insert cell plans.team.sup in plans.team at 2
  "Chat, same day"
insert cell plans.org.sup in plans.org at 2
  "Phone, 24/7"
set plans.note.1 colspan=4
```

To remove one, set `cols` and delete that cell from every row (`delete plans.head.2`, and so
on). Give each new cell an id, as above. A cell keeps the id it was read with, so after a
column goes in or out, `plans.team.3` is still the cell that was third: read the ids from the
document, do not count again.

- **Merge** with `colspan` or `rowspan` on the first cell, and delete the cells it now covers;
  **split** by setting it back to 1 and inserting cells in their place.
- **A row taller than the table's box is `table-overflow`**: raise the table's `h`.

## Slides on layouts

A slide on a layout (`reference/nodes.md`) is edited by its page id. Its words are an
override, its boxes take nodes with `in`, and a new slide is a page with `layout=` and no
`w`/`h`:

```vd-patch
set s2.title
  "Where we stand"
insert chart s3.mix in s3.right x=0 y=0 w=(100%) h=(100%) kind=doughnut labels=A,B
  series s3.mix.share values=60,40
insert frame s5 after s4 x=5520 y=0 layout=content
  override title
    "Next steps"
  slot body
    text s5.list x=0 y=0 w=(100%) h=(100%) size=32
      "Hire two engineers"
set s3 layout=content
```

- **`set s3 layout=content`** moves a slide: its words and slot content carry over by key,
  then into empty slots. What finds no slot stays on the page at the x and y it had in its
  slot and comes back as `unplaced-content`: move it into a slot or place it.
- **A new slide sits beside the last**, its `x` the last slide's plus its width and 100.
- **To change every slide**, `set` the layout's own node (`set content.title size=52`) or
  the master's; set on a slide, it changes that slide only.
- **A slide's parts are not inserted into, moved or deleted**, except into a slot. Hide one
  with `set s2.chrome hidden`.

## Things worth knowing

**Statements apply in order, against a document that grows as they do.** So this works:

```vd-patch
insert rect a after step.2 x=0 y=0 w=10 h=10
insert rect b after a x=0 y=0 w=10 h=10
```

**A bad statement fails alone.** The rest of the patch still lands, and the failure comes
back naming the line. Fix that line; do not resend the whole patch.

**Deleting leaves dangling references.** Nothing rewrites expressions for you, so removing
`logo` while `poster.title` reads `x=(@logo.right + $gap)` produces an `unknown-reference`
error on the next read. Before deleting, check what points at it, and either repoint those
first or delete them too:

```vd-patch
set poster.title x=$gap
delete logo
```

A shape a picture uses as its `mask` is the exception: `delete avatar.shape` also unsets
`mask` on every picture it cut, in the same patch, and they draw whole. Merging a mask
shape is fine; the picture is masked by the result. `move` a masked picture to another
page and **move its shape too**, or the mask is `mask-wrong-page` and the picture draws
whole.

**Moving a node changes what `prev` and `next` mean** for its neighbours. If siblings are
chained with `y=(prev.y)`, reordering them is fine — that is the chain doing its job — but
a `prev` reference on the node that used to be first now points somewhere new.

**A part of an instance is edited through its page id, and that edit stays on that
instance.** `set bill.nav.item.billing fill=accent` writes an `override` on `bill.nav`;
`unset` the same property and the part follows its component again. To change every page,
`set` the component's own node (`set item.billing fill=accent`). A part cannot be inserted
into, moved or deleted from an instance — those are refused, because its place is the
component's; hide it with `set bill.nav.item.billing hidden`, or change the component.
An instance goes in like any node, with its overrides under it:

```vd-patch
insert instance team.nav in team component=sidebar x=0 y=0
  override nav.indicator y=@item.team.y
```

**You cannot delete or move the canvas**, and you cannot give a group geometry. A frame is
an ordinary node: insert, move and delete it like any other, and everything on it travels
with it.
