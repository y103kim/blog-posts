---
layout: post
category: development
title: How GDB reads stack-frame.
tags: [GDB, dwarf]
---

## Purpose of study

While debugging linux with Ramdump, analyzing callstack is not easy because of inline functions and lack of information about the stack's parameters and local variables. Famous tool [crash](https://github.com/crash-utility/crash/issues/49) does not provide detailed information about inline functions and local variable.

But vmlinux has DWARF sections which includes tons of information about functions, inline functions, parameters and variables. As of this writing, GDB is the one of the greatest tool for reading DWARF. I am going to explore the way how GDB reads the DWARF and callstack on this article.

## Starting point

When you open *vmlinux* with GDB, it starts to read debug sections in vmlinux. As you can see with `readelf -S`, *vmlinux* has many sections to store debug information. Everytime you want to read dwarf, the starting point should be section `.debug_info` which is tree of **DIE**s(Debugging Information Entry)  always.

```
$readelf -S ~/common-kernel/out/android-msm-floral-4.14/dist/vmlinux
There are 48 section headers, starting at offset 0xfd6e828:

  [36] .debug_line       PROGBITS         0000000000000000  01ee2185
       0000000000f1d2eb  0000000000000000           0     0     1
  [37] .debug_info       PROGBITS         0000000000000000  02dff470
       0000000009234331  0000000000000000           0     0     1
  [38] .debug_abbrev     PROGBITS         0000000000000000  0c0337a1
       00000000002e2ab9  0000000000000000           0     0     1
  [39] .debug_aranges    PROGBITS         0000000000000000  0c316260
       0000000000000780  0000000000000000           0     0     16
  [40] .debug_ranges     PROGBITS         0000000000000000  0c3169e0
       000000000064b710  0000000000000000           0     0     16
  [41] .debug_str        PROGBITS         0000000000000000  0c9620f0
       00000000004a65ad  0000000000000001  MS       0     0     1
  [42] .debug_loc        PROGBITS         0000000000000000  0ce0869d
       00000000026550d4  0000000000000000           0     0     1
  [43] .debug_macinfo    PROGBITS         0000000000000000  0f45d771
       0000000000000a99  0000000000000000           0     0     1
  [44] .debug_frame      PROGBITS         0000000000000000  0f45e210
       000000000020dcd0  0000000000000000           0     0     8
```

GDB uses **BFD**(Binary File Descriptor library) to seperate sections in vmlinux. From `symbol_file_add` GDB starts to scan symbols in ELFs like vmlinux. This function is called whenever you give ELF file to first argument of executing GDB command line or you use `file` or `add-symbol-file` command in GDB prompt.

```cpp
/* Process a symbol file, as either the main file or as a dynamically
   loaded file.  See symbol_file_add_with_addrs's comments for details.  */

struct objfile *
symbol_file_add (const char *name, symfile_add_flags add_flags,
		 section_addr_info *addrs, objfile_flags flags)
{
  gdb_bfd_ref_ptr bfd (symfile_bfd_open (name));

  return symbol_file_add_from_bfd (bfd.get (), name, add_flags, addrs,
				   flags, NULL);
}
```

Section information seperated by BFD is uesd to create `struct objfile` which is corresponding to vmlinux. `syms_from_objfile` will scan symbols from debug sections.

On loading vmlinux file, GDB only collects partial information about the symbol. It means GDB lazily access to subtree of DIEs. Otherwise, if you use `-readnow` flag, GDB will traverse whole DIEs with `expand_all_symtabs`. Thanks to lazy access, vmlinux which has huge debug information can be loaded quickly.

```c++
static struct objfile *
symbol_file_add_with_addrs (bfd *abfd, const char *name,
			    symfile_add_flags add_flags,
			    section_addr_info *addrs,
			    objfile_flags flags, struct objfile *parent)
{
  ...
  objfile = objfile::make (abfd, name, flags, parent);
  ...
    if ((flags & OBJF_READNOW))
    {
      if (objfile->sf)
	objfile->sf->qf->expand_all_symtabs (objfile);
```

The result of first quick lookup to dwarf sections, gdb constructs **partial symbol table**. I refer three types of symbol table of gdb from [GDB internals](https://www.sourceware.org/gdb/5/onlinedocs/gdbint.pdf) below.

- full symbol tables (symtabs). These contain the main information about symbols and addresses.
- partial symbol tables (psymtabs). These contain enough information to know when to read the corresponding part of the full symbol table.
- minimal symbol tables (msymtabs). These contain information gleaned from nondebugging symbols.

## DIE(Debugging Information Entry)

DIE is basic node of tree which makes up `.debug_info` section. Into elf file, DIE is represented as serialized binary form. You can deserialize `.debug_info` section with `readelf -wi`. Below examples are from the vmlinux's `readelf` output.

There are 2 DIEs in the exmaple. Each of them have different types, `DW_TAG_base_type` and `DW_TAG_variable`. Same as other serialized binary data, like python's pickle, type are started with leading tag-byte. You can find the `<1>`s from front of the each DIEs, it shows the level of the DIE node in the tree. And also you can find the hex numbers like `<12d>` from the starts of the each lines, it shows offset from the `.debug_info` section.

```
 # DIE 1
 <1><12d>: Abbrev Number: 6 (DW_TAG_base_type)
    <12e>   DW_AT_name        : (indirect string, offset: 0x1e4f0f): char
    <132>   DW_AT_encoding    : 8	(unsigned char)
    <133>   DW_AT_byte_size   : 1
 
 # DIE 2
 <1><134>: Abbrev Number: 2 (DW_TAG_variable)
    <135>   DW_AT_name        : (indirect string, offset: 0x33366d): __ksymtab_static_key_initialized
    <139>   DW_AT_type        : <0xf6>
    <13d>   DW_AT_decl_file   : 2
    <13e>   DW_AT_decl_line   : 144
    <13f>   DW_AT_location    : 9 byte block: 3 b0 50 81 9 80 ff ff ff 	(DW_OP_addr: ffffff80098150b0)
```

Let's take a look at `struct partial_die_info` corresponding to DIE. Catch from the name `partial`, it does not not have complete information about symbol. It just have offset, tag, and program counter(PC) information. Even the name of the symbol is not included in `struct partial_die_info`.

```c++
/* When we construct a partial symbol table entry we only
   need this much information.  */
struct partial_die_info : public allocate_on_obstack
  {
    ...
    /* Offset of this DIE.  */
    const sect_offset sect_off;

    /* DWARF-2 tag for this DIE.  */
    const ENUM_BITFIELD(dwarf_tag) tag : 16;

    /* If HAS_PC_INFO, the PC range associated with this DIE.  */
    CORE_ADDR lowpc = 0;
    CORE_ADDR highpc = 0;
    ...

    /* Compute the name of this partial DIE.  This memoizes the
       result, so it is safe to call multiple times.  */
    const char *name (dwarf2_cu *cu);
  }
```



## References

1. [IBM: Exploring the DWARF debug format information](https://developer.ibm.com/technologies/systems/articles/au-dwarf-debug-format/#compilation-unit)
2. [Debugging with DWARF](http://dwarfstd.org/doc/Debugging%20using%20DWARF-2012.pdf)
3. [GDB internals](https://www.sourceware.org/gdb/5/onlinedocs/gdbint.pdf)