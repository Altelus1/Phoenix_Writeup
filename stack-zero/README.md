# stack-zero

let's start looking at the source code:

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/1.png)

We can see a struct is defined inside the main function.
The ```BANNER``` is printed.
```locals.changeme``` has been set to 0 then a gets() is called saving the
user input to ```locals.buffer```.

# GOAL:
In order to win, we have to change the value of locals.changeme

### Digging Deeper:

Let's look at the assembly of the main function in GDB

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/2.png)

Here's our stack. It's oversimplified but it will be enough
to visualize what we need to solve the problem
![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/3.png)

Let's look at the criticial instructions (i.e. instructions that affect the stack)
1st instruction is: ```0x00000000004005dd <+0>:	push   rbp```
![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/4.png)

The current value of ```$rbp``` is pushed onto the stack and ```$rsp``` is 
lowered by 8 bytes since we're pushing an 8 byte register. The gray block
is 8 bytes.

Next is: ```0x00000000004005de <+1>:	mov    rbp,rsp```

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/5.png)

Whatever the value of ```$rsp``` is now also the value of ```$rbp```.
This happens most of the time whenever entering a function. This
is also the address of a new ```stack frame``` pointed by the
frame pointer or the base pointer which is ```$ebp```.
(more about stack frames: https://en.wikipedia.org/wiki/Call_stack#Stack_and_frame_pointers)

Next is: ```0x00000000004005e1 <+4>:	sub    rsp,0x60```

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/5_5.png)

The large gap between ```$rsp``` and ```$rbp``` is 0x60 = 96 bytes.

Next is: 
```0x00000000004005e5 <+8>:	mov    DWORD PTR [rbp-0x54],edi```
```0x00000000004005e8 <+11>: mov    QWORD PTR [rbp-0x60],rsi```


![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/7.png)

The program stores values from ```$rsi``` and ```$edi at ```$rbp-0x60```
and ```$rsi-0x54``` respectively.

Next is: ```0x00000000004005f6 <+25>:	mov    DWORD PTR [rbp-0x10],0x0```
![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/7_5.png)

We can see that it's setting a part of the stack to 0.
Could this be the ```locals.changeme```?
Upon further investigation at our disassembly:

```0x0000000000400609 <+44>:	mov    eax,DWORD PTR [rbp-0x10]```
```0x000000000040060c <+47>:	test   eax,eax```
```0x000000000040060e <+49>:	je     0x40061c <main+63>```

It loads the address of that part of the stack and it checks the content
if it equals to 0. If it is equal to 0 then it jumps to 
```<main+63>```. If you think about it, the logic may
be opposite to the C source code however, this is
just the same. The instructions that will be jumped
into must conform to the logic of the C source code.

(```test eax, eax``` is basically ```$eax AND $eax```
and the flag register is updated depending on the result.)

Next is: ```0x00000000004005fd <+32>:	lea    rax,[rbp-0x50]```

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/8.png)

We can conclude that it is the start of the buffer since we can
see that the address is being passed to the gets()

See:
```0x00000000004005fd <+32>:	lea    rax,[rbp-0x50]```
```0x0000000000400601 <+36>:	mov    rdi,rax```
```0x0000000000400604 <+39>:	call   0x400430 <gets@plt>```

in x86_64, the first argument for the function is either stored
in stack or in register ```$rdi```

We have to realize something. The user input!:

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/9.png)

Whenever the getting input, the program places each byte from
a lower address to higher address. We know that ```$rbp-0x50```
is lower than ```$rbp-0x10``` and the actual buffer size is
```0x50 - 0x10 = 0x40 = 64 bytes```!

Just as written in the source code! The ```locals.buffer```
indeed has 64 bytes! But what happens if we exceed 64 bytes?

# Finally

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-zero/images/10.png)

If we exceed 64 bytes, the exceeded bytes actually overwrite the locals.changeme value.

See that if we have 65 'A's, the challenge has been won 
but if we only gave it 64 'A's, the challenge is lost.

This could've been won blindly by just giving the program
hundreds or thousands of character inputs but the way things
work underneath and why is it working needs to be known
since those are the keys for solving the higher challenges
effectively. 

Thank you very much For reading!

See you on stack-one!



