# Apple Fusion Drive Imaging

Apple fusion drives consist of two physical storage devices combined as a logical volume.  One device is Solid State, and the other SATA.  Imaging is most easily accomplished by placing the Apple computer with the fusion drive in [Target Disk Mode](https://support.apple.com/en-us/HT201462) because the logical volume is presented.  Image the logical volume and you're done.

But what if you can't place your device into Target Disk Mode?  Can you image the devices separately and still create the logical volume?  Yes, you can.

---

## Device Imaging

Imaging of Apple Macintosh computers (Intel Macs) can be accomplish with [DEFT Zero](http://www.deftlinux.net/2018/09/01/deft-zero-2018-2-ready-for-download/).  DEFT Zero is a personal favorite of mine because it is EFI ready which means it will boot modern Windows and Mac systems.  It is small, able to load entirely into RAM and free the USB port used to boot DEFT.  Booting with DEFT means no computer disassembly, and that can be a big deal with some Macs.

DEFT Zero includes the [Guymager](https://guymager.sourceforge.io/) imaging tool which is multi-threaded (*fast*), easy to use, and uses efficient compression.  It also includes other imaging tools should you need them, such as [dc3dd](https://sourceforge.net/projects/dc3dd/) and [ddrescue](https://www.gnu.org/software/ddrescue/) for damaged drives.  

> **NOTE:** It was a damaged SATA drive that led to this post.

Imaging in Expert Witness Format (EWF, aka "EnCase" format) or Advanced Forensics Format (AFF) is acceptable and, in fact, preferred.

---

## Fusing Images

The Apple fusion is proprietary and has not been recreated.  Therefore, you'll need a Mac to recreate the logical volume.  First, the images need to be presented to macOS as raw devices.

### xmount

The [xmount](https://www.pinguin.lu/xmount) tool converts an EWF or AFF disk image into a virtual raw image that can be attached to macOS as a device.  For example, assume you have created two EWF images from the two devices making up the fusion drive (e.g., fusion0.E01, fusion1.E01).  The following commands, run in macOS, will mount the virtual raw images with cache files to catch any changes attempted by macOS (*think file system checks*).

```bash
xmount --in ewf fusion0.E01 --cache fusion0.cache mnt/fusion0/
xmount --in ewf fusion1.E01 --cache fusion1.cache mnt/fusion1/
```
### hdiutil

The macOS builtin tool [hdiutil](https://ss64.com/osx/hdiutil.html) is used to attach the virtual disk images.  This process presents the image as a devie to macOS.

```bash
hdiutil attach -nomount -noverify mnt/fusion0/fusion0.dmg
hdiutil attach -nomount -noverify mnt/fusion1/fusion1.dmg
```

The operating system will attach the virtual images as the next available devices, e.g., /dev/disk3 and /dev/disk4, as well as list the partitions (slices).  The `-nomount` option is necessary or macOS will refuse to attach the device because there are no readable file systems.  

> **IMPORTANT:** Each virtual image is one part of the logical volume, thus they do not independently contain readable file systems.

The `-noverify` option skips what could be a lengthy disk verification process.

## Mounting 

The operating system will note the two parts of the fusion drive once they are attached and present the Core Storage logical volume for mounting as the next available device, e.g. /dev/disk5.  It will not have any partitions (slices) because it is a logical volume.

If the logical volume is not automatically attached, then one or both of the virtual devices likely has a damaged file system.  This can be corrected, without altering the original data (*recall the xmount cache option*).

### diskutil

The macOS builtin tool [diskutil](https://ss64.com/osx/diskutil.html) can both mount and repair file systems.  To repair the *Macintosh HD* file system that is the object of our attention, use the `repairVolume` verb and the xmount cache file will catch and integrate the changes to the filesystem in the virtual images.

```bash
diskutil repairVolume disk3s2
diskutil repairVolume disk4s2
```

When the file system(s) are repaired, the logical volume will be presented (view with `diskutil list`), but not mounted.  To mount the volume, `diskutil` is utilized again.

```bash
diskutil mount readOnly disk5
```

> **NOTE:** In practice, the mount command can return a failure code but still mount.  This appears to be an issue of timing: the mount command reports failure after *n* seconds but the mount process continues until completion.


