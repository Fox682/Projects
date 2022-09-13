### Encrypted and Cached DNS for your Local Network

This setup allows your DNS queries to be encrypted and cached at the local network level. This is NOT to be confused with the commonly referred to as "DOH" (DNS Over HTTPS) or any Secure Transport of DNS directly. This is DNS over a VPN to prevent your ISP from knowing your DNS queries and Caching the DNS queries for your local network for faster access.

This setup is an intermediate to advanced difficulty depending on your experience with Linux. The Example provided in the Optional section is what I use personally. The simple VPN on the Pihole is currently **untested** by me. This guide may be updated and changed with no notice by me at anytime.


Requirements:
  * Access to your Wifi Router/Home Router
    * Ability to change your DNS settings
  * Virtual Machine/Spare Computer or Raspberry Pi (B or Better recommended)
    * Install [PiHole](https://docs.pi-hole.net/main/basic-install/) on this system
    * Install and setup a VPN client for PiHole (OpenVPN, [Wireguard](https://github.com/angristan/wireguard-install) <- Preferred, if using the VPS)
    * Use `dig` on Pihole to verify DNS is working
      * As root: `apt install dnsutils` (dig needs to be installed on Debian)
      * If using the Optional Setup, this will ensure the requests are going over the encrypted connection.
    * Optional (Advanced) Steps: Setup a VPS with Unbound to be a multi-purpose VPN hub for the Network
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

Install the Wireguard Client and tools: (if going down the VPS rabbit hole)

`apt install wireguard wireguard-tools`

**SCP Usage Crash Course**

If you have a downloaded file (ie a *.ovpn file or *.conf file for wireguard) and you need to transfer that file you'll need an SCP client. On linux it's as easy as `scp` but on other systems (windows) it's not straight forward. On linux if you need to transfer a file from your computer to your PiHole this should get you there.

If your file is located on your system in `/home/user/openvpnfile.ovpn` and your home folder on your pihole is `/home/pi/` then the command to transfer it using scp is `scp [source location] [destination location]`:

`scp /home/user/ovpnfile.ovpn pi@192.168.0.200:/home/pi/`

From there you can tell the openvpn client to use that file.

`openvpn --config /home/pi/openvpnfile.ovpn`

If you're using wireguard the equivalent command is: (this assumes that your config is in `/etc/wireguard/wg0.conf`)

`wg-quick up wg0` (without the .conf that's your interface name)

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

The hash symbol `#` (or pound if you're old school like me) denotes that this line is a `comment` line, meaning it's just there for notes and not processed. Above is the contents of my interfaces file on my PiHole (almost). My network interface is `ens3` and the modified config is under `# Primary Network Interface (static)`. Be sure no other computers are currently using the IP address you want to give it, mine is `192.168.0.200` that should be outside the DHCP range that routers normally assign IPs for, so it *should* be safe to use on your network also.

Please note that your network interface will likely be different. To find yours on your pi (or linux machine) you can use:

```
user@pihole:~$ ip -br -c a
lo               UNKNOWN        127.0.0.1/8 ::1/128
ens3             UP             192.168.0.200/24 fe80::5054:ff:fe9e:e7d8/64
```

`ens3` is mine, yours will be listed, the `lo` is just a local loopback interface, ignore it for now. If your IP on your computer is 192.168.1.X instead of 192.168.0.X, be sure to account for that and your gateway IP will end in 1.1 instead of 0.1 sometimes they're different so be aware of that!

You will need to set your DNS server in the PiHole web interface. Be sure to log into the PiHole and set the DNS servers you'd like to use. There are some pre-listed, but you can put in any DNS servers you've found there instead. This should be located in `Settings` then the `DNS` tab under `Upstream DNS Providers`. As long as the VPN connection is working, this *should* send the request over the VPN to those providers.

You can check using [DNSLeaktest](https://www.dnsleaktest.com/), the queries should ultimately be coming from your VPN provider, NOT your ISP or from multiple DNS providers.

##### Warning!
  * You will need to verify that the requests are actually being sent through your VPN provider FROM the Pihole, your requests are being sent to the PiHole, but the PiHole will forward these requests to a provider of your choosing, you need to ensure these requests are being sent through the VPN.
  * There are various methods for doing this, you will need to find and use one.
  * Personally I used `termshark` to review the traffic to see if the dns requests are being sent over the encrypted link. How to use Termshark is outside the scope of this general guide.

From here we should be almost done, all that's left should be to setup the Pi-Hole's IP address to be the sole DNS provider for your network. So you will need to login to your Wifi/Home Router and change the DNS provider to be the PiHole. You may need to restart the router and/or your computer's wifi interface to get the new information.

Using a PI is recommended as it's a nice low-power device that can keep running 24/7. If your PiHole does down, your internet will look like it just went out. If your internet doesn't work **check your PiHole first** make sure it's online and talking to the internet.

If you're using the optional setup, then the PiHole will be pointed to your VPS as the DNS provider instead, see below if you want to go further down that rabbit hole!

Please note that this setup above is currently **untested** by me personally, but it *should* work. It will need tested and I need feedback as I will test this later. I use the optional setup which is somewhat easier to verify but more difficult to setup.


#### Step Four (Optional Setup!)
The following steps are optional but I have been able to confirm that this is working. All DNS traffic forwarded from the PiHole goes through the Wireguard connection to the VPS (Virual private Server) and the VPS fetches the requests on it's end.

As an Added bonus the VPS will act as a hub for your own VPN network allowing you to access computers on your network attached to your VPN. It will also serve as a DNS provider to use while you are outside your normal network.

**Warning:** Do not do "illegal" things using your VPS. Any activity that violates the TOS/AUP for the VPS provider will cause them to destroy your VPS and depending on legality, have your information sent to authorities. You are encouraged to do research on VPS providers.

##### Notes:
You will need a VPS provider to be your endpoint from which all your DNS traffic will ultimately originate from.

  - VPS Providers I've used (simple $5/mo node works fine):
    * [Linode](https://www.linode.com/products/shared/) (Best)
    * [Digital Ocean](https://www.digitalocean.com/products/droplets) (Good)
    * [Vultr](https://www.vultr.com/products/cloud-compute/) (Cheaper, I use these guys as they're very good for the price)

**Setup your VPS**

Please include a firewall (such as `ufw`) and setup SSH Key Authentication on the system. Optionally setup Fail2Ban if needed. Also please disable login as root.

If you need a refresher on how to set this up, you can check out Digital Oceans tutorial on [How to setup SSH Key-Based Authentication](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)

It is **absolutely ok** to interact with your VPS and do the rest of the config from the PiHole if you're comfortable. If you do you will need generate a private/public key pair for the Pi if you do! This is a great option if you do not have a virtual machine or another linux/unix based computer around.

Make sure to generate a private/public keypair for your system if you plan on accessing your VPS, remember you only need to give the server your PUBLIC key information it's in the `id_rsa.pub` file.

This system faces the internet directly and you will need to take *some* precautions at the minimum.

**UFW Config:** Basic setup of UFW to allow SSH connection so you can setup the server.

**Run commands interacting with ufw as root!**

Setup default deny incoming packets `ufw default deny incoming`

Setup to allow outgoing packets `ufw default allow outgoing`

Allow ssh connections `ufw allow ssh`

Enable ufw. If you forgot to allow ssh, then then next time you try to login via ssh on your VPS, you are going to have a bad time, so **be sure you allow ssh**.

`ufw enable`



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
Setup the PiHole to have access to the VPS via Wireguard and confirm the connection is usable.

Install the Wireguard Client and tools on the PiHole if you haven't already.

`apt install wireguard wireguard-tools`

Generate a client wireguard.conf file by re-running the Wireguard script from the VPS (not the PiHole!).

```
root@debian:~# ls
wireguard-install.sh
root@debian:~# ./wireguard-install.sh
Welcome to WireGuard-install!
The git repository is available at: https://github.com/angristan/wireguard-install

It looks like WireGuard is already installed.

What do you want to do?
   1) Add a new user
   2) Revoke existing user
   3) Uninstall WireGuard
   4) Exit
Select an option [1-4]:
```

Verify that you've generated a config file.

```
root@vulp:~# ls -hal
-rw-r--r--  1 root root  325 Aug 25 19:16 wg0-client-pihole.conf
```

This `wg0-client-pihole.conf` needs to be transferred. If you setup the key-based authentication it should be straight forward, use SCP to transfer the file, you can use the PiHole for this.

**Note:** If you've used linux for a good length of time (if you have not, this is important!) you'll notice that the file is owned by root and since we can't login as root, we wont be able to transfer the file directly, we'll need to change the ownership and location for the file.

Assuming `user` is the username of the account on your VPS:

`chown user:user wg0-client-pihole.conf`

Move the file to the user account

`mv wg0-client-pihole.conf /home/user`

Login to your PiHole, using SCP you should be able to transfer a file with something similar to (change your IP to match your VPS):

`scp user@29.26.30.32:/home/user/wg0-client-pihole.conf /home/pi/`

Next is to transfer this file to the correct location with a simpler interface name so that you can fire up the interface and make sure it works!

**Do this as root**

`mv wg0-client-pihole.conf /etc/wireguard/wg0.conf`

Let's fire it up!

`wg-quick up wg0`

Next, run `ip a` to see if the interface has been created, if this worked you should see something like this with an entry for `wg0`.

```
root@debian:~# ip a

3: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.10.10.1/24 scope global wg0
       valid_lft forever preferred_lft forever
    inet6 fd42:42:42::1/64 scope global
       valid_lft forever preferred_lft forever
```

Ping it!

`ping -c 4 10.10.10.1`

If you get a response, you're successfully setup a wireguard link. So we now have a PiHole with a WireGuard link to the VPS. Onward!

#### Step Seven
Setup Unbound on the VPS to process DNS queries.

`unbound` is a DNS server that is relatively simple to setup, and since we're only using this for your network, it doesn't need a complex setup.

Install unbound as **root**:

`apt install unbound unbound-host`

Get the list of root DNS servers:

`curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache`

Protect the Program from being run by anything else:

`chown -R unbound:unbound /var/lib/unbound`

Lets enable the service after modifying the Unbound configuration file. Open the file and lets make a couple changes:

`nano /etc/unbound/unbound.conf`

The only change on the lines needed are:

`interface: 10.10.10.1`

and

`access-control: 10.10.10.0/24         allow`

Adjusting these lines ensure that the interface for the DNS is the WireGuard interface and to ensure that ONLY addresses from WireGuard can submit requests to the DNS server. The `unbound.conf` file is listed below.

```
server:

  num-threads: 2

  #Enable logs
  verbosity: 1

  #list of Root DNS Server
  root-hints: "/var/lib/unbound/root.hints"

  #Use the root servers key for DNSSEC
  auto-trust-anchor-file: "/var/lib/unbound/root.key"

  #Respond to DNS requests on all interfaces
  interface: 10.10.10.1
  max-udp-size: 3072

 #Authorized IPs to access the DNS Server
 # access-control: 0.0.0.0/0                 refuse
 # access-control: 127.0.0.1                 allow
  access-control: 10.10.10.0/24         allow

  #not allowed to be returned for public internet  names
  #private-address: 10.200.200.0/24

  # Hide DNS Server info
  hide-identity: yes
  hide-version: yes

  #Limit DNS Fraud and use DNSSEC
  harden-glue: yes
  harden-dnssec-stripped: yes
  harden-referral-path: yes

  #Add an unwanted reply threshold to clean the cache and avoid when possible a>
  unwanted-reply-threshold: 10000000

  #Have the validator print validation failures to the log.
  val-log-level: 1

  #Minimum lifetime of cache entries in seconds
  cache-min-ttl: 1800

  #Maximum lifetime of cached entries
  cache-max-ttl: 14400
  prefetch: yes
  prefetch-key: yes
```

Next step is to enable the `unbound` service and ensure that it's running on the system.

```
root@debian:~# systemctl start unbound
root@debian:~# systemctl status unbound
● unbound.service - Unbound DNS server
     Loaded: loaded (/lib/systemd/system/unbound.service; enabled; vendor prese>
     Active: active (running) since Mon 2022-09-12 18:34:22 MDT; 7s ago
       Docs: man:unbound(8)
    Process: 17603 ExecStartPre=/usr/lib/unbound/package-helper chroot_setup (c>
    Process: 17606 ExecStartPre=/usr/lib/unbound/package-helper root_trust_anch>
   Main PID: 17609 (unbound)
      Tasks: 2 (limit: 250)
     Memory: 9.7M
        CPU: 35ms
     CGroup: /system.slice/unbound.service
             └─17609 /usr/sbin/unbound -d -p

Sep 12 18:34:22 deb systemd[1]: Starting Unbound DNS server...
Sep 12 18:34:22 deb unbound[17609]: [17609:0] notice: init module 0: subnet
Sep 12 18:34:22 deb unbound[17609]: [17609:0] notice: init module 1: validator
Sep 12 18:34:22 deb unbound[17609]: [17609:0] notice: init module 2: iterator
Sep 12 18:34:22 deb systemd[1]: Started Unbound DNS server.
Sep 12 18:34:22 deb unbound[17609]: [17609:0] info: start of service (unbound 1
```

We now have a running Unbound service on our VPS that responds to ONLY the WireGuard interface. Next lets make sure it actually works and tell the PiHole to use this as the DNS provider, we're almost done!

**Optional Step**

You can setup DNS providers on your VPS if you have a preference, this will ultimately be the source of your DNS queries. Most VPS providers will automatically give you the DNS they use.

If you want to see what that is you can `cat` or `nano` the `resolv.conf` file on the VPS, ie.

`nano /etc/resolv.conf`

You can modify this file however it will be overwritten if the server is rebooted. A more permanent change is to edit the the `/etc/network/interfaces` file to add DNS entries there (recommended).

Add `dns-nameservers [ip] [ip]` line below the `iface` line (file shows some unfiltered DNS servers, beware! Use OpenDNS servers if you prefer):

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens3
iface ens3 inet dhcp
dns-nameservers 89.233.43.71 76.76.2.0 #MODIFY THIS LINE TO THE FILE
```

Onward!

#### Step Eight
Verify DNS is working from the VPS and the WireGuard interface. Setup the PiHole to use the VPS as it's DNS provider.

Let's go back to the PiHole Command Line. Be sure you installed `dig`, if you forgot, lets go ahead and do it now.

As root: `apt install dnsutils` (dig needs to be installed on Debian)

Let's use `dig`!

`dig @10.10.10.1 random.com`

```
root@deb:~# dig random.com

; <<>> DiG 9.16.27-Debian <<>> random.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17462
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 9f52705a58a3caba7c03e0e1631fd4661b53bde17410223e (good)
;; QUESTION SECTION:
;random.com.                    IN      A

;; ANSWER SECTION:
random.com.             604792  IN      A       207.21.195.86

;; AUTHORITY SECTION:
random.com.             172791  IN      NS      ns1.salepage.com.
random.com.             172791  IN      NS      ns2.salepage.com.

;; ADDITIONAL SECTION:
ns1.salepage.com.       165728  IN      A       66.113.234.96
ns2.salepage.com.       165728  IN      A       5.1.88.159

;; Query time: 19 msec
;; SERVER: 10.10.10.1#53(10.10.10.1)
;; WHEN: Mon Sep 12 18:52:54 MDT 2022
;; MSG SIZE  rcvd: 160
```
When you run this on your PiHole, you should have an entry where it says `SERVER:` under `Query time:` it should show the WireGuard address of your VPS. If this works, then the DNS queries are able to go over the wireguard interface to your VPS. Yay!

Make sure the PiHole is set to use your VPS as the DNS server using the WireGuard interface.

  * Login to your Pihole
  * Goto `Settings` on the left
  * Goto `DNS` Tab on the top
  * Under Upstream DNS Servers change the DNS IP to your WireGuard interface ie. `10.10.10.1`
  * Goto the Bottom Right of the Page hit `SAVE`
  * Go back to the `System` Tab on the top, click `Restart DNS resolver`

This will ensure the changes take effect.

We can make sure that DNS is working properly by running `dig` on your systems to make sure DNS is being routed properly.

`dig` according to my desktop linux system: (goes to the PiHole to filter the requests)

```
;; Query time: 19 msec
;; SERVER: 192.168.0.200#53(192.168.0.200)
;; WHEN: Mon Sep 12 18:52:54 MDT 2022
;; MSG SIZE  rcvd: 160
```

`dig` according to the PiHole: (goes to the VPS via WireGuard!)

```
;; Query time: 19 msec
;; SERVER: 10.10.10.1#53(10.10.10.1)
;; WHEN: Mon Sep 12 18:52:54 MDT 2022
;; MSG SIZE  rcvd: 160
```

Make sure your Wifi/Home Router is using the ip address of your PiHole as it's sole DNS Provider! You may need to restart your router and/or your wifi network interface(s) for the changes to take effect.

Login to the PiHole to view the Dashboard to ensure you're getting queries from devices.


That should do it!

Now you have a VPS with a WireGuard VPN that you can route all your DNS requests to your PiHole which can filter your queries over an encrypted connection for all your devices on your network.

As a bonus the VPN setup works even if your mobile! It will not only give you DNS responses as long as your on the VPN (get the wireguard app for your Phone!), No more DNS leaks! It will also give you access to the PiHole and other computers on your network that are also on your VPN.

**Optional**

You can modify the Unbound Conf file to change the lifetime of the cached entries if you want the DNS server to hold on to the queries for a longer time, if you have any significant number of devices on your network, this will help keep things responding quickly.
