---
title: "An Introduction to 6502 Assembly!"
date: 2019-10-13T17:24:36+01:00
draft: false
description: "Week 1 of bookclub! We will be introducing my contribution - an introduction to 6502 assembly!"
tags: ["6502", "tutorial", "assembly", "6502 tutorial", "Introduction to 6502"]
categories: ["bookclub", "6502"]
---

## Bookclub Week 1

For this session of my office's book club I'm going to be looking at the 6502 assembly programming language. Each week I'll try explain a little more about assembly and how to write it. I'm not sure if the structure will change from week to week, or how I will split up the topics but I like the idea of using my webpage to host the content. The 6502 was a really popular microprocessor and I think a good starting point to learning assembly.

Currently, the majority of the most popular programming languages (like Python or Java) are high-level [1]. High-level languages aim to use clear and simple syntax and structure, hiding large amounts of potential complexity that may come from the computer. This can result in a language is easy to read and write, which is an obvious benefit. However, the ease of use comes with a trade-off. By abstracting away how the computer's hardware functions you lose an amount of efficiency and precise control of the system. Why is this? Well, though a language like Python is very "human-readable" this does not mean it is easy for your computer to read as well. For your computer to find it easy to read it needs to be "translated" into 1's and 0's. During this "translation" the computer will try to interpret what your code means (what is 5, what is "+", or "=", what does the word "loop" represent, etc). It will need to make assumptions (where do you want to store this? How much precision do you want? etc) - there is no guarentee these assumptions will be correct. Then, there is also a computational cost to this translation, it will take time for your computer to do this.

You could imagine a programming language that was significantly closer to the binary numbers the computer wants to read, more low level. A language like this would be devoid of the objects and structures high-level languages give you (arrays, lists, statements such as "while" and "for" and almost everything that makes programming a language like Python "nice") and instead have a set of capabilities entirely determined by the hardware and manufacturer. Here, if you wanted to do something you would need it entirely by yourself - if you want to save a value you need to choose precisely where it will be stored, if you want to multiply something you need to tell it exactly what multiplying means, and so on. A language like this may be limited in what you could do (in a sense..) but you would have the benefit of knowing precisely what the computer was doing.

Beginning this week we will begin to investigate the 6502 assembly programming language. This is a very different language to something like Python, R, or c++. 6502 assembly is a very low-level language that works specifically for the 6502 microprocessor - a very popular processor from the 1970s. We will begin by taking a closer look at what assembly exactly is and why we have chosen to spend time learning a variant of it that is almost 50 years old. For the time being, I will not dwell on things like how binary and hexadecimal numbers work, I will come back in a later post and give more thorough definitions. Right now, I'm just trying to present a flavour of the elements of the language.

## Why do you need a language like Assembly?

We started by thinking about high-level languages and then imagining what a low-level one would look like in comparison, here we will try to go the other way. We are going to explore what is the most basic inputs a processor wants to accept, then build up a language around that.

### Binary

A processor, based on its hardware, will have several Instructions it will accept. The complete set of instructions a processor can perform is known as its instruction set. You can ask the processor to perform an instruction by passing it the appropriate binary pattern. This is effectively as low-level we can go. For example, if the 6502 is given the 8-bit pattern ```10000101``` it will interpret this as ```Store the contents of the Accumulator in a specific memory location```. At the moment, don't worry about what exactly this means, the aim is to show that by passing binary patterns to the processor it will perform an instruction.

We can use these instructions to construct more complicated processes than the ones available to us as individual instructions. This is a program. A program is more though than just a list of instructions. It will need to include data to work with and locations in memory of where to store and access things. Telling the processor to add something is pointless if you can't tell it what numbers to sum and where to store the result! Here is an example program:

```
10100101
01100000
01100101
01100001
10000101
01100010
```

It's obvious what this does, right? This will add 2 numbers, and save the result somewhere. To be clear, this is not written in Assembly, this is Machine Language. Eventually, all code you write, regardless of what language, will be converted to look something like this. As someone who wants to write code though there are many very obvious flaws with machine language:

* It is very hard to read and understand, at a glance every line looks identical.
* It is very hard to write, how are you meant to remember each instruction?
* There are lots of other reasons, it should not be hard to think of more.

### Hexadecimal

We need a better way to represent our data, instructions and memory locations. Binary is nice for a computer but it is not very nice for a human. Our first step will be writing everything in hexadecimal. Here is the same program as above but written in hexadecimal:

```
A5
60
65
61
85
62
```

This is a little better than the binary equivalent. It is easier to read, write and debug. Earlier, I made the statement that the processor will only accept things in the form of an 8-bit pattern - this is still true. Before this program can be run by the 6502 it will need to be translated into binary - this is done using a hexadecimal loader. As we begin to construct a higher-level language than machine code we begin to make trade-offs. We need, for our sanity, a more human-readable programming language, but the cost of using a loader is that is must also occupy space in memory, memory you can no longer use for running your program. Writing a program in binary (or probably even hexadecimal) would not be a sensible thing to do, so using a loader is worth the cost.

Writing in hexadecimal still has lots of clear issues. Mainly, how would you know what that code does if I had not told you beforehand? How is someone meant to remember what all those hexadecimal values represent? To reduce this issue we will add another layer of abstraction to our code: mnemonics.

### Instruction Mnemonics

Here, we will give our instruction set memorable (hopefully) names. Then, instead of referring to the instructions by a hexadecimal value we will use their given names. A program that uses these mnemonics is said to be written in Assembly. If I write the same addition example as before in 6502 assembly it will look like:

```
LDA   $60
ADC   $61
STA   $62
```

The program is beginning to look a lot cleaner and readable. While at the moment all this might not mean much to you, I promise it is easier to remember the mnemonics than the hex symbols. Knowing that loosely ```LDA``` means load, ```ADC``` means add and ```STA``` kinda means save allows us to begin to get an idea of what the program is doing. Even if we were not told the purpose of the code, but we knew what instructions meant, we would have a good idea of what it would do.

I have been careful here to refer to the above code as being written in "6502 assembly" and not simply "assembly". This is because instruction names are decided upon by the processor manufacturer and can therefore wildly differ from processor to processor. A combination of this and different processor hardware allowing for different instructions to exist means that assembly code written for one type of processor will not run on another. This is a huge difference compared to a modern high-level language. I can write a Python script on a MacBook that has an Intel i7 processor and then easily (in theory) run it on a Windows 10 machine with an i3 processor. This is not the case with assembly. 6502 assembly code will not run on a modern x86 machine.

To translate our assembly code into something the machine can understand and run we use an assembler. This will take our program written in assembly (our source program) and return something the 6502 can run (an object program). As with the case of using the hexadecimal loader there is a cost to this but it is entirely worth it so we can avoid machine language.

Of course though, I admit this is still significantly more complicated than the Python equivilent. That would look something like:

```
x = y + z
```

However, thats just not as fun as assembly!

## Why bother learning this?

Even the small assembly addition example I presented is intimidating to look at. It is (for now!) still hard for us to read, composed of symbols that we are not used to having to read. In my opinion, it is worth persevering and learning assembly. This is for a few reasons:

* While you probably won't use this for any practical application learning a different programming language (I think) makes you better at any other one. This is amplified when learning assembly. If you are like me and do not know a lot of computer science, learning assembly teaches you a lot about how computers work. It demystifies what is going on underneath the hood when you run your Python program. It forces you to think about things like the stack, and memory locations and more fundamental parts of the computer. These are things you can mostly get away with (thankfully!) not knowing much about if you write in high-level languages.
* I think its fun, but maybe I'm a nerd.
* Mainly, it will enable to solve Euler-Project problems incredibly fast, and isn't that the main reason to learn anything?

## Why the 6502?

You may wonder why we are learning 6502 assembly, especially after I said it is incompatible with modern x86-64 assembly. Here are my reasons:

* 6502 assembly has fewer instructions than modern assembly. This, in my opinions, is a really good reason to learn 6502. 6502 assembly has around 50 instructions while you can argue (it's complicated..) that x86-64 has around 3000 [2]. This means that will be able to  realistically understand the entirety of the 6502 instruction set and get into how it works. With only around 50 instructions as well I find it less intimidating than the idea of learning modern assembly. Even if you do want to learn x86-64 I would argue this is a good place to start. The basic idea of assembly will be the same regardless of processor, so you may as well start on the smaller instruction set and work your way up. By having the smaller instruction set you also open yourself up to having to face some interesting problems. For example, there is no multiply instruction on the 6502!
* The instructions on the 6502 are varied and compared to other processors at the time offer a lot of options (in certain cases - we'll discuss exactly what is meant here later).
* The 6502 was incredibly popular. It was used in the Commodore 64, the NES, the Atari-2600 and more. Hundreds of millions of 6502 processors exist [3]. By learning 6502 assembly you are able to develop things for all these classic systems and understand their hardware.
* I think its fun, but maybe I'm a nerd.

## What Next?

There is a lot of ground to cover on this subject and lots of different ways to talk about it but my idea is to split it up like this:

* Prerequisite knowledge - how do binary and hexadecimal numbers work, the 6502 hardware, etc?
* A simple assembly program and how to run it.
* Slightly less "simple" assembly programs.
* Memory Addressing (Part 1)
* Memory Addressing (Part 2)
* IO (Part 1)
* IO (Part 2)

And then we'll see how it goes! Hopefully I'll have a new post each week, to keep up with book club. Most of the content I talk about will come from two main sources: "6502 Assembly Language Programming" by Lance A.Leventhal and "Programming the 6502" by Rodney Zaks.


the end.

<hr>

[1] https://stackify.com/popular-programming-languages-2018/

[2] https://stefanheule.com/blog/how-many-x86-64-instructions-are-there-anyway/

[3] http://www.westerndesigncenter.com/wdc/
