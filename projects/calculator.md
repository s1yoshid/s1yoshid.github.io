---
layout: page
title: IR Remote Calculator
description: >
  How you install Hydejack depends on whether you start a new site,
  or change the theme of an existing site.
hide_description: true
sitemap: false
permalink: /calculator
---
This project was for my ECE 25 class at UCSD. An IR (infrared) remote calculator is a calculator which takes the IR signals from a TV remote, decodes the signals into numbers, adds 2 numbers and displays the result on the 7 segement display on the FPGA board. The circuit must be first turned on by flipping the 'enable' switch. Selecting Operand 1 or Operand 2 is each tied to a switch on the BASYS-3 board. When you want to input a value to calculate, you have to flip an operand switch on, press the value on the remote, then flip the switch back off. Do the same for the other operand. After both inputs entered, flip the 'result' switch on to see the result displayed on the BABYS-3 display.

### Equipment Used
* BASYS-3 Artix-7 FPGA Board
* TSOP348 IR Receiver
* 51k Ohm resistor
* Breadboard
* A few wires
* Function Generator
* Power Supply

### How TV Remote signals work

When you press a button the the remote, a beam of infared light pulses are send in the direction it is pointed. The tv is fitted with an infared receiver that takes the beam of infared light pulses and converts it into analog signal pulses that can be understood by electronics. Using light to transmit information can be an issue when many different light sources are present including the sun, so the infared light beam is modulated at certain frequencies to differenciate them from other lights.

The sony TV remote is modulated at around 40 kHz. The IR receiver we use removes this modulation and amplifies the signal so it is at a voltage level compatible with the FPGA.

The signal is a collection of fixed amplitude pulses. This encoding technuque is called Pulse Width Modulation (PWM) with the logic values represented by timing information instead of the amplitude of fixed width pulses. 
Note: PWM is commonly used to control the brightness of LED lights. The smaller the width of each pulse (duty cycle), the dimmer the LED light.

The sony tv remote signal is a group of 13 bits of data. The first 'start bit' is ~2.4 ms long. There is a ~0.6 ms delay between each bit. Low bits are ~0.6 ms long while high bits are ~1.2 ms long.

![Signal Waveform](/projects/assets/img/signal.JPG)

The first bit is the 'start bit' followed by 7 'control bits' followed by 1 'separator bit' followed by 4 'device bits'.

Start bit - Notifies the IR receiver that a signal has been sent and to start reading.

Control bits - Identifies the button pressed on the remote.

Device bits - identifies the device the remote is meant for (DVD, TV, etc.) For this project it is always 0000 for TV.

![SONY Remote Codes](/projects/assets/img/sony-remote-code.JPG)

### Building sequence identifier

To do this, we need to transform the analog waveform from the IR remote into a sequence of bits. This is accomplished by sending the signal to an input of a sequential circuit and then clocking the circuit (sampling) at a known rate. We set the sampling rate at 3.9 kHz or 0.256 ms.
Since the 'start bit' is 2.4 ms long, if the sampling rate is 0.256 ms, we should see a 10000000001 or 100000000001. We do not know if it is 9 or 10 '0's as the remote signal is not synchronized with the sampling clock. The same issue stands with identifying the 'one' and 'zero' bits of the remote signal.

![Start bit senarios](/projects/assets/img/startbit.JPG)

Three combinational logic blocks are created for identifying each type of signal: 'start bit', 'one bit', and the 'zero bit'. In each block, we have a 12-bit SIPO shift register. Each clock, we check the 12 bit parallel output and see if the sequence matches any type of signal. 

For the 'start bit' we need 10000000001 or 100000000001. 

For the 'one bit' we need 100001 or 1000001. 

For the 'zero bit' we need 1001 or 10001.

![Sequence Recognizer](/projects/assets/img/sequence-recognizer.JPG)

This is what the waveforms should look like for the sequence recognizer.

![Sequence Waveform](/projects/assets/img/sequence-waveform.JPG)

Next, we build a 12-bit shifter that starts when we detect a 'start bit' and shifts everytime we detect a 'one bit' or 'zero bit'. After 12 shifts, we output in parallel the 12 bit code that will be used by the decoder to decode the 4-bit numerical value of the code.

### Attaching the IR signal receiver

Attaching the IR signal receiver is simple as all you need to do is supply 5V to pin 3 from an external power supply. After that, connect a 51k Ohm resistor from pin 1 to GND. Also connect pin 1 to the 'data' pin on the FPGA. Also GND pin 2. Make sure to also GND the BASYS-3 board.

### Seven Segment Display

To display the values on the seven segment display, simply map the values 0-9 to the corresponding groupings of segments to display those values. For example, to display '1', drive the signals CB and CA high. To display in the ones place, also assert AN0 high. To display in the tens place, assert AN1 high.

![Seven-Segment Display](/projects/assets/img/display.JPG)

### Putting it all together

The 'clk' will come from a function generator generating a 3.9kHz 3.3V pk-pk square wave. Create a top module for where the decoder, adder, display, and shift modules will be instantiated.

![Full Diagram](/projects/assets/img/diagram.JPG)


