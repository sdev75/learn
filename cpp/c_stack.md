## Program Stack

The stack is a region shared with the program. It's designed to be used for storing temporary data, especially the parameters and local variables for different execution context. The stack memory is usually located at the highest virtual memory address and thus growing downwards.

The stack memory is handled using pointer arithmetics. `push` and `pop` are common instructions that help making the arithmetics easier. A `push` call to a 64-bit register, would make the `stack pointer` re-adjust to an offset of 64-bits (8 bytes), and then writing to that memory location.

The stack is merely a memory region, which the program has read and write access to. The heap on the other hand must be requested from the operating system with system calls. The stack is automatically reserver by the operating system on program's startup. The program can manipulate the stack memory region with the use of `stack pointers`. `push` and `pop` instructions are just wrappers to make things easier, including but not limited to `call`, `leave` and `ret`.

The following example should give you a better picture of how the stack is designed inside the memory of a running process:

```
  |
9000  +------------------------< memory end
7000  |                        | <kernel memory>
5000  |                        |
2000  |                        |
0700  |                        |
0500  |                        | <args, env memory>
      +------------------------< [stack] begin
0300  |                        |
      |                        |   |
0390  |                        |   |           ^
      |                        |   | push (-)  |
0380  |                        |   |           | pop (+)
      |                        |   |           |
0370  |                        |  \ /
      |                        |
0360  |                        |
      +------------------------< [stack] end
0320  |                        |
      |                        |
0320  |                        < [heap] begin
      |                        |
0200  +------------------------|
      |                        |
0100  |                        < [heap] end
      |                        |
      |                        |
      |                        | <process sections>
      |                        |
0000  +------------------------< memory begin
```

A simple `cat /proc/{PID}/maps` would show more information about the stack and heap memory regions. Both the stack and the heap regions are allocated with read and write permissions.
It's also important to note that some details are implementation-specific and might different on different systems.

#### Stack Frame

The `stack frame` (also known as `activation record`) is the collection of data associated with a an execution context, a function for example. This collection of data, allows the CPU to keep track of the logic state of the execution of our program.

The `stack frame` generally consists of the following components:
| Component | Description |
|-----------|:-------------|
| Return address | Address to return upon completition |
| Local data | Memory allocated for local variables |
| Local params | Memory allocated for function's parameters |
| Stack and base pointers | Memory locations to keep track of the runtime system |

Consider the following code:

```c++
int fn(int x, int y) {
  int a = 5;
  int b = 3;
  int c = x + b + a + b;
  return c;
}

int main() {
  int a = 2;
  int b = 4;
  int c = fn(a, b);
  c = 10;
  return 0;
}
```

The CPU does not differentiate a function from another. Everything is a sequence of memory executed one by one. Thus, the stack memory is used for storing logical state so that it can be restored later on. It basically acts as a sort of a memory for the CPU to remember later on where and how to get back to the previous execution context.

Equivalent assembly code on a 64-bit Intel machine:

```s
main:
    0x0000555555555148 <+0>:     push   %rbp
    0x0000555555555149 <+1>:     mov    %rsp,%rbp
    0x000055555555514c <+4>:     sub    $0x10,%rsp
    0x0000555555555150 <+8>:     movl   $0x2,-0xc(%rbp)
    0x0000555555555157 <+15>:    movl   $0x4,-0x8(%rbp)
    0x000055555555515e <+22>:    mov    -0x8(%rbp),%ecx
    0x0000555555555161 <+25>:    mov    -0xc(%rbp),%eax
    0x0000555555555164 <+28>:    mov    $0x6,%edx
    0x0000555555555169 <+33>:    mov    %ecx,%esi
    0x000055555555516b <+35>:    mov    %eax,%edi
    0x000055555555516d <+37>:    callq  0x555555555119 <fn>
    0x0000555555555172 <+42>:    mov    %eax,-0x4(%rbp)
    0x0000555555555175 <+45>:    movl   $0xa,-0x4(%rbp)
    0x000055555555517c <+52>:    mov    $0x0,%eax
    0x0000555555555181 <+57>:    leaveq
    0x0000555555555182 <+58>:    retq
```

The `main` execution context is pushing the address of the register of `rbp` onto the stack. The stack memory is pushed downwards by the size of `rbp` (64-bit, 8 bytes) or (RBP-0x8), adjusting the `stack pointer (rsp)` register accordingly. Again `push rbp` is setting `rsp = rsp-8` then `*rsp = rbp`, making the `stack pointer` point to the top of the stack. A `pop (+)` operation would move the stack upwards. It's important to understand that the memory is never cleared, so accessing old stack memory offsets will result in undefined behaviour.

```
  # PUSH <operandvalue>
  #        SP = SP - sizeof(<operandvalue>)
  #   MEM[SP] = <operandvalue>
  #

  0xFF -0x00 |                                                       < bp
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < sp moved by 8 bytes
             |---------------------------------------------------
             |
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

<small>\* Reading SP would read +8 bytes, printing the address of the old BP address.</small>

The `mov` instruction is storing the value of SP to BP, making it the new base pointer for the new execution context or activated record.

```
  # MOV BP, SP
  #   -  BP = SP

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < bp, sp
             |---------------------------------------------------
             |
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

`sub 0x10, rsp` is pushing/moving the pointers down (pushing (-)), by the size of `0x10` or 16 bytes. This allows the stack memory to reserve this space for the use of our exectution context, making the `stack pointer (rsp)` point to `rsp=rsp-0x10`. This block of memory wll be used by main to store data.

```

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < bp
             |---------------------------------------------------
             |
             |
  0xFF -0x18 | mem: SP-0x10 | val: ?      | size: 16 bytes (0x10)    < sp
             |---------------------------------------------------
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

`movl` copying data to the data storage we reserved. The base and stack pointers facilitate the access to this data during the execution of the program.

```

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < bp
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)    < sp
             |---------------------------------------------------
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

The stack pointer has never changed. We used the relative position to the base pointer to fill the memory.

The next instructions are setting the values we just stored in the stack into the registers `ecx` and `eax`, including setting the `edx` register directly with the value 0x06 per our source code.

A new exection context is entered with the `callq` instructions. This pushes (-) the address of the next instruction (the value of the `rip` register) onto the stack, also referred to as `return address`. The register `rip` is then adjusted to point to address of the execution context (function) `fn`.

```
  # CALLQ FN (0x1000)
  #   -  PUSH RIP (0x1234)
  #   -   RIP = &FN (0x1000)
  #   -  JUMP RIP (0x1234)

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < bp
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)    < sp:before-push
             |---------------------------------------------------
  0xFF -0x1C | mem: SP-0x08 | val: *RIP   | size: 08 bytes (0x08)    < sp
             |---------------------------------------------------
             |
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /

```

The code jumps to `fn` and executes the following instructions:

```s
fn:
    0x0000555555555119 <+0>:     push   %rbp
    0x000055555555511a <+1>:     mov    %rsp,%rbp
    0x000055555555511d <+4>:     mov    %edi,-0x14(%rbp)
    0x0000555555555120 <+7>:     mov    %esi,-0x18(%rbp)
    0x0000555555555123 <+10>:    mov    %edx,-0x1c(%rbp)
    0x0000555555555126 <+13>:    movl   $0x2,-0x8(%rbp)
    0x000055555555512d <+20>:    mov    -0x14(%rbp),%edx
    0x0000555555555130 <+23>:    mov    -0x18(%rbp),%eax
    0x0000555555555133 <+26>:    add    %eax,%edx
    0x0000555555555135 <+28>:    mov    -0x1c(%rbp),%eax
    0x0000555555555138 <+31>:    add    %eax,%edx
    0x000055555555513a <+33>:    mov    -0x8(%rbp),%eax
    0x000055555555513d <+36>:    imul   %edx,%eax
    0x0000555555555140 <+39>:    mov    %eax,-0x4(%rbp)
    0x0000555555555143 <+42>:    mov    -0x4(%rbp),%eax
    0x0000555555555146 <+45>:    pop    %rbp
    0x0000555555555147 <+46>:    retq
```

1. Again, the `base pointer` is pushed onto the stack and adjusted to point to the `stack pointer`. Making BP and SP point to the same address in memory. This is also called `prologue`.

```

  # PUSH RBP
  # MOV RBP, RSP

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |---------------------------------------------------
  0xFF -0x1C | mem: SP-0x08 | val: *RIP   | size: 08 bytes (0x04)    < sp:before
             |---------------------------------------------------
  0xFF -0x24 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < bp, sp
             |---------------------------------------------------
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

2. Parameters and local variables are processed.

Assignment of params and variables in done in the following order.

1. EDI = x
2. ESI = y
3. EDX = z
4. a = 0x02

The order of assignment differs in memory:

```

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |---------------------------------------------------
  0xFF -0x1C | mem: SP-0x08 | val: *RIP   | size: 08 bytes (0x04)
             |---------------------------------------------------
  0xFF -0x24 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < bp, sp
             |---------------------------------------------------
  0xFF -0x2C | mem: BP-0x08 | val: 0x02   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x38 | mem: BP-0x14 | val: *EDI   | size: 12 bytes (0x0C)
             |---------------------------------------------------
  0xFF -0x3C | mem: BP-0x18 | val: *ESI   | size: 04 bytes (0x04)
             |---------------------------------------------------
  0xFF -0x40 | mem: BP-0x1C | val: *EDX   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

3. Further arithmetics are performed on the registers and then the `pop RBP` instructions is used to pop out 8 bytes off of the stack. The memory on the stack is left untouched and it could be accessed again, even though this might result in undefined behaviour.

```

  # POP <operandtarget>
  #   <operandtarget> = MEM[SP]
  #                SP = SP + sizeof(<operandtarget>)

  # SP and BP point to the same address
  # BP = MEM[SP] = (*BP previously stored @0xFF-0x24))
  # SP = SP + 8 (moving SP to the return address)

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)   !!! bp is moved here
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |---------------------------------------------------
  0xFF -0x1C | mem: SP-0x08 | val: *RIP   | size: 08 bytes (0x04)    < sp+8
             |---------------------------------------------------
  0xFF -0x24 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)    < bp -> *bp = (0xFF -0x08)
             |---------------------------------------------------
  0xFF -0x2C | mem: BP-0x08 | val: 0x02   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x38 | mem: BP-0x14 | val: *EDI   | size: 12 bytes (0x0C)
             |---------------------------------------------------
  0xFF -0x3C | mem: BP-0x18 | val: *ESI   | size: 04 bytes (0x04)
             |---------------------------------------------------
  0xFF -0x40 | mem: BP-0x1C | val: *EDX   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

The `RSP` register points to the saved RIP address, which is our return address.

4. The `retq` instruction relies on the address value stored in `RSP` and jumps to that address.

```

  # RET
  #   SP = MEM[SP]


  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)   < bp
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)    < sp after ret
             |---------------------------------------------------
             |---------------------------------------------------
  0xFF -0x1C | mem: SP-0x08 | val: *RIP   | size: 08 bytes (0x04)    < sp = *RIP
             |---------------------------------------------------
  0xFF -0x24 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x2C | mem: BP-0x08 | val: 0x02   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x38 | mem: BP-0x14 | val: *EDI   | size: 12 bytes (0x0C)
             |---------------------------------------------------
  0xFF -0x3C | mem: BP-0x18 | val: *ESI   | size: 04 bytes (0x04)
             |---------------------------------------------------
  0xFF -0x40 | mem: BP-0x1C | val: *EDX   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

The callee `fn` returns to the caller `main` using the return address stored in the stack.

`main` finally uses `leaveq` instruction to set the `stack pointer` to the address of `base pointer` and then pop the `base pointer` to return to the previous stack frame.

```

  # LEAVEQ
  #   RSP = RBP
  #   POP RBP
  #      RBP = MEM[SP]
  #       SP = SP + sizeof(RBP)
  # RETQ
  #   SP = MEM[SP]

After RSP = RBP

  0xFF -0x00 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)   < bp, sp
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |---------------------------------------------------
  0xFF -0x1C | mem: SP-0x08 | val: *RIP   | size: 08 bytes (0x04)
             |---------------------------------------------------
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /

After POP RBP (the addresses are fictitious)

  0xFF -0x01 |                                                      < 2. bp
  0xFF -0x02 |
  0xFF -0x03 |                                                      < 3. sp (return address)
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)   < 1. bp = mem[sp] = *bp
             |---------------------------------------------------
  0xFF -0x10 | mem: BP-0x08 | val: 0x04   | size: 08 bytes (0x08)
             |---------------------------------------------------
  0xFF -0x14 | mem: BP-0x0C | val: 0x02   | size: 04 bytes (0x04)
             |---------------------------------------------------
             |---------------------------------------------------
  0xFF -0x1C | mem: SP-0x08 | val: *RIP   | size: 08 bytes (0x04)
             |---------------------------------------------------
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /

After RET

  # RET
  #   SP = MEM[SP]

  0xFF -0x01 |                                                      < 2. bp
  0xFF -0x02 |
  0xFF -0x03 |
             |---------------------------------------------------
  0xFF -0x08 | mem: SP-0x08 | val: *BP    | size: 08 bytes (0x08)
             |---------------------------------------------------
             |
             |
             |   < new stack memory is moving downwards
             |
            \ /
```

Here `rsp` should be pointing back to a previous return address. The mechanism repeats itself for the entire duration of the program.
