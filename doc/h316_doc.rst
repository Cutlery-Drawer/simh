.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. |₈| replace:: :sub:`8`

********************
H316 simulator usage
********************
:Date: 2020-04-30
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

This memorandum documents the Honeywell H316/H516 simulator.

Simulator files
===============
The H316 requires the following files:

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

``sim/h316/``
    | ``h316_defs.h``
    | ``h316_cpu.c``
    | ``h316_dp.c``
    | ``h316_fhd.c``
    | ``h316_hi.c``
    | ``h316_imp.c``
    | ``h316_lp.c``
    | ``h316_mi.c``
    | ``h316_mt.c``
    | ``h316_rtc.c``
    | ``h316_stddev.c``
    | ``h316_sys.c``
    | ``h316_udp.c``

H316/H516 features
==================
The Honeywell 316/516 simulator is configured as follows:

==============  ======================================================
Device name(s)  Simulates
==============  ======================================================
``CPU``         H316/H516 CPU with 16/32KW memory
``PTR``         316/516-50 paper tape reader
``PTP``         316/516-52 paper tape punch
``LPT``         316/516 line printer
``TTY``         316/516-33 console terminal
``MT``          4100 seven track magnetic tape with four drives
``CLK``         316/516-12 real time clock
``FHD``         4400 fixed head disk
``DP``          4623/4653/4720 disk pack controller with eight drives
``WDT``         4400 fixed head disk
``RTC``         4400 fixed head disk
``IMP``         IM/TIP Specific Hardware
``MI1``         IM/TIP Modem Interface
``MI2``         IM/TIP Modem Interface
``MI3``         IM/TIP Modem Interface
``MI4``         IM/TIP Modem Interface
``MI5``         IM/TIP Modem Interface
``HI1``         IMP Host Interface
``HI2``         IMP Host Interface
``HI3``         IMP Host Interface
``HI4``         IMP Host Interface
==============  ======================================================

The H316/H516 simulator implements several unique stop conditions:

- Decode of an undefined instruction, and ``STOP_INST`` is set
- Reference to an undefined I/O device, and ``STOP_DEV`` is set
- More than ``INDMAX`` indirect references are detected during memory reference address decoding
- DMA/DMC direction does not agree with I/O device operation
- A write operation is initiated on a write locked magnetic tape unit
  (hangs the real system)
- A disk write overruns the specified record size
  (destroys the rest of the track on the real system)
- A disk track has an illegal format

The ``LOAD`` and ``DUMP`` commands are not implemented.

.. |CPU| replace:: ``CPU``
.. _CPU:

|CPU|
-----
|CPU| options include choice of instruction set,
memory size,
DMC option,
and number of DMA channels. ::

    SET CPU HSA                    High speed arithmetic instructions
    SET CPU NOHSA                  No high speed arithmetic instructions
    SET CPU 4K                     Set memory size = 4K
    SET CPU 8K                     Set memory size = 8K
    SET CPU 12K                    Set memory size = 12K
    SET CPU 16K                    Set memory size = 16K
    SET CPU 24K                    Set memory size = 24K
    SET CPU 32K                    Set memory size = 32K
    SET CPU DMC                    Enable DMC option
    SET CPU NODMC                  Disable DMC option
    SET CPU DMA=n                  Set number of DMA channels to n (0-4)

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 32K.
By default,
the ``HSA`` and DMC options are enabled,
and four DMA channels are configured.

Extended interrupts
"""""""""""""""""""
The H316 came with one interrupt vector and 16 individually maskable interrupt sources as standard,
but could optionally be extended to support up to 48 additional individually maskable interrupt sources,
each with its own unique vector.
Extended interrupts are enabled with the command ::

    SET CPU EXTINT=16

and the command ::

    SET CPU EXTINT=0

restores the original default H316 single interrupt behavior.
Note that the IMP and TIP custom hardware required one additional bank of 16 interrupts.

Only 16 extended interrupts (out of a possible 48) are currently implemented.

DMA channels
""""""""""""""""
The |CPU| includes special ``show`` commands to display the state of the DMA channels::

    SHOW CPU DMAn                  Show DMA channel n

Break on write
""""""""""""""
The H316 emulation supports "break on memory write" breakpoints in addition to the standard "break on execution" type.
For example, the command ::

    BREAK –W 2000

will cause a break to occur any time memory location ``2000``\ |₈| is written.
All the usual SIMH breakpoint options,
including address ranges and commands to be executed automatically upon breaking,
work here too.
Break on write has two restrictions –
first, setting a write break on a DMC channel pointer location,
``20``\ |₈| thru ``57``\ |₈|,
will break only on explicit programmed writes to that address.
Implicit DMC operations will not cause a break.
Second,
remember that when any break occurs,
SIMH prints the PC and the instruction after the one which actually caused the break.
It will always be the previous instruction which actually modified the breakpoint location.

CPU state
"""""""""
CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

=====================  ====  ================================================
Name                   Size  Comments
=====================  ====  ================================================
``P``                  15    Program counter
``A``                  16    A register
``B``                  16    B register
``X``                  16    Index register
``SC``                 16    Shift count
``C``                  1     Carry flag
``EXT``                1     Extend flag
``PME``                1     Previous mode extend flag
``EXT_OFF``            1     Extend off pending flag
``DP``                 1     Double precision flag
``SS1``\ ..\ ``SS14``  1     Sense switches 1 to 4
``ION``                1     Interrupts enabled
``INODEF``             1     Interrupts not deferred
``INTREQ``             16    Interrupt requests
``EXTINT``             16    Extended interrupt requests
``EXTENB``             16    Extended interrupt enables
``DEVRDY``             16    Device ready flags (read-only)
``DEVENB``             16    Device interrupt enable flags (read-only)
``CHREQ``              20    DMA/DMC channel requests
``DMAAD[0:3]``         16    DMA channel current address, channels 1 to 4
``DMAWC[0:3]``         15    DMA channel word count, channels 1 to 4
``DMAEOR[0:3]``        1     DMA end of range flag, channels 1 to 4
``STOP_INST``          1     Stop on undefined instruction
``STOP_DEV``           1     Stop on undefined device
``INDMAX``             8     Indirect address limit
``PCQ[0:63]``          15    PC prior to last ``JMP``, ``JSB``, or interrupt;
                             most recent PC change first
``WRU``                8     Interrupt character
=====================  ====  ================================================

The |CPU| can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY                Clear history buffer
    SET CPU HISTORY=0              Disable history
    SET CPU HISTORY=n              Enable history, length = n
    SHOW CPU HISTORY               Print CPU history
    SHOW CPU HISTORY=n             Print first 'n' entries of CPU history

The maximum length for the history is 65,536 entries.

Programmed I/O devices
----------------------

.. |PTR| replace:: ``PTR``
.. _PTR:

316/516-50 paper tape reader (|PTR|)
""""""""""""""""""""""""""""""""""""
The paper tape reader (|PTR|) reads data from a disk file.
The ``POS`` register specifies the number of the next data item to be read.
Thus, by changing ``POS``,
the user can backspace or advance the reader.

The paper tape reader can bet set to operate in binary, ASCII, or Unix ASCII mode::

    SET PTR BINARY                 Binary mode
    SET PTR ASCII                  ASCII mode
    SET PTR UASCII                 Unix ASCII mode

The mode can also be set by a switch setting in the ``ATTACH`` command::

    ATT –B PTR <file>              Binary mode
    ATT –A PTR <file>              ASCII mode
    ATT –U PTR <file>              Unix ASCII mode

In ASCII or Unix ASCII mode,
all non-zero characters have the high-order bit forced on.
In Unix ASCII mode,
newline is converted to ``CR``,
and ``LF`` is inserted as the following character.

The paper tape reader supports the ``BOOT`` command.
``BOOT PTR`` copies the absolute binary loader into memory and starts it running.

The paper tape reader implements these registers:

============  ======  =====================================
Name          Size    Comments
============  ======  =====================================
``BUF``       8       Last data item processed
``INTREQ``    1       Device interrupt request
``READY``     1       Device ready
``ENABLE``    1       Device interrupts enabled
``POS``       32      Position in the input file
``TIME``      24      Time from I/O initiation to interrupt
``STOP_IOE``  1       Stop on I/O error
============  ======  =====================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of tape           |
+--------------+--------------+-----------------------+
| end of file  | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of tape           |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

.. |PTP| replace:: ``PTP``
.. _PTP:

316/516-52 paper tape punch (|PTP|)
"""""""""""""""""""""""""""""""""""
The paper tape punch (|PTP|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the punch.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The paper tape punch can bet set to operate in binary, ASCII, or Unix ASCII mode::

    SET PTP BINARY                 Binary mode
    SET PTP ASCII                  ASCII mode
    SET PTP UASCII                 Unix ASCII mode

The mode can also be set by a switch setting in the ``ATTACH`` command::

    ATT –B PTP <file>              Binary mode
    ATT –A PTP <file>              ASCII mode
    ATT –U PTP <file>              Unix ASCII mode

In ASCII or Unix ASCII mode,
all characters are masked to ``0x7B`` before being written to the output file.
In Unix ASCII mode,
``LF`` is converted to newline,
and ``CR`` is discarded.

The paper tape punch implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       8     Last data item processed
``INTREQ``    1     Device interrupt request
``READY``     1     Device ready
``ENABLE``    1     Device interrupts enabled
``POWER``     1     Device powered up
``POS``       32    Position in the output file
``TIME``      24    Time from I/O initiation to interrupt
``PWRTIME``   24    Time from I/O request to power up
``STOP_IOE``  1     Stop on I/O error
============  ====  =====================================

Error handling is as follows:

+---------------+-----------------+-----------------------+
| Error         | ``STOP_IOE``    | Processed as          |
+===============+=================+=======================+
| Not attached  | 1               | Report error and stop |
|               +-----------------+-----------------------+
|               | 0               | Out of tape           |
+---------------+-----------------+-----------------------+
| OS I/O error  | x               | Report error and stop |
+---------------+-----------------+-----------------------+

.. |TTY|  replace:: ``TTY``
.. |TTY0| replace:: ``TTY0``
.. |TTY1| replace:: ``TTY1``
.. |TTY2| replace:: ``TTY2``
.. |TTY3| replace:: ``TTY3``
.. _TTY:
.. _TTY0:
.. _TTY1:
.. _TTY2:
.. _TTY3:

316/516-33 console teletype (|TTY|)
"""""""""""""""""""""""""""""""""""
The console Teletype (|TTY|) consists of four separate units:

======   ==================
|TTY0|   Keyboard
|TTY1|   Printer
|TTY2|   Paper tape reader
|TTY3|   Paper tape punch
======   ==================

The keyboard and printer (|TTY0|, |TTY1|) can be set to one of four modes,
``KSR``, ``7P``, ``7B``, or ``8B``:

========  ======================  ===================================
Mode      Input characters        Output characters
========  ======================  ===================================
``KSR``   Lowercase converted     Lowercase converted to uppercase,
          to uppercase,           high-order bit cleared,
          high-order bit set      non-printing characters suppressed
``7P``    High-order bit cleared  High-order bit cleared,
                                  non-printing characters suppressed
``7B``    High-order bit cleared  High-order bit cleared
``8B``    No changes              No changes
========  ======================  ===================================

The default mode is ``KSR``.
The Teletype keyboard reads from the console keyboard,
and the printer writes to the simulator console window.

The paper tape reader (|TTY2|) can be set to operate in binary, ASCII, or Unix ASCII mode::

    SET TTY2 BINARY                Binary mode
    SET TTY2 ASCII                 ASCII mode
    SET TTY2 UASCII                Unix ASCII mode

The mode can also be set by a switch setting in the ``ATTACH`` command::

    ATT –B TTY2 <file>             Binary mode
    ATT –A TTY2 <file>             ASCII mode
    ATT –U TTY2 <file>             Unix ASCII mode

In ASCII or Unix ASCII mode,
all non-zero characters have the high-order bit forced on.
In Unix ASCII mode,
newline is converted to ``CR``,
and ``LF`` is inserted as the following character.

The paper tape reader is started by program output of ``XON`` or by the command ``SET TTY2 START``.
The paper tape reader is stopped by reader input of ``XOFF`` or by the command ``SET TTY2 STOP``.

The Teletype paper tape punch (|TTY3|) can be set to operate in binary, ASCII, or Unix ASCII mode::

    SET TTY2 BINARY                Binary mode
    SET TTY2 ASCII                 ASCII mode
    SET TTY2 UASCII                Unix ASCII mode

The mode can also be set by a switch setting in the ``ATTACH`` command::

    ATT –B TTY3 <file>             Binary mode
    ATT –A TTY3 <file>             ASCII mode
    ATT –U TTY3 <file>             Unix ASCII mode

In ASCII or Unix ASCII mode,
all characters are masked to ``0x7B`` before being written to the output file.
In Unix ASCII mode,
``LF`` is converted to newline,
and ``CR`` is discarded.

The Teletype paper tape punch is started by program output of ``TAPE`` or by the command ``SET TTY3 START``.
The punch is stopped by program output of ``XOFF`` or by the command ``SET TTY3 STOP``.

The |TTY| implements these registers:

==========  ====  ==========================================
Name        Size  Comments
==========  ====  ==========================================
``BUF``     8     Last data item processed
``IN2ND``   9     Holding buffer, input busy wait; the
                  high-order bit indicates character present
``MODE``    1     Read/write mode
``READY``   1     Device ready flag
``BUSY``    1     Device busy flag
``INT``     1     Device interrupt request
``ENABLE``  1     Device interrupt enabled
``KPOS``    32    Number of keyboard characters input
``KTIME``   24    Keyboard polling interval
``KBTIME``  24    Keyboard busy wait after receive
``TPOS``    32    Number of printer characters output
``TTIME``   24    Time from I/O initiation to interrupt
``RPOS``    32    Current reader character position
``PPOS``    32    Current punch character position
==========  ====  ==========================================

.. |CLK| replace:: ``CLK``
.. _CLK:

316/516-12 real-time clock (|CLK|)
""""""""""""""""""""""""""""""""""
The real-time clock (|CLK|) frequency can be adjusted as follows::

    SET CLK 60HZ                   Set frequency to 60Hz
    SET CLK 50HZ                   Set frequency to 50Hz

The default is 60Hz.

The clock implements these registers:

==========  =====  =========================
Name        Size   Comments
==========  =====  =========================
``INTREQ``  1      Device interrupt request
``READY``   1      Device ready
``ENABLE``  1      Device interrupts enabled
``TIME``    24     Clock interval
==========  =====  =========================

The real-time clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

.. Note::
   Previous releases of SIMH did not allow the |CLK| device to be disabled.
   However, this device was optional and it was possible to configure an H316 system without one
   (although this apparently rarely happened).

   Current SIMH releases will allow the |CLK| device to be disabled.
   When the |CLK| device is disabled,
   it does not respond to the clock-specific IO instructions;
   it does not increment location ``61``\ |₈|,
   and it does not generate interrupts.
   The ``SMK`` and ``OTK`` instructions are unaffected.

.. |LPT| replace:: ``LPT``
.. _LPT:

316/516 line printer (|LPT|)
----------------------------
The line printer (|LPT|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the printer.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The line printer can be connected to the IO bus,
a DMC channel,
or a DMA channel::

    SET LPT IOBUS                  Connect to IO bus
    SET LPT DMC=n                  Connect to DMC channel n (1-16)
    SET LPT DMA=n                  Connect to DMA channel n (1 to 4)

By default, the line printer is connected to the IO bus.

The line printer implements these registers:

============  ====  ================================
Name          Size  Comments
============  ====  ================================
``WDPOS``     6     Word position in current scan
``DRPOS``     6     Drum position
``CRPOS``     1     Carriage position
``PRDN``      1     Print done flag
``RDY``       1     Ready flag
``EOR``       1     (DMA/DMC) End of range flag
``DMA``       1     Transfer using DMA/DMC
``INTREQ``    1     Device interrupt request
``ENABLE``    1     Device interrupt enable
``SVCST``     2     Service state
``SVCCH``     2     Service channel
``BUF``       8     Buffer
``POS``       32    Position in the output file
``XTIME``     24    Delay between transfers
``ETIME``     24    Delay at end of scan
``PTIME``     24    Delay for shuttle/line advance
``STOP_IOE``  1     Stop on I/O error
============  ====  ================================

Error handling is as follows:

+---------------+-----------------+-----------------------+
| Error         | ``STOP_IOE``    | Processed as          |
+===============+=================+=======================+
| Not attached  | 1               | Report error and stop |
|               +-----------------+-----------------------+
|               | 0               | Out of paper          |
+---------------+-----------------+-----------------------+
| OS I/O error  | x               | Report error and stop |
+---------------+-----------------+-----------------------+

.. |FHD| replace:: ``FHD``
.. _FHD:

4400 fixed head disk (|FHD|)
----------------------------
Fixed head disk options include the ability to set the number of surfaces to a fixed value between 1 and 16,
or to autosize the number of surfaces from the attached file::

    SET FHD 1S                     One surface (98K)
    SET FHD 2S                     Two platters (196K)
    ⋮
    SET FHD 16S                    Sixteen surfaces (1568K)
    SET FHD AUTOSIZE               Autosized on ATTACH

The default is one surface.

The fixed head disk can be connected to the IO bus,
a DMC channel,
or a DMA channel::

    SET FHD IOBUS                  Connect to IO bus
    SET FHD DMC=n                  Connect to DMC channel n (1-16)
    SET FHD DMA=n                  Connect to DMA channel n (1 to 4)

By default, the fixed head disk is connected to the IO bus.

The fixed head disk implements these registers:

============  ====  ===========================================
Name          Size  Comments
============  ====  ===========================================
``CW1``       16    Control word 1 (read write, surface, track)
``CW2``       16    Control word 2 (character address)
``BUF``       16    Data buffer
``BUSY``      1     Controller busy flag
``RDY``       1     Transfer ready flag
``DTE``       1     Data transfer error flag
``ACE``       1     Access error flag
``EOR``       1     (DMA/DMC) End of range
``DMA``       1     Transfer using DMA/DMC
``CSUM``      1     Transfer parity checksum
``INTREQ``    1     Device interrupt request
``ENABLE``    1     Device interrupt enable
``TIME``      24    Delay between words
``STOP_IOE``  1     Stop on I/O error
============  ====  ===========================================

Error handling is as follows:

+---------------+-----------------+-----------------------+
| Error         | ``STOP_IOE``    | Processed as          |
+===============+=================+=======================+
| Not attached  | 1               | Report error and stop |
|               +-----------------+-----------------------+
|               | 0               | Disk not ready        |
+---------------+-----------------+-----------------------+

Fixed head disk data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

.. |MT| replace:: ``MT``
.. _MT:

4100 7-track magnetic tape (|MT|)
---------------------------------
Magnetic tape options include the ability to make units write enabled or write locked. ::

    SET MTn LOCKED                 Set unit 'n' write locked
    SET MTn WRITEENABLED           Set unit 'n' write enabled

Magnetic tape units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET MTn CAPAC=m                Set unit 'n' capacity to 'm' MB (0 = unlimited)
    SHOW MTn CAPAC                 Show unit 'n' capacity in MB

Units can also be set ``ENABLED`` or ``DISABLED``.

The magnetic tape controller can be connected to the IO bus,
a DMC channel,
or a DMA channel::

    SET MT IOBUS                   Connect to IO bus
    SET MT DMC=n                   Connect to DMC channel 'n' (1-16)
    SET MT DMA=n                   Connect to DMA channel 'n' (1 to 4)

By default, the magnetic tape controller is connected to the IO bus.

The magnetic tape controller implements these registers:

=================  ====  =============================
Name               Size  Comments
=================  ====  =============================
``BUF``            16    Data buffer
``USEL``           2     Unit select
``BUSY``           1     Controller busy flag
``RDY``            1     Transfer ready flag
``ERR``            1     Error flag
``EOF``            1     End of file flag
``EOR``            1     (DMA/DMC) End of range
``DMA``            1     Transfer using DMA/DMC
``MDIRQ``          1     Motion done interrupt request
``INTREQ``         1     Device interrupt request
``ENABLE``         1     Device interrupt enable
``DBUF[0:65535]``  8     Transfer buffer
``BPTR``           17    Transfer buffer pointer
``BMAX``           17    Transfer size (reads)
``CTIME``          24    Start/stop time
``XTIME``          24    Delay between words
``POS[0:3]``       32    Position, units 0 to 3
``STOP_IOE``       1     Stop on I/O error
=================  ====  =============================

Error handling is as follows:

=============  =====================================
Error          Processed as
=============  =====================================
Not attached   Tape not ready; if ``STOP_IOE``, stop
End-of-file    Bad tape
OS I/O error   Parity error; if ``STOP_IOE``, stop
=============  =====================================

.. |DP| replace:: ``DP``
.. _DP:

4623/4651/4720 disk packs (|DP|)
--------------------------------
The disk controller can be configured as a 4623,
supporting 10-surface disk packs;
a 4651,
supporting 2-surface disk packs;
or a 4720,
supporting 20-surface disk packs::

    SET DP 4623                    Controller is 4623
    SET DP 4651                    Controller is 4651
    SET DP 4720                    Controller is 4720

The default is 4651.
All disk packs on the controller must be of the same type.

Individual units can be write-enabled or write-locked::

    SET DPn LOCKED                 Set unit n write locked
    SET DPn WRITEENABLED           Set unit n write enabled

Units can be also be set ``ENABLED`` or ``DISABLED``.

The disk pack controller can be connected to a DMC channel or a DMA channel;
it cannot be connected to the IO bus::

    SET DP DMC=n                   Connect to DMC channel n (1-16)
    SET DP DMA=n                   Connect to DMA channel n (1 to 4)

The disk pack controller supports variable track formatting.
Each track can contain between 1 and 103 records,
with a minimum size of 1 word and a maximum size of 1893 words.
Record addresses are unconstrained.
The simulator provides a command to perform a simple,
fixed record size format of a new disk::

    SET DPn FORMAT=k               Format unit n with k words per record
    SET -R DPn FORMAT=k            Format unit n with k records per track

Record addresses can either be geometric (cylinder/track/sector) or simple sequential starting from 0::

    SET DPn FORMAT=k               Format with geometric record addresses
    SET -S DPn FORMAT=k            Format with sequential record addresses

Geometric address have the cylinder number in ``bits<1:8>``,
the head number in ``bits<9:13>``,
and the sector number in bits ``<14:16>``.

A summary of the current format,
and its validity,
can be obtained with the command::

    SHOW DPn FORMAT                Display format of unit n

To accommodate the variable formatting,
each track is allocated 2048 words in the data file.
A record consists of a three-word header,
the data,
and a five-word trailer:

======================  =========================================================
:math:`word 0`          Record length in words, not including header/trailer
:math:`word 1`          Record address
:math:`word 2`          Number of extension words used (0-4)
:math:`word 3`          Start of data record
:math:`word 3+n-1`      End of data record
:math:`word 3+n..7+n`   Record trailer: up to four extension words, plus checksum
======================  =========================================================

A record can "grow" by up to four words without disrupting the track formatting;
writing more than four extra words destroys the formatting of the rest of the track and causes a simulator error.

The disk pack controller implements these registers:

================  ====  ==========================================
Name              Size  Comments
================  ====  ==========================================
``STA``           16    Status
``BUF``           16    Data buffer
``FNC``           4     Controller function
``CW1``           16    Command word 1
``CW2``           16    Command word 2
``CSUM``          16    Record checksum
``BUSY``          1     Controller busy
``RDY``           1     Transfer ready
``EOR``           1     (DMA/DMC) End of range
``DEFINT``        1     Seek deferred interrupt pending
``INTREQ``        1     Interrupt request
``ENABLE``        1     Interrupt enable
``TBUF[0:2047]``  16    Track buffer
``RPTR``          11    Pointer to start of record in track buffer
``WPTR``          11    Pointer to current word in record
``BCTR``          15    Bit counter for formatting
``STIME``         24    Seek time, per cylinder
``XTIME``         24    Transfer time, per word
``BTIME``         24    Controller busy time
================  ====  ==========================================

Error handling is as follows:

==============  ====================================
Error           Processed as
==============  ====================================
Not attached    Pack off line; if ``STOP_IOE``, stop
End-of-file     Ignored
OS I/O error    Data error; if ``STOP_IOE``, stop
==============  ====================================

Symbolic display and input
==========================
The H316/H516 simulator implements symbolic display and input.
Display is controlled by command-line switches:

======  ======================================
``-a``  Display as ASCII character
``-c``  Display as two packed ASCII characters
``-m``  Display instruction mnemonics
======  ======================================

Input parsing is controlled by the first character typed in or by command-line switches:

===============  ===========================
``'`` or ``-a``  ASCII character
``"`` or ``-c``  Two packed ASCII characters
Alphabetic       Instruction mnemonic
Numeric          Octal number
===============  ===========================

Instruction input uses standard H316/H516 assembler syntax.
There are six instruction classes:
memory reference,
I/O,
control,
shift,
skip,
and operate.

.. compound::
   Memory reference instructions have the format ::

       memref{*} {C/Z} address{,1}

   where ``*`` signifies indirect,
   ``C`` a current sector reference,
   ``Z`` a sector zero reference,
   and ``1`` indexed.
   The ``address`` is an octal number in the range 0–077777;
   if ``C`` or ``Z`` is specified,
   the ``address`` is a page offset in the range 0–0777.
   Normally, ``C`` is not needed;
   the simulator figures out from the address what mode to use.
   However, when referencing memory outside the |CPU|,
   there is no valid PC,
   and ``C`` must be used to specify current sector addressing.

I/O instructions have the format ::

    io function,device

The ``function`` is an octal number in the range 0–17.
The ``device`` is a symbolic name (e.g., ``TTY``)
or an octal number in the range 0–77.

Control and operate instructions consist of a single opcode::

    opcode

Shift instructions have the format ::

    shift n

where ``n`` is an octal number in the range 0–77.

Skip instructions have the format ::

    sub-op sub-op sub-op...

The simulator checks that the combination of sub-opcodes is legal.
