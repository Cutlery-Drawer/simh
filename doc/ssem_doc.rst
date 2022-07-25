.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

***********************************
       SSEM simulator usage
***********************************
:Date: 2011-05-14
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

This memorandum documents the SSEM simulator.

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

``sim/ssem/``
    | ``ssem_defs.h``
    | ``ssem_cpu.c``
    | ``ssem_sys.c``

SSEM features
=============
The SSEM is configured as follows:

============  ================================
Device names  Simulates
============  ================================
``CPU``       SSEM CPU with 32 words of memory
============  ================================

The ``LOAD`` and ``DUMP`` commands are implemented.
They use a binary file with the extension ``.st`` ("store").

CPU
---
The CPU implements Manchester SSEM (Small Scale Experimental Machine):

Memory size is fixed at 32 words.
Memory is 32b wide.

CPU registers include the visible state of the processor system.

======  ====  ===================
Name    Size  Comments
======  ====  ===================
``CI``  5     Current instruction
``A``   32    Accumulator
======  ====  ===================

Symbolic display and input
==========================
The SSEM simulator implements symbolic display and input.
Display is controlled by command-line switches:

======  =========================================================
``-h``  Display as hexadecimal
``-d``  Display as decimal
``-b``  Display as backwards binary. Least significant bit first.
``-m``  Display instruction mnemonics
======  =========================================================

Input parsing is controlled by the first character typed in or by command-line switches::

    opcode            instruction mnemonic

There is only one instruction format::

    op address

Instructions follows the mnemonic style used in the 1998 competition reference manual,
`The Manchester University Small Scale Experimental Machine Programmer's Reference manual`__.

__ https://web.archive.org/web/20110209134408/http://www.computer50.org/mark1/prog98/ssemref.html

``op`` is one of the following ``JMP``, ``JRP``, ``LDN``, ``STO``, ``SUB``, ``CMP`` or ``STOP``.
A linear address is one decimal number between 0 and 31.
For example::

    sim> d -m 31 LDN 21
    sim> ex -m 31
    31:   LDN 21
    sim> ex -h 31
    31:   00004015
    sim> ex -b 31
    31:   10101000000000100000000000000000
    sim> ex -d 31
    31:   16405
