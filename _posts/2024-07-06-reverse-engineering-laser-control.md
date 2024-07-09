---
title: Reverse-engineering USB control of a laser unit
categories:
- tools
feature_image: "https://picsum.photos/2560/600?image=872"
image: "https://jmtayloruk.github.io/assets/img/usb_icon.png"
---

We have several Vortran Versalase™ units giving multiple laser lines with a fibre-coupled output. I'm a big fan of them.
One of the only drawbacks is that there is no API for programmatically controlling the laser
(for example, turning different wavelengths on and off, altering the power, etc).
This can be done over an RS232 connection if you wire that up to their D connector (which is also used for other purposes).
There's a perfectly good USB port on the laser, which their own PC-only GUI-based program uses to interact with the laser.
However, unfortunately there is no information (or drivers) for how to control the laser programmatically,
and Vortan were not interested in providing any such information. So, I set out to figure this out for myself.
If you just want a python script to control the laser, [here it is](https://github.com/jmtayloruk/scripts/blob/main/versalase-usb-comms-demonstration.ipynb).
This post summarises *how* I was able to do this, in case it is useful for anyone wanting to do similar reverse-engineering in future.

## Wireshark

The key to this was the [Wireshark network protocol analyzer](https://www.wireshark.org).
I had not used this previously, but with some familiarity with what to expect from network protocols,
and a [simple tutorial](https://github.com/liquidctl/liquidctl/blob/main/docs/developer/capturing-usb-traffic.md),
I was up and running pretty quickly. I actually found it simpler than that tutorial implied.
Perhaps by luck, I had plugged the USB cable in to a port that did not have any other active devices on the same hub.
It was therefore clear when I had found the correct usbmon3 interface: without the laser plugged in, there was no traffic.
With the laser plugged in and Vortran's GUI running, I could see USB traffic between the GUI and the laser.

## Observing communications

I thought there was a good chance that the payload being sent over the USB link would be plain-text commands matching
the RS232 protocol documented in section 9 of the Stradus Versalase™ User manual. 
Indeed, I had even hoped that inside the laser unit was simply an USB-serial converter such as those made by FTDI.
(That might still be the case, but if so I did not manage to identify the manufacturer; the traffic does not look like an FTDI device).

Happily, the USB payload *did* match the documented text-based protocol.
When their GUI is running it is constantly polling the laser for status updates,
and the text payload is clearly visible in the USB packets.
In fact, the most common messages are *not* documented (status polling such as `?GUI1=1*1*1*1*1.0.4*1.0.0`),
but if we take an action through the GUI such as changing the power setting for one of the lasers,
a recognisable command appears in the USB traffic logged by Wireshark.

So, it was clear that the basic command structure was the same as already documented in their user manual,
and that saved a lot of effort. The remaining task was to figure out what the USB communication protocol was.
After some quick reading up on the basics of USB communications, it was clear that all messages were being
passed as "control transfers" rather than "data transfers". This makes sense in terms of the short,
command/response nature of the expected communications, and it certainly meant that the traffic was simpler and
easier to interpret.

From looking at the traffic, it was clear that it was not quite as simple as sending a single USB control packet containing the text command
and receiving a single USB packet containing the text response, but it seemed reasonable to assume that
there was a standardised sequence of USB packets that would be exchanged every time a command was being sent.
Since their GUI was constantly polling the laser for status updates, it was necessary to take a block of about
10 seconds of USB traffic and sift through it, looking at the timing and patterns of the messages to figure out
what was going on.  

## Observed communication protocol    

The main message can be seen to be very straightforward.
It is a control transfer with `bRequest=0x40` (which some quick reading about the USB protocol reveals indicates a message from host to device),
`bRequestType=0xa0` and a payload consisting of the text command.

This is then followed by a control transfer with `bRequest=0xc0` (indicating device to host direction) and
`bRequestType=0xa2`, which receives a single-character response `0`.

Then there is a control transfer with `bRequest=0xc0` and `bRequestType=0xa1`,
which receives the text response from the device.

We then see `bRequest=0x40` and `bRequestType=0xa3`, which gets a response of `1`.

A further read with `bRequestType=0xa1` gets a two-character response.

Finally there is another round of `bRequestType=0xa3`, which gets a response of `0` this time.

To figure this out was really just a case of starting with the Wireshark trace,
identifying the packets that contain the recognisable text commands, and then looking at repeating patterns
and relative timing of packets to identify the other packets that are exchanged in conjuction with each command.

## What is the protocol?

At about this point it made sense to try and identify this traffic as a "recognised" communications protocol.
That would make the traffic easier to interpret, and increase confidence that I have understood it all correctly.
Unfortunately after some googling and asking around, I was only able to identify what it is *not*:
- It is not the USB HID protocol. That is designed for communication with e.g. mice and keyboards, but it might not have
been completely outlandish for this to be used, since it is a standardised and relatively lightweight protocol.
Although what we see has a superficially similar structure to the HID protocol, if it was then we would see mandatory traffic
exchanged when the device is first connected.
- It is not [USBTMC](https://sigrok.org/wiki/USBTMC), which behaves a lot like a serial port but without having to
worry about baud rates etc like you would on a true physical USB-serial device. USBTMC uses USB data packets, which we do not see here.
- It does not match what I observe if I use Wireshark to monitor traffic through an FTDI USB-serial device.
I didn't keep a copy of what I *did* see there, but in retrospect it's possible that *was* USBTMC.

Just for fun, I also tried describing my observations to ChatGPT and asking it what the protocol is (it’s what our students would do…).
Unfortunately it made a series of wrong suggestions, and when I gave it the reasons why these were wrong,
it settled into a pattern of authoritatively echoing back to me what I was telling it. So no luck there.

The fact that the protocol seems more complicated than it should be 
reinforces my belief that this is some sort of "off-the-shelf" protocol that has been used because
a suitable library and device chip were available. Effectively, this would mean we are looking at the protocol at "too low a level",
like we would if we were looking at raw USB HID protocol packets without understanding the standardised structure of that protocol
(and using a suitable library that wraps that protocol, in our Python code).
Or indeed if Wireshark were showing us the raw bytes of the USB packets rather than it separating out the basic nuts and bolts of
the USB packets from their data payload.

It seems weird that Vortran (or the comms chip manufacturer) would invent their own vendor-specific protocol 
rather than using an existing off-the-shelf on - particularly one that seems more complicated than I think it needs to be.
However, as far as I can figure out, that seems to be what has happened. 
So the only option is to replay the pattern of messages that we see.  


## Replicating the communication protocol

Having identified what the pattern was, I attempted to replicate the command sequence by sending my own commands from Python code,
using the `pyusb` module. This required a little trial and error. 
One of the issues I encountered was that timing of commands has some importance. If we poll for a response before one is ready,
we receive an empty response. And if we send the wrong command out of sequence
(e.g. because my code ploughed on to the next part of the protocol after receiving an empty response!)
then the device can get unhappy and communications may stall until a 1 second timeout has elapsed.

But with those issues dealt with, I was able to successfully exchange commands and responses with the laser device over USB,
from a Python script that I had written.

## So what is the protocol doing?

For a long time I was puzzled why the protocol is as complicated as it is!
On some level the structure will be driven by the fact that USB control transfers are initiated by the computer,
which naturally lends a polling-based style to the communications.
Combine that with the fact that the host doesn't know when the device response will be available,
and we perhaps end up with a few different messages to be passed back and forth.
But it still seemed more complicated than it needs to be.
There seems to be scope for the host to read a response from the device more than once,
with the same response being "displayed" for reading until it is cleared through sending one of the mysterious anonymous command packets.

I got a clearer understanding of the motivation purely by chance, when I used my code with a laser unit that was configured
to provide the "Stradus>" prompt on its serial comms.
That revealed that the second 0xA1 read (which got an empty '\r\n' response) was now getting the stradus prompt.
Now the protocol makes more sense:  

## The protocol

bRequestType=0xA0 - send a text command in packet payload
bRequestType=0xA1 - read the current text response (if any) that the laser is "displaying" for us
bRequestType=0xA2 - check to see if a new text response is available (returns 0 or 1)
bRequestType=0xA3 - acknowledge that we have read the current text response (laser will now display the next one, if any)

Puzzlingly, it sees that we need to send the A1 command before we can send A2, 
even though the initial A1 may not receive any response yet.

So the sequence is: Send an A0 command, send an A1 read, poll A2 until it confirms a response is available, read that with A1, 
clear message with A3, and poll A2 again until it tells us no more responses are available. 

## Conclusion

In this (admittedly relatively simple) scenario it was pretty straightforward to reverse-engineer the communications protocol
between the computer and the laser, and write Python code to communicate with the laser myself.
Although I have reasonably good general appreciation of low-level computer issues,
I had never previously done any work with USB, and had no idea what USB protocols look like.
In spite of this, it took me less than a day from start to finish to work this out, from first wondering if I could do it,
to having a complete and working Python script we can use in our lab experiments.
Initially I could replicate the protocol as I was observing it, but I did not understand the purpose.
It look a bit more thinking (and luck) to feel like I understood *why* it is the way it is.

Epilogue: fresh and cocky from this success, I thought I'd see if I could understand and replicate the USB protocol used
by the Andor Zyla cameras (their lack of a Mac driver has long been a grumble of mine).
After a couple of hours poring over the Wireshark trace that results just from their driver opening the initial connection to the camera,
it's safe to say this would *not* be a friday afternoon project...  
