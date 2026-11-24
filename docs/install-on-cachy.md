# Install Flea locally on CachyOS (niri + Noctalia)

This guide builds and installs the local checkout as an Arch package. It does not use the AUR. Flea's GUI is the GPUI frontend; Quickshell is not a runtime dependency of that launch path. Noctalia is optional to Flea itself and is only needed for the Noctalia-generated theme files.

## Before building

Make sure the checkout contains the changes you want to test. The build archive below is made from `HEAD`, so commit those changes first:

```bash
cd /path/to/flea
git status
git add -A
git commit -m "Prepare local Flea build"
```

Use a separate build directory: the source tree itself contains a `src/` directory, which makepkg also uses for unpacked sources.

```bash
mkdir -p ../flea-build
cp PKGBUILD ../flea-build/
pkgver=$(. ./PKGBUILD && echo "$pkgver")
git archive --prefix="flea-$pkgver/" -o "../flea-build/flea-v$pkgver.tar.gz" HEAD
```

The local `PKGBUILD` has a placeholder checksum, so skip checksum verification for this locally generated archive. It does not skip the package's configured build checks.

## Build and install

```bash
cd ../flea-build
makepkg -fCsi --skipchecksums
```

`-f` forces a rebuild, `-C` clears the previous unpacked build sources, `-s` asks pacman to install missing dependencies, and `-i` installs the resulting package. Review pacman's transaction before confirming. Keep the recipe's declared runtime dependencies: they support Flea features such as GVFS remotes, media playback, HTML previews, and terminal launching. Quickshell should not be included as a dependency for this GPUI launch path.

## CachyOS mirror or 404 errors

If pacman reports a 404 for a package file (for example a CachyOS `luajit` package), the package database and selected mirror may be out of sync. Do not install selected packages with a stale database or perform a partial upgrade. First update the system as a whole:

```bash
sudo pacman -Syu
```

If that also returns 404s, refresh the CachyOS mirror list with the mirror-management tool supplied by your CachyOS installation, then retry the full upgrade. After it succeeds, rerun `makepkg -fCsi --skipchecksums` from `../flea-build`.

## First launch and theme

Launch the GUI and TUI to check the basic install:

```bash
flea
flea --tui
```

### Optional desktop integration

These commands configure **per-user** desktop preferences; you can choose either or both:

```bash
flea --default       # folder handler, “Show in folder” and system file chooser
flea --picker        # system file chooser only
```

Undo them with `flea --default off` and `flea --picker off`, respectively. Run the command in your
user session, not with `sudo`: the preferences belong to your home directory. `--default` includes
the picker integration; use `--picker` alone if you want to keep another file manager as the folder
handler.

The system chooser integration uses **xdg-desktop-portal**. It affects apps that request a chooser
through the portal, not applications that create their own GTK/Qt dialog. After enabling it, test
Open and Save from an app known to use the portal. If the old chooser is still shown, check that the
portal service is available and running in your user session, then restart it and retry:

```bash
systemctl --user status xdg-desktop-portal.service
systemctl --user restart xdg-desktop-portal.service
```

Noctalia theming is optional. To follow its palette, configure the Flea user template; the template
writes theme data under `$XDG_STATE_HOME/flea/theme` (normally `~/.local/state/flea/theme`). The
current template setup is recorded in the local porting notes `docs/PIANO-CACHYOS-NIRI-NOCTALIA.md`.

### Verify and remove

Check the folder association with:

```bash
xdg-mime query default inode/directory
```

To remove Flea cleanly, undo whichever integrations you enabled before removing the package:

```bash
flea --default off
flea --picker off
sudo pacman -Rns flea
```

If you enabled only one integration, only its matching `off` command is needed.

## Rebuild and remove

For another test build, commit the desired changes and recreate the archive using the commands above, then run `makepkg -fCsi --skipchecksums` again. Pacman tracks the installed files; do not copy binaries into `/usr` manually.

To remove Flea, first undo any per-user default-file-manager setting, then remove the package:

```bash
flea --default off
sudo pacman -Rns flea
```
