# Motion

Motion is how a page moves in time, never what the page is. Write every page as it looks when
finished; motion says how it gets there. Three things read it: Play presents it, the canvas
timeline scrubs it, and a page exports as a video (MP4 or GIF). The canvas at rest and every
still export show the page as written.

Add motion when the person is making something to present, to click through, or to post as a
video. For a deck keep it light: a transition per page and a reveal or two. For a video see
`reference/video.md` first.

## Where it goes

After the last page, one block per page that has any:

```vd
motion intro enter=fade
  intro.title enter=rise
  intro.chart enter=fade step
  intro.note enter=fade together
  intro.cta enter=pop go=#pricing

motion pricing enter=push duration=600
  pricing.back go=back
```

A header names a page — a top-level frame. Each line under it names a node on that page and
says what the node does, or is a `camera` line. Never copy a page to animate between the
copies: each page is written once.

## The header

`motion <page> [enter=…] [duration=…] [scroll] [length=…] [w=… h=…] [bpm=… meter=…]`

| `enter` | Play arrives on the page by…                     |
| ------- | ------------------------------------------------ |
| `cut`   | nothing at all — the default                     |
| `fade`  | fading in over the page before                   |
| `push`  | pushing the page before out, sideways            |
| `slide` | sliding in over the page before, which stays put |

You never write a direction: going back runs the leaving page's `enter` in reverse.

- `scroll` — Play fits a page taller than a screen to the window's width and scrolls it. Give
  it to a web page or a long article; leave it off slides, posters and phone screens.
- `length=8000` — how long the page plays as a video. Leave it out: the runtime takes the end
  of the last line plus 1.5 s.
- `w` and `h` (`w=1080 h=1920`) — the video's size, when it is not the page's. A wide page holding several
  scenes side by side plays at this size while `camera` lines move between them.
- `bpm` and `meter` (`bpm=120 meter=4`) — the page's tempo, so lines can be timed in beats and
  bars (see Beats, below). `meter` is beats in a bar, 4 unless said.

## A line

`<node> [enter=…|exit=…|emphasis=…] [from=…] [by=…] [stagger=…] [duration=…] [hold=…] [delay=…] [at=…|after=…|with=…|together|step] [on=…] [ease=…] [go=…]`

| effect  | entering                                                                                    | leaving             |
| ------- | ------------------------------------------------------------------------------------------- | ------------------- |
| `cut`   | appears at once                                                                             | vanishes at once    |
| `fade`  | fades in                                                                                    | fades out           |
| `rise`  | rises from a little below                                                                   | rises away          |
| `drop`  | drops from a little above                                                                   | drops away          |
| `pop`   | grows in from slightly smaller                                                              | shrinks away        |
| `wipe`  | revealed behind an edge, `from=left` (or `right top bottom`)                                | hidden the same way |
| `grow`  | scales up from one side, `from=bottom` (or `top left right`): bars, lines                   | shrinks back        |
| `count` | a text's number runs from `from=0` up to its written value (`"12,480"`, `"$1.2M"`, `"98%"`) | —                   |
| `draw`  | a stroked `vector` draws along its path; its fill fades in at the end                       | —                   |

The sides `from=` names are `left`, `right`, `top` and `bottom`.

`emphasis=pulse` stresses a node that is already there — `pulse` is the one stress there is: it swells a little and settles. It goes
on its own line: `cta emphasis=pulse at=6000`.

A node may have one `enter`, one `exit` and one `emphasis` line — never two on one line.

**Pieces.** `by=` takes `word` or `letter`, and splits a text so its pieces enter one after another,
`stagger=80` ms apart (letters default to 30, words to 80). `stagger=` on a group or frame
without `by` staggers its children. A line with pieces finishes when its last piece does.

**Holding.** `hold=1800` on an `enter` line stays that long, then leaves the way it came — the
cheapest caption: `cap.1 enter=fade hold=1800`.

**Easing.** `ease=` takes `out` (entrances, the default), `in` (exits), `inout` (camera and emphasis),
`linear`, or `spring`, which overshoots and settles.

## When a line starts

Lines run in the order written, each starting when the line above has finished — including
anything it holds — unless it says one of:

- `together` — with the line above;
- `with=title` — with the latest line above about `title`;
- `after=title` — when the latest line above about `title` has finished;
- `at=6000` — that many milliseconds into the page. Use it for captions timed to a voice, not
  for everything: relations keep working when a headline gains a word;
- `step` — in Play, waits for the presenter to press → or click. A video does not wait.

`delay=300` waits that long after the line would otherwise start. A line may only name lines
above it.

**Time is milliseconds, as a plain number**: `duration=600`, never `600ms` or `0.6s` — or, on a
page with `bpm=`, beats and bars: `duration=2beats`, `at=4bars`. A duration defaults to 400 (a
`count` to 1200, a `draw` to 800, a camera move to 800) and may be at most 10000 ms; `at` and
`length` may be up to 180000.

## Beats

With `bpm=` on the header, every time on the page — `at`, `delay`, `duration`, `stagger`,
`hold` and the header's `length` — may be counted in `beat`s or `bar`s instead of milliseconds,
and the runtime works out the milliseconds. Never multiply a tempo out yourself.

- Spell it with the unit attached, singular only for exactly one: `1beat`, `0.5beats`, `2beats`,
  `1bar`, `6bars`.
- Counted from the start, like milliseconds: `at=3bars` is three bars in, the fourth bar's
  downbeat, and `at=0bars` is the very start.
- `on=beat` or `on=bar` moves a line's start up to the next beat or bar, after everything else
  (`after=`, `delay=`) has been worked out. A line already on the grid stays. It keeps the
  chain relational and still lands on the beat: put `on=bar` on camera moves and `on=beat` on
  entrances. It cannot go with `together` or `with=`.

```vd
motion promo length=6bars w=1080 h=1350 bpm=120
  camera hook duration=0
  hook.title enter=rise by=word stagger=0.5beats
  hook.underline enter=draw duration=2beats on=beat
  camera proof duration=1beat on=bar
```

At 120 bpm a beat is 500 ms and a bar of four is 2000 ms. The tempo comes from the person or
from the track's own tag; never guess one. Without it, time the page in milliseconds.

## Music

A video page may have one track under it: a `music` line, anywhere under the header (it prints
first), naming an audio asset.

`music <asset> [from=…] [volume=…] [fadein=…] [fadeout=…] [loop]`

```vd
motion ad length=3bars w=1080 h=1920 bpm=100
  music bed from=1200 volume=0.8 fadein=300 fadeout=1beat loop
  ad.headline enter=rise by=word stagger=1beat
```

- The track starts with the video, and the video's end cuts it.
- `from=1200` — where in the file to begin, always milliseconds. Set it to a downbeat and the
  grid lines up with the track.
- `volume=0.8` — 0 to 1, 1 unless said.
- `fadein=` and `fadeout=` — times, so beats on a page with `bpm=`. With no `fadeout`, a track
  the video cuts short fades over the last second; one that ends first just ends.
- `loop` — repeats from `from=` to fill the video.

The asset is a sound file added with `vd_add_asset` from a URL the person gave, never one you
found, since music is someone's work. A track the person has but you cannot send is a slot,
as a picture is: write `music bed` anyway, with no asset line, and tell them to drop the file
on the canvas. The video is silent until they do (`music-missing`).

Keep tracks light: a 128 kbps MP3 or M4A of the part the video uses, about 1 MB a minute. A GIF
has no sound.

## The camera

`camera <target> [pad=…] [duration=…] [delay=…] [at=…|after=…|with=…|together] [ease=…]`

The shot moves to the target — a node on the page, or the page itself to pull back — fitted to
the video's size. A frame fills the shot exactly; anything else gets a little room, or `pad=`.
The first camera line sets the opening shot (`camera hook duration=0` cuts straight to it).
Between moves the shot holds. A page with no camera line plays whole.

```vd
motion promo w=1080 h=1350
  camera promo.hook duration=0
  promo.title enter=rise by=word
  camera promo.proof
  promo.value enter=count together
```

## Buttons: `go=`

Any line can say where a click on its node goes in Play. A click anywhere inside the node
counts, so put `go=` on the group or frame that _is_ the button, not on its label.

- `go=#pricing` — that page. A frame inside a page goes to its page.
- `go=back` — wherever the presenter came from.
- `go=next` — the next page, whatever is left to reveal on this one.
- `go=https://…` or `go=mailto:…` — only a link the person gave you.

A `go=` works only in Play. A link that should also work in an exported PDF is a text run's
`link=` (see nodes).

## Editing motion

A patch replaces a page's whole block: write `motion <page>`, its header, and every line as it
should end up — resend the whole block to change one timing, its `bpm=` and its `music` line
included, or they are gone. A bare `motion <page>` clears it.

```vd-patch
motion intro enter=fade
  intro.title enter=rise
  intro.chart enter=fade step
```

Deleting a node removes its lines; an `after=` or `with=` naming it falls back to "after the
line above". The `vd_read` outline ends with the pages that have motion.

## What is checked

An **error** refuses the write: an effect or key that does not exist, a unit on a time (other
than beats and bars on a page with `bpm=`), a beat time or `on=` with no `bpm=`, a `bpm` outside
20–300 or a `meter` outside 1–16, `on=` with `together` or `with=`, a second `music` line or a
key it does not take (`fade`, `start`, `gain`), `from=` in beats, two
effects or two start rules on one line, a `from=` that does not fit its effect, `by=` with
anything but `cut fade rise drop pop`, `hold=` without `enter`, `count` as an exit, a camera
line with an effect, a line that says nothing, or a second `enter`, `exit`, `emphasis` or `go`
for one node. In a patch, naming a node that is not on the page is an error too.

A **warning** keeps the line: a node that is not there or is on another page, a header that is
not a page, a `go=#` to no frame, an `after=`/`with=` naming no line above, a `count` on text
with no number, a `draw` on nothing stroked, `by=` on a node that is not text or is rotated,
`stagger=` on something with no children, a line that starts after `length`, and text on
screen too briefly to read (`video-text-brief`). For music: no file yet (`music-missing`), a file
that is not sound (`music-not-audio`), a track that ends before the video without `loop`
(`music-short`), `from=` past its end (`music-from-past-end`), and a track far heavier than the
video needs (`audio-heavy`).
