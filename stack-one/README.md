# stack-one

Hello Again!

This challenge is about AMD64 stack-one

Let's start looking at the source code:

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-one/images/1.png)

It's basically the same as stack-zero. The only differences are:
- It's getting user input from the command line. Specifically the 1st argument.
- The user input is now stored in the locals.buffer using strcpy() instead of gets()
- local.changeme has to have a specific value which is 0x496c5962

(stack-zero : https://github.com/Altelus1/Hacking_Adventures/tree/master/Phoenix/stack-zero)

Question:
If it's almost exact the same, does the exploit from stack-zero (with a little tweaking) would still work?

Let's see.

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-one/images/2.png)

We can see the message:

```Getting closer! changeme is currently 0x00000061, we want 0x496c5962```

The exploit still works! We are able to change the first byte
of locals.changeme. However, how are we going to make it equal
to 0x496c5962. We also have to notice that the first byte
is in the right most of the locals.changeme. This is cause by the
endiannes (https://en.wikipedia.org/wiki/Endianness) since x86_64
is little endian. So we have to reverse the ordering of the
insertion of values.

### Finally

![](https://raw.githubusercontent.com/Altelus1/Hacking_Adventures/master/Phoenix/stack-one/images/3.png)

```b = 0x62``` <br/>
```Y = 0x59``` <br/>
```l = 0x6c``` <br/>
```I = 0x49``` <br/>

Thank you for reading!
See you in stack-two

