# Editing a document

Given a document and a request to change it, reply with **statements**, not a new document.
An edit is usually a handful of lines against hundreds, and each statement is applied as an
operation — so it is undoable as one step and nothing else in the document is disturbed.

Statements are never indented, except for the body of an `insert`.

## `set`

```vd-patch
set $gap = 40                              a token
set hero.title size=34 weight=700          one or more properties
set card.2 w=(@card.1.w * 2)               an expression, same rules as anywhere
```

**Reach for the token first.** If four cards are laid out with `$gap`, "more breathing
room" is `set $gap = 40` — one line, and the relationships survive. Writing four `set`
statements with new coordinates destroys exactly what the document was for.

### Text

`content` is the one thing a node line does not carry, so `set` reaches it two ways:

```vd-patch
set hero.title content="A new headline"
```

```vd-patch
set hero.title
  "Open tooling for the "
  "agent era" fill=highlight
```

The first is for plain copy and is what you want almost always. The indented body is for
when the runs and their marks matter — the same shape a `text` node declares in a document.
Use one or the other, never both in a statement.

Rewording is always `set`. Never `delete` and `insert` to change copy: the node loses its
place in the document and every `@` reference to it breaks.

## `unset`

Removes a property so its default comes back.

```vd-patch
unset hero.sub opacity
unset card.1 stroke strokeWidth
```

For a token, use `delete $name` — a token is a thing, not a property.

## `insert`

```vd-patch
insert rect card.4 after card.3 x=(@card.3.right + $gap) y=(prev.y) w=@card.1.w h=200
insert text caption before footer x=$gap y=0 w=400 h=20
insert group cards in page
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
move logo before hero.bg
move card.3 in cards at 0
```

## `delete`

```vd-patch
delete hero.tagline
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
title "Pricing page"
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

## Things worth knowing

**Statements apply in order, against a document that grows as they do.** So this works:

```vd-patch
insert rect a after card.2 x=0 y=0 w=10 h=10
insert rect b after a x=0 y=0 w=10 h=10
```

**A bad statement fails alone.** The rest of the patch still lands, and the failure comes
back naming the line. Fix that line; do not resend the whole patch.

**Deleting leaves dangling references.** Nothing rewrites expressions for you, so removing
`logo` while `hero.title` reads `x=(@logo.right + $gap)` produces an `unknown-reference`
error on the next read. Before deleting, check what points at it, and either repoint those
first or delete them too:

```vd-patch
set hero.title x=$gap
delete logo
```

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
