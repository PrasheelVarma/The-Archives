---
layout: default
title: The 3-Day Infrastructure Trial (GTA V)
permalink: /infra/gta5_infra/
---

# 🏗️ The 3-Day Infrastructure Trial: GTA V on a Radeon APU

> **Document Type: Technical Case Study**
> This document archives the architectural struggle, hardware bottlenecks, and ultimate resolution of running a heavy Windows title on a constrained Radeon APU laptop running EndeavourOS. It is a study in absolute system control and resource optimization.

## Contents
1. [The Catalyst: Testing the Limits of EndeavourOS](#1-the-catalyst-testing-the-limits-of-endeavouros)
2. [The Lutris Bottleneck](#2-the-lutris-bottleneck)
3. [The Steam Pivot & Engine Roulette](#3-the-steam-pivot--engine-roulette)
4. [The Crash Pipeline: Windows vs. Linux Rendering](#4-the-crash-pipeline-windows-vs-linux-rendering)
5. [The Architecture of the Fix: NVMe Swap](#5-the-architecture-of-the-fix-nvme-swap)
6. [The Core Question: Why is Linux Smoother?](#6-the-core-question-why-is-linux-smoother)

<br>

## 1. The Catalyst: Testing the Limits of EndeavourOS

Once you establish a comfortable, highly customized EndeavourOS workflow, the natural next step for an engineer is a stress test. Can a lightweight, terminal-centric Linux distribution actually run a massive, resource-heavy Windows game like Grand Theft Auto V? 

If the answer is yes, it proves the inherent power and flexibility of the Linux kernel. It ceases to be just an operating system and becomes a modular engine.

## 2. The Lutris Bottleneck

The initial approach utilized Lutris. As a standalone manager, Lutris is mathematically sound for users with high-end, modern hardware who want a clean separation from corporate storefronts. 

However, on a constrained laptop footprint (a Radeon APU where the processor is the bottleneck), Lutris requires too much manual intervention. The overhead of hunting for specific Wine dependencies, manually tweaking DXVK versions, and managing the prefix environment choked the limited system resources. After extensive troubleshooting, it became clear that Lutris was the wrong tool for this specific hardware tier.

## 3. The Steam Pivot & Engine Roulette

The pivot involved adding GTA V as a "Non-Steam Game" to leverage Valve's automated backend. This is where the realization hit: **performance drops were not a graphics issue, but an engine translation issue.** Lowering the in-game graphics to minimum did not stop the stuttering. Switching the game’s internal API from DirectX 11 down to DirectX 10 yielded better stability, but the underlying engine conflict remained. 

Finding the correct translation layer required methodical testing:
* **Proton Experimental:** Unstable for this specific hardware combination.
* **Proton Hotfix:** Ineffective.
* **The Breakthrough:** Forcing the compatibility tool to **Proton 9.0** combined with the **Steam Linux Runtime 3.0 (sniper)**. This specific configuration bridged the gap between the Radeon APU and the game's DirectX calls, allowing the game to successfully boot with higher graphics fidelity than it ever could on Windows.

## 4. The Crash Pipeline: Windows vs. Linux Rendering

Observing the game run on Linux revealed a massive difference in how the two operating systems handle rendering failures and memory bottlenecks.

* **The Windows Behavior (Input Lag & Pauses):** When Windows runs out of resources rendering roads or buildings, the game violently stutters, pauses, and introduces heavy input lag. If you wait, the OS eventually forces the textures to load through brute force.
* **The Linux Behavior (Smoothness to Hard Crash):** On Linux, the game actually felt smoother with less input lag. It could handle heavier graphics effortlessly. However, if a severe stutter occurred (usually due to a sudden lack of memory when loading a new city zone), Linux wouldn't pause—it would simply crash the application a minute later to protect the system.

## 5. The Architecture of the Fix: NVMe Swap

The hard crashes on Linux were a direct result of the specific hardware memory split. The laptop has 12GB of physical RAM, but **2GB is hardware-reserved as dedicated VRAM for the Radeon GPU**. This leaves only 10GB of usable system memory for the OS and the game engine.

Once that 10GB of RAM hit 100% capacity during high-speed rendering, the Linux kernel's Out-Of-Memory (OOM) killer terminated the game instantly.

**The Solution:** Creating a massive Virtual Memory Swap file on the high-speed NVMe SSD. 

By allocating NVMe storage to act as emergency RAM, the system finally had a safety net. When the 10GB of physical RAM filled up, the overflow data spilled safely into the NVMe swap space instead of triggering a fatal engine crash. 

*(Note: Shader caching was also attempted via launch options, but monitoring the designated cache folder showed little to no activity, rendering its actual impact inconclusive on this specific setup).*

## 6. The Core Question: Why is Linux Smoother?

How is it possible that a laptop running a non-native game through a translation layer (DirectX to Vulkan) experiences *less* input lag and handles higher graphics better than native Windows?

The answer lies in absolute OS optimization and the elimination of background noise.

1. **The Idle Footprint:** Windows 11 idles at roughly 4GB to 6GB of RAM. EndeavourOS idles at around 800MB. On a machine with only 10GB of usable system memory, Linux instantly gives the game almost 4GB of extra physical memory that Windows was hoarding for itself.
2. **Unkillable Corporate Bloat:** In Windows, you cannot permanently kill the Antimalware Service Executable, the Desktop Window Manager (which forces Vsync and adds input lag), or background telemetry. They constantly steal CPU cycles from the APU.
3. **The Translation Advantage:** DXVK translates old DirectX 11 calls into modern Vulkan. Vulkan is a low-overhead, highly parallel API. In many cases, translating DX11 to Vulkan is actually faster and more efficient on the Radeon GPU than letting Windows process the native, aging DX11 code.

In Linux, you own the hardware. When you launch the game, 100% of the APU's attention is focused on rendering the next frame, completely untouched by corporate background processes.
