---
layout: post
title: Cmake_symlink_library Error in Shared Folder of VirtualBox
date: 2016-03-19
modified: 2016-03-25
comments: true
tags: code error
---

# Problem

When building OpenCV-3.1.0 in Ubuntu 14.04 installed inside VirtualBox 5.0.16 (in Win7 host), I encountered the following error:   

```shell
cmake_symlink_library: System Error: Read-only file system
```

Both the OpenCV source folder and build folder locate in a shared folder that resides outside the VirtualBox's virtual hard disk (in my Win7 host drive). And I created a symbolic link inside the Ubuntu home folder   

```shell
~/project/ -> /media/sf_D_DRIVE/project/
```

I have full access to the share folder (I can read and write without any issue).

# Solution

After Google, I found [several web pages with similar problem](https://cmake.org/Bug/view.php?id=15837),
which appears to be a well-known issue for VirtualBox existing for a while.

I solved the issue according to [this](https://www.virtualbox.org/ticket/10085#comment:32):

1. Shut down the VirtualBox
2. Open CMD.exe:  
   
   ```shell
   cd C:/Program Files/Oracle/VirtualBox
   VBoxManage setextradata VM_NAME VBoxInternal2/SharedFoldersEnableSymlinksCreate/SHARE_NAME 1
   ```  
where VM_NAME=ubuntu14 and SHARE_NAME=D_DRIVE (Yes, remove the "sf_" prefix) in my case.
3. Open VirtualBox as Administrator, and start the ubuntu, rebuild OpenCV, everything works fine now!