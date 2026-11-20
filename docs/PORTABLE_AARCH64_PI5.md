# AArch64 on Raspberry Pi 5

This standalone repository packages the native-layout TH08 source as a Linux
AArch64 PIE and fixes signed-byte behavior that differs between x86 and ARM.
It retains the upstream portable branch history; the VC7 comparison target and
its original layouts are unchanged.

## Signed byte semantics

`inttypes.hpp` originally declared `i8` as plain `char`. ARM64 GCC defaults to
unsigned plain `char`, while VC7 and x86 GCC use signed `char`. On the first
Stage 5 attract enemy, `(i8)-2` therefore became `254` in
`EnemyManager::SpawnEnemy1`, leading to item script 315 and an invalid ANM
instruction pointer in `ItemManager::SpawnItem`.

The type is explicitly `signed char` when the compiler defines
`__CHAR_UNSIGNED__`; other platforms retain the original typedef. This covers
all reconstructed signed-byte uses without altering serialized sizes.

## Raspberry Pi 5 validation

The corrected build was cross-compiled with Ubuntu 24.04 GCC 13.3.0 and tested
natively on a Raspberry Pi 5 running Debian 13 with labwc/Xwayland:

- `verify-portable-linux.sh` accepts the ELF64 AArch64 PIE and native globals.
- All runtime libraries resolve through the Pi's ARM64 packages.
- The title and attract-mode Stage 5 gameplay render at approximately 60 FPS.
- The original build crashed in the item-drop path; the corrected build passes
  that same transition and returns to the title without a crash report.
- A separate 30-second launch through the installed desktop entry reaches the
  title and exits without a crash report.
- The owner subsequently tested gameplay and reported no bugs.

These checks do not constitute the full 108-recording corpus, a complete
manual playthrough, or MIDI/audio certification. Existing portable rendering
limitations retain their separate status.

## Package

`th08-portable-linux-aarch64.tar.gz` contains the executable, launcher, icon,
license, and this port's runtime guide. Supply `th08.dat` and `thbgm.dat` from
an original TH08 installation; neither archive nor the original executable is
distributed.

Extract it, then run:

```bash
./run-th08.sh "/path/to/original/TH08 directory"
```

The executable expects a Japanese font through Fontconfig or `TH08_FONT`.
The Pi 5 test used the installed Droid Sans Fallback font. SDL2, SDL2_image,
SDL2_ttf, Fontconfig, FreeType, and OpenGL runtime libraries are required.
