.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

********************************
     PDP-11 simulator usage
********************************
:Date: 2021-08-15
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

This memorandum documents the DEC PDP-11 simulator.

Simulator files
===============
To compile the PDP-11, you must define ``VM_PDP11`` as part of the compilation command-line.
If you want expanded file support, you must also define ``USE_INT64`` and ``USE_ADDR64`` as part of the compilation command-line.

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

``sim/pdp11/``
    | ``pdp11_cpumod.h``
    | ``pdp11_cr_dat.h``
    | ``pdp11_defs.h``
    | ``pdp11_mscp.h``
    | ``pdp11_uqssp.h``
    | ``pdp11_xq.h``
    | ``pdp11_xq_bootrom.h``
    | ``pdp11_cpu.c``
    | ``pdp11_cpumod.c``
    | ``pdp11_cr.c``
    | ``pdp11_dc.c``
    | ``pdp11_dl.c``
    | ``pdp11_dz.c``
    | ``pdp11_fp.c``
    | ``pdp11_hk.c``
    | ``pdp11_ke.c``
    | ``pdp11_kg.c``
    | ``pdp11_io.c``
    | ``pdp11_lp.c``
    | ``pdp11_pclk.c``
    | ``pdp11_pt.c``
    | ``pdp11_rc.c``
    | ``pdp11_rf.c``
    | ``pdp11_rh.c``
    | ``pdp11_rk.c``
    | ``pdp11_rl.c``
    | ``pdp11_rp.c``
    | ``pdp11_rq.c``
    | ``pdp11_rx.c``
    | ``pdp11_ry.c``
    | ``pdp11_stddev.c``
    | ``pdp11_sys.c``
    | ``pdp11_ta.c``
    | ``pdp11_tc.c``
    | ``pdp11_tm.c``
    | ``pdp11_tq.c``
    | ``pdp11_ts.c``
    | ``pdp11_tu.c``
    | ``pdp11_vh.c``
    | ``pdp11_xq.c``
    | ``pdp11_xu.c``

PDP-11 features
===============
The PDP-11 simulator is configured as follows:

======================  =================================================================
Device name(s)          Simulates
======================  =================================================================
``CPU``                 PDP-11 CPU with 256KB of memory
``PTR``, ``PTP``        PC11 paper tape reader/punch
``TTI``, ``TTO``        DL11 console terminal
``CR``                  CR11/CD11 card reader
``LPT``                 LP11 line printer
``CLK``                 KW11-L line frequency clock
``PCLK``                KW11-P programmable clock
``DCI``, ``DCO``        DC11 additional serial lines (up to 16)
``DLI``, ``DLO``        KL11/DL11 additional serial lines (up to 16)
``DZ``                  DZ11 8-line terminal multiplexer (up to 4)
``VH``                  DHU11/DHQ11 8-line terminal multiplexer (up to 4)
``RK``                  RK11/RK05 cartridge disk controller with eight drives
``HK``                  RK611/RK06,RK07 cartridge disk controller with eight drives
``RC``                  RC11 fixed head disk
``RF``                  RF11/RS11 fixed head disk
``RL``                  RL11(RLV12)/RL01,RL02 cartridge disk controller with four drives
``RH``                  RH11/RH70 Massbus adapter (up to 3)
``RP``                  RP04/05/06/07, RM02/03/05/80 Massbus disks with eight drives
``RQ``                  RQDX3/UDA50 MSCP controller with four drives
``RQB``                 Second RQDX3/UDA50 MSCP controller with four drives
``RQC``                 Third RQDX3/UDA50 MSCP controller with four drives
``RQD``                 Fourth RQDX3/UDA50 MSCP controller with four drives
``RX``                  RX11/RX01 floppy disk controller with two drives
``RY``                  RX211/RX01 floppy disk controller with two drives
``TA``                  TA11/TU60 cassette controller with two drives
``TC``                  TC11/TU56 DECtape controller with eight drives
``TM``                  TM11/TU10 magnetic tape controller with eight drives
``TS``                  TS11/TSV05 magnetic tape controller with one drive
``TQ``                  TQK50/TU81 TMSCP magnetic tape controller with four drives
``TU``                  TM02/TM03 magnetic tape formatter with eight drives
``XQ``                  DELQA/DEQNA Qbus Ethernet controller
``XQB``                 Second DELQA/DEQNA Qbus Ethernet controller
``XU``                  DELUA/DEUNA Unibus Ethernet controller
``XUB``                 Second DELUA/DEUNA Unibus Ethernet controller
``KE``                  KE11A extended arithmetic option
``KG``                  KG11A communications arithmetic option
======================  =================================================================

The DZ, VH, CR, LPT, DCI/DCO, DLI/DLO, RK, HK, RC, RF, RL, RP, RQ, RQB, RQC, RQD, RX, RY, TA, TC, TM, TS, TQ, XQ, XQB, XU, XUB, KE, and KG devices can be set ``DISABLED``.
DCI/DCO, DLI/DLO, RC, RF, RQB, RQC, RQD, RY, TA, TS, VH, XQB, XU, XUB, KE, and KG are disabled by default.

The PDP-11 simulator implements several unique stop conditions:

- Abort during exception vector fetch, and register ``STOP_VEC`` is set
- Abort during exception stack push, and register ``STOP_SPA`` is set
- Trap condition ``n`` occurs, and register ``STOP_TRAP<n>`` is set
- Wait state entered, and no I/O operations outstanding (i.e., no interrupt can ever occur)
- A simulated DECtape runs off the end of its reel, and flag ``STOP_OFFR`` is set

The ``LOAD`` command supports standard binary format tapes.
The ``DUMP`` command is not implemented.

CPU and system
--------------

CPU
"""
The CPU options include CPU type, CPU instruction set options for the specified type, and the size of main memory. ::

    SET CPU 11/03                 Set CPU type to 11/03
    SET CPU 11/04                 Set CPU type to 11/04
    SET CPU 11/05                 Set CPU type to 11/05
    SET CPU 11/20                 Set CPU type to 11/20
    SET CPU 11/23                 Set CPU type to 11/23
    SET CPU 11/23+                Set CPU type to 11/23+
    SET CPU 11/24                 Set CPU type to 11/24
    SET CPU 11/34                 Set CPU type to 11/34
    SET CPU 11/40                 Set CPU type to 11/40
    SET CPU 11/44                 Set CPU type to 11/44
    SET CPU 11/45                 Set CPU type to 11/45
    SET CPU 11/53                 Set CPU type to 11/53
    SET CPU 11/60                 Set CPU type to 11/60
    SET CPU 11/70                 Set CPU type to 11/70
    SET CPU 11/73                 Set CPU type to 11/73
    SET CPU 11/73B                Set CPU type to 11/73B
    SET CPU 11/83                 Set CPU type to 11/83
    SET CPU 11/84                 Set CPU type to 11/84
    set CPU 11/93                 Set CPU type to 11/93
    set CPU 11/94                 Set CPU type to 11/94
    SET CPU U18                   Deprecated; same as 11/45
    SET CPU URH11                 Deprecated; same as 11/84
    SET CPU URH70                 Deprecated; same as 11/70
    SET CPU Q22                   Deprecated; same as 11/73
    SET CPU NOEIS                 Disable EIS instructions
    SET CPU EIS                   Enable EIS instructions
    SET CPU NOFIS                 Disable FIS instructions
    SET CPU FIS                   Enable FIS instructions
    SET CPU NOFPP                 Disable FPP instructions
    SET CPU FPP                   Enable FPP instructions
    SET CPU NOCIS                 Disable CIS instructions
    SET CPU CIS                   Enable CIS instructions
    SET CPU NOBEVENT              Disable BEVENT interrupt
    SET CPU BEVENT                Enable BEVENT interrupt
    SET CPU NOMMU                 Disable MMU functionality
    SET CPU MMU                   Enable MMU functionality
    SET CPU 16K                   Set memory size = 16KB
    SET CPU 32K                   Set memory size = 32KB
    SET CPU 48K                   Set memory size = 48KB
    SET CPU 64K                   Set memory size = 64KB
    SET CPU 96K                   Set memory size = 96KB
    SET CPU 128K                  Set memory size = 128KB
    SET CPU 192K                  Set memory size = 192KB
    SET CPU 256K                  Set memory size = 256KB
    SET CPU 384K                  Set memory size = 384KB
    SET CPU 512K                  Set memory size = 512KB
    SET CPU 768K                  Set memory size = 768KB
    SET CPU 1024K (or 1M)         Set memory size = 1024KB
    SET CPU 2048K (or 2M)         Set memory size = 2048KB
    SET CPU 3072K (or 3M)         Set memory size = 3072KB
    SET CPU 4096K (or 4M)         Set memory size = 4096KB

The CPU types and their capabilities are shown in the following table:

=======  ====  ====  ====  =====  ====  ====  ====  ====  =======
Type     Bus   Mem   MMU?  Umap?  EIS?  FIS?  FPP?  CIS?  BEVENT?
=======  ====  ====  ====  =====  ====  ====  ====  ====  =======
11/03    Q     64K   no    no     std   opt   no    no    opt
11/04    U     64K   no    no     no    no    no    no    no
11/05    U     64K   no    no     no    no    no    no    no
11/20    U     64K   no    no     no    no    no    no    no
11/23    Q     4M    std   no     std   no    opt   opt   opt
11/23+   Q     4M    std   no     std   no    opt   opt   no
11/24    U     4M    std   std    std   no    opt   opt   no
11/34    U     256K  std   no     std   no    opt   no    no
11/40    U     256K  std   no     std   opt   no    no    no
11/44    U     4M    std   std    std   no    opt   opt   no
11/45    U     256K  std   no     std   no    opt   no    no
11/53    Q     4M    std   no     std   no    std   opt   no
11/60    U     256K  std   no     std   no    std   no    no
11/70    U     4M    std   std    std   no    opt   no    no
11/73    Q     4M    std   no     std   no    std   opt   no
11/73B   Q     4M    std   no     std   no    std   opt   no
11/83    Q     4M    std   no     std   no    std   opt   no
11/84    U     4M    std   std    std   no    std   opt   no
11/93    Q     4M    std   no     std   no    std   opt   no
11/94    U     4M    std   std    std   no    std   opt   no
=======  ====  ====  ====  =====  ====  ====  ====  ====  =======

If a capability is standard, it cannot be disabled;
if a capability is not included, it cannot be enabled.

The CPU implements a ``SHOW`` command to display the I/O address assignments::

    SHOW CPU IOSPACE              Show I/O space address assignments

If memory size is being reduced, and the memory being truncated contains non-zero data, the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 256KB.
If memory size is increased to more than 256KB, or the bus structure is changed, the simulator disables peripherals that can't run in the current bus structure.

These switches are recognized when examining or depositing in CPU memory:

-v     Interpret address as virtual
-t     If mem mgt enabled, force data space
-k     If mem mgt enabled, force kernel mode
-s     If mem mgt enabled, force supervisor mode
-u     If mem mgt enabled, force user mode
-p     If mem mgt enabled, force previous mode
-b     Display a byte at a time rather than a word

CPU registers include the architectural state of the PDP-11 processor as well as the control registers for the interrupt system.

===============================  ====  ================================================
Name                             Size  Comments
===============================  ====  ================================================
``PC``                           16    Program counter
``R0`` .. ``R5``                 16    R0..R5, current register set
``SP``                           16    Stack pointer, current mode
``R00`` .. ``R05``               16    R0..R5, register set 0
``R10`` .. ``R15``               16    R0..R5, register set 1
``KSP``                          16    Kernel stack pointer
``SSP``                          16    Supervisor stack pointer
``USP``                          16    User stack pointer
``PSW``                          16    Processor status word
``CM``                           2     Current mode, PSW<15:14>
``PM``                           2     Previous mode, PSW<13:12>
``RS``                           2     Register set, PSW<11>
``IPL``                          3     Interrupt priority level, PSW<7:5>
``T``                            1     Trace bit, PSW<4>
``N``                            1     Negative flag, PSW<3>
``Z``                            1     Zero flag, PSW<2>
``V``                            1     Overflow flag, PSW<1>
``C``                            1     Carry flag, PSW<0>
``PIRQ``                         16    Programmed interrupt requests
``STKLIM``                       16    Stack limit
``FAC0H`` .. ``FAC5H``           32    FAC0..FAC5, high 32 bits
``FAC0L`` .. ``FAC5L``           32    FAC0..FAC5, low 32 bits
``FPS``                          16    Floating point status
``FEA``                          16    Floating exception address
``FEC``                          4     Floating exception code
``MMR0`` .. ``MMR3``             16    Memory management registers 0..3
``{K/S/U}{I/D}{PAR/PDR}{0..7}``  16    Memory management registers
``IREQ[0:7]``                    32    Interrupt pending flags, IPL 0 to 7
``TRAPS``                        18    Trap pending flags
``WAIT``                         0     Wait state flag
``WAIT_ENABLE``                  0     Wait state enable flag
``STOP_TRAPS``                   18    Stop on trap flags
``STOP_VECA``                    1     Stop on read abort in trap or interrupt
``STOP_SPA``                     1     Stop on stack abort in trap or interrupt
``PCQ[0:63]``                    16    PC prior to last jump, branch, or interrupt; most recent PC change first
``WRU``                          8     Interrupt character
===============================  ====  ================================================

The CPU attempts to detect when the simulator is idle.
When idle, the simulator does not use any resources on the host system.
Idle detection is controlled by the ``SET IDLE`` and ``SET NOIDLE`` commands::

    SET CPU IDLE                  Enable idle detection
    SET CPU NOIDLE                Disable idle detection

Idle detection is disabled by default.
The CPU is considered idle if a ``WAIT`` instruction is executed.
This will work for RSTS/E and RSX-11M+, but not for RT-11 or UNIX. 

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY               Clear history buffer
    SET CPU HISTORY=0             Disable history
    SET CPU HISTORY=n             Enable history, length = n
    SHOW CPU HISTORY              Print CPU history
    SHOW CPU HISTORY=n            Print first n entries of CPU history

The maximum length for the history is 262144 entries.

The CPU supports a number of different breakpoint types.
The default is type ``-e``, the usual break on instruction virtual address (PC) match.

======  =============================
``-e``  Instruction virtual address
``-p``  Instruction physical address
``-r``  Memory read virtual address
``-s``  Memory read physical address
``-w``  Memory write virtual address
``-x``  Memory write physical address
======  =============================

Instruction fetches are treated as memory reads,
so a read breakpoint will trigger for an instruction fetched from that address,
not just for a data read from that address.

In most cases, breakpoints are handled as if the instruction has not yet executed.
If necessary (for write breakpoints), state is restored from the start of the instruction to make it appear that way.
When execution is continued, the instruction is re-executed and the previously encountered breakpoint is suppressed for that instruction cycle.
If a given instruction encounters multiple breakpoints, for example an instruction breakpoint for the instruction address as well as a read breakpoint for the memory it references, both breakpoints will be seen (the instruction will stop twice), then after the second break execution continues past the instruction.

CIS instructions are designed to be interruptible, using the FPD bit in the processor status word.
For those instructions, a read or write breakpoint is treated as an interrupt, so the instruction is left partially completed.
When execution continues, the instruction is resumed from where it left off, provided the instruction state is not modified by the user.
The CIS instruction description in the PDP11 documentation spells out what the CIS state is that must be left untouched.

Interrupts involve two reads (the vector) and two writes (the stack push of the previous PC and PSW).
Those may match memory read or write breakpoints.
If so, the interrupt setup processing is completed normally,
and the simulator stops at the first instruction of the interrupt handler.

System registers (SYSTEM)
"""""""""""""""""""""""""
The SYSTEM device implements registers that vary among CPU types:

===========  ======================  ====  =========================================
Name         Models                  Size  Comments
===========  ======================  ====  =========================================
``SR``       11/04, 11/05, 11/20,    16    Switch register or configuration register
             11/23+, 11/34, 11/40,
             11/44, 11/45, 11/60,
             11/70, 11/73B, 11/83,
             11/84, 11/93, 11/94
``DR``       11/04, 11/05, 11/20,    16    Display register or board LEDs
             1123+, 11/24, 11/34,
             11/70, 11/73B, 11/83,
             11/84, 11/93, 11/94
``MEMERR``   11/44, 11/60, 11/70,    16    Memory error register
             11/53, 11/73, 11/73B,
             11/83, 11/84, 11/93,
             11/94
``CCR``      11/44, 11/60, 11/70,    16    Cache control register
             11/53, 11/73, 11/73B,
             11/83, 11/84, 11/93,
             11/94
``MAINT``    11/23+, 11/44, 11/70,   16    Maintenance register
             11/53, 11/73, 11/73B,
             11/83, 11/84, 11/93,
             11/94
``HITMISS``  11/44, 11/60, 11/70,    16    Hit/miss register
             11/53, 11/73, 11/73B,
             11/83, 11/84, 11/93,
             11/94
``CPUERR``   11/24, 11/44, 11/70,    16    CPU error register
             11/53, 11/73, 11/73B,
             11/83, 11/84, 11/93,
             11/94
``MBRK``     11/45, 11/70            16    Microbreak register
``SYSID``    11/70                   16    System ID (default = 1234 hex)
``JCSR``     11/53, 11/73B, 11/83,   16    Board control/status
             11/84, 11/93, 11/94
``JPCR``     11/23+, 11/53, 11/73B,  16    Page control register
             11/83, 11/84, 11/93,
             11/94
``JASR``     11/93, 11/94            16    Additional status
``UDCR``     11/84, 11/94            16    Unibus map diag control
``UDDR``     11/84, 11/94            16    Unibus map diag data
``UCSR``     11/84, 11/94            16    Unibus map control/status
``ULAST``    11/24                   23    Last Unibus map result
===========  ======================  ====  =========================================

For the 11/83, 11/84, 11/93, and 11/94, the user can set the default value of the clock frequency::

    SET SYSTEM JCLK_DEFAULT={LINE|50Hz|60HZ|800HZ}

The user can check the default value with the ``SHOW SYSTEM JCLK_DEFAULT`` command.

I/O devices
-----------

Unibus and Qbus DMA devices
"""""""""""""""""""""""""""
DMA peripherals function differently, depending on whether the CPU type supports the Unibus or the Qbus,
and whether the Unibus supports 22b direct memory access (11/70 with RH70 controllers):

==========  ========  ========  ======================
Peripheral  11/70     All       Qbus
            +RH70     other
                      Unibus
==========  ========  ========  ======================
``CD``      18b       18b       Disabled
``RC``      18b       18b       Disabled
``RF``      18b       18b       Disabled
``RK``      18b       18b       Disabled if mem > 256K
``HK``      18b       18b       Disabled if mem > 256K
``RL``      18b       18b       22b RLV12
``RP``      22b       18b       22b third party
``RQ``      18b       18b       22b RQDX3
``RY``      18b       18b       Disabled if mem > 256K
``TC``      18b       18b       Disabled
``TM``      18b       18b       Disabled if mem > 256K
``TS``      18b       18b       22b TSV05
``TQ``      18b       18b       22b TQK50
``TU``      22b       18b       22b third party
``VH``      18b       18b       22b DHQ11
``XQ``      Disabled  Disabled  22b DELQA
``XU``      18b       18b       Disabled
==========  ========  ========  ======================

Non-DMA peripherals work the same in all configurations.
Unibus-only peripherals are disabled in a Qbus configuration, and Qbus-only peripherals are disabled in a Unibus configuration.
In addition, Qbus DMA peripherals with only 18b addressing capability are disabled in a Qbus configuration with more than 256KB memory.

I/O device addressing
"""""""""""""""""""""
PDP-11 I/O space and vector space are not large enough to allow all possible devices to be configured simultaneously at fixed addresses.
Instead, many devices have floating addresses and vectors;
that is, the assigned device address and vector depend on the presence of other devices in the configuration:

=============  ===============================================
DZ11           All instances have floating addresses
DHU11/DHQ11    All instances have floating addresses
RL11           First instance has fixed address, rest floating
RX11/RX211     First instance has fixed address, rest floating
DEUNA/DELUA    First instance has fixed address, rest floating
MSCP disk      First instance has fixed address, rest floating
TMSCP tape     First instance has fixed address, rest floating
=============  ===============================================

In addition, some devices with fixed I/O space addresses have floating vector addresses.
DCI/DCO and DLI/DLO have floating vector addresses.

To maintain addressing consistency as the configuration changes,
the simulator implements DEC's standard I/O address and vector autoconfiguration.
This allows the user to enable or disable devices without needing to manage I/O addresses and vectors.
For example, if RY is enabled while RX is present, RY is assigned an I/O address in the floating I/O space range;
but if RX is disabled and then RY is enabled, RY is assigned the fixed "first instance" I/O address for floppy disks.

Autoconfiguration cannot solve address conflicts between devices with overlapping fixed addresses.
For example, with default I/O page addressing, the PDP-11 can support either a TM11 or a TS11, but not both, since they use the same I/O addresses.

In addition to autoconfiguration, most devices support the ``SET <device> ADDRESS`` command,
which allows the I/O page address of the device to be changed,
and the ``SET <device> VECTOR`` command, which allows the vector of the device to be changed.
Explicitly setting the I/O address of a device that normally uses autoconfiguration DISABLES autoconfiguration for that device and for the entire system.
As a consequence, the user may have to manually configure all other autoconfigured devices, because the autoconfiguration algorithm no longer recognizes the explicitly configured device.
A device can be reset to autoconfigure with the ``SET <device> AUTOCONFIGURE`` command.
Autoconfiguration can be restored for the entire system with the ``SET CPU AUTOCONFIGURE`` command.

The current I/O map can be displayed with the ``SHOW CPU IOSPACE`` command.
Addresses that have set by autoconfiguration are marked with an asterisk (\*).

All devices support the ``SHOW <device> ADDRESS`` and ``SHOW <device> VECTOR`` commands,
which display the device address and vector, respectively.

Programmed I/O devices
----------------------

PC11 paper tape reader (PTR)
""""""""""""""""""""""""""""
The paper tape reader (PTR) reads data from a disk file.
The POS register specifies the number of the next data item to be read.
Thus, by changing POS, the user can backspace or advance the reader.

The paper tape reader implements these registers:

============  ====  ==============================
Name          Size  Comments
============  ====  ==============================
``BUF``       8     Last data item processed
``CSR``       16    Control/status register
``INT``       1     Interrupt pending flag
``ERR``       1     Error flag (CSR<15>)
``BUSY``      1     Busy flag (CSR<11>)
``DONE``      1     Device done flag (CSR<7>)
``IE``        1     Interrupt enable flag (CSR<6>)
``POS``       32    Position in the input file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  ==============================

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

PC11 paper tape punch (PTP)
"""""""""""""""""""""""""""
The paper tape punch (PTP) writes data to a disk file.
The POS register specifies the number of the next data item to be written.
Thus, by changing POS, the user can backspace or advance the punch.
The default position after ATTACH is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The paper tape punch implements these registers:

==============  ====  =========================
Name            Size  Comments
==============  ====  =========================
``BUF``         8     Last data item processed
``CSR``         16    Control/status register
``INT``         1     Interrupt pending flag
``ERR``         1     Error flag (CSR<15>)
``DONE``        1     Device done flag (CSR<7>)
``IE``          1     Interrupt enable flag (CSR<6>)
``POS``         32    Position in the output file
``TIME``        24    Time from I/O initiation to interrupt
``STOP_IOE``    1     Stop on I/O error
==============  ====  =========================

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

DL11 terminal input (TTI)
"""""""""""""""""""""""""
The terminal interfaces (TTI, TTO) can be set to one of four modes, ``7P``, ``7B`` or ``8B``:

======  =======================  ======================
Mode    Input characters         Output characters
======  =======================  ======================
``UC``  High-order bit cleared,  High order-bit cleared,
        lowercase converted      lowercase converted
        to uppercase             to uppercase
``7P``  High-order bit cleared   High-order bit cleared,
                                 non-printing characters suppressed
``7B``  High-order bit cleared   High-order bit cleared
``8B``  No changes               No changes
======  =======================  ======================

The default mode is ``7B`` (TTI) and ``7P`` (TTO).

DL11 terminal output (TTO)
""""""""""""""""""""""""""
The terminal input (TTI) polls the console keyboard for input.
It implements these registers:

=========  =======  ============================================
Name       Size     Comments
=========  =======  ============================================
``BUF``    8        Last data item processed
``CSR``    16       Control/status register
``INT``    1        Interrupt pending flag
``ERR``    1        Error flag (CSR<15>)
``DONE``   1        Device done flag (CSR<7>)
``IE``     1        Interrupt enable flag (CSR<6>)
``POS``    32       Number of characters output
``TIME``   24       Input polling interval (if 0, the keyboard
                    is polled synchronously with the line clock)
=========  =======  ============================================

The terminal output (TTO) writes to the simulator console window.
It implements these registers:

=========  ====  ======================================
Name       Size  Comments
=========  ====  ======================================
``BUF``    8     Last data item processed
``CSR``    16    Control/status register
``INT``    1     Interrupt pending flag
``ERR``    1     Error flag (CSR<15>)
``DONE``   1     Device done flag (CSR<7>)
``IE``     1     Interrupt enable flag (CSR<6>)
``POS``    32    Number of characters input
``TIME``   24    Time from I/O initiation to interrupt
=========  ====  ======================================

LP11 line printer (LPT)
"""""""""""""""""""""""
The line printer (LPT) writes data to a disk file.
The POS register specifies the number of the next data item to be written.
Thus, by changing POS, the user can backspace or advance the printer.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The line printer implements these registers:

============  ====  =========================================
Name          Size  Comments
============  ====  =========================================
``BUF``       8     Last data item processed
``CSR``       16    Control/status register
``INT``       1     Interrupt pending flag
``ERR``       1     Error flag (CSR<15>)
``DONE``      1     Device done flag (CSR<7>)
``IE``        1     Interrupt enable flag (CSR<6>)
``POS``       32    Position in the output file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  =========================================

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

KW11-L Line-time clock (CLK)
""""""""""""""""""""""""""""
The line-time clock (CLK) frequency can be adjusted as follows::

    SET CLK 60HZ                  Set frequency to 60Hz
    SET CLK 50HZ                  Set frequency to 50Hz

The default is 60Hz.

The line-time clock implements these registers:

=========  =====  ==============================
Name       Size   Comments
=========  =====  ==============================
``CSR``    16     Control/status register
``INT``    1      Interrupt pending flag
``DONE``   1      Device done flag (CSR<7>)
``IE``     1      Interrupt enable flag (CSR<6>)
``TIME``   24     Clock interval
=========  =====  ==============================

The line-time clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

KW11-P programmable clock (PCLK)
""""""""""""""""""""""""""""""""
The programmable clock (PCLK) line frequency can be adjusted as follows::

    SET PCLK 60HZ                  Set frequency to 60Hz
    SET PCLK 50HZ                  Set frequency to 50Hz

The default is 60Hz.

The programmable clock implements these registers:

========================  ======  ===============================
Name                      Size    Comments
========================  ======  ===============================
``CSR``                   16      Control/status register
``CSB``                   16      Count set buffer
``CNT``                   16      Current count
``INT``                   1       Interrupt pending flag
``OVFL``                  1       Overflow (error) flag (CSR<15>)
``DONE``                  1       Device done flag (CSR<7>)
``IE``                    1       Interrupt enable flag (CSR<6>)
``UPDN``                  1       Up/down count mode (CSR<4>)
``MODE``                  1       Single/repeat mode (CSR<3>)
``RUN``                   1       Clock run (CSR<0>)
``TIME0`` .. ``TIME3``    32      Clock interval, rates 0 to 3
``TPS0`` .. ``TIME3``     32      Ticks per second, rates 0 to 3
========================  ======  ===============================

The programmable clock autocalibrates; the clock interval is adjusted up or down so that the clock tracks actual elapsed time.
Operation at the highest clock rate (100Khz) is not recommended.
The programmable clock is disabled by default.

TA11/TA60 cassette tape (CT)
""""""""""""""""""""""""""""
The TA11 is a programmed I/O controller supporting two cassette drives (0 and 1).
The TA11 can be used like a small magtape under RT11 and RSX-11M, and with the CAPS-11 operating system.
Cassettes are simulated as magnetic tapes with a fixed capacity (93,000 characters).
The tape format is always SIMH standard.
The TA11 is disabled by default.

TA11 options include the ability to make units write enabled or write locked. ::

    SET CTn LOCKED                Set unit n write locked
    SET CTn WRITEENABLED          Set unit n write enabled

Units can not be set ``ENABLED`` or ``DISABLED``.
The TA11 does not support the ``BOOT`` command.

The TA11 controller implements these registers:

============  ====  ==========================
Name          Size  Comments
============  ====  ==========================
``TACS``      16    Control/status register
``TAIDB``     8     Input data buffer
``TAODB``     8     Output data buffer
``INT``       1     Interrupt request
``ERR``       1     Error flag
``TR``        1     Transfer request flag
``IE``        1     Interrupt enable flag
``WRITE``     1     TA60 write operation flag
``BPTR``      17    Buffer pointer
``BLNT``      17    Buffer length
``STIME``     24    Operation start time
``CTIME``     24    Character latency
``STOP_IOE``  1     Stop on I/O errors flag
``POS[0:1]``  32    Position, units 0-1
============  ====  ==========================

Error handling is as follows:

============  =====================================
Error         Processed as
============  =====================================
not attached  Tape not ready; if ``STOP_IOE``, stop
end of file   Bad tape
OS I/O error  CRC error; if ``STOP_IOE``, stop
============  =====================================

Floppy disk drives
------------------

RX11/RX01 floppy disk (RX)
""""""""""""""""""""""""""
RX11 options include the ability to set units write enabled or write locked::

    SET RXn LOCKED                Set unit n write locked
    SET RXn WRITEENABLED          Set unit n write enabled

The RX11 supports the ``BOOT`` command.

The RX11 implements these registers:

===============  ====  ====================================
Name             Size  Comments
===============  ====  ====================================
``RXCS``         12    Status
``RXDB``         8     Data buffer
``RXES``         8     Error status
``RXERR``        8     Error code
``RXTA``         8     Current track
``RXSA``         8     Current sector
``STAPTR``       3     Controller state
``BUFPTR``       3     Buffer pointer
``INT``          1     Interrupt pending flag
``ERR``          1     Error flag (CSR<15>)
``TR``           1     Transfer ready flag (CSR<7>)
``IE``           1     Interrupt enable flag (CSR<6>)
``DONE``         1     Device done flag (CSR<5>)
``CTIME``        24    Command completion time
``STIME``        24    Seek time, per track
``XTIME``        24    Transfer ready delay
``STOP_IOE``     1     Stop on I/O error
``SBUF[0:127]``  8     Sector buffer array
===============  ====  ====================================

Error handling is as follows:

+--------------+-------------------+-------------------------+
| Error        | ``STOP_IOE``      | Processed as            |
+==============+===================+=========================+
| not attached | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Disk not ready          |
+--------------+-------------------+-------------------------+

RX01 data files are buffered in memory;
therefore, end of file and OS I/O errors cannot occur.

RX211/RX02 floppy disk (RY)
"""""""""""""""""""""""""""
RX211 options include the ability to set units write enabled or write locked, single or double density, or autosized::

    SET RYn LOCKED                Set unit n write locked
    SET RYn WRITEENABLED          Set unit n write enabled
    SET RYn SINGLE                Set unit n single density
    SET RYn DOUBLE                Set unit n double density (default)
    SET RYn AUTOSIZE              Set unit n to autosize at ATTACH

The RX211 supports the ``BOOT`` command.
The RX211 is disabled in a Qbus system with more than 256KB of memory.

The RX211 implements these registers:

===============  ======  ==============================
Name             Size    Comments
===============  ======  ==============================
``RYCS``         16      Status
``RYBA``         16      Buffer address
``RYWC``         8       Word count
``RYDB``         16      Data buffer
``RYES``         12      Error status
``RYERR``        8       Error code
``RYTA``         8       Current track
``RYSA``         8       Current sector
``STAPTR``       4       Controller state
``INT``          1       Interrupt pending flag
``ERR``          1       Error flag (CSR<15>)
``TR``           1       Transfer ready flag (CSR<7>)
``IE``           1       Interrupt enable flag (CSR<6>)
``DONE``         1       Device done flag (CSR<5>)
``CTIME``        24      Command completion time
``STIME``        24      Seek time, per track
``XTIME``        24      Transfer ready delay
``STOP_IOE``     1       Stop on I/O error
``SBUF[0:255]``  8       Sector buffer array
===============  ======  ==============================

Error handling is as follows:

+--------------+-------------------+-------------------------+
| Error        | ``STOP_IOE``      | Processed as            |
+==============+===================+=========================+
| not attached | 1                 | Report error and stop   |
|              +-------------------+-------------------------+
|              | 0                 | Disk not ready          |
+--------------+-------------------+-------------------------+

RX02 data files are buffered in memory;
therefore, end of file and OS I/O errors cannot occur.

Cartridge disk drives
---------------------

RK11/RK05 cartridge disk (RK)
"""""""""""""""""""""""""""""
RK11 options include the ability to make units write enabled or write locked::

    SET RKn LOCKED                Set unit n write locked
    SET RKn WRITEENABLED          Set unit n write enabled

Units can also be set ``ENABLED`` or ``DISABLED``.
The RK11 supports the ``BOOT`` command.
The RK11 is disabled in a Qbus system with more than 256KB of memory.

The RK11 implements these registers:

============  ====  ===============================
Name          Size  Comments
============  ====  ===============================
``RKCS``      16    Control/status
``RKDA``      16    Disk address
``RKBA``      16    Memory address
``RKWC``      16    Word count
``RKDS``      16    Drive status
``RKER``      16    Error status
``INTQ``      9     Interrupt queue
``DRVN``      3     Number of last selected drive
``INT``       1     Interrupt pending flag
``ERR``       1     Error flag (CSR<15>)
``DONE``      1     Device done flag (CSR<7>)
``IE``        1     Interrupt enable flag (CSR<6>)
``INT``       1     Interrupt pending flag
``STIME``     24    Seek time, per cylinder
``RTIME``     24    Rotational delay
``STOP_IOE``  1     Stop on I/O error
============  ====  ===============================

Error handling is as follows:

+--------------+-------------------+------------------------------+
| Error        | ``STOP_IOE``      | Processed as                 |
+==============+===================+==============================+
| not attached | 1                 | Report error and stop        |
|              +-------------------+------------------------------+
|              | 0                 | Disk not ready               |
+--------------+-------------------+------------------------------+
| end of file  | x                 | Assume rest of disk is zero  |
+--------------+-------------------+------------------------------+
| OS I/O error | x                 | Report error and stop        |
+--------------+-------------------+------------------------------+

RK611/RK06,RK07 cartridge disk (HK)
"""""""""""""""""""""""""""""""""""
RK611 options include the ability to set units write enabled or write locked,
to set the drive type to RK06, RK07, or autosize,
and to write a DEC standard 144 compliant bad block table on the last track::

    SET HKn LOCKED                Set unit n write locked
    SET HKn WRITEENABLED          Set unit n write enabled
    SET HKn RK06                  Set type to RK06
    SET HKn RK07                  Set type to RK07
    SET HKn AUTOSIZE              Set type based on file size at ATTACH
    SET HKn BADBLOCK              Write bad block table on last track

The type options can be used only when a unit is not attached to a file.
The bad block option can be used only when a unit is attached to a file.
Units can be set ``ENABLED`` or ``DISABLED``.
The RK611 supports the ``BOOT`` command.
The RK611 is disabled in a Qbus system with more than 256KB of memory.

The RK611 implements these registers:

==============  ====  =================================
Name            Size  Comments
==============  ====  =================================
``HKCS1``       16    control/status 1
``HKWC``        16    word count
``HKBA``        16    bus address
``HKDA``        16    desired surface, sector
``HKCS2``       16    control/status 2
``HKDS[0:7]``   16    drive status, drives 0 to 7
``HKER[0:7]``   16    drive errors, drives 0 to 7
``HKDB[0:2]``   16    data buffer silo
``HKDC``        16    desired cylinder
``HKOF``        8     offset
``HKMR``        16    maintenance register
``HKSPR``       16    spare register
``HKCI``        1     controller interrupt flop
``HKDI``        1     drive interrupt flop
``HKEI``        1     error interrupt flop
``INT``         1     interrupt pending flag
``ERR``         1     error flag (CSR<15>)
``DONE``        1     device done flag (CSR1<7>)
``IE``          1     interrupt enable flag (CSR1<6>)
``STIME``       24    seek time, per cylinder
``RTIME``       24    rotational delay
``MIN2TIME``    24    minimum time between DONE and ATA
``STOP_IOE``    1     stop on I/O error
==============  ====  =================================

Error handling is as follows:

+--------------+-------------------+------------------------------+
| Error        | ``STOP_IOE``      | Processed as                 |
+==============+===================+==============================+
| not attached | 1                 | Report error and stop        |
|              +-------------------+------------------------------+
|              | 0                 | Disk not ready               |
+--------------+-------------------+------------------------------+
| end of file  | x                 | Assume rest of disk is zero  |
+--------------+-------------------+------------------------------+
| OS I/O error | x                 | Report error and stop        |
+--------------+-------------------+------------------------------+

RL11(RLV12)/RL01,RL02 cartridge disk (RL)
"""""""""""""""""""""""""""""""""""""""""
RL11 options include the ability to set units write enabled or write locked,
to set the drive type to RL01, RL02, or autosize,
and to write a DEC standard 144 compliant bad block table on the last track::

    SET RLn LOCKED                Set unit n write locked
    SET RLn WRITEENABLED          Set unit n write enabled
    SET RLn RL01                  Set type to RL01
    SET RLn RL02                  Set type to RL02
    SET RLn AUTOSIZE              Set type based on file size at ATTACH
    SET RLn BADBLOCK              Write bad block table on last track

The type options can be used only when a unit is not attached to a file.
The bad block option can be used only when a unit is attached to a file.
Units can be set ``ENABLED`` or ``DISABLED``.
The RL11 supports the ``BOOT`` command.
In a Unibus system, the RL behaves like an RL11 with 18b addressing;
in a Qbus (Q22) system, the RL behaves like the RLV12 with 22b addressing.

The RL11 implements these registers:

================================  ====  ===================================
Name                              Size  Comments
================================  ====  ===================================
``RLCS``                          16    Control/status
``RLDA``                          16    Disk address
``RLBA``                          16    Memory address
``RLBAE``                         6     Memory address extension (RLV12)
``RLMP``, ``RLMP1``, ``RLMP2``    16    Multipurpose register queue
``INT``                           1     Interrupt pending flag
``ERR``                           1     Error flag (CSR<15>)
``DONE``                          1     Device done flag (CSR<7>)
``IE``                            1     Interrupt enable flag (CSR<6>)
``STIME``                         24    Seek time, per cylinder
``RTIME``                         24    Rotational delay
``STOP_IOE``                      1     Stop on I/O error
================================  ====  ===================================

Error handling is as follows:

+--------------+-------------------+------------------------------+
| Error        | ``STOP_IOE``      | Processed as                 |
+==============+===================+==============================+
| not attached | 1                 | Report error and stop        |
|              +-------------------+------------------------------+
|              | 0                 | Disk not ready               |
+--------------+-------------------+------------------------------+
| end of file  | x                 | Assume rest of disk is zero  |
+--------------+-------------------+------------------------------+
| OS I/O error | x                 | Report error and stop        |
+--------------+-------------------+------------------------------+

Massbus subsystems
------------------

RH70/RH11 Massbus adapters (RHA, RHB, RHC)
""""""""""""""""""""""""""""""""""""""""""
The RH70/RH11 Massbus adapters interface Massbus peripherals to the memory bus or Unibus of the CPU.
The simulator provides three Massbus adapters.
The first, RHA, is configured for the RP family of disk drives.
The second, RHB, is configured for the TU family of tape controllers.
The third, RHC, is configured for the RS family of fixed head disks.
By default, RHA is enabled and RHB and RHC are disabled.
In a Unibus system, the RH adapters implement 22b addressing for the 11/70 and 18b addressing for all other models.
In a Qbus system, the RH adapters always implement 22b addressing.

Each RH adapter implements these registers:

========  ====  =========================================
Name      Size  Comments
========  ====  =========================================
``CS1``   16    Control/status register 1
``WC``    16    Word count
``BA``    16    Bus address
``CS2``   16    Control/status register 2
``DB``    16    Data buffer
``BAE``   6     Bus address extension
``CS3``   16    Control/status register 3
``IFF``   1     Transfer complete interrupt request flop
``INT``   1     Interrupt pending flag
``SC``    1     Special condition (CSR1<15>)
``DONE``  1     Device done flag (CSR1<7>)
``IE``    1     Interrupt enable flag (CSR1<6>)
========  ====  =========================================

RP04/05/06/07, RM02/03/05/80 disk pack drives (RP)
""""""""""""""""""""""""""""""""""""""""""""""""""
The RP controller implements the Massbus family of large disk drives.
RP options include the ability to set units write enabled or write locked,
to set the drive type to one of six disk types or autosize,
and to write a DEC standard 144 compliant bad block table on the last track::

    SET RPn LOCKED           Set unit n write locked
    SET RPn WRITEENABLED     Set unit n write enabled
    SET RPn RM03             Set type to RM03
    SET RPn RM05             Set type to RM05
    SET RPn RM80             Set type to RM80
    SET RPn RP04             Set type to RP04
    SET RPn RP05             Set type to RP05
    SET RPn RP06             Set type to RP06
    SET RPn RP07             Set type to RP07
    SET RPn AUTOSIZE         Set type based on file size at ATTACH
    SET RPn BADBLOCK         Write bad block table on last track

The type options can be used only when a unit is not attached to a file.
The bad block option can be used only when a unit is attached to a file.
Units can be set ``ENABLED`` or ``DISABLED``.
The RP controller supports the ``BOOT`` command.

The RP controller implements the registers listed below.
Registers suffixed with ``[0:7]`` are replicated per drive.

============  ====  ======================================
Name          Size  Comments
============  ====  ======================================
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
============  ====  ======================================

Error handling is as follows:

+--------------+-------------------+------------------------------+
| Error        | ``STOP_IOE``      | Processed as                 |
+==============+===================+==============================+
| not attached | 1                 | Report error and stop        |
|              +-------------------+------------------------------+
|              | 0                 | Disk not ready               |
+--------------+-------------------+------------------------------+
| end of file  | x                 | Assume rest of disk is zero  |
+--------------+-------------------+------------------------------+
| OS I/O error | x                 | Report error and stop        |
+--------------+-------------------+------------------------------+

TM02/TM03/TE16/TU45/TU77 magnetic tapes (TU)
""""""""""""""""""""""""""""""""""""""""""""
The TU controller implements the Massbus family of 800/1600bpi magnetic tape drives.
TU options include the ability to select the formatter type (TM02 or TM03),
to set the drive type to one of three drives (TE16, TU45, or TU77),
and to set the drives write enabled or write locked. ::

    SET TU TM02                   Set controller type to TM02
    SET TU TM03                   Set controller type to TM03
    SET TUn TE16                  Set drive type to TE16
    SET TUn TU45                  Set drive type to TU45
    SET TUn TU77                  Set drive type to TU77

Magnetic tape units can be set to a specific reel capacity in MB, or to unlimited capacity::

    SET TUn CAPAC=m               Set unit n capacity to m MB (0 = unlimited)
    SHOW TUn CAPAC                Show unit n capacity in MB

Units can be set ``ENABLED`` or ``DISABLED``.
The TU controller supports the ``BOOT`` command.

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

RS03/RS04 fixed head disks
""""""""""""""""""""""""""
The RS controller implements the Massbus family fixed head disks.
RS options include the ability to set units write enabled or write locked and to set the drive type to RS03, RS04, or autosize. ::

    SET RSn LOCKED                Set unit n write lock enabled
    SET RSn WRITEENABLED          Set unit n write enabled
    SET RSn RS03                  Set type to RS03
    SET RSn RS05                  Set type to RS04
    SET RSn AUTOSIZE              Set type based on file size at ATTACH

The drive type options can be used only when a unit is not attached to a file.
Units can be set ``ENABLED`` or ``DISABLED``.
The RS controller supports the ``BOOT`` command.

The RS controller implements the registers listed below.
Registers suffixed with ``[0:7]`` are replicated per drive.

==============  ====  ====================================
Name            Size  Comments
==============  ====  ====================================
``CS1[0:7]``    16    Current operation
``DA[0:7]``     16    Desired track, sector
``DS[0:7]``     16    Drive status
``ER[0:7]``     16    Drive errors
``MR[0:7]``     16    Maintenance register
``WLK[0:7]``    6     Max write locked track, if enabled
``TIME``        24    Rotational delay, per word
``STOP_IOE``    1     Stop on I/O error
==============  ====  ====================================

Error handling is as follows:

+--------------+-------------------+------------------------------+
| Error        | ``STOP_IOE``      | Processed as                 |
+==============+===================+==============================+
| not attached | 1                 | Report error and stop        |
|              +-------------------+------------------------------+
|              | 0                 | Disk not ready               |
+--------------+-------------------+------------------------------+

RS data files are buffered in memory;
therefore, end of file and OS I/O errors cannot occur.

RQDX3/UDA50 MSCP disk controllers (RQ, RQB, RQC, RQD)
-----------------------------------------------------
The simulator implements four MSCP disk controllers, RQ, RQB, RQC, RQD.
Initially, RQB, RQC, and RQD are disabled.
Each RQ controller simulates an RQDX3 MSCP disk controller with four disk drives.
RQ options include the ability to set units write enabled or write locked,
and to set the drive type to one of many disk types::

    SET RQn LOCKED                Set unit n write locked
    SET RQn WRITEENABLED          Set unit n write enabled
    SET RQn RX50                  Set type to RX50
    SET RQn RX33                  Set type to RX33
    SET RQn RD32                  Set type to RD32
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
    SET RQn RRD40                 Set type to RRD40 (CD-ROM)
    SET RQn RAUSER{=n}            Set type to RA82 with n MB's
    SET -L RQn RAUSER{=n}         Set type to RA82 with n LBN's

The type options can be used only when a unit is not attached to a file.
RAUSER is a "user specified" disk;
the user can specify the size of the disk in either MB (1000000 bytes) or logical block numbers (LBN's, 512 bytes each).
The minimum size is 5MB; the maximum size is 2GB without extended file support, 1TB with extended file support.

Units can be set ``ENABLED`` or ``DISABLED``.
Each RQ controller supports the ``BOOT`` command.
In a Unibus system, an RQ supports 18b addressing and identifies itself as a UDA50.
In a Qbus system, an RQ supports 22b addressing and identifies itself as an RQDX3.

Drive units have changeable unit numbers.
Unit numbers can be changed with::

    SET RQn UNIT=val              Set unit plug value

Device RQ has 4 units (RQ0, RQ1, RQ2 and RQ3) which have unique MSCP unit numbers (0, 1, 2 and 3).
Device RQB has 4 units (RQB0, RQB1, RQB2 and RQB3) which have unique MSCP unit numbers (4, 5, 6 and 7).
Device RQC has 4 units (RQC0, RQC1, RQC2 and RQC3) which have unique MSCP unit numbers (8, 9, 10 and 11).
Device RQD has 4 units (RQD0, RQD1, RQD2 and RQD3) which have unique MSCP unit numbers (12, 13, 14 and 15).

Each RQ controller implements the following special ``SHOW`` commands::

    SHOW RQn TYPE                 Show drive type
    SHOW RQ RINGS                 Show command and response rings
    SHOW RQ FREEQ                 Show packet free queue
    SHOW RQ RESPQ                 Show packet response queue
    SHOW RQ UNITQ                 Show unit queues
    SHOW RQ ALL                   Show all ring and queue state
    SHOW RQn UNITQ                Show unit queues for unit n
    SHOW RQn UNIT                 Show unit plug value

Each RQ controller implements these registers:

===============  ====  ===========================================================
Name             Size  Comments
===============  ====  ===========================================================
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
``PLUG[0:3]``    16    Unit plug value, units 0 to 3
``INT``          1     Interrupt request
``ITIME``        1     Response time for initialization steps (except for step 4)
``QTIME``        24    Response time for 'immediate' packets
``XTIME``        24    Response time for data transfers
``PKTS[33*32]``  16    Packet buffers, 33W each, 32 entries
===============  ====  ===========================================================

Some DEC operating systems, notably RSX11M/M+, are very sensitive to the timing parameters.
Changing the default values may cause M/M+ to crash on boot or to hang during operation.

Error handling is as follows:

==============  =============================
Error           Processed as
==============  =============================
not attached    Disk not ready
end of file     Assume rest of disk is zero
OS I/O error    Report error and stop
==============  =============================

Fixed head disks
----------------

RC11 fixed head disk (RC)
"""""""""""""""""""""""""
RC11 options include the ability to set the number of platters to a fixed value between 1 and 4,
or to autosize the number of platters::

    SET RC 1P                     One platter (256K)
    SET RC 2P                     Two platters (512K)
    SET RC 3P                     Three platters (768K)
    SET RC 4P                     Four platters (1024K)
    SET RC AUTOSIZE               Autosized on ATTACH

The default is one platter.
The RC11 does not support the ``BOOT`` command.
The RC11 is disabled at startup and is automatically disabled in a Qbus system.

The RC11 is a DMA device.
The entire transfer occurs in a single DMA transfer.

The RC11 implements these registers:

============  ====  ===========================
Name          Size  Comments
============  ====  ===========================
``RCLA``      16    Look ahead register
``RCDA``      16    Current disk address
``RCER``      16    Error register
``RCCS``      16    Control/status
``RCWC``      16    Word count
``RCCA``      16    Current memory address
``RCMN``      16    Maintenance register
``RCDB``      16    Data buffer
``RCWLK``     32    Write lock switches
``INT``       1     Interrupt pending flag
``ERR``       1     Device error flag
``DONE``      1     Device done flag
``IE``        1     Interrupt enable flag
``TIME``      24    Rotational delay, per word
``STOP_IOE``  1     Stop on I/O error
============  ====  ===========================

Error handling is as follows:

+--------------+-------------------+------------------------------+
| Error        | ``STOP_IOE``      | Processed as                 |
+==============+===================+==============================+
| not attached | 1                 | Report error and stop        |
|              +-------------------+------------------------------+
|              | 0                 | Non-existent disk            |
+--------------+-------------------+------------------------------+

RC11 data files are buffered in memory;
therefore, end of file and OS I/O errors cannot occur.

RF11/RS11 fixed head disk (RF)
""""""""""""""""""""""""""""""
RF11 options include the ability to set the number of platters to a fixed value between 1 and 8,
or to autosize the number of platters::

    SET RF 1P                     One platter (256K)
    SET RF 2P                     Two platters (512K)
    SET RF 3P                     Three platters (768K)
    SET RF 4P                     Four platters (1024K)
    SET RF 5P                     Five platters (1280K)
    SET RF 6P                     Six platters (1536K)
    SET RF 7P                     Seven platters (1792K)
    SET RF 8P                     Eight platters (2048K)
    SET RF AUTOSIZE               Autosized on ATTACH

The default is one platter.
The RF11 supports the ``BOOT`` command.
The RF11 is disabled at startup and is automatically disabled in a Qbus system.

The RF11 implements these registers:

============  ====  ==========================
Name          Size  Comment
============  ====  ==========================
``RFCS``      16    Control/status
``RFWC``      16    Word count
``RFCMA``     16    Current memory address
``RFDA``      16    Current disk address
``RFDAE``     16    Disk address extension
``RFDBR``     16    Data buffer
``RFMR``      16    Maintenance register
``RFWLK``     32    Write lock switches
``INT``       1     Interrupt pending flag
``ERR``       1     Device error flag
``DONE``      1     Device done flag
``IE``        1     Interrupt enable flag
``TIME``      24    Rotational delay, per word
``BURST``     1     Burst flag
``STOP_IOE``  1     Stop on I/O error
============  ====  ==========================

The RF11 is a DMA device.
If ``BURST`` = 0, word transfers are scheduled individually;
if ``BURST`` = 1, the entire transfer occurs in a single DMA transfer.

Error handling is as follows:

+--------------+-------------------+------------------------------+
| Error        | ``STOP_IOE``      | Processed as                 |
+==============+===================+==============================+
| not attached | 1                 | Report error and stop        |
|              +-------------------+------------------------------+
|              | 0                 | Non-existent disk            |
+--------------+-------------------+------------------------------+

RF11 data files are buffered in memory;
therefore, end of file and OS I/O errors cannot occur.

TC11/TU56 DECtape (DT)
----------------------
The DT controller implements the TC11 DECtape controller and TU56 drives.
DECtape options include the ability to make units write enabled or write locked. ::

    SET DTn LOCKED                Set unit n write locked
    SET DTn WRITEENABLED          Set unit n write enabled

Units can be set ``ENABLED`` or ``DISABLED``.
The TC11 supports the ``BOOT`` command.
The TC11 is automatically disabled in a Qbus system.

The TC11 supports supports PDP-8 format, PDP-11 format, and 18b format DECtape images.
``ATTACH`` assumes the image is in PDP-11 format; the user can force other choices with switches:

-t   PDP-8 format
-f   18b format
-a   Autoselect based on file size

The DECtape controller is a data-only simulator;
the timing and mark track, and block header and trailer, are not stored.
Thus, the ``WRITE TIMING AND MARK TRACK`` function is not supported;
the ``READ ALL`` function always returns the hardware standard block header and trailer;
and the ``WRITE ALL`` function dumps non-data words into the bit bucket.

The TC controller implements these registers:

===============  ====  =================================
Name             Size  Comments
===============  ====  =================================
``TCST``         16    Status register
``TCCM``         16    Command register
``TCWC``         16    Word count register
``TCBA``         16    Bus address register
``TCDT``         16    Data register
``INT``          1     Interrupt pending flag
``ERR``          1     Error flag
``DONE``         1     Done flag
``IE``           1     Interrupt enable flag
``CTIME``        31    Time to complete transport stop
``LTIME``        31    Time between lines
``DCTIME``       31    Time to decelerate to a full stop
``SUBSTATE``     2     Read/write command substate
``POS[0:7]``     32    Position, in lines, units 0 to 7
``STATT[0:7]``   31    Unit state, units 0 to 7
``STOP_OFFR``    1     Stop on off-reel error
===============  ====  =================================

It is critically important to maintain certain timing relationships among the DECtape parameters,
or the DECtape simulator will fail to operate correctly.

- ``LTIME`` must be at least 6
- ``DCTIME`` needs to be at least 100 times ``LTIME``

Acceleration time is set to 75% of deceleration time.

Magnetic tape controllers
-------------------------

TM11 magnetic tape (TM)
"""""""""""""""""""""""
TM options include the ability to make units write enabled or write locked. ::

    SET TMn LOCKED                Set unit n write locked
    SET TMn WRITEENABLED          Set unit n write enabled

Magnetic tape units can be set to a specific reel capacity in MB, or to unlimited capacity::

    SET TMn CAPAC=m               Set unit n capacity to m MB (0 = unlimited)
    SHOW TMn CAPAC                Show unit n capacity in MB

Units can be set ``ENABLED`` or ``DISABLED``.

The TM11 supports the ``BOOT`` command.
The bootstrap supports both original and DEC standard boot formats.
Originally, a tape bootstrap read and executed the first record on tape.
To allow for ANSI labels, the DEC standard bootstrap skipped the first record and read and executed the second.
The DEC standard is the default; to bootstrap an original format tape, use the command ``BOOT -O MTn``.
The TM11 is automatically disabled in a Qbus system with more than 256KB of memory.

The TM controller implements these registers:

============  ====  ==========================
Name          Size  Comments
============  ====  ==========================
``MTS``       16    Status
``MTC``       16    Command
``MTCMA``     16    Memory address
``MTBRC``     16    Byte/record count
``INT``       1     Interrupt pending flag
``ERR``       1     Error flag
``DONE``      1     Device done flag
``IE``        1     Interrupt enable flag
``STOP_IOE``  1     Stop on I/O error
``TIME``      24    Delay
``UST[0:7]``  16    Unit status, units 0 to 7
``POS[0:7]``  32    Position, units 0 to 7
============  ====  ==========================

Error handling is as follows:

============  =====================================
Error         Processed as
============  =====================================
not attached  Tape not ready; if ``STOP_IOE``, stop
end of file   Bad tape
OS I/O error  Parity error; if ``STOP_IOE``, stop
============  =====================================

TS11/TSV05 magnetic tape (TS)
"""""""""""""""""""""""""""""
TS options include the ability to make the unit write enabled or write locked. ::

    SET TS LOCKED                 Set unit write locked
    SET TS WRITEENABLED           Set unit write enabled

The TS drive can be set to a specific reel capacity in MB, or to unlimited capacity::

    SET TS0 CAPAC=m               Set capacity to m MB (0 = unlimited)
    SHOW TS0 CAPAC                Show capacity in MB

The TS11 supports the ``BOOT`` command.
The bootstrap supports only DEC standard boot formats.
To allow for ANSI labels, the DEC standard bootstrap skipped the first record and read and executed the second.
In a Unibus system, the TS behaves like the TS11 and implements 18b addresses.
In a Qbus system, the TS behaves like the TSV05 and implements 22b addresses.

The TS controller implements these registers:

=========  ====  =====================================
Name       Size  Comments
=========  ====  =====================================
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
=========  ====  =====================================

Error handling is as follows:

============  ================
Error         Processed as
============  ================
not attached  Tape not ready
end of file   Bad tape
OS I/O error  Fatal tape error
============  ================

TQK50 TMSCP tape controller (TQ)
""""""""""""""""""""""""""""""""
The TQ controller simulates the TQK50 TMSCP tape controller.
TQ options include the ability to set units write enabled or write locked,
and to specify the controller type and tape length::

    SET TQn LOCKED                Set unit n write locked
    SET TQn WRITEENABLED          Set unit n write enabled
    SET TQ TK50                   Set controller type to TK50
    SET TQ TK70                   Set controller type to TK70
    SET TQ TU81                   Set controller type to TU81
    SET TQ TKUSER{=n}             Set controller type to TK50 with
                                  tape capacity of n MB

User-specified capacity must be between 50 and 2000 MB.

Regardless of the controller type,
individual units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET TQn CAPAC=m               Set unit n capacity to m MB (0 = unlimited)
    SHOW TQn CAPAC                Show unit n capacity in MB

Drive units have changeable unit numbers.
Unit numbers can be changed with::

    SET TQn UNIT=val              Set unit plug value

Device TQ has 4 units (TQ0, TQ1, TQ2 and TQ3) which have unique MSCP unit numbers (0, 1, 2 and 3).

The TQ controller supports the ``BOOT`` command.
In a Unibus system, the TQ supports 18b addressing.
In a Qbus system, the TQ supports 22b addressing.

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

===============  ====  ==========================================================
Name             Size  Comments
===============  ====  ==========================================================
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
``PLUG[0:3]``    16    Unit plug value, units 0 to 3
``POS[0:3]``     32    Tape position, units 0 to 3
``OBJP[0:3]``    32    Object position, units 0 to 3
``INT``          1     Interrupt request
``ITIME``        1     Response time for initialization steps (except for step 4)
``QTIME``        24    Response time for 'immediate' packets
``XTIME``        24    Response time for data transfers
``PKTS[33*32]``  16    Packet buffers, 33W each, 32 entries
===============  ====  ==========================================================

Some DEC operating systems, notably RSX11M/M+, are very sensitive to the timing parameters.
Changing the default values may cause M/M+ to crash on boot or to hang during operation.

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

DC11 additional terminal interfaces (DCI/DCO)
"""""""""""""""""""""""""""""""""""""""""""""
For very early system programs,
the PDP-11 simulator supports up to sixteen additional DC11 terminal interfaces.
The additional terminals consist of two independent devices, DCI and DCO.
The entire set is modeled as a terminal multiplexer, with DCI as the master controller.
The additional terminals perform input and output through Telnet sessions connected to a user-specified port.
The number of lines is specified with a ``SET`` command::

    SET DCIX LINES=n              Set number of additional lines to n [1-16]

The ``ATTACH`` command specifies the port to be used::

    ATTACH DCIX <port>            Set up listening port

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.
The additional terminals are disabled by default.

The additional terminals can be set to one of four modes: ``UC``, ``7P``, ``7B``, or ``8B``.

======  ======================  =====================================
Mode    Input characters        Output characters
======  ======================  =====================================
``UC``  Lowercase converted     Lowercase converted to uppercase,
        to uppercase,           high-order bit cleared,
        high-order bit cleared  Non-printing characters suppressed
``7P``  High-order bit cleared  High-order bit cleared,
                                non-printing characters suppressed
``7B``  High-order bit cleared  High-order bit cleared
``8B``  No changes              No changes
======  ======================  =====================================

The default mode is ``7P``.
In addition, each line can be configured to behave as though it was attached to a dataset, or hardwired to a terminal::

    SET DCOn DATASET              Simulate attachment to a dataset (modem)
    SET DCOn NODATASET            Simulate direct attachment to a terminal

Finally, each line supports output logging.
The ``SET DCOn LOG`` command enables logging on a line::

    SET DCOn LOG=filename         Log output of line n to filename

The ``SET DCOn NOLOG`` command disables logging and closes the open log file, if any.

Once DCI is attached and the simulator is running,
the terminals listen for connections on the specified port.
They assume that the incoming connections are Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a ``SET DCI DISCONNECT`` command,
or a ``DETACH DCI`` command.

Other special commands::

    SHOW DCI CONNECTIONS          Show current connections
    SHOW DCI STATISTICS           Show statistics for active connections
    SET DCOn DISCONNECT           Disconnects the specified line

The input device (DCI) implements these registers:

=============  ====  ===========================================
Name           Size  Comments
=============  ====  ===========================================
``CSR[0:15]``  16    Input control/stats register, lines 0 to 15
``BUF[0:15]``  16    Input buffer, lines 0 to 15
``IREQ``       16    Interrupt requests, lines 0 to 15
=============  ====  ===========================================

The output device (DCO) implements these registers:

==============  ====  ====================================================
Name            Size  Comments 
==============  ====  ====================================================
``CSR[0:15]``   16    Input control/stats register, lines 0 to 15
``BUF[0:15]``   8     Input buffer, lines 0 to 15
``IREQ``        16    Interrupt requests, lines 0 to 15
``TIME[0:15]``  31    Time from I/O initiation to interrupt, lines 0 to 15
==============  ====  ====================================================

The additional terminals do not support save and restore.
All open connections are lost when the simulator shuts down or DCI is detached.

KL11/DL11 additional terminal interfaces (DLI/DLO)
""""""""""""""""""""""""""""""""""""""""""""""""""
The PDP-11 simulator supports up to sixteen additional KL11/DL11 terminal interfaces. 
The additional terminals consist of two independent devices, DLI and DLO.
The entire set is modeled as a terminal multiplexer, with DLI as the master controller.
The additional terminals perform input and output through Telnet sessions connected to a user-specified port.
The number of lines is specified with a ``SET`` command::

    SET DLI LINES=n               Set number of additional lines to n [1-16]

The ``ATTACH`` command specifies the port to be used::

    ATTACH DLI <port>             Set up listening port

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.
The additional terminals are disabled by default.

The additional terminals can be set to one of four modes: ``UC``, ``7P``, ``7B``, or ``8B``.

======  ======================  ==================================
Mode    Input characters        Output characters
======  ======================  ==================================
``UC``  Lowercase converted     Lowercase converted to uppercase,
        to uppercase,           high-order bit cleared,
        high-order bit cleared  non-printing characters suppressed
``7P``  High-order bit cleared  High-order bit cleared,
                                non-printing characters suppressed
``7B``  High-order bit cleared  High-order bit cleared
``8B``  No changes              No changes
======  ======================  ==================================

 ::

    SET DLOn UC                   Set UC mode
    SET DLOn 7P                   Set 7P mode
    SET DLOn 7B                   Set 7B mode
    SET DLOn 8B                   Set 8B mode

The default mode is ``UC``.
In addition,
each line can be configured to behave as though it was attached to a dataset,
or hardwired to a terminal::

    SET DLOn DATASET              Simulate attachment to a dataset (modem)
    SET DLOn NODATASET            Simulate direct attachment to a terminal

Finally, each line supports output logging.
The ``SET DLOn LOG`` command enables logging on a line::

    SET DLOn LOG=filename         Log output of line n to filename

The ``SET DLOn NOLOG`` command disables logging and closes the open log file, if any.

Once DLI is attached and the simulator is running,
the terminals listen for connections on the specified port.
They assume that the incoming connections are Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a ``SET DLI DISCONNECT`` command,
or a ``DETACH DLI`` command.

Other special commands::

    SHOW DLI CONNECTIONS          Show current connections
    SHOW DLI STATISTICS           Show statistics for active connections
    SET DLOn DISCONNECT           Disconnects the specified line

The input device (DLI) implements these registers:

==============  ====  ===========================================
Name            Size  Comments
==============  ====  ===========================================
``CSR[0:15]``   16    Input control/stats register, lines 0 to 15
``BUF[0:15]``   16    Input buffer, lines 0 to 15
``IREQ``        16    Receive interrupt requests, lines 0 to 15
``DSI``         16    dataset interrupt requests, lines 0 to 15
==============  ====  ===========================================

The output device (DLO) implements these registers:

==============  ====  ====================================================
Name            Size  Comments
==============  ====  ====================================================
``CSR[0:15]``   16    Input control/stats register, lines 0 to 15
``BUF[0:15]``   8     Input buffer, lines 0 to 15
``IREQ``        16    Interrupt requests, lines 0 to 15
``TIME[0:15]``  31    Time from I/O initiation to interrupt, lines 0 to 15
==============  ====  ====================================================

The additional terminals do not support save and restore.
All open connections are lost when the simulator shuts down or DLO is detached.

DZ11 terminal multiplexer (DZ)
""""""""""""""""""""""""""""""
The DZ11 is an 8-line terminal multiplexer.
Up to 4 DZ11's (32 lines) are supported. 
The number of lines can be changed with the command ::

    SET DZ LINES=n                Set line count to n

The line count must be a multiple of 8, with a maximum of 32.

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
The connection remains open until disconnected by the simulated program, the Telnet client,
a ``SET DZ DISCONNECT`` command, or a ``DETACH DZ`` command.

Other special DZ commands::

    SHOW DZ CONNECTIONS           Show current connections
    SHOW DZ STATISTICS            Show statistics for active connections
    SET DZ DISCONNECT=linenumber  Disconnects the specified line

The DZ11 implements these registers:

==============  ====  =============================================
Name            Size  Comments
==============  ====  =============================================
``CSR[0:3]``    16    Control/status register, boards 0 to 3
``RBUF[0:3]``   16    Receive buffer, boards 0 to 3
``LPR[0:3]``    16    Line parameter register, boards 0 to 3
``TCR[0:3]``    16    Transmission control register, boards 0 to 3
``MSR[0:3]``    16    Modem status register, boards 0 to 3
``TDR[0:3]``    16    Transmit data register, boards 0 to 3
``SAENB[0:3]``  1     Silo alarm enabled, boards 0 to 3
``RXINT``       4     Receive interrupts, boards 3..0
``TXINT``       4     Transmit interrupts, boards 3..0
``MDMTCL``      1     Modem control enabled
``AUTODS``      1     Autodisconnect enabled
==============  ====  =============================================

The DZ11 does not support save and restore.
All open connections are lost when the simulator shuts down or the DZ is detached.

DHQ11 terminal multiplexer (VH)
"""""""""""""""""""""""""""""""
The DHQ11 is an 8-line terminal multiplexer for Qbus systems.
Up to 4 DHQ11's are supported.

The DHQ11 is a programmable asynchronous terminal multiplexer.
It has two programming modes: DHV11 and DHU11.
The register sets are compatible with these devices.
For transmission, the DHQ11 can be used in either DMA or programmed I/O mode.
For reception, there is a 256-entry FIFO for received characters, dataset status changes, and diagnostic information,
and a programmable input interrupt timer (in DHU mode).
The device supports 16-, 18-, and 22-bit addressing.
The DHQ11 can be programmed to filter and/or handle XON/XOFF characters independently of the processor.
The DHQ11 supports programmable bit width (between 5 and 8) for the input and output of characters.

The DHQ11 has a rocker switch for determining the programming mode.
By default, the DHV11 mode is selected, though DHU11 mode is recommended for applications that can support it.
The VH controller may be adjusted on a per controller basis as follows::

    SET VHn DHU                   Use the DHU programming mode and registers
    SET VHn DHV                   Use the DHV programming mode and registers

DMA output is supported.
In a real DHQ11, DMA is not initiated immediately upon receipt of ``TX.DMA.START`` but is dependent upon some internal processes.
The VH controller mimics this behavior by default.
It may be desirable to alter this and start immediately,
though this may not be compatible with all operating systems and diagnostics.
You can change the behavior of the VH controller as follows::

    SET VHn NORMAL                Use normal DMA procedures
    SET VHn FASTDMA               Set DMA to initiate immediately

The terminal lines perform input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the port to be used::

    ATTACH VH <port>              Set up listening port

where ``<port>`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.
This port is the point of entry for all lines on all VH controllers.

The number of lines can be changed with the command ::

    SET VH LINES=n                Set line count to n

The line count must be a multiple of 8, with a maximum of 32.

Modem and auto-disconnect support may be set on an individual controller basis.
The ``SET MODEM`` command directs the controller to report modem status changes to the computer.
The ``SET HANGUP`` command turns on active disconnects (disconnect session if computer clears Data Terminal Ready). ::

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

The DHQ11 implements these registers, though not all can be examined from SCP:

=============  ====  ======================================
Name           Size  Comments
=============  ====  ======================================
``CSR[0:3]``   16    Control/status register, boards 0 to 3
``RBUF[0:3]``  16    Receive buffer, boards 0 to 3
``LPR[0:3]``   16    Line parameter register, boards 0 to 3
``RXINT``      4     Receive interrupts, boards 3..0
``TXINT``      4     Transmit interrupts, boards 3..0
=============  ====  ======================================

    [more to be described...]

The DHQ11 does not support save and restore.
All open connections are lost when the simulator shuts down or the VH is detached.

Ethernet controllers
--------------------

DELQA-T/DELQA/DEQNA Qbus Ethernet controllers (XQ, XQB)
"""""""""""""""""""""""""""""""""""""""""""""""""""""""
The simulator implements two DELQA-T/DELQA/DEQNA Qbus Ethernet controllers (XQ, XQB).
Initially, XQ is enabled, and XQB is disabled.
Options allow control of the MAC address, the controller mode, and the sanity timer. ::

    SET XQ MAC=<mac-address>      ex. 08-00-2B-AA-BB-CC
    SHOW XQ MAC

These commands are used to change or display the MAC address.
``<mac-address>`` is a valid Ethernet MAC, delimited by dashes or periods.
The controller defaults to ``08-00-2B-AA-BB-CC``,
which should be sufficient if there is only one SIMH controller on your LAN.
Two cards with the same MAC address will see each other's packets, resulting in a serious mess. ::

    SET XQ TYPE={DEQNA|[DELQA]|DELQA-T}
    SHOW XQ TYPE

These commands are used to change or display the controller mode.
DELQA mode is better and faster but may not be usable by older or non-DEC OS's.
Also, be aware that DEQNA mode is not supported by many modern OS's.
The DEQNA-LOCK mode of the DELQA card is emulated by setting the the controller to DEQNA — there is no need for a separate mode.
DEQNA-LOCK mode behaves exactly like a DEQNA, except for the operation of the VAR and MOP processing. ::

    SET  XQ SANITY={ON|[OFF]}
    SHOW XQ SANITY

These commands change or display the INITIALIZATION sanity timer
(DEQNA jumper W3/DELQA switch S4).
The INITIALIZATION sanity timer has a default timeout of 4 minutes, and cannot be turned off, just reset.
The normal sanity timer can be set by operating system software regardless of the state of this switch.
Note that only the DEQNA (or the DELQA in DEQNA-LOCK mode (=DEQNA)) supports the sanity timer —
it is ignored by a DELQA in Normal mode, which uses switch S4 for a different purpose. ::

    SET  XQ POLL={DEFAULT|4..2500}
    SHOW XQ POLL

These commands change or display the service polling timer.
The polling timer is calibrated to run the service thread 200 times per second.
This value can be changed to accommodate particular system requirements for more (or less) frequent polling. ::

    SHOW XQ STATS

This command will display the accumulated statistics for the simulated Ethernet controller.

To access the network, the simulated Ethernet controller must be attached to a real Ethernet interface::

    ATTACH XQ0 {ethX|<device_name>}           ex. eth0 or /dev/era0
    SHOW XQ ETH

where ``X`` in ``ethX`` is the number of the Ethernet controller to attach, or the real device name.
The ``X`` number is system-dependent.
If you only have one Ethernet controller, the number will probably be 0.
To find out what your system thinks the Ethernet numbers are, use the ``SHOW XQ ETH command``.
The device list can be quite cryptic, depending on the host system, but is probably better than guessing.
If you do not attach the device, the controller will behave as though the Ethernet cable were unplugged.

XQ and XQB have the following registers:

========  ====  ==================================
Name      Size  Comments
========  ====  ==================================
``SA0``   16    Station address word 0
``SA1``   16    Station address word 1
``SA2``   16    Station address word 2
``SA3``   16    Station address word 3
``SA4``   16    Station address word 4
``SA5``   16    Station address word 5
``RBDL``  32    Receive buffer descriptor list
``XBDL``  32    Trans(X)mit buffer descriptor list
``CSR``   16    Control status register
``VAR``   16    Vector address register
``INT``   1     Interrupt request flag
========  ====  ==================================

One final note: because of its asynchronous nature,
the XQ controller is not limited to the ~1.5Mbit/sec of the real DEQNA/DELQA controllers,
nor the 10Mbit/sec of a standard Ethernet.
Attach it to a Fast Ethernet (100 Mbit/sec) card, and "Feel the Power!" 😀

DELUA/DEUNA Unibus Ethernet controllers (XU, XUB)
"""""""""""""""""""""""""""""""""""""""""""""""""
The simulator implements two DELUA/DEUNA Unibus Ethernet controllers (XU, XUB).
Its operation is analogous to the DELQA/DEQNA controller.

CR11/CD11 card reader (CR)
""""""""""""""""""""""""""
The card reader (CR) implements a single controller (either the CR11 or the CD11)
and card reader (e.g., Documation M200, GDI Model 100)
by reading a file and presenting lines or cards to the simulator.
Card decks may be represented by plain text ASCII files, card image files, or column binary files.
The CR11 controller is also compatible with the CM11-F, CME11, and CMS11.

Card image files are a file format designed by Douglas W. Jones at the University of Iowa to support the interchange of card deck data.
These files have a much richer information carrying capacity than plain ASCII files.
Card Image files can contain such interchange information as card-stock color, corner cuts, special artwork, as well as the binary punch data representing all 12 columns.
Complete details on the format, as well as sample code, are available at `Prof. Jones's site <http://homepage.divms.uiowa.edu/~jones/cards/>`_.

The card reader can be configured to support either of the two controllers supported by DEC::

    SET CR CR11                   Set controller type to CR11
    SET CR CD11                   Set controller type to CD11

The controller type must be set before attaching a virtual card deck to the device.
You may NOT change controller type once a file is attached.

The primary differences are summarized in the table below.
By default, the CR11 simulation is selected.

=============  ====================  ============
\              CR11                  CD11
=============  ====================  ============
BR             6                     4
Registers      4                     3
Data transfer  BR                    DMA
Card rate      200-600               1000-1200
Hopper cap.    <= 1000               1000-2250
Cards          Mark-sense & punched  Punched only
=============  ====================  ============

Examples of the CR11 include the M8290 and M8291 (CMS11).
All card readers use a common vector at 0230 and CSR at 177160.

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

DEC operating systems used a variety of methods to determine the end of a deck,
recognizing that 'hopper empty' does not necessarily mean the end of a deck.
Below is a summary of the various operating system conventions for signaling end of deck:

=======  ========================================================
RT-11:   ``12-11-0-1-6-7-8-9`` punch in column 1
RSTS/E:  ``12-11-0-1`` or ``12-11-0-1-6-7-8-9`` punch in column 1
RSX:     ``12-11-0-1-6-7-8-9`` punch
VMS:     ``12-11-0-1-6-7-8-9`` punch in first 8 columns
TOPS:    ``12-11-0-1`` or ``12-11-0-1-6-7-8-9`` punch in column 1
=======  ========================================================

Using the AUTOEOF setting, the card reader can be set to automatically generate an EOF card consisting of the ``12-11-0-1-6-7-8-9`` punch in columns 1-8.
When set to CD11 mode, this switch also enables automatic setting of the EOF bit in the controller after the EOF card has been processed.
[The CR11 does not have a similar capability].
By default AUTOEOF is enabled. ::

    SET CR AUTOEOF
    SET CR NOAUTOEOF

The default card reader rate for the CR11 is 285 cpm,
while the default rate for the CD11 is 1000 cpm.
The reader rate can be set to its default value or to anywhere in the range 200..1200 cpm.
This rate may be changed while the unit is attached. ::

    SET CR RATE={DEFAULT|200..1200}

It is standard operating procedure for operators to load a card deck and press the momentary action RESET button to clear any error conditions and alert the processor that a deck is available to read.
Use the following command to simulate pressing the card reader RESET button, ::

    SET CR RESET

Another common control of physical card readers is the STOP button.
An operator could use this button to finish the read operation for the current card and terminate reading a deck early.
Use the following command to simulate pressing the card reader STOP button. ::

    SET CR STOP

The simulator does not support the ``BOOT`` command.
The simulator does not stop on file I/O errors.
Instead the controller signals a reader check to the CPU.

The CR controller implements these registers:

==========  ====  ====================================
Name        Size  Comments
==========  ====  ====================================
``BUF``     8     ASCII value of last column processed
``CRS``     16    CR11 status register
``CRB1``    16    CR11 12-bit Hollerith character
``CRB2``    16    CR11 8-bit compressed character
``CRM``     16    CR11 maintenance register
``CDST``    16    CD11 control/status register
``CDCC``    16    CD11 column count
``CDBA``    16    CD11 current bus address
``CDDB``    16    CD11 data buffer, 2nd status
``BLOWER``  2     Blower state value
``INTCR``   1     Interrupt pending flag (CR11)
``INTCD``   1     Interrupt pending flag (CD11)
``ERR``     1     Error flag (CRS<15>)
``IE``      1     Interrupt enable flag (CRS<6>)
``POS``     32    File position - do not alter
``TIME``    24    Delay time between columns
==========  ====  ====================================

The CD11 simulation includes the Rev. J modification to make the CDDB act as a second status register during non-data transfer periods.

Arithmetic options
------------------

KE11A extended arithmetic option (KE)
"""""""""""""""""""""""""""""""""""""
The KE11A extended arithmetic option (KE) provides multiply, divide, normalization, and multi-bit shift capability on Unibus PDP-11's that lack the EIS instruction set.
In practice, it was only sold with the PDP-11/20.
The KE is disabled by default.

The KE implements these registers:

======  ====  ===================
Name    Size  Comments
======  ====  ===================
``AC``  16    Accumulator
``MQ``  16    Multiplier-quotient
``SC``  6     Shift count
``SR``  8     Status register
======  ====  ===================

KG11A communications arithmetic option (KG)
"""""""""""""""""""""""""""""""""""""""""""
The KG11-A is a programmed I/O, non-interrupting, dedicated arithmetic processor for the Unibus.
The device is used to compute the block check character (BCC) over a block of data, typically in data communication applications.
The KG11 can compute three different Cyclic Redundancy Check (CRC) polynomials (CRC-16, CRC-12, CRC-CCITT) and two Longitudinal Redundancy Checks (LRC, Exclusive-OR; LRC-8, LRC-16).
Up to eight units may be contiguously present in a single machine and are all located at fixed addresses.
This simulation implements all functionality of the device including the ability to single step computation of the BCC.
The KG is disabled by default.

The KG11 supports the following options::

    SET KG UNITS=n              Set the number of units [0-8]
    SET KG DEBUG={opt,opt…}     Set the debugging options:
                                REG - any time a register is touched
                                POLY - any time the polynomial is changed
                                CYCLE - each cycle computing the polynomial

The KG11 implements the following registers, replicated for each unit:

================  ====  =================================
Name              Size  Comments
================  ====  =================================
``SR[0:7]``       16    Control and status register; R/W
``BCC[0:7]``      16    Result block check character; R/O
``DR[0:7]``       16    Input data register; W/O
``PULSCNT[0:7]``  16    Polynomial cycle stage
================  ====  =================================

Symbolic display and input
==========================
The PDP-11 simulator implements symbolic display and input.
Display is controlled by command-line switches:

-a  Display as ASCII character
-c  Display as two packed ASCII characters
-m  Display instruction mnemonics

Input parsing is controlled by the first character typed in or by command-line switches:

===============  ===========================
``'`` or ``-a``  ASCII character
``"`` or ``-c``  Two packed ASCII characters
Alphabetic       Instruction mnemonic
Numeric          Octal number
===============  ===========================

Instruction input uses standard PDP-11 assembler syntax.
There are sixteen instruction classes:

==============  ======================  ===================================  ==========
Class           Operands                Examples                             Comments
==============  ======================  ===================================  ==========
no operands     none                    ``HALT``, ``RESET``
3b literal      literal [0 to 7]        ``SPL``
6b literal      literal [0-077]         ``MARK``
8b literal      literal [0-0377]        ``EMT``, ``TRAP``
register        register                ``RTS``
sop             specifier               ``SWAB``, ``CLR``, ``ASL``
reg-sop         register, specifier     ``JSR``, ``XOR``, ``MUL``
fop             flt specifier           ``ABSf``, ``NEGf``
ac-fop          flt reg, flt specifier  ``LDf``, ``MULf``
ac-sop          flt reg, specifier      ``LDEXP``, ``STEXP``
ac-moded sop    flt reg, specifier      ``LDCif``, ``STCfi``
dop             specifier, specifier    ``MOV``, ``ADD``, ``BIC``
cond branch     address                 ``BR``, ``BCC``, ``BNE``
sob             register, address       ``SOB``
cc clear        cc clear instructions   ``CLC``, ``CLV``, ``CLZ``, ``CLN``   Combinable
cc set          cc set instructions     ``SEC``, ``SEV``, ``SEZ``, ``SEN``   Combinable
==============  ======================  ===================================  ==========

For floating point opcodes, F and D variants, and I and L variants, may be specified regardless of the state of FPS.

The syntax for specifiers is as follows:

===============  =========  ===================  ========================
Syntax           Specifier  Displacement         Comments
===============  =========  ===================  ========================
``Rn``           ``0n``
``Fn``           ``0n``     —                    Only in flt reg classes
``(Rn)``         ``1n``     —
``@(Rn)``        ``7n``     ``0``                Equivalent to ``@0(Rn)``
``(Rn)+``        ``2n``     —
``@(Rn)+``       ``3n``     —
``-(Rn)``        ``4n``     —
``@-(Rn)``       ``5n``     —
``{+/-}d(Rn)``   ``6n``     ``{+/-}d``
``@{+/-}d(Rn)``  ``7n``     ``{+/-}d``
``#n``           ``27``     ``n``
``@#n``          ``37``     ``n``
``.+/-n``        ``67``     ``+/-n - 4``
``@.+/-n``       ``77``     ``+/-n - 4``
``{+/-}n``       ``67``     ``{+/-}n - PC - 4``  If on disk, 37 and ``n``
``@{+/-}n``      ``77``     ``{+/-}n - PC - 4``  If on disk, invalid
===============  =========  ===================  ========================

The UC15
========
The UC15 is a special, limited configuration of the PDP-11 simulator for use as the I/O processor in a PDP-15/76 system.
It is configured as follows:

================  =====================================================
Device name(s)    Simulates
================  =====================================================
``CPU``           PDP-11/05 CPU with 8KB-24KB of memory
``PTR``, ``PTP``  PC11 paper tape reader/punch
``TTI``, ``TTO``  DL11 console terminal
``CR``            CR11 card reader
``LPT``           LP11 line printer
``CLK``           KW11-L line frequency clock
``RK``            RK11/RK05 cartridge disk controller with eight drives
``UCA``, ``UCB``  DR11-C parallel interfaces
================  =====================================================

The card reader is disabled initially and is not supported by the default release of PIREX, the I/O executive that runs in the UC15.

The CPU model cannot be changed.
While memory size can be changed,
PIREX is configured for 16KB of memory.

The PDP-15/76 configuration requires the shared memory facility,
which is presently implemented only for Windows and Linux.

DR11 parallel interfaces (UCA, UCB)
-----------------------------------
The UC15 talks to the PDP-15's DR15 interface over a pair of DR11-C interfaces called UCA and UCB.
UCA implements these registers:

========  ====  ========================
Name      Size  Comments
========  ====  ========================
``CSR``   16    Control/status register
``BUF``   16    Input buffer
``APID``  1     CSR<7>, API done
``IE``    1     CSR<6>, interrupt enable
``POLL``  10    Polling interval
========  ====  ========================

UCB implements these registers:

========  ====  ==============================
Name      Size  Comments
========  ====  ==============================
``CSR``   16    Control/status register
``BUF``   16    Input buffer
``NTCB``  1     CSR<7>, new task control block
``IE``    1     CSR<6>, interrupt enable
========  ====  ==============================

UCA and UCB implement the ``SET``/``SHOW ADDRESS`` and ``SET``/``SHOW VECTOR`` commands,
but if the address or vector of either interface is changed,
PIREX will not run correctly.
