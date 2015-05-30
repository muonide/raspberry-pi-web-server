# Raspberry Pi Express Web Server
This is how I built a home web server with a Raspberry Pi and Express Server.

## Purchases

1. [Raspberry Pi B+](http://amzn.com/B00LPESRUK)
2. [Power Supply](http://amzn.com/B00GWDLJGS)
3. [Micro SD Card](http://amzn.com/B00DYQYLQQ) (At least 8 GB)
4. [Case](http://amzn.com/B00MQLB1N6) (Optional)

![Rasberry Pi](https://www.dropbox.com/s/euujwu3nowunjtk/raspberry-pi.jpg?dl=0)

## Hardware

1. Raspberry Pi B+
2. Power Supply for Raspberry Pi
3. Modem and Router
4. Ethernet Cord

## Setup

1. Install the operating system by following [these steps](https://www.raspberrypi.org/help/noobs-setup/)
2. Once the operating system is installed, from the command line update the software to the latest.

		$ sudo apt-get update; sudo apt-get upgrade
4. If wanted, change the default `pi` user password to something more secure.

		$ sudo raspi-config
6. Navigate to `2 Change User Password` and follow the steps.

## Install latest NodeJS

	$ wget http://node-arm.herokuapp.com/node_latest_armhf.deb
	$ sudo dpkg -i node_latest_armhf.deb
	$ node -v

## Install Git and clone this repo

	$ sudo apt-get install git
	$ git clone https://github.com/sean-hill/raspberry-pi-web-server.git

## Run the Webserver
	$ cd raspberry-pi-web-server
	$ npm install
	$ node server.js
	
Open a browser and go to <your-pi's-IP>:5000

You did it! You successfully ran a webserver on your Raspberry Pi. Now what? How about we open up your router to the outside world and setup a deployment method.

## Dynamic DNS

In order for your buddies to access your home web server, you'll need your own domain name that points to your router's dynamic IP provided by your local ISP. I went with free version of [noip.com](http://www.noip.com/). After creating an account follow these steps.

	$ wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz
	$ tar vzxf noip-duc-linux.tar.gz
	$ cd noip-2.1.9-1
	$ sudo make
	$ sudo make install

Follow the commands (entering in your username and password from noip) and then start the process.

	$ sudo /usr/local/bin/noip2

Now we want this to start whenever your raspberry pi boots, so follow [this](http://www.stuffaboutcode.com/2012/06/raspberry-pi-run-program-at-start-up.html) tutorial to create a startup script if you want it.

## Static Raspberry Pi IP Address

Whenever your Raspberry Pi reboots, it'll get a new dynamic IP from your router. We'd rather not have this happen, so let's make it a static IP instead. I followed [this](http://www.modmypi.com/blog/tutorial-how-to-give-your-raspberry-pi-a-static-ip-address) tutorial to accomplish this.

## Open your router to the world

You need to enable single port forwarding on your local router. Our webserver is running on port 5000, so I followed [this](http://www.noip.com/support/knowledgebase/port-forwarding-on-a-linksys-wrt610n-router/) tutorial to open up that port on my router. It may be different for each router.

![Single Port Forwarding](https://www.dropbox.com/s/w98zjpjvm5atnu7/single-port-forwarding.png?dl=1)

After saving these port forwarding settings, start up your server with `node server.js`, and you should be able to navigate to `http://<your-domain>.ddns.net:5000` and see our Raspian homepage!

I'd love to get rid of the `:5000` at the end of our url, but unfortunately my ISP restricts access to port `80`, so I can't use a proxy service such as `nginx` at this time to proxy pass through all the requests at port `80` to port `5000`.

## Automated Deployment

Well, you've done it, you can successfully view a webpage hosted on your raspberry pi from the outside world. For me, that's just the first step. I want to be able to development on my local computer, and push them changes to my pi simply!

### PM2

I ❤️ [pm2](https://github.com/Unitech/pm2). It's a great module to deploy your nodejs applications. First let's install it on both your computer and your raspberry pi.

	$ sudo npm install -g pm2
	
I like to store my web server in the `/var/www`. So let's first create that directory and then make the `pi` user the owner.

	$ sudo mkdir /var/www
	$ sudo chown pi /var/www
	$ sudo chgrp pi /var/www
	
Now we'll use pm2 to deploy straight from our local machine to our raspberry pi, but first we need to be able to ssh into our raspberry pi. 

	$ sudo raspi-config
	$ 8 Advanced Options
	$ A4 SSH
	$ Enable
	
Then you'll need to setup another single port forwarding on your router to allow SSH into your home. For my linksys router I had to find out what port allowed remote access on my router by going to `Administration` and seeing the port under `Remote Management Port`. It was `8080` for me. Then I setup the following single port forwarding.

![SSH port forwarding](https://www.dropbox.com/s/5yjmevrbm8glzq8/ssh-port-forwarding.png?dl=1)

	External Port: 8080
	Internal Port: 22
	Protocol: Both
	To IP Address: <your pi IP>
	Enabled: Checked
	
Test that your SSH works by executing this command.

	$ ssh -p 8080 pi@<your domain>.ddns.net
	
Now on your local computer clone this repo.

	$ git clone https://github.com/sean-hill/raspberry-pi-web-server.git
	
Then edit the `ecosystem.json` file and replace the `host` with your dynamic IP you received from `noip`. Once you've done that execute the following commands on your computer.

	$ pm2 deploy ecosystem.json production setup
	$ pm2 deploy ecosystem.json production
	
You'll be prompted to enter your `pi` password multiple times. If this bothers you, I recommend creating public / private ssh keys for your `pi` user and you local computer as described [here](https://www.raspberrypi.org/documentation/remote-access/ssh/passwordless.md).

Now head back over to your raspberry pi, and exectute these commands.

	$ cd /var/www/web-server
	$ pm2 list








