---
title: "The 6502 (sixty-five-oh-two)"
date: 2019-10-30T19:56:01+01:00
draft: false
description: "It's time to look at the architecture of the 6502. This will allow us to see how dependent the software we write will be on the hardware capabilities."
tags: ["architecture", "stack", "6502", "tutorial"]
categories: ["bookclub", "6502"]
---

## Bookclub Week 3

Assembly language is strongly linked to the architecture it is written on. Everything you write will, on some level, depend on the instruction set and hardware of the processor. To write a program you won't require a deep knowledge of how the 6502 works, but to write an efficient one you will. We have begun to see this already. In week one we saw how instructions can be different on different machines and in week 2 we saw how the hardware enables us to compute things more easily. This week we will take a look at the 6502 directly. We will start to be able to piece everything we have learned so far together. As we continue in the following weeks we will be able to refer back to here as well, and further appreciate the links between the hardware and software we come to write. 

I feel an aim of this is to provide a broad overview of the entire system. To do this we need to see not only how 6502 Assembly can be written but also how the 6502 hardware works. Understanding, in its entirety, how a modern processor works probably would not be possible. Here, however, we can get a really good idea of how the hardware works. When we write code we will be able to say, on a very elementary level, what is going on.

To accomplish this we will discuss the following figure:

{{< cupperFig
img="micro.png"
caption="Generic microprocessor layout. Credits: Zaks."
command="Resize"
options="700x" >}}

This is the layout of a standard microprocessor of the era. We will go through it bit by bit, trying to understand it all.

## Data Buses

We will begin outside of the microprocessor unit (MPU), with the data buses. A computer bus has the job of transferring data and signal around the machine. In the 6502, they are three of them:

* The _data-bus_: As the name might suggest, this transfers data around. The data-bus is 8-bits wide, which means it can only carry 8 bits at a time. If you want to carry more data than that two separate signals will have to be sent down the bus. Usually, it will carry data from memory to the MPU, from the MPU to memory, or the MPU to an Input/ Output device.

* The _address-bus_: This bus carries addresses. These address will usually be a source, or a destination, for data. The address bus also differs from the data-bus as it is 16-bits wide. This means you can access any 16-bit address. Practically, this means you only have access to ~64000 addresses. After you have stored something in them all, that's it! No more memory! Interestingly, this has been a problem up to quite recently - even modern 32-bit computers can only natively access around 3gb of ram (this problem is called the 3gb barrier). 
* The _control-bus_: The control bus will try to keep everything inside the machine synced up. Here, it has been removed for simplicity.

## MPU and Registers

The MPU in the above figure, in our case, is the 6502. It contains the arithmetic-logical-unit (ALU), the control-unit (CU) and the registers (the flags we saw in week 2). The CU has control over how the processor operates. It decides when it receives an instruction, data or an address how to interpret it. 

Below, is a figure showing the internals of the 6502 (sans the CU). Outside of this, it will also require precise timing. This is why it is connected to a timer in the above figure. 

{{< cupperFig
img="internal.png"
caption="Internals of the MPU (the 6502) [Credits](https://www.atariarchives.org/roots/chapter_3.php)."
command="Resize"
options="1000x" >}}

Let's go through this diagram, right to left, and discuss what is going on.

The _ALU_ has a recognisable 'V' shape. Its role in the 6502 is to perform the calculations. It will accept two inputs, one in the 'left input' and one into the 'right input'. With these is can do addition (the ```ADC``` instruction) or subtraction (the ```SBC``` instruction). The left input of the ALU is connected to the accumulator, the A register. When doing logical or arithmetic operations usually one of the values is stored in the accumulator and the other somewhere in memory. The result of whatever operation will then also, usually, be stored in the accumulator. It is after this accumulating behaviour the register gets its name.   

The next two parts are the 8-bit _X_ and _Y_ registers. These are used frequently as ways to store values. They are useful as the 6502 comes with several instructions to manipulate them. Among these are ```INX``` which will increment X up 1, ```DEY``` which reduces Y by 1 and ```TXA``` which transfers the contents of X to A. Having this variety registers gives us  ways to move and modify data without having to specify a memory location. We will later this chapter that this can improve the speed of the code we write.

Then, there is a range of registers stored in _P_:

* N: keeps track if the result in the ALU is negative
* V: the overflow flag!
* B: used to handle breaks (we will discuss this more in a later post)
* D: this helps handle BCD numbers. This is a different way to represent data. We will not discuss it here.
* I: this tracks how interrupts will be handled (we will discuss this more in a later post) 
* Z: keeps track if the result of a calculation is zero
* C: the carry flag!

Having all these registers allow us to keep track of data and make comparisons. For example, we might want to branch to a different part of our program if the result of a calculation is zero. By checking the Z register we have a way to know. Some of these registers make use of certain instructions (```BRK``` sets B to 1). As we encounter these I will describe the effect on the register.

The _Stack-Pointer_ (SP), points to a specific bit in a place in memory called the stack. Using it we can keep track of where in the stack we are. I will go into more detail about this when we discuss the stack later in this post.

Finally, the _Program-Counter_ (PC) is a 16-bit registry. It has the important job of storing the next instruction that needs to be carried out. Let's look more into this right now.

## Instructions and the Program Counter

As we run a program the processor be in a constant cycle of fetching instructions then decoding and executing them. 

Fetch: In this step, the contents of PC deposited onto the address bus. Then, the instruction at this location is put on the data-bus. The processor reads this and puts in a special internal register called the _instruction-register_ (IR). The fetch cycle is now finished!

Decode and execute: With the appropriate instruction in the IR, the CU can begin to decode it. At this point, it can generate the signals needed to carry out the instruction. The length of time this takes will differ depending on what is being generated. Some instructions can happen inside the MPU (for example, ```INX```), whereas some will need something else from memory (ie, data or a memory address). The duration of an instruction is measured in clock cycles. The 6502 uses a one-megahertz clock, therefore a clock cycle takes 1 microsecond. 

Finally, the PC will be incremented and the next instruction loaded into it. Now, the cycle can begin again.

Usually, the first cycle will read the opcode, the next two will read the 16-bit address, then a fourth will read the appropriate piece of data. Of course, depending on the instruction this number will change.

## Memory Maps

We have discussed how with a 16-bit address bus we can reach 65,535 different locations in memory. While we access each of these addresses with the same methods that does not mean that every section of memory is used for the same purpose. The 64,535 addresses are divided into _pages_. Each page is a block of 256. This means that from address 0 to 255 (or 0 to 100 in hexadecimal) is page 0, address 256 to 511 is page 1, and so on. In your code you will, therefore, be tracking the page you are on and your location within a page. As you cross a page boundary (eg, go from page 12 to 13) it can result in needing to execute an extra instruction. This is because you are not just updating your location within the page, but the page number as well.

Certain pages within the 6502's memory are there to perform specific tasks. A memory map is a figure that describes how the locations in your system are meant to be used. Regardless of what 6502 system you are using (Apple ii, c64, NES, etc..) a lot of the functionality will be the same. However, the memory map will be different. Below is a very generic 6502 memory map

{{< cupperFig
img="mmap.png"
caption="Generic 6502 memory map [Credits](https://www.ele.uva.es/~jesus/6502copy/proto.html)."
command="Resize"
options="200x" >}}

The first page in the 6502 has an important position. The _zero-page_, which ranges from 0 to 255, is the only area in memory that can be accessed with an 8-bit address. Since parsing an 8-bit address is quicker than parsing a 16-bit one, the zero-page becomes an area in memory where data can be read and written to more efficiently. Therefore, it is sensible to store important data there that will need to be read a lot. 

The second page is also very important. The _Stack_ ranges from 256 to 511. This area of memory is a last-in-first-out (LIFO) list. Effectively, this means that you can only retrieve from the stack the last thing you deposited into it. It is like a neat pile of papers on a desk, you can only ever access the top one. We then have a stack pointer (the register S) that points us to the memory address at the top of this pile. The 6502 even gives us some manual control, allowing us to push and pull things to the stack at will using ```PHA``` and ```PLA```. It might seem strange why you would want an area of memory like this. Only being able to retrieve the last item you deposited seems like it could make this a page of memory quite niche. However, we will see as we progress that the stack is a fundamental part of memory. It is used to track subroutines and interrupts and provides an area of highly efficient memory. We will see all of these applications in later chapters. 

The next page is reserved for I/O devices. Accessing these areas of memory will allow us to interact with the outside world. Data here would correspond to external devices. Reading and writing to these memory locations would allow us to communicate with printers, game controllers and other peripherals. 

In this diagram, the Random Access Memory (RAM) is mapped to $0300 to $E000 (these are hex values, remember!). This is an area we can write data to. 

The locations between $E000 and $FFFF are mapped to Read Only Memory (ROM). As the name suggests, we cannot write to here. In fact, any attempt to do so will simply be ignored. An example of ROM would be data stored on a game cartridge. As the console is booted up this memory can be read, but obviously cannot be written to or else the game data would be changed. If you wanted to modify something in ROM it would first need to be transferred into RAM. 

It is up to us, the programmers, to keep track of where the data is stored and if it is stored appropriately. As we change from system to system we will also need to be aware that memory may be mapped differently.

We will end the discussion on memory with a slightly different topic: _endianness_. Whether a processor and language are little-endian or big-endian is based on how it orders bytes within a number. For example, a little-endian system will place the most significant bit of a number on the right. It takes 2 bytes to store the hexadecimal number ABCD. As the 6502 is little-endian the 2 bytes will, therefore, be ordered: CD, AB. Big-endian machines do the opposite. This will become important as we start to write code ourselves.

## Conclusions

This week we have had a look at the main parts of the 6502's hardware. Important for us right now is the large range of registers and the memory map that the 6502 gives us. These have a very strong impact on what we can do with the machine. As we delve into the instruction set and write programs ourselves we will see how interlinked our software will be to the hardware. At the moment it might seem like we have learnt a lot of different and unrelated ideas but as we progress we will see how it all fits together.

Now, with knowledge of the hardware and how data is represented next week we will move onto writing some code! A set of very simple programs will be shown to broaden our knowledge of the instruction set and allow us to get a better idea of how assembly is written. Then, the following week we will aim to run this code on our computers!

the end.

