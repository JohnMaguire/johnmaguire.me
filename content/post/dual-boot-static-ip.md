---
title: "Getting static IP addresses to work with Windows dual-boot"
date: "2016-01-09"
URL: "2016/01/static-ip-address-windows-dual-boot/"
---

Static IP addresses are an incredibly handy tool for maintaining your local network. However, when you have a machine that dual-boots between Windows and another operating system, you may notice that your non-Windows machine doesn't always receive the expected IP address.

It seems Windows doesn't like to give up its lease on the IP address when shutting down. As long as your router uses dnsmasq as its DNS server (dd-wrt, OpenWrt, etc.) you can simply add the following option to your dnsmasq configuration: `dhcp-option=vendor:MSFT,2,1i`

Windows will now release the DHCP lease during its normal shutdown process, and your second operating system should be able to attain its lease properly.
