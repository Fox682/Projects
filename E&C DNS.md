### Encrypted and Cached DNS for your Local Network

This setup allows your DNS queries to be encrypted and cached at the local network level. This is NOT to be confused with the commonly referred to as "DOH" (DNS Over HTTPS) or any Secure Transport of DNS directly. This is DNS over a VPN to prevent your ISP from knowing your DNS queries and Caching the DNS queries for your local network.

This setup is an intermediate to advanced difficulty depending on your experience with Linux. The Example provided in the Optional section is what I use personally. The simple VPN on the Pihole is currently **untested** by me. This guide may be updated and changed with no notice by me at anytime.


Requirements:
  * Access to your Wifi Router/Home Router
    * Ability to change your DNS settings
  * Virtual Machine/Spare Computer or Raspberry Pi (B or Better recommended)
    * Install [PiHole](https://docs.pi-hole.net/main/basic-install/) on this system
    * Install and setup a VPN client for PiHole (OpenVPN, [Wireguard](https://github.com/angristan/wireguard-install) <- Preferred)
    * Use `dig` on Pihole to verify DNS is working
      * As root: `apt install dnsutils` (dig needs to be installed on Debian)
      * If using the Optional Setup, this will ensure the requests are going over the encrypted connection.
    * Optional: Setup a VPS with Unbound to be a multi-purpose VPN hub for the Network
      * Roll your own Source for Encrypted DNS!
      * Use as a normal VPN to access your Home Network!
      * Need to install Unbound and Wireguard/OpenVPN to connect with Pihole

#### The Following Steps are not exact as your network setup is different from mine.
This information serves as a guide to give you concepts to setup your network, as usual the standard disclaimer applies. (Including but not limited to; not being responsible for lost data/damage/inconvenience use at your own risk etc.)

#### Step One
Verify that you have access to your Wifi/Home Router.

Login to your router using your web browser and going to http://192.168.0.1 or http://192.168.1.1 etc.

Sign in to your router, by default the user/pass combo is normally admin/admin or some combination of random strings of characters in some cases. You'll need to check the manual that came with or grab the router and physically flip the unit over to read the info on the underside.

If you can sign in to the router and make changes, you're ready for the next step.

#### Step Two
Install Pihole on your Virtual Machine/Spare computer or Raspberry Pi.

I normally use Linux for all things and PiHole requires the use of Linux to install and Setup. Any decent computer (or Virtual Machine) or Rasbperry Pi system will work fine if using a Raspberry Pi I'd recommend a Pi B or better.

PiHole will serve as the DNS source and Cache that will translate those URLs to IP addresses for your computer. It will Cache entries that get used often, so a repeated query will be responded to instantaneously instead of waiting for a result from a DNS server on the internet (faster than cloudflare!). It will also filter results (if you set it up that way) to "blackhole" anything undesirable (such as ads and trackers, this is optional). If you plan on using any uncensored DNS providers upstream, then you will likely need this feature

Install [PiHole](https://docs.pi-hole.net/main/basic-install/) on this system

#### Step Three
Setup a VPN client on the PiHole so that all the traffic goes through the VPN including DNS requests.

This is one of those steps that depends entirely on the setup you choose. If you're using a VPN provider you will need to ensure you can setup the client on the PiHole. Since PiHole is a Debian Linux distribution you will need to find the information to install the client on Debian.

There is an OpenVPN client and a Wireguard Client that can be installed and use via the command line. If you are using a service such as ProtonVPN or NordVPN, you will need to find the OVPN file (for OpenVPN). I've used ProtonVPN and they have instructions for getting the file on their website.

##### Warning!
  * You will need to verify that the requests are actually being sent through your VPN provider FROM the Pihole, your requests are being sent to the PiHole, but the PiHole will forward these requests to a provider of your choosing, you need to ensure these requests are being sent through the VPN.
  * There are various methods for doing this, you will need to find and use one.
  * Personally I used `termshark` to review the traffic to see if the dns requests are being sent over the encrypted link. How to use Termshark is outside the scope of this general guide.

From here we should be done, please note that this setup above is currently **untested** by me personally, but it *should* work. It will need tested and I need feedback as I will test this later. I use the optional setup which is easier to verify but more difficult to setup.


#### Step Four (Optional Setup!)
This step is optional but I have been able to confirm that this is working. All DNS traffic forwarded from the PiHole goes through the Wireguard connection to the VPS (Virual private Server) and the VPS fetches the requests on it's end.

##### Notes:
You will need a VPS provider to be your endpoint from which all your DNS traffic will ultimately originate from.

  - VPS Providers I've used (simple $5/mo node works fine):
    * [Linode](https://www.linode.com/products/shared/) (Best)
    * [Digital Ocean](https://www.digitalocean.com/products/droplets) (Good)
    * [Vultr](https://www.vultr.com/products/cloud-compute/) (Cheaper, I use these guys as they're very good for the price)

**Setup your VPS**

Please include a firewall (such as `ufw`) and setup SSH Key Authentication on the system. Optionally setup Fail2Ban if needed. Also please disable login as root.

This system faces the internet directly and you will need to take *some* precautions at the minimum.

#### Step Five

Install and setup [Wireguard](https://github.com/angristan/wireguard-install) on the VPS after setting it up with a firewall and ssh key authentication (seriously... do not forget that part).

I use this automated install of Wireguard as it sets up a very nice Wireguard install and allows you to setup the individual computers and mobile devices with not just the keys but different keys for EACH device which adds more security. More information about the setup and why at the github link, I've reviewed the install and wireguard configs that are created and they're consistent and standardized and have worked flawlessly for years on my setups.

When running the Wireguard Install script be sure to run it again after install to setup a new client, the script is really easy to use.

#### Step Six
Setup the PiHole to have access to the VPS via Wireguard and confirm the connection is usable


#### Step Seven
Setup Unbound on the VPS to process DNS queries.

Add DNS servers to the config (odd quirk here to note about loosing config if it restarts)


#### Step Eight
Setup the PiHole to use the VPS as it's DNS providers

Final step is to tell your Wifi/Home Router to use the PiHole as your DNS provider!
