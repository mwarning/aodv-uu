# AODV-UU

## Introduction

This is an AODV implementation developed at Uppsala University,
Sweden, with some funding from Ericsson AB. It has been developed
mainly for use in the [APE testbed](http://apetestbed.sourceforge.net).
The code is released under the GNU General Public License (GPL). See
the GPL document for more information.

This release is based on AODV draft version 13. There are no
guarantees that it implements all features correctly, although this is
the goal. The code is provided as is. See the CHANGELOG for updates
and changes between releases.

This AODV implementation runs as a user-space daemon, maintaining the
kernel routing table. Netfilter is used to capture data
packets. Filtering is done in user-space, so there may be some
performance penalties, although coding is much simplified. Stable
operation has higher priority than performance.

The code has been successfully tested in a real ad-hoc environment
using up to 5 nodes (4 hops) without problems. It has been debugged in
ns-2 by running extensive simulations. The performance is usually on
par or better than the AODV version in ns-2. It has also been interop
tested with great results. If you happen to experience less successful
operation of this implementation, please contact the author(s) and
describe your problems.

## Requirements

Real world:
* Linux based OS (>= 3.6.0).
* Wireless LAN cards in ad-hoc/mesh mode (alternatively a wired setup can
  be used).

ns-2:
* See README.ns

## Installation

If you are running AODV-UU in NS-2, then you should read README.ns for
install instructions.

Make sure you have the kernel source (or at least headers) of the
kernel you are compiling against installed. Otherwise the kernel
modules might not compile. See the troubleshooting section if there
are problems.

Compile with `make`:

```
> make
```

Install (as "root"):

```
> make install
```

Run (as "root" with recommended options for debugging):

```
> aodvd -l -r 3
```

For command line options, run:

```
> aodvd --help
```

The following module must be loaded when running (or compiled into
the kernel):
* kaodv.{o,ko}

Module loading should happen automatically if AODV is installed and
the module loading system (modprobe) is properly configured.

## Debug output

To get debug output, make sure the daemon is compiled with the -DDEBUG
option set (check Makefile). Debug information is written to
/var/log/aodvd.log if the AODV is run with the "-l" flag:

```
> aodvd -l
```

This is the same output as written to STDOUT if running the daemon in
the foreground. To get printouts of the AODV internal routing table,
run AODV with:

```
> aodvd -r 2.5
```

where the number is the interval between routing table printing, in seconds.
The routing table is written to /var/log/aodvd.rtlog.

## Note about Local Repair

As of version 0.6 of AODV-UU, local repair is fully implemented.
However, please be aware of the fact that local repair does not always
help performance, it may in fact hurt it. Consider turning local
repair off if this is not a feature you are interested in.

## Note about HELLO messages

This implementation rely on HELLO messages. However, it has been
found, through real world testing, that HELLO messages are not a good
way to do neighbor sensing in a wireless environment (at least not
over 802.11). Therefore, you may experience bad performance when
running over wireless. There are several reasons for this:

* HELLO messages are broadcasted. In 802.11, broadcasting is done at a
lower bit rate than unicasting, thus HELLO messages travel further
than data.

* HELLO messages are small, thus less prone to bit errors than data
transmissions.

* Broadcast transmissions are not guaranteed to be bidirectional,
unlike unicast transmissions.

## Running a test

To test the basic functionality of AODV you need at least three
computers configured to run AODV. The nodes IP-address configuration
should be in the same subnet. Then try to place the computers so that
two of them are out of each others transmission range with the third
computer in the middle, as in the illustration below. It may also be
convenient to use a MAC-filter, like that part of the APE testbed,
http://apetestbed.sourceforge.net.

A <-> B <-> C

Run on either A or C:

```
> ping -R <IP A or C>
```

to ping the remote computer. The "-R" option will record the route
taken by ping packets, so that the actual route taken can be seen.

## Unidirectional links

This AODV implementation can detect the presence of unidirectional
links, and avoid them if necessary. It is done by sending a RREP
extension along with the hello messages containing the neighbor set of
a node. This functionality is not part of the AODV draft as of version
10, but similar functionality may be in future
versions. Unidirectional link detection can be enabled with the "-u"
option. This feature is experimental and may be BROKEN in any release.

## Internet gateway support

As of v0.8, AODV-UU implements gateway support by tunneling packets to
gateway configured nodes. This is much more robust than a default
route solution or similar. 

AODV-UU must be compiled with "CONFIG_GATEWAY" defined (see
Makefile). Nodes that run "aodvd" with the -w flag will automatically
act as gateways - thus answering to RREQs according to an "address
locality" decision, i.e., RREQs for addresses that a gateway thinks
lie outside the ad hoc network will generate a (proxy) RREP. This RREP
contains a special extension that automatically sets up a tunnel
between the source node and the gateway. Gateways currently implement
"address locality" through a prefix check, thus the ad hoc network
must share a prefix (e.g., `192.168.0.0/16`). Other "locality checks"
can easily be implemented in locality.c. Tunnels (i.e., routes to
gateways) are marked with a "G" flag in the routing table, while
tunnel entries (i.e., Internet destinations) are marked with and "I"
flag.

In case the ad hoc network does not use a globally valid prefix (or
runs Mobile IP or similar), gateways should also have NAT enabled:

```
> /sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Change eth0 to the name of the interface connected to the Internet if
necessary.

On the ad hoc nodes it is also necessary to add a default route in the
kernel routing table, pointing to the ad hoc interface. For example, if
the wireless ad hoc interface is eth1:

```
> route add default dev eth1
```

Otherwise, it will not be possible to communicate with destinations
outside the ad hoc prefix.

## Issues & Troubleshooting

* If a crash occurs, the kernel module `kaodv.o` may remain loaded and
can stop traffic from going through on the interface. Unload with
`/sbin/rmmod kaodv` (root permissions required).

* If the daemon refuse to start and complains about ipchains, make
sure that the `ipchains` compatibility kernel module is not loaded.
It will conflict with iptables. Do `ipchains -F` followed by `modprobe
-r ipchains` to unload it.

* For routing between nodes with arbitrary subnet addresses the
default gateway in the kernel routing table must point to the node
itself. Otherwise communication with addresses on a foreign subnet
will not be possible, since the kernel will complain that there is no
route available. Setting this gateway is typically done with the
command:

```
> route add default dev <wireless iface e.g., "eth1">
```

## Contact

Source code and implementation questions:
Erik Nordstr�m <erik.nordstrom@it.uu.se>

NS port questions:
Bj�rn Wiberg <bjorn.wiberg@home.se>

Misc. questions:
Henrik Lundgren <henrikl@docs.uu.se>
