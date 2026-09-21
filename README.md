# zmk-config

ZMK configuration for a PandaKB Lily58 RGB MX on nice_nano boards, using
[zmk-advanced-underglow-effects-vfx](https://github.com/Pr0dt0s/zmk-advanced-underglow-effects-vfx)
for the lighting.

Most of this file is about the LED wiring, because none of it is guessable
from the firmware and all of it was worked out by pressing keys and watching
what lit up.

## The board's LEDs

35 per half, 70 across the keyboard, on a single chain per side:

| Local index | What |
|---|---|
| 0–5 | underglow, facing down, spread across the half |
| 6–11 | number row |
| 12–17 | QWERTY row |
| 18–23 | home row |
| 24–29 | bottom letter row |
| 30–34 | thumb cluster, including the inner bracket key |

The per-key LEDs snake: each row runs the opposite way to the one above it.
Both halves use the same schema, mirrored, so "outer" means the pinky edge of
whichever half a pixel is on.

Rows 6–23 were measured on hardware and agree with a QMK community table for
the same 70-LED wiring, so those eighteen are solid. Rows 24–29 follow from
that table plus the serpentine. **The thumb order (30–34) has the right count
but an unverified internal order, and which of the six underglow LEDs comes
first is still unknown** — they answer to no key, so they affect the shape of
a ripple but never where one starts.

The map itself lives in the module, as `dts/vfx/pandakb-lily58.dtsi`.

### Why this mattered

Without a `key-pixels` map the engine falls back to reusing a key's ZMK
position number as a pixel index. Those are unrelated numbering schemes that
happen to overlap in range, so pressing `n` lit the LED under `9` and pressing
`m` lit the one under `8`. Nothing was wrong with the effects.

## Channels

The strip is split at the boundary the hardware already has:

| Id | Name | Pixels | Carries |
|---|---|---|---|
| 0 | `glow` | 0–5 | ambient scenes, and the reactive ones that ignore position |
| 1 | `keys` | 6–34 | everything |

The lists differ deliberately. A ripple aimed at a key position means nothing
on six LEDs mounted behind the board, and the indicator scenes need more than
six pixels to read at all.

Both overlays must declare **the same channels carrying the same scenes in the
same order**. Relative commands are resolved to an absolute index on the
central and relayed as a bare number, so halves that disagree land on
different scenes. This already bit once: the probe scenes were declared only
on the right, and `VFX_NEXT` silently wrapped at the left half's shorter list
so they could never be reached.

## Keys

All the VFX controls sit on the Mouse layer (`&lt 4 ESC`, held with the left
pinky). Each one drives the **per-key** channel on its own and the
**underglow** with **right shift** held, via `mod-morph` — one key per
control rather than two.

Right shift specifically: the layer is held with the left pinky and every VFX
key is left-hand, so a left-shift chord would want three fingers on one hand.

Position 0 of that layer is the `&lt 4` key itself. It is held down for as
long as the layer is active and so can never register a press of its own;
nothing useful goes there.

## Things to delete when they have served their purpose

- `config/vfx-probe.dtsi` — six single-pixel scenes, one per underglow LED,
  for reading off the order of that part of the chain. Included by both
  overlays and referenced from the glow channel's scene list in each.

The per-key colour bars that used to live alongside them are gone: the
`color-mapping` fix settled what they were there to check, and they were the
only thing in the probe file that touched the keys channel.

## Gotchas found the hard way

**`color-mapping` is not zero-based on red.** Zephyr's `LED_COLOR_ID` values
start `WHITE 0, RED 1, GREEN 2, BLUE 3`, so GRB is `<2 1 3>`. This config
carried `<1 0 2>`, which reads as red-white-green: it fed the chip's red line
a constant zero and never transmitted blue at all. Red could not appear no
matter what any scene asked for.

**The halves do not hear the same keys.** ZMK hands the central every
peripheral press in order to run the keymap, so the central reacts to
everything while the peripheral only hears itself. Crossing only ever needed
to be taught one way, which is what `CONFIG_ZMK_VFX_SPLIT_SYNCED=y` does.

## Building

GitHub Actions builds both halves on every push; see `build.yaml`. The module
is pinned by branch in `config/west.yml`.
