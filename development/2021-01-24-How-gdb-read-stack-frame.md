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

On loading file, GDB only collects partial information about the symbol. It means GDB lazily access to subtree of DIEs. Otherwise, if you use `-readnow` flag, GDB will traverse whole DIEs with `expand_all_symtabs`.

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

