.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. |SET| replace:: :html:`<samp><b>SET</b></samp>`
.. |MPX| replace:: :html:`<samp>MPX</samp>`
.. |STK| replace:: :html:`<samp>STK</samp>`
.. |n|   replace:: :html:`<samp><var>n</var></samp>`
.. |#|   replace:: :html:`<samp><var>#</var></samp>`
.. |##|  replace:: :html:`<samp><var>##</var></samp>`

*******************************
Dec PDP-10/KA10 simulator usage
*******************************
:Date: 2022-10-22
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
Originally the DEC PDP-10 computer started as the PDP-6.
This was a 36-bit computer that was designed for timesharing,
which was introduced in 1964.
The original goal of the machine was to allow for processing of many 6-bit characters at a time.
36 bits were also common in machines like the IBM 7090, GE 645, and Univac 11xx lines.
Several systems influenced the design of the PDP-6,
like CTSS, Lisp, support for larger memory.
The PDP-6 was canceled by DEC due to production problems.
The engineers designed a smaller replacement,
which happened to be a 36-bit computer that looked very much like the PDP-6.
This was called the PDP-10, later renamed to "DECSystem-10".
The system supported up to 256K words of memory.

The first PDP-10 was labeled KA10,
and added a few instructions to the PDP-6.
Both the PDP-6 and PDP-10 used a base and limit relocation scheme.
The KA10 generally offered two registers,
one of user data and the second for user shared code.
These were referred to the Low-Segment and High-Segment,
the High-Segment could be shared with several jobs.
The next version was called KI10 for Integrated.
This added support for paging and double precision floating-point instructions.
It also added 4 sets of registers to improve context switching time.
It could also support up to 4 megawords of memory.
Following the KI10 was the KL10 (for Low-Cost).
The KL10 added double precision integer instructions and instructions to improve COBOL performance.
This was the first version which was microcoded.
The KL10 was extended to support user programs larger then 256k.
The final version to make it to market was the KS10 (for Small),
this was a bit-slice version of the PDP-10 which used UNIBUS devices which were cheaper then the KL10 devices.

The original operating system for the PDP-6/PDP-10 was just called "Monitor".
It was designed to fit into 6K words.
Around the third release, swapping was added.
The sixth release saw the addition of virtual memory.
Around the fourth release, it was given the name "TOPS-10"
Around this time, BBN was working on a paging system and implemented it on the PDP-10.
This was called "Tenex".
This was later adopted by DEC and became "Tops-20".

During the mid-60s,
a group at MIT,
who where not happy with how Multics was being developed,
decided to create their own operating system which they called Incompatible Timesharing System ("ITS"),
which was a play on the original project called Compatible Timesharing System ("CTSS").
CTSS was implemented by MIT on their IBM 7090 as an experimental system that allowed multiple time sharing users to co-exist on the same machine running batch processing.
(hence the term "compatable").

Also during the mid-60s,
a group at Stanford Artificial Intelligence Laboratory (SAIL),
started with a version of TOPS-10 and heavily modified it to become WAITS.

During the 70s,
Tymshare starting with DEC TOPS-10 system,
modified it to support random access files,
paging with working sets and spawn-able processes.
This ran on the KI10, KL10 and KS10.

The PDP-10 was ultimately replaced by the VAX.

Simulator files
===============
To compile the DEC 10/KA10 simulator,
you must define ``USE_INT64`` as part of the compilation command-line.

+--------------+-------------------+------------------------------------------------+
| Subdirectory | File              | Contains                                       |
+==============+===================+================================================+
| ``PDP10/``   | ``kx10_defs.h``   | KA10 simulator definitions                     |
|              +-------------------+------------------------------------------------+
|              | ``kx10_cpu.c``    | KA10 CPU                                       |
|              +-------------------+------------------------------------------------+
|              | ``kx10_df.c``     | DF10/C controller                              |
|              +-------------------+------------------------------------------------+
|              | ``kx10_rh.c``     | RH10 controller                                |
|              +-------------------+------------------------------------------------+
|              | ``kx10_diskh``    | Disk formatter definitions                     |
|              +-------------------+------------------------------------------------+
|              | ``kx10_disk.c``   | Disk formatter routine                         |
|              +-------------------+------------------------------------------------+
|              | ``kx10_sys.c``    | KA10 system interface                          |
|              +-------------------+------------------------------------------------+
|              | ``kx10_cty.c``    | Console terminal interface                     |
|              +-------------------+------------------------------------------------+
|              | ``kx10_pt.c``     | Paper tape reader and punch                    |
|              +-------------------+------------------------------------------------+
|              | ``kx10_rc.c``     | RC10 disk controller, RD10 and RM10 disks      |
|              +-------------------+------------------------------------------------+
|              | ``kx10_dp.c``     | RP10 disk controller, RP01/2/3 disks           |
|              +-------------------+------------------------------------------------+
|              | ``kx10_rp.c``     | RH10 disk controller, RP04/5/6 disks           |
|              +-------------------+------------------------------------------------+
|              | ``kx10_rs.c``     | RH10 disk controller, RS04 fixed-head disks    |
|              +-------------------+------------------------------------------------+
|              | ``kx10_dt.c``     | TD10 DECtape controller                        |
|              +-------------------+------------------------------------------------+
|              | ``kx10_mt.c``     | TM10A/B magnetic tape controller               |
|              +-------------------+------------------------------------------------+
|              | ``kx10_tu.c``     | RH10/TM03 magnetic tape controller             |
|              +-------------------+------------------------------------------------+
|              | ``kx10_lp.c``     | Line printer                                   |
|              +-------------------+------------------------------------------------+
|              | ``kx10_cr.c``     | Punch card reader                              |
|              +-------------------+------------------------------------------------+
|              | ``kx10_cp.c``     | Punch card punch                               |
|              +-------------------+------------------------------------------------+
|              | ``kx10_dk.c``     | DK10 interval timer                            |
|              +-------------------+------------------------------------------------+
|              | ``kx10_dc.c``     | DC10 communications controller                 |
|              +-------------------+------------------------------------------------+
|              | ``kx10_ddc.c``    | DDC-10 drum controller                         |
|              +-------------------+------------------------------------------------+
|              | ``ka10_tk10.c``   | Knight kludge, TTY scanner                     |
|              +-------------------+------------------------------------------------+
|              | ``ka10_mty.c``    | MTY, Morton terminal multiplexer               |
|              +-------------------+------------------------------------------------+
|              | ``ka10_dpk.c``    | DK-10 datapoint kludge                         |
|              +-------------------+------------------------------------------------+
|              | ``ka10_dpy.c``    | DEC 340 graphics interface                     |
|              +-------------------+------------------------------------------------+
|              | ``ka10_dkb.c``    | Stanford microswitch scanner for III display   |
|              +-------------------+------------------------------------------------+
|              | ``ka10_iii.c``    | Stanford triple III display                    |
|              +-------------------+------------------------------------------------+
|              | ``ka10_stk.c``    | Stanford keyboard for 340                      |
|              +-------------------+------------------------------------------------+
|              | ``ka10_pclk.c``   | Petit calendar clock                           |
|              +-------------------+------------------------------------------------+
|              | ``ka10_pd.c``     | DeCoriolis clock                               |
|              +-------------------+------------------------------------------------+
|              | ``ka10_imx.c``    | Analog input device                            |
|              +-------------------+------------------------------------------------+
|              | ``ka10_ten11.c``  | PDP-11 interface                               |
|              +-------------------+------------------------------------------------+
|              | ``kx10_lights.c`` | Front panel interface                          |
|              +-------------------+------------------------------------------------+
|              | ``kx10_imp.c``    | IMP10 interface to ethernet                    |
|              +-------------------+------------------------------------------------+
|              | ``ka10_ch10.c``   | Chaosnet 10 interface                          |
|              +-------------------+------------------------------------------------+
|              | ``ka10_pmp.c``    | PMP interface to IBM 3330 drives               |
|              +-------------------+------------------------------------------------+
|              | ``ka10_ai.c``     | System Concepts DC-10 IBM 2314 disk controller |
|              +-------------------+------------------------------------------------+
|              | ``pdp6_dct.c``    | PDP-6 data controller                          |
|              +-------------------+------------------------------------------------+
|              | ``pdp6_dtc.c``    | PDP-6 551 DECtape controller                   |
|              +-------------------+------------------------------------------------+
|              | ``pdp6_mtc.c``    | PDP-6 556 magtape controller                   |
|              +-------------------+------------------------------------------------+
|              | ``pdp6_dsk.c``    | PDP-6 270 disk controller                      |
|              +-------------------+------------------------------------------------+
|              | ``pdp6_dcs.c``    | PDP-7 630 terminal multiplexer                 |
+--------------+-------------------+------------------------------------------------+

KA10 features
=============
The KA10 simulator is configured as follows:

.. |TEN11|  replace:: :html:`<samp>TEN11</samp>`
.. |AUXCPU| replace:: :html:`<samp>AUXCPU</samp>`

+----------------+------------------------------------------------+
| Device name(s) | Simulates                                      |
+================+================================================+
| |CPU|_         | KA10 CPU with 256KW of memory                  |
+----------------+------------------------------------------------+
| |CTY|_         | Console TTY                                    |
+----------------+------------------------------------------------+
| |PTP|_         | Paper tape punch                               |
+----------------+------------------------------------------------+
| |PTR|_         | Paper tape reader                              |
+----------------+------------------------------------------------+
| |LPT|_         | LP10 line printer                              |
+----------------+------------------------------------------------+
| |CR|_          | CR10 card reader                               |
+----------------+------------------------------------------------+
| |CP|_          | CP10 card punch                                |
+----------------+------------------------------------------------+
| |MTA|_         | TM10A/B tape controller                        |
+----------------+------------------------------------------------+
| |DPA|_         | RP10 disk controller                           |
+----------------+                                                |
| |DPB|_         |                                                |
+----------------+------------------------------------------------+
| |FSA|_         | RS04 disk controller via RH10                  |
+----------------+------------------------------------------------+
| |RPA|_         | RH10 disk controllers via RH10                 |
+----------------+                                                |
| |RPB|_         |                                                |
+----------------+                                                |
| |RPC|_         |                                                |
+----------------+                                                |
| |RPD|_         |                                                |
+----------------+------------------------------------------------+
| |PMP|_         | |PMP| IBM 3330 disk controller                 |
+----------------+------------------------------------------------+
| |AI|_          | System concepts DC-10 IBM 2314 disk controller |
+----------------+------------------------------------------------+
| |TUA|_         | TM02 tape controller via RH10                  |
+----------------+------------------------------------------------+
| |FHA|_         | RC10 disk controller                           |
+----------------+------------------------------------------------+
| |DDC|_         | DDC10 disk controller                          |
+----------------+------------------------------------------------+
| |DT|_          | TD10 DECtape controller                        |
+----------------+------------------------------------------------+
| |DC|_          | DC10 communications controller                 |
+----------------+------------------------------------------------+
| |DK|_          | Clock timer module                             |
+----------------+------------------------------------------------+
| |PCLK|_        | Petit calendar clock                           |
+----------------+------------------------------------------------+
| |PD|_          | Coriolis clock                                 |
+----------------+------------------------------------------------+
| |IMP|_         | IMP network interface                          |
+----------------+------------------------------------------------+
| |CH|_          | CH10 Chaosnet interface                        |
+----------------+------------------------------------------------+
| |IMX|_         | A/D input multiplexer                          |
+----------------+------------------------------------------------+
| |TK|_          | Knight kludge, TTY scanner                     |
+----------------+------------------------------------------------+
| |MTY|_         | MTY, Morton terminal multiplexer               |
+----------------+------------------------------------------------+
| |DPK|_         | DK-10 datapoint kludge                         |
+----------------+------------------------------------------------+
| |DKB|_         | Stanford microswitch scanner                   |
+----------------+------------------------------------------------+
| |III|_         | Stanford triple III display                    |
+----------------+------------------------------------------------+
| |TEN11|        | PDP-11 interface                               |
+----------------+------------------------------------------------+
| |AUXCPU|       | PDP-6 interface                                |
+----------------+------------------------------------------------+
| |DCT|_         | PDP-6 data control type 136                    |
+----------------+------------------------------------------------+
| |DTC|_         | PDP-6 type 551 DECtape controller              |
+----------------+------------------------------------------------+
| |MTC|_         | PDP-6 type 516 magtape controller              |
+----------------+------------------------------------------------+
| |DSK|_         | PDP-6 type 270 disk controller                 |
+----------------+------------------------------------------------+
| |DCS|_         | PDP-6 type 630 terminal multiplexer            |
+----------------+------------------------------------------------+

.. |CPU| replace:: :html:`<samp>CPU</samp>`

|CPU|
-----
The CPU options include setting memory size and O/S customization.

=============================================  ===========================================  =========
:html:`<samp><b>SET CPU 16K</b></samp>`        Set memory to 16K
:html:`<samp><b>SET CPU 32K</b></samp>`        Set memory to 32K
:html:`<samp><b>SET CPU 48K</b></samp>`        Set memory to 48K
:html:`<samp><b>SET CPU 64K</b></samp>`        Set memory to 64K
:html:`<samp><b>SET CPU 96K</b></samp>`        Set memory to 96K
:html:`<samp><b>SET CPU 128K</b></samp>`       Set memory to 128K
:html:`<samp><b>SET CPU 256K</b></samp>`       Set memory to 256K
:html:`<samp><b>SET CPU 512K </b></samp>`      Set memory to 512K                           ITS & BBN
:html:`<samp><b>SET CPU 768K</b></samp>`       Set memory to 768K                           ITS & BBN
:html:`<samp><b>SET CPU 1024K</b></samp>`      Set memory to 1024K                          ITS & BBN
:html:`<samp><b>SET CPU NOMAOFF</b></samp>`    Set traps to default of 040
:html:`<samp><b>SET CPU MAOFF</b></samp>`      Move trap vectors from 040 to 0140           WAITS
:html:`<samp><b>SET CPU ONESEG</b></samp>`     Set to one segment register
:html:`<samp><b>SET CPU TWOSEG</b></samp>`     Set to two segment registers
:html:`<samp><b>SET CPU ITS</b></samp>`        Add ITS pager and instruction support to KA  ITS
:html:`<samp><b>SET CPU NOMPX </b></samp>`     Disable |MPX| interrupt support for ITS
:html:`<samp><b>SET CPU MPX</b></samp>`        Enable |MPX| interrupt support for ITS       ITS
:html:`<samp><b>SET CPU WAITS</b></samp>`      Add support for special WAITS instructions   WAITS
:html:`<samp><b>SET CPU NOWAITS</b></samp>`    Disable special WAITS instructions
:html:`<samp><b>SET CPU BBN</b></samp>`        Enable BBN pager and Tenex support           TENEX
:html:`<samp><b>SET CPU NOIDLE</b></samp>`     Disable idle detection
:html:`<samp><b>SET CPU IDLE</b></samp>`       Enable idle detection
=============================================  ===========================================  =========

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

.. |FM0|         replace:: :html:`<samp class="register">FM0</samp>`
.. |FM17|        replace:: :html:`<samp class="register">FM17</samp>`
.. |SW|          replace:: :html:`<samp class="register">SW</samp>`
.. |MI|          replace:: :html:`<samp class="register">MI</samp>`
.. |AS|          replace:: :html:`<samp class="register">AS</samp>`
.. |ABRK|        replace:: :html:`<samp class="register">ABRK</samp>`
.. |ACOND|       replace:: :html:`<samp class="register">ACOND</samp>`
.. |PIR|         replace:: :html:`<samp class="register">PIR</samp>`
.. |PIH|         replace:: :html:`<samp class="register">PIH</samp>`
.. |PIE|         replace:: :html:`<samp class="register">PIE</samp>`
.. |PIENB|       replace:: :html:`<samp class="register">PIENB</samp>`
.. |BYF5|        replace:: :html:`<samp class="register">BYF5</samp>`
.. |UUO|         replace:: :html:`<samp class="register">UUO</samp>`
.. |PL|          replace:: :html:`<samp class="register">PL</samp>`
.. |PH|          replace:: :html:`<samp class="register">PH</samp>`
.. |RL|          replace:: :html:`<samp class="register">RL</samp>`
.. |RH|          replace:: :html:`<samp class="register">RH</samp>`
.. |PFLAG|       replace:: :html:`<samp class="register">PFLAG</samp>`
.. |PUSHOVER|    replace:: :html:`<samp class="register">PUSHOVER</samp>`
.. |MEMPROT|     replace:: :html:`<samp class="register">MEMPROT</samp>`
.. |NXM|         replace:: :html:`<samp class="register">NXM</samp>`
.. |CLK|         replace:: :html:`<samp class="register">CLK</samp>`
.. |OV|          replace:: :html:`<samp class="register">OV</samp>`
.. |FOV|         replace:: :html:`<samp class="register">FOV</samp>`
.. |APRIRQ|      replace:: :html:`<samp class="register">APRIRQ</samp>`
.. |PAGE_ENABLE| replace:: :html:`<samp class="register">PAGE_ENABLE</samp>`
.. |PAGE_FAULT|  replace:: :html:`<samp class="register">PAGE_FAULT</samp>`
.. |AC_STACK|    replace:: :html:`<samp class="register">AC_STACK</samp>`
.. |PAGE_RELOAD| replace:: :html:`<samp class="register">PAGE_RELOAD</samp>`
.. |FAULT_DATA|  replace:: :html:`<samp class="register">FAULT_DATA</samp>`
.. |TRP_FLG|     replace:: :html:`<samp class="register">TRP_FLG</samp>`
.. |LAST_PAGE|   replace:: :html:`<samp class="register">LAST_PAGE</samp>`
.. |EXEC_MAP|    replace:: :html:`<samp class="register">EXEC_MAP</samp>`
.. |NXT_WR|      replace:: :html:`<samp class="register">NXT_WR</samp>`
.. |MON_BASE|    replace:: :html:`<samp class="register">MON_BASE</samp>`
.. |USER_BASE|   replace:: :html:`<samp class="register">USER_BASE</samp>`
.. |USER_LIMIT|  replace:: :html:`<samp class="register">USER_LIMIT</samp>`
.. |PER_USER|    replace:: :html:`<samp class="register">PER_USER</samp>`
.. |DBR1|        replace:: :html:`<samp class="register">DBR1</samp>`
.. |DBR2|        replace:: :html:`<samp class="register">DBR2</samp>`
.. |DBR3|        replace:: :html:`<samp class="register">DBR3</samp>`
.. |JPC|         replace:: :html:`<samp class="register">JPC</samp>`
.. |AGE|         replace:: :html:`<samp class="register">AGE</samp>`
.. |FAULT_ADDR|  replace:: :html:`<samp class="register">FAULT_ADDR</samp>`
.. |OPC|         replace:: :html:`<samp class="register">OPC</samp>`
.. |MAR|         replace:: :html:`<samp class="register">MAR</samp>`
.. |QUA_TIME|    replace:: :html:`<samp class="register">QUA_TIME</samp>`

================  ====  =============================  =======
Name              Size  Comments                       OS type
================  ====  =============================  =======
|PC|              18    Program counter
|FLAGS|           18    Flags
|FM0|\ –\ |FM17|  36    Fast memory
|SW|              36    Console SW register
|MI|              36    Memory indicators
|AS|              18    Console AS register
|ABRK|            1     Address break
|ACOND|           5     Address condition switches
|PIR|             8     Priority interrupt request
|PIH|             8     Priority interrupt hold
|PIE|             8     Priority interrupt enable
|PIENB|           1     Enable priority system
|BYF5|            1     Byte flag
|UUO|             1     UUO cycle
|PL|              18    Program limit low
|PH|              18    Program limit high
|RL|              18    Program relation low
|RH|              18    Program relation high
|PFLAG|           1     Relocation enable
|PUSHOVER|        1     Push overflow flag
|MEMPROT|         1     Memory protection flag
|NXM|             1     Non-existing memory access
|CLK|             1     Clock interrupt
|OV|              1     Overflow enable
|FOV|             1     Floating overflow enable
|APRIRQ|          3     APR interrupt number
|PAGE_ENABLE|     1     Paging enabled                 ITS/BBN
|PAGE_FAULT|      1     Page fault                     ITS/BBN
|AC_STACK|        18    AC stack                       ITS/BBN
|PAGE_RELOAD|     18    Page reload                    ITS/BBN
|FAULT_DATA|      36    Page fault data                ITS/BBN
|TRP_FLG|         1     Trap flag                      ITS/BBN
|LAST_PAGE|       9     Last page                      ITS/BBN
|EXEC_MAP|        8     Executive mapping              BBN
|NXT_WR|          1     Map next write                 BBN
|MON_BASE|        8     Monitor base                   BBN
|USER_BASE|       8     User base                      BBN
|USER_LIMIT|      3     User limit                     BBN
|PER_USER|        36    Per user data                  BBN
|DBR1|            18    Paging control register 1      ITS
|DBR2|            18    Paging control register 2      ITS
|DBR3|            18    Paging control register 3      ITS
|JPC|             18    Last jump instruction address  ITS
|AGE|             4     Age                            ITS
|FAULT_ADDR|      18    Fault address                  ITS
|OPC|             36    Saved PC and flags             ITS
|MAR|             18    Memory access register         ITS
|QUA_TIME|        36    Quantum time                   ITS
================  ====  =============================  =======

The CPU can maintain a history of the most recently executed instructions.

This is controlled by the :html:`<samp><b>SET CPU HISTORY</b></samp>` and :html:`<samp><b>SHOW CPU HISTORY</b></samp>` commands:

=========================================================  ======================================
:html:`<samp><b>SET CPU HISTORY</b></samp>`                Clear history buffer
:html:`<samp><b>SET CPU HISTORY=</b>0 </samp>`             Disable history
:html:`<samp><b>SET CPU HISTORY=</b><var>n</var></samp>`   Enable history, length = |n|
:html:`<samp><b>SHOW CPU HISTORY</b></samp>`               Print CPU history
:html:`<samp><b>SHOW CPU HISTORY=</b><var>n</var></samp>`  Print first |n| entries of CPU history
=========================================================  ======================================

Instruction tracing shows the program counter,
the contents of |AC| selected,
and the computed effective address.
|AR| is generally the contents the memory location referenced by |EA|.
|RES| is the result of the instruction.
|FLAGS| shows the flags after the instruction is executed.
|IR| shows the instruction being executed.

Unit record I/O devices
-----------------------
.. _Console teletype (CTY):
.. _Console teletype:
.. _device 120:
.. _CTY:
.. |CTY| replace:: :html:`<samp>CTY</samp>`

Console teletype (|CTY|) (device 120)
"""""""""""""""""""""""""""""""""""""
The console station allows for communications with the operating system.

======================================  ================================
:html:`<samp><b>SET CTY 7B</b></samp>`  7-bit characters, space parity
:html:`<samp><b>SET CTY 8B</b></samp>`  8-bit characters, space parity
:html:`<samp><b>SET CTY 7P</b></samp>`  7-bit characters, space parity
:html:`<samp><b>SET CTY UC</b></samp>`  Translate lowercase to uppercase
======================================  ================================

The CTY also supports a method for stopping the TOPS10 operating system.

========================================  ========================
:html:`<samp><b>SET CTY STOP</b></samp>`  Deposit 1 in location 30
========================================  ========================

.. _Paper tape reader (PTR):
.. _Paper tape reader:
.. _device 104:
.. _PTR:
.. |PTR| replace:: :html:`<samp>PTR</samp>`

Paper tape reader (|PTR|) (device 104)
""""""""""""""""""""""""""""""""""""""
Reads a paper tape from a disk file.

.. _Paper tape punch (PTP):
.. _Paper tape punch:
.. _device 100:
.. _PTP:
.. |PTP| replace:: :html:`<samp>PTP</samp>`

Paper tape punch (|PTP|) (device 100)
"""""""""""""""""""""""""""""""""""""
Punches a paper tape to a disk file.

.. _Card reader (CR):
.. _Card reader:
.. _device 150:
.. _CR:
.. |CR| replace:: :html:`<samp>CR</samp>`

Card reader (|CR|) (device 150)
"""""""""""""""""""""""""""""""
The card reader (|CR|) reads data from a disk file.
Card reader files can either be text (one character per column) or column binary (two characters per column).
The file type can be specified with a |SET| command:

===================================================================  =================================
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=TEXT</b></samp>`    Set ASCII text mode
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=BINARY</b></samp>`  Set for binary card images
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=BCD</b></samp>`     Set for BCD records
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=CBN</b></samp>`     Set for column binary BCD records
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=AUTO</b></samp>`    Automatically determine format
===================================================================  =================================

or in the |ATTACH| command:

=============================================================================================  ==================================================================
:html:`<samp><b>ATTACH CR</b><var>n</var> <var>file</var></samp>`                              Attach a file
:html:`<samp><b>ATTACH CR</b><var>n</var> <b>-f</b> <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp><b>ATTACH CR</b><var>n</var> <b>-s</b> <var>file</var></samp>`                    Add file onto current cards to read
:html:`<samp><b>ATTACH CR</b><var>n</var> <b>-e</b> <var>file</var></samp>`                    After file is read in, the reader will receive an end-of-file flag
=============================================================================================  ==================================================================

.. _Card punch (CP):
.. _Card punch:
.. _device 110:
.. _CP:
.. |CP| replace:: :html:`<samp>CP</samp>`

Card punch (|CP|) (device 110)
""""""""""""""""""""""""""""""
The card reader (|CP|) writes data to a disk file.
Cards are simulated as ASCII lines with terminating newlines.
Card punch files can either be text (one character per column) or column binary (two characters per column).
The file type can be specified with a |SET| command:

================================================  =================================
:html:`<samp><b>SET CP FORMAT=TEXT</b></samp>`    Set ASCII text mode
:html:`<samp><b>SET CP FORMAT=BINARY</b></samp>`  Set for binary card images
:html:`<samp><b>SET CP FORMAT=BCD</b></samp>`     Set for BCD records
:html:`<samp><b>SET CP FORMAT=CBN</b></samp>`     Set for column binary BCD records
:html:`<samp><b>SET CP FORMAT=AUTO</b></samp>`    Automatically determine format
================================================  =================================

or in the |ATTACH| command:

==========================================================================  ===================================
:html:`<samp><b>ATTACH CP</b> <var>file</var></samp>`                       Attach a file
:html:`<samp><b>ATTACH CP -f</b> <var>format</var> <var>file</var></samp>`  Attach a file with the given format
==========================================================================  ===================================

.. _Line printer (LPT):
.. _Line printer:
.. _device 124:
.. _LPT:
.. |LPT| replace:: :html:`<samp>LPT</samp>`

Line printer (|LPT|) (device 124)
"""""""""""""""""""""""""""""""""
The line printer (|LPT|) writes data to a disk file as ASCII text with terminating newlines.
Currently set to handle standard signals to control paper advance.

================================================================================  ==============================================================
:html:`<samp><b>SET LPT</b><var>n</var> <b>LC</b></samp>`                         Allow printer to print lowercase
:html:`<samp><b>SET LPT</b><var>n</var> <b>UC</b></samp>`                         Print only uppercase
:html:`<samp><b>SET LPT</b><var>n</var> <b>UTF8</b></samp>`                       Print control characters as UTF8 characters
:html:`<samp><b>SET LPT</b><var>n</var> <b>LINESPERPAGE=</b><var>n</var></samp>`  Set the number of lines before an auto form feed is generated.
                                                                                  There is an automatic margin of 6 lines.
                                                                                  There is a maximum of 100 lines per page.
:html:`<samp><b>SET LPT</b><var>n</var> <b>DEV=</b><var>n</var></samp>`           Set device number to |n|\ (octal)
================================================================================  ==============================================================

These characters control the skipping of various number of lines.

=========  =========================================
Character  Effect
=========  =========================================
014        Skip to top of form
013        Skip mod 20 lines
020        Skip mod 30 lines
021        Skip to even line
022        Skip to every 3 line
023        Same as line feed (12), but ignore margin
=========  =========================================

.. |DPY| replace:: :html:`<samp>DPY</samp>`
.. _DPY:
.. _Type 340 graphics display:
.. _device 130:

Type 340 graphics display (|DPY|) (device 130)
""""""""""""""""""""""""""""""""""""""""""""""
Primarily useful under ITS,
TOPS-10 does have some limited support for this device.
By default,
this device is not enabled.
When enabled and commands are sent to it,
a graphics windows will display.

.. |III| replace:: :html:`<samp>III</samp>`
.. _III:
.. _Stanford triple III display:
.. _Stanford triple III display (III):
.. _device 430:

Stanford triple III display (|III|) (device 430)
""""""""""""""""""""""""""""""""""""""""""""""""
Primarily useful under WAITS.
By default,
this device is not enabled.
Used |DKB|_ device for keyboard input.

.. |DKB| replace:: :html:`<samp>DKB</samp>`
.. _DKB:
.. _DKB Stanford microswitch keyboard scanner:
.. _DKB Stanford microswitch keyboard scanner (DKB):
.. _device 310:

DKB Stanford microswitch keyboard scanner (|DKB|) (device 310)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Used by |III|_ to handle input.
Currently only one keyboard is supported.

.. |DK|   replace:: :html:`<samp>DK</samp>`
.. |DK10| replace:: :html:`<samp>DK10</samp>`
.. _DK:
.. _DK10:
.. _DK10 timer module:
.. _DK10 timer module (DK):
.. _device 070:

|DK10| timer module (|DK|) (device 070)
"""""""""""""""""""""""""""""""""""""""
The |DK10| timer module does not have any settable options.

.. |MTA|   replace:: :html:`<samp>MTA</samp>`
.. |TM10|  replace:: :html:`<samp>TM10</samp>`
.. |TM10A| replace:: :html:`<samp>TM10A</samp>`
.. |TM10B| replace:: :html:`<samp>TM10B</samp>`
.. _MTA:
.. _TM10 magnetic tape:
.. _TM10 magnetic tape (MTA):
.. _device 340:
.. _device 344:

TM10 magnetic tape (|MTA|) (device 340/344)
"""""""""""""""""""""""""""""""""""""""""""
The |TM10| was the most common tape controller on KA10 and KI10.
The |TM10| came in two models,
the |TM10A| and the |TM10B|.
The B model added a DF10 which allowed the tape data to be transferred without intervention of the CPU.
The device has 2 options:

=====================================================  ================================================
:html:`<samp><b>SET MTA TYPE=</b><var>t</var></samp>`  Set the type of the controller to ``A`` or ``B``
:html:`<samp><b>SET MTA MPX=</b><var>#</var></samp>`   For ITS, set the MPX interrupt to channel |#|
=====================================================  ================================================

Each individual tape drive support several options:
|MTA| used as an example.

==================================================================  =======================================
:html:`<samp><b>SET MTA</b><var>n</var> <b>7T</b></samp>`           Set the mag tape unit to 7-track format
:html:`<samp><b>SET MTA</b><var>n</var> <b>9T</b></samp>`           Set the mag tape unit to 9-track format
:html:`<samp><b>SET MTA</b><var>n</var> <b>LOCKED</b></samp>`       Set the mag tape to be read-only
:html:`<samp><b>SET MTA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set the mag tape to be writable
==================================================================  =======================================

.. |DT|   replace:: :html:`<samp>DT</samp>`
.. |TD10| replace:: :html:`<samp>TD10</samp>`
.. _DT:
.. _TD10 DECTape:
.. _device 320:
.. _device 324:

TD10 DECTape (|DT|) (device 320/324)
""""""""""""""""""""""""""""""""""""
The |TD10| was the standard DECTape controller for KA and KIs.
This controller loads the tape into memory and uses the buffered version.
For ITS,
you needed to connect it to an |MPX| channel to handle I/O.

===================================================  ===============================================
:html:`<samp><b>SET DT MPX=</b><var>#</var></samp>`  For ITS, set the |MPX| interrupt to channel |#|
===================================================  ===============================================

Each individual tape drive support several options:
|DT| used as an example.

=================================================================  ================================================
:html:`<samp><b>SET DT</b><var>n</var> <b>LOCKED</b></samp>`       Set the mag tape to be read-only
:html:`<samp><b>SET DT</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set the mag tape to be writable
:html:`<samp><b>SET DT</b><var>n</var> <b>18b</b></samp>`          Default, tapes are considered to be 18-bit tapes
:html:`<samp><b>SET DT</b><var>n</var> <b>16b</b></samp>`          Tapes are converted from 16-bit to 18-bit
:html:`<samp><b>SET DT</b><var>n</var> <b>12b</b></samp>`          Tapes are converted from 12-bit to 18-bit
=================================================================  ================================================

.. |PCLK| replace:: :html:`<samp>PCLK</samp>`
.. _PCLK:
.. _PCLK Petit calendar clock:
.. _device 730:

|PCLK| Petit calendar clock (device 730)
""""""""""""""""""""""""""""""""""""""""
This device keeps track of time and date for WAITS.
It was custom designed by Peter Petit.
The device supports two settings: ``ON`` and ``OFF``.
By default,
this device is disabled.

.. |PD| replace:: :html:`<samp>PD</samp>`
.. _PD:
.. _PD DeCoriolis clock:
.. _device 500:

|PD| DeCoriolis clock (device 500)
""""""""""""""""""""""""""""""""""
This is a device which keeps track of the time and date.
An access will return the number of ticks since the beginning of the year.
There are 60 ticks per second.
The device was made by Paul DeCoriolis at MIT.
By default, this device is disabled.

.. |IMX| replace:: :html:`<samp>IMX</samp>`
.. _IMX:
.. _A/D input multiplexer:
.. _A/D input multiplexer (IMX):
.. _device 574:

A/D input multiplexer (|IMX|) (device 574)
""""""""""""""""""""""""""""""""""""""""""
Provides 128 channels of 12-bit analog to digital converters.
Currently, only input from USB game controllers are supported.
To map game inputs to channels, use ::

    SET IMX CHANNEL=123;UNIT0;AXIS1

Where ``123`` is the A/D channel in octal notation,
``UNIT0`` is first USB game controller,
and ``AXIS1`` is the second axis on that controller.
Add ``;NEGATE`` to invert the signal.

Disk I/O devices
----------------
The PDP-10 supported many disk controllers.

.. |FHA| replace:: :html:`<samp>FHA</samp>`
.. _FHA:
.. _FHA disk controller:
.. _device 170:

|FHA| disk controller (device 170)
""""""""""""""""""""""""""""""""""
The RC10 disk controller used a DF10 to transfer data to memory.
There were two types of disks that could be connected to the RC10:
the RD10 and RM10.
The RD10 could hold up to 512K words of data.
The RM10 could hold up to 345K words of data.

Each individual disk drive support several options:
|FHA| used as an example.

==================================================================  =============================
:html:`<samp><b>SET FHA</b><var>n</var> <b>RD10</b></samp>`         Set this unit to be an RD10
:html:`<samp><b>SET FHA</b><var>n</var> <b>RM10</b></samp>`         Set this unit to be an RM10
:html:`<samp><b>SET FHA</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET FHA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
==================================================================  =============================

.. |DPA| replace:: :html:`<samp>DPA</samp>`
.. |DPB| replace:: :html:`<samp>DPB</samp>`
.. _DPA:
.. _DPB:
.. _DPA/DPB disk controller:
.. _device 250:
.. _device 254:

|DPA|\ /\ |DPB| disk controller (device 250/254)
""""""""""""""""""""""""""""""""""""""""""""""""
The RP10 disk controller used a DF10 to transfer data to memory.
There were three types of disks that could be connected to the RP10:
the RP01, RP02, and RP03.
The RP01 held up to 1.2M words,
the RP02 5.2M words,
and the RP03 10M words.
The second controller is |DPB|.
Disks can be stored in one of several file formats:
|SIMH|, |DBD9|, and |DLD9|.
The latter two are for compatibility with other simulators.

Each individual disk drive support several options:
|DPA| used as an example.

==================================================================  ==================================================
:html:`<samp><b>SET DPA</b><var>n</var> <b>RP01</b></samp>`         Set this unit to be an RP01
:html:`<samp><b>SET DPA</b><var>n</var> <b>RP02</b></samp>`         Set this unit to be an RP02
:html:`<samp><b>SET DPA</b><var>n</var> <b>RP03</b></samp>`         Set this unit to be an RP03
:html:`<samp><b>SET DPA</b><var>n</var> <b>HEADERS</b></samp>`      Enable the RP10 to execute write headers function
:html:`<samp><b>SET DPA</b><var>n</var> <b>NOHEADERS</b></samp>`    Prevent the RP10 to execute write headers function
:html:`<samp><b>SET DPA</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET DPA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
==================================================================  ==================================================

To attach a disk,
use the |ATTACH| command:

==============================================================================================  ===================================
:html:`<samp><b>ATTACH DPA</b><var>n</var> <var>file</var></samp>`                              Attach a file
:html:`<samp><b>ATTACH DPA</b><var>n</var> <b>-f</b> <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp><b>ATTACH DPA</b><var>n</var> <b>-n</b> <var>file</var></samp>`                    Create a blank disk
==============================================================================================  ===================================

.. |SIMH|   replace:: :html:`<samp>SIMH</samp>`
.. |DBD9|   replace:: :html:`<samp>DBD9</samp>`
.. |DLD9|   replace:: :html:`<samp>DLD9</samp>`
.. |format| replace:: :html:`<samp><var>format</var></samp>`

|format| can be |SIMH|, |DBD9|, or |DLD9|.

.. |PMP| replace:: :html:`<samp>PMP</samp>`
.. _PMP:
.. _PMP disk controller:
.. _device 504:

|PMP| disk controller (device 500/504)
""""""""""""""""""""""""""""""""""""""
The |PMP| disk controller allowed for IBM type drives to be connected to the PDP-10.
This is the controller used by WAITS.
This device is disabled by default. 

Each individual disk drive support several options:
|DPA| used as an example.

===========================================================================  ===================================================================================
:html:`<samp><b>SET PMP</b><var>n</var> <b>TYPE=</b><var>type</var></samp>`  Set this unit to be of type :html:`<samp><var>type</var></samp>` (generally 3330-2)
:html:`<samp><b>SET PMP</b><var>0</var> <b>DEV=</b><var>##</var></samp>`     Set the addresses of all disks to start at |##| hex
:html:`<samp><b>SET PMP</b><var>n</var> <b>DEV=</b><var>##</var></samp>`     Set this unit to be at address |##|
===========================================================================  ===================================================================================

.. |AI| replace:: :html:`<samp>AI</samp>`
.. _AI:
.. _System Concepts DC-10 disk controller:
.. _device 610:
.. _device 614:

System Concepts DC-10 disk controller (device 610/614)
""""""""""""""""""""""""""""""""""""""""""""""""""""""
System Concepts DC-10 disk controller allowed IBM 2314 compatible disk drives to be attached to the MIT AI Lab PDP-10 running ITS.
This device is disabled by default.

.. |DDC| replace:: :html:`<samp>DDC</samp>`
.. _DDC:

DDC10 drum controller for Tenex
"""""""""""""""""""""""""""""""
DEC RES-10 drum controller is used by Tenex for swapping.
This device is disabled by default.

Massbus devices
---------------
Massbus devices are attached via RH10s.
Devices start at device 270 and go up (274,360,364,370,374).
For Tops10,
devices need to be in the order |RS|_, |RP|_, |TU|_.
By default,
|RS|_ disks are disabled.
The first unit which is not enabled will get device 270,
units will be assigned the next available address automatically.
The device configuration must match that which is defined in the OS.

.. |RP|  replace:: :html:`<samp>RP</samp>`
.. |RPA| replace:: :html:`<samp>RPA</samp>`
.. |RPB| replace:: :html:`<samp>RPB</samp>`
.. |RPC| replace:: :html:`<samp>RPC</samp>`
.. |RPD| replace:: :html:`<samp>RPD</samp>`
.. _RP:
.. _RPA:
.. _RPB:
.. _RPC:
.. _RPD:

|RP| disk drives
""""""""""""""""
Up to 4 strings of up to 8 RP drives can be configured.
By default,
the third and fourth controllers are disabled.
These are addresses as |RPA|, |RPB|, |RPC|, and |RPD|.
Disks can be stored in one of several file formats:
|SIMH|, |DBD9|, and |DLD9|.
The latter two are for compatibility with other simulators.

==================================================================  ========================================
:html:`<samp><b>SET RPA</b><var>n</var> <b>RP04</b></samp>`         Set this unit to be an RP04 (19 MWords)
:html:`<samp><b>SET RPA</b><var>n</var> <b>RP06</b></samp>`         Set this unit to be an RP06 (39 MWords)
:html:`<samp><b>SET RPA</b><var>n</var> <b>RP07</b></samp>`         Set this unit to be an RP07 (110 MWords)
:html:`<samp><b>SET RPA</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET RPA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
==================================================================  ========================================

To attach a disk,
use the |ATTACH| command:

==============================================================================================  ===================================
:html:`<samp><b>ATTACH RPA</b><var>n</var> <var>file</var></samp>`                              Attach a file
:html:`<samp><b>ATTACH RPA</b><var>n</var> <b>-f</b> <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp><b>ATTACH RPA</b><var>n</var> <b>-n</b> <var>file</var></samp>`                    Create a blank disk
==============================================================================================  ===================================

|format| can be |SIMH|, |DBD9|, or |DLD9|.

.. |RS|  replace:: :html:`<samp>RS</samp>`
.. |FSA| replace:: :html:`<samp>FSA</samp>`
.. _RS:
.. _FSA:

|RS| disk drives
""""""""""""""""
One string of up to 8 RS drives can be configured.
These drives are fixed head swapping disks.
By default,
they are disabled.

==================================================================  ========================================
:html:`<samp><b>SET FSA</b><var>n</var> <b>RS03</b></samp>`         Set this unit to be an RS03 (262 KWords)
:html:`<samp><b>SET FSA</b><var>n</var> <b>RS04</b></samp>`         Set this unit to be an RS04 (262 KWords)
:html:`<samp><b>SET FSA</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET FSA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
==================================================================  ========================================

.. |TUA| replace:: :html:`<samp>TUA</samp>`
.. |TU|  replace:: :html:`<samp>TU</samp>`
.. _TU:
.. _TUA:

|TU| tape drives
""""""""""""""""
The |TUA| is a Mass bus tape controller using a TM03 formatter.
There can be one per RH10 and it can support up to 8 drives.

==================================================================  ================================
:html:`<samp><b>SET TUA</b><var>n</var> <b>LOCKED</b></samp>`       Set the mag tape to be read-only
:html:`<samp><b>SET TUA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set the mag tape to be writable
==================================================================  ================================

Terminal multiplexer I/O devices
--------------------------------
.. |ATTACH| replace:: :html:`<samp><b>ATTACH</b></samp>`
.. _ATTACH:

All terminal multiplexers must be attached in order to work.
The |ATTACH| command specifies the port to be used for Telnet sessions:

====================================================================  =====================
:html:`<samp><b>ATTACH</b> <var>device</var> <var>port</var></samp>`  Set up listening port
====================================================================  =====================

... where :html:`<samp><var>port</var></samp>` is a decimal number between 1 and 65535 that is not being used other TCP/IP activities.

Once attached and the simulator is running,
the multiplexer listens for connections on the specified port.
It assumes that any incoming connection is a Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a :html:`<samp><b>SET</b> <var>device</var> <b>DISCONNECT</b></samp>` command,
or a :html:`<samp><b>DETACH</b> <var>device</var></samp>` command.

================================================================================  ===================
:html:`<samp><b>SET</b> <var>device</var> <b>DISCONNECT=</b><var>n</var></samp>`  Disconnect line |n|
================================================================================  ===================

.. |device| replace:: :html:`<samp><var>device</var></samp>`
.. |SHOW|   replace:: :html:`<samp><b>SHOW</b></samp>`

The |device| implements the following special |SHOW| commands:

=====================================================================  ===========================================
:html:`<samp><b>SHOW</b> <var>device</var> <b>CONNECTIONS</b></samp>`  Display current connections to the |device|
:html:`<samp><b>SHOW</b> <var>device</var> <b>STATISTICS</b></samp>`   Display statistics for active connections
:html:`<samp><b>SHOW</b> <var>device</var> <b>LOG</b></samp>`          Display logging for all lines
=====================================================================  ===========================================

Logging can be controlled as follows:

====================================================================================================  ==================================================================
:html:`<samp><b>SET</b> <var>device</var> <b>LOG=</b><var>n</var><b>=</b><var>filename</var></samp>`  Log output of line |n| to :html:`<samp><var>filename</var></samp>`
:html:`<samp><b>SET</b> <var>device</var> <b>NOLOG</b></samp>`                                        Disable logging and close log file
====================================================================================================  ==================================================================

.. |DC|     replace:: :html:`<samp>DC</samp>`
.. |DC10|   replace:: :html:`<samp>DC10</samp>`
.. |DC10E|  replace:: :html:`<samp>DC10E</samp>`
.. |DEC10E| replace:: :html:`<samp>DEC10E</samp>`
.. _DC:
.. _DC10:
.. _DC10E:
.. _DC10E terminal controller:
.. _device 240:

|DC10E| terminal controller (device 240)
""""""""""""""""""""""""""""""""""""""""
The |DC| device was the standard terminal multiplexer for the KA10 and KI10s.
Lines came in groups of 8.
For modem control,
there was a second port for each line.
These could be offset by any number of groups.

=====================================================  ==============================================================
:html:`<samp><b>SET DC LINES=</b><var>n</var></samp>`  Set the number of lines on the |DC10|, multiple of 8
:html:`<samp><b>SET DC MODEM=</b><var>n</var></samp>`  Set the start of where the modem control |DEC10E| lines begins
=====================================================  ==============================================================

.. |TK| replace:: :html:`<samp>TK</samp>`
.. _TK:
.. _TK Knight kludge, TTY scanner:
.. _device 0600:

|TK| Knight kludge, TTY scanner (device 0600)
"""""""""""""""""""""""""""""""""""""""""""""
This is a device with 16 terminal ports.
It's specific to the MIT AI lab and Dynamic Modeling PDP-10s.
By default,
this device is disabled.

.. |MTY| replace:: :html:`<samp>MTY</samp>`
.. _MTY:
.. _MTY Morton terminal multiplexer:
.. _device 400:

|MTY| Morton terminal multiplexer (device 400)
""""""""""""""""""""""""""""""""""""""""""""""
This is a device with 32 high-speed terminal lines.
It's specific to the MIT Mathlab and Dynamic Modeling PDP-10s.
By default,
this device is disabled.

.. |DPK| replace:: :html:`<samp>DPK</samp>`
.. _DPK:
.. _device 604:

|DPK| DK-10 Datapoint kludge (device 604)
"""""""""""""""""""""""""""""""""""""""""
The System Concepts DK-10,
also known as the Datapoint Kludge,
is a device with 16 terminal ports.
This device is specific to ITS,
and is disabled by default.

Network devices
---------------

.. |IMP| replace:: :html:`<samp>IMP</samp>`
.. _IMP:
.. _IMP interface message processor:
.. _device 460:

|IMP| interface message processor (device 460)
""""""""""""""""""""""""""""""""""""""""""""""
This allows the KA/KI to connect to the Internet.
Currently, only supported under ITS.
ITS and other OSes that used the |IMP| did not support modern protocols,
and typically required a complete rebuild to change the IP address.
Because of this,
the |IMP| processor includes built-in NAT and DHCP support.
For ITS,
the system generated IP packets which are translated to the local network.
If the HOST is set to ``0.0.0.0``,
there will be no translation.
If HOST is set to an IP address,
then it will be mapped into the address set in IP.
If DHCP is enabled,
the |IMP| will issue a DHCP request at startup and set IP to the address that is provided.
DHCP is enabled by default.

======================================================================================================  ===========================================================================
:html:`<samp><b>SET IMP MAC=</b><var>xx:xx:xx:xx:xx:xx</var></samp>`                                    Set the MAC address of the |IMP| to the value given
:html:`<samp><b>SET IMP IP=</b><var>ddd.ddd.ddd.ddd/dd</var></samp>`                                    Set the external IP address of the |IMP| along with the net mask in bits
:html:`<samp><b>SET IMP GW=</b><var>ddd.ddd.ddd.ddd</var></samp>`                                       Set the gateway address for the |IMP|
:html:`<samp><b>SET IMP HOST=</b><var>ddd.ddd.ddd.ddd</var></samp>`                                     Set the IP address of the PDP-10 system
:html:`<samp><b>SET IMP DHCP</b></samp>`                                                                Allow the |IMP| to acquire an IP address from the local network via DHCP
                                                                                                        Only HOST must be set if this feature is used
:html:`<samp><b>SET IMP NODHCP</b></samp>`                                                              Disable the |IMP| from making DHCP queries
:html:`<samp><b>SET IMP ARP=</b><var>ddd.ddd.ddd.ddd</var><b>=</b><var>xx:xx:xx:xx:xx:xx</var></samp>`  Create a static ARP entry for the IP address with the indicated MAC address
:html:`<samp><b>SET IMP MIT</b></samp>`                                                                 Set the |IMP| to look like the |IMP| used by MIT for ITS
:html:`<samp><b>SET IMP MPX=</b><var>#</var></samp>`                                                    For ITS, set the |MPX| interrupt to channel |#|
:html:`<samp><b>SET IMP BBN</b></samp>`                                                                 Set the |IMP| to behave like generic BBN |IMP| (Not implemented yet)
:html:`<samp><b>SET IMP WAITS</b></samp>`                                                               Set the |IMP| for Stanford style |IMP| for WAITS
:html:`<samp><b>SHOW IMP ARP</b></samp>`                                                                Display the IP address to MAC address table
======================================================================================================  ===========================================================================

.. |0readme_ethernet.txt| replace:: ``0readme_ethernet.txt``
.. _0readme_ethernet.txt: ../0readme_ethernet.txt

The |IMP| device must be attached to an available Ethernet interface.
To determine which devices are available,
use the :html:`<samp><b>SHOW ETHERNET</b></samp>` command to list the potential interfaces.
Check out the |0readme_ethernet.txt|_ file from the top of the source directory.

The |IMP| device can be configured in several ways.
Ether it can connect directly to an ethernet port (via TAP),
or it can be connected via a TUN interface.
If configured via TAP interface,
the |IMP| will behave like any other Ethernet interface,
and if asked,
grab its own address.
In environments where this is not desired,
the TUN interface can be used.
When configured under a TUN interface,
the simulated network is a collection of ports on the local host.
These can be mapped based on configuration options;
see the |0readme_ethernet.txt|_ file as to options.

With the |IMP| interface,
the IP address of the simulated system is static,
and under ITS is configured into the system at compile time.
This address should be given to the |IMP| with the :html:`<samp><b>SET IMP HOST=</b><var>ip</var></samp>` command,
the |IMP| will direct all traffic it sees to this address.
If this address is not the same as the address of the system as seen by the network,
then this address can be set with :html:`<samp><b>SET IMP IP=</b><var>ip</var></samp>`,
and :html:`<samp><b>SET IMP GW=</b><var>ip</var></samp>`,
or :html:`<samp><b>SET IMP DHCP</b></samp>`,
which will allow the |IMP| to request an address from a local DHCP server.
The |IMP| will translate the packets it receives/sends to look like the appeared from the desired address.
The |IMP| will also correctly translate FTP requests in this configuration.

When running under a TUN interface,
|IMP| is on a virtual ``10.0.2.0`` network.
The default gateway is ``10.0.2.1``,
with the default |IMP| at ``10.0.2.15``.
For this mode,
DHCP can be used.

.. |CH| replace:: :html:`<samp>CH</samp>`
.. _CH:
.. _CH Chaosnet interface:
.. _device 470:

|CH| Chaosnet interface (device 470)
""""""""""""""""""""""""""""""""""""
Chaosnet was another method of network access for ITS.
Chaosnet can be connected to another ITS system or a VAX/PDP-11 running BSD Unix or VMS.
Chaosnet runs over UDP protocol under Unix.
You must specify a node number,
peer to talk to,
and your local UDP listening port.
All UDP packets are sent to the peer for further processing.

=======================================================================  ====================================
:html:`<samp><b>SET CH NODE=</b><var>n</var></samp>`                     Set the Node number for this system
:html:`<samp><b>SET CH PEER=</b><var>ddd.ddd.ddd.ddd:dddd</var></samp>`  Set the Peer address and port number
=======================================================================  ====================================

The |CH| device must be attached to a UDP port number.
This is where it will receive UDP packets from its peer.

PDP-6 devices
-------------
These devices could be attached to the KA10.
Stanford WAITS used them.
Early versions of TOPS10 could also run with these devices.
By default,
these devices are disabled.

.. |DCT| replace:: :html:`<samp>DCT</samp>`
.. _DCT:
.. _DCT type 136 data control:

|DCT| type 136 data control (Device 200/204)
""""""""""""""""""""""""""""""""""""""""""""
This device acted as a data multiplexer for the DECtape/Magtape and disk.
Individual devices could be assigned channels on this device.
Devices which use the |DCT| include a |DCT| option with takes two octal digits,
the first is the |DCT| device 0 or 1,
the second is the channel 1 to 7.

.. |DTC| replace:: :html:`<samp>DTC</samp>`
.. _DTC:
.. _DTC type 551 DECtape controller:
.. _device 210:

|DTC| type 551 DECtape controller (device 210)
""""""""""""""""""""""""""""""""""""""""""""""
This was the standard DECtape controller for the PDP-6.
This controller loads the tape into memory and uses the buffered version.
You need to specify which DCT unit and channel the tape is connected to.

=====================================================  ==============================================
:html:`<samp><b>SET DTC DCT=</b><var>uc</var></samp>`  Set the DTC to connect to DCT unit and channel
=====================================================  ==============================================

Each individual tape drive support several options: 

==================================================================  ================================================
:html:`<samp><b>SET DTC</b><var>n</var> <b>LOCKED</b></samp>`       Set the mag tape to be read-only
:html:`<samp><b>SET DTC</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set the mag tape to be writable
:html:`<samp><b>SET DTC</b><var>n</var> <b>18b</b></samp>`          Default, tapes are considered to be 18-bit tapes
:html:`<samp><b>SET DTC</b><var>n</var> <b>16b</b></samp>`          Tapes are converted from 16-bit to 18-bit
:html:`<samp><b>SET DTC</b><var>n</var> <b>12b</b></samp>`          Tapes are converted from 12-bit to 18-bit
==================================================================  ================================================

.. |MTC| replace:: :html:`<samp>MTC</samp>`
.. _MTC:
.. _MTC type 516 magtape controller:
.. _device 220:

|MTC| type 516 magtape controller (device 220)
""""""""""""""""""""""""""""""""""""""""""""""
This was the standard Magtape controller for the PDP-6.
This device handled only 7-track tapes.
The simulator has the option to automatically translate 8-track tapes into 7-track tapes.
You need to specify which DCT unit and channel the tape is connected to.

=====================================================  ===============================================
:html:`<samp><b>SET MTC DCT=</b><var>uc</var></samp>`  Sets the MTC to connect to DCT unit and channel
=====================================================  ===============================================

Each individual tape drive supports several options:

==================================================================  =======================================================
:html:`<samp><b>SET MTC</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET MTC</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
:html:`<samp><b>SET MTC</b><var>n</var> <b>9T</b></samp>`           Set this unit to simulated 9-track format
:html:`<samp><b>SET MTC</b><var>n</var> <b>7T</b></samp>`           Set this unit to read/write 7-track tapes (with parity)
==================================================================  =======================================================

.. |DSK| replace:: :html:`<samp>DSK</samp>`
.. _DSK:
.. _DSK type 270 disk controller:
.. _device 270:

|DSK| type 270 disk controller (device 270)
"""""""""""""""""""""""""""""""""""""""""""
The 270 disk could support up to 4 units.
The controller had to be connected to a type 136 data controller.
You need to specify which DCT unit and channel the disk is connected to.

=====================================================  ==============================================
:html:`<samp><b>SET DSK DCT=</b><var>uc</var></samp>`  Set the DSK to connect to DCT unit and channel
=====================================================  ==============================================

Each individual disk drive support several options:

==================================================================  =============================
:html:`<samp><b>SET DSK</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET DSK</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
==================================================================  =============================

.. |DCS| replace:: :html:`<samp>DCS</samp>`
.. _DCS:
.. _DCS type 630 terminal multiplexer:
.. _device 300:

|DCS| type 630 terminal multiplexer (device 300)
""""""""""""""""""""""""""""""""""""""""""""""""
See section on terminal multiplexers on generic setup.

Symbolic display and input
==========================
.. |EXAMINE| replace:: :html:`<samp><b>EXAMINE</b></samp>`
.. |DEPOSIT| replace:: :html:`<samp><b>DEPOSIT</b></samp>`

The KA10 simulator implements symbolic display and input.
These are controlled by the following switches to the |EXAMINE| and |DEPOSIT| commands:

======  ================================================================================
``-v``  Look up the address via translation; will return nothing if address is not valid
``-u``  With the ``-v`` option, use user space instead of executive space
``-a``  Display/enter ASCII data
``-p``  Display/enter packed ASCII data
``-c``  Display/enter Sixbit character data
``-m``  Display/enter symbolic instructions
\       Display/enter octal data
======  ================================================================================

Symbolic instructions can be of the formats:

* ``Opcode ac,operand``
* ``Opcode operands``
* ``I/O Opcode device,address``

Operands can be one or more of the following in order:

* Optional ``@`` for indirection
* ``+`` or ``-`` to set sign
* Octal number
* Optional (``ac``) for indexing

Breakpoints can be set at real memory address.
The PDP-10 supports 3 types of breakpoints:
Execution, Memory Access, and Memory Modify.
The default is execution.
Adding ``-R`` to the breakpoint command will stop the simulator on access to that memory location,
either via fetch,
indirection,
or operand access.
Adding ``-w`` will stop the simulator when the location is modified.

The simulator can load RIM files,
SAV files,
EXE files,
WAITS octal DMP files,
and MIT SBLK files.

When instruction history is enabled,
the history trace shows internal status of various registers at the start of the instruction execution.

.. |PC|    replace:: :html:`<samp class="register">PC</samp>`
.. |AC|    replace:: :html:`<samp class="register">AC</samp>`
.. |EA|    replace:: :html:`<samp class="register">EA</samp>`
.. |AR|    replace:: :html:`<samp class="register">AR</samp>`
.. |RES|   replace:: :html:`<samp class="register">RES</samp>`
.. |FLAGS| replace:: :html:`<samp class="register">FLAGS</samp>`
.. |IR|    replace:: :html:`<samp class="register">IR</samp>`

=======  ======================================================================
|PC|     Show the PC of the instruction executed
|AC|     The contents of the referenced AC
|EA|     The final computed effective address
|AR|     Generally the operand that was computed
|RES|    The AR register after the instruction was complete
|FLAGS|  The values of the |FLAGS| register before execution of the instruction
|IR|     The fetched instruction followed by the disassembled instruction
=======  ======================================================================

The PDP-10 simulator allows for memory reference and memory modify breakpoints with the ``-r`` and ``-w`` options given to the break command.

OS specific configurations
==========================
TOPS10
------
The default configuration supports Tops10,
so there is no extra configuration required. 

ITS
---
To run ITS,
the CPU must be :html:`<samp><b>SET CPU ITS MPX 1024K</b></samp>`;
this will enable the ITS pager,
special instructions,
and the interrupt multiplexer.
|DK| should be disabled,
along with all Mass bus devices.
The following devices should be enabled:
|PD|, (also optionally) |DPY|, |STK|, |TK|, |MTY|, |TEN11|, |AUXCPU|, |IMP|, |CH|.
ITS used a second interrupt multiplexer,
so some devices need to be configured to specific channels.
|TM10| (|MTA|) was at ``MPX=7``,
DECtapes (|DT|) was at ``MPX=6``,
and the |IMP| was at ``MPX=4``.

WAITS
-----
WAITS has two configurations,
one uses the default two relocation registers.
This can be done with :html:`<samp><b>SET CPU WAITS MAOFF</b></samp>`;
the |PMP|_ disk is the only supported disk.
WAITS used some PDP-6 devices
(|DTC|_, |MTC|_, |DCS|_, |DTC|_).
Also, the following special devices for WAITS should be enabled (|DKB|_, |III|_).
The |DTC|_ should be configured to ``dct=02``,
and the |MTC| should be configured to ``dct=01``.
After 1975,
Stanford added a BBN pager unit to their KA,
to run later versions of WAITS,
:html:`<samp><b>set cpu waits maoff bbn 512k</b></samp>`.
The :html:`<samp><b>waits</b></samp>` flag enables the FIX and XCTR instructions for waits.
:html:`<samp><b>maoff</b></samp>` moves the trap vectors to the 0140.
For systems numbers over 49,
BBN pager needs to also be enabled on the CPU.
