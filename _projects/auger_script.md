---
layout: project          # always "project" — leave as-is
published: false

# --- REQUIRED ---
title: "Husky Robotics Auger Control Script"
tagline: "Script to collect and deposit dirt for testing in a robotics competition."
order: 3                # grid sort: lower numbers appear first

# --- METADATA STRIP (optional, fill what you can) ---
status: "complete"         # active | complete | paused  -> colored badge
role: "Firmware + scripting"     # e.g. "Firmware + board bring-up"
organization: "Husky Robotics"    # e.g. "Husky Robotics", "UW coursework", "Personal"
date_range: "May 2026"
tech:                    # shows as tags; first 4 show on the card
  - C

# --- LINKS (optional) ---
repo: "https://github.com/huskyroboticsteam/HR-pi/blob/main/C_Code/SystemsTesting/Automation/dirtSample.c"   # GitHub button; omit if none
demo: ""                 # video/paper/live link -> 2nd button; "" hides it

# --- IMAGES (optional) ---
# Files live in images/projects/ ; write only the part AFTER images/
thumb: ""                # card image, e.g. "projects/myproject-thumb.jpg"
header_image: "projects/auger-hero.jpg"         # detail-page hero, e.g. "projects/myproject-hero.jpg"

# --- CARD PLACEHOLDER (used only when thumb is empty) ---
placeholder_color: "#2b5797"   # background of the colored block
placeholder_label: "Auger Script"     # text on the block; omit to use the title

# --- ORGANIZATION (optional, for later) ---
featured: false          # set true to spotlight later in a featured row
category: "Embedded"     # Embedded | FPGA | Coursework | Personal — for future grouping
---

## Context

This project was built for Husky Robotics, UW's University Rover Challenge team. URC is a competition where teams design a Mars-style rover and put it through a series of mission tasks at a competition held in the desert of Utah. One of those is the science mission, where the rover collects a soil sample for on-board testing. The auger is the mechanism that gets it. It sits dead center on the rover: a drill that bores into the ground, pulls up dirt, and deposits it into a collection box for analysis. My job was to build the control script that runs that motion end to end.

The sampling sequence is the same every run, which makes it a natural fit for automation. It's also unforgiving to do manually, as much of the motion is timing sensitive. On top of that, the robot's controls are built on embedded C spread across several files, each requiring its own set of unintuitive hardware constants for inputs. Automating it was essential, especially given the importance of the motion. The auger is the heart of the science mission, as all of the rest of the points come from the onboard testing performing well on the dirt it collects. If the auger didn't run, there wasn't much else we could do to score.

## My Role

I’m part of the instrumentation team for husky robotics, which means my work focuses on the various onboard scientific instruments and tools. To be specific, I’m on the electronics subteam, which means working on anything and everything related to the electronics. Depending on the day, this might mean soldering, crimping, writing firmware, testing sensors, or any number of other tasks. In this case, three days before the competition (when the actual assembly was finalized) I was assigned the task of making the script for the auger. I had already built the collection box’s open/close control and the auger’s PWM map, and the rest of the relevant pieces of code were written by other teammates. I just had to tie everything together.
I'd shipped smaller firmware for the team before, like the temperature/humidity sensor communication code, but this was my first large C project. The big jump wasn't the individual pieces so much as getting them to cooperate.
This collaboration quickly caused a pretty major problem. I had to #include a number of dependencies from throughout the codebase, but each had their own main function. When #include-ing in C, C physically pastes in the entire file, which led to a definition conflict. We still needed each of the other files to have their main methods (in case we needed to manually raise/lower the column, for example) so I had to figure out a workaround. At first, I considered patching only the files I needed, but I decided to check with my lead, and we agreed this was worth fixing across the full codebase. There was likely more automation to be done, and this would be a headache each time. Thus, before I even started on the file, I also refactored the entire codebase to guard every `main()` behind an `#ifdef` (the `BUILD_DSAMPLE_MAIN` guard at the bottom of this file is one of them) and rewrote the makefile to define the right guard per build target. It was more work than my task strictly required, but it cleared the conflict for everyone downstream, not just me.

## Approach

The technical meat. Architecture decisions, the hard parts, tradeoffs. Drop in
images, schematics, or code where they help:

<!-- ![Block diagram](https://Ae0lis.github.io/images/projects/myproject-diagram.png) -->

```c
// short, illustrative code snippets are great here — keep them tight
```

## Outcome

What happened? Results, measurements, what you would do differently, what you
learned. Numbers land well: "closed-loop control at 1 kHz", "cut boot time 40%".
