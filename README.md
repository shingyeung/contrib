# WIP, Incomplete, DO NOT USE

[Add Disclaimers, exclusion clauses and risk warnings]

[Add copyright]

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

[Add why one would want to self-host a POD for self/family/friends]

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

```ubuntu-16.04-preinstalled-server-armhf+raspi3.img```

Select "Ubuntu Classic Server 16.04 Raspberry Pi 3 (216MB)" from https://ubuntu-pi-flavour-maker.org/download/  
* Direct link from www.finnie.org/software/raspberrypi/ubuntu-rpi3/ubuntu-16.04-preinstalled-server-armhf+raspi3.img.xz

Note: If you prefer you may want to use ubuntu-18 img instead and will likely work also but these steps have not been tested with ubuntu 18.

Use [7zip](https://sourceforge.net/projects/sevenzip/) to extract the .img

You now need to burn this .img onto the SD card from your laptop or desktop:

On Windows:
Download and install Win32DiskImager
https://sourceforge.net/projects/win32diskimager/

On Mac:

TODO

* Burning the OS, and powering up your rpi3

[Add pics on how to do - like here ]

* Insert into the rpi3 SD slot, connect the ethernet cable from your router port to the rpi3, and plug in power to the rpi3; watch the lights go on and blink

* After a few mintues, from your desktop/laptop browser log into to your router admin screen and locate your rpi3's IP address, say 1.2.3.4 for remainder of this how-to

[add screenshot for Asus router admin screen]

## Log into your rpi3

* Download ssh program:  putty https://putty.org/

* Log in to your rpi3 with putty, or from a command line with ssh program:

```ssh ubuntu@1.2.3.4```

The default password is '```ubuntu```'
// forces you to change the password, do so

* Set up root access (you need root to install all the software and dependencies)

Note:  If you don't like enabling the root account and knows enough about sudo, you can skip this step.

```sudo passwd```

Supply a password for root.  It will ask for it a 2nd time to confirm.

[Add screenshot]

## Prepare tooling to install Solid

Switch over to root acount:

```su -```

And run all of these commands one at a time.  Look for errors and do not proceed to next step if any [TODO: add common errors and fixes]

```
apt-get update -y && apt-get upgrade -y

apt-get install -y software-properties-common

add-apt-repository -y ppa:certbot/certbot

apt-get update -y

apt-get install -y certbot

apt-get update -y

curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

apt-get install -y nodejs

apt-get install -y npm
```

## Domain name or DDNS, and Port Forwarding

[TODO]

Disclaimer:  Security risks and considerations for opening up your home network to the internet at port 443.  Do so only if you know of the potential risks or have experience with protecting your rpi3 NSS server.

## Get your certificate from Letsencrypt

Note: You need to configure your home router to forward port 80 while you run the command below to obtain your certificate.  Reason is certbot talks to letsencrypt and contacts back to yourdomain.com:80 for a response before the process can complete successfully.

[TODO - also refer to NSS installation page at inrupt]

```
certbot certonly --authenticator standalone -d yourdomain.com
```
Answer the questions and provide your email.

If all goes well, you will see:
```
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/yourdomain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/yourdomain.com/privkey.pem
   Your cert will expire on 2019-07-15. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Your certificate files are found here:
```
root@ubuntu:~# find /etc/letsencrypt/live
/etc/letsencrypt/live
/etc/letsencrypt/live/README
/etc/letsencrypt/live/yourdomain.com
/etc/letsencrypt/live/yourdomain.com/cert.pem
/etc/letsencrypt/live/yourdomain.com/chain.pem
/etc/letsencrypt/live/yourdomain.com/README
/etc/letsencrypt/live/yourdomain.com/privkey.pem
/etc/letsencrypt/live/yourdomain.com/fullchain.pem

```

You will be supplying privkey.pem and fullchain.pem at Solid initialization, below.

**Now, don't forget to goto your router and remove the port 80 forward configuration.**

## Install Solid with npm

As root:

This takes a while...

```npm install -g solid-server@4.4.2```

**Note: Latest version NSS 5.0.1 (4/2019) appears to peg CPU on a rpi3 after a trustedApp is added; 4.4.2 does not so recommend this version for rpi3 use.  Same is also observed on a full Unbuntu 16.0.4 LTS server on a VM but is not as exaggerated due to higher CPU and memeory specs there.**

Check which version is installed:

```
root@ubuntu:~# npm list -g solid-server
/usr/lib
└── solid-server@4.4.2
```

If you need to start over, uninstall with:

```npm uninstall -g solid-server```

## Configure and Run Solid

As root, run all commands below

Create directory structure in prep for Solid init:

```
mkdir -p /var/www/yourdomain.com

cd /var/www/yourdomain.com

mkdir ./data

mkdir ./config

mkdir ./.db
```

Now, initialize solid with this one-time (per installation) command:

```
cd /var/www/yourdomain.com

solid init
```

Answer all the questions per [NSS installation steps](https://solid.inrupt.com/docs/installing-running-nss)

Should look something like:

```
root@ubuntu:/var/www/yourdomain.com# solid init
? Path to the folder you want to serve. Default is /var/www/yourdomain.com/data
? SSL port to run on. Default is 443
? Solid server uri (with protocol, hostname and port) https://yourdomain.com
? Enable WebID authentication Yes
? Serve Solid on URL path /
? Path to the config directory (for example: /etc/solid-server) /var/www/yourdomain.com/config
? Path to the config file (for example: ./config.json) /var/www/yourdomain.com/config.json
? Path to the server metadata db directory (for users/apps etc) /var/www/yourdomain.com/.db
? Path to the SSL private key in PEM format /etc/letsencrypt/live/yourdomain.com/privkey.pem
? Path to the SSL certificate key in PEM format /etc/letsencrypt/live/yourdomain.com/fullchain.pem
? Enable multi-user mode No
? Do you want to set up an email service? No
? A name for your server (not required, but will be presented on your server's frontpage) yourdomain.com
? A description of your server (not required)
? A logo that represents you, your brand, or your server (not required)
config created on /var/www/yourdomain.com/config.json
```

Now, run Solid:

```cd /var/www/yourdomain.com

nohup solid start
```

If you want to see its logs:

```
tail -f nohup.out
```

(ctrl-C to exit)

If you don't see any errors, your POD is ready to be accessed from a browser.

## Log on to your POD

From a browser load:

```https://yourdomain.com
```

* Click 'Register' and add your single user
* Now, log in by selecting from the choices listed: "yourdomain.com" as the authorization provider
* Supply the user/pass created above and you should see this

[add screenshot]

## Backing up your data

[TODO]

## DR

[TODO]

## HA

[TODO]
