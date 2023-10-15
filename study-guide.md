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
