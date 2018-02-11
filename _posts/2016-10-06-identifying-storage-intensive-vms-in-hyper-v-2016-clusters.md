---
id: 41
title: Identifying storage intensive VMs in Hyper-V 2016 Clusters
date: 2016-10-06T20:17:24+00:00
author: Ben Thomas
layout: post
guid: https://bcthomas.com/?p=41
permalink: /2016/10/identifying-storage-intensive-vms-in-hyper-v-2016-clusters/
force_ssl:
  - "1"
categories:
  - Hyper-V
tags: [powershell, s2d, ws2016]
---
We&#8217;ve all had the case where there was a volume running hot on your cluster and you spend ages wrestling with perf counters to try to find that VM that&#8217;s causing your storage to burn. Well let me introduce you to a magical new command in Windows Server 2016

```powershell
Get-StorageQoSFlow
```

This miracle command can give you insights on all the VHD(x)s running on your cluster, revealing IOPS, Latency and Bandwidth stats for them all without the need for large-scale monitoring solutions.

Let&#8217;s look at a few examples of how we can use this to our advantage, starting with the top 5 busiest VHD(x)s

```powershell
Get-StorageQoSFlow -CimSession ClusterName | Sort-Object InitiatorIOPS -Descending | select -First 5
```

[<img class="aligncenter wp-image-46 size-full" src="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?resize=640%2C69&#038;ssl=1" alt="Get-StorageQoSFlow Top 5" width="640" height="69" srcset="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?w=2284&ssl=1 2284w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?resize=300%2C33&ssl=1 300w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?resize=768%2C83&ssl=1 768w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?resize=1024%2C111&ssl=1 1024w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?w=1280 1280w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?w=1920 1920w" sizes="(max-width: 640px) 100vw, 640px" data-recalc-dims="1" />](https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-8.45.34-PM.jpg?ssl=1)<del></del>

In my example above, we can see straight away both the busiest machine, and exactly which of its disks is creating the load.

Next let&#8217;s move on to checking the performance of a specific VM, which would look like this

```powershell
Get-StorageQoSFlow -InitiatorName VMName -CimSession ClusterName
```

[<img class="aligncenter wp-image-50 size-full" src="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?resize=640%2C28&#038;ssl=1" alt="Get-StorageQoSFlow Specific VM" width="640" height="28" srcset="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?w=2248&ssl=1 2248w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?resize=300%2C13&ssl=1 300w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?resize=768%2C33&ssl=1 768w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?resize=1024%2C45&ssl=1 1024w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?w=1280 1280w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?w=1920 1920w" sizes="(max-width: 640px) 100vw, 640px" data-recalc-dims="1" />](https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-06-at-9.01.33-PM.jpg?ssl=1)

Now these two uses of the command alone have already helped a fellow admin find several VMs that were thought to be idle, actually consuming around about 15,000 IOPS due to a few rough processes. These were precious IOPS that were better served being available to more critical services.

For bonus points, here are a few more commands to look at when looking for storage stats

```powershell
# Get stats for any CSV
Get-StorageQoSVolume -CimSession ClusterName
# Get stats for a Storage Spaces Direct Cluster
Get-StorageSubSystem clu*  | Get-StorageHealthReport 
Get-Volume -CimSession ClusterName | ?{$_.FileSystem -eq 'CSVFS'} | Get-StorageHealthReport -CimSession ClusterName
```

Until next time!