---
title: "Sync folders in Vagrant but exclude some folders from syncing"
description: "Here is an example how to exclude folder in your Vagrant box from syncing. this saves you a lot of time and nerves"
date: 2017-01-19
---

Maybe you know this:     
you have a huge Java project, maybe the project itself is a sum of many (not even so small) Java projects.     
For sharing the complete setting with other developers you normally use a [Vagrant box](https://www.vagrantup.com) to make it easy.    
At work we needed 2 big projects in one Vagrant box, which included over 10 small maven projects.   
Normally I would share it like this:    
```bash
config.vm.synced_folder "my-project1", "/home/vagrant/my-project1"
config.vm.synced_folder "my-project2", "/home/vagrant/my-project2"
```
You can imagine, the normal start-time of our server with permanently syncing all folders, including all the target folders took much too long.     
On a local machine, without a Vagrant box, it normally needs around 2 minutes to start      
But with syncing all target folders, it needs between 7-10 minutes to start the server     

Of course, this is not acceptable at all.    

So i was looking for some better way for syncing it, and I came to a good solution rsync

```bash
config.vm.synced_folder "my-project1", "/home/vagrant/my-project1", type: "rsync", :rsync__exclude => ['my-project1/, my-project1/mini_project2/target,my-project1/mini_project2/target,my-project1/mini_project3/target,my-project1/mini_project4/target']
config.vm.synced_folder "my-project2", "/home/vagrant/my-project2", type: "rsync", :rsync__exclude => ['my-project2/, my-project2/mini_project2/target,my-project2/mini_project2/target,my-project2/mini_project3/target,my-project2/mini_project4/target']
```

You do this with every folder you need in your Vagrant box and where you don't want everything synced.

At the moment it only syncs the folder from you host to your guest system on `vagrant up` and `vagrant reload`    
Of course you want to have any changes to be synced to your developer box.     

So the solution is:     
Open two terminal shells:     

First terminal:
```bash
vagrant rsync-auto
```

Second terminal:
```bash
vagrant up
```

When you change now something in the host system, it's automatically synced on your guest system
