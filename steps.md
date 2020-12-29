# General 

start with raspbian lite, boot it up so that the resize partition thing happens (tried to disable this but ran into trouble). then shutdown and create another primary partition with ext4 (give about 4 GB to each) using gparted or whatever from another machine.

then boot and run raspi-config

* change pw to music
* change hostname 
* disable wait for network on boot
* change wifi location to US
* change timezone
* enter wifi ssid 
* under interfacing disable serial console, enable port 
* enable ssh

update

    apt-get update 


change pi username to music. first make root password

    sudo passwd root

use 'music'.  

then logout 

    logout

and login as root 

them change name

    usermod -l music pi

change home dir

    usermod -m -d /home/music music

change group name

    groupmod --new-name music pi
    
reboot, then raspi-config and enable auto login, reboot

install git

    apt-get install git

configure

    git config --global user.email "..."
    git config --global user.name "..."
    


# setup PCM5102a audio driver

update kernel
 
    rpi-update 
    reboot 

# config.txt

comment:

    #dtparam=audio=on

uncomment:

    hdmi_force_hotplug=1
    dtparam=i2c_arm=on
    dtparam=i2s=on
    dtparam=spi=on
    
add these:

    dtoverlay=hifiberry-dac
    dtoverlay=gpio-poweroff,gpiopin=12,active_low="y"
    dtoverlay=pi3-miniuart-bt
    dtoverlay=midi-uart0 
    dtoverlay=pi3-act-led,gpio=24,activelow=on
    gpu_mem=64

# install packages 
    
    apt-get install zip jwm xinit x11-utils x11-xserver-utils lxterminal pcmanfm adwaita-icon-theme gnome-themes-standard gtk-theme-switch conky libasound2-dev liblo-dev liblo-tools python-pip mpg123 dnsmasq hostapd
    
python stuff 

    pip2 install Cython
    pip2 install pyliblo
    
install wiringpi

    git clone git://github.com/WiringPi/WiringPi.git
    cd ~/WiringPi
    sudo ./build
    
install pd 0.49

    wget -q -O - https://blokas.io/gpg.public.key | sudo apt-key add -
    sudo wget -q -O /etc/apt/sources.list.d/blokas.list https://blokas.io/blokas.list
    sudo apt-get update
    sudo apt-get install puredata-core=0.49.0-3

cherrypy, try running it again if you get an error

    pip2 install cherrypy==11.0.0
    
enable vnc in raspi-config (this will download software)

# config

    ? systemctl disable hciuart.service
    systemctl disable vncserver-x11-serviced.service
    systemctl disable dnsmasq.service
    ? systemctl disable hostapd.service
    ? systemctl disable dhcpcd.service
    ? systemctl disable wpa_supplicant.service

after startx, then run from terminal

    gtk-theme-switch2 /usr/share/themes/Adwaita

unmute hifi out in alsamixer to enable audio out
enable mic boost
enable line in 
enable mic in

make stuff in /root readable 

    chmod +xr /root
    
make sdcard and usb directories
    
    mkdir /sdcard
    chown music:music /sdcard
    
    mkdir /usbdrive
    chown music:music /usbdrive

allow sudo with no password

    sudo visudo

add this to end of file

    music ALL=(ALL) NOPASSWD: ALL
    
add this to /etc/fstab to mount the patches partition:

    /dev/mmcblk0p3 /sdcard  ext4 defaults,noatime 0 0
   
reboot and change owner

    chown music:music /sdcard 
    
enable rt.  in /etc/security/limits.conf add to end:

    @music - rtprio 99
    @music - memlock unlimited
    @music - nice -10
    
fiddle with pcmanfm till it works. some of this stuff gets put in config file, but others get stored who knows where.  in preferences uncheck "Display simplified user interface.." in Layout.  uncheck everything under auto mount in Volume Management. change home to /sdcard under Advanced.  uncheck "Add deleted files to wastebasket" in General

in /etc/systemd/system.conf add:

    DefaultTimeoutStartSec=10s
    DefaultTimeoutStopSec=5s

then 
    
    systemctl daemon-reload

# make it read only

clean up

    apt-get autoremove --purge
    
add to /boot/cmdline.txt

    fastboot noswap ro
 
remove fsck.repair=yes, add fsck.mode=skip

move /var/spool to /tmp
    
    rm -rf /var/spool
    ln -s /tmp /var/spool

in /etc/ssh/sshd_config

    UsePrivilegeSeparation no

in /usr/lib/tmpfiles.d/var.conf replace "spool 0755" with "spool 1777"

move dhcpd.resolv.conf to tmpfs
    
    touch /tmp/dhcpcd.resolv.conf
    rm /etc/resolv.conf
    ln -s /tmp/dhcpcd.resolv.conf /etc/resolv.conf
    
in /etc/fstab add "ro" to /boot and /, then add:

    tmpfs /var/log tmpfs nodev,nosuid 0 0
    tmpfs /var/tmp tmpfs nodev,nosuid 0 0
    tmpfs /tmp     tmpfs nodev,nosuid 0 0
    
stop time sync cause it is not working anyway.  (also causes issues with LINK when the clock changes abruptly). need solution

    timedatectl set-ntp false
    
reboot

# release

    pull latest Organelle_OS and deploy
    remove .viminfo
	remove git config
    clear command history
    run fsck
    reboot and test

