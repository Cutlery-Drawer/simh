.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. |ATTACH| replace:: :html:`<samp><b>ATTACH</b></samp>`
.. |SET|    replace:: :html:`<samp><b>SET</b></samp>`
.. |i|      replace:: :html:`<samp><var>i</var></samp>`
.. |n|      replace:: :html:`<samp><var>n</var></samp>`
.. |q|      replace:: :html:`<samp><var>q</var></samp>`
.. |8|      replace:: :sub:`8`
.. |10|     replace:: :sub:`10`

*****************************************************
ARPANET IMP and TIP Extensions for the H316 simulator
*****************************************************
:Date: 2013-12-03
:Contact: bob@jfcl.com
:Revision: $Format:%H$
:Copyright: See `LICENSE.txt <../LICENSE.txt>`_ for terms of use.

.. topic:: **Copyright notice**

   The following copyright notice applies to the SIMH source, binary, and documentation:

   .. include:: ../LICENSE.txt

.. sectnum:: :suffix: .
.. contents::
   :backlinks: none
   :depth: 3
   :local:

Introduction
============
This memorandum documents the extensions made to the SIMH Honeywell H316 simulator to allow it to run the ARPANET Interface Message Processor (aka |IMP|) and Terminal Interface Processor (|TIP|) software.
A single |IMP| or |TIP| instance isn't very useful,
however these extensions allow multiple SIMH instances,
each running a copy of the |IMP| or |TIP| code,
to communicate using virtual modem connections implemented with physical serial ports on the host computer.
Alternatively,
virtual modem connections may be tunneled over TCP/IP to a remote SIMH |IMP/TIP| instance anywhere in the world.
Moreover,
each SIMH |IMP| instance can then be connected via virtual host interface cards to other local PDP-10 or PDP-11 SIMH instances running ARPANET host software.

.. note::
   Presently, |TIP| support is not implemented,
   and |IMP| host card support is only partially implemented.
   The current implementation is sufficient to allow multiple |IMPs| to talk via virtual modem connections,
   and that's all.

Simulator files
===============
These additional |IMP/TIP| specific files are added to SIMH:

+---------------+----------------+------------------------------------------+
| Subdirectory  | File           | Contains                                 |
+===============+================+==========================================+
| ``sim/h316/`` | ``h316_imp.h`` | IMP/TIP and ARPAnet specific definitions |
|               +----------------+------------------------------------------+
|               | ``h316_imp.c`` | The IMP pseudo device                    |
|               +----------------+------------------------------------------+
|               | ``h316_rtc.c`` | Real-time clock and watch dog timer      |
|               +----------------+------------------------------------------+
|               | ``h316_hi.c``  | Host interfaces                          |
|               +----------------+------------------------------------------+
|               | ``h316_mi.c``  | Modem interfaces                         |
|               +----------------+------------------------------------------+
|               | ``h316_udp.c`` | UDP support for MI and HI modules        |
+---------------+----------------+------------------------------------------+

Additional features and devices
===============================
The |IMP/TIP| adds the following devices to the H316 configuration:

===========  =========================================
Device name  Simulates
===========  =========================================
|MI|         Modem interface (up to 5)
|HI|         Host interface (up to 4)
|WDT|        Watchdog timer
|RTC|        Real-time clock
             (replaces H316 |CLK| device)
|IMP|_       |TASK|, |IMPN|, and |MLC| functions
             (addresses 41\ |8| and 42\ |8|)
===========  =========================================

.. |TASK|    replace:: :html:`<samp>TASK</samp>`
.. |IMPN|    replace:: :html:`<samp>IMPN</samp>`
.. |IMP|     replace:: :html:`<samp>IMP</samp>`
.. |TIP|     replace:: :html:`<samp>TIP</samp>`
.. |IMPs|    replace:: |IMP|\ s
.. |TIPs|    replace:: |TIP|\ s
.. |IMP/TIP| replace:: |IMP|\ /\ |TIP|
.. _IMP:

|IMP| pseudo-device
-------------------
The |IMP| pseudo-device implements two sets of miscellaneous I/O instructions in the original |IMP/TIP| hardware.
This device responds both to IO address 41\ |8|,
which implements the |TASK| switching interrupt and the |IMP| identification number,
and to IO address 42\ |8|,
which implements the |MLC| test.
The |IMP| device may be enabled with the standard command :html:`<samp><b>SET IMP ENABLED</b></samp>`.

The |IMP| address,
which is returned by the ``RDIMPN`` (``INA 1041``) instruction,
may be set with the command

====================================================  ==============================
:html:`<samp><b>SET IMP NUM=</b><var>n</var></samp>`  Set IMP station address to |n|
====================================================  ==============================

This value was apparently hardwired into each original IMP/TIP station.
The |IMP| device also implements the ``AMIMLC`` (``SKS 0042``) instruction,
which skips if the current hardware is a multi-line controller (a.k.a, a |TIP|).
Since |TIP| emulation is not currently supported,
this instruction is presently hard-wired as a NOP
(i.e., it never skips).

The |IMP| device implements the following registers:

.. |MLC| replace:: :html:`<samp class="register">MLC</samp>`
.. |IEN| replace:: :html:`<samp class="register">IEN</samp>`
.. |IRQ| replace:: :html:`<samp class="register">IRQ</samp>`

=====  ====  ======================
Name   Size  Comments
=====  ====  ======================
|MLC|  1     Always zero (TIP flag)
|IEN|  1     Task interrupt enabled
|IRQ|  1     Task interrupt pending
=====  ====  ======================

These registers can be viewed with the command :html:`<samp><b>EXAMINE IMP STATE</b></samp>`.

The |IMP| device implements these debugging flags:

==============================================  =====================================
:html:`<samp><b>SET IMP DEBUG=WARN</b></samp>`  Print warnings for unusual conditions
:html:`<samp><b>SET IMP DEBUG=IO</b></samp>`    Trace all IMP device I/O instructions
==============================================  =====================================

Remember that you must enable debugging output first before these settings will be effective;
refer to the `Controlling debugging` section of the `SIMH Users' Guide <./simh_doc.rst>`_ for more information.

.. |CLK| replace:: :html:`<samp>CLK</samp>`
.. |RTC| replace:: :html:`<samp>RTC</samp>`
.. _RTC:
.. _CLK:

Real-time clock
---------------
The |IMP/TIP| hardware contained a custom real-time clock which was unique to that implementation,
and further,
the |IMP/TIP| did not have the standard H316 power line clock or real-time clock.
The |IMP/TIP| clock is in some ways similar to the H316 real-time clock option,
*but it is not the same*.
Normally the |CLK| device would be disabled when emulating the |IMP|,
however there is no actual conflict between the |CLK| and the |RTC| devices and both can be enabled simultaneously.

The |RTC| device supports the following |SET| commands:

=========================================================  ===============================================
:html:`<samp><b>SET RTC ENABLED</b></samp>`                Enable |RTC| emulation
:html:`<samp><b>SET RTC INTERVAL=</b><var>i</var></samp>`  Set the |RTC| tick interval to |i| microseconds
:html:`<samp><b>SET RTC QUANTUM=</b><var>q</var></samp>`   Set the |RTC| resolution to |q| ticks
=========================================================  ===============================================

.. |INTERVAL| replace:: :html:`<samp>INTERVAL</samp>`
.. |QUANTUM|  replace:: :html:`<samp>QUANTUM</samp>`

The |INTERVAL| parameter sets the interval between |RTC| clock ticks,
in microseconds,
and the |QUANTUM| parameter sets how often,
as a number of clock ticks,
SIMH updates the |RTC| count.
To give an example,
if |INTERVAL| was set to 20 and |QUANTUM| was 1,
SIMH would attempt to increment the |RTC| count by one at a rate of 50,000 times per second.
Modern PCs are very fast and it would probably be possible for SIMH to do this,
but the amount of overhead would be huge and it would be impractical.

Instead,
for example,
if |INTERVAL| was still set to 20 but |QUANTUM| was now set to 50,
SIMH would add 50 to the |RTC| count at a rate of 1000 times per second.
This represents much less overhead and yet gives the same net counting rate as before.
However,
now the clock count jumps in increments of 50 rather than 1.
Most software wouldn't notice,
but it's always possible this could cause problems.

It's important to note that the |QUANTUM| does not affect the overall clock frequency —
as long as the |INTERVAL| is set to 20,
the clock would count at an effective rate of 50 kHz regardless of the |QUANTUM| value.

The |RTC| device implements the following registers:

.. |ENA|   replace:: :html:`<samp class="register">ENA</samp>`
.. |COUNT| replace:: :html:`<samp class="register">COUNT</samp>`
.. |TPS|   replace:: :html:`<samp class="register">TPS</samp>`
.. |WAIT|  replace:: :html:`<samp class="register">WAIT</samp>`

=======  ====  ==================================
Name     Size  Comments
=======  ====  ==================================
|ENA|    1     |RTC| is enabled
|COUNT|  16    Current count
|IEN|    1     |RTC| interrupt enabled
|IRQ|    1     |RTC| interrupt pending
|TPS|    1     Effective ticks per second
|WAIT|   24    Simulator time until the next tick
=======  ====  ==================================

These registers can be viewed with the command :html:`<samp><b>EXAMINE RTC STATE</b></samp>`.

The |RTC| device implements these debugging flags:

==============================================  =======================================
:html:`<samp><b>SET RTC DEBUG=WARN</b></samp>`  Print warnings for unusual conditions
:html:`<samp><b>SET RTC DEBUG=IO</b></samp>`    Trace all |RTC| device I/O instructions
==============================================  =======================================

Please refer to the `Controlling Debugging` section of the `SIMH Users' Guide`_ for more information,
and remember that you must enable debugging output first before these settings will be effective.

.. |WDT| replace:: :html:`<samp>WDT</samp>`
.. _WDT:

Watch dog timer
---------------
The |IMP/TIP| hardware also had a custom watch dog timer implementation which would force a non-maskable interrupt if was ever allowed to expire.
The |WDT| device in SIMH implements this feature.
The |IMP/TIP| watch dog timer also implemented a couple of other unrelated features —
these include the status display panel,
which you can see featured prominently in some |IMP| photos,
and a flag to indicate whether the CPU was a H316 or DDP-516.

The |WDT| device supports the following |SET| commands:

======================================================  =======================================
:html:`<samp><b>SET WDT ENABLED</b></samp>`             Enable |WDT| emulation
:html:`<samp><b>SET WDT DELAY=</b><var>n</var></samp>`  Set the |WDT| delay to |n| milliseconds
======================================================  =======================================

.. |DELAY| replace:: :html:`<samp><var>DELAY</var></samp>`

The |DELAY| parameter sets the |WDT| timeout,
in milliseconds.
This is a 16-bit unsigned value,
so the maximum |WDT| delay is just over a minute.
Note that there are no H316 instructions that disable the |WDT| —
only one to reset it —
so if *the WDT device is enabled, the WDT will run anytime simulation is active*.
If the code fails to reset the |WDT| in a timely fashion,
the |WDT| interrupt will occur regardless of the enabled or disabled state of the interrupt system.

.. compound::

   Setting the |WDT| delay to zero ::

       SET WDT DELAY=0

   prevents the |WDT| from ever generating a timeout.
   This special feature allows the |WDT| device to be enabled,
   so that code which executes |WDT|,
   status light,
   or AMI512 instructions can be executed and debugged,
   but without inadvertently triggering a |WDT| timeout.

The |WDT| device implements the following registers:

.. |TMO|    replace:: :html:`<samp class="register">TMO</samp>`
.. |LIGHTS| replace:: :html:`<samp class="register">LIGHTS</samp>`

========  ====  ==================================
Name      Size  Comments
========  ====  ==================================
|COUNT|   16    Current countdown
|TMO|     1     |WDT| timed out
|LIGHTS|  16    Last "set status lights"
|WAIT|    24    Calcuated time until the next tick
========  ====  ==================================

These registers can be viewed with the command :html:`<samp><b>EXAMINE WDT STATE</b></samp>`.

The |WDT| device implements these debugging flags:

================================================  =======================================
:html:`<samp><b>SET WDT DEBUG=WARN</b></samp>`    Print warnings for unusual conditions
:html:`<samp><b>SET WDT DEBUG=IO</b></samp>`      Trace all |WDT| device I/O instructions
:html:`<samp><b>SET WDT DEBUG=LIGHTS</b></samp>`  Trace |IMP| status light changes
================================================  =======================================

Remember that you must enable debugging output first before these settings will be effective;
refer to the `Controlling Debugging` section of the `SIMH Users' Guide`_ for more information.

Communications devices
----------------------

.. |MI|  replace:: :html:`<samp>MI</samp>`
.. |MI1| replace:: :html:`<samp>MI1</samp>`
.. |MI2| replace:: :html:`<samp>MI2</samp>`
.. |MI3| replace:: :html:`<samp>MI3</samp>`
.. |MI4| replace:: :html:`<samp>MI4</samp>`
.. |MI5| replace:: :html:`<samp>MI5</samp>`
.. _MI:
.. _MI1:
.. _MI2:
.. _MI3:
.. _MI4:
.. _MI5:

Modem interface
"""""""""""""""
|IMPs| and |TIPs| communicated with each other via leased telephone lines and synchronous modem links.
These modem links were point-to-point —
each |IMP/TIP| modem communicated with exactly one other modem attached to exactly one other |IMP/TIP|.
SIMH simulates these modem interfaces and allows them to be attached to either a real,
physical,
serial port on the host computer or to a virtual UDP/IP port.
Either can then be connected to another SIMH instance running the |IMP/TIP| software,
or even a real |IMP| or |TIP| if someone gets one working again,
and a network can be assembled.

SIMH implements five modem interfaces, |MI1| thru |MI5|.
Initially,
|MI1| thru :html:`<samp>3</samp>` are enabled and |MI4| and :html:`<samp>5</samp>` are not,
however this can be changed with the commands:

==============================================================  ======================
:html:`<samp><b>SET MI</b><var>n</var> <b>ENABLED</b></samp>`   Enable modem line |n|
:html:`<samp><b>SET MI</b><var>n</var> <b>DISABLED</b></samp>`  Disable modem line |n|
==============================================================  ======================

A limitation of the original |IMP| hardware is that the DMC channels used by modem lines 4 and 5 conflict with those used by host interfaces 3 and 4.
Thus it not possible to enable all modem lines and all host interfaces at the same time
(see `Physical serial tunnels`).

Modem interfaces implement one parameter which may be explicitly set::

    SET MIn BPS=56000

This parameter sets the simulated line speed,
in bits per second,
for UDP/IP virtual modem connections.

Modem interfaces also implement an interface (local) and a line (remote) loopback feature.
This cause the modem to receive its own transmitted messages,
and are analogous to features in the IMP modem hardware.
Loopback may be enabled or disabled with these commands:

=====================================================================  =====================================
:html:`<samp><b>SET MI</b><var>n</var> <b>LOOPINTERFACE</b></samp>`    Enable interface loopback on line |n|
:html:`<samp><b>SET MI</b><var>n</var> <b>NOLOOPINTERFACE</b></samp>`  Disable interface loopback
:html:`<samp><b>SET MI</b><var>n</var> <b>LOOPLINE</b></samp>`         Enable line loopback on line |n|
:html:`<samp><b>SET MI</b><var>n</var> <b>NOLOOPLINE</b></samp>`       Disable line loopback
=====================================================================  =====================================

Modem interfaces support the following registers:

.. |RXPOLL| replace:: :html:`<samp class="register">RXPOLL</samp>`
.. |RXPEND| replace:: :html:`<samp class="register">RXPEND</samp>`
.. |RXERR|  replace:: :html:`<samp class="register">RXERR</samp>`
.. |RXIEN|  replace:: :html:`<samp class="register">RXIEN</samp>`
.. |RXIRQ|  replace:: :html:`<samp class="register">RXIRQ</samp>`
.. |RXTOT|  replace:: :html:`<samp class="register">RXTOT</samp>`
.. |TXDLY|  replace:: :html:`<samp class="register">TXDLY</samp>`
.. |TXIEN|  replace:: :html:`<samp class="register">TXIEN</samp>`
.. |TXIRQ|  replace:: :html:`<samp class="register">TXIRQ</samp>`
.. |TXTOT|  replace:: :html:`<samp class="register">TXTOT</samp>`
.. |LINK|   replace:: :html:`<samp class="register">LINK</samp>`
.. |BPS|    replace:: :html:`<samp class="register">BPS</samp>`
.. |ILOOP|  replace:: :html:`<samp class="register">ILOOP</samp>`
.. |LLOOP|  replace:: :html:`<samp class="register">LLOOP</samp>`

========  ====  ========================================
Name      Size  Comments
========  ====  ========================================
|RXPOLL|  32    Receiver polling interval
|RXPEND|  1     Receiver waiting for input
|RXERR|   1     Receiver error flag
|RXIEN|   1     Receiver interrupt enable
|RXIRQ|   1     Receiver interrupt request
|RXTOT|   32    Count of total messages received
|TXDLY|   32    Calculated delay before transmitter done
|TXIEN|   1     Transmitter interrupt enable
|TXIRQ|   1     Transmitter interrupt request
|TXTOT|   32    Count of total messages transmitted
|LINK|    32    Link number for ``h316_udp`` module
|BPS|     32    Simulated line speed for UDP connections
|ILOOP|   1     Interface (local) loopback enabled
|LLOOP|   1     Line (remote) loopback enabled
========  ====  ========================================

These registers can be viewed with the command :html:`<samp><b>EXAMINE MI</b><var>n</var> <b>STATE</b></samp>`.

The modem device implements these debugging flags:

================================================================  ==========================================
:html:`<samp><b>SET MI</b><var>n</var> <b>DEBUG=WARN</b></samp>`  Print warnings for unusual conditions
:html:`<samp><b>SET MI</b><var>n</var> <b>DEBUG=IO</b></samp>`    Trace all modem interface I/O instructions
:html:`<samp><b>SET MI</b><var>n</var> <b>DEBUG=UDP</b></samp>`   Trace all UDP packets and connections
:html:`<samp><b>SET MI</b><var>n</var> <b>DEBUG=MSG</b></samp>`   Trace all |IMP| messages sent or received
================================================================  ==========================================

Remember that you must enable debugging output first before these settings will be effective;
refer to the `Controlling debugging` section of the `SIMH Users' Guide`_ for more information.

UDP/IP tunnels
''''''''''''''
A virtual modem may also be tunneled using UDP/IP to another SIMH instance.
UDP connections are symmetrical —
each virtual modem listens for incoming packets on a port which you define,
and transmit outgoing packets to another host and port which you also define.
It's your responsibility to define the ports appropriately so that the input of one modem is logically connected to the output of another modem.
*These connections must be one-to-one.*
Connecting more than one modem output to the same modem input will not cause any problems for SIMH,
but the |IMP| software will become hopelessly confused.

The general form of a virtual modem |ATTACH| command for UDP connections is like this::

    ATTACH MIn llll:w.x.y.z:rrrr

This will listen for incoming packets on port ``llll``,
which should be a decimal number,
and will transmit outgoing packets to port ``rrrr`` on the host with IP address ``w.x.y.z``.
The actual port numbers you use are arbitrary;
however some care must be used to avoid conflicts with any other network applications.

.. compound::

   The UDP attach command also has a few alternative forms;
   for example ::

       ATTACH MIn 4431:imp.jfcl.com:4432

   will listen on port 4431 of the current host,
   do a DNS lookup to determine the IP of the host ``imp.jfcl.com``,
   and then transmit to port 4432 on that host.
   In another example ::

       ATTACH MIn 1201::1202

   will listen to port 1201 on the current, local, host and transmit to port 1202 also on the local host.
   This is useful when both SIMH instances are running on the same PC.
   In this case, *you must use different ports for transmitting and receiving.*
   If they are the same,
   the modem will transmit to itself!

Either end of the UDP connection may be disconnected with the command :html:`<samp><b>DETACH MI</b><var>n</var></samp>`.

Physical serial tunnels
'''''''''''''''''''''''
.. compound::

   The command ::

       ATTACH -p MIn COMnn

   is reserved for a possible future option to attach a physical serial port to a virtual modem.
   This functionality is not currently implemented.

.. |HI|  replace:: :html:`<samp>HI</samp>`
.. |HI1| replace:: :html:`<samp>HI1</samp>`
.. |HI2| replace:: :html:`<samp>HI2</samp>`
.. |HI3| replace:: :html:`<samp>HI3</samp>`
.. |HI4| replace:: :html:`<samp>HI4</samp>`
.. _HI:
.. _HI1:
.. _HI2:
.. _HI3:
.. _HI4:

Host interface
""""""""""""""
.. note::
   The current host interface implementation is presently only a skeleton —
   just enough code exists to allow the |IMP| software to run.
   In this version,
   the |IMP| effectively sees all attached hosts as permanently powered down.
   Hopefully the host interface implementation can be completed at some point,
   just as soon as there is a suitable host emulator for it to talk to.

The |IMP| used the host interface to connect the H316 to an Arpanet host mainframe.
This could be a PDP-10,
an SDS Sigma 7,
a CDC 6600,
an IBM 360,
or any one of many other machines.
Each |IMP| could support up to four host interface cards;
in SIMH these are the devices |HI1| thru |HI4|.
Initially,
|HI1| and :html:`<samp>2</samp>` are enabled and |HI3| and :html:`<samp>4</samp>` are not,
however this can be changed with the commands:

==============================================================  ==========================
:html:`<samp><b>SET HI</b><var>n</var> <b>ENABLED</b></samp>`   Enable host interface |n|
:html:`<samp><b>SET HI</b><var>n</var> <b>DISABLED</b></samp>`  Disable host interface |n|
==============================================================  ==========================

Host interfaces support the following registers:

.. |POLL|  replace:: :html:`<samp class="register">POLL</samp>`
.. |READY| replace:: :html:`<samp class="register">READY</samp>`
.. |FULL|  replace:: :html:`<samp class="register">FULL</samp>`
.. |ERROR| replace:: :html:`<samp class="register">ERROR</samp>`

=======  ====  ===================================
Name     Size  Comments
=======  ====  ===================================
|POLL|   32    Host polling interval
|RXIEN|  1     Receiver interrupt enable
|RXIRQ|  1     Receiver interrupt request
|RXTOT|  32    Count of total messages received
|TXIEN|  1     Transmitter interrupt enable
|TXIRQ|  1     Transmitter interrupt request
|TXTOT|  32    Count of total messages transmitted
|READY|  1     Host ready
|FULL|   1     Host buffer full
|ERROR|  1     Host error
|LLOOP|  1     Local loopback enabled
=======  ====  ===================================

These registers can be viewed with the command :html:`<samp><b>EXAMINE HI</b><var>n</var> <b>STATE</b></samp>`.

The modem device implements these debugging flags:

==============================================================  =========================================
:html:`<samp><b>SET HI</b><var>n</var> <b>DEBUG=IO</b></samp>`  Trace all host interface I/O instructions
==============================================================  =========================================

Remember that you must enable debugging output first before these settings will be effective;
refer to the `Controlling Debugging` section of the `SIMH Users' Guide`_ for more information.

UDP/IP tunnels
''''''''''''''
In SIMH,
host interfaces are simulated using UDP connections to other SIMH instances that simulate the host mainframe and run the Arpanet host operating system.
The |ATTACH| command is used to connect a host interface to this other SIMH instance::

    ATTACH HIn tba

The host connection may be broken with the command ::

    DETACH HIn

Summary
=======
Additional devices
------------------
`Table 1`_ lists the |IMP/TIP| specific devices added to SIMH along with their I/O address,
interrupt number and vector,
and DMC channel assignment.
Note that all the |IMP/TIP| devices use the H316 extended interrupts and so interrupt 1,
for example,
corresponds to A bit 1 in the SMK 120 instruction.
`Table 2`_ shows the extended interrupt mask associated with these devices.

.. table:: Table 1 – |IMP/TIP| devices
   :name: Table 1

   +-----------------------------+-------------------------------+-----------+-----------------------------+
   | Device                      | Interrupt                     | DMC\ |10| | Description                 |
   +--------------+--------------+--------------+----------------+           |                             |
   | Name         | Address\ |8| | Number\ |10| | Vector\ |8|    |           |                             |
   +==============+==============+==============+================+===========+=============================+
   | |WDT|        | ``26``       |              | ``000062``     |           | Watch dog timer             |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |RTC|        | ``40``       | ``15``       | ``000102``     |           | Real-time clock             |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |IMP|        | ``41``       | ``16``       | ``000103``     |           | Task switch                 |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |IMP|        | ``42``       |              |                |           | MLC support                 |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |HI4|\ [#1]_ | ``50``       | ``9``        | ``000074``     | ``10``    | Host interface #4 (RX)      |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``4``        | ``000067``     | ``5``     | Host interface #4 (TX)      |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |HI3|\ [#2]_ | ``51``       | ``10``       | ``000075``     | ``16``    | Host interface #3 (RX)      |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``5``        | ``000070``     | ``15``    | Host interface #3 (TX)      |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |HI2|        | ``60``       | ``14``       | ``000101``     | ``14``    | Host interface #2 (RX)      |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``12``       | ``000077``     | ``12``    | Host interface #2 (TX)      |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |HI1|        | ``70``       | ``13``       | ``000100``     | ``13``    | Host interface #1 (RX)      |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``11``       | ``000076``     | ``11``    | Host interface #1 (TX)      |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |MI1|        | ``71``       | ``1``        | ``000064``     | ``1``     | Modem interface #1 (RX)     |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``6``        | ``000071``     | ``6``     | Modem interface #1 (TX)     |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |MI2|        | ``72``       | ``2``        | ``000065``     | ``2``     | Modem interface #2 (RX)     |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``7``        | ``000072``     | ``7``     | Modem interface #2 (TX)     |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |MI3|        | ``73``       | ``3``        | ``000066``     | ``3``     | Modem interface #3 (RX)     |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``8``        | ``000073``     | ``8``     | Modem interface #3 (TX)     |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |MI4|        | ``74``       | ``4``        | ``000067``     | ``4``     | Modem interface #4 (RX)     |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``9``        | ``000074``     | ``9``     | Modem Interface #4 (TX)     |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+
   | |MI5|        | ``75``       | ``5``        | ``000070``     | ``5``     | Modem interface #5 (RX)     |
   |              |              +--------------+----------------+-----------+-----------------------------+
   |              |              | ``10``       | ``000075``     | ``10``    | Modem interface #5 (TX)     |
   +--------------+--------------+--------------+----------------+-----------+-----------------------------+

.. table:: Table 2 – Extended interrupt mask
   :name: Table 2

   +----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+---------+----------+
   | 1        | 2        | 3        | 4        | 5        | 6        | 7        | 8        | 9        | 10       | 11       | 12       | 13       | 14       | 15      | 16       |
   +==========+==========+==========+==========+==========+==========+==========+==========+==========+==========+==========+==========+==========+==========+=========+==========+
   | ``M1RX`` | ``M2RX`` | ``M3RX`` | ``M4RX`` | ``M5RX`` | ``M1TX`` | ``M2TX`` | ``M3TX`` | ``M4TX`` | ``M5TX`` | ``H1TX`` | ``H2TX`` | ``H1RX`` | ``H2RX`` | ``RTC`` | ``TASK`` |
   +----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+---------+----------+
   |          |          |          | ``H4TX`` | ``H3TX`` |          |          |          | ``H4RX`` | ``H3RX`` |          |          |          |          |         |          |
   +----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+----------+---------+----------+

.. [#1] The modem 4 interrupt and DMC conflict with host 4 – only one of the two may be active.

.. [#2] The modem 5 interrupt and DMC conflict with host 3 – only one of the two may be active.

Modem interface I/O instructions
--------------------------------
`Table 3`_ summarizes the modem interface I/O instructions.
In this table,
|n| represents the modem number,
(1 thru 5),
and |dd| is the device I/O address assigned to that modem
(71\ |8| thru 75\ |8|).

.. |dd| replace:: :html:`<samp><var>dd</var></samp>`

.. table:: Table 3 – Modem instructions
   :name: Table 3

   ======================================  ======================================  ============================
   Mnemonic                                Opcode                                  Operation
   ======================================  ======================================  ============================
   :html:`<samp>M<var>n</var>OUT</samp>`   :html:`<samp>0300<var>dd</var></samp>`  Start modem output
   :html:`<samp>M<var>n</var>UNXP</samp>`  :html:`<samp>0301<var>dd</var></samp>`  Un-cross patch modem
   :html:`<samp>M<var>n</var>LXP</samp>`   :html:`<samp>0302<var>dd</var></samp>`  Enable line cross patch
   :html:`<samp>M<var>n</var>IXP</samp>`   :html:`<samp>0303<var>dd</var></samp>`  Enable interface cross patch
   :html:`<samp>M<var>n</var>IN</samp>`    :html:`<samp>0304<var>dd</var></samp>`  Start modem input
   :html:`<samp>M<var>n</var>ERR</samp>`   :html:`<samp>0704<var>dd</var></samp>`  Skip on modem error
   ======================================  ======================================  ============================

Host interface I/O instructions
-------------------------------
`Table 4`_ summarizes the host interface I/O instructions.
In this table,
|n| represents the host number
(1 thru 4),
and |dd| is the device I/O address assigned to that host
(50\ |8|, 51\ |8|, 60\ |8|, or 70\ |8|).

.. table:: Table 4 – Host Instructions
   :name: Table 4

   ======================================  ======================================  ============================
   Mnemonic                                Opcode                                  Operation
   ======================================  ======================================  ============================
   :html:`<samp>H<var>n</var>ROUT</samp>`  :html:`<samp>0300<var>dd</var></samp>`  Start regular output to host
   :html:`<samp>H<var>n</var>IN</samp>`    :html:`<samp>0301<var>dd</var></samp>`  Start host input
   :html:`<samp>H<var>n</var>FOUT</samp>`  :html:`<samp>0302<var>dd</var></samp>`  Start host final output
   :html:`<samp>H<var>n</var>XP</samp>`    :html:`<samp>0303<var>dd</var></samp>`  Cross patch host
   :html:`<samp>H<var>n</var>UNXP</samp>`  :html:`<samp>0304<var>dd</var></samp>`  Un-cross patch host
   :html:`<samp>H<var>n</var>ENAB</samp>`  :html:`<samp>0305<var>dd</var></samp>`  Enable host interface
   :html:`<samp>H<var>n</var>ERR</samp>`   :html:`<samp>0700<var>dd</var></samp>`  Skip on host error
   :html:`<samp>H<var>n</var>RDY</samp>`   :html:`<samp>0701<var>dd</var></samp>`  Skip on host ready
   :html:`<samp>H<var>n</var>EOM</samp>`   :html:`<samp>0702<var>dd</var></samp>`  Skip on end of host message
   :html:`<samp>H<var>n</var>FULL</samp>`  :html:`<samp>0705<var>dd</var></samp>`  Skip on host buffer full
   ======================================  ======================================  ============================

Other I/O instructions
----------------------
`Table 5`_ summarizes the additional miscellaneous I/O instructions.

.. |CPU| replace:: :html:`<samp>CPU</samp>`

.. table:: Table 5 - Other Instructions
   :name: Table 5

   =============================  ===========================  ======  ======================================================
   Mnemonic                       Opcode                       Device  Operation
   =============================  ===========================  ======  ======================================================
   :html:`<samp>SMK 120</samp>`   :html:`<samp>170120</samp>`  |CPU|   Set extended interrupt mask
   \                              :html:`<samp>030026</samp>`  |WDT|_  Reset watch dog timer
   \                              :html:`<samp>170026</samp>`  |WDT|_  Set status lights
   :html:`<samp>AMI512</samp>`    :html:`<samp>070026</samp>`  |WDT|_  Skip if this machine is a DDP-516\ [#3]_
   :html:`<samp>CLKON</samp>`     :html:`<samp>030040</samp>`  |RTC|_  Enable |RTC|
   :html:`<samp>CLKOFF</samp>`    :html:`<samp>031040</samp>`  |RTC|_  Disable |RTC|
   :html:`<samp>RDCLOK</samp>`    :html:`<samp>131040</samp>`  |RTC|_  Read |RTC| count and always skip
   :html:`<samp>TASK</samp>`      :html:`<samp>030041</samp>`  |IMP|_  Cause task switch interrupt
   :html:`<samp>RDIMPN</samp>`    :html:`<samp>131041</samp>`  |IMP|_  Read |IMP| number and skip
   :html:`<samp>AMIMLC</samp>`    :html:`<samp>070042</samp>`  |IMP|_  Skip if this machine is a multi-line controller\ [#4]_
   =============================  ===========================  ======  ======================================================

.. [#3] On SIMH, this instruction is a NOP and never skips
        (SIMH simulates an H316, not the DDP-516).

.. [#4] "Multi-line controller" was the official name for the |TIP|.
        |MLC| support is not implemented and this instruction is currently a NOP and never skips.
