## Fix your Bluetooth in Dual-Boot (Finally)
For months, I was trying to find a one consolidated guide to fix Bluetooth across my dualboot as I was a newbie to linux at that time (still am). Yet I never found anything solid, just half baked answers on reddit or an outdated video that barely worked. So I went off to research, used ChatGPT, Reddit, Reddit's Answers AI and many websites after I could finally fix the bluetooth across Dualboot altogether. Then I decided to put it up here so atleast no other newbie like me has to wander off to a number of places only to end up finding half baked answers or years old solutions that don't even work anymore.
## The Problem
Understanding the underlying problem with Dualbooting and bluetooth can already clear up your mind on what needs to be done and why this happens. So in caveman terms, here it is:
- Two OS
- One Bluetooth Device
- One bluetooth adapter
- Device connects in one OS, creates a pair key (basically an identifier) to recognise the adapter 
- Device tries to connect in another OS, recognises adapter, but different pair key
- Stranger Danger!
- Also both OS have their own methods to put the adapter into a half-asleep state so it can be woken up faster upon the next boot.

Now let's get down to the solution, also a huge thanks to @ademlabs and his beautiful python script called [Blue-Sync](https://github.com/ademlabs/synckeys) without which, none of this would've been possible for a newbie like me. Also the steps to use his script I've taken directly from his repo, as I don't think I can explain how to use his script better than him (I don't own the script, neither have I created it or the steps to use it, I'm just putting up an exampled and consolidated one stop guide for people trying to fix Bluetooth across their Dual boot system!)
## Pre-Requisites 
### Windows
- Turn off Fast Startup
- Turn off Power saving under BT Adapter properties in device manager
- Download PSExec
### Linux
- Python Installed
- synckeys.py Downloaded
- Create a systemd service to shutdown BT adapter on shutdown
```
sudo nano /etc/systemd/system/bt-shutdown.service
```
- Now type out this in the .service file using nano
```
[Unit]
Description=Power off internal Bluetooth on shutdown
DefaultDependencies=no
Before=shutdown.target

[Service]
Type=oneshot
ExecStart=/usr/bin/hciconfig hci0 down

[Install]
WantedBy=shutdown.target
```
- Reload the daemon to reload all the services
```
sudo systemctl daemon-reload
```
- Enable the systemd .service file
```
sudo systemctl enable bt-shutdown.service
```
## Syncing Keys across Windows & Linux 

1. Boot into your Linux installation and pair all the devices that you wish to share with your Windows system. This is necessary to create the required initial pairing configurations.
2. Reboot into your Windows system and pair the same devices again. We will use the keys generated from this OS.
3. Open a command prompt in Administrator mode and navigate to the directory where you downloaded PSExec into. Run the following command to dump the keys:
```
psexec -s -i regedit /e c:\keydump.reg HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\BTHPORT\Parameters\Keys
```
5. Copy the file from c:\keydump.reg into a removeable storage or a location which is accessible by your Linux system.
6. Reboot into your Linux system.
7. Copy the keydump.reg file to an accessible location in your Linux filesystem and reboot your PC again to linux.
8. Open a terminal and navigate to the location where synckeys.py is located.
9. Run the synckeys.py Python 3 script with root or sudo:
```
sudo ./synckeys.py /path/to/keydump.reg
```
10. The adapters and devices from the key dump will be compared to the pairing in Linux and if a difference is detected, it will prompt you to update the keys. You can choose Yes or No (default). If you choose Yes, a timestamped backup file is created in the /var/lib/bluetooth/{ADAPTER_MAC}/{DEVICE_MAC} directory before the update is performed.
11
. Once the keys are updated, you can restart the bluetooth service with the following (or the equivalent on your system):
```
sudo systemctl restart bluetooth
```
> **Important:** Whenever switching OSes, make sure previous one being used is shutdown completely before next one is booted, try not to restart directly to use grub.
