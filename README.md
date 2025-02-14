# MODKernel-V2-Cepheus
Modified XDA original MOD Kernel https://github.com/mrslezak/Xiaomi_Kernel_OpenSource -b cepheus-p-oss with a higher regulator on GPU830 for greater stability, it can now run 3DMark and other extremely demanding applications without lag (provided memory is sufficient, ZRAM at 2GB LZ4 for 6GB Mi9 versions suggested for 3DMark).  Added BOEFFLA Wakelock Blocker 1.1.0 by andip71, following the files listed here: https://gitlab.e.foundation/search?utf8=%E2%9C%93&search=BOEFFLA&group_id=&project_id=295&search_code=true&repository_ref=; Change drivers/base/power/Makefile; Add boeffla_wl_blocker.c and .h in /drivers/base/power/, edit main.c, edit wakeup.c, add config for BOEFFLA_WL_BLOCKER in kernel/power/Kconfig; add to defconfig CONFIG_BOEFFLA_WL_BLOCKER (enable).  Most files are in /kernel/power/ except defconfig arch/arm64/cepheus-def-config.  Evira kernel used the MOD Kernel original repo so files can be likely all copied from https://github.com/EviraKernel/Officialmi9/tree/Evira/ if needed for Xiaomi 855 changes.  Added Dynamic Fsync which is developed by Paul Reioux aka Faux123 <reioux@gmail.com> in all implementations with 1 update for SDM845 pulled from https://github.com/pappschlumpf/SmurfKernelOP7.  Kernel is likely the final build with performance where it stands.

M.O.D. Kernel - Mi9
==========================

Note this is a patched version of the original source code for the Xiaomi Mi9, and has been tested on MIUI, Xiaomi.eu, MiGlobe,
AOSP, and many other ROMs.  9.8.1 versions are usually the last Android 9 compatible versions.  Q and 10 do not have source ATM.
The sourcecode on the Xiaomi website will not compile as it is missing many flags and patches.

In it's current state, this version can be compiled via Clang (QCOM Clang in the case of root directory file QClang8_Build_F2FS).
This also has several additional features over the stock implementation.  Note in order to use these functions, the kernel must 1)
be compiled from source 2) packed into an existing boot.img - the easiest way is to use Android Image Kitchen available on XDA
3) flashed to the device 4) patched with Magisk 5) a kernel manager which supports the added functions must be installed 
(APK for SmartPack or Google PlayStore for EX Kernel Manager or FK Kernel Manager) and the settings must be turned ON. There is
also a Magisk Module here that will setup the optimal parameters or a battery saver mode.  It has this repo prebuilt:
https://forum.xda-developers.com/Mi-9/development/m-o-d-kernel-mi-9-android-9-pie-t3960271

1) Fsync toggle - enable / disable.  This an be set with any Kernel manager - SmartPack is a good free one although I use
FK kernel manager generally - not for its ease of use but because I have it.  Note that disabling Fsync improves ROM write 
performance but can be dangerous if the system crashes - in a worst case sceanrio, the phone could be bricked.  I've used 
for 4 months on the Mi9 without any issues (disabled) but your experience may differ.  Fsync can be read about online, it's 
really a hack more or less to get over slow write speeds.  This setting is usually under Misc or I/O.

2) GPU can be Over Clocked to 830ghz.  This frequency was chosen because of how it behaves on my Mi 9 with Fsync disabled.
Faster frequencies result in graphics issues or crashing; 830mhz seems to be the ideal setting where graphics are smooth
and no crashing or artifacts occur on the GPU rendering.  FK Kernel Manager is able to set a maximum GPU that sticks (sometimes)
whereas the method used by SmartPack does not max out the CPU to the selected frequency.  Still a workaround is to use
the dialcode for ROMS based on Global & China Dev variants - dial *#*#8106#*#* and turn on GPU Overfreq Mode.

3) LZ4 compression has been added to ZRAM.  Use a kernel manager to select.  It seems to be fastest with smaller ZRAM sizes,
limited testing shows 512mb-1gb range gives good performance.  Smaller allocations seem faster.

4) F2FS support is enabled on this kernel with regular ICE encrption.  What does this mean?  F2FS is a fast filesystem compared 
to EXT4.  Nearly all ROMs and kernels will default to EXT4 as it is a stable and long living format.  The kernel here can run EXT4
or F2F2 both formats are enabled in the cepheus_user_defconfig for the Mi9 device.

F2FS can only be installed on the Mi9 data and cache partitions with twrp-3.3.1-41-cepheus-mauronofrio.img+ at this time.  No other 
TWRP supports mounting F2FS partitions.  To use F2FS, you will lose all programs and everything on userdata and cache partitions.  
You should backup programs via whatever route you have - Google backups will save all PlayStore programs, but I am unsure how often
they update your device.  MiCloud backups will save all root file program APKs as well as program settings.  You can also use 
Titanium Backup Pro (at a cost) and save the backups to your computer or a flash drive via OTG.  There are other Android backup 
solutions available, just be sure to use one of them before formatting to F2FS.  You will be setting up a new phone and will
have to login to all your apps over again.  

The process to setup F2FS is to first download a compatible ROM (MIUI based, Android 9 (Pie), this includes MIUI Dev, Xiaomi.eu, 
MiGlobe, Revolution OS, etc. ENSURE ANDROID 9: 9.8.1 variants are usually the last Android Pie build, but using <=9.7.22 is safe.  
No Android 10 / Q source has been released and therefore no kernels can support it yet.  After downloading, you can install it as
a new phone.  Do not bother setting up anything as you will have to redo this part. Ensure you have installed Mauro TWRP noted above
by booting into Fastboot (volume down + power button buttons), then install via: fastboot flash recovery 
twrp-3.3.1-41-cepheus-mauronofrio.img *** NOTE THE fstab.qcom FILE IN THIS REPO MUST BE IN /vendor/etc/ OR YOUR PHONE WILL NOT BE 
ABLE TO MOUNT THE DATA PARTITION!!!! *** So your next step is to copy the fstab.qcom via a root browser or from recovery:
adb push fstab.qcom /vendor/etc/fstab.qcom while in TWRP with Mount- Vendor (volume up + power buttons, release power when the Logo 
shows on the screen).  The XDA post mentioned earlier has TWRP flashable zip files that automatically do this for you, also in this repo.

Flash this kernel and you can also flash the dtbo.img.  This can be created by downloading and installing this
https://android.googlesource.com/platform/system/libufdt/+/master/utils/ download the .tgz file, extract and from the src directory,
run python mkdtboimg.py dtbo.img /out/arch/arm64/boot/dts/qcom/*.dts.  This image will be flashable to DTBO in TWRP, and required.
The image which builds in /out/arch/arm64/boot/Image-dtb has to be repacked into a boot.img and flashed to BOOT im TWRP.  Then
Magisk 19.3 or 19.4 (Canary beta builds, for root) or disable-dm-verity-force-encrypt.zip (no root) needs to be installed in TWRP to 
boot.  Be sure to flash one of the above files or the ROM will not boot!

Detailed formatting to F2FS from TWRP Wipe/Advanced/Change or repair fileaystem' choose F2FS for data and cache.  See the XDA post.

After converting to F2FS and with the fstab.qcom properly placed in vendor/etc/fstab.qcom, you should now be able to setup as a new
phone.  The data partition will be filled as software installs.  Once complete you can take TWRP Nandroid backups of the new F2FS
partition.  It is also wise to make a simple backup of the flashed BOOT and DTBO partitions as a safety precaution, especially if 
you are going to try out new tweaks to the source code.  Enjoy!
