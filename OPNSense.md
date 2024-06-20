# OPNSense
This is a beginners guide to using OPNSense with Private Internet Access (PIA) VPN to bypass CG-NAT that Starlink puts on its users.
Huge shout out to the scripts and guides previously made to the community for making it simple to deploy a PIA tunnel on OPNSense. (FingerlessGlov3s)

## Prerequisites:
- Obtained a PIA subscription 
- installed OPNSense as your main router on Starlink (Starlink router in bypass mode). 
- OPNSense version 24.0 or newer.

## Steps:
1. You can follow the PIA guide [(Awesome PIA guide that I initially followed:)](https://github.com/FingerlessGlov3s/OPNsensePIAWireguard/tree/main) for setting up PIA or follow the steps below. 
NOTE: you can only have 1 port per VPN tunnel on PIA, with OPNSense you can have multiple VPN tunnels created to have multiple ports, some edits to the config will be needed for this. 
2. Navigate to `System -> Firmware -> Plugins`, here search for the `os-wireguard` plugin.
Click the `+` symbol to install the Wireguard Plugin.  
![alt text](/Images/PIA-Portforwarding/wireguard%20plugin.png)
3. Ensure proper configurations inside OPNSense:
Navigate to `System -> Settings -> Administration`:
    1. Ensure that HTTPS is enabled. ![alt text](/Images/PIA-Portforwarding/administration%20-%20https%20enabled.png)
    2. Enable Secure Shell (Scroll down from the HTTPS section to find the SSH section.)
    3. Permit root user login
    4. Permit password login. 
    5. Listen Interfaces: LAN, it is a good idea to change this so SSH can only be accessed from your main LAN network. 
    ![alt text](/Images/PIA-Portforwarding/administration%20-%20ssh%20enabled.png)
### Setting up PIA: 
1. We will need a new API user to run the script from:
    1. Go to System -> Access -> Users
    ![alt text](/Images/PIA-Portforwarding/system%20-%20pia%20api%20user.png)
    2. Click the orange plus in the top. 
    3. Enter a username, I chose `PIA-APIUser`
    4. Check the box that says `Generate a scrambled password to prevent local database logins for this user.` 
    5. Click Save
    ![alt text](/Images/PIA-Portforwarding/user%20config.png)