---
title: "[WIP] A very minecraft-y Turing Machine"
layout: post
categories:
  - Minecraft
  - ZHAW
  - Turing Machine
tags:
  - draft
---

![Minecraft Turing machine](http://edualc.github.io/images/minecraft_turing_machine/overview.png)

Back in spring 2016 I attended a course called THIN (theoretical IT) where as part of an assignment each student was given the task to build a working example of a Turing machine in any way, form or shape. If it ended up being a fully functional [Turing machine](https://en.wikipedia.org/wiki/Turing_machine), you got the full points. Most of my fellow classmates programmed their Turing machine either as a simple web app with a beautiful UI or they kept it as simple as possible using just the console output.

There were two different approaches alltogether. Two students paired up wanting to build not just a working Turing machine, but an [universal Turing machine](https://en.wikipedia.org/wiki/Universal_Turing_machine). This means it not only could compute anything that a Turing machine could, but it would also be able to read its instructions from the given input. I don't want to go into too much detail, if you're interested in the difference, the [Wikipedia entry](https://en.wikipedia.org/wiki/Universal_Turing_machine) describes it pretty well.

The other approach is my own, which I want to describe further in this post. I set out to build a working Turing machine in Minecraft without using programmable Commandblocks, but instead build a circuit that visibly shows the inner workings.

---

## Basic Turing machine understanding
> #### Disclaimer
> To follow along with my explanations you should bring at least a basic understanding of a Turing machine. I will try to explain the nitty-gritty details concerning the Minecraft implementation but I'm probably not the best person to explain the Turing machine as a whole without leaving out some important details.

Still, here is a quick recap:
A Turing machine has a **tape** divided into **cells**. These cells contain **symbols** - for simplicity we can think of zeros and ones as in our computers. There is a **head** resting above one cell at a time that can scan which symbol is contained in it. The **state register** holds the current state of the machine and lastly there is an **instruction table** containing all the possible **transitions** from one state to another, according to the current cell's content. Every transitions either moves the head to the left or right - and that's it!

---

## Minecraft's Redstone Toolset
Initially I had to try out different circuits in Minecraft using their Redstone circuits. The base game at that time (Minecraft 1.9) had fewer items available to play with than today.

Minecraft's **Redstone Dust** offers a dust-like material that can be placed on top of any block in the Minecraft world, leaving behind a red line. It can be compared to copper strips on a board, transporting current (here called power levels). Redstone power levels are from 0 to 15, with 0 being not powered and 15 being fully powered. If a trail of redstone dust or any other block outputting a signal points into a block, that block radiates that power level to neighboring blocks reading from said block. Furthermore, redstone power decays at a rate of one per block travelled. There are exceptions where a signal strength can be kept at the same rate indefinitely, but this would involve much more complex wiring than just the regular Redstone dust.

![Redstone Dust](http://edualc.github.io/images/minecraft_turing_machine/redstone_dust.png)
> Redstone dust (note how the signal decays in strength)

A **Redstone Torch** acts as an inverter if placed on a block or as a maximum level (15) power source.

![Redstone Torch](http://edualc.github.io/images/minecraft_turing_machine/redstone_torch.png)
> Redstone torch either powering dust or negating the input signal

The **Redstone Repeater** is a block that takes an input power level and refreshes its power level back to 15 if the input was greater than 0.

![Redstone Repeater](http://edualc.github.io/images/minecraft_turing_machine/redstone_repeater.png)
> Redstone repeater refreshes decaying signal

The **Redstone Comparator** has a variatey of uses. Its main functionality is to compare the input with its side inputs and output the difference. There is a secondary option in using it where it acts as a subtractor.

![Redstone Comparator](http://edualc.github.io/images/minecraft_turing_machine/redstone_comparator.png)
> Redstone comparator with uneven input signal strengths

There are other minor redstone blocks, such as the **Lever** which can be toggled to either output a full signal (15) or no signal at all (0) or the **Button** which can be pressed to output a short pulse.

![Pressure Plate, Button and Lever](http://edualc.github.io/images/minecraft_turing_machine/redstone_pressure_plate_button_lever.png)
> Pressure plate, button and lever

Using the blocks above one can combine them to build small contraptions which act as T-Flip-Flops, Timer Circuits, etc.

![Redstone AND Gate](http://edualc.github.io/images/minecraft_turing_machine/redstone_and_gate.png)
> AND Gate (inputs are on the top side, output on the bottom side)

![Redstone OR Gate](http://edualc.github.io/images/minecraft_turing_machine/redstone_or_gate.png)
> OR Gate (note how redstone signals can only travel through a repeater in one way and not the other)

Mumbo Jumbo has [a great video](https://www.youtube.com/watch?v=9EY_XoEImjM) on the basics of Redstone logic gates - I can highly recommend it!

---

## Prototyping

The tape needs to have cells with content. I originally intended to use containers (such as chests) from which a Redstone signal could be read. Different amount of inputs would result in different signal strengths to show all the possible symbols on the tape. Theoretically it is possible, but from a realistic standpoint, I realized it would only be viable for a very limited amount of symbols and I didn't know at the time how many states, symbols or transitions I would end up with.

![Tape: Reading From Chests](http://edualc.github.io/images/minecraft_turing_machine/tape_reading_from_chests.png)
> Tape: Reading signal strengths from chests (note the different signal strengths in the brightness of the dust)

As for the head, Minecraft has a rail system using minecarts which can contain items. I would've loved to use them to visually represent a moving head across the tape, but loading and unloading the minecart with precision seemed unfeasible.

![Tape: Reading From Minecarts](http://edualc.github.io/images/minecraft_turing_machine/tape_reading_from_minecarts.png)
> Tape: Reading signal strengths from minecarts

Instead, I started focusing on the instruction table with all the programmable transitions and this is where I had a breakthrough. By using Redstone lamps (blocks that emit light when powered with any Redstone signal) I could visually represent the input flowing through the instruction table. There was no need for a moving minecart anymore if I could just build small displays for each cell that would show both the current content and the head position.

![Instruction Table: Visually seeing a transition](http://edualc.github.io/images/minecraft_turing_machine/instruction_table_mechanic.png)
> Instruction table: Visually seeing how an input signal travels through a transition. As configured, the signal in the middle can travel down towards the left lamp but not the lamp on the right side.

---

## Final Version

To simulate the whole Turing machine, there needs to be:
- A tape with multiple cells containing symbols (finite length is acceptable, since you could always prolong the tape on either end)
- A head positioned at a single cell, capable of reading the cell's content and moving to either the left or right neighboring cell
- An instruction table mapping the read cell content and the current state to a new cell content, state and direction for the head to move in
- A way of keeping track of the current state

To accomplish these tasks, the Minecraft Turing machine is split up into three distinct sections containing multiple copies of the same module.

```TODO: FORMEL? input/output/symbole... ```

#### The Tape

Each cell module contains three input lines for the symbols (0, 1 and 'blank'), two input lines for the direction ('left' or 'right') and three output lines for the currently read cell content (0, 1, 'blank')


---

## Minecraft limitations

- loaded world size
- limit of redstone interactions per tick
- generally poor performance

---

## Performance

My Minecraft Turing machine ended up