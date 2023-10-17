# Data representation

## Integers

* Integers are signed variables using 2's complement: -2^(n-1) to 2^(n-1) - 1
* C allows for variables to either be declared as signed or unsigned
* Integers are declared signed by default

# C programming

## Compiling

* C is a compiled language
  * Code is genereally converte into machine code
  * Java, by contrast, indirectly converts to machine code using byte code
 
* C compiler will typically convert *.c source files into an intermediate *.o object file  

# x86 ASM

* Assmely: Human-readable representation of machine code
* Machine code: what a computer actually runs

* Machine language instructions are the patterns of bits that a process reads to know what to do
* Assembly language is human-readble textual representation of machine language

* Programs writen in C are generally translated to assembly and then into machine code (Remeber, C is a compiled language)

## x86 Basics

* An ISA is the interface that a CPU presents to the programmer
* The ISA defines:
  * What the CPU can do
  * What registers it has
  * The machine language

* The ISA does not define:
  * How to design the hardwawre

### RISC
* RISC: "Reduced Instruction Set Computer
* ISA designed to make it easy to
  * Build the CPU hardware
  * make that hardware run fast
  * write compilers that make machine code
* A small number of instructions
* Instructions are very simple

### CISC
* CISC: Complex Instruction Set Computer
* ISA designed for humans to write asm
* Lost of isntructions and complex (multi-step) instructions to shorten and simplify programs
* x86 is CISC

### x86 Registers (general)

* Like MIPS, there are a set of general-purpose registers
* Unlike MIPS, you can refer to parts of each register (partial registers)

![image](https://github.com/clester331/0449-study/assets/122314614/73e8f928-3cbd-46fd-95a4-a9075d15f90b)

* x86 Registers (specialized)
* There are also registers that you cannot directly interact with
* Like MIPS, x86 has a program counter ( %rip )

* There is also a FLAGS status register which has information about the CPU state after an instruction is completed

### Instruction Types

* In x86 (CISC), you generally can havt instruction refer to data anywhere it is:

```assembly
mov %rbx, %rax   ; rax = rbx
mov $0x100, %rax   ; rax = 0x100  (immediats prefixed by $)
mov (ptr), %rax   ; rax = *ptr  (Memory load within parethases)
mov %rax, (ptr)  ; *ptr = rax  (Memory store)
lea (ptr), %rax
mov %rax, 4(%rax) ; *(ptr + 4) = rax (Displacement can be -4, etc)
```

### Complex Addressing

* In MIPS, you would carefully craft the set of instructions necessary to interface with an array. (RISC)
* In x86, you can do alot with just a single instruction. (CISC)

```assembly
.data
arr: .int 1, -2, , -4, 11

.text

.global _start

_start:
  lea (arr), %rbx  ; rbx = addr to arr
  mov $2, %rdi  ; rdi = 2
  mov (%rbx, %rdi, 4)  ; rdi = arr[2]
  lea (%rbx, %rdi, 4), %rdi  ; rdi = &arr[2]
```

### x86 Instruction Qualifiers

* In x86 you can operate on any part of a register (64b, 32b, 16b)
  * mov -> assembler figures it out
  * movq -> "quad word" = 64 bits
  * movl -> "long word" = 32 bits
  * movw -> "word" = 16 bits
 
### Hello World in x86
```assembly
; Assumes Linux system calls
.data
db: .asciz "Hello, world!\n"

.text

.global _start

_start:
# write(1, db, 14)
mov $1, %rax  ; system call 1 is write
mov $1, %rdi  ; file handle 1 is stdout
lea (db), %rsi  ; address of string
mov $14, %rdx  ; number of bytes
syscall  ; invoke OS to print

# exit(0)
mov $60, %rax  ; system call 60 is exit
xor %rdi, %rdi  ; we want return code 0
syscall  ; invoke OS to exit
```

### Computing absolute value in x86

```assembly
push %rbp    ; push base pointer onto stack
mov %rsp, %rbp   ; move out stack pointer onto the base pointer
mov %edi,-0x4(%rbp)   ; argument 1 (x)
cmpl $0x0,-0x4(%rbp)   ; compare if (x < 0)
jns 1149 <abs+0x10>   ; if not jump to 1149
negl -0x4(%rbp)   ; negate x (x = -x)
mov -0x4(%rbp), %eax   ; (1149) move x into eax
pop %rbp   ; pop base pointer
retq   ; return x (%rax(
```

# Memory and Pointers

## Memory Model

* Memory is a continous series of bits
  * Can be divided into bytes or words
* Memory is byte-addressable, which means individual bytes can be read
* With byte-addressable memory, each and every byte (8 bits) has its own unique address
  * Addresses start at 0, second byte is at address 1, and increases upwards as you add more data

### Code

* Code has a few known properties
  * It likely should not change
  * It must be loaded before a program can start

### Static Data

* Static data does not change (name refers to size/location_
* It generally must be loaded before a program starts

### The Stack

* The stack is a space for temporary data
  * Holds local variables and function arguments
  * Allocated when functions are called, freed on return
  * Grows "downwards" (Allocates lower addresses)
 
### The Heap

* The Heap is the dynamic data section
  * Managing this memory can be very complex
  * No garbage collection provided
 
## Arrays

* An array is simply a continuous span of memory
  * All elements are the same type
* You can declare an array on the stack

* Variables declared on the stack are temporary
* All arrays can be considered pointers, but addresses to the stack are not reliable

```C
int powers_of_two(void) {
  int array[5] = {1, 2, 4, 8, 16}; // stack allocation
  return array;
} // stack deallocation
```

```C
void powers_of_two(int* buffer, size_t length) { // Arrays don't store length. Must be passed in
                  // Pass in array by referencec
  int value = 1;
  for(int i = 0; i < length; i++) {
    buffer[i] = vlaue; // Works just like an array
    value *= 2;
  }
}

void main(void) {
  int buffer[10] = {0}; // Now it is on the stack of main
  powers_of_two(buffer, 10); // Now we pass a reference to the array in main
  for(int i = 0; i < 10; i++) {
    printf("%d\n", buffer[i]);
  }
}
```

## Pointers

* A pointer is a specific variable type that holds a memory address
* You can create a pointer that points to any address in memory
* Furthermore, you can tell what type of data is should interpret that memory to be: just place the * at the end
```C
int* my_integer;
float* fl;
struct Song* song_type;
```

![image](https://github.com/clester331/0449-study/assets/122314614/fc727b6b-1839-492f-87f7-752b4b75fc22)

* If we have a variable that holds and address, normal operations change the address not the value referenced by the pointer
* We can use the dereference oeprator (*)

```C
int* _data = 0x00800000; // Arbitrary address
_data = 0xffffffff; // Reassigns the address
*_data = 42; // Reassigns the value

int data = 0xc0de; // Initializes a new variable
data = *_data; // Assignes value from pointer
```

* We can also pull out the address to data and assign that to a pointer
* We use the reference operator (&)

```C
int* _data = 0x00800000; // This address is arbitrary

int data = 0xc0de; // Initializes a new variable
_data = &data; // Assigns address to pointer
*_data = 42; // Assigns 42 to memory
printf("%d\n", data); // prints 42
```

```C
int data = 42;
int* dataptr = &data; // store address of data

// pointer to a pointer of an int:
int** dataptrptr = &dataptr; // store address of dataptr

// dereference dataptrptr... then dereference that...
*(*dataptrptr) = -64; // store VALUE into 'data'
```

### Back to the heap

* The heap is the dynamic data section
  * You interact with the heap entirely with pointers
  * malloc returns the address to the heap with at east the number of bytes requested. Or NULL on error 

### Pointer Arithmetic

* Ideally, pointers should align to their values in memory
  * Goal, incrementing an int pointer should go to the next int in memory
* Therefore, pointer sum is scaled to the element size
  * Multipliation and other operators are undefined and result in a compiler error

```C
int* ptr = (int*)0x400; // Arbitrary value
ptr++ // ptr is 0x404 // (assuming 32-bit int)
ptr *= 2; // Error: multiplication not valid
```

### Arithmetic with pointers

```C
type *ptr; // Some pointer to type
ptr = ptr + n // What really happens is ptr = ptr + n*sizeof(type)

int *ptr = 4; // Some pointer to int
ptr = ptr + 1 // What really happens is ptr = ptr + 1*sizeof(int)
// ptr = 8

• char *ptr = 12; // Some pointer to char
ptr = ptr + 2 // What really happens is ptr = ptr + 2*sizeof(char)
// ptr = 14

• long*** *ptr = 0x18; // Some pointer to long*** wtf? Maybe a 3D matrix :D
ptr = ptr + 2 // What really happens is ptr = ptr + 2*sizeof(long***)
// ptr = 0x28
```

## Strings

* String are arrays, and as such, inherit all their limitations/issues.
  * The size is not stores
  * They are essentially just pointers to memory
* Text is represented as an array of char elements

* Arrays in C are just pointers and as such, do not store their lengths
  * Simply are continuous sections of memory
* Two common ways to express length
  * Storing the length alongsied the array
  * Storing a special value within the array to ark the end. (sentinel value)
 
* Strings in C commonly employ a sentinel value
  * Such a value must be something considered invalid for actual data
  * To find the length. Simply search for the value (O(n) time)

### The String Literal

* String literals in C are char pointers (char*)
* The contents of the literal are read-only (immutable), so its a: const char*

```C
void main(void) {
  const char* my_string = "Hello World";
  int legnth = strlen(my_string);
  printf("length: %d\n", length);
  printf("sentinel: %x\n", my_string[length]);
}
```

### Some string functions

```C
#include <string.h>
strcpy(char* dst, const char* src) Copies src to dst overwriting dst.
strncpy(char* dst, const char* src, size_t max) //Copies up to ‘max’ to dst.
strcat(char* dst, const char* src) //Copies string from src to end of dst.
strncat(char* dst, const char* src, size_t max) //Copies up to ‘max’ to end of dst.
strcmp(const char* a, const char* b) //Returns difference between strings. (0 if equal)
strncmp(const char* a, const char* b, size_t max) //Compares up to ‘max’ bytes.
```

# Files & I/O

### Opening Files

* File operations: Open

```C
FILE *fopen(const char *path, const char *mode);
```

* The path is the location on the computer
* The mode selects which operations can be done

![image](https://github.com/clester331/0449-study/assets/122314614/88a34d3f-7d34-404b-8656-e0da66c1c857)

* What it returns:
  * FILE * -> A file stream that can be used to access it's contents
  * NULL on error

### Closing Files
 
* File operations: close
```C
int fclose (FILE *stream);
```

* What it returns:
  * 0 on success
  * EOF (end of file) on error
 
### Reading Files

* File operations: read
```C
size_t fread(void *ptr, size_t size, size_t n_items, FILE *stream);
```

* Arguments:
  * void *ptr -> The memory address where the data is stored
  * size_t size -> The size of each item being read
  * size_t n_items -> The number of items to read
  * FILE *stream -> The file stream to read from

* What it returns
 * Number of items read
 * Something went wrong if zero or less than n_items returned

### Writing binary files

* File operations: write
```C
size_t fread(void *ptr, size_t size, size_t n_items, FILE *stream);
```

* Arguments:
  * void *ptr -> The memory address where the data is stored
  * size_t size -> The size of each item being read
  * size_t n_items -> The number of items to read
  * FILE *stream -> The file stream to write to

* What it returns
  * The number of items written
  * Something went wrong if less than n_items is returned
 
### What happens on read/write

* WHen a file is open, a cursor is initialized to the beginning of the file
* On read/write, the cursor is moved forwards and the data read/writen

### Controlling the cursor

* File opearations: move the cursor
```C
int fseek(FILE* stream, long offset, int whence)
```

* Arguments:
  * FILE *stream -> The file stream to read from
  * long offset -> the offset to move the cursor
  * int whence -> Where to move the cursor from (use constants)
   * Option 1 - SEEK_SET -> Move relative to the beginning of the file
   * Option 2 - SEEK_CUR -> Move relative to the current cursor position
   * Option 3 - SEEK_END -> Move relative to the end of the file
 
* What it returs:
  * 0 if successful
  * -1 if error

### Memory access and pointer arithmetic

![image](https://github.com/clester331/0449-study/assets/122314614/70bcf10b-960d-4e6a-8f5b-03ca46d4094e)

```C
short *sp; // 16b
int *ip = 0x3C // 32b
// pointers are 64b
```

* ip = **bf**
* (ip - 2) = *(ip - sizeof(int) * 2) = (ip - 8) = **6a**
* ip-2 = 0x3C - 2 = **0x3A**
* sp = ip -> *(sp - 2) = **error** (no cast)
* (char **)ip - 2 = **0000 0000 0000 003B

# Memory Management

* When small gaps interfere with allocation, this is called fragmentation
* When you allocate a lot of small things and free every other one, and then attempt to allocate a bigger thing:
	*	Although there is technically enough memory, there is no continuous space, and our naive malloc will fail
*	We can move things around with a defragmentation process/algorithm
	* Pointers refering to data within a block must be updated
 	* When block move, pointers to anything within them must be updated	 	

## Linked Lists

* A linked list is a non-continuous data structure representing an ordered list
* Each item in the linked list is represented by metadata called a node
	* Indirectly refers to the actual data

* Creation of a list occurs when one allocates a single node and tracks it in a pointer. This is the head of our list (first element)

```C
typedef struct _Node {
	struct _Node* next;
	char* data;
}

Node* list = (Node*)malloc(siezof(Node));
list->next = NULL; // NULL is our end-of-list marker
list->data = NULL; // Allocate/copy the data you want
```

* If we want to append an item we can add a node anywhere

```C
void append(Node* tail, const char* value) {
	Node* node = (Node*)malloc(sizeof(Node));
	node->next = NULL; // The new end of our list
	tail->next = node; // We attach this node to the old last node
	node->data = (char*)malloc(strnlen(value, 100) + 1);
	strncpy(node->data, value, 100);
}
```

* And if we update our append to take any node

```C
void linkedListAppend(Node* curNode, const char* value) {
	Node* node = (Node*)malloc(sizeof(Node));
	node->next = curNode->next;
	curNode->next = node;
	node->data = (char*)malloc(strnlen(value, 100) + 1);
	strcpy(node->data, value);
}
```

* The appeand always allocates the same amount
* it always copies the same amount

### Traversal

* Accesing an array element is simple:
	* arr[42] is the same as *(arr + 42) becuase its location is well known

### Malloc Implementations

* First fit: start at lowest address, find first available section
	* Fast but small blocks clog up the works
* Next-fit: Do "First-fit" but start where we last allocated
	* Fast and spreads small blocks around better
* Best-fit: laboriously look for the smallest available section to divide up
	* Slow, but limits fragmentation   	  
