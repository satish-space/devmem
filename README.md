# devmem2

Program to read/write from/to any location in physical memory

This is enhanced variant of "devmem" or "devmem2" utility.
Source and documentation: http://git.io/vZ5iD

For Linux 2.6 - 3.x - 4.x

Origin: https://github.com/VCTLabs/devmem2
By lartware, 2004
http://www.lartmaker.nl/lartware/port/

This variant is not 100% compatible with the original, do not use it as a drop-in replacement.

## Command line

    devmem [-switches] ADDRESS [w|b|h]      -- read memory and print
    devmem [-switches] ADDRESS w|b|h VALUE  -- write VALUE into ADDRESS

ADDRESS is a physical address: a number, like 0x12345678 (*)

VALUE is a number like 0x1234 or 42 (*)

The **w|b|h** designate the size of the value to read or write. W is 4-bytes (int32), H is two bytes (int16), B means one byte (int8). Reads and writes are performed as single operation.
For reads, the size is W if not specified. For writes, the size must be specified. Letters W,B,H are **not case sensitive**.

The size parameter w|b|h can also be moved before ADDRESS:

    devmem [-switches] w|b|h ADDRESS        -- read memory and print
    devmem [-switches] w|b|h ADDRESS VALUE  -- write VALUE into ADDRESS

(*) Decimal numbers for address and values are OK. Actually the numbers are read using C strtoul function, so it will interpret numbers like 0123 as octal! Probably not what you want!

NOTE: It is not guaranteed that any physical address can be accessed by this program. Validity of the ADDRESS may be checked by the OS. See the source of the kernel driver which provides the /dev/mem device for details. 

Some physical addresses are hardware registers; writing **or even reading** them can cause your computer/device crash or melt down or explode. You've been warned!

## Switches 
`-r` - read back after write, and print

`-a` - do not require correct alignment

`-A` - Absolute addresses. This does nothing in this version (it always works with absolute addresses)

`-V, --version` - show version

`-d` - debug. print some debug spew.

`--help` - show usage

NOTE: The original program does not have switches. Giving it "--help" will read from address 0.

To check whether you have this or the original version, add a bogus 2nd parameter which will make the old version fail. For example: "devmem -V -".

## Examples

~~~~~~~~~~~~~
devmem 0xC0001234 w   -- read 32-bit value at 0xC0001234 and print
devmem 3221230132     -- same as above (size "w" used by default). Decimal numbers are OK.
devmem W 0xC0001234   -- same as above

devmem 0xbf001234 h 0x55aa -- write 16-bit value 0x55AA at 0xBF001234
devmem -r W 0xC0001234 1   -- write 32-bit value 1 at 0xC0001234, read back and print
devmem 0xC0004321 b 0x55   -- write 8-bit value 0x55 at 0xC0004321

# Examples of parameter validation 

devmem 0xC00043210      -- this will normally fail on a 32-bit system (address longer than 32 bits)
devmem b 0xC0004321 257 -- will fail. Value does not fit in a byte.
devmem 0xC0004321 w     -- will fail. Address not aligned on 4 bytes (use -a if you really mean it)
devmem 0xC0004321 0     -- will fail. Access size (w/h/b) must be specified for write
~~~~~~~~~~

## Output

Read command prints the data in format "%08X\n" to stdout.

Example: data 0x3 will print as 00000003

Write command prints nothing to stdout, unless with read back switch (-r)


All error output prints to stderr.
Help (usage) prints to stderr.

Version prints to stdout.

## Exit codes

0 - success

Other value - parameter or runtime error (TBD)

NOTE: accessing "wrong" memory address can cause immediate exit without any error message.
Exit status needs to be checked. Use `set -e` to make shell check exit status.

## Installation

The utility is a single file, it does not need special installation. 

Creating copies or links with different names (ex. "mem" instead of "devmem") to the executable file is ok.

**Runtime dependencies**:  only the C library (libc.so, etc.) unless built statically.

This program needs that access to /dev/mem file is enabled in the kernel.
If it does not work, check the kernel config file (CONFIG_DEVMEM must be defined, etc.).

To access physical addresses > 32 bits:  use mmap2 instead of mmap (or maybe build the program in 64-bit mode).
