# Starlink Network Diagram:
This is a beginners guide to installing a third party router to Starlink and the types of approaches. 

## Sources Cited:
- https://www.asus.com/us/support/FAQ/1011706/
- https://www.tp-link.com/us/support/faq/560/
- https://starlinkinsider.com/starlink-bypass-mode/
- https://en.wikipedia.org/wiki/Network_address_translation

## Example Architectures: 

### Dish V2 and Router V2
![alt text](/Images/starlink2.drawio.png)

## Steps:

1. Locate the "WAN" port on your third party router. 
2. Connect the Starlink Ethernet Adapter to your WAN port with the third party router. 
3. Log into your Starlink app on your mobile device, select settings, then select "Bypass Mode" option listed below. Slide the bypass to enable it. 
4. With the Starlink router in bypass mode your third party router will now handle all the local routing. 
5. Login to your third party router's IP address and configure your router settings. 

### Notes: 
- Third party router static IP settings will vary per router model and brand. 
- Why use bypass mode? Starlink uses CG-NAT to assign private IP addresses to most plans. The Starlink router will perform a Network Address Translation (NAT), NAT allows a single ip to represent multiple devices with independent ip's (view the cited sources for more information). Starlink will in turn be a double NAT which can be problematic for certain devices. If you do not bypass the Starlink router you create a triple NAT, which can lead to more problems on your network. 
- Portforwarding: If you are not paying for a Public IP from Starlink, a third party router alone will not let you portforward. 
- Starlink Public IP's are not static, if you are paying for a public IP or using IPv6 the IP may change. It is a 1:! CG-NAT mapping. 
- Starlink support will be a headache to deal with when using third party devices. They will require you to detach the device and attach the basic starlink provided router for diagnosis.

Thanks for reading :) 
Questions or concerns, reach out to me on discord @shiroe_2000