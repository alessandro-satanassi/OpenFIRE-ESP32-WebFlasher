# OpenFIRE-ESP32-WebFlasher

## Local firmware catalog

The version menu reads `firmware/versions.json`, not the GitHub Releases API.
It lists only the versions explicitly included in this file, in the given order:

```json
{
  "versions": [
    { "tag_name": "v7.0.0", "prerelease": false },
    { "tag_name": "v7.0.0-RC1", "prerelease": true }
  ]
}
```

- The first entry is the default version. Reorder entries to change the menu order.
- `tag_name` must exactly match a directory under `firmware/`. Use only letters,
  digits, dots, underscores and hyphens, starting with a letter or digit.
- `prerelease` is a JSON boolean (`true` / `false`), not quoted text.
- Each directory contains its existing `manifest.json`, the listed binaries and
  its local `board_pics/` illustrations.
- `?tag=v7.0.0` selects that version only if it is in the catalog. Otherwise the
  site selects the default. There is no ten-version limit.
- A missing or invalid per-version manifest disables that version. If the
  requested version is unavailable, the first available version is selected.
- Removing a catalog entry hides it, including from direct links. Its files may
  remain on disk. Publishing a different version will not add it back.
- An empty `versions` list is valid and disables firmware installation.

## Automatic and manual publication

The firmware workflow updates this repository only when `update_webflasher` is
enabled (and its existing release/tag conditions are satisfied). It replaces the
complete directory of the target version, generates its manifest, and updates the
catalog in the same commit. Stale files in that directory are removed. Other
version directories are not changed.

A new tag is inserted at the beginning. An existing tag keeps its position;
only its `prerelease` flag is updated in the catalog. Other entries, their order
and extra metadata are preserved. Explicitly republishing a hidden tag adds it
back as a new catalog entry.

Availability on the WebFlasher is independent of GitHub release drafts,
publication or deletion: it starts when the updated site is deployed. Updating
the repository does not bypass the normal GitHub Pages deployment delay.

You can also publish manually: prepare the version directory and its manifest,
add or update its catalog entry, and commit the files together. Deploy this
catalog and the updated `index.html` together before using the new firmware
workflow. If the catalog is absent or invalid, synchronization fails rather than
rebuilding it from directories and undoing manual choices.

Do not edit a version directory or the catalog while an Action is publishing that
same content. Conflicting edits are not forcibly overwritten: the Action fails
and must be rerun after the conflict is resolved.

## Versioned board illustrations

Previews load from `firmware/<tag_name>/board_pics/`, never from the firmware
repository or an external image service. A missing picture shows the existing
placeholder; it does not prevent firmware installation.

The workflow takes **only the five preview SVGs used by the page** from
`docs/board_scheme/` in the exact firmware commit being built. It reuses
`lightgun/scripts/build_board_pics.py` to remove editor metadata and optimize
embedded raster pictures (WebP quality 85, maximum raster side 1024 pixels).
Vector content stays vector. The WebApp source files and caches are not changed.

These five shared files cover all 5 Lightgun, 7 Dongle and 5 Pedal entries in
`index.html`:

- `ESP32S3-Devkit-C.svg`
- `esp32-s3-pico.svg`
- `esp32-s3-zero.svg`
- `LILYGO-T-Dongle-S3-ESP32-S3.svg`
- `esp32-s3-pocket-dongle-s3.svg`

Other diagrams in the source directory are not processed or copied. When adding
a board that needs a new image, update the page's `img` field and the workflow's
`required_pictures` list together. Preview filenames must exist in every version
that uses them.

Pictures are prepared before replacing the previous version and committed
together with its binaries, manifest and catalog. Invalid or missing required
pictures fail synchronization before publishing. This also runs when
`update_webapp` is disabled. Manually published versions must include their
`board_pics/` directory too.

## Installation modes

All versions use the same filename convention:
`OpenFIRE-<DEVICE>-<BOARD>.bin`. There is no legacy filename detection.

Both Lightgun modes use the same image **without a filesystem image**, named
`OpenFIRE-LIGHTGUN-<BOARD>.bin`. Historical versions must be converted to this
convention before being listed in the catalog; do not use their old full images
under this filename, since a base update would then overwrite stored settings.

- Base update: write the image without erasing the whole flash.
- Clean install: erase the whole flash first, then write the same image.

Dongle and Pedal keep their existing behavior. The esptool-js library and the
firmware installation logic are unchanged.
