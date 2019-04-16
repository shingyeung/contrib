# WIP to be completed

# Self hosting a Solid POD on a Raspberry Pi 3 using node-solid-server (NSS)

The steps below have been tested to work on the stated hardware and software combination.

For setup using Docker on a Raspberry Pi, see TBD.

Assumptions/Dependencies:
* Hosting one POD (single user)
* Hosted in a home network
* You already have a domain name, or your router brand support one via DDNS, e.g. Asus
* You are somewhat familiar with getting around in Linux command line
* You known enough to edit a text file with vi in Linux
* You are okay with setting up port forwarding on your home router, and know/accept the potential security risks or know how to protect your router from attacks. **If not, do not proceed.**

## Get the hardware

* Raspberry Pi3 Model B ("rpi3")
* MicroSD Card 32GB
* Power supply
* Ethernet cable
* One free port on your home router
* Enclosure for rpi3 (Optional)

They can be purchased at the usual online stores or from craigslist.

You will also need the ability to burn an OS image onto a microSD card from a laptop or desktop computer with a card reader/writer.

[Add picture of setup]

## Install Linux

Download the OS for the rpi3:
ubuntu-16.04-preinstalled-server-armhf+raspi3.img

Select "Ubuntu Classic Server 16.04 Raspberry Pi 3 (216MB)" from https://ubuntu-pi-flavour-maker.org/download/  
* Direct link from www.finnie.org/software/raspberrypi/ubuntu-rpi3/ubuntu-16.04-preinstalled-server-armhf+raspi3.img.xz

Use 7zip to extract the .img

You now need to burn this .img onto the SD card from your laptop or desktop:

On Windows:
Download and install Win32DiskImager
https://sourceforge.net/projects/win32diskimager/

On Mac:
TODO

* Burning the OS, and powering up your rpi3

[Add pics on how to do - like here ]

* Insert into the rpi3 SD slot and plugin power

* After a few mintues, from your desktop/laptop browser log into to your router admin screen and locate your rpi3's IP address, say 1.2.3.4 for remainder of this how-to

[add screenshot for Asus router admin screen]

## Log into your rpi3

* Download ssh program:  putty https://putty.org/

* Log in to your rpi3 with ssh

```ssh ubuntu@1.2.3.4```

The default password is '```ubuntu```'
// forces you to change the password, do so

* Set up root access (you need root to install all the software and dependencies)

Note:  If you don't like enabling the root account and knows enough about sudo, you can skip this step.

```sudo passwd```

Supply your ubuntu password to run the sudo command.

Then, supply a password for root.  It will ask for it a 2nd time.

[Add screenshot]

## Prepare tooling to install Solid

Switch over to root acount:

```su -```

```apt-get update```

```apt-get upgrade```

```apt-get update -y && apt-get upgrade -y```

```apt-get install software-properties-common```

```add-apt-repository ppa:certbot/certbot```

```apt-get update```

```apt-get install certbot```

```apt-get install npm```

## Domain name or DDNS, and Port Forwarding

[TODO]

Disclaimer:  Security risks and considerations for opening up your home network to the internet at port 443.  Do so only if you know of the potential risks or have experience with protecting a web server, which is what NSS is.

## Get your certificate from Letsencrypt

[TODO - also refer to NSS installation page at inrupt]

## Install Solid with npm

As root:

```npm install -g solid-server@4.4.2```
Note: Latest version NSS 5.0.1 (4/2019) appears to peg CPU on a rpi3 after a trustedApp is added; 4.4.2 does not

Check which version is installed:
```npm list -g solid-version```

If you need to start over, uninstall with:
```npm uninstall -g solid-server```

## Configure and Run Solid

As root, run all commands below

```cd /var/www```

```mkdir -p yourdomain.com/data```

```mkdir -p yourdomain.com/config```

```mkdir -p yourdomain.com/.db```

```cd /var/www/yourdomain.com```

[Add screenshot of dir structure resulted]

Now, initialize solid with this one-time (per installation) command:

```solid init```

Run Solid:

```cd /var/www/yourdomain.com```

```nohup solid start```

```tail -f nohup.out```

[Add screenshot of successful startup]

If you don't see any errors, your POD is ready to be accessed from a browser.

[refer to remainder from NSS installation steps https://solid.inrupt.com/docs/installing-running-nss]

## Log on to your POD

From a browser load:

```https://yourdomain.com```

* Click 'Register' and add your single user
* Now, log in and select from the choices listed, click "yourdomain.com"
* Supply the user/pass created above and you should see this

[add screenshot]

## Backing up your data

[TODO]

## DR

[TODO]

## HA

[TODO]
