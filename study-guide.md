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

*ip = **bf**
*(ip - 2) = *(ip - sizeof(int) * 2) = (ip - 8) = **6a**
*ip-2 = 0x3C - 2 = **0x3A**
sp = ip -> *(sp - 2) = **error** (no cast)
(char **)ip - 2 = **0000 0000 0000 003B
