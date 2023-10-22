# GRPCurl:
This is a guide for setting up automatic rebooting of your Starlink device. 

## Sources Cited:
- https://github.com/fullstorydev/grpcurl
- https://github.com/sparky8512/starlink-grpc-tools
- https://crontab.guru/
  
## prerequisites:
I set this up in a proxmox LXC environment.
Can be installed on any linux device with cron and GRPCurl installed. 

## Steps:
1. Download GRPCurl: `curl -OL https://github.com/fullstorydev/grpcurl/releases/download/v1.8.8/grpcurl_1.8.8_linux_x86_64.tar.gz`
2. Extract the GRPCurl tar file: `sudo tar -C /usr/local -xvf grpcurl_1.8.8_linux_x86_64.tar.gz`
3. Navigate to the extracted location `cd /usr/local`
4. Run GRPCurl: `./grpcurl -help`
5. At this stage you should see the GRPCurl output a help screen with a list of commands.
6. Create a shell file called `reboot.sh` in the /usr/local path we extracted the GRPCurl files to. `cd /usr/local` then `nano reboot.sh`
7. Inside the `reboot.sh` file we need to specify the command for the reboot for cron: 
```
cd /usr/local
./grpcurl -plaintext -d {\"reboot\":{}} 192.168.100.1:9200 SpaceX.API.Device.Device/Handle
```
8. Run cron: `crontab -e`
9. If options appear choose option 1.
10. At the bottom of the cron file add the following: `0 4 * * * /usr/bin/bash /usr/local/reboot.sh` 
*Note: this will reboot every morning at 4am, use the crontab.guru calculator to find values you need*

Thanks for reading :) 
Questions or concerns, reach out to me on discord @shiroe_2000
