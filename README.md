braindance-spacefix - make Braindance's download pre-flight space check work
under Wine/Proton.

### Why it is needed

Braindance (Tauri/Rust) asks the native crate `sysinfo` for disk information.
On Windows sysinfo enumerates volumes via `FindFirstVolumeW()` and then calls
`GetDriveTypeW("\\\\?\\Volume{...}\\")`.  Wine returns `DRIVE_UNKNOWN (0)` for those
volume-GUID paths, while sysinfo only accepts `DRIVE_FIXED (3)` or
`DRIVE_REMOVABLE (2)`.  The disk list therefore ends up empty, and the app's
`get_available_disk_space` / `get_total_disk_space` commands return 0, so every
download is refused with "Not enough space to download the scene".

### What this script does

The app's web UI is embedded in Braindance.exe as brotli-compressed assets.
This script finds the JS bundle that contains the two IPC wrappers

- `async function XXX(e){return await F("get_available_disk_space",{path:e})}`
- `async function YYY(e){return await F("get_total_disk_space",{path:e})}`

and replaces their bodies with the free/total bytes of the filesystem where the
app actually stores its downloads (read with `df(1)` on the Linux side).  The
compressed asset is rewritten in place, so nothing else in the app changes.

### Usage
Add `braindance-spacefix` to your PATH (for example, copy it to `~/.local/bin`), then:
- `braindance-spacefix                          # auto-detect exe + prefix`
- `braindance-spacefix --exe /path/Braindance.exe --prefix /path/pfx # the Steam/proton prefix`
- `braindance-spacefix --exe ... --space-path /mnt/bigdisk`
- `braindance-spacefix --revert                 # restore the original exe`
- `braindance-spacefix --dry-run                # show what would be done`

All options can also be given through environment variables:
`BD_EXE`, `BD_PREFIX`, `BD_SPACE_PATH`, `BD_STEAM` (see --help).
