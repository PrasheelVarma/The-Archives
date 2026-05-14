---
layout: default
title: The OS Migration
permalink: /infra/migration_to_os/
---

# ☁️ The OS Migration: A Side Experimentation That Revealed Some Truths

> "The hardware is never a problem; it is about what OS you are."
> This is a technical log of transitioning from Windows to an Arch based Linux environment. It documents the backup protocols, the bootloader conflicts, and the realization of how deeply commercial operating systems hide the reality of our hardware.

## Contents
1. The Operating System Reality Check
2. The Backup Protocol
3. The Bootloader Rebellion
4. The Ventoy Bypass
5. Establishing the Storage Architecture

<br>

## 1. The Operating System Reality Check
For a long time, when the system felt heavy or a process lagged, the immediate assumption was that the hardware had reached its limit. My Lenovo V15 ADA laptop with an AMD Athlon Silver 3050U processor and 12GB of RAM felt like a bottleneck under Windows 11.

The initial plan was simply to install a Linux distribution alongside Windows as a side experiment to see if resource usage could be optimized. But booting into an Arch based system (EndeavourOS) felt like stepping out of a simulation. 

Commercial OSes like Windows and macOS are designed to hide the underlying machinery from the user. They manage processes behind closed visual interfaces so the user does not have to think. Linux does the exact opposite. It exposes the reality of how resource allocation, window managers, and compositors actually function. It forces you to experiment, break things, and ultimately understand the machine. 

This side experiment quickly revealed a core truth. The hardware was never the problem. The heavy Windows OS was.

## 2. The Backup Protocol
Before I could even think about installing Linux, I needed to back up all my important data from Windows. Standard drag and drop to Google Drive is notoriously unreliable for massive file transfers, requiring a more robust tool called Rclone.

Rclone acts as a command line interface for cloud storage. Running the transfer through the terminal removed the guesswork of visual loading bars. The CLI provided real time metrics:
* Live transfer speeds
* Exact number of files transferred versus pending
* The status of concurrent active streams

This ensured no hidden files or project folders were dropped during the migration, providing a clean state to begin wiping partitions.

## 3. The Bootloader Rebellion
Once the data was safe in the cloud, the goal was a standard Windows "Reset this PC" to clear out bloatware before officially setting up the dual boot. However, the system refused to reset.

The Blocker: A previous Linux installation had already tinkered with the drive. Because Linux had taken over the UEFI and written its own GRUB bootloader, the built in Windows Recovery Environment was essentially hijacked. 

Windows could not verify its own boot path. Every attempt to trigger the internal reset tool failed entirely. The OS was trapped in its own corrupted state, proving how fragile closed recovery systems can be when forced to interact with outside bootloaders.

## 4. The Ventoy Bypass
To bypass the broken recovery environment, the installation had to be approached from outside the OS. The solution was Ventoy.

Instead of repeatedly formatting USB drives with Rufus every time a new OS was tested, Ventoy allows formatting the USB once and simply dragging and dropping ISO files onto it. 
* The Ventoy USB was loaded with a fresh Windows 11 ISO.
* The system booted directly into the USB, completely bypassing the corrupted GRUB bootloader loop on the hard drive.
* A completely fresh Windows install was executed directly over the old partition. 

This wiped the corrupted bootloader and established a clean and stable base.

## 5. Establishing the Storage Architecture
With the foundation fixed and Windows freshly installed, EndeavourOS was finally installed alongside it. Inside Linux, strict partition rules were established to govern the machine moving forward. 

To maintain system speed and prevent the OS from slowing down over time, the storage is heavily segmented:
* The Main OS Partition (80GB): Strictly reserved for the EndeavourOS root system, critical libraries, and a dedicated 16GB Swap file (Virtual RAM) to handle intensive computing workloads.
* The Secondary Partition: All heavy data is routed to a separate drive. This includes project repositories, Python environments, and large game installations.

By keeping the root partition small and lean (80GB), the Linux system is forced to remain efficient, while the secondary partition handles the heavy lifting. The side experiment had officially become the core infrastructure.