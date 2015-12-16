---
title: "Microcorruption Lagos Writeup"
categories: CTF
comments: true
---

This is my writeup for the "Lagos" level in the [Microcorruption](https://microcorruption.com) CTF. It goes through my thought process and my initial solution. If you haven't already solved it or given it your best shot, you should probably stop reading and do that first.

## My thought process

This level takes your input from the heap, and filters out non-alphanumeric characters before copying them to the stack. As soon as a non-alphanumeric character is reached, the copying stops. There is a familiar `conditional_unlock_door` function that uses the `INT 0x7E` interrupt. It also uses `memset` to wipe the heap clean before testing the password copied to the stack. The overflow is potentially quite large, as up to 0x200 characters are taken as input.

Let's check out if the filtering is done correctly. It sets up by grabbing the first byte, putting it in r15, clearing r14, and putting 0x9 in r12 and 0x19 in r13.

```nasm
45ae:  mov.b     r15, r11
45b0:  add.b     #0xffd0, r11
45b4:  cmp.b     r11, r12
45b6:  jc        #0x45a0 <login+0x42>
```

These instructions test if the byte is between 0x30 and 0x39 inclusive (in ascii, these are the numbers). If so, it jumps to 0x45a0, where it copies the byte to the stack and grabs the next one.

```nasm
45b8:  add.b    #0xffef, r11
45bc:  cmp.b    r11, r13
45be:  jc       #0x45a0 <login+0x42>
45c0:  add.b    #0xffe0, r11
45c4:  cmp.b    r11, r13
45c6:  jc       #0x45a0 <login+0x42>
```

Similarly, the first three instructions test for uppercase letters, and the last three for lowercase. If none of the 3 jumps succeed, the function continues on to wipe the heap clean, and no more bytes are copied to the stack. It looks pretty robust.

It seems impossible to directly place non-alphanumeric characters on the stack ourselves. So where does that leave us? We can still jump to any alphanumeric location in the executable, so perhaps we can use return oriented programming techniques to execute code. However, at some point we will have to call the interrupt with 0xff00 in the sr register, or 0x7f00 on the stack. Quick searches for those two strings leave us empty handed.

What else can we do? The one location where we can place unrestricted shellcode is the heap. So let's try to find a way to execute on the heap. Our buffer on the heap is between addresses 0x2400 and 0x2600, so we can't jump directly to it. Not only that, but we will have to execute before `memset` wipes our shellcode (or find a way to prevent `memset` from being called). 

Fortunately, we notice a couple of properties:

1. The stack is tucked right under the base of the executable
2. The buffer overflow is large, in the order of 0x200 bytes
3. `conditional_unlock_door` is placed before `login`

In fact, we can rewrite the executable all the way until the code where the actual copying occurs. Testing this, the last word overwritten is at 0x45a0. Not coincidentally, this is where the sp is moved into r11 (as a result, subsequent bytes are written to lower addresses, close to the value of r14, or the counter recording how many bytes have been written).

We need to find a way to jump to the heap. Let's take a look at what kind of instructions we can get past the alphanumeric filter.

Using the mspgcc [instruction set docs](http://mspgcc.sourceforge.net/manual/x223.html) we can take a look at all the instructions. It's trivial to see that all one-operand instructions are out (the most significant nibble is 0x1, which places that byte out of range of the alphanumeric characters). We can use some of the relative jumps, but the range is limited, and not enough to get to the heap. Luckily, `mov`, `add`, `addc` and `subc` are all available for carefully selected addressing modes, or byte instructions. This is good news! You'll note that we can build `nop`, `pop`, and `ret` instructions with `mov`. That is even better news.

If we place a `ret` at 0x45a0, we can jump to our own (alphanumeric) shellcode before `memset` is called. What kind of shellcode can we build using only the aforementioned instructions? A little more digging reveals a problem: a two-operand instruction with an indexed (or indirect) destination operand falls outside alphanumeric bytes (since the destination addressing mode is the most significant bit of the second nibble). That's unfortunate. Combined with no `push`, we won't be able to modify memory (as far as I know! Perhaps there is a way).

So what now? We do have the ability to freely modify registers. And we also have the ability to jump to anywhere within the executable. What if we use the existing interrupt call? And set up the sr register to contain 0xff00? Then we could use one of the jumps to get to the `INT` function where the interrupt is called. Let's write the shellcode to do that.

First we need to clear sr and set it to contain 0xff00.

```nasm
mov      r6, sr
add      #0x5444, sr
add      #0x5566, sr
add      #0x5556, sr
```

The first instruction clears sr, although that wasn't strictly necessary (it contained 0x1, I could have calculated that into the three constants). It turns out the 5th least significant bit of the sr register is the `CPUOFF` flag. So the addition is designed to always keep a 0 in that bit, or else the program terminates.

Next, we need to jump to 0x460c. That's a jump of size 0x1b8. We will need half that to make that jump, so 0xdc. Unfortunately 0xdc isn't alphanumeric. But we can split it up into two jumps of size 0x76 and 0x66, since we've overwritten that entire portion of memory anyway.

The `jmp` instruction isn't alphanumeric. But `jl` is, and with 0xff00 in sr, the condition is satisfied. So that will do just find.

```nasm
jl       $+0xee
...
jl       $+0xca
```

The first instruction is placed right after our earlier shellcode, and the second at the destination address of the first.

Putting it all together, our alphanumeric input is as follows.

```
DCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCDDBF2PDT2PfU2PVUv8CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCCCCCCCCCCCCCCCd8CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC0A
```

The 0A at the end is a return, which is hit during the copying loop. It returns to the first address at DD (the first byte is copied twice, I didn't look into why). That's where my shellcode is (I used DD to mark the position for easier editing). BF clears sr, 2PDT2PfU2PVU fills sr with 0xff00, v8 jumps to d8, and d8 jumps to the interrupt call, which opens the lock. Success!

## Alternatives

The main thing I should have thought of was to immediately brute force a list of all available instructions. I spent a lot of time assuming I wouldn't be able to write the shellcode I needed, and looked for other options instead. It turns out there is quite a lot you can do. Even later when I realized it, I only ended up using a very small subset, the ones I was quickly able to glean from the instruction set documentation.

Aside from that, since registers can be freely modified (including pc), I didn't need to use relative jumps. It certainly wasn't necessary to write past `conditional_unlock_door` and prevent `memset` from being called (an artifact from my plan to execute on the heap). Judging from the hall of fame board, it probably wasn't even necessary to overwrite `conditional_unlock_door`. It might be easier to use the `login` return address, set up sr, and then jump directly to address 0x10.
