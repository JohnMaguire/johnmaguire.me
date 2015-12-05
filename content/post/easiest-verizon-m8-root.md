---
title: "Easiest Method for S-OFF and Rooting the Verizon HTC One M8 (Lollipop)"
date: "2015-11-25"
URL: "2015/11/easiest-verizon-m8-root/"
---

After a long and arduous battle with S-ON, I've rooted my Verizon HTC One M8. Finding every bit of information needed and compiling it took some time, so I'm documenting this for the next guy.

Please know that you will need $25 for this method to work. Unfortunately, it's the only working method right now.

I should also note that I did do a factory reset prior to following this method, though I don't believe it is necessary. I had to do it however because I managed to get KingRoot into a state where I could not reinstall it.

1. Check your firmware version. If you are on OS 4.17.605.9 you must downgrade to OS 4.17.605.5 first. You may be able to get away without this step, but chances are it will cause KingRoot to make your system unstable if you don't (I eventually downgraded mine.) You can find instructions on how to downgrade [here](http://forum.xda-developers.com/verizon-htc-one-m8/general/downgrade-s-sunshine-purposes-t3202237). I was only able to get method three to work (download the version of fastboot linked &mdash; this is important) and flash the RUU provided in the thread.
2. Download [KingRoot](http://kingroot.net). It is a semi-sketchy application that requests every single permission possible (supposedly because some methods of root require each one.) It also installs an app or two without your permission after achieving root. However, it's the only working method of gaining root on the Verizon HTC One M8 without S-OFF.
3. Download [SunShine](http://theroot.ninja). Using it will cost you $25, but it is the only way to disable S-ON, the bootloader protection Verizon hides behind to prevent you from flashing custom ROMs. It requires root, which is why we need KingRoot first.
4. Run KingRoot. It will probably cause your phone to reboot at least once. When it does, simply run it again. You should gain root relatively quickly. Some guides online suggest you need to install some version of SuperSU at this point. You don't, and it will only cause you pain.
5. Run SunShine. It will test that your device can have S-ON disabled. If it can, it will ask you to pay $25. Pay it. Your phone will probably reboot again. Open SunShine and finish the process. It will remove itself when it has finished.

Congrats! You have root through KingRoot and you have disabled S-ON. At this point, you can flash whatever you want. I chose to grab the 5.1 GPE ROM available for my phone, and install a SuperSU binary zip. Have fun!
