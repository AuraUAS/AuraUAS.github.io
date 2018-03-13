# AuraUAS

AuraUAS is a complete, self-contained autopilot ecosystem.  It is not
derived from any of the other popular diy autopilot systems, but has
an independent development history.

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

- Simplicity - like weight when building an airplane, simplicity must
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
  
- Question: if the python code is robust and runs for days and days
  without errors, and if the memory footprint is stable and never
  increases during the program lifetime, and if the performance meets
  the 100hz goal, and if the code is far simpler, shorter, more
  readable, easier to write, easier to validate ... then why not use
  python in strategic areas where it's strengths greatly benefit the
  overall system?

## EKF

The AuraUAS EKF is a 15-state filter developed at the University of
Minnesota Aerospace engineering department.  The original algorithm was
developed by Professor Demoz Gebre in matlab, then ported to C for
real time use.  The current version has been wrapped in a simple C++
class. The internal matrix/vector libraries have been replaced with
Eigen3 (C++).  The internal code strategically uses single precision
floating point math where ever possible.

The inputs required are:

- 3 axis gyro (p, q, r)
- 3 axis accelerometer (ax, ay, az)
- GPS position (lat, lon, alt)
- GPS velocity (vn, ve, vd)

There is a variant of the EKF (included) that uses the 3 axis
magnetometer in the measurement update portion of the algorithm to
help track orientation in low dynamic portions of the flight.  The
accuracy of the results are highly dependent on the quality of the
magnetometer calibration and the amount of magnetic
interference/distortion.

The 15 estimated states are:

- Attitude: roll angle, pitch angle, (true) yaw angle.
- Location: latitude, longitude, altitude.
- Velocity: vn, ve, vd.
- 3 Gyro biases.
- 3 Accelerometer biases.

In the AuraUAS system, the EKF is updated at 100hz.

## Telemetry

AuraUAS currently does not support MAVlink.  I observed that all the
major autopilot systems essentially require running the paired ground
station software anyway (Ardupilot + Mission Planner, PX4 +
QGroundControl, etc.)  The reality is that MAVlink doesn't provide all
the interoperability one would hope for.

Instead I devised a simpler communication system, purpose built for
AuraUAS.  It is similar with a packet header and 2-byte checksum.

## Ground Station

The AuraUAS ground station uses a slightly different architecture than
other systems.  It is composed of two pieces:

### auralink.py

This is a python application that connects directly to the aircraft via
a radio modem.  It is responsible for:

- receiving and decoding aircraft telemetry messages.
- sending operator commands to the aircraft.
- logging the received data.
- hosting the ground station interactive web applications.

### web apps

The actual user interface is composed of a set of web applications.
The advantage is that you can run the ground station on any platform
of your choosing, ranging from windows, to mac, to linux, to android
to iOS.  You simply point your favorite browser on your favorite
device to the local web server link.

- **maps:** A traditional top down map is provided that shows the
  aircraft path.  This tool can also be used for route planning.

- **panel:** A simple aircraft panel is also provided.  This is
  modeled after a modern C172-S instrument panel with working gauges.

- **props:** An engineering/debugging page is provided for inspecting
  all the internal values available at the ground station.  This is
  live updated as the data is received from the aircraft.

## Mission Manager

A flexible task sequencing system runs on board the aircraft.  Tasks
are entirely written in python so they are easier to script.  They
have access to all the same internal state variables that the C++ code
has access to, so there is literally no limits (beyond time and space)
to what can be done as a python task.