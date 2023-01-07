.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

*********************************
     SDS 940 simulator usage
*********************************
:Date: 2014-04-20
:Revision: $Format:%H$
:Copyright: See `LICENSE.txt <../LICENSE.txt>`_ for terms of use.

.. topic:: **Copyright notice**

   The following copyright notice applies to the SIMH source, binary, and documentation:

   .. include:: ../LICENSE.txt

.. sectnum:: :suffix: .
.. contents::
   :backlinks: none
   :depth: 2
   :local:

This memorandum documents the SDS 940 simulator.

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

``sim/sds/``
    | ``sds_defs.h``
    | ``sds_cpu.c``
    | ``sds_drm.c``
    | ``sds_dsk.c``
    | ``sds_io.c``
    | ``sds_lp.c``
    | ``sds_mt.c``
    | ``sds_mux.c``
    | ``sds_rad.c``
    | ``sds_stddev.c``
    | ``sds_sys.c``

SDS 940 features
================
The SDS-940 simulator is configured as follows:

==============  =========================================
Device name(s)  Simulates
==============  =========================================
|CPU|_          SDS-940 CPU with 16KW to 64KW of memory
|CHAN|_         I/O channels
|PTR|_          Paper tape reader
|PTP|_          Paper tape punch
|TTI|_          Console input
|TTO|_          Console output
|LPT|_          Line printer
|RTC|_          Real-time clock
|MUX|_          Terminal multiplexor
|DRM|_          Project Genie drum
|RAD|_          Fixed head disk
|DSK|_          9164/9165 rapid access (moving head) disk
|MT|_           Magnetic tape
==============  =========================================

Most devices can be disabled or enabled with the ``SET <dev> DISABLED`` and ``SET <dev> ENABLED`` commands,
respectively.

The ``LOAD`` command is used to load a line printer carriage-control tape.
The ``DUMP`` command is not implemented.

.. |CPU| replace:: ``CPU``

|CPU|
-----
The CPU options set the size of main memory and the configuration of peripherals. ::

    SET CPU 16K                   Set memory size = 16KW
    SET CPU 32K                   Set memory size = 32KW
    SET CPU 48K                   Set memory size = 48KW
    SET CPU 64K                   Set memory size = 64KW
    SET CPU GENIE                 Enable DRM, set terminal mux to GENIE mode
    SET CPU SDS                   Disable DRM, set terminal mux to SDS mode

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 64KW.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

===============  ====  ===============================================================
Name             Size  Comments
===============  ====  ===============================================================
``P``            14    Program counter
``A``            24    Accumulator A
``B``            24    Accumulator B
``X``            24    Index register
``OV``           1     Overflow indicator
``EM2``          3     Memory extension, quadrant 2
``EM3``          3     Memory extension, quadrant 3
``RL1``          24    User relocation register 1
``RL2``          24    User relocation register 2
``RL4``          12    Kernel relocation register
``NML``          1     Normal mode flag
``USR``          1     User mode flag
``MONUSR``       1     Monitor-to-user trap enable
``ION``          1     Interrupt enable
``INTDEF``       1     Interrupt defer
``INTREQ``       32    Interrupt request flags
``APIACT``       5     Highest active API level
``APIREQ``       5     Highest requesting API level
``XFRREQ``       32    Device transfer request flags
``BPT``          4     Breakpoint switches
``ALERT``        6     Outstanding alert number
``STOP_INVINS``  1     Stop on invalid instruction
``STOP_INVDEV``  1     Stop on invalid device number
``STOP_INVIOP``  1     Stop on invalid I/O operation
``INDLIM``       8     Maximum indirect nesting depth
``EXULIM``       8     Maximum execute nesting depth
``PCQ[0:63]``    14    P prior to last branch or interrupt; most recent P change first
``WRU``          8     Interrupt character
===============  ====  ===============================================================

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY              Clear history buffer
    SET CPU HISTORY=0            Disable history
    SET CPU HISTORY=n            Enable history, length = n
    SHOW CPU HISTORY             Print CPU history
    SHOW CPU HISTORY=n           Print first n entries of CPU history

The maximum length for the history is 65536 entries,
the minimum length is 64 entries.
This history records the CPU mode when the instruction was executed (normal, monitor or user).
The ``SET CPU HISTORY`` command accepts one switch value to optionally suppress recording of instructions in a particular CPU mode::

    SET –n CPU HISTORY=n         Don't record if in normal mode
    SET –m CPU HISTORY=n         Don't record if in monitor mode
    SET –u CPU HISTORY=n         Don't record if in user mode

The SDS 940 simulator implements four types of execution breakpoints,
controlled by command-line switches:

======  ===============================================================
``-e``  Break if ``P`` equals address, unqualified by machine mode
``-m``  Break if ``P`` equals address, machine in monitor mode
``-n``  Break if ``P`` equals address, machine in normal (SDS 930) mode
``-u``  Break if ``P`` equals address, machine in user mode
======  ===============================================================

Breakpoint commands default to ``-e`` behavior if no switch is specified.

Next command
""""""""""""
The Next command is supported to step over a ``BRM``, ``POP``, or ``SYSPOP`` instruction by installing temporary breakpoints at ``P``\ +1 and ``P``\ +2
(and ``P``\ +3 in the case of a ``BRM``)
and then executing the Go command.
The temporary breakpoints are removed should execution be interrupted for any other reason.
For other instructions,
Next is converted to a Step command.

Like the Step command,
the Next command accepts a repeat count as an argument.
Given this code sequence::

    1532: BRM   1220  <-- P
    1533: LDA   4663
    1534: POP   3214
    1535: BRU   4436  <-- non-skip return from POP
    1536: BRM   3260  <-- skip return from POP
    1537: ZRO   3455  <-- argument to subroutine
    1540: BRU   1600  <-- error return
    1541: STA   4652  <-- normal return

The following behavior would be observed for various Next commands executed when ``P``\ =1532:

======  ============================================
Step    Advance to location 1220
Next    Advance to location 1533
Next 2  Advance to location 1534
Next 3  Advance to location 1535 or 1536,
        depending on whether ``POP`` skips on return
======  ============================================

With ``P``\ =1536,
the behavior is thus:

====  ================================
Step  Advance to location 3260
Next  Advance to location 1540 or 1541
====  ================================

Unlike the Step command,
a side-effect of using temporary breakpoints is that diversions into interrupt or trap code become invisible.
With a Step command,
execution dutifully follows into the interrupt or trap routine.

Next command accepts two mutually exclusive switches to modify its behavior.

The ``-f`` switch ("forward") instructs the Next command to set temporary breakpoint(s) following the current location,
regardless of instruction type.
This is useful at the bottom of loops or to avoid going off into unrelated code should an interrupt or memory paging trap occur.
In this example::

    1532  BRX   1220  <-- P
    1533  LDA   4663

a normal Next would advance to either location 1220 or 1533 depending upon the value in the ``X`` register
(it is converted to a Step).
However, Next ``-f`` would only advance to location 1533,
allowing the loop code at 1220 to be executed as many times as specified by ``X``.
Note that if the loop code branches elsewhere,
rather than make the ``BRX`` test,
control will be lost unless caught by another breakpoint.

The ``-a`` switch ("atomic") alters Next behavior to be more like a Step command,
but using temporary breakpoints.
That is,
it will plant a breakpoint at the effective address of a ``BRM`` or ``BRX`` instruction to catch any transfer there.
Its primary use is to step through code ignoring interrupts and traps.

The behavior of various alternative forms of Next are illustrated in this table for different opcodes,
showing where temporary breakpoints are placed.
*"Step"* means Next is converted to a Step command.
*"LDA, etc."* means all non-skipping,
non-branching opcodes,
including I/O instructions.
*"SKx"* means all Skip instructions.
The behavior of Next with an ``EXU`` (execute) instruction is dependent on the instruction that is eventually executed.

.. list-table::
   :header-rows: 1
   :stub-columns: 1

   * - Opcode
     - Next
     - Next ``-a`` (Atomic)
     - Next ``-f`` (Forward)
   * - Bad op
     - Step
     - Step
     - Step
   * - ``BRI``
     - Step
     - ``EA``
     - Step
   * - ``BRM``, ``SBRM``
     - ``P``\ +1, ``P``\ +2, ``P``\ +3
     - ``EA``\ +1, ``P``\ +1, ``P``\ +2, ``P``\ +3
     - ``P``\ +1, ``P``\ +2, ``P``\ +3
   * - ``BRR``
     - Step
     - ``EA``
     - Step
   * - ``BRU``
     - Step
     - ``EA``
     - Step
   * - ``BRX``
     - Step
     - ``EA``, ``P``\ +1
     - ``P``\ +1
   * - ``EXU``
     - ?
     - ?
     - ?
   * - ``HLT``
     - Step
     - Step
     - Step
   * - ``LDA``, etc
     - Step
     - ``P``\ +1
     - ``P``\ +1
   * - ``SKx``
     - Step
     - ``P``\ +1, ``P``\ +2
     - ``P``\ +1, ``P``\ +2
   * - ``POP``
     - ``P``\ +1, ``P``\ +2
     - 100+\ ``OP``, ``P``\ +1, ``P``\ +2
     - ``P``\ +1, ``P``\ +2
   * - ``SYSPOP``
     - ``P``\ +1, ``P``\ +2
     - 100+\ ``OP``, ``P``\ +1, ``P``\ +2
     - ``P``\ +1, ``P``\ +2

Note that these temporary breakpoints are CPU-mode insensitive,
so there is the potential for conflict if execution in a different CPU mode should encounter an instruction address numerically equal to any of these temporary breakpoints.

.. _Channels:
.. _CHAN:
.. |CHAN| replace:: ``CHAN``

Channels (|CHAN|)
-----------------
The SDS 940 has up to eight I/O channels,
designated W, Y, C, D, E, F, G, and H.
W, Y, C, and D are time-multiplexed communications channels (``TMCC``);
E, F, G, and H are direct access communications channels (``DACC``).
Unlike real SDS 940 channels,
the simulated channels handle 6b, 12b, and 24b transfers simultaneously.
The association between a device and a channel is displayed by the ``SHOW <dev> CHAN`` command::

    SHOW LPT CHAN
    channel=W

The user can change the association with the ``SET <dev> CHAN=<chan>`` command,
where ``<chan>`` is a channel letter::

    SET LPT CHAN=E
    SHOW LPT CHAN
    channel=E

Each channel has nine registers.
The registers are arrays,
with entry [0] for channel W,
entry [1] for channel Y, etc.

=============  ====  =======================================
Name           Size  Comments
=============  ====  =======================================
``UAR[0:7]``   6     Unit address register
``WCR[0:7]``   15    Word count register
``MAR[0:7]``   16    Memory address register
``DCR[0:7]``   6     Data chaining register
``WAR[0:7]``   24    Word assembly register
``CPW[0:7]``   2     Characters per word
``CNT[0:7]``   3     Character count
``MODE[0:7]``  12    Channel mode (from ``EOM`` instruction)
``FLAG[0:7]``  9     Channel flags
=============  ====  =======================================

The user can display all the registers in a channel with the command::

    SHOW CHAN channel-letter

.. _Console input:
.. _TTI:
.. |TTI|  replace:: ``TTI``

Console input (|TTI|)
---------------------
The console input (|TTI|) polls the console keyboard for input.
It implements these registers:

========  ====  ==========================
Name      Size  Comments
========  ====  ==========================
``BUF``   6     Data buffer
``XFR``   1     Transfer ready flag
``POS``   32    Number of characters input
``TIME``  24    Polling interval
========  ====  ==========================

By default,
the console input is assigned to channel W.

.. _Console output:
.. _TTO:
.. |TTO| replace:: ``TTO``

Console output (|TTO|)
----------------------
The console output (|TTO|) writes to the simulator console window.
It implements these registers:

========  ====  =====================================
Name      Size  Comments
========  ====  =====================================
``BUF``   6     Data buffer
``XFR``   1     Transfer ready flag
``POS``   32    Number of characters output
``TIME``  24    Time from I/O initiation to interrupt
========  ====  =====================================

By default,
the console output is assigned to channel W.

.. _Paper tape reader:
.. _PTR:
.. |PTR| replace:: ``PTR``

Paper tape reader (|PTR|)
-------------------------
The paper tape reader (|PTR|) reads data from a disk file.
The ``POS`` register specifies the number of the next data item to be read.
Thus, by changing ``POS``,
the user can backspace or advance the reader.

The paper tape reader implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       6     Data buffer
``XFR``       1     Transfer ready flag
``SOR``       1     Start of record flag
``CHAN``      4     Active channel
``POS``       32    Position in the input file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  =====================================

The paper-tape reader supports the ``BOOT`` command.
``BOOT PTR`` simulates the standard console fill sequence.

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of tape           |
+--------------+--------------+-----------------------+
| End of file  | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of tape           |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

By default,
the paper tape reader is assigned to channel W.

.. _Paper tape punch:
.. _PTP:
.. |PTP| replace:: ``PTP``

Paper tape punch (|PTP|)
------------------------
The paper tape punch (|PTP|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the punch.

The paper tape punch implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       6     Data buffer
``XFR``       1     Transfer ready flag
``LDR``       1     Punch leader flag
``CHAN``      4     Active channel
``POS``       32    Position in the output file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  =====================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of tape           |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

By default,
the paper tape punch is assigned to channel W.

.. _Line printer:
.. _LPT:
.. |LPT| replace:: ``LPT``

Line printer (|LPT|)
--------------------
The line printer (|LPT|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the printer.

The line printer implements these registers:

==============  ====  ===================================
Name            Size  Comments
==============  ====  ===================================
``BUF[0:131]``  8     Data buffer
``BPTR``        8     Buffer pointer
``XFR``         1     Transfer ready flag
``ERR``         1     Error flag
``CHAN``        4     Active channel
``CCT[0:131]``  8     Carriage control tape
``CCTP``        8     Pointer into carriage control tape
``CCTL``        8     Length of carriage control tape
``SPCINST``     24    Spacing instruction
``POS``         32    Position in the output file
``CTIME``       24    Inter-character time
``PTIME``       24    Print time
``STIME``       24    Space time
``STOP_IOE``    1     Stop on I/O error
==============  ====  ===================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of paper          |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

By default,
the line printer is assigned to channel W.

.. _Real-time clock:
.. _RTC:
.. |RTC| replace:: ``RTC``

Real-time clock (|RTC|)
-----------------------
The real-time clock (|RTC|) frequency can be adjusted as follows::

    SET RTC 60HZ                 Set frequency to 60Hz
    SET RTC 50HZ                 Set frequency to 50Hz

The default is 60Hz.

The clock implements these registers:

========  ====  ================
Name      Size  Comments
========  ====  ================
``PIE``   1     Interrupt enable
``TIME``  24    Tick interval
========  ====  ================

The real-time clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

.. _Terminal multiplexer:
.. _MUX:
.. _MUXL:
.. |MUX|  replace:: ``MUX``
.. |MUXL| replace:: ``MUXL``

Terminal multiplexer (|MUX|)
----------------------------
The terminal multiplexer provides 32 asynchronous interfaces.
In Genie mode,
the interfaces are hard-wired;
in SDS mode,
they implement modem control.
The multiplexer has two controllers:
|MUX| for the scanner,
and |MUXL| for the individual lines.
The terminal multiplexer performs input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the port to be used::

    ATTACH MUX <port>            Set up listening port

where ``port`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

Each line (each unit of |MUXL|) supports one option:
``UC``,
when set,
causes lowercase input characters to be automatically converted to uppercase.
In addition,
each line supports output logging.
The ``SET MUXLn LOG`` command enables logging on a line::

    SET MUXLn filename           Log output of line n to filename

The ``SET MUXLn NOLOG`` command disables logging and closes the open log file, if any.

Once |MUX| is attached and the simulator is running,
the multiplexor listens for connections on the specified port.
It assumes that the incoming connections are Telnet connections.
The connections remain open until disconnected,
either by the Telnet client,
a ``SET MUX DISCONNECT`` command,
or a ``DETACH MUX`` command.

Other special multiplexer commands::

    SHOW MUX CONNECTIONS         Show current connections
    SHOW MUX STATISTICS          Show statistics for active connections
    SET MUXLn DISCONNECT         Disconnect the specified line

The controller (|MUX|) implements these registers:

================  ====  ==============================
Name              Size  Comments
================  ====  ==============================
``STA[0:31]``     6     Status, lines 0 to 31
``RBUF[0:31]``    8     Receive buffer, lines 0 to 31
``XBUF[0:31]``    8     Transmit buffer, lines 0 to 31
``FLAGS[0:127]``  1     Line flags, 0 to 3 for line 0,
                        4 to 7 for line 1, etc
``SCAN``          7     Scanner current flag number
``SLCK``          1     Scanner locked flag
``TPS``           8     Character polls per second
================  ====  ==============================

The lines (|MUXL|) implements these registers:

==============  ====  ============================
Name            Size  Comments
==============  ====  ============================
``TIME[0:31]``  24    Transmit time, lines 0 to 31
==============  ====  ============================

The terminal multiplexor does not support save and restore.
All open connections are lost when the simulator shuts down or |MUX| is detached.

.. _Project Genie drum:
.. _DRM:
.. |DRM| replace:: ``DRM``

Project Genie drum (|DRM|)
--------------------------
The Project Genie drum (|DRM|) implements these registers:

============  ====  ==========================
Name          Size  Comments
============  ====  ==========================
``DA``        19    Drum address
``CA``        16    Core address
``WC``        14    Word count
``PAR``       12    Cumulative sector parity
``RW``        1     Read/write flag
``ERR``       1     Error flag
``STA``       2     Drum state
``FTIME``     24    Channel program fetch time
``XTIME``     24    Interword transfer time
``STOP_IOE``  1     Stop on I/O error
============  ====  ==========================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Drum not ready        |
+--------------+--------------+-----------------------+

Drum data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.
Unlike conventional SDS 940 devices,
the Project Genie drum does not use a channel.

.. _Rapid access (fixed-head) disk:
.. _Rapid access disk:
.. _RAD:
.. |RAD| replace:: ``RAD``

Rapid access (fixed-head) disk (|RAD|)
--------------------------------------
The rapid access disk (|RAD|) implements these registers:

============  ====  ====================================
Name          Size  Comments
============  ====  ====================================
``DA``        15    Disk address
``SA``        6     Sector word address
``BP``        1     Sector byte pointer
``XFR``       1     Data transfer flag
``NOBD``      1     Inhibit increment across track
``ERR``       1     Error flag
``CHAN``      4     Active channel
``PROT``      8     Write protect switches
``TIME``      24    Interval between half-word transfers
``STOP_IOE``  1     Stop on I/O error
============  ====  ====================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Disk not ready        |
+--------------+--------------+-----------------------+

The rapid access disk is buffered in memory;
end-of-file and OS I/O errors cannot occur.
If it is assigned to channel W,
bootstrap fill from the device is permitted.
By default,
the rapid access disk is assigned to channel E and bootstrapping is not permitted.

.. _Moving head disk:
.. _DSK:
.. |DSK| replace:: ``DSK``

Moving head disk (|DSK|)
------------------------
|DSK| options include the ability to make the drive write-enabled or write-locked::

    SET RAD LOCKED               Set write-locked
    SET RAD WRITEENABLED         Set write-enabled

The moving head disk implements these registers:

=============  ====  ====================================
Name           Size  Comments
=============  ====  ====================================
``BUF[0:63]``  8     Transfer buffer
``BPTR``       9     Buffer pointer
``BLNT``       9     Buffer length
``DA``         21    Disk address
``INST``       24    Disk instruction
``XFR``        1     Data transfer flag
``ERR``        1     Error flag
``CHAN``       4     Active channel
``WTIME``      24    Interval between character transfers
``STIME``      24    Seek interval
``STOP_IOE``   1     Stop on I/O error
=============  ====  ====================================

Error handling is as follows:

+--------------+--------------+-----------------------------+
| Error        | ``STOP_IOE`` | Processed as                |
+==============+==============+=============================+
| Not attached | 1            | Report error and stop       |
|              +--------------+-----------------------------+
|              | 0            | Disk not ready              |
+--------------+--------------+-----------------------------+
| End of file  | x            | Assume rest of disk is zero |
+--------------+--------------+-----------------------------+
| OS I/O error | x            | Report error and stop       |
+--------------+--------------+-----------------------------+

By default,
the moving head disk is assigned to channel F.

.. _Magnetic tape:
.. _MT:
.. |MT| replace:: ``MT``

Magnetic tape (|MT|)
--------------------
|MT| options include the ability to make units write-enabled or write-locked. ::

    SET MTn LOCKED               Set unit n write-locked
    SET MTn WRITEENABLED         Set unit n write-enabled

Magnetic tape units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET MTn CAPAC=m              Set unit n capacity to m MB (0 = unlimited)
    SHOW MTn CAPAC               Show unit n capacity in MB

Units can also be set ``ENABLED`` or ``DISABLED``.
The magnetic tape controller supports the ``BOOT`` command.
``BOOT MTn`` simulates the standard console fill sequence for unit *n*.

The magnetic tape implements these registers:

=================  ====  ====================================
Name               Size  Comments
=================  ====  ====================================
``BUF[0:131071]``  8     Transfer buffer
``BPTR``           18    Buffer pointer
``BLNT``           18    Buffer length
``XFR``            1     Data transfer flag
``CHAN``           4     Active channel
``INST``           24    Magtape instruction
``EOF``            1     End-of-file flag
``GAP``            1     Inter-record gap flag
``SKIP``           1     Skip data flag
``CTIME``          24    Interval between character transfers
``GTIME``          24    Gap interval
``POS[0:7]``       32    Position, drives 0 to 7
``STOP_IOE``       1     Stop on I/O error
=================  ====  ====================================

Error handling is as follows:

============  =====================================
Error         Processed as
============  =====================================
Not attached  Tape not ready; if ``STOP_IOE``, stop
End of file   End of tape
OS I/O error  End of tape; if ``STOP_IOE``, stop
============  =====================================

By default,
the magnetic tape is assigned to channel W.

Symbolic display and input
==========================
The SDS 940 simulator implements symbolic display and input.
Display is controlled by command-line switches:

======  ==============================================
``-a``  Display as three SDS internal ASCII characters
``-c``  Display as four packed SDS 6b characters
``-m``  Display instruction mnemonics
======  ==============================================

Input parsing is controlled by the first character typed in or by command-line switches:

===============  ==========================================
``'`` or ``-a``  Three packed SDS internal ASCII characters
``"`` or ``-c``  Four packed SDS 6b characters
Alphabetic       Instruction mnemonic
Numeric          Octal number
===============  ==========================================

Instruction input uses (more or less) standard SDS 940 assembler syntax.
There are ten instruction classes:

====================  =======================  ================  =============
Class                 Operands                 Examples          Comments
====================  =======================  ================  =============
No operand            None                     ``EIR``
``POP`` (prog op)     ``op,addr{,tag}``        ``POP 66,100``
I/O                   ``addr{,tag}``           ``EOM 1266``
Mem reference         ``addr{,tag}``           | ``LDA 400,2``  
                                               | ``STA* 300``    Indirect addr
Reg change            ``op`` ``op`` ``op``...  ``CLA CLB``       Opcodes OR
Shift                 ``cnt{,tag}``            ``LSH 10``
Operand ignored       Any                      ``NOP``
Chan command          ``chan``                 ``ALC W``
Chan test             ``chan``                 ``CAT Y``
``SYSPOP`` (prog op)  ``op,addr{,tag}``        | ``BRS 42``
                                               | ``SBRM 500``
                                               | ``CIN* 400,2``
====================  =======================  ================  =============

All numbers are octal.
Channel designators can be alphabetic (W, Y, C, D, E, F, G, H) or numeric (0-7).
Tags must be 0-7,
with 2 indicating indexing.

In addition,
all display and input commands may specify how the address is mapped to main memory via a command-line switch:

======  ====================================================
``-n``  Normal, address is an absolute physical address
``-x``  Monitor, map address through the monitor memory map
``-u``  User, map address through the user memory map
``-v``  Current, map address through the monitor or user map
        depending upon current machine mode
======  ====================================================

Memory mapping defaults to ``-n`` behavior if no switch is specified.
