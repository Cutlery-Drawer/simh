.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. |ATTACH|   replace:: :html:`<samp>ATTACH</samp>`
.. |DEPOSIT|  replace:: :html:`<samp>DEPOSIT</samp>`
.. |EXAMINE|  replace:: :html:`<samp>EXAMINE</samp>`
.. |SET|      replace:: :html:`<samp>SET</samp>`
.. |n|        replace:: :html:`<samp><var>n</var></samp>`
.. |`|        replace:: :literal:`\``

**********************************
     IBM 701 simulator usage
**********************************
:Date: 2018-07-25
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

Introduction
============
The IBM 701,
also known as the "Defense Calculator",
was introduced by IBM on April 7, 1953.
This computer was the start of the IBM 700 and 7000 lines.
Memory was 2048 36-bit words.
Each instruction could be signed plus or minus;
plus would access memory as 18-bit words,
minus as 36-bit words.
There was an expansion option to add another 2048 words of memory,
but I can't find documentation on how it worked.
Memory cycle time was 12 microseconds.
The 701 was withdrawn from the market on October 1, 1954,
replaced by the 704 and 702.
A total of 19 machines were installed.

Simulator files
===============
To compile the IBM 701,
you must define ``USE_INT64`` and ``I701`` as part of the compilation command-line.

+--------------+------------------+-----------------------------------------+
| Subdirectory | File             | Contains                                |
+==============+==================+=========================================+
| ``I7000/``   | ``i7000_defs.h`` | IBM 7000 simulators general definitions |
|              +------------------+-----------------------------------------+
|              | ``i701_defs.h``  | IBM 701 simulator specific definitions  |
|              +------------------+-----------------------------------------+
|              | ``i7000_chan.c`` | Generic channel interface               |
|              +------------------+-----------------------------------------+
|              | ``i701_cpu.c``   | 701 CPU, channel, interface             |
|              +------------------+-----------------------------------------+
|              | ``i701_chan.c``  | 701 channel                             |
|              +------------------+-----------------------------------------+
|              | ``i701_sys.c``   | 701 system interface                    |
|              +------------------+-----------------------------------------+
|              | ``i7090_cdr.c``  | 711 card reader                         |
|              +------------------+-----------------------------------------+
|              | ``i7090_cdp.c``  | 721 card punch                          |
|              +------------------+-----------------------------------------+
|              | ``i7090_lpr.c``  | 716 line printer                        |
|              +------------------+-----------------------------------------+
|              | ``i7090_drum.c`` | 733 drum memory interface               |
|              +------------------+-----------------------------------------+
|              | ``i7000_mt.c``   | 729 tape controller                     |
+--------------+------------------+-----------------------------------------+

IBM 701 features
================
The IBM 701 simulator is configured as follows:

==============  ============================
Device name(s)  Simulates
==============  ============================
|CPU|_          701 CPU with 2KW of memory
|CH|_           701 channel device
|MT|_           729 magnetic tape controller
|CDR|_          711 card reader
|CDP|_          721 card punch
|LP|_           716 line printer
|DR0|_          733 drum
==============  ============================

The 701 simulator implements several unique stop conditions:

* Undefined CPU instruction
* Divide check on a divide and halt instruction
* Write select of a write protected device

.. |LOAD| replace:: :html:`<samp>LOAD</samp>`
.. _LOAD:

The |LOAD| command will load a card binary image file into memory.
An octal dump file,
or a pseudo assembly code.

.. |CPU| replace:: ``CPU``
.. _CPU:

|CPU|
-----
Memory size is 2KW on a standard CPU.

|CPU| registers include the visible state of the processor as well as the control registers for the interrupt system.

====================  ====  ======================
Name                  Size  Comments
====================  ====  ======================
``IC``                15    Program counter
``AC``                38    Accumulator
``MQ``                36    Multiplier-quotient
``SW1``\ ..\ ``SW6``  1     Sense switches 1..6
``SW``                6     Sense switches
``SL1``\ ..\ ``SL4``  1     Sense lights 1..4
``ACOVF``             1     AC overflow indicator
``DVC``               1     Divide check indicator
``IOC``               1     I/O check indicator
====================  ====  ======================

The CPU can maintain a history of the most recently executed instructions.
This is controlled by the :html:`<samp>SET CPU HISTORY</samp>` and :html:`<samp>SHOW CPU HISTORY</samp>` commands:

==================================================  ======================================
:html:`<samp>SET CPU HISTORY</samp>`                Clear history buffer
:html:`<samp>SET CPU HISTORY=0 </samp>`             Disable history
:html:`<samp>SET CPU HISTORY=<var>n</var></samp>`   Enable history, length = |n|
:html:`<samp>SHOW CPU HISTORY</samp>`               Print CPU history
:html:`<samp>SHOW CPU HISTORY=<var>n</var></samp>`  Print first |n| entries of CPU history
==================================================  ======================================

.. |CH| replace:: ``CH``
.. _I/O channel:
.. _CH:

I/O channel (|CH|)
------------------
The channel device on the 701 is only used by the simulator,
and has no controls or registers.

Peripherals
-----------

.. _711 card reader:
.. _CDR:
.. |CDR| replace:: ``CDR``

711 card reader (|CDR|)
"""""""""""""""""""""""
The card reader (|CDR|) reads data from a disk file.
Cards are simulated as ASCII lines with terminating newlines.
Card reader files can either be text (one character per column) or column binary (two characters per column).
The file type can be specified with a |SET| command:

==========================================  =================================
:html:`<samp>SET CDR FORMAT=TEXT</samp>`    Set ASCII text mode
:html:`<samp>SET CDR FORMAT=BINARY</samp>`  Set for binary card images
:html:`<samp>SET CDR FORMAT=BCD</samp>`     Set for BCD records
:html:`<samp>SET CDR FORMAT=CBN</samp>`     Set for column binary BCD records
:html:`<samp>SET CDR FORMAT=AUTO</samp>`    Automatically determine format
==========================================  =================================

or in the |ATTACH| command:

====================================================================  ==================================================================
:html:`<samp>ATTACH CDR <var>file</var></samp>`                       Attach a file
:html:`<samp>ATTACH CDR -f <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp>ATTACH CDR -s <var>file</var></samp>`                    Add file onto current cards to read
:html:`<samp>ATTACH CDR -e <var>file</var></samp>`                    After file is read in, the reader will receive an end-of-file flag
====================================================================  ==================================================================

The card reader can be booted with:

=============================  ==========================
:html:`<samp>BOOT CDR</samp>`  Load first 3 words of card
=============================  ==========================

Error handling is as follows:

============  =====================
Error         Processed as
============  =====================
Not attached  Report error and stop
End of file   Out of cards
OS I/O error  Report error and stop
============  =====================

.. _721 card punch:
.. _CDP:
.. |CDP| replace:: ``CDP``

721 card punch (|CDP|)
""""""""""""""""""""""
The card reader (|CDP|) writes data to a disk file.
Cards are simulated as ASCII lines with terminating newlines.
Card punch files can either be text (one character per column) or column binary (two characters per column).
The file type can be specified with a |SET| command:

==========================================  =================================
:html:`<samp>SET CDP FORMAT=TEXT</samp>`    Set ASCII text mode
:html:`<samp>SET CDP FORMAT=BINARY</samp>`  Set for binary card images
:html:`<samp>SET CDP FORMAT=BCD</samp>`     Set for BCD records
:html:`<samp>SET CDP FORMAT=CBN</samp>`     Set for column binary BCD records
:html:`<samp>SET CDP FORMAT=AUTO</samp>`    Automatically determine format
==========================================  =================================

or in the |ATTACH| command:

====================================================================  ===================================
:html:`<samp>ATTACH CDP <var>file</var></samp>`                       Attach a file
:html:`<samp>ATTACH CDP -f <var>format</var> <var>file</var></samp>`  Attach a file with the given format
====================================================================  ===================================

Error handling is as follows:

============  =====================
Error         Processed as
============  =====================
Not attached  Report error and stop
OS I/O error  Report error and stop
============  =====================

.. _716 line printer:
.. _LP:
.. |LP| replace:: ``LP``

716 line printer (|LP|)
"""""""""""""""""""""""
The line printer (|LP|) writes data to a disk file as ASCII text with terminating newlines.
Currently set to handle standard signals to control paper advance.

=======================================================  =============================================
:html:`<samp>SET LP NO/ECHO</samp>`                      Set echoing to console of line-printer output
:html:`<samp>SET LP LINESPERPAGE=<var>lpp</var></samp>`  Set number of lines per page on printer
=======================================================  =============================================

The Printer supports the following :html:`<samp>SPRA <var>n</var></samp>` selection pulses for controlling spacing
(spacing occurs before the line is printed):

===========================  ========================================================
:html:`<samp>SPRA 1</samp>`  To top of form
:html:`<samp>SPRA 2</samp>`  Single-space
:html:`<samp>SPRA 3</samp>`  Double-space, before printing line
:html:`<samp>SPRA 4</samp>`  Triple-space, before printing line
:html:`<samp>SPRA 9</samp>`  Suppress line-feed after print, prints characters 73-120
:html:`<samp>SPT</samp>`     Will skip if any printer line has been pulsed
===========================  ========================================================

Default with no :html:`<samp>SPRA</samp>` is to single-space before printing.

Error handling is as follows:

============  =====================
Error         Processed as
============  =====================
Not attached  Report error and stop
OS I/O error  Report error and stop
============  =====================

.. _729 magnetic tape:
.. _MT:
.. |MT| replace:: ``MT``

729 magnetic tape (|MT|)
""""""""""""""""""""""""
These come in groups of 10 units each.
``MT0`` is unit 10.

Each individual tape-drive support several options:
``MTA`` used as an example.

===================================================  ==================================
:html:`<samp>SET MT<var>n</var> REWIND</samp>`       Set the mag tape to the load point
:html:`<samp>SET MT<var>n</var> LOCKED</samp>`       Set the mag tape to be read-only
:html:`<samp>SET MT<var>n</var> WRITEENABLE</samp>`  Set the mag tape to be writable
:html:`<samp>SET MT<var>n</var> LOW</samp>`          Set mag tape to low density
:html:`<samp>SET MT<var>n</var> HIGH</samp>`         Set mag tape to high density
===================================================  ==================================

Options: Density LOW/HIGH does not change the format of how tapes are written,
and is only for informational purposes only.

Tape drives can be booted with:

========================================  ===================================
:html:`<samp>BOOT MT<var>n</var></samp>`  Read in first three words of record
========================================  ===================================

.. _733 drum:
.. _DR:
.. _DR0:
.. |DR|  replace:: ``DR``
.. |DR0| replace:: ``DR0``

733 drum (|DR|)
"""""""""""""""
Up to 16 units can be attached to the CPU,
all are on pseudo-channel 0.
Each drum is 2048K words in size.
They are all stored in one file.

===============================================  ===========================================
:html:`<samp>SET DR0 UNITS=<var>n</var></samp>`  Set number of units to of storage to attach
===============================================  ===========================================

Drum unit 0 can be booted with:

=========================================  ===================================
:html:`<samp>BOOT DR0<var>n</var></samp>`  Read in first three words of record
=========================================  ===================================

Symbolic display and input
==========================
The IBM 701 simulator implements symbolic display and input.
These are controlled by the following switches to the |EXAMINE| and |DEPOSIT| commands:

.. table::
   :class: switches

   ======  ===================================
   ``-m``  Display/enter symbolic machine code
   ``-c``  Display/enter BCD characters
   \       Display/enter octal data
   ======  ===================================

The symbolic input/display supports one format for instruction display::

    <opcode>,<sign><octal address>,<opcode>,<sign><octal address>

A negative address specifies the lower 18 bits of the given memory location.

Sim load
========
The load command looks at the extension of the file to determine how to load the file.

========  ========================================
``.crd``  | Load a card image file into memory.
          | standard 709 format + 1 card loader.
``.oct``  | Load an octal deck:
          | address <blank> octal <blank> octal...
``.sym``  | Load a 709 symbolic deck.
          | address instruction..
          | address BCD string
          | address OCT octal
          | octal
========  ========================================

Character codes
===============

==========  ==========  ===============  ===  ========  ============
Commercial  Scientific  ASCII            BCD  Card      Remark
==========  ==========  ===============  ===  ========  ============
\           \           \                00   \         Blank
``1``       \           0                01   1
``2``       \           0                02   2
``3``       \           0                03   3
``4``       \           0                04   4
``5``       \           0                05   5
``6``       \           0                06   6
``7``       \           0                07   7
``8``       \           0                10   8
``9``       \           0                11   9
``0``       \           0                12   10
``#``       ``=``       ``=``            13   3-8
``@``       ``'``       ``'``\ /\ ``@``  14   4-8
``:``       \           ``:``            15   5-8
``>``       \           ``>``            16   6-8
``√``       \           ``"``            17   7-8       Tape mark
``ƀ``       \           ``_``            20   2-8
``/``       \           ``/``            21   10-1
``S``       \           ``S``            22   10-1
``T``       \           ``T``            23   10-2
``U``       \           ``U``            24   10-3
``V``       \           ``V``            25   10-4
``W``       \           ``W``            26   10-5
``X``       \           ``X``            27   10-6
``Y``       \           ``Y``            30   10-7
``Z``       \           ``Z``            31   10-8
``#``       \           ``#``            32   10-2-8    Word mark
``,``       \           ``,``            33   10-3-8
``%``       ``(``       ``%``\ /\ ``(``  34   10-4-8
|`|         \           |`|              35   10-5-8
``\``       \           ``\``            36   10-6-8
``⧻``       \           ``{``            37   10-7-8    Segment mark
``-``       \           ``-``            40   11        Also ``-0``
``J``       \           ``J``            41   11-1
``K``       \           ``K``            42   11-2
``L``       \           ``L``            43   11-3
``M``       \           ``M``            44   11-4
``N``       \           ``N``            45   11-5
``O``       \           ``O``            46   11-6
``P``       \           ``P``            47   11-7
``Q``       \           ``Q``            50   11-8
``R``       \           ``R``            51   11-9
``!``       \           ``!``            52   11-2-8
``$``       \           ``$``            53   11-3-8
``*``       \           ``*``            54   11-4-8
``]``       \           ``]``            55   11-5-8
``;``       \           ``;``            56   11-6-8
``△``       \           ``^``            57   11-7-8
``&``       ``+``       ``&``\ /\ ``+``  60   12        Also ``+0``
``A``       \           ``A``            61   12-1
``B``       \           ``B``            62   12-2
``C``       \           ``C``            63   12-3
``D``       \           ``D``            64   12-4
``E``       \           ``E``            65   12-5
``F``       \           ``F``            66   12-6
``G``       \           ``G``            67   12-7
``H``       \           ``H``            70   12-8
``I``       \           ``I``            71   12-9
``?``       \           ``?``            72   12-2-8
``.``       \           ``.``            73   12-3-8
``⌑``       ``)``       ``)``            74   12-4-8    Lozenge
``[``       \           ``[``            75   12-5-8
``<``       \           ``<``            76   12-3-8
``⧻*``      \           ``|``            77   12-7-8    Group mark
==========  ==========  ===============  ===  ========  ============
