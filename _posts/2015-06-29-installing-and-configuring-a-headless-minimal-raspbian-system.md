---
title: "RPi: Installing and Configuring a Headless Minimal Raspbian System"
date: 2015-06-29
---

Now that I'm done with college degree number three, I have time to pursue some project ideas that have been accumulating. The [Raspberry Pi](https://www.raspberrypi.org) represents a pretty sweet platform for several of those projects. [Raspbian](https://www.raspbian.org) is based on Debian Linux and is the standard operating system for the Raspberry Pi. The Raspberry Pi was initially developed as an educational tool, and as such, the Raspbian image and installer provided by the [Raspberry Pi Foundation](https://www.raspberrypi.org) both result in a system with much more stuff (e.g. a full desktop environment) than I need or want for my embedded projects. The most recent official image, as of the time of writing, is a whopping 3.28 GiB.

In this tutorial I discuss using [_Raspbian (minimal) unattended netinstaller_ (raspbian-ua-netinst)](https://github.com/debian-pi/raspbian-ua-netinst) to install a minimal Raspbian system on a headless Raspberry Pi. There are other ways to get a minimal system set up, including using the [interactive installer](https://www.raspbian.org/RaspbianInstaller) available from Raspbian, or an image that someone else has made such as [MINIBIAN](https://minibianpi.wordpress.com). My reasons for doing it this way are:

1. I don't own a TV or computer monitor with HDMI, nor do I own a USB keyboard so I can't use any sort of interactive installer. _raspbian-ua-netinst_ allows for completely unattended installation and will even set up the root account, SSH access, and networking for post-install access.
2. Although MINIBIAN looks pretty close to what I need, it might not be exactly, and the documentation isn't the best.
3. By doing it myself, I will develop a more thorough understanding of the system upon which I will be building projects.

## Installing Raspbian

### Creating the SD card

Visit the [_raspbian-ua-netinst_ page](https://github.com/mikecamilleri/raspbian-ua-netinst) on GitHub, read about the features and requirements, and download the latest release. In this tutorial I'm using [1.0.7](https://github.com/debian-pi/raspbian-ua-netinst/tree/v1.0.7). The README on GitHub has very good instructions for formatting your SD card and writing the image to it. On my Mac, I simply used the Disk Utility app included with OS X. After opening the app I selected the card, selected the _Erase_ tab, selected _MS-DOS (FAT)_ as the format, named the disk _RASPBIAN_, and clicked _Erase_. I then copied the contents of the _.zip_ version of the release to the card using Finder. There are a lot of hidden files (dotfiles) in the archive, so make sure those get copied too.

### Customizing the installer

In the root of the SD card, create a file named `installer-config.txt`. That file is where you can customize the installation. See `README.md` on GitHub for details. My few customizations are below, with comments. The next release ([1.1.x](https://github.com/debian-pi/raspbian-ua-netinst/tree/v1.1.x)) will offer more configuration options which will replace some of what I do later in the tutorial. If you are working with that release, take an extra thorough look at the documentation.

```bash
# This is default, but here as a reminder.
preset=server

# Hostname and root password
hostname=rubus
rootpw=raspbian
```

I have a background in biology. I tend to name my computers after famous biologists or using biological terminology. [_Rubus_](https://en.wikipedia.org/wiki/Rubus) is the genus that contains raspberries, thus the hostname. As I get more Raspberry Pis, they will each get a species name as well (e.g. _Rubus-strigosus_, the American red raspberry).

### Running the installer

Eject the card from your computer, stick it in the Raspberry Pi, connect the Pi via the Ethernet port to your internet connected network, and plug it in.

This installation will take some time. When the installation is complete, you will see the Pi on your home network and will be able to connect to it via SSH at its IP. The IP will likely be something other than the one below.

```bash
ssh root@192.168.2.2
```

## Post Installation Linux Configuration

There are some things that you will need to do by hand after installation completes. If you're installing on many Pis, the steps below could be automated using a `post-install.txt` file. In that case though, it might make more sense to create a master image and clone it using `dd` or a similar utility.

### Locales and timezone

There are sometimes issues with the locale settings on new Raspbian installs. Read [here](http://raspberrypi.stackexchange.com/questions/1132/which-locale-should-i-select-during-the-raspbian-setup), [here](http://seafoid.org/2013/02/raspberry-pi-ssh-locale/),  [here](https://www.thomas-krenn.com/en/wiki/Perl_warning_Setting_locale_failed_in_Debian), and [here](https://wiki.debian.org/Locale#Standard). Considering all of that, this is what I did, but I'm not sure it's the best way to solve the problem, so please do your own research. I live in the United States, so my locale is en_US.utf8; yours may be different.

```bash
locale-gen en_US.UTF-8
# The command below wouldn't run until I ran the previous command.
dpkg-reconfigure locales
# Once the menu was displayed, I selected "en_US.UTF-8 UTF-8" to be generated;
# on the next screen I selected "None" as the default locale.
dpkg-reconfigure tzdata
# Once the menu was displayed, I navigated to my time zone. Note that the
# United States has it's own category on the first screen.
# And now I reboot
shutdown -r now
```

### Make sure everything is up to date with `apt-get`

```bash
apt-get update
apt-get upgrade
```

### Install `sudo`, create a user, and disable root

There are some strong opinions on how to best handle to root user on a UNIX system. I like to use `sudo` and disable the root user.

```bash
# install sudo
apt-get install sudo
# add a user
adduser pi
# make the user a sudoer
visudo
```

When your editor launches (probably Vim), under "User privilege specification", copy the line beginning with "root", replacing "root" with the name of the new user. In my case, the line was `pi ALL=(ALL:ALL) ALL`.

Logout with `logout`, then SSH back in with your new user. Now we will disable the root user.

```bash
# disable the root user
sudo passwd -l root
```

### Configure the editor

There are many editors available for Linux. I like Vim, but you may also want to consider Nano or Emacs.

The `preset=server` in `installer-config.txt` caused `vim-tiny` to be installed as `/usr/bin/vim.tiny`. I'm okay with that for now, but I want `/usr/local/bin/vim` symlinked to it so I can run it using the `vim` command.

```bash
sudo ln -s /usr/bin/vim.tiny /usr/local/bin/vim
```

While I'm at it, I'm going to set the default editor by adding the following to `~/.bashrc`:

```bash
export VISUAL=vim
export EDITOR="$VISUAL"
```

## Final Hardware Configuration

`/boot/config.txt` contains settings for the Raspberry Pi hardware similar to what might be found in a typical computer's BIOS settings. Check out the documentation [here](https://www.raspberrypi.org/documentation/configuration/config-txt.md). I added the following to my file. Note that I have the camera module installed. Uncommenting `disable_camera_led=1` will cause the LED on the camera not to illuminate when recording.

```
# Camera Module
startx=1
#disable_camera_led=1

# GPU Memory Allocation (min 128 for camera module)
gpu_mem=128
```

This may seem out of order, and it might be better done immediately after installation, but I wanted to wait until I had my editor set up. Plus it leads nicely into the next section on the camera module.

Now we reboot so the new settings become active.

```bash
sudo shutdown -r now
```

## Some More Tools

That completes the headless minimal base system with networking and access via SSH. Now, I'm going to install a few things that expand on that system. Here I install some tools that I use in my development environment.

### Git and Python 3

```bash
sudo apt-get install git python3
```

### Camera Module Tools

I plan to use the camera module via the command line and Python 3. I was expecting to have to install the [command line tools](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) myself, but they were already installed in `/opt/vc` and symlinked in `/usr/bin`. How convenient! I did have to add my user _pi_ to the _video_ group by executing the following command:

```bash
usermod -a -G video pi
```

After logging out and back in. I took a quick test photo from the command line. I use the `-vf` and `-hf` arguments because my camera is upside down.

```bash
raspistill -vf -hf -o test.jpg
```

All that leaves is the [library for Python 3](https://www.raspberrypi.org/documentation/usage/camera/python/README.md).

```bash
sudo apt-get install python3-picamera
```

### GPIO library for Python 3

```bash
sudo apt-get install python3-rpi.gpio
```

### Bonjour support with `avahi-daemon`

After all of that logging in an out, I'm sick of typing the Raspberry Pi's IP address. Instead I want to access it using a `hostname.local` domain name like the rest of the computers on my network. What Apple calls _Bonjour_ also goes by Rendesvous (the old Apple name for the technology), Zeroconf, mDNS/DNS-SD, and several other names.

Installation and setup are really simple on Raspbian.

```bash
sudo apt-get install avahi-daemon
```

Now I can log in via SSH with `ssh pi@rubus.local`. That was easy!

### Cleanup to recover to space

This recovers space by removing the package files downloaded during `apt-get install` commands. Those files are only needed for installation and can be safely deleted, which this command does.

```bash
sudo apt-get clean
```

## That's All Folks

We have installed and configured a headless minimal base Raspbian system on the Raspberry Pi complete with network access via SSH. We have also configured the camera module, and installed Python 3 and Git as a foundation for future development, along with Python 3 libraries to control GPIO and the camera module. Bonjour support was added as a convenience.

I hope that this tutorial can help smooth over some gaps in the existing documentation and help others configure a headless minimal Raspbian system to serve as a foundation for their projects.
