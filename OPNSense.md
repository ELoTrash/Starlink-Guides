# OPNSense
- This is a beginners guide to using OPNSense with Private Internet Access (PIA) VPN to bypass CG-NAT that Starlink puts on its users.
- Huge shout out to the scripts and guides previously made to the community for making it simple to deploy a PIA tunnel on OPNSense. (FingerlessGlov3s)
- Note: PIA does not support portforwarding in the USA, I used canada as a close alternative and have no issues with latency for Minecraft / NGINX. 

## Prerequisites:
- Obtained a PIA subscription 
- installed OPNSense as your main router on Starlink (Starlink router in bypass mode). 
- OPNSense version 24.0 or newer.

## Steps:
1. You can follow the PIA guide [(Awesome PIA guide that I initially followed)](https://github.com/FingerlessGlov3s/OPNsensePIAWireguard/tree/main) for setting up PIA or follow the steps below. 
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
    3. Enter a username, I chose `PIA-WireguardAPI`
    4. Check the box that says `Generate a scrambled password to prevent local database logins for this user.` 
    5. Click Save
    ![alt text](/Images/PIA-Portforwarding/user%20config.png)
2. Once you click save you can begin adding permissions to the user. 
    1. Scroll up to the `effective Privileges` section and add the following: 
    - `Firewall: Alias: Edit` 
    - `Firewall: Aliases` (can be found by searching for "Alias")
    - `System: Static Routes`
    - `VPN: WireGuard` 
    - Select the `+` icon to create an API key, save this for a later step.
    - Scroll down and save the changes. 
    ![alt text](/Images/PIA-Portforwarding/user%20permissions%20and%20api%20key.png)
3. SSH to OPNSense:
    1. Open the SSH tool of your choice, I use PUTTY. 
    2. SSH to the IP of your OPNSense router (mine is `192.168.5.1`).
    3. Login with your `root` user and password you created when setting up OPNSense (same as logging into the router via WebUI).
    4. When prompted, `select option 8`. 
    ![alt text](/Images/PIA-Portforwarding/putty%20opnsense%20option.png)
    5. Inside the shell run the following commands: 
        - `fetch -o /conf https://raw.githubusercontent.com/FingerlessGlov3s/OPNsensePIAWireguard/main/PIAWireguard.py` 
        - `fetch -o /conf https://raw.githubusercontent.com/FingerlessGlov3s/OPNsensePIAWireguard/main/ca.rsa.4096.crt`
        - `fetch -o /usr/local/opnsense/service/conf/actions.d https://raw.githubusercontent.com/FingerlessGlov3s/OPNsensePIAWireguard/main/actions_piawireguard.conf`
    6. Download the latest release from the FingerlessGove3s repo to your machine. 
        - https://github.com/FingerlessGlov3s/OPNsensePIAWireguard/releases
    7. Edit the `PIAWireguard.json` configuration 
        - Open the OPNSense key that was previously downloaded, copy the values for the Key into the `opnsenseKey` line. 
        - Copy the secret value into the `opnsenseSecret` line. 
        - Insert your PIA username (The p####### value).
        - Insert your PIA password. 
        - OPNSense prefix, change this to `PIA`
        - Add your instancees, set the `portForward` section to `true`
        - For every instance, the `opnsenseWGPort` needs to increase by one. 
    8. Example configuration: 
```
    {
    "opnsenseURL": "https://127.0.0.1",
    "opnsenseKey": "xxx",
    "opnsenseSecret": "xxx",
    "piaUsername": "p########",
    "piaPassword": "YOURPASSWORD",
    "tunnelGateway": null,
    "opnsenseWGPrefixName": "PIA",
    "instances": {
        "Minecraft": {
            "regionId": "ca_toronto",
            "dipToken": "",
            "dip": false,
            "portForward": true,
            "opnsenseWGPort": "51815"
        },
		"Minecraft_Modded": {
            "regionId": "ca_toronto",
            "dipToken": "",
            "dip": false,
            "portForward": true,
            "opnsenseWGPort": "51816"
        },
		"Plex": {
            "regionId": "ca_ontario-so",
            "dipToken": "",
            "dip": false,
            "portForward": true,
            "opnsenseWGPort": "51817"
        }
    }
}
```
4. Configuration: 
    1. cd into the `/conf` folder
    2. Run `nano PIAWireguard.json` paste in your JSON configuration. 
    3. Hit `CTRL + X` then `y` to save it. 
     ![alt text](/Images/PIA-Portforwarding/nano.png)