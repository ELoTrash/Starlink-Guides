# DEPRECATED 
this guide no longer works as mullvad has removed its portforwarding features. 
It should work with other VPN Providers!

First things first, you will need an OPNsense setup already running and configured. I have already had mine running for some time now. 

# Installing The Plugin
Navigate to System -> Firmware -> Plugins.

Install the "**_os-wireguard_**" plugin.  

# Configuring the endpoint
Once wireguard is installed navigate to VPN -> WireGuard -> Endpoints. 

In the endpoint section click on the + to create an end point. 
Name the endpoint whatever you would like, I named mine Mullvad_ATL since I am using the Atlanta Mullvad server. 
You can find the Mullvad server list [here.](https://mullvad.net/en/servers/)
Navigate to the server list and select the server you wish to use. I chose a server in Atlanta since my StarLink dish is using a POP in Atlanta.

 Example Mullvad server config: 
 
![mullvad-atlanta-image](https://user-images.githubusercontent.com/54679918/192128996-bd0afbe9-f808-4486-9553-13ce04e611cf.PNG)


In the public key section of the endpoint config add the Mullvad server you have selected's public key. 
Shared secret key, keep this empty. 
Allowed IP's: Add "0.0.0.0/0" and the IP address of your mullvad tunnel. 
To get the IP address of your Mullvad Tunnel run the following command: 
> "**_curl -sSL https://api.mullvad.net/wg/ -d account=[mullvad-account-number] --data-urlencode pubkey=[mullvad-public-key]_**" 

Input your account number with mullvad and the server you have selected's public key. 
In the Endpoint Address section add the Public IPV4 of the Mullvad server you selected. 
Endpoint Port: 51820
Keep alive: 20

Save your endpoint, you have now successfully created a WireGuard Endpoint

# Wireguard Local Config
Navigate to the VPN -> Wireguard -> Local and click + to create a local config. 
Enable the advanced mode as it will be needed for this part. 
* Name: name it anything, I named mine "Mullvad_ATL" 
* Public key: leave empty
* Private key: leave empty
* Listen port: leave empty
* MTU: Leave empty
* DNS server: leave empty
* Tunnel Address: add your tunnel address ran from the command, add only the IPV4 address (I did not configure IPv6)
* Peers: Select the endpoint name, mine is "Mullvad_ATL" 
* Check disable Routes.
* Gateway: Add any ipv4 here, I used "1.2.3.4". 

# Enable WireGuard
Navigate to VPN -> WireGuard -> General
Click Enable WireGuard box. 

# Interface Assignments
Navigate to Interfaces -> assignments 
Add the WireGuard interface and save it. 
Click on the name of the WireGuard interface. 

* Enable: Enable this
* Lock: enable this
* Description: I put mine as "WAN_MULLVAD_ATL"
* Save it.

# Gateway setup
Navigate to System -> Gateways and click on the + to add a gateway
* Name: I named mine "Mullvad_ATL"
* Interface: "WAN_Mullvad_ATL" 
* Address Family: IPv4
* IP address: "1.2.3.4" this is the gateway ip we created in the local config. 
* Enable the Far Gateway. 
* Monitor IP: Select a DNS that you are not using, for example use 9.9.9.9
* Save it. 

#Firewall NAT:
Navigate to Firewall -> NAT -> Outbound
Change the firewall type to outbound, save and apply.
Create a new Rule by clicking the +

* Interface: "WAN_Mullvad_ATL"
* TCP/IP version: IPv4
* Protocol: any
* Source address: I sent mine to a subnet I had created, this can be your LAN net, I put mine as my VPNSubnet net.
* Source Port: any
* destination address: any
* destination port: any
* Translation / Target: interface address.

Save and apply the new rule. 

# Firewall Rules: 
## Floating:

Navigate to Firewall -> Rules -> floating and click on the + to add a rule. 

* Action: pass
* Direction: out
* TCP/IP version: IPv4
* protocol: any
* Source: "WAN_Mullvad_ATL address"
* Destination: "WAN_Mullvad_ATL net" 
* Gateway: "Mullvad_ATL"

Save it and apply if needed. 

Click the + to add a second floating rule

* Action: Block
* Quick: Enable this
* Interface: WAN
* Direction: out
* TCP/IP Version: IPv4
* protocol: any
* source: any
* destination: any
* enable the advanced options
* Match local tag: "NO_WAN_EGRESS"

save and apply 

## Subnet Rules:
navigate to Firewall -> Rules -> VPN subnet (may be lan for you if you do not have VLAN's configured). 
Click the + to add a rule. 
* Action: Pass
* Quick: enable this
* Interface: VPNSubnet
* Direction: in
* TCP/IP version: ipv4
* protocol: any
* Source: VPNSubnet net
* Gateway: Mullvad_ATL
* Show advanced options:
* Set local tag: NO_WANN_EGRESS

Save and apply 


Congrats your WireGuard should be setup and working. 
