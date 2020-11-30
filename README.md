
# Implementing a Reliable Transport Protocol

# CSE322: Computer Networks Sessional


## 1. Overview

In this laboratory programming assignment, you will be writing the sending and receiving
transport-level code for implementing a simple reliable data transfer protocol. You will
perform alternating-bit protocol which is quite simple and straightforward. This lab should
be **fun** since your implementation will differ very little from what would be required in a
real-world situation.

Since you probably don’t have standalone machines (with an OS that you can modify), your
code will have to execute in a simulated hardware/software environment. However, the
programming interface provided to your routines, i.e., the code that would call your entities
from above and from below is very close to what is done in an actual UNIX environment.
(Indeed, the software interfaces described in this programming assignment are much more
realistic than the infinite loop senders and receivers that many texts describe).
Stopping/starting of timers are also simulated, and timer interrupts will cause your timer
handling routine to be activated.

From this Proect you will also learn how to write such simulators using discrete event
simulation; though this is not an objective of this lab.

## 2. The Routines to Implement

The procedures you will write are for the sending entity (A) and the receiving entity (B). Only
unidirectional transfer of data (from A to B) is required. Of course, the B side will have to
send packets to A to acknowledge (positively or negatively) receipt of data. Your routines
are to be implemented in the form of the procedures described below. These procedures will
be called by (and will call) procedures that have been written in the network emulator. The
overall structure of the environment is shown in the figure below (structure of the emulated
environment):


The unit of data passed between the upper layers and your protocol is a _message_ , which is
declared as:

struct msg {

char data[20];

};

Your sending entity will thus receive data in 20-byte chunks from layer5; your receiving
entity should deliver 20-byte chunks of correctly received data to layer5 at the receiving side.

The unit of data passed between your routines and the network layer is the _packet_ , which is
declared as:

struct pkt {

int seqnum;

int acknum;
int checksum;

char payload[20];

};

Your routines will fill in the payload field from the message data passed down from layer5.
The other packet fields will be used by your protocol to ensure reliable delivery.

The routines you will write are detailed below. As noted above, such procedures in real-life
would be part of the operating system, and would be called by other procedures in the
operating system.

```
* A_output(message) , where message is a structure of type msg, containing data to
be sent to the B-side. This routine will be called whenever the upper layer at the
sending side (A) has a message to send. It is the job of your protocol to ensure that
the data in such a message is delivered in-order, and correctly, to the receiving side
upper layer.
```

```
* A_input(packet) , where packet is a structure of type pkt. This routine will be
called whenever a packet sent from B-side (i.e., as a result of a tolayer3() being
done by a B-side procedure) arrives at the A-side. packet is the (possibly corrupted)
packet sent from B-side.  

* A_timerinterrupt() This routine will be called when A’s timer expires (thus
generating a timer interrupt). You’ll probably want to use this routine to control the
retransmission of packets. See starttimer() and stoptimer() below for how
the timer is started and stopped.
* A_init() This routine will be called once, before any of your other A-side routines are
called. It can be used to do any required initialization.
* B_input(packet) , where packet is a structure of type pkt. This routine will be
called whenever a packet sent from the A-side (i.e., as a result of a tolayer3()
being done by a A-side procedure) arrives at the B-side. packet is the (possibly
corrupted) packet sent from A-side.
* B_init() This routine will be called once, before any of your other B-side routines are
called. It can be used to do any required initialization.
```
## 3. Software Interfaces

The procedures described above are the ones that you will write. However, the following
procedures have already been written which can be called by your routines:

```
* starttimer(calling_entity,increment) , where calling_entity is either 0 (for
starting the A-side timer) or 1 (for starting the B-side timer), and increment is a
float value indicating the amount of time that will pass before the timer interrupts.
A’s timer should only be started (or stopped) by A-side routines, and similarly for the
B-side timer. To give you an idea of the appropriate increment value to use: a packet
sent into the network takes an average of 5 time units to arrive at the other side when
there are no other messages in the medium.
* stoptimer(calling_entity) , where calling_entity is either 0 (for stopping the A-
side timer) or 1 (for stopping the B-side timer).
* tolayer3(calling_entity,packet) , where calling_entity is either 0 (for the A-
side send) or 1 (for the B-side send), and packet is a structure of type pkt. Calling
this routine will cause the packet to be sent into the network, destined for the other
entity.
* tolayer5(calling_entity,message) , where calling_entity is either 0 (for the A-
side send) or 1 (for the B-side send), and message is a structure of type msg. With
unidirectional data transfer, you would only be calling this with calling_entity
equal to 1 (delivery to the B-side). Calling this routine will cause data to be passed to
layer 5.
```

## 4. The Simulated Network Environment

A call to procedure tolayer3() sends packets into the medium (i.e., into the network
layer). Your procedures A_input() and B_input() are called when a packet is to be
delivered from the medium to your protocol layer.

The medium is capable of corrupting and losing packets. It will not reorder packets. When
you compile your procedures and the given procedures together and run the resulting
program, you will be asked to specify values regarding the simulated network environment:

```
* Number of messages to simulate: The emulator (and your routines) will stop after
this number of messages have been transmitted from entity (A) to entity (B).
* Loss: You are asked to specify a packet loss probability. A values of 0.1 would mean
that one in ten packets (on average) are lost.
* Corruption: You are asked to specify a packet corruption probability. A value of 0.
would mean that one in five packets (on average) are corrupted. Note that the
contents of payload, sequence, ack, or checksum fields can be corrupted. Your
checksum should include the data, sequence, and ack fields.
* Tracing: Setting a tracing value of 1 or 2 will print out useful information about what
is going on inside the emulation (e.g., what’s happening to packets and timers). A
tracing values of 0 will turn this off. A tracing value greater than 2 will display all sorts
of odd messages that are for my own emulator-debugging purposes. A tracing value
of 2 may be helpful to you in debugging your code. You should keep in mind that real
developers do not have underlying networks that provide such nice information
about what is going to happen to their packets!
* Average time between messages from sender’s layer5: You can set this value to
any non-zero positive value. Note that the smaller the value you choose, the faster
packets will be arriving at your sender.
```
## 5. Implementation of Alternating-Bit-Protocol

You are to write the procedures, A_output(), A_input(), A_timerinterrupt(),
A_init(), B_input(), and B_init() which together will implement a stop-and-wait
(i.e., the alternating bit protocol, which we referred to as rdt3.0 in the text) unidirectional
transfer of data from the A-side to the B-side. Your protocol should use both ACK and NACK
messages. Since there is no type field in the packet, NACK has to be implemented as by
sending ACK again for the last correctly received packet.

You should choose a very large value for the average time between messages from sender’s
layer5, so that your sender is never called while it still has an outstanding, unacknowledged
message it is trying to send to the receiver. I’d suggest you choose a value of 1000. You should
also perform a check in your sender to make sure that when A_output() is called, there is


no message currently in transit. If there is, you can simply ignore (drop) the data being
passed to the A_output() routine.

## 6. Helpful Hints

```
* Checksumming: You can use whatever approach for checksumming you want.
Remember that the sequence number and ack field can also be corrupted. We would
suggest a TCP-like checksum, which consists of the sum of the (integer) sequence and
ack field values, added to a character-by-character sum of the payload field of the
packet (i.e., treat each character as if it were an 8-bit integer and just add them
together)
* Note that any shared “state” among your routines needs to be in the form of global
variables. Note also that any information that your procedures need to save from one
invocation to the next must also be a global (or static) variable. For example, your
routines will need to keep a copy of a packet for possible retransmission. It would
probably be a good idea for such a data structure to be a global variable in your code.
Note, however, that if one of your global variables is used by your sender side, that
variable should NOT be accessed by the receiving side entity, since in real life,
communicating entities connected only by a communication channel cannot share
global variables.
* There is a float global variable called time that you can access from within your code
to help you out with your diagnostics messages.
* START SIMPLE: Set the probabilities of loss and corruption to zero and test out your
routines. Better yet, design and implement your procedures for the case of no loss
and no corruption, and get them working first. Then handle the case of one of these
probabilities being non-zero, and then finally both being non-zero.
* Debugging: We’d recommend that you set the tracing level to 2 and put lots of
printf’s in your code while debugging your procedures.
* Random Numbers: The emulator generates packet loss and errors using a random
number generator. Our past experience is that random number generators can vary
widely from one machine to another. You may need to modify the random number
```

```
generation code in the emulator we have supplied you. Our emulation routines have
a test to see if the random number generator on your machine will work with our
code. If you get an error message:
```
```
It is likely that random number generation on your machine is different from
what this emulator expects. Please take a look at the routine jimsrand() in
the emulator code. Sorry.
```
```
then you’ll know you’ll need to look at how random numbers are generated in the
routine jimsrand(); see the comments in that routine.
```

## 7. Resources on Internet

You will find a few code bases available for similar problem on the internet. For this very
reason, we have made some changes in the original specification to differentiate the problem.
Hence, be careful and avoid dumb copy (without changing variable names, data structures,
naming convention, logic flow, etc.) from those sources. We will run copy checker against all
those publicly available codes for this problem.


## 9. Reference

This problem specification and accompanying source code has been adapted from the book’s
companion website. You can get the original description under the following link:

https://media.pearsoncmg.com/aw/aw_kurose_network_3/labs/lab5/lab5.html



### Sample Output-


----- Stop and Wait Network Simulator Version 1.1 --------

Enter the number of messages to simulate: 10

Enter packet loss probability \[enter 0.0 for no loss\]:.3

Enter packet corruption probability \[0.0 for no corruption\]:.3

Enter average time between messages from sender's layer5 \[ &gt;
0.0\]:1000

Enter TRACE:3

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 0.000000

INSERTEVENT: future time will be 1870.573975

EVENT time: 1870.573975, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 1870.573975

INSERTEVENT: future time will be 3512.483887

MAINLOOP: data given to student: aaaaaaaaaaaaaaaaaaa

Successfully Packet 0 sent to Network layer from A\_output

TOLAYER3: seq: 0, ack 0, check: 1843 aaaaaaaaaaaaaaaaaaa

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 1870.573975

INSERTEVENT: future time will be 1876.039062

START TIMER: starting timer at 1870.573975

INSERTEVENT: time is 1870.573975

INSERTEVENT: future time will be 2869.573975

EVENT time: 1876.039062, type: 2, fromlayer3 entity: 1

Corrupted Packet received in transport Layer B\_input. Sending negative
acknowledgement to network layer

TOLAYER3: seq: -1, ack 1, check: 0

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 1876.039062

INSERTEVENT: future time will be 1878.220703

EVENT time: 1878.220703, type: 2, fromlayer3 entity: 0

Negative Acknowledgement received in A\_input.

EVENT time: 2869.573975, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 1 retransmitted to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 2869.573975

INSERTEVENT: time is 2869.573975

INSERTEVENT: future time will be 3868.573975

EVENT time: 3512.483887, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 3512.483887

INSERTEVENT: future time will be 3739.260742

MAINLOOP: data given to student: bbbbbbbbbbbbbbbbbbb

Packet 2 Ignored because acknowledgement of previous packet not received
in A\_output

EVENT time: 3739.260742, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 3739.260742

INSERTEVENT: future time will be 4610.656250

MAINLOOP: data given to student: ccccccccccccccccccc

Packet 3 Ignored because acknowledgement of previous packet not received
in A\_output

EVENT time: 3868.573975, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 1 retransmitted to Network layer from A\_output

TOLAYER3: seq: 0, ack 0, check: 1843 aaaaaaaaaaaaaaaaaaa

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 3868.573975

INSERTEVENT: future time will be 3873.041260

START TIMER: starting timer at 3868.573975

INSERTEVENT: time is 3868.573975

INSERTEVENT: future time will be 4867.574219

EVENT time: 3873.041260, type: 2, fromlayer3 entity: 1

Corrupted Packet received in transport Layer B\_input. Sending negative
acknowledgement to network layer

TOLAYER3: packet being lost

EVENT time: 4610.656250, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 4610.656250

INSERTEVENT: future time will be 5469.190918

MAINLOOP: data given to student: ddddddddddddddddddd

Packet 4 Ignored because acknowledgement of previous packet not received
in A\_output

EVENT time: 4867.574219, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 1 retransmitted to Network layer from A\_output

TOLAYER3: seq: 0, ack 0, check: 1843 aaaaaaaaaaaaaaaaaaa

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 4867.574219

INSERTEVENT: future time will be 4872.846680

START TIMER: starting timer at 4867.574219

INSERTEVENT: time is 4867.574219

INSERTEVENT: future time will be 5866.574219

EVENT time: 4872.846680, type: 2, fromlayer3 entity: 1

Successfully Packet received from network layer in B\_input

Acknowledgement is sending for the Packet received to Network layer from
B\_input

TOLAYER5: data received: aaaaaaaaaaaaaaaaaaa

TOLAYER3: seq: 0, ack 0, check: 0

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 4872.846680

INSERTEVENT: future time will be 4877.049316

EVENT time: 4877.049316, type: 2, fromlayer3 entity: 0

STOP TIMER: stopping timer at 4877.049316

Success Acknowledge received in A\_input

EVENT time: 5469.190918, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 5469.190918

INSERTEVENT: future time will be 7251.116211

MAINLOOP: data given to student: eeeeeeeeeeeeeeeeeee

Successfully Packet 4 sent to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 5469.190918

INSERTEVENT: time is 5469.190918

INSERTEVENT: future time will be 6468.190918

EVENT time: 6468.190918, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 5 retransmitted to Network layer from A\_output

TOLAYER3: seq: 1, ack 0, check: 1920 eeeeeeeeeeeeeeeeeee

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 6468.190918

INSERTEVENT: future time will be 6472.254395

START TIMER: starting timer at 6468.190918

INSERTEVENT: time is 6468.190918

INSERTEVENT: future time will be 7467.190918

EVENT time: 6472.254395, type: 2, fromlayer3 entity: 1

Corrupted Packet received in transport Layer B\_input. Sending negative
acknowledgement to network layer

TOLAYER3: seq: -1, ack 0, check: 1163 aaaaaaaaaaaa

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 6472.254395

INSERTEVENT: future time will be 6474.038086

EVENT time: 6474.038086, type: 2, fromlayer3 entity: 0

Negative Acknowledgement received in A\_input.

EVENT time: 7251.116211, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 7251.116211

INSERTEVENT: future time will be 8417.564453

MAINLOOP: data given to student: fffffffffffffffffff

Packet 6 Ignored because acknowledgement of previous packet not received
in A\_output

EVENT time: 7467.190918, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 5 retransmitted to Network layer from A\_output

TOLAYER3: seq: 1, ack 0, check: 1920 eeeeeeeeeeeeeeeeeee

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 7467.190918

INSERTEVENT: future time will be 7474.392578

START TIMER: starting timer at 7467.190918

INSERTEVENT: time is 7467.190918

INSERTEVENT: future time will be 8466.191406

EVENT time: 7474.392578, type: 2, fromlayer3 entity: 1

Corrupted Packet received in transport Layer B\_input. Sending negative
acknowledgement to network layer

TOLAYER3: seq: -1, ack 0, check: 134 p!V

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 7474.392578

INSERTEVENT: future time will be 7477.023438

EVENT time: 7477.023438, type: 2, fromlayer3 entity: 0

Negative Acknowledgement received in A\_input.

EVENT time: 8417.564453, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 8417.564453

INSERTEVENT: future time will be 10364.064453

MAINLOOP: data given to student: ggggggggggggggggggg

Packet 7 Ignored because acknowledgement of previous packet not received
in A\_output

EVENT time: 8466.191406, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 5 retransmitted to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 8466.191406

INSERTEVENT: time is 8466.191406

INSERTEVENT: future time will be 9465.191406

EVENT time: 9465.191406, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 5 retransmitted to Network layer from A\_output

TOLAYER3: seq: 1, ack 0, check: 1920 eeeeeeeeeeeeeeeeeee

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 9465.191406

INSERTEVENT: future time will be 9466.250000

START TIMER: starting timer at 9465.191406

INSERTEVENT: time is 9465.191406

INSERTEVENT: future time will be 10464.191406

EVENT time: 9466.250000, type: 2, fromlayer3 entity: 1

Successfully Packet received from network layer in B\_input

Acknowledgement is sending for the Packet received to Network layer from
B\_input

TOLAYER5: data received: eeeeeeeeeeeeeeeeeee

TOLAYER3: packet being lost

EVENT time: 10364.064453, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 10364.064453

INSERTEVENT: future time will be 11235.591797

MAINLOOP: data given to student: hhhhhhhhhhhhhhhhhhh

Packet 8 Ignored because acknowledgement of previous packet not received
in A\_output

EVENT time: 10464.191406, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 5 retransmitted to Network layer from A\_output

TOLAYER3: seq: 1, ack 0, check: 1920 eeeeeeeeeeeeeeeeeee

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 10464.191406

INSERTEVENT: future time will be 10469.770508

START TIMER: starting timer at 10464.191406

INSERTEVENT: time is 10464.191406

INSERTEVENT: future time will be 11463.191406

EVENT time: 10469.770508, type: 2, fromlayer3 entity: 1

Unknown Packet received in B\_input. Retransmitting previous
acknowledgement

TOLAYER3: seq: 1, ack 1, check: 1214 eeeeeeeeeeee

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 10469.770508

INSERTEVENT: future time will be 10476.020508

EVENT time: 10476.020508, type: 2, fromlayer3 entity: 0

Data corrupted Acknowledgement received in A\_input. Retransmitting
packet after timer interupt

EVENT time: 11235.591797, type: 1, fromlayer5 entity: 0

GENERATE NEXT ARRIVAL: creating new arrival

INSERTEVENT: time is 11235.591797

INSERTEVENT: future time will be 11881.031250

MAINLOOP: data given to student: iiiiiiiiiiiiiiiiiii

Packet 9 Ignored because acknowledgement of previous packet not received
in A\_output

EVENT time: 11463.191406, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 5 retransmitted to Network layer from A\_output

TOLAYER3: seq: 1, ack 0, check: 1920 eeeeeeeeeeeeeeeeeee

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 11463.191406

INSERTEVENT: future time will be 11466.069336

START TIMER: starting timer at 11463.191406

INSERTEVENT: time is 11463.191406

INSERTEVENT: future time will be 12462.191406

EVENT time: 11466.069336, type: 2, fromlayer3 entity: 1

Unknown Packet received in B\_input. Retransmitting previous
acknowledgement

TOLAYER3: seq: 1, ack 1, check: 1214 eeeeeeeeeeee

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 11466.069336

INSERTEVENT: future time will be 11469.731445

EVENT time: 11469.731445, type: 2, fromlayer3 entity: 0

STOP TIMER: stopping timer at 11469.731445

Success Acknowledge received in A\_input

EVENT time: 11881.031250, type: 1, fromlayer5 entity: 0

MAINLOOP: data given to student: jjjjjjjjjjjjjjjjjjj

Successfully Packet 9 sent to Network layer from A\_output

TOLAYER3: seq: 0, ack 0, check: 2014 jjjjjjjjjjjjjjjjjjj

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 11881.031250

INSERTEVENT: future time will be 11889.942383

START TIMER: starting timer at 11881.031250

INSERTEVENT: time is 11881.031250

INSERTEVENT: future time will be 12880.031250

EVENT time: 11889.942383, type: 2, fromlayer3 entity: 1

Corrupted Packet received in transport Layer B\_input. Sending negative
acknowledgement to network layer

TOLAYER3: seq: -1, ack 1, check: 39 \_x0010\_!V

TOLAYER3: packet being corrupted

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 11889.942383

INSERTEVENT: future time will be 11895.559570

EVENT time: 11895.559570, type: 2, fromlayer3 entity: 0

Negative Acknowledgement received in A\_input.

EVENT time: 12880.031250, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 10 retransmitted to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 12880.031250

INSERTEVENT: time is 12880.031250

INSERTEVENT: future time will be 13879.031250

EVENT time: 13879.031250, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 10 retransmitted to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 13879.031250

INSERTEVENT: time is 13879.031250

INSERTEVENT: future time will be 14878.031250

EVENT time: 14878.031250, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 10 retransmitted to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 14878.031250

INSERTEVENT: time is 14878.031250

INSERTEVENT: future time will be 15877.031250

EVENT time: 15877.031250, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 10 retransmitted to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 15877.031250

INSERTEVENT: time is 15877.031250

INSERTEVENT: future time will be 16876.031250

EVENT time: 16876.031250, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 10 retransmitted to Network layer from A\_output

TOLAYER3: packet being lost

START TIMER: starting timer at 16876.031250

INSERTEVENT: time is 16876.031250

INSERTEVENT: future time will be 17875.031250

EVENT time: 17875.031250, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 10 retransmitted to Network layer from A\_output

TOLAYER3: seq: 0, ack 0, check: 2014 jjjjjjjjjjjjjjjjjjj

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 17875.031250

INSERTEVENT: future time will be 17877.144531

START TIMER: starting timer at 17875.031250

INSERTEVENT: time is 17875.031250

INSERTEVENT: future time will be 18874.031250

EVENT time: 17877.144531, type: 2, fromlayer3 entity: 1

Successfully Packet received from network layer in B\_input

Acknowledgement is sending for the Packet received to Network layer from
B\_input

TOLAYER5: data received: jjjjjjjjjjjjjjjjjjj

TOLAYER3: packet being lost

EVENT time: 18874.031250, type: 0, timerinterrupt entity: 0

In A\_timerinterrupt- Enough waiting, No Successful Acknowledgement
received. Retransmitting

Successfully Packet 10 retransmitted to Network layer from A\_output

TOLAYER3: seq: 0, ack 0, check: 2014 jjjjjjjjjjjjjjjjjjj

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 18874.031250

INSERTEVENT: future time will be 18880.263672

START TIMER: starting timer at 18874.031250

INSERTEVENT: time is 18874.031250

INSERTEVENT: future time will be 19873.031250

EVENT time: 18880.263672, type: 2, fromlayer3 entity: 1

Unknown Packet received in B\_input. Retransmitting previous
acknowledgement

TOLAYER3: seq: 0, ack 0, check: 1272 jjjjjjjjjjjj

TOLAYER3: scheduling arrival on other side

INSERTEVENT: time is 18880.263672

INSERTEVENT: future time will be 18882.746094

EVENT time: 18882.746094, type: 2, fromlayer3 entity: 0

STOP TIMER: stopping timer at 18882.746094

Success Acknowledge received in A\_input

Simulator terminated at time 18882.746094

after sending 10 msgs from layer5
