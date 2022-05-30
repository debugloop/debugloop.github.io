---
title: QMK keymaps and custom keys
date: 2021-03-08T19:23:51+01:00
tags:
  - keyboard
  - project
  - hardware
---

I recently decided to get a new keyboard, as I have been typing on my notebook
until now. I've aways been interested in mechanical keyboards, but haven't been
willing to break the bank for parts I'd even solder myself. Being more of a
software guy, I started out by looking at firmware, specifically
[QMK](https://qmk.fm/#/).

## Finding a keyboard

I browsed the list of supported hardware specifically to find a complete build
that I can ~~upgrade~~ break open and flash with QMK, i.e. a regular consumer
keyboard that uses an Arduino Pro internally.

I decided on a Durgod K320 with Cherry Brown switches. It comes with an US ANSI
layout and comes apart without too much of a fuss, although very solidly built.
The [initial flash][0] was easy enough, and from then on a magic key
combination is available to enter flashing mode.

## The good stuff

Coding your own layouts into your firmware has some advantages. I've done the
usual things of fixing annoying LEDs and removing unwanted functionality, but
here are some of the more fun ones.

### Mouse Mode

QMK supports [mouse keys][1], which I toggle using two different actions:

* `LT(_MS, KC_ESC)` on my escape key, which makes it act as escape when hit, but enables mouse mode when held down
* `TT(_MS)` on my so-called context menu button (left of right control), which
  enables mouse mode on a double tap (to prevent accidental presses)

Mouse mode itself is an assemblage of different key maps on an otherwise blank
layer. It basically maps WASD, HJKL and the arrow keys to mouse movement, with
somewhat ergonomic mouse button locations when using either of these options.
You can see the full layer [here][2].

### Umlauts

I've always had Caps remapped to Escape in software, but using QMK I can repeat
above trick of holding down this new escape key to access a Umlaut layer.
Umlauts are fancy vowels with two dots, i.e. `ä`, `ü`, and `ö`, that are
equivalent to `ae`, `oe`, and `oe` respectively[^0]. Using this, I could easily
type them as Caps plus the base vowel. However, I still find myself using the
international alt key mappings that generally come with Linux, i.e. right Alt
plus Q (`ä`), plus Y (`ü`) and plus P (`ö`).


### Additional extra keys

I've always liked having my window manager act on keys such as `XF86VolumeUp`.
Laptops often come with a whole load of extra ones on their keys upwards of F7.
At first I was stumped on how to map them, but using some rather arcane tables
and some trial and error, I've found the following ones to work reliably. I can
just define those keys to emit F20 for instance, and I get to use the valuable
`XF86AudioMicMute` in my window manager. As you can see on any product image,
Durgod printed audio mute, lower and raise volume on F5 to F7. F8 however has
been neglected and remains without a print, but it is predestined to be mapped
to F20.

Key | Key Code | Name in graphical environment
---:|---------:|-----------
F13 | 191 | XF86Tools
F14 | 192 | XF86Launch5
F15 | 193 | XF86Launch6
F16 | 194 | XF86Launch7
F17 | 195 | XF86Launch8
F18 | 196 | XF86Launch9
F19 | 197 | N/A
F20 | 198 | XF86AudioMicMute
F21 | 199 | XF86TouchpadToggle
F22 | 200 | XF86TouchpadOn
F23 | 201 | XF86TouchpadOff


[0]: https://github.com/qmk/qmk_firmware/blob/master/keyboards/durgod/k3x0/readme.md#initial-flash
[1]: https://docs.qmk.fm/#/feature_mouse_keys
[2]: https://github.com/debugloop/qmk_firmware/blob/0eafc281bb8dbc8e7003dbb5b55ab33e5fbe954d/keyboards/durgod/k3x0/k320/keymaps/danieln/keymap.c#L155-L163

[^0]: Germans love them, and even my name got one.
