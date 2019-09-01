These are updated instructions by Frank Gould for consideration.

## PiShop.us UPS HAT for Raspberry Pi

The UPS HAT includes a very low powered battery and for any time away from wall power, a larger capacity (Amperage) battery is required. However, for initial setup, tests, and any short power outages (e.g. brownout), the included .46Amp battery is sufficient. 

Once installed, the HAT needs to be initialized with code instructions (included in `ups.sh`) when powered on to operate with the battery. The default without intialization will result in the HAT powering down within 12 seconds after wall power is removed. It will switch to battery power only after the initialization code executes.

Note: The button located on the top side of the HAT is used to boot the Raspberry Pi using the battery when there is no wall power. Normally, when wall power is connected, the UPS HAT will automatically boot.

The instructions below are to install and configure the `ups.sh` bash script on Debian and Arch versions of Linux.

## Debian Linux: 
To install on Debian versions of Linux, like Raspbian, use the `ups.sh` script to autostart after boot and initialize the UPS HAT. After unzipping the download file, the script can be located in the `scripts` folder. To initialize the HAT after boot, follow the instructions below.

Change the permissions for the script:
```
  sudo chmod +x ups.sh
```

Copy the script to the init.d directory to run the script on startup:
```
  sudo cp ups.sh /etc/init.d/
```

Update the rc file:
```
  sudo update-rc.d ups.sh defaults
```

## Arch Linux: 
To install on Arch Linux, use the `ups.sh` script to autostart after boot and initialize the UPS HAT. After unzipping the download file, the script can be located in the `scripts` folder. To initialize the HAT after boot, copy the following lines from below, or these same lines at the top of the `ups.sh` file. Create a new file in the `ups` folder, paste these lines into it, and name it `ups_start.sh`.
```
#!/bin/bash

#GPIO17 (input) used to read current power status. 
#0 - normal (or battery power switched on manually). 
#1 - power fault, swithced to battery. 
echo 17 > /sys/class/gpio/export;
echo in > /sys/class/gpio/gpio17/direction;

#GPIO27 (input) used to indicate that UPS is online
echo 27 > /sys/class/gpio/export;
echo in > /sys/class/gpio/gpio27/direction;

#GPIO18 used to inform UPS that Pi is still working. After power-off this pin returns to Hi-Z state. 
echo 18 > /sys/class/gpio/export;
echo out > /sys/class/gpio/gpio18/direction;
echo 0 > /sys/class/gpio/gpio18/value;
```
In the `/etc/systemd/system` folder, create a new file named `ups-start.service`, paste the following lines, and save it. Note: change the `ExecStart` location to match your username and folders.
```
[Unit]
Description=PiShop.us Desktop Battery Service
Requires=time-sync.target
After=time-sync.target

[Service]
Type=simple
User=root
WorkingDirectory=/home/archpi/ups
ExecStart=/usr/bin/bash /home/archpi/ups/ups_start.sh
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```
In a terminal window, paste or type the following commands:
```
sudo systemctl enable ups-start
sudo systemctl start ups-start
```
You should now be able to remove power from the Raspberry Pi and the battery power will last as long as the battery has capacity. With this UPS HAT, there is no way to determine battery capacity. 
