---
title: "Arch Linux and Eduroam on a Raspberry Pi, No Ethernet Cable Required"
date: 2018-03-05
image: archRpi-neofetchoutput.png
categories:
    - Technology
tags:
    - Eduroam
    - Raspberry Pi
    - Arch Linux
    - ARM
---

*This article was originally written by me for UMass IT Techbytes Blog. Since then the blog has been discontinued. This article is republished here for archival purposes, the original can still be found [here](https://blogs.umass.edu/Techbytes/2018/03/05/firing-up-arch-linux-on-a-raspberry-pi-no-ethernet-cable-required/).*


Raspbian may be the most common OS on Raspberry Pi devices, but it is definitely not alone in the market. Arch Linux is one such competitor, offering a minimalist disk image that can be customized and specialized for any task, from the ground up – with the help of Arch Linux’s superb package manager, [Pacman](https://wiki.archlinux.org/index.php/Pacman).

The office website for [Arch Linux Arm](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3) contains all the necessary files and detailed instructions for the initial setup. After a reasonably straightforward process, plugging in the Raspberry Pi will great you with a command line interface, CLI, akin to old Microsoft DOS.

Luckily for those who enjoy a graphical interface, Arch Linux supports a wide variety in its official repository, but for that, we need the internet.  Plenty of tutorials detail how to connect to a typical home wifi, but Eduroam is a bit more challenging. To save everyone several hours of crawling through wikis and forums, the following shall focus on Eduroam.

To begin, we will need root privilege; by default this can be done with the following command:

```bash
su
```

After entering the password, we need to make the file:

```bash
nano /etc/wpa_supplicant/eduroam
```

Quick note: The file doesn’t need to be named eduroam.

Now that we’re in the `nano` text editor we need to write the configuration for eduroam. Everything except the identity and password field needs to be copied exactly. For the propose of this Tutorial I’ll be John Smith, jsmith@umass.edu, with password Smith12345.

```
network={
    ssid=”eduroam”
    key_mgmt=WPA-EAP
    eap=TTLS
    phase2=”auth=PAP”
    identity=”jsmith@umass.edu”
    password=”Smith12345”
}
```

Quick note: the quotation marks are required, this will not work without them.

Now that that’s set, we need to set the file permissions to root only, as its never good to have passwords in plain text, unsecured.

```bash
chmod og-r /etc/wpa_supplicant/eduroam
```

Now just to make sure that everything was set properly, we will run

```bash
ls -l /etc/wpa_supplicant | cut -d ' ' -f 1,3-4,9
```

The correct output should be the following

```bash
-rw------- root root eduroam
```

If you named the config file something other than eduroam, it will show up on the output as that name.

Now that that’s all set, we can finally connect to the internet.

```bash
wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/eduroam &
```

Provided everything is set correctly, you will see `wlan0: link becomes ready` halfway through the last line of the page, hit enter and just one more command.

```bash
dhcpcd
```

Now, just to check we’re connected, we’ll ping google

```bash
ping google.com -c 5
```

If everything is set, you should see 5 packets transmitted, 5 packets received.
Now that we’re connected, its best to do a full update

```bash
pacman -Syyu
```

At this point, you are free to do what you’d like with Arch. For the sake of brevity I will leave off here, for extra help I highly recommend the official [Arch Linux Wiki](https://wiki.archlinux.org/). For a graphical UI, I highly recommend setting up `XFCE4`, as well as a network (wifi) manager.

![Example of a customized XFCE4 desktop by Erik Dubois](XFCE4.png)

**Disclaimer: UMass IT does not currently offer technical support for Raspberry Pi.**