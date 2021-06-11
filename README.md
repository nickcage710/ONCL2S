# ONCL2S
Oh No Covid Layer 2 Solution


The goal of this project is to create an easy way to deploy raspberry pi remote dropboxes that allow for remote nessus scanning, saving cost and time on setup.


The Lab: 

1. Home router branched from my main router
2. Intel NUC with VirtualBox running Detection Lab (4 VMs)***
3. Raspberry Pi

***Detection Lab was modified to connect directly to the public network and use static IPs, this allowed them to be discoverable by external hosts. 

Paths attempted: 

1. SSH Tunnel -- Scanned remote network, but was routed and missed some findings and a system
2. SSHuttle -- Not successful using default settings
4. OpenVPN -- OpenVPN-AS does not support ARM, so the L2 VPN feature was a no go
3. ZeroTier + OpenVSwitch -- Success
