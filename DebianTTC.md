## Debian Tap to Click  
Tap to Click for Touchpad  

Enable Tap to Click for Debian with Libinput
- Find this file libinput.conf (XX-libinput.conf... etc)  

```
#find / | grep libinput.conf
```  
Should show a location similar to this:  
```
/usr/share/X11/xorg.conf.d/40-libinput.conf
```  

Add this section to the end of the file:  

``` 
#Added for Tap to Click
Section "InputClass"
        Identifier "libinput tablet catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Option "Tapping" "on"
EndSection
```  

Tap to click should now be enabled for the touchpad. May need to logout/login or reboot to complete
