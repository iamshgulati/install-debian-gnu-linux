# Install and configure Arch Linux


###### Author: Shubham Gulati

###### Hardware configuration:
```
Sony VAIO SVE15138CN
Intel HM76 Express Chipset
Intel® Core™ i7-3632QM CPU @ 2.20GHz × 8
2GB AMD ATI Radeon HD 7650M Graphics Card
8 GB DDR3 1600 MHz RAM
250GB Samsung SSD 750 EVO
1TB Seagate SSHD (installed in Optical Drive bay)
Qualcomm Atheros AR9485 Wireless Network Adapter
Foxconn / Hon Hai Bluetooth
Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller
```

## Prepration

###### Download Arch Linux ISO

https://www.debian.org/distrib/


###### Create a bootable USB installation device for Debian
```
sudo dd bs=4M if= of= status=progress && sync

if= path to iso file, for example: if=~/Downloads/debian.iso
of= mount point of usb media, for example: of=/dev/sdc
```

## Boot the installation medium

Point the current boot device to the drive containing the Debian installation media.


#### Enable administrator rights [if root account is not disabled]
```
su -

apt-get install sudo gksu

usermod -a -G sudo shubham

reboot
```


## Configure the system

#### Fix the kworker CPU bug

Find the interrupts with very high interrupt values
```
grep . -r /sys/firmware/acpi/interrupts/
```
Add the following code to /etc/rc.local to disable stray interrupts from userspace
```
sudo nano /etc/rc.local
```
```
# Disable stray GPE ACPI interrupts hogging the CPU
echo disable > /sys/firmware/acpi/interrupts/gpe13
```

#### Fix the keyboard backlight bug

If keyboard backlight does not turn on automatically on Sony VAIO laptops, add the following code to /etc/rc.local to enable it from userspace
```
sudo nano /etc/rc.local
```
```
# Enable keyboard backlight
echo 1 > /sys/devices/platform/sony-laptop/kbd_backlight
```

#### Optimize font rendering
```
sudo apt-get install fonts-liberation fonts-noto
```
Download Ubuntu Fonts from

http://packages.ubuntu.com/ttf-ubuntu-font-family

Install package with:
```
sudo dpkg -i ttf-ubuntu-font-family*.deb
```
After installing fonts, update fontconfig font cache with:
```
fc-cache -fv
```
Query the current font settings with:
```
fc-match monospace
```
Open the Gnome Tweak Tool, go to Fonts tab and change:
```
Monospace to "Ubuntu Mono Regular 11"
Hinting to "Slight"
Antialiasing to "Rgba"
```
Add font configuration to ~/.config/fontconfig/fonts.conf
```
mkdir -p ~/.config/fontconfig/
nano ~/.config/fontconfig/fonts.conf
```
Add following code to fonts.conf
```
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>
 <match target="font">
  <edit mode="assign" name="rgba">
   <const>rgb</const>
  </edit>
 </match>
 <match target="font">
  <edit mode="assign" name="hinting">
   <bool>true</bool>
  </edit>
 </match>
 <match target="font">
  <edit mode="assign" name="hintstyle">
   <const>hintslight</const>
  </edit>
 </match>
 <match target="font">
  <edit mode="assign" name="antialias">
   <bool>true</bool>
  </edit>
 </match>
  <match target="font">
    <edit mode="assign" name="lcdfilter">
      <const>lcddefault</const>
    </edit>
  </match>
</fontconfig>
```

#### Update sources.list

Edit /etc/apt/sources.list to include contrib and non-free repositories
```
sudo nano /etc/apt/sources.list
```
```
#

# main archive
deb http://ftp.us.debian.org/debian/ testing main contrib non-free
deb-src http://ftp.us.debian.org/debian/ testing main contrib non-free

# security archive
deb http://security.debian.org/debian-security testing/updates main contrib non-free
deb-src http://security.debian.org/debian-security testing/updates main contrib non-free

# testing-updates archive
deb http://ftp.us.debian.org/debian/ testing-updates main contrib non-free
deb-src http://ftp.us.debian.org/debian/ testing-updates main contrib non-free

# multimedia archive
deb ftp://ftp.deb-multimedia.org testing main non-free
```

Add multimedia archive verification keyring
```
sudo apt-get -y update && sudo apt-get --allow-unauthenticated install deb-multimedia-keyring && sudo apt-get -y update
```

#### Install hardware drivers

Linux Firmware: Intel CPU Microcode, Non-Free Firmware and some other things

Qualcomm Atheros AR9485 Wireless Network Adapter Firmware AND Foxconn / Hon Hai Bluetooth Firmware [Kernel driver: ath9k, Kernel modules: ath9k]

Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller Firmware [Kernel driver: r8169, Kernel modules: r8169]
```
sudo apt-get -y install firmware-linux firmware-linux-nonfree firmware-atheros firmware-realtek
```

#### Enable Firewall
```
sudo apt-get install gufw && sudo ufw default deny && sudo ufw enable
```

#### Keep the system updated
```
sudo apt-get -y update && sudo apt-get -y upgrade && sudo apt-get -y dist-upgrade
```

#### Hide the Grub Bootloader menu

Edit /etc/default/grub to change bootloader settings
```
sudo nano /etc/default/grub
```
Find the line
```
GRUB_TIMEOUT=5
```
Update it to
```
GRUB_TIMEOUT=0
```
Update Grub Bootloader
```
sudo update-grub
```

#### Install Bootsplash

Install Plymouth and Themes
```
sudo apt-get install plymouth plymouth-themes
```
Edit the file /etc/initramfs-tools/modules
```
sudo nano /etc/initramfs-tools/modules
```
```
# KMS
drm
radeon modeset=1
```
Edit the file /etc/default/grub
```
sudo nano /etc/default/grub
```
Search for the line #GRUB_GFXMODE=640x480 and uncomment it.
```
GRUB_GFXMODE=1366x768
```
Search for and edit the line GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```
Choose a theme for bootsplash

To display all installed themes run
```
/usr/sbin/plymouth-set-default-theme --list
```
Then, to set your desired theme run
```
sudo /usr/sbin/plymouth-set-default-theme spinfinity
```
Update plymouth configuration
```
sudo nano /etc/plymouth/plymouthd.conf
```

Update Grub Bootloader
```
sudo update-grub
```
Update initial RAM disk
```
sudo update-initramfs -u
```

#### Configure power saving on Intel CPU
```
sudo apt-get install tlp powertop gnome-power-manager
```

To check power statistics
```
powertop
```

#### Install temperature sensors
```
sudo apt-get install lm-sensors hddtemp

sudo sensors-detect

sensors

sudo hddtemp /dev/sd?
```

#### Reboot system to apply changes
```
sudo reboot
```

#### Tweak PulseAudio

Tweak PulseAudio to isolate master volume control from user applications

Update the code in /etc/pulse/daemon.conf
```
sudo nano /etc/pulse/daemon.conf
```
Find the line of code
```
; flat-volumes = yes
```
And update it to
```
flat-volumes = no
```

#### Troubleshoot bluetooth a2dp issues

In order to prevent gdm from capturing the a2dp sink on session start

Create a new file /var/lib/gdm3/.config/pulse/client.conf
```
sudo nano /var/lib/gdm3/.config/pulse/client.conf
```
Add following lines of code to client.conf
```
autospawn = no
daemon-binary = /bin/true
```
Make Debian-gdm user the owner of this file
```
sudo chown Debian-gdm:Debian-gdm /var/lib/gdm3/.config/pulse/client.conf
```
Restart the bluetooth service.
```
sudo service bluetooth restart
```
Kill Pulse Audio Server
```
killall pulseaudio
```

#### Softwares
```
sudo apt-get install gvim rar unrar vlc gimp bleachbit gparted emacs synaptic
```

#### Install Version Control System
```
sudo apt-get -y install git

git config --global user.name "Shubham Gulati" && git config --global user.email "shubhamgulati91@gmail.com"
git config --global credential.helper cache && git config --global credential.helper 'cache --timeout=86400'
```

#### Install Android Platform Tools
```
sudo apt-get install android-tools-fastboot android-tools-adb
```

#### Install Google Chrome Browser
```
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'

sudo apt-get -y update && sudo apt-get -y install google-chrome-stable
```

#### Install Sublime Text 3 Editor
```
wget https://download.sublimetext.com/sublime-text_build-3103_amd64.deb

sudo dpkg -i sublime-text_build-3103_amd64.deb

sudo ln -s /opt/sublime_text/sublime_text /usr/bin/sublime_text
```
Edit Sublime Text dash entry
```
sudo nano /usr/share/applications/sublime_text.desktop
```
```
Exec=/usr/bin/sublime_text %F
```
Activate Sublime Text using following license as sudo and as local user
```
—– BEGIN LICENSE —–
Michael Barnes
Single User License
EA7E-821385
8A353C41 872A0D5C DF9B2950 AFF6F667
C458EA6D 8EA3C286 98D1D650 131A97AB
AA919AEC EF20E143 B361B1E7 4C8B7F04
B085E65E 2F5F5360 8489D422 FB8FC1AA
93F6323C FD7F7544 3F39C318 D95E6480
FCCC7561 8A4A1741 68FA4223 ADCEDE07
200C25BE DBBC4855 C4CFB774 C5EC138C
0FEC1CEF D9DCECEC D3A5DAD1 01316C36
—— END LICENSE ——
```

#### Install Oracle Java SE JDK
```
sudo mkdir /opt/jdk && sudo tar -zxf jdk-*-linux-x64.tar.gz -C /opt/jdk

sudo update-alternatives --install /usr/bin/java java /opt/jdk/jdk*/bin/java 100

update-alternatives --display java

sudo update-alternatives --install /usr/bin/javac javac /opt/jdk/jdk*/bin/javac 100

update-alternatives --display javac
```
Configure the default version of Java
```
sudo update-alternatives --auto java

sudo update-alternatives --auto javac
```
OR to select particular version of JDK
```
sudo update-alternatives --config java

sudo update-alternatives --config javac
```

#### Remove OpenJDK and IcedTea Browser Plugin
```
sudo apt-get purge openjdk-\* icedtea*
```
Check the jdk directory to confirm deletion
```
sudo ls /usr/lib/jvm
```

#### Install Eclipse Java EE IDE

Download Eclipse Java EE IDE for Linux

https://www.eclipse.org/downloads/
```
sudo tar -zxf eclipse-*.tar.gz -C /opt/

sudo ln -s /opt/eclipse/eclipse /usr/bin/eclipse
```
Add Eclipse to dash
```
sudo nano /usr/share/applications/eclipse.desktop

[Desktop Entry]
Version=1.0
Name=Eclipse
GenericName=IDE
Comment=Eclipse Integrated Development Environment
Exec=/usr/bin/eclipse %F
Icon=/opt/eclipse/icon.xpm
Type=Application
Terminal=false
Categories=Development;IDE;
```

#### Install MySQL Server
```
sudo apt-get -y install mysql-server

sudo mysql_secure_installation

# Answer YES to all options

mysql -u root -p

status
```

###### Create a new user in MySQL
```
mysql -u root -p

CREATE USER 'shubham'@'localhost' IDENTIFIED BY 'tiger';

GRANT ALL PRIVILEGES ON * . * TO 'shubham'@'localhost';

FLUSH PRIVILEGES;
```
Verify creation of new user
```
SELECT user FROM mysql.user;

exit
```
###### Add test database to MySQL

wget https://launchpad.net/test-db/employees-db-1/1.0.6/+download/employees_db-full-1.0.6.tar.bz2
```
tar -xjvf employees_db-full-*.tar.bz2 && cd employees_db

mysql -u root -p < employees.sql && cd .. && rm -rf employees_db
```
More instructions at

https://dev.mysql.com/doc/employee/en/employees-installation.html

###### To remove MySQL

sudo apt-get purge mysql* && sudo rm -rf /etc/mysql /var/lib/mysql


#### Install GNOME Shell Extensions

Use Firefox Web Browser and go to https://extensions.gnome.org/

Choose "Allow and remember" when prompted for shell integration permission. 

Install and configure extensions:
```
Drop Down Terminal
https://extensions.gnome.org/extension/442/drop-down-terminal/

Todo.txt
https://extensions.gnome.org/extension/570/todotxt/

Caffeine
https://extensions.gnome.org/extension/517/caffeine/

Refresh Wifi Connections
https://extensions.gnome.org/extension/905/refresh-wifi-connections/

NetSpeed
https://extensions.gnome.org/extension/104/netspeed/
```
Extensions can me managed after installation from Gnome Tweak Tool.


#### Reboot system to test for unwanted behavior
```
sudo reboot
```

#### Install LaTeX Document preparation system

Install TexLive
```
sudo apt-get install texlive-latex-base texlive-latex-recommended
```
Add ModernCV package for Resume
```
wget http://mirrors.ctan.org/macros/latex/contrib/moderncv.zip

unzip moderncv.zip && cd moderncv

sudo mkdir -p /usr/local/share/texmf-dist/tex/latex/moderncv

sudo cp *.sty *.cls -t /usr/local/share/texmf-dist/tex/latex/moderncv/ && cd .. && rm -rf moderncv
```
Rebuild latex package cache
```
sudo mktexlsr
```
Check if moderncv.cls is correctly installed with:
```
kpsewhich moderncv.cls
```

#### Keep system updated
```
sudo apt-get -y update && sudo apt-get -y upgrade && sudo apt-get -y dist-upgrade
```

#### Clean up the system
```
sudo apt-get autoremove && sudo apt-get autoclean && sudo apt-get clean
```

## THE END
