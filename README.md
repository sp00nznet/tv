# Terminal Velocity — Static Recompilation

Static recompilation of **Terminal Velocity** (Terminal Reality / 3D Realms,
1995) from its shipping DOS binary to native C.

Built on the [pcrecomp](https://github.com/sp00nznet/pcrecomp) toolchain.

## Project Status: **P0 complete. P1 blocked on a front end that does not exist yet.**

This is the honest status and it is the reason the project is interesting.

---

## What P0 found

```
GAME.EXE   624,695 bytes   MZ stub + DOS/4GW + LE payload, Watcom C/C++
TV.EXE      29,238 bytes   MZ launcher, Watcom
```

`GAME.EXE` is a **Linear Executable (LE)** wrapped in a DOS/4GW extender stub.
The toolchain has three front ends — PE32, NE16 and flat MZ — and this is none
of them. `pe/catalog.py` correctly names LE/LX images rather than reporting
them as broken, and then routes them nowhere, because there is nowhere to route
them to.

So P1 here is not "run the disassembler". P1 is **build the LE/LX front end**,
the way El-Fish built the NE one and Civilization built the 16-bit one.

## Why that is worth doing

LE/LX is not a one-game format. DOS/4GW shipped with Watcom C and it is what
the entire 32-bit DOS era was built on:

> Doom · Duke Nukem 3D · Descent · Rise of the Triad · Warcraft II ·
> Blood · Shadow Warrior · Quake (DOS) · Master of Orion II · Magic Carpet

Most of those have source releases or mature reimplementations, which is
exactly why Terminal Velocity is the right one to build the front end *on*:
nobody has reimplemented it, and its neighbours give you a shelf of binaries to
check the decoder against afterwards.

There is also a second consumer already in the collection. OS/2's 32-bit
executable format is **LX**, the same family, and the `os2/` shareware corpus
here is 10,000 programs waiting on it.

The 32-bit instruction decoding itself is already solved — `disasm32.py` is
Capstone-based and does not care how the bytes were packaged. What LE needs is:

- the page table and the fixup (relocation) records, which are per-page rather
  than per-section
- the object table, and the flat-model address space DOS/4GW sets up
- the extender's own interface: INT 21h/INT 31h DPMI calls made from 32-bit
  protected mode

The last one overlaps heavily with `runtime/recomp16/`, which already
implements a whole DOS machine — INT handlers, VGA, an SDL2 HAL — for
Civilization and DinoPark. That is a 16-bit runtime, but the INT surface is
largely the same one; the difference is the register width and who is
translating the pointers.

## Same engine house as the rest of the shelf

Terminal Reality again: Terminal Velocity (1995) → Fury³ (1995) → Hellbender
(1996) → Monster Truck Madness 2 (1998) → Nocturne (1999). Fury³ is
*contemporary with this* and is already playable in recompiled form. Whether
the voxel code in `GAME.EXE` is recognisably the same code as the voxel code in
Fury³ is a question the LE front end would answer on its first run, and a Watcom
DOS build versus an MSVC Windows build of related code is the sharpest possible
test of whether the lifter is compiler-independent.

---

## Where it goes next

1. `disasm/le_parse.py` — object table, page table, fixup records. Model it on
   `ne/ne_parse.py`, which solved the same shape of problem for 16-bit
   segmented images.
2. Point `disasm32.py` at the resulting flat image.
3. Decide how much of `runtime/recomp16/` the DPMI layer can reuse.

Before any of that: Terminal Velocity's own shareware episode is freely
distributable and is the right thing to build the decoder against, because it
can be shared as a test fixture and the retail binary cannot.

---

## Layout

```
tv/
  original/TVINST/   GAME.EXE, TV.EXE
  original/          terminal-velocity-1.2.iso
  analysis/
  docs/
```

## Credits

Terminal Velocity © 1995 Terminal Reality / 3D Realms. This project neither
contains nor distributes any part of it.
