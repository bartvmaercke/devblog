---
title: Instal pihole and ubntcontroller
date: "2019-08-30"
---

goal: use an oldraspberry pi I had laying around as a pi-hole and ubiquity controller.

<!-- end -->

## create the pi-hole

download latest raspbian lite from  
<https://downloads.raspberrypi.org/raspbian_lite_latest>  
flash sd card with rufus, create ssh file in boot partition, insert in pi, boot, log in with default username.

username: pi
password: raspberry
(hint: make new user and remove pi, or at least change the password to something secure)

If you forgot to add the ssh file, you can still enable ssh from raspi-config is you have a screen attached:  
sudo raspi-config > Interfacing Options > SSH > Yes > OK > Finish  
source: <https://www.raspberrypi.org/documentation/remote-access/ssh/>

My frist try was with an old pi1 I had laying around, so I also overclocked it with raspi-config. Since I'll be using it headless, I also changed the video memory to 16 with raspi-config

I tend to do an upgrade before anything else, to be sure that I'm working with the latest versions

```bash
sudo apt-get update && sudo apt-get upgrade
```

After that I followed <https://github.com/pi-hole/pi-hole/#one-step-automated-install>

```bash
sudo apt-get install git

git clone --depth 1 https://github.com/pi-hole/pi-hole.git Pi-hole
cd "Pi-hole/automated install/"
sudo bash basic-install.sh
```

And we just run trough the install script.  
Don't forget to note password it displays, you'll need that to go to the webportal

Just to be sure that it was running properly, I rebooted. (Not needed. I know.)

```bash
reboot
```

Made some changes in the config file to make sure there will not be that much logs collected. (Old pi1 with not that much free space)

```bash
sudo vi /etc/pihole/pihole-FTL.conf
PRIVACYLEVEL=0
MAXLOGAGE=24.0
IGNORE_LOCALHOST=yes
MAXDBDAYS=30
```

## install ubnt controller

Next step, installing my ubiquity controller.
Some days had already passed and I wanted to be sure that everything was up to date, so did some upgrading

```bash
sudo apt-get update && sudo apt-get upgrade && sudo apt-get autoremove && sudo apt-get autoclean
```

install the pre-requisites

```bash
sudo apt-get install haveged
sudo apt-get install openjdk-8-jre-headless
```

followed the guide at  
<https://help.ubnt.com/hc/en-us/articles/220066768-UniFi-How-to-Install-and-Update-via-APT-on-Debian-or-Ubuntu>

```bash
echo 'deb http://www.ui.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
sudo wget -O /etc/apt/trusted.gpg.d/unifi-repo.gpg https://dl.ubnt.com/unifi/unifi-repo.gpg
sudo apt-get update; sudo apt-get install unifi -y
sudo apt --fix-broken install
sudo apt-get install unifi -y
```

Don't need mongodb

```bash
sudo systemctl stop mongodb
sudo systemctl disable mongodb
```

rebooted for the fun of it

```bash
sudo reboot
```

created a new gmail account to send mails from
configure account to send mails  
-> settings -> controller -> email server  
<https://help.ubnt.com/hc/en-us/articles/226590847-UniFi-How-To-Use-a-Free-Email-Service-as-SMTP-Server-Gmail->

set data retention  
-> settings -> maintenance -> data retention

enabled ips and setup networks, vpn etc.  
Don't forget to point the dns to your pi-hole.  
(I've also created a noip address and added that too)

## setup a new controller on a newer pi

Since the minimum requirements were not met for the ubnt controller, I decided to use an other pi and and see if i could recover the settings from the backup.

made backup  
installed new raspberry pite lite with rufus  
made ssh in boot

set video memory to 16 (again, headless server, so why use more?)

Now something went wrong.  
My pi didn't got a default gateway, or a dns server. After some troubleshooting I noticed the usg didn't have those configured properly
I had to go to the console and add some settings for my network:

```
set service dhcp-server shared-network-name LAN_192.168.101.0-24 subnet 192.168.101.0/24 default-router 192.168.101.1
set service dhcp-server shared-network-name LAN_192.168.101.0-24 subnet 192.168.101.0/24 dns-server 1.1.1.1
commit
save
```

(I'll correct the dns once I have the controller up and running again)

So now it's time to nstall everything again.

```bash
sudo apt-get install haveged
sudo apt-get install openjdk-8-jre-headless
echo 'deb http://www.ui.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
sudo wget -O /etc/apt/trusted.gpg.d/unifi-repo.gpg https://dl.ubnt.com/unifi/unifi-repo.gpg
sudo apt-get update && sudo apt-get upgrade && sudo apt-get autoremove && sudo apt-get autoclean
sudo apt-get install unifi -f
sudo systemctl stop mongodb
sudo systemctl disable mongodb
```

reinstall pi-hole again.

```bash
sudo apt-get install git
git clone --depth 1 https://github.com/pi-hole/pi-hole.git Pi-hole
cd "Pi-hole/automated install/"
sudo bash basic-install.sh
```

Set same ip address again in wizard.  
Note password again.

Changed password for pi user to something strong.

```bash
sudo reboot
```

Go to controller interface and restore backup.

That's it!
