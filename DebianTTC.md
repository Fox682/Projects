## Debian Tap to Click  
Tap to Click for Touchpad

Enable Tap to Click for Debian with Libinput
- Find this file libinput.conf (XX-libinput.conf... etc)
- Add this

'''
#Added for Tap to Click
Section "InputClass"
        Identifier "libinput tablet catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Option "Tapping" "on"
EndSection
'''