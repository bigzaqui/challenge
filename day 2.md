#day 2

###TODOs
- read about linux TCP congestion control algorithms
- TCP for FLN [RFC](https://www.ietf.org/rfc/rfc1323.
- CPU affinity binding
- [MTUPD](https://tools.ietf.org/html/rfc1191)
- [codel](https://www.bufferbloat.net/projects/codel/wiki/)


##Causes of packet loss
**Hard failure**: 100% packet loss, **soft failure**: "is slow"

- Network Congestion
 - Easy to confirm via SNMP, easy to fix with $$
 - This is not a ‘soft failure’, but just a network capacity issue
 - often people assume congestion is the issue when it fact it is not.
- Under-buffered switch dropping packets and/or firewall dropping packets
 - Hard to confirm
- Dirty fibers or connectors, failing opGcs/light levels
 - Sometimes easy to confirm by looking at error counters in the routers
- Overloaded or slow receive host dropping packets
 - Easy to confirm by looking at CPU load on the host

	
##Kernel tunning of network settings:
###Concepts
**BPD**: (bandwith delay product) data that has been transmitted but not yet acknowledged, used to calculate buffer sizes
- this [section](https://fasterdata.es.net/host-tuning/) fasterdata has, is worth the time, this [one](http://www.linux-admins.net/2010/09/linux-tcp-tuning.html) is also good.

- Steps on Linux when sending data across the network:
	1. The application first writes the data to a socket which in turn is put in the transmit buffer.
	2. The kernel encapsulates the data into a PDU - protocol data unit.
	3. The PDU is then moved onto the per-device transmit queue.
	4. The NIC driver then pops the PDU from the transmit queue and copies it to the NIC.
	5. The NIC sends the data and raises a hardware interrupt.
	6. On the other end of the communication channel the NIC receives the frame, copies it on the receive buffer and raises a hard interrupt. 
	7. The kernel in turn handles the interrupt and raises a soft interrupt to process the packet.
	8. Finally the kernel handles the soft interrupt and moves the packet up the TCP/IP stack for decapsulation and puts it in a receive buffer for a process to read from.
- In older versions of the linux kernel, the buffer sizes on receiver and sender per socket had a default value and if desired the application could modify them via system calls, but now Linux now does a good job of auto-tuning the TCP buffers.
- TCP negotiates the buffer size using as input the values defined in both client and server, which means that both should be tuned, an a usual practice is having the server with a big buffer so the client is the one determining the optimal value
- theres a [formula](https://www.netcraftsmen.com/tcp-performance-and-the-mathis-equation/) (and even a [site](https://www.switch.ch/network/tools/tcp_throughput/)) for calculating TCP theoretical throughput based on the vars:
	- 	MSS
	-  RTT
	-  % of packet loss
- While playing with buffer sizes, one must be aware: every TCP socket has the potential to request this amount of memory even for short connections, making it easy to exhaust system resources. 
- **Possible reason of scp being slow:**A common tool for copying files across the internet is scp. Unfortunately, TCP tuning does not help with scp throughput, because scp uses OpenSSL, which uses statically defined internal flow control buffers. These buffers act as a bottleneck on network throughput, especially on long-delay, high-bandwidth network links. The Pittsburgh Supercomputing Center's High Performance SSH/SCP page explains this in more detail and also has a patch for OpenSSL to fix this problem.


##Tools
####Iperf3
- Different versions of Iperf have different throughtputs, the difference is not that big though.
- parallel TCP connections, objective is to saturate the link, make sure that there's no bottleneck in on thread only (CPU)

 
#### Pathrate
this [tool](https://www.cc.gatech.edu/~dovrolis/bw-est/pathrate.html) determines capacity of internet paths, TODO: [read paper](https://www.cc.gatech.edu/~dovrolis/Papers/ton_dispersion.pdf) about packet dispersion techniques
#### Nuttcp burst mode
nuttcp includes a 'burst mode' for UDP, which is useful to find paths that are constrained by network devices with not enough buffering. 
 
#### Perfsonar
Imagine ATLAS probes but for [university networks](https://www.perfsonar.net/about/comparison-with-other-monitoring/)

####Ping and MTU path discovery


-M hint

Select Path MTU Discovery strategy. hint may be either do (prohibit fragmentation, even local one), want (do PMTU discovery, fragment locally when packet size is large), or dont (do not set DF flag).

###Open questions EIDE
[here, page 34](https://services.geant.net/sites/edupert/Resources/Documents/BoF_TNC2016_TNC-performance-workshop-Tierney-June12-2016.pdf) what problems can UDP find that TCP dont, besides de under buffered devices on the path

###Misc resources
- [High-Performance	TCP	 Tips and	Tricks](https://services.geant.net/sites/edupert/Resources/Documents/BoF_TNC2016_TNC-performance-workshop-Tierney-June12-2016.pdf): really damn good slides with plenty of resources
- [Linode's guide for using mtr, nothing special](https://linode.com/docs/networking/diagnostics/diagnosing-network-issues-with-mtr/)
