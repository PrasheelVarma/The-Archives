---
layout: default
title: The Windows Debloat
permalink: /workshop/windows_debloat/
---

# 🧪 The Windows Debloat: Stripping the OS for Dual Boot Survival

> A fresh installation of Windows is not a clean installation. It is a telemetry heavy environment that silently consumes hardware resources. This guide covers the advanced, system level methods used to strip Windows 11 down to its bare minimum so it does not interfere with the primary Linux setup.

## Contents
1. [The Custom ISO Trap (Why Not AtlasOS?)](#1-the-custom-iso-trap-why-not-atlasos)
2. [Advanced Telemetry Annihilation](#2-advanced-telemetry-annihilation)
3. [The Dual Boot Imperative: Fast Startup](#3-the-dual-boot-imperative-fast-startup)
4. [The Reality Check: Limitations](#4-the-reality-check-limitations)

<br>

## 1. The Custom ISO Trap (Why Not AtlasOS?)
When searching for ways to debloat Windows, YouTube and tech forums immediately push custom operating systems like AtlasOS, Ghost Spectre, or Tiny11. 

I do not suggest using these. 

While they look incredibly lightweight on paper, custom ISOs are a trap for dual boot systems. They achieve their low RAM usage by forcefully ripping out core Windows dependencies. This creates a cascade of hidden problems:
* Broken academic software dependencies.
* Missing security certificates.
* Completely broken Windows Update systems.

If Windows is reduced to a secondary OS for specific tasks, it still needs to be stable. Relying on an anonymous third party to modify the core system files is a security and stability risk. We can achieve 90% of the performance gains of AtlasOS on an official Windows image simply by taking control of the system policies ourselves.

## 2. Advanced Telemetry Annihilation
Basic debloating guides tell you to uninstall Spotify and turn off background apps in the settings menu. That does nothing to stop the core Windows services from running. Real debloating happens in PowerShell and the Registry.

To strip the system safely but aggressively, we bypass the settings menu entirely and run a consolidated PowerShell utility. 

Run PowerShell as Administrator and execute:

    iwr -useb [https://christitus.com/win](https://christitus.com/win) | iex

This summons a GUI wrapper for advanced system policies. The critical modifications to make here are:
* **Remove all Appx Packages:** This violently uninstalls every piece of pre packaged bloatware that cannot be removed via normal settings.
* **Disable Telemetry:** Kills the background processes constantly phoning home to Microsoft servers.
* **Disable Web Search:** Stops the start menu from pinging Bing every time you search for a local file, saving massive amounts of CPU cycles on the Athlon processor.
* **Set Services to Manual:** Prevents update services and background intelligence from running until explicitly triggered.

## 3. The Dual Boot Imperative: Fast Startup
This is the most critical step for anyone running a Linux dual boot alongside Windows. 

By default, when you tell Windows to "Shut Down," it does not actually shut down. It logs you out and hibernates the kernel to the hard drive to make the next boot faster. This is called **Fast Startup**.

If you leave Fast Startup enabled, Windows locks the NTFS partitions. When you boot into your Linux environment and try to access your secondary drives, Linux will refuse to mount them because Windows flagged them as "in use."

To permanently kill Fast Startup, open the Command Prompt as Administrator and run:

    powercfg -h off

This single command disables hibernation, deletes the massive `hiberfil.sys` file from your drive, and forces Windows to fully release the hardware when it powers down.

## 4. The Reality Check: Limitations
There is a limit to how much you can bend a commercial OS to your will. 

After running the PowerShell utilities and disabling hibernation, the background process count will drop significantly. The system will feel faster, and the idle RAM usage will be manageable. 

However, Windows will never be Arch Linux. 

Even fully debloated, Windows relies on a massive interconnected web of background services (svchost.exe) just to keep the desktop environment stable. Microsoft will also frequently use security updates as an excuse to stealthily reinstall deleted services or revert registry changes. 

The reality is that debloating Windows is an ongoing battle of maintenance, not a permanent fix. It makes the OS usable as a secondary tool, but it reinforces exactly why transitioning the core architecture to Linux was the correct choice.
