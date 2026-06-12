---
layout: project          # always "project" — leave as-is

# --- REQUIRED ---
title: "Line-Following Robot"
tagline: "Autonomous line-following robot."
order: 10                # grid sort: lower numbers appear first

# --- METADATA STRIP (optional, fill what you can) ---
status: "complete"         # active | complete | paused  -> colored badge
role: "PCB design + board bring-up + assembly"     # e.g. "Firmware + board bring-up"
organization: "UW coursework"    # e.g. "Husky Robotics", "UW coursework", "Personal"
date_range: "Feb 2026 – Mar 2026"
tech:                    # shows as tags; first 4 show on the card
  - KiCad
  - CAD
  - Arduino
  - Breadboarding

# --- LINKS (optional) ---
repo: "https://github.com/Ae0lis/line-following-robot"   # GitHub button; omit if none
demo: "https://www.youtube.com/watch?v=ZdqaWTLLXEM"  # video/paper/live link -> 2nd button; "" hides it

# --- IMAGES (optional) ---
# Files live in images/projects/ ; write only the part AFTER images/
thumb: "projects/line_robot-thumb.jpg"                # card image, e.g. "projects/myproject-thumb.jpg"
header_image: "projects/line_robot-hero.jpg"         # detail-page hero, e.g. "projects/myproject-hero.jpg"

# --- CARD PLACEHOLDER (used only when thumb is empty) ---
placeholder_color: "#2b5797"   # background of the colored block
placeholder_label: "Line robot"     # text on the block; omit to use the title

# --- ORGANIZATION (optional, for later) ---
featured: false          # set true to spotlight later in a featured row
category: "Coursework"     # Embedded | FPGA | Coursework | Personal — for future grouping
---

## Context

This project was the capstone of EE 201, an introductory electrical engineering course at UW. This capstone was a competition; design the fastest, most accurate line-following robot possible over the course of one month, then compete on a faculty-designed track. This presented a classic controls problem. The robot had to sense where the line was, decide how hard to correct, and effectively actuate that course correction. The difficulty of this project lived equally in the design and manufacturing of the robot, and the calibration and behavior of the control system. My team was also a bit of an outlier: all three of us were in our first EE course, while most competing teams had juniors and seniors on them.

## My Role

My team had three members, and with only a month to build, we each wore several hats. For my part, I was responsible for the sensor PCB, board bring-up, configuring the code, and much of the physical build. I designed and laid out the photoresistor sensor board in KiCad. I configured and tuned the Arduino control software, mapping it to my sensor board and dialing in the PID constants against the track. Finally, I soldered the components together and assembled the entire sensing apparatus, including a last-minute cardboard light shield to improve sensor reading accuracy.

## Approach

The robot breaks down into three systems: a custom sensor array that reads the line, a control loop that decides how to steer, and a drivetrain that steers the actual robot. All three components are mounted on a given stock chassis. We prototyped a 3D-printed chassis early on, but unforeseen mechanical difficulties arose and the stock chassis functioned much better. Without time to redesign, we decided to work with what we already had. We did, however, print a battery holder for the motor battery combined with an Arduino holder to stabilize the wiring.
<!-- ![Block diagram of the robot's systems](https://Ae0lis.github.io/images/projects/line_robot-diagram.png) -->

### Sensing

This was the custom PCB. The robot's steering system takes input from a seven-channel photoresistor array that I personally designed and laid out as a custom PCB in KiCad. Each of the seven channels has a photoresistor and an LED to ensure the photoresistor encounters minimal noise from ambient light variations. The LEDs each have a 1 kΩ current-limiting resistor to protect them. The requirement for the control board was to have holes at most 50x100mm apart, but I shrunk it to 40x80mm without losing functionality. The ground and VCC are copper pours on opposite sides of the board rather than hand-routed traces, which certainly helped the size. After we ordered and tested the board, we found hanging it around an inch off the ground gave the most consistent black/white separation. Still, we ran into a few issues in extreme circumstances such as particularly bright external light, so I designed a cardboard light shield to reduce noise. After recalibration, it worked like a charm!

<figure>
  <img src="/images/projects/line_robot-sch.png" alt="Seven-channel photoresistor sensor schematic">
  <figcaption>Schematic of the seven-channel sensor array.</figcaption>
</figure>

<figure>
  <img src="/images/projects/line_robot-pcb.png" alt="Image of a 3D PCB render, with 7 LEDs in line with 7 photoresistors.">
  <figcaption>3D render of the PCB</figcaption>
</figure>

### Power

The original design of the robot had the full system running on one 6V supply, but we found that the motor's power draw could sometimes spike and affect the arduino. To rectify this, we split the power system into two; a 9V battery to power the motor shield, and the original 6V battery to power the arduino itself. This was a win-win, as it both protected the arduino from brownouts and gave the motors more power headroom to pull from if needed.

### Controls 

The steering runs on a PID controller provided as course scaffolding, which I configured for my board and tuned to a working setup. The pipeline starts with the sensor readings, which the controller takes a weighted average of. This provides an estimation of how far off-center the robot is, which is then used to drive the wheels at different speeds to pull the robot back towards center. Rather than designing the algorithm (the skeleton was provided to all teams) my role was to configure and tune it to work for our robot. I mapped the controller onto my sensor pins, connected it up to our potentiometer circuit, and together my team tuned the PID constants from that circuit until we were satisfied with the values on the serial monitor. 


## Outcome

Our robot successfully completed the full track. After calibration the sensors cleanly separated the surfaces, with a black line reading around 2500, and white around 1000.

<figure>
  <img src="/images/projects/line_robot-assembly.jpg" alt="Image of the line-following robot.">
  <figcaption>The finished robot</figcaption>
</figure>

The cart wasn't the fastest in the class, and there was one major reason why: length. One of the final turns had the cart draw close to a wall, and because we were using the bulky stock chassis, it scraped against the wall and stalled. There's a video attached below of the specific turn. This clip is from one of the earlier tests, where it didn't turn at all. We did eventually get it to turn passably, but I unfortunately don't have video.

<figure>
  <video controls muted width="560" src="/images/projects/line_robot-turn.mp4"></video>
  <figcaption>An early test; note the long chassis getting stuck on the wall.</figcaption>
</figure>

While the final design would eventually right itself, it still stalled for a good deal of time before it completed the turn. This cost us a substantial amount of time compared to our competitors. If I could do this again, I would definitely want to custom print a chassis. Not only would it save us time here, but it would make the robot lighter overall and therefore most likely faster. Still, despite the issues, my team still finished on the leaderboard. Thus, I count this project a success!
