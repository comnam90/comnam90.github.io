---
id: 63
title: My Wishlist for System Center Virtual Machine Manager
date: 2016-10-24T22:40:36+00:00
author: Ben Thomas
layout: post
guid: https://bcthomas.com/?p=63
permalink: /2016/10/my-wishlist-for-system-center-virtual-machine-manager/
force_ssl:
  - "1"
force_ssl_children:
  - "1"
image: /wp-content/uploads/2016/10/system-center-virtual-machine-manager.png
categories:
  - System Center
  - Virtual Machine Manager
tags:
  - SCVMM
  - System Center
  - Virtual Machine Manager
  - wishlist
---
I&#8217;ve used System Center Virtual Machine Manager for a few years now, and I&#8217;ve come to like its ups and deal with its downs.

It has a lot of great features, like bare-metal deployments and logical networks, which when executed correctly are both huge time savers and take away a lot of human error.

With SCVMM, let&#8217;s start with some of a new additions in 2016:

  * Converting &#8216;Standard Switches&#8217; on hosts to Logical Switches 
      * Too often in the past I&#8217;ve retro-fitted VMM into a Hyper-V Environment and had to wrestle with removing existing Standard Switches and replacing them with Logical switches and had to deal with migrating VMs, losing connectivity, rolling back. Now all that pain is gone with a single button that allows you to convert your Standard Switch to a Logical Switch if one with matching properties exists.
  * Full Orchestration for Rolling Cluster Upgrades 
      * One of the coolest features of 2016 is Rolling Cluster upgrades, it makes upgrades forever easier by allowing you to run mixed mode clusters during an upgrade process and then raise then cluster functional level once all hosts are upgraded. Think of it like you would Active Directory and Domain Controllers, upgrade them all to the new level one at a time and then raise your functional level.
      * VMM builds on top of this by combining it with bare-metal deployments to automate the entire process! How freaking cool is that? Especially with the next point
  * Support for &#8216;Storage Spaces Direct&#8217; and Storage QoS 
      * You can deploy a complete hyperconverged cluster using VMMs bare-metal deployment
      * Create new volumes on your S2D clusters
      * Manage your Storage QoS policies with a nice GUI
      * And many more storage tasks

Now some of the things that I feel are still missing in VMM:

  * &#8216;Getting started&#8217; GUI after initial installation 
      * Anyone who&#8217;s deployed a VMware vCenter will know what I&#8217;m talking about, it&#8217;s that lovely &#8216;Getting Started&#8217; tab on everything in the GUI that helps you work your way through creating your Datacenters (aka. Host Groups), Adding your hosts, creating your clusters, deploying your first VM. It makes dealing with vCenter for the first time just that little bit easier, and it doesn&#8217;t feel like you need a PHD to configure it or if you do something wrong the thing will implode.
      * For those that have never touch VMware, here&#8217;s what I&#8217;m talking about [<img class="aligncenter size-large wp-image-69" src="https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-10.58.45-PM-1024x697.png?resize=640%2C436&#038;ssl=1" alt="vCenter Datacenter Getting Started view" width="640" height="436" srcset="https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-10.58.45-PM.png?resize=1024%2C697&ssl=1 1024w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-10.58.45-PM.png?resize=300%2C204&ssl=1 300w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-10.58.45-PM.png?resize=768%2C523&ssl=1 768w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-10.58.45-PM.png?w=1584&ssl=1 1584w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-10.58.45-PM.png?w=1280 1280w" sizes="(max-width: 640px) 100vw, 640px" data-recalc-dims="1" />](https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-10.58.45-PM.png?ssl=1)

  * Advanced use of Storage QoS Features 
      * Now creation of Storage QoS policies in VMM has a nice easy to use GUI in front of it, but they could&#8217;ve done so much more with it
      * Delegation of policies to Clouds/Tenants so that VM admins can assign the right policies to their VMs without needing to be fabric admins
      * Ability to set policies against VMs/Clouds/Host Groups. It&#8217;d be nice to get a default QoS policy against a cloud and have it automatically propagate to all child VM disks. And then be able to get more granular setting policies against a VM or specific disks on a VM that need special treatment. Instead my only option is to set a policy manually (or scripted) against even VM disk, just like if I didn&#8217;t have VMM.
  * Quotas not counting compute resources of powered off VMs 
      * Now quotas and Cloud Capacity limitations are a great way to give a department or client access to a subset of your resources and not have to worry about them consuming everything while you&#8217;re not looking, but what if that department if full of Devs who have multiple environments but only have 1 running at a time? Well they don&#8217;t need resources for all of their environments, but just enough to power on 1 of them. Too bad for you VMM counts the CPU and Memory usage of powered off VMs towards their Quota too.
  * Configuration of VLANs on host physical adapters 
      * Bare-metal deployment is great, it allows you to push out an identical configuration to all of your hosts. But what if you have NICs you don&#8217;t want to put on a switch, say for storage traffic? And they need to have a VLAN configured on them? Well VMM currently doesn&#8217;t let you do that. Instead after VMM has deployed the host you need to go through with your Set-NetAdapter command and configure those VLANs.
  * Automatic configuration of VMQ settings for Bare-metal deployed hosts 
      * VMQ is something all Hyper-V admins will struggle with at some stage or another. It would be nice if VMM could take away the pain when deploying hosts by looking at the host hardware, the networking it&#8217;s going to apply and then work out the best VMQ settings to apply.
  * Tenant view of cloud capacity 
      * Now this may seem obvious, but it&#8217;s not there. You as an Administrator can see the lovely cloud capacity view, however the tenant admin cannot.[<img class="aligncenter size-large wp-image-71" src="https://i2.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-11.22.14-PM-1024x478.png?resize=640%2C299&#038;ssl=1" alt="VMM Cloud Summary View" width="640" height="299" srcset="https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-11.22.14-PM.png?resize=1024%2C478&ssl=1 1024w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-11.22.14-PM.png?resize=300%2C140&ssl=1 300w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-11.22.14-PM.png?resize=768%2C359&ssl=1 768w, https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-11.22.14-PM.png?w=1314&ssl=1 1314w" sizes="(max-width: 640px) 100vw, 640px" data-recalc-dims="1" />](https://i0.wp.com/bcthomas.com/wp-content/uploads/2016/10/Screen-Shot-2016-10-24-at-11.22.14-PM.png?ssl=1)
  * No native support of Hyper-V Replica 
      * This one for me is just mind-boggling. VMM by name is designed to sit overtop of Hyper-V to help you manage your VMs. How is it meant to do that though when it doesn&#8217;t support all the core Hyper-V features? Like say Hyper-V Replica?
      * FYI don&#8217;t bother telling me if I integrate VMM with Azure Site Recovery then I&#8217;ll get Hyper-V Replica in VMM, because why should I pay for a cloud service to manage my on-prem service to manage my on-prem to on-prem service? That&#8217;s also just a lot of moving parts I don&#8217;t want to have to worry about.
  * One Powershell to rule them all 
      * Again taking a leaf out of VMware&#8217;s book, it would be create if there was just 1 powershell module and 1 set of commands for managing Hyper-V hosts and VMs whether they be standalone hosts, in a cluster or managed by VMM.
      * It would be nice if the VMM powershell commands returned at least the same details that their Hyper-V modules counterparts do &#8211; eg. where is my VM uptime in Get-SCVirtualMachine? And why must you alias Get-VM and break my scripts?
  * UPDATE: Tenants unable to update VM Configuration Versions 
      * If the whole point of having Tenant Admins is to give power back to the users for all things related to their VMs, then why can&#8217;t they upgrade the VM Configuration version? Why must this be done by a fabric admin?

This may have ended up sounding a bit ranty, but I do like VMM, it does make my life easier on the whole when managing Hyper-V environments. If they could just get the last few pieces right, then I think we&#8217;d see more people coming across from VMware land.