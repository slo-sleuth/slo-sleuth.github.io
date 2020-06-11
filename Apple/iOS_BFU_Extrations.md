# iOS Forensics: USB Restricted Mode
_This document contains information about extracting iPhone metadata and stored data using freeware and open source tools._


Since iOS 11.4.1, forensic examiners face a difficult problem: __USB Restricted Mode__.  iOS now has a requirement for the pin or passcode to entered for the device to communicate over USB.  There are necessary exceptions to USB restricted mode that can be exploited for examination purposes.

## USB Restricted Mode Exceptions

USB restricted mode is not present in several iDevice modes, including __Disk Firmware Update (DFU)__ and __Diagnotics__ modes.  DFU is a state where iOS and other software can be pushed to the device.  Diagnostics mode allows Apple to remotely access the device.

Under the right condititions, these two modes can be exploited to recover iDevice information and stored data.

## Obtaining Device Information

The [libimobiledevice](http://www.libimobiledevice.org/) library and tools are useful for interrogating an iDevice for it's core information such as device model, Unique Device ID (UDID), device name, iOS version, Wifi MAC address, and more.  However, it requires USB access to the device which is blocked by USB Restricted Mode.

Device information can be obtained from an iDevice by placing it into Diagnostics Mode.  Besides revealing reveals the device serial number, MEID and IMEI when selecting the :information_source: "Information" button in the Diagnostics Mode home screen, USB Restricted mode is disabled allowing the libimobiledevice tool `ideviceinfo` to function.

__WARNING:__ _This process requires the device to be rebooted.  If your device is in After First Unlock (AFU) mode, you will loose access to decrypted data.  If you have the means (i.e., commercial tools) to access AFU data, use them before using this process._

### Installing libimobiledevice

On a Mac, libimobiledevice can be installed with the [homebrew software package manager](https://brew.sh/) as follows:

```sh
% brew install libimobiledevice
```

NOTE: _The libimobiledevice package pulls the libusbmuxd package which includes the `iproxy` tool.  This tool will be used during data extraction to make a tcp connection to the device._

### Placing iDevice in Diagnostics Mode

Diagnostics mode is entered from a power off state.  The method varies depending on the hardware type (i.e., the presence of a physiscal home button).

* iPhone 6s or earlier
  * Power off device
  * Press and hold home and volume up buttons
  * Plug Lightning cable (connected to computer) into device

* iPhone 7 or later
  * Power off device
  * Press and hold volume up and down buttons
  * Plug Lightning cable (connected to computer) into device

### Running ideviceinfo

The `ideviceinfo` command must be run with the `-s / --simple` option because the device is locked.

```sh
% ideviceinfo -s
BasebandCertId: ###########
BasebandKeyHashInformation: 
 AKeyStatus: 2
 SKeyHash: <base64 value>
 SKeyStatus: 0
BasebandSerialNumber: <base64 value>
BasebandVersion: 7.60.01
BoardId: 10
BuildVersion: 17F80
CPUArchitecture: arm64
ChipID: 32784
DeviceClass: iPhone
DeviceColor: 1
DeviceName: iPhone
DieID: ################
HardwareModel: D11AP
HasSiDP: true
PartitionType: GUID_partition_scheme
ProductName: iPhone OS
ProductType: iPhone9,2
ProductVersion: 13.5.1
ProductionSOC: true
ProtocolVersion: 2
SupportedDeviceFamilies[1]: 
 0: 1
TelephonyCapability: true
UniqueChipID: ################
UniqueDeviceID: <hex ID>
UntrustedHostBUID: <uuid>
WiFiAddress: <mac address>
```
_NOTE: Some values obscured above for privacy_

## Obtaining Device Data

Without commercial tools, extraction of device information and data from locked iOS devices was not possible after USB Restricted Mode was implemented.  

In September, 2019, the [checkm8](https://checkm8.info/checkra1n-jailbreak-exploit/) exploit was made public making it possible to extract data from devices.  The freeware [checkra1n](https://checkra.in/) tool was released shortly thereafter that allowed data extraction from unlocked devices.  Some commercial tools incorporated checkra1n into their products.

### Jail Breaking with checkra1n

The checkra1n jail break is a (currently) closed source, tethered exploit (does not survive reboot) that works on iDevices with with A5 through A11 processors running iOS 12.3 and above.  This means it works on iPhone 4s through iPhone 10, for example.

Checkra1n currently runs on macOS and Linux.  In macOS, there is a GUI tool that can also be run through the command line in ncurses (default) or command line modes (option -c).  The GUI and ncurses modes walks the user through placing the device in DFU mode.  However, these modes can refuse to function for "unsupported devices" (e.g., older or new iOS versions) and require the phone have USB Restricted Mode disabled for the device to be detected.

NOTE: _At the time of this writing, checkra1n 0.10.2 is the latest release but will not operate on the latest iOS 13.5.1.  However, the checkra1n successfully exploits iOS 13.5.1 in command line mode._

I prefer the command line mode because it will attempt the exploit on any device in DFU mode, event those that are unsupported by the GUI.  If you recall from the beginning of this document, DFU mode does not employ USB restricted mode out of necessity.  Another benefit to command line mode is that checkra1in remains running allowing checkra1n to be used repeatedly without restarting the application.

Checkra1n can be used to extract data from iDevices in the following states:

* With Pin / Passcode
  * Full file system extraction
* Without Pin / Passcode
  * Before First Unlock (BFU) extraction

#### Create a Link to checkra1n

On your macOS device, [download](https://checkra.in) and install the latest checkra1n version.  To make execution easier, create a simlink to the checkra1n executable in your home directory.  This only needs to be done once.  Open Terminal and run the following command:

```sh
% ln -s /Applications/checkra1n.app/Contents/MacOS/checkra1n ~/checkra1n
```

#### Executing Checkrain Command Line Mode

Launch `checkra1n` from the link you have created, remembering to use option `-c`:

```sh
% ./checkra1n -c
```

NOTE: _If you receive a popup window with the error `"checkra1n.app" cannot be opened because the developer cannont be verified`, click the "Cancel" button and navigate to "Security and Privacy" in the Settings App.  At the bottom of the "General" tab, click the "Open Anyway" next to the checkra1n warning.  Select the "Open" button at the next warking and you will now be able to execute checkra1n._

After executing `checkra1n` in command line mode, checkrain will print information about the applications version and the developers and contributors.  It will then report it is waiting for a device to be connected to the computer in DFU mode.

```sh
% ./checkra1n -c
#
# Checkra1n beta 0.10.2
#
# Proudly written in nano
# (c) 2019-2020 Kim Jong Cracks
#
#========  Made by  =======
# argp, axi0mx, danyl931, jaywalker, kirb, littlelailo, nitoTV
# never_released, nullpixel, pimskeks, qwertyoruiop, sbingner, siguza
#======== Thanks to =======
# haifisch, jndok, jonseals, xerub, lilstevie, psychotea, sferrini
# Cellebrite (ih8sn0w, cjori, ronyrus et al.)
#==========================

 - [*]: Waiting for DFU devices
```

Connect an iDevice in DFU mode.  This can be done from a running state, recovery mode, or from a powered off state.  There are many tutorials online.  I will only cover the basic key combinations here.  If you are having difficulty, search online.

* iPhone 6s and earlier:
  * Press and hold power and the home button for 10 seconds (if a running phone, at least three seconds past screen blank)
  * Release power and continue to hold the home button for another 7 seconds, or until checkra1n registers DFU mode.
* iPhone 7 and later:
  * Press and hold power and volume down for 10 seconds (if a running phone, at least three seconds past screen blank)
  * Release power and continue to hold the volume down button for another 7 seconds, or until checkra1n registers in DFU mode.

Once a phone is connected in DFU mode, checkra1n will immediately proceed to exploit the phone.

```sh
 - [*]: Exploiting
 - [*]: Checking if device is ready
 - [*]: Setting up the exploit (this is the heap spray)
 - [*]: Right before trigger (this is the real bug setup)
 - [*]: Entered download mode
 - [*]: Booting...
 ```
 
IMPORTANT: _If the phone is locked and the passcode is unknown, press and hold the volume up and volume down buttons while the exploit as soon as the exploit indicates it is booting.  This will cause the phone to boot into Diagnostics mode.  If you miss diagnostics mode, re-run the checkra1n._

NOTE: _If you have the passcode, you can let the phone boot normally._

System messages about the exploit will be displayed on the screen.  They will be hard to read on high DPI devices.  If the exploit succeeds, you will see the final messages in Terminal:

```sh
 - [*]: Uploading bootstrap...
 - [*]: All Done
```

### Extracting Data

If you booted to Diagnostics mode and you have the device passcode, use checkra1n again and let the device boot normally and enter the passcode.  You will be able to perform a full file system extraction.  Diagnotics mode will only allow extracted data in BFU mode, that is, data that is not encrypted with the passcode.  In either case, the data extraction process is the same.

#### Establish TCP connnection to iDevice

_Diagnostics mode will time out in a few minutes if there is not interaction with it.  It is expecting the user to connect to a WiFi network.  If extracting data in Diagnostics mode, keep the screen awake or connect to a safe WiFi network.  More study is needed here to determine if the timeout can be disabled by connecting to a router disconnected from the Internet.  An Internet connection may lead to remote wiping._

With the checkra1n exploited iDevice connected to the computer by USB cable, use the `iproxy` command, installed with libimobiledevice, to create a TCP connection to the iDevice:

```sh
% iproxy 4444 4444
````

This command connects the local computer port 4444 (first argument) to iDevice port 4444 (second argument).  You can use any available computer port for the connection, but checkra1n has an ssh process listening on port 4444.

From here, you can log into the phone over ssh.  However, I will only demonstrate the command to extract all accessible files to the local directory.

```sh
% ssh root@localhost -p 4444 tar -cf - / > bfu.tar
```

If this is the first connection to the iDevice, ssh will warn that the authenticty of the connection cannot be confirmed (because you don't have the RSA key stored).  Confirm that you want to connect and the key will be stored in the ~./ssh/ directory.  You will then be prompted for the passcode which is "alpine".  Finally, you will see tar command messages as the command executes.

The tar command option `c` creates a new archive and writes it to a file, option `f` which in this case is `-` a shortcut for stdout.  The data is redirected `>` to a local file `bfu.tar`, which can be any file name of your choosing.  When the tar command completes, the ssh connection is dropped automatically.

NOTE: _You can choose to compress with tar or use verbose mode to monitor progress, but that slows the process quite a bit because the iDevice is doing all the work.  It is more effient to compress on the computer side of the transaction or after the transfer is complete.

