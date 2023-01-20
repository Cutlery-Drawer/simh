.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

******************************
GRI-909/GRI-99 simulator usage
******************************
:Date: 2008-12-01
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

This memorandum documents the GRI-909 simulator.

Simulator files
===============

``sim/``
    | ``scp.h``
    | ``sim_console.h``
    | ``sim_defs.h``
    | ``sim_fio.h``
    | ``sim_rev.h``
    | ``sim_sock.h``
    | ``sim_timer.h``
    | ``sim_tmxr.h``
    | ``scp.c``
    | ``sim_console.c``
    | ``sim_fio.c``
    | ``sim_sock.c``
    | ``sim_timer.c``
    | ``sim_tmxr.c``

``sim/gri/1``
    | ``gri_defs.h``
    | ``gri_cpu.c``
    | ``gri_stddev.c``
    | ``gri_sys.c``

GRI-909/GRI-99 features
=======================
The GRI-909 is configured as follows:

==============  =============================================
Device name(s)  Simulates
==============  =============================================
``CPU``         GRI-909/GRI-99 CPU with up to 32KW of memory
``HSR``         S42-004 high-speed reader
``HSP``         S42-004 high-speed punch
``TTI``         S42-001 Teletype input
``TTO``         S42-002 Teletype output
``RTC``         Real-time clock
==============  =============================================

The GRI-909 simulator implements the following unique stop conditions:

- An unimplemented operator is referenced, and register ``STOP_OPR`` is set
- An invalid interrupt request is made

The ``LOAD`` command has an optional argument to specify the load address::

    LOAD <filename> {<starting address>}

The ``LOAD`` command loads a paper-tape bootstrap format file at the specified address.
If no address is specified, loading starts at location 200.
The ``DUMP`` command is not supported.

CPU
---
The only CPU options are the presence of the extended arithmetic operator and the size of main memory. ::

    SET CPU GRI909   Set CPU model to GRI-909
    SET CPU GRI99    Set CPU model to GRI-99
    SET CPU AO       Enable extended arithmetic operator
    SET CPU EAO      Disable arithmetic operator
    SET CPU EAO      Enable extended arithmetic operator
    SET CPU NOEAO    Disable extended arithmetic operator
    SET CPU GPR      Enable general registers
    SET CPU NOGPR    Disable general registers
    SET CPU BSWPK    Enable byte swap/pack
    SET CPU NOBSWPK  Disable byte swap/pack
    SET CPU 4K       Set memory size = 4K
    SET CPU 8K       Set memory size = 8K
    SET CPU 12K      Set memory size = 12K
    SET CPU 16K      Set memory size = 16K
    SET CPU 20K      Set memory size = 20K
    SET CPU 24K      Set memory size = 24K
    SET CPU 28K      Set memory size = 28K
    SET CPU 32K      Set memory size = 32K

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 32K.
The default configuration is a GRI-909 with ``AO``, ``EAO``, and ``GPR``.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

====================  ======  ============================================
Name                  Size    Comments
====================  ======  ============================================
``SC``                15      Sequence counter
``AX``                16      Arithmetic operator input register 1
``AY``                16      Arithmetic operator input register 2
``AO``                16      Arithmetic operator output register
``TRP``               16      TRP register
``MSR``               16      Machine status register
``ISR``               16      Interrupt status register
``BSW``               16      Byte swapper buffer
``BPK``               16      Byte packer buffer
``GR1``\ ..\ ``GR6``  16      General registers 1 to 6
``XR``                16      Index register (GRI-99 only)
``BOV``               1       Bus overflow (``MSR<15>``)
``L``                 1       Link (``MSR<14>``)
``FOA``               2       Arithmetic operator function (``MSR<9:8>``)
``AOV``               1       Arithmetic overflow (``MSR<0>``)
``IR``                16      Instruction register (read-only)
``MA``                16      Memory address register (read-only)
``SWR``               16      Switch register
``DR``                16      Display register
``THW``               6       Selects operator displayed in DR
``IREQ``              16      Interrupt requests
``ION``               1       Interrupts enabled
``INODEF``            1       Interrupts not deferred
``BKP``               1       Breakpoint request
``SCQ[0:63]``         15      SC prior to last jump or interrupt;
                              most recent SC change first
``STOP_OPR``          1       Stop on undefined operator
``WRU``               8       Interrupt character
====================  ======  ============================================

Programmed I/O devices
----------------------

S42-004 high speed reader (HSR)
"""""""""""""""""""""""""""""""
The paper tape reader (HSR) reads data from or a disk file.
The ``POS`` register specifies the number of the next data item to be read.
Thus, by changing ``POS``, the user can backspace or advance the reader.

The paper tape reader implements these registers:

==============  ======  =====================================
Name            Size    Comments
==============  ======  =====================================
``BUF``         8       Last data item processed
``IRDY``        1       Device ready flag
``IENB``        1       Device interrupt enable flag
``POS``         32      Position in the input file
``TIME``        24      Time from I/O initiation to interrupt
``STOP_IOE``    1       Stop on I/O error
==============  ======  =====================================

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

S42-001 teletype input (TTI)
""""""""""""""""""""""""""""
The Teletype interfaces (TTI, TTO) can be set to one of four modes,
``KSR``, ``7P``, ``7B``, or ``8B``:

=========  ======================  ===================================
Mode       Input characters        Output characters
=========  ======================  ===================================
``KSR``    Lowercase converted     Lowercase converted to uppercase,
           to uppercase,           high-order bit cleared,
           high-order bit set      non-printing characters suppressed
``7P``     High-order bit cleared  High-order bit cleared,
                                   non-printing characters suppressed
``7B``     High-order bit cleared  High-order bit cleared
``8B``     No changes              No changes
=========  ======================  ===================================

The default mode is ``KSR``.

The Teletype input (TTI) polls the console keyboard for input.
It implements these registers:

========  ====  ============================
Name      Size  Comments
========  ====  ============================
``BUF``   8     Last data item processed
``IRDY``  1     Device ready flag
``IENB``  1     Device interrupt enable flag
``POS``   32    Number of characters input
``TIME``  24    Keyboard polling interval
========  ====  ============================

S42-002 Teletype output (TTO)
"""""""""""""""""""""""""""""
The Teletype output (TTO) writes to the simulator console window.
It implements these registers:

========  ====  =====================================
Name      Size  Comments
========  ====  =====================================
``BUF``   8     Last data item processed
``ORDY``  1     Device ready flag
``IENB``  1     Device interrupt enable flag
``POS``   32    Number of characters output
``TIME``  24    Time from I/O initiation to interrupt
========  ====  =====================================

Real-time clock (RTC)
"""""""""""""""""""""
The real-time clock (CLK) implements these registers:

========  ====  ========================
Name      Size  Comments
========  ====  ========================
``RDY``   1     Device ready flag
``IENB``  1     Interrupt enable flag
``TIME``  24    Clock interval
========  ====  ========================

The real-time clock autocalibrates;
the clock interval is adjusted up or down so that the clock tracks actual elapsed time.

Symbolic display and input
==========================
The GRI-909 simulator implements symbolic display and input.
Display is controlled by command-line switches:

.. table::
   :class: switches

   ======  ======================================
   ``-a``  Display as ASCII character
   ``-c``  Display as two packed ASCII characters
   ``-m``  Display instruction mnemonics
   ======  ======================================

Input parsing is controlled by the first character typed in or by command-line switches:

.. table::
   :class: switches

   ===============  ===========================
   ``'`` or ``-a``  ASCII character
   ``"`` or ``-c``  Two packed ASCII characters
   Alphabetic       Instruction mnemonic
   Numeric          Octal number
   ===============  ===========================

Instruction input uses modified GRI-909 basic assembler syntax.
There are thirteen different instruction formats.
Operators, functions, and tests may be octal or symbolic;
jump conditions and bus operators are always symbolic.
Addresses may be prefixed with ``#``,
indicating indexing (GRI-99 only).

Function out, general
    :Syntax:           ``FO`` *function*\ ,\ *operator*
    :Function symbols: ``INP``, ``IRDY``, ``ORDY``, ``STRT``
    :Example:          ``FO ORDY,TTO``

Function out, named
    :Syntax:           ``FO{M|I|A}`` *function*
    :Function symbols: | ``M``: ``CLL``, ``CML``, ``STL``, ``HLT``
                       | ``I``: ``ICF``, ``ICO``
                       | ``A``: ``ADD``, ``AND``, ``XOR``, ``OR``
    :Example:          ``FOA XOR``

Sense function, general
    :Syntax:       ``SF`` *operator*\ ,\ ``{NOT}`` *tests*
    :Test symbols: ``IRDY``, ``ORDY``
    :Example:      ``SF HSR,IRDY``

Sense function, named
    :Syntax:       ``SF{M|A} {NOT}`` *tests*
    :Test symbols: | ``M``: ``POK``, ``BOV``, ``LNK``
                   | ``A``: ``SOV``, ``AOV``
    :Example:      ``SFM NOT BOV``

Register to register
    :Syntax:         ``RR{C}`` *src*\ ,{\ *bus op*\ ,}\ *dst*
    :Bus op symbols: ``P1``, ``L1``, ``R1``
    :Example:        ``RRC AX,P1,AY``

Zero to register
    :Syntax:         ``ZR{C}`` {\ *bus op*\ ,}\ *dst*
    :Bus op symbols: ``P1``, ``L1``, ``R1``
    :Example:        ``ZR P1,GR1``

Register to self
    :Syntax:         ``RS{C}`` *dst*\ {,\ *bus op*\ }
    :Bus op symbols: ``P1``, ``L1``, ``R1``
    :Example:        ``RS AX,L1``

Jump unconditional or named condition
    :Syntax:         ``J{U|O|N}{D}`` *address*
    :Example:        ``JUD 1400``

Jump conditional
    :Syntax:         ``JC{D}`` *src*\ ,\ *cond*\ ,\ *address*
    :Cond symbols:   ``NEVER``, ``ALWAYS``, ``ETZ``, ``NEZ``, ``LTZ``, ``GEZ``, ``LEZ``, ``GTZ``
    :Example:        ``JC AX,LEZ,200``

Register to memory
    :Syntax:         ``RM{I|D|ID}`` *src*\ ,{\ *bus op*\ ,}\ *address*
    :Bus op symbols: ``P1``, ``L1``, ``R1``
    :Example:        ``RMD AX,P1,1315``

Zero to memory
    :Syntax:         ``ZM{I|D|ID}`` {\ *bus op*\ ,}\ *address*
    :Bus op symbols: ``P1``, ``L1``, ``R1``
    :Example:        ``ZM P1,5502``

Memory to register
    :Syntax:         ``MR{I|D|ID}`` *address*\ ,{\ *bus op*\ ,}\ *dst*
    :Bus op symbols: ``P1``, ``L1``, ``R1``
    :Example:        ``MRI 1405,GR6``

Memory to self:
    :Syntax:         ``MS{I|D|ID}`` *address*\ {,\ *bus op*\ }
    :Bus op symbols: ``P1``, ``L1``, ``R1``
    :Example:        ``MS 3333,P1``
