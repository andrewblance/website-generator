---
title: "Assembly Basics"
date: 2019-11-03T13:10:01+01:00
draft: false
katex: true
markup: "mmark"
description: "We will now look at a few basic examples of programs that we will be able to make work on the 6502. There is a simple example of addition, then a more involved multiplication is shown"
tags: ["programming", "assembly", "6502", "tutorial"]
categories: ["bookclub", "6502"]
---

## Bookclub Week 4

Finally, after 3 weeks of Bookclub, we will start to actually write code. As we will see, we might not be able to copy-paste it into an assembler yet and make it run. After today though we'll be 90% of the way there. We will go over two examples: an easy one and a less-easy one. Looking at these will expand our knowledge of the instruction set of a 6502 and give us a better understanding of what kind of instructions are available. As well as this we will see that our assembly code can be decorated with information that the assembler will understand. This will allow us to make our code easier to read and use. 

## Addition

Our easiest example will be 1 + 2. Heres some code:

```
    LDA     #01      
    ADC     #02      
    STA     $0402    
```

On the left of each line is the instruction while on the right is the data or memory location needed to make the instruction work. This style is how assembly is written. Let's go through the example one line at a time.

_Line 1_: ```LDA``` is telling the 6502 to load the piece of data that follows into the Accumulator. Here, it is ```#01```. We can prefix our data with a symbol to tell the assembler what sort of data we are passing it. In this case we are using a hashtag which it means our data is a 'literal' - we are literally passing it a number. After this line, the contents of A is simply 1.

_Line 2_: ```ADC``` means to add the following value onto whatever is currently in the Accumulator. The 'C' in the instruction tells us it will be keeping track on any carry's that have happened in previous calculations. Here, we assume C = 0. Now, the value in A is 3.

_Line 3_: The ```STA``` instruction will send the contents of the Accumulator to a location in memory. In our example, this will send the value 3 to the memory address $0402. Note, we have not used a ```#``` - this is because it is not a literal, it is an address. Furthermore, we have used ```$``` which tells the assembler this value is in binary.

After these three instructions, we have successfully done 1 + 2 - well done! If we wanted to view the answer we would have to look in ```$0402```.

## Addressing and Flags

Last week, we discussed the numerous registers that could be set in the 6502. We also mentioned, above, that ```ADC``` will also add anything that has been carried from the last calculation (ie, anything in the carry register). We need to be careful that what is happening is actually what we want to happen. We don't want to be adding in the carry when we don't want to and we don't want to miss it out if we actually want it. We can manually clear the flags to ensure no unexpected values are involved in our arithmetic - better safe than sorry. This is what ```CLC``` (clear carry flag) and ```CLD``` (clear decimal flag) do.

In the previous example, we referred to the values 1 and 2 directly. There, it was ok as we only have a small number of values we need to use in our calculation. However, if we were doing something more involved this would get tiring. In Python, you can call a variable 'x' or 'y' and use them throughout your code. What is common to do in is to store your values somewhere in memory, then in your code point to that memory location. This will save us explicitly passing values around. For example, the value 1 might be stored in the location ```$0400``` and the value 2 in ```$0401```. Now, we can just refer to them and get the same result as before.

Below is some code with these two changes made.

```
    CLC            
    CLD            

    LDA     $0400   
    ADC     $0401  
    STA     $0402   
```

As well as built-in features of the assembly language, functionality can come from the assembler you choose. Here are some useful features available in most assemblers:

* You can store values in a "variable". So, you could say ```ADR1 = $0400``` and then whenever you called ```ADR1``` the assembler would understand you meant the address contained within in.
* You can leave comments with ";".
* You can "name" blocks of code. This allows us to refer to it numerous times in our program. We will see an example soon where this can be useful.

The code below incorporates the above suggestions:

```
    ADR1 = $0400
    ADR2 = $0401
    ADR3 = $0402

    CLC             ; clear carry bit
    CLD             ; clear decimal bit

ADD LDA     ADR1    ; load the contents of ADR1 into accumulator
    ADC     ADR2    ; add the contents of ADR2 to accumulator
    STA     ADR3    ; save result to ADR3
```

Now, our simple code uses more features of the language and assemblers. It is hopefully easier to read and will be simpler for us to use and build upon.

## A Small Division Cheat

Before attempting multiplication we will have a quick look at dividing by 2. This can be done easily by looking at operations we can perform on binary numbers. Let's look at what happens if we shift a binary number to the right. The number 15 (```0000 1110```) becomes 7 (```0000 0111```) and  55 (```0011 0111```) becomes 27 (```0001 1011```). I'm no expert, so I cannot say with 100% certainty that this works 100% of the time but it does provide an easy way to half numbers. Compared to doing "proper" division and multiplication it is significantly easier.

This sort of shifting is made possible by numerous instructions the 6502 has. The instruction set contains 2 shifts and 2 rotates. The following figure shows an example of each:

{{< cupperFig
img="shifts.png"
caption="Two examples of shift instructions in the 6502 instruction set: ROL and LSR. Made using draw.io"
command="Resize"
options="500x" >}}

```ROL``` rotates the number to the left, with the left-most bit falling into the carry register while the old contents of C move into the right-most bit.

```LSR``` shifts the number to the right. Here though, the right-most bit falls into the carry, but the left-most is replaced by 0.

These two instructions have equivalents that go in the opposite direction as well. For the meantime, however, ```ROL``` and ```LSR``` will be necessary for us to do multiplication. 

## Multiplication

Multiplication is more difficult than addition. This is primarily because we don't have a single instruction that can do it - we need to make our own way. This is a really good example to learn as it uses a nice variety of different instructions and techniques. We will start by looking at how we normally do multiplication and build from there.

$$
\begin{aligned}
& \enspace\enspace\enspace\; 12 \\
& \underline{\enspace\enspace\times23} \\
& \enspace\enspace\enspace\; 36 \\
& \underline{\enspace+240\enspace} \\
& =276 \\
\end{aligned}
$$

This (hopefully) looks familiar. This is how we can multiply normal, decimal numbers. Going forward, we will call the top number the multiplicand (MPD) and bottom the multiplier (MPR). Here then, 12 is our MPD and 23 our MPR. To do this multiplication we have to: 

* multiply 3 by 12, giving us 36.
* then we shift the MPD (12) to the left, giving us 120 and then multiply by 2. This gives us 240.
* combining these gives us 276.

This is the correct result. To do this with binary numbers to do the same procedure:

$$
\begin{aligned}
& \enspace\enspace\enspace\enspace\; 101 \\
& \underline{\enspace\enspace\enspace\times 011} \\
& \enspace\enspace\enspace\enspace\; 101 \\
& \enspace+1010 \\
& \underline{\enspace+00000\enspace} \\
& =01111 \\
\end{aligned}
$$

In this example we are doing 5 multiplied by 3. 101 is the multiplicand (MPD) and 011 is the multiplier (MPR)

* We take the first bit of the MPR (011). This is 1. We multply the MPD (101) by this. We are left with 101 
* Then, we shift the MPD to the left, giving us 1010. The next bit of the MPR is also 1 so we are left with 1010.
* We shift the MPD left once more. This results in 10100. The final bit of the MPR is 0 which means this gives us 00000
* Adding them altogether leaves us with 01110, which is 15.

Again, we have the correct result. If the significant bit of the MPR is 1 (the multiplier), you keep the result. Then, shift the original thing (the multiplicand - MPD). You repeat this procedure until you are sure all the bits of your number have been inspected. 

This can be shown in a flow chart. This is useful to do before it comes to actually write the code as it can help you visualise the problem you are tackling. This is good as, compared to our nice high-level modern languages, it is harder to just go for it and write a function.

{{< cupperFig
img="mult.png"
caption="Flow chart of how binary multiplication will work. Made using draw.io"
command="Resize"
options="700x" >}}

It might be confusing at first, but the above figure shows the same logic we had to do to multiply the binary numbers above. It's worth thinking about the chart and trying to understand it. Our next step is to convert this into code. Lets quickly consider some problems/ edge cases we might face while doing this though:

* Multiplying two 8-bit numbers could result in 16-bit one. To get around this we will need to store the result in 2 8-bit locations. One for the lower bits and one for the higher bits. For example, we can store 326 as two 8-bit numbers like ```00000001``` and ```01000110``` and then read it all together like ```00000001 01000110```. You can check, that's 326!
* We have to keep track of a lot of information to do this calculation. We will be forced to use more than just the registers the 6502 has. 
* There is no way to test and make a comparison on every bit of a number at once. To do this, we have to individually move the bits into A or C and do our comparisons there.

I will just present the code and then, as before, we can go through it line by line.

```
START   LDA     #0       ; zero accumulator
        STA     TMP      ; clear address
        STA     RESULT   ; clear
        STA     RESULT+1 ; clear
        LDX     #8       ; x is a counter
MULT    LSR     MPR      ; shift mpr right - pushing a bit into C
        BCC     NOADD    ; test carry bit
        LDA     RESULT   ; load A with low part of result
        CLC
        ADC     MPD      ; add mpd to res
        STA     RESULT   ; save result
        LDA     RESULT+1 ; add rest off shifted mpd
        ADC     TMP
        STA     RESULT+1
NOADD   ASL     MPD      ; shift mpd left, ready for next "loop"
        ROL     TMP      ; save bit from mpd into temp
        DEX              ; decrement counter
        BNE     MULT     ; go again if counter 0
```

We have multiple named blocks of code now. Why is this useful? Well, we can make comparisons and jump to these blocks depending on the result. This is what ```BCC``` and ```BNE``` will do. If Carry = 0, ```BCC``` will do a jump. The instruction ```BNE```  can cause a branch as well if the Z flag is not equal to 0. Knowing this, we can go through our code, while trying to keep in mind the flow chart we drew.

### START

```
START   LDA     #0       ; zero accumulator
        STA     TMP      ; clear address
        STA     RESULT   ; clear
        STA     RESULT+1 ; clear
        LDX     #8       ; x is a counter
```

_Line 1_: We load the Accumulator with 0, this will be used to set areas in memory blank.

_Line 2/ 3/ 4_: Three locations in memory are set to be empty by transferring the contents of A into them. These locations include a temporary holding area for values (TMP) and two locations where we will store the result (one for the upper bits of the result and one for the lower 8 bits). 

_Line 5_: The X register is loaded with the value 8. This will be used to count how many times we have shifted our values left. We can increment X downwards by using the instruction ```DEX```.

### MULT

```
MULT    LSR     MPR      ; shift mpr right
        BCC     NOADD    ; test carry bit
        LDA     RESULT   ; load a with low res
        CLC
        ADC     MPD      ; add mpd to res
        STA     RESULT   ; save result
        LDA     RESULT+1 ; add rest off shifted mpd
        ADC     TMP
        STA     RESULT+1
```

_Line 1_: ```LSR``` is one of the shifts we saw above. This will cause the significant bit of our multiplier to fall into the carry register. 

_Line 2_: ```BCC``` test the contents of the carry. If it is 1 we will continue with the next line. However, if it is 0 we will jump to the block of code we have titled "NOADD". This will ignore the rest of the MULT block. 

_Line 3_: Assuming the contents of the carry register is 1, we now put the current contents of the lower part of our result into the accumulator.

_Line 4_: As there is no need to carry anything into the lower half of the result we can do a ```CLC```. This won't be necessary when taking care of the top half of the bits though - we will want to know if something has carried from the lower to the higher part.

_Line 5_: As the MPR is 1 we can include the MPD in the result. The ```ADC``` here will add the multiplicand to the result.

_Line 6_: We now save the lower result back to RESULT

_Line 7/ 8/ 9_: this does a similar thing as lines 3/ 5 and 6 but for the top bits.

### NOADD

```
NOADD   ASL     MPD      ; shift mpd left
        ROL     TMP      ; save bit from mpd
        DEX              ; decrement counter
        BNE     MULT     ; go again if counter 0
```

This is where the code will jump to from Line 2 of MULT if C = 0. However, we reach here as well after Line 9 of MULT. 

_Line 1_: ```ASL``` shifts the multiplicand left. It is the "opposite" of ```LSR```. This gets it ready for when we loop around and find the next bit of MPR.

_Line 2_: After Line 1 something will fall into the Carry. This can be recovered into TMP by using the instruction ```ROL```, which will push the contents of the carry register into its right-most bit. This is done so this bit can be included in the upper bits of the result (see Line 8 of MULT)

_Line 3_: ```DEX``` reduces the X register by 1. 

_Line 4_: ```BNE``` branches if the contents of Z does not equal 0. Z will be set to 1 when the ```DEX``` instruction on Line 3 sets X to 0. This will happen after all the bits in MPR have been used. 

Done! Doing this whole procedure will multiply 2 numbers. It is a little complicated, right? It does follow our flow chart though, which follows the logic of how we did the multiplication to begin with. It is a useful example, one worth trying to fully understand, as it contains a lot of important instructions and ideas that we will be using to write the rest of the code we will see.

## Instruction Set

We have now seen a variety of the 6502 instruction set. However, there is still a lot more. These two examples do show us what sort of instructions are available. Broadly, they fit into the following categories:

* Data processing (ADC, DEX)
* Data transfer (LDA, STA, LDX)
* Shifts (ROL, LSR, ASL)
* Testing and Branching (BCC, BNE)
* Control (CLC, CLD)

Many more exist in each category. At this point, most resources will take time (usually around 100 pages) to discuss every instruction. This sort of reference material can be very useful and would provide an in-depth description of every aspect of the instructions. However, I am not going to bother to do this here. Its a bit dull to write, and not super interesting to read. Furthermore, you can easily find all this on the internet. A lot of the instructions that we have not seen now though are variations of the ones we have (```DEY``` decrements the Y register, ```SBC``` subtracts...). The best way to learn how they all work is to use them or see them used in examples. That will be how we will go about learning the rest of them as we go on in the following weeks. 

## Conclusion

We have seen two useful examples of code: addition and multiplication. Obviously, you can do subtraction and division to but I just did not show that here. These examples, in particular, the multiplication one, exposes us to a range of instructions that will be useful going forward. 

Next week, we will concentrate on trying to get this code to actually run. To do this we will need to install an assembler and emulator. Then, after that, we will return to more examples of how to use the instruction set and the many methods of accessing areas in memory.

the end.

<hr>

As always, this is heavily based of off Zaks' Programming the 6502
