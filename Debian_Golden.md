## Debian Golden Image  
### Description  
Below includes the software and configuration info for create a base useful Debian Buster system. This is a prelude to automation with Ansible. 
  
    
Setup for Buster Debian 10

- NetInstall ISO
- Utilities and SSH only
- Flat filesystem (no swap on setup)
- User Accounts  
User:user  
Root:root  

Update Sources.list file
- nano /etc/apt/sources.list  
- Uncomment deb-src
- Add contrib and non-free to ends
- May need to replace file instead during automation

```
deb http://deb.debian.org/debian/ buster main contrib non-free
#deb-src http://deb.debian.org/debian/ buster main

deb http://security.debian.org/debian-security buster/updates main
#deb-src http://security.debian.org/debian-security buster/updates main

# buster-updates, previously known as 'volatile'
deb http://deb.debian.org/debian/ buster-updates main contrib non-free
#deb-src http://deb.debian.org/debian/ buster-updates main
```

### Install & Configure Software  
```
apt update && apt upgrade
apt install tmux htop ufw fdisk cloud-guest-utils
```  
#### Configure System  

**User Section:**

alias su='su -'

**Root Section**

- Enable ls colors
- Check Bashrc  
```
alias poweroff='systemctl poweroff'
alias reboot='systemctl reboot'  
```  
**UFW**
- Set defaults  
- First Setup  
```
ufw default deny incoming  
ufw default allow outgoing
```

Setup rules (ssh)
```
ufw allow ssh
```  

Enable Firewall
```
ufw enable

#Turn off with
ufw disable
```

Enable Logging
```
ufw logging on
#Logs are in /var/logs/ufw
```  

Reset and clear all rules  (if all goes weird)  
```
#Turn firewall off first!
ufw reset
```

Other Examples (Allow only from port/protocol)
```
ufw allow 8080/udp
Allow SSH
```  

Mosh?
- Good for Mobile
- Cannot do agent forwarding  

### Setup Swapfile  

Create a Swapfile for use as swap instead of a partition. VPS's usually don't use partitions.  
```
#AS ROOT
#Create the Swapfile itself
dd if=/dev/zero of=/swapfile bs=1024 count=1048576

#Make readonly for root user
chmod 600 /swapfile

#Make the file a swapfile
mkswap /swapfile

#Turn on Swap for the system
swapon /swapfile

#Goto /etc/fstab, append to end
/swapfile swap swap defaults 0 0

#Verify Swapfile is setup
swapon --show
```

-------

**Colorful Prompts Prompts**  
Append to the end of the .bashrc files of each user (root and user)

- User
```
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;90m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
```
- Root
```
PS1='\[$(tput bold)\]\[\033[38;5;1m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
```
**== Automation & TO DO's ==**  

**TO DO:**
- Adjust Bash History to keep all commands Longer?
-- Pretty involved, more thought needed for this.
-- Possible Solution:
https://stackoverflow.com/questions/338285/prevent-duplicates-from-being-saved-in-bash-history#answer-7449399


**Automation To Do:**
- Use Ansible
- Create Playbook for this

