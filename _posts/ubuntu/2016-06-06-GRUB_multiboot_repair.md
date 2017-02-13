---
layout: post
title: 'GRUB multiboot repair(멀티부팅 재설치)'
tags: [ubuntu]
description: >
  GRUB multiboot repair
---
멀티부팅을 위한 Rrub 재설치 방법

To Reinstall Grub
You need to have Ubuntu Live CD or Live USB. Normal session can be used to repair the grub. Boot using your Ubuntu Live CD or Live USB, while booting choose Try Ubuntu.
Once booted then open a terminal, and run the following command one by one to install the boot repair.
To add boot-repair to the repository

```
sudo add-apt-repository ppa:yannubuntu/boot-repair
```

To Update your repository

```
sudo apt-get update
```

To install boot-repair

```
sudo apt-get install -y boot-repair
```

Once Installation complete run boot-repair on terminal by typing the following command or select it by System->Aministration->Boot Repair.

```
boot-repair
```

NOTE: Update the Boot Repair if its newer version is available.
It will scan the System for few seconds and will show you the options Recommended repair and Create a BootInfo summary. By clicking the Recommended Repair it will start repair the grub. Check the screen shots below.

<img align="left" src="/images/post/ubuntu/boot_repair/1.png" width="800">

after waiting for second..

<img align="left" src="/images/post/ubuntu/boot_repair/2.png" width="800">

Once done click ok and restart your system, your grub should work now. If not run the boot-repair again using live cd / usb. Then follow the steps below.
Select the Advanced options, In Main options tab check whether the following options are selected or not. If not select it, the options are Reinstall Grub and unhide boot menu for 10 seconds. Check the screen shot below

<img align="left" src="/images/post/ubuntu/boot_repair/3.png" width="800">

Then select the GRUB locations tab and check the following options are selected or not. The options are OS to boot by default and place grub into, In “OS to boot by default” option choose the OS which you want to be default on boot. Then select the drive where you need to reinstall the grub in “place grub into” option and click apply. Check the screen shots below

<img align="left" src="/images/post/ubuntu/boot_repair/4.png" width="800">

and then 

<img align="left" src="/images/post/ubuntu/boot_repair/5.png" width="800">

waiting for a while

<img align="left" src="/images/post/ubuntu/boot_repair/6.png" width="800">


Click ok and restart your System. To restore MBR Click Here.
Hope this will be helpful for you!!!

