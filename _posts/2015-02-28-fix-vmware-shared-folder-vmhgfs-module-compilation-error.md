---
id: 190
title: Fix VMware Shared Folder vmhgfs Module Compilation Error
date: 2015-02-28T15:04:26+00:00
author: Ke
layout: post
guid: http://www.keblog.me/?p=190
permalink: /2015/02/fix-vmware-shared-folder-vmhgfs-module-compilation-error/
comments: true 
dsq_thread_id:
  - 3556560439
categories:
  - programming
tags:
  - Linux
---
Today my ubuntu 14 updated its kernel to 3.16 then VMware shared folder disappeared in /mnt/hgfs. I tried to recompile vmhgfs modules with vmware-config-tools.pl and reinstall vmware-tools, but both failed with error messages:

<pre class="lang:sh decode:true " >from /tmp/modconfig-qqjwHn/vmhgfs-only/inode.c:29:
/tmp/modconfig-qqjwHn/vmhgfs-only/inode.c: In function ‘HgfsPermission’:
include/linux/kernel.h:793:27: error: ‘struct dentry’ has no member named ‘d_alias’</pre>

This bug has already been reported to [VMware](https://communities.vmware.com/message/2477575 "https://communities.vmware.com/message/2477575"). Before VMware release official patch, we can manually fix this problem.

1. Uncompress your VMwareTools-\*\\*\*-\*\*\***.tar.gz (eg. VMwareTools-9.9.2-2496486.tar.gz)

2. cd YOURPATH/vmware-tools-distrib/lib/modules/source/

3. tar xvf vmhgfs.tar

4. cd vmhgfs-only

5. substitute d\_alias with d\_u.d_alias in inode.c

6. tar cvf vmhgfs.tar vmhgfs-only

Then re-run ./vmware-install.pl, everything should be fine now.
