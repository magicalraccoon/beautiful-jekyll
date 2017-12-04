---
layout: post
title: "Whole Network Ad-blocking with Pi-hole"
date: 2017-08-04 23:05.00
categories: tutorial
---

I don’t like ads. They’re invasive, intrusive, and a [security risk](https://www.techdirt.com/articles/20160111/05574633295/forbes-site-after-begging-you-turn-off-adblocker-serves-up-steaming-pile-malware-ads.shtml). Web content that uses a donation or paid-premium model seems to be the way to go. If you’d like to save yourself from being bombarded by web advertisements, Pi-hole is your answer. Before I begin, I’d like to remind you to financially support the content that you enjoy.

## What is Pi-hole?
[Pi-hole](https://pi-hole.net/) is a lightweight DNS server that filters known advertising domains. It works by checking incoming websites against a filter list, and “black-holing” domains that match. Afterwards, it directs the cleaned up DNS traffic as usual. This process reduces excessive network traffic as well as protecting you from malvertising domains.

## Installing Pi-hole
### Requirements
Before beginning, you will need to have administrative access to your router.
#### Hardware
Pi-hole doesn’t require much in terms of hardware, only ~52MB of free space and 512 MB of RAM. I chose to install Pi-hole on my Raspberry Pi (which is also functioning as my NAS).

#### Software
Pi-hole is officially supported on the following platforms:
* Raspbian: Jessie (lite / with pixel)
* Ubuntu: 14.04 / 16.04 / 16.10
* Fedora: 24 / 25
* Debian: 8.6
* CentOS: 7.2.1511 / 7.3.1611

### Automated Installation
Pi-hole provides an automated install script, but I highly recommend taking a look at the source before executing bash scripts you find on the internet.
```
$ wget -O basic-install.sh https://install.pi-hole.net
$ cat basic-install.sh
```
If the code is acceptable, go ahead and run the installer script:
`$ bash basic-install.sh`

The on-screen instructions are straightforward, and provide good default options.

![Pi-hole settings](/img/pihole1.png)

At this point you’ll want to direct a web browser to the IP address of the device that you installed Pi-hole.
In my case, `http://10.0.2.15/admin/index.php`

After logging in, you’ll be greeted with a screen that looks like this
![Pi-hole settings](/img/pihole2.png)

From here, I recommend going to the Settings tab and expanding the Pi-Hole's Block Lists box.
The default Pi-hole filters may work for you, but I wanted extra privacy. [WaLLy3K](https://wally3k.github.io/about.html) was nice enough to provide a [sourced list of sites](https://v.firebog.net/hosts/lists.php?type=tick) to add to your filter.  
You’ll be able to copy this list in entirety into the text box, then save and update your filters.

**Don’t forget to click the Enable tab!**

Now, in order to get the filters to apply to your entire network, you’ll need to go to your router’s configuration page. Use the IP address of your Pi-hole device as a Static DNS.

If you are unable to change your DNS settings, Pi-hole also offers a built-in DHCP server.

Once this is all done, it’s a good idea to reboot your router so that your new DNS settings take effect.
