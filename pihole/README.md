# PiHole

PiHole is a filtering DNS server for your local network. 
On a fresh raspbian setup follow [one-step automated install](https://github.com/pi-hole/pi-hole/#one-step-automated-install).

## Setup
* Raspberry Pi with raspbian buster installed
* Configured for static IP address via router
* DHCP not enabled in PiHole
* DNS set in router for DHCP configured clients

Be sure, you gave the pi hole service a static IP via your router.
This is also good for nextcloud and other services!

## Installation
```bash
curl -sSL https://install.pi-hole.net | bash
```
Select proper network device and do not select DHCP.
As upstream DNS, you can use the IP address of your router.

## Router setup
Within Fritzbox go to (German interface used here)
```
Heimnetz -> Netzwerk -> IPv4-Adressen
```
and set the address of the PiHole device.

## Testing
go to http://YOUR_DEVICE/pihole or http://YOUR_DEVICE/admin and login with the password shown during installation.

Reconnect any device in your network and browse a site with ads on it.


