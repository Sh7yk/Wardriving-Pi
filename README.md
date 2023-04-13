# Wardriving-Pi
Complete instructions for setting up Raspberry Pi for auditing Wi-Fi using a phone  or tablet

!!!All explanations of the GUI part are relevant for the xfce graphical shell, which comes by default 
if you download Kali for raspberry pi from the official site!!!
Instructions for beginners and not only, so the explanations go step by step.
So, you already have a cd card (take the fastest one you can find,SanDisk Extreme Go worked great) 
with downloaded kali linux image for raspberry pi,
Raspberry is connected to the monitor, mouse and keyboard, VNC Viewer or any other application for 
remote access is installed on the phone or tablet.


These materials are provided for informational and educational purposes only and should not harm anyone. 
The author is not responsible for your actions. stay on the bright side ;)
let's go!

-----------------------------------------------------------------------------------------------------------------------------
1. We set up sections of the cd card, this is important, otherwise there will be no place to store files
~~~~~~~~~~~~~~~~~~~~~~
>fdisk /dev/mmcblk0
~~~~~~~~~~~~~~~~~~~~~~
delete the second partition and re-create, choose like this:
d, 2, n, p, 2, enter, enter, n, w

do next:
~~~~~~~~~~~~~~~~~~~~~~~~~
>partprobe
>resize2fs /dev/mmcblk0p2
~~~~~~~~~~~~~~~~~~~~~~~~~~
**********************************************
2. Now we are updating:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>apt-get update
>apt-get upgrade -y
>apt-get dist-upgrade
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

***********************************************
3.Set up VNC access:
open raspberry pi settings:
~~~~~~~~~~~~~~
>kalipi-config
~~~~~~~~~~~~~~

select from the menu:

Boot Options>Desktop /CLI>Desktop Autologin

This makes the login
automatic when turned on, no login password needed

If you do not enter the username, and just press enter, you will automatically log into the system as root, 
if you register, then you will be logged in under this name.

confirm

SSH> choose yes
This allows ssh access.

finished here.

*****************************************************************************************

4. Next, install x11vnc:
~~~~~~~~~~~~~~~~~~~~
>apt install x11vnc
~~~~~~~~~~~~~~~~~~~~

set the password for the vnc connection:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
> x11vnc -storepasswd "your_password" /etc/x11vnc.pass
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

create a service for vnc and add it to startup:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>nano /lib/systemd/system/x11vnc.service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

add these lines to it:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[unit]
Description=Start x11vnc at startup.
After=multi-user.target
[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /etc/x11vnc.pass -rfbport 5900 -shared
[Install]
WantedBy=multi-user.target
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
save and exit

Add the service we created to systemct, make it available and run:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>systemctl daemon-reload
>systemctl enable x11vnc.service
>systemctl start x11vnc.service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
***********************************************************

5. turn on wifi on your phone or whatever you have;
connect to him with a raspberry;
assign it a static ip:
right click on the wifi icon, select
connection information and connection settings.
In the parameters, select your network, click the gear> main
(check the boxes "connect automatically" and
"all users can connect")>IPv4 settings
(we take the data from the "connection information" window)

prescribe:
Address: IP address (raspberry address, which you will specify when connecting via vnc or ssh (remember!)),
Subnet mask: 255.255.255.0,
Gateway: Default gateway (address of the device that distributes wifi)
click add, save
**********************************************************************************************************
6. In order for the desktop to be displayed correctly on the client side, set the default resolution:
edit the file:
~~~~~~~~~~~~~~~~~~~~~~~~~
>nano /boot/config.txt
~~~~~~~~~~~~~~~~~~~~~~~~~
uncomment (delete #) on these lines:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
framebuffer_width=1280
framebuffer_height=720
save and exit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
restart raspberry

_________________________________________________________________________________________________________________________________________________________________
From now on, it will be possible to connect to the raspberry from any vnc client (or ssh) by entering IP +: 5900 (for example 192.168.100.4:5900) and password
ATTENTION! in order to connect to raspberry, you must be on the same network! we try to connect from the phone using the vnc client, everything should work.
_________________________________________________________________________________________________________________________________________________________________
To work via vnc or ssh via bluetooth:
\|/\|/ why is this needed \|/\|/
To audit wireless networks, we need to disable all interfering services with the "airmon-ng check kill" command,
one of them is NetworkManager, which is responsible for all network connections of the raspberry.
That is, we lose control over the raspberry while working in vnc by running the "airmon-ng check kill" command.
But bluetooth has its own network service, which is not affected by this command, so we work through bluetooth.
******************************************************* ******************************************************* *****************************************

7. The Bluetooth connection is not always stable, so let's make sure that the raspberry checks every minute whether the connection is working and if not, 
then connects again.
In order for the bt-pan.py script to work properly, it needs adequate Python 2.7 which is not present since 2019.3 version.
therefore, in the /usr/lib directory, we need to replace the python2.7 folder with the one in this repository,
this is python2.7 straight from version 2019.3
After the replacement, we take two files bt-pan.py and check-and-connect-bt-pan.sh and transfer them to the "/bin" directory.
Making bash script executable:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>chmod +x check-and-connect-bt-pan.sh
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wake up the bluetooth service on raspberry and install all dependencies:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>apt install bluetooth bluez bluez-tools bluez-firmware pi-bluetooth
>systemctl enable hciuart.service
>systemctl start hcuart.service
>systemctl enable bluetooth.service
>systemctl start bluetooth.service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
reboot

8. The bluetooth icon should appear on the panel, click on it:

Adapters> select the desired adapter (the built-in one is enough, but you can use an external dongle) and make it "always visible"> close
******************************************************* *********************
9. Turn on Bluetooth on your phone, look for our raspberry in the list of available ones, connect.
We confirm the connection on both sides, wait until the phone and raspberries finally make friends.
On the phone, turn on the Access Point and the Bluetooth tethering function.
******************************************************* ********************
10. Now we return to the bin folder, open the check-and-connect-bt-pan.sh script, and write the mac address of our phone there instead of what is indicated there:
BT_MAC_ADDR=(mac of your phone)

******************************************************* *********************

11. Again, click on the bluetooth icon with the left mouse button and select: (!) Devices> select our phone> click the checkmark so that the raspberry trusts your phone.
Right-click on our phone in the list of devices, select "network access point" (and check the boxes for Auto-connect)

From the top right, notifications about the connection of bnep0 and the ip address issued to the raspberry should pop up, let's write it down.
it is at this address that we will work in vnc via bluetooth, that is, connecting via wifi and bluetooth will occur at our personal addresses (two different sessions)!

12. Click again on the bluetooth icon:
Service Center>PAN support and DUN support choose blueman
(IT IS IMPORTANT!!!)
done, you can reboot and get to work.
******************************************************* ***************
Now let's make sure that our raspberry checks every minute if the connection to our phone is lost,
and if lost, it will reconnect:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>crontab -e
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

select the editor nano or vim, at the end of the file we write the following line:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
* * * * * /bin/check-and-connect-bt-pan.sh
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

save and exit

reboot

FINALLY THIS IS ALL

Now you just need to take the phone, find the raspberry in the list of bluetooth devices, connect to it, turn on the bluetooth modem (bluetooth access point)
on the phone and by the ip address of the raspberry, connect to vnc or in the ssh terminal emulator.

--DO NOT BLINDLY TEST FOR THE FIRST TIME! CONNECT A MONITOR, MOUSE, KEYBOARD TO ELIMINATE POSSIBLE PROBLEMS--

We do a trial shutdown of network services using the "airmon-ng check kill" command, the connection should not be lost,
if you got kicked out, something went wrong and the bt-pan script didn't work, check the logs.
Also try to interrupt the session yourself, for example, by turning off the bluetooth on the phone, 
then turn it back on, also check that the bluetooth modem, it must be turned on. Within a minute, you should be able to access
the raspberry again. If this is not the case, check if you wrote the task in cron correctly.
==================================================================================================
  This is the minimum settings that are necessary for the normal operation of the wardriver with the Raspberry Pi from the start.
Good luck Warrior driving!
************************************************************************************************************************
BONUS
The raspberry pi has a wifi adapter installed that supports all the functions we need, injections, monitoring, and even AP,
although the Internet says that this is not the case.
This is done with just one command:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>iw phy phy0 interface add mon0 type monitor
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
this command is enough to create a virtual interface based on the built-in physical interface in monitor mode
with the name mon0, which is what we specify when working with aircrack tools
To remove this virtual interface, simply stop it in airmon-ng:
~~~~~~~~~~~~~~~~~~~~~
>airmon-ng stop mon0
~~~~~~~~~~~~~~~~~~~~~
or reboot the system
Bye
