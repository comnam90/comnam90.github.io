---
id: 23
title: Expanding Storage Spaces Direct Volumes
date: 2016-10-03T19:53:46+00:00
author: Ben Thomas
layout: post
guid: https://bcthomas.com/?p=23
permalink: /2016/10/expanding-storage-spaces-direct-volumes/
force_ssl:
  - "1"
force_ssl_children:
  - "1"
categories:
  - Storage Spaces Direct
  - Windows Server
tags:
  - Powershell
  - S2D
  - Storage Spaces Direct
  - Windows Server
  - WS2016
---
As many of you would have seen, Windows Server 2016 has been [officially launched](https://blogs.technet.microsoft.com/hybridcloud/2016/09/26/announcing-the-launch-of-windows-server-2016/), with evaluation media available and General Availability slated for later this month.

One of the great new features in this release, is Storage Spaces Direct, a Software-Defined Storage Solution. There is already plenty of information available on how to get this up and running on [Technet](https://technet.microsoft.com/en-us/windows-server-docs/storage/storage-spaces/storage-spaces-direct-overview), but I thought I&#8217;d share some of the operational tasks that aren&#8217;t so obvious, starting with expanding volumes.

Firstly we want to get the current size of the existing storage tiers that make up our volume

{% highlight powershell %}
Get-VirtualDisk -FriendlyName vol1 | Get-StorageTier | Format-Table FriendlyName, @{Label=’Freespace(GB)’;Expression={$_.Size/1GB}} -autosize
{% endhighlight %}
<figure id="attachment_29" style="width: 300px" class="wp-caption aligncenter">

<img class="wp-image-29 size-medium" src="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-03-at-7.53.44-PM-300x73.png?resize=300%2C73&#038;ssl=1" alt="Current size of Storage Tiers" width="300" height="73" srcset="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-03-at-7.53.44-PM.png?resize=300%2C73&ssl=1 300w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-03-at-7.53.44-PM.png?w=446&ssl=1 446w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /><figcaption class="wp-caption-text">Current size of Storage Tiers</figcaption></figure> 

Now in this example, we&#8217;re using a Multi-Resiliency Virtual Disk, which allows us to combine the performance of Mirror with the efficiency of Parity to get the best of both worlds. Let&#8217;s say this volume has a SQL Server on it, and I know my working data set hasn&#8217;t increased but I&#8217;ve got lots of cold data and my volume is almost full. I want to just add capacity to the volume to continue to store more cold data.

We want to select just the Parity tier and expand that one

```powershell
Get-VirtualDisk -FriendlyName vol1 | Get-StorageTier | ?{$_.ResiliencySettingName -eq 'Parity'} | Resize-StorageTier -Size 1000GB
```

And then expand the partition to match the new size


```powershell
Get-VirtualDisk -FriendlyName vol1 | Get-Disk | Get-Partition | ?{$_.type -eq 'Basic'} | Resize-Partition -Size 1100GB
```

Now if we check the storage tiers again we should see our new sizes

<pre class="wrap:true lang:ps decode:true ">Get-VirtualDisk -FriendlyName vol1 | Get-StorageTier | Format-Table FriendlyName, @{Label=’Freespace(GB)’;Expression={$_.Size/1GB}} -autosize</pre><figure id="attachment_33" style="width: 300px" class="wp-caption aligncenter">

<img class="size-medium wp-image-33" src="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-03-at-8.43.17-PM-300x72.png?resize=300%2C72&#038;ssl=1" alt="Size of Storage Tiers after expansion" width="300" height="72" srcset="https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-03-at-8.43.17-PM.png?resize=300%2C72&ssl=1 300w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-03-at-8.43.17-PM.png?w=440&ssl=1 440w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /><figcaption class="wp-caption-text">Size of Storage Tiers after expansion</figcaption></figure> 

If you wanted to add more Performance Tier to your volume to account for an increase in your active working set, or the data that is write intensive, then you would just have to swap out Parity for Mirror in the command where we expanded the tier.

Shrinking of S2D (Storage Spaces Direct) volumes is not currently supported, however this is a less common scenario.

I&#8217;ll be posting more tips like this around Storage in Windows Server 2016 over the next few weeks

&nbsp;