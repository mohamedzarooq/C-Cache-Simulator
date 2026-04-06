# C-Cache-Simulator
A basic L3 cache simulator that I created in C that simulates how hits and misses and loading data from different hierarchies or main memory works in modern cpu's. L3 simply means level 3, which is the highest level in a cache hierarchy in most modern cpu's (L1, L2, L3). 
Currently here is just L1 cache that I will begin to expand to larger hierarchies (hopefully L3).

The basic functionality of this is that it takes some example 16 bit address (0x0000 for example) and deconstructs it by directly mapping it to a tag, index, and offset. I then compare this to each address line in my cache and see if it matches and if the valid bit is true (the valid bit prevents false hits). I then return the number of hits and misses in the pool.

#include <stdio.h>
#include <stdint.h>
#include <math.h>
#include <stdbool.h>

#define CACHELINE 16 //defining number of cachelines I'm using for the L1 cache
#define BYTEBLOCK 16                            
#define ADDRESS 16 //addresses used
#define INDEX 4 //index, offset, and tag for DMC
#define OFFSET 4 
#define TAG (ADDRESS - INDEX - OFFSET)

/*
Direct Mapped Cache(DMC) takes memory from main(RAM) and organizes each one
with a unique address, composed of a tag index and offset.
If the memory is found within the cache by its address, its
considered a cache hit. otherwise its a cache miss and the
computer must go and retrieve the memory needed from the RAM
This mapping allows for high speed memory and overall increases
the performance of the CPU

- Address: Tag + Index + Offset
- If Address = Data in cache, Cache hit
- Else Cache miss
- Tag can be attributed to multiple different pieces of memory
depending on the size of memory used and available space within cache
- if there are N lines in cache, DMC takes the index using modN and
the index takes log_2 (N) bits
- so 32 bit addresses  64 cache lines (2^6) and 16 byte blocks (2^4),
then you offset is 4 bits(log_2 16), index is 6 bits (log_2 64) and tag
is the rest (so 22 bits)

*/

uint16_t get_offset(uint16_t address){
    return address & ((1 << OFFSET) - 1); //takes the address we received and masks it with the our offset. it is subtracted by 1 because since our offset is 4 bits, the maximum integer value we can have is 15 or 1111, subtracting allows us to move it from 10000 to 1111
}

uint16_t get_index(uint16_t address){    //gets the index by shifting the address over and masking it with 10000 (1 shifted by the index value) 
    return (address >> OFFSET)  & ((1 << INDEX)-1);
}
uint16_t get_tag(uint16_t address){ 
    return address >> TAG;
} 
struct Cache_lines { //struct thats needed to verify the valid bit is 1(true here) and if the tag is the same
    bool valid;
    uint16_t tag;
};

int main(){

    struct Cache_lines cache[CACHELINE]; 

     for(int i = 0; i < CACHELINE; i++){    //at start up, cache is empty so all are set to false and empty
        cache[i].valid = false;
        cache[i].tag = 0;
    }

    uint16_t addresses[] = {0x0000, 0x0001, 0x0004, 0x0100}; //set of example addresses I used as tests
    int hit = 0; //hits and misses
    int miss = 0;
    for(int i = 0; i < 4; i++){
        if((cache[get_index(addresses[i])].valid == true) && (cache[get_index(addresses[i])].tag == get_tag(addresses[i]))){ 
            printf("HIT\n"); 
            hit++;  //for each address in the set, if the directly mapped address's valid bit is 1(true) and the address tag matches the tag in the cache, its a cache hit.
        }
        else{
            printf("MISS\n"); 
            cache[get_index(addresses[i])].valid = true;
            cache[get_index(addresses[i])].tag = get_tag(addresses[i]);
            miss++;
            //if not then it's a cache miss and the correct data is moved into the cache
        }
    }
    printf("Total Hits: %d\n", hit );
    printf("Total Misses: %d\n", miss); 
    //prints the cache hits and misses

}
