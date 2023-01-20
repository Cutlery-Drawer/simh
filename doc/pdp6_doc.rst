.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. |ATTACH|   replace:: :html:`<samp>ATTACH</samp>`
.. |SET|      replace:: :html:`<samp>SET</samp>`
.. |SHOW|     replace:: :html:`<samp>SHOW</samp>`
.. |EXAMINE|  replace:: :html:`<samp>EXAMINE</samp>`
.. |DEPOSIT|  replace:: :html:`<samp>DEPOSIT</samp>`
.. |n|        replace:: :html:`<samp><var>n</var></samp>`
.. |device|   replace:: :html:`<samp><var>device</var></samp>`
.. |filename| replace:: :html:`<samp><var>filename</var></samp>`
.. |port|     replace:: :html:`<samp><var>port</var></samp>`

*************************
DEC PDP-6 simulator usage
*************************
:Date: 2022-10-22
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
Originally the DEC PDP-10 computer started as the PDP-6.
This was a 36-bit computer that was designed for timesharing,
which was introduced in 1964.
The original goal of the machine was to allow for processing of many 6-bit characters at a time.
36 bits were also common in machines like the IBM 7090, GE 645, and Univac 11xx lines.
Several systems influenced the design of the PDP-6,
like CTSS, Lisp, support for larger memory.
The PDP-6 was canceled by DEC due to production problems.
The engineers designed a smaller replacement,
which happened to be a 36-bit computer that looked very much like the PDP-6.
This was called the PDP-10.
Later renamed to "DECSystem-10".
The system supported up to 256K words of memory.

The first PDP-10 was labeled KA10,
and added a few instructions to the PDP-6.
Both the PDP-6 and PDP-10 used a base and limit relocation scheme.
The KA10 generally offered two registers,
one of user data and the second for user-shared code.
These were referred to the Low-Segment and High-Segment;
the High-Segment could be shared with several jobs.
The next version was called KI10 for Integrated.
This added support for paging and double-precision floating-point instructions.
It also added 4 sets of registers to improve context-switching time.
It could also support up to 4Mega words of memory.
Following the KI10 was the KL10 (for Low-Cost).
The KL10 added double-precision integer instructions and instructions to improve COBOL performance.
This was the first version which was microcoded.
The KL10 was extended to support user programs larger then 256k.
The final version to make it to market was the KS10 (for Small),
this was a bit-slice version of the PDP-10 which used UNIBUS devices which were cheaper then the KL10 devices.

The original operating system for the PDP-6/PDP-10 was just called "Monitor".
It was designed to fit into 6K words.
Swapping was added around the third release.
The sixth release saw the addition of virtual memory.
Around the fourth release, it was given the name "TOPS-10".
Around this time, BBN was working on a paging system and implemented it on the PDP-10.
This was called "Tenex".
This was later adopted by DEC and became "Tops-20".

During the mid-1960s,
a group at MIT,
who where not happy with how Multics was being developed,
decided to create their own operating system which they called Incompatible Timesharing System ("ITS"),
was a play on the original project called Compatible Timesharing System ("CTSS").
CTSS was implemented by MIT on their IBM 7090 as an experimental system that allowed multiple timesharing users to co-exist on the same machine running batch processing.
Hence the term "compatable".

Also during the mid-1960s,
a group at Stanford Artificial Intelligence Laboratory (SAIL),
started with a version of TOPS-10 and heavily modified it to become WAITS.

During the 1970s,
Tymshare starting with DEC TOPS-10 system modified it to support random access files,
paging with working sets and spawn-able processes.
This ran on the KI10, KL10 and KS10.

The PDP-10 was ultimately replaced by the VAX.

Simulator files
===============
To compile the DEC 6 simulator,
you must define ``USE_INT64`` as part of the compilation command-line.

+--------------+-----------------+--------------------------------+
| Subdirectory | File            | Contains                       |
+==============+=================+================================+
| ``PDP10/``   | ``kx10_defs.h`` | KA10 simulator definitions     |
|              +-----------------+--------------------------------+
|              | ``kx10_cpu.c``  | PDP-6 CPU                      |
|              +-----------------+--------------------------------+
|              | ``kx10_sys.c``  | PDP-6 system interface         |
|              +-----------------+--------------------------------+
|              | ``kx10_cty.c``  | Console terminal interface     |
|              +-----------------+--------------------------------+
|              | ``kx10_pt.c``   | Paper tape reader and punch    |
|              +-----------------+--------------------------------+
|              | ``kx10_cr.c``   | Punch card reader              |
|              +-----------------+--------------------------------+
|              | ``kx10_cp.c``   | Punch card punch               |
|              +-----------------+--------------------------------+
|              | ``ka10_dpy.c``  | DEC 340 graphics interface     |
|              +-----------------+--------------------------------+
|              | ``pdp6_dct.c``  | PDP-6 data controller          |
|              +-----------------+--------------------------------+
|              | ``pdp6_dtc.c``  | PDP-6 551 DECtape controller   |
|              +-----------------+--------------------------------+
|              | ``pdp6_mtc.c``  | PDP-6 556 magtape controller   |
|              +-----------------+--------------------------------+
|              | ``pdp6_dsk.c``  | PDP-6 270 disk controller      |
|              +-----------------+--------------------------------+
|              | ``pdp6_dcs.c``  | PDP-7 630 terminal multiplexer |
+--------------+-----------------+--------------------------------+

PDP-6 features
==============
The PDP-6 simulator is configured as follows:

.. |LPT|  replace:: :html:`<samp>LPT</samp>`
.. |LP10| replace:: :html:`<samp>LP10</samp>`

==============  ==============================
Device name(s)  Simulates
==============  ==============================
|CPU|_          PDP-6 CPU with 256KW of memory
|CTY|_          Console TTY
|PTP|_          Paper tape punch
|PTR|_          Paper tape reader
|LPT|           |LP10| line printer
|CR|_           |CR10| card reader
|CP|_           |CP10| card punch
|DCT|_          Data control type 136
|DTC|_          Type 551 DECtape controller
|MTC|_          Type 516 magtape controller
|DSK|_          Type 270 disk controller
|DCS|_          Type 630 terminal multiplexer
==============  ==============================

.. |CPU| replace:: :html:`<samp>CPU</samp>`

|CPU|
-----
The |CPU| options include setting memory size,
and other options.

====================================  ==================================
:html:`<samp>SET CPU 16K</samp>`      Set memory to 16K
:html:`<samp>SET CPU 32K</samp>`      Set memory to 32K
:html:`<samp>SET CPU 48K</samp>`      Set memory to 48K
:html:`<samp>SET CPU 64K</samp>`      Set memory to 64K
:html:`<samp>SET CPU 96K</samp>`      Set memory to 96K
:html:`<samp>SET CPU 128K</samp>`     Set memory to 128K
:html:`<samp>SET CPU 256K</samp>`     Set memory to 256K
:html:`<samp>SET CPU NOMAOFF</samp>`  Set traps to default of 040
:html:`<samp>SET CPU MAOFF</samp>`    Move trap vectors from 040 to 0140
:html:`<samp>SET CPU NOIDLE</samp>`   Disable Idle detection
:html:`<samp>SET CPU IDLE</samp>`     Enable Idle detection
====================================  ==================================

|CPU| registers include the visible state of the processor as well as the control registers for the interrupt system.

=============================  ====  ==========================
Name                           Size  Comments
=============================  ====  ==========================
:html:`<samp>PC</samp>`        18    Program counter
:html:`<samp>FLAGS</samp>`     18    Flags
:html:`<samp>FM0-FM17</samp>`  36    Fast memory
:html:`<samp>SW</samp>`        36    Console SW Register
:html:`<samp>MI</samp>`        36    Memory indicators
:html:`<samp>AS</samp>`        18    Console AS register
:html:`<samp>ABRK</samp>`      1     Address break
:html:`<samp>ACOND</samp>`     5     Address condition switches
:html:`<samp>PIR</samp>`       8     Priority interrupt request
:html:`<samp>PIH</samp>`       8     Priority interrupt hold
:html:`<samp>PIE</samp>`       8     Priority interrupt enable
:html:`<samp>PIENB</samp>`     1     Enable priority system
:html:`<samp>BYF5</samp>`      1     Byte flag
:html:`<samp>UUO</samp>`       1     UUO cycle
:html:`<samp>PL</samp>`        18    Program limit low
:html:`<samp>RL</samp>`        18    Program relation low
:html:`<samp>PUSHOVER</samp>`  1     Push overflow flag
:html:`<samp>MEMPROT</samp>`   1     Memory protection flag
:html:`<samp>NXM</samp>`       1     Non-existing memory access
:html:`<samp>CLK</samp>`       1     Clock interrupt
:html:`<samp>OV</samp>`        1     Overflow enable
:html:`<samp>PCCHG</samp>`     1     PC change interrupt
:html:`<samp>APRIRQ</samp>`    3     APR interrupt number
=============================  ====  ==========================

The CPU can maintain a history of the most recently executed instructions.

.. |SET CPU HISTORY|  replace:: :html:`<samp>SET CPU HISTORY</samp>`
.. |SHOW CPU HISTORY| replace:: :html:`<samp>SHOW CPU HISTORY</samp>`
.. _SET CPU HISTORY:
.. _SHOW CPU HISTORY:

This is controlled by the |SET CPU HISTORY| and |SHOW CPU HISTORY| commands:

==================================================  ========================================
|SET CPU HISTORY|                                   Clear history buffer
:html:`<samp>SET CPU HISTORY=0</samp>`              Disable history
:html:`<samp>SET CPU HISTORY=<var>n</var></samp>`   Enable history, length = |n|
|SHOW CPU HISTORY|                                  Print CPU history
:html:`<samp>SHOW CPU HISTORY=<var>n</var></samp>`  Print first |n| entries of CPU history
==================================================  ========================================

Instruction tracing shows the Program counter,
the contents of AC selected,
the computed effective address.
``AR`` is generally the contents the memory location referenced by EA.
``RES`` is the result of the instruction.
``FLAGS`` shows the flags after the instruction is executed.
``IR`` shows the instruction being executed.

Unit record I/O devices
-----------------------

.. _Device 120:
.. _Console teletype:
.. |CTY| replace:: :html:`<samp>CTY</samp>`
.. _CTY:

Console teletype (|CTY|) (Device 120)
"""""""""""""""""""""""""""""""""""""
The console station allows for communications with the operating system.

===============================  ==============================
:html:`<samp>SET CTY 7B</samp>`  7-bit characters, space parity
:html:`<samp>SET CTY 8B</samp>`  8-bit characters, space parity
:html:`<samp>SET CTY 7P</samp>`  7-bit characters, space parity
:html:`<samp>SET CTY UC</samp>`  Translate lowercase to uppercase
===============================  ==============================

The |CTY| also supports a method for stopping TOPS-10 operating system.

=================================  ========================
:html:`<samp>SET CTY STOP</samp>`  Deposit 1 in location 30
=================================  ========================

.. _Device 104:
.. _Paper tape reader:
.. |PTR| replace:: :html:`<samp>PTR</samp>`
.. _PTR:

Paper tape reader (|PTR|) (Device 104)
""""""""""""""""""""""""""""""""""""""
Reads a paper tape from a disk file.

.. _Device 100:
.. _Paper tape punch:
.. |PTP| replace:: :html:`<samp>PTP</samp>`
.. _PTP:

Paper tape punch (|PTP|) (Device 100)
"""""""""""""""""""""""""""""""""""""
Punches a paper tape to a disk file.

.. _Device 150:
.. _Card reader:
.. |CR|   replace:: :html:`<samp>CR</samp>`
.. |CR10| replace:: :html:`<samp>CR10</samp>`
.. _CR:
.. _CR10:

Card reader (|CR|) (Device 150)
"""""""""""""""""""""""""""""""
The card reader (|CR|) reads data from a disk file.
Card reader files can either be text
(one character per column)
or column binary
(two characters per column).
The file type can be specified with a |SET| command:

=====================================================  =================================
:html:`<samp>SET CR<var>n</var> FORMAT=TEXT</samp>`    Set ASCII text mode
:html:`<samp>SET CR<var>n</var> FORMAT=BINARY</samp>`  Set for binary card images
:html:`<samp>SET CR<var>n</var> FORMAT=BCD</samp>`     Set for BCD records
:html:`<samp>SET CR<var>n</var> FORMAT=CBN</samp>`     Set for column binary BCD records
:html:`<samp>SET CR<var>n</var> FORMAT=AUTO</samp>`    Automatically determine format
=====================================================  =================================

or in the |ATTACH| command:

===============================================================================  ==================================================================
:html:`<samp>ATTACH CR<var>n</var> <var>file</var></samp>`                       Attach a file
:html:`<samp>ATTACH CR<var>n</var> -f <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp>ATTACH CR<var>n</var> -s <var>file</var></samp>`                    Add file onto current cards to read
:html:`<samp>ATTACH CR<var>n</var> -e <var>file</var></samp>`                    After file is read in, the reader will receive an end-of-file flag
===============================================================================  ==================================================================

.. _Device 110:
.. _Card punch:
.. |CP|   replace:: :html:`<samp>CP</samp>`
.. |CP10| replace:: :html:`<samp>CP10</samp>`
.. _CP:
.. _CP10:

Card punch (|CP|) (Device 110)
""""""""""""""""""""""""""""""
The card reader (|CP|) writes data to a disk file.
Cards are simulated as ASCII lines with terminating newlines.
Card punch files can either be text
(one character per column)
or column binary
(two characters per column).
The filetype can be specified with a |SET| command:

=========================================  =================================
:html:`<samp>SET CP FORMAT=TEXT</samp>`    Set ASCII text mode
:html:`<samp>SET CP FORMAT=BINARY</samp>`  Set for binary card images
:html:`<samp>SET CP FORMAT=BCD</samp>`     Set for BCD records
:html:`<samp>SET CP FORMAT=CBN</samp>`     Set for column binary BCD records
:html:`<samp>SET CP FORMAT=AUTO</samp>`    Automatically determine format
=========================================  =================================

or in the |ATTACH| command:

===================================================================  ===================================================================
:html:`<samp>ATTACH CP <var>file</var></samp>`                       Attach a file
:html:`<samp>ATTACH CP -f <var>format</var> <var>file</var></samp>`  Attach a file with the given :html:`<samp><var>format</var></samp>`
===================================================================  ===================================================================

.. _Device 124:
.. _Line printer:
.. |LP| replace:: :html:`<samp>LP</samp>`
.. _LP:

Line printer (|LP|) (Device 124)
""""""""""""""""""""""""""""""""
The line printer (|LP|) writes data to a disk file as ASCII text with terminating newlines.
Currently set to handle standard signals to control paper advance.

============================================  ============================================
:html:`<samp>SET LP<var>n</var> LC</samp>`    Allow printer to print lowercase
:html:`<samp>SET LP<var>n</var> UC</samp>`    Print only uppercase
:html:`<samp>SET LP<var>n</var> UTF8</samp>`  Print control characters as UTF-8 characters
============================================  ============================================

The first character of the line controls skipping.

=========  ======================
Character  Effect
=========  ======================
``014``    Skip to top of form
``013``    Skip mod 20 lines
``020``    Skip to next even line
``021``    Skip to third line
``022``    Skip one line
``023``    Skip mod 10 lines
=========  ======================

.. _Device 130:
.. _Type 340 graphics display:

Type 340 graphics display (Device 130)
""""""""""""""""""""""""""""""""""""""
By default,
this device is not enabled.
When enabled and commands are sent to it,
a graphics windows will display.

.. _Device 300:
.. _DCS type 630 terminal multiplexer:
.. |DCS| replace:: :html:`<samp>DCS</samp>`
.. _DCS:

|DCS| type 630 terminal multiplexer (Device 300)
""""""""""""""""""""""""""""""""""""""""""""""""
Terminal multiplexers must be attached in order to work.
The |ATTACH| command specifies the port to be used for Telnet sessions:

=============================================================  =====================
:html:`<samp>ATTACH <var>device</var> <var>port</var></samp>`  Set up listening port
=============================================================  =====================

where |port| is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

Once attached and the simulator is running,
the multiplexer listens for connections on the specified port.
It assumes that any incoming connection is a Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a :html:`<samp>SET <var>device</var> DISCONNECT</samp>` command,
or a :html:`<samp>DETACH <var>device</var></samp>` command.

==================================================================  ===================
:html:`<samp>SET <var>device</var> DISCONNECT=<var>n</var></samp>`  Disconnect line |n|
==================================================================  ===================

The |device| implements the following special |SHOW| commands:

=======================================================  ===========================================
:html:`<samp>SHOW <var>device</var> CONNECTIONS</samp>`  Display current connections to the |device|
:html:`<samp>SHOW <var>device</var> STATISTICS</samp>`   Display statistics for active connections
:html:`<samp>SHOW <var>device</var> LOG</samp>`          Display logging for all lines
=======================================================  ===========================================

Logging can be controlled as follows:

===============================================================================  ====================================
:html:`<samp>SET <var>device</var> LOG=<var>n</var>=<var>filename</var></samp>`  Log output of line |n| to |filename|
:html:`<samp>SET <var>device</var> NOLOG</samp>`                                 Disable logging and close log file
===============================================================================  ====================================

.. _Device 200:
.. _Device 204:
.. _Device 200/204:
.. _DCT type 136 data control:
.. _type 136 data controller:
.. |DCT| replace:: :html:`<samp>DCT</samp>`
.. _DCT:

|DCT| type 136 data control (Device 200/204)
""""""""""""""""""""""""""""""""""""""""""""
This device acted as a data multiplexer for the DECtape/magtape and disk.
Individual devices could be assigned channels on this device.
Devices which use the |DCT| include a |DCT| option which takes two octal digits,
the first is the |DCT| device 0 or 1,
the second is the channel 1 to 7.

.. _Device 210:
.. _DTC type 551 DECtape controller:
.. |DTC| replace:: :html:`<samp>DTC</samp>`
.. _DTC:

|DTC| type 551 DECtape controller (Device 210)
""""""""""""""""""""""""""""""""""""""""""""""
This was the standard DECtape controller for the PDP-6.
This controller loads the tape into memory and uses the buffered version.
You need to specify which |DCT| unit and channel the tape is connected to.

==============================================  ==================================================
:html:`<samp>SET DTC DCT=<var>uc</var></samp>`  Set the |DTC| to connect to |DCT| unit and channel
==============================================  ==================================================

Each individual tape drive support several options:

====================================================  =====================================
:html:`<samp>SET DTC<var>n</var> LOCKED</samp>`       Set the magtape to be read-only
:html:`<samp>SET DTC<var>n</var> WRITEENABLE</samp>`  Set the magtape to be writable
:html:`<samp>SET DTC<var>n</var> 18b</samp>`          (Default) Consider tapes to be 18-bit
:html:`<samp>SET DTC<var>n</var> 16b</samp>`          Convert tapes from 16-bit to 18-bit
:html:`<samp>SET DTC<var>n</var> 12b</samp>`          Convert tapes from 12-bit to 18-bit
====================================================  =====================================

.. _Device 220:
.. _MTC type 516 magtape controller:
.. |MTC| replace:: :html:`<samp>MTC</samp>`
.. _MTC:

|MTC| type 516 magtape controller (Device 220)
""""""""""""""""""""""""""""""""""""""""""""""
This was the standard Magtape controller for the PDP-6.
This device handled only 7-track tapes.
The simulator has the option to automatically translate 8-track tapes into 7-track tapes.
You need to specify which |DCT| unit and channel the tape is connected to.

==============================================  ==================================================
:html:`<samp>SET MTC DCT=<var>uc</var></samp>`  Set the |MTC| to connect to |DCT| unit and channel
==============================================  ==================================================

Each individual tape drive support several options:

====================================================  =======================================================
:html:`<samp>SET MTC<var>n</var> LOCKED</samp>`       Set this unit to be read-only
:html:`<samp>SET MTC<var>n</var> WRITEENABLE</samp>`  Set this unit to be writable
:html:`<samp>SET MTC<var>n</var> 9T</samp>`           Set this unit to simulated 9-track format
:html:`<samp>SET MTC<var>n</var> 7T</samp>`           Set this unit to read/write 7-track tapes (with parity)
====================================================  =======================================================

.. _Device 270:
.. _DSK type 270 disk controller:
.. |DSK| replace:: :html:`<samp>DSK</samp>`
.. _DSK:

|DSK| type 270 disk controller (Device 270)
"""""""""""""""""""""""""""""""""""""""""""
The 270 disk could support up to 4 units.
The controller had to be connected to a `type 136 data controller`_.
You need to specify which |DCT| unit and channel the disk is connected to.

==============================================  ==================================================
:html:`<samp>SET DSK DCT=<var>uc</var></samp>`  Set the |DSK| to connect to |DCT| unit and channel
==============================================  ==================================================

Each individual disk drive support several options:

====================================================  =============================
:html:`<samp>SET DSK<var>n</var> LOCKED</samp>`       Set this unit to be read-only
:html:`<samp>SET DSK<var>n</var> WRITEENABLE</samp>`  Set this unit to be writable
====================================================  =============================

Symbolic display and input
==========================
The KA10 simulator implements symbolic display and input.
These are controlled by the following switches to the |EXAMINE| and |DEPOSIT| commands:

.. table::
   :class: switches

   ======  ===================================
   ``-a``  Display/enter ASCII data
   ``-p``  Display/enter packed ASCII data
   ``-c``  Display/enter Sixbit character data
   ``-m``  Display/enter symbolic instructions
   \       Display/enter octal data
   ======  ===================================

Symbolic instructions can be of the formats:

* :html:`<samp>Opcode <var>ac</var>,<var>operand</var></samp>`
* :html:`<samp>Opcode <var>operands</var></samp>`
* :html:`<samp>I/O opcode <var>device</var>,<var>address</var></samp>`

Operands can be one or more of the following in order:

* Optional ``@`` for indirection
* ``+`` or ``–`` to set sign
* Octal number
* Optional (\ :html:`<samp><var>ac</var></samp>`\ ) for indexing
