# SuperSID on Raspberry Pi

## Hardware

The following setup procedures have been tested using the Raspberry Pi 3 Model B, Raspberry Pi 3 Model B+, Raspberry Pi 4 B, and Raspberry Pi Zero 2 W using the Raspberry Pi OS Buster (Legacy). 
The sound card used was a StarTech ICUSBAUDIO2D 

## Raspberry Pi Setup

[Set up your Raspberry Pi](https://www.raspberrypi.com/documentation/computers/getting-started.html#setting-up-your-raspberry-pi) with the image **Raspberry OS (32-bit)**. AT the date of the installation this corresponds to *buster*.
Boot on the new micro-SD card, follow normal process for any fresh system install. Connect to the internet.

Execute the classic:
```console
    $ sudo apt-get update
    $ sudo apt-get upgrade
```
After the setup is complete, click on the Raspberry icon in the upper left of the screen.
From the drop down, choose Preferences > Raspberry Pi Configuration.
Under the Interface tab, Enable SSH & VNC - this will allow you to access the RPi remotely from your PC.
In a terminal window type fbset to find the current screen resolution and record it.
In the terminal window
Type the command sudo raspi-config
Under Advanced Options choose Expand Filesystem
Move your cursor to the upper right of the window and hover over the icon that will show the wlan0 (WiFi) address that your router has assigned to the RPi.  Record this address (192.168.1.xxx) if you intend to run the RPi headless using RealVNC.  


## Headless Setup

[Here](https://youtu.be/NWBmYnNvN3A) is a link to a youtube video with information on how to install RealVNC on a Windows or Mac computer and here is one for Ubuntu.
  
If you get the message “currently cannot show desktop” when trying to connect to the RPi from your PC, you must lower the screen resolution of the RPi.

If you have disconnected the RPi from a monitor, reconnect it and choose a lower resolution.

**Buster OS:** under Preferences > Raspberry Pi Configuration > Display > Set Resolution.  Choose a lower resolution than the one you recorded earlier when using fbset.

**Bullseye OS:** under Preferences > Raspberry Pi Configuration > Display > Headless Resolution.  Choose a lower resolution than the one you recorded earlier when using fbset.

Reboot and try again to log in.

Alternatively, if you have an app such as PuTTy, you can SSH into the Pi using its WiFi address.  
**Buster OS:** After connecting type sudo raspi-config, choose #2 Display Options, then D1 Resolution.  Choose a lower resolution than the one you recorded earlier when using fbset.  Choose Finish and Reboot to have the change take effect.

**Bullseye OS:**  After connecting type sudo raspi-config, choose #2 Display Options, then D5 VNC Resolution.  Choose a lower resolution than the one you recorded earlier when using fbset.  Choose Finish and Reboot to have the change take effect.


## SuperSID Installation

## 1) Get the latest supersid software

Open a terminal window and get the source from GitHub.com

```console
    $ cd ~
    $ git clone https://github.com/sberl/supersid.git
```

To update (pull) to the latest version, do:
```console
    $ cd ~/supersid
    $ git pull
```

Now do the following:

~$ cd supersid

~/supersid $ mkdir Data

~/supersid $ mkdir outgoing

These directories will be used to store the data that will be sent via ftp to Stanford.
If your RPi is connected to the internet you can ignore 2. Extra Software and 3.1 optional virtual environment.
Proceed with the commands under 3.2

## 2) Extra software

Time synchro over the Internet:
```console
    $ sudo apt-get install ntpdate ntp
```
Follow the tutorial [Raspberry Pi sync date and time](https://victorhurdugaci.com/raspberry-pi-sync-date-and-time)

Optional: Virtualenv management for Python:
```console
    $ sudo apt-get install mkvirtualenv
```

## 3) Installing SuperSID

### 3.1) optional virtual environment

This step is optional. Creating your own environment allows to install libraries in all freedom,
without `sudo` and ensure you have a coherent and working set of libraries (sound card).
If your Raspi is dedicated to SuperSID then you can skip this step and install all globally.

From /home/pi:
```console
    $ cd ~/supersid
    $ mkvirtualenv -p /usr/bin/python3 supersid
    $ workon supersid
    $ toggleglobalsitepackages
```

Your prompt should now start with '(supersid)'

This also ensures that we run in Python 3.7.3 as per current configuration.


### 3.2) Global or local installation

This Raspi 3 is dedicated to SuperSid or you do not plan to mix various libraries: install at system level all the libraries.
You can do so exactly like you would do in linux, for an local installation inside the virtual environement by first executing 'workon supersid'.


```console
    $ sudo apt-get install python3-matplotlib
    $ sudo apt-get install libasound2-dev
    $ sudo apt-get install libatlas-base-dev

    $ cd ~/supersid
    $ pip3 install -r requirements.txt
```

Optional and not required. Install when you want to test additonal audio libraries:

```console
    $ sudo apt install libportaudio2
    $ pip3 install sounddevice
    $ pip3 install PyAudio
```


Identify Sound Card
________________




Connect your sound card (which should be done when the RPi has been powered down) and SuperSID preamp.
In SuperSIDonRaspi3.md, follow the instructions under 5) Choose your USB Sound Card to identify the card.  This will be used in the supersid.cfg file under [Capture].  Ex: DEVICE=plughw:CARD=Device,DEV=0 


SuperSID Configuration File
________________




Using the File Manager, navigate to /home/pi/supersid/Config and open supersid.cfg with the Text Editor.
Edit the file to add the information for your station.  Refer to the file ConfigHelp.md under docs https://github.com/sberl/supersid  


Viewer can be either text or tk.  Text is a basic text display.  The tk parameter will display the spectrograph which is useful in positioning the antenna for the strongest signals.  


In the [FTP] section of supersid.cfg, set “automatic_upload” to yes if you wish to send your data to Stanford.  Add the stations that you wish to send under “call_signs”.  Separate the stations with commas without spaces.  The file to be sent must be in supersid_format - one file for all stations.  Using “log_format=both_extended” in the supersid.cfg file will create a file in supersid_format and also a file for each station in sid_format.  The sid_format files can be useful if a plot file of an individual station is desired.
The supersid.cfg file uses the /home/pi/tmp directory to store the files to be sent via ftp.


Sending the ftp is accomplished by the program ftp_to_stanford.py
Crontab can be used to run the program at a specific time each day.
In a terminal window create a script in the /home/pi directory by typing nano ftp_stanford.sh   The script should contain the following: 
#!/bin/bash
cd ~/supersid/supersid
./ftp_to_stanford.py -y -c ~/supersid/Config/supersid.cfg


Save the file (Ctrl O) and exit (Ctrl X).
Make it executable by typing sudo chmod +x ftp_stanford.sh 


In a terminal window type sudo crontab -e.  Choose #1 for the nano editor.  Add the following at the bottom of the file:
5 18 * * * /home/pi/ftp_stanford.sh > /home/pi/ftp.log 2>&1
Save the file and exit.
In this case, the script will run at 5 minutes after 1800 hours (midnight UTC) and create a log file showing the results.  Adjust this to reflect your timezone. 


Starting SuperSID
________________




In a terminal window type cd ~/supersid/supersid
Start the program using: ./supersid.py -c ../Config/supersid.cfg


If your spectrograph looks like the image below, your sound card is only sampling up to 48000 Hz.  The stations in the US require a sampling rate or 96000 Hz.  
  



Your sound card may not be capable of the higher sampling rate or it could be that the RPi does not recognize the card properly.  


The spectrograph should look more like this:
  



SD Card Backup
________________




It is advisable to make a copy of your SD card once you determine that everything is set up and working.  Under Accessories there is a utility called SD Card Copier that can be used along with a USB SD card reader to clone your card.


Automatic Restart After Power Outage
________________




If you would like this option, do the following:
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
add the following to the bottom of the file:
@lxterminal --command “/home/pi/runSID.sh”
Save and Exit


In the /home/pi directory, create a file called runSID.sh
nano runSID.sh
add the following to the file:
#!/bin/sh
sleep 30
cd /home/pi/supersid/supersid
./supersid.py
Save and Exit
Make it executable by doing:
sudo chmod +x runSID.sh


Plot commands
________________




In a terminal window, navigate to /home/pi/supersid/supersid
For a standard plot:
./supersid_plot.py -f ../Data/<filename>.csv -c ../Config/supersid.cfg


To create a plot and save it without viewing:
./supersid_plot.py -f ../Data/filename.csv -n -p ../Data/<filename>.pdf -c ../Config/supersid.cfg


For a plot containing NOAA flare data:
./supersid_plot.py -w -f ../Data/<filename>.csv -c ../Config/supersid.cfg


For an interactive plot that enables you to turn stations off/on:
./supersid_plot_gui.py ../Data/<filename>.csv
For this, use the file in supersid_format that contains all of the stations as listed in your supersid.cfg.


For a plot that can be sent via email:
./supersid_plot.py -e -n -f ../Data/<filename>.csv
 -c ../Config/email.cfg
The email.cfg file in the supersid/Config directory must be filled out with your information.


supersid_plot arguments:
-f        location and name of csv file
-c        location and name of config file
-n        create plot without showing on the screen
-p        create PDF file - ex: -p myplot.pdf
-e        email
-w        retrieve NOAA flare information




Directory structure for SuperSID
________________




/home/pi
        /supersid
                /Config
                /Data
                /docs
                /outgoing
                /supersid


Sound Cards
________________




Based on the chipset, the following cards should work at 96kHz
StarTech ICUSBAUDIO2D
Syba SD-DAC63094
Syba SD-DAC63095
