# Stack and Heap Memory

Here's a deep dive into stack and heap memory — two of the most fundamental concepts in how programs manage data.

---

### The core idea

Every running program needs memory to store its data. But not all data is the same — some data has a predictable lifespan (you know when it'll be created and destroyed), and some data lives and dies unpredictably at runtime. This is why computers use two very different memory regions: the **stack** and the **heap**.

---

### The Stack

The stack is a region of memory that operates exactly like the data structure it's named after: **last in, first out (LIFO)**. It's fast, compact, and tightly managed by the CPU itself.

Every time you call a function, the program pushes a **stack frame** onto the top of the stack. That frame holds:

- The function's local variables
- Its parameters
- The return address (where to go back when the function finishes)

When the function returns, its frame is simply _popped off_ — memory is freed instantly, with zero overhead. No garbage collection, no manual cleanup.

**Example in C:** 

c

```c
void greet(int age) {
    char name[20] = "Alice"; // stored on the stack
    // 'age' and 'name' live here until greet() returns
}
```

When `greet()` is called, a frame is pushed. When it returns, the frame is popped. `name` and `age` are gone.

**Key traits of the stack:**

- Extremely fast (just moving a pointer)
- Limited in size (typically 1–8 MB)
- Memory is automatically managed
- Data must have a known, fixed size at compile time
- Variables only live as long as the function they belong to

---

### The Heap

The heap is a large pool of memory used for data whose size or lifespan isn't known at compile time. It's flexible, but slower and requires manual (or garbage-collected) management.

When you say "I need a list that could hold 5 or 5 million items", that goes on the heap. You request a chunk at runtime (`malloc` in C, `new` in C++/Java, or automatically in Python/JS), and the system finds a free block and gives you a pointer to it.

**Example in C:**

c

```c
int* numbers = malloc(100 * sizeof(int)); // allocates on the heap
// ... use numbers ...
free(numbers); // you must free it yourself!
```

In managed languages like Python, Java, or JavaScript, a **garbage collector** does the cleanup for you — but it comes with overhead.

**Key traits of the heap:**

- Large (limited only by available RAM)
- Slower to allocate (finding free space is non-trivial)
- Flexible — data can be any size, created/destroyed at any time
- Data persists beyond the function that created it
- Prone to memory leaks if not properly managed

Side by side: a visual comparison
![[Screenshot from 2026-04-29 17-51-33.png]]

Notice the pointer arrow in the diagram: the `main()` stack frame holds a pointer (`ptr`) that references a block on the heap. This is the classic relationship — the _address_ lives on the stack, but the _data itself_ lives on the heap.

### A concrete example: walking through memory

Let's trace what happens when this C code runs:

```c
int main() {
    int x = 10;                        // stack: main's frame
    int* arr = malloc(5 * sizeof(int)); // heap: 20 bytes allocated
    arr[0] = 42;

    greet(x);                          // new frame pushed
    free(arr);                         // heap block released
    return 0;
}

void greet(int age) {
    char name[20] = "Alice";           // stack: greet's frame
    // frame is popped when greet() returns
}
```

Here's how the memory looks at the moment `greet()` is executing:


![[Screenshot from 2026-04-29 17-51-57.png]]

---

### What can go wrong?

**Stack overflow** — If functions call each other too deeply (or infinitely via bad recursion), the stack runs out of space. The OS kills the program. This is literally what a "stack overflow" is.

**Memory leak** — If you allocate on the heap and never free it (in languages without a GC), the memory is never returned to the OS. Long-running programs that leak slowly grind to a halt.

**Dangling pointer** — If you free a heap block but still have a pointer pointing to it, reading through that pointer is undefined behavior — you might read garbage, or crash.

**Heap fragmentation** — After many alloc/free cycles, the heap can end up with lots of small scattered free blocks. A new large allocation might fail even if there's theoretically enough total free memory.

---

### How high-level languages handle this

|Language|Stack|Heap|
|---|---|---|
|C / C++|Automatic|Manual (`malloc`/`free`, `new`/`delete`)|
|Java|Automatic|Garbage collected (GC)|
|Python|Automatic|Reference counting + GC|
|Rust|Automatic|Ownership system — no GC, no leaks|
|JavaScript|Automatic|Garbage collected|

Rust is worth a special mention: it tracks ownership at _compile time_, so it gets the performance of manual management without the danger of leaks or dangling pointers.

---

### The one-sentence summary

The **stack** is fast, automatic, and limited — great for short-lived function-local data. The **heap** is flexible and large — essential for data whose size or lifetime can't be known until the program is running. Most programs use both constantly, in tandem.