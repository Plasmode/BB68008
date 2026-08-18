# Barebones 68008 Prototype
Barebones 68008 is a very simple 68008 SBC similar to barebones Z80 and barebones 6502.  The prototype BB68008 is constructed by modifying BB6580 pc board.

![bb68008proto](BB68008_prototype_topview.jpg)

### Features
- 8MHz MC68008
- 128K/512K RAM
- 57600 N82 bit-bang serial port
- Serial bootstrap
- ATF22V10 as glue logic and bootstrap ROM
- 64-byte ROM in ATF22V10

### Theory of Operation
Refer to here for a short explaination for embedding ROM in 22V10

At power up, 22V10-based ROM provides the program for 68008 until location $40 is access which will switch out the ROM and replaced with RAM. While ROM is enabled, the RAM is enabled and writable so the role of ROM bootstrap is to decode incoming serial data and write to itself starting from location $0. When location $40 is written, the ROM is replaced with RAM but the bootstrap program remained the same. When 512 bytes of serial data are received, the program starts at location $40.

```
SerRx    equ $fffffff4
page     equ $fffffffe
         org $0
;serial bootstrap with 22V10
;serial receive is hooked up to D[7]
;CPU clock is 8MHz
         dc.l $8000
         dc.l start
         org $14
start:
         lea $0,a0                 ;start serial load from $0
         move.w #$200,d3            ;write 512 bytes into RAM
setup:
         moveq #7,d1               ;8 bits in serial receive
startBit:
         move.b SerRx,d2
         bmi startBit
         movem d0-d7/a0-a3,(a7)     ;wait 1.5 bit time
getSer:
         asl SerRx                  ; shift into extend bit
         roxr.b #1,d2                 ; D2 holds the received value
         movem d0-d7,(a7)           ;wait 1 bit time
         dbra d1,getSer
         move.b d2,(a0)+
         dbra d3,setup

         bra.s bootend               ;prefetch this instruction
         nop
bootend:
;this is location $40
;loaded program resume execution here.
```

### Design Files
- Schematic
- ATF22V10 design file

### Software
- Serial bootstrap algorithm embedded in ATF22V10 ROM

- S-record loader. Send BB68K8Load.bin as binary file first, then send S record file to be loaded and execute at $400

- Hello world demonstration software

