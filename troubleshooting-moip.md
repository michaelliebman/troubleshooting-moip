---
title: Media IP Troubleshooting Tips and Tricks
author: Michael S. Liebman, CEV, CBNT
institute: SiriusXM
date: April 12, 2024
bibliography: troubleshooting-moip.bib
csl: ieee-with-url
link-citations: true
link-bibliography: true
monofont: Fira Code
---

::: notes

Good morning. I want to quickly introduce myself. I'm a Principal Engineer on
SirusXM's Media Content Engineering architecture team, where we design linear
and file-based media technology systems that power the audio, metadata, and
video content on all the SirusXM and Pandora distribution platforms.

:::

### Topics

:::::::::::::: {.columns}
::: {.column width="50%"}

* The Troubleshooting Process & Getting Ready
* Hardware Tools
* Software Techniques

:::
::: {.column width="50%"}

![[^credit-fast]](https://source.unsplash.com/xzLKOczi4As "Title: Time laps of cars on a road in Thailand at night")

[^credit-fast]: Photo by [jet dela cruz](https://unsplash.com/@jetjetdelacruz)
on [Unsplash](https://unsplash.com/photos/time-lapse-photography-of-cars-on-road-during-night-time-xzLKOczi4As)

:::
::::::::::::::

::: notes

We're going to cover a lot of material in a short time, so hold on tight! First,
we'll review the troubleshooting process and what you can do to get ready to
work on MoIP systems. Then we will briefly look at the hardware tools network
engineers use before going through a bunch of tips and tricks for the software
we use for troubleshooting.

:::

## The Troubleshooting Process & Getting Ready

::: notes

Let's get to it...

:::

### It's not that different

![from traditional media systems![^credit-kozk]](images/process/robert-linder-uizuCGhJ1Rw-unsplash.jpg "TV Station KOZK control room in 1975")

[^credit-kozk]: Photo by [Robert Linder](https://unsplash.com/@rwlinder) on [Unsplash](https://unsplash.com/photos/a-desk-with-a-bunch-of-electronic-equipment-on-top-of-it-uizuCGhJ1Rw)

::: notes

The first tip is to remember that troubleshooting MoIP systems is not that
different from working on traditional media systems. The process you follow and
the techniques you use will feel familiar to you and build on the skills you
already have.

:::

### Check the cable

![Network Patch Bay[^credit-cable]](images/process/jordan-harrison-40XgDxBfYXM-unsplash.jpg)

[^credit-cable]: Photo by [Jordan Harrison](https://unsplash.com/@jordanharrison) on [Unsplash](https://unsplash.com/photos/blue-utp-cord-40XgDxBfYXM)

::: notes

It *nearly* goes without saying that when troubleshooting Media over IP systems,
you should not overlook basic checks like cables or turning software switches
on.

:::

### Step 0: Prepare

:::::::::::::: {.columns}
::: {.column width="50%"}

#### Know *your* network

* DNS servers
* NTP & PTP servers
* Typical `traceroute`
* Firewalls & VLANs
* Use DCIM & IPAM like [NetBox](https://netbox.dev/)

#### Install applications

:::
::: {.column width="50%"}

#### Bootable tool kit

[Ventoy](https://www.ventoy.net/) with:

* [Clonezilla](https://clonezilla.org/)/[Rescuezilla](https://rescuezilla.com/)
* [Gandalf's Windows 10PE](https://www.fcportables.com/gandalf-boot-iso/)
* [MS DaRT 10](https://learn.microsoft.com/en-us/microsoft-desktop-optimization-pack/dart-v10/)
* [Memtest86](https://www.memtest86.com/)
* [Rescatux](https://www.supergrubdisk.org/rescatux/)
* [UltimateBootCD](https://www.ultimatebootcd.com/)

:::
::::::::::::::

::: notes

Before you start troubleshooting MoIP (or other) systems, you should be
prepared with the knowledge and tools for the situation.

* Know your network's DNS and time servers
* Know the usual route hops you see on your network (more on `traceroute` later)
* Know where the firewalls are and the VLAN structure
* Use a data center information management (DCIM) and IP address management
  (IPAM) system like NetBox
* Install any software you need --- and there's a *bunch* we are going to cover
  --- before you need it
* Consider preparing a USB bootable tool kit. Ventoy makes it easy to not have
  to pick one. Instead, you can fill a big thumb drive with a variety, like the
  ones listed here.

:::

### Practice Makes Perfect

:::::::::::::: {.columns}
::: {.column width="50%"}

* Use [GNS3](https://www.gns3.com/) or [EVE-NG](https://www.eve-ng.net/) for
hands-on lab time
* Approach from another angle:

  [J. F. Kurose and K. W. Ross, Computer
networking: A top-down approach, Eighth edition. Hoboken: Pearson,
2021.](https://gaia.cs.umass.edu/kurose_ross/index.php)

:::
::: {.column width="50%"}

![Using GNS3 to simulate a spine-leaf architecture](images/process/gns3.png)

:::
::::::::::::::

::: notes

We all now how to get to Carnegie Hall, right? But how do you practice with MoIP
systems? Network simulators can be a little tricky to set up and resource hungry
once running, but you can get hands-on experience with major manufacturer
switches in realistic topologies. GNS3 and EVE-NG have open source or free
community editions available.

Manufacturers have different policies on using their hardware OS in a simulator.
Some require an active support contract to get access to downloads, and others
offer simulator-specific versions for training purposes. When trying to learn
outside of work, you may need to substitute open source components, like the
OpnSense firewall or the Open vSwitch virtual software switch, for commercial
components you don't have access to. In this example, I started building a
spine-and-leaf network with simulated Arista switches on an Intel NUC-style
computer hosting the GNS3 VM. I was able to test basic connectivity with ease,
but I was probably pushing the limits of my hardware if I tried to build a fully
redundant red/blue network.

Most network courses and texts start at the bottom of the OSI model and work
their way up. Kurose and Ross take the rare, opposite approach of starting with
the application layer and working down. This can be more relatable to new
network engineers. (Note that this most recent edition is only available as a
physical or digital rental or a digital purchase.) Older editions are available
used, and I've learned something from every edition I've looked at.

:::

### Troubleshooting Methods

:::::::::::::: {.columns}
::: {.column width="50%"}

#### CompTIA [@comptiaGuideNetworkTroubleshooting]

1. Identify the problem.
2. Develop a theory.
3. Test the theory.
4. Create an action plan.
5. Implement the solution.
6. Document the issue.

:::
::: {.column width="50%"}

#### DECSAR Method [@rossTeachingStructuredTroubleshooting2009]

1. Define the problem.
2. Examine the situation.
3. Consider the causes.
4. Consider the solution.
5. Act and test.
6. Review troubleshooting.

:::
::::::::::::::

::: notes

Let's look at the steps we'll follow for troubleshooting networked systems. Note
that the network-specific CompTIA method is nearly identical to the generic
DECSAR method. We'll identify the problem, work up a theory about the cause,
test that theory, create a plan for correcting the issue, put that plan in
motion, and close out the documentation we've been keeping throughout the
process.

:::

### Troubleshooting Model

``` {.mermaid width=1600}
flowchart LR
  classDef sub opacity:0
  classDef note fill:#fff, stroke:#fff, font-size:24pt

  subgraph notes [" "]
    OSI
    tcp(TCP/IP)
  end
  subgraph Check the Application
    direction TB
    Application --- Presentation --- Session
    app2[Application]
  end
  OSI -.-> Application
  tcp  -.-> app2
  Session --- Transport
  app2 --- trans2[Transport]
  app2 ~~~ Session
  subgraph Check the Protocol Stack
    direction TB
    Transport --- Network --- ll(Logical Link)
    trans2 --- Internet
    Internet --- na(Network Access)
  end
  ll --- Physical
  na --- phys2[Physical*]
  subgraph Check the Cable/NIC
    Physical
    phys2
  end
  class notes sub
  class OSI,tcp note
```

::: notes

I find it useful to translate the formal OSI and TCP/IP models for networks into
a troubleshooting model. Think of it as translator for between what you see in
the wild and what you learn. Having the different perspectives helps with the
theorizing steps that we just looked at.

The upper three layers of the OSI model roughly correspond to problems or fixes
in the application that you are troubleshooting. There's disagreement on whether
the TCP/IP model includes the physical layer, but nothing works without it.
That's where our cable checks come in. In the middle are the three layers that
point you towards looking at the device protocol stack. Is the network mask
correct? Are you getting the correct DNS answer? You get the idea of the clues
that point here.

:::

## Hardware Tools

::: notes

We've prepared ourselves, now what are some hardware tools that can help with
troubleshooting?

:::

### Cable Testers

:::::::::::::: {.columns}
::: {.column width="50%"}

![Klein Tools Cable Tester](images/tools/hardware/continuity.jpg)

:::
::: {.column width="50%"}

:heavy_check_mark: Continuity

:heavy_check_mark: Pinout

:x: Connectivity

:x: Crosstalk, attenuation, interference...

:x: Cable break location

:::
::::::::::::::

::: notes

Cable testers will verify continuity on the conductors and tell you if the
pinout on both ends matches. Basic testers, like this Klein Tools, will *not*
tell you if a cable will work with certainty or at a specific speed.

:::

### Network Testers

:::::::::::::: {.columns}
::: {.column width="50%"}

![Fluke Networks LinkIQ+](images/tools/hardware/better-continuity.jpg)

:::
::: {.column width="50%"}

:heavy_check_mark: Continuity

:heavy_check_mark: Pinout

:heavy_check_mark: Connectivity

:x: Crosstalk, attenuation, interference...

:x: Cable break location

:::
::::::::::::::

::: notes

A step-up in price, network testers will actually verify connectivity and line
rate of a connection. Features vary, but most network testers don't tell you the
quality of the cable (crosstalk, attenuation, interference, etc.) or tell you
where a break in a cable is.

:::

### Certifiers

:::::::::::::: {.columns}
::: {.column width="50%"}

![Fluke Networks DSX CableAnalyzer](images/tools/hardware/certifier.jpg)

:::
::: {.column width="50%"}

:heavy_check_mark: Continuity

:heavy_check_mark: Pinout

:heavy_check_mark: Connectivity

:heavy_check_mark: Crosstalk, attenuation, interference...

:x: Cable break location

:::
::::::::::::::

::: notes

Cable certifiers are often a corporate purchase and bought as part of a testing
kit. There are lots of accessories and add-ons needed to cover the cable types
you want to test. Again, features vary, but certifiers will give you statistics
for the quality of a cable and features for managing bulk cable tests.

:::

### TDRs & OTDRs

:::::::::::::: {.columns}
::: {.column width="50%"}

![EXFO MaxTester 715D](images/tools/hardware/otdr.jpg)

:::
::: {.column width="50%"}

:heavy_check_mark: Continuity

:heavy_check_mark: Pinout

:heavy_check_mark: Connectivity

:heavy_check_mark: Crosstalk, attenuation, interference...

:heavy_check_mark: Cable break location

:information_source:  Have & [know how to use](https://www.bicsi.org/docs/default-source/conference-presentations/2017-winter/launch-cables.pdf?sfvrsn=b1af814c_2)
a launch fiber/box

:::
::::::::::::::

::: notes

Time domain reflectometers and their optical cousins are the ultimate physical
layer troubleshooting tool. They can help you locate cable breaks and passive
network components like patches. But they are *expensive* and make sure you have
all the accessories you need, like a launch box, before you need to use your
OTDR.

:::

### Other Hardware Tools

:::::::::::::: {.columns}
::: {.column width="50%"}

* Wire stripper
* Crimp tool
* Punch down tool
* Optical Power Meter (aka light meter)
* Loopback adapter
* Tone generator

:::
::: {.column width="50%"}

![ ](https://i.redd.it/m61xhejx90d51.jpg "Title: The messiest racks you have ever seen")

:::
::::::::::::::

::: notes

Most of the other tools you need for working on professional media networks are
probably already in your toolkit.

* We all have a preferred style of wire stripper.
* RJ45 crimp tools come in a variety of styles. I'm sure people get religious
  about their preferences, so find one that is right for you.
* 110 punch down tools are important for patch bays and keystone jacks. Make
  sure you understand which way the cut side should face.
* An optical power meter or a visual fault locator are the optical equivalents
  of a continuity tester.
* I haven't used a loopback adapter often, but I am glad that I always put it
  back in the same pocket in my tool bag.
* Tone generators get a workout when the electricians labelled all the cables
  "1".

And I hope we've all learned how to build better racks than these.

:::

## Software Techniques

::: notes

Let's turn our attention to tips and tricks for software-based troubleshooting
techniques.

:::

### Check the deets (Windows)

#### `ipconfig`

* `/all`: Prints everything
* `/release` & `/renew`: use wildcards for interface
* `/flushdns`: Force clearing DNS cache

#### `netsh`

* Print all IPv4 network info: `netsh interface ipv4 show config`
* Save config: `netsh int dump > mycfg.dat`
* Restore config: `netsh exec mycfg.dat`
* Work with a remote machine: `netsh set machine <remotecomputer>`

::: notes

Unless you have a remarkable memory of a small network, you are going to want to
find out how the network is configured on a computer.

On Windows, you do that with `ipconfig`, usually with the `/all` switch. That
will print out the IP address, network mask, default router, DNS servers, and
more for every network interface. You can also `/release` or `/renew`
DHCP-assigned addresses, which you can limit to specific interfaces. Windows
interface names are long, so wildcards save some typing. Sometimes you need to
clear the DNS cache with `/flushdns` to get a new record that has an unexpired
TTL.

If you need to script setting or getting the configuration, you can use `netsh`.
Like with most switch or router command line interfaces, you can shorten
subcommands for `netsh`. If you want to list all the network information, like
you can do with `ipconfig`, you can shorten the command to `netsh int ip sho
conf`. `netsh` has an interactive mode and allows you to set, as well as get,
the configuration. Even more useful, you can `dump` the configuration, make
changes and restore the saved configuration. You can even make changes to
another computer. Be careful that you don't make that remote computer
unreachable.

:::

### Just the facts (Linux)

#### `ip` & `ifconfig`

* `ip` combines `ifconfig` with `route` & `arp`
* `ip` has a simpler, more consistent interface [@kenlonIpVsIfconfig2023]
* `ip a`: list all addresses
* `ip a show up`: list addresses on active interfaces
* `ip -ts mon`: monitor network changes

#### DHCP release/renew

:warning: Releasing/renewing an IP address always brings down the interface.

* `sudo dhclient -r && sudo dhclient`
* `sudo dhcpcd -k && sudo dhcpcd`
* `sudo networkctl renew wlo1`
* `sudo pump -r && sudo pump -R`

::: notes

On Linux, older documentation talks about `ifconfig` (interface config) for
getting most of the same information as Windows' `ipconfig`. But newer
distributions ship with the `ip` utility. You will find it easier to use, plus
it combines information about the `arp` cache and `route` table that you need to
run separately from `ifconfig` and `ipconfig`. You still need to use other
utilities for DHCP and DNS.

Like with `netsh`, you can shorten subcommands for `ip`. Listing all IP
addresses is as quick as `ip a`. If you want to filter the address list, say to
print active interfaces, add `show` plus the query, in this case `up`. Keep
track of changes to the network connections with `ip mon`. Adding `-ts` gives
you a scroll-friendly timestamp.

The exact procedure for releasing and renewing DHCP leases on Linux depends on
the distribution. One of these commands should do the trick. Make sure you have
a backup plan for accessing the device if you are working on it remotely.

:::

### Finding your way

#### Address Resolution Protocol

* Print table
  * Windows: `arp -a`
  * Linux: `ip n`
* Add an entry
  * Windows: `arp -s 192.0.2.5 00-a0-b0-c0-d0-e0`
  * Linux: `sudo ip n a 192.0.2.5 ll 00:a0:b0:c0:d0:e0 dev eth0`

#### Static routes

* Print table
  * Windows: `route print`
  * Linux: `ip r`
* Add a route
  * Windows: `route add 198.51.100.0 mask 255.255.255.0 192.0.2.1 METRIC 3 IF 1`
  * Linux: `sudo ip r a 198.51.100.0/24 via 192.0.2.1 dev eth0`

::: notes

When everything is working right, layer 2 to layer 3 handoffs happen without
needing a second thought. But every once in a while, we need to take a look at
the Address Resolution Protocol table. You can do that with `arp -a` on Windows
or `ip n` on Linux.

Even less often, you need to add a table entry, when you, for example change the
IP address of a remote computer to something inaccessible, and you are desperate
to not have to go into the office at 2am. That's something that I've definitely
never ever done. On Windows, add an ARP entry by entering the target IP address
and the matching MAC address separating the octets with hyphens. On Linux, make
sure you put `lladdr`, or `ll` for short, between the IP and MAC addresses and
specify the interface.

In professional media systems, you may need to work with static routes on
endpoint devices if you separate multicast and unicast on different virtual
networks or if you have red & blue networks. Print the route table on Windows
with `route print` and on Linux with `ip r`. Setting temporary static routes on
Windows means entering the destination network with its network mask in dotted
decimal form, the gateway, the cost metric, and the interface. Note that `route`
uses interface numbers that aren't used in any of the other commands we've
looked out. Printing the route table also includes the interface number mapping.

On Linux, the command is more straightforward. You enter the destination in CIDR
notation, include the word `via` to identify the gateway, and enter the standard
interface name.

:::

### Give it your best Sean Connery

:::::::::::::: {.columns}
::: {.column width="50%"}

#### `ping`

* Test reachability of a host
* Reports dropped packets and round trip time
* `-t` (Windows): infinite packets
* `-4`: force IPv4

#### `tracert` (Windows)/`traceroute` (Linux)

* Test each route hop to a host
* `-h` (Windows)/`-m` (Linux): max hops

:::
::: {.column width="50%"}

![ ](https://i.imgflip.com/4fakj6.jpg "Title:Sean Connery in Hunt for Red October")

:::
::::::::::::::

::: notes

After checking the configuration of a problem computer, you will want to check
if you can reach another device. `ping` uses ICMP to send an **echo request**,
wait for an **echo reply**, and measure the round trip time. The RTT tells you
the latency of a connection. If you ignore Captain Ramius's order and add `-t`
to your Windows ping, you can measure RTT over time and look for intermittent
dropped packets. Linux defaults to infinite pings. Use [Ctrl+C]{.smallcaps} to
end the ping and show the calculated statistics. If you are pinging by name, in
particular outside your own network, `ping` may default to an IPv6 address. Use
`-4` to force pinging the target's IPv4 address.

Each "hop" on the path between devices can be the cause of dropped packets or
high latency. `traceroute`/`tracert` manipulate the time-to-live (TTL) to ping
*router* hops one by one. Note the emphasis on routers there. You can't directly
measure the impact of any switches or other layer 2 devices with `ping` or
`traceroute`. Router configurations may prevent a traceroute from ever
finishing. Use `-h` on Windows and `-m` on Linux to set the hop limit.

:::

### Exotic Pings

:::::::::::::: {.columns}
::: {.column width="50%"}

#### `pathping` (Windows) / `mtr` (Linux)

* `ping` and `tracert` mashed together
* hop behavior over time

#### `arping` (Linux)

* Give it an IP, send an L2 ARP `who-has`
* Give it a MAC address, send an ICMP ping

:::
::: {.column width="50%"}

![ ](https://cds.cern.ch/images/CERN-PHOTO-202109-138-6/file?size=large "Title: A chain of Large Hadron Collider magnets")

:::
::::::::::::::

::: notes

Sometimes `ping` and `traceroute` don't give you the clues you are looking for.
ICMP pings aren't the same as TCP or UDP. You can get `ping`-like tests with
Nmap, which we'll cover later.

`ping` and `tracert` will give you slightly different views of the network, so
why not get the best of both worlds? On Windows, PathPing gives you individual
hop statistics over a longer period of time. My Trace Route (`mtr`) will give
you similar measurements on Linux. Note that you will need to install it as it
isn't a standard utility.

Linux lets you test layer 2 connectivity with `arping`. Give `arping` an IP
address, it will send link layer frames to the associated MAC address and report
the round trip time. Give it a MAC address, and it will ping the resolved IP
address.

:::

### Outside Looking In

:::::::::::::: {.columns}
::: {.column width="50%"}

#### Looking Glass Servers

* [BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol) routes, `ping`,
  & `traceroute`
* Pick the physical/network location to test from
* Lists: [Traceroute.org](http://www.traceroute.org/),
  [LookinGlass.org](https://lookinglass.org/)
* Sites: [Hurricane Electric](https://bgp.he.net/),
  [Lumen](https://lookingglass.centurylink.com/)

#### SmokePing

* Long term pings with latency
* [How to read graphs](https://oss.oetiker.ch/smokeping/doc/reading.en.html)
* External pings up to 28 days: [DSLReports](https://www.dslreports.com/smokeping)
* Try [vaping](https://20c.com/project/vaping) locally
  [(Docker!)](https://github.com/20c/vaping/blob/main/docs/container.md)

:::
::: {.column width="50%"}

![SmokePing](https://oss.oetiker.ch/smokeping/demo.png)

:::
::::::::::::::

::: notes

All the pings we've talked about so far test from a device that you control out
to another device, but sometimes you need to test from the public Internet in to
your network. Looking glass servers let you remotely run `ping` or `traceroute`
to another device. Significant connectivity providers, like Hurricane Electric
and Lumen, run these servers, which lets you choose where you test from,
geographically and network connectivity-wise. Check the server lists to find the
right one to test with for your situation.

Want to do a quick, longer term (up to about a month) test from the outside?
DSLReports still runs a SmokePing service, which does periodic packet loss and
latency tests. The higher on the graph, the higher the latency. The darker the
"smoke", the more pings with a given latency. The smokier the graph, the more
jitter there is. Before using the DSLReports service, check the terms of service
to make sure it works for your situation or consider setting up `vaping` for a
similar self-hosted test.

:::

### Name That Computer: `dig` (Windows/Linux)

:::::::::::::: {.columns}
::: {.column width="50%"}

* Windows install: [Download](https://www.isc.org/download/) & **Tools Only** install
* Specific/all record type: `dig example.com <rrtype|ANY>`
* Specific DNS server: `@mydnsserver.example.com`
* Be brief: `+short`
* Long but readable: `+multiline +noall +answer +nocmd`
* Traceroute: `+trace`
* Interactive: omit host

#### GUI Alternative (Windows)

* [DNSDataView](https://www.nirsoft.net/utils/dns_records_viewer.html)
* Some automation for reverse lookups

:::
::: {.column width="50%"}

![ ](https://static.wikia.nocookie.net/name-that-tune/images/7/7f/Ntt50s.png/revision/latest?cb=20150214165431 "Title: 1950s Name That Tune title card")

:::
::::::::::::::

::: notes

You will still frequently work with IP addresses, but DNS is an important part
of modern MoIP systems. You can perform basic DNS queries with `nslookup`. When
you need to look up more specialized or complex records, like NMOS can use, you
will need to `dig` for the information. While part of the default packages of
most Linux distributions, you will need to install `dig` from the ISC BIND DNS
tools. Download the installer, which also includes the BIND DNS server, and
start the installation wizard. Be sure to select Tools Only, unless you really
want that DNS server.

With `dig` you can request a specific record type, like `PTR` or `TXT` (used by
NMOS), or all records for a name with the pseudotype of `ANY`.

`nslookup` can be confusing when you need to query a specific DNS server. With
`dig`, you identify the DNS server by prefacing it with an `@` symbol. `dig`
defaults to printing a lot of information about your query, but sometimes that's
TMI. Fortunately, you can format the response for the situation. `+short` gives
you the answer to the query with nothing else, great for scripting. If you still
want a longer view of the answer, but without the default query information,
there are options for that, too. You can even follow the resolution path from
root server to authoritative server, for a sort of DNS traceroute. If you have a
lot of DNS queries to run, you can enter interactive mode by leaving off the
target address.

If you prefer a GUI, NirSoft offers DNSDataView for Windows. It will only query
a limited set of record types. But it can create reports. Diagnosing DNS issues
sometimes means following a trail of CNAMEs and reverse lookups, which
DNSDataView will automate for you.

:::

### What's This Computer Doing?

:::::::::::::: {.columns}
::: {.column width="50%"}

#### `netstat` (Windows)

* Choose protocol: `-p (tcp|udp)`
* Only listening ports: `netstat -a | find /i "listening"`
* Executable & process ID: `netstat -abo`
* GUI: [TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview)
  or [Resource Monitor](https://stackoverflow.com/a/23718720/6283412)

#### `ss` (Linux)

* Choose protocol: `--tcp` or `--udp`
* Only listening ports: `-l`
* [Filters](https://manpages.debian.org/bookworm/iproute2/ss.8.en.html)
  * Source address: `ss -nt src 192.0.2.0/24`
  * Destination port: `ss -ant dst :8080`

:::
::: {.column width="50%"}

![[^credit-microscope]](https://source.unsplash.com/6q5QG8iIgRo "Title: Woman looking through a microscope")

[^credit-microscope]: Photo by [Trust "Tru" Katsande](https://unsplash.com/@iamtru)
on [Unsplash](https://unsplash.com/photos/woman-looking-on-microscope-inside-room-6q5QG8iIgRo)

:::
::::::::::::::

::: notes

What a computer is (or isn't) doing on a network isn't always visible from the
applications we use. Is the server actually waiting for incoming connections?
Which server did the app actually connect to?

On Windows, the command line tool `netstat` ("network statistics") can show
incoming and outgoing connections. It can be helpful to limit the protocol to
TCP or UDP individually, since the lists get long. Printing all the listening
ports with `-l` answers that first question about waiting for incoming
connections.

Piping the output to find lets you filter out all the incoming and outgoing
connections. If you are using Powershell, you will need triple quotes around
listening. It isn't surprising that your computer might sit for a while, seeming
to do nothing. Remember find is going through that long list of incoming,
outgoing, and listening ports to sift out the listening ports.

Now you know that some program is listening, but *which* program is it? Add the
`b` and `o` switches, and you get the executable path and process ID of the
listening application. I recommend against piping to find this time, because
netstat uses a multi-line output format with `b` option.

If you need that kind of detailed information, you should consider switching to
a GUI program for viewing the connection status, and you have two choices.
TCPView gives you more control over what you see, but Resource Monitor is
available on nearly every Windows device that should still be in use.

For Linux, you can still use `netstat`, though the syntax is a little different
from the Windows command. The newer `ss` utility gives you more control over
that long list of connections with filters. Working on a server and want to see
if a particular device connected? Filter on the source address or even a whole
subnet, like shown here in CIDR notation. You can check which servers a device
connected to on a specific port by using a colon prefixed port number.

:::

### What's That Computer Doing?

:warning: Get permission and think about maintenance windows before using these tools!

#### Port scanning: [Nmap](https://nmap.org/)

* Scan multiple ports & addresses: `nmap -p 80,25,443,110 198.51.100.20-100`
* TCP Scan: `-sS` (SYN) or `-sT` (Connect)
* UDP Scan: `-sU`
* Service & version detection: `-sV`
* OS detection: `-O`

#### Vulnerability Scanning

* [Nessus](https://www.tenable.com/products/nessus) or [OpenVAS](https://github.com/greenbone/openvas-scanner)
* Use [the containerized build](https://greenbone.github.io/docs/latest/22.4/container/)

::: notes

Before we talk about finding out what is going on with another computer on a
network, proceed with caution. Make sure you have permission to use these tools.
If your enterprise security team is doing their job, they will know what you are
up to. Besides being concerned about policy violation risks, there is another,
familiar risk to consider. Depending on how you use these tools, they carry the
danger of causing outages. Consider running these tools during planned
maintenance windows and on offline systems.

With that out of the way, let's talk about how to find out what is going on with
another device, especially one you don't have the ability to work directly on.
(But you have permission to muck around, right?) You can use Nmap to scan for
open ports, those programs waiting for incoming connections. Specify the scan
type, UDP or one of the TCP methods, depending on what service you want to ping.
Nmap can also guess at the specific service and version running or detect the OS
of the scanned device.

Vulnerability scanning is an essential part of security preparedness. You want
to know what your exposure is, so you can take the right mitigation steps.
The most common response is remediation, or patching the vulnerability, but you
might take other actions until you verify the patch with a vendor or test it out
yourself. Nessus and the less commercialized OpenVAS (aka Greenbone) scanners
are the most common in use. Use OpenVAS's Docker container to get up and running
fast and without unnecessary complexity.

:::

### Packet Capture

:::::::::::::: {.columns}
::: {.column width="50%"}

* [Wireshark](https://www.wireshark.org/)
* [tcpdump](https://www.tcpdump.org/) (Linux command line)
* Capture at the command line, analyze in Wireshark later
* Use capture/pcap [filters](https://www.tcpdump.org/manpages/pcap-filter.7.html)
([cheat sheet](https://packetlife.net/media/library/12/tcpdump.pdf))

:::
::: {.column width="50%"}

![ ](images/tools/software/use-the-filter.jpg "Title: Use the filter, Luke meme")

:::
::::::::::::::

::: notes

We'll be covering packet capture and analysis in more depth later today, but I
do want to share one Wireshark tip with you. Use *capture*, or pcap, filters as
much as you can to keep your memory and file sizes manageable. Display filters
are easier to work with and more expressive, but media systems are chatty,
generating a lot of packets that take some sifting. I try to start with a short,
unfiltered capture, figure out the packets I'm generally interested in, and then
run a longer capture that filters out as much of the noise as possible.

It can also be more convenient to run a packet capture on the command line with
`tcpdump` and then analyze it later with Wireshark.

:::

### Packet Generators

:::::::::::::: {.columns}
::: {.column width="50%"}

* [PacketStorm](https://packetstorm.com/): MoIP-focused tools
* [Keysight](https://www.keysight.com/us/en/products/network-test/protocol-load-test.html)
  (formerly Ixia): :moneybag::moneybag::moneybag:
* [Ostinato](https://ostinato.org/): Build yourself or reasonably priced binaries

:::
::: {.column width="50%"}

![[^credit-turbine]](https://source.unsplash.com/MkKK8fH3dpA "Title: Wind turbines")

[^credit-turbine]: Photo by [Shinichiro Ichimura](https://unsplash.com/@califso)
on [Unsplash](https://unsplash.com/photos/a-row-of-wind-turbines-in-front-of-a-mountain-MkKK8fH3dpA)

:::
::::::::::::::

::: notes

Working with "real" network traffic isn't always possible or ideal. Packet
generators let you synthesize or replay network traffic. One common use case is
for load testing. Some packet generators can pump out *a lot* of flows, more
than you can reasonably get going by hand with a few clients in a lab
environment.

But commercial packet generators can be well beyond our budgets, and we need
some packets to learn or experiment with. [Ostinato](https://ostinato.org/) is
open source, free as in beer if you build it yourself, or affordably available
as pre-built binaries.

:::

### Performance Benchmarking

:::::::::::::: {.columns}
::: {.column width="50%"}

#### Disk

* [IOMeter](http://www.iometer.org/): Use v1.0.1
* \# of Workers = # of Cores/vCPUs
* Work up a realistic test plan

#### Network

* [Iperf2](https://sourceforge.net/projects/iperf2/): Multicast & other media flows
* [Iperf3](https://github.com/esnet/iperf): IT workloads or public Internet

:::
::: {.column width="50%"}

![[^credit-car]](https://source.unsplash.com/bdHBAFLMMOc "Title: An analog car dashboard")

[^credit-car]: Photo by [Frankie Lopez](https://unsplash.com/@frankielopez) on [Unsplash](https://unsplash.com/photos/gray-analog-vehicle-guage-bdHBAFLMMOc)

:::
::::::::::::::

::: notes

"Is it fast enough?" The specs say that the disk, CPU, RAM... name your
component should be, but we all know reality doesn't match spec sheets. Even
though IOMeter hasn't been updated in a while, it remains the best disk
performance benchmarking tool readily available. Spend a bit of time working out
a realistic test plan for your benchmark. You can simulate different phases of
disk use. Generally, you can have as many workers to create I/O load as you have
cores or vCPUs available to your test machine. Be careful about selecting disks
to test, you don't want to put a system disk or project files in danger.

For network performance tests, Iperf2 and Iperf3 have overlapping feature sets.
Iperf2 is better for multicast and other media flows. Use Iperf3 for more
typical IT workload or public Internet tests. No matter which version you use,
you need to install Iperf on two devices. One simulates a server and listens for
incoming connections, while the other plays the client role and sends simulated
data to the server.

:::

### SNMP

:::::::::::::: {.columns}
::: {.column width="50%"}

* Net-SNMP [Windows Binaries](https://sourceforge.net/projects/net-snmp/files/net-snmp%20binaries/5.7-binaries/net-snmp-5.7.0-1.x86.exe/download)
* Query a tree: `snmpwalk -Os -c <community> -v 1 <host> <mib>`
* MIB Browsers (Windows):
  * [iReasoning](https://ireasoning.com/mibbrowser.shtml)
  * [FrameFlow](https://kb.frameflow.com/free-snmp-browser/)

:::
::: {.column width="50%"}

![[^credit-boatgauge]](https://source.unsplash.com/-yBvef_mAaQ "Title: Analog gauges")

[^credit-boatgauge]: Photo by [Mikhail McVerry](https://unsplash.com/@mcverry)
on [Unsplash](https://unsplash.com/photos/black-speedometer--yBvef_mAaQ)

:::
::::::::::::::

::: notes

SNMP is still widely deployed for device monitoring even with advances in
observability tools. The built-in Windows utilities limit the troubleshooting
you can do. The Net-SNMP project gives you a rich set of command line tools. The
project team is not producing Windows binaries with recent releases, so you
either need to use them on a Linux machine, install the package in Windows
Subsystem for Linux, build binaries yourself, or install older versions. Once
you have access to Net-SNMP, `snmpwalk` prints out a whole tree's worth of
information.

The simple part of SNMP refers to the protocol design, not about its
user-friendliness. A graphical MIB browser helps explore a new device's metrics.
iReasoning and FrameFlow each offer popular GUI clients.

:::

### Time

:::::::::::::: {.columns}
::: {.column width="50%"}

#### NTP

* 1, 2 with `prefer`, or 3+ servers
* Windows: Use the [Meinberg NTP port](https://www.meinbergglobal.com/english/sw/ntp.htm)
* Simple monitoring with `ntpq` or [Meinberg NTP Monitor](https://www.meinbergglobal.com/english/sw/ntp-server-monitor.htm)

#### PTP

* Monitor with [Meinberg PTP Track Hound](https://www.meinbergglobal.com/english/sw/ptp-track-hound.htm)
* Analyze with [EBU LIST](https://github.com/ebu/pi-list)

:::
::: {.column width="50%"}

![PTP Track Hound v2](images/tools/software/ptptrackhound.jpg)

:::
::::::::::::::

::: notes

When troubleshooting time related issues, one of your first checks should be a
device's NTP source. You need to specify 1 or 3 or more NTP servers for the
client to work properly. If you have 2 NTP servers, common in media facilities
with a pair of redundant sync generators, you have to set one of those sources
as the preferred source. The NTP algorithm won't be able to figure out the
correct adjustment without a preferred server.

Another common problem is using the built-in Windows time client. Do yourself a
favor, disable that service and install the Meinberg port of the reference NTP
tools. That client works, uses the same easy to distribute configuration file as
the Linux version, and gives you `ntpq` for basic monitoring.

Meinberg also has a separate NTP monitoring tool, as well as free and paid PTP
monitoring solutions. For PTP, you can use EBU LIST for detailed analysis.

:::

### Multicast

:::::::::::::: {.columns}
::: {.column width="50%"}

* Use [VLC to send RTP](https://support.adder.com/tiki/tiki-index.php?page=Network%3A%20Multicast%20test%20using%20VLC)
* Stress test multicast with [Multicast Hammer](https://support.pelco.com/s/article/Using-Multicast-Hammer-1538586730634?language=en_US)
* [Simple Command Line Multicast Testing Tool](https://github.com/enclave-networks/multicast-test#readme)
  (Windows/Linux)

:::
::: {.column width="50%"}

![[^credit-screens]](https://source.unsplash.com/3uQWtvUPjyg "Title: Assorted CRT monitors")

[^credit-screens]: Photo by [Rubenz Arizta](https://unsplash.com/@ayakoz) on
[Unsplash](https://unsplash.com/photos/assorted-crt-monitor-3uQWtvUPjyg)

:::
::::::::::::::

::: notes

Sometimes all you need is a barebones RTP stream for a quick test. VLC can
stream files over RTP. Choose your source file, enter your multicast address,
set your media configuration, and your stream is going. Use VLC, or another
program to receive the stream on another device.

Multicast Hammer helps you stress test multicast networking. Like Iperf, it can
operate in client or server mode. When acting as a client, you specify the
number of multicast groups to subscribe, the address of the first group, and the
port number to listen on. Multicast Hammer then subscribes to the sequential
addresses starting at the one you specified. The server works similarly, with
the addition of transmit speed, packet size, and burst options.

We talked about iperf for benchmarking multicast before, but if you need a
multicast test that isn't at all finicky and lets you choose the network
interface to use, Simple Command Line Multicast Testing Tool does what it says
on the tin.

:::

### APIs

:::::::::::::: {.columns}
::: {.column width="50%"}

* Use API testers like [milkman](https://github.com/warmuuh/milkman),
  [bruno](https://github.com/usebruno/bruno), and [insomnium](https://github.com/ArchGPT/insomnium)
  * Find more [Awesome API Clients](https://github.com/stepci/awesome-api-clients?tab=readme-ov-file)
* Use [OpenAPI](https://openapis.org/) descriptions or
[smithy](https://smithy.io/) contracts
  * Example: [NMOS IS-05](https://specs.amwa.tv/is-05/releases/v1.1.2/APIs/ConnectionAPI.html)
  ([source](https://github.com/AMWA-TV/is-05/blob/v1.1.x/APIs/ConnectionAPI.raml))
* Don't know anything about the API? [Ncat](https://nmap.org/ncat/)

:::
::: {.column width="50%"}

![Testing an NMOS API with Bruno](images/tools/software/api.png)

:::
::::::::::::::

::: notes

The first time you come face to face with a RESTful API, it can feel like a bit
of a dark art, but there are two ways to turn that art into science. First,
don't try to handcraft an API request. Use an API testing tool. Leading free or
open source testers have recently made user un-friendly changes, so I can't
strongly recommend one right now. There are plenty of choices depending on your
needs and preferences.

* Milkman is modular, offering a variety of transport and other useful plugins.
* Bruno is an offline, file-based storage API tool. Note that the developers are
  planning to release some transports that are of interest to professional media
  networks in a freemium version
* insomnium forked one of the popular API tools from before the problematic
  changes, but hasn't had active development since then.

The second tip for API troubleshooting is you should try to work from formal API
specifications, often in the form of OpenAPI descriptions or smithy contracts
for event-based APIs.

If you know little about an API, a full test tool might not be a good place to
start. Ncat, from the Nmap team, is a modern update to netcat. Ncat can be a
sender, a receiver, or a proxy. If you want to figure out something like a
quirky tally protocol, start Ncat, type commands, and view responses. It can
even use TLS encryption.

:::

### NMOS & 2110

:::::::::::::: {.columns}
::: {.column width="50%"}

* [sdpoker](https://github.com/AMWA-TV/sdpoker) for troubleshooting SDP files
* AMWA NMOS [Testing Tool](https://specs.amwa.tv/nmos-testing/) for automated
compliance testing
* [Easy-NMOS](https://github.com/rhastie/easy-nmos): Docker compose for NMOS
  registry, virtual node, & Testing Tool
* Riedel [NMOS Explorer](https://www.riedel.net/en/downloads/firmware-software)
for quick browsing devices
* [EBU LIST](https://github.com/ebu/pi-list) for timing analysis, ANC decoding,
  A/V sync, redundancy comparison, and more!

:::
::: {.column width="50%"}

![ ](images/tools/software/2110-nmos.png "Title: SMPTE ST 2110 & NMOS logos")

:::
::::::::::::::

::: notes

There are some great open source and freely available tools for NMOS and SMPTE
ST 2110 troubleshooting. Mentioned before, EBU LIST is great for analyzing your
video and audio flows. Different vendors have different implementations and
interpretations of SDP descriptions, which sdpoker can help you reconcile.

AMWA, the organization behind NMOS has also released an automated testing tool.
This is the same setup used during the JT-NM tested events that you can use for
verifying compatibility for the devices and systems that you use in your
facility.

Whether you are trying to learn more, test new devices, or you are actively
troubleshooting an issue, how many times have you wished you could set up your
own router control system? In the MoIP world, we can actually do that, and there
even is an "easy button" for it. Easy-NMOS is a docker compose-based recipe for
spinning up an NMOS registry, a virtual node, and the testing tool.

Also, Riedel has a freely available browser for quick peeks into an NMOS
registry.

:::

### Containers

:::::::::::::: {.columns}
::: {.column width="50%"}

* Understand [docker networks](https://dev.to/manojpatra1991/docker-cheat-sheet-docker-networks-49k4)
  * Don't use `bridge0`
  * Expose minimal ports with `-p`
* [netshoot](https://github.com/nicolaka/netshoot): "a Docker +
  Kubernetes network trouble-shooting swiss-army container"
* Go back to basics: [traefik/whoami](https://github.com/traefik/whoami)

:::
::: {.column width="50%"}

![ ](https://camo.githubusercontent.com/ac0ee4f2ad3b7d8b6f01804412061675d50367c2d6002631e51adab110863045/687474703a2f2f7777772e6272656e64616e67726567672e636f6d2f506572662f6c696e75785f6f62736572766162696c6974795f746f6f6c732e706e67 "Title: Linux Performance Observability Tools")

:::
::::::::::::::

::: notes

Containers can make your life easier, but if you have to troubleshoot Docker
networking, you might be questioning your tech stack choices. First, make sure
you understand how docker networks work. A good cheat sheet will help with some
secure best practices, like not using the default network and only exposing
necessary ports.

You can use the `netshoot` container to access some great troubleshooting tools.
But its real strength is being able to troubleshoot from inside a specific
docker network or even inside another container's namespace. You'll need to do
that when you don't have those troubleshooting tools already built in to the
problem container.

If you still can't make sense of a Docker issue, it can help to go back to the
basics. The `whoami` container, from the Traefik reverse proxy team, gives you
all the building blocks of a real container --- latency, a JSON api, health
checks --- stripped down to lightweight essentials. Get your network problem
sorted out with `whoami` and then go back to your real container.

:::

### Keep Searching

:::::::::::::: {.columns}
::: {.column width="50%"}

* Check [Awesome Real Time Communications](https://github.com/rtckit/awesome-rtc#readme)
  for SIP tools
* Find lots of [Awesome Broadcasting](https://github.com/ebu/awesome-broadcasting#readme)
  tools
* [Awesome Networking](https://github.com/nyquist/awesome-networking)
* [Awesome](https://github.com/sindresorhus/awesome#readme)
* [Awesome Awesomness](https://github.com/bayandin/awesome-awesomeness#readme)

:::
::: {.column width="50%"}

![[^credit-library]](https://source.unsplash.com/OG9NZVNCnFo "Title: Interior of the Vasconcelos Library in Mexico City")

[^credit-library]: Photo by [Girl with red hat](https://unsplash.com/@girlwithredhat) on [Unsplash](https://unsplash.com/photos/white-and-green-concrete-building-OG9NZVNCnFo)

:::
::::::::::::::

::: notes

The last tip I'll leave you with is to keep your eyes open. You never know when
you'll find the next tool or technique that will help you solve problems faster
or deal with more complex or just plain weird issues. There's lots more Awesome
out there.

:::

## References

::: {#refs}
:::
