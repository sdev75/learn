## C pointers

Pointers are an important part of the C language. Using pointers improves performance and allows control over dynamic memory allocation as well as making the program more efficient.

In C variables can be alloced globally during the program initialization and destroyed when the program is exited. These `global variables` can be accessed by any functions. Variables can also be declared as `static variuables`, making their scope restricted to their defining function.

Whenever a function call is made, a stack call is created and `local variables` as pushed onto the stack. This is known as the stack memory, which is a temporary mechanism to store data used in function call.

With the use of pointers, a C program can allocate memory on the heap, which is a region in memory allocated and deallocated at runtime on demand. Pointers are important in order to keep a reference to blocks of allocated `dynamic variables` in the heap in order to prevent memory leaks.

#### Pointers downsides

Pointers are a powerful tool to create performant code, dynamic memory management and improving the overall execution of the program. However, if used incorrectly, points can cause many problems such as accessing memory regions out of bound, with or without the required priviledges, referencing memory that is no longer allocated by the pointer.

## Pointers declaration

Pointers are declared using a data type followed by an asterisk, as follows:

```c
int foo; // foo is an integer
int* foo; // foo is a pointer to an integer
```

The asterisk symbol is also used for multiplication and dereferencing a pointer.

## Address of operator

The `address operator` `&` is used to return the address of a variable. It's important to note that pointers and interger are not the same. The value pointed to by the pointer can be accessed using the `indirection operator` `*`, which is frequently referred to dereferencing a pointer.

The following example shows how to use both of them:

```c++
int foo = 10;
int* bar = &foo;
printf("%p\n",*bar); // 0xA
```

In this case the field specifier `%p` is used for displaying the value in an implementation-specific manner, which in most machine it's in hexadecimal notation.

## Pointer to function

The declaration to a pointer to function looks like the following:

```c
void (*foo)(); // foo is a pointer to a function returnig void
```

## Pointer to void

A pointer to void `void* foo` is used for data pointers, not function pointers, they help address polymorphic behaviour.

## Global and static pointers

If a pointer is declared as global or static, it is initialized to `NULL` when the program is executed.

```c++
int *foo;
void bar(){
    static int *baz;
    printf("%p\n",baz); // nil
}
int main(){
    printf("%p\n",foo); // nil
    bar();
}
```

## Pointer sizes

Pointers in C have different types and sizes with different notations. The `size_t` type is an unisgned integer that represents the maximum size and object can be in C language.

```c++
size_t foo = -5;
printf("%zu\n",foo); // %zu field specifier prints an unsigned integer
```

The types `intptr_t` and `uintptr_t` provide a portable way of declaring and storing pointer addresses. Unlike other types, these are guaranteed to have the same size on any system, allowing for conversion from a pointer to its integer representation.

```c++
int* foo;
uintptr_t bar = &foo;
printf("%p\n",&bar);
```

## Pointer arithmetic

Pointers can be added, subtracted and compared. These arithmetic operations are dependent on the data type, having different sizes. For example, incrementing a `pointer to char` by 1, would increase the pointer by 1 byte. Whereas, incrementing a `pointer to short` by 1 would increment the memory to 2 bytes instead.

```c++
int foo[] = {1,2,3};
int* p = foo;

printf("%d\n",*p);  // 1
p += 1;             // inc by 4 bytes
printf("%d\n",*p);  // 2
p += 1;             // inc by 4 bytes
printf("%d\n",*p);  // 3

p -= 1;             // dec by 4 bytes
printf("%d\n",*p);  // 2
p -= 1;             // dec by 4 bytes
printf("%d\n",*p);  // 1
```

### Subtracting two pointers

When two pointers are subtracted, the result is the difference between their addresses.

```c++
int foo[] = {1, 2, 3};
int* begin = foo;
int* end = foo + 2;

// pointer subtraction
printf("Unit diff: %d\n", end - begin); // 2 units

// pointer comparison
printf("End is greater than begin: %d\n", end > begin); // 1 units
```

The pointer's subtraction above returns 2, which refers to the actual units depending on the data type size. In this case the two addresses would be 8 bytes apart.

An alternative and more portable way to express the difference between two pointers is the `ptrdiff_t` type.

#### Multiple indirection

It's quite common to see declarations of pointer to a pointer, also called double pointer. It helps to structure data without making any copies, thus making the code very efficient. Below is an example with comments and ficititious addresses to facilitate the understanding of the mechanism:

```c++
// `foo` is declared as an `array` of `pointers` to `char`
// foo has 2 pointers to char (char*) on the stack
// equivalent to 8 bytes each on a 64-bit machine
// foo     = 0x100 -> 0x500 (pointer value)
// foo [0] = 0x100 -> 0x500 (pointer value)
// foo [1] = 0x108 -> 0x514 (pointer value)
char *foo[] = {

    "1234567890123456789",  // const storage 0x500 (14 bytes)
    "123456789"             // const storage 0x514 (10 bytes)
};

char **bar[2];  // bar is allocating 2 pointers storing a pointer to a char
// bar     = 0x200 -> (nil)
// bar [0] = 0x200 -> (nil)
// bar [0] = 0x208 -> (nil)

bar[0] = &foo[0];       // 0x200 -> 0x100
bar[1] = &foo[1];       // 0x208 -> 0x108

printf(*bar[0]);        // 1234567890123456789
printf(**bar);          // 1234567890123456789
printf(**(bar+1));      // 123456789

// Array subscripting has precendence over indirection
// *(bar[0])    = *(0x100)        = foo[0]
// *(bar[1])    = *0x108          = foo[1]
// **bar      = (*(*bar))       = (*0x100) = foo[0]
// **(bar+1)  = (*(*(bar+1)))   = (*(*(0x208))) \
            = (*0x514)        = foo[1]
```

Foo is stored on the `stack`. The stack is allocated with enough space to store 2 pointers to char `char*` (8 bytes each on 64-bit machine).
For each block of memeory it is assigned an address, the foo addresses pointing to the location of the constant strings defined in our example.

Bar is stored on the `stack`. The stack is allocated with enough space to store 2 pointers, and each of these pointers point themselves to another pointer.

`bar[0]` array subcripting resolves to the fictitious address of `0x200`.

`&(foo[0])` array subcripting resolves to the address on the stack, then the `address-of` operator returns the address of the value pointed by `foo[0]`, which it would be the 20-byte constant string `1234567890123456789`.

`*bar[0]` array subcripting resolves to the address of `foo[0]` (`0x100`) and the indirection resolves to the value of `foo[0]` (`0x100`): `1234567890123456789`.

`**(bar+1)` is resolved from right to left based on the operator precendence. In this case the address of `bar` is increased by 1 unit size `(char*)` resolving to `0x208`.

Then, from right to left, each indirection taken place. `0x208` (`bar[1]`) is dereferenced resulting in `0x108` (`foo[1]`), which in turn dereferences to `0x514` (`123456789`). It would look like this in form of a chain linkage: `*(*0x208) -> *(0x108) -> 0x514`. Printf ends up writing bytes starting from address `0x514` until a NULL character is found.

It's common practice to use `cat /proc/{PID}/maps`. This helps to reveal more information regarding the stack and heap including constant storage.

##### Memory allocation delagation

Using double pointers is also very useful when allocating memory within other functions. It allows to pass in a pointer by address without losing its reference once the pointer is popped from the stack. Here below is an example using fictitious addresses:

```c++
void bar(int* p){
                            // p is on the stack (addr: 0x200)
                            // p (0x200) value is NULL (pass by value)

    p = malloc(4);          // alloc 4 bytes and assign memory addr to p (0x200)
                            // p (0x200) now pointing at 0x2500 (heap)

    *p = 100;               // 0x2500 = (int) 100

                            // p (0x200) goes out of scope
                            // reference to 0x2500 (heap) LOST
}

void baz(int **p){
                            // p is on the stack (addr: 0x300)
                            // p (0x300) value is 0x100 (&main.foo)
    *p = malloc(4);         // alloc & assign mem addr to p (0x100) (&main.foo)

                            // p (0x300) goes out of scope
                            // reference retained by 0x100 (&main.p)
}
int main(){
    int* foo = NULL;        // foo is on the stack, addr: 0x100
    bar(foo);               // pass value of foo
    baz(&foo);              // pass address of foo
    *foo = 50;              // 0x100 = (int) 50
    printf("%d\n",*foo);    // 50
    free(foo);
}
```

As you can see from the example above, every stack variable has its own scope and this is what determines its lifetime. It's important to design the logic based on this.

## Pointers and constants

Pointers can be pointing to constant values, or can be constant themselves, or both.

The following table should help to visualize the most commonly used pointer variations:

| Ptr Type           | Ptr Const | Val Const | Declaration             | Example                |
| :----------------- | :-------: | :-------: | :---------------------- | :--------------------- |
| Ptr                |     Y     |     Y     | `<type>` \*             | int\* foo;             |
| Ptr to const       |     Y     |     N     | const l`<type>` \*      | const int\* foo;       |
| Const ptr          |     N     |     Y     | `<type>` \* const       | int \* const foo;      |
| Const ptr to const |     N     |     N     | const `<type>` \* const | const int \* const foo |

Pointers can be defined to point to a constant, preventing the pointer from changing the value it is referencing.

```c++
const int foo = 500;   // const int
int* bar = &foo;       // pointer to int
*bar = 1000;           // change value of foo to 1000
```

Attempting to change the value of `const int foo` via indirection, would result in a compiler warning: `warning: assignment discards ‘const’ qualifier from pointer target type`

Pointers can also be declared to be constant pointers pointing to a non-constant value.

```c++
int foo = 500;
int *const bar = &foo;      // bar is a const pointer to int
*bar = 1500;                // change foo to 1500
int baz = 100;
bar = &baz;     // error: assignment of read-only variable ‘bar’
```

Constant pointers must be initialized to a nonconstant variable. The pointer itself being a constant variable, cannot be reassigned once it's defined. It's possible, however, to modify the value it's pointing to. Defining `foo` as `const` would also result in a compilation warning: `warning: initialization discards ‘const’ qualifier from pointer target type`.

Constant pointers to a constant allow to be defined, never changed or reassigned.

```c++
const int foo = 10;             // const int
const int * const bar = &foo;   // const pointer to const int
int * baz = &bar;               // pointer to int
```

Any attempts to modify the pointer or the value it points to would cause a compilation error, thus making the pointer and its value immutable.

It's worth nothing that a double pointer could easily circumvent this protection, as follows:

```c++
int** baz = (int**)&bar;      // pointer to pointer to int
**baz = 100;                  // foo will be written to 100
```

## Memory Allocation

Pointers to string literals initialize an array of characters , which usually points to the .rodata section. On a 64-bit little endian system, `foobarbaz` would be stored in 2 words, having 8 bytes each:

```c++
// foo would allocated 2 words (64-bit little endian):
// 0x100: 0x61627261626f6f66      0x208: 0x3b031b010000007a
// contents of 0x208 would be 0x7a in little endian: 'z'

// Hex dump of section '.rodata':
// 0x00002000 01000200 666f6f62 61726261 7a00
char *foo = "foobarbaz";
```

It's worth nothing that changing the contents of the value pointed to by `foo` would result in undefined behaviour.

A char array would instead be stored on the stack, thus in the .text segment of an ELF image.

```c++
// foo would allocated 2 words (64-bit little endian):
// 0x100: 0x61627261626f6f66      0x208: 0x3b031b010000007a
// contents of 0x208 would be 0x7a in little endian: 'z'
// 0x100[8] = 7a
// 0x208[0] = 7a

// gcc -O0 -c test.c && objdump -s -j .text test.o
// foobarbaz is stored in the .text segment:
// # 0010  00488945 f831c048  b8{666f6f 62617262  .H.E.1.H.foobarb
//                              ^
// # 0020 61}488945 ee66c745 f6{7a}00bf 05000000  aH.E.f.E.z......
//          ^                  ^
char foo[] = "foobarbaz";
```

#### Dynamic Memory

Pointers are very useful for managing memory allocated on the heap. The heap is a memory area that the operating system manages. A program has a set of function available to interact with the operating system. These functions are declared in the `stdlib.h` header.

| Function Name | Description                                     |
| :-----------: | :---------------------------------------------- |
|    malloc     | Allocated memory from the heap                  |
|    realloc    | Reallocated existing memory (larger or smaller) |
|    calloc     | Allocates memory and zeros it out               |
|     free      | Returns allocated block to the heap             |

Dynamic memory is allocated on the heap. The heap is a dynamic memory area which is allocated and freed by the functions listed above.

```c++

char *foo = (char*) malloc (sizeof(char) * 5);
if (foo == NULL){
    // it's good practice to check for alloc errors
     exit(1);
}
foo = calloc(1, sizeof(char)*5 );
strcpy(foo, "1234");
char *bar = (char*) realloc(foo, sizeof(char) * 9)
if (bar == NULL){
    // Old memory is not freed in case of an error
    free(foo);
    exit(1);
}
free(bar);
```

The code above, `malloc` is requesting 5 character units, allowing the string `1234` to be copied by `strcpy`, which is appending a final `NULL` character at the end of it, thus the reason of allocating 5 character units in total.

The `realloc` is increasing the memory allocated to by `malloc`, reusing the same block of memory and returning a new one with the copy of the data. `Realloc` is useful for expading or contracting existing memory block, in this case the block `foo` is pointing to. If a new memory block is allocated, the old one will be automatically be freed by `realloc`.

`free` is deallocating, returning memory previously allocated by `malloc`, `calloc` or `realloc`. It's worth nothing that running `free` on a NULL pointer will do nothing. Unlike freeing a pointer twice (also referred to as `double free`) would cause undefined behaviour. It's common practice to assign NULL to a freed pointer in order to avoid runtime exceptions.

#### Dangling Pointers

A dangling pointer is a pointer that has been freed, and an attempt to access its memory is done by the application. The behavious is undefined, including potential security risks.

```c++
int *foo = (int*) malloc(sizeof(int));
*foo = 100;
int *bar = foo;
free(foo);
*bar = 100;
```

The code above, is writing to a memory via indirection. Even though the memory might still be writable, it is no longer owned by the program. Its ownership has ended the moment the free() call returned the block of memory back to the operating system for reuse.

It's important to pay attention when using block statements, as variables go out of scope and any pointer pointing to a variable within that scope will likely point to garbage memory.

```c++
int* foo;
{
    int bar = 100; // stack variable
    foo = &bar;
}
// bar goes out of scope
// foo points to an invalid memory address
```

## Passing and returning pointers

In C language, parameters, including pointers are passed by value. A copy is passed onto the next execution context. The main reason of passing a copy of a pointer is to allow another execution context to modify the memory location pointed to by the pointers arguments as follows:

```c++
void foo(int *a, int *b) {
  printf("a=%p, b=%p\n", &a, &b);  // a=0x500, 0x600
  printf("a=%p, b=%p\n", a, b);    // a=0x100, 0x200
  int tmp;
  tmp = *a;
  // [0x500]a is pointing to mem[0x100]
  *a = *b;
  *b = tmp;
}

int main() {
  int a = 10, b = 100;
  printf("a=%p, b=%p\n", &a, &b);  // a=0x100, 0x200
  foo(&a, &b);
  printf("a=%d, b=%d\n", a, b);  // a=100, b=10
  return 0;
}
```

We are passing the pointers by value, thus a new copy for each pointer is created by `foo`. Both copies are having different memory addresses but pointing to the same memory addresses we setup before the call. We are modifying the value pointed to by the pointer using indirection.

## Returning a pointer

Returning a pointer from a function is useful memory allocation delegation, returning a pointer to the newly allocated memory address.

```c++
int* foo (size_t size){
    int* ptr = (int*) malloc(size*sizeof(int));
    printf("addr ptr itself      : %p\n",&ptr);     // 0x200
    printf("address ptr points to: %p\n",ptr);      // 0x123456
    return ptr;
}
int main(){
    int *x = foo(100);
    printf("addr ptr itself      : %p\n",&x);       // 0x100
    printf("address ptr points to: %p\n",x);        // 0x123456
    free(x);
}
```

It's important to note that it's the responsability of the caller to free the memory once done and before the local pointer goes out of scope, thus losing the reference to the memory address.

One easy caveat is when returning pointers to local variables stored on the stack. As the values go out of scope when returning to the previous stack frame, any pointers referencing to the value on the stack memory will point to invalid memory, thus resulting in undefined behaviour.

```c++
int *foo() {
                                                // Return Address   +8 bytes
                                                // Previous RBP     +8 bytes
  int bar = 100;                                // Int              +4 bytes
  int *baz = &bar;                              // Pointer to int   +8 bytes
  printf("address of bar: %p\n", &bar);         // 0x100
  return baz;
}

int bar(void *p) { *(uintptr_t *)p = 500; }

int main() {
                                                // Return Address   +8 bytes
                                                // Previous RBP     +8 bytes
  register uintptr_t bp asm("rbp");             // Base Pointer     +8 bytes
  int *ptr = foo();                             // Pointer to int   +8 bytes
  bar((void *)((char *)bp - 0x2C));
  printf("addr pointed to by ptr: %p\n", ptr); // 0x100
  printf("val of ptr: %d\n", *ptr);            // 500
}
```

In the example above, `foo` is creating a local pointer `baz` to store the address of the local stack variable `bar`. The function `bar()` is then called using the `stack pointer` register in `main()` to overwrite the local variable located on the stack.

If using the GCC compiler, `-fno-stack-protector` might be required in order to recreate the same output.

#### Pointers to pointers

All pointers are passed to a function by value. In order to modify the original pointer, the function must receive a pointer to a pointer.

```c++
void foo(int *bar) {
  printf("            addr of bar: %p\n", &bar);  //        &bar = 0x200
  printf("addr pointing to by bar: %p\n", bar);   // bar (0x200) = NULL
  bar = malloc(sizeof(int));                      // bar (0x200) = 0x123456
                                                  // bar is lost on the stack
}

int main() {
  int *ptr = NULL;
  printf("            addr of p: %p\n", &ptr);  //        &ptr = 0x100
  printf("addr pointing to by p: %p\n", ptr);   // ptr (0x100) = NULL

  foo(ptr);

  printf("             addr of p: %p\n", &ptr);  //        &ptr = 0x100
  printf(" addr pointing to by p: %p\n", ptr);   // ptr (0x100) = 0x000
}
```

In the example above, a copy of the pointer `ptr` is passed to `foo`. `foo.bar` is a new pointer pointing to the same value `NULL`. The address returned by `malloc` is stored on the copy of the pointer: `mem[foo.bar] = malloc`. The original pointer is untouched and the only reference to the malloc'd memory is lost when the function goes out of scope.

Consider the following example using a pointer to a pointer to solve this issue:

```c++
void foo(int **bar) {
                                                        // foo.bar = &main.ptr
  printf("            addr of  bar: %p\n", &bar);       // &foo.bar = 0x3333 (copy)
  printf("addr point to by  bar: %p\n", bar);           // mem[foo.bar] == mem[main.ptr]
  printf("addr point to by *bar: %p\n", *bar);          // mem[mem[main.ptr]] = NULL
  *bar = malloc(sizeof(int));                           // mem[mem[main.ptr]] = 0x12345
                                                        // local foo.bar is gone
  printf("mem addr allocated by malloc: %p\n", *bar);   // 0x12345
}

int main() {
  int *ptr = NULL;
  printf("            addr of p: %p\n", &ptr);          // &ptr = 0x100
  printf("addr pointing to by p: %p\n", ptr);           // ptr (0x100) = NULL

  foo(&ptr);

  printf("             addr of p: %p\n", &ptr);         // &ptr = 0x100
  printf(" addr pointing to by p: %p\n", ptr);          // ptr (0x100) = 0x12345 (malloc)
  free(ptr);
}
```

#### Function pointers

A function pointer is nothing but a regular pointer that holds the address of a function. Declaring pointers to function is similar to declaring a pointer to another type.

```c++
int (*fptr)(int);
int foo(int bar){
    return bar * 2;
}
fptr = foo; // fptr = &foo; (implied by the compiler)
fptr(100); // 200
```

It is convenient to declare a type definition `typedef` for function pointers to make the declarations easier to read.

```c++
typedef int (*fptr)(int);
int foo(int bar){
    return bar * 2;
}
fptr bar = foo;
bar(100); // 200
```

Function pointers can be passed on and returned to from other functions:

```c++
int foo(int bar) { return bar * 2; }
int bar(int baz) { return baz / 2; }

typedef int (*foobar)(int);
int baz(foobar f) { return f(10); }

foobar qux(){
    return foo;
}

int main() {
  baz(foo);
  baz(bar);
  foobar foo = qux();
}
```

Below is an example of an array of function pointers:

```c++
typedef int (*fooptr)(int);

int foo(int x) { return x * 2; }
int bar(int x, int y) { return x + y; }
int main() {
  int (*barptr[8])(int, int) = {NULL};
  fooptr array[8] = {NULL};

  barptr[4] = bar;
  array[2] = foo;

  printf("%d\n", barptr[4](2, 4));  // 6
  printf("%d\n", array[2](4));      // 8
  printf("%d\n", foo == array[2]);      // 1
  printf("%d\n", bar == barptr[4]);      // 1
  printf("%d\n", (void*)barptr[4] == (void*)barptr[4]);      // 0
}
```

Function pointers can also be casted to another type.

```c++
typedef int (*foo)(int);
typedef int (*bar)(int, int);

int baz(int a) { return a * 2; }

int main() {
  foo a = baz;
  bar b = (bar)a;
  int c = ((foo)b)(10);
  printf("%d\n", c);

  typedef void (*base)();
  base baseptr = (void*)baz;
  c = ((foo)baseptr)(10);
  printf("%d\n", c);
}
```

### Pointers and Arrays

An array is a collection of data adjacent to one another in memory having the same type, thus the same size per item.

```c++
int foo[3] = {1, 2, 3};                  // one-dimensional array
int bar[2][3] = {{1, 2, 3}, {1, 2, 3}};  // two-dimensional array
int baz[4][2][3] = {                     // multi-dimensional array
    {
    {
        1,              // a[0][0][0] = 1
        2,              // a[0][0][1] = 2
        3               // a[0][0][2] = 3
    },
        {
        4,              // a[0][1][0] = 4
        5,              // a[0][1][1] = 5
        6               // a[0][1][2] = 6
        },
    },
    {
    {
        7,              // a[1][0][0] = 6
        8,              // a[1][0][1] = 6
        9               // a[1][0][2] = 6
    },
        {10, 11, 12},
    },
    {
        {13, 14, 15},
        {16, 17, 18},
    },
    {
        {19, 20, 21},     // <- baz[3][0][2]
        {22, 23, 24},
    },
};
printf("%d\n", baz[3][0][2]); // 21

int *p = (int *)&baz;
*(p + 20) = 100;
printf("%d\n", baz[3][0][2]); // 100
*(((int *)&baz + 20)) = 120;
printf("%d\n", baz[3][0][2]); // 120
```

Array and pointers generate different machine code, including the results of the `sizeof` function differ too. However, from the example above, pointers can be used to manipulate arrays if used properly.

#### Pointer to to one-dimensional array

Consider the following example:

```c++
int main() {
  char *str[3];                   // 3 pointers to char, adjacent in memory
  str[0] = malloc(sizeof(char));  // allocate 8 bytes of memory
  str[1] = malloc(sizeof(char));  // allocate 8 bytes of memory
  str[2] = NULL;

  *str[0] = 'x';  // &str[0] = 0x100; str[0 = mem[0x12345]] = 'x'
  *str[1] = 'y';  // &str[1] = 0x108; str[1 = mem[0x23456]] = 'y'

  printf("[%p] > %p = 0x%lX\n", &str[0], str[0], *str[0]);
  printf("[%p] > %p = 0x%lX\n", &str[1], str[1], *str[1]);
  printf("[%p] > %p\n", &str[2], str[2]);

  printf("%p\n", *(str + 1));   // mem[0x23456]
  printf("%p\n", **(str + 1));  // 0x79 ('y')
}
```

Every adjacent item in the array is allocated using malloc. Using pointers and pointer to pointers can be useful in order to manipulate the array.

Another example:

```c++
int main() {
int* foo[3];                         // array of pointers to int
foo[0] = (int*)malloc(sizeof(int));  // each array index is a pointer to int
foo[1] = (int*)malloc(sizeof(int));  // each index is allocated 4 bytes
foo[2] = (int*)malloc(sizeof(int));

*foo[0] = 2;  // writing to memory address stored by foo[0] (malloc)
*foo[1] = 4;  // writing to memory address stored by foo[1] (malloc)
*foo[2] = 8;  // writing to memory address stored by foo[2] (malloc)

printf("%p\n", *(foo + 1));   // memory address stored by foo[1]
printf("%d\n", **(foo + 1));  // dereference memory address stored by foo[1]
printf("%d\n", foo[1][0]);    // same thing as **(foo + 1)
}
```

#### Passing multi-dimensional array to functions

```c++
void foo(int (*arr)[3]) {
  int i = 0;
  for (; i < 2; i++) {
    for (int j = 0; j < 3; j++) {
      printf("%d > %d\n", i, arr[i][j]);
    }
  }
}

void bar(int arr[][3]) {
  int i = 0;
  for (; i < 2; i++) {
    for (int j = 0; j < 3; j++) {
      printf("%d > %d\n", i, arr[i][j]);
    }
  }
}

int main() {
  int arr[2][3] = {
      {1, 2, 3},
      {3, 4, 6},
  };

  foo(arr);
  bar(arr);
}
```

#### Allocating memory for multi-dimensional array

```c++
int **arr = (int **)malloc(3 * sizeof(int *));
for (int i = 0; i < 3; i++) {
    arr[i] = (int *)malloc(2 * sizeof(int));
}
```

In this example, we are allocating 3 rows and 2 columns. The rows are allocated first for enough storage to store 3 pointers to integers. Each row will store a memory address large enough to reference an integer.

```
         +----------------------------------
0x10 [0] | int* 8 bytes -> ?
         +----------------------------------
0x20 [1] | int* 8 bytes -> ?
         +----------------------------------
0x30 [2] | int* 8 bytes -> ?
         +----------------------------------
```

The columns are then created with the use of malloc with enough space to store 2 integers.

```
         +------------------------------------------
0x10 [0] | int* 8 bytes -> malloc 0x100
         +------------------------------------------
0x20 [1] | int* 8 bytes -> malloc 0x200
         +------------------------------------------
0x30 [2] | int* 8 bytes -> malloc 0x300
         +------------------------------------------

         +------------------------------------------
0x100    | malloc'd 8 bytes (2 int)
         +------------------------------------------
0x200    | malloc'd 8 bytes (2 int)
         +------------------------------------------
0x300    | malloc'd 8 bytes (2 int)
         +------------------------------------------
```

It's also common to allocate contiguous memory for two-dimensional arrays:

```c++
int rows = 3;
int cols = 2;
int **arr = (int **)malloc(rows * sizeof(int *));
arr[0] = (int *)malloc(cols * rows * sizeof(int));
for (int i = 0; i < rows; i++) {
    arr[i] = arr[0] + i * cols;
}
```

Array is declared a pointer to pointer to int, with enough space to hold 3 pointers to int, for a total of 24 bytes. Every index access is has an offset of 8 bytes each (the size of a pointer to int on 64-bit CPU). Subsequently, the index at position 0 (arr[0]), meaning, the first 8 bytes (pointer to int) are used to store the address returned by malloc, which is allocating 18 bytes in total.

```
arr is an array of 3 pointers to int:

            arr
           +------------------------------------------+
    0x1000 | 0x7000 - malloc(3 ptr to int) = 24 bytes |
           +------------------------------------------+

            memory area pointing by arr (0x1000)
           +------------------------------------------+
    0x7000 |    8 bytes  |   8 bytes   |   8 bytes    |
           +------------------------------------------+
                arr[0]       arr[1]        arr[2]


The first pointer to int is used to store the new address of malloc for 6 ints (18 bytes), enough to store all elements 2*3:

            arr[0] = 6 ints (0x7000)
           +------------------------------------------+
    arr[0] | 0x9000 -> malloc(6 int) = 18 bytes       |
           +------------------------------------------+

            memory area pointing by arr[0] (0x7000)
            arr[0] + 1 = arr[0] + sizeof(*arr[0]) = 4 bytes
           +-----------------------------------------------+
    0x9000 |   4   |   4   |   4   |   4   |   4   |   4   |
           +-----------------------------------------------+
           |   0   |   1   |   2   |   3   |   4   |   5   |
           +-----------------------------------------------+
             [0][1]  [0][2]  [0][3]  [0][4]  [0][5]   [0][6]


i=0; cols=2;
arr[0] = (arr[0] + ((0*4) * 2)) = 0x9000
arr[1] = (arr[0] + ((1*4) * 2)) = 0x9008
arr[2] = (arr[0] + ((2*4) * 2)) = 0x9010

            memory area pointing by arr
           +------------------------------------------+
    0x7000 |    0x9000   |    0x9008   |    0x9010    |
           +------------------------------------------+
                arr[0]       arr[1]        arr[2]
                ___|           |             |
               |               |             '--.
               |               |                |
           +---'---------------'----------------'----------+
    0x9000 | 0x9000| 0x9004| 0x9008| 0x900C| 0x9010| 0x9014|
           +-----------------------------------------------+
           |   0   |   1   |   2   |   3   |   4   |   5   |
           +-----------------------------------------------+
             [0][1]  [0][2]  [0][3]  [0][4]  [0][5]   [0][6]

```

Setting values of the array would result in the following:

```c++
arr[0][0] = 1;
arr[0][1] = 2;

arr[1][0] = 3;
arr[1][1] = 4;

arr[2][0] = 5;
arr[2][1] = 6;
```

```
            memory area pointing by arr
           +-----------------------------------------------+
    0x7000 |    0x9000     |    0x9008   |       0x9010    |
           +-----------------------------------------------+
                    |             |                |
           +--------'-------------'----------------'-------+
    0x9000 | 0x9000| 0x9004| 0x9008| 0x900C| 0x9010| 0x9014|
           +-----------------------------------------------+
           |   1      2    |   3      4   |    5      6    |
           +-----------------------------------------------+
             [0][1]  [0][2]  [0][3]  [0][4]  [0][5]   [0][6]
                       |              |                |
arr[0][1] == 2 ---------              |                |
arr[1][1] == 4 ------------------------                |
arr[2][1] == 6 -----------------------------------------

  *(*(arr+1))   = *(0x7000+0x08) = 0x9008 -> 3
     **arr+2    = *(0x7000+0x10) = 0x9010 -> 5
*((*(arr+2))+1) = *(0x7000+0x10) = 0x9010+0x04 = 0x9014 -> 6
      **arr + 4 =  (*(*arr)) + 4 = (*0x7000)+4 = 0x9000+ 4  = 0x9010 = 5
```

```c++
  printf("%d\n", *(arr[0]+1));              // 2
  printf("%d\n", **(arr+1));                // 3
  printf("%d\n", *(*(arr + 1))));           // 3
  printf("%d\n", *((*arr+2)+1));            // 3
  printf("%d\n", arr[0][1]);                // 3
  printf("%d\n", **arr + 4);                // 5

```

#### Jagged arrays and pointers

Multi-dimensional arrays can be instantiated with different number of column for each row. This is typically done by the use of compound literals:

```c++
int(*(arr[])) = {
    (int[]){1,2,3},
    (int[]){4,5,6},
};
```

#### Array and pointers to string

```c++
char* gStr = "Foo";
char gArr[] = "Foo";

int foo() {
  char* lStr = "Foo";
  char lArr[] = "Foo";
  printf("%p > %p = %s\n", &lStr, lStr, lStr); // 0x7ffe5803b2f8 > 0x560f990e2004 = Foo
  printf("%p > %p = %s\n", &lArr, lArr, lArr); // 0x7ffe5803b2f4 > 0x7ffe5803b2f4 = Foo
}

int main() {
  printf("%p > %p = %s\n", &gStr, gStr, gStr); // 0x560f990e4040 > 0x560f990e2004 = Foo
  printf("%p > %p = %s\n", &gArr, gArr, gArr); // 0x560f990e4038 > 0x560f990e4038 = Foo

  static char* sStr = "Foo";
  static char sArr[] = "Foo";
  printf("%p > %p = %s\n", &sStr, sStr, sStr); // 0x560f990e4048 > 0x560f990e2004 = Foo
  printf("%p > %p = %s\n", &sArr, sArr, sArr); // 0x560f990e403c > 0x560f990e403c = Foo

  char* lStr = "Foo";
  char lArr[] = "Foo";
  printf("%p > %p = %s\n", &lStr, lStr, lStr); // 0x7ffe5803b318 > 0x560f990e2004 = Foo
  printf("%p > %p = %s\n", &lArr, lArr, lArr); // 0x7ffe5803b314 > 0x7ffe5803b314 = Foo

  foo();
}
```

```
  mem 560f990e2000-560f990e3000 (r--p permissions)

               +------------------------------------------------------------+
0x560f990e2004 |     1 byte    |    1 byte   |     1 byte    |    1 byte    |
            \__'------------------------------------------------------------+
          gStr |        F      |       o     |       o       |      \0      |
               +------------------------------------------------------------+
                0x560f990e2004 0x560f990e2005 0x560f990e2006  0x560f990e2007
                    gStr[0]        gStr[1]        gStr[2]         gStr[3]


  mem 560f990e4000-560f990e5000 (rw-p permissions)

               +------------------------------------------------------------+
0x560f990e403c |     4 bytes       |
            \__'        :          |
               |     "Foo\0"       |
               +------------------------------------------------------------+
                   0x560f990e403c
                      sArr[0]

               +------------------------------------------------------------+
0x560f990e4038 |     8 bytes       |      8 bytes      |       8 bytes      |
            \__'        :          |         :         |          :         |
               |      "Foo\0"      |   0x560f990e2004  |   0x560f990e2004   |
               +------------------------------------------------------------+
                   0x560f990e4038      0x560f990e4040     0x560f990e4048
                      gArr[0]             gStr[0]             sStr[0]
                                             |                   |
                                            gStr                gStr

  mem 7ffe5801d000-7ffe5803e000 (rw-p permissions) [stack memory]


               +------------------------------------------------------------+
0x7ffe5803b2f4 |     8 bytes       |      8 bytes      |
            \__'        :          |         :         |
               |      "Foo\0"      |   0x560f990e2004  |
               +------------------------------------------------------------+
                   0x7ffe5803b2f4      0x7ffe5803b2f8
                    foo.lArr[0]          foo.lStr[0] -> gStr

               +------------------------------------------------------------+
0x7ffe5803b314 |     8 bytes       |      8 bytes      |
            \__'        :          |         :         |
               |      "Foo\0"      |   0x560f990e2004  |
               +------------------------------------------------------------+
                   0x7ffe5803b314      0x7ffe5803b318
                   main.lArr[0]          main.lStr[0]  -> gStr


```

A quick peek into the maps `cat/proc/{PID}/maps` will reveal the position of the strings in the program's memory:

```
560f990e0000-560f990e1000 r--p 00000000 08:03 8656113                    [program memory]
560f990e1000-560f990e2000 r-xp 00001000 08:03 8656113                    [program memory]
560f990e2000-560f990e3000 r--p 00002000 08:03 8656113                    [program memory]
560f990e3000-560f990e4000 r--p 00002000 08:03 8656113                    [program memory]
560f990e4000-560f990e5000 rw-p 00003000 08:03 8656113                    [program memory]
7ffe5801d000-7ffe5803e000 rw-p 00000000 00:00 0                          [stack]
```

Based on the memory information we have, we can see that all global, static and local pointer to arrays declared are pointing to the same address in memory. The memory they point to is read-only memory, thus they are immutable blocks of memory. All arrays point to their own place in memory on the stack, making them mutable. This layout is implementation-dependend so it is just used as an example.

Another example showing the difference between arrays and pointers. Arrays are not pointers. When an array is used as a value, the actual address of the first element is returned. Thus `arr` is equal to `arr[0]`.

```c++
int foo(char *str) { printf("foo %p > %p = %s\n", &str, str, str); }

int main() {
  char *str = (char *)malloc(100);
  char arr[100] = "foobar";
  strcpy(str, "foobar");
  printf("%p > %p = %s\n", &str, str, str);
  printf("%p > %p = %s\n", &arr, arr, arr);

  foo(str);
  foo(arr);
  foo((char *)&arr);
  foo(&arr[0]);
  foo(&str[0]);
}
```

#### Passing buffers to parameters

```c++
char* fmt(char* buf, size_t size, const char* name) {
  printf("   foo: %p > %p = %p\n", &buf, buf, buf);     // 0x200 -> null = null
  char* fmt = "Hello %s!";
  if (buf == NULL) {
    size = (strlen(fmt) - 2) + strlen(name) + 1;
    buf = (char*)malloc(size);
    printf("   foo: malloc -> %p\n", buf);              // 0x200 -> 0x12345
  }
  snprintf(buf, size, fmt, name);
  return buf;                                           // ret 0x12345
}

int main() {
  char* buf = NULL;
  printf("before: %p > %p = %p\n", &buf, buf, buf);     // 0x100 -> null = null
  buf = fmt(buf, 10, "Foo");
  printf(" after: %p > %p = %s\n", &buf, buf, buf);     // 0x100 -> 0x12345 = Hello Foo!
  printf("%s\n", buf);
}
```

#### char\*\* argv vs char\* argv[]

It's quite common to see the two variants in the main function of a C program, `char** argv` and `char* argv[]`. Both are identical.

```
                   8 bytes                    dynamic length
                     |                              |
               +------------+           +------------------------+
0x1000 argv[0] |   0x7000   |    0x7000 |   10 bytes (10 chars)  |
               +------------+           +------------------------+
0x1008 argv[1] |   0x7050   |    0x7050 |   15 bytes (15 chars)  |
               +------------+           +------------------------+
0x1010 argv[2] |   0x7100   |    0x7100 |   25 bytes (25 chars)  |
               +------------+           +------------------------+

argv       == 0x1000
argv[0]    == *(argv+0*0x8) == 0x7000
argv[1]    == *(argv+1*0x8) == 0x7050
argv[2]    == *(argv+2*0x8) == 0x7100
**argv     == *(*argv)      == *(0x7000 + 0*0x1) == 0x7000[0] == 0x7000
*(*argv+1) == *(*argv)      == *(0x7000 + 1*0x1) == 0x7000[1] == 0x7001
**argv+1   == (*(*argv))+1  == (*(0x7000))+1     == inc first char to +1

**argv+1    = *0x7000+1 = 'F' + 1 = 0x70+1 = 0x71 = 'G'
**argv      = *(0x7000) = 0x7000
argv[2]     = *(argv+2) = *(0x1000+2*0x8) = *(0x1010) = 0x7100
argv[2][1]  = *(*(argv+2))+1 = *(*(0x1010))+1 = *(0x7100)+1 = 0x7101

       +-----------------------------------------------------+
0x7000 | 1 byte | 1 byte | 1 byte | 1 byte | 1 byte | 1 byte |
       |-----------------------------------------------------|
       |    F   |    o   |    o   |    b   |    a   |    r   |
       +-----------------------------------------------------+
         0x7000   0x7001   0x7002   0x7003   0x7004   0x7005

```

#### Returning strings

A function can return the address of a literal string, static literal, malloc'd string or local string.

```c++
char* foo(int code){
    switch(code){
        case 100: return "Literal string 1";
        case 200: return "Literal string 2";
        default: return "Default literal string";
    }
}
char* bar(int code){
    static char* str[] = {
        "Static string 1",
        "Static string 2",
        "Default static string",
    };
        switch(code){
        case 100: return str[0];
        case 200: return str[1];
        default: return str[2];
    }
}
```

Both of the above function would return the address of the location of the strings within the program's memory. Recalling the same function with the same code, would return the exact same address.

It's also possible to return the address of a malloc'd memory block as follows:

```c++
char* foo() {
  char* buf = malloc(3);
  memset(buf, 0x61, 1);
  memset(buf + 1, 0x62, 1);
  buf[3] = 0;
  return buf;
}

int main(int argc, char* argv[]) {
  char* buf[2];
  buf[0] = malloc(5);
  memset(buf[0], 0x31, 1);
  memset(buf[0] + 1, 0x32, 1);
  memset(buf[0] + 2, 0x33, 1);
  memset(buf[0] + 3, 0x34, 1);
  buf[0][5] = 0;

  buf[1] = foo();
  printf("%s\n", buf[0], buf[0]);  // 1234
  printf("%s\n", buf[1], buf[1]);  // ab
}
```

Returning a local buffer from a function will result in undefined behaviour, due to the stack memory being overwritten on subsequent function calls.

```c++
char* foo(){
    char str[5];
    strcpy(str,"test");
    printf("%p > %s\n", &str, str);
    return str;
}
int main(){
    char *str = foo();
    printf("%p > %s\n", &str, str);
}
```

Some compilers such as GCC will return a null pointer instead of a stack address. This is because this behaviour is not defined by the C standard and the compiler is generating the most efficient set of instructions so that your program will fail as soon as possible.

Here below is another example using pointer to strings:

```c++
char* foo(const char* buf) {
  char* tmp = (char*)malloc(strlen(buf) + 1);
  char* p = tmp;
  while (*buf != 0) {
    *tmp++ = tolower(*buf++);
  }
  *tmp = 0;
  return p;
}
int main() {
  char* str;
  str = foo("FooBar");
  printf("%p > %s\n", str, str);
}
```
