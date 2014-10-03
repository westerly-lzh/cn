---
layout: post
title:  FreeBSD--Jail
categories:
- Unix
tags:
- jail
- virtualization

---

The first day of SysAdvent talked about Linux Containers (LXC), and how they are an "operating system level virtualization", as opposed to "platform virtualization" choices like Xen or VMWare. Today, I'll focus on jails in FreeBSD and how they achieve a similar goal.

####BACKGROUND

If you think of a traditional OS it looks something like this:

![Traditional OS Architecture](/media/img/bsd-imgs/bsd014.jpg "Traditional OS Architecture")

Among other things, the kernel controls access to hardware, makes sure processes are not stomping all over each other's memory, does the necessary access control checks for actions, and also ensures that packets land at the appropriate sockets. However, even if everything is perfect, a sufficiently privileged process can cause a lot of havoc if it misbehaves.

Let's say you have a process running as root that gets compromised and is now running arbitrary code of the attacker's choosing. Typically, this comes in the form of a shell listening on a socket, or a connect-back shell; both of which are very bad. Through this arbitrary execution of code, the attacker can do whatever he/she is allowed to through the access controls in place. In most cases (where things like mandatory access controls are not in place) this is effectively game over for the system administrator.

Before I go any further, I should probably explain how jails are different from what most people are familiar with. To be consistent with the earlier article I'll call it "platform virtualization." That solution looks something like this:

![Platform Virtualization Architecture](/media/img/bsd-imgs/bsd015.jpg "Platform Virtualization Architecture")

There are different approaches but it is essentially the same goal. Insert a small "virtual machine monitor" (VMM) layer - often called a hypervisor - that brokers access to the real hardware and emulates whatever hardware the systems administrator wants to the OS running on top of it. Modern chips have support for helping do this in hardware (AMD calls it "AMD-V" and Intel calls it "VT-x"). Just about every chip shipping now has this built in.

"Platform virtualization" has many benefits to it. You can choose what hardware to provide to the guest operating system. The VMM is hopefully small enough that it can be properly secured and verified. Finally, you can decide which operating system to run as the guest. The fact that it is virtualized should be transparent - with the exception of needing driver support for whatever hardware is emulated, which every major OS has support for.

There are some drawbacks to this approach. It can be very resource intensive as the more virtual machines you spin up the more hardware and state has to be kept in memory. With modern hardware this is becoming less of a problem, but, for some environments, it may still hold true.

####ENTER JAILS

Jails are best thought of as a means to contain and isolate processes from each other, even if those processes are privileged.

![FreeBSD Jail](/media/img/bsd-imgs/bsd016.jpg "FreeBSD Jail")

In this case, we are running multiple processes, but the kernel has been modified to limit the resources that each process can affect or view. This is the concept upon which jails are built: The name of the game is process isolation and containment, not virtualization.

####THE DETAILS

I'm going to skip over the details of how jails are created and what that means from a data structure standpoint and skip straight to how to set up a jail and how to use it. My examples will be from a fairly recent development snapshot ("current," if you are familiar with FreeBSD terminology) that is not yet a finished release, so some of the things I will describe are not completely accurate to all versions of FreeBSD but the concepts are the important part.

Jails have been around in FreeBSD for a long time now. They were first introduced in FreeBSD 4.0 (10 years ago). Since that release, jails have been refined and extended to support many of the things people want from them. Recent releases of FreeBSD include the ability for IPv6, hierarchical jails, resource utilization limits, and even virtual network stacks (which is out of scope for this article).

####SETUP

For the purposes of this article, I'm going to use the term host to indicate the FreeBSD base operating system upon which the jails will run.

A jail only requires a handful of things in order to operate. The most important of which is a working userland. Usually, people run the same version of the userland inside a jail as the one that is running on the host, but it doesn't have to be this way. If you want to run an older userland in a jail, it will likely work because backwards compatibility in the kernel is usually preserved.

Actually getting a working userland is outside of the scope of this article. There are many ways to pick from: building your own, using your existing install, or installing the binaries straight from release media. The means of getting the binaries on disk is up to you. You also don't need a full world (FreeBSD's term for the base OS), if you know exactly what you are doing you can populate it with just what you need. There are also other tricks you can do involving null mounting in other paths. For the purposes of this article I've installed an exact copy of my host to /jails/test (minus any package installations).

####STARTING JAILS

With a world installed to /jails/test, I don't need anything else installed in order to start a jail. Everything you need is provided by the base FreeBSD install. Starting a jail manually is done using the jail(8) command.

    wxs@ack wxs % sudo jail /jails/test test 192.168.1.100 /bin/sh
    # id
    uid=0(root) gid=0(wheel) groups=0(wheel),5(operator)
    # 

The arguments to the jail command are pretty straight forward. It takes a path where the root of the jail should live, a hostname, an IP address and a command inside the jail to run. Once I'm inside the jail you can see that I am automatically the root user.

From outside of the jail, on the host, you can use the jls(8) command to list existing jails.

    wxs@ack wxs % jls
       JID  IP Address      Hostname                      Path
         4  192.168.1.100   test                          /jails/test
    wxs@ack wxs % 

The one catch is that while the jail says it has an IP address, the host OS knows nothing about that IP address. In order to have your jail respond to an IP address you must add it to a network interface by adding an alias:

    wxs@ack wxs % sudo jail -r 4 # Kill existing jail, so it can get the new IP
    wxs@ack wxs % sudo ifconfig bge0 alias 192.168.1.100 netmask 255.255.255.255
    wxs@ack wxs % sudo jail /jails/test/ test 192.168.1.100 /bin/sh
    #

And now inside our jail we can see that we have an IP address.

    # ifconfig bge0 | grep inet
            inet 192.168.1.100 netmask 0xffffffff broadcast 192.168.1.100
    # 

So, at this point, we just need a working devfs inside our jail and we should have a normal, contained, system.

    wxs@ack wxs % sudo mount -t devfs devfs /jails/test/dev
    wxs@ack wxs % mount | grep jails
    data/jails on /jails (zfs, local, noatime)
    devfs on /jails/7/dev (devfs, local, multilabel)
    devfs on /jails/test/dev (devfs, local, multilabel)
    wxs@ack wxs % 

You might want your jail to be accessible from the network, so let's run an SSH daemon! I'll permit root logins for this jail, just to make my life easier, but you can add users inside jails as normal. Also, be sure to set your root password before you expose any network services. From the root shell you have after running the jail(8) command, do this:

    # sed -i '.bak' -e 's/^#PermitRootLogin no/PermitRootLogin yes/' /etc/ssh/sshd_config
    # /etc/rc.d/sshd onestart
    [... Lots of output about host key generation ...]
    # sockstat -4l | grep 22
    root     sshd       61052 3  tcp4   192.168.1.100:22      *:*
    # 

And outside the jail:

    wxs@ack wxs % ssh root@192.168.1.100
    The authenticity of host '192.168.1.100 (192.168.1.100)' can't be
    established.
    RSA key fingerprint is f9:87:e1:41:3c:27:56:fd:5a:0e:c9:0b:c5:9a:d5:15.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '192.168.1.100' (RSA) to the list of known hosts.
    Password:
    Last login: Mon Dec  6 01:43:39 2010 from 192.168.1.100
    FreeBSD ?.?.?  (UNKNOWN)

    Welcome to FreeBSD!

    [... MOTD ...]
    test#

A keen eye would spot the weirdness '?.?.?' in the MOTD above. Normally that is cleared up when the computer boots, but since we didn't really "boot" this jail, that step never happened. Let's explore what it takes to get a jail to start automatically upon boot.

####BOOTING AUTOMATICALLY

One thing you must do when starting a jail is make sure the host services are set to listen on only IP addresses that belong to the host, and not the jail. Failure to do this will cause your host services to listen on IP addresses that should be for the jail. This can have unintended consequences such as exposing host services to places they shouldn't be. For now I'm assuming you know how to do that.

Like most things in FreeBSD they are controlled with settings in /etc/rc.conf. There are actually a whole bunch of settings available, but here's the simple set I'm using:

    wxs@ack head % fgrep test /etc/rc.conf  
    jail_list="test"
    jail_test_rootdir="/jails/test"
    jail_test_hostname="test"
    jail_test_interface="bge0"
    jail_test_ip="192.168.1.100"
    jail_test_devfs_enable="YES"
    wxs@ack head % 

Using this configuration, the 'test' jail will boot and start automatically.

####BOOTING MANUALLY

You can use the /etc/rc.d/jail script to boot a jail manually provided that the appropriate settings are set in /etc/rc.conf. Another option is to set the IP address alias manually and mount devfs manually then call /etc/rc yourself:

    wxs@ack wxs % sudo jail -r 2 # Kill existing jail...
    wxs@ack wxs % sudo jail /jails/test test 192.168.1.100 /bin/sh /etc/rc
    /etc/rc: WARNING: $hostname is not set -- see rc.conf(5).
    Creating and/or trimming log files.
    Starting syslogd.
    ELF ldconfig path: /lib /usr/lib /usr/lib/compat
    32-bit compatibility ldconfig path: /usr/lib32
    Clearing /tmp (X related).
    Updating motd:.
    Starting cron.

    Sun Dec 12 16:16:37 UTC 2010
    wxs@ack wxs % jls
       JID  IP Address      Hostname                      Path
         3  192.168.1.100   test                          /jails/test
    wxs@ack wxs % 

####RESTRICTIONS

So if jails are all about isolation and containment, what can and what can't you do inside a jail? The general rule of thumb is that if it affects the host or other jails it is restricted by default. There are knobs you can turn to allow these things, but in the interests of not breaking the security model, they are turned off by default. Exactly what is restricted and what knobs are available is highly dependent upon the version of FreeBSD you are running. As more and more things are being designed to work better with jails, the set of restricted operations is shrinking. For example, in earlier releases root inside a jail was not allowed to change any network stack configuration information. With the addition of virtualized network stacks in newer releases of FreeBSD this restriction is gone, provided the jail is using a virtualized stack.

For more information on this it is best to read the documentation available.

####TRADE OFFS

Jails are a great way of getting "operating system level virtualization" on FreeBSD, but like anything else, they come with a series of trade-offs which must be considered prior to implementation.

A kernel level compromise does break the isolation provided by jails. In a "platform virtualization" solution it would require a bug in the hypervisor for that to happen. Jails are not necessarily any more or less secure than a "platform virtualization" solution as it is going to come down to implementation details. Bugs do happen in both worlds.

Another trade-off is that a jail can not emulate arbitrary hardware like a VMM can. If you want to add a new virtual disk to your VM in a "platform virtualization" solution it is a simple operation - the physical disk doesn't really exist as it is just a file on the filesystem of the host. In a jail you can not emulate arbitrary hardware.

As a jail is really just isolating processes from each other, it is important to realize that root on the host has complete control over every jail. This is an important thing to keep in mind when setting up a jail environment. Root on the host should be trusted and controlled far more than root in any of the jails.

Lastly, a user on the host can get access to things inside one of the jails if the UIDs are the same. For example, on one of my hosts my UID is 1001, and inside one of the jails a different user (user1) has UID 1001. From the viewpoint of user1, only he has access to his files inside the jail. From my viewpoint, outside of the jail, the files are owned by me. The host is going to use it's copy of /etc/passwd to determine ownership of files, which means there can be overlapping information. This is another important consideration to keep in mind when setting up a jail environment.

####USES

There are many uses for jails. Lots of places use them to isolate web hosting environments from each other, to provide root inside a contained system for a customer, and to isolate developers from each other. I personally use them to isolate test environments from each other. As a developer who spends most of his time up in userland, this is a great solution to my need to be able to quickly setup a clean test environment. As you spend more time with jails, you begin to see different opportunities for application.

As more parts of FreeBSD become friendlier to jails you can start to build very interesting things. Virtual network stacks, zfs, multi-IP jails, hierarchical jails and many other things are fertile areas for exploration as a systems administrator. As is often the case, the best way to get familiar with jails is to dive in heads first!


FROM: http://sysadvent.blogspot.hk/2010/12/day-14-freebsd-jails.html