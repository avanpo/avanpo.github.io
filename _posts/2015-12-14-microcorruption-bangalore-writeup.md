---
title: "Microcorruption Bangalore Writeup"
categories: CTF
comments: true
---

This level introduces the concept of DEP, or Data Execution Prevention. You can read a lot about DEP elsewhere, but the gist of it is in the name -- it prevents instructions from being executed in certain areas of memory. So simply injecting and running our favorite shellcode will no longer work, if that area of memory is marked as non-executable. We will have to find a way around DEP, first. Let's check the Lock Manual for the specifics.

Under the interrupts, we can see that `INT 0x10` without any arguments turns on DEP. There doesn't appear to be a way to turn it off. `INT 0x11` marks a page as either only executable, or only writeable. It takes two arguments, the first argument being the page number, the second being 1 if writeable, 0 if executable. Okay, that sound pretty simple.

The executable is pretty short and straightforward compared to previous levels. `set_up_protection` is called, which first marks the pages as writeable or executable, and then turns on DEP.

```
44e0:  0f43           clr	r15
44e2:  b012 b444      call	#0x44b4 <mark_page_executable>
```

The very first page is marked executable. This makes sense, since address 0x10 has a callgate causing the software interrupt we've used so many times.

```
44e6:  1b43           mov	#0x1, r11
44e8:  0f4b           mov	r11, r15
44ea:  b012 9c44      call	#0x449c <mark_page_writable>
44ee:  1b53           inc	r11
44f0:  3b90 4400      cmp	#0x44, r11
44f4:  f923           jne	#0x44e8 <set_up_protection+0xa>
```

This next section marks the next 0x43 pages as writeable. Since the base of the executable is at address 0x4400, we can probably safely assume the page size is 256 bytes for the MSP430. The remaining pages are all marked as executable.

Strangely, looking at `main`, `login` and `conditional_unlock_door`, the password doesn't even appear to be read or validated anywhere. Nowhere is INT 0x7E called. So this lock doesn't appear open at all. Executing shellcode is probably the only way past this level.

```
4526:  3e40 3000      mov	#0x30, r14
452a:  0f41           mov	sp, r15
452c:  b012 6244      call	#0x4462 <getsn>
4530:  3f40 6524      mov	#0x2465, r15
4534:  b012 7a44      call	#0x447a <puts>
4538:  3150 1000      add	#0x10, sp
453c:  3041           ret
```

Breaking on this section shows us that we've got a buffer overflow of size 0x20 on the stack, the first word of which is the return address, at 0x3ffe.

We can't jump directly into our controlled buffer, since we still have to take care of DEP first. We have to make the stack executable first. How do we do that? First we look at `mark_page_executable`. It takes its arguments from the registers, so we can't use it directly.

```
44b6:  0312           push	#0x0
44b8:  0e12           push	r14
44ba:  3180 0600      sub	#0x6, sp
44be:  3240 0091      mov	#0x9100, sr
44c2:  b012 1000      call	#0x10
44c6:  3150 0a00      add	#0xa, sp
```

It pushes 0x0 and the page number in r14 to the stack, and then calls the interrupt, with 0x9100 in the status register, sr. You'll remember 0x11 was the interrupt to mark a page writeable or executable. It's been ANDed with 0x8000. It appears that the most significant bit is some kind of flag. This is not relevant now, though. Since we control the stack, we can simply place 0x0 and our page number of choice on the stack, and skip to 0x44ba (we could also skip to 0x44bc, but it would change the alignment for the rest of the steps by 0x6).

So which page do we set to executable? Our buffer spans page 0x3f and 0x40. Keep in mind that some of our buffer will be overwritten by the rest of the code in `mark_page_executable`. However, the shellcode to open the lock is quite small. So in the interest of keeping the entered password short, let's mark page 0x3f as executable.

First let's try it. Constructing our input:

```
16 dummy bytes | 0xba44 | 0x3f00 0x0000 | 0xee3f
```

The 16 dummy bytes are the password buffer. The next word is the `login` return address. We overwrite this in with 0xba44 in order to jump to the middle of `mark_page_executable`. The next two words are the arguments to the interrupt -- 0x3f00 is the page number, 0x0000 designates it as executable. The next word is the `mark_page_executable` return address. We overwrite it with 0xee3f in order to jump to our own shellcode at the beginning of the buffer (right now, these are the dummy bytes).

If you try this, you'll note there is no segfault. Success! That means the first part of our buffer is now executable.
