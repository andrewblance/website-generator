---
title: "Assembling and running code with CA65 and the Apple ii"
date: 2019-11-12T17:35:01+01:00
draft: false
description: "Using CA65 and an Apple ii emulator we will go through the steps of writing, assembling and running some code." 
tags: ["programming", "assembly", "6502", "tutorial", "CA65", "appleii"]
categories: ["bookclub", "6502"]
---

## Bookclub Week 5

The past weeks we have been learning a lot of theory but, like Pheobe teaching Joey how to play the guitar, we have not put it into practice. This week we will see how to write code, assemble it, and make it run on an emulator.

This is a multi-stage process. First, we will have to find a way to run any assembled code. The code we write here is for a 6502, not a modern processor, and therefore will not natively run on our machines. An Apple ii emulator called AppleWin will instead be used as the target machine. The next problem is we will need to find a way to transfer code written on our current machine to the emulator. A program called Ciderpress will be used to do this. This brings along with it a problem in itself as both AppleWin and Ciderpress only run on Windows. Therefore, we need to also install a program called Wine which will allow Linux and Mac users to run Windows applications. Writing the code we will try to run will be done on the machine you are currently using to read this post. Using cc65 we can assemble a 6502 program on our modern machines, even if we cannot run it.


## Wine

If you run Mac or Linux, obviously, you cannot run Windows programs. The programs we want to use to do this week's post though are exclusively on Windows. However, we don't want to install and pay for a large emulator or dual-boot our laptops. To avoid these thig we can use Wine. Wine (an acronym of Wine Is Not an Emulator) is described as not being an emulator or virtual machine. Instead, it is described as a "compatibility layer capable of running Windows applications". Honestly, I don't know what that means. It will though allow us to run the Window's programs required.

I'll provide instructions on how to install this on Mac and Linux:

### Mac

Open the terminal and enter the following commands:

* brew cask install xquartz
* brew cask install wine-stable

### Linux

Go to the terminal and enter:

* sudo apt-get install wine-stable

Some websites give instructions on how to install newer versions of Wine on Linux, but the above command works correctly as well [1].

### Let's get installing!

We need to download the two Windows programs we need. They are available here:

* Ciderpress: https://github.com/fadden/ciderpress/releases
* AppleWin: https://github.com/AppleWin/AppleWin/releases

Find a version you like the look of (the newest, probably..) and download the ```.zip``` file under that "Assets" dropdown.

Let's get these both working, starting with Ciderpress:

1) unzip the downloaded file.

2) navigate to the folder in the terminal.

3) using the magic on Wine we can install the ```.exe``` file provided! To do this type the command ```wine Setup403.exe```. If the ```.exe``` is not called the same thing for you then enter the appropriate name instead.

4) go through the installation process.

5) Now, with Ciderpress installed navigate to: ```~/.wine/drive_c/Program\ Files/```.

6)  Inside you should be able to navigate to a Ciderpress folder. For me, it was located at ```Faddensoft/Ciderpress```

7) Finally, to run Ciderpress, just type ```wine Ciderpress.exe```

And like magic, Ciderpress, a Windows program, should be running on your Mac or Linux machine.

For some reason, the AppleWin process is easier.

1) unzip the downloaded file.

2) navigate to the folder in the terminal.

3) The command ```wine applewin.exe``` should launch the application.

We now should have versions of AppleWin and Ciderpress we can use. I am not going to discuss it here but an application called WineBottler exists for macs. This aims to improve the Wine experience by allowing you to access these programs like any other normal Apple program.

## Ciderpress

Ciderpress is a disk image utility which gives us a way to create and manage Apple ii disks. Effectively, this will be a way to transfer things onto emulator from our machine. This will allow us to write a program in our editor of choice then send it to the emulator to run. Before we begin to use AppleWin we first need to do some preparation. We need to create a disk with an operating system on it for the Apple ii to boot into.

To do this:

1) open Ciderpress.

2) click ```File``` > ```New``` > ```Disk image..```.

3) under ```Filesystem``` select ```DOS 3.3``` and leave everything else as default.

4) give it a nice name, make sure it has the extension ```.do``` and save it in a sensible place.

We are now ready to play with our emulator!

## AppleWin 

The Apple ii is one of the most best-selling computers in the world. It was designed by Steve Wozniak in the seventies and, importantly for us, runs on a 6502 processor. There are lots of interesting and neat things about this machine but for today we will focus on getting some assembly code running. First, open the emulator using Wine. It should open something similar to the image below.

{{< cupperFig
img="app2.png"
caption="An emulated Apple ii"
command="Resize"
options="500x" >}}

Now, let's boot into it.

1) press disk 1 icon, this selects what disk will be in the Apple's first disk drive.

2) A file browser should open, find the ```.do``` file you just made and select it.

3) now press the rainbow Apple logo to launch the machine. By default, whatever is in the first drive will launch when this button is pressed.

Now, the emulator should boot up and give a nice, happy, beep. If this does not happen, you may need to open the options (using the button with a speaker and joystick on) and configure the sound device settings.

On the screen should be a ```]``` followed by another flashing symbol. This is telling us the machine is ready to accept input. Something cool here is that we can now just start typing code right into the first screen we see after launching the machine! This language is called BASIC. Try typing the following lines in:

```
NEW
10 PRINT "HELLO WORLD"
20 GOTO 10
RUN
```

This is an infinite loop that prints "HELLO WORLD". You can press the rainbow Apple button to reboot and stop it. The first line "NEW" clears the memory, and "RUN" will run the BASIC program we have entered. Interestingly, we have manually given the lines numbers. We have chosen 10 and 20 as this would give us space to add more in-between if, hypothetically, we wanted to add more functionality to this. BASIC, as you might guess, is a very basic language. So much so that you would be able to buy magazines with BASIC code for entire games you could write into your Apple ii and play.

A final command: typing ```CATALOG``` shows us any contents of the disk.

## Our Code

We have managed to get the emulator running now, hopefully. Let's move onto our code. The example we will use is based on last week's addition example. In fact, it is almost identical. The code we will use is below:

```
CLC      ; CLEAR CARRY BIT
CLD      ; CLEAR DECIMAL BIT

ADR1 = $6100 ; WHERE IN MEMORY ARE THESE THINGS
ADR2 = $6101
ADR3 = $6102

LDA #01
STA ADR1
LDA #02
STA ADR2

LDA ADR1 ; LOAD CONTENTS OF ADR1 INTO ACCUMULATOR
ADC ADR2 ; ADD CONTENTS OF ADR2 INTO ACCUMULATOR 
STA ADR3 ; TRANSFER CONTENT OF ACC TO ADR3

RTS
```

The only difference is the inclusion of the instruction ```RTS```. This tells the machine to return from the program. Copy the code into a file and save it as ```add.asm```

### ca65 

We will assemble this code using a program called cc65. Before we discuss it too much, we will try to get it installed.

For a Mac, in the terminal, enter the command:

```
brew install cc65
```

For a Linux machine, in the terminal, enter the command:

```
sudo apt-get install cc65
```

This will have installed cc65 - a program which can compile C code into something a 6502 machine can run. It can also assemble the file we have just written. Go into the folder you saved the code and run the command:

```
ca65 add.asm
```

This will have assembled our code. However, we are not yet ready! We need another stage, where we link this code, using a linker, to any other information it may need to know. This might be libraries it uses or, in our case, information about the machine it will be run on. This needs to be done so it knows the areas of memory it is allowed to access. At the moment, being compiled on a modern machine, it will not have any idea of the memory map of the Apple ii. We can provide the relevant parts of it ourselves.

```
MEMORY {
RAM: start = $6000, size = $8E00, file = %O;
}
SEGMENTS {
CODE: load = RAM, type = ro;
DATA: load = RAM, type = rw;
}

```

The above code should be saved in a file called ```apple2bin.cfg``` in the same directory as our assembled code. It is telling the linker where the RAM will start, where the program can store the code and data, and its read/ write privileges. To link this to our assembled file, and output a binary the Apple ii can run we need to run the following command:


```
ld65 -o add#066000.bin -C apple2bin.cfg add.o
```

Let's discuss this command:

Part 1: ```-o add#066000.bin```: this names our output. It is given this strange name to provide Ciderpress with some necessary information about its file type and its starting memory location.

Part 2: ```-C applebin.cfg``` tells the linker about the config file we made

Part 3: ```add.o``` - this is the code we assembled

We are now ready to begin loading this back onto the emulator.

### Running our code

To get our code to run we will first need to make a blank disk to store it on. Then, when we have this we can transfer our program file onto it and run it on the emulator. To do this:

1) In the emulator, press the button for Disk 2. 

2) A file explorer will pop up, navigate to a place you want to store the disk.

3) Enter a sensible name for the new disk and press ok

4) Now, in the Apple ii command line type ```INIT BLANK,D2```. This will format our new disk.

5) We should now see the name of our disk under in the disk 2 button.

Now, hop into Ciderpress. Here, we want to:

1) click ```Actions``` > ```Add files```

2) find and select ```add#066000.bin```

3) under "File Attribute Preservation" select "Use File Attribute Preservation Tags" - this is very important, it tells Ciderpress to use the information encoded in the name when adding it to the disk.
 
4) Now, press "Accept"

Now, in the Ciderpress window, we should see the file "ADD" in the disk. Hop back to the Apple ii. Lets actually run our program! Note, to see the effect of us modifying the contents of the disk we will need to close the emulator and open it again using wine.

1) Press the rainbow button to boot into the Apple ii.

2) Press the switch-drive button (has two numbered drives switching position on it) to load into our second drive.

3) enter the command ```CATALOG``` - we should hopefully see a new file called "ADD"

4) to now run the program, enter the command ```BRUN ADD```

The program we have spent 4 weeks making should now have run! Congratulations! However, we will notice there is no immediate output. That is because at no point in our code we told it to print anything. We just asked it to move things in memory. To see the effects of our work we need to look into the memory locations we have affected. The Apple ii, helpfully, gives us the power to do that. Run the following three commands:

* PRINT PEEK (24832)
* PRINT PEEK (24833)
* PRINT PEEK (24834)

This is telling the Apple ii to print the contents of the memory locations 24823, 24833 and 24834. These are all decimal values. In hex, they are written as 6100, 6101 and 6102 - exactly where we stored the two numbers to add up and the result. Hopefully, your results will agree with mine:

{{< cupperFig
img="add.png"
caption="Output on the Apple ii of running our program and Peek commands"
command="Resize"
options="500x" >}}

## Conclusions

Well done! You have finally written, assembled and ran your first assembly program. This same procedure we went over today can be used for all the code we will write in bookclub. Also, with the knowledge of how to transfer things onto the Apple ii you can download and run classic Apple games and programs. There is lots more to say about assembling and emulation, obviously, but I will just give a few recommendations of interesting things to use and play around with. First, there is easy6502 [2]. This is a website that has an in-browser assembler and emulator which you can use to write 6502 assembly to modify the colour and position of pixels. It even gives a great example of the game Snake. Another, slightly more advanced site, is 8bitworkshop [3]. This has a full 6502 IDE on it to use to make games for classic consoles. Finally, an alternative way to write and assemble your code is to do it directly on the Apple ii using an editor called Merlin [4]. A link to more information on all of these can be found below.

After a small book-club hiatus, I will continue by discussing slightly more advanced assembly ideas. Then, I will move onto a variety of ways the 6502 allows memory addressing.

the end.

<hr>

[1] http://tipsonubuntu.com/2019/02/01/install-wine-4-0-ubuntu-18-10-16-04-14-04/

[2] https://skilldrick.github.io/easy6502/ 

[3] https://8bitworkshop.com/

[4] https://www.youtube.com/watch?v=GG6tfYyzzbM
