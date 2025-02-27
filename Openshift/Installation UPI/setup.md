OpenShift Installation on-prem using UPI Online Method
======================================================

Git repo to pull the installation stuff
---------------------------------------------------------
https://github.com/asimehsan/devops-v...

HW Specs:
-----------------
haproxy 1GB RAM, 20GB Disk, 1 CPU, 2 NIC
master1 24GB RAM, 150GB Disk, 4 CPU and 1 NIC
master2 24GB RAM, 150GB Disk, 4 CPU and 1 NIC
master3 24GB RAM, 150GB Disk, 4 CPU and 1 NIC
worker1 32GB RAM, 200GB Disk, 4 CPU and 1 NIC
worker2 32GB RAM, 200GB Disk, 4 CPU and 1 NIC


For VM's on ESXi configure disk.EnableUUID:
----------------------------------------------------------------------
Open the Host Client, and log in to the ESXi.
After power-off the VM's, right-click the virtual machine, and choose Edit Settings.
Click VM Options tab, and select Advanced.
Click Edit Configuration in Configuration Parameters.
Click Add parameter.
In the Key column, type disk.EnableUUID.
In the Value column, type TRUE.
Click OK and click Save.
Power on the virtual machine


redhat developer account login with 60 days trial:
-------------------------------------------------------------------------------
https://console.redhat.com/openshift


OpenShift Admin, sshkey Share, Router, Loadbalancer, DHCP and DNS Setup:

1. Install RHEL/CentOS 9
----------------------------------------

2. Disable firewalld and selinux
-------------------------------------------------

3. Enable NAT on Loadbalancer 
--------------------------------------------------
dnf install nftables iptables-services (Install Iptable)
systemctl enable nftables 
systemctl enable iptables
systemctl start nftables
systemctl start iptables 
iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE (enable natting on ens192 interface)
iptables -t nat -L -v -n (If you want to list iptables rules)
service iptables save (To make the iptables rules persistent)
iptables -t nat -F (If you want to clear the iptable rules)
vim /etc/sysctl.conf (File to update kernel features)
net.ipv4.ip_forward = 0 (Add this in sysctl.conf)
sysctl -p (Apply the new kernel feature)


4. Install and configure BIND DNS:
------------------------------------------------------
dnf install bind bind-utils -y

Apply configuration files
cp dns/named.conf /etc/named.conf
cp -R dns/zones /etc/named/

Enable and start the service
systemctl enable named
systemctl start named
systemctl status named

At the moment DNS will still be pointing to the LAN DNS server. You can see this by testing with dig cluster1.local.

Change the LAN nic (ens192) to use 127.0.0.1 for DNS AND ensure Ignore automatically Obtained DNS parameters is ticked

Restart Network Manager
systemctl restart NetworkManager

Confirm dig now sees the correct DNS results by using the DNS Server running locally

dig cluster1.local
The following should return the answer ocp-bootstrap.lab.ocp.lan from the local server
dig -x 10.10.10.1

5. Install & configure DHCP:
--------------------------------------------

Install the DHCP Server
dnf install dhcp-server -y

Edit dhcpd.conf from the cloned git repo to have the correct mac address for each host and copy the conf file to the correct location for the DHCP service to use

cp dhcpd.conf /etc/dhcp/dhcpd.conf

Enable and start the service

systemctl enable dhcpd
systemctl start dhcpd
systemctl status dhcpd

6. Install & configure HAProxy:
------------------------------------------------

dnf install haproxy -y

Copy HAProxy config

cp haproxy.cfg /etc/haproxy/haproxy.cfg

Enable and start the service

systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
