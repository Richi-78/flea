# Porting Flea → CachyOS + niri + Noctalia

Piano di riferimento: `docs/PIANO-CACHYOS-NIRI-NOCTALIA.md`.
Target: **niri only**, tema **Noctalia**, packaging locale `makepkg -si`.
Nessuna dipendenza o test che richieda Hyprland, hyprshell, `omarchy-*`,
`/usr/share/omarchy` o una sessione Omarchy.

## Stato

| Fase | Contenuto | Stato |
|---|---|---|
| 0 | Lettura piano, ricognizione fork, decisioni | ✅ fatto |
| 1 | Shim QML vendorizzato, attribuzione e installazione | ✅ implementato (smoke QML da verificare in sessione) |
| 2 | Tema Noctalia: path, marker, `Theme.qml`, `shell.toml` | ✅ implementato; prova live già confermata dall’utente |
| 3 | Runtime niri: scale, hyprkeys, update, TUI, TabDrag, testi | 🚧 parziale; vedi checklist sotto |
| 4 | Packaging: PKGBUILD locale, Makefile, dipendenze | 🚧 parziale; makefile installa i moduli vendorizzati |
| 5 | Test: smoke suite CachyOS/niri, JS stabili, cargo | 🚧 parziale |
| 6 | Rifinitura: icon theme, documentazione, verifica manuale | ⏳ da fare |

## Fase 1 — Shim QML

- [x] File Omarchy `quattro` scaricati e verificati (9 file + 2 qmldir)
- [x] `ui/vendor/omarchy/` creato con licenza e nota di attribuzione
- [x] I quattro path QML puntano ai moduli vendorizzati (symlink relativi, nessun path `/usr/share/omarchy`)
- [ ] `Style.qml` patchato per niri (corner radius 10, niente Hyprland)
- [ ] `Color.qml` patchato (path Flea, `urgent` esplicito)
- [x] `Makefile` installa il layer vendorizzato; PKGBUILD locale non ancora validato end-to-end

## Fase 2 — Tema Noctalia

- [ ] `ui/Theme.qml` legge `$XDG_STATE_HOME/flea` (override `FLEA_THEME_DIR`)
- [x] Marker `theme.name` e `shell.toml` creati al primo avvio di GPUI/chooser (`src/gui.rs`, rispettando `FLEA_THEME_DIR`)
- [ ] `niri msg --json focused-output` sostituisce `hyprctl monitors`
- [ ] Rimosso il query animazioni Hyprland; resta `FLEA_REDUCED_MOTION`
- [x] Template Noctalia utente creato in `~/.config/noctalia/templates/flea-colors.toml`, registrato in `~/.config/noctalia/templates.toml` e applicato con `noctalia msg templates-apply` (palette custom `desktop-ahrntal-001-sft`, mode attuale light)
- [x] Verifica manuale utente: tema applicato e switch light/dark confermato
- [x] Sorgente tema GPUI (`crates/flea-gui/src/presentation/source.rs`, `palette.rs`) legge `FLEA_THEME_DIR/theme/colors.toml` o `$XDG_STATE_HOME/flea/theme/colors.toml`; override shell in `$XDG_CONFIG_HOME/flea/shell.toml`
- [x] Percorso tema TUI allineato a `$FLEA_THEME_DIR` / `$XDG_STATE_HOME/flea`

## Fase 3 — Runtime niri

- [x] `src/hyprkeys.rs`: `NIRI_SOCKET` attiva il no-op; claim/release e picker restituiscono istruzioni KDL senza leggere o scrivere configurazione Hyprland
- [ ] `src/update.rs`: messaggio pacchetto locale
- [x] `src/tui/theme.rs`: legge `$FLEA_THEME_DIR/theme/colors.toml` o `$XDG_STATE_HOME/flea/theme/colors.toml` (fallback `~/.local/state`)

- [ ] `ui/TabDragGeometry.qml`: tear-off disabilitato sotto niri
- [ ] `ui/NetworkDialog.qml`: niente `omarchy-launch-terminal`
- [~] Etichette Display aggiornate: Follow System, scala niri, Rounded icons; restano testi About e altre etichette legacy
- [x] Icon theme lookup for Open With icons: GTK 4 `settings.ini` → GTK 3 → gsettings → hicolor; Noctalia writes Breeze in GTK 4 settings. File/folder listing glyphs remain Flea vectors by choice.

## Fase 4 — Packaging

- [ ] `PKGBUILD` senza `omarchy`, con `quickshell>=0.3.1`
- [ ] `package()` installa `ui/vendor/omarchy` in `ui/Commons` e `ui/Ui`
- [ ] `check()` usa `tests/cachyos-niri/check.sh`

## Fase 5 — Test

- [ ] `tests/cachyos-niri/smoke.sh`
- [ ] `tests/cachyos-niri/check.sh`
- [ ] `cargo test --release` verde
- [ ] Suite JS stabili verdi (esclusa `themes`)

## Fase 6 — Rifinitura

- [x] Icon theme fallback for Open With app icons (GTK 4/3, gsettings, hicolor).
- [ ] Snippet keybind e window rule niri documentati
- [ ] Verifica manuale dark/light e cambio palette
- [ ] Note di fork aggiornate

## Prova manuale GPUI (2026-11-21)

- [x] Build release workspace e avvio sotto niri: riusciti secondo verifica manuale.
- [x] Velocità iniziale e operazioni avanzate (conversione, copia, taglia, duplica) giudicate ottime.
- [ ] Follow-up aggiornati da `docs/Testing.md`:
  - [ ] Tag rimossi restano nella lista TAGS fino al riavvio; alcune tinte tag sono errate (white squares risolto).
  - [ ] Taildrop/LocalSend abilitazione; lingua selezionabile; icona empty-state Omarchy; font e icone Omarchy. About: tagline niri aggiornata.
  - [~] Lingua: rimosso il vincolo Omarchy dal runtime i18n: override `FLEA_LANGUAGE_FILE`, configurazione predefinita `$XDG_CONFIG_HOME/flea/language`, roster locale incluso come fallback e lingua iniziale derivata da `LC_ALL`/`LC_MESSAGES`/`LANG` (con fallback al sottotag primario e poi inglese). Il reload watcher già esistente applica le modifiche live quando il file cambia. About > Language usa ora un controllo di scelta GPUI alimentato dal roster lingue; click/step emette il codice selezionato, scritto atomicamente in `$XDG_CONFIG_HOME/flea/language` e applicato live dal watcher. TAGS esplicitamente sospesi su richiesta utente.
  - Tema non applicato e badge bianchi sono risolti; cambio light/dark verificato.

## Lavoro recente (2026-11-21)

- [x] Impostazioni Display: “Follow System”, scala niri e “Rounded icons” (chiave persistita `display.hyprlandIcons` conservata per compatibilità).
- [x] Annunci dimensione testo aggiornati per dire “the desktop”.
- [x] About GPUI non si presenta più come file manager Omarchy: tagline “A file manager for niri” aggiornata in catalogo inglese e string pool; `cargo test -p flea-gui settings_pages::board_tests --lib` (7 test) verde.
- [x] `./tests/js.sh textsize` (92 controlli) e `./tests/js.sh settings` (686 controlli) verdi.
- [x] Corretto il test TUI rimasto ancorato a `.local/state/omarchy/current/theme/colors.toml`; ora usa `.local/state/flea/theme/colors.toml`.
- [!] `cargo test --release --locked`: 4965 passati, 6 falliti. Due test richiedono `7z` (assente), un test bsdtar fallisce nell’ambiente, e la vecchia asserzione sul path TUI è stata corretta dopo il run.
- [x] Test i18n: `cargo test --lib i18n::tests:: -- --skip contracts` — 12 passati; verificati roster bundled, preferenza lingua salvata e reload file.
- [x] Rimpiazzata l'azione Omarchy/editor in About > Language con il selettore nel pannello GPUI, popolato dal roster; la scelta è persistita con rename atomico. `cargo test -p flea-gui --lib window::settings::` — 88 passati.

## Lavoro recente (2026-11-22)

- [x] Windows-like view shortcuts now match Mac-like: Ctrl+1/2/3/4 for List/Columns/Grid/Dual; regenerated `keys.toml` outputs and updated shortcut-help coverage. `tests/keymap-gen.sh`, `./tests/js.sh keymap` (823 checks) and GPUI Dual shortcut test pass.
- [x] Noctalia `shell.lang` is read from `~/.config/noctalia/config.toml` and `~/.local/state/noctalia/settings.toml` when no explicit Flea language is set; both files are watched. Font presentation reads Noctalia config/settings as sources and resolves `shell.font_family`; Flea font-size config remains supported. Font/source tests and i18n tests pass.
- [x] `flea --picker` portal backend was temporarily registered user-locally for real browser/app testing. D-Bus backend activation and FileChooser methods verified; browser upload test confirmed working. Undo with `target/release/flea --picker off`, then remove the temporary files `~/.local/share/xdg-desktop-portal/portals/flea.portal` and `~/.local/share/dbus-1/services/org.freedesktop.impl.portal.desktop.flea.service`.
- [x] Settings > About “Make Flea the default” already claims/releases both directory-handler and FileChooser portal routes and restarts xdg-desktop-portal; this requires the installed package desktop entry and portal backend. Claim tests: 16 passed.
- [x] GTK 4 icon theme lookup validated against Noctalia’s Breeze setting; Open With app icons use the selected theme. Listing file/folder glyphs deliberately remain palette-tinted Flea vectors.

## Decisioni e scostamenti dal piano

- Il checkout è GPUI-first: `flea --gui` lancia `flea-gui`, non Quickshell.
  Il piano assumeva `qs -p ui/boot/shell.qml`; le patch QML restano valide per
  il percorso legacy/picker, ma il tema viene letto anche dal codice Rust.
- I file Omarchy sono presi dal branch `quattro` del repo `omacom/omarchy`
  (il repo upstream è cambiato rispetto a `basecamp/omarchy`).
- `docs/` è gitignored e locale: questo `PROGRESS.md` è la traccia condivisa.
