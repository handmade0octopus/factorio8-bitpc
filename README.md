# 8-bit Turing complete computer in Factorio

Easy to read/view version of this file: https://imgur.com/a/tVB9xOx

Hello everyone. I wish to share with you project I have done in Factorio using logic that this games offer.
Project was inspired by great mind who put step-by-step guide on how to make nearly same machine but in real life. I encourage you to watch it, as it will help you understand and replicate this project.
https://www.youtube.com/watch?v=HyznrdDSSGM&list=PLowKtXNTBypGqImE405J2565dvjafglHU

I bow down to Ben Eater, who taught me so much with this channel and I wish to dedicate this little project to him. Keep it up bro!

Here is computer calculating Fibonacci number and once limit of 8 bits (number 255) are exceeded it conditionally jumps to start over again:

![gif](https://giant.gfycat.com/BigNeighboringAttwatersprairiechicken.gif)

_[Full version](https://gfycat.com/bigneighboringattwatersprairiechicken) <==_

## Let's get down to understanding how is this computer made. And don't be afraid, once you get some basics I'm confident you will make one too!

First start with general computer layout. Here I highlighted areas of interest. I'll explain how I build them later on.

CLK - clock, that gives our machine synchronisation. Our modern CPUs are capable of doing 4-5GHz (4-5 000 000 000 Hz). My current setup allows to run it on 2 Hz, below that limitations of Factorio logic gates comes into place as each input have to be calculated for each Combinator (gate) so if we have 10 in a row we have to wait 10 game ticks (fps) before we can clock the system once more - otherwise the signals will mess up and the calculation wouldn't be performed.

PC (Program Counter) - the counter tells us what part of the program we are at. The programs are read up from 16 byte (one byte contains 8 bits) memory. Counter can count up to 16 (4 bits) in Binary (0000, 0001, 0010, 0011, 0100, 0101...1111) so each of those calculations give us register address which we later on can take up from memory and do stuff with it. It also features Jump - which resets the counter and then we can input another value to go into the specific place in our memory/code.

BUS - main hooking point of all the computer components. We can read or send data from/to it. To do that, we use control signals that opens/closes gates on each component so there are never more than two open (so data doesn't mix up).

ALU - our 'calculator' that performs adding and subtraction operations (in more advanced CPUs it can do much more!). It instantly takes whatever is in registers A & B and performs logic based operations (we chose which operation with our instruction decoder). This data then can be outputted on the bus. it also stores flags for us that we can use for conditional jump functions.

A & B Registers - they can store 8 bit number that is later brought together in ALU. Both can input and output data from/on the bus.

Register address/Decoder (RAD) - reads 4 bit address from the bus and decodes which part of RAM are we reading. For example 1110 address hold 0000 0011 value inside (pic).

RAM - modern PCs have usually around 16 GBytes of memory (16 000 000 000 bytes). We only have 16 bytes... This gives us enough space for 16 instructions, or less number of instructions and then we can store some data/variables in other places in memory. RAM here is basically 16 different registers like we use elsewhere, they are just access by register decoder.

Instruction register (IR) / decoder (DC) - we can put data to the instruction register from the BUS and then decode it to tell how program should perform. Only 4 bits (cyan) are used and it gives us 16 different types of commands that we can program. Let's imagine we have OUT instruction that prints whatever is stored in our A register. It's hardcoded as 1110 - so if instruction like this will enter our register we can decode it and tell our computer how to act.

Microcode counter (MC) - is like program counter but internal to instruction decoder. It gives us ability to step-by-step execute each command.

LCD/Screen - is actually a register but more sophisticated as it prints it's content on a LCD screen (Lamp-Combinator-Display)

Switch board (SB) - the board shows us which switch functions are we sending to control each computer components. There are currently 17 different switches controlling various things. For example if we want to read from the BUS to the A register, or write to the memory or instruction register etc. The switches beneath can be used for manual control of the machine.

Flags (F) - register to store flags (carry out [T] - when we exceed 8 bit values while adding, and 0 out [O] - when sum/subtraction is equal to 0). It will help us for our conditional jump instructions.

![1](https://i.imgur.com/u42tHVX.jpg)


## Let me first explain each component and at the end we will see how to actually program the computer as I think it will be more clear for everyone. If you are here just for programming skip to the last part.

CLK - our clock. Bread and butter of every computing. I wanted to build the clock so it has same time being high [C = 1] and low [C = 0].

(1) Here is base constant combinator that feeds our clock.  It goes to (2) where input and output are tied together. This configuration makes every game tick (UPS) value [C] is increased by 1. When it reaches [Z] it resets to 0. So Z is telling us how many game ticks are needed for clock to reset. There is also simple divider by 2 underneath that gives our clock half of the time on high as on low. When C is smaller than value [Y] (which is half of [Z]), clock is high, otherwise, its low.

This (4) inserter functions as our secondary clock if we want more control over the ticks. Putting anything (I used wood, just out joke for 'wood running PC'..) into the first Chest will make clock tick. If we want 5 ticks, just put 5 things in there.

(5) is our first control signal. [H] stays for HALT {HLT} command. When it's low [H = 0] clock runs an normal, when high, it goes into manual mode. Control gates help that, they are (5a) for normal work, and when [H] signal is not equal to 0, then the manual mode is engaged and [C] (our CLK) is outputted.

I also made inverted signal with gate (6) - when output is low, the inverted signal is high. I do not use it here, but it's good to remember how to make it when we will use it in the future.

[C] signal goes around the system with green wire. I tried to put it on completely separate wire (our BUS is on red wire for example) so it's easy to track and does not confuse itself with different signals.

![2](https://i.imgur.com/w9BjjeF.jpg)

## Registers - don't be scared with this. It's probably most complicated part of the whole system, but it's crucial to understand working whole machine.

Registers hold (store) values. In normal life, we would have to make each of those for each of 8 bits and other signals. But fortunately, Factorio allows multiple signal on one line. They are basically J-K flip flops.

To TL;DR how they work. On each clock pulse, they clear whatever they got inside, and store input value. If there is no value incoming (all 0), they clear out on a clock cycle. Of course we don't want always want them to clear out, after all we want to store value in there right? This is why we use control logic that I shall explain now, and we will take the black magic of the construction of the flip flop later.

The values (1) stored are shown with lamps. Lamp on means 1, off means 0. As we can see, we are currently storing value 1110 1001.

To output value to the bus, we are using control logic on gate (2). When signal [K] is low, this gate outputs whatever is inside register to the main bus.

Why we use it when signal is low instead of high? When, because logic gate outputs whatever goes into it (the red * sign), we would end up with bus with signal [K] - and we don't want that, we only want [7, 6, 5, 4, 3, 2, 1, 0] to be there. For same reason, we need to filter out control signals with gate (3) so we only get [K] when we need it.

Gate (4) is pinned to both bus (red wire) and control signals (green wire). Same as before, when [A] is low, register gets it's inputs. To filter out rest of the signals we use logic gate (4a). It basically get all the inputs both from bus and unwanted control signals, and adds it to the (4b) combinator which inputs always signals [7, 6, ... 0] = 1. Then, if any signal is equal to 2, it outputs each of those signals = 1. Simple right? This way only the things from the bus that matter to us will enter registers (0 values will still be  1, it blinks for a tick and then stays off during whole high clk cycle.

In this situation, when [C] goes high, gate (6) outputs [BLACK] signal, and gate (6a) negates [C]. But, because negating takes 1 more UPS, in this little time, gate (6) outputs the signal.

This signal is then carried out to gate (7), and it also opens briefly. Gate (7b) negates [BLACK] signal so it's not stored on gate (8), which functions as a holder for our signal. It does that in similar way as our CLK circuit, because both input and output are tied together. If there is no input, it will stay like this. If we notch our clock once more, without inputting any more data, gate (7a) will input negated signal of whatever is saved in the register to clear it out.

Now, if we know how edge detection and registers works, we nearly know everything.

![3](https://i.imgur.com/QYGjkHF.jpg)


## ALU - it adds/subtracts whatever is inside register (A) and (B) at all times. We only control weather to output in on the bus [Z], or if we want to change mode to subtraction [S].

How does it work? To fully understand it I really recommend watching some of Mr Ben Eater videos, as explanation would triple contents of this page.

I will just explain how to build an adder like this in Factorio.

For this, we need 3 types of gates. XOR (1), AND (2) and OR (3). Fortunately they are quite easy to make. Now, because we can use multiple signals in one line, so does our first XOR and AND gate can be simplified to just 2, and we don't have to do all of them for all 8 bits. This way, we can do (4) part of the circuit and duplicate it for every bit.

Subtraction is done by signal [S] which inverts signals coming from (B) register.

ALU also outputs carry out (when sum exceeds 8 bit) and carry zero and stores it in register on right (F on main computer picture).

![4](https://i.imgur.com/itV4Vkj.jpg)

## LCD / Screen - it looks scary, but to be hones it's one of the easiest things to make. It just take time to hook it up properly.

First, we make register, that input is controlled with signal [P]. Then, we multiply each bit with its value, in process converting it to decimal to get same signal with decimal value (it's kinda factorio cheating, but lack of programmable EEPROMs here doesn't allow us for much play there). To convert, we just need to get first bit [0] and multiply it by *1, then take second bit [1] and multiply *2, 3rd [2] * 4.. and so on. In process we just take output to some arbitrary value that we will use to determine which number we have (in this case [WATER DROPLET]).

LCD is lit by 9 stages for numbers (3). We just need to set on lamps which stages fits where (1), and then use gates (2) to output value where exactly we want. Just remember to get separate constant combinator (3) and hook it only to one special gate (2). Then just wire up all lamps together and give them instructions in which stage they are (1).

![5](https://i.imgur.com/OuLLWvY.jpg)

## RAM / memory register (RAD) - here I'll roughly explain how ram is constructed.

We already know registers, which uses clock pulses to store value. RAM is just grid of 16 (in our case) different ones (2). The input to them is controlled by different register (1) - that stores 4 bit [0, 1, 2, 3] that tells it which memory cell we are pointing at. It's performed by address decoder (3), which works in similar way to our LED/Screen. Each gate takes value from constant combinator (in our case 1100 bin = 10 dec), and then outputs proper register signal name (in our case [M]) so we can access value (in our case 00110 0011).

It's also good place to mention how to program our memory manually. We can do it by [W] signal which is turned on or off with our constant combinator (4). Then, another combinator (5) lets us change address, and to input value we use another combinator (6). At the end we just put anything in chest (7) to manually clock in the values to RAM without touching our main computer CLK.

![6](https://i.imgur.com/ehsmblB.jpg)

## Program counter (PC) - it's role is to count at which step of the program we are currently at (1). When we start its at 0000, and this address is read from RAM and transferred to our instruction register for it to be interpreted. Once instruction is finished, we can increment our counter with signal [X], then it goes to 0001, and in next iteration, this address is to memory, and cycle continues.

Of course, sometimes we want to jump, or conditionally jump to other parts of programs. With signal [J], we can do that. Once it's low (low means active in our control cases), it resets, reads which address it should jump to from the bus, and stores it in the register (2). Once [J] goes high again, it's fed by edge detector (just below 2) to the PC.

Counter itself works similar way to CLK, but instead of ticking all the time, it ticks only on edge detection of CLK (actually only when X and CLK are active) - you can see it just above (1).

Then signal can be put to the bus with control signal [C].

![7](https://i.imgur.com/mHs5Gvn.jpg)

## Switch board (SB) - it's good moment to explain each control signal used for the program. 

Signals are split into two colours, greens goes left, reds goes right. Each signal from constant combinators are actually send as [-1] values. Then combinators set to * != 0 can output 1 signal. This way, when our control logic sends out signal [1], they cancel out, and we receive [0] - and for all cases, this is what we really want (read about it why in part where I explained registers).

[H] - halts the clocks (switches to manual), high means CLK is not switching.

[Q] - ram address register in, when high - ram address register will store value from the bus on next CLK tick.
[Y] - ram memory in, when high - ram will store value from the bus on next CLK tick (on address stored in address register).
[R] - ram out, when high - ram will output value on the bus on next CLK tick (from address stored in address register).

[V] - instruction register in, when high - instruction register will store value from the bus on next CLK tick.
[U] - instruction register out, when high - instruction register will output value on the bus on next CLK tick (only 4 last bits [3, 2, 1, 0]).

[C] - program counter out, when high - program counter will output value on the bus on next CLK tick (only first 4 bits [7, 6, 5, 4]).
[J] - jump address in, when high - program counter will set value from the bus on next CLK tick (only 4 last bits [3, 2, 1, 0]).
[X] - program counter count up, when high - program counter will increment on next CLK tick.

[A] - A register in, when high - A register will store value from the bus on next CLK tick.
[K] - A register out, when high - A register will output value on the bus on next CLK tick.

[Z] - ALU out, when high - ALU will output value on the bus on next CLK tick.
[S] - subtraction (ALU), when high - ALU will change its mode from addition to subtraction.

[B] - B register in, when high - B register will store value from the bus on next CLK tick.
[L] - B register out, when high - B register will output value on the bus on next CLK tick.

[P] - LCD/screen register in, when high - LCD/screen register will store value from the bus on next CLK tick, and will display it's value.

[W] - flags register in, when high - flags register will store from ALU: carry out (when exceeds 8 bit), carry zero (when ALU operation = 0000 0000).

[pink signal] - carry out flag on [T]
[cyan signal] - carry zero flag on [O]

Now let say, we want to perform action OUT - which takes whatever is in A register and prints in on the LCD/screen (register). To do it manually, we just need to turn on (by turning off constant combinator for specific letter) signal [K] (A register out -> bus) and signal [P] (bus - > lcd/screen register in), then clock CLK.

![8](https://i.imgur.com/9uQ0Kwc.jpg)

## Instruction register / decoder / microcode counter - this is where magic begins. Now we know how to manually control our computer, this will help us to understand what we need to do, so it can control itself.

(1) microcode counter will count up to 8 (can be lowered if these many are not needed), so we can do 8 different switch on/off instructions to perform actions within one instruction.
(2) instructions are read to register from bus, to do it we need to turn signals [C] (program counter out -> bus) and [Q] (bus -> memory address in), then read out ram [R] (ram out -> bus) to our instruction register [V] (bus -> instruction register), and also clock the counter up [X].
Because this needs to be done each time, I have connected exactly that (4) straight to microcode counter so it will happen each time this counter goes through 0 and 1 steps.

Once we have something in our register, we can use our truth tables that we made in similar fashion in RAM address register and LCD/screen printing.

The [D] values from both instruction register (always being higher than 8) and microcode counter (always being lower or equal 8) will be added together, and with this number we can have create some logic gates. This is done by gates (3).

Example shows instruction 0110 XXXX (48 + X in dec, which I programmed to JMP instruction) then being added to step 2 of our microcode counter, making it 50.
For another example, ADD command (0010 XXXX - 16 + X in dec), after step 0 and 1, microcode will be at 2, so registers 18-24 can be used for different part of code (in this case, we only need 18-20 as ADD is 3 step process).

(5) carry flags are handled by simple logic gates, input to them is locked unless carry out [T] or carry zero [O] is fed to the logic gates.

Below, my complete list of instructions implemented (feel free to add/modify your own!):

0 NOP   - 0000 XXXX - does nothing.

1 LDA X - 0001 XXXX - loads value from RAM address X to register A.

2 ADD X - 0010 XXXX - loads value from RAM address X to register B, then outputs addition and places it in A register.

3 ADD X - 0011 XXXX - loads value from RAM address X to register B, then outputs subtraction and places it in A register.

4 STA X - 0100 XXXX - loads value from A register and stores it in RAM in X address.

5 LDI X - 0101 XXXX - fast loads value from instruction register (only 4 bit value) to A register.

6 JMP X - 0110 XXXX - unconditional (will always happen) jump to X value (sets PC to value X).

7 JC  X - 0111 XXXX - jump carry (when carry out [T]) is true, jumps to X value (sets PC to value X).

8 JO  X - 1000 XXXX - jump zero (when carry out [O]) is true, jumps to X value (sets PC to value X).

9 OUR X - 1001 XXXX - outputs to screen value from RAM address X.

.

.


.

14 OUT  - 1110 XXXX - outputs to screen from A register (X does nothing).

15 HLT  - 1111 XXXX - stops clock (X does nothing).



Let’s write a simple program and see how it runs!

0 LDA 3 - we load value to A register from memory location 3

1 OUT - we display on screen from A register.

2 HLT - stops the CLK, which halts whole machine.

3 42 - stored value

So basically this program will output value saved in RAM address 3 (0011 bin).

So let translate it to binary:

0 Address: 0000

Value: 0001 0011


1 Address: 0001

Value: 1110 0000



2 Address: 0010

Value: 1111 0000



3 Address: 0011

Value: 0010 1010


So now, to write program we need to go write to our memory (W on memory panel - see RAM picture part for explanation), start from address 0000 and inputs inside value 0001 0011 (0001 means LDA instruction, 0011 being X - address 3 in our memory).

Then do it the same for other instructions.
Remember to turn [W] back to green light, and keep halt on the counter.
You can also reset PC by jumping J for a while (don't need to tick the CLK).


![9](https://i.imgur.com/38YNIUy.jpg)


Thanks everyone for support. Especially Ben Eater for being amazing teacher and inspiring me for doing this.
Credits:
http://www.sorek.uk/
