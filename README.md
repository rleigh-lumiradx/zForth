
![zForth](/zforth.png)

zForth
======

zForth is yet another Forth, but with some special features not found in most
other forths. zForth was written as a lightweight scripting language for
extending embedded applications on small microprocessors. It's not particulary
fast, but should be easy to integrate on any platform.

Some of zForth's highlights:

- Small dictionary: instead of relying on a fixed cell size, the dictionary is
  written in variable length cells: small and common numbers take less space
  then larger, resulting in 30% to 50% space saving

- Portable: zForth is written in 100% ANSI C, and runs on virtually all
  architectures. Tested on x86, x86_64, ARM, ARM thumb, Mips, Atmel AVR and 8051

- Small footprint: the kernel C code compiles to about 3 or 4 kB of machine
  code, depending on the architecture and chosen cell data types.

- Tracing: zForth is able to show a nice trace of what it is doing under the
  hood, see below for an example.

- Implemented as a small virtual machine: not the fastest, but safe and
  flexible. Instead of having direct access to host memory, the forth VM memory
  is abstracted, allowing proper boundary checking on memory accesses and stack
  operations.

- Flexible data types: at compile time the user is free to choose what C data
  type should be used for the dictionary and the stacks. zForth supports signed
  integer sizes from 16 to 128 bit, but also works seamlessly with floating point
  types like float and double.

- Easy extendibility: calling C code from forth is easy through a host system
  call primitive, and code has access to the stack for exchanging data between
  Forth and C.



Usage
=====

zForth consists of only two files: zforth.c and zforth.h. Add both to your
project and call `zf_init()` and `zf_bootstrap()` during initialisation. Read
forth statements from a file or terminal and pass the strings to `zf_eval()` to
interpret, compile and run the code. Check the embedded documentation in
`zforth.h` for details.

A demo application for running zForth in linux is provided here, simply run `make`
to build.

To start zforth and load the core forth code, run:

````
./zfort forth/core.zf
````

And zForth will welcome you with the startup message:

````
Welcome to zForth, 786 bytes used
````

zForth is now ready to use. Try some of the following:

Adding one and one or calculate the 144 squared:

````
1 1 + .
144 dup * .
````

Print the sine of 10 numbers between 0 and PI

````
: pi 3.141592654 ;
: demo 0 begin dup sin . pi 10 / + dup 10 > until ;
demo
````

Load and run the demo Mandelbrot fractal:

````
include forth/mandel.zf
````


Tracing
=======

zForth can write verbose traces of the code it is compiling and running. Make sure the 
feature ZF_ENABLE_TRACING is enabled in zforth.h. Below is an example trace
of the following program:

````
: square dup * ;
: test 5 square . ;
test
````

The following symbols are used in tracing:

- stack operations are prefixed with a double arrow, `«` means pop, `»` means push.
  for operations on the return stack the arrow is prefixed with an `r`

- the current word being executed is shown in square brackets, the format
  is `[<name>/<address>]`

- lines starting with a + show values being added to the dictionary

- lines starting with a space show the current line being executed, format
  `<address> <data>`

- lines starting with `===` show the creation of a new word


Here is a trace of execution of the word `test` as defined above.

````
test

r»0 
[test/0326] 
 0326 0001 ┊  (lit) »5 
 0328 031c ┊  square/031c r»810 
 031c 000b ┊  ┊  (dup) «5 »5 »5 
 031d 0007 ┊  ┊  (*) «5 «5 »25 
 031e 0000 ┊  ┊  (exit) r«810 
 032a 0133 ┊  ./0133 r»812 
 0133 0001 ┊  ┊  (lit) »1 
 0135 0019 ┊  ┊  (sys) «1 «25 
 0136 0000 ┊  ┊  (exit) r«812 
 032c 0000 ┊  (exit) r«0 25 
````

This is the trace of the definition of the `square` and `test` words

````
: square dup * ;

r»0 
[:/002c] 
 002c 0003 ┊  (:) 
 002c 0003 ┊  (:) 
=== create 'square'
+0313 0006 ¹ 
+0314 02eb ² 
+0316 0000 s 'square'
===
 002d 0000 ┊  (exit) r«0 
+031c 000b ¹ +dup
+031d 0007 ¹ +*r»0 
[;/0031] 
 0031 0004 ┊  (;) 
+031e 0000 ¹ +exit
===
 0032 0000 ┊  (exit) r«0 

: test 5 square . ;

r»0 
[:/002c] 
 002c 0003 ┊  (:) 
 002c 0003 ┊  (:) 
=== create 'test'
+031f 0004 ¹ 
+0320 0313 ² 
+0322 0000 s 'test'
===
 002d 0000 ┊  (exit) r«0 
+0326 0001 ¹ +lit
+0327 0005 ¹ 
+0328 031c ² +square
+032a 0133 ² +.r»0 
[;/0031] 
 0031 0004 ┊  (;) 
+032c 0000 ¹ +exit
===
 0032 0000 ┊  (exit) r«0 

````
