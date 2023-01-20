.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

*******************
LGP simulator usage
*******************
:Date: 2011-03-31
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

This memorandum documents the LGP-30 simulator.

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

``sim/lgp/``
    | ``lgp_defs.h``
    | ``lgp_cpu.c``
    | ``lgp_stddev.c``
    | ``lgp_sys.c``

LGP features
============
The LGP is configured as follows:

============  ==============================================
Device names  Simulates
============  ==============================================
``CPU``       LGP-30 or LGP-21 CPU with 4096 words of memory
``TTI``       Typewriter input (keyboard and reader)
``TTO``       Typewriter output (printer and punch)
``PTR``       High-speed paper tape reader
``PTP``       High-speed paper tape punch
============  ==============================================

The LGP simulator implements the following unique stop conditions:

- LGP-30 only: arithmetic overflow
- LGP-21 only: reference to undefined I/O device.

The ``LOAD`` and ``DUMP`` commands are not implemented.

.. |CPU| replace:: ``CPU``
.. _CPU:

|CPU|
-----
The |CPU| implements either the LGP-30 or the LGP-21::

    SET CPU LGP30                 Set LGP-30
    SET CPU LGP21                 Set LGP-21

The default is the LGP-30.
Memory size is fixed at 4096 words.
Although memory is 32b wide,
``bit<31>`` (the low-order bit) is always zero.

The following commands implement various front panel functions::

    D A value                     Equivalent to the MANUAL INPUT button
    SET CPU FILL{=value}          Equivalent to the FILL INSTRUCTION button;
                                  if no value is given, fills IR from A;
                                  else, fills IR from the specified value
    SET CPU EXEC{=value}          Equivalent to the EXECUTE button;
                                  if no value is given, executes the
                                  instruction in IR; else, executes the
                                  instruction specified by the value
    SET CPU MANUAL                Equivalent to setting the MANUAL INPUT
                                  switch on the Typewriter; Typewriter input
                                  is taken from the keyboard
    SET CPU TAPE                  Equivalent to clearing the MANUAL INPUT
                                  switch on the Typewriter; Typewriter input
                                  is taken from the Typewriter paper tape reader

The following commands control the display of information::

    SET CPU LGPHEX                Numeric displays use LGP hexadecimal encoding
    SET CPU STANDARD              Numeric displays use standard hexadecimal
    SET CPU TRACK                 Symbolic addresses are ttss, where
                                  tt = track (0-63) and
                                  ss = sector (0-63)
    SET CPU NORMAL                Symbolic addresses are normal linear
                                  addresses, from 0 to 4095.

The defaults are ``STANDARD`` hex and ``TRACK`` addresses.

The LGP-30 implements the following additional commands::

    SET CPU 4B                    Sets the CPU to 4-bit input mode
    SET CPU 6B                    Sets the CPU to 6-bit input mode
    SET CPU INPUT=TTI             Sets the CPU to read from the Typewriter
    SET CPU INPUT=PTR             Sets the CPU to read from the high-speed reader
    SET CPU OUTPUT=TTO            Sets the CPU to output to the Typewriter
    SET CPU OUTPUT=PTP            Sets the CPU to output to the high-speed punch

The defaults are 4-bit input mode,
input and output assigned to the Typewriter.

|CPU| registers include the visible state of the processor as well as the control registers for the interrupt system.

=========  ====  ===========================
Name       Size  Comments
=========  ====  ===========================
``PC``     12    Counter
``A``      32    Accumulator
``IR``     32    Instruction register
``OVF``    1     Overflow flag (LGP-21 only)
``TSW``    1     Transfer switch
``BP32``   1     Breakpoint 32 switch
``BP16``   1     Breakpoint 16 switch
``BP8``    1     Breakpoint 8 switch
``BP4``    1     Breakpoint 4 switch
``INPST``  1     Input pending flag
``INPDN``  1     Input done flag
``OUTST``  1     Output pending flag
``OUTDN``  1     Output done flag
``WRU``    8     Interrupt character
=========  ====  ===========================

.. |TTI| replace:: ``TTI``
.. _TTI:

Typewriter input (|TTI|)
------------------------
The Typewriter input consists of two units:
the keyboard (unit 0) and the paper-tape reader (unit 1).
The keyboard is permanently associated with the console window.
The paper-tape reader can be attached to a disk file.
The ``RPOS`` register specifies the number of the next data item to be read.
Thus, by changing ``RPOS``, the user can backspace or advance the reader.

The Typewriter input has the following options::

    SET TTI1 ASCII          Default tape file format is ASCII-encoded Flex
    SET TTI1 FLEX           Default tape file format is transposed Flex
    SET TTI1 CSTOP          Reader recognizes conditional stop
    SET TTI1 NOCSTOP        Reader ignores conditional stop
    SET TTI RSTART          Start the reader; equivalent to the START READER lever
    SET TTI RSTOP           Stop the reader; equivalent to the STOP READER lever
    SET TTI START           Send START signal to the CPU; equivalent
                            to the START COMPUTE lever

Transposed Flex has the tape channels in this order: ``6-1-2-3-4-5``.

The ``ATTACH`` command recognizes two switches::

    ATT -A TTI1 <file>      File format is ASCII-encoded Flex
    ATT -F TTI1 <file>      File format is transposed Flex

The Typewriter input implements these registers:

============  ====  =================================================
Name          Size  Comments
============  ====  =================================================
``BUF``       6     Data buffer
``RDY``       1     Data ready flag
``KPOS``      32    Count of keyboard characters
``RPOS``      32    Position in the reader input file
``TIME``      24    Time between keyboard polls or reader characters
``STOP_IOE``  1     Stop on I/O error
============  ====  =================================================

Error handling for the Typewriter paper-tape reader is as follows:

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

.. |TTO| replace:: ``TTO``
.. _TTO:

Typewriter output (|TTO|)
-------------------------
The Typewriter output consists of two units:
the printer (unit 0) and the paper-tape punch (unit 1).
The printer is permanently associated with the console window.
The paper-tape punch can be attached to a disk file.
The ``PPOS`` register specifies the number of the next data item to be written.
Thus, by changing ``PPOS``, the user can backspace or advance the punch.

The Typewriter output has the following options::

    SET TTO1 ASCII        Default tape file format is ASCII-encoded Flex
    SET TTO1 FLEX         Default tape file format is transposed Flex
    SET TTO1 FEED=n       Punch 'n' feed (0) characters

Transposed Flex has the tape channels in this order: ``6-1-2-3-4-5``.
The default is ASCII-encoded Flex.

The ``ATTACH`` command recognizes two switches::

    ATT -A TTO1 <file>    File format is ASCII-encoded Flex
    ATT -F TTO1 <file>    File format is transposed Flex

The Typewriter output implements these registers:

============  ==========  ======================================
Name          Size        Comments
============  ==========  ======================================
``BUF``       6           Data buffer
``UC``        1           Upper case flag
``TPOS``      32          Count of output characters
``PPOS``      32          Position in the punch output file
``TIME``      24          Time from I/O initiation to completion
``STOP_IOE``  1           Stop on I/O error
============  ==========  ======================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of tape           |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

.. |PTR| replace:: ``PTR``
.. _PTR:

High speed paper-tape reader (|PTR|)
------------------------------------
The paper tape reader (|PTR|) reads data from or a disk file.
The ``POS`` register specifies the number of the next data item to be read.
Thus, by changing ``POS``, the user can backspace or advance the reader.

The paper-tape reader has the following options::

    SET PTR ASCII          Default tape file format is ASCII-encoded Flex
    SET PTR FLEX           Default tape file format is transposed Flex

Transposed Flex has the tape channels in this order: ``6-1-2-3-4-5``.
The default is ASCII-encoded Flex.

The ``ATTACH`` command recognizes two switches::

    ATT -A PTR <file>      File format is ASCII-encoded Flex
    ATT -F PTR <file>      File format is transposed Flex

The paper tape reader implements these registers:

============  ====  ======================================
Name          Size  Comments
============  ====  ======================================
``BUF``       6     Last data item processed
``RDY``       1     Data ready flag
``POS``       32    Position in the input file
``TIME``      24    Time from I/O initiation to completion
``STOP_IOE``  1     Stop on I/O error
============  ====  ======================================

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

High speed paper-tape punch (|PTP|)
-----------------------------------
The paper tape punch (|PTP|) writes data to a disk file.
The ``POS`` register specifies the number of the next data item to be written.
Thus, by changing ``POS``,
the user can backspace or advance the punch.

The paper tape punch has the following options::

    SET PTP ASCII         Default tape file format is ASCII-encoded Flex
    SET PTP FLEX          Default tape file format is transposed Flex
    SET PTP FEED=n        Punch 'n' feed (0) characters

Transposed Flex has the tape channels in this order: ``6-1-2-3-4-5``.
The default is ASCII-encoded Flex.

The ``ATTACH`` command recognizes two switches::

    ATT -A PTP <file>     File format is ASCII-encoded Flex
    ATT -F PTP <file>     File format is transposed Flex

The paper tape punch implements these registers:

============  ====  ======================================
Name          Size  Comments
============  ====  ======================================
``BUF``       6     Last data item processed
``POS``       32    Position in the output file
``TIME``      24    Time from I/O initiation to completion
``STOP_IOE``  1     Stop on I/O error
============  ====  ======================================

Error handling is as follows:

+--------------+--------------+-----------------------+
| Error        | ``STOP_IOE`` | Processed as          |
+==============+==============+=======================+
| not attached | 1            | Report error and stop |
|              +--------------+-----------------------+
|              | 0            | Out of tape           |
+--------------+--------------+-----------------------+
| OS I/O error | x            | Report error and stop |
+--------------+--------------+-----------------------+

Symbolic display and input
==========================
The LGP simulator implements symbolic display and input.
Display is controlled by command-line switches:

.. table::
   :class: switches

   ======  ======================================
   ``-a``  Display as character (tape files only)
   ``-h``  Display as standard hexadecimal
   ``-l``  Display as LGP hexadecimal
   ``-m``  Display instruction mnemonics
   ``-n``  Display addresses in normal format
           (overrides ``SET CPU TRACK``)
   ``-t``  Display addresses as track/sector
           (overrides ``SET CPU NORMAL``)
   ======  ======================================

Input parsing is controlled by the first character typed in or by command-line switches:

.. table::
   :class: switches

   ===============  ====================
   ``'`` or ``-a``  Flex character
   ``-`` or opcode  Instruction mnemonic
   Numeric          Hexadecimal number
   ===============  ====================

LGP hexadecimal differs from standard hexadecimal in the characters used for digits 10-15

======  ============  =======
Digit   Standard hex  LPG hex
======  ============  =======
``10``  ``A``         ``F``
``11``  ``B``         ``G``
``12``  ``C``         ``J``
``13``  ``D``         ``K``
``14``  ``E``         ``Q``
``15``  ``F``         ``W``
======  ============  =======

There is only instruction format::

    {-}op address

``op`` is always a single letter.
A track/sector address
(specified by ``SET CPU TRACK`` or switch ``-t``)
is two decimal numbers between 0 and 63,
representing the track and sector.
A linear address
(specified by ``SET CPU NORMAL`` or switch ``-n``)
is one decimal number between 0 and 4095.
For example::

    sim> d -n 64 10640
    sim> ex -mn 64
    64:   B 400
    sim> ex -mt 100
    0100: B 0616

Character set
=============
The LGP Typewriter was a Friden Flexowriter.
Input was always uppercase;
output could be either uppercase or lowercase.
The following table provides equivalences between LPG Typewriter coding and ASCII.

.. |-| replace:: illegal

=====================  =================  ===========  ===========
Typewriter code (hex)  Input              Lowercase    Uppercase
                                          output       output
=====================  =================  ===========  ===========
``00``                 |-|                |-|          |-|
``01``                 ``z`` or ``Z``     ``z``        ``Z``
``02``                 ``0`` or ``)``     ``0``        ``)``
``03``                 space              space        space
``04``                 |-|                lowercase    lowercase
``05``                 ``b`` or ``B``     ``b``        ``B``
``06``                 ``1`` or ``L``     ``1``        ``L``
``07``                 ``-`` or ``_``     ``0``        ``_``
``10``                 |-|                uppercase    uppercase
``11``                 ``y`` or ``Y``     ``y``        ``Y``
``12``                 ``2`` or ``*``     ``2``        ``*``
``13``                 ``+`` or ``=``     ``+``        ``=``
``14``                 |-|                color shift  color shift
``15``                 ``r`` or ``R``     ``r``        ``R``
``16``                 ``3`` or ``"``     ``3``        ``"``
``17``                 ``;`` or ``:``     ``;``        ``:``
``20``                 newline            newline      newline
``21``                 ``i`` or ``I``     ``i``        ``I``
``22``                 ``4`` or ``^``     ``4``        ``^``
``23``                 ``/`` or ``?``     ``/``        ``?``
``24``                 |-|                backspace    backspace
``25``                 ``d`` or ``D``     ``d``        ``D``
``26``                 ``5`` or ``%``     ``5``        ``%``
``27``                 ``.`` or ``]``     ``.``        ``]``
``30``                 tab                tab          tab
``31``                 ``n`` or ``N``     ``n``        ``N``
``32``                 ``6`` or ``$``     ``6``        ``$``
``33``                 ``,`` or ``[``     ``,``        ``[``
``34``                 |-|                |-|          |-|
``35``                 ``m`` or ``M``     ``m``        ``M``
``36``                 ``7`` or ``~``     ``7``        ``~``
``37``                 ``v`` or ``V``     ``v``        ``V``
``40``                 ``'`` (cond stop)  ``'``        ``'``
``41``                 ``p`` or ``P``     ``p``        ``P``
``42``                 ``8`` or ``#``     ``8``        ``#``
``43``                 ``o`` or ``O``     ``o``        ``O``
``44``                 |-|                |-|          |-|
``45``                 ``e`` or ``E``     ``e``        ``E``
``46``                 ``9`` or ``(``     ``9``        ``(``
``47``                 ``x`` or ``X``     ``x``        ``X``
``50``                 |-|                |-|          |-|
``51``                 ``u`` or ``U``     ``u``        ``U``
``52``                 ``f`` or ``F``     ``f``        ``F``
``53``                 |-|                |-|          |-|
``54``                 |-|                |-|          |-|
``55``                 ``t`` or ``T``     ``t``        ``T``
``56``                 ``g`` or ``G``     ``g``        ``G``
``57``                 |-|                |-|          |-|
``60``                 |-|                |-|          |-|
``61``                 ``h`` or ``H``     ``h``        ``H``
``62``                 ``j`` or ``J``     ``j``        ``J``
``63``                 |-|                |-|          |-|
``64``                 |-|                |-|          |-|
``65``                 ``c`` or ``C``     ``c``        ``C``
``66``                 ``k`` or ``K``     ``k``        ``K``
``67``                 |-|                |-|          |-|
``70``                 |-|                |-|          |-|
``71``                 ``a`` or ``A``     ``a``        ``A``
``72``                 ``q`` or ``Q``     ``q``        ``Q``
``73``                 |-|                |-|          |-|
``74``                 |-|                |-|          |-|
``75``                 ``s`` or ``S``     ``s``        ``S``
``76``                 ``w`` or ``W``     ``w``        ``W``
``77``                 |-|                |-|          |-|
=====================  =================  ===========  ===========

Certain characters on the Flexowriter keyboard don't exist in ASCII.
The following table provides ASCII substitution characters for the unique Flexowriter characters
(this is compatible with the coding in the LGP30 paper tape archive):

========================  =======  =====
Typewriter code (hex)     Flex     ASCII
========================  =======  =====
``UC 12``                 delta    ``^``
``UC 1E``                 pi       ``~``
``UC 22``                 sigma    ``#``
========================  =======  =====

Certain Flexowriter codes have no character equivalent of any kind.
For paper-tape reader and punch files,
these are encoded as ``!dd``,
where ``dd`` is a decimal number between 0 and 63.
