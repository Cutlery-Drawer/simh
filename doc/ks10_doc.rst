.. -*- coding: utf-8; mode: rst; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; truncate-lines: t; -*- vim:set et ts=4 ft=rst nowrap:

.. role:: html(raw)
   :format: html

.. |ATTACH|  replace:: :html:`<samp>ATTACH</samp>`
.. |DEPOSIT| replace:: :html:`<samp>DEPOSIT</samp>`
.. |EXAMINE| replace:: :html:`<samp>EXAMINE</samp>`
.. |SHOW|    replace:: :html:`<samp>SHOW</samp>`
.. |device|  replace:: :html:`<samp><var>device</var></samp>`
.. |n|       replace:: :html:`<samp><var>n</var></samp>`

*******************************
Dec PDP-10/KS10 simulator usage
*******************************
:Date: 2022-02-17
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
This was a 36-bit computer that was designed for time-sharing,
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
The final version to make it to market was the KS10 (for Small);
this was a bit-slice version of the PDP-10 which used UNIBUS devices,
which were cheaper then the KL10 devices.

The original operating system for the PDP-6/PDP-10 was just called "Monitor".
It was designed to fit into 6K words.
Around the third release, swapping was added.
The sixth release saw the addition of virtual memory.
Around the fourth release,
it was given the name "TOPS-10".
Around this time, BBN was working on a paging system and implemented it on the PDP-10.
This was called "Tenex".
This was later adopted by DEC and became "Tops-20".

During the mid-60s,
a group at MIT,
who where not happy with how Multics was being developed,
decided to create their own operating system which they called Incompatible Timesharing System "ITS",
was a play on the original project called Compatible Timesharing System "CTSS".
CTSS was implemented by MIT on their IBM 7090 as an experimental system that allowed multiple time-sharing users to co-exist on the same machine running batch processing.
Hence the term Compatable.

Also during the mid-60s,
a group at Stanford Artificial Intelligence Laboratory (SAIL) started with a version of TOPS-10 and heavily modified it to become WAITS.

During the 70s,
Tymshare starting with DEC TOPS-10 system modified it to support random access files,
paging with working sets and spawn-able processes.
This ran on the KI10, KL10 and KS10.

The PDP-10 was ultimately replaced by the VAX.

Simulator files
===============
To compile the DEC 10/KS10 simulator,
you must define ``USE_INT64`` as part of the compilation command-line.

+--------------+-------------------+--------------------------------------------+
| Subdirectory | File              | Contains                                   |
+--------------+-------------------+--------------------------------------------+
| ``PDP10/``   | ``kx10_defs.h``   |  KL10 simulator definitions                |
|              +-------------------+--------------------------------------------+
|              | ``kx10_cpu.c``    |  KL10 CPU                                  |
|              +-------------------+--------------------------------------------+
|              | ``kx10_disk.h``   |  Disk formatter definitions                |
|              +-------------------+--------------------------------------------+
|              | ``kx10_disk.c``   |  Disk formatter routine                    |
|              +-------------------+--------------------------------------------+
|              | ``kx10_sys.c``    |  KS10 system interface                     |
|              +-------------------+--------------------------------------------+
|              | ``kx10_rp.c``     |  RH10/RH20 disk controller, RP04/5/6 disks |
|              +-------------------+--------------------------------------------+
|              | ``kx10_tu.c``     |  RH10/RH20 TM03 magnetic tape controller   |
|              +-------------------+--------------------------------------------+
|              | ``kx10_rh.c``     |  RH11 controller                           |
|              +-------------------+--------------------------------------------+
|              | ``ks10_lp.c``     |  LP 10 line printer                        |
|              +-------------------+--------------------------------------------+
|              | ``ks10_dz.c``     |  DZ11 communications controller            |
|              +-------------------+--------------------------------------------+
|              | ``ks10_cty.c``    |  KS10 front panel                          |
|              +-------------------+--------------------------------------------+
|              | ``ks10_uba.c``    |  KS10 Unibus interface                     |
|              +-------------------+--------------------------------------------+
|              | ``kx10_imp.c``    |  IMP11 interface to Ethernet               |
|              +-------------------+--------------------------------------------+
|              | ``ks10_ch11.c``   |  Chaosnet 11 interface                     |
|              +-------------------+--------------------------------------------+
|              | ``ks10_dup.c``    |  DUP11 interface                           |
|              +-------------------+--------------------------------------------+
|              | ``ks10_dup.h``    |  Header file for DUP11 interface           |
|              +-------------------+--------------------------------------------+
|              | ``pdp11_ddcmp.h`` |  Header to define interface for DECnet     |
|              +-------------------+--------------------------------------------+
|              | ``ks10_kmc.c``    |  KMC communications controller             |
+--------------+-------------------+--------------------------------------------+

KS10 features
=============
The KS10 simulator is configured as follows:

==============  ===================================
Device name(s)  Simulates
==============  ===================================
|CPU|_          KS10 CPU with 256KW of memory
|CTY|_          Console TTY
|LP20|_         LP10 line Printer
|RPA|_          RH10/RH20 disk controllers via RH10
|TUA|_          TM02 tape controller via RH10/RH20
|DZ|_           DC10 communications controller
|IMP|_          IMP network interface
|CH|_           CH10 Chaosnet interface
|DUP|_          DUP11 interface
|KDP|_          KMC11 interface
==============  ===================================

.. |CPU| replace:: ``CPU``

|CPU|
-----
The |CPU| options include setting memory size and O/S customization.

===================================  ===========================================  ===
:html:`<samp>SET CPU 256K</samp>`    Set memory to 256K
:html:`<samp>SET CPU 512K</samp>`    Set memory to 512K
:html:`<samp>SET CPU 1024K</samp>`   Set memory to 1024K
:html:`<samp>SET CPU ITS</samp>`     Add ITS pager and instruction support to KS  ITS
:html:`<samp>SET CPU NOIDLE</samp>`  Disable idle detection
:html:`<samp>SET CPU IDLE</samp>`    Enables idle detection
===================================  ===========================================  ===

CPU registers include the visible state of the processor,
as well as the control registers for the interrupt system.

===============  ====  ==========================
Name             Size  Comments
===============  ====  ==========================
``PC``           18    Program counter
``FLAGS``        18    Flags
``FM0-FM17``     36    Fast memory
``SW``           36    Console SW register
``MI``           36    Monitor display
``PIR``          8     Priority interrupt request
``PIH``          8     Priority interrupt hold
``PIE``          8     Priority interrupt enable
``PIENB``        1     Enable priority system
``BYF5``         1     Byte flag
``UUO``          1     UUO cycle
``NXM``          1     Non-existing memory access
``CLK``          1     Clock interrupt
``OV``           1     Overflow enable
``FOV``          1     Floating overflow enable
``APRIRQ``       3     APR interrupt number
``UB``           22    User base pointer
``EB``           22    Executive base pointer
``FMSEL``        8     Register set selection
``SERIAL``       10    System serial number
``PAGE_ENABLE``  1     Paging system enabled
``PAGE_FAULT``   1     Page fault flag
``FAULT_DATA``   36    Page fault data
``SPT``          18    Special page table
``CST``          18    Memory status table
``PU``           36    User data
``CSTM``         36    Status mask
===============  ====  ==========================

The CPU can maintain a history of the most recently executed instructions.

This is controlled by the :html:`<samp>SET CPU HISTORY</samp>` and :html:`<samp>SHOW CPU HISTORY</samp>` commands:

==================================================  ======================================
:html:`<samp>SET CPU HISTORY</samp>`                Clear history buffer
:html:`<samp>SET CPU HISTORY=<var>0</var></samp>`   Disable history
:html:`<samp>SET CPU HISTORY=<var>n</var></samp>`   Enable history, length = |n|
:html:`<samp>SHOW CPU HISTORY</samp>`               Print CPU history
:html:`<samp>SHOW CPU HISTORY=<var>n</var></samp>`  Print first |n| entries of CPU history
==================================================  ======================================

Instruction tracing shows the program counter,
the contents of AC selected,
the computed effective address.
AR is generally the contents the memory location referenced by EA.
RES is the result of the instruction.
FLAGS shows the flags after the instruction is executed.
IR shows the instruction being executed.

KS10 front end processor
------------------------

.. _Console teletype:
.. _CTY:
.. |CTY| replace:: ``CTY``

Console teletype (|CTY|)
""""""""""""""""""""""""
The console station allows for communications with the operating system.

===============================  ================================
:html:`<samp>SET CTY 7B</samp>`  7-bit characters, space parity
:html:`<samp>SET CTY 8B</samp>`  8-bit characters, space parity
:html:`<samp>SET CTY 7P</samp>`  7-bit characters, space parity
:html:`<samp>SET CTY UC</samp>`  Translate lowercase to uppercase
===============================  ================================

The |CTY| also supports a method for stopping TOPS10 operating system.

Unit record I/O devices
-----------------------

.. _LP20 printer:
.. _LP20:
.. |LP20| replace:: ``LP20``

LP20 printer (|LP20|)
"""""""""""""""""""""
The line printer (|LP20|) writes data to a disk file as ASCII text with terminating newlines.
Currently set to handle standard signals to control paper advance.
Currently, only one line printer device is supported.

=======================================================  ==============================================================
:html:`<samp>SET LP20 LC</samp>`                         Allow printer to print lowercase
:html:`<samp>SET LP20 UC</samp>`                         Print only uppercase
:html:`<samp>SET LP20 LINESPERPAGE=<var>n</var></samp>`  Set the number of lines before an auto form feed is generated.
                                                         There is an automatic margin of 6 lines.
                                                         There is a maximum of 100 lines per page.
:html:`<samp>SET LP20 NORMAL</samp>`                     Allow the VFU to be loading into the printer
:html:`<samp>SET LP20 OPTICAL</samp>`                    VFU is fixed in the printer.
:html:`<samp>SET LP20 ADDR=<var>value</var></samp>`      Set the Unibus address of device (default: 775400)
:html:`<samp>SET LP20 VECT=<var>value</var></samp>`      Set the interrupt vector of device (default: 754)
:html:`<samp>SET LP20 BR=<var>value</var></samp>`        Set the interrupt priority of device (default: 5)
:html:`<samp>SET LP20 CTL=<var>value</var></samp>`       Set the Unibus adapter number (default: 3)
=======================================================  ==============================================================

These characters control the skipping of various number of lines.

=========  =======================
Character  Effect
=========  =======================
``014``    Skip to top of form
``013``    Skip mod 20 lines
``020``    Skip mod 30 lines
``021``    Skip to even line
``022``    Skip to every 3 line
``023``    Same as line feed (12),
           but ignore margin
=========  =======================

Disk I/O devices
----------------
The PDP-10 supported many disk controllers.
The KS10 supports only mass bus devices.

Massbus devices
---------------
Massbus devices are attached via RH11s.

.. |RPA| replace:: ``RPA``
.. _RPA:

RP disk drives
""""""""""""""
The KS10 supports one RH11 controller with up to 8 RP drives can be configured.
These are addresses as RP disks can be stored in one of several file formats,
SIMH, DBD9 and DLD9.
The latter two are for compatibility with other simulators.

====================================================  ==================================================
:html:`<samp>SET RPA<var>n</var> RP04</samp>`         Set this unit to be an RP04 (19MWords)
:html:`<samp>SET RPA<var>n</var> RP06</samp>`         Set this unit to be an RP06 (39MWords)
:html:`<samp>SET RPA<var>n</var> RP07</samp>`         Set this unit to be an RP07 (110MWords)
:html:`<samp>SET RPA<var>n</var> LOCKED</samp>`       Set this unit to be read-only
:html:`<samp>SET RPA<var>n</var> WRITEENABLE</samp>`  Set this unit to be writable
:html:`<samp>SET RPA ADDR=<var>value</var></samp>`    Set the Unibus address of device (default: 776700)
:html:`<samp>SET RPA VECT=<var>value</var></samp>`    Set the interrupt vector of device (default: 254)
:html:`<samp>SET RPA BR=<var>value</var></samp>`      Set the interrupt priority of device (default: 6)
:html:`<samp>SET RPA CTL=<var>value</var></samp>`     Set the Unibus adapter number (default: 1)
====================================================  ==================================================

To attach a disk,
use the |ATTACH| command:

===============================================================================  ===================================
:html:`<samp>ATTACH RPA<var>n</var> <var>file</var></samp>`                      Attach a file
:html:`<samp>ATTACH RA<var>n</var> -f <var>format</var> <var>file</var></samp>`  Attach a file with the given format
:html:`<samp>ATTACH RPA<var>n</var> -n <var>file</var></samp>`                   Create a blank disk
===============================================================================  ===================================

:html:`<samp><var>format</var></samp>` can be ``SIMH``, ``DBD9``, or ``DLD9``.

.. _Tape drives:
.. _TUA:
.. _TU:
.. |TUA| replace:: ``TUA``
.. |TU|  replace:: ``TU``

|TUA| tape drives
"""""""""""""""""
The |TU| is a mass bus tape controller using a TM03 formatter.

====================================================  ==================================================
:html:`<samp>SET TUA<var>n</var> LOCKED</samp>`       Set the mag tape to be read-only
:html:`<samp>SET TUA<var>n</var> WRITEENABLE</samp>`  Set the mag tape to be writable
:html:`<samp>SET TUA ADDR=<var>value</var></samp>`    Set the Unibus address of device (default: 772440)
:html:`<samp>SET TUA VECT=<var>value</var></samp>`    Set the interrupt vector of device (default: 224)
:html:`<samp>SET TUA BR=<var>value</var></samp>`      Set the interrupt priority of device (default: 6)
:html:`<samp>SET TUA CTL=<var>value</var></samp>`     Set the Unibus adapter number (default: 3)
====================================================  ==================================================

Terminal multiplexer I/O devices
--------------------------------
All terminal multiplexers must be attached in order to work.
The ``ATTACH`` command specifies the port to be used for Telnet sessions:

=============================================================  ====================
:html:`<samp>ATTACH <var>device</var> <var>port</var></samp>`  Setup listening port
=============================================================  ====================

where :html:`<samp><var>port</var></samp>` is a decimal number between 1 and 65535 that is not being used other TCP/IP activities.

Once attached and the simulator is running,
the multiplexer listens for connections on the specified port.
It assumes that any incoming connection is a Telnet connection.
The connections remain open until disconnected either by the Telnet client,
a :html:`<samp>SET <var>device</var> DISCONNECT</samp>` command,
or a :html:`<samp>DETACH <var>device</var> command</samp>`.

==================================================================  ===================
:html:`<samp>SET <var>device</var> DISCONNECT=<var>n</var></samp>`  Disconnect line |n|
==================================================================  ===================

The |device| implements the following special |SHOW| commands:

=======================================================  ===========================================
:html:`<samp>SHOW <var>device</var> CONNECTIONS</samp>`  Display current connections to the |device|
:html:`<samp>SHOW <var>device</var> STATISTICS</samp>`   Display statistics for active connections
:html:`<samp>SHOW <var>device</var> LOG</samp>`          Display logging for all lines
=======================================================  ===========================================

Logging can be controlled as follows:

===============================================================================  ==================================================================
:html:`<samp>SET <var>device</var> LOG=<var>n</var>=<var>filename</var></samp>`  Log output of line |n| to :html:`<samp><var>filename</var></samp>`
:html:`<samp>SET <var>device</var> NOLOG</samp>`                                 Disable logging and close log file
===============================================================================  ==================================================================

.. |DZ11| replace:: ``DZ11``
.. |DZ|   replace:: ``DZ``
.. _DZ11:
.. _DZ:

|DZ11| terminal controller
""""""""""""""""""""""""""
The DZ device was the standard terminal multiplexer for the KS10.
Lines came in groups of 8.
A maximum of 32 lines can be supported.

=================================================  ==================================================
:html:`<samp>SET DZ LINES=<var>n</var></samp>`     Set the number of lines on the DZ10, multiple of 8
:html:`<samp>SET DZ ADDR=<var>value</var></samp>`  Set the Unibus address of device (default: 760000)
:html:`<samp>SET DZ VECT=<var>value</var></samp>`  Set the interrupt vector of device (default: 340)
:html:`<samp>SET DZ BR=<var>value</var></samp>`    Set the interrupt priority of device (default: 5)
:html:`<samp>SET DZ CTL=<var>value</var></samp>`   Set the Unibus adapter number (default: 3)
=================================================  ==================================================

Network devices
---------------

.. |IMP| replace:: ``IMP``
.. _IMP:

IMP interface message processor
"""""""""""""""""""""""""""""""
This allows the KS to connect to the Internet.
Currently only supported under ITS.
ITS and other OSes that used the |IMP| did not support modern protocols and typically required a complete rebuild to change the IP address.
Because of this,
the |IMP| processor includes built-in NAT and DHCP support.
For ITS, the system generated IP packets which are translated to the local network.
If the HOST is set to ``0.0.0.0``,
there will be no translation.
If HOST is set to an IP address,
then it will be mapped into the address set in IP.
If DHCP is enabled,
the |IMP| will issue a DHCP request at startup and set IP to the address that is provided.
DHCP is enabled by default.

========================================================================================  =================================================
:html:`<samp>SET IMP MAC=<var>xx:xx:xx:xx:xx:xx</var></samp>`                             Set the MAC address of the |IMP| to the value given
:html:`<samp>SET IMP IP=<var>ddd.ddd.ddd.ddd</var>/<var>dd</var></samp>`                  Set the external IP address of the |IMP| along with the net mask in bits
:html:`<samp>SET IMP GW=<var>ddd.ddd.ddd.ddd</var></samp>`                                Set the Gateway address for the |IMP|
:html:`<samp>SET IMP HOST=<var>ddd.ddd.ddd.ddd</var></samp>`                              Sets the IP address of the PDP-10 system
:html:`<samp>SET IMP DHCP</samp>`                                                         Allow the |IMP| to acquire an IP address from the local network via DHCP. Only HOST must be set if this feature is used
:html:`<samp>SET IMP NODHCP</samp>`                                                       Disable the |IMP| from making DHCP queries
:html:`<samp>SET IMP ARP=<var>ddd.ddd.ddd.ddd</var>=<var>xx:xx:xx:xx:xx:xx</var></samp>`  Create a static ARP entry for the IP address with the indicated MAC address
:html:`<samp>SET IMP ADDR=<var>value</var></samp>`                                        Set the Unibus address of device (default: 767600)
:html:`<samp>SET IMP VECT=<var>value</var></samp>`                                        Set the interrupt vector of device (default: 250)
:html:`<samp>SET IMP BR=<var>value</var></samp>`                                          Set the interrupt priority of device (default: 6)
:html:`<samp>SET IMP CTL=<var>value</var></samp>`                                         Set the Unibus adapter number (default: 3)
:html:`<samp>SHOW IMP ARP</samp>`                                                         Display the IP address to MAC address table
========================================================================================  =================================================

.. |0readme_ethernet.txt| replace:: ``0readme_ethernet.txt``
.. _0readme_ethernet.txt: ../0readme_ethernet.txt

The |IMP| device must be attached to an available Ethernet interface.
To determine which devices are available,
use the :html:`<samp>SHOW ETHERNET</samp>` command to list the potential interfaces.
Check out the |0readme_ethernet.txt|_ file from the top of the source directory.

The |IMP| device can be configured in several ways.
Ether it can connect directly to an ethernet port (via TAP),
or it can be connected via a TUN interface.
If configured via TAP interface,
the |IMP| will behave like any other Ethernet interface and if asked,
grab its own address.
In environments where this is not desired,
the TUN interface can be used.
When configured under a TUN interface,
the simulated network is a collection of ports on the local host.
These can be mapped based on configuration options;
see the |0readme_ethernet.txt|_ file as to options.

With the |IMP| interface,
the IP address of the simulated system is static,
and under ITS,
is configured into the system at compile time.
This address should be given to the |IMP| with the :html:`<samp>set imp host=<var>ip</var></samp>` command;
the |IMP| will direct all traffic it sees to this address.
If this address is not the same as the address of the system as seen by the network,
then this address can be set with :html:`<samp>set imp ip=<var>ip</var></samp>`,
and :html:`<samp>set imp gw=<var>ip</var></samp>`,
or :html:`<samp>set imp dhcp</samp>` which will allow the |IMP| to request an address from a local DHCP server.
The |IMP| will translate the packets it receives/sends to look like the appeared from the desired address.
The |IMP| will also correctly translate FTP requests in this configuration.

When running under a TUN interface,
|IMP| is on a virtual ``10.0.2.0`` network.
The default gateway is ``10.0.2.1``,
with the default |IMP| at ``10.0.2.15``.
For this mode,
DHCP can be used.

.. |CH11| replace:: ``CH11``
.. |CH|   replace:: ``CH``
.. _CH11:
.. _CH:

|CH11| Chaosnet interface
"""""""""""""""""""""""""
Chaosnet was another method of network access for ITS.
Chaosnet can be connected to another ITS system or a VAX/PDP-11 running BSD Unix or VMS.
Chaosnet runs over UDP protocol under Unix.
You must specify a node number,
peer to talk to,
and your local UDP listening port.
All UDP packets are sent to the peer for further processing.

================================================================  ==================================================
:html:`<samp>SET CH NODE=<var>n</var></samp>`                     Set the Node number for this system
:html:`<samp>SET CH PEER=<var>ddd.ddd.ddd.ddd:dddd</var></samp>`  Set the Peer address and port number
:html:`<samp>SET CH ADDR=<var>value</var></samp>`                 Set the Unibus address of device (default: 764140)
:html:`<samp>SET CH VECT=<var>value</var></samp>`                 Set the interrupt vector of device (default: 270)
:html:`<samp>SET CH BR=<var>value</var></samp>`                   Set the interrupt priority of device (default: 6)
:html:`<samp>SET CH CTL=<var>value</var></samp>`                  Set the Unibus adapter number (default: 3)
================================================================  ==================================================

The |CH| device must be attached to a UDP port number.
This is where it will receive UDP packets from its peer.

.. |KMC11| replace:: ``KMC11``
.. |KDP|   replace:: ``KDP``
.. _KMC11:
.. _KDP:

|KMC11| communications controller
"""""""""""""""""""""""""""""""""
The KMC11-A is a general purpose microprocessor that is used in several DEC products.
The |KDP| is an emulation of one of those products: COMM IOP-DUP.
The COMM IOP-DUP microcode controls and supervises 1 - 16 |DUP-11| synchronous communications line interfaces,
providing scatter/gather DMA,
message framing,
modem control,
CRC validation,
receiver resynchronization,
and address recognition.
The |DUP-11| lines are assigned to the |KMC11| by the (emulated) operating system,
but SIMH must be told how to connect them.

.. |DUP-11| replace:: ``DUP-11``
.. |DUP11|  replace:: ``DUP11``
.. |DUP|    replace:: ``DUP``
.. _DUP11:
.. _DUP:

|DUP11| network interface
"""""""""""""""""""""""""
|DUP11| devices are used by DECnet to link machines together.
They are a synchronous serial interface.
This is implemented by UDP to send packets to the remote machine.

=====================================================================  ======================================================
:html:`<samp>SET DUP LINES=<var>n</var></samp>`                        Set number of DUP-11 lines supported; current max is 2
:html:`<samp>SET DUP<var>n</var> SPEED=<var>bits/sec</var></samp>`     Set speed of link, default of 0 means no restrictions
:html:`<samp>SET DUP<var>n</var> CORRUPTION=<var>factor</var></samp>`  Specify corruption factor
:html:`<samp>SET DUP<var>n</var> W3</samp>`                            Enable Reset option
:html:`<samp>SET DUP<var>n</var> NOW3</samp>`                          Disable reset option
:html:`<samp>SET DUP<var>n</var> W5</samp>`                            Enable A dataset control option
:html:`<samp>SET DUP<var>n</var> NOW5</samp>`                          Disable A dataset control option
:html:`<samp>SET DUP<var>n</var> W6</samp>`                            Enable A & B dataset control option
:html:`<samp>SET DUP<var>n</var> NOW6</samp>`                          Disable A & B dataset control option
:html:`<samp>SET DUP ADDR=<var>value</var></samp>`                     Set the Unibus address of device (default: 760300)
:html:`<samp>SET DUP VECT=<var>value</var></samp>`                     Set the interrupt vector of device (default: 570)
:html:`<samp>SET DUP BR=<var>value</var></samp>`                       Set the interrupt priority of device (default: 5)
:html:`<samp>SET DUP CTL=<var>value</var></samp>`                      Set the Unibus adapter number (default: 3)
=====================================================================  ======================================================

The |DUP| device must be attached to a UDP port number.
This is where it will receive UDP packets from its peer.

.. _ATTACH:

Attach
''''''
The communication line performs input and output through a TCP session (or UDP session) connected to a user-specified port.
The |ATTACH| command specifies the port to be used as well as the peer address::

    sim> ATTACH DUP0 {interface:}port{,UDP},Connect=peerhost:port

where ``port`` is a decimal number between 1 and 65535 that is not being used for other TCP/IP activities.

Specifying symmetric attach configuration
(with both a listen port and a peer address)
will cause the side receiving an incoming connection to validate that the connection actually comes from the connecction destination system.

A symmetric attach configuration is required when using UDP packet transport.

The default connection uses TCP transport between the local system and the peer.
Alternatively,
UDP can be used by specifying UDP on the |ATTACH| command.

For more connection options,
view the :html:`<samp>Help attach</samp>` command for the |DUP|.

Symbolic display and input
==========================
The KL10 simulator implements symbolic display and input.
These are controlled by the following switches to the |EXAMINE| and |DEPOSIT| commands:

======  ===============================================================================
``-v``  Lookup the address via translation, will return nothing if address is not valid
``-u``  With the ``-v`` option, used user space instead of executive space
``-a``  Display/enter ASCII data
``-p``  Display/enter packed ASCII data
``-c``  Display/enter six-bit character data
``-m``  Display/enter symbolic instructions
\       Display/enter octal data
======  ===============================================================================

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
The PDP-10 supports 3 types of breakpoints: Execution, Memory Access, and Memory Modify.
The default is execution.
Adding ``-R`` to the breakpoint command will stop the simulator on access to that memory location,
either via fetch, indirection or operand access.
Adding ``-w`` will stop the simulator when the location is modified.

The simulator can load RIM files,
SAV files,
EXE files,
Waits Octal DMP files,
and MIT SBLK files.

When instruction history is enabled,
the history trace shows the internal status of various registers at the start of the instruction execution.

=========  ========================================================================
``PC``     Show the PC of the instruction executed
``AC``     The contents of the referenced AC
``EA``     The final computed EA (effective address)
``AR``     Generally the operand that was computed
``RES``    The AR register after the instruction was complete
``FLAGS``  The values of the ``FLAGS`` register before execution of the instruction
``IR``     The fetched instruction followed by the disassembled instruction
=========  ========================================================================

The PDP-10 simulator allows for Memory reference and memory modify breakpoints with the ``-r`` and ``-w`` options given to the break command.

OS specific configurations
==========================
ITS
---
To run ITS,
the |CPU| must be :html:`<samp>set cpu ITS</samp>`;
this will enable the ITS pager.
