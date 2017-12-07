---
layout: post
published: false
title: Hosting a free Tor Relay with AWS EC2
---

1. Spin up an Ubuntu Server 16.04 t2.micro instance
2. Generate your private key
3. Launch the server through the EC2 management console
4. `chmod 400` privatekey.pem
5. `ssh -i "path/to/privatekey.pem" ubuntu@instance.public.ip.address`
6. `sudo nano /etc/apt/sources.list`
7. Copy and paste
```deb http://deb.torproject.org/torproject.org xenial main
deb-src http://deb.torproject.org/torproject.org xenial main
```

8. 
```
gpg --keyserver keys.gnupg.net --recv A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -
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

9. `sudo apt update && sudo apt upgrade -y` making sure to "Keep local changes" if grub gets updated.

10. `sudo apt install tor deb.torproject.org-keyring`

11. `sudo vim /etc/tor/torrc`

```
ORPort 9001
 
Nickname <YOUR NODE NICKNAME>
 
RelayBandwidthRate 75 KBytes # Throttle traffic to 75KB/s (600Kbps)
 
RelayBandwidthBurst 200 KBytes # But allow bursts up to 200KB (1600Kb)
 
AccountingMax 1 GBytes
 
AccountingStart month 3 15:00
 
ExitPolicy reject *:* # no exits allowed
```

12. `service tor reload`

13. `sudo shutdown -r now`

14. Do it all over again!

Congratulations! You're helping support anoyminity across the globe!