.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. |API| replace:: :html:`<abbr title="Automatic Priority Interrupt">API</abbr>`
.. |EAE| replace:: :html:`<abbr title="Extended Arithmetic Element">EAE</abbr>`

****************************
PDP-4/7/9/15 simulator usage
****************************
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

This memorandum documents the DEC PDP-4, PDP-7, PDP-9, and PDP-15 simulators.

Simulator files
===============
To compile a particular model in the 18b family,
you must include the appropriate switch in the compilation command-line::

    PDP-4/      PDP4
    PDP-7/      PDP7
    PDP-9/      PDP9
    PDP-15/     PDP15

If no model is specified,
the default is the PDP-15.

+-----------------+---------------------+-------+-------+-------+--------+
| Directory       | File                | PDP-4 | PDP-7 | PDP-9 | PDP-15 |
+=================+=====================+=======+=======+=======+========+
| ``sim/``        | ``sim_defs.h``      | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``sim_rev.h``       | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``sim_sock.h``      | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``sim_tape.h``      |       |       | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``sim_tmxr.h``      | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``scp.c``           | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``scp_tty.c``       | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``sim_sock.c``      | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``sim_tape.c``      |       |       | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``sim_tmxr.c``      | X     | X     | X     | X      |
+-----------------+---------------------+-------+-------+-------+--------+
| ``sim/pdp18b/`` | ``pdp18b_defs.h``   | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_cpu.c``    | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_drm.c``    | X     | X     | X     |        |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_dt.c``     | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_fpp.c``    |       |       |       | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_lp.c``     | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_mt.c``     |       |       | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_rb.c``     |       | X     | X     |        |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_rf.c``     |       |       | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_rp.c``     |       |       |       | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_stddev.c`` | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_sys.c``    | X     | X     | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_tt1.c``    |       |       | X     | X      |
|                 +---------------------+-------+-------+-------+--------+
|                 | ``pdp18b_dr15.c``   |       |       |       | X      |
+-----------------+---------------------+-------+-------+-------+--------+

18b PDP features
================
The four 18b PDPs (PDP-4, PDP-7, PDP-9, PDP-15) are very similar and are configured as follows:

+--------+---------------------+----------------------------------------------+
| System | Device name(s)      | Simulates                                    |
+========+=====================+==============================================+
| PDP-4  | |CPU|_              | PDP-4 CPU with 8KW of memory                 |
|        +---------------------+----------------------------------------------+
|        | —                   | Type 18 extended arithmetic element (|EAE|)  |
|        +---------------------+----------------------------------------------+
|        | |PTR|_\ ,\ |PTP|_   | Integral paper tape/Type 75 punch            |
|        +---------------------+----------------------------------------------+
|        | |TTI|_\ ,\ |TTO|_   | KSR28 console terminal (Baudot code)         |
|        +---------------------+----------------------------------------------+
|        | |LPT|_              | Type 62 line printer (Hollerith code)        |
|        +---------------------+----------------------------------------------+
|        | |CLK|_              | Integral real-time clock                     |
|        +---------------------+----------------------------------------------+
|        | |DT|_               | Type 550/555 DECtape                         |
|        +---------------------+----------------------------------------------+
|        | |DRM|_              | Type 24 serial drum                          |
+--------+---------------------+----------------------------------------------+
| PDP-7  | |CPU|_              | PDP-7 CPU with 32KW of memory                |
|        +---------------------+----------------------------------------------+
|        | —                   | Type 177 extended arithmetic element (|EAE|) |
|        +---------------------+----------------------------------------------+
|        | —                   | Type 148 memory extension                    |
|        +---------------------+----------------------------------------------+
|        | |PTR|_\ ,\ |PTP|_   | Type 444 paper tape reader/Type 75 punch     |
|        +---------------------+----------------------------------------------+
|        | |TTI|_\ ,\ |TTO|_   | KSR 33 console terminal                      |
|        +---------------------+----------------------------------------------+
|        | |LPT|_              | Type 647 line printer                        |
|        +---------------------+----------------------------------------------+
|        | |CLK|_              | Integral real-time clock                     |
|        +---------------------+----------------------------------------------+
|        | |DT|_               | Type 550/555 DECtape                         |
|        +---------------------+----------------------------------------------+
|        | |DRM|_              | Type 24 serial drum                          |
|        +---------------------+----------------------------------------------+
|        | |RB|_               | |RB09| fixed head disk                       |
+--------+---------------------+----------------------------------------------+
| PDP-9  | |CPU|_              | PDP-9 CPU with 32KW of memory                |
|        +---------------------+----------------------------------------------+
|        | —                   | KE09A extended arithmetic element (|EAE|)    |
|        +---------------------+----------------------------------------------+
|        | —                   | KF09A automatic priority interrupt (|API|)   |
|        +---------------------+----------------------------------------------+
|        | —                   | KG09B memory extension                       |
|        +---------------------+----------------------------------------------+
|        | —                   | KP09A power detection                        |
|        +---------------------+----------------------------------------------+
|        | —                   | KX09A memory protection                      |
|        +---------------------+----------------------------------------------+
|        | |PTR|_\ ,\ |PTP|_   | PC09A paper tape reader/punch                |
|        +---------------------+----------------------------------------------+
|        | |TTI|_\ ,\ |TTO|_   | KSR 33 console terminal                      |
|        +---------------------+----------------------------------------------+
|        | |TTIX|_\ ,\ |TTOX|_ | 1-4 LT09A additional terminals               |
|        +---------------------+----------------------------------------------+
|        | |LP9|_              | LP09 line printer                            |
|        +---------------------+----------------------------------------------+
|        | |LPT|_              | Type 647E line printer                       |
|        +---------------------+----------------------------------------------+
|        | |CLK|_              | Integral real-time clock                     |
|        +---------------------+----------------------------------------------+
|        | |DRM|_              | |RM09| serial drum                           |
|        +---------------------+----------------------------------------------+
|        | |RB|_               | |RB09| fixed-head disk                       |
|        +---------------------+----------------------------------------------+
|        | |RF|_               | |RF09|\ /\ |RS09| fixed-head disk            |
|        +---------------------+----------------------------------------------+
|        | |DT|_               | |TC02|\ /\ |TU55| DECtape                    |
|        +---------------------+----------------------------------------------+
|        | |MT|_               | |TC59|\ /\ |TU10| magnetic tape              |
+--------+---------------------+----------------------------------------------+
| PDP-15 | |CPU|_              | PDP-15 CPU with 32KW of memory               |
|        +---------------------+----------------------------------------------+
|        | —                   | KE15 extended arithmetic element (|EAE|)     |
|        +---------------------+----------------------------------------------+
|        | —                   | KA15 automatic priority interrupt (|API|)    |
|        +---------------------+----------------------------------------------+
|        | —                   | KF15 power detection                         |
|        +---------------------+----------------------------------------------+
|        | —                   | KM15 memory protection                       |
|        +---------------------+----------------------------------------------+
|        | —                   | KT15 memory relocation and protection        |
|        +---------------------+----------------------------------------------+
|        | —                   | XVM memory relocation and protection         |
|        +---------------------+----------------------------------------------+
|        | |FPP|_              | FP15 floating point processor                |
|        +---------------------+----------------------------------------------+
|        | |PTR|_\ ,\ |PTP|_   | PC15 paper tape reader/punch                 |
|        +---------------------+----------------------------------------------+
|        | |TTI|_\ ,\ |TTO|_   | KSR 35 console terminal                      |
|        +---------------------+----------------------------------------------+
|        | |TTIX|_\ ,\ |TTOX|_ | 1-16 LT15/LT19 additional terminals          |
|        +---------------------+----------------------------------------------+
|        | |LP9|_              | LP09 line printer                            |
|        +---------------------+----------------------------------------------+
|        | |LPT|_              | LP15 line printer                            |
|        +---------------------+----------------------------------------------+
|        | |CLK|_              | Integral real-time clock                     |
|        +---------------------+----------------------------------------------+
|        | |RP|_               | |RP15|\ /\ |RP02|\ /\ |RP03| disk pack       |
|        +---------------------+----------------------------------------------+
|        | |RF|_               | |RF15|\ /\ |RS09| fixed-head disk            |
|        +---------------------+----------------------------------------------+
|        | |DT|_               | |TC15|\ /\ |TU56| DECtape                    |
|        +---------------------+----------------------------------------------+
|        | |MT|_               | |TC59|\ /\ |TU10| magnetic tape              |
|        +---------------------+----------------------------------------------+
|        | |DR|_               | |DR15C| parallel buffer (for UC15)           |
+--------+---------------------+----------------------------------------------+

Most devices can be disabled or enabled by the commands::

    SET <dev> DISABLED
    SET <dev> ENABLED

The simulator allows most device numbers to be changed,
using the command::

    SET <dev> DEV=<number>

However, devices can only be booted with their default device numbers.

The 18b PDP simulators implement several unique stop conditions:

* An unimplemented instruction is decoded,
  and register ``STOP_INST`` is set
* More than ``XCT_MAX`` nested executes are detected during instruction execution
* An FP15 instruction is decoded,
  the FP15 is disabled,
  and register ``STOP_FPP`` is set
* A simulated DECtape runs off the end of its reel,
  and register ``STOP_OFFR`` is set

The ``LOAD`` command supports three different file formats:

* PDP-7/9/15 hardware read-in RIM format files
  (data only loaded into sequential addresses)
* PDP-4/7 "second stage" RIM format files
  (alternating DAC address instructions and data)
* PDP-9/15 binary loader format files

The load file format can be specified by switches:

.. table::
   :class: switches

   ======  ===========================
   ``-R``  Hardware read-in RIM format
   ``-S``  Second stage RIM format
   ``-B``  Binary loader format
   ======  ===========================

If no switch is specified,
the load file format is determined from the file extension.
Files ending in ``.RIM`` are assumed to be RIM format
(hardware versus second stage is determined from the data);
files ending in any other extension are assumed to be binary loader format.
Examples::

    LOAD -R file address     Load PDP-9/PDP-15 RIM format file starting at address
    LOAD -S file             Load PDP-4/PDP-7 RIM format file
    LOAD file.RIM address    Assume file is RIM, determine type from data
    LOAD -B file             Load PDP-9/PDP-15 BIN format file
    LOAD file.BIN            Assume file is PDP-9/PDP-15 BIN format

If no address is given for a RIM format load,
a starting address of 200 (octal) is assumed.

The ``DUMP`` command is not supported.

.. |CPU| replace:: ``CPU``

|CPU|
-----
The |CPU| options are the presence of the |EAE|,
the presence of the |API| and memory protection
(for the PDP-9 and PDP-15),
the presence of relocation or XVM
(PDP-15 only),
and the size of main memory.

======  ==================  ==========================================
System  Option              Comment
======  ==================  ==========================================
All     ``SET CPU EAE``     Enable |EAE|
All     ``SET CPU NOEAE``   Disable |EAE|
9,15    ``SET CPU API``     Enable |API|
9,15    ``SET CPU NOAPI``   Disable |API|
9,15    ``SET CPU PROT``    Enable memory protection
15      ``SET CPU RELOC``   Enable memory relocation
15      ``SET CPU XVM``     Enable XVM relocation
9,15    ``SET CPU NOPROT``  Disable protection, relocation, XVM
4       ``SET CPU 4K``      Set memory size = 4K
All     ``SET CPU 8K``      Set memory size = 8K
All     ``SET CPU 12K``     Set memory size = 12K
All     ``SET CPU 16K``     Set memory size = 16K
All     ``SET CPU 20K``     Set memory size = 20K
All     ``SET CPU 24K``     Set memory size = 24K
All     ``SET CPU 28K``     Set memory size = 28K
All     ``SET CPU 32K``     Set memory size = 32K
15      ``SET CPU 48K``     Set memory size = 48K
15      ``SET CPU 64K``     Set memory size = 64K
15      ``SET CPU 80K``     Set memory size = 80K
15      ``SET CPU 96K``     Set memory size = 96K
15      ``SET CPU 112K``    Set memory size = 112K
15      ``SET CPU 128K``    Set memory size = 128K
======  ==================  ==========================================

Memory sizes greater than 8K are only available on the PDP-7, PDP-9, and PDP-15;
memory sizes greater than 32KW are only available on the PDP-15.
If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 8K for the PDP-4,
32K for the PDP-7 and PDP-9,
and 128K for the PDP-15.

The ``PROT`` option corresponds to the KX09A on the PDP-9 and the KM15 for the PDP-15.
The ``PROT`` option is required to run the Foreground/Background Monitor.
The ``RELOC`` option corresponds to the KT15 on the PDP-15,
and the XVM option corresponds to the XM15 on the PDP-15.
ADSS-15,
ADSS-15 Foreground/Background,
and standard DOS-15 will *not* run if these options are enabled.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

======  ===============  =========  ===========================================================
System  Name             Size       Comments
======  ===============  =========  ===========================================================
All     ``PC``           ``addr``   Program counter
All     ``AC``           18         Accumulator
All     ``L``            1          Link
All     ``MQ``           18         Multiplier-quotient
All     ``SC``           6          Shift counter
All     ``EAE_AC_SIGN``  1          |EAE| AC sign
All     ``SR``           18         Front panel switches
All     ``ASW``          ``addr``   Address switches for RIM load
All     ``INT[0:4]``     32         Interrupt requests, 0:3 = |API| levels 0 to 3, 4 = PI level
All     ``IORS``         18         IORS register
All     ``ION``          1          Interrupt enable
All     ``ION_DELAY``    2          Interrupt enable delay
15      ``ION_INH``      1          Interrupt inhibit
9,15    ``APIENB``       1          API enable
9,15    ``APIREQ``       8          API requesting levels
9,15    ``APIACT``       8          API active levels
9,15    ``BR``           18         Memory protection bounds
15      ``XR``           18         Index register
15      ``LR``           18         Limit register
9,15    ``BR``           18         Memory protection bounds
15      ``RR``           18         Memory protection relocation
15      ``MMR``          18         Memory protection control
9,15    ``USMD``         1          User mode
9,15    ``USMDBUF``      1          User mode buffer
9,15    ``USMDDEF``      1          User mode load defer
9,15    ``NEXM``         1          Non-existent memory violation
9,15    ``PRVN``         1          Privilege violation
7,9     ``EXTM``         1          Extend mode
7,9     ``EXTM_INIT``    1          Extend mode value after reset
15      ``BANKM``        1          Bank mode
15      ``BANKM_INIT``   1          Bank mode value after reset
7       ``TRAPM``        1          Trap mode
7,9,15  ``TRAPP``        1          Trap pending
7,9     ``EMIRP``        1          EMIR instruction pending
9,15    ``RESTP``        1          DBR or RES instruction pending
9,15    ``PWRFL``        1          Power fail flag
All     ``PCQ[0:63]``    ``addr``   PC prior to last ``JMP``, ``JMS``, ``CAL``, or interrupt;
                                    most recent PC change first
All     ``STOP_INST``    1          Stop on undefined instruction
All     ``XCT_MAX``      8          Max number of chained XCTs allowed
All     ``WRU``          8          Interrupt character
======  ===============  =========  ===========================================================

``addr`` signifies the address width of the system
(13b for the PDP-4,
15b for the PDP-7 and PDP-9,
17b for the PDP-15).

The CPU attempts to detect when the simulator is idle.
When idle,
the simulator does not use any resources on the host system.
Idle detection is controlled by the ``SET IDLE`` and ``SET NOIDLE`` commands::

    SET CPU IDLE                  Enable idle detection
    SET CPU NOIDLE                Disable idle detection

Idle detection is disabled by default.
At present,
the CPU is considered idle if it is executing a ``KSF/JMP *-1`` loop with interrupts disabled (DECSYS) or a ``JMP * loop`` (XVM/RSX).
There is no idle loop detector for ADSS, F/B, or DOS.

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the ``SET CPU HISTORY`` and ``SHOW CPU HISTORY`` commands::

    SET CPU HISTORY               Clear history buffer
    SET CPU HISTORY=0             Disable history
    SET CPU HISTORY=n             Enable history, length = n
    SHOW CPU HISTORY              Print CPU history
    SHOW CPU HISTORY=n            Print first n entries of CPU history

The maximum length for the history is 65536 entries.

.. _Floating point processor:
.. |FPP| replace:: ``FPP``
.. _FPP:

Floating point processor (|FPP|)
--------------------------------
The PDP-15 features an optional floating point processor,
the FP15 (|FPP|).
The |FPP| can be enabled and disabled;
by default it is disabled.

The |FPP| implements these registers:

============  ====  =======================================================
Name          Size  Comments
============  ====  =======================================================
``FIR``       12    Floating instruction register
``EPA``       18    EPA (A exponent)
``FMAS``      1     FMA sign
``FMAH``      17    FMA<1:17>
``FMAL``      18    FMA<18:35>
``EPB``       18    EPB (B exponent)
``FMBS``      1     FMB sign
``FMBH``      17    FMB<1:17>
``FMBL``      18    FMB<18:35>
``FGUARD``    1     Guard bit
``FMQH``      17    FMQ<1:17>
``FMQL``      18    FMQ<18:35>
``JEA``       18    Exception address register
``STOP_FPP``  1     Stop if FP15 instruction decoded while FP15 is disabled
============  ====  =======================================================

Programmed I/O devices
----------------------

.. _Paper tape reader:
.. |PTR| replace:: ``PTR``
.. _PTR:

Paper tape reader (|PTR|)
"""""""""""""""""""""""""
The paper tape reader (|PTR|) reads data from a disk file.
The ``POS`` register specifies the number of the next data item to be read.
Thus, by changing ``POS``,
the user can backspace or advance the reader.

The paper tape reader supports the ``BOOT`` command.
The specific forms recognized vary from system to system:

======  ==================  ==================================================================
System  Command             Comments
======  ==================  ==================================================================
4,7     ``BOOT PTR``        Load RIM loader and start it running
4,7     ``BOOT -F PTR``     Load funny format loader and start it running
7       ``BOOT -H PTR``     Start hardware RIM load at address given by address switches (ASW)
9,15    ``BOOT {-H} PTR``   Start hardware RIM load at address given by address switches (ASW)
======  ==================  ==================================================================

The PDP-4 does not have a hardware read-in mode load capability.

The ``ATTACH PTR`` command recognizes two switches,
``-A`` for ASCII mode and ``–K`` for KSR mode.
In ASCII mode,
data returned by the read alphabetic command has even parity.
This allows normal text files to be used as input to the paper tape reader on the PDP-9 and PDP-15.
In KSR mode,
data returned by the read alphabetic command has forced ones parity.
This allows normal text files to be used as input to the paper tape reader on the PDP-7.

The paper tape reader implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       8     Last data item processed
``INT``       1     Interrupt pending flag
``DONE``      1     Device done flag
``ERR``       1     Error flag (PDP-9, PDP-15 only)
``POS``       32    Position in the input file
``TIME``      24    Time from I/O initiation to interrupt
``STOP_IOE``  1     Stop on I/O error
============  ====  =====================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached |  1           | Report error and stop |
|              +--------------+-----------------------+
|              |  0           | Out of tape           |
+--------------+--------------+-----------------------+
| End-of-file  |  1           | Report error and stop |
|              +--------------+-----------------------+
|              |  0           | Out of tape           |
+--------------+--------------+-----------------------+
| OS I/O error |  x           | Report error and stop |
+--------------+--------------+-----------------------+

.. _Paper tape punch:
.. |PTP| replace:: ``PTP``
.. _PTP:

Paper tape punch (|PTP|)
""""""""""""""""""""""""
The paper tape punch (|PTP|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the punch.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

The ``ATTACH PTP`` command recognizes one switch,
``-A`` for ASCII mode.
In ASCII mode,
data is punched with the high order bit clear,
and NULL and DEL characters are suppressed.
This allows punch output to be processed with normal text-editing utilities.

The paper tape punch implements these registers:

============  ====  =====================================
Name          Size  Comments
============  ====  =====================================
``BUF``       8     Last data item processed
``INT``       1     Interrupt pending flag
``DONE``      1     Device done flag
``ERR``       1     Error flag (PDP-9, PDP-15 only)
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

.. _Terminal input:
.. |TTI| replace:: ``TTI``
.. _TTI:

Terminal input (|TTI|)
""""""""""""""""""""""
On the PDP-7, PDP-9, and PDP-15,
the terminal interfaces (|TTI|, |TTO|_) can be set to one of four modes,
``KSR``, ``7P``, ``7B``, or ``8B``.
On the PDP-7 and PDP-9,
"Unix v0" mode is also available:

=========  =======================  ========================================
Mode       Input characters         Output characters
=========  =======================  ========================================
``KSR``    Lowercase converted      Lowercase converted to uppercase,
           to uppercase,            high-order bit cleared,
           high-order bit set       non-printing characters suppressed
``7P``     High-order bit cleared   High-order bit cleared,
                                    non-printing characters suppressed
``7B``     High-order bit cleared   High-order bit cleared
``8B``     No changes               No changes
``UNIX``   High-order bit set,      No changes
           CR, LF interchanged,
           ESC mapped to ALTMODE
=========  =======================  ========================================

The default mode is ``KSR``.

The console terminal operates,
by default,
with local echo.
The terminal input can be set to ``FDX`` (full duplex),
which suppresses local echo.

The terminal input (|TTI|) polls the console keyboard for input.
It implements these registers:

==========  ====  ============================================
Name        Size  Comments
==========  ====  ============================================
``BUF``     8     Last data item processed
``BUF2ND``  5     (PDP-4 only) saved character
``INT``     1     Interrupt pending flag
``DONE``    1     Device done flag
``POS``     32    Number of characters input
``TIME``    24    Input polling interval (if 0, the keyboard
                  is polled synchronously with the line clock)
==========  ====  ============================================

.. _Terminal output:
.. |TTO| replace:: ``TTO``
.. _TTO:

Terminal output (|TTO|)
"""""""""""""""""""""""
The terminal output (|TTO|) writes to the simulator console window.
It implements these registers:

=========  ====  =====================================
Name       Size  Comments
=========  ====  =====================================
``BUF``    8     Last data item processed
``SHIFT``  5     (PDP-4 only) letters/figures flag
``INT``    1     Interrupt pending flag
``DONE``   1     Device done flag
``POS``    32    Number of characters output
``TIME``   24    Time from I/O initiation to interrupt
=========  ====  =====================================

.. _Line printers:
.. |LPT| replace:: ``LPT``
.. |LP9| replace:: ``LP9``
.. _LPT:
.. _LP9:

Line printers (|LPT|, |LP9|)
""""""""""""""""""""""""""""
The line printers (|LPT|, |LP9|) write data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the printer.
The default position after ``ATTACH`` is to position at the end of an existing file.
A new file can be created if you attach with the ``-N`` switch.

|LPT| is the "default" line printer for a CPU:
Type 62 for the PDP-4,
Type 647 for the PDP-7 and PDP-9,
and LP15 for the PDP-15.
|LP9| is the LP09 line printer controller for the PDP-9.
It may be needed on the PDP-15 to run certain software packages.
|LP9| is disabled by default.

The LP15 is a 3-cycle data break device.
The current address register is in memory.
It can be examined and modified with ``SET`` and ``SHOW`` commands::

    SHOW LPT CA                   Display current
    SET LPT CA=value              Set current address to value

The Type 62 printer controller implements these registers:

===============  ====  =====================================
Name             Size  Comments
===============  ====  =====================================
``BUF``          8     Last data item processed
``INT``          1     Interrupt pending flag
``DONE``         1     Device done flag
``SPC``          1     Spacing done flag
``BPTR``         6     Print buffer pointer
``POS``          32    Position in the output file
``TIME``         24    Time from I/O initiation to interrupt
``STOP_IOE``     1     Stop on I/O error
``LBUF[0:119]``  8     Line buffer
===============  ====  =====================================

The Type 647 printer controller implements these registers:

===============  ====  =====================================
Name             Size  Comments
===============  ====  =====================================
``BUF``          8     Last data item processed
``INT``          1     Interrupt pending flag
``DONE``         1     Device done flag
``ENABLE``       1     Interrupt enable (PDP-9 only)
``ERR``          1     Error flag
``BPTR``         7     Print buffer pointer
``POS``          32    Position in the output file
``TIME``         24    Time from I/O initiation to interrupt
``STOP_IOE``     1     Stop on I/O error
``LBUF[0:119]``  8     Line buffer
===============  ====  =====================================

The LP09 printer controller implements these registers:

==========  ====  =====================================
Name        Size  Comments
==========  ====  =====================================
``BUF``     7     Output character
``INT``     1     Interrupt pending flag
``DONE``    1     Device done flag
``ENABLE``  1     Interrupt enable
``ERR``     1     Error flag
``POS``     32    Position in the output file
``TIME``    24    Time from I/O initiation to interrupt
==========  ====  =====================================

The LP15 printer controller implements these registers:

===============  ====  =====================================
Name             Size  Comments
===============  ====  =====================================
``STA``          18    Status register
``MA``           18    DMA memory address
``INT``          1     Interrupt pending flag
``ENABLE``       1     Interrupt enable
``LCNT``         8     Line counter
``BPTR``         7     Print buffer pointer
``POS``          32    Position in the output file
``TIME``         24    Time from I/O initiation to interrupt
``STOP_IOE``     1     Stop on I/O error
``LBUF[0:131]``  8     Line buffer
===============  ====  =====================================

For all printers,
error handling is as follows:

+--------------+--------------+------------------------+
| Error        | ``STOP_IOE`` | Processed as           |
+==============+==============+========================+
| Not attached | 1            | Report error and stop  |
|              +--------------+------------------------+
|              | 0            | Out of tape or paper   |
+--------------+--------------+------------------------+
| OS I/O error | x            | Report error and stop  |
+--------------+--------------+------------------------+

.. _Real-time clock:
.. |CLK| replace:: ``CLK``
.. _CLK:

Real-time clock (|CLK|)
"""""""""""""""""""""""
The real-time clock (|CLK|) frequency can be adjusted as follows::

    SET CLK 60HZ                  Set frequency to 60Hz
    SET CLK 50HZ                  Set frequency to 50Hz

The default is ``60Hz``.

The clock implements these registers:

==========  ====  ======================
Name        Size  Comments
==========  ====  ======================
``INT``     1     Interrupt pending flag
``DONE``    1     Device done flag
``ENABLE``  1     Clock enable
``TIME``    24    Clock frequency
==========  ====  ======================

The real-time clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

.. _Additional terminals:
.. |TTIX| replace:: ``TTIX``
.. |TTOX| replace:: ``TTOX``
.. _TTIX:
.. _TTOX:

Additional terminals (|TTIX|, |TTOX|)
"""""""""""""""""""""""""""""""""""""
The additional terminals consist of two independent devices,
|TTIX| and |TTOX|.
The entire set is modeled as a terminal multiplexer,
with |TTIX| as the master unit.
The additional terminals perform input and output through Telnet sessions connected to a user-specified port.
The ``ATTACH`` command specifies the port to be used::

    ATTACH TTIX <port>            Set up listening port

where ``port`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

The PDP-9 supports 1–4 additional terminals.
The PDP-15 supports 1–16 additional terminals.
The number of additional terminals can be changed with the command::

    SET TTIX LINES=n              Set number of lines to n

The default is one additional terminal.

The additional terminals can be set to one of four modes,
``KSR``, ``7P``, ``7B``, or ``8B``:

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
Finally, each line supports output logging.
The ``SET TTOXn LOG`` command enables logging on a line::

    SET TTOXn LOG=filename        Log output of line n to filename

The ``SET TTOXn NOLOG`` command disables logging and closes the open log file, if any.

Once |TTIX| is attached and the simulator is running,
the terminals listen for connections on the specified port.
They assume that the incoming connections are Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a ``SET TTOXn DISCONNECT`` command,
or a ``DETACH TTIX`` command.

Other special commands::

    SHOW TTIX CONNECTIONS         Show current connections
    SHOW TTIX STATISTICS          Show statistics for active connections
    SET TTOXn DISCONNECT          Disconnects the specified line

The input device (|TTIX|) implements these registers:

=================  ====  ========================================
Name               Size  Comments
=================  ====  ========================================
``BUF[0:3/0:15]``  8     Last character received, lines 0 to 3/15
``DONE``           16    Input ready flags, line 0 on right
``INT``            1     Interrupt pending flag
``TIME``           24    Keyboard polling interval
=================  ====  ========================================

The output device (|TTOX|) implements these registers:

==================  ====  ======================================================
Name                Size  Comments
==================  ====  ======================================================
``BUF[0:3/0:15]``   8     Last character transmitted, lines 0 to 3/15
``DONE``            16    Output ready flags, line 0 on right
``INT``             1     Interrupt pending flag
``TIME[0:3/0:15]``  24    Time from I/O initiation to interrupt, lines 0 to 3/15
==================  ====  ======================================================

.. _RP15/RP02/RP03 disk pack:
.. |RP|   replace:: ``RP``
.. |RP15| replace:: ``RP15``
.. |RP02| replace:: ``RP02``
.. |RP03| replace:: ``RP03``
.. _RP:
.. _RP15:
.. _RP02:
.. _RP03:

|RP15|\ /|RP02|\ /\ |RP03| disk pack (|RP|)
-------------------------------------------
|RP15| options include the ability to make units write-enabled or write-locked and to select the type of disk drive::

    SET RPn RP02                  Set unit n to be an RP02 (default)
    SET RPn RP03                  Set unit n to be an RP03
    SET RPn LOCKED                Set unit n write-locked
    SET RPn WRITEENABLED          Set unit n write-enabled

Units can also be set ``ENABLED`` or ``DISABLED``.

The |RP15| implements these registers:

============  ====  =======================
Name          Size  Comments
============  ====  =======================
``STA``       18    Status A
``STB``       18    Status B
``DA``        18    Disk address
``MA``        18    Current memory address
``WC``        18    Word count
``INT``       1     Interrupt pending flag
``BUSY``      1     Control busy flag
``STIME``     24    Seek time, per cylinder
``RTIME``     24    Rotational delay
``STOP_IOE``  1     Stop on I/O error
============  ====  =======================

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

.. _Type 24/RM09 serial drum:
.. |DRM|  replace:: ``DRM``
.. |RM09| replace:: ``RM09``
.. _DRM:
.. _RM09:

Type 24/\ |RM09| serial drum (|DRM|)
------------------------------------
The serial drum (|DRM|) implements these registers:

============  ====  ============================
Name          Size  Comments
============  ====  ============================
``DA``        9     Drum address (sector number)
``MA``        16    Current memory address
``INT``       1     Interrupt pending flag
``DONE``      1     Device done flag
``ERR``       1     Error flag
``WLK``       32    Write lock switches
``TIME``      24    Rotational latency, per word
``STOP_IOE``  1     Stop on I/O error
============  ====  ============================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Disk not ready        |
+--------------+--------------+-----------------------+

Drum data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

.. _RB09 fixed-head disk:
.. |RB|   replace:: ``RB``
.. |RB09| replace:: ``RB09``
.. _RB:
.. _RB09:

|RB09| fixed-head disk (|RB|)
-----------------------------
The |RB09| was an early fixed-head disk for the PDP-7 and PDP-9.
It was superceded by the |RF09|_\ /\ |RS09|_.
It is disabled by default.

The |RB09| implements these registers:

============  ====  =========================================================
Name          Size  Comments
============  ====  =========================================================
``STA``       18    Status
``DA``        20    Current disk address
``WC``        16    Word count
``MA``        15    Memory address
``INT``       1     Interrupt pending flag
``WLK``       20    Write lock switches for track groups, 10 tracks per group
``TIME``      24    Rotational delay, per word
``BURST``     1     Burst flag
``STOP_IOE``  1     Stop on I/O error
============  ====  =========================================================

The |RB09| is a data break device.
If ``BURST`` equals 0,
word transfers are scheduled individually;
if ``BURST`` equals 1,
the entire transfer occurs in a single data break.

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Disk not ready        |
+--------------+--------------+-----------------------+

|RB09| data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

.. _RF09/RF15/RS09 fixed-head disk:
.. |RF09/RF15| replace:: |RF09|\ /\ |RF15|
.. |F15/RF09|  replace:: |F15|\ /\ |RF09|
.. |RF|        replace:: ``RF``
.. |F15|       replace:: ``F15``
.. |RF09|      replace:: ``RF09``
.. |RF15|      replace:: ``RF15``
.. |RS09|      replace:: ``RS09``
.. _RF:
.. _F15:
.. _RF09:
.. _RF15:
.. _RS09:

|RF09|\ /\ |RF15|\ /\ |RS09| fixed-head disk (|RF|)
---------------------------------------------------
|RF09/RF15| options include the ability to set the number of platters to a fixed value between 1 and 8,
or to autosize the number of platters from the attached file::

    SET RF 1P                     One platter (256K)
    SET RF 2P                     Two platters (512K)
    SET RF 3P                     Three platters (768K)
    SET RF 4P                     Four platters (1024K)
    SET RF 5P                     Five platters (1280K)
    SET RF 6P                     Six platters (1536K)
    SET RF 7P                     Seven platters (1792K)
    SET RF 8P                     Eight platters (2048K)
    SET RF AUTOSIZE               Autosize on ATTACH

The default is ``AUTOSIZE``.

The |RF09/RF15| is a 3-cycle data break device.
The word count and current address registers are in memory.
They can be examined and modified with ``SET`` and ``SHOW`` commands::

    SHOW RF CA(WC)                Display current address (word count)
    SET RF CA(WC)=value           Set current address (word count) to value

The |RF09/RF15| implements these registers:

============  ====  ====================================
Name          Size  Comments
============  ====  ====================================
``STA``       18    Status
``DA``        21    Current disk address
``BUF``       18    Data buffer (diagnostic only)
``INT``       1     Interrupt pending flag
``WLK[0:7]``  16    Write lock switches for disks 0 to 7
``TIME``      24    Rotational delay, per word
``BURST``     1     Burst flag
``STOP_IOE``  1     Stop on I/O error
============  ====  ====================================

The |RF09/RF15| is a three-cycle data break device.
If ``BURST`` equals 0,
word transfers are scheduled individually;
if ``BURST`` equals 1,
the entire transfer occurs in a single data break.

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| Not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Disk not ready        |
+--------------+--------------+-----------------------+

|F15/RF09| data files are buffered in memory;
therefore,
end-of-file and OS I/O errors cannot occur.

.. _Type 550/555, TC02/TU55, and TC15/TU56 DECtape:
.. |DT|   replace:: ``DT``
.. |TC02| replace:: ``TC02``
.. |TU55| replace:: ``TU55``
.. |TC15| replace:: ``TC15``
.. |TU56| replace:: ``TU56``
.. _DT:
.. _TC02:
.. _TU55:
.. _TC15:
.. _TU56:

Type 550/555, |TC02|\ /\ |TU55|, and |TC15|\ /\ |TU56| DECtape (|DT|)
---------------------------------------------------------------------
The PDP-4 and PDP-7 use the Type 550 DECtape,
a programmed I/O controller.
The PDP-9 uses the |TC02|,
and the PDP-15 uses the |TC15|.
The |TC02| and |TC15| are DMA controllers and programmatically identical.
Except for the first five units of the Type 550,
PDP-4/7/9/15 DECtape format has 5 18b words in the block header and trailer,
like all other DECtapes.

In the 550/555,
DECtapes drives are numbered 1–8;
in the simulator,
drive 8 is unit 0.
In the ``TX02``\ /\ |TC15|,
DECtape drives are numbered 0–7.
DECtape options include the ability to make units write-enabled or write-locked. ::

    SET DTn WRITEENABLED          Set unit n write enabled
    SET DTn LOCKED                Set unit n write locked

Units can also be set ``ENABLED`` or ``DISABLED``.

The Type 550, |TC02|, and |TC15| support PDP-8 format, PDP-11 format, and 18b format DECtape images.
``ATTACH`` assumes the image is in 18b format;
the user can force other choices with switches:

.. table::
   :class: switches

   ======  =============================
   ``-t``  PDP-8 format
   ``-s``  PDP-11 format
   ``-a``  Autoselect based on file size
   ======  =============================

The DECtape controller is a data-only simulator;
the timing and mark track,
and block header and trailer,
are not stored.
Thus, the ``WRITE TIMING AND MARK TRACK`` function is not supported;
the ``READ ALL`` function always returns the hardware standard block header and trailer;
and the ``WRITE ALL`` function dumps non-data words into the bit bucket.

The |TC02| and |TC15| are 3-cycle databreak devices.
The word count and current address registers are in memory.
They can be examined and modified with ``SET`` and ``SHOW`` commands::

    SHOW DT CA(WC)                Display current address (word count)
    SET DT CA(WC)=value           Set current address (word count) to value

The DECtape controller implements these registers:

======  ==============  ====  =================================
System  Name            Size  Comments
======  ==============  ====  =================================
All     ``DTSA``        12    Status register A
All     ``DTSB``        12    Status register B
All     ``DTDB``        18    Data buffer
All     ``INT``         1     Interrupt pending flag
9,15    ``ENB``         1     Interrupt enable flag
All     ``DTF``         1     DECtape flag
7       ``BEF``         1     Block end flag
All     ``ERF``         1     Error flag
All     ``LTIME``       31    Time between lines
All     ``DCTIME``      31    Time to decelerate to a full stop
All     ``SUBSTATE``    2     Read/write command substate
All     ``POS[0:7]``    32    Position, in lines, units 0 to 7
All     ``STATT[0:7]``  18    Unit state, units 0 to 7
All     ``STOP_OFFR``   1     Stop on off-reel error
======  ==============  ====  =================================

It is critically important to maintain certain timing relationships among the DECtape parameters,
or the DECtape simulator will fail to operate correctly.

* ``LTIME`` must be at least 6
* ``DCTIME`` needs to be at least 100 times ``LTIME``

Acceleration time is set to 75% of deceleration time.

.. _TC59/TU10 magnetic tape:
.. |MT|   replace:: ``MT``
.. |TC59| replace:: ``TC59``
.. |TU10| replace:: ``TU10``
.. _MT:
.. _TC59:
.. _TU10:

|TC59|\ /\ |TU10| magnetic tape (|MT|)
--------------------------------------
Magnetic tape options include the ability to make units write-enabled or or write-locked. ::

    SET MTn LOCKED                Set unit n write-locked
    SET MTn WRITEENABLED          Set unit n write-enabled

Magnetic tape units can be set to a specific reel capacity in MB,
or to unlimited capacity::

    SET MTn CAPAC=m               Set unit n capacity to m MB (0 = unlimited)
    SHOW MTn CAPAC                Show unit n capacity in MB

Units can also be set ``ENABLED`` or ``DISABLED``.

The |TC59| is a 3-cycle data break device.
The word count and current address registers are in memory.
They can be examined and modified with ``SET`` and ``SHOW`` commands::

    SHOW MT CA(WC)                Display current address (word count)
    SET MT CA(WC)=value           Set current address (word count) to value

The magnetic tape controller implements these registers:

============  ====  =========================
Name          Size  Comments
============  ====  =========================
``CMD``       18    Command
``STA``       18    Main status
``INT``       1     Interrupt pending flag
``STOP_IOE``  1     Stop on I/O error
``TIME``      24    Record delay
``UST[0:7]``  24    Unit status, units 0 to 7
``POS[0:7]``  32    Position, units 0 to 7
============  ====  =========================

Error handling is as follows:

============  =====================================
Error         Processed as
============  =====================================
Not attached  Tape not ready; if ``STOP_IOE``, stop
End-of-file   Bad tape
OS I/O error  Parity error; if ``STOP_IOE``, stop
============  =====================================

.. _DR15C Parallel Interface:
.. |DR|    replace:: ``DR``
.. |DR15C| replace:: ``DR15C``
.. _DR:
.. _DR15C:

|DR15C| parallel interface (PDP-15/76 only)
-------------------------------------------
The |DR15C| is a parallel interface that provides the PDP-15 side of the UC15 control interface in a PDP-15/76 system.
It is disabled by default.
Enabling the |DR| creates the shared memory and status interfaces for communicating with the UC15.

The |DR15C| implements these registers:

==============  ====  =========================================
Name            Size  Comments
==============  ====  =========================================
``TCBP``        18    TCBP pointer
``TCBACK``      1     TCBP write acknowledge
``IE``          1     Interrupt enable
``REQ``         4     API requests on levels 3..0
``API0..3``     1     Interrupt request, API levels 0..3
``APIVEC0..3``  7     API vectors, API levels 0..3
``POLL``        8     Polling interval for shared state changes
==============  ====  =========================================

Usage of the |DR15C| is covered in a separate document on running a PDP-15/76 configuration.

Symbolic display and input
==========================
The 18b PDP simulators implement symbolic display and input.
Display is controlled by command-line switches:

.. table::
   :class: switches

   ======  ================================================
   ``-a``  Display as ASCII character
   ``-b``  Display as three DECsys Baudot packed characters
   ``-c``  Display as three SIXBIT packed characters
   ``-f``  Display as three FIODEC packed character
   ``-m``  Display instruction mnemonics
   ======  ================================================

The PDP-7 and PDP-9 recognize one additional switch:

.. table::
   :class: switches

   ======  =================================================
   ``-u``  Display as Unix v0 ASCII
           (two 7b ASCII characters in 9b bytes, big-endian)
   ======  =================================================

The PDP-15 recognizes two additional switches:

.. table::
   :class: switches

   ======  =====================================================
   ``-u``  Display as PDP11 ASCII
           (two 7b ASCII characters in 8b bytes, little-endian);
           16b devices only
   ``-p``  Display as packed ASCII
           (five 7b ASCII characters in two 18b words)
   ======  =====================================================

Input parsing is controlled by the first character typed in or by command-line switches:

.. table::
   :class: switches

   ===============  ==============================
   ``'`` or ``-a``  ASCII character
   ``"`` or ``-c``  Three packed SIXBIT characters
   Alphabetic       Instruction mnemonic
   Numeric          Octal number
   ===============  ==============================

The PDP-7 and PDP-9 recognize one additional input mode:

.. table::
   :class: switches

   ======  ===================================================
   ``-u``  Unix v0 ASCII (two 7b ASCII characters in 9b bytes)
   ======  ===================================================

The PDP-15 also recognizes an additional input mode:

.. table::
   :class: switches

   ======  =====================================================
   ``-p``  Five character packed ASCII string in two 18b words
   ``-u``  PDP11 ASCII (two 7b ASCII in 8b bytes, little-endian)
   ======  =====================================================

Instruction input uses standard 18b PDP assembler syntax.
There are eight instruction classes:
memory reference,
EAE,
index (PDP-15 only),
IOT,
operate,
LAW,
FP15 memory reference (PDP-15 only),
and FP15 no operand (PDP-15 only).

Memory reference instructions have the format ::

    PDP-4, PDP-7:     memref {I} address
    PDP-9:            memref{*} address
    PDP-15:           memref{*} address{,X}

where ``I`` (PDP-4, PDP-7) or ``*`` (PDP-9, PDP-15) signify indirect reference,
and ``X`` signifies indexing (PDP-15 in page mode only).
The address is an octal number in the range 0 – 017777
(PDP-4,
PDP-7,
PDP-9,
and PDP-15 in bank mode)
or 0 – 07777
(PDP-15 in page mode).

IOT instructions consist of single mnemonics;
e.g., ``KRB``, ``TLS``.
IOT instructions may be OR'd together::

    iot iot iot...

IOTs may also include the number 10,
signifying clear the accumulator ::

    iot 10

The simulator does not check the legality of IOT combinations.
IOTs for which there is no opcode may be specified as ``IOT n``,
where ``n`` is an octal number in the range 0 – 07777.

EAE instructions have the format ::

    eae {+/- shift count}

EAE instructions may be OR'd together::

    eae eae eae...

The simulator does not check the legality of EAE combinations.
EAEs for which there is no opcode may be specified as ``EAE n``,
where ``n`` is an octal number in the range ``0 – 037777``.

Index instructions (PDP-15 only) have the format ::

    index {immediate}

The ``immediate``,
if allowed,
must be in the range of −0400 to +0377.

Operate instructions have the format ::

    opr opr opr...

The simulator does not check the legality of the proposed combination.
The operands for ``MUY`` and ``DVI`` must be deposited explicitly.

The LAW instruction has the format ::

    LAW immediate

where ``immediate`` is in the range of 0 to 017777.

FP15 memory reference instructions occupy two successive words and have the format ::

    fpmem{*} address

where ``*`` signifies indirect addressing.
The ``address`` is a number in the range 0 – 0377777.

FP15 no operand instructions occupy two successive words and have the format ::

    fpop

The second word is ignored on output and set to 0 on input.

Character sets
==============
The PDP-4's console was an ASR-28 Teletype;
its character encoding was Baudot.
The PDP-4's line printer used a modified Hollerith character set.
The PDP-7's and PDP-9's consoles were KSR-33 Teletypes;
their character sets were basically ASCII.
The PDP-7's and PDP-9's line printers used sixbit encoding
(ASCII codes 040 – 0137 masked to six bits).
The PDP-15's I/O devices were all ASCII.
The following table provides equivalences between ASCII characters and the PDP-4's I/O devices.
In the console table,
"FG" stands for "figures" (uppercase).

===================  ===============  ==================
ASCII                PDP-4            PDP-4
                     console          line printer
===================  ===============  ==================
``000`` – ``006``    None             None
Bell                 FG+\ ``024``     None
``010`` – ``011``    None             None
LF                   ``010``          None
``013`` – ``014``    None             None
CR                   ``002``          None
``016`` – ``037``    None             None
Space                ``004``          ``000``
``!``                FG+\ ``026``     None
``"``                FG+\ ``021``     None
``#``                FG+\ ``005``     None
``$``                FG+\ ``062``     None
``%``                None             None
``&``                FG+\ ``013``     None
``'``                FG+\ ``032``     None
``(``                FG+\ ``036``     ``057``
``)``                FG+\ ``011``     ``055``
``*``                None             ``072``
``+``                None             ``074``
``,``                FG+\ ``006``     ``033``
``-``                FG+\ ``030``     ``054``
``.``                FG+\ ``007``     ``073``
``/``                FG+\ ``027``     ``021``
``0``                FG+\ ``015``     ``020``
``1``                FG+\ ``035``     ``001``
``2``                FG+\ ``031``     ``002``
``3``                FG+\ ``020``     ``003``
``4``                FG+\ ``012``     ``004``
``5``                FG+\ ``001``     ``005``
``6``                FG+\ ``025``     ``006``
``7``                FG+\ ``034``     ``007``
``8``                FG+\ ``014``     ``010``
``9``                FG+\ ``003``     ``011``
``:``                FG+\ ``016``     None
``;``                FG+\ ``017``     None
``<``                None             ``034``
``=``                None             ``053``
``>``                None             ``034``
``?``                FG+\ ``023``     ``037``
``@``                None             {MID DOT} ``040``
``A``                ``030``          ``061``
``B``                ``023``          ``062``
``C``                ``016``          ``063``
``D``                ``022``          ``064``
``E``                ``020``          ``065``
``F``                ``026``          ``066``
``G``                ``013``          ``067``
``H``                ``005``          ``070``
``I``                ``014``          ``071``
``J``                ``032``          ``041``
``K``                ``036``          ``042``
``L``                ``011``          ``043``
``M``                ``007``          ``044``
``N``                ``006``          ``045``
``O``                ``003``          ``046``
``P``                ``015``          ``047``
``Q``                ``035``          ``050``
``R``                ``012``          ``051``
``S``                ``024``          ``022``
``T``                ``001``          ``023``
``U``                ``034``          ``024``
``V``                ``017``          ``025``
``W``                ``031``          ``026``
``X``                ``027``          ``027``
``Y``                ``025``          ``030``
``Z``                ``021``          ``031``
``[``                None             None
``\``                None             {OVERLINE} ``056``
``]``                None             None
``^``                None             {UP ARROW} ``035``
``_``                None             UC+\ ``040``
``0140`` – ``0177``  None             None
===================  ===============  ==================

DECsys Baudot packs the 5-bit character code,
and the figure/letters flag,
into six bits as follows::

    bits<0:4>         Baudot 5b character code
    bit<5>            0 = letters, 1 = figures
