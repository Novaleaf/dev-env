# ***USING VNC (or Teamviewer) for bastion host access is OBSOLETE***

# why obsolete?
- teamviewer is stuck at a low res (1300x780 or something)
- vnc+gnome (ubuntu 1804 default) uses too much ram and seems to have a memory leak (1.4gb on idle after an hour)

# what instead?
If you can do everything you need via console (no gui), use byobu.  
- http://byobu.co/
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-byobu-for-terminal-management-on-ubuntu-16-04

# obsolete instructions below

if we ever revisit gnome, try this process: https://eklitzke.org/lobotomizing-gnome

## setup a bastion host 

interact with vm's without external ip's




***NOTE: the following process works, but isn't prettied up yet for general use***
- since we only allow port 22 SSH access, a password for vncserver seems unnessicary
- it would be nice to setup multiple screen geometries, but "get stuff done"
- generalize ssh tunnel notes and give better explanation on what's happening


this process uses vnc to setup a persistent gui shell.  from there, you can open terminal windows to the host you need

### why VNC 
This process was adapted from this youtube vid: https://www.youtube.com/watch?v=sT9JUL7q2uM    This was chosen because we could actually control the screen resolution.    

Watch that video if you need a step-by-step

### why not Teamviewer
Initially tried to use teamviewer, as shown in this video:  https://www.youtube.com/watch?v=c8pXhTmxZRA   but could not figure out how to change the screen resolution so gave up on that.

### why not SSH
you can!  we use vnc+gnome as a nice "staging space" that can persist even when our server drops connection.  (could also use ```tmux``` or ```screen``` instead)

if you want a quick and easy SSH session:

- ```ssh``` to the bastion, then ```ssh -A INTERNAL_IP``` ***within 2 minutes of the first ssh call*** as described here: https://cloud.google.com/compute/docs/ssh-in-browser#known_issues
- or maybe (not tested yet on internal ip only) ```gcloud compute ssh instance_name --internal-ip```   see: https://cloud.google.com/sdk/gcloud/reference/beta/compute/ssh#--internal-ip




### CURRENT WORKFLOW: using tigervnc+modern gnome (ubuntu 1804)
because tightvnc requires password, which is annoying.   with tigerVnc we are ok with no password, because the vnc port is blocked by the firewall, and we access it only via ssh tunnel

instructions from: https://www.cyberciti.biz/faq/install-and-configure-tigervnc-server-on-ubuntu-18-04/





apt remove gnome-shell ubuntu-gnome-desktop autocutsel tightvncserver gnome-core gnome-panel gnome-themes-standard
sudo apt remove gnome-core gnome-panel synaptic ubuntu-desktop 


on bastion:

```bash
#apt remove tightvncserver
## the above deletes .vnc/xstartup, so need to make it again after installing tigervnc
apt install tigervnc-standalone-server tigervnc-xorg-extension tigervnc-viewer
## use "--no-install-recommends" to avoid installing things like libreoffice, firefox , thunderbird more info here: https://askubuntu.com/questions/53822/how-do-you-run-ubuntu-server-with-a-gui
sudo apt-get install --no-install-recommends ubuntu-desktop
#if you want lightweight desktop, can try instructions from here: https://medium.com/google-cloud/graphical-user-interface-gui-for-google-compute-engine-instance-78fccda09e5c
# apt install --no-install-recommends gnome-core gnome-panel synaptic

##maybe don't need below:
#systemctl enable gdm
#systemctl start gdm



### use the modern gnome default theme.  
## not using flashback theme, because it's buggy with settings (opening the settings menu reverts any settings you changed via cmdline!!!)
cat > ~/.vnc/xstartup <<EOF
#!/bin/sh
# Start Gnome 3 Desktop 
xsetroot -solid grey 
export XKL_XMODMAP_DISABLE=1 
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
vncconfig -iconic &
dbus-launch --exit-with-session gnome-session --disable-acceleration-check &
EOF



#disable screen lock: https://stackoverflow.com/questions/28281077/how-do-i-disable-the-gnome-desktop-screen-lock
## also look here: https://ask.fedoraproject.org/en/question/36256/how-do-i-disable-the-gnome-lock-screen/
## and look here: https://www.fosslinux.com/3893/how-to-disable-screen-locking-in-ubuntu-18-04-lts.htm
## unset dbus session first, as per: https://askubuntu.com/questions/457016/how-to-change-gsettings-via-remote-shell
# unset DBUS_SESSION_BUS_ADDRESS 


#get lockscreen settings, from: https://askubuntu.com/questions/1048774/disabling-lock-screen-18-04
gsettings get org.gnome.desktop.lockdown disable-lock-screen 
sudo gsettings set org.gnome.desktop.session idle-delay 0
sudo gsettings set org.gnome.desktop.lockdown disable-lock-screen true
gsettings set org.gnome.desktop.session idle-delay 0
gsettings set org.gnome.desktop.lockdown disable-lock-screen true
gsettings get org.gnome.desktop.lockdown disable-lock-screen
## see for explanation on dbus-launch: https://askubuntu.com/questions/323776/gsettings-not-working-over-ssh
dbus-launch gsettings set org.gnome.desktop.session idle-delay 0
dbus-launch gsettings set org.gnome.desktop.lockdown disable-lock-screen true

## another potential, but doesn't seem to work: https://askubuntu.com/questions/1041230/how-to-disable-screen-locking-in-ubuntu-18-04-gnome-shell
gsettings set org.gnome.desktop.screensaver lock-enabled false
dbus-launch gsettings set org.gnome.desktop.screensaver lock-enabled false



## disable suspend lock: https://askubuntu.com/questions/1029696/disable-password-request-from-from-suspend-18-04
## check current setting
gsettings get org.gnome.desktop.screensaver ubuntu-lock-on-suspend
## set to false
gsettings set org.gnome.desktop.screensaver ubuntu-lock-on-suspend false

#disable gnome animations: https://askubuntu.com/questions/654976/gnome-3-14-how-to-disable-window-effects
gsettings set org.gnome.desktop.interface enable-animations false


#remove online accounts
sudo apt remove --purge gnome-online-accounts

apt remove --purge gnome-calendar

################  disable evolution bloat, from: https://askubuntu.com/a/694515/267280
cd /usr/share/dbus-1/services
# This part create a copy of your original files
sudo cp org.gnome.evolution.dataserver.AddressBook.service org.gnome.evolution.dataserver.AddressBook.service.backup
sudo cp org.gnome.evolution.dataserver.Calendar.service org.gnome.evolution.dataserver.Calendar.service.backup
sudo cp org.gnome.evolution.dataserver.Sources.service org.gnome.evolution.dataserver.Sources.service.backup
sudo cp org.gnome.evolution.dataserver.UserPrompter.service org.gnome.evolution.dataserver.UserPrompter.service.backup

# This part makes the trick
sudo ln -snf /dev/null  org.gnome.evolution.dataserver.AddressBook.service
sudo ln -snf /dev/null  org.gnome.evolution.dataserver.Calendar.service
sudo ln -snf /dev/null  org.gnome.evolution.dataserver.Sources.service
sudo ln -snf /dev/null  org.gnome.evolution.dataserver.UserPrompter.service


##### disable tracker https://ask.fedoraproject.org/en/question/9822/how-do-i-disable-tracker-in-gnome/
gsettings set org.freedesktop.Tracker.Miner.Files enable-monitors false
gsettings set org.freedesktop.Tracker.Miner.Files crawling-interval -2
gsettings set org.freedesktop.Tracker.Miner.Files ignored-files ["'*'"]




```

### HOW TO RUN VNC?

- we keep the vnc port blocked.   (only open port on the machine is SSH:22) so need to create a SSH tunnel.
  - so locally (on your dev machine) run:    ```gcloud compute ssh jasons@vnc-bastion-usc --project pjsc-proxy-direct-20190303 --ssh-flag "-L 5901:localhost:5901"```  this also has the side effect of opening a normal ssh window to the bastion.
  - on the bastion SSH session make sure no other VNC servers are running:   ```vncserver -list```
  - if no other servers, start one:  ```vncserver -geometry 3600x2000 -SecurityTypes None```   
      - this is using a no password option from: https://serverfault.com/questions/376302/tigervnc-ssh-without-a-vnc-password
      - also ***IMPORTANT: do not use "-depth 16" option as it causes some gui to crash***  instead just set the vncclient to 256 color mode.
  - the vncserver will stay up until the server reboots.  if you need to kill the vncservers ```vncserver -kill :*```
      

- on your dev machine, use vncclient to connect to ```localhost:5901```  (this connects via the ssh tunnel done in the first step above)
   - if the screen is too big, you can resize it smaller in gnome options:  ```Gnome-->Settings-->Devices-->Displays-->Resolution```

- Other Notes:
  - vncserver doesn't work under ```crontab @reboot```.   For some reason gnome doesn't load properly (blank gui), probably something to do with dbus.
  - vncserver logs can be found at ```/home/jasons/.vnc/vnc-bastion-usc.c.pjsc-proxy-direct-20190303.internal:1.log``` 
  - can increase gnome log verbosity by adding ```--debug``` to the ```gnome-session``` line in ```~/.vnc/xstartup```




### errors with .XAuthority?
 see https://www.digitalocean.com/community/questions/timeout-in-locking-authority-file-home-username-xauthority
basically, it seems to be because you ran vncserver and configed it from root.   make sure you do this as normal user otherwise can't ever run as normal user
due to the config files being locked to "root" access only.

after messing this up, here's what I basically did to get it working for user again:

```bash
pushd ~
sudo chown --recursive jasons:jasons .
```




### scratch area while setting up / debugging VNC server
```bash
echo $DBUS_SESSION_BUS_ADDRESS 
cat ~/.vnc/xstartup

cat > ~/.vnc/xstartup <<EOF
#!/bin/sh 
autocutsel -fork 
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey 
export XKL_XMODMAP_DISABLE=1 
unset DBUS_SESSION_BUS_ADDRESS 
gnome-session --disable-acceleration-check --debug &
EOF

vncserver -geometry 1920x2000 -depth 16 -SecurityTypes None

 vncserver -kill :*

touch ~/.Xresources

cat > ~/.vnc/xstartup <<EOF
#!/bin/sh 
autocutsel -fork 
xrdb $HOME/.Xresources 
xsetroot -solid grey 
export XKL_XMODMAP_DISABLE=1 
export XDG_CURRENT_DESKTOP="GNOME-Flashback:Unity" 
export XDG_MENU_PREFIX="gnome-flashback-" 
unset DBUS_SESSION_BUS_ADDRESS 
gnome-session --session=gnome-flashback-metacity --disable-acceleration-check --debug &
EOF

vncserver -kill :*

 
vncserver -geometry 1920x2000 -depth 16 -SecurityTypes None




cat > ~/.vnc/xstartup <<EOF
#!/bin/sh 
# extra settings from youtube vid
xrdb $HOME/.Xresources 
xsetroot -solid grey 
export XKL_XMODMAP_DISABLE=1 
export XDG_CURRENT_DESKTOP="GNOME-Flashback:Unity" 
export XDG_MENU_PREFIX="gnome-flashback-" 
# unset DBUS_SESSION_BUS_ADDRESS 
# Start Gnome 3 Desktop 
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
vncconfig -iconic &
# docs on dbus-launch: https://linux.die.net/man/1/dbus-launch
dbus-launch --exit-with-session gnome-session --session=gnome-flashback-metacity --disable-acceleration-check &
EOF

```

### ***OBSOLETE*** old steps, getting VNC working with tightvncserver and old gnome desktop theme

***why this is obsolete***:  because tightvncserver doesn't allow no-password logins, and the old gnome theme messes up gnome settings (it overrides settings you pass to cmdline, such as disabling lock screen)

- install vnc client, from: https://www.tightvnc.com/

- setup ubnutu 1804
  - need at least 10gb, probably use a 2core and decrease to 1core-micro when done with setup
- run os config script (install helpers, etc)
- run the following on the bastion
```bash
apt-get install gnome-shell ubuntu-gnome-desktop autocutsel tightvncserver 
touch ~/.Xresources
apt get install gnome-core gnome-panel gnome-themes-standard


# run vncserver, set password interactively: https://www.systutorials.com/39549/changing-linux-users-password-in-one-command-line/
sudo echo -e "smokingg\nsmokingg" | sudo vncserver

cat >> /home/jasons/.vnc/xstartup <<EOF
#!/bin/sh 
autocutsel -fork 
xrdb $HOME/.Xresources 
xsetroot -solid grey 
export XKL_XMODMAP_DISABLE=1 
export XDG_CURRENT_DESKTOP="GNOME-Flashback:Unity" 
export XDG_MENU_PREFIX="gnome-flashback-" 
unset DBUS_SESSION_BUS_ADDRESS 
gnome-session --session=gnome-flashback-metacity --disable-acceleration-check --debug &
EOF

##kill running server
vncserver -kill :1
## start the server
vncserver -geometry 1920x1080


```

-  on local dev machine create a ssh tunnel between the bastion's vnc port and a local port
   ```
   gcloud compute ssh jasons@vnc-bastion-usc --project pjsc-proxy-direct-20190303 --ssh-flag "-L 5901:localhost:5901"
   ```
- then use vncclient to connect to ```localhost:5901```