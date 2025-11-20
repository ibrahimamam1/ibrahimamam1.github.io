# Some title 
In my quest of understanding how systems work i built a very basic and simple memory allocator. If you are a beginner this might sound like a real tough thing to do but it is not really.

A memory allocator is simpy a program that manages the internal address space(or memory layout) of a process. Huh... Addres space? What is that? What we call address space is just a clever abstraction of physical memory that makes it easier for us to program. It is basically how a program sees memory when it is running. Every running procees has it's own address space(representation of memory) that belongs to it alone.

A process address space is divided into disctinct sections: the code, data, bss, stack and heap. You might have already heard of the stack and heap if you ever took a programming course.The stack is where local variables, functions return addresses and more are stored. The heap is the dynamically allocatable part of the memory and where dynamically allocated objects are stored. To know more about the code, data and bss sections go [here]{https://mirzafahad.github.io/2021-05-08-text-data-bss/}.

## How Memory Allocators Work
Memory Allocators are responsible for managing the heap. This is done by building an abstraction of the heap as a sequence of blocks of arbitrary size. The Memory allocator can then reserve blocks of any given size for your program, make them bigger or smaller, request more blocks from the os etc to meet the demands of your program.

## Building the Abstraction: The block
There are many ways one can represent a block. In my cases i decided to keep it as simple as possible:
```c 
 typedef struct mem_block{
    bool free;
    size_t size;
    struct mem_block* next = nullptr;
}mem_block_t;

#define MEM_BLOCK_SIZE sizeof(mem_block_t)   
```
Each block has a free attribute which represents whether it has been allocated already or not, size attribute holds the address of the next block after it. 
If for example the program requests a 64 byte block, a `mem_block_t` object will be instantiated and free will be set to false, size will be set to 64 and the address of the next block will be set to `&this_block + 64`. The address of this block is then returned to the user and all the space(64 bytes) between the two blogs is reserved for use.

## Iniatialize memory pool
Before we have any space on the heap to manage we need to request some from the OS. To do that we can either provide a function for the user to call or do what we call lazy initialisation. Lazy initialisation is an elegant technique where we request for the resource the first time the user calls our allocate function. This provide the advantages that we only request the resource when the program really needs to use the heap allocated memory and hence it is not wasted. The other advantage is that now the user does not have to worry about these details, they merely have to call alocate and free. 

Here is the code for lazy initialisation:
```c
#define INITIAL_BLOCK_SIZE  1024
#define STARTING_SIZE INITIAL_BLOCK_SIZE+MEM_BLOCK_SIZE


mem_block_t* head = nullptr;

void init_mem_pool() {
    head = (mem_block_t*) sbrk(STARTING_SIZE);
    if(head == (void*) -1) {
        head = nullptr; // sbrk failed
        return;
    }
    head->size = INITIAL_BLOCK_SIZE;
    head->free = true;
    head->next = nullptr;
}

void* mem_alloc(size_t size){
    if(head == nullptr) {
        init_mem_pool();
    }
    ...
}

```

The pointer `head` is initialised to nullptr so the very first time the user calls `mem_alloc` the if statement evaluates to true and our `init_mem_pool` function is called. This function uses the `sbrk` system call to change the ending address of the heap. To know more about `sbrk` check the {man pages}[https://linux.die.net/man/2/sbrk].

We request an initial block of 1024 bytes + `MEM_BLOCK_SIZE` to hold the metadata(size, free and next ptr). We then set all the attribute values and that's it.

## Allocating Memory
At this point we have defined what a block is and how to initialise our memory pool. The start state is one big block of 1024 bytes that is free. To allocate blocks that are various strategies namely: first fit, best fit and next fit. The first fit strategy is the simplest, we simply traverse the list until we find the first block big enough to accomodate the request. The best fit strategy involves traversing the whole list and find the best block that matches the request and in the next fit stategy you keep track of what the last allocated block was and start the search from there rather than from the head of the list.

I decided to implement a first fit strategy because it is the simplest to implement. So to now allocate a block we need to start traversing from the head and check if it is large enough to accomodate the request. if no such block is found we request more memory from the OS. If we find one we do one of two things:

we either return allocate that block if it's size matches exactly the request or if it is larger we split the block into two blocks and then allocate.

Here is the full code:
```c 
void* mem_alloc(size_t size){
    
    if(head == nullptr) {
        init_mem_pool();
    }
    mem_block_t* current = head;
    mem_block_t* prev = nullptr;
    while(current != nullptr){
       if(current->free && current->size == size){
            //Found a suitable block
            current->free = false;
            return (void*)(current + 1);
       }else if(current->free && current->size > size){
            //found a larger block,try to split
            size_t remaining_size = current->size - size;
            if(remaining_size <= MEM_BLOCK_SIZE){
                //Not enough space to split, allocate entire block
                current->free = false;
                return (void*)(current + 1);
            }
            //Suitable for split

            //splitting block
            mem_block_t* new_block = (mem_block_t*)((char*)(current + 1) + size);
            new_block->free = true;
            new_block->size = current->size - size - MEM_BLOCK_SIZE;
            new_block->is_aligned = false;
            new_block->next = current->next;

            //update current block
            current->free = false;
            current->size = size;
            current->is_aligned = false;
            current->next = new_block;
            return (void*)(current + 1); 
       }
       prev = current; 
       current = current->next;
    };

    //No suitable block found, request more memory
    mem_block_t* new_block = (mem_block_t*) sbrk(size + MEM_BLOCK_SIZE);
    if(new_block == (void*) -1) {
        return nullptr; //sbrk failed
    }
    new_block->size = size;
    new_block->free = false;
    new_block->is_aligned = false;
    new_block->next = nullptr;
    
    //Link the new block
    if (prev != nullptr) {
        prev->next = new_block;
    } else {
        head = new_block;  // This would be the first block
    }
    return (void*)(new_block + 1);
}

```
## Byte Alignment
Congrats at this point you should have a working allocator. However there is still work to do, a working allocator does not mean a correct one. An important piece is missing and that piece is called Byte Alignment.

## Freeing Memory
Every alloc function needs a free counterpart to release the memory when no longer in need. The free function has one simple job which is to mark the block as free. yep that's it. It doesn't need to return the memory to the OS since it might be needed again in the future. By simply marking it as free it becomes usable again within the program. One additional functionality is free is to do some house keeping, when memory is free it should check if it is next to other free blocks. If it is then we can combine the free blogs into one to reduce internal fragmentation. This process is called Coalescing memory.

Here is the full free code: 
```c 
void mem_free(void* ptr) {
    if(ptr == nullptr) return;
    
    mem_block_t* block = (mem_block_t*)ptr - 1;
    void* actual_ptr = ptr;
    
    if(block->is_aligned) {
        void** back_ptr = reinterpret_cast<void**>(ptr) - 1;
        actual_ptr = *back_ptr;
        block = (mem_block_t*)actual_ptr - 1;
    }
    block->free = true;

    //Coalesce adjacent free blocks on the right
    if(block->next != nullptr && block->next->free) {
        //update size to be sum of both blocks
        block->size += MEM_BLOCK_SIZE + block->next->size;
        //update next pointer to next of the next block
        block->next = block->next->next;
        //they are now 1 block
    }

    //do same for block on the left
    mem_block_t* current = head;
    while(current->next != nullptr) {
        if(current->next == block){
            if(current->free) {
                current->size += MEM_BLOCK_SIZE + block->size;
                current->next = block->next;
            }
            break;
        }
        current = current->next;
    }
}
```

As you must have noticed for aligned blocks we move the pointer 1 block back to find the actual pointer that was returned by `mem_alloc`.
## Missing Piece
Great work if you made it this far. This allocator contains all the elements of what would be considered a correct allocator. However there is still one piece i deliberately left out for you to work on. That is thread safety. The current allocator is not thread safe, if a program running multiple threads tries to allocate memory concurently you are basically guaranteed to be screwed. 

You can read about thread safety {here}[https://en.wikipedia.org/wiki/Thread_safety] or {here}[https://grokipedia.com/page/Thread_safety]. For C++ specifics check {this}[https://en.cppreference.com/w/cpp/language/raii.html].
You can also just read the full code {here}[https://github.com/ibrahimamam1/memcpp]

## References
- https://mirzafahad.github.io/2021-05-08-text-data-bss/
- https://linux.die.net/man/2/sbrk
- https://grokipedia.com/page/Thread_safety
- https://en.wikipedia.org/wiki/Thread_safety
- https://en.cppreference.com/w/cpp/language/raii.html
