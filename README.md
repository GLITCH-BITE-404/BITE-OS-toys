# bite-os-toys

**The toy catalog for [BITE-OS](https://github.com/GLITCH-BITE-404/BITE-OS).**

`bite-toys` is a hub for the fun half of the system — browse it, install what you
want, tweak it, throw it away. This repo is where the downloadable toys live.

Toys bundled on the ISO work offline from `/usr/share/bite-os/toys`. Everything
else comes from here: `bite-toys update` reads `catalog.tsv`, and
`bite-toys fetch <toy>` pulls the matching tarball and checks it against its
published hash before unpacking a single byte.

```
catalog.tsv                    name <TAB> version <TAB> sha256 <TAB> description
toys/<name>-<version>.tar.gz
```

![the hub](docs/hub-commands.png)

---

## Install a toy

```sh
bite-toys update              # refresh the catalog from this repo
bite-toys fetch bitemask      # download it
bite-toys install bitemask    # put it on your PATH as a real command
bitemask                      # run it
```

Or just run `bite-toys` with no arguments and use the hub.

```
 ██████╗ ██╗████████╗███████╗       ██████╗ ███████╗
 ██████╔╝██║   ██║   █████╗        ██║   ██║███████╗
 ╚═════╝ ╚═╝   ╚═╝   ╚══════╝       ╚═════╝ ╚══════╝
 ░▒▓█▓▒░ // t o y s ░▒▓█▓▒░

  1 INSTALLED   2 GET MORE   3 CUSTOMIZE   4 PREVIEW   5 COMMANDS
```

| key | what it does |
|---|---|
| `1`–`5`, `Tab` | switch tabs |
| `← ↑ ↓ →` | move through the toys |
| `Enter` | run it · install it · open its settings |
| `a` | read what the highlighted toy actually does |
| `k` | show the toy's key list |
| `c` | jump to its settings |
| `x` `x` | uninstall (press twice — it asks first) |
| `u` | upgrade this toy |
| `q` | quit |

---

## The toys

### `bitecam` — your webcam, live, as ASCII

Real-time ASCII from your camera at whatever size the window is. ffmpeg captures
and scales, numpy does the pixel→character mapping, so it stays smooth
full-screen.

- `r` records straight to **mp4**, **gif**, or an asciinema **cast**
- `v` publishes it as a **real webcam** — Zoom, Discord and Meet can select it
  (run `bitecam --setup-camera` once first)
- `n` cycles character ramps, `c` flips truecolour/mono, `i` inverts, `m` unmirrors

![bitecam](docs/bitecam.png)

Settings include `charset`, `color`, `fps`, `mirror`, `invert`, `recformat`,
`recdir`, `reccell`. **Needs:** ffmpeg, python-numpy, python-pillow.

### `bitemask` — covers your face, and profiles it like you're being tracked

Finds your face in the webcam feed and refuses to show it. `b` cycles how it's
covered:

| mode | what it does |
|---|---|
| `bar` | the black REDACTED strip |
| `eyes` | anonymity strip across your eyes only |
| `visor` | a glowing band |
| `vanish` | learns the room behind you and paints it back over your head |
| `mosaic` / `blur` | pixelation / softening |
| `glitch` | tears the colour channels apart |
| `static` | punches a hole of noise |
| `ascii` | redraws the face as characters |

`vanish` is the strange one — you don't get censored so much as edited out. Give
it a second or two of empty frame to learn the background first.

Press `p` for the **profiler**: brackets, a sweeping scan line, and a dossier
that types itself out beside your head. `TAB` opens a settings panel you can
change anything from without leaving the toy; `w` saves it.

It works on **real pixels, not ASCII**, so `v` puts your censored face into a
call as an ordinary webcam, and `r` records to mp4 — set `recshape=vertical` and
it comes out 1080×1920, ready to post. The terminal preview is coarse by nature
(one glyph per character cell); press `f` for a real window at full resolution.

**Needs:** ffmpeg, python-opencv, opencv, python-numpy.

### `bitebeat` — whatever you're playing, with synced lyrics

Reads whatever is playing over MPRIS — mpv, Spotify, VLC, a browser tab — and
shows the track with its lyrics sweeping in time, big, karaoke style.

Looks for an `.lrc` next to the audio file first, then your lyrics folder, then
lrclib.net if online lookup is on. Big text is rasterised at runtime, so it
renders Hebrew and any other script just as happily as English.

![bitebeat](docs/bitebeat.png)

`space` play/pause, `←/→` seek, `[` `]` nudge sync, `n`/`p` track, `w` toggles
the audio reaction. The key list sits along the bottom; `?` hides it.
**Needs:** playerctl, cava, python-numpy, python-pillow.

> mpv publishes no MPRIS without the `mpv-mpris` package.

### `bitemuseum` — your machine's history as an exhibition

Turns your own disk into a museum. Opens on a title card, then walks through
chapters: the day the machine was installed, the oldest files you still own,
your earliest photos shown for real, the first sites you ever visited, the
heaviest things you're carrying, and what you've forgotten.

`T` starts a guided tour that advances on its own every 7 seconds so you can
film it hands-free. `Enter` opens the real file, `F` opens its folder.

![bitemuseum](docs/bitemuseum.png)

Strictly read-only. Browser history stays hidden until you press `B`.
**Needs:** qt6-declarative, python.

---

## Trying settings before you keep them

The **PREVIEW** tab renders each toy from a built-in sample scene, so nothing
runs until you press Enter and no camera or music player is ever opened for it.

![the preview tab](docs/hub-preview.png)

Press `c` and the picture holds still while you change settings; `Enter` runs it
again with the new values so you can see the difference, and `w` keeps them.
Anything you have not kept is marked `●` and is discarded when you back out —
your config file is not touched until you say so.

## Settings

Every toy's settings live in `~/.config/bite-os/toys/<toy>.conf` and are handed
to it as `TOY_<KEY>` environment variables — a toy never parses configuration
itself.

```sh
bite-toys config bitemask                 # list them
bite-toys config bitemask cover           # read one
bite-toys config bitemask cover vanish    # change one
bite-toys reset bitemask                  # back to defaults
```

Values are checked against the toy's declared `@choices` and `@range`, so a typo
is rejected instead of being silently ignored at runtime.

---

## Security

`catalog.tsv` is the trust anchor. **A toy is arbitrary code**, so `fetch` checks
every tarball against its published sha256 and refuses to unpack one that does
not match. A toy with no published checksum installs with a visible warning.

`bite-toys remove` never deletes your only copy of a toy — it takes it off your
PATH and tells you where the files are. Deleting them is a separate, explicit
`bite-toys purge`, which asks first.

---

## Publishing a toy

Don't build tarballs or checksums by hand — the point of the packer is that the
catalog and the tarballs can't drift apart:

```sh
bite-toys new mytoy          # scaffold one that already works
$EDITOR ~/.local/share/bite-os/toys/mytoy/run
bite-toys install mytoy      # try it
bite-toys pack --out .       # rebuild every tarball + catalog.tsv
```

`pack` is reproducible — the same toy always produces the same bytes and the
same checksum, so rebuilding doesn't churn the catalog.

### A toy is just a directory

| file | what it is |
|---|---|
| `toy.meta` | `name`, `version`, `desc`, `category`, `exec`, `deps`, plus `usage`, `keys`, `about`, `preview`, `input` |
| `run` | the executable (or whatever `exec=` names) |
| `config.def` | defaults; `#` lines above a key are its help text |

In `config.def`, `# @choices a,b,c` and `# @range 0-10` above a key let the hub
cycle values with the arrows.

`deps=` entries are `cmd`, `cmd:package`, or `py:module:package` — commands are
what the toy needs at runtime, packages are what pacman has to install, and they
are often not the same word. Missing dependencies are offered as a `pacman -S`
on first run rather than crashing.

---

## License

GPL-3.0 — see [LICENSE](LICENSE).

BITE-OS, its name and branding are the original work of **GLITCH-BITE-404**
(Raphael Virnik). Forks must keep the credit and stay GPL-3.0.

`// THE SYSTEM BIT YOU`
