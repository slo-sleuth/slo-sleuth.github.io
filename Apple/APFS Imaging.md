# Imaging APFS Volumes

Images from Apple products can be made from recovery mode without the need of external boot devices (Sumuri's Recon, Blackbag's MacQuisition, Linux boot disks).  A users password is required for filevault decryption, if in use, an on newer devices it usually is. 

## T2 Macs

Macs with T2 chips (iMacs 2017+, MacBooks 2018+) must be extracted logically.  The T2 chip is part of the device encryption schema and physical partition images cannot be read apart from the specific chip in the device being extracted. 

### Prepare the Evidence Computer

- Boot into recovery 
  - press and hold "CMD + R" on boot
- Open Terminal 
  - In the main meny, click Utilities | Terminal
  - Provide a user password to access the terminal
- Determine APFS volumes to image 
  - `# diskutil apfs list`
- Mount the volume to be extracted
  - `# diskutil apfs unlockVolume disk2s1 -nomount -passphrase ######`
  - If you prefer, leaving off the passphrase option will bring up a password prompt
  - Imaging the unlocked volume device (i.e., disk2s1) will not result in an unencrypted volume image
- Mount the unlocked volume read-only
  - `# diskutil mount readOnly disk2s1`
- Determine the size of the logical data
  - `# df -h`
  
### Prepare the Target Device

- Plug in an external drive formatted exFAT, HFS+, or APFS
  - exFAT preferred for universal access.
  - Drive will automount read-write
  
It is possible to take two different tracks at this point  
  
- Create a disk image large enough to hold the evidence volume's logical data
  - hdiutil create -size <integer><m|g|t> -volname <desired name> -fs <HFS+|APFS> -attach <diskimage>
  - APFS volumes are relatively new and not universally or always well supported by analysis tools.  Consider using HFS+ to make full use of your toolbox (testing ongoing for differences in APFS/HFS+ disk images)
    - `# hdiutil create -size 70g -volname "Macintosh HD" -fs HFS+ -attach /Volumes/Target/Macbook.dmg`
  - An empty, uncompressed disk image will be created an mounted.  Choosing a volume name that matches the evidence volume will result in the for the disk image mountpoint being appended with a "1"
  - If you failed to attach the image on creation, attach and mount with `hdiutil`
    - `# hdiutil attach /Volumes/Target/Macbook.dmg`
    
### Copy the logical data

- The `ditto` command is present in the Apple recovery environment and will preserve Apple metadata and file ACLs.  It does this by default, so the execution command is quite simple.
  - ditto <source> <destination>
    - '# ditto /Volumes/Macintosh\ HD/  /Volumes/Macintosh\ HD\ 1/`

### Shutdown the system

- There is no need to unmount all the volumes, close the terminal, and shutdown the computer from the main menu. Simply halt the computer with one of two commands and all volumes will be gracefully unmounted.
  - `# halt`
  - `# shutdown -h now`
- Alternatively, the computer may be rebooted.
  - `# shutdown -r now'
 
