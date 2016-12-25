
A memory allocator for embedded systems (microcontrollers)
==========================================================

NOTE: this repository was created before the original author created his own one,
so now, please, see the [original repo by Ralph Hempel](https://github.com/rhempel/umm_malloc).

All improvements which I've done are already pulled there.

-----------------------------------------------------------------------------

This is a memory management library specifically designed to work with the   
ARM7 embedded processor, but it should work on many other 32 bit processors, 
as well as 16 and 8 bit devices.

Initially I've found this great memory allocator here:
http://hempeldesigngroup.com/embedded/stories/memorymanager/ ,
fixed little configuration issues, and uploaded to the github.

I successfully tested it on Microchip PIC32 device, and it is
extremely effective comparing to default memory allocator provided by Microchip.

I was seriously beaten by fragmentation with default allocator: my project often
allocates blocks of various size, from several bytes to several hundreds of bytes,
and sometimes I faced 'out of memory' error.
My project has total 8192 bytes of heap, at the particular moment there is more
than 5K of free memory, but default allocator has maximum non-fragmented
memory block just of about 700 bytes, because of fragmentation.
This is too bad, so I decided to look for more efficient solution. 

When I tried this memory allocator, in exactly the same situation it has more
than 3800 bytes block! It was so unbelievable to be, and I performed hard test:
device worked heavily more than 30 hours. No memory leaks, everything works
as it should work.

I also found this allocator in the FreeRTOS repository:
http://svnmios.midibox.org/listing.php?repname=svn.mios32&path=%2Ftrunk%2FFreeRTOS%2FSource%2Fportable%2FMemMang%2F&rev=1041&peg=1041# ,
and this fact is an additional evidence of stability of ```umm_malloc```.

So I completely switched to ```umm_malloc```, and I'm quite happy with it.


Acknowledgements                                                             
----------------

Joerg Wunsch and the avr-libc provided the first malloc() implementation     
that I examined in detail.                                                   
                                                                             
http://www.nongnu.org/avr-libc                                               
                                                                             
Doug Lea's paper on malloc() was another excellent reference and provides    
a lot of detail on advanced memory management techniques such as binning.    
                                                                             
http://g.oswego.edu/dl/html/malloc.html                                      
                                                                             
Bill Dittman provided excellent suggestions, including macros to support     
using these functions in critical sections, and for optimizing realloc()     
further by checking to see if the previous block was free and could be       
used for the new block size. This can help to reduce heap fragmentation      
significantly.                                                               
                                                                             
Yaniv Ankin suggested that a way to dump the current heap condition          
might be useful. I combined this with an idea from plarroy to also           
allow checking a free pointer to make sure it's valid.                       

---------------------------------------------------------------

The memory manager assumes the following things:                           
                                                                           
1. The standard POSIX compliant malloc/realloc/free semantics are used     
2. All memory used by the manager is allocated at link time, it is aligned 
   on a 32 bit boundary, it is contiguous, and its extent (start and end   
   address) is filled in by the linker.                                    
3. All memory used by the manager is initialized to 0 as part of the       
   runtime startup routine. No other initialization is required.           
                                                                           
The fastest linked list implementations use doubly linked lists so that    
its possible to insert and delete blocks in constant time. This memory     
manager keeps track of both free and used blocks in a doubly linked list.  
                                                                           
Most memory managers use some kind of list structure made up of pointers   
to keep track of used - and sometimes free - blocks of memory. In an       
embedded system, this can get pretty expensive as each pointer can use     
up to 32 bits.                                                             
                                                                           
In most embedded systems there is no need for managing large blocks        
of memory dynamically, so a full 32 bit pointer based data structure       
for the free and used block lists is wasteful. A block of memory on        
the free list would use 16 bytes just for the pointers!                    
                                                                           
This memory management library sees the malloc heap as an array of blocks, 
and uses block numbers to keep track of locations. The block numbers are   
15 bits - which allows for up to 32767 blocks of memory. The high order    
bit marks a block as being either free or in use, which will be explained  
later.                                                                     
                                                                           
The result is that a block of memory on the free list uses just 8 bytes    
instead of 16.                                                             
                                                                           
In fact, we go even one step futher when we realize that the free block    
index values are available to store data when the block is allocated.      
                                                                           
The overhead of an allocated block is therefore just 4 bytes.              
                                                                           
Each memory block holds 8 bytes, and there are up to 32767 blocks          
available, for about 256K of heap space. If that's not enough, you         
can always add more data bytes to the body of the memory block             
at the expense of free block size overhead.                                
                                                                           
Detailed explanation of algorithms can be found in ```umm_malloc.c``` file.

Usage
-----

 - Clone repository ```umm_malloc``` somewhere (or just download zip file and unpack it)
 - Add ```umm_malloc.c``` file to your project
 - Edit ```umm_malloc_cfg.h``` file depending on your needs
   (if you haven't defined ```UMM_REDEFINE_MEM_FUNCTIONS``` macro, make sure you use
   ```umm_malloc()``` and ```umm_free()``` functions in your project, instead of standard
   ```malloc()``` and ```free()``` functions)


