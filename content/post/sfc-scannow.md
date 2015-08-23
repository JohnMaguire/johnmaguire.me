---
title: "Forcing “sfc /scannow” in Windows 7 Startup Repair"
date: "2013-10-15"
---

I was attempting to repair a computer today, and after some updates were installed via Windows Update during shutdown, when the computer turned on I suddenly began receiving a BSOD (Blue Screen of Death) with STOP code 0xc000021a.

I could not find much information on the stop code, except that it probably meant something was wrong with winlogon.exe or csrss.exe.

I tried to use Startup Repair, as per Windows’ suggestion, but it was of little help. I decided to drop to the Command Prompt within the Startup Repair and attempt to run “sfc /scannow” to attempt to fix any corrupt system files. I was greeted with this ugly error message:

```
There is a system repair pending which requires reboot to complete. Restart Windows and run sfc again.
```

However, because Windows never got past the boot animation (stuck in a boot loop), the pending update was never finished. After some searching around, I found that these two files were the culrprits of the error message:

```
C:\Windows\WinSXS\pending.xml
C:\Windows\WinSXS\reboot.xml
```

There was little information about whether removing them was harmful or not. Some people said not to touch them or you could risk botching the entire install. Others said to use it as a last resort.

Rather than delete them, I decided to stay on the safe side and ran the following two commands from the Command Prompt (in Startup Repair, the Windows install typically residing at C: is mounted to D: instead):

```
move D:\Windows\WinSXS\pending.xml D:\
move D:\Windows\WinSXS\reboot.xml D:\
```

I once again tried running sfc, but was greeted with the same error message. I decided to try a reboot, was sent back to Startup Repair, but this time Startup Repair did it’s magic and Windows booted it up! The updates that were installed during shutdown also configured themselves properly despite pending.xml and reboot.xml being removed.

It is worth noting that I did run “sfc /scannow” from within Windows later, and did have corrupt files. I had to run it twice before they were all fixed.

Hope this helps someone!
