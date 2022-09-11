---
title: "How to run Home Assistant OS on MacOS M1 with UTM"
author: Kris Bogaerts
date: 2022-09-10
category: [Virtualization, Smart Home, MacOS ]
tags: [Home Assistant, Home Assistant Operating system, MacOS, Macbook Pro M1, arm64, UTM]
---

Testing configuration/add-ons on my Home Assistant production instance comes with a risk. 

I always used the Home Assistant add-on development container with Visual Studio Code, but today I needed the journal inside the Crowdsec add-in, and it seems that this is not available in the dev container. 

Also, rebuilding HA and restoring backups comes with some frustrations :-). Using a MacBook Pro M1 and VMWare fusion does not seem to with Home Assistant Operating system. 

This note is about installing the Home Assistant Operating system on my MacBook Pro M1 with UTM as Hypervisor. This should give me full virtualization and cloning (but no snapshots), which should make my life easier when testing and creating tech notes.

## Home Assistant Operating system

Download the [HA OS aarch64 qcow2](https://github.com/home-assistant/operating-system/releases/download/9.0.rc2/haos_generic-aarch64-9.0.rc2.qcow2.xz) image and extract

## UTM (Securely run operating systems on your Mac) 
Download and install [UTM ](https://mac.getutm.app/)


## Install the Home Assistant Operating system virtual machine

Create a Virtual machine:
![](/assets/img/2022-09-10-21-16-31.png){: w="700" h="400" }

Choose Virtualize
![](/assets/img/2022-09-10-21-17-37.png){: w="700" h="400" }

Type Linux
![](/assets/img/2022-09-10-21-32-55.png){: w="700" h="400" }

Select the downloaded (extracted) image
![](/assets/img/2022-09-10-21-22-29.png){: w="700" h="400" }

![](/assets/img/2022-09-10-21-22-58.png){: w="700" h="400" }

leave default
![](/assets/img/2022-09-10-21-23-23.png){: w="700" h="400" }

![](/assets/img/2022-09-10-21-23-42.png){: w="700" h="400" }

![](/assets/img/2022-09-10-21-24-02.png){: w="700" h="400" }

On the summary page, enter a name and select open VM settings
![](/assets/img/2022-09-10-21-24-47.png){: w="700" h="400" }

Change the networking to bridge mode
![](/assets/img/2022-09-10-21-25-18.png){: w="700" h="400" }

Select the VirtIO drive and delete
![](/assets/img/2022-09-10-21-25-53.png){: w="700" h="400" }

Select "New Drive"
![](/assets/img/2022-09-10-21-26-20.png){: w="700" h="400" }

Import the qcow2 HA OS image and save
![](/assets/img/2022-09-10-21-26-46.png){: w="700" h="400" }

![](/assets/img/2022-09-10-21-29-08.png){: w="700" h="400" }

Start the VM and this should give you the details to connect to the Home Assistant interface
![](/assets/img/2022-09-10-21-30-25.png){: w="700" h="400" }

## Update 11/09/22: Expand disk
Default image comes with a disk size of 5GB which is not sufficient.

### Install Qemu On MacOS
> Assuming that you already have homebrew installed

qemu-img lies within the qemu package, so we'll need to install that. Open Terminal on your Mac.
Wait for the installation to finish.
```
brew install qemu
```

### Resizing the disk

1. Open UTM, right click your VM and select "Show in Finder"
2. Right click the .utm file and "Show Package Contents"
3. Open the "Images" folder, and now you'll see your imported and created disk images. Your hard drive should be named something like disk.qcow2.
4. Open Terminal on your Mac
5. Drag your disk.qcow2 into the terminal window, to paste its path. You can also manually enter the path if you prefer that.
6. Change your command, it  should look something like this:
   ```
   qemu-img resize /Users/user/Library/Containers/com.utmapp.UTM/Data/Documents/Home\ Assistant\ OS\.utm/Images/haos_generic-aarch64-9.0.rc2.qcow2 20G
   ```
7. Press Enter to execute the command
   It should say: ```Image resized.```
   
8. Now you should be able to start the VM, and the disk should be resized to your desired amount. Home Assistant OS will automatically use this new space and resizes the partition.

9. Go to Setting -> System -> Storage for validation

![](/assets/img/2022-09-11-17-10-53.png)

Quick tech note with the steps that worked for me. I am still testing and validating how stable this works. Good luck!