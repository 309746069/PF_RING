Sample Applications
===================

If you are new to PF_RING, you can start with some examples. The *PF_RING/userland* 
folder is rich of ready-to-use PF_RING applications. They are small standalone applications 
which demonstrate various features of PF_RING. Users interested in getting started with 
PF_RING can test the applications and extend them based on their needs.

- *PF_RING/userland/examples* contains sample application using the PF_RING API
- *PF_RING/userland/examples_zc* contains sample application using the PF_RING ZC API
- *PF_RING/userland/examples_ft* contains sample application using the PF_RING FT API

Please note that the applications features many options to try to cover all use cases,
please check the help (-h) for a full list of options.

Compiling the Applications
--------------------------

To compile the sample application see the `Installing From GIT <https://www.ntop.org/guides/pf_ring/get_started/git_installation.html#libpfring-and-libpcap-installation>`_
section. Please note that if you are installing PF_RING from packages, some of the
sample applications described here are also distributed in binary format with the
*pfring* package.

Basic Packet Capture
--------------------

**pfcount** (in *PF_RING/userland/examples*) is a sample application that allows you 
to capture raw packets and print some statistics. Example:

.. code-block:: console

   sudo ./pfcount -i zc:eth1
   =========================
   Absolute Stats: [64415543 pkts rcvd][0 pkts dropped]
   Total Pkts=64415543/Dropped=0.0 %
   64'415'543 pkts - 5'410'905'612 bytes [4'293'748.94 pkt/sec - 2'885.39 Mbit/sec]
   =========================
   Actual Stats: 14214472 pkts [1'000.03 ms][14'214'017.15 pps/9.55 Gbps]
   =========================

The same application can also be used to parse packets and extract metadata from L2/L3/L4
headers, adding the -v option. Example:

.. code-block:: console

   sudo ./pfcount -i eth1 -v 1
   Dumping statistics on /proc/net/pf_ring/stats/15773-eno1.279
   11:31:41.968485349 [TX][if_index=6][hash=2169540001][00:26:90:D3:CC:F1 -> 0C:C7:7A:CC:C1:4D] [IPv4][192.168.1.20:22 -> 192.168.1.21:34762] [l3_proto=TCP][hash=2169540001][tos=16][tcp_seq_num=415123802] [caplen=254][len=254][eth_offset=0][l3_offset=14][l4_offset=34][payload_offset=66]
   11:31:41.968557503 [TX][if_index=6][hash=2169540001][00:26:90:D3:CC:F1 -> 0C:C7:7A:CC:C1:4D] [IPv4][192.168.1.20:22 -> 192.168.1.21:34762] [l3_proto=TCP][hash=2169540001][tos=16][tcp_seq_num=415123990] [caplen=166][len=166][eth_offset=0][l3_offset=14][l4_offset=34][payload_offset=66]
   11:31:41.968598956 [TX][if_index=6][hash=2169540001][00:26:90:D3:CC:F1 -> 0C:C7:7A:CC:C1:4D] [IPv4][192.168.1.20:22 -> 192.168.1.21:34762] [l3_proto=TCP][hash=2169540001][tos=16][tcp_seq_num=415124090] [caplen=390][len=390][eth_offset=0][l3_offset=14][l4_offset=34][payload_offset=66]

**zcount** (in *PF_RING/userland/examples_zc*) is similar to **pfcount**, however
it is based on the PF_RING ZC API. Example:

.. code-block:: console

   sudo ./zcount -i zc:eth1 -c 10
   =========================
   Absolute Stats: 89415341 pkts (0 drops) - 7510888644 bytes
   Actual Stats: 14'218'113.27 pps (0.00 drops) - 9.55 Gbps
   =========================

Where:

- The interface specified with -i can be any interface (in the example above we are using a ZC driver)
- The number specified with -c is the cluster ID (the ZC API requires a unique identifier to identify a cluster instance)

Basic Packet Transmission
-------------------------

**pfsend** (in *PF_RING/userland/examples*) allows you to generate traffic, forging synthetic 
packets or replaying a *pcap* file.
By default packets are transmitted at the maximum rate supported by the driver, however it is 
possible to use -p <pps> or -r <Gbps> to control the tramsission rate. It is also possible to 
specify the number of packets to send with -n <num>. 

Example with synthetic traffic on a standard interface:

.. code-block:: console

   sudo ./pfsend -i eth1
   TX rate: [current 1'275'650.89 pps/0.86 Gbps][average 1'275'650.89 pps/0.86 Gbps][total 1'275'656.00 pkts]
 
Example replaying a pcap file on a ZC interface, controlling the rate:
 
.. code-block:: console

   sudo ./pfsend -i zc:eth1 -f 64byte_packets.pcap -n 0  -r 5
   TX rate: [current 7'508'239.00 pps/5.05 Gbps][average 7'508'239.00 pps/5.05 Gbps][total 7'508'239.00 pkts]
   
**zsend** (in *PF_RING/userland/examples_zc*) is similar to **pfsend**, however
it is based on the PF_RING ZC API. Example:

.. code-block:: console

   sudo ./zsend -i eth1 -c 10
   =========================
   Absolute Stats: 2'604'538 pkts - 218'781'192 bytes
   Actual Stats: 1'305'510.19 pps - 0.88 Gbps [109672836 bytes / 1.0 sec]
   =========================

Where:

- The interface specified with -i can be any interface (in the example above we are using a standard kernel driver)
- The number specified with -c is the cluster ID (the ZC API requires a unique identifier to identify a cluster instance)

Load Balancing
--------------

Multi-Threaded
~~~~~~~~~~~~~~

**zbalance** (in *PF_RING/userland/examples_zc*) is a sample application able to capture traffic
from one or multiple interfaces, and load-balance packets to multiple consumer threads.

.. code-block:: console

   sudo ./zbalance -i eno1 -c 10 -m 1 -r 0 -g 1:2
   Starting balancer with 2 consumer threads..
   =========================
   Thread #0: 17 pkts - 2'723 bytes
   Thread #1: 19 pkts - 3'011 bytes
   =========================
   Absolute Stats: 36 pkts - 5'734 bytes
   Actual Stats: 15.00 pps - 0.00 Gbps
   =========================

Where:

- The interface specified with -i can be a comma-separated list of interfaces
- The number specified with -c is the cluster ID (the ZC API requires a unique identifier to identify a cluster instance)
- With -m it is possible to select the hash function for traffic distribution across threads (please see the help with -h for the list). There are a few built-in options, but it is also possible to define custom distribution functions (please see **zbalance_ipc** for more distribution functions examples)
- The -r option selects the CPU core where the load-balancer thread will be running
- The -g option selects the CPU cores where the consumer threads will be running (as many threads as the number of cores)

Multi-Process
~~~~~~~~~~~~~

**zbalance_ipc** (in *PF_RING/userland/examples_zc*) is a sample application that can be used 
for capturing traffic from one or multiple interfaces, and load-balance packets to multiple consumer 
processes. Please read the `ZC Load-Balancing <https://www.ntop.org/guides/pf_ring/rss.html#zc-load-balancing-zbalance-ipc>`_ 
section to learn more about this application.




