---
title: "PSA: Avoid WD NVMe SSDs due to bad ASPM"
description: "Avoid WD NVMe SSDs if you want stable working PCIe ASPM. And no I/O errors."
date: "2024-10-15T22:33:02+08:00"
lastmod: "2024-10-15T22:33:02+08:00"
slug: "2024/avoid-wd-nvme-aspm"
toc: true
categories: [storage, hardware, disaster-recovery]
---
# PSA: **Avoid Western Digital (WD) NVMe SSDs** if you want stable working PCIe ASPM. And no I/O errors.

> Note: This is merely my experience with WD NVMe SSDs, and is not a fully objective or scientific post. You may (and most likely do) know better than me on this topic. Whatever the case is, please keep it civil and respectful if you would like to let me (and/or others) know more on this topic, for educational purposes. I would love to learn more about this. But I'm done with buying any more WD NVMe SSDs, and that's final.

## First strike: WD Blue SN550 1TB for 2 years, on Blackhawk, my laptop.

Earlier this year (2024), I had a 1TB SN550 in my laptop whose controller became very unstable at 98TB writes (it's supposed to take at least 4x that amount of writes, if not way more) and only by disabling PCIe ASPM both in UEFI and in Linux boot arguments could I have I/O operations working for more than 100MB without crashing. This SSD had been used for 2 years, maybe a little longer. Just a little.

I came up with the idea of disabling PCIe ASPM after looking at dmesg and noticing that I could sometimes get as far as login and launching SwayWM and i3status-rs with my config intact before I/O error would occur, indicating that the flash was probably fine and it was probably a controller error (after all, it was showing `controller reset` in dmesg), and in my head thinking that one of the factors that would cause the controller to change its state and thus go from working fine in boot process to crashing after boot finished, is either power or thermal related.

And that's how my solution came down to disabling PCIe ASPM (and all PCIe and drive power management options I could find), and pointing a Noctua fan directly at the NVMe drive.

To recover my data, I moved the NVMe to a different machine (we'll call it recovery rig), did the above mitigations on the recovery rig, and had to use `ddrescue` with its map file to extract my data out in cycles (boot into live ISO with `ddrescue` script in USB, start ddrescue reading from SSD, controller crash, shut down and drain power fully (every last light on the motherboard must be off), boot into live ISO, repeat). 

The laptop is a 2020 Lenovo ThinkPad T14 Gen 1 AMD. The recovery rig is a custom gaming rig, with an ASUS MAXIMUS VIII HERO motherboard.

Also, because I bought 4x 1TB SN550 drives from Amazon US during an Amazon SG sale in 2021, when I went to enquire about warranty coverage for this failure in Feb 2024, I was told that it was ineligible for any warranty as the drive came from US and I was based in SG.

## Second strike: WD Black SN770 2TB for <1 week, on Nighthawk, my gaming desktop.

Last month, I bought a 2TB SN770 for use in my gaming rig, thinking "well, it's only a gaming rig, with mainly games on it (due to SMB Folder Redirection storing my user folders on my NAS), surely, going with WD again due to it being the cheapest 2TB on Amazon SG at the time, that's no big deal right? I don't mind replacing 3 years down the road once it wears!". Oh, how naive last month me was.

I `dd` transferred my data from a different 1TB SN550 (the one from the gaming rig) to the 2TB SN770. Less than a week into using the SN770, I run a `chkdsk` on Windows (as GParted partition move needed it), and I start getting BSoDs on 9/10 boot attempts. Even after booting, it was 50/50 whether it will crash in a few hours or not. 

I was unsure if the `chkdsk` had screwed my data in any way, and wanted to continue gaming, and honestly wanted to take the easy way out using my 30 day Amazon return, so I swapped back to the previous 1TB SN550 for my gaming rig, put the SN770 in the recovery rig, this time to wipe its data for the Amazon return as the old gaming rig SN550 was still working fine (*knocks on wood*), and lo and behold: only the UEFI could detect it.

Using an EndeavourOS live ISO, its boot process included scanning all disks for LVM volumes to activate, and consistently on the first boot since the machine was previously fully power drained (power switch off *and* all lights off), the LVM activation would cause the controller to I/O error but /dev/nvme0 was still detected and populated in the filesystem, and then subsequent reboots without full power drain would cause the SSD to not be detected entirely after UEFI boot menu.

I was insistent in wiping my data before the return, so I thought about anything to make the controller stable, and I recalled my previous SN550 experience where I had stumbled on the idea of disabling PCIe ASPM and doing so had stabilized the controller. Thus, despite the lack of `controller reset` messages in dmesg (instead going straight to `Input/output error`) and the fact that this was a brand new SSD, I disabled PCIe ASPM in the UEFI and Linux boot args *again*, and what do you know: I can now `fdisk -l /dev/nvme0n1` and `dd if=/dev/urandom of=/dev/nvme0n1 bs=1M oflag=direct status=progress`.

I can finally return the drive in peace. Amazon is waiting.

The gaming rig is a custom gaming rig, with an MSI B550-A Pro that supports PCIe 4.0 NVMe SSDs. The recovery rig supports PCIe 3.0 so the NVMe drive was running at 3.0x4.

> *Side note*: In my head, I had a theory: in the recovery rig the SN770 would run cooler as the speed is limited, as I had thought the issue was PCIe 4.0 causing thermal throttling on the SSD. The heat source that would cause the throttle in my theory would be from the RTX 2060 and the CPU heatsink being basically "above" the NVMe slot on the B550-A Pro, but despite the much better NVMe slot positioning on the MAXIMUS rig and the lack of GPU, alas it was not the case.

## No third strike

No more Amazon sales shall tempt me to getting another WD NVMe SSD. I want my PCIe ASPM so I can avoid paying for unnecessary electricity, and WD to me now has a bad track record for PCIe ASPM support. Especially when a failure occurs not even a week into using the drive. I'm done.

For my laptop, I got a 2TB SK Hynix P31 Gold from Amazon SG with no offers (as I had to replace the drive ASAP to continue my personal endeavours), and for the gaming rig, the recent (Oct 2024) Prime Day sales had the 2TB Samsung 990 EVO for even cheaper than what I got the WD SN770 for, so I got 2 of those, 1 as a cold spare.

I still have 2x WD SN550 drives in use today on my R730xd server, and once those give out (hopefully not soon), it won't be replaced with another WD NVMe SSD, though I'm not sure what it'll be yet or even what size as I plan to restructure that part of my homelab (or more realistically, home-prod).

> When I wrote this, I had just finished the `dd if=/dev/urandom of=/dev/nvme0n1` on the SN770, it's now a few days later and my money has been refunded after returning the SN770 to Amazon, so I'm now publishing this post.
