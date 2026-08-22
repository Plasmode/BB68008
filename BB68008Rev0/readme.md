barebones 68008, BB68008, is a simple computer consisted of 68008 CPU, 128K/512K RAM, and 22V10 as boot ROM and glue logic. It is similar to barebones Z80 and 6502 in concept and prototyped using a BB6580 board.

![rev0pcb](bb68008_rev0_topview.jpg)

Like barebones 6502 and Z80, BB68008 is bootstrapped over serial port every power cycle. The serial port is bit-bang 57600-N82. There is also a page flipflop that switches out ROM when location $40 is accessed. Most of 22V10 is devoted to bootstrap ROM for 68008. The bootstrap code is 48 bytes so which is near the limit of 22V10 sum-of-product matrix.

This is the serial bootstrap algorithm:
At power up, 22V10-based ROM provides the program for 68008 until location $40 is access which will switch out the ROM and replaced with RAM. While ROM is enabled, the RAM is enabled and writable so the role of ROM bootstrap is to decode incoming serial data and write to itself starting from location $0. When location $40 is written, the ROM is replaced with RAM but the bootstrap program remained the same. When 512 bytes of serial data are received, the program starts at location $40.
