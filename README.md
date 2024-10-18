# My GitHub repository for my CTF Write-Up

## How to investigate

### File type & is the binary stripped ?

```bash
$ file compilation_example.o
compilation_example.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), **not stripped**
```

If the binary is not stripped, it means references are still there. We should try to find some interesting ones and (after more investigation) break with gdb on these references such as the main function and then disassemble.

### Check content of compressed archive without extracting it

```bash
$ file -z myfile
```

### In gdb

info files -> check entry point to break on it

info locals -> check variable values

### Explore dependencies

```bash
$ ldd myfile
```

### Viewing File content with xdd 

You can change the number of bytes displayed per line using the xxd
program’s -c option. For instance, xxd -c 32 will display 32 bytes per line. You
can also use -b to display binary instead of hexadecimal, and you can use -i
to output a C-style array containing the bytes, which you can directly
include in your C or C++ source. To output only some of the bytes, you can
use the -s (seek) option to specify a file offset at which to start, and you can
use the -l (length) option to specify the number of bytes to dump.

```bash
$ xxd 67b8601 | head -n 15
00000000: 424d 3800 0c00 0000 0000 3600 0000 2800 BM8.......6...(.
00000010: 0000 0002 0000 0002 0000 0100 1800 0000 ................
00000020: 0000 0200 0c00 c01e 0000 c01e 0000 0000 ................
00000030: 0000 0000 ➊7f45 4c46 0201 0100 0000 0000 .....ELF........
00000040: 0000 0000 0300 3e00 0100 0000 7009 0000 ......>.....p...
00000050: 0000 0000 4000 0000 0000 0000 7821 0000 ....@.......x!..
00000060: 0000 0000 0000 0000 4000 3800 0700 4000 ........@.8...@.
00000070: 1b00 1a00 0100 0000 0500 0000 0000 0000 ................
00000080: 0000 0000 0000 0000 0000 0000 0000 0000 ................
00000090: 0000 0000 f40e 0000 0000 0000 f40e 0000 ................
000000a0: 0000 0000 0000 2000 0000 0000 0100 0000 ...... .........
000000b0: 0600 0000 f01d 0000 0000 0000 f01d 2000 .............. .
000000c0: 0000 0000 f01d 2000 0000 0000 6802 0000 ...... .....h...
000000d0: 0000 0000 7002 0000 0000 0000 0000 2000 ....p......... .
000000e0: 0000 0000 0200 0000 0600 0000 081e 0000 ................
```

### Extract content with dd

In the xxd output for the bitmap file, the ELF magic bytes appear at offset
0x34 ➊, which corresponds to 52 in the decimal system. This tells you where
in the file the suspected ELF library begins.

before you try to extract the complete ELF
file, begin by extracting only the ELF header instead.
This is easier since you know that 64-bit ELF headers contain exactly 64 bytes.

You can then examine the ELF header to figure out how large the complete file is.

To extract the header, you use dd to copy 64 bytes from the bitmap file,
starting at offset 52, into a new output file called elf_header.

```bash
$ dd skip=52 count=64 if=67b8601 of=elf_header bs=1
64+0 records in
64+0 records out
64 bytes copied, 0.000404841 s, 158 kB/s
```
dd is an extremely versatile tool.
