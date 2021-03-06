% Smashing the Stack
% Robin McCorkell
% 2017-11-23

# What's this about?

* Variables in memory are stored sequentially (the stack)
* Many programming languages do not check out-of-bounds access to arrays
* We can exploit this to modify *other* variables!

"But I thought we solved this years ago?"

* C is the main culprit, still in use in many places 
* For those at the Rust workshop a few weeks ago: Rust prevents these attacks!

<div class="notes">
</div>

#

[x86 Memory Layout]{.toc-item-selected}

[Adversarial Debugging]{.toc-item}

[Wargaming]{.toc-item}

# Getting started

## Connecting to OverTheWire

1. Mac and Linux have the `ssh` command in a terminal, on Windows download PuTTY.
2. Connect to `narnia0@narnia.labs.overthewire.org` port 2226, password `narnia0`.
3. Once connected, run `cd $(mktemp -d)`, then download the files for this workshop
   with `git clone http://github.com/hackersatcambridge/smashing-the-stack`.

<div class="notes">
* OverTheWire provides a known Linux system to play on
* They host wargames (security challenges)
* Everyone gets their own directory to prevent stepping on each others' toes
* The downloaded files contain some simple C programs that we will test with
* Should see `bin` directory with binaries and C source code, cd into
</div>

# x86 Memory Layout

## Stack Variables

![00-addition.c](img/00-addition.png)

<div class="notes">
* Each declared variable exists on the stack
* Values moved into/out of memory when storing/using variables
* Ordering in memory is compiler-dependent
</div>

## A Deeper Look at the Stack

![The x86 stack](img/stack.png){height=400px}

<div class="notes">
* Note: the stack grows downwards from 0xffffffff
* Terminology: 'above in the stack' means below in memory, and vice versa
* When a function is called, a new stack frame is created
* The bottom of the stack frame is pointed to by `ebp` register
* The top of the stack frame is pointed to by `esp` register
* Below the stack frame (at `ebp` - 4) lies the return address of the function
</div>

## Arrays on the Stack

![01-buffer.c](img/01-buffer.png)

<div class="notes">
* Here, the array goes from low to high memory
* This is down the stack, towards the bottom of the stack frame!
* If we write something into the array larger than the size of the array,
  it will overflow and we can overwrite other variables or the return address
</div>

#

[x86 Memory Layout]{.toc-item}

[Adversarial Debugging]{.toc-item-selected}

[Wargaming]{.toc-item}

# Adversarial Debugging

## GNU Debugger

1. Attach gdb to a program with `gdb ./00-addition` (for example)
2. Activate PEDA with `source /usr/local/peda/peda.py`
3. Run the program with `run [args]`

<div class="notes">
* gdb allows us to pause execution at target functions, then step through
  each instruction
* By default, gdb is difficult to use, the Python Exploit Development Assistance
  for gdb (PEDA) adds awesome features

Go to live example
* Attach to `00-addition`
* Disassemble `main` with `pdisas main`
* Break on `main`
* Observe variables changing when stepping through with `ni`
* Run again, change memory to affect result
* `set *0x12345678 = 0x87654321`
</div>

## Hijacking Function Returns

We overwrote a variable in memory, but we could overwrite any memory we want.
What about overwriting the return address in a stack frame?

1. Open `02-function` in gdb
2. Find the address of `hijack()` with `pdisas`
3. Break execution somewhere inside `foo()`
4. Overwrite the return address

<div class="notes">
* Need to disassemble `hijack()` to find the address
* Within the `foo()` stack frame, recall the return address is below `ebp`
* Take a look at C source code for what to expect
* Program will exit with non-zero error if hijacking failed, otherwise with zero
</div>

#

[x86 Memory Layout]{.toc-item}

[Adversarial Debugging]{.toc-item}

[Wargaming]{.toc-item-selected}

# Wargaming

## The Narnia Wargame

http://overthewire.org/wargames/narnia/

"This wargame is for the ones that want to learn basic exploitation. You can see the most
common bugs in this game and we've tried to make them easy to exploit. You'll get the
source code of each level to make it easier for you to spot the vuln and abuse it."

<div class="notes">
* Wargames are security challenges posted on the internet
* Narnia is a beginner-level binary exploitation wargame
* You access each level with a username and password to SSH into the server
* Each binary has setuid access to give you access to read the next password
* The next password is stored in `/etc/narnia_pass` -> this is our target
* Each binary is in `/narnia`
</div>

## Narnia Level Zero

* For added challenge, don't look at the C source code (only use disassembly)
* Run the program and give it some inputs, see if you can affect `val`

<div class="notes">
Go to live example

* 20 characters before hitting `val`
* Use `(python -c 'print("a"*20 + ...)'; cat) | /narnia/narnia0` to get shell
* Remember little-endian, specify bytes backwards
* Demonstrate `whoami` inside the shell
* `cat /etc/narnia_pass/narnia1`
</div>

## Take it Further

Try the other Narnia levels! Useful tricks:

* Insert special characters with `python -c 'print("\xde\xad\xbe\xef")'`
* Fill buffers with `python -c 'print("a"*n)'`
* But sometimes we need to write our own code to get a shell...

## Shellcode

What if we don't have a handy `hijack()` function to execute?

Shellcode!

http://shell-storm.org/shellcode/

<div class="notes">
* Shellcode is raw x86 encoded instructions that aims to run /bin/sh usually
* Find the 21-byte shellcode under Linux/x86 -> tiny execve sh

Go to live example

* Notice program copies from stdin to a buffer -> buffer overflow
* Inject shellcode and overwrite return addr
</div>

# Questions?

