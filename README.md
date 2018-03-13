# AuraUAS

AuraUAS is a complete autopilot ecosystem.  (It does not run on
pixhawk hardware, it is not an apm or px4 software port.)

## Hardware Architecture

- Teensy (arduino/teensyduino) processor used for sensor I/O, RC
  subsystem, smart mixing, stability damping, and auto/manual safety
  switch
  
- Embedded Linux computer runs EKF, controls (PID's), mission manager,
  logging, ground station communication.
  
- This big/little processor architecture pushes the critical hard real
  time tasks onto the Teensy, freeing up the linux computer to run the
  more computational intensive code.
  
- The reference hardware design can be self assembled for around $100.
  Professional grade boards are also available for purchase from
  Bolder Flight Systems.

## Design Principles

Every system has core values and goals that focus the development
process.  Here are some of mine:

- Robustness - the system and the code can never fail.

- Simplicity - like weight when buiding an airplane, simplicity must
  be a focus from the very beginning.
  - Limits the number of CPU boards it supports.
  - Limits the number of sensors and external devices it supports.
  - Strong (and strategic) investment in python on board.
  
- Cost - the system should be affordable at a DIY level.

- Performance
  - 100hz main loop update rate.
  - Highly accurate and robust EKF.
  
- MIT open-source software license used throughout all the components.

- Fixed Wing Aircraft - this is a subtle point, but one that should be
  mentioned somewhere.  AuraUAS has been designed around fixed-wing
  aircraft.  It has never been tested on multi-rotors.

## Python

A key differentiator from other popular autopilot systems is a heavy
investment in Python on-board inside the critical main loop.

- 10 to 15 years ago, C++ was simply not used for embedded systems,
  but today it has earned it's rightful place as a valuable tool for
  embedded systems development.
  
- Today, python is simply not used for embedded systems.  It typically
  can't even run on small arduino class processors.  However, for the
  linux-based systems such as raspberry pi, beaglebone, gumstix,
  edison, etc. python is available, fast, and robust.
  
- A task written in python is often 30-40% the number of lines of code
  compared to the equivalent task written in C++.
  
- A function that crashes in python returns control to the calling
  layer and the program continues.  A function that crashes in C++
  aborts the entire system.

- Question: if the python code is robust and runs for days and days
  without errors, and if the memory footprint is stable and never
  increases during the program lifetime, and if the performance meets
  the 100hz goal, and if the code is far simpler, shorter, more
  readable, easier to write, easier to validate ... then why not use
  python in strategic areas where it's strength greatly benefit the
  overall system?