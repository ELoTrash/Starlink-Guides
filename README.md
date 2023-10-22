# Tailscale
This is a beginners guide to installing tailscale and adding routes for local IP traffic. 
I have been using this setup for over a year with no issues on my Starlink ISP to get around the limitations of CG-NAT or double NAT. 

##prerequisites
- Debian/Ubuntu VM setup and up to date. * I run my node on proxmox in my homelab.
- Subnet range of your network. *can be found with ipconfig on windows or ip addr on linux*

##Steps: 
1. download and run the tailscale installer:
`curl -fsSL https://tailscale.com/install.sh | sh` 
2. Download and trust the tailscale key file: 
`curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/focal.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null` 
3. start tailscale with:
