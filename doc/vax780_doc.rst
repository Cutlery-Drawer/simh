.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

**************************
VAX-11/780 simulator usage
**************************
:Date: 2021-08-08
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

This memorandum documents the DEC VAX-11/780 simulator.

Simulator files
===============
To compile the VAX-11/780, you must define ``VM_VAX``, ``VAX780``, and ``USE_INT64`` as part of the compilation command-line.
To enable extended file support (files greater than 2GB), you must define ``USE_ADDR64`` as part of the command-line as well.

``sim/``
    | ``scp.h``
    | ``sim_console.h``
    | ``sim_defs.h``
    | ``sim_disk.h``
    | ``sim_ether.h``
    | ``sim_fio.h``
    | ``sim_rev.h``
    | ``sim_serial.h``
    | ``sim_sock.h``
    | ``sim_tape.h``
    | ``sim_timer.h``
    | ``sim_tmxr.h``
    | ``scp.c``
    | ``sim_console.c``
    | ``sim_disk.c``
    | ``sim_ether.c``
    | ``sim_fio.c``
    | ``sim_serial.c``
    | ``sim_sock.c``
    | ``sim_tape.c``
    | ``sim_timer.c``
    | ``sim_tmxr.c``

``sim/vax/``
    | ``vax_defs.h``
    | ``vax780_defs.h``
    | ``vax_vmb_exe.h``
    | ``vax_cis.c``
    | ``vax_cmode.c``
    | ``vax_cpu.c``
    | ``vax_cpu1.c``
    | ``vax_fpa.c``
    | ``vax_mmu.c``
    | ``vax_octa.c``
    | ``vax_sys.c``
    | ``vax_syscm.c``
    | ``vax7x0_mba.c``
    | ``vax780_mem.c``
    | ``vax780_sbi.c``
    | ``vax780_stddev.c``
    | ``vax780_syslist.c``
    | ``vax780_uba.c``

``sim/pdp11/``
    | ``pdp11_cr_dat.h``
    | ``pdp11_mscp.h``
    | ``pdp11_uqssp.h``
    | ``pdp11_xu.h``
    | ``pdp11_cr.c``
    | ``pdp11_dmc.c``
    | ``pdp11_dz.c``
    | ``pdp11_hk.c``
    | ``pdp11_lp.c``
    | ``pdp11_rl.c``
    | ``pdp11_rp.c``
    | ``pdp11_rq.c``
    | ``pdp11_ry.c``
    | ``pdp11_tq.c``
    | ``pdp11_ts.c``
    | ``pdp11_tu.c``
    | ``pdp11_vh.c``
    | ``pdp11_xu.c``

Additional files are:

``sim/vax/``
    | ``vmb.exe`` (Standard boot code)

VAX780 features
===============
The VAX780 simulator is configured as follows:

==========================  =============================================================================
Device name(s)              Simulates
==========================  =============================================================================
``CPU``                     VAX-11/780 CPU
``TLB``                     Translation buffer
``SBI``                     System bus controller
``MCTL0``, ``MTCL1``        Memory controllers, MS780C with 4MB memory each, or MS780E with 8MB-64MB each
``UBA``                     DW780 Unibus adapter
``MBA0``, ``MBA1``          RH780 Massbus adapters
``TODR``                    Time-of-day clock
``TMR``                     Interval timer
``TTI``, ``TTO``            Console terminal
``RXC``                     Console RX01 floppy disk
``DZ``                      DZ11 8-line terminal multiplexer (up to 32)
``VH``                      DHU11 16-line terminal multiplexer (up to 4)
``CR``                      CR11 card reader
``LPT``                     LP11 line printer
``RP``                      RP04/05/06/07, RM02/03/05/80 Massbus disks, up to eight
``HK``                      RK611/RK06(7) cartridge disk controller with eight drives
``RL``                      RL11/RL01(2) cartridge disk controller with four drives
``RQ``                      UDA50 MSCP controller with four drives
``RQB``                     Second UDA50 MSCP controller with four drives
``RQC``                     Third UDA50 MSCP controller with four drives
``RQD``                     Fourth UDA50 MSCP controller with four drives
``RY``                      RX211 floppy disk controller with two drives
``TS``                      TS11 magnetic tape controller with one drive
``TQ``                      TUK50 TMSCP magnetic tape controller with four drives
``TU``                      TM03 tape formatter with eight TE16/TU45/TU77 drives
``XU``                      DEUNA/DELUA Ethernet controller
``XUB``                     Second DEUNA/DELUA Ethernet controller
``DMC``                     DMC11/DMR11 Synchronous network controller
==========================  =============================================================================

The DZ, VH, LPT, RP, RL, RQ, RQB, RQC, RQD, RY, TS, TQ, TU, XU, XUB and DMC devices can be set ``DISABLED``.
RQB, RQC, RQD, VH, XU, XUB and DMC are disabled by default.

The VAX780 simulator implements several unique stop conditions:

- Change mode to interrupt stack
- Illegal vector (``bits<1:0>`` = 2 or 3)
- Unexpected exception during interrupt or exception
- Process PTE in P0 or P1 space instead of system space
- Unknown IPL
- Infinite loop (BRB/W to self at ``IPL 1F``)

The ``LOAD`` command supports a simple binary format,
consisting of a stream of binary bytes without origin or checksum, for loading memory.
The ``DUMP`` command is not implemented.

CPU and system devices
----------------------

CPU
"""
CPU options include the size of main memory and the treatment of the HALT instruction. ::

    SET CPU 8M                    Set memory size = 8MB
    SET CPU 16M                   Set memory size = 16MB
    SET CPU 32M                   Set memory size = 32MB
    SET CPU 48M                   Set memory size = 48MB
    SET CPU 64M                   Set memory size = 64MB
    SET CPU 128M                  Set memory size = 128MB

The CPU also implements a command to display a virtual to physical address translation::

    SHOW {-kesu} CPU VIRTUAL=n    Show translation for address n
                                  in kernel/exec/supervisor/user mode

The CPU also implements the console ``AUTORESTART`` switch::

    SET CPU CONHALT               Enable AUTORESTART
    SET CPU SIMHALT               Disable AUTORESTART

Notes on memory size:

- The first version of the VAX-11/780 used MS780C controllers,
  which supported 1-4MB of memory per controller.
  This is the only memory controller recognized by VMS V1.
  MS780E controllers supported 4MB-64MB per controller.
- The controller type is set automatically based on memory size.

Initial memory size is 8MB.

Memory can be loaded with a binary byte stream using the ``LOAD`` command.
The ``LOAD`` command recognizes three switches:

.. table::
   :class: switches

   ======  ================================
   ``-o``  Origin argument follows filename
   ``-r``  Load ROM in memory controller 0
   ``-s``  Load ROM in memory controller 1
   ======  ================================

These switches are recognized when examining or depositing in CPU memory:

.. table::
   :class: switches

   ======  =========================================================
   ``-b``  Examine/deposit bytes
   ``-w``  Examine/deposit words
   ``-l``  Examine/deposit longwords
   ``-d``  Data radix is decimal
   ``-o``  Data radix is octal
   ``-h``  Data radix is hexadecimal
   ``-m``  Examine (only) VAX instructions
   ``-p``  Examine/deposit PDP-11 (compatibility mode) instructions
   ``-r``  Examine (only) RADIX50 encoded data
   ``-v``  Interpret address as virtual, current mode
   ``-k``  Interpret address as virtual, kernel mode
   ``-e``  Interpret address as virtual, executive mode
   ``-s``  Interpret address as virtual, supervisor mode
   ``-u``  Interpret address as virtual, user mode
   ======  =========================================================

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

=================  ====  ========================================
Name               Size  Comments
=================  ====  ========================================
``PC``             32    Program counter
``R0`` .. ``R14``  32    R0 to R14
``AP``             32    Alias for R12
``FP``             32    Alias for R13
``SP``             32    Alias for R14
``PSL``            32    Processor status longword
``CC``             4     Condition codes, PSL<3:0>
``KSP``            32    Kernel stack pointer
``ESP``            32    Executive stack pointer
``SSP``            32    Supervisor stack pointer
``USP``            32    User stack pointer
``IS``             32    Interrupt stack pointer
``SCBB``           32    System control block base
``PCBB``           32    Process control block base
``P0BR``           32    P0 base register
``P0LR``           22    P0 length register
``P1BR``           32    P1 base register
``P1LR``           22    P1 length register
``SBR``            32    System base register
``SLR``            22    System length register
``SISR``           16    Software interrupt summary register
``ASTLVL``         4     AST level register
``MAPEN``          1     Memory management enable
``PME``            1     Performance monitor enable
``TRPIRQ``         8     Trap/interrupt pending
``CRDERR``         1     Correctible read data error flag
``MEMERR``         1     Memory error flag
``PCQ[0:63]``      32    PC prior to last PC change or interrupt;
                         most recent PC change first
``WRU``            8     Interrupt character
=================  ====  ========================================

The CPU attempts to detect when the simulator is idle.
When idle, the simulator does not use any resources on the host system.
Idle detection is controlled by the ``SET IDLE`` and ``SET NOIDLE`` commands::

    SET CPU IDLE{=platform}{:n}  Enable idle detection for the specified platform.
                                 Supported platform names are:
                                 VMS, ULTRIX, ULTRIXOLD, ULTRIX-1.X, 3BSD,
                                 4.0BSD, 4.1BSD, 4.2BSD, QUASIJARUS, NETBSD,
                                 NETBSDOLD, OPENBSD, OPENBSDOLD, 32V, ELN
    SET CPU NOIDLE               Disable idle detection

Idle detection is disabled by default.
If idle detection is enabled with an incorrect operating system setting,
simulator performance could be impacted.
The default operating system setting is VMS.
The value ``n``, if present in the ``SET CPU IDLE={OS}:n`` command,
indicates the number of seconds the simulator must run before idling starts.

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY               Clear history buffer
    SET CPU HISTORY=0             Disable history
    SET CPU {-T} HISTORY=n{:file} Enable history, length = n
    SHOW CPU HISTORY              Print CPU history
    SHOW CPU HISTORY=n            Print first n entries of CPU history

The ``-T`` switch causes simulator time to be recorded (and displayed) with each history entry.
When writing history to a file (``SET CPU HISTORY=n:file``), ``n`` specifies the buffer flush frequency.
Warning: prodigious amounts of disk space may be consumed.
The maximum length for the history is 250000 entries.

Translation buffer (TLB)
""""""""""""""""""""""""
The translation buffer consists of two units,
representing the system and user translation buffers, respectively.
It has no registers.
Each translation buffer entry consists of two 32b words, as follows:

===============  ==========
``word n``       Tag
``word n+1``     Cached PTE
===============  ==========

An invalid entry is indicated by a tag of 0xFFFFFFFF.

SBI controller (SBI)
""""""""""""""""""""
The SBI is the VAX-11/780 system bus.
The simulated SBI implements these registers:

==========  ====  ===============================
Name        Size  Comments
==========  ====  ===============================
``NREQ14``  16    Nexus IPL14 interrupt requests
``NREQ15``  16    Nexus IPL15 interrupt requests
``NREQ16``  16    Nexus IPL16 interrupt requests
``NREQ17``  16    Nexus IPL17 interrupt requests
``WCSA``    16    Writeable control store address
``WCSD``    32    Writeable control store data
``MBRK``    13    Microbreak register
``SBIFS``   32    SBI fault status
``SBISC``   32    SBI silo compare
``SBIMT``   32    SBI maintenance register
``SBIER``   32    SBI error status
``SBITMO``  32    SBI timeout address
==========  ====  ===============================

Memory controllers (MCTL0, MCTL1)
"""""""""""""""""""""""""""""""""
The memory controllers implement the registers for the MS780C (8MB memory) or MS780E (16MB or greater memory).
Each controller implements these registers:

===============  ====  ================================
Name             Size  Comments
===============  ====  ================================
``CRA``          32    Control register A
``CRB``          32    Control register B
``CRC``          32    Control register C
``CRD``          32    Control register D (MS780E only)
``ROM[0:1023]``  32    Bootstrap ROM
===============  ====  ================================

ROM can be loaded from a file with the commands ::

    LOAD -R <file>                Load MCTL0 ROM
    LOAD -S <file>                Load MCTL1 ROM

Time-of-day clock (TODR)
""""""""""""""""""""""""
The TODR tracks time since an arbitrary start in 10 millisecond intervals.
It has these registers:

========  ====  ====================
Name      Size  Comments
========  ====  ====================
``TODR``  32    Time-of-day register
``TIME``  24    Delay between ticks
========  ====  ====================

The TODR register autocalibrates against real-world time.

There are two modes of TODR operation:

1. **Default VMS mode.**
   Without initializing the TODR it returns the current time of year offset which VMS would set the clock to if VMS knew the correct time (i.e., by manual input).
   This is correct almost all the time unless a VMS disk hadn't been booted from in the current year.
   This mode produces strange time results for non-VMS OSes on each system boot.
2. **OS agnostic mode.**
   This mode behaves precisely like the VAX780 TODR and works correctly for all OSes.
   This mode is enabled by attaching the CLK to a battery backup state file for the TOY clock (i.e., ``sim> attach CLK TOY_CLOCK``).
   When operating in OS agnostic mode, the TODR will initially start counting from 0 and be adjusted differently when an OS specifically writes to the TODR.
   VMS determines if the TODR currently contains a valid time if the value it sees is less than about 1 month.
   If the time isn't valid, VMS will prompt to set the time during the system boot.
   While prompting for the time it will wait for an answer to the prompt for up to the ``SYSGEN`` parameter ``TIMEPROMPTWAIT`` seconds.
   A value of 0 for ``TIMEPROMPTWAIT`` will disable the clock setting prompt.

Interval timer (TMR)
""""""""""""""""""""
The interval timer implements the VAX architectural timer, with 1 microsecond intervals.
It has these registers:

========  ====  =================================
Name      Size  Comments
========  ====  =================================
``ICCS``  32    Interval timer control and status
``ICR``   32    Interval count register
``NICR``  32    Next interval count register
``INT``   1     Interrupt request
========  ====  =================================

For standard VMS intervals (10 milliseconds),
the interval timer autocalibrates against real-world time.

Unibus adapter (UBA)
""""""""""""""""""""
The Unibus adapter (UBA) simulates the DW780.
It recognizes these options::

    SET UBA AUTOCONFIGURE         Enable autoconfiguration
    SET UBA NOAUTOCONFIGURE       Disable autoconfiguration

and this ``SHOW`` command::

    SHOW UBA IOSPACE              Display IO address space assignments

The UBA also implements a command to display a Unibus address to physical address translation::

    SHOW UBA VIRTUAL=n            Show translation for Unibus address n

Finally, the UBA implements main memory examination and modification via the Unibus map.
The data width is always 16b::

    EX UBA 0/10                   Examine main memory words corresponding
                                  to Unibus addresses 0-10

The UBA has these registers:

==============  ====  =======================================
Name            Size  Comments
==============  ====  =======================================
``IPL14``       32    Unibus IPL14 interrupt requests
``IPL15``       32    Unibus IPL15 interrupt requests
``IPL16``       32    Unibus IPL16 interrupt requests
``IPL17``       32    Unibus IPL17 interrupt requests
``CNFR``        32    Configuration register
``CR``          32    Control register
``SR``          32    Status register
``DR``          32    Diagnostic register
``INT``         1     Internal UBA interrupt request
``NEXINT``      1     UBA Nexus interrupt request
``AIIP``        1     Adapter initialization in progress flag
``UIIP``        1     Unibus initialization in progress flag
``FMER``        32    Failing memory address
``FUBAR``       32    Failing UBA map register
``BRSVR0``      32    Spare register 0
``BRSVR1``      32    Spare register 1
``BRSVR2``      32    Spare register 2
``BRSVR3``      32    Spare register 3
``BRRVR4``      32    Vector register, IPL 14
``BRRVR5``      32    Vector register, IPL 15
``BRRVR6``      32    Vector register, IPL 16
``BRRVR7``      32    Vector register, IPL 17
``DPR[0:15]``   32    Data path registers 0 to 15
``MAP[0:495]``  32    Map registers 0 to 495
``AITIME``      24    Adapter initialization time
``UITIME``      24    Unibus initialization time
==============  ====  =======================================

Massbus adapters (MBA0, MBA1)
"""""""""""""""""""""""""""""
The Massbus adapters (MBA0, MBA1) simulate RH780s.
MBA0 is assigned to the RP disk drives, MBA1 to the TU tape drives.
Each MBA has these registers:

==============  ====  ===========================
Name            Size  Comments
==============  ====  ===========================
``CNFR``        32    Configuration register
``CR``          32    Control register
``SR``          32    Status register
``VA``          17    Virtual address register
``BC``          32    Byte count register
``DR``          32    Diagnostic register
``SMR``         32    Selected map register
``MAP[0:255]``  32    Map registers
``NEXINT``      1     MBA Nexus interrupt request
==============  ====  ===========================

I/O device addressing
---------------------
Unibus I/O space is not large enough to allow all possible devices to be configured simultaneously at fixed addresses.
Instead, many devices have floating addresses;
that is, the assigned device address depends on the presence of other devices in the configuration:

===================  ===============================================
``DZ11``             All instances have floating addresses
``RL11``             First instance has fixed address, rest floating
``RX11``/``RX211``   First instance has fixed address, rest floating
``DEUNA``/``DELUA``  First instance has fixed address, rest floating
``MSCP`` disk        First instance has fixed address, rest floating
``TMSCP`` tape       First instance has fixed address, rest floating
===================  ===============================================

To maintain addressing consistency as the configuration changes,
the simulator implements DEC's standard I/O address and vector autoconfiguration algorithms for all Unibus devices.
This allows the user to enable or disable devices without needing to manage I/O addresses and vectors.

Autoconfiguration cannot solve address conflicts between devices with overlapping fixed addresses.
For example, with default I/O page addressing, the PDP-11 can support either a TM11 or a TS11,
but not both, since they use the same I/O addresses.

In addition to autoconfiguration, most devices support the ``SET <device> ADDRESS`` command,
which allows the I/O page address of the device to be changed,
and the ``SET <device> VECTOR`` command,
which allows the vector of the device to be changed.
Explicitly setting the I/O address or vector of a device DISABLES autoconfiguration for that device and for the entire system.
As a consequence, the user may have to manually configure all other autoconfigured devices,
because the autoconfiguration algorithm no longer recognizes the explicitly configured device.
A device can be reset to autoconfigure with the ``SET <device> AUTOCONFIGURE`` command.
Autoconfiguration can be restored for the entire system with the ``SET UBA AUTOCONFIGURE`` command.

The current I/O map can be displayed with the ``SHOW UBA IOSPACE`` command.
Addresses that have set by autoconfiguration are marked with an asterisk (\*).

All devices support the ``SHOW <device> ADDRESS`` and ``SHOW <device> VECTOR`` commands,
which display the device address and vector, respectively.

Programmed I/O devices
----------------------

Console input (TTI)
"""""""""""""""""""
The terminal interfaces (TTI, TTO) can be set to one of three modes, ``7P``, ``7B`` or ``8B``:

======  ======================  ==================================
Mode    Input characters        Output characters
======  ======================  ==================================
``7P``  High-order bit cleared  High-order bit cleared,
                                non-printing characters suppressed
``7B``  High-order bit cleared  High-order bit cleared
``8B``  No changes              No changes
======  ======================  ==================================

The default mode is ``8B``.

When the console terminal is attached to a Telnet session, it recognizes ``BREAK``.
If ``BREAK`` is entered, and BDR<7> is set, control returns to the console firmware;
otherwise, ``BREAK`` is treated as a normal terminal input condition.

The terminal input (TTI) polls the console keyboard for input.
It implements these registers:

========  ====  ===========================================
Name      Size  Comments
========  ====  ===========================================
``BUF``   8     Last data item processed
``CSR``   16    Control/status register
``INT``   1     Interrupt pending flag
``ERR``   1     Error flag (CSR<15>)
``DONE``  1     Device done flag (CSR<7>)
``IE``    1     Interrupt enable flag (CSR<6>)
``POS``   32    Number of characters input
``TIME``  24    Input polling interval (if 0, the keyboard
                is polled synchronously with the TODR)
========  ====  ===========================================

Console output (TTO)
""""""""""""""""""""
The terminal output (TTO) writes to the simulator console window.
It implements these registers:

========  ====  ================================
Name      Size  Comments
========  ====  ================================
``BUF``   8     Last data item processed
``CSR``   16    Control/status register
``INT``   1     Interrupt pending flag
``ERR``   1     Error flag (CSR<15>)
``DONE``  1     Device done flag (CSR<7>)
``IE``    1     Interrupt enable flag (CSR<6>)
``POS``   32    Number of characters input
``TIME``  24    Time from I/O initiation to interrupt
========  ====  ================================

RX01 console floppy disk (RXC)
""""""""""""""""""""""""""""""
RX01 options include the ability to set units write enabled or write locked::

    SET RXCn LOCKED               Set unit n write locked
    SET RXCn WRITEENABLED         Set unit n write enabled

The RX01 implements a special command, ``FLOAD``,
for loading VAX executables from an RT11-formatted console floppy disk image::

    FLOAD <file_name> {<origin>}

``FLOAD`` searches the floppy disk image attached to the RX01 for the named file and then loads it into VAX-11/780 memory starting at the origin.
If no origin is specified, the default origin is 200 (hex).

The RX01 implements these registers:

===============  ====  ==============================
Name             Size  Comments
===============  ====  ==============================
``FNC``          8     Function select
``ES``           8     Error status
``ECODE``        8     Error code
``TA``           8     Track address
``SA``           8     Sector address
``STATE``        4     Protocol state
``BPTR``         7     Data buffer pointer
``CTIME``        24    Command initiation delay
``STIME``        24    Seek time delay, per track
``XTIME``        24    Transfer time delay, per byte
``STOP_IOE``     1     Stop on I/O error
``DBUF[0:127]``  8     Data buffer
===============  ====  ==============================

Error handling is as follows:

+---------------+-------------------+-----------------------+
| Error         | ``STOP_IOE``      | Processed as          |
+===============+===================+=======================+
| not attached  | 1                 | Report error and stop |
|               +-------------------+-----------------------+
|               | 0                 | Disk not ready        |
+---------------+-------------------+-----------------------+

RX01 data files are buffered in memory;
therefore, end of file and OS I/O errors cannot occur.

Line printer (LPT)
""""""""""""""""""
The line printer (LPT) writes data to a disk file.
The POS register specifies the number of the next data item to be written.
Thus, by changing POS, the user can backspace or advance the printer.

The line printer implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       8     Last data item processed
``CSR``       16    Control/status register
``INT``       1     Interrupt pending flag
``ERR``       1     Error flag (CSR<15>)
``DONE``      1     Device done flag (CSR<7>)
``IE``        1     Interrupt enable flag (CSR<6>)
``POS``       32    Position in the output file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  =====================================

Error handling is as follows:

+---------------+-------------------+-----------------------+
| Error         | ``STOP_IOE``      | Processed as          |
+===============+===================+=======================+
| not attached  | 1                 | Report error and stop |
|               +-------------------+-----------------------+
|               | 0                 | Out of paper          |
+---------------+-------------------+-----------------------+
| OS I/O error  | x                 | Report error and stop |
+---------------+-------------------+-----------------------+

Disks
-----
All VAX-11/780 disks, and the TUK50 MSCP tape,
support a special form of the ``boot`` command,
with the following syntax::

    BOOT <unit>{/R5:<value>}

For example, ::

    BOOT RP0/R5:1

The optional switch, ``/R5``,
specifies that R5 is to be loaded with the specified value prior to booting.
If the switch is omitted, ``R5`` is loaded with 0.

RP04/05/06/07, RM02/03/05/80 disk pack drives (RP)
""""""""""""""""""""""""""""""""""""""""""""""""""
The RP controller implements the Massbus family of large disk drives.
RP options include the ability to set units write enabled or write locked,
to set the drive type to one of six disk types, or autosize,
and to write a DEC standard 144 compliant bad block table on the last track::

    SET RPn LOCKED                Set unit n write locked
    SET RPn WRITEENABLED          Set unit n write enabled
    SET RPn RM03                  Set type to RM03
    SET RPn RM05                  Set type to RM05
    SET RPn RM80                  Set type to RM80
    SET RPn RP04                  Set type to RP04
    SET RPn RP05                  Set type to RP05
    SET RPn RP06                  Set type to RP06
    SET RPn RP07                  Set type to RP07
    SET RPn AUTOSIZE              Set type based on file size at attach
    SET RPn BADBLOCK              Write bad block table on last track

The type options can be used only when a unit is not attached to a file.
The bad block option can be used only when a unit is attached to a file.
Units can be set ``ENABLED`` or ``DISABLED``.
The RP controller supports the ``BOOT`` command.

The RP controller implements the registers listed below.
Registers suffixed with ``[0:7]`` are replicated per drive.

============  ====  ================================
Name          Size  Comments
============  ====  ================================
``CS1[0:7]``  16    Current operation
``DA[0:7]``   16    Desired surface, sector
``DS[0:7]``   16    Drive status
``ER1[0:7]``  16    Drive errors
``OF[0:7]``   16    Offset
``DC[0:7]``   16    Desired cylinder
``ER2[0:7]``  16    Error status 2
``ER3[0:7]``  16    Error status 3
``EC1[0:7]``  16    ECC syndrome 1
``EC2[0:7]``  16    ECC syndrome 2
``MR[0:7]``   16    Maintenance register
``MR2[0:7]``  16    Maintenance register 2 (RM only)
``HR[0:7]``   16    Holding register (RM only)
``STIME``     24    Seek time, per cylinder
``RTIME``     24    Rotational delay
``STOP_IOE``  1     Stop on I/O error
============  ====  ================================

Error handling is as follows:

+---------------+-------------------+-----------------------------+
| Error         | ``STOP_IOE``      | Processed as                |
+===============+===================+=============================+
| not attached  | 1                 | Report error and stop       |
|               +-------------------+-----------------------------+
|               | 0                 | Disk not ready              |
+---------------+-------------------+-----------------------------+
| End of file   | x                 | Assume rest of disk is zero |
+---------------+-------------------+-----------------------------+
| OS I/O error  | x                 | Report error and stop       |
+---------------+-------------------+-----------------------------+

RL11/RL01/RL02 cartridge disk (RL)
""""""""""""""""""""""""""""""""""
RL11 options include the ability to set units write enabled or write locked,
to set the drive type to RL01, RL02, or autosize,
and to write a DEC standard 144 compliant bad block table on the last track::

    SET RLn LOCKED                Set unit n write locked
    SET RLn WRITEENABLED          Set unit n write enabled
    SET RLn RL01                  Set type to RL01
    SET RLn RL02                  Set type to RL02
    SET RLn AUTOSIZE              Set type based on file size at attach
    SET RLn BADBLOCK              Write bad block table on last track

The type options can be used only when a unit is not attached to a file.
The bad block option can be used only when a unit is attached to a file.
Units can be set ``ENABLED`` or ``DISABLED``.
The RL11 supports the ``BOOT`` command.

The RL11 implements these registers:

==============================  ====  ================================
Name                            Size  Comments
==============================  ====  ================================
``RLCS``                        16    Control/status
``RLDA``                        16    Disk address
``RLBA``                        16    Memory address
``RLBAE``                       6     Memory address extension (RLV12)
``RLMP``, ``RLMP1``, ``RLMP2``  16    Multipurpose register queue
``INT``                         1     Interrupt pending flag
``ERR``                         1     Error flag (CSR<15>)
``DONE``                        1     Device done flag (CSR<7>)
``IE``                          1     Interrupt enable flag (CSR<6>)
``STIME``                       24    Seek time, per cylinder
``RTIME``                       24    Rotational delay
``STOP_IOE``                    1     Stop on I/O error
==============================  ====  ================================

Error handling is as follows:

+---------------+-------------------+-----------------------------+
| Error         | ``STOP_IOE``      | Processed as                |
+===============+===================+=============================+
| not attached  | 1                 | Report error and stop       |
|               +-------------------+-----------------------------+
|               | 0                 | Disk not ready              |
+---------------+-------------------+-----------------------------+
| End of file   | x                 | Assume rest of disk is zero |
+---------------+-------------------+-----------------------------+
| OS I/O error  | x                 | Report error and stop       |
+---------------+-------------------+-----------------------------+

RK611/RK06/RK07 cartridge disk (HK)
"""""""""""""""""""""""""""""""""""
RK611 options include the ability to set units write enabled or write locked,
to set the drive type to RK06, RK07, or autosize,
and to write a DEC standard 144 compliant bad block table on the last track::

    SET HKn LOCKED                Set unit n write locked
    SET HKn WRITEENABLED          Set unit n write enabled
    SET HKn RK06                  Set type to RK06
    SET HKn RK07                  Set type to RK07
    SET HKn AUTOSIZE              Set type based on file size at attach
    SET HKn BADBLOCK              Write bad block table on last track

The type options can be used only when a unit is not attached to a file.
The bad block option can be used only when a unit is attached to a file.
Units can be set ``ENABLED`` or ``DISABLED``.
The RK611 supports the ``BOOT`` command.

The RK611 implements these registers:

===============  ========  ==================================
Name             Size      Comments
===============  ========  ==================================
``HKCS1``        16        Control/status 1
``HKWC``         16        Word count
``HKBA``         16        Bus address
``HKDA``         16        Desired surface, sector
``HKCS2``        16        Control/status 2
``HKDS[0:7]``    16        Drive status, drives 0 to 7
``HKER[0:7]``    16        Drive errors, drives 0 to 7
``HKDB[0:2]``    16        Data buffer silo
``HKDC``         16        Desired cylinder
``HKOF``         8         Offset
``HKMR``         16        Maintenance register
``HKSPR``        16        Spare register
``HKCI``         1         Controller interrupt flop
``HKDI``         1         Drive interrupt flop
``HKEI``         1         Error interrupt flop
``INT``          1         Interrupt pending flag
``ERR``          1         Error flag (CSR<15>)
``DONE``         1         Device done flag (CSR1<7>)
``IE``           1         Interrupt enable flag (CSR1<6>)
``STIME``        24        Seek time, per cylinder
``RTIME``        24        Rotational delay
``MIN2TIME``     24        Minimum time between DONE and ATA
``STOP_IOE``     1         Stop on I/O error
===============  ========  ==================================

Error handling is as follows:

+---------------+-------------------+-----------------------------+
| Error         | ``STOP_IOE``      | Processed as                |
+===============+===================+=============================+
| not attached  | 1                 | Report error and stop       |
|               +-------------------+-----------------------------+
|               | 0                 | Disk not ready              |
+---------------+-------------------+-----------------------------+
| End of file   | x                 | Assume rest of disk is zero |
+---------------+-------------------+-----------------------------+
| OS I/O error  | x                 | Report error and stop       |
+---------------+-------------------+-----------------------------+

UDA50 MSCP disk controllers (RQ, RQB, RQC, RQD)
"""""""""""""""""""""""""""""""""""""""""""""""
The simulator implements four MSCP disk controllers, RQ, RQB, RQC, RQD.
Initially, RQB, RQC, and RQD are disabled.
Each RQ controller simulates an UDA50 MSCP disk controller with four drives.
RQ options include the ability to set units write enabled or write locked,
and to set the drive type to one of many disk types::

    SET RQn LOCKED                Set unit n write locked
    SET RQn WRITEENABLED          Set unit n write enabled
    SET RQn RX50                  Set type to RX50
    SET RQn RX33                  Set type to RX33
    SET RQn RD51                  Set type to RD51
    SET RQn RD52                  Set type to RD52
    SET RQn RD53                  Set type to RD53
    SET RQn RD54                  Set type to RD54
    SET RQn RD31                  Set type to RD31
    SET RQn RA81                  Set type to RA81
    SET RQn RA82                  Set type to RA82
    set RQn RA71                  Set type to RA71
    SET RQn RA72                  Set type to RA72
    SET RQn RA90                  Set type to RA90
    SET RQn RA92                  Set type to RA92
    SET RQn RRD40                 Set type to RRD40 (CD ROM)
    SET RQn RAUSER{=n}            Set type to RA82 with n MBs
    SET -L RQn RAUSER{=n}         Set type to RA82 with n LBNs

The type options can be used only when a unit is not attached to a file.
``RAUSER`` is a "user specified" disk;
the user can specify the size of the disk in either MB (1000000 bytes) or logical block numbers (LBNs, 512 bytes each).
The minimum size is 5MB;
the maximum size is 2GB without extended file support,
1TB with extended file support.

Units can be set ``ENABLED`` or ``DISABLED``.
The RQ controllers support the ``BOOT`` command.

Drive units have changeable unit numbers.
Unit numbers can be changed with::

    SET RQn UNIT=val              Set unit plug value

Each device has 4 units which have unique MSCP unit numbers (0, 1, 2 and 3).

Each RQ controller implements the following special SHOW commands::

    SHOW RQn TYPE                 Show drive type
    SHOW RQ RINGS                 Show command and response rings
    SHOW RQ FREEQ                 Show packet free queue
    SHOW RQ RESPQ                 Show packet response queue
    SHOW RQ UNITQ                 Show unit queues
    SHOW RQ ALL                   Show all ring and queue state
    SHOW RQn UNITQ                Show unit queues for unit n
    SHOW RQn UNIT                 Show unit plug value

Each RQ controller implements these registers:

===============  ====  ======================================
Name             Size  Comments
===============  ====  ======================================
``SA``           16    Status/address register
``S1DAT``        16    Step 1 init host data
``CQBA``         22    Command queue base address
``CQLNT``        8     Command queue length
``CQIDX``        8     Command queue index
``RQBA``         22    Request queue base address
``RQLNT``        8     Request queue length
``RQIDX``        8     Request queue index
``FREE``         5     Head of free packet list
``RESP``         5     Head of response packet list
``PBSY``         5     Number of busy packets
``CFLGS``        16    Controller flags
``CSTA``         4     Controller state
``PERR``         9     Port error number
``CRED``         5     Host credits
``HAT``          17    Host available timer
``HTMO``         17    Host timeout value
``CPKT[0:3]``    5     Current packet, units 0 to 3
``PKTQ[0:3]``    5     Packet queue, units 0 to 3
``UFLG[0:3]``    16    Unit flags, units 0 to 3
``PLUG[0:3]``    16    Unit plug values, units 0 to 3
``INT``          1     Interrupt request
``ITIME``        1     Response time for initialization steps
                       (except for step 4)
``QTIME``        24    Response time for 'immediate' packets
``XTIME``        24    Response time for data transfers
``PKTS[33*32]``  16    Packet buffers, 33W each, 32 entries
===============  ====  ======================================

While VMS is not timing sensitive,
most of the BSD-derived operating systems (NetBSD, OpenBSD, etc) are.
The ``QTIME`` and ``XTIME`` parameters are set to values that allow these operating systems to run correctly.

Error handling is as follows:

==============  ===========================
Error           Processed as
==============  ===========================
not attached    Disk not ready
end of file     Assume rest of disk is zero
OS I/O error    Report error and stop
==============  ===========================

Tapes
-----

TM03/TE16/TU45/TU77 magnetic tapes (TU)
"""""""""""""""""""""""""""""""""""""""
The TU controller implements the Massbus family of 800/1600bpi magnetic tape drives.
TU options include the ability to set the drive type to one of three drives (TE16, TU45, or TU77),
and to set the drives write enabled or write locked. ::

    SET TUn TE16                  Set unit n drive type to TE16
    SET TUn TU45                  Set unit n drive type to TU45
    SET TUn TU77                  Set unit n drive type to TU77
    SET Tun LOCKED                Set unit n write locked
    SET Tun WRITEENABLED          Set unit n write enabled

Magnetic tape units can be set to a specific reel capacity in MB, or to unlimited capacity::

    SET TUn CAPAC=m               Set unit n capacity to m MB (0 = unlimited)
    SHOW TUn CAPAC                Show unit n capacity in MB

Units can be set ``ENABLED`` or ``DISABLED``.
The TU controller does not support the ``BOOT`` command.

The TU controller implements the following registers:

============  ====  ==========================
Name          Size  Comments
============  ====  ==========================
``CS1``       6     Current operation
``FC``        16    Frame count
``FS``        16    Formatter status
``ER``        16    Formatter errors
``CC``        16    Check character
``MR``        16    Maintenance register
``TC``        16    Tape control register
``TIME``      24    Operation execution time
``UST``       17    Unit status, drives 0 to 7
``POS``       32    Position, drive 0 to 7
``STOP_IOE``  1     Stop of I/O error
============  ====  ==========================

Error handling is as follows:

==============  =====================================
Error           Processed as
==============  =====================================
not attached    Tape not ready; if ``STOP_IOE``, stop
end of file     Bad tape
OS I/O error    Parity error; if ``STOP_IOE``, stop
==============  =====================================

TS11 magnetic tape (TS)
"""""""""""""""""""""""
TS options include the ability to make the unit write enabled or write locked. ::

    SET TS LOCKED                 Set unit write locked
    SET TS WRITEENABLED           Set unit write enabled

The TS drive can be set to a specific reel capacity in MB, or to unlimited capacity::

    SET TS0 CAPAC=m               Set capacity to m MB (0 = unlimited)
    SHOW TS0 CAPAC                Show capacity in MB

The TS11 does not support the ``BOOT`` command.

The TS controller implements these registers:

=========  ====  ====================================
Name       Size  Comments
=========  ====  ====================================
``TSSR``   16    Status register
``TSBA``   16    Bus address register
``TSDBX``  16    Data buffer extension register
``CHDR``   16    Command packet header
``CADL``   16    Command packet low address or count
``CADH``   16    Command packet high address
``CLNT``   16    Command packet length
``MHDR``   16    Message packet header
``MRFC``   16    Message packet residual frame count
``MXS0``   16    Message packet extended status 0
``MXS1``   16    Message packet extended status 1
``MXS2``   16    Message packet extended status 2
``MXS3``   16    Message packet extended status 3
``MXS4``   16    Message packet extended status 4
``WADL``   16    Write char packet low address
``WADH``   16    Write char packet high address
``WLNT``   16    Write char packet length
``WOPT``   16    Write char packet options
``WXOPT``  16    Write char packet extended options
``ATTN``   1     Attention message pending
``BOOT``   1     Boot request pending
``OWNC``   1     If set, tape owns command buffer
``OWNM``   1     If set, tape owns message buffer
``TIME``   24    Delay
``POS``    32    Position
=========  ====  ====================================

Error handling is as follows:

==============  ================
Error           Processed as
==============  ================
not attached    Tape not ready
end of file     Bad tape
OS I/O error    Fatal tape error
==============  ================

TUK50 TMSCP tape controller (TQ)
""""""""""""""""""""""""""""""""
The TQ controller simulates the TUK50 TMSCP tape controller.
TQ options include the ability to set units write enabled or write locked,
and to specify the controller type and tape length::

    SET TQn LOCKED                Set unit n write locked
    SET TQn WRITEENABLED          Set unit n write enabled
    SET TQ TK50                   Set controller type to TK50
    SET TQ TK70                   Set controller type to TK70
    SET TQ TU81                   Set controller type to TU81
    SET TQ TKUSER{=n}             Set controller type to TK50 with tape
                                  capacity of n MB

User-specified capacity must be between 50 and 2000 MB.
The TUK50 supports the ``BOOT`` command.

Regardless of the controller type,
individual units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET TQn CAPAC=m               Set unit n capacity to m MB (0 = unlimited)
    SHOW TQn CAPAC                Show unit n capacity in MB

Drive units have changeable unit numbers.
Unit numbers can be changed with::

    SET TQn UNIT=val              Set unit plug value

Device TQ has 4 units (TQ0, TQ1, TQ2 and TQ3) which have unique MSCP unit numbers (0, 1, 2 and 3).

The TQ controller implements the following special ``SHOW`` commands::

    SHOW TQ TYPE                  Show controller type
    SHOW TQ RINGS                 Show command and response rings
    SHOW TQ FREEQ                 Show packet free queue
    SHOW TQ RESPQ                 Show packet response queue
    SHOW TQ UNITQ                 Show unit queues
    SHOW TQ ALL                   Show all ring and queue state
    SHOW TQn UNITQ                Show unit queues for unit n
    SHOW TQn UNIT                 Show unit plug value

The TQ controller implements these registers:

===============  ====  ======================================
Name             Size  Comments
===============  ====  ======================================
``SA``           16    Status/address register
``S1DAT``        16    Step 1 init host data
``CQBA``         22    Command queue base address
``CQLNT``        8     Command queue length
``CQIDX``        8     Command queue index
``RQBA``         22    Request queue base address
``RQLNT``        8     Request queue length
``RQIDX``        8     Request queue index
``FREE``         5     Head of free packet list
``RESP``         5     Head of response packet list
``PBSY``         5     Number of busy packets
``CFLGS``        16    Controller flags
``CSTA``         4     Controller state
``PERR``         9     Port error number
``CRED``         5     Host credits
``HAT``          17    Host available timer
``HTMO``         17    Host timeout value
``CPKT[0:3]``    5     Current packet, units 0 to 3
``PKTQ[0:3]``    5     Packet queue, units 0 to 3
``UFLG[0:3]``    16    Unit flags, units 0 to 3
``PLUG[0:3]``    16    Unit plug values, units 0 to 3
``POS[0:3]``     32    Tape position, units 0 to 3
``OBJP[0:3]``    32    Object position, units 0 to 3
``INT``          1     Interrupt request
``ITIME``        1     Response time for initialization steps
                       (except for step 4)
``QTIME``        24    Response time for 'immediate' packets
``XTIME``        24    Response time for data transfers
``PKTS[33*32]``  16    Packet buffers, 33W each, 32 entries
===============  ====  ======================================

Error handling is as follows:

==============  ================
Error           Processed as
==============  ================
not attached    Tape not ready
end of file     End of medium
OS I/O error    Fatal tape error
==============  ================

Communications devices
----------------------

DZ11 terminal multiplexer (DZ)
""""""""""""""""""""""""""""""
The DZ11 is an 8-line terminal multiplexer.
Up to 32 DZ11s (256 lines) are supported.
The default number of lines is 32.
The number of lines can be changed with the command ::

    SET DZ LINES=n                Set line count to n

The line count must be a multiple of 8, with a maximum of 256.

The DZ11 supports three character processing modes, ``7P``, ``7B``, and ``8B``:

======  ======================  ==================================
Mode    Input characters        Output characters
======  ======================  ==================================
``7P``  High-order bit cleared  High-order bit cleared,
                                non-printing characters suppressed
``7B``  High-order bit cleared  High-order bit cleared
``8B``  No changes              No changes
======  ======================  ==================================

The default is ``8B``.

The DZ11 supports logging on a per-line basis.
The command ::

    SET DZ LOG=line=filename

enables logging for the specified line to the indicated file.
The command ::

    SET DZ NOLOG=line

disables logging for the specified line and closes any open log file.
Finally, the command ::

    SHOW DZ LOG

displays logging information for all DZ lines.

The terminal lines perform input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the port to be used::

    ATTACH {-am} DZ <port>        Set up listening port

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.
The optional switch ``-m`` turns on the DZ11's modem controls;
the optional switch ``-a`` turns on active disconnects
(disconnect session if computer clears Data Terminal Ready).
Without modem control, the DZ behaves as though terminals were directly connected;
disconnecting the Telnet session does not cause any operating system-visible change in line status.

Once the DZ is attached and the simulator is running,
the DZ will listen for connections on the specified port.
It assumes that the incoming connections are Telnet connections.
The connection remains open until disconnected by the simulated program,
the Telnet client, a ``SET DZ DISCONNECT`` command, or a ``DETACH DZ`` command.

Other special DZ commands::

    SHOW DZ CONNECTIONS           Show current connections
    SHOW DZ STATISTICS            Show statistics for active connections
    SET DZ DISCONNECT=linenumber  Disconnects the specified line

The DZ11 implements these registers:

==============  ====  ============================================
Name            Size  Comments
==============  ====  ============================================
``CSR[0:3]``    16    Control/status register, boards 0 to 3
``RBUF[0:3]``   16    Receive buffer, boards 0 to 3
``LPR[0:3]``    16    Line parameter register, boards 0 to 3
``TCR[0:3]``    16    Transmission control register, boards 0 to 3
``MSR[0:3]``    16    Modem status register, boards 0 to 3
``TDR[0:3]``    16    Transmit data register, boards 0 to 3
``SAENB[0:3]``  1     Silo alarm enabled, boards 0 to 3
``RXINT``       4     Receive interrupts, boards 3 to 0
``TXINT``       4     Transmit interrupts, boards 3 to 0
``MDMTCL``      1     Modem control enabled
``AUTODS``      1     Autodisconnect enabled
==============  ====  ============================================

The DZ11 does not support save and restore.
All open connections are lost when the simulator shuts down or the DZ is detached.

DHU11 terminal multiplexer (VH)
"""""""""""""""""""""""""""""""
The DHU11 is an 16-line terminal multiplexer for Unibus systems.
Up to 4 DHU11s are supported.

The number of active DHU11 devices is configured by setting the number of multiplexer lines desired.
The number of DHU11 lines can be set as follows::

    SET VH LINES=n                n must be a multiple of 16

The DHU11 is a programmable asynchronous terminal multiplexer.
The register sets are compatible with these devices.
For transmission, the DHU11 can be used in either DMA or programmed I/O mode.
For reception, there is a 256-entry FIFO for received characters, dataset status changes, and diagnostic information, and a programmable input interrupt timer.
The device supports 16-, 18-, and 22-bit addressing.
The DHU11 can be programmed to filter and/or handle XON/XOFF characters independently of the processor.
The DHU11 supports programmable bit-width (between 5 and 8) for the input and output of characters.

The VH controller may be adjusted on a per controller basis as follows::

    SET VHn DHU                   Use the DHU programming mode and registers
    SET VHn DHV                   Use the DHV programming mode and registers

DMA output is supported.
In a real DHU11, DMA is not initiated immediately upon receipt of ``TX.DMA.START`` but is dependent upon some internal processes.
The VH controller mimics this behavior by default.
It may be desirable to alter this and start immediately,
though this may not be compatible with all operating systems and diagnostics.
You can change the behavior of the VH controller as follows::

    SET VHn NORMAL                Use normal DMA procedures
    SET VHn FASTDMA               Set DMA to initiate immediately

The terminal lines perform input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the TCP listening port to be used::

    ATTACH VH <port>              Set up listening port

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.
This port is the point of entry for al lines on all VH controllers.

Modem and auto-disconnect support may be set on an individual controller basis.
The ``SET MODEM`` command directs the controller to report modem status changes to the computer.
The ``SET HANGUP`` command turns on active disconnects
(disconnect session if computer clears Data Terminal Ready). ::

    SET VHn [NO]MODEM             Disable/enable modem control
    SET VHn [NO]HANGUP            Disable/enable disconnect on DTR drop

Once the VH is attached and the simulator is running,
the VH will listen for connections on the specified port.
It assumes that the incoming connections are Telnet connections.
The connection remains open until disconnected by the simulated program,
the Telnet client, a ``SET VH DISCONNECT`` command, or a ``DETACH VH`` command.

Other special VH commands::

    SHOW VH CONNECTIONS           Show current connections
    SHOW VH STATISTICS            Show statistics for active connections
    SET VH DISCONNECT=linenumber  Disconnects the specified line

The DHU11 implements these registers,
though not all can be examined from SCP:

==============  ====  ======================================
Name            Size  Comments
==============  ====  ======================================
``CSR[0:3]``    16    Control/status register, boards 0 to 3
``RBUF[0:3]``   16    Receive buffer, boards 0 to 3
``LPR[0:3]``    16    Line parameter register, boards 0 to 3
``RXINT``       4     Receive interrupts, boards 3..0
``TXINT``       4     Transmit interrupts, boards 3..0
==============  ====  ======================================

The DHU11 partially supports save and restore.
Restoring a saved configuration will reestablish the listening port configuration,
however all active sessions will be lost.
All open connections are lost when the simulator shuts down or the VH is detached.

DELUA/DEUNA Unibus Ethernet controllers (XU, XUB)
"""""""""""""""""""""""""""""""""""""""""""""""""
The simulator implements two DELUA/DEUNA Unibus Ethernet controllers (XU, XUB).
Initially, both XU and XQB are disabled.
Options allow control of the MAC address, the controller mode, and the sanity timer. ::

    SET  XU MAC=<mac-address>     ex. 08-00-2B-CC-DD-EE
    SHOW XU MAC

These commands are used to change or display the MAC address.
``<mac-address>`` is a valid Ethernet MAC, delimited by dashes or periods.
The controller defaults to ``08-00-2B-CC-DD-EE``,
which should be sufficient if there is only one SIMH controller on your LAN.
Two cards with the same MAC address will see each other's packets,
resulting in a serious mess. ::

    SET  XU TYPE={DEUNA|[DELUA]}
    SHOW XU TYPE

These commands are used to change or display the controller mode.
DELUA mode is better and faster but may not be usable by older or non-DEC OSes. ::

    SHOW XU STATS

This command will display the accumulated statistics for the simulated Ethernet controller. ::

    SHOW XU FILTERS

This command will display the MAC addresses which the controller will receive packets addressed to.

To access the network,
the simulated Ethernet controller must be attached to a real Ethernet interface::

    ATTACH XU {ethX|<device_name>}      ex. eth0 or /dev/era0
    SHOW XU ETH

where ``X`` in ``ethX`` is the number of the Ethernet controller to attach, or the real device name.
The ``X`` number is system-dependent.
If you only have one Ethernet controller, the number will probably be 0.
To find out what your system thinks the Ethernet numbers are, use the ``SHOW XQ ETH`` command.
The device list can be quite cryptic, depending on the host system, but is probably better than guessing.
If you do not attach the device, the controller will behave as though the Ethernet cable were unplugged.

One final note: because of its asynchronous nature,
the XU controller is not limited to the ~1.5Mbit/sec of the real DEUNA/DELUA controllers,
nor the 10Mbit/sec of a standard Ethernet.
Attach it to a Fast Ethernet (100 Mbit/sec) card, and "Feel the Power!" 😀

DMC11/DMR11 Unibus DDCMP controllers
""""""""""""""""""""""""""""""""""""
The DMC11 is a communication subsystem which consists of a microprocessor based,
intelligent synchronous communications controller which employs the DIGITAL Data Communications Message Protocol (DDCMP).

Hardware description
''''''''''''''''''''
The DMC11 consists of a microprocessor module and a synchronous line unit module.

Models
``````
There were a number of microprocessor controlled DDCMP devices produced.

DMC11
+++++
The original kmc11 microprocessor board with DMC microcode and a sync line unit.

DMR11
+++++
The more advanced kmc11 microprocessor board with DMR microcode and a sync line unit.

DMP11
+++++
A newly-designed Unibus board with a more complete programming interface and a sync line unit.

DMV11
+++++
A Qbus version of the DMP11 with some more advanced refinements and a sync line unit.

Registers
`````````
The DMC device implements these registers:

=====================  ====  =========================
Name                   Size  Description
=====================  ====  =========================
``DMC RXINT``          32    Input interrupt summary
``DMC TXINT``          32    Output interrupt summary
``DMC POLL``           32    Connect poll interval
``DMC SEL0[0:7]``      16    Select 0 CSR
``DMC SEL2[0:7]``      16    Select 2 CSR
``DMC SEL4[0:7]``      16    Select 4 CSR
``DMC SEL6[0:7]``      16    Select 6 CSR
``DMC SPEED[0:7]``     32    Line speed
``DMC PEER[0:5119]``   8     Peer address:port
``DMC PORT[0:5119]``   8     Listen port
``DMC BASEADDR[0:7]``  32    Program set base address
``DMC BASESIZE[0:7]``  16    Program set base size
``DMC MODEM[0:7]``     8     Modem control bits
=====================  ====  =========================

These registers contain the emulated state of the device.
These values don't necessarily relate to any detail of the original device being emulated but are merely internal details of the emulation.

Configuration
'''''''''''''
A DMC device is configured with various SIMH ``SET`` and ``ATTACH`` commands.

``Set`` commands
````````````````
=============================================================================  =============================================
``set DMC LINES=n``                                                            Sets the number of DMC devices being emulated
``set DMC CONNECTPOLL=seconds``                                                Sets the connection poll frequency
``set DMC ADDRESS=val``                                                        Bus address
``set DMC VECTOR=val``                                                         Interrupt vector
``set DMC ENABLE``                                                             Enables device DMC
``set DMC DISABLE``                                                            Disables device DMC
``set DMC DEBUG``                                                              Enables debugging for device DMC
``set DMC NODEBUG``                                                            Disables debugging for device DMC
``set DMC DEBUG=TRACE;WARN;REG;INTREG;INFO;DATA;DATASUM;MODEM;CONNECT;INT``    Enables specific debugging for device DMC
``set DMC NODEBUG=TRACE;WARN;REG;INTREG;INFO;DATA;DATASUM;MODEM;CONNECT;INT``  Disables specific debugging for device DMC
``set DMCn PEER=address:port``
``set DMCn SPEED=bits/sec (0=unrestricted)``
``set DMCn TYPE={DMR,DMC}``
``set DMCn STATS``                                                             Display/Clear statistics
=============================================================================  =============================================

Lines
+++++
A maximum of 8 DMC11 devices can be emulated concurrently in the VAX 11/780 simulator.
The number of simulated DMC11 devices or lines can be specified with command::

    sim> SET DMC0 LINES=n

Peer
++++
To set the host and port to which data is to be transmitted use the following command::

    sim> SET DMC0 PEER=host:port

Connectpoll
+++++++++++
The minimum interval between attempts to connect to the other side is set using the following command::

    sim> SET DMC0 CONNECTPOLL=n

Where ``n`` is the number of seconds.
The default is 2 seconds.

Speed
+++++
If you want to experience the actual data rates of the physical hardware you can set the bit rate of the simulated line can be set using the following command::

    sim> SET DMC0 SPEED=n

Where ``n`` is the number of data bits per second that the simulated line runs at.
In practice this is implemented as a delay while transmitting bytes to the socket.
Use a value of zero to run at full speed with no artificial throttling.

Type
++++
The type of device being emulated can be changed with the following command::

    sim> SET DMC0 TYPE={DMR,DMC}

A ``SET TYPE`` command should be entered before the device is attached to a listening port.

Attach
``````
The device must be attached to a receive port,
use the ``ATTACH`` command specifying the receive port number. ::

    sim> ATTACH DMC0 port

The Peer host:port value must be specified before the ``attach`` command.

Examples
````````
To configure two simulators to talk to each other use the following example:

    Machine 1::

        sim> SET DMC ENABLE
        sim> SET DMC0 PEER=LOCALHOST:2222
        sim> ATTACH DMC0 1111

    Machine 2::

        sim> SET DMC ENABLE
        sim> SET DMC0 PEER=LOCALHOST:1111
        sim> ATTACH DMC0 2222

Monitoring
''''''''''
The DMC device and DMC0 line configuration and state can be displayed with one of the available ``show`` commands.

Show commands
`````````````

========================  ========================================
``show DMC LINES``        Display number of devices
``show DMC CONNECTPOLL``  Display connection poll interval
``show DMC ADDRESS``      Bus address
``show DMC VECTOR``       Interrupt vector
``show DMC DEBUG``        Displays debugging status for device DMC
``show DMCn PEER``        Display destination/source
``show DMCn SPEED``       Display rate limit
``show DMCn TYPE``        Set/Display device type
``show DMCn STATS``       Display/Clear statistics
``show DMCn QUEUES``      Display Queue state
``show DMCn DDCMP``       Display DDCMP state information
========================  ========================================

Restrictions
''''''''''''
Real hardware synchronous connections could operate in Multi-Point mode.
Multi-Point mode was a way of sharing a single wire with multiple destination systems or devices.
Multi-Point mode is not currently emulated by this or other simulated synchronous devices.

In real hardware, the DMC11 spoke a version of DDCMP which peer devices needed to be aware of.
The DMR11, DMP11, and DMV11 boards have configuration switches or programatic methods to indicate that the peer device was a DMC11.
The emulated devices all speak the same level of DDCMP so no special remote device awareness need be considered.

Implementation
''''''''''''''
A real DMC11 transports data using DDCMP via a synchronous connection,
the emulated device makes a TCP/IP connection to another emulated device which either speaks DDCMP over the TCP connection directly,
or interfaces to a simulated computer where the operating system speaks the DDCMP protocol on the wire.

The DMC11 can be used for point-to-point DDCMP connections carrying DECnet and other types of networking,
e.g., from ULTRIX or DSM.

Debugging
'''''''''
The simulator has a number of debug options, these are:

===========  ==============================================================================
``REG``      Shows whenever a CSR is programatically read or written and the current value.
``INTREG``   Shows internal register value changes.
``INFO``     Shows higher-level tracing only.
``WARN``     Shows any warnings.
``TRACE``    Shows more detailed trace information.
``DATASUM``  Brief summary of each received and transmitted buffer. Ignored if DATA is set.
``DATA``     Shows the actual data sent and received.
``MODEM``    Shows modem signal transitions details.
``CONNECT``  Shows sockets connection activity.
``INT``      Shows Interrupt activity.
===========  ==============================================================================

To get a full trace use ::

    sim> SET DMC DEBUG

However it is recommended to use the following when sending traces::

    sim> SET DMC DEBUG=REG;INFO;WARN

Related devices
'''''''''''''''
The DMC can facilitate communication with other SIMH simulators which have emulated synchronous network devices available.
These include the following:

==============  =================================================
``DUP11`` [1]_  Unibus PDP11 simulators
``DPV11`` [1]_  Qbus PDP11 simulators
``KDP11`` [1]_  Unibus PDP11 simulators and PDP10 simulators
``DMR11``       Unibus PDP11 simulators and Unibus VAX simulators
``DMC11``       Unibus PDP11 simulators and Unibus VAX simulators
``DMP11``       Unibus PDP11 simulators and Unibus VAX simulators
``DMV11``       Qbus VAX simulators
==============  =================================================

.. [1] Indicates systems which have OS provided DDCMP implementations.
       The DMC11 is a synchronous serial point-to-point communications device.
       A real DMC11 transports data using DDCMP,
       the emulated device makes a TCP/IP connection to another emulated device and sends length-prefixed messages across the connection,
       each message representing a single buffer passed to the DMC11.
       The DMC11 can be used for point-to-point DDCMP connections carrying DECnet and other types of networking,
       e.g., from ULTRIX or DSM.

CR11 card reader (CR)
'''''''''''''''''''''
The card reader (CR) implements a single controller (the CR11) and card reader (e.g., Documation M200, GDI Model 100) by reading a file and presenting lines or cards to the simulator.
Card decks may be represented by plain text ASCII files, card image files, or column binary files.
The CR11 controller is also compatible with the CM11-F, CME11, and CMS11.

Card image files are a file format designed by Douglas W. Jones at the University of Iowa to support the interchange of card deck data.
These files have a much richer information carrying capacity than plain ASCII files.
Card Image files can contain such interchange information as card-stock color, corner cuts, special artwork, as well as the binary punch data representing all 12 columns.
Complete details on the format, as well as sample code, are available at `Prof. Jones's site <http://homepage.divms.uiowa.edu/~jones/cards/>`_.

Examples of the CR11 include the M8290 and M8291 (CMS11).
All card readers use a common vector at 0230 and CSR at 177160.
Even though the CR11 is normally configured as a BR6 device,
it is configured for BR4 in this simulation.

The card reader supports ASCII, card image, and column binary format card "decks".
When reading plain ASCII files, lines longer than 80 characters are silently truncated.
Card image support is included for 80 column Hollerith, 82 column Hollerith (silently ignoring columns 0 and 81), and 40 column Hollerith (mark-sense) cards.
Column binary supports 80 column card images only.
All files are attached read-only (as if the ``-R`` switch were given). ::

    ATTACH -A CR <file>           File is ASCII text
    ATTACH -B CR <file>           File is column binary
    ATTACH -I CR <file>           File is card image format

If no flags are given, the file extension is evaluated.
If the filename ends in ``.TXT``, the file is treated as ASCII text.
If the filename ends in ``.CBN``, the file is treated as column binary.
Otherwise, the CR driver looks for a card image header.
If a correct header is found the file is treated as card image format,
otherwise it is treated as ASCII text.

The correct character translation MUST be set if a plain-text file is to be used for card deck input.
The correct translation SHOULD be set to allow correct ASCII debugging of a card image or column binary input deck.
Depending upon the operating system in use, how it was generated, and how the card data will be read and used,
the translation must be set correctly so that the proper character set is used by the driver.
Use the following command to explicitly set the correct translation::

    SET TRANSLATION={DEFAULT|026|026FTN|029|EBCDIC}

This command should be given after a deck is attached to the simulator.
The mappings above are completely described at http://homepage.divms.uiowa.edu/~jones/cards/codes.html.
Note that DEC typically used 029 or 026FTN mappings.

DEC operating systems used a variety of methods to determine the end of a deck
(recognizing that 'hopper empty' does not necessarily mean the end of a deck).
Below is a summary of the various operating system conventions for signaling end of deck:

==========  ========================================================
RT-11:      ``12-11-0-1-6-7-8-9`` punch in column 1
RSTS/E:     ``12-11-0-1`` or ``12-11-0-1-6-7-8-9`` punch in column 1
RSX:        ``12-11-0-1-6-7-8-9`` punch
VMS:        ``12-11-0-1-6-7-8-9`` punch in first 8 columns
TOPS:       ``12-11-0-1`` or ``12-11-0-1-6-7-8-9`` punch in column 1
==========  ========================================================

Using the ``AUTOEOF`` setting,
the card reader can be set to automatically generate an EOF card consisting of the ``12-11-0-1-6-7-8-9`` punch in columns 1-8.
When set to CD11 mode,
this switch also enables automatic setting of the EOF bit in the controller after the EOF card has been processed.
[The CR11 does not have a similar capability].
By default ``AUTOEOF`` is enabled. ::

    SET CR AUTOEOF
    SET CR NOAUTOEOF

The default card reader rate for the CR11 is 285 cpm.
The reader rate can be set to its default value or to anywhere in the range 200 to 1200 cpm.
This rate may be changed while the unit is attached. ::

    SET CR RATE={DEFAULT|200 to 1200}

It is standard operating procedure for operators to load a card deck and press the momentary action RESET button to clear any error conditions and alert the processor that a deck is available to read.
Use the following command to simulate pressing the card reader RESET button::

    SET CR RESET

Another common control of physical card readers is the STOP button.
An operator could use this button to finish the read operation for the current card and terminate reading a deck early.
Use the following command to simulate pressing the card reader STOP button::

    SET CR STOP

The simulator does not support the ``BOOT`` command.
The simulator does not stop on file I/O errors.
Instead the controller signals a reader check to the CPU.

The CR controller implements these registers:

==============  ====  ====================================
Name            Size  Comments
==============  ====  ====================================
``BUF``         8     ASCII value of last column processed
``CRS``         16    CR11 status register
``CRB1``        16    CR11 12-bit Hollerith character
``CRB2``        16    CR11 8-bit compressed character
``CRM``         16    CR11 maintenance register
``CDST``        16    CD11 control/status register
``CDCC``        16    CD11 column count
``CDBA``        16    CD11 current bus address
``CDDB``        16    CD11 data buffer, 2nd status
``BLOWER``      2     Blower state value
``INT``         1     Interrupt pending flag
``ERR``         1     Error flag (CRS<15>)
``IE``          1     Interrupt enable flag (CRS<6>)
``POS``         32    File position - do not alter
``TIME``        24    Delay time between columns
==============  ====  ====================================

Symbolic display and input
==========================
The VAX simulator implements symbolic display and input.
Display is controlled by command-line switches:

.. table::
   :class: switches

   ================  ====================================
   ``-a``, ``-c``    Display as ASCII data
   ``-m``            Display instruction mnemonics
   ``-p``            Display compatibility mode mnemonics
   ``-r``            Display RADIX50 encoding
   ================  ====================================

Input parsing is controlled by the first character typed in or by command-line switches:

.. table::
   :class: switches

   ===============  =======================================
   ``'`` or ``-a``  ASCII characters (determined by length)
   ``"`` or ``-c``  ASCII string (maximum 60 characters)
   ``-p``           Compatibility mode instruction mnemonic
   Alphabetic       Instruction mnemonic
   Numeric          Octal number
   ===============  =======================================

VAX instruction input uses standard VAX assembler syntax.
Compatibility mode instruction input uses standard PDP-11 assembler syntax.

The syntax for VAX specifiers is as follows:

=====================  =========  ============  =================================
Syntax                 Specifier  Displacement  Comments
=====================  =========  ============  =================================
``#s^n``, ``#n``       ``0n``     —             Short literal, integer only
``[Rn]``               ``4n``     —             Indexed, second specifier follows
``Rn``                 ``5n``     —             PC illegal
``(Rn)``               ``6n``     —             PC illegal
``-(Rn)``              ``7n``     —             PC illegal
``(Rn)+``              ``8n``     —
``#i^n``, ``#n``       ``8F``     ``n``         Immediate
``@(Rn)+``             ``9n``     —
``@#addr``             ``9F``     ``addr``      Absolute
``{+/-}b^d(Rn)``       ``An``     ``{+/-}d``    Byte displacement
``b^d``                ``AF``     ``d - PC``    Byte PC relative
``@{+/-}b^d(Rn)``      ``Bn``     ``{+/-}d``    Byte displacement deferred
``@b^d``               ``BF``     ``d - PC``    Byte PC relative deferred
``{+/-}w^d(Rn)``       ``Cn``     ``{+/-}d``    Word displacement
``w^d``                ``CF``     ``d - PC``    Word PC relative
``@{+/-}w^d(Rn)``      ``Dn``     ``{+/-}d``    Word displacement deferred
``@w^d``               ``DF``     ``d - PC``    Word PC relative deferred
``{+/-}l^d(Rn)``       ``En``     ``{+/-}d``    Long displacement
``l^d``                ``EF``     ``d - PC``    Long PC relative
``@{+/-}l^d(Rn)``      ``Fn``     ``{+/-}d``    Long displacement deferred
``@l^d``               ``FF``     ``d - PC``    Long PC relative deferred
=====================  =========  ============  =================================

If no override is given for a literal (``s^`` or ``i^``) or for a displacement or PC relative address (``b^``, ``w^``, or ``l^``),
the simulator chooses the mode automatically.
