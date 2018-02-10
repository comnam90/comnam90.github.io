---
id: 77
title: S2D Storage Tiers misconfiguration bug
date: 2016-11-09T07:26:40+00:00
author: Ben Thomas
layout: post
guid: https://bcthomas.com/?p=77
permalink: /2016/11/s2d-storage-tiers-misconfiguration-bug/
force_ssl:
  - "1"
force_ssl_children:
  - "1"
categories:
  - Storage Spaces Direct
  - Windows Server
tags:
  - bug
  - S2D
  - Storage Spaces Direct
  - WS2016
---
I&#8217;ve been deploying a few Storage Spaces Direct (S2D) clusters lately, and I noticed a slight mis-configuration that can occur on deployment.

Normally when deploying S2D, the disk types in the nodes are detected and the fastest disk (usually NVMe or SSD) is assigned to the cache, while the next fastest is used for the Performance Tier and the slowest being used in the Capacity Tier. So if you have NVMe, SSD and HDD, you would end up with an NVMe Cache, a SSD Performance Tier and a HDD Capacity Tier. With only 2 disk types, either NVMe+SSD, NVMe+HDD or SSD+HDD, the fastest disk is used for Cache and the other used for both the Performance and Capacity Tiers.

[<img class="aligncenter size-full wp-image-79" src="https://i2.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Expected-Settings.jpg?resize=473%2C48&#038;ssl=1" alt="s2d-storage-tiers-expected-settings" width="473" height="48" srcset="https://i2.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Expected-Settings.jpg?w=473&ssl=1 473w, https://i2.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Expected-Settings.jpg?resize=300%2C30&ssl=1 300w" sizes="(max-width: 473px) 100vw, 473px" data-recalc-dims="1" />](https://i2.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Expected-Settings.jpg?ssl=1)It seems there is a bug that is causing the Performance Tier to set its mediatype to the fastest disk in the system, even if that disk is already allocated to cache.

In my deployments, I&#8217;ve got 4x SSD and 12x HDD in each my hosts, with the expectation that the SSD is used for cache and the HDD is mirrored for the Performance Tier and in Parity for Capacity Tier. However even though the SSD is being assigned correctly to the Cache, the Performace Tier shows as an SSD mediatype, preventing me from deploying volumes with mirrored HDD until this is correctly.

[<img class="aligncenter size-full wp-image-78" src="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Actual-Settings.jpg?resize=475%2C54&#038;ssl=1" alt="s2d-storage-tiers-actual-settings" width="475" height="54" srcset="https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Actual-Settings.jpg?w=475&ssl=1 475w, https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Actual-Settings.jpg?resize=300%2C34&ssl=1 300w" sizes="(max-width: 475px) 100vw, 475px" data-recalc-dims="1" />](https://i1.wp.com/bcthomas.com/wp-content/uploads/2016/11/S2D-Storage-Tiers-Actual-Settings.jpg?ssl=1)You can check your own systems by running the below Powershell Command

```powershell
Get-StorageTier | select FriendlyName, ResiliencySettingName, MediaType, PhysicalDiskRedundancy
```

If you&#8217;ve got the same problem, it can be very easily corrected by simply changing the mediatype on the Tier back to HDD

```powershell
Get-StorageTier -FriendlyName Performance | Set-StorageTier -MediaType HDD
```

I&#8217;m sure there will be a permanent fix along for this soon, but in the mean time I hope this helps others who come across the same problem.