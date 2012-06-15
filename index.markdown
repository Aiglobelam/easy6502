---
layout: default
---

<h2 id="intro">Introduction</h2>

I'm going to show you how to get started writing 6502 assembly language, the
language used to program the processor used by famous computers like the [BBC
Micro](http://en.wikipedia.org/wiki/BBC_Micro), [Atari
2600](http://en.wikipedia.org/wiki/Atari_2600), [Commodore
VIC-20](http://en.wikipedia.org/wiki/Commodore_VIC-20) and [Commodore
64](http://en.wikipedia.org/wiki/Commodore_64). Bender in Futurama [has a 6502
processor for a brain](http://www.transbyte.org/SID/SID-files/Bender_6502.jpg).
[Even the Terminator was programmed in 6502](http://www.pagetable.com/docs/terminator/00-37-23.jpg).

So, why would you want to learn 6502? It's a dead language isn't it? Well,
yeah, but so's Latin. And they still teach that.
[Q.E.D.](http://en.wikipedia.org/wiki/Q.E.D.)

Seriously though, I think it's valuable to have an understanding of assembly
language. Assembly language is the lowest level of abstraction in computers -
the point at which the code is still readable. Assembly language translates
directly to the bytes that are executed by your computer's processor.
If you understand how it works, you've basically become a computer
[magician](http://skilldrick.co.uk/2011/04/magic-in-software-development/).

Then why 6502? Why not a *useful* assembly language, like
[x86](http://en.wikipedia.org/wiki/X86)? Well, I don't think learning x86 is
useful. I don't think you'll ever have to *write* assembly language in your day
job - this is purely an academic exercise, something to expand your mind and
your thinking. 6502 was written in a different age, a time when the majority of
developers were writing assembly directly, rather than in these new-fangled
high-level programming languages. So, it was designed to be written by humans.
More modern assembly languages are meant to written by compilers, so let's
leave it to them.

{% include widget.html %}


<h2 id="getting-started">Getting started</h2>

Hopefully by this point I've persuaded you that this is going to be worth it.
There are a few things you'll want on your computer in order to get up and
running.  The first is an assembler. This is the program that converts your
assembly language into machine code. The second is a monitor. This is like a
debugger - it lets you step through your program from any point and inspect the
memory and registers. At some point you'll want an Atari emulator, so you run
your programs in their natural habitat, but we'll leave that for now.

I'm going to assume you're using a Mac. All the tools I'm going to recommend
will work cross-platform, but the installation instructions might be different.

Finally, a warning. We'll mainly be representing numbers and memory locations
in [hexadecimal format](http://en.wikipedia.org/wiki/Hexadecimal). If that
means nothing to you, click on that link there.

{% include widget.html %}

###The assembler

We'll be using the DASM assembler. If you have
[Homebrew](http://mxcl.github.com/homebrew/) installed with version 0.9 or
later, installation should be as easy as

    brew install dasm

Otherwise, [here's a link to download
it](http://mac.softpedia.com/progDownload/DASM-Download-34013.html) and [here are
some instructions on installing it](http://blog.feltpad.net/dasm-on-mac-osx/).

###The monitor

To make sense of our assembled files, we'll run them in a monitor. The best one
I've found so far is called [py65](https://github.com/mnaberez/py65). You
should just be able to install it with

    easy_install -U py65

but if you have trouble [go to the Github page](https://github.com/mnaberez/py65).



<h2 id="first-program">Our first program</h2>

Now, let's try writing a working program. Fire up your favourite text editor
and enter this:

      processor 6502
      ORG $C000

      LDA #$01
      STA $01
      LDA #$0a
      STA $02
      LDA #$0f
      STA $03
      BRK

Make sure each line has a two-space indent (the left margin is saved for
labels). Save the file as `example.asm`.

Now, go to the terminal and run:

    dasm example.asm -f3 -v1 -oexample.bin

The `-f` flag is the output format. You always want that to be `3`. `-v` is for
verbose (there are *five* different verbosity levels - possibly slightly
excessive) and `-o` specifies the output filename (in this case `example.bin`).

You can use the `hexdump` tool to see what this program looks like. `hexdump`
outputs each byte in the program as a hex pair. Run `hexdump example.bin` to
see the bytes. This can be a useful debugging tool.

Hopefully this will compile without error, and you'll have a new file called
`example.bin` in your directory. The next step is to load this file up in the
py65 monitor program. To start the monitor, run `py65mon`. This should open up
with a load of output, and give you a `.` prompt. At the prompt type:

    .load example.bin c000

This will load the file into the memory location c000. The output should look
something like:

    Wrote +13 bytes from $c000 to $c00c

We can use the `mem` command to inspect the memory in that byte range like so:

    .mem c000:c00c

This should output something like:

    c000:  a9  01  85  01  a9  0a  85  02  a9  0f  85  03  00

which should be what `hexdump` output as well. We can also convert the machine
code back into assembler language with the `disassemble` command:

    .disassemble c000:c00c

which will output a list of the bytes and their equivalent assembly language
instructions.  By this point you should see that there is basically a
one-to-one mapping between assembly language instructions and compiled bytes.
We really are speaking the computer's language now.

You may notice that the first two lines of the `.asm` file aren't in the
disassembly - these were just instructions to the compiler so they don't end up in
the compiled program.

###Stepping through the program

With this program loaded into the monitor, we can step through it to see how
the computer reacts to each of the instructions. Machine code is read
instruction-by-instruction by the processor. The computer keeps track of the current
instruction using its program counter, which increments after every instruction. To
step through our program we'll first have to move the program counter to the start
of the program. This can be done like so:

    .registers pc=c000

This sets the `pc` (program counter) register to the memory location of the first
instruction of our program. You'll see in the output that `PC` is now `c000`.

Type `step` to execute the first instruction. The first thing to be output
(confusingly) is the next instruction (which hasn't been executed yet). After
that the values of the registers will be output. You'll hopefully notice that
two of the registers have changed. `PC` has increased to `c002` and `AC` is now
`01`. `LDA #$01` means "Load the number 1 into register A". The `$` denotes
hexadecimal values, and the `#` means "this actual number" rather than the
memory location `$01`.

Type `step` again to execute the next command. `STA $01` means "Store the value
in register A into the memory location $01". We can view the contents of memory
in the bytes from `$01` to `$03` with the command:

    .mem 01:03

You'll see from this output that the value 1 has been stored in the second byte
of memory. Now keep stepping through the code, keeping an eye on the A register
and inspecting the memory in the first three bytes, until you reach the end of
the code. Once you've finished, we'll have stored the values `$01`, `$0a` and
`$0f` into memory locations `$01`, `$02` and `$03`. Pretty awesome eh? You
can't do *that* in JavaScript!



<h2 id='registers'>Registers and flags</h2>

When you run the py65 monitor you'll constantly see output like this:

           PC  AC XR YR SP NV-BDIZC
    6502: 0000 00 00 00 ff 00110000

So, what does all this mean? `PC` is the program counter - it's how the
processor knows at what point in the program it currently is. It's like the
current line number of an executing script.

`AC`, `XR` and `YR` are the `A`, `X` and `Y` registers (`A` is often called the
"accumulator"). Each register holds a single byte. Most operations work on the
contents of these registers.

`SP` is the stack pointer. I won't get into the stack yet, but basically this
register is decremented every time a byte is pushed onto the stack, and
incremented when a byte is popped off the stack.

The last section shows the processor flags. Each flag is one bit, so all seven
flags live in a single byte.  The flags are set by the processor to give
information about the previous instruction. More on that later. [Read more
about the registers and flags
here](http://www.obelisk.demon.co.uk/6502/registers.html).



<h2 id='instructions'>Instructions</h2>

Instructions in assembly language are like a small set of predefined functions.
All instructions take between zero and three arguments. Here's some annotated
source code to introduce a few different instructions:

      processor 6502 ;Set the processor (compiler instruction)
      ORG $c000      ;Set the memory origin to c000 (compiler instruction)

      LDA #$c0       ;Load the hex value $c0 into the A register
      TAX            ;Transfer the value in the A register to the X register
      INX            ;Increment the value in the X register
      STA $01        ;Store the value of the A register at memory location $01
      ADC #$c0       ;Add the hex value $c0 to the A register
      STA $02        ;Store the value of the A register at memory location $02
      BRK            ;Break - we're done

As before, put this in a text editor (with a two-space indent), and compile it
with `dasm example.asm -f3 -v1 -oexample.bin`. Then load it into `py65mon`
using `.load example.bin c000`, and set the program counter to the start of the
code with `.registers pc=c000`.  Now step through the code, paying special
attention to what happens when you execute the instruction `ADC #$c0`.
According to this processor, `$C0 + $C0 = $80`. Huh? If you open up OSX
calculator and change the view to "Programmer" you can see what the actual
result of `$C0 + $C0` is (it's `$180`).

So why does the processor give the wrong answer? The problem is, `$180` is too
big to fit in a single byte (the max is `$FF`), and the registers can only hold
a single byte.  It's OK though; the processor isn't actually dumb. If you were
looking carefully enough, you'll have noticed that the carry flag was set to
`1` after this operation. So that's how you know.

Still in the monitor, type the following commands:

    .reset
    .registers pc=c000
    .assemble

The monitor is now in assemble mode, and will treat all input as assembly
language. Type in the following:

    LDA #$80
    STA #01
    ADC $01

Press enter again to finish entering assembly language. An important thing to
notice here is the distinction between `ADC #$01` and `ADC $01`. The first one
adds the value `$01` to the `A` register, but the second adds the value stored
at memory location `$01` to the `A` register.

Now, step through these three instructions. `$80 + $80` should equal `$100`,
but because this is bigger than a byte, the `A` register is set to `$00` and
the carry flag is set. As well as this though, the zero flag is set. The zero
flag is set by all instructions where the result is zero.

A [full list of the 6502 instruction set is available
here](http://www.6502.org/tutorials/6502opcodes.html) [and
here](http://www.obelisk.demon.co.uk/6502/reference.html) (I usually refer to
both pages as they have their strengths and weaknesses). These pages detail the
arguments to each instruction, which registers they use, and which flags they
set. They are your bible.



<h2 id='branching'>Branching</h2>

So far we're only able to write basic programs without any branching logic.
Let's change that.

6502 assembly language has a bunch of branching instructions, all of which
branch based on whether certain flags are set or not. In this example we'll be
looking at `BNE`: "Branch on not equal".

      processor 6502
      ORG $c000

      LDX #$08

    decrement:
      DEX
      CPX #$02
      BNE decrement
      BRK

First we load the value `$08` into the `X` register. The next line is a label.
Labels just mark certain points in a program so we can return to them later.
After the label we decrement `X`, and then compare it to the value `$02`.
[`CPX`](http://www.obelisk.demon.co.uk/6502/reference.html#CPX) compares the
value in the `X` register with another value. If the two values are equal,
the `Z` flag is set to `1`, otherwise it is set to `0`.

The next line, `BNE decrement`, will shift execution to the decrement label if
the `Z` flag is set to `0` (meaning that the two values in the `CPX` comparison
were not equal), otherwise it does nothing and we reach the end of
the program.

The disassembly of this program looks like this:

    $c000  a2 08     LDX #$08
    $c002  ca        DEX
    $c003  e0 02     CPX #$02
    $c005  d0 fb     BNE $c002
    $c007  00        BRK

The labels don't actually exist in the compiled program, so they can't be
regenerated when the program is disassembled. They only exist in our
source code so we don't have to hard-code memory addresses like `$c002`.

Take a look at the hex values in the second column of the disassembly above.
The first pair is always the opcode (the binary version of the three-letter
mnemonic/instruction), and the second pair is the argument, when needed. So,
after assembly `LDX` becomes `$a2`, `DEX` becomes `$ca`, etc. The argument is
usually passed through untouched. So, `LDX #$08` becomes `a2 08` and `CPX #$02`
becomes `e0 02`. Some instructions map to more than one opcode. For example,
`LDX #$08` (load `X` with the hex value `$08`) becomes `a2 08`, but `LDX $08`
(load `X` with the value stored in memory location `$08`) becomes `a6 08`. This
makes sense if you think about it - the argument is just the value `$08`, so
the processor needs to know whether to treat that number as a value or memory
location.

You might notice that the argument to `BNE` in the assembled code is `fb`, not
`c002`. This is because the branch instructions take a relative offset. The
offset takes a signed byte. Unsigned bytes map to decimal numbers like this:

    00        7F 80      FF
    0        127 128     256

Signed bytes map to decimal numbers like this:

      80      FF 00      7F
    -128      -1 0       127

So, `fb` means -5. The program counter has already been incremented to the
next instruction (`$c007`) by the time the branch happens, so a relative offset
of -5 moves it to `$c002`. Thankfully, both the assembler and the disassembler
know how to generate these relative offsets, so we don't generally have to
calculate them.

The only reason it's worth knowing all the intricacies of relative offsets is
because the argument to a branch instruction can only be one byte. That means
that the processor is only able to branch back and forward around 128 bytes.



<h2 id='first-drawing'>Our first drawing</h2>

All this theory gets a bit boring after a while so let's draw some pretty pictures.


<!--

Need to know about different addressing modes here - another section?

  LDA #$00
  STA $00
  LDA #$02
  STA $01
  ; $00 and $01 are now $00 and $02, so will point to memory $0200
  TXA
  STA ($00),Y

<h2 id='jumping'>Jumping</h2>

Jumping is like branching with two main differences. First, jumps are not
conditionally executed, and second, they take a two-byte absolute address. For
small programs, this second detail isn't very important, as you'll mostly be
using labels, and the assembler works out the correct memory location from the
label.

`JMP` is simplest.
`JSR` pushes location onto stack
`RTS` returns location from stack
-->
