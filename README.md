OpenWrtScripts
==============

This is a set of scripts (sometimes also called "Openscripts") that report, configure and measure (and improve) latency in home routers (and everywhere else!) These scripts include:

* [getstats.sh](#getstatssh) - a script to collect troubleshooting information that helps us diagnose problems in the OpenWrt distribution.

* [config-openwrt.sh](#config-openwrtsh) - a script to configure the OpenWrt router consistently after flashing factory firmware.

* [betterspeedtest.sh](#betterspeedtestsh) & [netperfrunner.sh](#netperfrunnersh) & [networkhammer.sh](#networkhammersh) - scripts that measure the performance of your router or offer load to the network for testing.

* [tunnelbroker.sh](#tunnelbrokersh) - a script to set up a IPv6 6-in-4 tunnel to TunnelBroker.net. *This script has not been converted for OpenWrt*

These scripts can be saved in the `/usr/lib/OpenWrtScripts` directory. The easiest way to do this is to use ssh into the router and enter these commands:

```
opkg update
opkg install netperf
opkg install git
cd /usr/lib
git clone git://github.com/richb-hanover/OpenWrtScripts.git
```
---
## getstats.sh

The `getstats.sh` script helps diagnose problems with OpenWrt. 
If you report a problem, it is always helpful to include the output of this script. 

`getstats.sh` executes a built-in set of commands and writes the collected output to `/tmp/openwrtstats.txt`. 
The script also executes commands passed as arguments on the command line. 
In the example below, the output would contain results from the standard set of commands plus the two additional arguments: 

**Example:** `sh getstats.sh "ls /usr/lib" "ls -al /etc/config"`

**To install and run this script:** The script is self-contained, and can be placed in any directory. Read the top of the [getstats.sh](./getstats.sh) file for a simple procedure. 

**Sample output file:** See a sample output file - [openwrtstats.txt](./sample_output/openwrtstats.txt)

---

## config-openwrt.sh

The `config-openwrt.sh` script updates the factory settings of OpenWrt to a known-good configuration.
If you frequently update your firmware, you can use this script to reconfigure
the router to a consistent state.
You should make a copy of this script, customize it to your needs,
then use the "To run this script" procedure (below).

This script is designed to configure the settings after an initial "factory" firmware flash. 
There are sections below to configure many aspects of your router.
All the sections are commented out. There are sections for:

- Set up the WAN interface to connect to your provider
- Update the software packages
- Update the root password
- Set the time zone
- Enable SNMP for traffic monitoring and measurements
- Enable mDNS/ZeroConf on the WAN interface 
- Set the SQM (Smart Queue Management) parameters

_[Note: the remaining items have not been converted to work on OpenWrt yet
- Enable NetFlow export for traffic analysis
- Change default IP addresses and subnets for interfaces
- Change default DNS names
- Set the radio channels
- Set wireless SSID names
- Set the wireless security credentials]_

**To run this script**

Flash the router with factory firmware. Then telnet in and execute these statements. 
You should do this over a wired connection because some of these changes
may reset the wireless network.

    telnet 192.168.1.1
    cd /tmp
    cat > config.sh 
    [paste in the contents of this file, then hit ^D]
    sh config.sh
    Presto! (You should reboot the router when this completes.)

**Note:** If you use a secondary OpenWrt router, you can create another copy of this script, and use it to set different configuration parameters (perhaps different subnets, radio channels, SSIDs, enable mDNS, etc).  

---
## betterspeedtest.sh

The `betterspeedtest.sh` script emulates the web-based test performed by speedtest.net, but does it one better. While script performs a download and an upload to a server on the Internet, it simultaneously measures latency of pings to see whether the file transfers affect the responsiveness of your network. 

Here's why that's important: If the data transfers do increase the latency/lag much, then other network activity, such as voice or video chat, gaming, and general network activity will also work poorly. Gamers will see this as lagging out when someone else uses the network. Skype and FaceTime will see dropouts or freezes. Latency is bad, and good routers will not allow it to happen.

The betterspeedtest.sh script measures latency during file transfers. To invoke it:

    sh betterspeedtest.sh [ -4 | -6 ] [ -H netperf-server ] [ -t duration ] [ -p host-to-ping ] [-n simultaneous-streams ]

Options, if present, are:

* -H | --host: DNS or Address of a netperf server (default - netperf.bufferbloat.net)  
Alternate servers are netperf-east (east coast US), netperf-west (California), 
and netperf-eu (Denmark)
* -4 | -6:     Enable ipv4 or ipv6 testing (default - ipv4)
* -t | --time: Duration for how long each direction's test should run - (default - 60 seconds)
* -p | --ping: Host to ping to measure latency (default - gstatic.com)
* -n | --number: Number of simultaneous sessions (default - 5 sessions)

The output shows separate (one-way) download and upload speed, along with a summary of latencies, including min, max, average, median, and 10th and 90th percentiles so you can get a sense of the distribution. The tool also displays the percent packet loss. The example below shows two measurements, bad and good. 

On the left is a test run without SQM. Note that the latency gets huge (greater than 5 seconds), meaning that network performance would be terrible for anyone else using the network. 

On the right is a test using SQM: the latency goes up a little (less than 23 msec under load), and network performance remains good.

    Example with NO SQM - BAD                                     Example using SQM - GOOD
    
    root@openwrt:/usr/lib/OpenWrtScripts# sh betterspeedtest.sh   root@openwrt:/usr/lib/OpenWrtScripts# sh betterspeedtest.sh
    [date/time] Testing against netperf.bufferbloat.net (ipv4)    [date/time] Testing against netperf.bufferbloat.net (ipv4)
       with 5 simultaneous sessions while pinging gstatic.com        with 5 simultaneous sessions while pinging gstatic.com
       (60 seconds in each direction)                                (60 seconds in each direction)
    
     Download:  6.65 Mbps                                         Download:  6.62 Mbps
      Latency: (in msec, 58 pings, 0.00% packet loss)              Latency: (in msec, 61 pings, 0.00% packet loss)
          Min: 43.399                                                  Min: 43.092
        10pct: 156.092                                               10pct: 43.916
       Median: 230.921                                              Median: 46.400
          Avg: 248.849                                                 Avg: 46.575
        90pct: 354.738                                               90pct: 48.514
          Max: 385.507                                                 Max: 56.150
    
       Upload:  0.72 Mbps                                           Upload:  0.70 Mbps
      Latency: (in msec, 59 pings, 0.00% packet loss)              Latency: (in msec, 53 pings, 0.00% packet loss)
          Min: 43.699                                                  Min: 43.394
        10pct: 352.521                                               10pct: 44.202
       Median: 4208.574                                             Median: 50.061
          Avg: 3587.534                                                Avg: 50.486
        90pct: 5163.901                                              90pct: 56.061
          Max: 5334.262                                                Max: 69.333

---         
## netperfrunner.sh

The `netperfrunner.sh` script runs several netperf commands simultaneously.
This mimics the stress test of [netperf-wrapper](https://github.com/tohojo/netperf-wrapper) [Github] but without the nice GUI result.

When you start this script, it concurrently uploads and downloads several
streams (files) to a server on the Internet. This places a heavy load 
on the bottleneck link of your network (probably your connection to the Internet), 
and lets you measure both the total bandwidth and the latency of the link during the transfers.

To invoke the script:

    sh netperfrunner.sh [ -4 | -6 ] [ -H netperf-server ] [ -t duration ] [ -p host-to-ping ] [-n simultaneous-streams ]

Options, if present, are:

* -H | --host: DNS or Address of a netperf server (default - netperf.bufferbloat.net)  
Alternate servers are netperf-east (east coast US), netperf-west (California), 
and netperf-eu (Denmark)
* -4 | -6: Enable ipv4 or ipv6 testing (default - ipv4)
* -t | --time: Duration for how long each direction's test should run - (default - 60 seconds)
* -p | --ping: Host to ping to measure latency (default - gstatic.com)
* -n | --number: Number of simultaneous sessions (default - 4 sessions)

The output of the script looks like this:

    root@openwrt:/usr/lib/OpenWrtScripts# sh netperfrunner.sh
    [date/time] Testing netperf.bufferbloat.net (ipv4) with 4 streams down and up 
        while pinging gstatic.com. Takes about 60 seconds.
    Download:  5.02 Mbps
      Upload:  0.41 Mbps
     Latency: (in msec, 61 pings, 15.00% packet loss)
         Min: 44.494
       10pct: 44.494
      Median: 66.438
         Avg: 68.559
       90pct: 79.049
         Max: 140.421

**Note:** The download and upload speeds reported may be considerably lower than your line's rated speed. This is not a bug, nor is it a problem with your internet connection. That's because the acknowledge messages sent back to the sender consume a significant fraction of the link's capacity (as much as 25%). 

---
## networkhammer.sh


The `networkhammer.sh` script continually invokes the netperfrunner script to provide a heavy load. It runs forever - Ctl-C will interrupt it. 
 

---
## tunnelbroker.sh

_[This script has not been converted yet]_

The `tunnelbroker.sh` script configures OpenWrt to create an IPv6 tunnel via Hurricane Electric. 
It's an easy way to become familiar with IPv6 if your ISP doesn't offer native IPv6 capabilities. There are three steps:

1. Go to the Hurricane Electric [TunnelBroker.net](http://www.tunnelbroker.net/)  site to set up your free account. There are detailed instructions for setting up an account and an IPv6 tunnel at the
   [IPv6 Tunnel page.](http://www.bufferbloat.net/projects/cerowrt/wiki/IPv6_Tunnel) 
2. Edit the tunnelbroker.sh script, using the parameters supplied by Tunnelbroker.net. They're on the site's "Tunnel Details" page. Click on the "Example
Configurations" tab and select "OpenWRT Backfire 10.03.1". Use the info to fill in the corresponding lines of the script. 
3. ssh into the OpenWrt router and execute this script with these steps.
    
        ssh root@172.30.42.1
        cd /tmp
        cat > tunnel.sh 
        [paste in the contents of this file, then hit ^D]
        sh tunnel.sh
        [Restart your router. This seems to make a difference.]
  
Presto! Your tunnel is up! Your computer should get a global IPv6 address, and should be able to communicate directly with IPv6 devices on the Internet. To test it, try: `ping6 ivp6.google.com`

