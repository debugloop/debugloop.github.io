---
title: A toy CHIP-8 Interpreter
date: 2022-04-27T20:23:51+02:00
tags:
  - python
  - interpreter
  - project
---

A few days ago, I discovered an old project on an even older HDD. I suspect
that this was my first time tinkering with emulators, and also my first time
trying out Python 3. It turned out quite alright.

The project is an interpreter for a [CHIP-8][0], an assembler-like language
developed in the 70s. Implementations of CHIP-8 VMs like this one are often
called emulators, although this is strictly speaking incorrect, as the CHIP-8
is not a real hardware platform. Having just 35 opcodes, writing a toy CHIP-8
interpreter takes rather low commitment aside from actually learning how an
emulator/interpreter works and trying out Python 3.

[This implementation][3] is from around 2012 I suspect, but as of yet
unpublished. It is poorly tested and probably ridden with bugs, and the
perfectionist inside prevented me from ever just putting it out there.
The actual result however is rather cool, the [ncurses][1] TUI sells the retro
look and feel very well, as the following "screenshot" of "Breakout!" can prove:

<center class="override-fontsize" style="font-size: .8rem; line-height: 1;">

```text
┌────────────────────────────────────────────────────────────────┬────────────────────────────────┐
│█ █ █ █ █                                              ████   █ │ 0x025A:       sprite VC,VD,0x1 │
│                                                       █  █  ██ │ 0x025C:              mvi 0x30E │
│                                                       █  █   █ │ 0x025E:       sprite V6,V7,0x1 │
│                                                       █  █   █ │ 0x0260:              add V6,V8 │
│                                                       ████  ███│ 0x0262:              add V7,V9 │
│                                                                │ 0x0264:              mov V0,63 │
│███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ │ 0x0266:              and V6,V0 │
│                                                                │ 0x0268:              mov V1,31 │
│███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ │ 0x026A:              and V7,V1 │
│                                                                │ 0x026C:             skne V7,31 │
│███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ │                                │
│                                                                │ 0x0270:              skne V6,0 │
│███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ │                                │
│                                                                │ 0x0274:             skne V6,63 │
│███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ ███ │                                │
│                                                                │ 0x0278:              skne V7,0 │
│███ ███ ███ ███ ███ ███     ███ ███ ███ ███ ███ ███ ███ ███ ███ │                                │
│                                                                │ 0x027C:       sprite V6,V7,0x1 │
│                                                                │ 0x027E:              skeq VF,1 │
│                            █                                   │ 0x0280:              jmp 0x2AA │
│                                                                │                                │
│                                                                ├────────────────────────────────┤
│                                                                │ PC:  0x02AA                    │
│                                                                │  I:  0x030E    SP:  0x0000     │
│                                                                │ V0:  0x003F    V8:  0x0001     │
│                                                                │ V1:  0x001F    V9:  0x0001     │
│                                                                │ V2:  0x0001    VA:  0x0040     │
│                                                                │ V3:  0x003C    VB:  0x0012     │
│                                                                │ V4:  0x0000    VC:  0x0020     │
│                                                                │ V5:  0x0001    VD:  0x001F     │
│                                                                │ V6:  0x001C    VE:  0x0005     │
│                                ██████                          │ V7:  0x0013    VF:  0x0000     │
└────────────────────────────────────────────────────────────────┴────────────────────────────────┘
```
</center>

The game window itself is on the left hand side, while the smaller side
panes on the right allow a peek into interpreter internals: The upper portion
shows the current memory location of the execution flow, as well as a limited
view of its previous locations. Skips and jumps are visualized by empty lines.
The lower portion shows the interpeters full internal state, with all relevant
registers and fields.


## An Incomplete Intro to Processors and Microcontrollers
Processors generally use commands called opcodes to manipulate the system's
memory and its peripherals. To this end, opcodes are read from memory locations
specified by the program counter (PC), which is incremented after each
processed instruction. This behaviour can be seen in above screenshot, where
the upper right pane shows memory locations and opcodes, which have been
decoded from their binary formats to their matching [mnemonic][2].

Often, these instructions are about arithmetics using either two registers or a
single register and a constant. Other instructions manipulate the PC, thus
jumping the execution to another part of the program contained in the memory.
These jumps can be conditional and with or without fixed return addresses.
Additionally, even simple processors implement timers and interrupts, among a
plethora of other instructions.

## Instructions of CHIP-8
As stated above, the implementation needs just 35 opcodes, which are mapped
using a simple Dict:

```python {linenos=table}
op_map = {
        0x00e0: ("cls", clear_screen),              # 0x00e0 cls
        0x00ee: ("rts", return_from_subroutine),    # 0x00ee rts
        0x1000: ("jmp NNN", jump),                  # 0x1NNN jmp NNN
        ...
        0x8004: ("add VX,VY", add_reg),             # 0x8XY4 add VX,VY
        0x8005: ("sub VX,VY", sub_reg),             # 0x8XY5 sub VX,VY
        ...
```

Decoding an instruction works by blanking the irrelevant nibbles of an opcode.
The opcode for adding constants to a register for instance is `0x7XRR`, its
mnemonic `add VX,RR`. The nibble marked as X in the hexadecimal code
corresponds to one of the 16 general purpose registers (named V0 to V15), while
the constant itself is encoded in the byte marked with R. An `add` instruction
is thus decoded by taking the raw memory value and masking it with a binary
`AND 0xf000`, effectively hiding all "instruction arguments" while doing the
lookup in above Dict.

## Creating an Emulator/Interpreter

This very simple CHIP-8 interpreter is basically a loop which decodes
instructions and executes them on an object which keeps the state. Most
interpreters are obviously much more complex than that, but work on the same
principle.

Each time an instruction is parsed, the function matching the instruction is
called with the correct parameters and the UI is updated to include the latest
internals and (possibly, depending on the instruction) update the drawn screen.

For instance, adding a constant to an existing register value is done using the
following function:

```python
def add_const(self): # 0x7XRR add VX,RR
    result = self.v[(self.opcode & 0x0f00) >> 8] + self.opcode & 0x00ff
    self.v[(self.opcode & 0x0f00) >> 8] = result & 0xffff
    return "add V{:X},{}".format((self.opcode & 0x0f00) >> 8, self.opcode & 0x00ff)
```

In Line 2, `(self.opcode & 0x0f00) >> 8` extracts the nibble corresponding to
the register number, while `self.v[...]` accesses its contents. The same is
done for the opcode byte representing the constant. Both values are summed and
stored in a result variable. Line 3 writes the result back into the correct
register. As Python supports more than 4 bytes of integers, the result is
clipped using `& 0xffff`. As per CHIP-8 documentation, nothing is done with
possible overflow bits in this instruction. The `return` statement passes a
fully decoded representation of the binary opcode back to the main loop, which
will display it in the TUI's side pane.

## The Screen

CHIP-8 also has a very simple screen. There are very few monochrome pixels
which are simply toggled on or off, with no color value. The TUI does this
using the following function which adds or removes a coordinate tuple from a
set, which gets drawn after an instruction has finished toggling pixels. The
implementation with a set is very simple and stable. There is no need to
arrange the pixels on a plane using arrays, as all instructions operate in
coordinates anyways.

{{< highlight python "linenos=1" >}}
def toggle_pixel(self, x, y):
    self.screen_contents = self.screen_contents ^ {(x, y)}
{{< / highlight >}}

The most common way to draw on the screen is the `sprite` instruction. This
instruction takes a position (x, y) as well as a height n and toggles pixels
according to the memory location specified in the I register. It starts at (x,
y), moves 7 pixels to the right and starts a new line. This is repeated at most
n times, toggling pixels in an 8 times n area. A pixel is only toggled when the
memory content at the location from I is 1. The memory content `0xF0, 0x90,
0xF0, 0x90, 0x90` would yield the following view on a formerly empty screen
(left) with said memory content repeated on the right hand side.

<center style="line-height: .9;">

```text
┌────────┬────────┐
|████    |11110000|
|█  █    |10010000|
|████    |11110000|
|█  █    |10010000|
|█  █    |10010000|
└────────┴────────┘
```
</center>

## Timers, Sound and Peripherals

Even a basic interpeter such as CHIP-8 needs some additional work. I mentioned
timers before, which have to run at the correct speed and need to be set from
some instructions. This implementations does this by counting off 5 cycles, which in
turn are rate limited at 300Hz, resulting in the proper 60Hz timers.
One of these timers triggers a beep tone, which is needs to be produced by the
interpeter.

Input is the other key peripheral that any CHIP-8 interpreter needs to
implement. Certain instructions can request key inputs and the interpeter needs
to pass that information into the internal state.

## Wrapping up

I remember writing this emulator as a fun project, but also as an educational
one. I think I haven't used Python 2 in a fresh project since, and recapping
how interpreters or processors work didn't hurt either. The other takeaway from
this -- ten years later -- is that one shouldn't be afraid to publish imperfect
code, lest it gets buried in an old backup.

Again, you can find the code [here][3]

[0]: https://en.wikipedia.org/wiki/CHIP-8
[1]: https://www.gnu.org/software/ncurses/
[2]: https://en.wikipedia.org/wiki/Assembly_language#Opcode_mnemonics_and_extended_mnemonics
[3]: https://github.com/debugloop/chip8

[^1]: [Computer Architecture: A Quantitative Approach](https://dl.acm.org/doi/book/10.5555/1999263) by Hennessy & Patterson
