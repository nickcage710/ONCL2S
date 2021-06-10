# OpenVSwitch Method
## Initial design by Jared Sutton

## Intro

Jared Sutton had recommended I look at ZeroTier for accessing the dropboxes, however the IP routing and access to local subnet seemed to be an issue. I presented this issue to Jared who took it as a challenge. The next day he had a rough write-up on how to achieve L2 access on a remote subnet. 

## Results 

The OpenVSwitch+ZeroTier (OZ) worked, Nessus found the same number of vulnerabilities through the tunneled connection as it did when locally connected to the lab network.  

## Specs

Jared originally ran this on CentOS, however since I was scolded by the senior testers for enjoying CentOS, I used Ubuntu to make people happy.  

This was installed on Raspberry Pi with an additional USB NIC  

## Assumptions

1. Raspberry Pi is imaged with Ubuntu
2. ZeroTier Network is created
3. Ubuntu has had initial setup completed (password changed, `apt update`, and `apt upgrade -y`)
4. Nessus is installed on an Ubuntu system (VM/Physical)
5. Scanner and Remote systems have access to internet
6. The remote network offers DHCP

## Terms

eth0 - OS Traffic, used for ZeroTier and other attacks  
eth1 - The bridge port/switch port, the port used by the remote system  
scanner - the nessus scanning box  
remote - the PBJ Raspberry Pi  

## Config

#### OS Setup

##### Install packages needed for config and testing
`apt install net-tools tcpdump  `

##### Verify the second NIC is working
`ifconfig eth1 up   `

#### ZeroTier

##### Install ZeroTier
`curl -s https://install.zerotier.com | sudo bash  `

##### The zerotier-one service will be started automatically after install completes.
##### Determine your ZT network ID, and run the following (replacing NETWORK_ID with the actual ID):
`zerotier-cli join NETWORK_ID`

##### Create a map for ZeroTier to connect to (replace NETWORK_ID with the actual ID): 
`echo "NETWORK_ID=zt0" > /var/lib/zerotier-one/devicemap`

##### Set ZeroTier to ethernet bridge 

![ethernet bridge settings](https://github.com/nickcage710/pbj/blob/master/images/2020-09-29_19-25.png)

##### Disable IP assignment globally and routing

![IP addresses and routing](https://github.com/nickcage710/pbj/blob/master/images/2020-09-29_19-25_1.png)

# Reboot


#### OpenVSwitch - Run the following steps to install, start, and configure OpenVSwitch:



##### Install/start OpenVSwitch by running the following commands:
`apt install -y openvswitch-switch  `

##### Enable the OpenVSwitch Service 
`systemctl enable --now openvswitch-switch  `


##### Run the following commands to create and configure the OVS bridge (replace "eth1" with your own local bridge NIC device name):

`ovs-vsctl add-br br-zt  `

`ovs-vsctl add-port br-zt eth1  `

`ovs-vsctl add-port br-zt zt0  `

`ovs-vsctl set bridge br-zt stp_enable=false`  



##### Because the local bridge NIC is excluded from NetworkManager, there's nothing to bring the NIC's link state up, so we'll need to create our own startup service to handle that.  Follow the steps below:

###### Create a file called /etc/systemd/system/zt-bridge-startup.service and populate it as follows (replace "eth1" with your own local bridge NIC device name):
    [Unit]
    After=network.target zerotier-one.service
    
    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStart=/usr/sbin/ifconfig eth1 up
    
    [Install]
    WantedBy=multi-user.target

###### Run the following to enable the service on boot:
`chmod 644 zt-bridge-startup.service`

`systemctl enable zt-bridge-startup.service`

### Reboot the system

#### On Scanner

##### Install ZeroTier
`curl -s https://install.zerotier.com | sudo bash  `

##### The zerotier-one service will be started automatically after install completes.
##### Determine your ZT network ID, and run the following (replacing NETWORK_ID with the actual ID):
`zerotier-cli join NETWORK_ID`

##### to force DHCP from Scanner
`dhclient`



# Results

![Results](https://github.com/nickcage710/pbj/blob/master/images/2020-09-29_18-52_1.png)

# Troubleshooting

#### For scanners that aren't getting dhcp or have stale dhcp or that were reconnected
`dhclient -r zt0`

# To-Do 

1. Minimum Specs for Pi 
2. Glances? 
3. Ansible Script to Automate
4. Parts List? 
5. Create service to start eth1/switchp port

# Cool Stuff

1. Total RAM Utilization during Nessus Scan on Pi - 500MB (NICE!)



Notes from 3/15/2021

1: add br-zt to a startup script 
2: add kali instructions for zero tier (git clone, make, make install, zerotier-one -d, zerotier-cli join ####) 




