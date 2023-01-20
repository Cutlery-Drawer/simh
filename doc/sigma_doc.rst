.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

*****************************
XDS Sigma 32b simulator usage
*****************************
:Date: 2016-03-27
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
    | ``sigma_defs.h``
    | ``sigma_io_defs.h``
    | ``sigma_cis.c``
    | ``sigma_coc.c``
    | ``sigma_cpu.c``
    | ``sigma_dk.c``
    | ``sigma_dp.c``
    | ``sigma_fp.c``
    | ``sigma_io.c``
    | ``sigma_lp.c``
    | ``sigma_map.c``
    | ``sigma_mt.c``
    | ``sigma_pt.c``
    | ``sigma_rad.c``
    | ``sigma_rtc.c``
    | ``sigma_sys.c``
    | ``sigma_tt.c``

XDS Sigma 32b features
======================
The XDS Sigma 32b simulator is configured as follows:

========================  =====================================================
Device name(s)            Simulates
========================  =====================================================
``CPU``                   XDS Sigma 5, 6, 7, 7 "big mem", 8, 9, or XDS 550, 560
``CHANA``\ ..\ ``CHANH``  I/O channels, up to 8
``PT``                    7060 paper tape reader/punch
``TT``                    7012 console Teletype
``LP``                    7440 or 7450 line printer
``RTC``                   Real-time clock
``MUX``                   Character-oriented communications subsystem
``DK``                    7250 cartridge disk subsystem with up to 8 drives
``DPA``                   Disk pack subsystem with up to 15 drives
``DPB``                   Disk pack subsystem with up to 15 drives
``RAD``                   7211/7212 or 7231/7232 fixed head disk
``MT``                    732X 9-track magnetic tape with up to 8 drives
========================  =====================================================

Most devices can be disabled or enabled with the ``SET <dev> DISABLED`` and ``SET <dev> ENABLED`` commands, respectively.

The ``LOAD`` and ``DUMP`` commands are not implemented.

.. |CPU| replace:: ``CPU``
.. _CPU:

Central processor (|CPU|)
-------------------------
Central processor options include the |CPU| model,
the |CPU| features,
and the size of main memory. ::

    SET CPU SIGMA5                Set CPU model to Sigma 5
    SET CPU SIGMA6                Set CPU model to Sigma 6
    SET CPU SIGMA7                Set CPU model to Sigma 7
    SET CPU SIGMA7B               Set CPU model to Sigma 7 "big" memory
    SET CPU SIGMA8                Set CPU Model to Sigma 8
    SET CPU SIGMA9                Set CPU model to Sigma 9
    SET CPU 550                   Set CPU model to 550
    SET CPU 560                   Set CPU model to 560
    SET CPU FP                    Enable floating point, if available
    SET CPU NOFP                  Disable floating point, if optional
    SET CPU DECIMAL               Enable decimal, if available
    SET CPU NODECIMAL             Disable decimal, if optional
    SET CPU LASLAM                Enable LAS/LAM instructions (6/7 only)
    SET CPU NOLASLAM              Disable LAS/LAM instructions
    SET CPU MAP                   Enable memory map, if available
    SET CPU NOMAP                 Disable memory map, if optional
    SET CPU WRITELOCK             Enable write locks, if available
    SET CPU NOWRITELOCK           Disable write locks, if optional
    SET CPU RBLKS=n               Set number of register blocks
    SET CPU CHAN=n                Set number of channels
    SET CPU 32K                   Set memory size = 32KW
    SET CPU 64K                   Set memory size = 64KW
    SET CPU 128K                  Set memory size = 64KW
    SET CPU 256K                  Set memory size = 256KW
    SET CPU 512K                  Set memory size = 512KW
    SET CPU 1M                    Set memory size = 1024KW

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial configuration is Sigma 7 CPU,
4 channels,
128KW of memory,
floating point,
decimal,
map and writelocks options enabled.

|CPU| registers include the visible state of the processor as well as the control registers for the interrupt system.

======================  ====  ========================================
Name                    Size  Comments
======================  ====  ========================================
``PC``                  17    Program counter
``R0``\ ..\ ``R15``     32    Active general registers
``PSW1``                32    Processor status word 1
``PSW2``                32    Processor status word 2
``PSW4``                32    Processor status word 4 (``5x0`` only)
``CC``                  4     Condition codes
``RP``                  5     Register block selector
``SSW1``\ ..\ ``SSW4``  1     Sense switches
``PDF``                 1     Processor fault flag
``ALARM``               1     Console alarm
``ALENB``               1     Console alarm enable
``PCF``                 1     Console controlled flop
``EXULIM``              8     Limit for nested EXUs
``STOP_ILL``            1     If 1, stop on undefined instruction
``REG``                 512   Register blocks, 32 × 16
``WRU``                 8     Interrupt character
======================  ====  ========================================

The |CPU| provides an address converter to display the byte
(halfword, word, or doubleword)
address equivalent of a byte
(halfword, word, or doubleword)
input address.
Optionally,
the input address can be run through memory relocation::

    SHOW {-flags} CPU {BA,HA,WA,DA}=address

``BA``, ``HA``, ``WA``, ``DA`` specify that the input address is a byte, halfword, word, or doubleword address, respectively.
The flags are:

.. table::
   :class: switches

   ======  ================================
   ``-v``  Input address is virtual
   ``-b``  Output is byte address
   ``-h``  Output is halfword address
   ``-w``  Output is word address (default)
   ``-d``  Output is doubleword address
   ======  ================================

For example::

    SHOW –B CPU WA=100
    Physical word 100: physical byte 400

The |CPU| can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY               Clear history buffer
    SET CPU HISTORY=0             Disable history
    SET CPU HISTORY=n             Enable history, length = n
    SHOW CPU HISTORY              Print CPU history
    SHOW CPU HISTORY=n            Print first 'n' entries of CPU history

The maximum length for the history is 1M entries.

.. |MAP| replace:: ``MAP``
.. _MAP:

Memory map (|MAP|)
------------------
The memory map implements two distinct forms of protection:
memory mapping and access protection on virtual addresses;
and write lock protection on physical addresses.
It also includes a skeleton implementation of the memory status registers from the Sigma 8 and 9,
and the XDS 550 and 560.
It implements these registers:

===============  ====  ========================
Name             Size  Comments
===============  ====  ========================
``REL[0:511]``   11    Relocation registers
``ACC[0:511]``   2     Access controls
``WLK[0:2047]``  4     Write locks
``SR0[32]``      32    Memory status register 0
``SR1[32]``      32    Memory status register 1
===============  ====  ========================

.. |INT| replace:: ``INT``
.. _INT:

Interrupt system (|INT|)
------------------------
The Sigma series implements a complex,
multi-level interrupts system,
with a minimum of 32 distinct interrupts.
The interrupt system can be expanded to up to 224 interrupt levels.
It implements these registers:

===================  ====  =====================================
Name                 Size  Comments
===================  ====  =====================================
``IHIACT``           9     Highest active interrupt level
``IHIREQ``           9     Highest outstanding interrupt request
``IREQ[0:15]``       16    Interrupt requests
``IENB[0:15]``       16    Interrupt enabled flags
``IARM[0:15]``       16    Interrupt armed flags
``S9_SNAP``          32    Sigma 9 snapshot register
``S9_MARG``          32    Sigma 9 margins registers
``S5X0_IREG[0:31]``  32    5X0 internal registers (unused)
===================  ====  =====================================

The simulator supports a variable number of external interrupt blocks.
The minimum number is one,
the maximum is four (5X0) to thirty-two (Sigma 7).
The user can change the number of external interrupt blocks with the ``SET INT EIBLKS`` command::

    SET INT EIBLKS=4              Configure four external interrupt blocks

Although the Sigma series supports configurable interrupt group priorities,
the simulator uses a fixed priority arrangement,
high to low as follows:

- Counters
- Counter overflow
- I/O and panel interrupt
- External group 2 (there is no external group 1)
- External group 3
- Etc.

.. |CHANA| replace:: ``CHANA``
.. |CHANH| replace:: ``CHANH``
.. _CHANA:
.. _CHANH:

Channels (|CHANA| .. |CHANH|)
-----------------------------
A Sigma 32b system has up to eight I/O channels,
designated A, B, C, D, E, F, G, and H.
The association between a device and a channel is displayed by the commands ``SHOW <dev> CHAN`` and ``SHOW <dev> DVA``::

    SHOW MT CHAN
    channel=A
    SHOW MT DVA
    address=00

The user can change the association with the ``SET <dev> CHAN=<chan>`` command,
where ``<chan>`` is a channel letter,
and with the ``SET <dev> DVA=<addr>`` command,
where ``<addr>`` is a legal device address::

    SET MT CHAN=C
    SET MT DVA=4
    SHOW MT CHAN,DVA
    channel=B
    address=04

The default channel assignments for the simulator are:

=======  =======  =========
Device   Channel  Address
=======  =======  =========
``MT``   chan A   address 0
``TT``   chan A   address 1
``LP``   chan A   address 2
``PT``   chan A   address 5
``COC``  chan A   address 6
``RAD``  chan B   address 0
``DK``   chan B   address 1
``DPA``  chan C   address 0
``DPB``  chan D   address 1
=======  =======  =========

The user can display all the channel registers associated with a device with the ``SHOW <dev> CSTATE`` command::

    SHOW MT CSTATE

Each channel has eight registers.
The registers are arrays,
with entry 1 for device ``[0]``,
entry ``[1]`` for device 1, etc.
A channel supports a maximum of 32 devices.

==============  ====  =====================================
Name            Size  Comments
==============  ====  =====================================
``CLC[0:31]``   16    Channel location counter (doubleword)
``CMD[0:31]``   8     Current channel command
``CMF[0:31]``   8     Channel command flags
``BA[0:31]``    24    Byte address (byte)
``BC[0:31]``    16    Byte count
``CMF[0:31]``   16    Channel status flags
``CHI[0:31]``   8     Channel interrupt flags
``CHSF[0:31]``  8     Channel simulator flags
==============  ====  =====================================

.. |TT| replace:: ``TT``
.. _TT:

7012 console Teletype (|TT|)
----------------------------
The console Teletype (|TT|) consists of two units.
Unit 0 is for console input,
unit 1 for console output.
The console implements these registers:

=========  ====  =====================================
Name       Size  Comments
=========  ====  =====================================
``KPOS``   32    Number of characters input
``TPOS``   32    Number of characters input
``TTIME``  24    Time from I/O initiation to interrupt
``PANEL``  8     Character code for panel interrupt
=========  ====  =====================================

The ``PANEL`` variable defaults to Control-P.
If the user types the control panel character,
it is not echoed;
instead, a control panel interrupt is generated.

The console can be set to one of two modes, ``7P`` or ``UC``:

======  ===================  ==================================
Mode    Input characters     Output characters
======  ===================  ==================================
``UC``  Lowercase converted  Lowercase converted
        to uppercase         to uppercase
``7P``                       Non-printing characters suppressed
======  ===================  ==================================

The default mode is ``UC``.
The console character set is a subset of EBCDIC.
By default,
the console is assigned to channel A as device 1.

.. |PT|  replace:: ``PT``
.. |PT0| replace:: ``PT0``
.. |PT1| replace:: ``PT1``
.. _PT:
.. _PT0:
.. _PT1:

7060 paper tape reader (|PT|)
-----------------------------
The paper tape controller implements two units.
Unit 0 (|PT0|) is for paper tape input,
unit 1 (|PT1|) for paper tape output.
Paper tapes are simulated as disk files.
For the reader,
register ``RPOS`` specifies the number of the next data item to be read.
For the ``punch``, register ``PPOS`` specifies the number of the next data item to be written.
Thus, by changing ``RPOS`` or ``PPOS``,
the user can backspace or advance the reader or punch.

The paper tape controller implements these registers:

=============  ====  ===========================================
Name           Size  Comments
=============  ====  ===========================================
``CMD``        9     Current command or state
``NZC``        1     Non-zero character seen since ``ATTACH``
``RPOS``       32    Position in the reader input file
``RTIME``      24    Time from reader I/O initiation to response
``RSTOP_IOE``  1     Stop on reader I/O error
``PPOS``       32    Position in the punch output file
``PTIME``      24    Time from punch I/O initiation to response
``PSTOP_IOE``  1     Punch stop on I/O error
=============  ====  ===========================================

The paper-tape reader supports the ``BOOT`` command.
``BOOT PT0`` simulates the standard console fill sequence.

Reader error handling is as follows:

+---------------+---------------+-----------------------+
| Error         | ``RSTOP_IOE`` | Processed as          |
+===============+===============+=======================+
| Not attached  | 1             | Report error and stop |
|               +---------------+-----------------------+
|               | 0             | Out of tape           |
+---------------+---------------+-----------------------+
| End-of-file   | 1             | Report error and stop |
|               +---------------+-----------------------+
|               | 0             | Out of tape           |
+---------------+---------------+-----------------------+
| OS I/O error  | x             | Rport error and stop  |
+---------------+---------------+-----------------------+

Punch error handling is as follows:

+---------------+---------------+-----------------------+
| Error         | ``PSTOP_IOE`` | Processed as          |
+===============+===============+=======================+
| Not attached  | 1             | Report error and stop |
|               +---------------+-----------------------+
|               | 0             | Out of tape           |
+---------------+---------------+-----------------------+
| OS I/O error  | x             | Rport error and stop  |
+---------------+---------------+-----------------------+

By default,
the paper tape reader is assigned to channel A as device 5.

.. |LP|  replace:: ``LP``
.. |LPT| replace:: ``LPT``
.. _LP:
.. _LPT:

7440 or 7450 line printer (|LP|)
--------------------------------
The line printer (|LPT|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the printer.

The line printer implements these registers:

==============  ====  ====================================
Name            Size  Comments
==============  ====  ====================================
``CMD``         9     Current command or state
``BUF[0:131]``  7     Data buffer
``PASS``        1     Printing pass (7440 only)
``INH``         1     Inhibit spacing flag
``RUNAWAY``     1     Runaway carriage-control tape flag
``CCT[0:131]``  8     Carriage control tape
``CCTP``        8     Pointer into carriage control tape
``CCTL``        8     Length of carriage control tape
``POS``         32    Position in the output file
``TIME``        24    Time from I/O initiation to response
``STOP_IOE``    1     Stop on I/O error
==============  ====  ====================================

The line printer model can be set to either the 7440 or the 7450::

    SET LP 7440                   Set model to 7440
    SET LP 7450                   Set model to 7450

The default model is the 7450.

A carriage control tape can be loaded with the ``SET LP CCT`` command::

    SET LP CCT=<file>             Load carriage-control tape from <file>

The format of a carriage control tape consists of multiple lines.
Each line contains an optional repeat count,
enclosed in parentheses,
optionally followed by a series of column numbers separated by commas.
Column numbers must be between 0 and 7.
The following are all legal carriage control specifications:

================  ==============================
``<blank line>``  No punch
``(5)``           5 lines with no punches
``1,5,7``         Columns 1, 5, 78 punched
``(10)2``         10 lines with column 2 punched
``1``             Column 1 punched
================  ==============================

The default form is 1 line long, with every column punched.

Error handling is as follows:

+---------------+---------------+-----------------------+
| Error         | ``STOP_IOE``  | Processed as          |
+===============+===============+=======================+
| Not attached  | 1             | Report error and stop |
|               +---------------+-----------------------+
|               | 0             | Out of paper          |
+---------------+---------------+-----------------------+
| OS I/O error  | x             | Rport error and stop  |
+---------------+---------------+-----------------------+

By default, the line printer is assigned to channel A as device 2.

.. |RTC| replace:: ``RTC``
.. _RTC:

Real-time clock (|RTC|)
-----------------------
The Sigma 32b series implements four real-time clocks.
Three of them can be set to a variety of frequencies,
including off, 50Hz, 60Hz, 100Hz, and 500hz.
The fourth always runs at 500Hz.
The frequency of each clock can be can be adjusted as follows::

    SET RTC {C1,C2,C3}=freq       Set clock 1/2/3 to specified frequency

The frequency of each clock can be displayed as follows::

    SHOW RTC {C1,C2,C3,C4}        Show clock 1/2/3/4 frequency

Clocks 1 and 2 default to off,
clocks 3 and 4 to 500Mhz.

The |RTC| can also show the state of all real-time events in the simulator::

    SHOW RTC EVENTS

The real-time clocks autocalibrate;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

.. |MUX|  replace:: ``MUX``
.. |MUXL| replace:: ``MUXL``
.. _MUX:
.. _MUXL:

Character-oriented communications subsystem (|MUX|)
---------------------------------------------------
The character-oriented communications subsystem implements up to 64 asynchronous interfaces,
with modem control.
The subsystem has two controllers:
|MUX| for the scanner,
and |MUXL| for the individual lines.
The terminal multiplexer performs input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the port to be used::

    ATTACH MUX <port>             Set up listening port

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

Unlike the console,
the |MUX| operates in ASCII.
Each line (each unit of |MUXL|) supports four character processing modes: ``UC``, ``7P``, ``7B``, and ``8B``.

======  =======================  ==================================
Mode    Input characters         Output characters
======  =======================  ==================================
``UC``  High-order bit cleared,  High-order bit cleared,
        lowercase converted      lowercase converted
        to uppercase             to upper case
``7P``  High-order bit cleared   High-order bit cleared,
                                 non-printing characters suppressed
``7B``  High-order bit cleared   High-order bit cleared
``8B``  No changes               No changes
======  =======================  ==================================

The default is ``UC``.
In addition,
each line supports output logging.
The ``SET MUXLn LOG`` command enables logging on a line::

    SET MUXLn filename            Log output of line n to filename

The ``SET MUXLn NOLOG`` command disables logging and closes the open log file, if any.

Once |MUX| is attached and the simulator is running,
the multiplexer listens for connections on the specified port.
It assumes that the incoming connections are Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a ``SET MUX DISCONNECT`` command,
or a ``DETACH MUX`` command.

Other special multiplexer commands::

    SHOW MUX CONNECTIONS          Show current connections
    SHOW MUX STATISTICS           Show statistics for active connections
    SET MUXLn DISCONNECT          Disconnect the specified line

The controller (|MUX|) implements these registers:

==============  ====  ===============================
Name            Size  Comments
==============  ====  ===============================
``STA[0:63]``   8     Status, lines 0 to 63
``RBUF[0:63]``  8     Receive buffer, lines 0 to 63
``XBUF[0:63]``  8     Transmit buffer, lines 0 to 63
``SCAN``        6     Current scanner line
``SLCK``        1     Scanner locked flag
``CMD``         2     Channel command or state
==============  ====  ===============================

The lines (|MUXL|) implements these registers:

==============  ====  ===============================
Name            Size  Comments
==============  ====  ===============================
``TIME[0:63]``  24    Transmit time, lines 0 to 31
==============  ====  ===============================

The terminal multiplexer does not support save and restore.
All open connections are lost when the simulator shuts down or |MUX| is detached.
By default, the multiplexer is assigned to channel A as device 6.

.. |DPA| replace:: ``DPA``
.. |DPB| replace:: ``DPB``
.. |DP|  replace:: ``DP``
.. _DP:
.. _DPA:
.. _DPB:

7240, 7270, 7260, 7265, 7275, and T3281 moving head disk controllers (|DPA|, |DPB|)
-----------------------------------------------------------------------------------
The Sigma 32b series supports two moving head disk controllers (|DPA|, |DPB|).
Each can be set to model one of six controllers
(7240, 7270, 7260, 7265, 7275, or Telefile 3281)::

    SET DP{A,B} 7240              Set DPA (or DPB) to 7240
    SET DP{A,B} 7270              Set DPA (or DPB) to 7270
    SET DP{A,B} 7260              Set DPA (or DPB) to 7260
    SET DP{A,B} 7265              Set DPA (or DPB) to 7265
    SET DP{A,B} 7275              Set DPA (or DPB) to 7275
    SET DP{A,B} T3281             Set DPA (or DPB) to Telefile 3281

The default for |DPA| is the 7270, for |DPB|, the 3T281.

The 7240 and 7270 support up to 8 7242 and 7271 drives, respectively.
The 7260, 7265, and 7275 support up to 15 7261, 7266, and 7276 drives, respectively.
The T3281 supports up to 15 drives of three different types,
which can be mixed:

=====  =========  =====  =======
Model  Cylinders  Heads  Sectors
=====  =========  =====  =======
3288   17         5      823
3282   11         19     815
3283   17         19     815
=====  =========  =====  =======

The command to set a drive to a particular model is::

    SET DPn <drive_type>          SET unit n to the specified drive type

The T3281 can also be set to autosize on ``ATTACH``::

    SET DPn AUTO                  SET unit n to autosize on ATTACH

Units can be set ``ENABLED`` or ``DISABLED``.
The |DP| controller supports the ``BOOT`` command.

The |DP| controllers implements the registers listed below.
Registers suffixed with ``[0:14]`` are replicated per drive.

==============  ====  ====================================
Name            Size  Comments
==============  ====  ====================================
``FLAGS``       8     Controller flags
``DIFF``        16    Cylinder difference
``SKI``         16    Queued seek interrupts
``TEST``        16    Test mode flags
``ADDR[0:14]``  32    Current disk address, drives 0 to 14
``CMD[0:14]``   32    Current disk command, drives 0 to 14
``TIME``        24    Time between word transfers
``STIME``       24    Seek time
``WLK``         16    Write lock switches
``STOP_IOE``    1     Stop on I/O error
==============  ====  ====================================

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

By default, |DPA| is assigned to channel C as device 0,
and |DPB| is assigned to channel D as device 0.

.. |DK| replace:: ``DK``
.. _DK:

7250 cartridge disk controller (|DK|)
-------------------------------------
The cartridge disk controller (|DK|) supports up to 8 drives.
|DK| options include the ability to make drives write enabled or write locked::

    SET DKn LOCKED                Set drive 'n' write locked
    SET DKn WRITEENABLED          Set drive 'n' write enabled

The cartridge disk controller implements these registers:

============  ====  ===============================
Name          Size  Comments
============  ====  ===============================
``CMD``       9     Current command or state
``FLAGS``     8     Controller flags
``ADDR``      8     Current disk address
``TIME``      4     Interval between data transfers
``STIME``     4     Seek interval
``STOP_IOE``  1     Stop on I/O error
============  ====  ===============================

Error handling is as follows:

+--------------+--------------+-----------------------------+
| Error        | ``STOP_IOE`` | Processed as                |
+==============+==============+=============================+
| Not attached | 1            | Report error and stop       |
|              +--------------+-----------------------------+
|              | 0            | Non-existent disk           |
+--------------+--------------+-----------------------------+

|DK| data files are buffered in memory;
therefore, end-of-file and OS I/O errors cannot occur.
By default, the cartridge disk controller is assigned to channel B as device 1.

.. |RAD| replace:: ``RAD``
.. _RAD:

7211/7212 or 7231/7232 fixed head disk (|RAD|)
----------------------------------------------
The Sigma 32b series supports two models of fixed head disk::

    7211/7212         1.343MW
    7231/7232         1.572MW

The user can select the model as follows::

    SET RAD 7211      Set model to 7211/7212
    SET RAD 7231      Set model to 7231/7232

The fixed head disk controller supports four units (drives).
Units can be set ``ENABLED`` or ``DISABLED``.

The fixed head disk controller implements these registers:

=========  ===  ===============================
``CMD``    9    Current command or state
``FLAGS``  8    Controller flags
``ADDR``   15   Current disk address
``WLK``    16   Write lock switches
``TIME``   24   Interval between data transfers
=========  ===  ===============================

Error handling is as follows:

+--------------+--------------+-----------------------------+
| Error        | ``STOP_IOE`` | Processed as                |
+==============+==============+=============================+
| Not attached | 1            | Report error and stop       |
|              +--------------+-----------------------------+
|              | 0            | Non-existent disk           |
+--------------+--------------+-----------------------------+

|RAD| data files are buffered in memory;
therefore, end-of-file and OS I/O errors cannot occur.
By default, the fixed head disk is assigned to channel B as device 0.

.. |MT| replace:: ``MT``
.. _MT:

723X nine-track magnetic tape (|MT|)
------------------------------------
The magnetic tape controller supports up to eight units.
|MT| options include the ability to make units write enabled or write locked. ::

    SET MTn LOCKED                Set unit 'n' write locked
    SET MTn WRITEENABLED          Set unit 'n' write enabled

Magnetic tape units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET MTn CAPAC=m               Set unit 'n' capacity to 'm' MB (0 = unlimited)
    SHOW MTn CAPAC                Show unit 'n' capacity in MB

Units can also be set ``ENABLED`` or ``DISABLED``.
The magnetic tape controller supports the ``BOOT`` command.
``BOOT MTn`` simulates the standard console fill sequence for unit ``n``.

The magnetic tape implements these registers:

================  ====  ====================================
Name              Size  Comments
================  ====  ====================================
``BUF[0:65535]``  8     Transfer buffer
``BPTR``          17    Buffer pointer
``BLNT``          17    Buffer length
``RWINT``         8     Queued rewind interrupts
``TIME``          24    Interval between character transfers
``CTIME``         24    Channel response time
``RWTIME``        24    Rewind time
``UST[0:7]``      8     Drive status, drives 0 to 7
``UCMD[0:7]``     8     Channel command, drives 0 to 7
``POS[0:7]``      32    Position, drives 0 to 7
``STOP_IOE``      1     Stop on I/O error
================  ====  ====================================

Error handling is as follows:

==============  =====================================
Error           Processed as
==============  =====================================
Not attached    Tape not ready; if ``STOP_IOE``, stop
End-of-file     End of tape
OS I/O error    End of tape; if ``STOP_IOE``, stop
==============  =====================================

By default, the magnetic tape is assigned to channel A as device 0.

Symbolic display and input
==========================
The Sigma 32b simulator implements symbolic display and input.
Display is controlled by command-line switches:

.. table::
   :class: switches

   =======  ====================================================
   ``-a``   Display as ASCII character (byte addressing)
   ``-b``   Display as byte (byte addressing)
   ``-e``   Display at EBCDIC character (byte addressing)
   ``-h``   Display as halfword (halfword addressing)
   ``-ca``  Display as four ASCII characters (word addressing)
   ``-ce``  Display as four EBCDIC characters (word addressing)
   ``-m``   Display instruction mnemonics (word addressing)
   =======  ====================================================

Input parsing is controlled by the first character typed in or by command-line switches:

.. table::
   :class: switches

   ================  ===============================================
   ``#`` or ``-a``   ASCII character (byte addressing)
   ``'`` or ``-e``   EBCDIC character (byte addressing)
   ``-b``            Hexadecimal byte (byte addressing)
   ``-h``            Hexadecimal halfword (halfword addressing)
   ``"`` or ``-ac``  Four packed ASCII characters (word addressing)
   ``-ae``           Four packed EBCDIC characters (word addressing)
   Alphabetic        Instruction mnemonic (word addressing)
   Numeric           Hexadecimal word (word addressing)
   ================  ===============================================

Instruction input uses (more or less) standard XDS Sigma assembler syntax.
All instructions are variants on the same basic form::

    mnemonic{,reg}    {*{address{,index}}}

Mnemonics are symbolic names for instructions.
Registers are decimal values between 0 and 15.
``*`` represents indirect addressing.
Addresses are hexadecimal and can be signed if used as literals.
Index registers are always less than 8 and thus can be considered decimal.

Examples::

    LW,5        *100,7
    AI,14       -3
    LCFI        A
    WAIT
    BCR,12      400
    BNOV        10FE
    SLS,6       -8
