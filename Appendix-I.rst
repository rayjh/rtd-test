
Appendix - I Multiple Cluster Support (manual setup, not yet automated)
=======================================================================

Multiple clusters can be supported with manual configuration of the
management switch and bridges on the deployer node. In the diagram
below, two clusters are shown. Each cluster has it's own container on
the deployer node. Since some of the Genesis function runs outside the
container, it is advisable to backup the inventory.yml file, the
config.yml file and the playbooks / hosts file in any existing
containers before creating additional containers.

Backing up an existing cluster;

-  *ubuntu@sm8:~/cluster-genesis$ sudo mv /var/oprc/inventory.yml
   /var/oprc/inventory.cluster1.yml*
-  *ubuntu@sm8:~/cluster-genesis$ cp -p config.yml config.cluster1.yml*
-  *ubuntu@sm8:~/cluster-genesis$ cp -p playbooks/hosts
   playbooks/hosts.cluster1*
-  *ubuntu@sm8:~/cluster-genesis$ git checkout -- playbooks/hosts *

**Note: To update to the latest Genesis; (from cluster-genesis)**

*git checkout -- . *

*git fetch --all*

**

**Next change the "container\_name" value in
playbooks/host\_vars/localhost **

*deployer@dv3:~/cluster-genesis$ cat playbooks/host\_vars/localhost *

*container\_name: ubuntu-14-04-deployer-cluster\ **m***

******

**Update the container subnet *\ *value in playbooks/group\_vars/all
*\ * (this will be the subnet used for the new cluster). * *

*container\_mgmt\_subnet: 192.168.\ **m**.0/24*

**NOTE: this must match the value of* ipaddr-mgmt-network: *in the
cluster's config.yml file.**

******

**Create the container for the new cluster;**

*ubuntu@sm8:~/cluster-genesis/playbooks$ ansible-playbook -i hosts
lxc-create.yml*

******

**As root, edit the *\ */var/lib/lxc/container-name/config file and
*\ *update the lxc.network.ipv4 value for the br0 bridge to reflect the
management subnet ip address of the new cluster.**

*lxc.network.ipv4 = 192.168.\ **m**.2/24*

**

As root, add the following to the /var/lib/lxc/container-name/config
file (br1 is the common bridge used by all containers to access the
switch management ports. Each container needs a unique ip address in the
common subnet.)

lxc.network.type = veth

lxc.network.flags = up

lxc.network.link = br1

lxc.network.ipv4 = 192.168.10.\ **m**/24

Next restart the container to setup the additional bridge (br1 in this
case)

*lxc-stop -n container-name*

*lxc-start -n container-name -d*

**

**

**Check that the bridge for accessing the switches has a new veth pair
attached to it;**

*brctl show*

**Login to or attach to the container and verify that you can ping the
management and data switches management interfaces.**

.. figure:: https://raw.githubusercontent.com/wiki/open-power-ref-design/cluster-genesis/images/cluster-genesis-multi_cluster_network_configuration.png
   :alt: Illustration 3: Multi-Cluster Network Configuration
   :width: 6.51930in
   :height: 4.00910in

   Illustration 3: Multi-Cluster Network Configuration

Install the vlan package on the deployer node;

apt-get install vlan

Edit the /etc/network/interfaces file. In the example file below, the
first cluster is configured for vlan id = 98, the second cluster is
configured for vlan id = 99.

auto p1p1#Here port p1p1 is to be used for connection to the management
switch

iface p1p1 inet manual

iface p1p1.98 inet manual

 vlan-raw-device p1p1

iface p1p1.99 inet manual

 vlan-raw-device p1p1

#cluster management bridge (unique to each cluster)

auto br0

iface br0 inet static

bridge\_stp off

bridge\_waitport 0

bridge\_fd 0

bridge\_ports p1p1.98

address 192.168.0.1 # any private subnet unique to this cluster

netmask 255.255.255.0

offload-sg off

#switch management bridge (common to all clusters)

auto br1

iface br0 inet static

bridge\_stp off

bridge\_waitport 0

bridge\_fd 0

bridge\_ports p1p1

address 192.168.10.1 # this subnet must be different than any of the
cluster subnets

netmask 255.255.255.0

offload-sg off

auto br2

iface br2 inet static

bridge\_stp off

bridge\_waitport 0

bridge\_fd 0

bridge\_ports p1p1.98

address 192.168.1.1 # any private subnet unique to this cluster

netmask 255.255.255.0

offload-sg off

Configure the management switch port connected to the deployer node for
trunk mode with nvlan=1 and allowed vlan =98 and 99;

interface port p

switchport mode trunk

switchport trunk allowed vlan add 98

switchport trunk allowed vlan add 99

exit

 **NOTE**: Be sure to *add* the vlan. If you use this command without
the add option, you will change the native vlan and likely lose your
connection.

Configure the managment switch ports used for BMC and PXE port
connection for access mode with pvid=98 or 99 depending on the cluster.
If the management network is to be connected to the customers network,
be sure to choose a pvid that is not in use on the customers network.
(Be careful not to place the deployer host OS port into access mode on
this vlan or you will lose connection to the switch.)

interface port p

switchport access vlan 98

exit

Repeat for all BMC and PXE ports in this cluster.

write the switch config to startup config to make it permanent;

wr

End of Document 
================


