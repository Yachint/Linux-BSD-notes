# Helper Document
Some tips on setting up certain features from a Clean/Base install. Instructions are given with ID like AB1234 to easily list if one process is dependent on the other and vice versa. Some instructions might be based on openrc init system. Please find suitable commands in runit, s6 or systemd to follow along.

OpenRC cheatsheet for Systemd commnads: https://wiki.gentoo.org/wiki/OpenRC_to_systemd_Cheatsheet

## Fix SSH terminal backspace/ text issue with ZSH (ZS101)
Sometimes when using ZSH + SSH on remote systems, the backspace and other keys are not interpretted properly resulting to a mangled mess. To fix this, just add this line to the .zshrc:

```
export TERM=xterm-256color
```

## Make scripts executable from anywhere! (ZS001)
Usually when we create small scripts we need to go to their direct location to execute them and this can be cumbersome if the file is deeply nested.

One solution to this is to designate a folder on your drive where you can point your shell to check for the program/ script directly without providing the full path. This is done through what is called a '.zprofile' file which is placed in the home folder.

This file is always read at login and is used for commands and variables which should be set once or which don't need to be updated frequently.

Example:
```
PATH="$PATH:/$HOME/.local/bin"
```
This will be always read at login and make sure that whatever programs/ scripts are located in '.local/bin/' are gloabally available.


## Lock screen implementation (LK133)
Traditionally, when we install a desktop environment, we assume that the login screen and lock screen are one and the same. But this is only the case in instances where the full desktop environment is involved.

For example, if we use a window manger like DWM and specially a terminal based display/session manager like 'Ly', then that kind of synergy will not be possible, i.e once we suspend or lock the screen, we will go back to the login screen since to do that, the current X-server would have to be shut down along with DWM to again launch the login manager (kind of like Sign-out on Windows/Mac)

### Solution?
We build our own Lock screen! Sounds cumbersome but it is not. We can build it with the help of many packages that can be used together.

Packages required:
- xss-lock - for auto detecting that system enetered suspend/sleep state
- i3lock-color - for creating the lockscreen and locking the Window manager
- imagemagick - for constructing the lockscreen image
- scrot - for taking screenshot required for blurred lockscreen image creation
- xautolock (optional) - if you want to auto-sleep + lock after certain time
- xidlehook (optional) - if you want to lock after certain idle time

```
paru -S xss-lock i3lock-color imagemagick scrot
```

NOTE: We will put the scripts mentioned below in the '.local/bin/' folder for global access. See **ZS001** for more info.

***Lock Script***
This script focuses on locking the screen, much like 'Win key + L' in Windows. 

```
#!/bin/bash

# Delete previous screenshot
rm /tmp/screenshot.png  

# Take new screenshot
scrot /tmp/screenshot.png

# Add blur to it using imagemagick
convert /tmp/screenshot.png -blur 0x20 /tmp/screenshotblur.png

# Add Arch linux banner on top of the blur for final lock-screen picture
convert /tmp/screenshotblur.png /home/yachint/.local/bin/banner.png -gravity center -composite -matte /tmp/screenlock.png

# Use the image with i3lock to immediately lock the screen
i3lock -i /tmp/screenlock.png
```

We can manually run this script by just entering 'lock' in the terminal (since it is located in .local/bin for global access) or we can run a custom service that will always run this script when the computer enters sleep/hibernate.

### **--->THIS SECTION IS SYSTEMD SPECIFIC!<---**

First, create the 'locker.service' file here:
```
cd /etc/systemd/system
sudo nvim locker.service
```
And write these contents to the file:
```
#/etc/systemd/system/lock_screen@.service
[Unit]
Description=Activate lock screen when leaving active state
Before=suspend.target
Before=sleep.target
Before=hibernate.target
Before=suspend-then-hibernate.target

[Service]
User=yachint
Type=forking
Environment=DISPLAY=:0
ExecStart=/home/yachint/.local/bin/lock
ExecStartPost=/usr/bin/sleep 2

[Install]
WantedBy=suspend.target
WantedBy=sleep.target
WantedBy=hibernate.target
WantedBy=suspend-then-hibernate.target

```

## Mount NTFS drives on kernel 5.16 and upwards (NT006)
After 5.16 (not exactly sure when this change happened...), we have to manually add udev rule to allow us to mount the ntfs file systems to fix the error shown below:

```
mount: /mnt: unknown filesystem type 'ntfs'
```

Steps are as follows:

1. Edit or create this file
   ```
   nvim /etc/udev/rules.d/ntfs3_by_default.rules
   ```
2. Add the lines shown below
   ```
   SUBSYSTEM=="block", ENV{ID_FS_TYPE}=="ntfs", ENV{ID_FS_TYPE}="ntfs3"
   ```
3. Reboot the system
   ```
   sudo reboot
   ```
4. Create a folder in /mnt (optional)
   ```
   sudo mkdir /mnt/ntfs1
   ```
5. Mount the disk (use lsblk to check the disk nomenclature)
   ```
   sudo mount -t ntfs3 /dev/sdc1 /mnt/ntfs1
   ```
6. Unmount after use (its not 'un-mount', its 'u-mount')
   ```
   sudo umount /mnt/ntfs1
   ```

References:
- https://wiki.archlinux.org/title/NTFS
- https://www.reddit.com/r/linuxquestions/comments/smubo1/ntfs_and_516_kernel/
- https://phoenixnap.com/kb/mount-ntfs-linux

## Support other language fonts in browsers (FN001)
By default, the font enabled by base install only supports english and few other major languages. To support other languages like Hindi, Japanese, Korean etc, we need to install 2 font families that cover almost all the languages available out there.

```
pacman -S noto-fonts noto-fonts-cjk
```

## Suspend with DWM not working (SU111)
This primarily happens due to not having a display manager (Login Manager is a more appropriate term) installed due to which system does not know how to restart xserver which leads to the system not able to continue our dwm session.

Simple solution is to install a login/display manager like LightDM. Steps for LightDM install are mentioned at **LD312**

References:
- https://forum.artixlinux.org/index.php/topic,1662.0.html

## Install fonts Manually (FN004)
Fonts have to be manually installed if you don't have a desktop env. already set up and have created your system by base install.

- Download and unpack your fonts in some folder
- Create the folder shown below if not already there and move all the fonts there:
```
mkdir ~/.local/share/fonts
mv ttf/Hack-Regular.ttf ~/.local/share/fonts/Hack-Regular.ttf
```

- Then clear and regenerate your Font cache:
```
fc-cache -f -v
```
- To test if font has been installed and to also get the name by which it has installed:
```
fc-list | grep "Hack"
```

References:
- https://medium.com/source-words/how-to-manually-install-update-and-uninstall-fonts-on-linux-a8d09a3853b0


## Add Arch repositories in Artix (RP101)
By default, Artix only ships with a fixed number of repos like:
- system
- world 
- galaxy

To add other community made packages like LightDM greeters, we need to add other repos which are maintained by Arch like:
- extra
- community
- multilib
- universe

To add them, we first need to install the Arch linux support package:
```
pacman -Syu artix-archlinux-support
```

Then, we need to edit the pacman.conf file:
```
nvim /etc/pacman.conf
```

And add these lines (these lines are also printed after installation of arch linux support package):

```
+ #[testing]
+ #Include = /etc/pacman.d/mirrorlist-arch
+ 
+ 
+ [extra]
+ Include = /etc/pacman.d/mirrorlist-arch
+ 
+ 
+ #[community-testing]
+ #Include = /etc/pacman.d/mirrorlist-arch
+ 
+ 
+ [community]
+ Include = /etc/pacman.d/mirrorlist-arch
+ 
+ 
+ #[multilib-testing]
+ #Include = /etc/pacman.d/mirrorlist-arch
+ 
+ 
+ #[multilib]
+ #Include = /etc/pacman.d/mirrorlist-arch
```

After that, write and come back to the terminal to run this command which will populate the trusted keys for these repos in our trust db:

```
pacman-key --populate archlinux
```

References:
- https://dev.to/nabbisen/artix-linux-add-arch-linux-repos-extra-community-35ab
- https://wiki.artixlinux.org/Main/Repositories#Universe

## Nvidia screen tearing fix (NV700)
This is caused due to vsync not being enabled by default on both the compositor (picom in my case) and on driver level. It is more efficient to enable through driver level than through a compositor. We can run the below mentioned command to enable it:

```
nvidia-settings --assign CurrentMetaMode="nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"
```

NOTE: This will reset on startup so will have to include in a script that runs on startup. Check the instruction for **LD312** for more info.

References:
- https://www.reddit.com/r/archlinux/comments/858q6h/nvidia_screen_tearing/

## Cannot edit crontab since "/bin/sh: /usr/bin/vi: No such file or directory" (CE002)

This happens since the editor is not defined properly for sudo/ root operations. Solution is mentioned below.

```
sudo visudo
```
Then add this line:
```
Defaults env_keep += "EDITOR"
```

Reference:
- https://stackoverflow.com/a/64166227
- https://wiki.gentoo.org/wiki/Cron#Installation

## Terminal/Vim Color helper links (CL001)
In order to precisely set color in terminal/vim, we can either set colors with name like 'DarkBlue' or to be more precise, we can use the color chart which consists of various numbers that can be used.

```
*cterm-colors*

NR-16   NR-8    COLOR NAME 
0       0       Black
1       4       DarkBlue
2       2       DarkGreen
3       6       DarkCyan
4       1       DarkRed
5       5       DarkMagenta
6       3       Brown, DarkYellow
7       7       LightGray, LightGrey, Gray, Grey
8       0*      DarkGray, DarkGrey
9       4*      Blue, LightBlue
10      2*      Green, LightGreen
11      6*      Cyan, LightCyan
12      1*      Red, LightRed
13      5*      Magenta, LightMagenta
14      3*      Yellow, LightYellow
15      7*      White
```

Color Chart: 
- https://upload.wikimedia.org/wikipedia/commons/1/15/Xterm_256color_chart.svg


## Fix no logs being generated by programs like CRON (SY003)
When we do a base install of artix/arch specially with an init system other than systemd, we do not get a system logger pre-installed which has the responsibility to write logs for various programs that rely on calls like SYSLOG().

**For systemd** -> we have syslogd to take care of system logs

**For runit, openrc, S6** -> no such mechanism is inbuilt as their ideology is to be focused on 1 job only that is to be the main process to starts other daemons. Other 'bloat' is not included.

To fix this, we need to install a custom syslog implementaion called syslog-ng which will delegate this task for us.

```
pacman -S syslog-ng syslog-ng-openrc
sudo rc-update add syslog-ng default
sudo rc-service syslog-ng start
```

Reference:
- https://forum.artixlinux.org/index.php/topic,4380.0.html
- https://wiki.gentoo.org/wiki/Syslog-ng
- https://askubuntu.com/questions/911692/crontab-xinput-returns-empty-results

## LightDM setup with DWM (LD312)
***! 
[IGNORE IF NOT ON ARTIX LINUX]
This process is liked with instruction **RP101**, please first carry out that to follow instructions here. 
!***

The main components are:
- lightdm
- lightdm-webkit2-greeter (Login theme/GUI)

```
sudo pacman -S lightdm lightdm-webkit2-greeter
```

After that we basically need to create a desktop entry that would start dwm when we login. The catch is that like .xinitrc, we need to launch various other programs like nitrogen and picom before launching dwm. For that we will give the executable as a script rather than just 'dwm'.

---
***It is very important to chmod +x both the desktop entry and the script to make it so that LightDM can execute them after we login with our user since by default only root user can do it*** 

---

### I. Creating the XSession entry for DWM 
Firstly we will go to the directory which stores the entry's for XSessions that the Display Manager can read. If this directory is not available go ahead and create it.

```
cd /usr/share/xsessions
```
Next up is creating the actual entry:

```
sudo vim dwm-session.desktop
```

Contents:
```
[Desktop Entry]
Encoding=UTF-8
Name=dwm
Comment=Dynamic window manager
Path=/usr/local/bin/dwm-session.sh
Exec=/usr/local/bin/dwm-session.sh
TryExec=/usr/local/bin/dwm-session.sh
Icon=dwm
Type=XSession
```
Now make it executable by LightDM in the context of our user:
```
sudo chmod +x dwm-session.desktop
```

### II. Creating custom startup script for DWM & its dependencies
As we saw in the previous step, we pointed to a file called "dwm-session.sh". We will create that file now.

Go to the directory:
```
cd /usr/local/bin
```

Create the script:
```
sudo vim dwm-session.sh
```

Contents:
```
#!/bin/bash
/home/yachint/dwm-bar/dwm_bar.sh &
setxkbmap us &
picom -f &
nitrogen --restore &
sxhkd &
xinput --set-prop 12 'libinput Accel Speed' -0.7 &
nvidia-settings --assign CurrentMetaMode="nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }" &
exec dwm
```
NOTE: For nvidia based command check instruction **NV700** and for xinput based command check **MA002**.

Change the execution permissions here also:
```
sudo chmod +x dwm-session.sh
```

References:
- https://www.reddit.com/r/linux4noobs/comments/rw2s5s/comment/hr96zx2/
- https://www.reddit.com/r/suckless/comments/jj61py/how_do_i_make_dwm_appear_on_my_display_manager/

### III. Add greeter session to Lightdm.conf
In order for Lightdm to identify and use our installed greeter, we have to specifically tell it to use it in the config file.

```
cd /etc/lightdm/ 
vim lightdm.conf 
```

Change the marked line as shown below
```
#xdmcp-manager=
#xdmcp-port=177
#xdmcp-key=
greeter-session=lightdm-webkit2-greeter <=====
#greeter-hide-users=false
#greeter-allow-guest=true
#greeter-show-manual-login=false
#greeter-show-remote-login=true
#user-session=default
#allow-user-switching=true
#allow-guest=true
#guest-session=
session-wrapper=/etc/lightdm/Xsession
#greeter-wrapper=
#guest-wrapper=
display-setup-script=/etc/lightdm/display_setup.sh
#display-stopped-script=
```

After doing this change, LightDM will automatically start the relevant programs like picom, nitrogen and dwm-bar before starting dwm, exactly like in .xinitrc just that with a Display manager we get proper login screen and ability to log back into the session after sleep/hibernation.

## Sound Setup (SU004)
To enable sound we require these packages:
- alsa
- alsa-utils
- alsamixer (To control volume through terminal)
- pulseaudio (Required by browsers like Firefox and GUI Frontend)
- pavucontrol (GUI for controlling audio)

```
sudo pacman -S alsa alsa-utils alsamixer pulseaudio pavucontrol
```
### Volume control scripts
We can use custom scripts to control various aspects of sound like change default sink (playback-device), change volume on all apps etc. and that too dynamically using shell scripts linked to **pacmd**.

- Alias based approach
Can be used through terminal but is less modular as only the terminal can use it where alias is defined.

```
# Used to get list of active audio sinks/ sources based on their index
pa-list() { pacmd list-sinks | awk '/index/ || /name:/' ;}

# Once we run the previous command, we can get the index and 
# pass it to pa-set() to change the sink to the index provided
# for all the playback sources like Firefox, mpv etc.
pa-set() { 
	# list all apps in playback tab (ex: cmus, mplayer, vlc)
	inputs=($(pacmd list-sink-inputs | awk '/index/ {print $2}')) 
	# set the default output device
	pacmd set-default-sink $1 &> /dev/null
	# apply the changes to all running apps to use the new output device
	for i in ${inputs[*]}; do pacmd move-sink-input $i $1 &> /dev/null; done
}

# To increase the volume of currently active sink
pa-vol-inc() {
	input=$(pacmd list-sinks | awk '$1 == "*" { print $3 }') 
	pactl set-sink-volume $input +10%
}

# To decrease the volume of currently active sink
pa-vol-dec() {
	input=$(pacmd list-sinks | awk '$1 == "*" { print $3 }') 
	pactl set-sink-volume $input -10%
}

# We can call all this functions by the below listed aliases
alias vlist=pa-list
alias vset=pa-set
alias vinc=pa-vol-inc
alias vdec=pa-vol-dec
```

- Script based approach
This approach is more modular as it can be linked to various keybinds like DWM keybind to easliy control volume and sources. Below I will list some example scripts which can be modified based on our requirements.


**Switch audio to headphone**
```
#!/bin/sh

# In the list of sinks, find the one which has 'usb' in its name
# and get its index. Once that is done, other code is similar to
# the alias method mentioned above where we switch all audio sources 
# as well as the default sink to that index.
headphone_id=$(pacmd list-sinks | awk '/index/ || /name:/' | awk '/index/{u=$0}/usb/{print u}' | grep -oP ":\s+\K\w+") 

# list all apps in playback tab (ex: cmus, mplayer, vlc)
inputs=($(pacmd list-sink-inputs | awk '/index/ {print $2}'))

# set the default output device
pacmd set-default-sink $headphone_id &> /dev/null
	
# apply the changes to all running apps to use the new output device
for i in ${inputs[*]}; do pacmd move-sink-input $i $headphone_id &> /dev/null; done

```
---

**Switch audio to Speaker**
```
#!/bin/sh

# search for in-built sound card with "pci"
speaker_id=$(pacmd list-sinks | awk '/index/ || /name:/' | awk '/index/{u=$0}/pci/{print u}' | grep -oP ":\s+\K\w+") 

# list all apps in playback tab (ex: cmus, mplayer, vlc)
inputs=($(pacmd list-sink-inputs | awk '/index/ {print $2}'))

# set the default output device
pacmd set-default-sink $speaker_id &> /dev/null
	
# apply the changes to all running apps to use the new output device
for i in ${inputs[*]}; do pacmd move-sink-input $i $speaker_id &> /dev/null; done
```
Reference for scripts:
- https://www.youtube.com/watch?v=jhv-2pNWfr4

## Mouse Accelaration (MA002)
Find your mouse in the list of input devices using:
```
xinput list
```

Note down its ID and use the following command to decrease/increase speed (Range: -1 to 1):
```
xinput --set-prop 12 'libinput Accel Speed' -0.7
```