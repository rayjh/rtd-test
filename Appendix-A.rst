

Appendix - A Cheat Sheet
========================

Setting up the Deployer Node

-  Install Ununtu 14.04LTS or 16.04LTS to the deployer node. Insure SSH
   login is enabled.
-  apt-get update
-  apt-get install vim vlan bridge-utils
-  Bring down the interface to be used for providing PXE support.

   -  *sudo ifdown eth0* (eth0 used as example)
   -  add the following lines to the /etc/network/interfaces file

      -  auto br0
      -  iface br0 inet static

         -  address 192.168.16.1 (example address)
         -  netmask 255.255.255.0 (example netmask. This must match the
            submask specified by ipaddr-mgmt-network: in the config.yml)
         -  bridge\_ports eth0 (eth0 used as example)

   -  sudo ifup eth0

Installing the OpenPOWER Cluster Genesis Software

-  export GIT\_SSL\_NO\_VERIFY=1
-  Change the root password to passw0rd

   -  sudo passwd root -> passw0rd
   -  If the root account does not exist (is not unlocked);

      -  *sudo passwd root* (then enter passw0rd twice)
      -  *sudo -u root* (to unlock the root passwd) (to lock again after
         git clone: sudo passwd -l root)

-  sudo apt-get install git
-  git clone https://github.com/open-power-ref-design/cluster-genesis

Running the OpenPOWER Cluster Genesis Software

-  cd cluster-genesis
-  *./scripts/install.sh* (this will take a few minutes to complete)
-  *source scripts/setup-env* (if you restart your shell session, you
   need to re-execute this and the next line.
-  *export ANSIBLE\_HOST\_KEY\_CHECKING=False*
-  copy your config.yml file to the /cluster-genesis directory
-  cd playbooks
-  ansible-playbook -i hosts lxc-create.yml -K (create container. Verify
   container networks)
-  ansible-playbook -i hosts install.yml -K (begins cluster genesis)
   Allow several minutes to run.
-  After several minutes a prompt should appear;

   -  "Please reset BMC interfaces to obtain DHCP leases. Press <enter>
      to continue"

After Genesis completes, run the following to see the status/progress of
operating system load for each cluster node.

-  sudo cobbler status (from container /home/deployer/cluster-genesis)

Setting up networks on the cluster nodes;

-  ansible-playbook -i ../scripts/python/cluster-genesis/inventory.py
   gather\_mac\_addresses.yml -u root
   --private-key=~/.ssh/id\_rsa\_ansible-generated
-  ansible-playbook -i ../scripts/python/cluster-genesis/inventory.py
   configure\_operating\_systems.yml -u root
   --private-key=~/.ssh/id\_rsa\_ansible-generated

Accessing the deployment container;

-  

   -  To see a list of containers on the deployer;

      -  sudo lxc-ls

   -  To access the container;

      -  sudo lxc-attach -n yourcontainername

Note : this logs you in as root

alternately, you can ssh into the container;

To get the login information;

*ubuntu@wmdepos: grep "^deployer" ~/cluster-genesis/playbooks/hosts*

**

*deployer ansible\_user=deployer
ansible\_ssh\_private\_key\_file=/home/ubuntu/.ssh/id\_rsa\_ansible-generated
ansible\_host=192.168.0.2*

**

*ssh -i ~/.ssh/id\_rsa\_ansible-generated deployer@192.168.0.2*

Notes:

-  if you change the ip address of the container, (ie if you recreate
   the container) you may need to replace the cached ECDSA host key in
   the .ssh/known\_hosts file ;

   -  ssh-keygen -R container-ip-address

-  if you reboot the deployer node you need to restart the deployment
   container;

   -  *lxc-start -d-n <container name>*

Checking the Genesis Log

Genesis writes status and error messages to;
/home/deployer/cluster-genesis/log.txt

You can print this file;

cat /home/deployer/cluster-genesis/log.txt

You can monitor this file from another container window;

tail -f /home/deployer/cluster-genesis/log.txt

Checking the DHCP lease table

*deployer@ubuntu-14-04-deployer:~$ cat /var/lib/misc/dnsmasq.leases*

Logging into the cluster nodes;

from the deployer node (host namespace);

ssh -i ~/.ssh/id\_rsa\_ansible-generated userid-default@a.b.c.d

or as root;

ssh -i ~/.ssh/id\_rsa\_ansible-generated root@a.b.c.d #(as root -i not
needed from cluster nodes)

with password; from deployer or cluster node

ssh userid-default@a.b.c.d # password: password-default (from
config.yml)

