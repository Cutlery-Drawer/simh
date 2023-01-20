.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

*********************************
Interdata 16b/32b simulator usage
*********************************
:Date: 2022-07-12
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

This memorandum documents the Interdata 16b and 32b simulators.

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

``sim/interdata/``
    | ``id_defs.h``
    | ``id16_cpu.c``    [\ ``id32_cpu.c``\ ]
    | ``id16_dboot.c``  [\ ``id32_dboot.c``\ ]
    | ``id_dp.c``
    | ``id_fd.c``
    | ``id_fp.c``
    | ``id_idc.c``
    | ``id_io.c``
    | ``id_lp.c``
    | ``id_mt.c``
    | ``id_pas.c``
    | ``id_pt.c``
    | ``id_tt.c``
    | ``id_ttp.c``
    | ``id_uvc.c``
    | ``id16_sys.c`` [\ ``id32_sys.c``\ ]

Interdata features
==================
The Interdata simulator includes simulators for a variety of
16b (I3, I4, I5, 70, 80, 7/16, 8/16, 8/16E) and
32b (7/32, 8/32) models.
This is by no means a complete sampling of all the variations in the Interdata/Perkin-Elmer family.
The 32b family included options for special communications instructions (7/32C, 8/32C),
as well as a later extension for virtual memory (3200 series).

The Interdata simulator is configured as follows:

==================  ===============================================================
Device names        Simulates
==================  ===============================================================
``CPU - 16b``       | Interdata 3, 4, 5, 70, 80, 7/16, or 8/16 CPU with 64KB memory
                    | Interdata 8/16E CPU with 256KB memory
``CPU - 32b``       | Interdata 7/32 or 8/32 CPU with 1MB memory;
                    | 8/32 supports 2 or 8 register banks
|SELCH|_            Selector channel (1-4)
|PT|_               Paper tape reader/punch
|TT|_               Console terminal, Teletype interface
|TTP|_              Console terminal, PASLA interface
|LFC|_              Line frequency clock
|PIC|_              Programmable interval clock
|LPT|_              Line printer
|FD|_               Floppy disk
|DP|_               2.5MB/10MB cartridge disk with four disk drives
|DM|_               Mass storage module (``MSM``)/intelligent (``IDC``)
                    disk controller with four disk drives
``MT``              Magnetic tape
``PAS``             Programmable asynchronous line controller
``PASL``            Programmable asynchronous lines, up to 32
==================  ===============================================================

The Interdata simulator implements two unique stop conditions:

* Decode of an undefined instruction, and ``STOP_INST`` is set
* Runaway carriage control tape in the line printer

.. |DUMP| replace:: ``DUMP``
.. |LOAD| replace:: ``LOAD``
.. _DUMP:
.. _LOAD:

The ``LOAD`` command is used to load a carriage control tape for the line printer.
The ``DUMP`` command is used to dump a contiguous portion of memory as a self-loading bootstrap paper tape.
The syntax for the ``DUMP`` command is::

    DUMP <filename> lowaddr-highaddr

The low address must be greater than or equal to X'D0'.

Devices are assigned their default device numbers,
as documented in the Interdata literature.
Device numbers can be changed by the command::

    SET <device> DEVNO=num

Device number conflicts are not checked until simulation starts.
If there is a device number conflict,
simulation stops immediately with an error message.

Selector channel devices are assigned by default to selector channel 0.
Selector channel assignments can be changed by the command::

    SET <dev> SELCH=num

Selector channel assignments cannot introduce conflicts.

Most devices can be disabled and enabled,
with the commands::

    SET <dev> DISABLED
    SET <dev> ENABLED

All devices are enabled by default.

CPU (16b)
---------
The CPU options include memory size and CPU type::

    SET CPU I3          Interdata 3
                        (base instruction set)
    SET CPU I4          Interdata 4
                        (base + single-precision floating-point)
    SET CPU 716         Interdata 7/16
                        (extended instruction set)
                        (equivalent to Models 5, 70, and 80)
    SET CPU 816         Interdata 8/16
                        (extended + double-precision floating-point)
    SET CPU 816E        Interdata 8/16E
                        (extended + double-precision + expanded memory)
    SET CPU 8K          Set memory size = 8KB
    SET CPU 16K         Set memory size = 16KB
    SET CPU 24K         Set memory size = 24KB
    SET CPU 32K         Set memory size = 32KB
    SET CPU 48K         Set memory size = 48KB
    SET CPU 64K         Set memory size = 64KB
    SET CPU 128K        Set memory size = 128KB (8/16E only)
    SET CPU 256K        Set memory size = 256KB (8/16E only)
    SET CPU CONSINT     Assert console interrupt (7/16, 8/16, and 8/16E only)

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 64KB.

These switches are recognized when examining or depositing in CPU memory:

.. table::
   :class: switches

   ======  =======================================
   ``-a``  Examine/deposit ASCII characters
   ``-b``  Examine/deposit bytes
   ``-c``  Examine/deposit packed ASCII characters
   ``-f``  Examine/deposit fullwords
   ``-d``  Data radix is decimal
   ``-o``  Data radix is octal
   ``-h``  Data radix is hexadecimal
   ``-m``  Examine as instruction mnemonics
   ``-v``  Interpret address as virtual
   ======  =======================================

Packed characters, halfwords, fullwords, and instructions must be aligned on a halfword (16b) boundary.
If an odd address is specified,
the low-order bit is ignored.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

=====================  ====  =================================================================
Name                   Size  Comments
=====================  ====  =================================================================
``PC``                 16    Program counter
``R0``\ ..\ ``R15``    16    General registers
``FR0``\ ..\ ``F14``   32    Single-precision floating-point registers
``D0H``\ ..\ ``D14H``  32    Double-precision floating-point registers, high order
``D0L``\ ..\ ``D14L``  32    Double-precision floating-point registers, low order
``PSW``                16    Processor status word
``CC``                 4     Condition codes, PSW<12:15>
``SR``                 16    Switch register
``DR``                 32    Display register low 16 bits
``DRX``                8     Display register extension
``DRMOD``              1     Display mode
``DRPOS``              2     Display pointer position
``SRPOS``              1     Switch pointer position
``IRQ[0:3]``           32    Interrupt requests
``IEN[0:3]``           32    Interrupt enables
``STOP_INST``          1     Stop on undefined instruction
``STOP_WAIT``          1     Stop if wait state and no I/O events pending
``PCQ[0:63]``          16    PC prior to last branch or interrupt; most recent PC change first
``WRU``                8     Interrupt character
=====================  ====  =================================================================

The CPU detects when the simulator is idle.
When idle,
the simulator does not use any resources on the host system.
Idle detection is controlled by the ``SET IDLE`` and ``SET NOIDLE`` commands::

    SET CPU IDLE          Enable idle detection
    SET CPU NOIDLE        Disable idle detection

Idle detection is disabled by default.
The CPU is considered idle if the ``WAIT STATE`` flag is set in the PSW.

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY       Clear history buffer
    SET CPU HISTORY=0     Disable history
    SET CPU HISTORY=n     Enable history, length = n
    SHOW CPU HISTORY      Print CPU history
    SHOW CPU HISTORY=n    Print first n entries of CPU history

The maximum length for the history is 65536 entries.

CPU (32b)
---------
The CPU options include memory size and CPU type::

    SET CPU 732           Interdata 7/32,
                          single-precision floating-point
    SET CPU DPFP          Interdata 7/32,
                          double-precision floating-point
    SET CPU 832           Interdata 8/32
                          (double-precision floating-point,
                          8 general register sets)
    SET CPU 2RS           Interdata 8/32
                          (double-precision floating-point,
                          2 general register sets)
    SET CPU 64K           Set memory size = 64KB
    SET CPU 128K          Set memory size = 128KB
    SET CPU 256K          Set memory size = 256KB
    SET CPU 512K          Set memory size = 512KB
    SET CPU 1M            Set memory size = 1024KB
    SET CPU CONSINT       Assert console interrupt

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 1024KB.

These switches are recognized when examining or depositing in CPU memory:

.. table::
   :class: switches

   ======  =======================================
   ``-a``  Examine/deposit ASCII characters
   ``-b``  Examine/deposit bytes
   ``-c``  Examine/deposit packed ASCII characters
   ``-w``  Examine/deposit halfwords
   ``-d``  Data radix is decimal
   ``-o``  Data radix is octal
   ``-h``  Data radix is hexadecimal
   ``-m``  Examine as instruction mnemonics
   ``-v``  Interpret address as virtual
   ======  =======================================

Packed characters, halfwords, fullwords, and instructions must be aligned on a halfword (16b) boundary.
If an odd address is specified,
the low-order bit is ignored.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

========================  ======  =================================================================
Name                      Size    Comments
========================  ======  =================================================================
``PC``                    20      Program counter
``R0``\ ..\ ``R15``       32      Active general register set
``GREG[32]``              32      General register sets, 16 × 2
``FR0``\ ..\ ``FR14``     32      Single-precision floating-point registers
``D0H``\ ..\ ``D14H``     32      Double-precision floating-point registers, high order
``D0L``\ ..\ ``D14L``     32      Double-precision floating-point registers, low order
``PSW``                   16      Processor status word
``CC``                    4       Condition codes, ``PSW<12:15>``
``SR``                    16      Switch register
``DR``                    32      Display register low 16 bits
``DRX``                   8       Display register extension (x/16 only)
``DRMOD``                 1       Display mode
``DRPOS``                 2       Display pointer position
``SRPOS``                 1       Switch pointer position
``MACREG[0:15]``          32      Memory access controller segment registers
``MACSTA``                5       Memory access controller interrupt status
``IRQ[0:3]``              32      Interrupt requests
``IEN[0:3]``              32      Interrupt enables
``STOP_INST``             1       Stop on undefined instruction
``STOP_WAIT``             1       Stop if wait state and no I/O events pending
``PCQ[0:63]``             20      PC prior to last branch or interrupt; most recent PC change first
``WRU``                   8       Interrupt character
========================  ======  =================================================================

The CPU detects when the simulator is idle.
When idle,
the simulator does not use any resources on the host system.
Idle detection is controlled by the ``SET IDLE`` and ``SET NOIDLE`` commands::

    SET CPU IDLE          Enable idle detection
    SET CPU NOIDLE        Disable idle detection

Idle detection is disabled by default.
The CPU is considered idle if the ``WAIT STATE`` flag is set in the ``PSW``.

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY       Clear history buffer
    SET CPU HISTORY=0     Disable history
    SET CPU HISTORY=n     Enable history, length = n
    SHOW CPU HISTORY      Print CPU history
    SHOW CPU HISTORY=n    Print first n entries of CPU history

The maximum length for the history is 65536 entries.

.. _Selector channel:
.. |SELCH|  replace:: ``SELCH``
.. |SELCH0| replace:: ``SELCH0``
.. |SELCH1| replace:: ``SELCH1``
.. |SELCH2| replace:: ``SELCH2``
.. |SELCH3| replace:: ``SELCH3``
.. _SELCH:
.. _SELCH0:
.. _SELCH1:
.. _SELCH2:
.. _SELCH3:

Selector channel (|SELCH0|, |SELCH1|, |SELCH2|, |SELCH3|)
---------------------------------------------------------
An Interdata system can have 1 to 4 selector channels
(|SELCH0|, |SELCH1|, |SELCH2|, |SELCH3|).
The default number of channels is 2.
The number of channels can be changed with the command::

    SET SELCH CHANNELS=num

All the state for a selector channel can be displayed with the command::

    SHOW SELCH num

The selector channels implement these registers:

============   ====  ==================================================
Name           Size  Comments
============   ====  ==================================================
``SA[0:3]``    20    Start address, channels 0 to 3
``EA[0:3]``    20    End address, channels 0 to 3
``CMD[0:3]``   8     Command, channels 0 to 3
``DEV[0:3]``   8     Active device, channels 0 to 3
``RDP[0:3]``   2     Read byte pointer, channels 0 to 3
``WDC[0:3]``   3     Write data counter, channels 0 to 3
``IREQ``       4     Interrupt requests; right-to-left, channels 0 to 3
``IENB``       4     Interrupt enables; right-to-left, channels 0 to 3
============   ====  ==================================================

Programmed I/O devices
----------------------

.. _Paper Tape Reader/Punch:
.. |PT| replace:: ``PT``
.. _PT:

Paper Tape Reader/Punch (|PT|)
""""""""""""""""""""""""""""""
The paper tape reader and punch (|PT| units 0 and 1) read data from or write data to disk files.
The ``RPOS`` and ``PPOS`` registers specify the number of the next data item to be read and written,
respectively.
Thus, by changing ``RPOS`` or ``PPOS``,
the user can backspace or advance these devices.

The paper tape reader supports the ``BOOT`` command.
``BOOT PTR`` copies the so-called '50 loader' into memory and starts it running.

The paper tape controller implements these registers:

==============  ====  =====================================
Name            Size  Comments
==============  ====  =====================================
``RBUF``        8     Reader buffer
``RPOS``        32    Reader position in the input file
``RTIME``       24    Time from reader start to interrupt
``RSTOP_IOE``   1     Reader stop on I/O error
``PBUF``        8     Punch buffer
``PPOS``        32    Punch position in the output file
``PTIME``       24    Time from punch start to interrupt
``PSTOP_IOE``   1     Punch stop on I/O error
``IREQ``        1     Paper tape interrupt request
``IENB``        1     Paper tape interrupt enable
``IARM``        1     Paper tape interrupt armed
``RD``          1     Paper tape read/write mode
``RUN``         1     Paper tape running
``SLEW``        1     Paper tape reader slew mode
``EOF``         1     Paper tape reader end-of-file
==============  ====  =====================================

Error handling is as follows:

.. |in,out| replace:: ``in``,\ ``out``

+----------+--------------+--------------+-----------------------+
| Type     | Error        | ``STOP_IOE`` | Processed as          |
+==========+==============+==============+=======================+
| |in,out| | Not attached | 1            | Report error and stop |
|          |              +--------------+-----------------------+
|          |              | 0            | Out-of-tape           |
+----------+--------------+--------------+-----------------------+
| ``in``   | End-of-file  | 1            | Report error and stop |
|          |              +--------------+-----------------------+
|          |              | 0            | Out-of-tape           |
+----------+--------------+--------------+-----------------------+
| |in,out| | OS I/O error | x            | Report error and stop |
+----------+--------------+--------------+-----------------------+

.. _Console, Teletype Interface:
.. |TT|  replace:: ``TT``
.. |TT0| replace:: ``TT0``
.. |TT1| replace:: ``TT1``
.. _TT:
.. _TT0:
.. _TT1:

Console, Teletype Interface (|TT|)
""""""""""""""""""""""""""""""""""
The Teletype keyboard (|TT0|) reads from the console keyboard;
the Teletype printer (|TT1|) writes to the simulator console window.
The Teletype units (|TT0|, |TT1|) can be set to one of four modes,
``KSR``, ``7P``, ``7B``, or ``8B``:

=======  ======================  =================================
Mode     Input characters        Output characters
=======  ======================  =================================
``KSR``  Lowercase converted     Lowercase converted to uppercase,
         to uppercase,           high-order bit cleared,
         high-order bit set      non-printing characters suppressed
``7P``   High-order bit cleared  High-order bit cleared,
                                 non-printing characters suppressed
``7B``   High-order bit cleared  High-order bit cleared
``8B``   No changes              No changes
=======  ======================  =================================

Changing the mode of either unit changes both.
The default mode is ``KSR``.

The Teletype has a :html:`<samp class="guilabel"><b>BREAK</b></samp>` key,
which is not present on today's keyboards.
To simulate pressing the break key,
stop the simulator and use the command::

    SET TT BREAK

Break status will be asserted,
and will remain asserted for the interval specified by ``KTIME``.

The Teletype interface implements these registers:

=========  ====  ============================================
Name       Size  Comments
=========  ====  ============================================
``KBUF``   8     Input buffer
``KPOS``   32    Number of characters input
``KTIME``  24    Input polling interval (if 0, the keyboard
                 is polled synchronously with the line clock)
``TBUF``   8     Output buffer
``TPOS``   32    Number of characters output
``TTIME``  24    Time from output start to interrupt
``IREQ``   1     Interrupt request
``IENB``   1     Interrupt enable
``IARM``   1     Interrupt armed
``RD``     1     Read/write mode
``FDPX``   1     Half-duplex
``CHP``    1     Input character pending
=========  ====  ============================================

.. _Console, PASLA Interface:
.. |TTP| replace:: ``TTP``
.. _TTP:

Console, PASLA Interface (|TTP|)
""""""""""""""""""""""""""""""""
Later Interdata system connect the system console via the first PASLA interface rather than the Teletype interface.
The PASLA console can be simulated with a Telnet session on the first PAS line.
Alternately,
the PASLA console can be attached to the simulator console window,
using the |TTP| device in place of |TT|.

To switch the simulator console window to |TTP|,
use the command::

    SET TTP ENABLED or
    SET TT DISABLED

Device |TT| is automatically disabled and device |TTP| is enabled.
To switch the simulator console window back to |TT|,
use the command::

    SET TT ENABLED or
    SET TTP DISABLED

Device |TTP| is automatically disabled and device |TT| is enabled.
If |TTP| is enabled at its default device settings,
the base address for the ``PAS`` multiplexer must be changed::

    SET PAS DEVNO=12

Otherwise,
a device number conflict occurs.

.. |TTP0| replace:: ``TTP0``
.. |TTP1| replace:: ``TTP1``
.. _TTP0:
.. _TTP1:

The PASLA keyboard (|TTP0|) reads from the console keyboard;
the PALSA printer (|TTP1|) writes to the simulator console window.
The PASLA units (|TTP0|, |TTP1|) can be set to one of four modes,
``UC``, ``7P``, ``7B``, or ``8B``:

======  ======================  ==========================================
Mode    Input characters        Output characters
======  ======================  ==========================================
``UC``  Lowercase converted     Lowercase converted to upper case,
        to uppercase,           high-order bit cleared,
        high-order bit cleared  non-printing characters suppressed
``7P``  High-order bit cleared  High-order bit cleared,
                                non-printing characters suppressed
``7B``  High-order bit cleared  High-order bit cleared
``8B``  No changes              No changes
======  ======================  ==========================================

Changing the mode of either unit changes both.
The default mode is ``7B``.

To simulate pressing the break key,
stop the simulator and use the command::

    SET TTP BREAK

Break status will be asserted,
and will remain asserted for the interval specified by ``KTIME``.

The PASLA console interface implements these registers:

=========  ====  ============================================
Name       Size  Comments
=========  ====  ============================================
``CMD``    16    Command register
``STA``    8     Status register
``KBUF``   8     Input buffer
``KPOS``   32    Number of characters input
``KTIME``  24    Input polling interval (if 0, the keyboard
                 is polled synchronously with the line clock)
``KIREQ``  1     Input interrupt request
``KIENB``  1     Input interrupt enabled
``KARM``   1     Input interrupt armed
``CHP``    1     Input character pending
``TBUF``   8     Output buffer
``TPOS``   32    Number of characters output
``TTIME``  24    Time from output start to interrupt
``TIREQ``  1     Output interrupt request
``TIENB``  1     Output interrupt enable
``TIARM``  1     Output interrupt armed
=========  ====  ============================================

.. _Line Printer:
.. |LPT| replace:: ``LPT``
.. _LPT:

Line Printer (|LPT|)
""""""""""""""""""""
The line printer (|LPT|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the printer.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

In addition,
the line printer can be programmed with a carriage control tape.
The ``LOAD`` command loads a new carriage control tape::

    LOAD <file>              Load carriage control tape file

The format of a carriage control tape consists of multiple lines.
Each line contains an optional repeat count,
enclosed in parentheses,
optionally followed by a series of column numbers separated by commas.
Column numbers must be between 0 and 7;
column seven is by convention top of form.
The following are all legal carriage control specifications::

    <blank line>        No punch
    (5)                 5 lines with no punches
    1,5,7               Columns 1, 5, 7 punched
    (10)2               10 lines with column 2 punched
    0                   Column 0 punched

The default form is 1 line long,
with all columns punched.

The line printer implements these registers:

===============  ====  ==============================
Name             Size  Comments
===============  ====  ==============================
``BUF``          7     Last data item processed
``BPTR``         8     Line buffer pointer
``LBUF[0:131]``  7     Line buffer
``VFUP``         8     Vertical forms unit pointer
``VFUL``         8     Vertical forms unit length
``VFUT[0:131]``  8     Vertical forms unit table
``IREQ``         1     Line printer interrupt request
``IENB``         1     Line printer interrupt enable
``IARM``         1     Line printer interrupt armed
``POS``          32    Position in the output file
``CTIME``        24    Character processing time
``STIME``        24    Spacing operation time
``STOP_IOE``     1     Stop on I/O error
===============  ====  ==============================

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

.. _Line Frequency Clock:
.. |LFC| replace:: ``LFC``
.. _LFC:

Line Frequency Clock (|LFC|)
""""""""""""""""""""""""""""
The line frequency clock (|LFC|) frequency can be adjusted as follows::

    SET LFC 60HZ             Set frequency to 60Hz
    SET LFC 50HZ             Set frequency to 50Hz

The default is 60Hz.

The line frequency clock implements these registers:

========  ====  =======================
Name      Size  Comments
========  ====  =======================
``IREQ``  1     Clock interrupt request
``IENB``  1     Clock interrupt enable
``IARM``  1     Clock interrupt armed
``TIME``  24    Clock frequency
========  ====  =======================

The line frequency clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

.. _Programmable Interval Clock:
.. |PIC| replace:: ``PIC``
.. _PIC:

Programmable Interval Clock (|PIC|)
"""""""""""""""""""""""""""""""""""
The programmable interval clock (|PIC|) implements these registers:

========  ====  =======================
Name      Size  Comments
========  ====  =======================
``BUF``   16    Output buffer
``RIC``   16    Reset interval and rate
``CIC``   12    Current interval
``DECR``  10    Current decrement value
``RDP``   1     Read byte select
``OVF``   1     Interval overflow flag
``IREQ``  1     Clock interrupt request
``IENB``  1     Clock interrupt enable
``IARM``  1     Clock interrupt armed
========  ====  =======================

If the interval requested is an exact multiple of 1 millisecond,
the programmable clock auto-calibrates;
if not,
it counts instructions.

.. _Floppy Disk Controller:
.. |FD| replace:: ``FD``
.. _FD:

Floppy Disk Controller (|FD|)
"""""""""""""""""""""""""""""
Floppy disk options include the ability to make units write-enabled or write-locked. ::

    SET FDn LOCKED            Set unit n write locked
    SET FDn WRITEENABLED      Set unit n write enabled

Units can also be set ``ENABLED`` or ``DISABLED``.

The floppy disk supports the ``BOOT`` command.
``BOOT FDn`` copies an autoload sequence into memory and starts it running.

The floppy disk controller implements these registers:

===============  ======  =============================
Name             Size    Comments
===============  ======  =============================
``CMD``          8       Command
``STA``          8       Status
``BUF``          8       Buffer
``LRN``          16      Logical record number
``ESTA[0:5]``    8       Extended status bytes
``DBUF[0:127]``  8       Transfer buffer
``DBPTR``        8       Transfer buffer pointer
``IREQ``         1       Interrupt request
``IENB``         1       Interrupt enabled
``IARM``         1       Interrupt armed
``CTIME``        24      Command response time
``STIME``        24      Seek time, per cylinder
``XTIME``        24      Transfer time, per byte
``STOP_IOE``     1       Stop on I/O error
===============  ======  =============================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Disk not ready        |
+--------------+--------------+-----------------------+

Floppy disk data is buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

.. _Programmable Asynchronous Line Adapters:
.. |PAS|  replace:: ``PAS``
.. |PASL| replace:: ``PASL``
.. _PAS:
.. _PASL:

Programmable Asynchronous Line Adapters (|PAS|, |PASL|)
"""""""""""""""""""""""""""""""""""""""""""""""""""""""
The Programmable Asynchronous Line Adapters (|PAS| and |PASL|) represent,
indistinguishably,
individual PASLA interfaces,
2-line asynchronous multiplexers,
and 8-line asynchronous multiplexers,
with a maximum of 32 lines.
All the lines are modelled as a terminal multiplexer,
with |PAS| as the multiplexer controller,
and |PASL| as the individual lines.
The PASLAs perform input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the port to be used::

    ATTACH PAS <port>          Set up listening port

where port is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

Each line (each unit of |PASL|) can be set to one of four modes,
``UC``, ``7P``, ``7B``, or ``8B``:

======  ======================  ===================================
Mode    Input characters        Output characters
======  ======================  ===================================
``UC``  Lowercase converted     Lowercase converted to uppercase,
        to uppercase,           high-order bit cleared,
        high-order bit cleared  non-printing characters suppressed
``7P``  High-order bit cleared  High-order bit cleared,
                                non-printing characters suppressed
``7B``  High-order bit cleared  High-order bit cleared
``8B``  No changes              No changes
======  ======================  ===================================

Each line (each unit of |PASL|) can also be set for modem control with the command ``SET PASLn DATASET``.
The defaults are ``7B`` mode and ``DATASET`` disabled.
Finally,
each line supports output logging.
The ``SET PASLn LOG`` command enables logging on a line::

    SET PASLn LOG=filename      Log output of line n to filename

The ``SET PASLn NOLOG`` command disables logging and closes the open log file,
if any.

Once ``PAS`` is attached and the simulator is running,
the terminals listen for connections on the specified port.
They assume that the incoming connections are Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a ``SET PAS DISCONNECT`` command,
or a ``DETACH PAS`` command.

Other special PASLA commands::

    SHOW PAS CONNECTIONS            Show current connections
    SHOW PAS STATISTICS             Show statistics for active connections
    SET PASLn DISCONNECT            Disconnects the specified line.

The controller (|PAS|) implements these registers:

==============  ======  =========================================
Name            Size    Comments
==============  ======  =========================================
``STA[0:31]``   8       Status, lines 0 to 31
``CMD[0:31]``   16      Command, lines 0 to 31
``RBUF[0:31]``  8       Receive buffer, lines 0 to 31
``XBUF[0:31]``  8       Transmit buffer, lines 0 to 31
``RIREQ``       32      Receive interrupt requests;
                        right-to-left, lines 0 to 31
``RIENB``       32      Receive interrupt enables
``RARM[0:31]``  1       Receive interrupt armed
``XIREQ``       32      Transmit interrupt requests;
                        right-to-left, lines 0 to 31
``XIENB``       32      Transmit interrupt enables
``XARM[0:31]``  1       Transmit interrupt armed
``RCHP[0:31]``  1       Receiver character present, lines 0 to 31
==============  ======  =========================================

The lines (|PASL|) implements these registers:

==============  ====  ============================
Name            Size  Comments
==============  ====  ============================
``TIME[0:31]``  24    Transmit time, lines 0 to 31
==============  ====  ============================

The additional terminals do not support save and restore.
All open connections are lost when the simulator shuts down or |PAS| is detached.

.. _Cartridge Disk Controller:
.. |DP| replace:: ``DP``
.. _DP:

Cartridge Disk Controller (|DP|)
--------------------------------
Cartridge disk options include the ability to make units write-enabled or write-locked,
and to select the type of drive::

    SET DPn LOCKED           Set unit n write locked
    SET DPn WRITEENABLED     Set unit n write enabled
    SET DPn 2315             Set unit n to 2315 (2.5MB)
    SET DPn 5440             Set unit n to 5440 (10MB)

Units can also be set ``ENABLED`` or ``DISABLED``.

The cartridge disk supports the ``BOOT`` command.
To boot OS16/32,
the hex form of the operating system file's extension must be placed in locations ``7E:7F``.
The disk bootstrap looks for a valid OS16/32 volume descriptor in block 0,
and uses that to locate the volume directory.
It then searches the directory for a filename of the form ``OS16xxxx.hhh`` or ``OS32xxxx.hhh``,
where the ``xxxx`` is ignored and ``hhh`` is the ASCII form of the extension from locations ``7E:7F``.
The 32b bootstrap can also boot Wollongong UNIX;
locations ``7E:7F`` must be 0.
The bootstrap normally boots from the first (removable) platter in a 5440;
to boot from the second (fixed) platter,
use ``BOOT -F``.

All drives have 256 8b bytes per sector.
The other disk parameters are:

=====  =========  ========  =======
Drive  Cylinders  Surfaces  Sectors
=====  =========  ========  =======
2315   203        2         24
5440   408        4         12
=====  =========  ========  =======

The cartridge disk controller implements these registers:

===============  ====  ==================================
Name             Size  Comments
===============  ====  ==================================
``CMD``          3     Current command
``STA``          8     Controller status
``BUF``          8     Controller buffer
``HDSC``         8     Current head/sector select
``CYL``          8     Current cylinder select
``DBUF[0:255]``  8     Transfer buffer
``DBPTR``        16    Transfer buffer point
``DBLNT``        16    Transfer buffer length
``FIRST``        1     First DMA service flag
``IREQ``         5     Interrupt requests; right-to-left,
                       controller, drives 0 to 3
``IENB``         5     Interrupt enables
``IARM[0:3]``    1     Interrupts armed, drives 0 to 3
``STIME``        24    Seek latency, per cylinder
``RTIME``        24    Rotational latency, per sector
``WTIME``        24    Inter-word latency
===============  ====  ==================================

Error handling is as follows:

==============  ===========================
Error           Processed as
==============  ===========================
Not attached    Disk not ready
End-of-file     Assume rest of disk is zero
OS I/O error    Report error and stop
==============  ===========================

.. _Mass Storage Module/Intelligent Disk Controller:
.. |DM| replace:: ``DM``
.. _DM:

Mass Storage Module/Intelligent Disk Controller (|DM|)
------------------------------------------------------
MSM/IDC disk controller options include the ability to make units write-enabled or write-locked,
and to select the type of drive::

    SET DMn LOCKED          Set unit n write locked
    SET DMn WRITEENABLED    Set unit n write enabled
    SET DMn MSM80           Set unit n to storage module, 80MB
                            (67MB formatted)
    SET DMn MSM300          Set unit n to storage module, 300MB
                            (262MB formatted)
    SET DMn MCCD16          Set unit n to medium capacity, 16MB
                            (13.5MB formatted)
    SET DMn MCCD48          Set unit n to medium capacity, 48MB
                            (40.5MB formatted)
    SET DMn MCCD80          Set unit n to medium capacity, 80MB
                            (67MB formatted)
    SET DMn MSM330F         Set unit n to storage module, 330MB
                            (300MB formatted)

Units can also be set ``ENABLED`` or ``DISABLED``.

The MSM/IDC controller supports the ``BOOT`` command.
To boot OS16/32,
the hex form of the operating system file's extension must be placed in locations ``7E:7F``.
The disk bootstrap looks for a valid OS16/32 volume descriptor in block 0,
and uses that to locate the volume directory.
It then searches the directory for a filename of the form ``OS16xxxx.hhh`` or ``OS32xxxx.hhh``,
where the ``xxxx`` is ignored and ``hhh`` is the ASCII form of the extension from locations ``7E:7F``.
The 32b bootstrap can also boot Wollongong UNIX;
locations ``7E:7F`` must be 0.
Note that only the MSM80 and MSM300 drives can be bootstrapped;
the boot code does not recognize the other drives.

All drives have 256 8b bytes per sector.
The other disk parameters are:

=======  =========  ========  =======
Drive    Cylinders  Surfaces  Sectors
=======  =========  ========  =======
MSM80    823        5         64
MSM300   823        19        64
MCCD16   823        1         64
MCCD48   823        3         64
MCCD80   823        5         64
MSM300F  1024       16        64
=======  =========  ========  =======

The MSM/IDC disk controller implements these registers:

===============  ======  ============================================================
Name             Size    Comments
===============  ======  ============================================================
``STA``          8       Controller status
``BUFC``         8       Controller buffer
``SEC``          8       Current sector select
``BUFD``         8       Drive buffer
``DBUF[0:767]``  8       Transfer buffer
``DBPTR``        16      Transfer buffer point
``DBLNT``        16      Transfer buffer length
``FIRST``        1       First DMA service flag
``CWDPTR``       2       Controller write data byte pointer
``DWDPTR``       1       Drive write data byte pointer
``IREQ``         5       Interrupt requests; right-to-left, controller, drives 0 to 3
``IENB``         5       Interrupt enables
``SIREQ``        5       Saved interrupt requests
``ICARM``        1       Controller interrupt armed
``IDARM[0:3]``   1       Drive interrupts armed, drives 0 to 3
``STIME``        24      Seek latency, per cylinder
``RTIME``        24      Rotational latency, per sector
``WTIME``        24      Inter-word latency
===============  ======  ============================================================

Error handling is as follows:

==============  ===========================
Error           Processed as
==============  ===========================
Not attached    Disk not ready
End-of-file     Assume rest of disk is zero
OS I/O error    Report error and stop
==============  ===========================

.. _Magnetic Tape Controller:
.. |MT| replace:: ``MT``
.. _MT:

Magnetic Tape Controller (|MT|)
-------------------------------
Magnetic tape options include the ability to make units write-enabled or write-locked. ::

    SET MTn LOCKED          Set unit n write locked
    SET MTn WRITEENABLED    Set unit n write enabled

Magnetic tape units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET MTn CAPAC=m         Set unit n capacity to m MB (0 = unlimited)
    SHOW MTn CAPAC          Show unit n capacity in MB

Units can also be set ``ENABLED`` or ``DISABLED``.

The magnetic tape supports the ``BOOT`` command.
``BOOT MTn`` copies an autoload sequence into memory and starts it running.

The magnetic tape controller implements these registers:

=================  ======  ================================================
Name               Size    Comments
=================  ======  ================================================
``CMD``            8       Command
``STA``            8       Status
``BUF``            8       Buffer
``DBUF[0:65535]``  8       Transfer buffer
``DBPTR``          16      Transfer buffer pointer
``DBLNT``          16      Transfer buffer length
``XFR``            1       Transfer in progress flag
``FIRST``          1       First DMA service flag
``IREQ``           4       Interrupt requests; right-to-left, drives 0 to 3
``IENB``           4       Interrupt enables
``IARM[0:3]``      1       Interrupts armed, drives 0 to 3
``STOP_IOE``       1       Stop on I/O error
``WTIME``          1       Word transfer time
``RTIME``          1       Interrecord latency
``UST[0:3]``       8       Unit status, drives 0 to 3
``POS[0:3]``       32      Tape position, drives 0 to 3
=================  ======  ================================================

Error handling is as follows:

==============  =====================================
Error           Processed as
==============  =====================================
Not attached    Tape not ready; if ``STOP_IOE``, stop
End-of-file     Set error flag
OS I/O error    Set error flag; if ``STOP_IOE``, stop
==============  =====================================

Symbolic display and input
==========================
The Interdata simulator implements symbolic display and input.
Display is controlled by command-line switches:

.. table::
   :class: switches

   ======  ===============================================
   ``-a``  Display byte as ASCII character
   ``-c``  Display halfword as two packed ASCII characters
   ``-m``  Display instruction mnemonics
   ======  ===============================================

Input parsing is controlled by the first character typed in or by command-line switches:

.. table::
   :class: switches

   ===============  ===========================
   ``'`` or ``-a``  ASCII character
   ``"`` or ``-c``  Two packed ASCII characters
   Alphabetic       Instruction mnemonic
   Numeric          Hexadecimal number
   ===============  ===========================

16b Instruction Input
---------------------
Instruction input uses standard Interdata assembler syntax.
There are seven instruction classes:
short branch,
extended short branch,
short immediate,
register,
register-register,
memory,
and register-memory.

Short branch instructions have the format ::

    sbop mask,address

where the ``mask`` is a hex (decimal) number between 0 and F (15),
and the address is within +32 (forward branch) or -32 (backward branch) of the current location.

Extended short branch instructions have the format ::

    sbxop address

where the ``address`` is within +32 or -32 of the current location.
For extended short branches,
the simulator chooses the forward or backward direction automatically.

Short immediate instructions have the format ::

    siop regnum,immed

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15),
and the immediate is a hex digit between 0 and F.

Register instructions have the format ::

    rop regnum

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15).

Register-register instructions have the format ::

    rrop regnum,regnum

where the register numbers are hex (decimal) numbers,
optionally preceded by ``R``,
between 0 and F (15).

Memory instructions have the format ::

    mop address{(index)}

where ``address`` is a hex number between 0 and 0xFFFF,
and the ``index`` register is a hex (decimal) number,
optionally preceded by ``R``,
between 1 and F (15).

Register-memory instructions have the format ::

    rmop regnum,address{(index)}

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15),
the ``address`` is a hex number between 0 and 0xFFFF,
and the ``index`` register is a hex (decimal) number,
optionally preceded by ``R``,
between 1 and F (15).

32b Instruction Input
---------------------
Instruction input uses standard Interdata assembler syntax.
There are nine instruction classes:
short branch,
extended short branch,
short immediate,
16b immediate,
32b immediate,
register,
register-register,
memory,
and register-memory.
Addresses,
where required,
can be specified as either absolute numbers or relative to the current location
(``.+n`` or ``.-n``).

Short branch instructions have the format ::

    sbop mask,address

where the ``mask`` is a hex (decimal) number between 0 and F (15),
and the address is within +32 (forward branch) or -32 (backward branch) of the current location.

Extended short branch instructions have the format ::

    sbxop address

where the address is within +32 or -32 of the current location.
For extended short branches,
the simulator chooses the forward or backward direction automatically.

Short immediate instructions have the format ::

    siop regnum,immed

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15),
and the immediate is a hex digit between 0 and F.

16b immediate instructions have the format ::

    i16op regnum,immed16{(index)}

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15),
the immediate is a hex number between 0 and 0xFFFF,
and the index register is a hex (decimal) number,
optionally preceded by ``R``,
between 1 and F (15).

32b immediate instructions have the format ::

    i32op regnum,immed32{(index)}

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15),
the immediate is a hex number between 0 and 0xFFFFFFFF,
and the index register is a hex (decimal) number,
optionally preceded by ``R``,
between 1 and F (15).

Register instructions have the format ::

    rop regnum

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15).

Register-register instructions have the format ::

    rrop regnum,regnum

where the register numbers are hex (decimal) numbers,
optionally preceded by ``R``,
between 0 and F (15).

Memory instructions have the format ::

    mop address{(index)} or
    mop address{(index1,index2)}

where ``address`` is a hex number between 0 and 0xFFFF,
and the index registers are hex (decimal) numbers,
optionally preceded by ``R``,
between 1 and F (15).

Register-memory instructions have the format ::

    rmop regnum,address{(index)} or
    rmop regnum,address{(index1,index2)}

where the register number is a hex (decimal) number,
optionally preceded by ``R``,
between 0 and F (15),
the address is a hex number between 0 and 0xFFFF,
and the index registers are hex (decimal) numbers,
optionally preceded by ``R``,
between 1 and F (15).

For memory operands,
the simulator automatically chooses the format
(``RX1``, ``RX2``, ``RX3``) that consumes the fewest bytes.
If both ``RX1`` and ``RX2`` are feasible,
the simulator chooses ``RX1``.
