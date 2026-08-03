# Compatibility

What actually runs where. Split into **verified** (tested on real hardware) and
**expected** (should work, nobody has confirmed it) — because a support matrix
full of untested claims is worse than a short honest one.

---

## Verified

| | |
|---|---|
| **OS** | BITE-OS (`ID_LIKE="cachyos arch"`), kernel 7.1.5-cachyos |
| **Shell** | GNU bash 5.3 |
| **Python** | 3.14 |
| **Terminal** | [foot](https://codeberg.org/dnkl/foot) |
| **Session** | Hyprland / Wayland |

Everything below is reasoning from what the code needs, not from a test run.

---

## The hub (`bite-toys`)

**Needs bash 4.0+.** It uses associative arrays (`declare -A`) for the settings
cache and the preview overrides. bash 3.2 — which is what macOS ships — will not
run it.

**Dependency auto-install is Arch-only.** When a toy is missing something, the
hub offers to `sudo pacman -S` it. On any other distro it prints the package
names and stops, so you install them yourself and everything still works. Only
that convenience is Arch-specific — nothing else is.

| distro family | hub runs | auto-installs deps |
|---|---|---|
| Arch, CachyOS, EndeavourOS, Manjaro | yes | yes |
| Debian, Ubuntu, Fedora, openSUSE | yes | no — install by hand |
| Alpine (busybox) | needs bash installed | no |
| macOS | needs a newer bash than the system one | no |

`~/.local/bin` has to be on your `PATH` for installed toys to work as commands.
`bite-toys doctor` checks that and tells you how to fix it.

---

## Terminals

The hub itself is plain text and works anywhere. **The toys are the demanding
part**, and two things matter: colour depth and font coverage.

### Truecolour

`bitecam` and `bitemask` emit 24-bit colour (`\e[38;2;R;G;Bm`). A 256-colour
terminal shows the picture, but banded and wrong.

| terminal | truecolour | notes |
|---|---|---|
| foot | yes | what this is developed on |
| kitty, WezTerm, Alacritty, Ghostty | yes | |
| GNOME Terminal, Konsole | yes | |
| xterm | depends on build | often 256-colour only |
| Linux console (TTY) | no | `bitecam` is readable with `color=off` |

`bitecam` has a `color` setting — turn it off and mono ASCII works on anything.

### Font coverage

`bitemask` draws video with **quadrant block glyphs** (`▘▝▖▗▚▞█` …, U+2596–259F)
to get two horizontal sub-pixels per cell. `bitebeat` uses half blocks
(`▀▄█`, U+2580/2584/2588) for its big text.

If your font lacks quadrants you get gaps or boxes. Set `preview=half` in
bitemask to fall back to half blocks, which almost every monospace font has.

Fonts confirmed to carry the quadrants: **JetBrains Mono Nerd Font**, DejaVu
Sans Mono, Noto Sans Mono.

> Sextants (U+1FB00–) and octants would be sharper still, but almost no font
> ships them — only about 17 on a full Arch install. Quadrants are the practical
> ceiling for terminal video.

### Sharpness

A terminal can only draw one glyph per character cell, so the preview is coarse
however big the window is. That is a property of terminals, not a bug.
`bitemask` has `f` — it opens a real window via `ffplay` at the full working
resolution. Use that when you want detail.

---

## Per-toy requirements

### `bitecam`
`ffmpeg`, `python-numpy`, `python-pillow` (pillow only for recording).
A V4L2 camera at `/dev/video0` — set `device` for anything else.
`v` needs `v4l2loopback-dkms`; run `bitecam --setup-camera` once, which writes
`exclusive_caps=1`. Without that flag Chrome and Zoom will not list the device.

### `bitemask`
`ffmpeg`, `python-numpy`, `python-opencv`, **and `opencv`** — the haar cascade
XML files live in the `opencv` package, not in the Python bindings, so the
bindings alone are not enough.

> OpenCV pulls in vtk, openmpi and hdf5 — around 550 MB. That is why bitemask
> is not preinstalled on the ISO; the hub offers the install on first run.

`f` needs `ffplay` (ships with ffmpeg).

### `bitebeat`
`playerctl`, `cava`, `python-numpy`, `python-pillow`.

Any MPRIS-capable player: Spotify, VLC, a browser tab, mpv.

> **mpv publishes no MPRIS without the `mpv-mpris` package.** Install it or
> playerctl will not see mpv at all.

Online lyric lookup contacts `lrclib.net` with the track and artist name. Set
`online=off` to stay fully offline; local `.lrc` files still work.

### `bitemuseum`
`qt6-declarative` (for `qml6`) and `python`.

> `qml` on most systems is **Qt5**. This needs `qml6`. They are different
> binaries and the Qt5 one will not run it.

Fullscreen Qt window — needs a graphical session, so it will not run in a TTY.
Strictly read-only: it reads your files to build the exhibition and never
writes to them.

---

## Wayland and X11

Everything is terminal-based except two windows: bitemuseum's Qt exhibition and
bitemask's `ffplay` window. Both work under Wayland and X11.

---

## If something does not work

```sh
bite-toys doctor        # PATH problems, broken toys, missing catalog
bite-toys info <toy>    # exactly what it needs and whether you have it
```

`info` checks each dependency individually and tells you the package name to
install — commands and packages are often not the same word (`qml6` lives in
`qt6-declarative`), so guessing from the command name sends you to a package
that does not exist.
