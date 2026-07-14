- [Chinese](README.md)
- [English](README_EN.md)

# ZMK Companion Template

The recommended ZMK firmware starting point for
[**zmk-companion**](https://github.com/oscampo/zmk-companion): fork this
repo and GitHub Actions builds you a ready-to-flash firmware with the BLE
display feature already enabled, plus a minimal keymap you make your own.
New to all this? Start with zmk-companion's
[`getting_started.md`](https://github.com/oscampo/zmk-companion/blob/main/docs/getting_started.md)
instead of this file, it walks the whole flow end to end.

Built and tested against the "eyelash_corne" split keyboard hardware
(see below). If you have a different ZMK board with a `nice_view` display,
see "Other boards" further down, the display feature itself isn't tied to
this specific hardware, that part just hasn't been tried elsewhere yet.

## Quick start

1. [Fork this repository](https://docs.github.com/en/get-started/quickstart/fork-a-repo#forking-a-repository).
2. [Open the **Actions** tab and enable workflows](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/disabling-and-enabling-a-workflow#enabling-a-workflow)
   if GitHub asks you to.
3. Wait for "Build ZMK firmware" to finish, then download the
   `eyelash_corne_left`/`eyelash_corne_right` artifacts from that run and
   flash each half.
4. Customize your keymap whenever you want, see [Keymap](#keymap) below.

**Already have your own ZMK config repo?** [Add this one as a module instead of forking it](https://zmk.dev/docs/features/modules#building-with-modules),
see "Other boards" below for why that's usually enough to get the display
feature without copying any files.

**Note for forks**: [`config/west.yml`](config/west.yml) already pulls
`boards/arm/eyelash_corne` from its upstream URL. If your fork still has a
local `boards/arm/eyelash_corne` folder alongside that (it may, depending on
how you forked), delete the local copy, the west-fetched one is what's
actually used.

## BLE Keyboard Display

The actual feature this template exists for: a BLE GATT service
(`custom_status_screen.c`) that renders live data (clock, weather, custom
text, whole page layouts) on the keyboard's `nice_view` display, fed by the
zmk-companion Windows app. See its
[user guide](https://github.com/oscampo/zmk-companion/blob/main/docs/user_guide.md)
for what it can do.

The feature itself isn't embedded in this repo's `config/` anymore, it lives
in its own west module,
[`oscampo/zmk-companion`](https://github.com/oscampo/zmk-companion)
(`firmware/`), pulled in via `config/west.yml` and enabled by
`CONFIG_ZMK_COMPANION_DISPLAY=y` (`build.yaml` already sets this for the
`eyelash_corne_left` build, the split's central half). Forking this repo
already gets both, nothing extra to wire up.

**Upgrading an existing fork**: this flag used to be named
`KBD_BLE_DISPLAY` and the source lived directly under `config/`. If your
fork predates this change and still sets `CONFIG_KBD_BLE_DISPLAY=y`
anywhere (a board `.conf`, `build.yaml`), that line is now a no-op, the
build will succeed but silently ship without the display. Update it to
`CONFIG_ZMK_COMPANION_DISPLAY=y` and merge in the `config/west.yml` module
entry described below.

**Other boards**: the display code has no eyelash_corne-specific
dependencies, it only needs the standard `nice_view` shield and a split ZMK
board (it runs on the central half only). You don't need to fork this repo
or copy any files: add `oscampo/zmk-companion` as a module in your own
config's `west.yml` and build your own board with
`-DCONFIG_ZMK_COMPANION_DISPLAY=y`, see
[`zmk-companion`'s getting-started guide](https://github.com/oscampo/zmk-companion/blob/main/docs/getting_started.md#1-firmware-one-time)
for the exact manifest snippet. This hasn't been verified on a board other
than eyelash_corne, if you try it, please open an issue (here or on
zmk-companion) with the result either way, working or not, so this note can
stop being a guess.

**Power-cycle both halves together**: turning only one half off and back
on (central or peripheral) while the other stays powered can leave the two
sides' internal sync mismatched, showing up as pages flashing briefly then
going blank in no consistent order. Confirmed with `zmk-companion`'s own
debug log showing every BLE write succeeding while this happens, so it's a
central/peripheral sync issue on the keyboard's own firmware, not something
the app can detect or fix. Always power both halves off and on together.

## Keymap

`config/eyelash_corne.keymap` is intentionally minimal: plain QWERTY, a
number layer, a nav/function layer, and an empty layer reserved for your
own use, no personal macros, combos, or custom hold-tap behaviors. It's a
starting point, not a finished layout, edit it to match how you actually
type. The easiest way is
[nickcoutsos.github.io/keymap-editor](https://nickcoutsos.github.io/keymap-editor/),
point it at your fork; edits there commit straight to `config/eyelash_corne.keymap`
and GitHub Actions rebuilds automatically. You can also edit the file by
hand, see [ZMK's keymap docs](https://zmk.dev/docs/keymaps) for the syntax.

**Jumping to a Canvas page on demand**: `adjust_layer` has a commented
example showing how to bind spare keys to `Ctrl+Alt+Shift+Win+1` through
`+9`, which `zmk-companion` picks up as global hotkeys to jump straight to
page 1-9, see its
[user guide](https://github.com/oscampo/zmk-companion/blob/main/docs/user_guide.md#pages)
for why that specific combo (short version: four modifiers together is rare
enough in everyday shortcuts to stay a safe choice, and it's a standard
keycode range guaranteed to actually work, unlike the F13-F24 range this
started with, which turned out not to reach Windows at all here).

## Keymap Diagram

![Diagram of config/eyelash_corne.keymap](keymap-drawer/eyelash_corne.svg "generated by @caksoylar's Keymap Drawer")

## Hardware

**This keyboard is not the same as [foostan's Corne](https://github.com/foostan/crkbd). It will not work with standard `corne` firmware.**

![Photo of Eyelash Peripherals Corne](https://ae01.alicdn.com/kf/Sa797fee25edd44248fbfdb0e13d44e00B.jpg)

This repo's board definitions originate from the "睫毛外设" (Eyelash
Peripherals) vendor's own ZMK config for this keyboard. For a 3D model of
the physical keyboard itself, contact the vendor at `380465425@qq.com`,
that's hardware support, unrelated to zmk-companion or this template.
