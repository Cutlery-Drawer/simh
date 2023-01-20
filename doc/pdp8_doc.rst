.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

*********************
PDP-8 simulator usage
*********************
:Date: 2020-05-01
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

This memorandum documents the PDP-8 simulator.

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

``sim/pdp8/``
    | ``pdp8_defs.h``
    | ``pdp8_cpu.c``
    | ``pdp8_ct.c``
    | ``pdp8_df.c``
    | ``pdp8_dt.c``
    | ``pdp8_fpp.c``
    | ``pdp8_lp.c``
    | ``pdp8_mt.c``
    | ``pdp8_pt.c``
    | ``pdp8_rf.c``
    | ``pdp8_rk.c``
    | ``pdp8_rl.c``
    | ``pdp8_rx.c``
    | ``pdp8_sys.c``
    | ``pdp8_td.c``
    | ``pdp8_tsc.c``
    | ``pdp8_tt.c``
    | ``pdp8_ttx.c``

PDP-8 features
==============
The PDP-8 simulator is configured as follows:

===============  ===========================================================
Device names(s)  Simulates
===============  ===========================================================
``CPU``          PDP-8/E CPU with 4KW-32KW of memory
—                KE8E extended arithmetic element (EAE)
—                KM8E memory management and timeshare control
``TSC``          TSC8-75 ETOS operating system timeshare control
``FPP``          FPP8A floating point unit
``PTR,PTP``      PC8E paper tape reader/punch
``TTI,TTO``      KL8E console terminal
``TTIX,TTOX``    KL8JA additional terminals, up to 16
``LPT``          LE8E line printer
``CLK``          DK8E line frequency clock (also PDP-8/A compatible)
``RK``           RK8E/RK05 cartridge disk controller with four drives
``RF``           RF08/RS08 fixed head disk controller with 1-4 platters
``DF``           DF32/DS32 fixed head disk controller with 1-4 platters
``RL``           RL8A/RL01 cartridge disk controller with four drives
``RX``           RX8E/RX01, RX28/RX02 floppy disk controller with two drives
``DT``           TC08/TU56 DECtape controller with eight drives
``TD``           TD8E/TU56 DECtape controller with two drives
``MT``           TM8E/TU10 magnetic tape controller with eight drives
``CT``           TA8E/TU60 cassette tape controller with two drives
===============  ===========================================================

Most devices can be disabled or enabled,
by the commands::

    SET <dev> DISABLED
    SET <dev> ENABLED

The simulator allows most device numbers to be changed,
by the command::

    SET <dev> DEV=<number>

The PDP-8 can support only one of the set {\ ``DF32``, ``RF08``, ``RL8A``\ } using the default device numbers,
since they all use device numbers 60–61.
The default is the RF08.
To change the disk at device numbers 60-61::

    SET RF DISABLED                Disable RF08
    SET DF ENABLED, or             Enable DF32
    SET RL ENABLED                 Enable RL8A

The PDP-8 can only support one of the set {\ ``TC08``, ``TD8E``\ } using the default device numbers,
since both use device number 77.
The default is the TC08.
To change the DECtape controller to the TD8E::

    SET DT DISABLED                Disable TC08
    SET TD ENABLED                 Enable TD8E

The PDP-8 can only support one of the set {\ ``TM8E``, ``TA8E``\ } using the default device numbers,
since both use device number 70.
The default is the TM8E.
To change the device at device number 70::

    SET MT DISABLED                Disable TM8E
    SET CT ENABLED                 Enable TA8E

Alternately,
the device conflict can be eliminated by changing device numbers::

    SET RL DEV=50
    SET RL ENA
    SET TD DEV=74
    SET TD ENA
    SET CT DEV=73
    SET CT ENA

However,
devices can only be ``BOOT``\ ed with their default device numbers.

The PDP-8 simulator implements several unique stop conditions:

* If an undefined instruction (unimplemented IOT or OPR) is decoded, and ``STOP_INST`` is set
* If a simulated DECtape runs off the end of its reel

The ``LOAD`` command supports both RIM format and BIN format tapes.
If the file extension is ``.RIM``,
or the ``r`` switch is specified with ``LOAD``,
the file is assumed to be RIM format;
if the file extension is not ``.RIM``,
or the ``-b`` switch is specified,
the file is assumed to be BIN format.
For BIN format tape loading will stop at the first trailer code (0x80).
If the tape contains multiple sections specify the ``-a`` flag to load all sections on the tape.
The number of sections loaded will be printed.
If the tape had non-BIN format data following the last valid leader it may cause checksum errors or load unwanted data into memory.

CPU
---
The only CPU options are the presence of the EAE and the size of main memory;
the memory extension and time-share control is always included,
even if memory size is 4K. ::

    SET CPU EAE                    Enable EAE
    SET CPU NOEAE                  Disable EAE
    SET CPU 4K                     Set memory size = 4K
    SET CPU 8K                     Set memory size = 8K
    SET CPU 12K                    Set memory size = 12K
    SET CPU 16K                    Set memory size = 16K
    SET CPU 20K                    Set memory size = 20K
    SET CPU 24K                    Set memory size = 24K
    SET CPU 28K                    Set memory size = 28K
    SET CPU 32K                    Set memory size = 32K

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 32K.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

=============  ====  ================================================
Name           Size  Comments
=============  ====  ================================================
``PC``         15    Program counter, including ``IF`` as high 3 bits
``AC``         12    Accumulator
``MQ``         12    Multiplier-quotient
``L``          1     Link
``SR``         12    Front panel switches
``IF``         3     Instruction field
``DF``         3     Data field
``IB``         3     Instruction field buffer
``SF``         7     Save field
``UF``         1     User mode flag
``UB``         1     User mode buffer
``SC``         5     EAE shift counter
``GTF``        1     EAE greater than flag
``EMODE``      1     EAE mode (0 = A, 1 = B)
``ION``        1     Interrupt enable
``ION_DELAY``  1     Interrupt enable delay for ION
``CIF_DELAY``  1     Interrupt enable delay for CIF
``PWR_INT``    1     Power fail interrupt
``UF_INT``     1     User mode violation interrupt
``INT``        15    Interrupt pending flags
``DONE``       15    Device done flags
``ENABLE``     15    Device interrupt enable flags
``PCQ[0:63]``  15    PC prior to last JMP, JMS, or interrupt;
                     most recent PC change first
``STOP_INST``  1     Stop on undefined instruction
``WRU``        8     Interrupt character
=============  ====  ================================================

The CPU attempts to detect when the simulator is idle.
When idle,
the simulator does not use any resources on the host system.
Idle detection is controlled by the ``SET IDLE`` and ``SET NOIDLE`` commands::

    SET CPU IDLE                   Enable idle detection
    SET CPU NOIDLE                 Disable idle detection

Idle detection is disabled by default.
At present,
the CPU is considered idle if it is executing a ``KSF/JMP *-1`` loop with interrupts disabled (OS/8, DMS-8) or a ``JMP *`` loop (TSS/8).

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY                Clear history buffer
    SET CPU HISTORY=0              Disable history
    SET CPU HISTORY=n              Enable history, length = n
    SHOW CPU HISTORY               Print CPU history
    SHOW CPU HISTORY=n             Print first n entries of CPU history

The maximum length for the history is 65536 entries.

.. |TSC| replace:: ``TSC``
.. _TSC8-75 ETOS timeshare control:
.. _TSC:

TSC8-75 ETOS timeshare control (|TSC|)
--------------------------------------
ETOS is a timeshared operating system for the PDP-8,
providing multiple virtual OS/8 environments for up to 32 users.
It requires a special timeshare control option,
the TSC8-75.
The TSC8-75 is normally disabled;
to run ETOS,
it must be enabled with the command::

    SET TSC ENABLED

The TSC8-75 implements these registers:

=======  ====  ============================================
Name     Size  Comments
=======  ====  ============================================
``IR``   12    Most recently trapped instruction
``PC``   12    PC of most recently trapped instruction
``CDF``  1     1 if trapped instruction is CDF, 0 otherwise
``ENB``  1     Interrupt enable flag
``INT``  1     Interrupt pending flag
=======  ====  ============================================

Except for operation of ETOS,
the TSC8-75 should be left disabled.

.. _FPP8A floating point unit:
.. |FPP| replace:: ``FPP``
.. _FPP:

FPP8A floating point unit (|FPP|)
---------------------------------
The floating point unit (|FPP|) is an add-on device that provides floating-point capabilities.
It operates as a coprocessor to the main CPU,
with its own program counter and instruction set.
The FPP8A is normally disabled;
to use it,
it must be enabled with the command::

    SET FPP ENABLED

The FPP8A implements these registers:

====================  ====  ===========================================
Name                  Size  Comments
====================  ====  ===========================================
``FPACE``             12    Floating AC exponent
``FPAC0``\ ..\ ``4``  12    Floating AC fraction words ``0``\ ..\ ``4``
``CMD``               12    FPP command register
``STA``               12    FPP status register
``APTA``              15    APT address pointer
``APTSVF``            3     APT save field
``FPC``               15    FPP program counter
``BRA``               15    Base register pointer
``XRA``               15    Index register pointer
``OPA``               15    Operand pointer
``SSF``               12    Single step flag
``LASTLOCK``          12    Last lock bit
``FLAG``              1     FPP flag
====================  ====  ===========================================

Except for environments that explicitly support it,
the FPP8A should be left disabled.

Programmed I/O devices
----------------------

.. _PC8E Paper Tape Reader:
.. |PTR| replace:: ``PTR``
.. _PTR:

PC8E Paper Tape Reader (|PTR|)
""""""""""""""""""""""""""""""
The paper tape reader (|PTR|) reads data from a disk file.
The ``POS`` register specifies the number of the next data item to be read.
Thus, by changing ``POS``,
the user can backspace or advance the reader.

The paper tape reader supports the ``BOOT`` command.
``BOOT PTR`` copies the RIM loader into memory and starts it running.

The paper tape reader implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       8     Last data item processed
``DONE``      1     Device done flag
``ENABLE``    1     Interrupt enable flag
``INT``       1     Interrupt pending flag
``POS``       32    Position in the input file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  =====================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out-of-tape           |
+--------------+--------------+-----------------------+
| End-of-file  | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out-of-tape           |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

.. _PC8E Paper Tape Punch:
.. |PTP| replace:: ``PTP``
.. _PTP:

PC8E Paper Tape Punch (|PTP|)
"""""""""""""""""""""""""""""
The paper tape punch (|PTP|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the punch.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The paper tape punch implements these registers:

============  ====  ========================================
Name          Size  Comments
============  ====  ========================================
``BUF``       8     Last data item processed
``DONE``      1     Device done flag
``ENABLE``    1     Interrupt enable flag
``INT``       1     Interrupt pending flag
``POS``       32    Position in the output file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  ========================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out-of-tape           |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

.. _KL8E terminal input:
.. |TTI| replace:: ``TTI``
.. _TTI:

KL8E terminal input (|TTI|)
"""""""""""""""""""""""""""
The terminal interfaces (|TTI|, |TTO|) can be set to one of four modes,
``KSR``, ``7B``, ``7B``, or ``8B``:

=======  ======================  ==================================
Mode     Input characters        Output characters
=======  ======================  ==================================
``KSR``  Lowercase converted     Lowercase converted to uppercase,
         to uppercase,           high-order bit cleared,
         high-order bit set      non-printing characters suppressed
``7P``   High-order bit cleared  High-order bit cleared,
                                 non-printing characters suppressed
``7B``   High-order bit cleared  High-order bit cleared
``8B``   No changes              No changes
=======  ======================  ==================================

The default mode is ``KSR``.

The terminal input (|TTI|) polls the console keyboard for input.
It implements these registers:

==========  ====  ============================
Name        Size  Comments
==========  ====  ============================
``BUF``     8     Last data item processed
``DONE``    1     Device done flag
``ENABLE``  1     Interrupt enable flag
``INT``     1     Interrupt pending flag
``POS``     32    Number of characters input
``TIME``    24    Input polling interval
==========  ====  ============================

The terminal input is normally polled synchronously with the real-time clock.
To avoid data loss,
a poll is scheduled 'TIME' instructions after the CPU reads a character.

.. _KL8E Terminal Output:
.. |TTO| replace:: ``TTO``
.. _TTO:

KL8E Terminal Output (|TTO|)
""""""""""""""""""""""""""""
The terminal output (|TTO|) writes to the simulator console window.
It implements these registers:

==========  ====  =====================================
Name        Size  Comments
==========  ====  =====================================
``BUF``     8     Last data item processed
``DONE``    1     Device done flag
``ENABLE``  1     Interrupt enable flag
``INT``     1     Interrupt pending flag
``POS``     32    Number of characters output
``TIME``    24    Time from I/O initiation to interrupt
==========  ====  =====================================

.. _LE8E Line Printer:
.. |LPT| replace:: ``LPT``
.. _LPT:

LE8E Line Printer (|LPT|)
"""""""""""""""""""""""""
The line printer (|LPT|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be read or written.
Thus, by changing ``POS``,
the user can backspace or advance the printer.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The line printer implements these registers:

===========  ====  =====================================
Name         Size  Comments
===========  ====  =====================================
``BUF``      8     Last data item processed
``ERR``      1     Error status flag
``DONE``     1     Device done flag
``ENABLE``   1     Interrupt enable flag
``INT``      1     Interrupt pending flag
``POS``      32    Position in the output file
``TIME``     24    Time from I/O initiation to interrupt
===========  ====  =====================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out-of-paper          |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

.. _DK8E Line-Frequency Clock:
.. |CLK| replace:: ``CLK``
.. _CLK:

DK8E Line-Frequency Clock (|CLK|)
"""""""""""""""""""""""""""""""""
The real-time clock (|CLK|) frequency can be adjusted as follows::

    SET CLK 60HZ                   Set frequency to 60Hz
    SET CLK 50HZ                   Set frequency to 50Hz
    SET CLK NORMAL                 Set clock to calibrated timing
    SET CLK DIAG                   Set clock to fixed timing

The default is 60Hz.

The clock implements these registers:

==========  ====  ==============================
Name        Size  Comments
==========  ====  ==============================
``DONE``    1     Device done flag
``ENABLE``  1     Interrupt enable flag
``INT``     1     Interrupt pending flag
``TIME``    24    Clock interval
==========  ====  ==============================

The real-time clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

.. _KL8JA Additional Terminals:
.. |TTIX| replace:: ``TTIX``
.. |TTOX| replace:: ``TTOX``
.. _TTIX:
.. _TTOX:

KL8JA Additional Terminals (|TTIX|, |TTOX|)
"""""""""""""""""""""""""""""""""""""""""""
The simulator supports 1 to 16 additional terminals,
with an initial default of 4 lines.
The additional terminals consist of two independent devices,
|TTIX| and |TTOX|.
The entire set is modeled as a terminal multiplexer,
with |TTIX| as the master controller.
The number of lines is specified with a ``SET`` command::

    SET TTIX LINES=n               Set number of additional lines to n [1-16]

The ``ATTACH`` command specifies the port to be used::

    ATTACH TTIX <port>             Set up listening port

where ``port`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.
The additional terminals are disabled by default.

The additional terminals can be set to one of four modes:
``UC``, ``7P``, ``7B``, or ``8B``.

======  ======================  =======================================
Mode    Input characters        Output characters
======  ======================  =======================================
``UC``  Lowercase converted     Lowercase converted to uppercase,
        to uppercase,           high-order bit cleared,
        high-order bit cleared  non-printing characters suppressed
``7P``  High-order bit cleared  High-order bit cleared,
                                non-printing characters suppressed
``7B``  High-order bit cleared  High-order bit cleared
``8B``  No changes              No changes
======  ======================  =======================================

The default mode is ``UC``.
Finally,
each line supports output logging.
The ``SET TTOXn LOG`` command enables logging on a line::

    SET TTOXn LOG=filename         Log output of line n to filename

The ``SET TTOXn NOLOG`` command disables logging and closes the open log file,
if any.

Once |TTIX| is attached and the simulator is running,
the terminals listen for connections on the specified port.
They assume that the incoming connections are Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a ``SET TTIX DISCONNECT`` command,
or a ``DETACH TTIX`` command.

Other special commands::

    SHOW TTIX CONNECTIONS  Show current connections
    SHOW TTIX STATISTICS   Show statistics for active connections
    SET TTOXn DISCONNECT   Disconnects the specified line

The input device (|TTIX|) implements these registers:

===============  =========  ==============================
Name             Size       Comments
===============  =========  ==============================
``BUF[0:15]``    8          Input buffer, lines 0 to 15
``DONE``         16         Device done flags (line 0 rightmost)
``ENABLE``       16         Interrupt enable flag
``TIME``         24         Initial polling interval
===============  =========  ==============================

The input device is normally polled synchronously with the real-time clock.
To avoid data loss,
a poll is scheduled 'TIME' instructions after the CPU reads a character from any line.

The output device (|TTOX|) implements these registers:

==============  ====  =================================================
Name            Size  Comments
==============  ====  =================================================
``BUF[0:15]``   8     Last data item processed, lines 0-15
``DONE``        16    Device done flag (line 0 rightmost)
``ENABLE``      16    Interrupt enable flag
``TIME[0:16]``  24    Time from I/O initiation to interrupt, lines 0-15
==============  ====  =================================================

The additional terminals do not support save and restore.
All open connections are lost when the simulator shuts down or |TTIX| is detached.

.. _TD8E/TU56 DECtape:
.. |TD| replace:: ``TD``
.. _TD:

TD8E/TU56 DECtape (|TD|)
""""""""""""""""""""""""
The TD8E is a programmed I/O,
non-interrupt controller,
supporting two DECtape drives (0 and 1).
The TD8E simulator puts a high burden on the host processor,
because tape activity is simulated a line (3b) at a time.
Unless the PDP-8 software requires the TD8E,
the TC08 should be used to simulate DECtapes.
The TD8E is disabled by default.

TD8E options include the ability to make units write-enabled or write-locked. ::

    SET TDn LOCKED                 Set unit n write locked
    SET TDn WRITEENABLED           Set unit n write enabled

Units can also be set ``ENABLED`` or ``DISABLED``.
The TD8E supports the ``BOOT`` command,
but only for unit 0.

The TD8E supports supports PDP-8 format,
PDP-11 format,
and 18b format DECtape images.
``ATTACH`` assumes the image is in PDP-8 format;
the user can force other choices with switches:

.. table::
   :class: switches

   ======  =====================================
   ``-s``  PDP-11 format
   ``-f``  18b format
   ``-a``  Autoselect based on file on file size
   ======  =====================================

The TD8E controller is a data-only simulator;
the timing and mark track,
and block header and trailer,
are not stored.
Thus, read always produces standard values for mark track, header, and trailer words,
and write throws mark track, header, and trailer words into the bit bucket.

The TD8E controller implements these registers:

==============  ====  =================================
Name            Size  Comments
==============  ====  =================================
``TDCMD``       4     Command register
``TDDAT``       12    Data register
``TDMTK``       6     Mark track register
``TDSLF``       1     Single line flag
``TDQLF``       1     Quad line flag
``TDTME``       1     Timing error flag
``TDQL``        2     Quad line counter
``LTIME``       31    Time between lines
``DCTIME``      31    Time to decelerate to a full-stop
``POS[0:1]``    32    Position, in lines, units 0 and 1
``STATT[0:1]``  18    Unit state, units 0 and 1
``STOP_OFFR``   1     Stop on off-reel error
==============  ====  =================================

The ``LTIME`` parameter should not be changed,
or OS/8 may fail to run correctly.
The ``DCTIME`` parameter should always be at least 100 times greater than ``LTIME``.
Acceleration time is 75% of deceleration time.

.. _TA8E/TA60 Cassette Tape:
.. |CT| replace:: ``CT``
.. _CT:

TA8E/TA60 Cassette Tape (|CT|)
""""""""""""""""""""""""""""""
The TA8E is a programmed I/O controller supporting two cassette drives (0 and 1).
The TA8E can be used with the MCPIP program under OS/8,
and with the CAPS-8 operating system.
Cassettes are simulated as magnetic tapes with a fixed capacity (93,000 characters).
The tape format is always SimH standard.
The TA8E is disabled by default.

TA8E options include the ability to make units write-enabled or write-locked. ::

    SET CTn LOCKED                 Set unit n write locked
    SET CTn WRITEENABLED           Set unit n write enabled

Units cannot be set ``ENABLED`` or ``DISABLED``.
The TA8E supports the ``BOOT`` command,
but only for CAPS-8,
and only for unit 0.

The TA8E controller implements these registers:

==============  ====  =========================
Name            Size  Comments
==============  ====  =========================
``CTSRA``       8     Status register A
``CTSRB``       8     Status register B
``CTDB``        8     Data buffer
``CTDF``        1     Data flag
``RDY``         1     Ready flag
``WLE``         1     Write lock error
``WRITE``       1     TA60 write operation flag
``INT``         1     Interrupt request
``BPTR``        17    Buffer pointer
``BLNT``        17    Buffer length
``STIME``       24    Operation start time
``CTIME``       24    Character latency
``STOP_IOE``    1     Stop on I/O errors flag
``POS[0:1]``    32    Position, units 0-1
==============  ====  =========================

Error handling is as follows:

==============  =====================================
Error           Processed as
==============  =====================================
Not attached    Tape not ready; if ``STOP_IOE``, stop
End-of-file     Bad tape
OS I/O error    CRC error; if ``STOP_IOE``, stop
==============  =====================================

Moving head disks
-----------------

.. _RK8E Cartridge Disk:
.. |RK| replace:: ``RK``
.. _RK:

RK8E Cartridge Disk (|RK|)
""""""""""""""""""""""""""
RK8E options include the ability to make units write-enabled or write-locked::

    SET RKn LOCKED                 Set unit n write locked
    SET RKn WRITEENABLED           Set unit n write enabled

Units can also be set ``ENABLED`` or ``DISABLED``.
The RK8E supports the ``BOOT`` command.

The RK8E implements these registers:

============  ====  =======================
Name          Size  Comments
============  ====  =======================
``RKSTA``     12    Status
``RKCMD``     12    Disk command
``RKDA``      12    Disk address
``RKMA``      12    Current memory address
``BUSY``      1     Control busy flag
``INT``       1     Interrupt pending flag
``STIME``     24    Seek time, per cylinder
``RTIME``     24    Rotational delay
``STOP_IOE``  1     Stop on I/O error
============  ====  =======================

Error handling is as follows:

+--------------+--------------+-----------------------------+
| Error        | ``STOP_IOE`` | Processed as                |
+==============+==============+=============================+
| Not attached | 1            | Report error and stop       |
|              +--------------+-----------------------------+
|              | 0            | Disk not ready              |
+--------------+--------------+-----------------------------+
| End-of-file  | x            | Assume rest of disk is zero |
+--------------+--------------+-----------------------------+
| OS I/O error | x            | Report error and stop       |
+--------------+--------------+-----------------------------+

.. _RL8A Cartridge Disk:
.. |RL| replace:: ``RL``
.. _RL:

RL8A Cartridge Disk (|RL|)
""""""""""""""""""""""""""
RL8A options include the ability to make units write-enabled or write-locked::

    SET RLn LOCKED                 Set unit n write locked
    SET RLn WRITEENABLED           Set unit n write enabled

Units can also be set ``ENABLED`` or ``DISABLED``.
The RL8A supports the ``BOOT`` command,
but only for unit 0.

The RL8A implements these registers:

============  ====  =========================
Name          Size  Comments
============  ====  =========================
``RLCSA``     12    Control/status A
``RLCSB``     12    Control/status B
``RLMA``      12    Memory address
``RLWC``      12    Word count
``RLSA``      6     Sector address
``RLER``      12    Error flags
``RLSI``      16    Silo top word
``RLSI1``     16    Silo second word
``RLSI2``     16    Silo third word
``RLSIL``     1     Silo read left/right flag
``INT``       1     Interrupt request
``DONE``      1     Done flag
``ERR``       1     Composite error flag
``STIME``     1     Seek time, per cylinder
``RTIME``     1     Rotational delay
``STOP_IOE``  1     Stop on I/O error
============  ====  =========================

Error handling is as follows:

+--------------+--------------+------------------------------+
| Error        | ``STOP_IOE`` | Processed as                 |
+==============+==============+==============================+
| Not attached | 1            | Report error and stop        |
|              +--------------+------------------------------+
|              | 0            | Disk not ready               |
+--------------+--------------+------------------------------+
| End-of-file  | x            | Assume rest of disk is zero  |
+--------------+--------------+------------------------------+
| OS I/O error | x            | Report error and stop        |
+--------------+--------------+------------------------------+

.. _RX8E/RX01, RX28/RX02 Floppy Disk:
.. |RX| replace:: ``RX``
.. _RX:

RX8E/RX01, RX28/RX02 Floppy Disk (|RX|)
---------------------------------------
The RX can be configured as an RX8E with two RX01 drives,
or an RX28 with two RX02 drives::

    SET RX RX8E                    Set controller to RX8E/RX01
    SET RX RX28                    Set controller to RX28/RX02

The controller is set to the RX8E by default.
The RX28 is not backwards-compatible with the RX8E and will not work with the standard OS/8 V3D floppy disk driver.

RX8E options include the ability to set units write enabled or write locked::

    SET RXn LOCKED                 Set unit n write locked
    SET RXn WRITEENABLED           Set unit n write enabled

RX28 options include,
in addition,
the ability to set the unit density to single density,
double density,
or autosized;
autosizing is the default::

    SET RXn SINGLE                 Set unit n single density
    SET RXn DOUBLE                 Set unit n double density
    SET RXn AUTOSIZE               Set unit n autosize

The RX8E and RX28 support the ``BOOT`` command.

The RX8E and RX28 implement these registers:

===============  ==========  ===================
Name             Size        Comments
===============  ==========  ===================
``RXCS``         12          Status
``RXDB``         12          Data buffer
``RXES``         12          Error status
``RXTA``         8           Current track
``RXSA``         8           Current sector
``STAPTR``       4           Controller state
``BUFPTR``       8           Buffer pointer
``INT``          1           Interrupt pending flag
``DONE``         1           Device done flag
``ENABLE``       1           Interrupt enable flag
``TR``           1           Transfer ready flag
``ERR``          1           Error flag
``CTIME``        24          Command completion time
``STIME``        24          Seek time, per track
``XTIME``        24          Transfer ready delay
``STOP_IOE``     1           Stop on I/O error
``SBUF[0:255]``  8           Sector buffer array
===============  ==========  ===================

Error handling is as follows:

+--------------+--------------+------------------------------+
| Error        | ``STOP_IOE`` | Processed as                 |
+==============+==============+==============================+
| Not attached | 1            | Report error and stop        |
|              +--------------+------------------------------+
|              | 0            | Disk not ready               |
+--------------+--------------+------------------------------+

RX01 and RX02 data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

Fixed head disks
----------------
With default device addressing,
either the RF08 or the DF32 can be present in a configuration,
but not both.

.. _RF08/RS08 Fixed Head Disk:
.. |RF| replace:: ``RF``
.. _RF:

RF08/RS08 Fixed Head Disk (|RF|)
""""""""""""""""""""""""""""""""
RF08 options include the ability to set the number of platters to a fixed value between 1 and 4,
or to autosize the number of platters::

    SET RF 1P                      One platter (256K)
    SET RF 2P                      Two platters (512K)
    SET RF 3P                      Three platters (768K)
    SET RF 4P                      Four platters (1024K)
    SET RF AUTOSIZE                Autosized on ATTACH

The default is one platter.

The RF08 implements these registers:

============  ====  ============================
Name          Size  Comments
============  ====  ============================
``STA``       12    Status
``DA``        20    Current disk address
``MA``        12    Memory address (in memory)
``WC``        12    Word count (in memory)
``WLK``       32    Write lock switches
``INT``       1     Interrupt pending flag
``DONE``      1     Device done flag
``TIME``      24    Rotational delay, per word
``BURST``     1     Burst flag
``STOP_IOE``  1     Stop on I/O error
============  ====  ============================

The RF08 supports the ``BOOT`` command.
The default bootstrap is for OS/8.
To bootstrap the 4K Disk Monitor,
use the ``BOOT -D RF`` command.

The RF08 is a three-cycle data break device.
If ``BURST = 0``,
word transfers are scheduled individually;
if ``BURST = 1``,
the entire transfer occurs in a single data break.

Error handling is as follows:

+--------------+--------------+------------------------------+
| Error        | ``STOP_IOE`` | Processed as                 |
+==============+==============+==============================+
| Not attached | 1            | Report error and stop        |
|              +--------------+------------------------------+
|              | 0            | Disk not ready               |
+--------------+--------------+------------------------------+

RF08 data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

.. _DF32/DS32 Fixed Head Disk:
.. |DF| replace:: ``DF``
.. _DF:

DF32/DS32 Fixed Head Disk (|DF|)
""""""""""""""""""""""""""""""""
DF32 options include the ability to set the number of platters to a fixed value between 1 and 4,
or to autosize the number of platters::

    SET DF 1P                      One platter (32K)
    SET DF 2P                      Two platters (64K)
    SET DF 3P                      Three platters (98K)
    SET DF 4P                      Four platters (128K)
    SET DF AUTOSIZE                Autosized on ATTACH

The default is one platter.

The DF32 implements these registers:

============  ======  =========================================
Name          Size    Comments
============  ======  =========================================
``STA``       12      Status, disk and memory address extension
``DA``        12      Low-order disk address
``MA``        12      Memory address (in memory)
``WC``        12      Word count (in memory)
``WLK``       16      Write lock switches
``INT``       1       Interrupt pending flag
``DONE``      1       Device done flag
``TIME``      24      Rotational delay, per word
``BURST``     1       Burst flag
``STOP_IOE``  1       Stop on I/O error
============  ======  =========================================

The DF32 supports the ``BOOT`` command.
The default bootstrap is for OS/8.
To bootstrap the 4K Disk Monitor,
use the ``BOOT -D DF`` command.

The DF32 is a three-cycle data break device.
If ``BURST = 0``,
word transfers are scheduled individually;
if ``BURST = 1``,
the entire transfer occurs in a single data break.

Error handling is as follows:

+--------------+--------------+------------------------------+
| Error        | ``STOP_IOE`` | Processed as                 |
+==============+==============+==============================+
| Not attached | 1            | Report error and stop        |
|              +--------------+------------------------------+
|              | 0            | Disk not ready               |
+--------------+--------------+------------------------------+

DF32 data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

.. _TC08/TU56 DECtape:
.. |DT| replace:: ``DT``
.. _DT:

TC08/TU56 DECtape (|DT|)
------------------------
DT implements the TC08 DECtape controller and TU56 drives.
TC08 options include the ability to make units write enabled or write locked. ::

    SET DTn LOCKED                 Set unit n write locked
    SET DTn WRITEENABLED           Set unit n write enabled

Units can also be set ``ENABLED`` or ``DISABLED``.
The TC08 supports the ``BOOT`` command,
but only for unit 0.

The TC08 supports supports PDP-8 format,
PDP-11 format,
and 18b format DECtape images.
``ATTACH`` assumes the image is in PDP-8 format;
the user can force other choices with switches:

.. table::
   :class: switches

   ======  =====================================
   ``-s``  PDP-11 format
   ``-f``  18b format
   ``-a``  Autoselect based on file on file size
   ======  =====================================

The TC08 controller is a data-only simulator;
the timing and mark track,
and block header and trailer,
are not stored.
Thus, the ``WRITE TIMING AND MARK TRACK`` function is not supported;
the ``READ ALL`` function always returns the hardware standard block header and trailer;
and the ``WRITE ALL`` function dumps non-data words into the bit bucket.

The DECtape controller implements these registers:

==============  ====  =============================================
Name            Size  Comments
==============  ====  =============================================
``DTSA``        12    Status register A
``DTSB``        12    Status register B
``INT``         1     Interrupt pending flag
``ENB``         1     Interrupt enable flag
``DTF``         1     DECtape flag
``ERF``         1     Error flag
``CA``          12    Current address (memory location 7754)
``WC``          12    Word count (memory location 7755)
``LTIME``       31    Time between lines
``DCTIME``      31    Time to decelerate to a full stop
``SUBSTATE``    2     Read/write command substate
``POS[0:7]``    32    Position, in lines, units 0 to 7
``STATT[0:7]``  31    Unit state, units 0 to 7
``STOP_OFFR``   1     Stop on off-reel error
==============  ====  =============================================

It is critically important to maintain certain timing relationships among the DECtape parameters,
or the DECtape simulator will fail to operate correctly.

* ``LTIME`` must be at least 6
* ``DCTIME`` needs to be at least 100 times ``LTIME``

Acceleration time is set to 75% of deceleration time.

.. _TM8E Magnetic Tape:
.. |MT| replace:: ``MT``
.. _MT:

TM8E Magnetic Tape (|MT|)
-------------------------
Magnetic tape options include the ability to make units write-enabled or write-locked. ::

    SET MTn LOCKED                 Set unit n write locked
    SET MTn WRITEENABLED           Set unit n write enabled

Magnetic tape units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET MTn CAPAC=m                Set unit n capacity to m MB (0 = unlimited)
    SHOW MTn CAPAC                 Show unit n capacity in MB

Units can also be set ``ENABLED`` or ``DISABLED``.

The magnetic tape controller implements these registers:

============  ======  ===============================
Name          Size    Comments
============  ======  ===============================
``CMD``       12      Command
``FNC``       12      Function
``CA``        12      Memory address
``WC``        12      Word count
``DB``        12      Data buffer
``STA``       12      Main status
``STA2``      6       Secondary status
``DONE``      1       Device done flag
``INT``       1       Interrupt pending flag
``STOP_IOE``  1       Stop on I/O error
``TIME``      24      Record delay
``UST[0:7]``  24      Unit status, units 0 to 7
``POS[0:7]``  32      Position, units 0 to 7
============  ======  ===============================

Error handling is as follows:

==============  =====================================
Error           Processed as
==============  =====================================
Not attached    Tape not ready; if ``STOP_IOE``, stop
End-of-file     Bad tape
OS I/O error    Parity error; if ``STOP_IOE``, stop
==============  =====================================

Symbolic display and input
==========================
The PDP-8 simulator implements symbolic display and input.
Display is controlled by command-line switches:

.. table::
   :class: switches

   ======  =============================================
   ``-a``  Display as ASCII character
   ``-c``  Display as two packed sixbit characters
   ``-t``  Display as two packed TSS/8 sixbit characters
   ``-m``  Display instruction mnemonics
   ======  =============================================

Input parsing is controlled by the first character typed in or by command-line switches:

.. table::
   :class: switches

   =================  ==================================
   ``'`` or ``-a``    ASCII character
   ``"`` or ``-c``    Two packed sixbit characters
   ``#`` or ``-t``    Two packed TSS/8 sixbit characters
   Alphabetic         Instruction mnemonic
   Numeric            Octal number
   =================  ==================================

Instruction input uses standard PDP-8 assembler syntax.
There are four instruction classes:
memory reference, IOT, field change, and operate.

Memory reference instructions have the format ::

    memref {I} {C/Z} address

where ``I`` signifies indirect,
``C`` a current page reference,
and ``Z`` a zero page reference.
The address is an octal number in the range 0 – 07777;
if ``C`` or ``Z`` is specified,
the address is a page offset in the range 0 – 177.
Normally,
``C`` is not needed;
the simulator figures out from the address what mode to use.
However,
when referencing memory outside the CPU
(e.g., disks),
there is no valid PC,
and ``C`` must be used to specify current page addressing.

IOT instructions consist of single mnemonics,
e.g., ``KRB``, ``TLS``.
IOT instructions may be OR'd together ::

    iot iot iot...

The simulator does not check the legality of the proposed combination.
IOTs for which there is no opcode may be specified as ``IOT n``,
where ``n`` is an octal number in the range 0 – 0777.

Field change instructions
(``CIF``, ``CDF``) have the format ::

    fldchg field

where field is an octal number in the range 0 – 7.
Field change instructions may be OR'd together.

Operate instructions have the format ::

    opr opr opr...

The simulator does not check the legality of the proposed combination.
EAE mode A and B mnemonics may be specified regardless of the EAE mode.
The operands for MUY and DVI must be deposited explicitly.
