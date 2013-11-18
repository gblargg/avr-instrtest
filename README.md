Atmega AVR Instruction Tester
=============================
Somewhat complete testing of a good portion of AVR instructions. Tested on atmega8 chip.

When run, outputs results on UART at 57600 bps.

An instruction failure shows offset to look up in main.lss, to find what instruction failed.

One of the I/O ports is used as an 8-bit RAM location, to test I/O instructions. You might have to use a different I/O port than the default (in Makefile); on some atmega8 chips I had to use port D, as ports B and C didn't work.


Limitations
-----------
* Doesn't test instructions not on atmega8
* Doesn't test lpm etc.
* Doesn't test ijmp/call/icall/ret/reti


Not tested
----------
* Every possible combination of fields in instructions, e.g. all register combinations, all constant values
* Every possible combination of register values for instructions
* Backward relative jumps/calls
* SEI's delayed effect, CLI and RETI's non-delayed effect
* DES
* EIJMP, JMP, EICALL, CALL
* SPM
* XCH, LAS, LAC, LAT
* BREAK, SLEEP, WDR


Design
------
Uses an approach like the zexall Z80 CPU test program, though not as elaborate.

Runs each instruction with all registers set to various values, then checksums values in registers and on stack, and compares with table of correct checksums. Many combinations of values are set up for each instruction, triggering edge cases and various status flag outputs. So it's purely pass/fail. Lots of testing bang-for-the-coding buck.

Exercises branches and other instructions both ways by testing with various combinations of status flags.

Prints out table of correct checksums and comments on ones that mismatch table. When run on hardware this can be pasted into program as a known-correct output. Then future runs will compare against those checksums.

Code is written such that addresses of code/variables shouldn't affect test, so you can edit non-functional aspects without disturbing checksums.


Notes
-----
* Kind of cluttered to have different register setup code, but it was necessary for LD/ST instructions that use pointers. Otherwise we want changing values in those registers.

* CALL/RET/etc. were horned in at the end. Probably better than cluttering test framework with them.


Ideas
-----
* Could index table by 16-bit opcode tested, so that reordering of them and insertion of new tests wouldn't cause false failures

* Might be nice to have X/Y/Z somewhat varying for LD/ST tests, but we've got to be sure they don't fall in sync with the repeating test fill values, e.g. X is pointing at a byte that already holds the value in r24 so the load/store doesn't modify r24, and thus the test doesn't detect a broken instruction.


Instructions tested
-------------------
lpm
lpm     r,Z
lpm     r,Z+

lds     r,k
ld      r,X
ld      r,X+
ld      r,-X
ld      r,Y
ld      r,Y+
ld      r,-Y
ldd     r,Y+q
ld      r,Z
ld      r,Z+
ld      r,-Z
ldd     r,Z+q

sts     k,r
st      X,r
st      X+,r
st      -X,r
st      Y,r
st      Y+,r
st      -Y,r
std     Y+q,r
st      Z,r
st      Z+,r
st      -Z,r
std     Z+q,r

pop     r
push    r

cp      r,r
cpc     r,r
add     r,r
adc     r,r
sub     r,r
sbc     r,r
and     r,r
or      r,r
eor     r,r

ldi     r,K
cpi     r,K
subi    r,K
sbci    r,K
andi    r,K
ori     r,K
adiw    r,K
sbiw    r,K

mov     r,r
movw    r,r

mul     r,r
muls    r,r
mulsu   r,r
fmul    r,r
fmuls   r,r
fmulsu  r,r

nop
swap    r
neg     r
dec     r
inc     r
com     r
asr     r
ror     r

bclr    b
bset    b
bld     r,b
bst     r,b
cbi     p,b
sbi     p,b
in      r,A
out     A,r

sbic    A,b
sbis    A,b
sbrc    r,b
sbrs    r,b
cpse    r,r
brbc    b,k
brbs    b,k

rjmp    k
rcall   k
ijmp
icall
ret
reti

-- 
Shay Green <gblargg@gmail.com>
