# My Decisions
This documents the decisions I made during the development the e6systems and the reasoning behind them.

## Why not Use ISO9660/El Torito for the Installer and for Booting?
E6os does not support ISO9660 because CD/DVD appears to be dying out and seems to be used less and less. For both booting and transferring files, a modern alternative is using a usb flash stick (or booting from a network).

To boot from a usb stick, it should be formatted with a GPT partitioning scheme and a FAT32 EFI System Partition should be created. For ordinary file transfers, Format the usb stick as FAT32.

## Why use Assembly Language for Booting and Installer?
Creation of the e6 compiler in a way neccessitated my use of Assembly language for the boot loader and installer of e6os. I wanted the compiler to be able to produce UEFI applications and this required knowledge of PE64 executables (particularly the headers), and what better way was there to gain that knowledge than by manually crafting the PE64 headers by hand in assembly language. 

## Miscellaneous
During the early stages of development, I decided I needed, as quickly and early as possible, functions that can help with debugging during development. For this reason, the first function I wrote was `e6_uefi_print_uefi_string`. This function prints out a UEFI string to screen.
