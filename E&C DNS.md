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

Install the OpenVPN Client:

`apt install openvpn`

Install the Wireguard Client and tools: (if needed)

`apt install wireguard wireguard-tools`

If you have a downloaded file (ie a *.ovpn file or *.conf file for wireguard) and you need to transfer that file you'll need an SCP client. On linux it's as easy as `scp` but on other systems (windows) it's not straight forward. On linux if you need to transfer a file from your computer to your PiHole this should get you there.

If your file is located on your system in `/home/user/openvpnfile.ovpn` and your home folder on your pihole is `/home/pi/` then the command to transfer it using scp is `scp [source location] [destination location]`:

`scp /home/user/ovpnfile.ovpn pi@192.168.0.200:/home/pi/`

From there you can tell the openvpn client to use that file.

`openvpn --config /home/pi/openvpnfile.ovpn`

If you're using wireguard the equivalent command is: (this assumes that your config is in `/etc/wireguard/wg0.conf`)

`wg-quick up wg0`

**Set your PiHole to have a static IP address:**

When your set your router to an IP to use for DNS, you don't want your Pihole to "move" from it's current location to one you don't know!

You will need to modify the file `/etc/network/interfaces` to include the IP address to use on the network.

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

#source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
#allow-hotplug ens3
#iface ens3 inet dhcp

# Primary Network interface (Static)
auto ens3
iface ens3 inet static
        address 192.168.0.200
        gateway 192.168.0.1
```

The hash symbol `#` (or pound if you're old school like me) denotes that this line is a `comment` line, meaning it's just there for notes and not processed. Below is the contents of my interfaces file on my PiHole. My network interface is `ens3` and the modified config is under `# Primary Network Interface (static)`. Be sure no other computers are currently using the IP address you want to give it, mine is `192.168.0.200` that should be outside the DHCP range that routers normally assign IPs for, so it *should* be safe to use on your network also. Please note that your network interface will likely be different. To find yours on your pi (or linux machine) you can use:

```
user@pihole:~$ ip -br -c a
lo               UNKNOWN        127.0.0.1/8 ::1/128
ens3             UP             192.168.0.200/24 fe80::5054:ff:fe9e:e7d8/64
```

`ens3` is mine, yours will be listed, the `lo` is just a local loopback interface, ignore it for now. If your IP on your computer is 192.168.1.X instead of 192.168.0.X, be sure to account for that and your gateway IP will end in 1.1 instead of 0.1. somtimes they're different so be aware of that!

##### Warning!
  * You will need to verify that the requests are actually being sent through your VPN provider FROM the Pihole, your requests are being sent to the PiHole, but the PiHole will forward these requests to a provider of your choosing, you need to ensure these requests are being sent through the VPN.
  * There are various methods for doing this, you will need to find and use one.
  * Personally I used `termshark` to review the traffic to see if the dns requests are being sent over the encrypted link. How to use Termshark is outside the scope of this general guide.

From here we should be almost done, all that's left should be to setup the Pi-Hole's IP address to be the sole DNS provider for your network. So you will need to login to your Wifi/Home Router and change the DNS provider to be the PiHole. You may need to restart the router and/or your computer's wifi interface to get the new information.

Using a PI is recommended as it's a nice low-power device that can keep running 24/7. If your PiHole does down, your internet will look like it just went out. If your internet doesn't work **check your PiHole first** make sure it's online and talking to the internet.

If you're using the optional setup, then the PiHole will be pointed to your VPS as the DNS provider instead, see below if you want to go further down that rabbit hole!

Please note that this setup above is currently **untested** by me personally, but it *should* work. It will need tested and I need feedback as I will test this later. I use the optional setup which is somewhat easier to verify but more difficult to setup.


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

If you need a refresher on how to set this up, you can check out Digital Oceans tutorial on [How to setup SSH Key-Based Authentication](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)

It is **absolutely ok** to interact with your VPS and do the rest of the config from the PiHole if you're comfortable. If you do you will need generate a private/public key pair for the Pi if you do!

Make sure to generate a private/public keypair for your system if you plan on accessing your VPS, remember you only need to give the server your PUBLIC key information it's in the `id_rsa.pub` file.

This system faces the internet directly and you will need to take *some* precautions at the minimum.

**UFW Config:** Basic setup of UFW to allow SSH connection so you can setup the server.

**Run commands interacting with ufw as root!**

Setup default deny incoming packets `ufw default deny incoming`

Setup to allow outgoing packets `ufw default allow outgoing`

Enable ssh connections `ufw allow ssh`



#### Step Five

Install and setup [Wireguard](https://github.com/angristan/wireguard-install) on the VPS after setting it up with a firewall and ssh key authentication (seriously... do not forget that part).

I use this automated install of Wireguard as it sets up a very nice Wireguard install and allows you to setup the individual computers and mobile devices with not just the keys but different keys for EACH device which adds more security. More information about the setup and why at the github link, I've reviewed the install and wireguard configs that are created and they're consistent and standardized and have worked flawlessly for years on my setups.

When running the Wireguard Install script be sure to run it again after install to setup a new client, the script is really easy to use.

You will need to transfer the resulting `wg0-client.conf` (or similarly named file) from the VPS to your PiHole.

##### Note
You will need to take down the **port** number used in the VPN setup so that you can tell UFW to filter any DNS requests from anywhere except your VPN connection. This ensures your VPS will not be used by someone else on the public internet. This tells the VPS to only respond to DNS queries over the VPN and nowhere else! The port number assigned is **random**, so be sure to **write it down**.

You can set it to use a specific port number, I like to use 11111. Pick whatever number you like above 1024 and below 65,535. Have fun with it.

**UFW setup:** (did you write down the port number?)

`ufw allow 11111`

Tell ufw to let in traffic for DNS (port 53) but ONLY through the Wiregaurd interface

`ufw allow in on wg0 to any port 53`

If you did the above correctly you should be able to check the results:

Run: `ufw status numbered`

```
     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    Anywhere
[ 2] 22/tcp                     ALLOW IN    Anywhere
[ 3] 11111/udp                  ALLOW IN    Anywhere
[ 4] 53 on wg0                  ALLOW IN    Anywhere
[ 5] 22 (v6)                    ALLOW IN    Anywhere (v6)
[ 6] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 7] 11111/udp (v6)             ALLOW IN    Anywhere (v6)
[ 8] 53 (v6) on wg0             ALLOW IN    Anywhere (v6)
```

#### Step Six
Setup the PiHole to have access to the VPS via Wireguard and confirm the connection is usable


#### Step Seven
Setup Unbound on the VPS to process DNS queries.

Add DNS servers to the config (odd quirk here to note about loosing config if it restarts). Fix is to modify the interfaces file to include the requested DNS servers after reboot?


#### Step Eight
Setup the PiHole to use the VPS as it's DNS providers

Final step is to tell your Wifi/Home Router to use the PiHole as your DNS provider!
