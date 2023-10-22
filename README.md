# Tailscale:
This is a beginners guide to installing tailscale and adding routes for local IP traffic. 
I have been using this setup for over a year with no issues on my Starlink ISP to get around the limitations of CG-NAT or double NAT. 
## Documents Cited:
https://tailscale.com/kb/1019/subnets/
https://tailscale.com/kb/1103/exit-nodes/#:~:text=Open%20the%20Tailscale%20menu%20and,select%20Allow%20local%20network%20access

## prerequisites:
- Debian/Ubuntu VM setup and up to date. * I run my node on proxmox in my homelab.
- Subnet range of your network. *can be found with ipconfig on windows or ip addr on linux*

## Steps: 
1. Download and run the tailscale installer: `curl -fsSL https://tailscale.com/install.sh | sh` 
2. Download and trust the tailscale key file:  `curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/focal.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null` 
3. Start tailscale with: `sudo tailscale up`
4. Setup a exit node: `sudo tailscale up --advertise-exit-node`
### Local Traffic setup:
1.  Forward IPv4 network: `echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/systctl.d/99-tailscale.conf`
2.  Forward IPv6 network: `echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee 0a /etc/systctl.d/99-tailscale.conf
3.  Run the forwarding: `sudo systctl -p /etc/systcl.d/99-tailscale.conf`
4.  If you are using firewalld run the following: `firewall-cmd --permanent --add-masquerade`
5.  Advertise the routes to tailscale: `sudo tailscale up --advertise-routes=192.168.5.0/24`
   *Note: You will need to change the `192.168.5.0/24` to your local networks specific IPv4 addresses. You can specify specific addresses instead of the whole subnet if needed.*
