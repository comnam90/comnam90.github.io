---
id: 89
title: Bug when applying KB4038782 September CU to Storage Spaces Direct Clusters
date: 2017-09-17T09:07:34+00:00
author: Ben Thomas
layout: post
guid: https://bcthomas.com/?p=89
permalink: /2017/09/bug-when-applying-kb4038782-september-cu-to-storage-spaces-direct-clusters/
force_ssl:
  - "1"
categories:
  - Storage Spaces Direct
  - Windows Server
tags:
  - bug
  - KB4038782
  - Patching
  - S2D
  - September CU
  - Storage Spaces Direct
  - Windows Server
  - WS2016
---
_UPDATE(2017-09-19): Microsoft have officially recognized the bug and have a KB describing the symptoms and workaround much like the below. See here: <https://support.microsoft.com/en-us/help/4043361/disks-in-maintenance-mode-status-after-september-cumulative-update-kb>_

I was patching our dev cluster the other day and came across a new issue when applying the latest September Cumulative Update (KB4038782), and it seems others on the internet have hit this issue as well.

## Background

First, a bit of background on the expected behaviour when performing maintenance:

Basically when you are performing maintenance on a Storage Spaces Direct (S2D) Cluster, you would normally use Failover Cluster Manager (GUI or Powershell) to Pause and Drain the node first, and then patch and reboot it, and eventually resume and fail back its rolls.

Now doing this would in turn, mark the physical disks associated with that host as being in maintenance mode during this period to redirect IO away from them, and then when the host is resumed, the disks are taken out of maintenance mode and IO is resumed to them and the storage re-sync begins.

This is all automated for you if you use the awesome Cluster Aware Updating (CAU) features, which will patch whole clusters for you without your manual involvement.

## Problem:

What I found while patching this month, both manually and with CAU, is that when the hosts go into maintenance mode and are paused and drained, their disks are also marked as being in maintenance mode, however after patching and rebooting, they get resumed but their disks stay in maintenance mode.

[<img class="aligncenter size-full wp-image-92" src="https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.45-AM.png?resize=558%2C542&#038;ssl=1" alt="" width="558" height="542" srcset="https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.45-AM.png?w=558&ssl=1 558w, https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.45-AM.png?resize=300%2C291&ssl=1 300w" sizes="(max-width: 558px) 100vw, 558px" data-recalc-dims="1" />](https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.45-AM.png?ssl=1) [<img class="aligncenter size-large wp-image-91" src="https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.15-AM-909x1024.png?resize=640%2C721&#038;ssl=1" alt="" width="640" height="721" srcset="https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.15-AM.png?resize=909%2C1024&ssl=1 909w, https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.15-AM.png?resize=266%2C300&ssl=1 266w, https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.15-AM.png?resize=768%2C865&ssl=1 768w, https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.15-AM.png?w=1410&ssl=1 1410w, https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.15-AM.png?w=1280 1280w" sizes="(max-width: 640px) 100vw, 640px" data-recalc-dims="1" />](https://i2.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.22.15-AM.png?ssl=1)

What this means, is that if you&#8217;re using CAU, it will fail to patch the cluster after the first host or two, depending on your resiliency, as it doesn&#8217;t have enough spare capacity left to take more disks out of maintenance mode, even though all the hosts are up and healthy.

You can check this by running the following Powershell on a cluster host

<pre class="lang:ps decode:true">Get-StorageNode |%{$_.Name;$_ | Get-PhysicalDisk -PhysicallyConnected}</pre>

## Workaround and Fix

Now, if you&#8217;ve already started patching and need to recover from this, you can use the following Powershell to remediate the hosts you&#8217;ve patched

<pre class="lang:ps decode:true ">Repair-ClusterStorageSpacesDirect -Node [hostname] -DisableStorageMaintenanceMode</pre>

[<img class="aligncenter size-large wp-image-94" src="https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.57.09-AM-1024x577.png?resize=640%2C361&#038;ssl=1" alt="" width="640" height="361" srcset="https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.57.09-AM.png?resize=1024%2C577&ssl=1 1024w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.57.09-AM.png?resize=300%2C169&ssl=1 300w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.57.09-AM.png?resize=768%2C433&ssl=1 768w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.57.09-AM.png?w=1416&ssl=1 1416w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.57.09-AM.png?w=1280 1280w" sizes="(max-width: 640px) 100vw, 640px" data-recalc-dims="1" />](https://i0.wp.com/bcthomas.com/wp-content/uploads/2017/09/Screen-Shot-2017-09-17-at-8.57.09-AM.png?ssl=1)

If you haven&#8217;t patched, then you can avoid this by choosing to &#8216;Pause and Don&#8217;t Drain&#8217; your hosts in Failover Cluster Manager, and then manually live migrate your VMs and Virtual Disks to other cluster members before patching and rebooting the host, as this will avoid the physical disks being put into maintenance mode in the first place.

## My take on the problem

Having dealt with S2D for a while, I think what we&#8217;re seeing here is a change in behaviour to how maintenance is performed on S2D Cluster hosts, and so when this new logic has been pushed out with the September CU, it&#8217;s conflicting with the old logic and leaving things in a halfway house.

I very much doubt we will see this problem in next month&#8217;s patching, as it looks like once September CU is applied, the issue isn&#8217;t reproducible when putting hosts in and out of maintenance mode.

Hopefully this helps anyone else coming across this issue