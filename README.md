# gigabyte-avx512-bios-mod
An instruction of how to modify bios image to enable avx512 for alderlake CPU on modern gigabyte motherboards

Assuming you have a compatible CPU (google "alderlake avx512" to see how you can spot one), here are the tools you need:
1. ***UPDATE***: If you have the Z690i AORUS Ultra Lite DDR4 mini-itx board, you can simply download the latest bios image, extract the latest *.Fxx bios image file and mod it directly, instead of having to first flash it, disable e-cores in bios, and back up bios image. As of this writing, version F22 is the latest. The good news is that unlike @RussianE39's experience, applying this mod to the bios of this board without first disabling e-cores does NOT prevent the computer to boot. This means after you apply this mod to stock bios image, simply go to bios settings and disable all e-cores, and AVX512 should be automatically detected! 
***OLD NOTE***: For those of you whose computer cannot boot up after you apply this modification directly to the stock bios image downloaded from Gigabyte, you will need a bios image of your gigabyte motherboard that has the damn "save bios" option in Q-Flash Utility, so that you can back up the bios image after you disable all e-cores and apply this mod to your backed up bios image. For my z690i a ultra lite d4, the latest bios with that feature is F2, located here: https://download.gigabyte.com/FileList/BIOS/mb_bios_z690i-a-ultra-lite-d4_f2.zip?v=52ad9b4e8a12b3d1709d0e520824ed96
Strangely, that "Save BIOS" option is completely disappeared in later versions (F20, F21, F22). Some redditors suggest that it's because the backed-up bios image generated with these later bios versions are corrupted, so Gigabyte developers decided to simply disable the bios backup feature completely (instead of spending time fixing it - I understand, those poor devs are nowadays got laid off like flies and bees, so they have their own reasoning. Or maybe they just forgot to enable it again? Who knows...). Once downloaded, you need to extract it to get the main bios binary file, usually with extension as the bios version. For example, mine is named "Z690IAULTRALITED4.F2"
2. MMtool to extract modules from bios image and to modify it, plus the microcode 15 patch. I followed this instruction: https://www.overclock.net/threads/12900k-patching-older-ucode-to-restore-avx512.1796070/
To download MMtool and the microcode 15 patch (m_03_90672_00000015.pdb).
3. HxD to edit a module file in hex format, download from here: https://mh-nexus.de/en/hxd/
4. A USB flash drive formatted in FAT32. I prefer to use a spare one I got for free from a technical conference.

Once you got all the tools above, it's time to do some bios flashing and hardcore bios image modification!

***UPDATE*** If your bios version does not have a "save bios" function, try to skip the following section and instead move directly to MMTool step and apply the modification to the extracted stock bios image (*.Fxx) directly. Most likely you will be lucky enough that your computer still boot up after applying this mod.

---Begin section disabling e-cores and bios backup---

First, flash the bios image you downloaded. You can either use the Q-flash utility in bios or the Q-flash Plus button method. Note that if you choose to use the Q-flash Plus button method, you need to rename the bios image file to exactly "gigabyte.bin" and make sure to format your USB as FAT32, then copy "gigabyte.bin" to the USB, plug it to the designated bios flashing port (usually marked with the word "BIOS" right above it, mine is a red USB port right next to the USB-C port). Then make sure the computer is shutdown, and push the Q-flash Plus button near the BIOS USB port. Then wait patiently while it's flashing. When it's done, it will reboot to the operating system. Some board may just shutdown, so YMMV. You can find more info about Q-flash Plus simply by googling it or youtube-search it.

Now that you have a bios with "save bios" function in Q-flash utility, go ahead and make whatever modification you need to your mainboard, including Overclocking, memory XMP, etc... Most importantly, you need to disable all E-cores (thanks @RussianE39). To disable all E-cores in my z690i board, I have to enable "Advanced Mode (F2)" (if you are currently in "Easy Mode", which should be by default), then under "Tweaker" tab, go to "Advanced CPU Settings" -> "CPU Cores Enabling Mode" -> "No. of CPU E-cores Enabled", change it to 0 (Sorry, I don't have any screenshots, as it's in BIOS. Will try to update with some phone camera photos later).

After you're done tweaking the bios to your liking, it's time to save it to an USB drive. Again, make sure to format the USB drive as FAT32, and plug it in to the board before turning your computer on. Go to Q-Flash Utility (F8), then choose the "Save Bios" option, which should be the second one from top down on the right panel. Next, give the output file name for the bios backup on the lower right text field. Let's say it's "f2_bak.bin". Start the backup. Once it's finish, you can reboot. Again, sorry for no screenshots.

---End section disabling e-cores and bios backup---


The next step is to mod the backed-up bios image (or if you're one of the lucky one, the latest stock bios image) with all the "puzzle pieces" from this thread. I'll try to credit the right person on winraid website for a specific "puzzle piece". If I miss anyone, feel free to alert me.

Run MMTool.exe. Click Load Image button, then change "File Type" to "All Files (*.*)" and browse to "f2_bak.bin" - the backed up bios image, or the stock image.
![alt text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_load.PNG)

After you load the image file, there should be a whole bunch of rows in the lower section. What you want to do now is to extract two modules in UNCOMPRESSED mode (thanks @pr0ton): SiInitFsp and GenericComponentPeiEntry (thanks @RussianE39). The module name is in the column "FileName" in the lower table. Browse and give the output module a name, and hit extract, once module at a time. I'll keep it simple: "s.module" for SiInitFsp and "g.module" for GenericComponentPeiEntry:
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_g_module.PNG)

![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_s_module.PNG)


Once you get the modules extracted, it's time to do the hardcore modification using hex editor HxD. Double-click on HxD64.exe after you install it, then open g.module (I'll try to follow the order in @RussianE39 post).
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/hxd_g_module.PNG)

Here is the juicy part: hit Ctrl+F, go to tab "Hex-value", search for pattern "83E0038365F4" (thanks @ptrang) in ALL direction (note: it's case-sensitive, but not space-sensitive: you can add spaces in between two-hex values, so "83 E0 03 83 65 F4" works):
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/hxd_g_module_search.PNG)

Then change the "03" to "00", so the resulting hex string should be "83 E0 00 83 65 F4", and SAVE IT:
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/hxd_g_module_modify.PNG)

We're done with g.module. Close it if you want.

Next, still in HxD, open s.module, and search for pattern "1A 24 01 8B E5" (again, make sure it's Hex-value tab, and search in all direction, case-sensitive!).
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/hxd_s_module_search.PNG)

Change "24 01" to "B0 00" (thanks @RussianE39, as well as @intruder16 and @Sweet_Kitten for bringing up and explaining what the hell it means):

![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/hxd_s_module_modify.PNG)

Save it and close HxD. We're done with hex editing.


Now, back to MMtool. We've modified the two modules, so it's time to replace them back into the bios image. To do so, (assuming you still have MMTool opened and loaded with the backed up bios image), go to "Replace" tab. Browse to "g.module" (the modified one, if you save it as a different file in HxD). Highlight the corresponding module to be replaced, which is GenericComponentPeiEntry. Then hit "Replace" button:
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_g_module_replace.PNG)

Now, still in "Replace" tab, browse to the modified s.module. Select component SiInitFsp, then click "Replace" button:
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_s_module_replace.PNG)

The last step is to deal with microcode 15 patch. Still in MMTool, go to "CPU Patch" tab. There should be several items on the lower table. Look at the column "Update Revision". That's the microcode patch number. First, you need to insert microcode 15 patch. To do so, browse to the m_03_90672_00000015.pdb. Then choose "Insert a patch data", and hit Apply. The microcode 15 patch should be added to the end of the lower table:

![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_insert_microcode_15.PNG)

Here is the important step that is missing in almost all other guides so far: you need to remove ALL microcode patches with revision higher than 15. In my case, I have to remove 17, 19, 1A, 1E, and 23. To do so, select the row corresponding to the higher-revision patch, then choose "Delete a patch data" and hit Apply button:
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_delete_higher_microcode_patches.PNG)

After you finish deleting all higher microcode patches, make sure microcode 15 is the highest revision of all remaining microcode patches:
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_microcode_table_result.PNG)


Don't forget to hit "Save image as..." button to save the modded bios image. Let's call it "f2_avx512.fd" (it won't allow any other extension).
![alt_text](https://github.com/thanghn90/gigabyte-avx512-bios-mod/blob/main/mmtool_save_image_as.PNG)


The last step is to copy and rename "f2_avx512.fd" to your USB stick as "gigabyte.bin", and use Q-flash Plus button method to flash that modded bios image to your board (thanks @RussianE39 and @pr0ton, I can confirm Q-flash utility won't work, and only flashing through the Q-flash Plus button works). Now, if you apply this mod on the stock bios image, after finish flashing the modded bios image, go back to bios and disable all e-cores (should be in Advanced Mode (F2), Tweaker tab, Advanced CPU settings, CPU E-cores Enabling Mode, set to 0). Once you do that, AVX512 should be automatically enabled.


Update: Thanks @RussianE39 for the hyperthreading clarification. In short: no need to disable hyperthreading for AVX512 to work. However, @RussianE39 said that he did not need to remove higher microcode CPU patches and instead simply enable AVX512 in bios after inserting the microcode-15 patch. My experience is different: AVX512 is consistently NOT detected until I removed ALL higher microcode CPU patches than 15. So YMMV. If you find the AVX 512 enabling option in your BIOS, you may not need to remove higher microcode CPU patches than microcode 15.


Now I need to figure out what byte/bit code in which module corresponding to the disabling of all E-cores. Figuring this out would allow me to modify the higher-version bios images that does not support bios backup. I'll tell you what I'm gonna do, and hopefully by the time I make a new post, some other brave hero would be faster than me and did it for me already (If not, I'll post it eventually). That is, I'll flash the original F2 bios image back to my board, then just make one single change that turn "No. of CPU E-cores enabled" to 0. Then save it. Then use MMTool to extract every single entry for the original and e-core-disabled images (as uncompressed), and compare them to see where they differ. It would take a while to do that since MMTool does not have a mechanism to extract all modules at once. My best bet is to look for entry with the word "CPU" in it first. But well, it's going to be tedious nonetheless.

Update on e-cores bios module: The closest thing I can say about this is that module at volume 03, index 03 (empty name) is the most likely to be the one that store the e-cores disabling/enabling toggle. However, it looks like it's also CPU-dependent, and when I try to extract that module from my modified F2 image and replace it to the latest F22 image, it has no effect. E-cores is still set at "Auto". And this is how I discovered that I don't have to disable e-cores before applying this mod to my board's bios image!!! The only other different module is at volume 00, index 00, but that's a huge one (256KB) and my guess is that it also store the current time, as each time I save bios image, it's different. The good news is that without disabling e-cores before applying this mod, my computer still boot up! So unless you're very unlucky to have no bios image with the "save bios" function AND your computer cannot boot up after applying this mod, you should be able to make it work.


Quick info about CPU temperature: originally in BIOS, CPU temp hovers around 42C doing nothing (this is with simple Noctua Nh-L9i-17xx air cooler). By disabling E-cores, it drops to 37C in bios (that's 5 degree difference, quite significant if you ask). And after I flash the modded bios image, CPU temp goes about 39C in bios (not much of a bump, reasonable for non-overclocking settings). If it's true that AVX512 benefits the emulators like the internet said, I think it's worth to do this mod.
