---
title: "RPi: Installing, Configuring, and Securing a Headless Minimal Raspbian Stretch System"
date: 2017-12-17
license: cc-by-sa
---

My previous, and now very outdated, tutorial on installing and configuring a minimal Raspbian system on the Raspberry Pi has been getting a lot of traffic. Because I want to upgrade my Raspberry Pi to Raspbian Stretch anyway, I've decided that now is a good time to to write a new guide. This time I'm going to include some basic information on security. It's important to mention though that security is hard, and what I describe here should not be relied upon if security is a great concern. In that case, there are much better guides available and even entire curricula devoted to Linux security.

## What is a "headless minimal system?"

Let's begin by understanding exactly what the words "headless" and "minimal" mean. A headless system is one that doesn't have a keyboard or display connected directly to it. We will be installing, configuring, and using our system initially by working directly with the SD card using another computer, and later over our home network. A minimal system, as I'm using it in relation to the Raspbian operating system, is a system that has a minimal amount of software installed and can serve as a basis for various projects. This system won't have a graphical user interface or any of the software that goes along with it. 

## Is this tutorial right for you?

Because we are installing a system without a GUI, this tutorial assumes that the reader has a basic knowledge of the Linux command line including how to navigate with commands like `cd` and `ls`, the ability to edit files with an editor of their choice, and a basic understanding of the `ssh` command. If you don't know these things an operating system without a GUI probably isn't for you. You also must have a way to learn the IP address of a new device on your network. Most people would do this by logging into their router. When we configure the SD card using another computer, I assume that the user is on a Unix-like operating system such as Linux or MacOS. I will provide instructions for both.

## Installing Raspbian Stretch Lite

_Rasbian Stretch Lite_ is the minimal version of the Raspbian operating system the we will be working with in this tutorial and is packaged by the [Raspberry Pi Foundation](https://www.raspberrypi.org/). Download the image from them. Once downloaded, open up a terminal window and navigate to the directory that contains the zipped file. It will be named something like `2017-11-29-raspbian-stretch-lite.zip`. Now that we are in the correct directory, let's unzip the file by running

```bash
unzip 2017-11-29-raspbian-stretch-lite.zip
``` 

We now have a file with the same name, but a `.img` extension. This is the file that we will write to our SD card. We need to be very careful that we don't accidentally write to the wrong device. If we write to the wrong device, we could obliterate other data on our computer! The way to determine what device is our destination SD card varies a bit depending on our operating system. On the Mac I run `diskutil list` before and after inserting the SD card, noting the identifier of the new device. The identifier will be something like `diskN` where `N` is a number. On Linux `df -h` can be used in a similar fashion, but the device name will be something like `/dev/mmcblkN` or `/dev/sdX` where `N` is a number and `X` is a letter.

In order to write our Raspbian image to the SD card we need to unmount it. On a Mac this is done my running 

```bash
diskutil unmountDisk /dev/diskN
```

replacing the `N` with the number you found above. On Linux use

```bash
umount /dev/<identifier>
```

replacing `<identifier>` with what you found above.

Now let's actually write the image. The command is subtly different on MacOS and Linux. In both cases be sure to use the correct file name and destination. You will likely be asked to enter your password -- this is that dangerous.

On MacOS be sure to note that we use `rdiskN` instead of `diskN`:

```bash
sudo dd bs=1m if=2017-11-29-raspbian-stretch-lite.img of=/dev/rdisk2 conv=sync
```

On Linux use:

```bash
sudo dd bs=1M if=2017-11-29-raspbian-stretch-lite.img of=/dev/<identifier> conv=fsync
```

In both cases these commands may take a long time to complete. Be patient.

If for some reason the above instructions didn't work for you, alternative instructions for copying the Raspbian image to an SD card are available at the [Raspberry Pi Foundation website](https://www.raspberrypi.org/).

We now have an SD card that can be inserted into a Raspbery Pi and booted from. Before we do that, however, we are going to get some things set up so that we can interact with our Raspberry Pi over a network, never connecting it to a keyboard or display. 

## Configuring headlessness

As our operating system stands now, we have no way of interacting with it other than by connecting a keyboard and display directly. We must do a few things to allow us to connect over a network. Conveniently, Raspbian allows us to do this configuration by modifying files on the SD card using another computer. 

### Enable SSH access

SSH access can be enabled simply by adding an empty file named `ssh` to the `boot` partition on the SD card. On my Mac I did this with the command

```bash
touch /Volumes/boot/ssh
```

On Linux, the same partition on the SD card will likely be found in `/mnt` or `/media`. You could also create and empty text file using a GUI test editor and save it on the boot partition.

### Configure networking

If you plan on connecting your Raspberry Pi to your network using an Ethernet cable, there is nothing more to do. This is the recommended option not only because of its simplicity, but also because a wired connection is much more reliable than wireless. If you do want to connect to a wireless network, create another file in the boot partition called `wpa_supplicant.conf` with the following content, modified where necessary:

```bash
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
    ssid="your network ssid"
    psk="your security key"
    key_mgmt=WPA-PSK
}
```

### Other pre-boot configuration

Some users may want to change some other settings. Also in the `boot` partition is a file called `config.txt` that contains settings that mostly configure various parts of the Raspbery Pi hardware. You can find documentation on that file in the Raspberry Pi Foundation [here](https://www.raspberrypi.org/documentation/configuration/config-txt/).

## Connecting to the network and first boot 

At this point the system isn't secure. SSH is enabled and the default `pi` user has the default password `raspberry`. Make sure that whatever network (wired or wireless) your Raspberry Pi is going to connect to on boot has a firewall between it an the internet, or connect the Raspberry Pi to your computer's ethernet port and use your operating system's internet sharing features.

Once physically (or wirelessly) connected to the network, find the Raspberry Pi's IP address using your router's web interface or your operating system if you've connected your Raspberry Pi directly to your computer. How exactly to do so will vary based on your network hardware or operating system so is beyond the scope of this tutorial. Alternatively, you may be able to connect using the hostname `raspberrypi` or `raspberrypi.local`. Whether the hostname options work depends on a lot of factors.

Connect to the Raspberry Pi via SSH using the Raspberry Pi's IP address or hostname `raspberrypi`. The default user name is `pi` and the default password is `raspberry`:

```bash
ssh pi@nnn.nnn.nnn.nnn
```

or 

```bash
ssh pi@raspberrypi
```

or

```bash
ssh pi@raspberrypi.local
```

After entering the password you should see a prompt.

## Initial configuration

### Updating the software

Before we go any further, let's make sure that all of the software on our system is up to date, we are going to do this using `apt-get`. Run these two commands:

```bash
sudo apt-get update
sudo apt-get dist-upgrade
```

### `raspi-config`

The `raspi-config` utility is a convenient way for us to change some Linux configuration. To launch the tool run 

```bash
sudo raspi-config`
```

In _Network Options_ you may want to change to _hostname_ to something else. This is the name of the computer.

_Localisation Options_ contains some important settings. As implied by the spelling, the system defaults to Great Britain as its locale, etc. You'll want to go through each item and change the settings as appropriate for your location. Read the instructions for each option carefully. In the timezone settings, if you are in the United States, use the "America" options, not "US." Also note that the time zones are by city, so choose a major city close to you that is in the same time zone. In Portland, Oregon, I choose Los Angeles.

When you are done making these changes and select _Finish_ you will be asked to reboot. Do so and SSH back in.

## Security

I mentioned this at the beginning, but it think it's important for me to say again: security is hard, and what I describe here should not be relied on if security is a great concern. In that case, please find a more thorough resource.

### Creating a new user and removing the `pi` user

Default user names and especially passwords are a tremendous security vulnerability so we are going to create a new user, give it a good password, and remove the `pi` user.

Run the following to create a new user and add it to the group of users who can use sudo. When you create the new user you will be prompted for a bunch of contact information. You can leave those lines blank and press enter if you don't want to add it:

```bash
sudo adduser mike
sudo usermod -aG sudo mike
```

Now log out and SSH back in as your new user. Run `sudo visudo` to confirm that your new user has sudo access. If an editor opens up, you're good. Exit the editor.

Now we can safely remove the `pi` user by running the following command:

```bash
sudo deluser --remove-home pi
```

### Securing `sudo`

The Raspbian uses a strange file to disable to password requirement for the `pi` user to use `sudo`. Since that user no longer exists, let's go ahead and remove that file.

```bash
sudo rm /etc/sudoers.d/010_pi-nopasswd 
```

Now that we've cleaned that up, we can disable the requirement that users enter their passwords to use `sudo`. Whether to do this is a judgement call. Requiring users to enter their passwords adds an extra layer of security, because if someone gains access to a user account via a means other than the password, he or she still won't be able to access things that require root privileges. However, it's inconvenient to enter your password every time you need to use `sudo`. I am going to go ahead and disable the password requirement. To do so, run:

```bash
sudo visudo
```

and edit the line that currently reads

```bash
%sudo   ALL=(ALL:ALL) ALL
```

to read

```bash
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

### Securing SSH

Because we access our Raspberry Pi over our network via SSH, we need to make sure it's secure. The level of security required depends on whether our Raspberry Pi will be accessible via the internet and our weighing of security vs. convenience.

I am going to use key-based authentication instead of a password. This is a somewhat involved process and may not be necessary for your needs. The process involves creating a pair of keys (one public and one private) on each computer that I will be using to access the Raspberry Pi, moving each public key to Raspberry Pi, and configuring the SSH server to use them. I will then disable password access via SSH. Be aware that if you lose your private key, you will not be able to log in via SSH and will need to connect a keyboard and monitor to the Raspberry Pi or use the serial interface. This is much more advanced than the previous work done in this tutorial so I am going to be less detailed and leave more up to the reader to figure out.

On the computer that you want to use to access the Raspberry Pi -- probably the one that you have been working from -- generate a pair of keys by running `ssh-keygen`. This will create two files in `~/.ssh` named `id_rsa` and `id_rsa.pub`. They are your private and public keys respectively. Different operating systems require various things to be done in order for the SSH client to use the keys. There is too much variability for it to be practical for me to include specific instructions here. If you aren't sure how to do this, you shouldn't have a difficult time finding a tutorial for your specific operating system.

Now that we have our SSH key pair, we need to move the public key to the Raspberry Pi and tell the SSH server to allow it by adding it to the `~/.ssh/authorized_keys` file. I did this by copying and pasting, but `ssh-copy-id` or something like the `scp` utility could also be used. The file permissions need to be 644 and can be set using:

```bash
sudo chmod 644 ~/.ssh/authorized_keys
```

Let's make sure everything worked by logging out and SSHing back in. If we aren't asked for a password the keys are working and we can safely disable password access.

To disable password access, we need to edit `/etc/ssh/sshd_config` with superuser privileges (`sudo`) and change the following settings to "no", if they are not already set to "no":

```bash
ChallengeResponseAuthentification no
PasswordAuthentification no
```

## The end

If everything above worked, you now have a basic Raspbian Linux system set up that can be accessed over your local network and is reasonably secure.
