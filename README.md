# Tutorial_softether_client_Setup_linux
# 🌐 SoftEther VPN Client on Linux – Full Tunnel Routing Guide

SoftEther VPN behaves differently across operating systems.  
While *Windows and macOS automatically handle DHCP and routing, **Linux requires manual configuration* to achieve full-tunnel VPN behavior.

This guide walks through installing SoftEther VPN Client on Linux, creating a VPN connection, and configuring routes so *all traffic flows through the VPN*.

---

## 🖥️ OS Behavior Overview

| Operating System | VPN Routing Behavior |
|------------------|----------------------|
| Windows          | Full tunnel (automatic) |
| macOS            | Full tunnel (automatic) |
| Linux            | Split tunnel (manual routing required) |

> ⚠️ On Linux, a successful VPN connection does *not* automatically change the default route.

---

Step 1: Extract and Build SoftEther VPN Client
tar -xvf softether-vpnclient-*.tar.gz
cd vpnclient
make

Step 2: Start VPN Client Service
Start the service
sudo ./vpnclient start

Stop the service (if needed)
sudo ./vpnclient stop

Step 3: Enter VPN Client Command Mode
sudo ./vpncmd


Select:

2) Management of VPN Client


Press Enter to connect to localhost.

Step 4: Create Virtual Network Adapter (NIC)
Create a virtual NIC
NicCreate your_nic_name

Verify NIC creation
NicList


Ensure the NIC status is Enabled.

Step 5: Create VPN Connection Account
Create the VPN account
AccountCreate your_vpn_connection_name \
  /SERVER:your_vpn_server_ip:443 \
  /HUB:DEFAULT \
  /USERNAME:your_username \
  /NICNAME:your_nic_name

Set the account password
AccountPasswordSet your_vpn_connection_name

Connect to the VPN
AccountConnect your_vpn_connection_name

Check connection status
AccountStatusGet your_vpn_connection_name


If you see:

Session Status | Connection Completed (Session Established)


✅ VPN connection is active

Linux Networking Limitation (Important)

Unlike Windows and macOS, Linux:

Does not automatically assign an IPv4 address

Does not modify routing tables

Does not tunnel traffic automatically

⚠️ Manual network configuration is required

Step 6: Request IPv4 Address for VPN Interface
sudo dhclient your_nic_name

Verify interface
ifconfig your_nic_name


You should now see an IPv4 address assigned.

Step 7: Inspect Current Routes
ip route


If you see:

default via your_local_gateway dev wlan_interface


Your traffic is still routed through your ISP.

Step 8: Add Route to VPN Server (Critical)

Before changing the default route, ensure the VPN server remains reachable:

sudo ip route add your_vpn_server_ip/32 via your_local_gateway dev wlan_interface

Placeholders

your_vpn_server_ip → VPN server public IP

your_local_gateway → Router IP (e.g. 192.168.1.1)

wlan_interface → wlp3s0, eth0, etc.

Step 9: Remove Existing Default Route
sudo ip route del default via your_local_gateway


⚠️ Do not skip Step 8, or the VPN connection will drop.

Step 10: Add Default Route via VPN

Identify the VPN gateway (usually .1 after DHCP):

sudo ip route add default via your_vpn_gateway dev your_nic_name

Confirm routing
ip route


Expected output:

default via your_vpn_gateway dev your_nic_name

Step 11: Verify VPN Public IP
curl -4 ifconfig.me


Or:

curl icanhazip.com


Or via browser:

https://ifconfig.me

https://ipinfo.io

✅ The public IP should now match the VPN location.

Restore Internet (If Needed)
sudo ip route add default via your_local_gateway dev wlan_interface

Key Takeaways

SoftEther VPN works reliably on Linux

Linux uses split tunneling by default

Manual DHCP and routing are required for full tunneling

Windows and macOS handle routing automatically

Useful Tips

Use ip route frequently for troubleshooting

Always add a host route for the VPN server before deleting the default route

Save routing commands in a script for reuse

For persistence across reboots:

systemd-networkd

NetworkManager dispatcher hooks

Conclusion

With proper routing configuration, SoftEther VPN provides a stable and secure full-tunnel VPN connection on Linux.
This guide bridges the networking behavior gap between Linux and other operating systems.

Happy tunneling 🔐🌍
