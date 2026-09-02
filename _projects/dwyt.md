---
layout: project
title: "Dancing with your Thumbs"
tagline: "Four-lane input-synchronized rhythm game for the DE1-SoC."
order: 2
status: "complete"
role: "RTL design + verification"
organization: "UW coursework"
date_range: "May 2026 - Jun 2026"
tech:
  - SystemVerilog
  - ModelSim
  - Quartus Prime
  - FPGA
repo: "https://github.com/Ae0lis/dancing-with-your-thumbs"
demo: ""
thumb: "projects/dwyt/dwyt_hero.jpg"
header_image: "projects/dwyt/dwyt_hero.jpg"
placeholder_color: "#34598a"
placeholder_label: "Rhythm game"
featured: true
category: "FPGA"
---

Dancing with your Thumbs is a four-lane rhythm game I built in SystemVerilog and deployed on DE1-SoC's FPGA. This was a solo project that I designed to target a 16x16 LED matrix add-on for the FPGA, and it ran successfully on the physical hardware. It features four lanes, each of which generates notes at pseudorandom intervals. The notes travel up the lanes until they reach the top, and the player must attempt to press the corresponding button at the right time to score. Getting perfect timing awards two points, and being off by one row awards one. Pressing early or letting the note run off the top deducts two points. With four different difficulty settings and a point tracker that goes up to 9999, Dancing with your Thumbs provides a fun experience for a variety of skill levels.

![The game being played](/images/projects/dwyt/dwyt_gif.gif)

## Context

I originally designed DWYT as the final project of EE 271 at the University of Washington. We were given the option of a few different games and similarly complex projects and were allowed to choose which one we wanted to implement. I chose ‘Dancing with your Thumbs’, as I’ve always had a soft spot for rhythm games and I figured I could do a pretty good job at it. The spec was pretty bare-bones; four lanes, lose 2 points on a miss, get 2 points on a perfect and 1 point on a good, and somehow display those points. That gave me a ton of freedom to implement the game how I wanted and to add additional features to improve player experience. Finally, we had an additional goal to make the design as physically small as possible. The size of a design was measured by the total number of combinational ALUTs combined with the total number of dedicated logic registers. Spoiler alert - I got it down to 524. For context, early attempts started at over 800! Keep in mind, those designs didn’t even have all the features of the final product. A roughly 35% size reduction while adding features at the same time is pretty good in my book, and I’m quite proud of it.

## My Role

I received two files from the course: the clock divider and the LED driver. Besides those two, I was responsible for writing, testing, and deploying all of the HDL in this project. 

## Architecture & Gameplay

The game itself is made up of four lanes, each a chain of FSMs. Each individual row in a lane is a separate FSM, and the rows watch their neighbors to determine how to change on the next cycle. This means that while the bottom row has to contain logic for generating notes, the rest of the rows can just pass notes along! The top three rows of each lane are green, yellow, and green to signify their status as good, perfect, and good rows respectively. Notes appear as red lines that travel continuously up a lane. When a player scores, all of the top three rows will light up corresponding to their timing. If they got a good, all will go green, whereas on a perfect all go yellow.

One of the core elements of this game is a special type of register known as a linear-feedback shift register (LFSR, for short). A maximal length LFSR will cycle every possible state except one in sequence. For example a 4-bit LFSR will cycle through 2^4-1 states. A deeper explanation can be found on [Wikipedia](https://en.wikipedia.org/wiki/Linear-feedback_shift_register), and the tutorial I personally used can be found [here](https://www.edn.com/tutorial-linear-feedback-shift-registers-lfsrs-part-1/). I used a 10-bit XNOR LFSR, meaning it cycles through 2^10 - 1 states with the one state missed being the state where all bits are 1.

LFSRs have a very specific feature that’s really useful for random generation. Due to how the register cycles through states, it produces a random-looking distribution. This means you can get pseudorandom number generation that you can use to drive gameplay mechanics! In the case of this game, each lane will roll a number every clock cycle via an LFSR. If that number is below a certain threshold, it will spawn a note. There are two important exceptions. The first is if a note already exists in that lane, as the game isn't built for multiple notes in one lane. The second is a bit trickier. When the player either scores or misses, no notes will spawn for the next game update. In addition, during the 'flash' animation for scoring good or perfect, no new notes will spawn. I found when playtesting early on that notes would often spawn in while my finger was still on the button, causing me to lose points unfairly, so I built this system to protect against that. I felt this was a better solution than just preventing spawns if the button was depressed as it prevented cheese strategies such as holding down all buttons except one to make the game much easier. 

The player can select from four different difficulty settings, which I've termed easy, medium, hard, and extreme. `SW[8:7]` on the DE1-SoC determine the difficulty, with easy being 0 and extreme being 3. Difficulty determines the update speed of the game, and ranges from around 6 updates per second to around 24. An increased number of updates means the notes travel faster and appear more often, meaning the game gets substantially harder with each setting. 

To prevent metastability, inputs are captured by a two-flip-flop synchronizer. The first flip-flop may go metastable temporarily, but the second will prevent that from messing with the rest of the circuit. Once the input is captured, it is held until the next game update to ensure that every button press is registered properly. This means that even if the player removes their finger early, the game will still recognize it as a press and score accordingly.

![A block diagram of the architecture](/images/projects/dwyt/dwyt_diagram.png)

## Implementation Challenges

Starting off, I had already designed the comparator and LFSR for other projects. I also had preliminary versions of the counters and score display, but I had to modify both substantially. I began by getting the very basics down. I designed a row of lights at the top to be the perfect scores, and set up the four columns. I initially wanted to do this system row by row, but I realized quickly that columns would be substantially easier and more efficient. Getting the good lights set up was trivial from here, so I did that early on as well.

The first big hurdle I ran into was the note speed. If I ran it at the speed of the clock, the notes would pass far too quickly. I initially used the clock divider to make a second clock to run the notes, and while this did actually work, it felt too sloppy. So, I went back and changed it to a counter enable system. This way, everything uses the same clock, but some aspects of the game (like the flash timer and the note speed) have a timer that counts up every cycle and they only step forward once it reaches a certain number. After this, of course, the timer resets and the cycle begins again.

Incidentally, this helped resolve another major design issue - flashing. I wanted to provide some feedback to the player when they get a good or perfect note, and I figured the best way to do that would be to make the score bar for that column flash all yellow for perfect or all green for good. The separate clocks made this difficult, as to pause for long enough I needed yet another clock! Having everything on an enable system meant this was much easier to implement.

## Verification and Results

The finished design met every required behavior, earned all functionality, creativity, size, and early-submission bonuses, and received the maximum available score of 138/100. The final synthesis used 330 combinational ALUTs and 194 registers, giving a course size of 524, and the design occupied 211 of the Cyclone V’s 32,070 ALMs. Verification relied heavily on module-level ModelSim waveform testbenches. I used these to check that each individual piece worked before integrating it into the design. I also created a full-design ModelSim testbench to make sure everything worked together before I tried it on hardware. Speaking of hardware, I've attached a video demo below. Everything worked the way it was intended to! I certainly had fun playing it, so I count this game as a success.

<video controls muted width="970" src="/images/projects/dwyt/DWYT_demo.mp4"></video>

## What I'd improve

Going forward, if I were to do this again I would definitely want to delay missed notes for the same amount of time as successful ones. Every so often, I still run into those "double jeopardy" scenarios where a note will immediately spawn after I miss one. I'd also like to substantially improve the testbenches. Since I finished this project, I've gotten much more familiar with SystemVerilog testbenches and automated testing, and I'd definitely want to put that to use here. The current testbenches are manual waveform tests without scoreboards or assertions and therefore debugging can take quite some time. Besides those two changes, I'm pretty happy with this project. The game is fun, albeit simple. I'd love to revisit this and make it more interesting at some point - maybe adding proper sound effects, or the ability to load note patterns - but that'll have to wait for another day!
