# Caution

UEFI involves a lot of dereferencing and indirection, so when writing a UEFI application in assembly language (in this case NASM) the square brackets must be precisely and accurately placed. The PE header of the installer's code illustrates this in the DOS stub field:
```
.DOS_SIGNATURE:              db "MZ"                                                             ; DOS Signature. This is required
    ; A DOS stub should normally be here but uefi doesn't need it, so this will be filled
    ; with something a bit more useful.
    .pre_start: 
        push rbx
        lea rbx, [DATA_DIRECTORIES.EFI_IMAGE_HANDLE]        ; We're going to store the efi image handle
        mov [rbx], rcx                                      ; Store the efi image handle
        add rbx, 8                                          ; Point to the memory address to store the system table
        mov [rbx], rdx                                      ; Store the pointer to the system table

        ; Point to the simple text output protocol
        add rdx, EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL 
        mov rdx, [rdx]
        mov rcx, rdx                                        ; Killing 2 birds with 1 stone. This happens to be the first parameter for Output String
        add rdx, EFI_SIMPLE_TEXT_OUTPUT_PROTOCOL_OutputString
        mov rbx, [rdx]
        lea rdx, [OPTIONAL_HEADER_START.BOOT_MESSAGE]
        sub rsp, 32
        call rbx
        add rsp, 32
        jmp $

        ; Print the ins
        pop rbx
        ret

        times 60 - ($ - STANDARD_HEADER) db 0    
```

## Caution: never hand-count structure offsets

When you store values at fixed offsets from a base register, such as [rbp + 24], the offset is a copy of your data layout kept in your head. If you later insert, remove or reorder a field in the layout, the code still compiles and still assembles, but it now writes to the wrong slot.

The failure shows up far from the cause. In the e6 installer, a function pointer was saved at offset 24 after a new field had moved that slot to offset 32. The startup code kept working, because it used the pointer straight from the UEFI table. Later, a helper read the correct slot, found it empty, and called address 0, which hung the machine with no error message.

Let the assembler do the counting. Define the layout once as a NASM struc, and refer to fields by name:

```nasm
struc e6_efi_resources
    .image_handle   resq 1
    .system_table   resq 1
    .output_string  resq 1
endstruc

mov [rbp + e6_efi_resources.output_string], rax
```

Now the offset always follows the layout, and adding a field can't silently break the code that uses the others. Even a plain `BSS.OutputString - BSS.EFI_IMAGE_HANDLE` expression is safer than a literal number.
