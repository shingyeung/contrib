# Self-hosting a Solid POD on a Raspberry Pi 3 using node-solid-server (NSS)

The steps below have been tested to work on the listed hardware and software combination.

For setup using Docker on a Raspberry Pi, see TBD.

Assumptions/Dependencies:
* Hosting one POD (single user)
* Hosted in a home network
* You already have a domain name, or your router brand support one via DDNS, e.g. Asus
* You are somewhat familiar with getting around in Linux command line
* You known enough to edit a text file with vi in Linux
* You are okay with setting up port forwarding on your home router, and know/accept the potential security risks or know how to protect your router from attacks. **If not, do not proceed.**

You want to host your own POD because
* You want to own your data physically
* (Not covered by this how-to) You want an ability to host a POD for your friends and family (this requires wildcard certificated setup, see TBD) for same reason.

## Get the hardware

* Raspberry Pi3 Model B ("rpi3")
* MicroSD Card 32GB
* Power supply
* Ethernet cable
* One free port on your home router
* Enclosure for rpi3 (Optional)

They can be purchased at the usual online stores or from craigslist.

You will also need the ability to burn an OS image onto a MicroSD card from a laptop or desktop computer with a card reader/writer.

[Add picture of setup]

## Install Linux

**Download the OS for the rpi3**

```ubuntu-16.04-preinstalled-server-armhf+raspi3.img```

Select "Ubuntu Classic Server 16.04 Raspberry Pi 3 (216MB)" from https://ubuntu-pi-flavour-maker.org/download/  
* Direct link from www.finnie.org/software/raspberrypi/ubuntu-rpi3/ubuntu-16.04-preinstalled-server-armhf+raspi3.img.xz

Note: If you prefer you may want to use Ubuntu-18 img instead and will likely work also but these steps have not been tested with ubuntu 18.

Use [7zip](https://sourceforge.net/projects/sevenzip/) to extract the .img

**Burning the MicroSD with OS image, and powering up your rpi3**

You now need to burn this .img onto the SD card from your laptop or desktop.

_On Windows:_ Download and install Win32DiskImager
https://sourceforge.net/projects/win32diskimager/
[Add pics]

_On Mac:_ [TODO]

1. Insert into the rpi3 SD slot, connect the Ethernet cable from a free port on your router to the rpi3's Ethernet jack, and plug in power to the rpi3; watch the lights go on and blink while it boots the OS.

2. After a few minutes, from your desktop/laptop browser log into to your router admin screen and locate your rpi3's IP address, say 1.2.3.4 for remainder of this how-to

[add screenshot for Asus router admin screen]

## Log into your rpi3

* Download ssh program:  putty https://putty.org/

* Log in to your rpi3 with putty, or from a command line with ssh program:

```ssh ubuntu@1.2.3.4```

The default password is '```ubuntu```'
// forces you to change the password, do so

* Set up root access (you need root to install all the software and dependencies)

Note:  If you don't like enabling the root account and knows enough about sudo, you can skip this step.
```
sudo passwd
```
Supply a password for root.  It will ask for it a 2nd time to confirm.

[Add screenshot]

## Prepare tooling to install Solid

Switch over to root account:
```
su -
```

And run all of these commands one at a time.  Look for errors and do not proceed to next step if any [TODO: add common errors and fixes]
Some of these take a while to complete so be patient...
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

_Getting your own domain name_

This is the recommended option for on-going long term hosting of your own POD.  While there is recurring annual cost, it will give you peace of mind and constant a domain name for your communications to the family/friends/world going forward.

Free alternatives to getting a domain name include:
* Configuring your router to enable a Dynamic DNS name:  Most mainstream router manufacturers, e.g. Asus, provides a single name to be registered through the manufacturer's DNS server, e.g. yourdomainname.asuscomm.com (for Asus).  Check your router manual on how to configure this.

* Via Freenom [instructions here](https://www.bourgeoa.ga/solid-wiki/index.php?title=Solid_server_with_docker_on_a_Synology_Nas#get_a_domain_name_with_freenom)

_Port Forwarding_

**Important**:  Security risks and considerations for opening up your home network to the internet at port 443.  Do so only if you know of the potential risks or have experience with protecting your rpi3 NSS server.

Different router brands all have different methods to configure port forwarding.  Please consult your router manual to configure forwarding incoming traffic from the Internet on port 443 to port 443 of your rpi3.

## Get your certificate from Letsencrypt

Note: You need to configure your home router to forward port 80 while you run the command below to obtain your certificate.  Reason is certbot talks to letsencrypt and contacts back to yourdomain.com:80 for a response before the process can complete successfully.

[Refer to NSS installation](https://solid.inrupt.com/docs/installing-running-nss)

Here is a highlight:
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
   ...
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
```
npm install -g solid-server@4.4.2
```

**Note: Latest version NSS 5.0.1 (4/2019) appears to peg CPU on an rpi3 after a trustedApp is added; 4.4.2 does not.  Similar delay/sluggishness is also observed on a NSS5.0.1/Unbuntu 16.0.4 LTS server on a VM. It is not as exaggerated here probably due to higher CPU and memory specs.**

Check which version is installed:
```
root@ubuntu:~# npm list -g solid-server
/usr/lib
└── solid-server@4.4.2
```
If you need to start over, uninstall with:
```
npm uninstall -g solid-server
```
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
```
cd /var/www/yourdomain.com

nohup solid start &
```

If you want to see its logs:

```
tail -f nohup.out
```

(Ctrl-C to exit)

If you don't see any errors, your POD is ready to be accessed from a browser.

## Log on to your POD

From a browser load:

```
https://yourdomain.com
```

* Click 'Register' and add your single user
* Now, log in by selecting from the choices listed: "yourdomain.com" as the authorization provider
* Supply the user/pass created above and you should see this

[add screenshot]

## Backing up your data

The easiest way is to clone your now-operational MicoSD image.

1. Power down the rpi3 (need to do this as root)
```
su -

shutdown -h now
```
After watching the green led blink 10 times in sequence, unplug the power adpater, remove the MicroSD card from the rpi3.

2. Plug the MicroSD card into your desktop/laptop card reader.  Using Win32DiskImager, create a backup image from it.  

You now have a backup of your data and also an image for disaster recovery in the scenario where your rpi3 suffers a hardware failure.

3. Replace the MicroSD and power up the rpi3, log in, switch over to root and start your server again

After log in:
```
su -
cd /var/www/yourdomain.com
nohup solid start&
```

Now you can log out
```
exit
exit
```

Alternative approaches for backing up your data that is not as high-touch as above is possible but too advanced for this how-to, e.g. mounting the /var/www/yourdomain.com/data directory to a redundant network drive, setting up dirsync, or a crontab that ftps a tar of the data directory elsewhere.
