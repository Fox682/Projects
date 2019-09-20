## Debian Golden Image  
### Description  
Below includes the software and configuration info for create a base useful Debian Buster system. This is a prelude to automation with Ansible. 
  
    
Setup for Buster Debian 10

User:user  
Root:root

- NetInstall ISO
- Utilities and SSH only
- Flat filesystem (no swap on setup)

Update Sources.list file
- nano /etc/apt/sources.list

- Uncomment deb-src
- Add contrib and non-free to ends

```
deb http://deb.debian.org/debian/ buster main contrib non-free
#deb-src http://deb.debian.org/debian/ buster main

deb http://security.debian.org/debian-security buster/updates main
#deb-src http://security.debian.org/debian-security buster/updates main

# buster-updates, previously known as 'volatile'
deb http://deb.debian.org/debian/ buster-updates main contrib non-free
#deb-src http://deb.debian.org/debian/ buster-updates main
```

#### == Install & Configure Software ==
```
apt update && apt upgrade
apt install tmux htop ufw fdisk cloud-guest-utils
```  
### Configure System  

**User Section:**

alias su='su -'

Root Section
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

- Then Setup rules  
```
ufw allow ssh
```  

- Enable
```
ufw enable
```  

- Logging! (For use with Fail2Ban)  
```
ufw logging on
``` 

- Logs are stored in:  
```
/var/logs/ufw
```  

- Reset and clear all rules  (if all goes weird)  
ufw reset

Other Examples  
- Allow only from port/protocol  
- ufw allow 8080/udp

- Allow SSH
- Mosh? (No agent forwarding)


-------

- Setup Swapfile
-- See Swapfile Note
-- Adjust Swappiness to 10  


-------

**Bash Colorful Prompts** 

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


**Automation:**

- Do we need a user specifically for Ansible?
-- May be able to login with regular user and Key

- Need playbook for Setup
-- VM would only need SSHd & Python3 (default) installed
