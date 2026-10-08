# XTIDE-ROM-Card

An 8-bit ISA card that holds a 27C256 EPROM as an option ROM in the upper memory area of a PC/XT-class machine, typically to load the [XTIDE Universal BIOS](https://www.xtideuniversalbios.org/).

The schematic is a redraw of Dan Hoover's "The $4 XTIDE" concept.

## Overview

- **ROM:** 27C256 (32 KB) EPROM in a 28-pin DIP socket
- **Address decode:** 74LS688 8-bit comparator in a 20-pin DIP socket
- **Address select:** 2x4 pin header (J1) with a SIP-5 pull-up resistor network (RN1)
- **Bus connector:** 8-bit ISA edge connector (`Bus_ISA_8bit`)
- **Decoupling:** 100 nF disc capacitors on each IC

### How it works

The 74LS688 compares A19-A15 against the levels set on J1. It also requires AEN low (no DMA cycle) and /MEMR low (memory read). When everything matches, it pulls `ROM_SEL` low and the EPROM drives D0-D7 for the read.

A19 is tied to match 1, so the 32 KB window always falls between 80000h and FFFFFh. RN1 pulls `SEL_A15` to `SEL_A18` high. Installing a jumper pulls that bit low.

### Address selection

| J1 pins | Address bit |
|---|---|
| 1-2 | A15 |
| 3-4 | A16 |
| 5-6 | A17 |
| 7-8 | A18 |

Jumper **in** = 0, jumper **out** = 1.

| ROM base | A18 | A17 | A16 | A15 | Jumpers installed |
|---|---|---|---|---|---|
| C8000h | 1 | 0 | 0 | 1 | 3-4, 5-6 |
| D0000h | 1 | 0 | 1 | 0 | 1-2, 5-6 |
| D8000h | 1 | 0 | 1 | 1 | 5-6 |
| E0000h | 1 | 1 | 0 | 0 | 1-2, 3-4 |
| E8000h | 1 | 1 | 0 | 1 | 3-4 |

- Keep the jumper off 7-8. A18 = 0 maps the ROM into A0000h-BFFFFh, which is video memory.
- C0000h is normally taken by the video BIOS.
- Pick a window that doesn't overlap video memory, a hard disk controller BIOS, or any other option ROM in the machine.

## Opening the project

1. Install [KiCad](https://www.kicad.org/) 10 or newer. The files use the KiCad 10 format and will not open in older versions.
2. Clone the repository:
   ```
   git clone https://github.com/hoffman373/XTIDE-ROM-Card.git
   ```
3. Open `xtide_rom_card.kicad_pro`.

Custom footprints live in `lib/footprints/` and are registered through the project's `fp-lib-table` using `${KIPRJMOD}`, so no library setup is needed. Everything else uses stock KiCad symbols and footprints.

## Programming the ROM

Burn the ROM image (for example, an XTIDE Universal BIOS build configured for your controller) to a 27C256 with any EPROM programmer. If the image is smaller than 32 KB, pad it to 32 KB with `FFh`. The BIOS checksum must be valid or the system BIOS will skip the ROM during its option ROM scan.

## License

CC BY-SA 4.0, share-alike with the original $4 XTIDE design.
