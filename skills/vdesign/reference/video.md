# Video

A video is one page played over time, exported as MP4 (social media, websites) or GIF (email,
docs). Write the page finished, then a `motion` block (`reference/motion.md`). Check it with
`vd_frames`, which shows the key moments and the computed timeline, before exporting.

## Sizes

Make the page — or the motion header's `w h` — the size of the platform:

| Where                          | Size      |
| ------------------------------ | --------- |
| Reels, TikTok, Shorts, Stories | 1080×1920 |
| Instagram and LinkedIn feed    | 1080×1350 |
| Square                         | 1080×1080 |
| YouTube, LinkedIn, X, websites | 1920×1080 |

For another size, make another page; nothing reflows by itself. Keep text clear of the top 250
and bottom 450 pixels of a 9:16 video, where the platform draws its own buttons.

## A storyboard: scenes side by side, one camera

The camera is what makes a video more than a slide. Put each scene in a frame on one wide page,
then move the camera between them. The page at rest is the storyboard, readable on the canvas.

```vd
frame promo x=0 y=0 w=3400 h=1350 background=ink
  frame hook x=0 y=0 w=1080 h=1350
  frame proof x=1160 y=0 w=1080 h=1350
  frame end x=2320 y=0 w=1080 h=1350

motion promo w=1080 h=1350
  camera hook duration=0
  hook.title enter=rise by=word
  camera proof delay=400
  proof.value enter=count together
  camera end delay=600
  end.logo enter=pop
```

## Recipes

- **Shape:** a hook in the first second, one idea per scene, an end card with the name and
  where to go. 6 to 15 seconds for an ad.
- **Kinetic headline:** `title enter=rise by=word stagger=80`, or `by=letter stagger=30` for a
  short word.
- **Counter:** a text whose content is the final number, `value enter=count from=0`.
- **Bars:** a group of rects, `bars enter=grow from=bottom stagger=120`.
- **Underline or logo stroke:** a stroked `vector`, `enter=draw duration=600`.
- **Lower third:** a group, `enter=wipe from=left hold=2500`.
- **Captions:** most social video plays muted. One text per line of speech, each
  `enter=fade at=… hold=…` long enough to read (about 3.5 words a second, at least 1.2 s).
- **End card:** its own scene; finish with `cta emphasis=pulse`.
- **Loop:** for a video that loops on a website, end on a shot that looks like the first.

## Cut to the beat

When the video has music and the person, or the track's own tag, gives its tempo, put `bpm=`
on the header and count in beats. Keep the lines relational as always, and let `on=` land them:
camera moves `on=bar`, entrances `on=beat`, staggers in half beats, and `length=` in bars, so the
video ends on a downbeat.

```vd
motion promo length=6bars w=1080 h=1350 bpm=120
  music bed fadeout=1500
  camera hook duration=0
  hook.title enter=rise by=word stagger=0.5beats
  hook.underline enter=draw duration=2beats on=beat
  camera proof duration=1beat on=bar
  proof.value enter=count from=0 duration=4beats together
  camera end duration=1beat on=bar
  end.cta enter=rise by=word stagger=0.5beats on=beat
```

`vd_frames` prints where each line falls (`@ 2bars`) and what `on=` moved, so check the cut
there. If the downbeat is not at the file's start, `from=` on the music line starts the track on
it. No tempo given: leave `bpm` out and time in milliseconds; a guessed tempo lands every cut
confidently off the beat.

## A bed under an ad

A short loop under a 9:16 ad: `loop` repeats it to fill the video, quieter so a voice or the
platform's own sound sits over it, with short fades either end.

```vd
motion ad length=3bars w=1080 h=1920 bpm=100
  music bed from=1200 volume=0.8 fadein=300 fadeout=300 loop
  ad.headline enter=rise by=word stagger=1beat
  ad.price enter=count from=0 duration=2beats after=ad.headline on=bar
  ad.cta enter=pop on=beat
```

The track is the person's: a URL they gave for `vd_add_asset`, or a slot they fill by dropping
the file on the canvas. A 128 kbps MP3 or M4A of the part the video uses, about 1 MB a minute,
keeps the file light.

## Don't

- Copy a page to make a second size or a second moment: each page is written once.
- Move more than one thing at a time on a phone-sized video.
- Hand-compute times with `at=` when `after=` or `with=` says it.

## Export

`vd_export` with `format: "mp4"` (where the app can encode) or `"gif"`, and the page as `scope`.
An MP4 carries the music (`audio: false` leaves it out); a GIF never has sound. A still with
`time: 3200` is the poster frame. On a website:
`<video src="promo.mp4" poster="poster.png" autoplay muted loop playsinline></video>` — a browser
autoplays only a muted video, so the music plays when the viewer unmutes or taps play.
