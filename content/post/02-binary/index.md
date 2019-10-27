---
title: "Binary Numbers and Data Representation"
date: 2019-10-22T18:20:01+01:00
draft: false
katex: true
markup: "mmark"
description: "Now on week 2 we will be looking at how data is represented in 6502 assembly. We will mainly focus on binary, looking specifically at what carry and overflow is."
tags: ["binary", "ASCII", "6502", "tutorial"]
categories: ["bookclub", "6502"]
---

## Bookclub Week 2

In week 1 we got to see a very brief look at how assembly, and specifically the 6502 variant, works. We built up an idea of what assembly is from the most basic binary patterns that a processor will want to accept. We then touched on the idea that a computer program will be made up of three parts: these instructions, data and memory locations. To get a good idea of how to effectively program the 6502 we will need to get a better understanding of all them. 

One of the many cool things about the 6502 is the numerous methods it can access locations in memory. Some of the methods though are slightly obtuse and can be hard to understand when beginning to learn. For this reason, we will wait until we have explored more of the architecture to delve into this part. The next option would be to go straight into learning the instruction set. However, without knowing how the input and output data behave we would not be able to fully appreciate what is happening. The last option then is to look at how data will be represented in our programs.

This week, we will take a long look at binary numbers. Here, we will aim to build up our understanding of binary, looking at where problems in the system lay and how different types of binary get around them. Then, we will look at ASCII and hexadecimal to round out different ways we can interact with the 6502. This won't be a complete look at all representation the processor can understand but it will give us a knowledge of the (in my opinion) most useful.

## Binary

Binary is something that is not necessary to know when doing modern, high-level programming. The 6502 can do many operations (add, subtracting, moving things in memory) for which you do not need to understand how binary works as well. However, other operations we might like to do, such as multiplication and divide, are not things the 6502 can do natively. We need to create these functions ourselves. This will require some understanding of the intricacies of binary calculation. Not least because internally, the 6502 will represent everything in the form of binary. We saw this when we discussed how the processor will try to "translate" everything down to binary patterns. At a very minimum being familiar reading binary numbers and with terms like bit, byte and nibble will be useful to understand generally how the 6502 operates. Luckily though for us, binary is intuitive and fun.

As of right now, the only practical way to store data in electronics is in a form of two-state logic. Something can either be "On" or "Off", "True" or "False". In a computer a singular binary digit, the most basic way to store information is called a *bit*. A bit can either be a ```1``` or a ```0```. Data in a computer is stored in groups of bits. In many cases, such as the 6502, the prefered way is in a group of eight (something like ```0001 0101```). This is called a *byte*. We can also have a smaller grouping of four bits called a *nibble* (lol). 

To adequately represent numeric data a few cases need to be looked at. Firstly, we will begin with unsigned binary. The eight bits ( $$b$$ ) of one byte will be represented as:

$$b_{7}b_{6}b_{5}b_{4}b_{3}b_{2}b_{1}b_{0}$$

We count the bits from right to left, indexing from 0. Each bit can be seen as being related to a binary number. The right most bit represents 2^0^, the next one to the left is 2^1^, then 2^2^, and so on. It will therefore be useful to remember the powers of 2, which are:

$$ 2^7 = 128,\;2^6 = 64,\;2^5 = 32,\;2^4 = 16,\;2^3 = 8,\;2^2 = 4,\;2^1 = 2,\;2^0 = 1 $$

To translate from binary to something we recognise we will need to sum all the appropriate powers of 2. If a bit is set to ```1``` we will include the corresponding power in our sum, if not we will discard it. This may already seem complicated, but looking at an example will hopefully help clear it up. If we take the binary number ```0010 0011``` we can see the only bits set are 0, 1 and 5. We can then "sum" the as such:

$$ 
\begin{aligned}
 &  \quad b_{5} + b_{1} + b_{0} \\
 &  = 2^{5} + 2^{1} + 2^{0} \\
 &  = 32 + 2 + 1 \\
 &  = 35 
\end{aligned}
$$

You can also try go the other way, from decimal to binary. We do this by dividing the number by two multiple times and tracking the remainder. The remainder being (which will always be a 1 or a 0) will map to the bits in a byte. For example, if you divide 35 by 2 you get 17 remainder 1. Therefore, bit 0 will be a 1. Again, this calculation is less strange than it sounds when you see it performed. Lets try 35 (or ```0010 0011```) again:

$$
\begin{aligned}
& 35 \div 2 = 17 r 1 \rightarrow 1 \\
& 17 \div 2 =\enspace8 r 1 \rightarrow 1 \\
&\enspace8 \div 2 =\enspace4 r 0 \rightarrow 0 \\
&\enspace4 \div 2 =\enspace2 r 0 \rightarrow 0 \\
&\enspace2 \div 2 =\enspace1 r 0 \rightarrow 0 \\
&\enspace1 \div 2 =\enspace0 r 1 \rightarrow 1 \\
&\enspace0 \div 2 =\enspace0 r 0 \rightarrow 0 \\
&\enspace0 \div 2 =\enspace0 r 0 \rightarrow 0 \\
\end{aligned}
$$

This method give us a result of ```0010 0011```, which is exactly what we started with!

### Addition

Now that we are comfortable reading and writing binary numbers we are going to want to try do some math with them. Addition works "normally". When you add two normal numbers (lets try 9 and 2) you "carry" everything that goes above ten to the next digit. So, 9 + 2 will give you 11. In binary you perform a similar process. Single bits follow these rules:

$$
\begin{aligned}
& 0 + 0 = \quad\;0 \\
& 1 + 0 = \quad\;1 \\
& 0 + 1 = \quad\;1 \\
& 1 + 1 = (1)0 \\
\end{aligned}
$$

The $$(1)$$ represents a carry. Its the same as us saying 9 + 2 = (1)1. Using what we have learnt before we can see that $$10$$ in binary is the equivilent of 2 in decimal, exactly what we would expect from 1 + 1!

If we try to do 3 (```0011```) + 1 (```0001```) we can see another example of how carrying in binary works:

$$
\begin{aligned}
& \enspace\; 0011 \\
& \underline{+0001} \\
& \enspace\; 0100 \\
\end{aligned}
$$

We can see here something has carried from the 1st bit to the 2nd. What if we add a larger number? Let's try 128 and 129:

$$
\begin{aligned}
& \enspace\;\; 1000 0001 \\
& \;\underline{+1000 0000} \\
& (1) 0000 0001\\
\end{aligned}
$$

We have now encountered our first problem. We know the answer should be 257, however, looking at what is in the first 8 bits it looks like the answer is 1. Only with the extra carry bit does the true answer become apparent. If we are going to be saving our binary numbers in 8-bits we will need a way to store and track this type of carry. The 6502 can do exactly that, phew! The 6502 has a set of flags built within it, one of which is called the Carry Flag. This can be set as 1 or 0. Whenever an addition happens for unsigned binary numbers (numbers that don't have a way of saying if they are positive or negative) that result in the answer being greater than 256 the Carry Flag is set to 1. This allows us to keep track of our maths and gives us a way to extend our additions past 8-bit numbers.

So, our binary numbers can add correctly, and we can represent our positive integers correctly. However, there are two problems:

* We can only count up to 255 (if we only use 8 bits).
* We can only represent positive numbers.

We need to be able to solve the latter issue. To do this we will introduce Signed Binary.

### Signed Binary

For unsigned binary, all 8-bits were used to store the value. This allowed us to have a range between 0 and 256. In signed binary, we will use bit 7 as a way to keep track of the sign of the number. However, this will come at the expense of a reduced maximum value our binary numbers can take.

In this new system, we will say that 1 is: ```0000 0001``` and -1 is: ```1000 0001```. Immediately, we have reduced the magnitude from 255 to 127 (oops) but we do have a way to represent negative numbers. Let us now go back and try doing a calculation with this new method. This is $$7$$ + $$-5$$:

$$
\begin{aligned}
& \enspace\; 0000 0111 \\
& \underline{+1000 0101} \\
& \enspace\; 1000 1100 \\
\end{aligned}
$$

Again, we have a problem, ```1000 1100``` is $$-12$$. This is wrong. To find a solution to this we will need to get to _two's complement_. To get to this point we will first have to introduce _one's complement_.

### One and Two's Complement

In **One's Complement** the positive number stays the same, ie, 1 is still```0000 0001```.
However, to get the negative we "swap" all the bits - all 0's become 1's and all 1's become 0's. This means -1 is: ```1111 1110```. As with signed binary, all positive numbers start with 0 and all negatives start with 1.

Lets try adding $$-4$$ and $$+6$$:

$$
\begin{aligned}
& \enspace\; 1111 1011 \\
& \underline{+0000 0110} \\
& \enspace\; 0000 0001 \\
\end{aligned}
$$

We have got a result of $$1$$, instead of $$2$$. Wrong, but we are getting there. We will have to expand one's complement before we get correct addition.

To arrive at **Two's Complement** you transform your value to One's Complement, then, for the negative number, add 1. So, 3 is ```0000 0011``` while -3 is ```1111 1101```.

Lets try adding $$-4$$ and $$6$$ again:

$$
\begin{aligned}
& \enspace\; 1111 1100 \\
& \underline{+0000 0110} \\
& \enspace\; 0000 0010 \\
\end{aligned}
$$

Which gives us 2. So now, finally, everything works! Getting here was slightly convoluted but two's complement is a popular and widely used way to represent binary numbers. Below is a table of two's complements with some 16-bit values shown as well.

{{< cupperFig
img="twos" 
caption="Two's Complements table [Credits](https://introcs.cs.princeton.edu/java/61data/)." 
command="Resize" 
options="1000x" >}}



### The Overflow

We saw that when adding unsigned numbers there can be a problem where we need to keep track of the carry. This was handled by the 6502's built-in Carry Flag. A similar problem can happen with signed binary numbers too. Here is 64 + 65:

$$
\begin{aligned}
& \enspace\; 0100 0000 \\
& \underline{+0100 0001} \\
& \enspace\; 1000 0001 \\
\end{aligned}
$$

This has given us an answer of -127. When the result of signed binary addition should be either >127 or <-127 we get an Overflow. This looks to us like two positive numbers summing to a negative, or two negatives summing to a positive. Again, the 6502 has a way to deal with this. The Overflow flag will be set when what is being carried into bit 7 does not match what is being carried out. Signed numbers can also set the Carry Flag too, whenever the result of the sum will need to be stored in more than 8-bits.

So, what should you do if you set the Overflow flag - what is the solution? Well, an unexpected overflow in practice is an exception and should not happen too often. While the flag is there to tell you something has gone wrong it is not going to help you fix it. You may need to go back, check your inputs and decide on the correct answer yourself.

Something neat is that when adding, the 6502 cannot tell the difference between signed and unsigned binary - it makes no distinction. During calculation, the flags are set automatically. We able then to use either signed or unsigned numbers, whatever we prefer. It is then also up to us, as the programmers, to be able to tell if the Carry and Overflow Flags being set are relevant to our problem. It also worth remembering that carrying and overflowing are entirely independant. We have just discussed addition so far but it should be said that all this is (mostly) true for subtraction as well. There is a lot to say about this topic but hopefully this provides a brief introduction to it. It can take a while to convince yourself how Carry and the Overflow works, it did for me at least. I have left a few really useful links at the bottom to help get a better understanding of the problem. 

## Hexadecimal

Now we understand more about flags and how the 6502 will attempt calculations we can try to find a nicer way to represent the data. Binary is nice, but it can be mentally taxing to have to read and write it. The way we will improve this is by making a switch to hexadecimal. Here, the decimal digits 0 to 15 are represented by the symbols $$0$$ to $$9$$, then $$A$$ to $$F$$. This range allows us to make the most of a nibble (4-bits, remember?!), using each combination to represent a symbol. For example, $$A$$ would become ```1010``` and $$F$$ would be ```1111```. Therefore, we can use a byte to hold 2 digits. We can convert from a hexadecimal digit (like $$FA$$) to decimal easily. You multiply the first digit by 16^1^, the second by 16^0^ and then sum! For example:

$$
\begin{aligned}
FA = 15 \times 16^1 + 10 \times 16^0 = 250
\end{aligned}
$$

Now, as we write our code for the 6502 we will aim to use hexadecimal to represent numbers. The same issues we discussed may still exist but trying to understand the data will be less difficult than if it were all just ones and zeroes. I have included a decimal-to-binary-to-hexadecimal table to give a better idea of how the representations relate to each other.

{{< cupperFig
img="hex.jpg" 
caption="Hexadecimal table [Credits](https://www.pinterest.co.uk/pin/419327415280550717)." 
command="Resize" 
options="1000x" >}}


## Alphanumeric Data

We have discussed how binary and hexadecimal is used to represent numbers but we also require an encoding for other symbols too. For this, we will use ASCII. We will want to encode the 26 letters in both upper and lower case, plus 10 numbers, and then around 20 special symbols. This can all be accomplished in 7 bits (which allows for 128 unique codes). The eighth bit could then (if you wanted..) be used to verify that the contents of the byte have not been accidentally changed. This bit is called the _parity bit_. For reference, here is a table with all the ASCII encodings that will be of use for us:

{{< cupperFig
img="ascii.png" 
caption="ASCII table [Credits](https://en.wikipedia.org/wiki/ASCII)." 
command="Resize" 
options="1000x" >}}

## Conclusions

I have tried to give a brief look at how data can be represented in the 6502. It has been a longer piece than I originally expected but it is still by no means an exhaustive list. Other forms exist in the form of BCD, EBCDIC and octadecimal. I decided to not talk about these though as binary, hexadecimal and ASCII are all we will need for the moment. For some things (NES dev, for example) BCD compatibility is removed. 

My original goal was to combine this with a discussion on 6502 hardware. As I started writing this though and realised the length I decided to save it for another time. How we represent data is important here and by starting to learn about the internal flags in the processor we have a good motivation to learn more about the architecture next week.


the end.

<hr>

[1] http://www.righto.com/2012/12/the-6502-overflow-flag-explained.html

[2] http://6502.org/tutorials/vflag.html#2.4

[3] https://planetcalc.com/747/
