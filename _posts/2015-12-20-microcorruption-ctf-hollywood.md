---
title: "Microcorruption CTF: Hollywood"
categories: CTF
comments: true
---

This is my writeup for "Hollywood", the very last level on the [Microcorruption](https://microcorruption.com) CTF. Like usual, if you haven't already solved it or given it your very best shot, you should stop reading and go do that now. This level doesn't have any clever tricks, it just takes perseverance.

## Intro

The introduction text reveals two things. One is randomization, and the second is that there are no hardware modules attached. If this lock is to open using a valid password, it means that the password is checked within the binary, and thus the password must also exist somewhere in the binary. All we have to do is reverse engineer until we find it. Of course, some previous locks lacked legitimate means of opening them, but given the obvious theme of this level, I figured it was probably safe to assume the password existed somewhere.

The assembly is obfuscated, that much is clear from a first glance. I'm not sure where it lies on a scale of hardness, having no prior experience reversing obfuscated code. In any case, it made me appreciate the need for a good debugger, and fine-tuned control over your environment. In microcorruption, you have neither.

I looked around a bit to see if I could find some kind of emulator so I could just copy the binary, modify it freely, and run it myself to see what was going on. I think I could have cleaned up the binary quite a bit to make it more legible. Unfortunately I found no such thing (I didn't look very hard either). The next time I try something like this, I will make sure to familiarize myself with a good debugger first. Instead, I resigned myself to the debugger on the site, and the accompanying assembler open in another window.

## Wading through the obfuscation

The first thing I did was check where my password (literally "password") was entered. It was entered at address 0x2600, and stayed there for the lifetime of the program. Okay, good. That gives us a useful reference point.

I also searched for interrupt calls, but I only found one, and that was for a `rand` request.

While stepping through the binary, I noticed all the 4 byte jump instructions were corrupting the disassembled machine code, since the pc register would jump halfway into what could be interpreted as a multi-operand instruction. I wanted to get an overview of what each block of code looked like, so I wrote a quick tool to remove those jumps and the skipped instructions from the machine code in C (it was also around this time that I decided I really needed to learn a scripting language). I first looked for cases where the code could potentially jump to immediately after a 4-byte jump, but that seemed unlikely. The odds that the binary would have blocks of machine code that could be executed in two different ways depending on the offset seemed small to me. So I just removed the jumps and the following word.

```c
uint32_t *w = malloc(size * sizeof(uint32_t));
uint32_t *v = w;

// read in all the words here (omitted)

for (; v != end; ++v) {
    if (*v = 0x013c) {     // jmp $+0x4
        *v = 0x4343;       // nop
        *(v + 1) = 0x4343; // nop
    }
}

// print out machine code, ignoring nops
```

This gave me a better overview of the blocks of code I was stepping through. It was exceedingly tedious, but I began to notice patterns, similar blocks of instructions being executed. There were a ton of garbage instructions and patterns that made no sense. I would have really liked to remove and/or substitute all of them and rerun the resulting binary, but unfortunately I had to make do. Eventually I noticed the block was writing something at a high memory address, in the 0xe000 range. It printed out 7 words, and then executed them! And the first instruction was `mov #0x2600, r5`! Finally, we're getting somewhere.

It turns out that each block of code writes out one useful instruction (and a branch and some others), and then copies an existing block elsewhere and removes the initial block. Eventually, after tediously stepping your way through the program and keeping track of all the single useful instructions, you end up with an overall loop that looks like this:

```nasm
add      @r5, r4
swpb     r4
xor      @r5+, r6
xor      r4, r6
xor      r6, r4
xor      r4, r6
tst      0x0(r5)
mov      sr, r7
and      #0x2, r7
rra      r7
xor      #0x1, r7
swpb     r7
rra      r7
sxt      r7
swpb     r7
sxt      r7
mov      #0x4b18, r8
and      r7, r8
xor      #-0x1, r7
and      #0x47aa, r7
add      r7, r8
clr      r7
```

It reads 2 bytes from the password, and places them both in r4 and r6, using add and xor respectively. r4 is byte swapped. Then the values in r4 and r6 are swapped (a clever XOR trick). Finally it tests the next word of the entered password. After that, it looks like a convoluted way to keep the loop going (if the next word exists) or move on.

Since my entered password was 4 words long, I had to step through this enormous, bloated loop multiple times like an idiot, haphazardly stepping large portions of the code, but not too large, lest I miss important instructions and lose all of the work I had done. Although it became apparent that the above was a loop, it was difficult to tell where it ended. No jump instructions were printed, the loop controls must have been hidden away in the code that decided what the next instruction to print would be. Without a proper disassembler, I didn't bother to find out how exactly that code worked.

My larger steps kept getting caught in useless loops that looked like this, for very large values in r15:

```nasm
add      #-0x1, r15
jnz      $-0x2
```

I realized way too late that I could just replace one instance of "fe23" with "430f" (equivalent to `mov #0x0, r15`, the primitive debugger on Microcorruption allows you to rewrite memory). After that, my stepping got a lot smoother. By stepping approximately 0x790 instructions at a time, I could see the instructions get written one by one.

Eventually, after there were no more password bytes, I reached the following:

```nasm
cmp      #0xfeb1, r4
mov      sr, r7
clr      r4
cmp      #0x9298, r6
and      sr, r7
clr      r6
rra      r7
xor      #0x1, r7
...
```

The instructions I omitted result in setting the CPUOFF bit in the status register if there was a 1 in the carry flag after the above instructions. So what does this mean? Both cmp instructions have to result in the zero flag being set. If not, a bit ends up being shifted into the CPUOFF flag. So we have the answer! We just have to make sure r4 and r6 contain the relevant values.

## The password(s)

I quickly wrote a ruby script to find if a 4-byte solution existed. If you're wondering where I got the equations from, I took the cmp values, reversed the last XOR swap that would have happened between r4 and r6, and also reversed the byte swap in r4.

```ruby
def swpb(w)
  return (w & 0xff) << 8 | (w & 0xff00) >> 8
end

# Look for 4-byte input
##################################################
# Working backwards from the two CMP instructions:
#
# w1 + w2       = 0x9892 (in register r4)
# swpb(w1) ^ w2 = 0xfeb1 (in register r6)
#
# This piece of code uses these relationships to
# test all possible values for the first word,
# since the next can be determined from the first.
0x0000.upto(0xffff) do |w1|
  w2 = swpb(w1) ^ 0xfeb1
  if (w1 + w2) & 0xffff == 0x9892
    puts "sol: #{w1.to_s(16)} #{w2.to_s(16)}"
  end
end
```

Unfortunately it didn't find a solution, which I found odd. I figured 4 bytes to match, with 4 bytes of input should have one solution. Unfortunately XOR doesn't play nice with addition, and the operations for a 4-byte input don't result in all of the linear independence of the 4 bytes being "used" as it were. So then I wrote a script to do it with 6 bytes.

However, I did that wrong, because I had learned ruby that very day and had already spent way too much time on the computer. It didn't output anything! I tried brute forcing all the 4-byte options, thinking there was a mistake in my logic or derived equations (brute forcing 6-byte input would take forever). I also spent a lot of time retracing my steps through the debugger and learning more about the status register and its flags. All very useful, but ultimately it was just a dumb mistake in my code! Here's the fixed script that outputs all six-byte (and probably five-byte if you wait long enough) passwords.

```ruby
# Look for 6-byte input
##################################################
# Working backwards from the two CMP instructions:
#
# (r4 + w3) & 0xffff = 0x9892
# r6 ^ w3            = 0xfeb1
0x0001.upto(0xffff) do |w1|
  0x0001.upto(0xffff) do |w2|
    r4 = w1
    r6 = swpb(w1)
    r4 = (r4 + w2) & 0xffff
    r4 = swpb(r4)
    r6 ^= w2
    r6 ^= r4
    r4 ^= r6
    r6 ^= r4

    w3 = r6 ^ 0xfeb1
    if (r4 + w3) & 0xffff == 0x9892
      puts "sol: %04x%04x%04x" % [swpb(w1), swpb(w2), swpb(w3)]
    end
  end
end
```

Running it:

```bash
$ ruby hollywood.rb
sol: 1101a55eee48
sol: 1101c57ece28
...
```

These passwords are valid. Success!

## Final thoughts

I really liked this CTF (and Microcorruption in general). It showed that no matter the level of obfuscation, it's always possible to reverse the code. It just takes a lot of time and perseverance. That said, this level would have been impossible with a better hashing function (probably).

It also got me thinking about other types of obfuscation. This binary had a lot of garbage ops and very roundabout ways of getting simple things done. But it was easy to follow the entered password, and as a result you could always be sure you were still on the right path. And when the cmp instructions revealed themselves, the answer was immediately evident. But what if that wasn't the case?

For example, the values of the password you entered could be used to alter the very flow of the code. The flow could be a part of the validation process itself.  It would make things much more difficult to follow and keep track of. In the end though, I suppose anything is reversible -- given enough resources.
