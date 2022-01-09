## The Stack

A program is a sequence of instructions stored in memory that the CPU executes. In order to keep track of the address being executed the CPU uses a special register to store the address of the currently executing instruction, this is referred to as the `program counter` register or `pc`. This register helps the CPU to keep track by incrementing whenever an instruction is executed. 
A `stack pointer` or `sp` is another special register used to store the address of the current call stack. 

The stack is nothing but a fixed chunck of memory allocated within the virtual memory address by the operating system. Each program has a stack and it is used to store local variables and save the state of the application especially when performing function calls.

### Procedure Activation Record

Whenever a function is invoked, a procedure activation record is used to store data on the stack. It records everything needed to get back to where the program came from before the call. 

Here's how a procedure activation record could look like in memory when pushed onto the stack: 

```
         Address space of a program 
          highest memory address
-----------------------------------------------
                    STACK
+-----------------------+   <-- activation record for foo()
| local variables       |
+-----------------------+
| arguments             |
|                       |
|                       |
+-----------------------+
| address prev frame    |
+-----------------------+
| return address    	|
+-----------------------+  <-- activation record for bar()
| local variables       |
+-----------------------+
| arguments             |
|                       |
|                       |
+-----------------------+
| address prev frame    |
+-----------------------+
| return address    	|
+-----------------------+



-----------------------------------------------
                 BSS SEGMENT


-----------------------------------------------
                 DATA SEGMENT



-----------------------------------------------
                 TEXT SEGMENT


-----------------------------------------------
                  UNMAPPED
-----------------------------------------------

           lowest memory address
```

The stack memory address would probably look similar to `7ffffffde000-7ffffffff000` on a x64 cpu. This could be found by looking up the process map using `/proc/<pid>/maps` within the [stack] segment.

When loading the program, the stack pointer might be initialized to a stack address such as `7fffffffe918` within the `main` subroutine. 

The code below will use comments to go through the stack pushing and popping operations to give a better idea on how it is interpreted by the CPU.

```
rbp -> 0
rsp -> 7fffffffe918
main:
			// == main frame ==
  push rbp		// rsp := 7fffffffe910 (sub rbp 8 bytes)
			// rsp @ 7fffffffe910 := 000000000000 (rbp)
			// RBP also referred to as the frame pointer

			// Pushing subtracts 8 bytes from the stack memory address
			// Then, RBP is stored at that location
			// In this case, again, (RSP -8) will save the value of RBP 
			// *7fffffffe910 (rsp) := 000000000000 (prev frame ptr)

			
  mov rbp, rsp		// rbp := 7fffffffe910 (main frame)
  mov rax, 0		// rax := 000000000000 

  call foo		// Equivalent of pushing subtracting RSP by 8
			// rsp := 7fffffffe908
			// *7fffffffe908 (rsp) := 555555555151 (* saved RIP)

			// 555555555151 refers to the next instruction *mov rax, 0 
			// This is the address that will be used to return to
			// When returning from the function call. It is the only 
                        // way for the CPU to remember how to go back.


	foo():
			// == foo frame ==
	push rbp	// rsp := 7fffffffe900 (push 8 bytes / subtract 8 bytes)
			// *7fffffffe900 (rsp) := 7fffffffe910 (main frame ptr)
			// 7fffffffe910 this would be the beginning of main stack frame

	mov rbp, rsp	// rbp := 7fffffffe900 (foo frame)

	sub rsp, 0x10   // subtract 10 bytes from RSP
			// rsp := 7FFFFFFFE8F0

	mov DWORD PTR	// assign value 0x64 to RBP- 0x04
        [rbp-0x04],0x64 // *7fffffffe900 := 0x64

	mov eax, 0	// set return value to zero

	call bar
			// sub RSP by 8 bytes
			rsp := 7FFFFFFFE8E8
			*7FFFFFFFE8E8 := 0x55555140 (return address) 
			// this will point to the *leave instruction within
			//  the foo subroutine.
				
		bar():
					// == bar frame ==
			push rbp	// rsp := 7FFFFFFFE8E0 (sub esp by 8 bytes)
					// *7FFFFFFFE8E0 = 7fffffffe900 (foo frame)
			mov rbp, rsp	// rbp := 7FFFFFFFE8E0
			mov [rbp-0x4]
				0xc8	// *7FFFFFFFE8E0 = 0xc8
			nop		// nothing, padding for CPU

			pop rbp		// RSP += 8 bytes
					// rbp := *rsp (7fffffffe900)
					// rbp := *7FFFFFFFE8E0 = 7fffffffe900
					// rsp = 7FFFFFFFE8E8 (rsp + 8);
			
			ret		// Will assign RIP the value contained in RSP
					// # rip := *rsp
					// rip := *7FFFFFFFE8E8‬ -> 0x55555140
					// # increments RSP + 8 
					// rsp := 7FFFFFFFE8F0
					// goes to rip 0x55555140
					

	nop		// padding for CPU
	*leave		// address @ 0x55555140
			// # rsp := rbp
			// rsp := 7fffffffe900
			// # pop rbp	
			// 	rsp += 8 bytes = 7fffffffe908
			// 	rbp := *rsp (*7fffffffe908) -> 0x55555151

ret Pop return address from stack and jump there
	
	ret		// Will assign RIP the value contained in RSP
			// # rip := *rsp
			// rip := *7FFFFFFFE908 -> 0x55555151
			// # increments RSP + 8 
			// rsp := 7FFFFFFFE910
			// goes to rip 0x555555555151
	
 *mov rax, 0		// address = 0x555555555151 
			// this is the return address after `foo` returns

 pop rbp		// rbp := *rsp (*7FFFFFFFE910) = 0x00 
			// rsp := rsp + 8 = 7FFFFFFFE918
			
  ret			// rip := *rsp (*7FFFFFFFE918) = 0xf7e16b25
			// rsp := rsp + 8 = 7FFFFFFFE920
			
```
