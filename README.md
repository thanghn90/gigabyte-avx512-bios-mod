# gigabyte-avx512-bios-mod
An instruction of how to modify bios image to enable avx512 for alderlake CPU on modern gigabyte motherboards

Assuming you have a compatible CPU (google "alderlake avx512" to see how you can spot one), here are the tools you need:
1. A bios image of your gigabyte motherboard that has the damn "save bios" option in Q-Flash Utility. For my z690i a ultra lite d4, the latest bios with that feature is F2, located here: https://download.gigabyte.com/FileList/BIOS/mb_bios_z690i-a-ultra-lite-d4_f2.zip?v=52ad9b4e8a12b3d1709d0e520824ed96
Strangely, that "Save BIOS" option is completely disappeared in later versions (F20, F21, F22). Some redditors suggest that it's because the backed-up bios image generated with these later bios versions are corrupted, so Gigabyte developers decided to simply disable the bios backup feature completely (instead of spending time fixing it - I understand, those poor devs are nowadays got laid off like flies and bees, so they have their own reasoning). Once downloaded, you need to extract it to get the main bios binary file, usually with extension as the bios version. For example, mine is named "Z690IAULTRALITED4.F2"
2. MMtool to extract modules from bios image and to modify it, plus the microcode 15 patch. I followed this instruction: https://www.overclock.net/threads/12900k-patching-older-ucode-to-restore-avx512.1796070/
To download MMtool and the microcode 15 patch (m_03_90672_00000015.pdb).
3. HxD to edit a module file in hex format, download from here: https://mh-nexus.de/en/hxd/
4. A USB flash drive formatted in FAT32. I prefer to use a spare one I got for free from a technical conference.

Once you got all the tools above, it's time to do some bios flashing and hardcore bios image modification!

First, flash the bios image you downloaded. You can either use the Q-flash utility in bios or the Q-flash Plus button method. Note that if you choose to use the Q-flash Plus button method, you need to rename the bios image file to exactly "gigabyte.bin" and make sure to format your USB as FAT32, then copy "gigabyte.bin" to the USB, plug it to the designated bios flashing port (usually marked with the word "BIOS" right above it, mine is a red USB port right next to the USB-C port). Then make sure the computer is shutdown, and push the Q-flash Plus button near the BIOS USB port. Then wait patiently while it's flashing. When it's done, it will reboot to the operating system. Some board may just shutdown, so YMMV. You can find more info about Q-flash Plus simply by googling it or youtube-search it.

Now that you have a bios with "save bios" function in Q-flash utility, go ahead and make whatever modification you need to your mainboard, including Overclocking, memory XMP, etc... Most importantly, you need to disable all E-cores (thanks @RussianE39). To disable all E-cores in my z690i board, I have to enable "Advanced Mode (F2)" (if you are currently in "Easy Mode", which should be by default), then under "Tweaker" tab, go to "Advanced CPU Settings" -> "CPU Cores Enabling Mode" -> "No. of CPU E-cores Enabled", change it to 0 (Sorry, I don't have any screenshots, as it's in BIOS. Will try to update with some phone camera photos later).

After you're done tweaking the bios to your liking, it's time to save it to an USB drive. Again, make sure to format the USB drive as FAT32, and plug it in to the board before turning your computer on. Go to Q-Flash Utility (F8), then choose the "Save Bios" option, which should be the second one from top down on the right panel. Next, give the output file name for the bios backup on the lower right text field. Let's say it's "f2_bak.bin". Start the backup. Once it's finish, you can reboot. Again, sorry for no screenshots.

The next step is to mod the backed-up bios image with all the "puzzle pieces" from this thread. I'll try to credit the right person for a specific "puzzle piece". If I miss anyone, feel free to alert me.
Run MMTool.exe. Click Load Image button, then change "File Type" to "All Files (*.*)" and browse to "f2_bak.bin", the backed up bios image.
![mmtool_load|690x411](upload://iJ8RxNBhQjeIdbtL2AqAafC21qR.png)
After you load the image file, there should be a whole bunch of rows in the lower section. What you want to do now is to extract two modules in UNCOMPRESSED mode (thanks @pr0ton): SiInitFsp and GenericComponentPeiEntry (thanks @RussianE39). The module name is in the column "FileName" in the lower table. Browse and give the output module a name, and hit extract, once module at a time. I'll keep it simple: "s.module" for SiInitFsp and "g.module" for GenericComponentPeiEntry:
![mmtool_s_module|690x361](upload://X9jYgkwskJytsInNDng7LfJ7QS.png)
![mmtool_g_module|690x391](upload://y0H5BpqQD0vFwoRp0SbZ6pXnDnr.png)

Once you get the modules extracted, it's time to do the hardcore modification using hex editor HxD. Double-click on HxD64.exe after you install it, then open g.module (I'll try to follow the order in @RussianE39 post).
![hxd_g_module|690x284](upload://dcMzVoMRnGUOTWEJDuVowZqUtEw.png)
Here is the juicy part: hit Ctrl+F, go to tab "Hex-value", search for pattern "83E0038365F4" (thanks @ptrang) in ALL direction (note: it's case-sensitive, but not space-sensitive: you can add spaces in between two-hex values, so "83 E0 03 83 65 F4" works):
![hxd_g_module_search|690x308](upload://uSACjzjLGivMa5N9ZfQ8BCAKum.png)
Then change the "03" to "00", so the resulting hex string should be "83 E0 00 83 65 F4", and SAVE IT:
![hxd_g_module_modify|690x337](upload://tloqjHIbBY2XG3LXbe5cXyoMQnD.png)
We're done with g.module. Close it if you want.

Next, still in HxD, open s.module, and search for pattern "1A 24 01 8B E5" (again, make sure it's Hex-value tab, and search in all direction, case-sensitive!).
![hxd_s_module_search|690x304](upload://1DuujdpmSbDXMfj7tKpXjMIxlp5.png)
Change "24 01" to "B0 00" (thanks @RussianE39, as well as @intruder16 and @Sweet_Kitten for bringing up and explaining what the hell it means):
![hxd_s_module_modify|408x253](upload://lpK6Rw7vLbuQVe2lnYPwlKT9lv3.png)
Save it and close HxD. We're done with hex editing.

Now, back to MMtool. We've modified the two modules, so it's time to replace them back into the bios image. To do so, (assuming you still have MMTool opened and loaded with the backed up bios image), go to "Replace" tab. Browse to "g.module" (the modified one, if you save it as a different file in HxD). Highlight the corresponding module to be replaced, which is GenericComponentPeiEntry. Then hit "Replace" button:
![mmtool_g_module_replace|690x375](upload://qJr4hhSvCvXlEPdQyMZvLQnaYPK.png)
Now, still in "Replace" tab, browse to the modified s.module. Select component SiInitFsp, then click "Replace" button:
![mmtool_s_module_replace|690x395](upload://xe4Ku4rRb0XbKtQmA1QzPtX78gO.png)

The last step is to deal with microcode 15 patch. Still in MMTool, go to "CPU Patch" tab. There should be several items on the lower table. Look at the column "Update Revision". That's the microcode patch number. First, you need to insert microcode 15 patch. To do so, browse to the m_03_90672_00000015.pdb. Then choose "Insert a patch data", and hit Apply. The microcode 15 patch should be added to the end of the lower table:
![mmtool_insert_microcode_15|690x424](upload://AdQqcJz09XNn3DZLWyCdPOYBPwL.png)
Here is the important step that is missing in almost all guide so far: you need to remove ALL microcode patches with revision higher than 15. In my case, I have to remove 17, 19, 1A, 1E, and 23. To do so, select the row corresponding to the higher-revision patch, then choose "Delete a patch data" and hit Apply button:
![mmtool_delete_higher_microcode_patches|690x389](upload://m2XEcqEGIuXhRuiqAtOXtq7EtZ5.png)
After you finish deleting all higher microcode patches, make sure microcode 15 is the highest revision of all remaining microcode patches:
![mmtool_microcode_table_result|690x96](upload://m5540PZu1VnFbsV6yF1tycCOwS1.png)

Don't forget to hit "Save image as..." button to save the modded bios image. Let's call it "f2_avx512.fd" (it won't allow any other extension).

The last step is to copy and rename "f2_avx512.fd" to your USB stick as "gigabyte.bin", and use Q-flash Plus button method to flash that modded bios image to your board (thanks @RussianE39 and @pr0ton, I can confirm Q-flash utility won't work, and only flashing through the Q-flash Plus button works).

Now, two things remain to be discovered. First, I've read somewhere that in order for AVX512 to work, hyperthreading must be disabled. It's quite strange that HWinfo reported AVX512 enabled in my CPU while hyperthreading is also enabled. This should be a simple check: I'll test RPCS3 running my favorite game God of War 3 with and without hyperthreading to see which one works better. If there is no difference, let's keep hyperthreading on (should be on (auto) by default).
The second is more challenging: I need to figure out what byte/bit code in which module corresponding to the disabling of all E-cores. Figuring this out would allow me to modify the higher-version bios images that does not support bios backup. I'll tell you what I'm gonna do, and hopefully by the time I make a new post, some other brave hero would be faster than me and did it for me already (If not, I'll post it eventually). That is, I'll flash the original F2 bios image back to my board, then just make one single change that turn "No. of CPU E-cores enabled" to 0. Then save it. Then use MMTool to extract every single entry for the original and e-core-disabled images (as uncompressed), and compare them to see where they differ. It would take a while to do that since MMTool does not have a mechanism to extract all modules at once. My best bet is to look for entry with the word "CPU" in it first. But well, it's going to be tedious nonetheless.

Quick info about CPU temperature: originally in BIOS, CPU temp hovers around 42C doing nothing (this is with simple Noctua Nh-L9i-17xx air cooler). By disabling E-cores, it drops to 37C in bios (that's 5 degree difference, quite significant if you ask). And after I flash the modded bios image, CPU temp goes about 39C in bios (not much of a bump, reasonable for non-overclocking settings). If it's true that AVX512 benefits the emulators like the internet said, I think it's worth to do this mod.
