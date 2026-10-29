# Brands

A brand is a document of its own kind: `kind brand`, written as `version 5`. It holds what a
brand is — its palette, the roles those colours play, its type, its logo, the rules a design
is held to, and how it speaks. Inside a brand, paint names the palette's tokens
(`fill=lime`); a role is a name for what a colour is for, which a design that uses the brand
will write as `fill=primary`.

```vd
version 5
kind brand
title "Ember"
brand ember@3 id=6f1c2a9e-4b7d-4e0a-9c55-2f8e1d3b7a10
  summary "Warm, plain-spoken kitchenware for people who cook every day."
  change 3 "Flame darkened for contrast on cream" at=2026-10-20
  source site url=https://ember.example at=2026-10-26

$ink = #1A1410
$cream = #FBF5EC
$flame = #E8562A

font display = Fraunces
  fallback "Georgia, serif"

role background light=cream dark=ink
role text light=ink dark=cream
role primary light=flame
role on-primary light=cream

rule no-gradients must "No gradients" check=no-gradient
rule reads must "Text reads at 4.5:1" check=contrast fg=text bg=background min=4.5
rule mix should "About 70% background, 25% ink, 5% flame"

guide voice
  "Direct, warm, never cute. Say what happened, then what to do."
  banned "seamless"
  rewrite from="Oops! Something went wrong." to="We couldn't save. Try again."

review
  role.primary from=site note="The most-used colour on the site's buttons"
```

The palette is the document's ordinary tokens and the type is its ordinary fonts. What a
brand adds is six statements, all at the start of a line.

## `brand`

`brand <slug>@<version>`, with an optional `id=`. The slug is lower case, digits and dashes —
the name an agent writes. The version counts saves from 1. The `id` stays the same for the
life of the brand, so two brands that share a slug are still told apart. Under it, indented:

- `summary "…"` — what the brand is, in a sentence.
- `change <version> "…" at=<date>` — what each version changed, newest first.
- `source <kind> key=value…` — where the brand came from: `source site url=…`,
  `source file name=… pages=…`.

## `role`

`role <name> light=<token or colour> dark=<token or colour>`. A role is what a colour is
for. Name the token bare (`light=cream`); a colour is written as one (`light=#6B5E55`). A
role with no `dark=` uses its light value in both themes.

Fill the standard roles, so that a design can be moved from one brand to another without
renaming anything: `background`, `surface`, `raised`, `border`, `text`, `muted`, `primary`,
`on-primary`, `accent-text`, `success`, `danger`, `warning` and `info`. A brand may add roles
of its own. The fonts follow the same idea: name them `display`, `body` and `mono`.

## `logo`

`logo <id> asset=<name> min=<px> clear=<fraction>`. The asset is in the registry like any
picture. `min` is the smallest it may be drawn; `clear` is the space around it, as a
fraction of its height.

A brand has four places for one, by id: `mark` for light grounds, `mark-dark` for dark ones,
and `favicon` and `favicon-dark`, the icon a browser shows in each theme. A mark drawn for a
dark ground is white, and on a light ground it cannot be seen: put it in `mark-dark`, never
in `mark`. A picture the person attached goes in with `replace logo mark asset=<name>`,
using the name it was attached under; the app registers it.

## `rule`

`rule <id> must|should "<what it asks>"`, then optionally `check=` and the check's own
parameters. A rule with a check can be tested by code; one without is guidance for whoever is
designing. The checks are `check=no-gradient`, `check=contrast` (with `fg=`, `bg=` naming
roles and `min=` a ratio), `check=font-only` and `check=logo-min`.

## `guide`

`guide <id>` on its own line, and under it the guide's lines: quoted prose, and

- `banned "…"` — a word or phrase the brand never uses;
- `avoid "…"` — something to stay away from;
- `rewrite from="…" to="…"` — a before and after;
- `ref asset=<name> note="…"` — a reference picture;
- `approved asset=…` and `rejected asset=…` — examples, in `guide examples`.

The usual guides are `naming`, `voice`, `imagery` and `examples`. Any other id is a guide too
(`guide ui`).

## `review`

What is still waiting for a person to confirm, one line each, by what it covers:
`role.muted`, `guide.voice`, `logo.mark`, `font.display`, `rule.mix`, with `from=` saying
where the guess came from and `note=` why. A brand nobody has anything left to confirm has no
`review` at all.

## Editing a brand

A brand statement is changed with `replace`, written as the document would write it, and
removed with `delete` and its kind. Both are in the edits reference.

```vd-patch
replace role primary light=flame dark=#F06A3F
delete rule mix
```

## Making a brand

`vd_create_brand` makes one from what the person has: their site (`{ url }`), text you
read in their guidelines (`{ text }` — the lines with colour values, font names and banned
words), design tokens (`{ tokens }`), a stylesheet (`{ css }`), or a vDesign document
(`{ document }`). Code reads the exact values and opens a draft the person sees; its
answer is a digest of what it found.

Then write what code cannot know — how the brand speaks, what it never says, the rules a
design is held to — as `replace` statements, in `judgement` or with `vd_patch`:

```vd-patch
replace brand ember@1
  summary "Warm, plain-spoken kitchenware for people who cook every day."
replace guide voice
  "Direct, warm, never cute. Say what happened, then what to do."
  banned "seamless"
replace rule no-gradients must "No gradients" check=no-gradient
replace rule reads must "Text reads at 4.5:1" check=contrast fg=text bg=background min=4.5
```

Pick colours by the token names the digest gives; never type a hex code it found. What you
inferred rather than read stays in `review` until the person confirms it. Show the draft
with `vd_show`, and save it with `vd_save_brand` when they are happy: that puts it in their
library, which in the app is their account.

## Designing in a brand

`vd_apply_brand` puts a brand on a design: its roles become the design's tokens, the dark
values that differ go in a `theme dark` block, and its fonts, logo, checkable rules and banned
words come along under the brand line. Colours written out that are a role become the role.

```vd
version 5
kind design
title "Spring launch"
brand ember@3 id=6f1c2a9e-4b7d-4e0a-9c55-2f8e1d3b7a10 theme=light
  rule no-gradients must "No gradients" check=no-gradient
  banned "seamless"

$background = #FBF5EC
$text = #1A1410
$primary = #E8562A

theme dark
  $background = #1A1410
  $text = #FBF5EC

frame page x=0 y=0 w=1280 h=720 background=background
  text hero.title x=64 y=96 w=720 h=80 size=56 font=display fill=text
    "Cook every day"
```

- **Name roles, never colours:** `fill=primary`, `background=background`, `fill=text`,
  `stroke=border`. Switching the design to another brand then changes the tokens and nothing
  else.
- **Use the brand's fonts by role:** `font=display` for headings, `font=body` for the rest.
- **Never write the brand line.** `vd_apply_brand` writes it, switches it, and updates it.
- `theme=dark` on the brand line draws the design with its `theme dark` values.
- `strict` on the brand line refuses a colour written out by an agent; name the role it
  suggests instead.
- `vd_problems` says where a design breaks its brand: a colour written out
  (`literal-colour`, `off-brand-colour`), contrast, fonts, gradients, banned words.
