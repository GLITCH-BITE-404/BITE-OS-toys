# How it works

The hub is one bash script and the toys are directories. There is no daemon, no
database and no plugin API — which is the point: you can read the whole thing
and write a toy without learning a framework.

---

## Two catalogs, one view

```
/usr/share/bite-os/toys/<name>/          bundled — ships on the ISO, works offline
~/.local/share/bite-os/toys/<name>/      fetched — pulled from this repo
```

A fetched copy **shadows** a bundled one, so a newer version can be published
without touching `/usr`. `bite-toys upgrade` replaces the files and leaves your
shim and your settings alone.

---

## A toy is a directory

```
mytoy/
├── toy.meta      what it is
├── run           the executable
└── config.def    its settings and their defaults
```

### `toy.meta`

```ini
name=mytoy
version=1.0
desc=One line, shown in every list
category=visual
author=you
exec=run
deps=ffmpeg py:numpy:python-numpy
preview=--preview
usage=mytoy   ·   q quits
keys=space:play|?:hide the key list|q:quit
about=The long version, shown when someone presses `a` on it in the hub.
```

`deps=` entries take three forms, because **the command a toy needs and the
package that provides it are often not the same word**:

| form | means |
|---|---|
| `ffmpeg` | a command whose package shares its name |
| `qml6:qt6-declarative` | command `qml6`, lives in package `qt6-declarative` |
| `py:numpy:python-numpy` | importable Python module `numpy`, package `python-numpy` |

Guessing the package from the command name sends people to a package that does
not exist, so the mapping is written down instead.

### `config.def`

```ini
# Comment lines directly above a key are its help text.

# Which colour to use
# @choices green,cyan,magenta
accent=green

# Redraw rate
# @range 5-60
fps=30
```

`@choices` and `@range` let the hub cycle a value with ← →, and they are
**enforced** — `bite-toys config` rejects anything outside them rather than
letting the toy silently clamp it, so what the hub shows and what runs are the
same thing.

`on/off`, `yes/no` and `true/false` defaults are treated as toggles without
needing an explicit `@choices`.

### `run`

Settings arrive as **`TOY_<KEY>` environment variables**. A toy never parses
configuration:

```bash
ACCENT="${TOY_ACCENT:-green}"
FPS="${TOY_FPS:-30}"
```

Keep the fallbacks — they are what lets the toy still work when run directly,
outside the hub.

Scaffold one that already does all of this:

```sh
bite-toys new mytoy
```

---

## Installing

Installing symlinks a shim into `~/.local/bin/<name>` pointing back at
`bite-toys`, which re-dispatches on `$0`. So the toy becomes a plain command,
and because the shim re-enters the hub, **settings are read at run time, not
baked in at install time** — change one and the next run picks it up.

The hub refuses to overwrite a file in `~/.local/bin` that is not its own
symlink, so it cannot silently hijack a command you already had.

---

## Removing vs purging

`remove` takes a toy off your `PATH`. It only deletes files when a **bundled
copy exists to fall back on** — with no bundled catalog, the fetched directory
is the only copy on the machine, and removing it would be data loss rather than
an uninstall.

`purge` is the explicit destructive one. It asks first, and refuses to delete
your only copy non-interactively without `--force`.

In the hub, `x` arms and a second `x` fires. Any other key cancels.

---

## The catalog

```
catalog.tsv     name <TAB> version <TAB> sha256 <TAB> description
toys/<name>-<version>.tar.gz
```

`bite-toys update` fetches `catalog.tsv`. `bite-toys fetch <toy>` downloads the
tarball, **checks it against the published sha256, and refuses to unpack one
that does not match**. A toy is arbitrary code, so the catalog is the trust
anchor — an unverified tarball never reaches your disk.

Tarballs contain exactly one top-level directory (`fetch` unpacks with
`--strip-components=1`).

`bite-toys pack` builds both halves together, so the checksums cannot drift from
the files they describe. It is reproducible: same toy in, same bytes out, same
hash — a rebuild does not churn the catalog.

Point the hub at your own fork by setting `REPO` in
`~/.config/bite-os/hub.conf`.

---

## Previews

A preview renders a **built-in sample scene**, never live hardware. Each toy
supports a `--preview` flag declared as `preview=` in its meta, and the hub
captures its stdout.

This is deliberate. Previews used to grab the real camera, which meant they
fought a running toy for `/dev/video0` (the freeze), cost seconds per redraw,
and made two settings impossible to compare because the picture had moved
underneath you. A fixed scene is instant, repeatable, and works on a machine
with no webcam at all.

In the PREVIEW tab nothing runs until you press Enter. Press `c` and the picture
holds still while you change settings — the toy is not re-run behind you.
Changes are held as `BITE_OVR_<KEY>` overrides passed to the preview
subprocess, so **nothing touches your config file until you press `w`**.
Anything unsaved is marked `●` and discarded when you back out.

A toy that cannot be sampled into a text pane says so via `nopreview=` in its
meta, rather than showing a generic excuse.

---

## Why bash

The hub redraws its entire frame on every keypress, so anything per-row must not
fork a process. An earlier version re-parsed `config.def` with `sed` and `awk`
once per setting per frame — about 96 processes and 700 ms per keystroke on a
12-setting toy, which read as the hub freezing. Settings are now parsed once
into arrays and the layout helpers are pure bash. Same work, ~60 ms.

The frame is assembled into a single string and written in one `printf`, so the
screen never tears mid-redraw. Nothing repaints on a timer — a redraw only
happens in response to a keypress.
