# Psikyo Shooting Collection Vol. 3 — Native 240p Patch (PS2, SLPM-62593)

Makes the PlayStation 2 release of *Psikyo Shooting Collection Vol. 3* (NTSC-J, **SLPM-62593**) output
**native progressive 240p** instead of 480i, with the **complete, correctly proportioned image** — including
Dragon Blaze in TATE on a CRT.

> **Status:** working. Verified with Dragon Blaze in **PCSX2** and on **real PS2 hardware**
> (composite video, Sony KV-6PR1 Trinitron, rotated for TATE). Other games in the collection are
> **not yet verified** — see [docs/TESTING.md](docs/TESTING.md).

This repository contains **no game data**. It is a small script that patches a disc image **you dump from
your own copy of the game**.

---

## Why this is not just a one-line video-mode patch

The game renders a 640×448 frame meant for interlaced output. Simply switching the console to progressive
scan (or changing the GS field/frame mode) makes the GS read consecutive rows of that buffer, so you see
only the **upper half, stretched**. Getting a correct 240p image requires actually shrinking the frame to
640×224 every frame. This patch injects a tiny GS drawing routine that does exactly that. The full story is in
[docs/HOW_IT_WORKS.md](docs/HOW_IT_WORKS.md).

## What the patch does (short version)

1. Switches the game's NTSC video modes to **non-interlaced (240p)**.
2. Hooks the game's display-environment call at the end of each frame.
3. For 448-row screens (menus and games), it sends one small GS packet that **downscales the finished frame
   2:1 (bilinear) via a scratch area, then copies it back** into the top 224 rows.
4. Waits for the GS to finish, then points the display at those 224 rows (DH = 223).

Screens that are already 224 rows (e.g. the title) are untouched apart from running in 240p. The Taito splash
remains 480i.

## Requirements

- Python 3.8 or newer (standard library only)
- **Your own dump** of SLPM-62593 as an `.iso` (the exact file size of the reference dump is
  232,974,336 bytes)
- To play: PCSX2, or a real PS2 able to run your disc image, plus a display that accepts 240p
  (CRT TVs/monitors via composite, S-Video, RGB or component)

## Installation / Usage

```bash
git clone https://github.com/calcification/psikyo-ps2-240p.git
cd psikyo-ps2-240p
python3 psikyo_vol3_240p.py down2 "SLPM-62593 original.iso" "Psikyo Vol3 240p.iso"
```

- The input file is only **read**; a new patched file is written.
- The script checks every patch location (and the code-cave contents) against the expected original bytes
  and **aborts without writing** if anything differs. A different dump, an already-patched ISO, or a
  different region/version will therefore be rejected rather than corrupted.

### Verify the result (optional)

```bash
python3 tools/verify_patch.py "SLPM-62593 original.iso" "Psikyo Vol3 240p.iso"
```

This confirms the output is exactly the original image plus the patch (nothing else changed) and prints
SHA-256 values. Reference values for the dump this was developed against:

| File | SHA-256 |
|---|---|
| Original dump | `d68d6a370d39ab3d4179779744d78534a25224ffa39f5f606692e503c38c358c` |
| Patched (`down2`) | `65612f340bf9e4b34af48f1140cad78c58f2d28fb0df8fc6992667d37552cf19` |

If your original has a different hash, it may still work (dumps can differ in non-essential ways), but the
script's byte checks are the real safety net.

### Running it

- **PCSX2:** load the patched ISO normally. The image is a 640×224 progressive picture.
- **Real PS2:** run the patched image through your usual method for your own backups. Use a display that
  supports 240p. Many modern TVs and scalers handle 240p poorly; CRTs and good scalers do best.

## Script variants

| Variant | Purpose |
|---|---|
| `down2` | **The patch.** Use this one. |
| `bands` | Diagnostic: draws four solid colour bands in place of the picture. |
| `inplace` | Diagnostic: single-pass in-place downscale. Reads and writes overlapping VRAM in one primitive; **not safe to rely on**. |

## Known limitations

- Only the Dragon Blaze gameplay path has been verified on hardware. Sol Divide and the menus use the same
  code path and are expected to work, but are unverified.
- NTSC-J SLPM-62593 only. PAL-mode entries in the game's video table are not modified.
- The Taito splash screen stays 480i.
- Image quality depends on your display's handling of 240p.

## FAQ

**My CRT buzzes in this mode.** A 240p signal has a different line rate than 480i. The reporter's display
produces the same buzz with *Ico*, so it appears to be normal for that set. The exact signal timing has not
been independently measured.

**Is the picture soft?** Not on a real CRT in testing. In PCSX2 it may look slightly soft because of the
emulator's scaling.

**Why not just use PCSX2's own settings?** The goal is a correct native 240p signal on real hardware, which
needs the game itself to produce a 224-row image.

**Can I use this for other Psikyo/PS2 games?** The mechanism is general, but the addresses are specific to
this executable.

## Documentation

- [docs/HOW_IT_WORKS.md](docs/HOW_IT_WORKS.md) — the problem, the game's frame pipeline, and the solution
- [docs/PATCH_SPEC.md](docs/PATCH_SPEC.md) — exact offsets, original/new bytes, cave code and GS packet
- [docs/INVESTIGATION.md](docs/INVESTIGATION.md) — what was tried, what failed, and why
- [docs/TESTING.md](docs/TESTING.md) — tested configurations and how to help test

## Legal

This project is not affiliated with or endorsed by Psikyo, Taito, Sony, or any rights holder. It contains no
copyrighted game data, only offsets and a small amount of original code and data written for the patch. You
are responsible for using it only with game images you are legally entitled to use. This is not legal advice.
Provided as-is with no warranty; see [LICENSE](LICENSE).
