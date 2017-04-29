# Instruction Format

This document describes the format and considerations of individual instructions
in memory.

There are 8 instructions in the Combinatron, so 3 bits are needed to encode
them.

```
0000 - Null
0001 - B
0010 - C
0011 - K
0100 - W
0101 - G
0110 - P
0111 - N
1000 - M
```

The intention is for the sentence index to live as a limited space area separate
from main memory. This would allow custom circuit structures to make it fast and
optimized for the Combinatron. With that in mind the number of bits required for
pointers is just enough to address every cell in the index. This does require
the translation of pointers in memory to pointers in the index. For that a
translation buffer is kept that takes the memory address and maps it to the
index address.

Suppose that the full instructions are two bytes long, that leaves 16-4 bits for
indexing the sentence index. Thus, the sentence index can have 2^12 = 4096
addresses.

So the full instruction format is:

---|-------------
0000 000000000000

Where the first four bits are the instruction and the last fourteen bits are
reserved for pointers.

## Program File Format

Right now the program file format is simple. A header that contains the string
"CBF" ASCII encoded, followed by a 13-bit little-endian integer of the number of
sentences in the file, followed by 3 0-bits. Following the header is simply the
sentences as they should be loaded into memory. Each sentence must be a full,
3-word sentence even if it contains nulls.
