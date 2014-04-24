---
layout: post
title: OS杂记[5]
categories:
- OS
tags:
- Ubuntu
- vmnet
- vmware

---

#### PROBLEM

这几天把Ubuntu系统升级到了14.04,中间基本上没有什么问题，但我系统上原有的VMWare却不能运行了，每次想打开VMWare-Workstation，总提示说是有些models需要重新编译，于是就按照提示一步步编译，但在vmnet-adaptor这一步总是出错。


> 2014-04-24T20:57:48.429+08:00| vthread-3| I120: Invoking modinfo on "vmnet".
> 
> 2014-04-24T20:57:48.430+08:00| vthread-3| I120: "/sbin/modinfo" exited with status 256.
> 
> 2014-04-24T20:57:48.753+08:00| vthread-3| I120: Setting destination path for vmnet to "/lib/modules/3.13.0-24-generic/misc/vmnet.ko".
> 
> 2014-04-24T20:57:48.753+08:00| vthread-3| I120: Extracting the vmnet source from "/usr/lib/vmware/modules/source/vmnet.tar".
> 
> 2014-04-24T20:57:48.757+08:00| vthread-3| I120: Successfully extracted the vmnet source.
> 
> 2014-04-24T20:57:48.757+08:00| vthread-3| I120: Building module with command "/usr/bin/make -j8 -C /tmp/modconfig-sQwsyG/vmnet-only auto-build HEADER_DIR=/lib/modules/3.13.0-24-generic/build/include CC=/usr/bin/gcc IS_GCC_3=no"
> 
> 2014-04-24T20:57:50.165+08:00| vthread-3| W110: Failed to build vmnet.  Failed to execute the build command.
> 
> root@jeff-pc:/tmp/vmware-root# /usr/bin/make -j8 -C /tmp/modconfig-sQwsyG/vmnet-only auto-build HEADER_DIR=/lib/modules/3.13.0-24-generic/build/include CC=/usr/bin/gcc IS_GCC_3=no
> 
> Using 2.6.x kernel build system.
> 
> make: Entering directory `/tmp/modconfig-sQwsyG/vmnet-only'
> 
> /usr/bin/make -C /lib/modules/3.13.0-24-generic/build/include/.. SUBDIRS=$PWD SRCROOT=$PWD/. \
> 
>           MODULEBUILDDIR= modules
> 
> make[1]: Entering directory `/usr/src/linux-headers-3.13.0-24-generic'
> 
>   CC [M]  /tmp/modconfig-sQwsyG/vmnet-only/filter.o
> 
>   CC [M]  /tmp/modconfig-sQwsyG/vmnet-only/smac.o
> 
>   CC [M]  /tmp/modconfig-sQwsyG/vmnet-only/vnetEvent.o
> 
>   CC [M]  /tmp/modconfig-sQwsyG/vmnet-only/vnetUserListener.o
> 
> /tmp/modconfig-sQwsyG/vmnet-only/filter.c:206:1: error: conflicting types for ‘VNetFilterHookFn’
> 
>  VNetFilterHookFn(unsigned int hooknum,                 // IN:
> 
>  ^
> 
> /tmp/modconfig-sQwsyG/vmnet-only/filter.c:64:18: note: previous declaration of ‘VNetFilterHookFn’ was here
> 
>  static nf_hookfn VNetFilterHookFn;
> 
>                   ^
> 
> /tmp/modconfig-sQwsyG/vmnet-only/filter.c:64:18: warning: ‘VNetFilterHookFn’ used but never defined [enabled by default]
> 
> /tmp/modconfig-sQwsyG/vmnet-only/filter.c:206:1: warning: ‘VNetFilterHookFn’ defined but not used [-Wunused-function]
> 
>  VNetFilterHookFn(unsigned int hooknum,                 // IN:
> 
>  ^
> 
> make[2]: *** [/tmp/modconfig-sQwsyG/vmnet-only/filter.o] Error 1
> 
> make[2]: *** Waiting for unfinished jobs....
> 
> make[1]: *** [_module_/tmp/modconfig-sQwsyG/vmnet-only] Error 2
> 
> make[1]: Leaving directory `/usr/src/linux-headers-3.13.0-24-generic'
> 
> make: *** [vmnet.ko] Error 2
> 
> make: Leaving directory `/tmp/modconfig-sQwsyG/vmnet-only'
> 

####PATCH

既然发现了是*VNetFilterHookFn*出的问题,那下一步就是先google下怎么回事了。网上给出了一个patch，是/usr/lib/vmware/models/source/vmnet.tar中的filter.c出的问题。该怎样打补丁我不会，但是可以把这个patch中的内容coding到vmnet.tar中的filter.c这个文件中。下面是这个patch的内容：


> --- vmnet-only/filter.c 2013-10-18 23:11:55.000000000 +0400
> 
> +++ vmnet-only/filter.c 2013-12-03 04:16:31.751352170 +0400
> 
> @@ -27,6 +27,7 @@
> 
>  #include "compat_module.h"
> 
>  #include <linux/mutex.h>
> 
>  #include <linux/netdevice.h>
> 
> +#include <linux/version.h>
> 
>  #if COMPAT_LINUX_VERSION_CHECK_LT(3, 2, 0)
> 
>  #   include <linux/module.h>
> 
>  #else
> 
> @@ -203,7 +204,11 @@
> 
>  #endif
> 
>  
> 
>  static unsigned int
> 
> +#if LINUX_VERSION_CODE < KERNEL_VERSION(3, 13, 0)
> 
>  VNetFilterHookFn(unsigned int hooknum,                 // IN:
> 
> +#else
> 
> +VNetFilterHookFn(const struct nf_hook_ops *ops,        // IN:
> 
> +#endif
> 
>  #ifdef VMW_NFHOOK_USES_SKB
> 
>                   struct sk_buff *skb,                  // IN:
> 
>  #else
> 
> @@ -252,7 +257,14 @@
> 
>  
> 
>     /* When the host transmits, hooknum is VMW_NF_INET_POST_ROUTING. */
> 
>     /* When the host receives, hooknum is VMW_NF_INET_LOCAL_IN. */
> 
> -   transmit = (hooknum == VMW_NF_INET_POST_ROUTING);
> 
> +
> 
> +#if LINUX_VERSION_CODE < KERNEL_VERSION(3, 13, 0)
> 
> +    transmit = (hooknum == VMW_NF_INET_POST_ROUTING);
> 
> +#else
> 
> +    transmit = (ops->hooknum == VMW_NF_INET_POST_ROUTING);
> 
> +#endif
> 
> +
> 
>     packetHeader = compat_skb_network_header(skb);
> 
>     ip = (struct iphdr*)packetHeader;
> 
> 

只要把上面的patch更新到/usr/lib/vmware/models/source/vmnet.tar中就可以再次运行VMWare-Workstation了，剩下的事情按照提示点ok就行了。