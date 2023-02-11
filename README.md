# Z170X-UD3 Audio Workaround

## Introduction

I recently wanted to use the audio jack on the back IO of my motherboard and discovered that it wasn't functional on an Arch Linux install with Pipewire. The following works around one of the issues I encountered during this (mis)adventure.

## Description

Workaround for the requirement on some motherboards that headphone volume must be at `100%` instead of `0%` in the ALSA mixer path configuration used by PulseAudio and PipeWire.

I know this impacts the GA-Z170X-UD3 from Gigabyte. YMMV on other boards.

```
$ cat /proc/asound/card0/codec\#0 | grep -E 'Codec|Vendor Id|Subsystem Id|Address' 
Codec: Realtek ALC1220
Address: 0
Vendor Id: 0x10ec1220
Subsystem Id: 0x1458a0c1

$ lspci -nn | grep -i audio
00:1f.3 Audio device [0403]: Intel Corporation 200 Series PCH HD Audio [8086:a2f0]
```

## Install

Merge the etc and usr directory trees into the root filesystem. If using another motherboard with the same issue, make sure to modify the `etc/udev/rules.d/91-pipewire-alsa.rules` file to contain the correct PCI vendor and device IDs for your motherboard. If your motherboard exposes the speaker jack as "speaker" instead of "lineout" then you will also need to copy the correct mixer path from upstream and change it as shown in the "Why this works" section below.

On my Arch install, the 90-pipewire-alsa.rules file is not copied to initcpio/initramfs but depending on your configuration that may differ. I suggest regenerating your initrd as a precaution.

## Why this works

The below diff is the actual fix. The udev rule and profile-set are just to maintain persistance when the upstream files are updated by your distro (https://archlinux.org/packages/extra/x86_64/alsa-card-profiles/ on Arch).

```diff
--- analog-output-lineout.conf  2023-02-01 00:43:09.000000000 -0800
+++ analog-output-lineout-gbworkaround.conf     2023-02-10 22:00:35.091669684 -0800
@@ -125,7 +125,7 @@
 ; else there will be a spike when plugging in headphones
 [Element Headphone]
 switch = off
-volume = off
+volume = merge
 
 [Element Headphone,1]
 switch = off
```

## Resources

- Tutorial on how to get multistreaming/splitting working on other boards: https://github.com/luisbocanegra/linux-guide-split-audio-ports
- Reddit thread pointing out how to get `/usr/share/alsa-card-profile` persistance working: https://www.reddit.com/r/archlinux/comments/qt70qq/alsacardprofiles_permanent_config_file_changes/

