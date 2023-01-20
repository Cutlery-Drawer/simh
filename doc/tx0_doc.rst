.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

***********************************
       TX-0 simulator usage
***********************************
:Date: 2012-11-09
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

This memorandum documents the TX-0 simulator.

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

``sim/TX-0/``
    | ``tx0_defs.h``
    | ``tx0_cpu.c``
    | ``tx0_stddev.c``
    | ``tx0_sys.c``
    | ``tx0_sys_orig.c``

TX-0 features
=============
The TX-0 is configured as follows:

===============  ================================================
Device name(s)   Simulates
===============  ================================================
``CPU``          TX-0 CPU with up to 64KW of memory
``PETR``         Photo Electric Tape Rader
``PTP``          Paper Tape Punch
``TTI``          FlexoWriter input
``TTO``          FlexoWriter output
``DPY``          512×512 7″ high-persistence phosphor CRT Display
===============  ================================================

The TX-0 simulator implements the following unique stop conditions:

- An unimplemented operator is referenced, and register ``STOP_OPR`` is set
- An invalid interrupt request is made

The ``LOAD`` commands has an optional argument to specify the load address::

    LOAD <filename> {<starting address>}

The ``LOAD`` command loads a paper-tape bootstrap format file at the specified address.
If no address is specified, loading starts at location 200.
The ``DUMP`` command is not supported.

CPU
---
The TX-0 was upgraded over the years from the original 1956 "Standard" instruction set to a later "Extended" instruction set completed in 1962.
In addition, the TX-0 CPU can operate in one of three operating modes:

============  ============================================
Read In	Mode  Instruction words are fetched from the tape.
Test Mode     Instruction words are fetched from the TBR.
Normal Mode   Instruction words are fetched from memory.
============  ============================================

Only CPU options are the presence of the extended arithmetic operator and the size of main memory. ::

    SET CPU TX0STD                Set CPU model to TX-0 "Standard"
    SET CPU TX0EXT                Set CPU model to TX-0 "Extended"
    SET CPU NORMAL                Set CPU mode to Normal Mode
    SET CPU TEST                  Set CPU Mode to Test Mode
    SET CPU READIN                Set CPU Mode to Read In Mode
    SET CPU 4K                    Set memory size = 4K (T-Memory)
    SET CPU 8K                    Set memory size = 8K (T-Memory)
    SET CPU 64K                   Set memory size = 64K (Core Memory)

If memory size is being reduced,
and the memory being truncated contains non-zero data,
the simulator asks for confirmation.
Data in the truncated portion of memory is lost.
Initial memory size is 64K.
The default configuration is a TX-0 with AO, EAO, and GPR.

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

==========  ====  =======================================================================
Name        Size  Comments
==========  ====  =======================================================================
``MBR``     18    Memory Buffer Register
``AC``      18    Accumulator
``MAR``     16    Memory Address Register
``PC``      16    Program Counter
``IR``      5     Instruction Register (5 bits in Extended mode, 2 bits in Standard mode)
``LR``      18    Live Register
``TBR``     18    Toggle Switch Buffer Register
``TAC``     18    Toggle Switch Accumulator
``XR``      14    Index Register (Extended mode only)
``T``       1     Test Mode flip-flop (Read only)
``R``       1     Read In Mode flip-flop (Read only)
``LP``      2     Light Pen / Light Gun flip-flops
==========  ====  =======================================================================

I/O devices
-----------
The TX-0 includes several I/O devices,
and unlike more modern machines,
these devices are not memory or I/O mapped,
but rather have specific CPU operate orders to access them.

Photoelectric tape reader (PETR)
""""""""""""""""""""""""""""""""
The PETR is a 250 line per minute Ferranti photoelectric paper tape reader using standard seven-hole Flexowriter tape that was modified to solid state circuitry.
Lines without seventh hole punched are ignored by the PETR.
As each line of the tape is read in,
the data is stored into an 18-bit ``BUF`` register with bits mapped as follows:

====  =======
Tape  ``BUF``
====  =======
0     0
1     3
2     6
3     9
4     12
5     15
====  =======

Up to three lines of tape may be read into a single the single ``BUF`` register.
Before subsequent lines are read,
the ``BUF`` register is cycled one bit right.

The PETR reads data from or a disk file.
The POS register specifies the number of the next data item to be read.
Thus, by changing POS, the user can backspace or advance the reader.

The PETR supports the ``BOOT`` command.
``BOOT PETR`` switches the CPU to Read-In mode, and starts the processor running.

The paper tape reader implements these registers:

=======  ====  ============================================================
Name     Size  Comments
=======  ====  ============================================================
``BUF``  18    18-bit buffer to store up to three lines of Paper tape input
``POS``  32    Position in the input file
=======  ====  ============================================================

Paper tape punch (PTP)
""""""""""""""""""""""
The paper tape punch (PTP) punches standard seven-hole Flexowriter tape.
The POS register specifies the number of the next data item to be written.
Thus, by changing POS, the user can backspace or advance the punch.

The paper tape punch implements these registers:

=======  ====   ===========================
Name     Size   Comments
=======  ====   ===========================
``BUF``  8      Last data item processed
``POS``  32     Position in the output file
=======  ====   ===========================

Console typewriter input (TTI), output (TTO)
""""""""""""""""""""""""""""""""""""""""""""
The Typewriter is a half-duplex electric Friden Flexowriter typewriter.
The typewriter input (TTI) polls the console keyboard for input.
The typewriter output (TTO) writes to the simulator console window.
On input, TTI converts the ASCII character received from the keyboard to Flexowriter code.
On output, the TTO converts the Flexowriter code to ASCII for display on the simulator console window.

The typewriter input implements these registers:

========  ====  ====================================
Name      Size  Comments
========  ====  ====================================
``BUF``   6     Typewriter buffer (shared)
``UC``    1     Uppercase/lowercase state (shared)
``DONE``  1     Input ready flag
``POS``   32    Number of characters input
``TIME``  24    Keyboard polling interval
========  ====  ====================================

The typewriter output implements these registers:

========  ====  ====================================
Name      Size  Comments
========  ====  ====================================
``BUF``   6     Typewriter buffer (shared)
``UC``    1     Uppercase/lowercase state (shared)
``RPLS``  1     Return restart pulse flag
``DONE``  1     Output done flag
``POS``   32    Number of characters output
``TIME``  24    Time from I/O initiation to interrupt
========  ====  ====================================

Display (DPY)
"""""""""""""
In 1957, a 10-inch, electro-static deflection,
cathode ray tube, having 512 by 512 addressable locations,
in a 7×7-inch raster, point-by-point display system was installed on the TX-0.
In simulation, the display is accomplished using a separate graphical display window.

The display is accessed via the ``DIS`` order,
with the coordinate specified in the ``AC``.
``DIS`` will intensify a point with *X* and *Y* coordinates,
where *X* is specified by ``AC`` digits 0-8,
and *Y* is specified by ``AC`` digits 9-17.
Bit 0 is the sign for *X*,
and bit 9 is the sign for *Y*.
The complement system is in effect when signs are negative.

Light pen (PEN)
"""""""""""""""
In 1958, a Light-Pen,
a solid state version of an idea being developed for the Sage System, was added to the TX-0.
The light-pen status is read using the ``PEN`` order,
with light-pen flip-flops 1 and 2 being read into ``AC`` positions 0 and 1 respectively.

In simulation,
the light-pen is implemented using a computer mouse or touch screen.

TX-0 usage examples
===================
Several example tapes can be used to test the TX-0 simulation.

Tic-tac-toe game
----------------
The tic-tac-toe game can be run using the ``tic.simh`` startup script::

    ; TX-0 Initialization file for the tic-tac-toe game
    set dpy enable
    att petr bin_tic-tac-toe_new_code_12-16-61.bin
    boot petr
    g

In this game, you simply use the light-pen (mouse) to select where you want to place the "X" (the TX-0 is "O.").
You must click right on top of or very close to the dot in the center of the square you want to place your "X" into.

MOUSE game
----------
The MOUSE game,
written by John E. Ward is a more complex game with requires involvement not only with the light pen,
but also the ``TAC``.
In simulation,
the ``TAC`` can be accessed by stopping the simulation and using the ``Deposit`` command to change the value of the ``TAC`` to select various modes.

Here is the configuration file, ``mouse.simh``::

    ; TX-0 Initialization file for the Mouse Maze Game
    set dpy enable
    att petr bin_newMouse_3-22-66.bin

    ; The mouse maze game mode is manipulated under TAC control.
    ; 400014 = Erase Wall mode.
    ; 400024 = Write Wall mode.
    ; 400011 = Erase Cheese mode.
    ; 400021 = Write Cheese mode.
    ; 400012 = Erase Mouse mode.
    ; 400022 = Write Mouse mode.
    ; 400002 = "do mouse" (start the mouse searching.)
    ; 400017 = "do over"

    ; Start in "Erase Wall" mode
    d tac 400014
    boot petr

When this game starts, it is in "Erase Wall" mode.
Use this mode to create the maze, by using the computer mouse to erase unwanted walls from the grid.
When done, press Control-E to get to the simulator command prompt,
and change to "Write Cheese" mode::

    C:\>TX-0.exe mouse.simh

    TX-0 simulator V4.0-0
    mouse.simh-16> boot petr

    Simulation stopped, PC: 000161 (trn 00275 (Transfer Negative))
    sim> d tac 400021
    sim> g

Next place the cheese with the computer mouse,
and then repeat the procedure above to change the ``TAC`` to "do mouse" mode.
The TX-0 should then solve the maze.
Sometimes the mouse gets stuck and is not able to find the cheese.
I tend to believe this is a bug in the simulation,
but I have not been able to find it yet.
