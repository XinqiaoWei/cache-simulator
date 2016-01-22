Cache and Memory Simulator
=======================

Parameters
---------
1. Address bits
2. Bytes/Word
3. Words/Block
4. Blocks/Set
5. Sets/Cache
6. Hit time (cycles/Access)
7. Memory read time in cycles/Block
8. Memory write time in cycles/Block

###“read-req”

First of all, check the address is valid or not since the total memory space is at most 8MB. Then check for the tag and valid bit. If tagMatch is true which means it is a hit, find the word to read and specified the matched word and read data. Otherwise, flush the cache and read data from memory.

###“write-req”

if tag matches, and valid bit is 1, then it is a hit and write data. Set the dirty bit to one, so next time cache data has to be written back to memory firstly. If the dirty bit is one, flush the memory and modify the block.

If tag is miss, LRU policy has to be used and then modify the block and flush it.

###“flush-req”

write the previous data stored in that block back to memory before it is modified.

