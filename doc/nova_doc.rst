.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

*****************************************
          Nova Simulator Usage
*****************************************
:Date: 2020-04-28
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

This memorandum documents the Nova simulator.

Simulator files
===============

``sim/``
    | ``scp.h``
    | ``sim_console.h``
    | ``sim_defs.h``
    | ``sim_fio.h``
    | ``sim_rev.h``
    | ``sim_sock.h``
    | ``sim_tape.h``
    | ``sim_timer.h``
    | ``sim_tmxr.h``
    | ``scp.c``
    | ``sim_console.c``
    | ``sim_fio.c``
    | ``sim_sock.c``
    | ``sim_tape.c``
    | ``sim_timer.c``
    | ``sim_tmxr.c``

``sim/nova/``
    | ``nova_defs.h``
    | ``nova_cpu.c``
    | ``nova_clk.c``
    | ``nova_dkp.c``
    | ``nova_dsk.c``
    | ``nova_lp.c``
    | ``nova_mta.c``
    | ``nova_plt.c``
    | ``nova_qty.c``
    | ``nova_sys.c``
    | ``nova_tt.c``
    | ``nova_tt1.c``

Nova features
=============
The Nova simulator is configured as follows:

==================  ====================================
Device names        Simulates
==================  ====================================
``CPU``             Nova, Nova 3, Nova 4 CPU with 32KW of memory, or Keronix CPU with 64KW of memory
``-``               Hardware multiply/divide
``PTR``, ``PTP``    Paper tape reader/punch
``TTI``, ``TTO``    Console terminal
``TTI1``, ``TTO1``  Second terminal
``LPT``             Line printer
``PLT``             Plotter
``RTC``             Real-time clock
``DSK``             Head-per-track disk controller
``DKP``             Moving head disk controller with four drives
``MTA``             Magnetic tape controller with eight drives
``QTY``             4060 multiplexer with up to 64 lines
``ALM``             4255 multiplexer with up to 64 lines
==================  ====================================

The Nova simulator implements these unique stop conditions:

- Reference to undefined I/O device, and ``STOP_DEV`` is set
- More than ``INDMAX`` indirect addresses are detected during an interrupt
- More than ``INDMAX`` indirect addresses are detected during memory reference address decoding

Note that indirect address loop detection does not exist on unmapped Novas.
Some DG diagnostics test thousands of levels of indirect addressing.
``INDMAX`` may have to be set to 32,000 to get diagnostics to run properly.

The ``LOAD`` command supports standard binary format tapes.
The ``DUMP`` command is not implemented.

All devices except ``TTI`` can be disabled or enabled, by the commands::

    SET <dev> DISABLED
    SET <dev> ENABLED

All devices except ``QTY`` are enabled by default.

CPU
---
The only CPU options are the presence of the optional instructions and the size of main memory.

===================  ====================================================
``SET CPU MDV``      Enable multiply/divide [1]_
``SET CPU EXT64KW``  Enable extended 64KW memory mode
``SET CPU NOVA3``    Set Nova3 CPU [2]_
``SET CPU NOVA4``    Set Nova4 CPU [3]_
``SET CPU KERONIX``  Set Keronix CPU [4]_
``SET CPU NONE``     Disable all optional instructions
``SET CPU 4K``       Set memory size = 4K
``SET CPU 8K``       Set memory size = 8K
``SET CPU 12K``      Set memory size = 12K
``SET CPU 16K``      Set memory size = 16K
``SET CPU 20K``      Set memory size = 20K
``SET CPU 24K``      Set memory size = 24K
``SET CPU 28K``      Set memory size = 28K
``SET CPU 32K``      Set memory size = 32K
``SET CPU 36K``      Set memory size = 36K
``SET CPU 40K``      Set memory size = 40K
``SET CPU 44K``      Set memory size = 44K
``SET CPU 48K``      Set memory size = 48K
``SET CPU 52K``      Set memory size = 52K
``SET CPU 56K``      Set memory size = 56K
``SET CPU 60K``      Set memory size = 60K
``SET CPU 64K``      Set memory size = 64K
===================  ====================================================

.. [1] MDV = unsigned multiply/divide instructions
.. [2] Nova 3 = unsigned multiply/divide, stack, trap instructions
.. [3] Nova 4 = unsigned and signed multiply/divide, stack, byte, trap instructions
.. [4] Keronix = unsigned multiply/divide, extended 64KW mode

If memory size is being reduced, and the memory being truncated contains non-zero data, the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 32K.

The CPU supports the ``BOOT`` command.
``BOOT CPU`` simulates the Nova hardware APL (automatic program load) feature.
The switch register (SR) bits 12:17 must contain the device code of the device to be booted.
If the device is a "high-speed" (channel) device, SR bit 0 should also be set.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

===================  ====  ====================================================================
Name                 Size  Comments
===================  ====  ====================================================================
``PC``               15    Program counter
``AC0`` .. ``AC3``   16    Accumulators 0..3
``C``                1     Carry
``SR``               16    Front panel switches
``PI``               16    Priority interrupt mask
``ION``              1     Interrupt enable
``ION_DELAY``        1     Interrupt enable delay for ION
``PWR``              1     Power fail interrupt
``INT``              15    Interrupt pending flags
``BUSY``             15    Device busy flags
``DONE``             15    Device done flags
``DISABLE``          15    Device interrupt disable flags
``STOP_DEV``         1     Stop on undefined IOT
``INDMAX``           16    Maximum number of nested indirects
``PCQ[0:63]``        15    PC prior to last JMP, JMS, or interrupt; most recent PC change first
``WRU``              8     Interrupt character
===================  ====  ====================================================================

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands:

======================  ========================================
``SET CPU HISTORY``     Clear history buffer
``SET CPU HISTORY=0``   Disable history
``SET CPU HISTORY=n``   Enable history, length = ``n``
``SHOW CPU HISTORY``    Print CPU history
``SHOW CPU HISTORY=n``  Print first ``n`` entries of CPU history
======================  ========================================

The maximum length for the history is 65536 entries.

Programmed I/O devices
----------------------

Paper tape reader (PTR)
"""""""""""""""""""""""
The paper tape reader (PTR) reads data from a disk file.
The POS register specifies the number of the next data item to be read.
Thus, by changing POS, the user can backspace or advance the reader.

The paper tape reader implements these registers:

=============  ====  ======================================
Name           Size  Comments
=============  ====  ======================================
``BUF``        8     Last data item processed
``BUSY``       1     Device busy flag
``DONE``       1     Device done flag
``DISABLE``    1     Interrupt disable flag
``INT``        1     Interrupt pending flag
``POS``        32    Position in the input file
``TIME``       24    Time from I/O initiation to interrupt
``STOP_IOE``   1     Stop on I/O error
=============  ====  ======================================

Error handling is as follows:

+--------------+-------------------+-------------------------+
| Error        | ``STOP_IOE``      | Processed as            |
+==============+===================+=========================+
| not attached | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Out of tape             |
+--------------+-------------------+-------------------------+
| end of file  | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Out of tape             |
+--------------+-------------------+-------------------------+
| OS I/O error | x                 | Report error and stop   |
+--------------+-------------------+-------------------------+

Paper tape punch (PTP)
""""""""""""""""""""""
The paper tape punch (PTP) writes data to a disk file.
The POS register specifies the number of the next data item to be written.
Thus, by changing POS, the user can backspace or advance the punch.

The paper tape punch implements these registers:

============  ===========  =========================================
Name          Size         Comments
============  ===========  =========================================
``BUF``       8            Last data item processed
``BUSY``      1            Device busy flag
``DONE``      1            Device done flag
``DISABLE``   1            Interrupt disable flag
``INT``       1            Interrupt pending flag
``POS``       32           Position in the output file
``TIME``      24           Time from I/O initiation to interrupt
``STOP_IOE``  1            Stop on I/O error
============  ===========  =========================================

Error handling is as follows:

+--------------+-------------------+-------------------------+
| Error        | ``STOP_IOE``      | Processed as            |
+==============+===================+=========================+
| not attached | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Out of tape             |
+--------------+-------------------+-------------------------+
| OS I/O error | x                 | Report error and stop   |
+--------------+-------------------+-------------------------+

Terminal input (TTI)
""""""""""""""""""""
The terminal input polls the console keyboard for input.
Terminal input options include the ability to set ANSI mode or limited Dasher compatibility mode:

======================  ==========================
``SET TTI ANSI``        Normal mode
``SET TTI DASHER``      Dasher mode
======================  ==========================

Setting either TTI or TTO changes both devices.
In Dasher mode, carriage return is changed to newline on input, and ``^X`` is changed to backspace.

The terminal input implements these registers:

==============  ====  ==========================
Name            Size  Comments
==============  ====  ==========================
``BUF``         8     Last data item processed
``BUSY``        1     Device busy flag
``DONE``        1     Device done flag
``DISABLE``     1     Interrupt disable flag
``INT``         1     Interrupt pending flag
``POS``         32    Number of characters input
``TIME``        24    Keyboard polling interval
==============  ====  ==========================

Terminal output (TTO)
"""""""""""""""""""""
The terminal output writes to the simulator console window.
Terminal output options include the ability to set ANSI mode or limited Dasher compatibility mode:

===================  ===================
``SET TTI ANSI``     Normal mode
``SET TTI DASHER``   Dasher mode
===================  ===================

Setting either TTI or TTO changes both devices.
In Dasher mode, carriage return is changed to newline on input, and ``^X`` is changed to backspace.

The terminal output implements these registers:

==============  ====  =====================================
Name            Size  Comments
==============  ====  =====================================
``BUF``         8     Last data item processed
``BUSY``        1     Device busy flag
``DONE``        1     Device done flag
``DISABLE``     1     Interrupt disable flag
``INT``         1     Interrupt pending flag
``POS``         32    Number of characters output
``TIME``        24    Time from I/O initiation to interrupt
==============  ====  =====================================

Line printer (LPT)
""""""""""""""""""
The line printer (LPT) writes data to a disk file.
The POS register specifies the number of the next data item to be written.
Thus, by changing POS, the user can backspace or advance the printer.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The line printer implements these registers:

==============  ====  ============================
Name            Size  Comments
==============  ====  ============================
``BUF``         8     Last data item processed
``BUSY``        1     Device busy flag
``DONE``        1     Device done flag
``DISABLE``     1     Interrupt disable flag
``INT``         1     Interrupt pending flag
``POS``         32    Position in the output file
``TIME``        24    Time from I/O initiation to interrupt
``STOP_IOE``    1     Stop on I/O error
==============  ====  ============================

Error handling is as follows:

+--------------+-------------------+-------------------------+
| Error        | ``STOP_IOE``      | Processed as            |
+==============+===================+=========================+
| not attached | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Out of paper            |
+--------------+-------------------+-------------------------+
| OS I/O error | x                 | Report error and stop   |
+--------------+-------------------+-------------------------+

Real-time clock (RTC)
"""""""""""""""""""""
The real-time clock (RTC) line frequency can be adjusted as follows:

=================  ==========================
``SET RTC 60HZ``   Set line frequency to 60Hz
``SET RTC 50HZ``   Set line frequency to 50Hz
=================  ==========================

The default is 60Hz.

The clock implements these registers:

============  ====  ===========================
Name          Size  Comments
============  ====  ===========================
``SELECT``    2     Selected clock interval
``BUSY``      1     Device busy flag
``DONE``      1     Device done flag
``DISABLE``   1     Interrupt disable flag
``INT``       1     Interrupt pending flag
``TIME0``     24    Clock frequency, select = 0
``TIME1``     24    Clock frequency, select = 1
``TIME2``     24    Clock frequency, select = 2
``TIME3``     24    Clock frequency, select = 3
============  ====  ===========================

The real-time clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

Plotter (PLT)
"""""""""""""
The plotter (PLT) writes data to a disk file.
The POS register specifies the number of the next data item to be written.
Thus, by changing POS, the user can backspace or advance the plotter.

The plotter implements these registers:

============  ====  =============================
Name          Size  Comments
============  ====  =============================
``BUF``       8     Last data item processed
``BUSY``      1     Device busy flag
``DONE``      1     Device done flag
``DISABLE``   1     Interrupt disable flag
``INT``       1     Interrupt pending flag
``POS``       32    Position in the output file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  =============================

Error handling is as follows:

+--------------+-------------------+-------------------------+
| Error        | ``STOP_IOE``      | Processed as            |
+==============+===================+=========================+
| not attached | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Out of paper            |
+--------------+-------------------+-------------------------+
| OS I/O error | x                 | Report error and stop   |
+--------------+-------------------+-------------------------+

Second terminal (TTI1, TTO1)
""""""""""""""""""""""""""""
The second terminal consists of two independent devices, TTI1 and TTO1.
The additional terminal performs input and output through a Telnet session connecting into a user-specified port.
The ``ATTACH`` command specifies the port to be used:

======================  =====================
``ATTACH TTI1 <port>``  Set up listening port
======================  =====================

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

Once TTI1 is attached and the simulator is running, the terminal listens for a connection on the specified port.
It assumes that the incoming connection is a Telnet connection.
The connection remains open until disconnected by the Telnet client, or by a ``DETACH TTI1`` command.

The second terminal has two options,
recognized on both devices,
for setting limited Dasher-compatibility mode or ANSI mode:

===================  ===========
``SET TTI1 ANSI``    Normal mode
``SET TTI1 DASHER``  Dasher mode
``SET TTO1 ANSI``    Normal mode
``SET TTO1 DASHER``  Dasher mode
===================  ===========

Setting either TTI1 or TTO1 changes both devices.
In Dasher mode, carriage return is changed to newline on input, and ``^X`` is changed to backspace.
TTO1 supports output logging.
The ``SET TTO1 LOG`` command enables logging:

=========================  ==========================
``SET TTO1 LOG=filename``  Log output to ``filename``
=========================  ==========================

The ``SET TTO1 NOLOG`` command disables logging and closes the open log file, if any.

Other special TTI1 commands:

=========================  ======================================
``SHOW TTI1 CONNECTIONS``  Show current connections
``SHOW TTI1 STATISTICS``   Show statistics for active connections
``SET TTO1 DISCONNECT``    Disconnects the line
=========================  ======================================

The second terminal input implements these registers:

=============  ====  =========================
Name           Size  Comments
=============  ====  =========================
``BUF``        8     Last data item processed
``BUSY``       1     Device busy flag
``DONE``       1     Device done flag
``DISABLE``    1     Interrupt disable flag
``INT``        1     Interrupt pending flag
``TIME``       24    Keyboard polling interval
=============  ====  =========================

The second terminal output implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       8     Last data item processed
``BUSY``      1     Device busy flag
``DONE``      1     Device done flag
``DISABLE``   1     Interrupt disable flag
``INT``       1     Interrupt pending flag
``TIME``      24    Time from I/O initiation to interrupt
============  ====  =====================================

Asynchronous multiplexers (QTY, ALM)
""""""""""""""""""""""""""""""""""""
The QTY and ALM are terminal multiplexers with up to 64 lines.
Either the QTY or ALM can be enabled, but not both;
the ALM is enabled by default.
The number of lines can be changed with the command ::

    SET {QTY|ALM} LINES=n

The line count maximum is 64.

The QTY and ALM support 8-bit input and output of characters.
8-bit I/O may be incompatible with certain operating systems;
7-bit is the default.
The command ::

    SET {QTY|ALM} 8B

enables 8-bit input and output.

The terminal lines perform input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the port to be used::

    ATTACH {-am} {QTY|ALM} <port>

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.
For the ALM multiplexer, the optional switch ``-m`` turns on the multiplexer modem controls;
the optional switch ``-a`` turns on active disconnects (disconnect session if computer clears Data Terminal Ready).
The QTY multiplexer does not support modem control.
Without modem control, the multiplexer behaves as though terminals were directly connected;
disconnecting the Telnet session does not cause any operating system-visible change in line status.

Once the multiplexer is attached and the simulator is running, it listens for connections on the specified port.
It assumes that the incoming connections are Telnet connections.
The connection remains open until disconnected by the simulated program, the Telnet client,
a ``SET {QTY|ALM} DISCONNECT`` command, or a ``DETACH {QTY|ALM}`` command.

Other special QTY/ALM commands:

==============================  ======================================
``SHOW {QTY|ALM} CONNECTIONS``  Show current connections
``SHOW {QTY|ALM} STATISTICS``   Show statistics for active connections
``SET {QTY|ALM} DISCONNECT=n``  Disconnects the specified line
==============================  ======================================

The QTY/ALM implement these registers:

============  ====  =========================
Name          Size  Comments
============  ====  =========================
``BUF``       8     Character buffer
``BUSY``      1     Device busy flag
``DONE``      1     Device done flag
``DISABLE``   1     Device disable flag
``INT``       1     Interrupt pending flag
``MDMCTL``    1     Modem control flag
``AUTODS``    1     Autodisconnect flag
``POLLS``     32    Number of service polls
``STOP_IOE``  1     Stop on I/O error
============  ====  =========================

The multiplexers do not support save and restore.
All open connections are lost when the simulator shuts down or the multiplexer is detached.

Fixed head disk (DSK)
---------------------
Fixed head disk options include the ability to set the number of platters to a fixed value between 1 and 8,
or to autosize the number of platters from the attached file:

====================  =======================
``SET DSK 1P``        One platter (256K)
``SET DSK 2P``        Two platters (512K)
``SET DSK 3P``        Three platters (768K)
``SET DSK 4P``        Four platters (1024K)
``SET DSK 5P``        Five platters (1280K)
``SET DSK 6P``        Six platters (1536K)
``SET DSK 7P``        Seven platters (1792K)
``SET DSK 8P``        Eight platters (2048K)
``SET DSK AUTOSIZE``  Autosized on ``ATTACH``
====================  =======================

The default is 1P (minimum size).
The fixed head disk controller supports the ``BOOT`` command.

The fixed head disk controller implements these registers:

============  ====  =============================
Name          Size  Comments
============  ====  =============================
``STAT``      16    Status
``DA``        16    Disk address
``MA``        16    Memory address
``BUSY``      1     Device busy flag
``DONE``      1     Device done flag
``DISABLE``   1     Device disable flag
``INT``       1     Interrupt pending flag
``WLK``       8     Write lock switches
``TIME``      24    Rotational delay, per sector
``STOP_IOE``  1     Stop on I/O error
============  ====  =============================

Error handling is as follows:

+--------------+-------------------+-------------------------+
| Error        | ``STOP_IOE``      | Processed as            |
+==============+===================+=========================+
| not attached | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Disk not ready          |
+--------------+-------------------+-------------------------+

Fixed head disk data files are buffered in memory;
therefore, end of file and OS I/O errors cannot occur.

Moving head disk (DKP)
----------------------
Moving head disk options include the ability to make units write enabled or write locked,
and to select the type of drive (or autosize):

=========================================  =========================================
``SET DKPn LOCKED``                        Set unit ``n`` write locked
``SET DKPn WRITEENABLED``                  Set unit ``n`` write enabled
``SET DKPn FLOPPY`` (or ``6030``)          Set unit ``n`` to floppy disk
``SET DKPn DSDD`` (or ``6097``)            Set unit ``n`` to double density floppy
``SET DKPn D31`` (or ``4047``)             Set unit ``n`` to Diablo 31
``SET DKPn D44`` (or ``4234``, ``6045``)   Set unit ``n`` to Diablo 44
``SET DKPn C111`` (or ``4048``)            Set unit ``n`` to Century 111
``SET DKPn C114`` (or ``2314``, ``4057``)  Set unit ``n`` to Century 114
``SET DKPn 6225``                          Set unit ``n`` to 6225
``SET DKPn 6099``                          Set unit ``n`` to 6099
``SET DKPn 6227``                          Set unit ``n`` to 6227
``SET DKPn 6070``                          Set unit ``n`` to 6070
``SET DKPn 6103``                          Set unit ``n`` to 6103
``SET DKPn 4231`` (or ``3330``)            Set unit ``n`` to 4231
``SET DKPn AUTOSIZE``                      Set type based on file size at ``ATTACH``
=========================================  =========================================

Units can also be set ``ENABLED`` or ``DISABLED``.
The moving head disk controller supports the ``BOOT`` command.

All drives have 256 16b words per sector.
The other disk parameters are:

===========  ===  ====  =======  =========  ==================
Drive        Cyl  Surf  Sectors  Size (MW)  Model numbers
===========  ===  ====  =======  =========  ==================
floppy       77   1     8        .158       6030
dsdd floppy  77   2     16       .632       6097
D31          203  2     12       1.247      4047, 4237, 4238
D44          408  4     12       5.014      4234, 6045
C111         203  10    6        3.118      4048
C114         203  20    12       12.472     2314, 4057
6225         245  2     20       2.508
6099         192  4     32       6.291
6227         245  6     20       7.526
6070         408  4     24       10.027
6103         192  8     32       12.583
4231         411  19    23       45.979     3330
===========  ===  ====  =======  =========  ==================

The moving head disk controller implements these registers:

===========  ====  =============================
Name         Size  Comments
===========  ====  =============================
``FCCY``     16    Flags, command, cylinder
``USSC``     16    Unit, surface, sector, count
``STAT``     16    Status
``MA``       16    Memory address
``BUSY``     1     Device busy flag
``DONE``     1     Device done flag
``DISABLE``  1     Interrupt disable flag
``INT``      1     Interrupt pending flag
``DIAG``     1     Diagnostic mode flag
``MAP``      2     Map select
``STIME``    24    Seek time, per cylinder
``RTIME``    24    Rotational delay
===========  ====  =============================

Error handling is as follows:

=============  ===========================
Error          Processed as
=============  ===========================
not attached   Disk not ready
end of file    Assume rest of disk is zero
OS I/O error   Report error and stop
=============  ===========================

Magnetic tape (MTA)
-------------------
Magnetic tape options include the ability to make units write enabled or write locked.

==========================  ==============================
``SET MTAn LOCKED``         Set unit ``n`` write locked
``SET MTAn WRITEENABLED``   Set unit ``n`` write enabled
==========================  ==============================

Magnetic tape units can be set to a specific reel capacity in MB, or to unlimited capacity:

==========================  ===================================================
``SET MTAn CAPAC=m``        Set unit ``n`` capacity to ``m`` MB (0 = unlimited)
``SHOW MTAn CAPAC``         Show unit ``n`` capacity in MB
==========================  ===================================================

Units can also be set ``ENABLED`` or ``DISABLED``.
The magnetic tape controller supports the ``BOOT`` command.

The magnetic tape controller implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``CU``        16    Command, unit
``MA``        16    Memory address
``WC``        16    Word count
``STA1``      16    Status word 1
``STA2``      16    Status word 2
``EP``        1     Extended polling mode (not supported)
``BUSY``      1     Device busy flag
``DONE``      1     Device done flag
``DISABLE``   1     Interrupt disable flag
``INT``       1     Interrupt pending flag
``STOP_IOE``  1     Stop on I/O error
``CTIME``     24    Controller delay
``RTIME``     24    Record delay
``UST[0:7]``  32    Unit status, units 0 to 7
``POS[0:7]``  31    Position, units 0 to 7
============  ====  =====================================

Error handling is as follows:

=============  ===========================
Error          Processed as
=============  ===========================
not attached   Tape not ready
end of file    Bad tape
OS I/O error   Report error and stop
=============  ===========================

Symbolic display and input
==========================
The Nova simulator implements symbolic display and input.
Display is controlled by command-line switches:

-a    Display as ASCII character
-c    Display as two packed ASCII characters
-m    Display instruction mnemonics

Input parsing is controlled by the first character typed in or by command-line switches:

================  ===========================
``'`` or ``-a``   ASCII character
``"`` or ``-c``   Two packed ASCII characters
Alphabetic        Instruction mnemonic
Numeric           Octal number
================  ===========================

Instruction input uses standard Nova assembler syntax.
There are three instruction classes: memory reference, IOT, and operate.

Memory reference instructions have the format ::

    memref {ac,}{@}address{,index}

LDA and STA require an initial register; ISZ, DSZ, JSR, and JMP do not.
The syntax for addresses and indices is as follows:

==============  =====  ============  =============================================
Syntax          Mode   Displacement  Comments
==============  =====  ============  =============================================
0 <= n < 0400   0      n
{+/-}n >= 0400  1      {+/-}n – PC   Must be in range [-200, 177], invalid on disk
.+/-n           1      {+/-}n        Must be in range [-200, 177]
{+/-}n,2        2      {+/-}n        Must be in range [-200, 177]
{+/-}n,3        3      {+/-}n        Must be in range [-200, 177]
==============  =====  ============  =============================================

IOT instructions have one of four formats:

==================  =========
Syntax              Example
==================  =========
``iot``             ``HALT``
``iot reg``         ``INTA``
``iot device``      ``SKPDN``
``iot reg,device``  ``DOAS``
==================  =========

Devices may be specified as mnemonics or as numbers in the range 0 - 077.

Operate instructions have the format::

    opcode{#} reg,reg{,skip}

In all Nova instructions, blanks may be substituted for commas as field delimiters.
