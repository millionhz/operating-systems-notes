# Persistence

## I/O

### Classical System Architecture

- Why hierarchical structure?
    * Faster Busses needs to short and are expensive to make.
    * Components of high demand needs to closer to the CPU

![Untitled](./img/Untitled%2072.png)

### Modern System Architecture

- Modern systems increasingly use specialized chipsets and faster point-to-point interconnects to improve performance
- CPU connects directly to high performance devices like Graphics and Memory
- I/O chips manages other basic devices by offering a number of different interconnects and forward the connection to the CPU using a standardized DMI bus.

![Untitled](./img/Untitled%2073.png)

## Canonical Device & Protocol

- A Canonical Device has:
    * **Interface:** Allows the system software (OS) to communicate with the device.
    * **Internals:** Implements the abstractions the device presents to the system.

![Untitled](./img/Untitled%2074.png)

### Interface/Controller

- **Status:** Reports the current status of the device (BUSY or not)
- **Command: ****Stores the commands that are sent by the OS and run by the device
- **Data:** Used to pass and get data to the device.

### Protocol

- The OS waits until the device is ready to receive command by repeatedly reading the status register (pooling).
- The OS writes data to the data register.
- The OS writes command to the command register which implicitly lets the device know that the data is present and it should start execution.
- The OS again waits until the device is ready to receive command by repeatedly reading the status register (pooling).

!!! note
    Pooling is CPU intensive. The CPU wastes cycles checking the status register of the device.

## Pooling and Interrupts

- Instead of polling the device repeatedly, the OS can issue a request, put the calling process to sleep, and context switch to another task.
- When the device is finally finished with the operation, it will raise a hardware interrupt, causing the CPU to jump into the OS’s interrupt handler.
- The interrupt handler will wake the process waiting for the I/O, which can then proceed as desired.
- Interrupts thus allow for overlap of computation and I/O, which is key for improved utilization.

![Untitled](./img/Untitled%2075.png)

### Problem with Interrupts

- The I/O device performs the request very quickly:
    * Ideally the first poll will find the device done with the task
    * but with interrupts the process (issuing IO) will have to wait for the next interrupt
    * For fast IO devices, use pooling; For devices whose speed is not knows, use hybrid approach; otherwise use interrupts.
- Network related I/O.
    * Livestock: When OS is doing nothing but processing interrupts.
    * Such conditions arise when programs like webservers suddenly receive a large number of traffic.
    * In case of interrupts, the OS will keep processing the interrupts instead of processing requests (requests are usually processed by other processes)
- **Coalescing**: Lower the overhead of interrupts by coalescing multiple interrupts into a single interrupt delivery unit,
    * Device delivering the interrupt waits for a while, this lets other requests to be completed whose interrupts can be coalesced together.

### Direct Memory Access - DMA

- While transferring large chunks of data to the device, the CPU has to waste a lot of time to populate the data register of the device. This trivial task wastes time that can be spent on running other processes.

![Untitled](./img/Untitled%2076.png)

- DMA is a specific device that can be programed to transfer data from memory to the CPU.
- OS programs the DMA engine by telling it where the data lives in memory, how much data to copy, and which device to send it to and DMA does the trivial task of copying by itself.
- After the transfer is complete, DMA raises an interrupt.

![Untitled](./img/Untitled%2077.png)

## Interacting with Device

### Explicit I/O

- Privileged `in` and `out` instructions.

### Memory-mapped I/O

- Device registers are available as memory locations, where the OS can write data the to.
- The data then gets routed to the device.
- Advantage: Can use the `load` and `store` instructions to interact with the device (no need for special instructions)

## Device Drivers

- Provide abstractions for device access management and control.
- Provided by the manufacturer of the device.
- Makes up majority of the code in an OS.
- Part of the kernel.

### Problem with Standardized Drivers

- Specialized functionalities provided by the device can not be utilized without updating the underlying driver.

## Types of Devices

![Untitled](./img/Untitled%2078.png)

## Hard Disk Drives

- Consists of many **sectors** (512-byte blocks), each of which can be read or written
- Sectors are numbered from 0 to n − 1 on a disk with n sectors.
    * Sectors have addresses.
- Multi-sector operations are possible; many file systems will read or write 4KB at a time.
- The only guarantee drive manufacturers make is that a single 512-byte write is atomic.
- Accessing two blocks near one-another within the drive’s address space will be faster than accessing two blocks that are far apart (not random access).
- Accessing blocks in a contiguous chunk (i.e., a sequential read or write) is the fastest access mode, and usually much faster than any more random-access pattern.

### Geometry

- **Platter:** a circular hard surface on which data is stored persistently by inducing magnetic changes to it.
- **Surface:** Each platter has 2 sides, each of which is called a surface.
- **Spindle:** Binds together multiple platters, and connected to a motor that spins the platters around at a constant rate
    * The rate of rotation is often measured in rotations per minute (RPM), and typical modern values are in the 7,200 RPM to 15,000 RPM range
- Data is encoded on each surface in concentric circles of sectors; we call one such concentric circle a track.
- A single surface contains many thousands and thousands of tracks, tightly packed together, with hundreds of tracks fitting into the width of a human hair.
- This process of reading and writing is accomplished by the disk head, one such head per surface of the drive.
- The disk head is attached to a single disk arm, which moves across the surface to position the head over the desired track.

### Rotational Delay

- The time it takes for the disk to rotate completely.

### Seek Time

- Time it takes to switch sectors.
- One of the costliest operations.

#### Phases

- Acceleration - the disk arm starts to move.
- Coasting
- Deceleration
- Settling - the arms position itself onto the track

### Track Skew

- Sequential reads must be properly serviced even when crossing track boundaries.
- When switching tracks the head needs some time to settle down. This time delay is adjusted by skewing the tracks by a few sectors.

![Untitled](./img/Untitled%2079.png)

### Multizoning

- Outer tracks are bigger and can accommodate more sectors.
- Disk is organized into multiple zones: a zone is consecutive set of tracks on a surface with the same number of sectors per track.
- Outer zones have more sectors than inner zones.

### Track Buffer

- Small amount of memory that acts as cache.
- For example, when reading a sector from the disk, the drive might decide to read in all the sectors on that track and cache them in its memory. (spatial locality)
- The OS reads and writes to this buffer.

### Write Back and Write Through

- **Write Back**
    * Acknowledge that write has completed after the disk has put the data in its memory.
    * This appears faster but can cause problems if the order of writes is important.
- **Write Through**
    * Acknowledge the write after the data has been written to the platter.

### Math

$$
\text{I/O Time} = T_{seek} + T_{rotation} + T_{transfer}
$$

$$
R_{I/O} = \frac{Size_{transfer}}{T_{I/O}}
$$

!!! note
    To calculate average rotation time, RPMs are divided by 2 as in average the disk will rotate only half way.

### Scheduling

### Shortest Seek Time First - SSTF

![Untitled](./img/Untitled%2080.png)

### SCAN

![Untitled](./img/Untitled%2081.png)

- The middle tracks gets served twice on average.

### C-SCAN

- Instead of sweeping in both directions across the disk, the algorithm only sweeps from outer-to-inner, and then resets at the outer track to begin again
- Doing so is a bit more fair to inner and outer tracks.

![Untitled](./img/Untitled%2082.png)

## File System

- A disk is divided into fixed sized portions called blocks (each of size 4KB)

![Untitled](./img/Untitled%2083.png)

- Most of the space in the file system is users data

![Untitled](./img/Untitled%2084.png)

- File system must track information about each file
    * Which data blocks (in the data region) comprise a file, the size of the file, its owner and access rights, access and modify times, and others → inode
    * inode table: reserved space on disk for inodes
    * 5 of 64 blocks for inodes
    * Typical inode size: 128-256 bytes
- Number of inodes represent the total number of files we can have on our filesystem.

![Untitled](./img/Untitled%2085.png)

- File system also needs to keep track of the unallocated or free blocks using bitmaps
    * Data bitmap: for the data region
    * inode bitmap: for the inode table
    * Each bit indicates whether corresponding object/block is free (0) or in-use (1)
    * We use an entire 4-KB block for each of these bitmaps for simplicity

![Untitled](./img/Untitled%2086.png)

- The superblock contains information about this particular file system, including, for example
    * how many inodes and data blocks are in the file system (80 and 56)
    * where the inode table begins (block 3), and so forth.
    * magic number to identify the file system
- while mounting the file, the is reads the superblock to initialize the parameters

![Untitled](./img/Untitled%2087.png)

### Inode

- Each inode is implicitly referred to by a number (called the i-number)
- Given an i-number, you should directly be able to calculate where on the disk the corresponding inode is located
    * Calculate the offset into the inode region (32 x sizeof(inode) (256 bytes) = 8192
    * Add start address of the inode table(12 KB) + inode region(8 KB) = 20 KB

```c
blk = (inumber * sizeof(inode_t)) / blockSize;
sector = ((blk * blockSize) + inodeStartAddr) / sectorSize;
```

![Untitled](./img/Untitled%2088.png)

- Direct Pointers: Inodes also have one or more direct pointers that point to the disk block containing the data.

![Untitled](./img/Untitled%2089.png)

### Multi-Level Index

- Indirect pointers are used to support files that can not be referenced by the limited number of direct pointers
- Instead of pointing to a block that contains user data, it points to a block that contains more pointers, each of which point to user data.
- To support even larger files double indirect or triple indirect pointers can be used.
- Multi-level indexes are imbalanced trees:
    * this does not effect performance as majority of the files are small and do not utilize the indirect pointers.

### Directories

- a directory basically just contains a list of (entry name, inode number) pairs.
- file system treats directories as special type of files; they are indexed in the inode table.
- For each file or directory in a given directory, there is a string and a number in the data block(s) of the directory
- For each string, there may also be a length (assuming variable-sized names)

!!! note
    in modern file systems, multiple blocks, lets say 8, are allocated (searched and marked for allocation) instead of 1 to store a file. This ensures that the files are stored contiguously.

## File Reading

![Untitled](./img/Untitled%2090.png)

![Untitled](./img/Untitled%2091.png)

!!! note
    Amount of I/O generated by the open is proportional to the length of the pathname.
For each additional directory in the path, we have to read its inode as well as its data.
This can be made worse if the directory is large.

## Writing Data

- Each write to a file logically generates five I/Os:
    * one to read the data bitmap (which is then updated to mark the newly-allocated block as used)
    * one to write the bitmap (to reflect its new state to disk)
    * two more to read and then write the inode (which is updated with the new block’s location),
    * one to write the actual block itself.
- *See Book for detailed walkthrough*
- Write operations generate a large amount of I/O calls.

## Caching and Buffering

- Every file open would require at least two reads for every level in the directory hierarchy (one to read the inode of the directory in question, and at least one to read its data)
- Most file systems aggressively use system memory to cache important blocks

### Static Partitioning

- Fixed size cache based on LRU policy
- Caches popular blocks of disk.
- Initialized at boot time and is generally 10% of the the total memory.
- Problem:
    * Wasteful - If the disk does not require 10% of the memory, the allocated pages go unused.

![Untitled](./img/Untitled%2092.png)

### Dynamic Partitioning

- Modern operating systems integrate virtual memory pages and file system pages into a unified page cache
- Memory can be allocated more flexibly across virtual memory and file system, depending on which needs more memory at a given time

![Untitled](./img/Untitled%2093.png)

!!! note
    Reads can be optimized by a large amount of cache but write requests need to generate I/O operations to be persistent.

### Write Buffering

- Delaying Writes
    * the file system can batch some updates into a smaller set of I/Os; for example, if an inode bitmap is updated when one file is created and then updated moments later as another file is created, the file system saves an I/O by delaying the write after the first update.
    * Some writes can be avoided all together, in case if an application created a file and then deletes it.
- By keeping writes in memory longer, performance can be improved by batching, scheduling, and even avoiding writes
- Problem:
    * if the system crashes before the updates have been propagated to disk, the updates are lost
