# My Decisions
This documents the decisions I made during the development the e6systems and the reasoning behind them.

## Why not Use ISO9660/El Torito for the Installer and for Booting?
E6os does not support ISO9660 because CD/DVD appears to be dying out and seems to be used less and less. For both booting and transferring files, a modern alternative is using a usb flash stick (or booting from a network).

To boot from a usb stick, it should be formatted with a GPT partitioning scheme and a FAT32 EFI System Partition should be created. For ordinary file transfers, Format the usb stick as FAT32.

## Why use Assembly Language for Booting and Installer?
Creation of the e6 compiler in a way neccessitated my use of Assembly language for the boot loader and installer of e6os. I wanted the compiler to be able to produce UEFI applications and this required knowledge of PE64 executables (particularly the headers), and what better way was there to gain that knowledge than by manually crafting the PE64 headers by hand in assembly language. 

## Decision: use struc for fixed-layout data

Fixed-layout data, such as the UEFI state saved at startup, is defined once as a NASM `struc` and referenced by field name (for example, `e6_efi_resources.output_string`), never by a literal offset (`[rbp + 32]`).

### Why?
- One source of truth. The assembler computes the offsets, so they can't drift from the layout. (See the Caution on hand-counted offsets.)
- Free at runtime. A struc reserves no memory and emits no code, so instruction sizes in the header stay the same.
- Readable. e6_efi_resources.con_out says what the slot is; `[rbp + 16]` doesn't.
- Scope and naming

The struct holds values set once at startup. Changing buffers stay outside it as plain BSS labels.

## Miscellaneous
During the early stages of development, I decided I needed, as quickly and early as possible, functions that can help with debugging during development. For this reason, the first function I wrote was `e6_uefi_print_uefi_string`. This function prints out a UEFI string to screen.
