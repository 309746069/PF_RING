SR-IOV with PF_RING ZC
======================

SR-IOV allows a single physical PCI adapter to be shared by means of 
different virtual functions. This is mainly used in virtual environments 
to allow different Virtual Machines to share a single physical interface 
and improve network performance.

PF_RING includes accelerated ZC drivers for Intel SR-IOV adapters based
on ixgbe (ixgbevf). 

Capture from a Virtual Function
-------------------------------

In order to enable SR-IOV on an Intel ixgbe adapter, you can use sysfs.
In the example below we enable 2 Virtual Functions for the interface eth1:

.. code-block:: console

   echo '2' > /sys/bus/pci/devices/$(ethtool -i enp2s0f1 | grep bus-info | cut -d ' ' -f2)/sriov_numvfs

At this point, 2 new interfaces using the *ixgbevf* driver should appear 
in *ifconfig -a*. Example:

.. code-block:: console

   ethtool -i enp3s16f1 | grep driver
   driver: ixgbevf

In order to steer the traffic to this interface, it is possible to use
*ip link*, applying a filter on *mac* and *vlan* (optional). Example:

.. code-block:: console

   ip link set enp3s16f1 vf 0 mac 00:01:02:03:04:05 vlan 1

In order to enable the ZC driver for those interfaces, we need to load 
the *ixgbevf* driver distributed with PF_RING. Example from source code:

.. code-block:: console

   cd PF_RING/drivers/intel
   make
   cd ixgbevf/ixgbevf-*-zc/src
   ./load_driver.sh

At this point you should be able to capture traffic from the Virtual 
Function in ZC mode prepending "zc:" to the interface name:

.. code-block:: console

   pfcount -i zc:enp3s16f1

Assign a Virtual Function to a VM
---------------------------------

In order to assign a Virtual Function to a VM with virsh, follow the 
following steps:

Edit /etc/default/grub as below:

.. code-block:: text

   GRUB_CMDLINE_LINUX_DEFAULT="iommu=1 msi=1 pci=assign-busses intel_iommu=on"

Update grub:

.. code-block:: console

   update-grub
   reboot

Shutdown the VM (in this example ubuntu16):

.. code-block:: console

   virsh shutdown ubuntu16

Identify the network device to assign and check the bus id:

.. code-block:: console

   ethtool -i enp3s16f3 | grep bus-info | cut -d ' ' -f2
   0000:03:10.3

Create a XML file (e.g. vf.xml) with bus/slot/function of the device:

.. code-block:: text

   <interface type='hostdev' managed='yes'>
     <source>
       <address type='pci' domain='0' bus='03' slot='10' function='3'/>
     </source>
   </interface>

Add the Virtual Function to the Virtual Machine configuration:

.. code-block:: console

   virsh attach-device ubuntu16 vf.xml --config

Assign more memory to the VM (optional):

.. code-block:: console

   virsh setmaxmem ubuntu16 2097152 --config
   virsh setmem ubuntu16 2097152

Start the VM:

.. code-block:: console

   virsh start ubuntu16

At this point you should be able to load the *ixgbevf* driver as explained
in the previous section, and capture traffic from the Virtual Function.
