.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. |ATTACH| replace:: :html:`<samp><b>ATTACH</b></samp>`
.. |MPX|    replace:: :html:`<samp>MPX</samp>`
.. |SET|    replace:: :html:`<samp><b>SET</b></samp>`
.. |SHOW|   replace:: :html:`<samp><b>SHOW</b></samp>`
.. |device| replace:: :html:`<samp><var>device</var></samp>`
.. |format| replace:: :html:`<samp><var>format</var></samp>`
.. |n|      replace:: :html:`<samp><var>n</var></samp>`
.. |#|      replace:: :html:`<samp><var>#</var></samp>`

*******************************
Dec PDP-10/KL10 simulator usage
*******************************
:Date: 2022-04-20
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
Originally,
the DEC PDP-10 computer started as the PDP-6.
This was a 36-bit computer that was designed for timesharing,
which was introduced in 1964.
The original goal of the machine was to allow for processing of many 6-bit characters at a time.
36 bits were also common in machines like the IBM 7090, GE 645, and Univac 11xx lines.
Several systems influenced the design of the PDP-6,
like CTSS, Lisp, support for larger memory.
The PDP-6 was cancelled by DEC due to production problems.
The engineers designed a smaller replacement,
which happened to be a 36-bit computer that looked very much like the PDP-6.
This was called the PDP-10, which was later renamed to "DECSystem-10".
The system supported up to 256K words of memory.

The first PDP-10 was labeled KA10,
and added a few instructions to the PDP-6.
Both the PDP-6 and PDP-10 used a base and limit relocation scheme.
The KA10 generally offered two registers,
one of user data and the second for user-shared code.
These were referred to the Low-Segment and High-Segment;
the High-Segment could be shared with several jobs.
The next version was called KI10 for Integrated.
This added support for paging and double-precision floating-point instructions.
It also added 4 sets of registers to improve context switching time.
It could also support up to 4Mega words of memory.
Following the KI10 was the KL10 (for Low-Cost).
The KL10 added double-precision integer instructions and instructions to improve COBOL performance.
This was the first version which was microcoded.
The KL10 was extended to support user programs larger then 256k.
The final version to make it to market was the KS10 (for Small):
this was a bit-slice version of the PDP-10 which used UNIBUS devices,
which were cheaper than the KL10 devices.

The original operating system for the PDP-6/PDP-10 was just called "Monitor".
It was designed to fit into 6K words.
Around the third release, Swapping was added.
The sixth release saw the addition of virtual memory.
Around the fourth release it was given the name "TOPS-10".
Around this time,
BBN was working on a paging system and implemented it on the PDP-10.
This was called "Tenex".
This was later adopted by DEC and became "Tops-20".

During the mid-60s,
a group at MIT,
who were not happy with how Multics was being developed,
decided to create their own operating system,
which they called Incompatible Timesharing System ("ITS"):
a play on the original project called Compatible Timesharing System ("CTSS").
CTSS was implemented by MIT on their IBM 7090 as an experimental system that allowed multiple time-sharing users to co-exist on the same machine running batch processing
(hence the term "compatable").

Also during the mid-60s,
a group at Stanford Artificial Intelligence Laboratory (SAIL)
started with a version of TOPS-10 and heavily modified it to become WAITS.

During the 70s,
Tymshare starting with DEC TOPS-10 system modified it to support random access files,
paging with working sets,
and spawn-able processes.
This ran on the KI10, KL10, and KS10.

The PDP-10 was ultimately replaced by the VAX.

Simulator files
===============
To compile the DEC 10/KL10 simulator,
you must define ``USE_INT64`` as part of the compilation command-line.

+--------------+-----------------+--------------------------------------------+
| Subdirectory | File            | Contains                                   |
+==============+=================+============================================+
| ``PDP10/``   | ``kx10_defs.h`` | KL10 simulator definitions                 |
|              +-----------------+--------------------------------------------+
|              | ``kx10_cpu.c``  | KL10 CPU                                   |
|              +-----------------+--------------------------------------------+
|              | ``kx10_df.c``   | DF10/C controller                          |
|              +-----------------+--------------------------------------------+
|              | ``kx10_rh.c``   | RH10/RH20 controller                       |
|              +-----------------+--------------------------------------------+
|              | ``kx10_diskh``  | Disk formatter definitions                 |
|              +-----------------+--------------------------------------------+
|              | ``kx10_disk.c`` | Disk formatter routine                     |
|              +-----------------+--------------------------------------------+
|              | ``kx10_sys.c``  | KL10 system interface                      |
|              +-----------------+--------------------------------------------+
|              | ``kx10_lp.c``   | LP 10 line printer                         |
|              +-----------------+--------------------------------------------+
|              | ``kx10_pt.c``   | Paper tape reader and punch                |
|              +-----------------+--------------------------------------------+
|              | ``kx10_dp.c``   | RP10 disk controller, RP01/2/3 disks       |
|              +-----------------+--------------------------------------------+
|              | ``kx10_rp.c``   | RH10 disk controller, RP04/5/6 disks       |
|              +-----------------+--------------------------------------------+
|              | ``kx10_rs.c``   | RH10 disk controller RS04 fixed head disks |
|              +-----------------+--------------------------------------------+
|              | ``kx10_dt.c``   | TD10 DECtape controller                    |
|              +-----------------+--------------------------------------------+
|              | ``kx10_mt.c``   | TM10A/B magnetic tape controller           |
|              +-----------------+--------------------------------------------+
|              | ``kx10_tu.c``   | RH10/TM03 magnetic tape controller         |
|              +-----------------+--------------------------------------------+
|              | ``kx10_lp.c``   | Line printer                               |
|              +-----------------+--------------------------------------------+
|              | ``kx10_cr.c``   | Punch card reader                          |
|              +-----------------+--------------------------------------------+
|              | ``kx10_cp.c``   | Punch card punch                           |
+--------------+-----------------+--------------------------------------------+

KL10 features
=============
The KL10 simulator is configured as follows:

+----------------+--------------------------------------+
| Device name(s) | Simulates                            |
+================+======================================+
| |CPU|_         | KL10 CPU with 256KW of memory        |
+----------------+--------------------------------------+
| |CTY|_         | Console TTY                          |
+----------------+--------------------------------------+
| |PTP|_         | Paper tape punch                     |
+----------------+--------------------------------------+
| |PTR|_         | Paper tape reader                    |
+----------------+--------------------------------------+
| |LPT|_         | LP10 line printer                    |
+----------------+--------------------------------------+
| |LP20|_        | LP20 line printer on FE              |
+----------------+--------------------------------------+
| |MTA|_         | TM10A/B tape controller              |
+----------------+--------------------------------------+
| |FSA|_         | RS04 disk controller via RH10        |
+----------------+--------------------------------------+
| |DPA|_         | RP10 disk controller                 |
+----------------+                                      |
| |DPB|_         |                                      |
+----------------+--------------------------------------+
| |RPA|_         | RH10/RH20 disk controllers via RH10  |
+----------------+                                      |
| |RPB|_         |                                      |
+----------------+                                      |
| |RPC|_         |                                      |
+----------------+                                      |
| |RPD|_         |                                      |
+----------------+--------------------------------------+
| |TUA|_         | TM02 tape controller via RH10/RH20   |
+----------------+--------------------------------------+
| |TTY|_         | TTY lines on FE                      |
+----------------+--------------------------------------+
| |DT|_          | TD10 DECtape controller              |
+----------------+--------------------------------------+
| |DC|_          | DC10 communications controller       |
+----------------+--------------------------------------+
| |PD|_          | Coriolis clock                       |
+----------------+--------------------------------------+
| |IMP|_         | IMP network interface                |
+----------------+--------------------------------------+
| |NI|_          | NIA20 network interface              |
+----------------+--------------------------------------+
| |CH|_          | CH10 Chaosnet interface              |
+----------------+--------------------------------------+

.. |CPU| replace:: :html:`<samp>CPU</samp>`

|CPU|
-----
The |CPU| options include setting memory size and O/S customization.

=========================================================  ==============================================
:html:`<samp><b>SET CPU 16K</b></samp>`                    Set memory to 16K
:html:`<samp><b>SET CPU 32K</b></samp>`                    Set memory to 32K
:html:`<samp><b>SET CPU 48K</b></samp>`                    Set memory to 48K
:html:`<samp><b>SET CPU 64K</b></samp>`                    Set memory to 64K
:html:`<samp><b>SET CPU 96K</b></samp>`                    Set memory to 96K
:html:`<samp><b>SET CPU 128K</b></samp>`                   Set memory to 128K
:html:`<samp><b>SET CPU 256K</b></samp>`                   Set memory to 256K
:html:`<samp><b>SET CPU 512K</b> </samp>`                  Set memory to 512K
:html:`<samp><b>SET CPU 768K</b></samp>`                   Set memory to 768K
:html:`<samp><b>SET CPU 1024K</b></samp>`                  Set memory to 1024K
:html:`<samp><b>SET CPU 2048K</b></samp>`                  Set memory to 2048K
:html:`<samp><b>SET CPU 4096K</b></samp>`                  Set memory to 4096K
:html:`<samp><b>SET CPU SERIAL=</b><var>val</var></samp>`  Set serial number of system (default: 1025)
:html:`<samp><b>SET CPU DF10</b></samp>`                   Set DF10 emulation to model A, max 256K memory
:html:`<samp><b>SET CPU DF10C</b></samp>`                  Set DF10 emulation to model C, no max
:html:`<samp><b>SET CPU ITS</b></samp>`                    Set CPU for ITS customization
:html:`<samp><b>SET CPU KL10A</b></samp>`                  Set CPU to KL10A
:html:`<samp><b>SET CPU KL10B</b></samp>`                  Set CPU to KL10B
:html:`<samp><b>SET CPU NOIDLE</b></samp>`                 Disable idle detection
:html:`<samp><b>SET CPU IDLE</b></samp>`                   Enable idle detection
=========================================================  ==============================================

CPU registers include the visible state of the processor as well as the control registers for the interrupt system.

.. |PC|          replace:: :html:`<samp class="register">PC</samp>`
.. |FLAGS|       replace:: :html:`<samp class="register">FLAGS</samp>`
.. |FM0|         replace:: :html:`<samp class="register">FM0</samp>`
.. |FM17|        replace:: :html:`<samp class="register">FM17</samp>`
.. |SW|          replace:: :html:`<samp class="register">SW</samp>`
.. |MI|          replace:: :html:`<samp class="register">MI</samp>`
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
|MI|              36    Monitor display
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
:html:`<samp><b>SET CPU HISTORY=0</b></samp>`              Disable history
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

.. |DTE 200| replace:: :html:`<samp>DTE 200</samp>`
.. _DTE 200:
.. _PDP-11 front-end processor:

PDP-11 front-end processor (|DTE 200|)
--------------------------------------
.. |CTY|  replace:: :html:`<samp>CTY</samp>`
.. |DTE0| replace:: :html:`<samp>DTE0</samp>`
.. _CTY:
.. _Console teletype:

Console teletype (|CTY|) (on |DTE0|)
""""""""""""""""""""""""""""""""""""
The console station allows for communications with the operating system.

======================================  ================================
:html:`<samp><b>SET CTY 7B</b></samp>`  7-bit characters, space parity
:html:`<samp><b>SET CTY 8B</b></samp>`  8-bit characters, space parity
:html:`<samp><b>SET CTY 7P</b></samp>`  7-bit characters, space parity
:html:`<samp><b>SET CTY UC</b></samp>`  Translate lowercase to uppercase
======================================  ================================

The front-end device needs to be configured for the specific operating system being run.
For TOPS-10 7 and TOPS-20,
this must be |RSX20|.
For TOPS-10 6.03,
set it to |RSX10|.

.. |RSX10| replace:: :html:`<samp>RSX10</samp>`
.. |RSX20| replace:: :html:`<samp>RSX20</samp>`

=========================================  =================================================
:html:`<samp><b>SET CTY RSX10</b></samp>`  Set the front-end to emulate a 11 running |RSX10|
:html:`<samp><b>SET CTY RSX20</b></samp>`  Set the front-end to emulate a 11 running |RSX20|
=========================================  =================================================

The |CTY| also supports a method for stopping the TOPS-10 operating system.

.. |LP20| replace:: :html:`<samp>LP20</samp>`
.. |DTE|  replace:: :html:`<samp>DTE</samp>`
.. _LP20:
.. _LP20 printer on front-end:

LP20 printer on front-end (|LP20|) (on |DTE|)
"""""""""""""""""""""""""""""""""""""""""""""
The line printer (|LP20|) writes data to a disk file as ASCII text with terminating newlines.
Currently set to handle standard signals to control paper advance.
Currently, only one line printer device is supported.

==============================================================  ==============================================================
:html:`<samp><b>SET LP20 LC</b></samp>`                         Allow printer to print lowercase
:html:`<samp><b>SET LP20 UC</b></samp>`                         Print only uppercase
:html:`<samp><b>SET LP20 LINESPERPAGE=</b><var>n</var></samp>`  Set the number of lines before an auto form feed is generated.
                                                                There is an automatic margin of 6 lines.
                                                                There is a maximum of 100 lines per page.
==============================================================  ==============================================================

.. _Front-end terminal connections:

Front-end terminal connections (|TTY|)
""""""""""""""""""""""""""""""""""""""
See section `Front-end terminals`_.

Unit record I/O devices
-----------------------

.. |PTR| replace:: :html:`<samp>PTR</samp>`
.. _PTR:
.. _Paper tape reader:
.. _Paper tape reader (PTR):
.. _device 104:

Paper tape reader (|PTR|) (device 104)
""""""""""""""""""""""""""""""""""""""
Reads a paper tape from a disk file.
By default,
device is disabled.

.. |PTP| replace:: :html:`<samp>PTP</samp>`
.. _PTP:
.. _Paper tape punch:
.. _Paper tape punch (PTP):
.. _device 100:

Paper tape punch (|PTP|) (device 100)
"""""""""""""""""""""""""""""""""""""
Punches a paper tape to a disk file.
By default,
device is disabled.

.. |CR| replace:: :html:`<samp>CR</samp>`
.. _CR:
.. _Card reader:
.. _Card reader (CR):
.. _device 150:

Card reader (|CR|) (device 150)
"""""""""""""""""""""""""""""""
The card reader (|CR|) reads data from a disk file.
Card reader files can either be text (one character per column) or column binary (two characters per column).
By default,
device is disabled.
The file type can be specified with a |SET| command:

===================================================================  =================================
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=TEXT</b></samp>`    Set ASCII text mode
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=BINARY</b></samp>`  Set for binary card images
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=BCD</b></samp>`     Set for BCD records
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=CBN</b></samp>`     Set for column binary BCD records
:html:`<samp><b>SET CR</b><var>n</var> <b>FORMAT=AUTO</b></samp>`    Automatically determine format
===================================================================  =================================

or in the |ATTACH| command:

=============================================================================================  ===================================================================
:html:`<samp><b>ATTACH CR</b><var>n</var> <var>file</var></samp>`                              Attach a file
:html:`<samp><b>ATTACH CR</b><var>n</var> <b>-f</b> <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp><b>ATTACH CR</b><var>n</var> <b>-s</b> <var>file</var></samp>`                    Add file onto current cards to read
:html:`<samp><b>ATTACH CR</b><var>n</var> <b>-e</b> <var>file</var></samp>`                    After file is read in, the reader will receive and end-of-file flag
=============================================================================================  ===================================================================

.. |CP| replace:: :html:`<samp>CP</samp>`
.. _CP:
.. _Card punch:
.. _Card punch (CP):
.. _device 110:

Card punch (|CP|) (device 110)
""""""""""""""""""""""""""""""
The card reader (|CP|) writes data to a disk file.
Cards are simulated as ASCII lines with terminating newlines.
Card punch files can either be text (one character per column) or column binary (two characters per column).
By default,
device is disabled.
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

.. |LPT| replace:: :html:`<samp>LPT</samp>`
.. _LPT:
.. _Line printer:
.. _Line printer (LPT):
.. _device 124:

Line printer (|LPT|) (device 124)
"""""""""""""""""""""""""""""""""
The line printer (|LPT|) writes data to a disk file as ASCII text with terminating newlines.
Currently set to handle standard signals to control paper advance.

================================================================================  ==============================================================
:html:`<samp><b>SET LPT</b><var>n</var> <b>LC<b></samp>`                          Allow printer to print lowercase
:html:`<samp><b>SET LPT</b><var>n</var> <b>UC<b></samp>`                          Print only uppercase
:html:`<samp><b>SET LPT</b><var>n</var> <b>UTF8<b></samp>`                        Print control characters as UTF-8 characters
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

.. |MTA|   replace:: :html:`<samp>MTA</samp>`
.. |TM10A| replace:: :html:`<samp>TM10A</samp>`
.. |TM10B| replace:: :html:`<samp>TM10B</samp>`
.. _MTA:
.. _TM10:
.. _TM10 magnetic tape:
.. _TM10 magnetic tape (MTA):
.. _device 340:
.. _device 344:

TM10 magnetic tape (|MTA|) (device 340/344)
"""""""""""""""""""""""""""""""""""""""""""
The TM10 was the most common tape controller on KA10 and KI10.
The TM10 came in two models,
the |TM10A| and the |TM10B|.
The B model added a DF10,
which allowed the tape data to be transferred without intervention of the CPU.
The device has 2 options.
By default,
this device is disabled.

=====================================================  ================================================
:html:`<samp><b>SET MTA TYPE=</b><var>t</var></samp>`  Set the type of the controller to ``A`` or ``B``
:html:`<samp><b>SET MTA MPX=</b><var>#</var></samp>`   For ITS, set the |MPX| interrupt to channel |#|
=====================================================  ================================================

Each individual tape drive support several options:
|MTA| used as an example.

==================================================================  =======================================
:html:`<samp><b>SET MTA</b><var>n</var> <b>7T</b></samp>`           Set the mag tape unit to 7-track format
:html:`<samp><b>SET MTA</b><var>n</var> <b>9T</b></samp>`           Set the mag tape unit to 9-track format
:html:`<samp><b>SET MTA</b><var>n</var> <b>LOCKED</b></samp>`       Set the mag tape to be read-only
:html:`<samp><b>SET MTA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set the mag tape to be writable
==================================================================  =======================================

.. |DT| replace:: :html:`<samp>DT</samp>`
.. _DT:
.. _TD10:
.. _TD10 DECTape:
.. _TD10 DECTape (DT):
.. _device 320:
.. _device 324:

TD10 DECTape (|DT|) (device 320/324)
""""""""""""""""""""""""""""""""""""
The TD10 was the standard DECTape controller for KA and KIs.
This controller loads the tape into memory and uses the buffered version.
Each individual tape drive support several options:
|DT| used as an example.
By default,
this device is disabled.

=================================================================  ================================================
:html:`<samp><b>SET DT</b><var>n</var> <b>LOCKED</b></samp>`       Set the mag tape to be read-only
:html:`<samp><b>SET DT</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set the mag tape to be writable
:html:`<samp><b>SET DT</b><var>n</var> <b>18b</b></samp>`          Default, tapes are considered to be 18-bit tapes
:html:`<samp><b>SET DT</b><var>n</var> <b>16b</b></samp>`          Tapes are converted from 16-bit to 18-bit
:html:`<samp><b>SET DT</b><var>n</var> <b>12b</b></samp>`          Tapes are converted from 12-bit to 18-bit
=================================================================  ================================================

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
By default,
this device is disabled.

Disk I/O devices
----------------
The PDP-10 supported many disk controllers.
The KL10 supports only mass bus devices.
But in practice could interface to older drives,
these are currently not enabled in the simulator.

.. |DPA| replace:: :html:`<samp>DPA</samp>`
.. |DPB| replace:: :html:`<samp>DPB</samp>`
.. _DPA:
.. _DPB:
.. _DPA disk controller:
.. _DPB disk controller:
.. _DPA/DPB disk controller:
.. _device 250:
.. _device 254:

|DPA|\ /\ |DPB| disk controller (device 250/254)
""""""""""""""""""""""""""""""""""""""""""""""""
.. |RP01| replace:: :html:`<samp>RP01</samp>`
.. |RP02| replace:: :html:`<samp>RP02</samp>`
.. |RP03| replace:: :html:`<samp>RP03</samp>`
.. |SIMH| replace:: :html:`<samp>SIMH</samp>`
.. |DBD9| replace:: :html:`<samp>DBD9</samp>`
.. |DLD9| replace:: :html:`<samp>DLD9</samp>`

The RP10 disk controller used a DF10 to transfer data to memory.
There were three types of disks that could be connected to the RP10:
the |RP01|, |RP02|, |RP03|.
The |RP01| held up to 1.2M words,
the |RP02| 5.2M words,
and the |RP03| 10M words.
The second controller is |DPB|.
Disks can be stored in one of several file formats,
|SIMH|, |DBD9|, and |DLD9|.
The latter two are for compatibility with other simulators.
By default,
this device is disabled.

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

|format| can be |SIMH|, |DBD9|, or |DLD9|.

Massbus devices
---------------
Massbus devices are attached via RH10s or RH20s.
RH10 devices start a device 270 and go up (274,360,364,370,374).
RH20 devices start at 540 and go up by +4 for each device.
For TOPS-10,
devices need to be in the order |RS|_, |RP|_, |TU|_.
By default,
|RS|_ disks are disabled.
The first unit which is not enabled will get device 270,
units will be assigned the next available address automatically.
The device configuration must match that which is defined in the OS.
TOPS-20 requires RH20 to work.

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
Up to 4 strings of up to 8 |RP| drives can be configured.
By default,
the third and fourth controllers are disabled.
These are addresses as |RPA|, |RPB|, |RPC|, |RPD|.
Disks can be stored in one of several file formats:
|SIMH|, |DBD9|, and |DLD9|.
The latter two are for compatibility with other simulators.

==================================================================  ========================================
:html:`<samp><b>SET RPA</b><var>n</var> <b>RP04</b></samp>`         Set this unit to be an RP04 (19 MWords)
:html:`<samp><b>SET RPA</b><var>n</var> <b>RP06</b></samp>`         Set this unit to be an RP06 (39 MWords)
:html:`<samp><b>SET RPA</b><var>n</var> <b>RP07</b></samp>`         Set this unit to be an RP07 (110 MWords)
:html:`<samp><b>SET RPA</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET RPA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
:html:`<samp><b>SET RPA</b> <b>RH10</b></samp>`                     Set controller to RH10 type
:html:`<samp><b>SET RPA</b> <b>RH20</b></samp>`                     Set controller to RH20 type
==================================================================  ========================================

To attach a disk,
use the |ATTACH| command:

==============================================================================================  ===================================
:html:`<samp><b>ATTACH RPA</b><var>n</var> <var>file</var></samp>`                              Attach a file
:html:`<samp><b>ATTACH RPA</b><var>n</var> <b>-f</b> <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp><b>ATTACH RPA</b><var>n</var> <b>-n</b> <var>file</var></samp>`                    Create a blank disk
==============================================================================================  ===================================

|format| can be |SIMH|, |DBD9|, or |DLD9|.

.. |RS|  replace:: :html:`<samp>RS</samp>`
.. |FSA| replace:: :html:`<samp>FSA</samp>`
.. _RS:
.. _FSA:

|RS| disk drives
""""""""""""""""
One string of up to 8 |RS| drives can be configured.
These drives are fixed head swapping disks.
By default,
they are disabled.

==================================================================  ========================================
:html:`<samp><b>SET FSA</b><var>n</var> <b>RS03</b></samp>`         Set this unit to be an RS03 (262 KWords)
:html:`<samp><b>SET FSA</b><var>n</var> <b>RS04</b></samp>`         Set this unit to be an RS04 (262 KWords)
:html:`<samp><b>SET FSA</b><var>n</var> <b>LOCKED</b></samp>`       Set this unit to be read-only
:html:`<samp><b>SET FSA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set this unit to be writable
:html:`<samp><b>SET FSA</b> <b>RH10</b></samp>`                     Set controller to RH10 type
:html:`<samp><b>SET FSA</b> <b>RH20</b></samp>`                     Set controller to RH20 type
==================================================================  ========================================

.. |TU|  replace:: :html:`<samp>TU</samp>`
.. |TUA| replace:: :html:`<samp>TUA</samp>`
.. _TU:
.. _TUA:

|TU| tape drives
""""""""""""""""
The |TUA| is a Mass bus tape controller using a TM03 formatter.
There can be one per RH10 and it can support up to 8 drives.

==================================================================  ================================
:html:`<samp><b>SET TUA</b><var>n</var> <b>LOCKED</b></samp>`       Set the mag tape to be read-only
:html:`<samp><b>SET TUA</b><var>n</var> <b>WRITEENABLE</b></samp>`  Set the mag tape to be writable
:html:`<samp><b>SET TUA RH10</b></samp>`                            Set controller to RH10 type
:html:`<samp><b>SET TUA RH20</b></samp>`                            Set controller to RH20 type
==================================================================  ================================

Terminal multiplexer I/O devices
--------------------------------
All terminal multiplexers must be attached in order to work.
The |ATTACH| command specifies the port to be used for Telnet sessions:

====================================================================  =====================
:html:`<samp><b>ATTACH</b> <var>device</var> <var>port</var></samp>`  Set up listening port
====================================================================  =====================

... where :html:`<samp><var>port</var></samp>` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

Once attached and the simulator is running,
the multiplexer listens for connections on the specified port.
It assumes that any incoming connection is a Telnet connections.
The connections remain open until disconnected either by the Telnet client,
a :html:`<samp><b>SET</b> <var>device</var> <b>DISCONNECT</b></samp>` command,
or a :html:`<samp><b>DETACH</b> <var>device</var></samp>` command.

================================================================================  ===================
:html:`<samp><b>SET</b> <var>device</var> <b>DISCONNECT=</b><var>n</var></samp>`  Disconnect line |n|
================================================================================  ===================

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

.. |DC|    replace:: :html:`<samp>DC</samp>`
.. |DC10E| replace:: :html:`<samp>DC10E</samp>`
.. _DC:
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
By default,
this device is disabled.

=====================================================  =================================================================================
:html:`<samp><b>SET DC LINES=</b><var>n</var></samp>`  Set the number of lines on the DC10, multiple of 8
:html:`<samp><b>SET DC MODEM=</b><var>n</var></samp>`  Set the start of where the modem control :html:`<samp>DEC10E</samp>` lines begins
=====================================================  =================================================================================

.. |TTY| replace:: :html:`<samp>TTY</samp>`
.. _TTY:
.. _Front-end terminals:

Front-end terminals (|TTY|)
"""""""""""""""""""""""""""
The front-end PDP-11 could be configured with many DH11s offering 16 lines per module.
All lines supported modem control and speed setting.

=====================================================  ===================================================================
:html:`<samp><b>SET DC LINES=</b><var>n</var></samp>`  Set the number of lines on the front-end PDP-11, in multiples of 16
=====================================================  ===================================================================

Network devices
---------------

.. |NI| replace:: :html:`<samp>NI</samp>`
.. _NI:
.. _NIA20:
.. _NIA20 network interface:
.. _NIA20 network interface (NI):
.. _device 564:

NIA20 network interface (|NI|) (device 564)
"""""""""""""""""""""""""""""""""""""""""""
The NIA20 actually sits in slot 5 of the RH20 rack and configures itself as a RH20 device.
The system will automatically move other conflicting RH20 devices around.

===================================================================  ==================================================
:html:`<samp><b>SET NI MAC=</b><var>xx:xx:xx:xx:xx:xx</var></samp>`  Set the MAC address of the |NI| to the value given
===================================================================  ==================================================

.. |IMP| replace:: :html:`<samp>IMP</samp>`
.. _IMP:
.. _IMP interface message processor:
.. _device 460:

|IMP| interface message processor (device 460)
""""""""""""""""""""""""""""""""""""""""""""""
This allows the KA/KI/KL to connect to the Internet.
Currently only supported under ITS.
ITS and other OSes that used the |IMP| did not support modern protocols and typically required a complete rebuild to change the IP address.
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
:html:`<samp><b>SET IMP DHCP</b></samp>`                                                                Allow the |IMP| to acquire an IP address from the local network via DHCP.
                                                                                                        Only HOST must be set if this feature is used.
:html:`<samp><b>SET IMP NODHCP</b></samp>`                                                              Disable the |IMP| from making DHCP queries
:html:`<samp><b>SET IMP ARP=</b><var>ddd.ddd.ddd.ddd</var><b>=</b><var>xx:xx:xx:xx:xx:xx</var></samp>`  Create a static ARP entry for the IP address with the indicated MAC address
:html:`<samp><b>SET IMP MIT</b></samp>`                                                                 Set the |IMP| to look like the |IMP| used by MIT for ITS
:html:`<samp><b>SET IMP MPX=</b><var>#</var></samp>`                                                    For ITS, set the |MPX| interrupt to channel |#|
:html:`<samp><b>SET IMP BBN</b></samp>`                                                                 Set the |IMP| to behave like generic BBN IMP (not implemented yet)
:html:`<samp><b>SET IMP WAITS</b></samp>`                                                               Set the |IMP| for Stanford-style |IMP| for WAITS
:html:`<samp><b>SHOW IMP ARP</b></samp>`                                                                Display the IP address to MAC address table
======================================================================================================  ===========================================================================

.. |0readme_ethernet.txt| replace:: ``0readme_ethernet.txt``
.. _0readme_ethernet.txt: ../0readme_ethernet.txt

The |IMP| device must be attached to an available Ethernet interface.
To determine which devices are available,
use the :html:`<samp><b>SHOW ETHERNET</b></samp>` command to list the potential interfaces.
Check out the |0readme_ethernet.txt|_ file from the top of the source directory.

The |IMP| device can be configured in several ways.
Ether it can connect directly to an Ethernet port (via TAP),
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
and under ITS is configured into the system at compile-time.
This address should be given to the |IMP| with the command :html:`<samp><b>SET IMP HOST=</b><var>ip</var></samp>`,
the |IMP| will direct all traffic it sees to this address.
If this address is not the same as the address of the system as seen by the network,
then this address can be set with :html:`<samp><b>SET IMP IP=</b><var>ip</var></samp>`,
and :html:`<samp><b>SET IMP GW=</b><var>ip</var></samp>`,
or :html:`<samp><b>SET IMP DHCP</b></samp>`,
which will allow the |IMP| to request an address from a local DHCP server.
The |IMP| will translate the packets it receives/sends to look like they appeared from the desired address.
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
Chaos net was another methods of network access for ITS.
Chaosnet can be connected to another ITS system or a VAX/PDP-11 running BSD Unix or VMS.
Chaosnet runs over UDP protocol under Unix.
You must specify a node number,
peer to talk to,
and your local UDP listening port.
All UDP packets are sent to the peer for further processing.

=======================================================================  ====================================
:html:`<samp><b>SET CH NODE=</b><var>n</var></samp>`                     Set the node number for this system
:html:`<samp><b>SET CH PEER=</b><var>ddd.ddd.ddd.ddd:dddd</var></samp>`  Set the peer address and port number
=======================================================================  ====================================

The |CH| device must be attached to a UDP port number.
This is where it will receive UDP packets from its peer.

Symbolic display and input
==========================
.. |EXAMINE| replace:: :html:`<samp><b>EXAMINE</b></samp>`
.. |DEPOSIT| replace:: :html:`<samp><b>DEPOSIT</b></samp>`

The KL10 simulator implements symbolic display and input.
These are controlled by the following switches to the |EXAMINE| and |DEPOSIT| commands:

======  ===========================================
``-v``  Look up the address via translation,
        will return nothing if address is not valid
``-u``  With the ``-v`` option,
        use user space instead of executive space
``-a``  Display/enter ASCII data
``-p``  Display/enter packed ASCII data
``-c``  Display/enter Sixbit character data
``-m``  Display/enter symbolic instructions
\       Display/enter octal data
======  ===========================================

Symbolic instructions can be of the formats:

* ``Opcode ac,operand``
* ``Opcode operands``
* ``I/O Opcode device,address``

Operands can be one or more of the following in order:

* Optional ``@`` for indirection
* ``+`` or ``-`` to set sign
* Octal number
* Optional (``ac``) for indexing

Breakpoints can be set at real memory addresses.
The PDP-10 supports 3 types of breakpoints:
execution, memory access, and memory modify.
The default is execution.
Adding ``-R`` to the breakpoint command will stop the simulator on access to that memory location,
either via fetch, indirection, or operand access.
Adding ``-w`` will stop the simulator when the location is modified.

The simulator can load RIM files,
SAV files,
EXE files,
WAITS octal DMP files,
and MIT SBLK files.

When instruction history is enabled,
the history trace shows internal status of various registers at the start of the instruction execution.

.. |AC|  replace:: :html:`<samp class="register">AC</samp>`
.. |AR|  replace:: :html:`<samp class="register">AR</samp>`
.. |EA|  replace:: :html:`<samp class="register">EA</samp>`
.. |RES| replace:: :html:`<samp class="register">RES</samp>`
.. |IR|  replace:: :html:`<samp class="register">IR</samp>`

=======  ======================================================================
|PC|     Show the PC of the instruction executed
|AC|     The contents of the referenced AC
|EA|     The final computed effective address
|AR|     Generally the operand that was computed
|RES|    The |AR| register after the instruction was complete
|FLAGS|  The values of the |FLAGS| register before execution of the instruction
|IR|     The fetched instruction followed by the disassembled instruction
=======  ======================================================================

The PDP-10 simulator allows for memory reference and memory modify breakpoints with the ``-r`` and ``-w`` options given to the break command.

OS specific configurations
==========================
TOPS-10
-------
The default configuration supports TOPS-10,
so there is no extra configuration required.
TOPS-10 versions 7 and up require a KL10B processor and the |CTY| to be set to |RSX20|.

TOPS-20
-------
TOPS-20 requires that all |RH| devices be of type RH20.
Depending on version,
it may require a KL10B processor.
Also, the |CTY| must be set to |RSX20|.

ITS
---
To run ITS,
the CPU must be configured with :html:`<samp><b>SET CPU ITS 1024K</b></samp>`,
this will enable the ITS pager.
