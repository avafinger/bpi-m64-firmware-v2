Banana Pi M64 Ubuntu 16.04 LXDE OS Image (firmware) - V2
========================================================

LXDE (Lightweight X11 Desktop Environment) is a desktop environment which is lightweight 
and fast and uses less RAM and less CPU while being a feature rich desktop environment.

If you want a richer desktop environment (but slower) you can install 
Ubuntu MATE on top of this Image and later de-install LXDE.

I use LXDE just because it is very fast, snappy  and responsive!
You can always improve, tweak and tune the way you want at any time.
This is a very LEAN and MEAN OS image to play and learn how to extend it.

This is a preliminary LXDE OS image for the Banana Pi M64 with fully working
----------------------------------------------------------------------------

- Latest kernel 3.10.105 with
- eMMC
- Wifi
- BT (bluetooth)
- OV5640 (camera with AF)
- HDMI 1080P / HDMI 720P
- HDMI digital 
- GbE (Gigabit ethernet)
- LEDs (Blue and Green)
- Support for HW decoding/encoding (cedrus H264) - https://github.com/avafinger/cedrusH264_vdpau_A64
- LCD 7" with Touch Screen (LCD Tested and works, Touch needs to be verified)

*Note*
- Don't power the board with microUSB or a PSU with less than 2.5V, Wifi+HDMI+LCD+eMMC draws a lot of power

Things you should do after flashing the OS Image
-----------------------------------------------

Run in Shell:

	sudo apt-get update
	sudo apt-get dist-upgrade
	sync
	sudo shutdown -h now (or use Shutdown Button)
	wait for the led to turn off
	unplug the power DC
	wait few seconds, and power the board again



Before you start downloading and flashing you should pay attention to this
--------------------------------------------------------------------------

- Read through all this document to understand the commands and the whole process,
  It may looks challenging but it is not.
- We need a linux box to flash the Image into SD CARD, i use LUbuntu 12.04 but could be any distro.
- Grab a good SD CARD (*), 80% of the issues you may have is due to counterfeit or bad sd card brand.
- SD/HC Card reader can also be a source of problems writing to a good SD CARD.
  (*) If you can be sure your SD card is fine and you still have problems, try using another SDHC card reader
- We need 8GB SD CARD, the OS Image fits in a 4GB but need more space as you will see.
- Get a good PSU with at least 2.5A (be on the safe side).
- Make sure you have HDMI (don't use HDMI to DVI if you can).
- Make sure HDMI is connected to the board very tight or you may experience some flickering or the image will not appear.
- Initial setup is for HDMI 1920x1080 (1080p@60), if you need support for the 7" LCD change a64-2GB.dtb with the correspondig a64-2GB_LCD7.dtb

7" LCD support
--------------

LCD Panel (tested by someone else)
![lcd](https://github.com/avafinger/bpi-m64-firmware-v2/raw/master/img/bpi-m64-lcd1.jpg)

![lcd wiring](https://github.com/avafinger/bpi-m64-firmware-v2/raw/master/img/bpi-m64-lcd2.jpg)


- After you create the SD Card with the OS image overwrite a64-2GB.dtb with a64-2GB_LCD7-v4.dtb


		cd /media/ubuntu/boot/a64
		cp -avf a64-2GB_LCD7-v4.dtb a64-2GB.dtb (PWM & Touch - FIXED)

for the Tocuh, see instructions below.

Installation
------------

This is a non orthodox way of flashing the image onto SD card and eMMC.
We will do the following steps:

  1. Download firmware with git clone
  2. Format SD card, and Decomress kernel
  3. Boot from SD card to detect the eMMC
  4. Format eMMC and unzip kernel

There will be no need for requesting unused space on SD card or eMMC, we don't use '.img' file.

Requirements
------------

- Install git and md5sum on your host PC
 

### Manual installation (using git)

1.  Download the files entirely with git 

    a.  **In shell type (host PC):**

            git clone https://github.com/avafinger/bpi-m64-firmware-v2
            cd bpi-m64-firmware-v2


    b. Rebuild boot and rootfs and check MD5 (must match)

            cat rootfs_m64_rc6.tar.gz.* > rootfs_m64_rc6.tar.gz
            md5sum rootfs_m64_rc6.tar.gz
            4a7d0075c547026135f895bbccf6d949  rootfs_m64_rc6.tar.gz

             md5sum boot_m64_rc6.tar.gz 
             2b56e71034d7e893e3ac9ee9ee95e600  boot_m64_rc6.tar.gz


    c.  **Insert a new SD card (get a good one, 8 GB or > )**


    d.  **Find your SD card**


            dmesg|tail
            [97286.659006] sdc: detected capacity change from 15523119104 to 0
            [99023.137526] sd 4:0:0:0: [sdc] 30318592 512-byte logical blocks: (15.5 GB/14.4 GiB)
            [99023.147516] sd 4:0:0:0: [sdc] No Caching mode page found
            [99023.147521] sd 4:0:0:0: [sdc] Assuming drive cache: write through
            [99023.162514] sd 4:0:0:0: [sdc] No Caching mode page found
            [99023.162518] sd 4:0:0:0: [sdc] Assuming drive cache: write through
            [99023.168535]  sdc: sdc1 sdc2


        in this example our sd card is /dev/sdc if we use an SD CARD reader (USB), it could be /dev/sdb if you have only one HDD on your host PC
        so the format is something like /dev/sdX where X is [b,c,d..,g]


    e.  **Start flashing... (Warning, make sure you get the correct device or you may WIPE your HDD)**


            sudo chmod +x *.sh
            sudo ./format_sd.sh /dev/sdc
            sudo ./flash_sd.sh /dev/sdc


    Now you have SD card with kernel in it, you can now boot up bpi-m64 with this SD card and it will detect the eMMC:

  
            user: ubuntu
            pass: ubuntu


2.  Flashing eMMC (the git way) after you boot with SD card 

    a.  **In shell type (host PC):**

            git clone https://github.com/avafinger/bpi-m64-firmware-v2
            cd bpi-m64-firmware-v2

    b. Flash the OS Image to eMMC

    **Start flashing... (eMMC)**

            sudo chmod +x *.sh
            sudo ./format_emmc.sh
            sudo ./flash_emmc.sh

    If everything is OK you can now shutdown and boot up **without the SD card**.



7" LCD with Touch Screen support (Needs to be verified)
-------------------------------------

This is the instructions to work with touch on the LCD 7" (S070WV20_MIPI_RGB).
The file a64-2GB_LCD7.dtb - DTB (Device Tree Blob) has supportfor LCD and Touch Screen.

a. Add **FT5X_TS** touch manually to **/etc/modules**


		sudo modprobe ft5x_ts
        	or
        	add ft5x_ts to /etc/modules file



see if all modules has been loaded:

		type: lsmod


		Module                  Size  Used by
		ft5x_ts                76213  0
		rfcomm                 41308  12
		bnep                   18807  2
		ir_lirc_codec          13350  0
		lirc_dev               18895  1 ir_lirc_codec
		ir_mce_kbd_decoder     13330  0
		ir_sanyo_decoder       12869  0
		ir_rc6_decoder         12871  0
		ir_jvc_decoder         12811  0
		ir_sony_decoder        12786  0
		ir_rc5_decoder         12757  0
		ir_nec_decoder         12892  0
		sunxi_ir_rx            14684  0
		vfe_v4l2              773139  0
		ss                     45757  0
		cedar_ve               20392  0
		videobuf2_dma_contig    20233  1 vfe_v4l2
		videobuf2_memops       12600  1 videobuf2_dma_contig
		videobuf2_core         35245  1 vfe_v4l2
		ov5640                 55857  0
		dw9714_act             13373  0
		actuator               12412  1 dw9714_act
		vfe_io                 44013  3 vfe_v4l2,ov5640,dw9714_act
		hci_uart               31036  1
		bluetooth             217400  34 bnep,hci_uart,rfcomm
		bcmdhd                793075  0
		cfg80211              430729  1 bcmdhd



b.  **Add TSLIB support or evdev Support**

In order to X11 accepts touch screen you will need **TSLIB** support or **EVDEV** support.

You can follow this for **TSLIB**: https://github.com/avafinger/pine64-touchscreen

Change/Add the file **xorg.conf** for something like this:



		# Bpi M64 TS - no need for calibration with TSLIB
		# no need for uvdev if using TSLIB
		Section "InputClass"
			Identifier "M64-Touchscreen"
		#	MatchIsTouchscreen "on"
			MatchDevicePath "/dev/input/event*"
			MatchProduct "ft5x_ts"
			Driver "tslib"
		#	Option "Mode" "Absolute"
		EndSection



or use evdev


Initial setup
-------------

1.  DHCP is activated by default

2.  Eth0 is not managed, if you connect using Wifi and later wish to get back
    to DHCP you must issue a ifdown and ifup command to renew DHCP.

3.  Output mode is HDMI 1080p@60 , to change it to 720p you need to generate a new DTB
    and set it to 720p or any other HDMI mode inside the DTB.




mini FAQ (Ubuntu Xenial 16.04)
------------------------------

1.  How to update the distro

    a.  **Use command line:**

            sudo apt-get update
            sudo apt-get dist-upgrade
            sync


2.  How to change keyboard layout

    a.  **Use command line:**

            sudo dpkg-reconfigure keyboard-configuration


3.  How to change timezone

    a.  **type in command line:**

            timedatectl list-timezones
            sudo timedatectl set-timezone desired_timezone
            sudo timedatectl set-timezone America/New_York


4.  How to change language

    Install your language package and generate new locale.


5.  How to play MP4 videos ( MPEG1, MPEG2, MPEG4 )

    Follow libvdpau-sunxi and install LIBVDPAU and instal mpv
    

6.  How to install Ubuntu MATE

            sudo apt-get update
            sudo apt-get dist-upgrade
            sync
            sudo apt-get install ubuntu-mate-desktop


7.  How to change Hostname

    Suppose you want to change the hostname to "myhostname":

    a.  **edit the file /etc/hostname and change the contents to: myhostname**

            sudo leafpad /etc/hostname
            change contents to: myhostname

    b.  **edit the file /etc/hosts and change or add the contents**

            sudo leafpad /etc/hosts
            change/add the following

            127.0.0.1	localhost
            127.0.1.1	myhostname

            # The following lines are desirable for IPv6 capable hosts
            ::1     ip6-localhost ip6-loopback
            fe00::0 ip6-localnet
            ff00::0 ip6-mcastprefix
            ff02::1 ip6-allnodes
            ff02::2 ip6-allrouters

    c.  **sync and reboot**

            sync
            sudo reboot


7.  How to play with the Leds (led1=Blue and led2=Green)

    a.  **LEDs Blue and Green are accessible in /sys/class/leds/**

            sudo cat /sys/class/leds/led1/trigger

    b.  **To disable or change the LED activity**

            sudo su (password: ubuntu)
            echo "none" > /sys/class/leds/led1/trigger

    c.  **To enable**

            echo "default-on" > /sys/class/leds/led1/trigger

    d.  **To trigger heartbeat again**

            echo "heartbeat" > /sys/class/leds/led1/trigger

8.  HDMI Digital sound output or Headphone JACK Analog stereo sound output

    a.  **Edit the file as below for JACK sound output**

            sudo leafpad /etc/asound.conf

    Write:

		pcm.!default {
		  type hw
		  card 1
		  device 0
		}
		ctl.!default {
		  type hw
		  card 1
		}
  

    b.  **Copy the file a64-2GB.dtb_analog_sound_output to /media/ubuntu/boot/a64/ a64-2GB.dtb**


    c.  **Edit the file /etc/modules and add**

		sunxi_codec
		sunxi_i2s
		sunxi_sndcodec
	

    **reboot**

    Install alsa-tools and alsa-base:

		sudo apt-get install alsa-base alsa-tools alsa-utils

    Check the sound cards:

		aplay -l
	
		**** List of PLAYBACK Hardware Devices ****
		card 0: sndhdmi [sndhdmi], device 0: SUNXI-HDMIAUDIO sndhdmi-0 []
		  Subdevices: 1/1
		  Subdevice #0: subdevice #0
		card 1: audiocodec [audiocodec], device 0: SUNXI-CODEC codec-aif1-0 []
		  Subdevices: 1/1
		  Subdevice #0: subdevice #0
		card 1: audiocodec [audiocodec], device 1: bb Voice codec-aif2-1 []
		  Subdevices: 1/1
		  Subdevice #0: subdevice #0
		card 1: audiocodec [audiocodec], device 2: bb-bt-clk codec-aif2-2 []
		  Subdevices: 1/1
		  Subdevice #0: subdevice #0
		card 1: audiocodec [audiocodec], device 3: bt Voice codec-aif3-3 []
		  Subdevices: 1/1
		  Subdevice #0: subdevice #0

    d.  Better audiocodec support

	- Install alsa-base and run alsamixer and unmute audiocodec (JACK)
	- For some weird thing alsa-utils breaks audiocodec, if you have problems with alsactl,
	  try to remove /etc/asound.conf (item **a.**) and revise alsamixer settings
	- In my experiments, i had to purge alsa-tools alsa-utils and restart installing againg the alsa
	  type: sudo apt-get purge alsa-tools alsa-utils alsa-base
	  reboot
	  re-install sudo apt-get install alsa-tools alsa-utils alsa-base and run alsamixer to unmute JACK.


Troublehooting
--------------

1.  Nothing on my LCD/Monitor TV display

    a.  **Make sure the HDMI conector is well connected to the board**

    b.  **Make sure your LCD/Monitor TV is 1080P capable, the board will boot in HDMI 1080P mode only**

    c.  **Don't use DVI to HDMI**

    d.  **Press HDMI connector with your finger after boot and see you images pops up on screen**

2.  Moving windows on screen is slow

    **Install Metacity over OpenBox**

3.  Camera is not working

    a.  **Make sure you connect the sensor**

    b.  **The camera connector at the board side is really horrible, you need to make sure the FPC cable touch the contacts (They should change it on next board revision)**

    c. **Make sure you have the correct DTB (Device Tree Blob) with camera enabled, please note the a64-2GB.dtb_leds has camera disbled to allow full Leds control

4.  Board does not boot

    a.  **Watch for some signs during normal boot**

        - Wait a few seconds, BROM waits for FEL button and then proceed loading
        
        - Mouse should blink twice

        - Ethernet led connector will blink and show some activities after login

        - Your monitor will switch to HDMI 1080p

        - Led Blue will show a heartbeat activitiy and Green Led will light when eMMC is in use (kernel 3.10.105 only)


    b.  **If you don't see any of this signs during boot, you most likely have run into the following**

        - Bad SD card even if does not show bad track or errors, try with another brand and size

        - PSU Under power or under voltage, use a good PSU, at least 2.5A and 5v output guarantee (**DCIN**)


    c.  **Check for SD card integrity**

        unmount the SD CARD fisrt: 
        sudo umount /dev/sdX (where X is your SD card letter [b,c..])
        sudo fsck.vfat -a /dev/sdX1 (where X is your SD card letter [b,c..])
        sudo fsck.ext4 -f /dev/sdX2  (where X is your SD card letter [b,c..])


    d. **DO NOT** Power the board with microUSB, use the DCIN (power barrel)



**Have i missed something or you found wrong or improper information/language grammar or spelling?**

    Just let me know and i fix it ASAP

    
*** WIP ***

History Log:
===========
* initial commit (readme file)
* Add support for LCD 7" and Touch
* Alternative LCD7 pwm
* Tocuh Enabled