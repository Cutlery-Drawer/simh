.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. role:: c(code)
   :language: C

.. |t_stat| replace:: :html:`<code class="c-type">t_stat</code>`
.. |uint32| replace:: :html:`<code class="c-type">uint32</code>`

***************************
Design notes on the Sigma 7
***************************
:Date: 2022-07-03
:Revision: 9258e25
:Author: Bob Supnik

Variations on status
--------------------
Over time, the |t_stat| data type has been overloaded to mean SCP status returns,
magtape library status returns,
and then simulator-specific status returns.
The result is a lot of confusion about overlapping ranges,
translation,
and so on.

There’s nothing that can be done about the overlap between SCP and magtape status;
any call on a magtape library routine must be followed by a "status translation" to get a valid SCP
(or other)
status code.

The Sigma 7 simulator uses |uint32| for its generalized status variables.
These status variables may contain values in the following ranges:

===========================================================  ================================
:html:`<samp>0x0000</samp>`\ –\ :html:`<samp>0x1000</samp>`  SCP or VM status codes
:html:`<samp>0x4000</samp>`\ –\ :html:`<samp>0x7FFF</samp>`  Channel error status codes
:html:`<samp>0x8000</samp>`\ –\ :html:`<samp>0xFFFF</samp>`  Channel information status codes
:html:`<samp>0x10000</samp>`\ +                              Trap codes
===========================================================  ================================

In practice,
channel codes cannot be passed back to higher-level routines.
Thus, as with magtape codes,
a status translation is required before a return is made,
and status codes confined to either SIMH status codes or trap codes.

The I/O interface
-----------------
The Sigma 7 has up to 8 channels (:html:`<samp>A</samp>`\ …\ :html:`<samp>H</samp>`).
Each channel has its own device address space,
with up to 32 device controllers.
I/O devices are reached through a dispatch routine,
with this calling sequence:

.. code:: C

   uint32 st = io_dispatch (uint32 op, uint32 dva, uint32 *dvst)

where:

* .. |st| replace:: :html:`<code class="c-var">st</code>`
  .. _st:

  |st| is an SCP or VM status code,
  or a trap code.

* .. |op| replace:: :html:`<code class="c-var">op</code>`
  .. _op:

  |op| is the opcode of the current instruction
  (bits ``<1:7>``).

* .. |dva| replace:: :html:`<code class="c-var">dva</code>`
  .. _dva:

  is the device address from the current instruction,
  except for AIO.
  AIO provides the base device address of the interrupting device,
  with no unit.

* .. |dvst| replace:: :html:`<code class="c-var">dvst</code>`
  .. _dvst:

  |dvst| is the device status returned by the instruction.

  |dvst| is broken up into the following fields:

  ===========  ===============================================
  Bits         Meaning
  ===========  ===============================================
  ``<7:0>``    Device status byte for TIO, TDV, or AIO
               (SIO and HIO use TIO status)
  ``<19:16>``  Condition codes
  ``<31:24>``  Unit number
               (AIO only on multi-unit devices only)
  ===========  ===============================================

Interrupts
----------
Channels only provision for one interrupt per device,
the controller interrupt.
When a device posts a controller interrupt,
the number of the interrupting unit is stored in the device’s channel data structures.
That is why multi-unit devices must always invoke the channel with a full device address,
including the unit number.
If the channel call results in posting an interrupt,
the unit number must be saved.

However, some devices have two levels of interrupts:
a controller interrupt and device-specific device interrupts.
That is why when AIO invokes a device,
it cannot provide the unit number;
it might be the saved unit for a controller interrupt,
or it might be a device-specified unit for a device interrupt.

Channels
--------
In the Sigma simulator,
devices drive the channel,
rather than the reverse.
Most devices implement a state-driven model,
roughly like this.

1. Init state.
   Following a successful SIO,
   the device enters INIT state.
   It calls the channel to get the next channel command.
   This will cause the channel to mark itself active and to set up certain internal state flags.
   The device uses the returned channel command to schedule one or more execution states.
2. Exec state.
   In execution state,
   the device executes the channel command.
   Depending on the command,
   it may call on the channel to provide a byte or word for output,
   or to absorb a byte or word of input.
   The device reschedules itself in exec state until:

   * It receives a “zero byte count” status from the channel
   * It reaches end of record (on input)
   * It encounters a device error.

   If a normal end is reached,
   the device will transition to END state.
   If an abnormal end is encountered,
   the device will call tell the channel to terminate with an unusual end condition and cease operations.
3. End state.
   In END state,
   the device tells the channel to terminate with a normal end.
   The channel may indicate that there is a chained command.
   In that case,
   the device returns to INIT state,
   just as though a new SIO had been issued.

Unimplemented register blocks
-----------------------------
On the later models,
selecting an unimplemented register block causes a trap,
but on the Sigma 5/6/7,
the machine continues to run.
All registers read as zero
(or :html:`<samp>0xFFFFFFFF</samp>`),
and writes are ignored.

The simulator uses a unified array for the register block.
To avoid testing for invalid register blocks on every register read or write,
a “cleaner” thread is scheduled if an invalid register block is selected.
This thread runs every instruction if the selected register block is unimplemented,
and it zeroes the currently-selected register block.
This means that the register state may not be correct at simulator stops but is always correct as seen by the simulated program.
