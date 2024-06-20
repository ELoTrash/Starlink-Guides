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
    4. Run `chmod +x PIAWireguard.json` to make the file executable. 
    5. Restart the configd service `service configd restart`. 
    6. run the script: `/conf.PIAWireguard.py --debug`.
5. Interfaces:
    1. In the OPNSense WebUI, navigate to: `Interfaces -> Assignments` 
    2. At the bottom in the `Assign a new interface` section, select the new Wireguard tunnel that you created, Click `Add`. 
    3. NOTE: If you created multiple tunnels like I did this will need to be repeated for every tunnel. 
    4. Add a name for the Interface, I named mine `WAN_PIAMINECRAFT`, `WAN_PIAPLEX`, `WAN_PIAMINECRAFT_MODDED` to keep track of each interfaces usecase. 
    5. Check `enable interface`.
    ![alt text](/Images/PIA-Portforwarding/wan%20piamc.png)
    6. Click save. 
6. Gateways: 
    1. This step will need to be repeated for each VPN Tunnel you created. 
    2. Go to `System -> Gateways -> Configuration` Click the `+` at the bottom right.
    ![alt text](/Images/PIA-Portforwarding/gateways.png)
    3. Make sure `Disabled` is not checked.
    4. Insert a name for the gateway. 
    5. In the interface, select the `WAN_PIAWG_NAME`
    6. Check `Far Gateway` 
    7. Uncheck `Disable Gateway Monitoring` 
    8. Save and apply the changes.  
    ![alt text](/Images/PIA-Portforwarding/mcgateway.png)
    9. Navigate to your `WAN_DHCP` gateway and select the pencil icon to edit it. 
    10. Change the priority to 1, click save. 
    ![alt text](/Images/PIA-Portforwarding/defaultgateway.png)
7. Enable the PIA tunnel:
    1. In PUTTY, run the following command: 
    - `/conf/PIAWireguard.py --debug --changeserver instancename`
    2. In OPNSense WebUI, you should see the updates to the gateways. 
8. CronJob
    1. Navigate to `System -> Settings -> Cron` and click the `+` icon. 
    ![alt text](/Images/PIA-Portforwarding/cronjob.png)
    2. Set the `Minutes` to `*/5` 
    3. Set the `Hours` to `*`
    4. Set the `Command` to `PIA WireGuard Monitor Tunnels`
    5. Add a description. 
    6. Click save. 
    ![alt text](/Images/PIA-Portforwarding/piacronjob.png)
9. Update the MSS Settings: 
    1. Navigate to `Firewall -> Settings -> Normalization` and click the `+` symbol. 
    ![alt text](/Images/PIA-Portforwarding/normalization%20settings.png)
    2. Select your Interfaces in the `Interfaces` section, if you created multiple PIA interfaces select all of them here. 
    3. Enter a description of `Maximum MSS for PIA WireGuard Tunnel` (taken from FingerlessGlov3s guide).
    4. Max MSS to `1380`
    5. Click save. 
    6. Click Apply Changes. 
10. Creating the NAT rule: 
The FingerlessGlov3s guide does not include this section. I am continuing where he left off in hopes of helping others that are new to OPNSense. 
1. Navigate to `Firewall -> NAT -> Port Forward` and click the `+` symbol. 
![alt text](/Images/PIA-Portforwarding/firewallNAT.png)
2. In the `interface` section, select your `WAN_PIA` interface that you created. 
3. Make sure `TCP/IP Version` is `IPv4`. 
4. Protocol: `TCP/UDP` 
5. Destination: `WAN_PIA address` 
6. Destination port: Choose the ALIAS that is created by the PIA script, this is the port opened by the VPN. `PIA_Minecraft_port` is mine. 
7. Redirect target IP: set this to the local IP of the host that is hosting your server `192.168.x.x` 
8. Redirect target Port: set this to your target port, for Minecraft it is `25565`
9. Click Save
![alt text](/Images/PIA-Portforwarding/nat%20rule.png)