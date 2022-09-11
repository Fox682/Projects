### Encrypted and Cached DNS for your Local Network

This setup allows your DNS queries to be encrypted and cached at the local network level

Requirements:
  * Access to your Wifi Router/Home Router
    * Ability to change your DNS settings
  * Virtual Machine/Spare Computer or Raspberry Pi (B or Better recommended)
    * Install [PiHole](https://docs.pi-hole.net/main/basic-install/) on this system
    * Install and setup VPN client for PiHole (Automated installs I've used! [OpenVPN](https://github.com/angristan/openvpn-install), [Wireguard](https://github.com/angristan/wireguard-install) <- Preferred)
    * Use `dig` on Pihole to verify DNS is working over OpenVPN
      * As root: `apt install dnsutils` (dig needs to be installed on Debian)
    * Optional: Setup a VPS with Unbound to be a multi-purpose VPN hub for the Network
      * Roll your own Source for Encrypted DNS!
      * Use as a normal VPN to access your Home Network!
      * Need to install Unbound and Wireguard/OpenVPN to connect with Pihole

#### The Following Steps are not exact as your network setup is different from mine.
This information serves as a guide to give you concepts to setup your network, as usual the standard disclaimer applies. (Including but not limited to; not being responsible for lost data/damage/inconvenience use at your own risk etc.)

#### Step One 
Verify that you have access to your Wifi/Home Router.
