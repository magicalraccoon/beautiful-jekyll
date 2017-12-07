---
layout: post
published: false
title: Hosting a free Tor Relay with AWS EC2
---

1. [Create an Ubuntu Server 16.04 t2.micro instance](https://aws.amazon.com/ec2/). In the Security Group setting, allow a Custom TCP port to be open at port 9001. Your final settings should look similar to this: ![]({{site.baseurl}}/img/torEC2.png)
2. Generate your private key. Keep this somewhere very safe! This is what you'll use to access your EC2 instance now and in the future. No key, no access!
3. Launch the server through the EC2 management console.
4. You'll need to change the permissions on your private key before connecting. `$ chmod 400 <PRIVATEKEY.PEM>`
5. Use your key to connect to your new instance. `$ ssh -i "path/to/privatekey.pem" ubuntu@INSTANCE.PUBLIC.IP.ADDRESS`
6. The Tor installation contained within Apt is out of date, so we'll need to add the official location. Open your sources list. `sudo nano /etc/apt/sources.list`
7. Add the following to the end of the file:
	```
	deb http://deb.torproject.org/torproject.org xenial main
	deb-src http://deb.torproject.org/torproject.org xenial main
	```

8. You'll also need to add the trusted key. 
```
$ gpg --keyserver keys.gnupg.net --recv A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89
$ gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -
```
	
    Your output should look like this:

	```
	ubuntu@ip-123-45-67-890:~$ gpg --keyserver keys.gnupg.net --recv A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89
	gpg: requesting key 886DDD89 from hkp server keys.gnupg.net
	gpg: /home/ubuntu/.gnupg/trustdb.gpg: trustdb created
	gpg: key 886DDD89: public key "deb.torproject.org archive signing key" imported
	gpg: no ultimately trusted keys found
	gpg: Total number processed: 1
	gpg:               imported: 1  (RSA: 1)

	ubuntu@ip-123-45-67-890:~$ gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -

	OK
	```

9. Update and upgrade out of date packages. `$ sudo apt update && sudo apt upgrade -y` making sure to "Keep local changes" if grub gets updated.

10. Now we'll be able to install the current version of Tor. `$ sudo apt install tor deb.torproject.org-keyring`

11. Open your torrc file `$ sudo nano /etc/tor/torrc`
	
    In order to keep under EC2's free bandwidth limits, we'll have to add some info to the bottom of our torrc file:
    
	```
	ORPort 9001
 
	Nickname <A NODE NICKNAME>
 
	RelayBandwidthRate 75 KBytes # Throttle traffic to 75KB/s (600Kbps)
 
	RelayBandwidthBurst 200 KBytes # But allow bursts up to 200KB (1600Kb)
 
	AccountingMax 1 GBytes
 
	AccountingStart month 3 15:00
 
	ExitPolicy reject *:* # no exits allowed
	```

12. Reload the Tor service. `service tor reload`

13. Restart the instance to finish applying upgrades. `sudo shutdown -r now`

14. (OPTIONAL) Do it all over again!

Congratulations! You're helping support anoyminity across the globe! The Tor project works better if more people are using it, so why not also use the [Tor Browser](https://www.torproject.org/download/download-easy.html.en) and add some helpful noise to the network!
