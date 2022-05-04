# Portable Pi Router

This repo documents the process of configuring a Raspberry Pi Zero W as a portable router, capable of chanelling wifi signals to our device over ethernet (or more suitably, ethernet over USB). For this project, I've used OpenWRT to run the Pi as a Router, instead of configuring Raspbian itself. OpenWRT has insanely low resource requirements and runs phenominally on the Zeros - it also has a great interface and removes all the bloat we would have with Raspbian! OpenWRT is running Linux under the hood which means I immediately liked it, but also means all of our normal Raspbian commands will work in the terminal. Most of the steps are pretty standard across OpenWRT disributions so you should be able to apply these steps to most Pi's (I've tried to mark the ones specific to the Zero). 

Originally I was just keen to play around with OpenWRT before overhauling my home network, but after getting aquainted with it on my Pi 4 I realised that I could set up a Zero as a super-portable, usb-powered router and got going with this project over the weekend! Towards the end of the project, the algorithms blessed me with [Network Chuck's](https://www.youtube.com/watch?v=jlHWnKVpygw&list=LL&index=7&t=1194s&ab_channel=NetworkChuck) video doing a similar thing, and I just had to add the VPN functionality too! I've included the instructions here to add the VPN but his video is really worth watching and following for that feature - you won't regret it! I should also note that all credit for the VPN addition goes to him - I just followed those steps and documented them here basically. I also added the ad-blocker on after once I'd shopped through the packages on offer, so this functionality is really easy to plug on and remove at the end of it all if you're not happy with the performance after adding it. 

After following these instructions you should have a Pi Zero router that can act as a gateway between your device and a public wifi network. We will have conifigured our gateway to run a VPN for some privacy and an ad-blocker to give it a little more functionality (although both of these are optional additions so no stress if you're not keen for those features). With this, you can pretty confidently and privately make use of public networks without exposing your actual device :)

## Requirements
1) Pi Zero W: I used a Pi Zero W and I'm sure this will work with a Zero W 2, too! Once the Pi shortage passes and I can get one in my hands I will test this out and update this repo with the results.
2) (optional) Pi Zero USB Stem: I used this so that I can just plug the Pi straight into my laptop instead of carrying a microUSB-USB adapter. Unfortunately I haven't found a usb-c stem yet, so if anyone reading this knows of one please let me know! I used [this one](https://www.sparkfun.com/products/14526) - you can get it in South Africa through [PiShop](https://www.pishop.co.za/store/pi-zero-usb-stem), [MicroRobotics](https://www.robotics.org.za/KIT-14526), or [DIY Electronics](https://www.diyelectronics.co.za/store/breakout-boards/2537-raspberry-pi-zero-usb-adapter-usb-a-connector-for-pi-zero.html?search_query=zero+usb&results=36). I believe that there are also solderless options - so have a quick Google for those if you don't have an iron or are uncomfortable soldering :)

## Bake your firmware

Alright, it's time to get started! So the first thing we want to do is load the OpenWRT firmware instead of Raspbian. [OpenWRT](https://openwrt.org/) have a [tool](https://chef.libremesh.org/?version=21.02.2&target=bcm27xx%2Fbcm2708&id=rpi) to configure your distribution and bake in packages beforehand. Make sure you have selected "Raspberry Pi B/B+/CM/Zero/ZeroW" and whatever version you would like to run (I used 21.02.2 as that was the latest version when I did this). Then, under the 'Customize' menu, we're going to specify what packages we would like to include - select all under 'Custom package selection' and paste this there instead:

```
base-files bcm27xx-gpu-fw busybox ca-bundle cypress-firmware-43430-sdio cypress-nvram-43430-sdio-rpi-zero-w dnsmasq dropbear e2fsprogs firewall fstools ip6tables iptables iwinfo kmod-brcmfmac kmod-fs-vfat kmod-ipt-offload kmod-nls-cp437 kmod-nls-iso8859-1 kmod-sound-arm-bcm2835 kmod-sound-core kmod-usb-hid libc libgcc libustream-wolfssl logd luci mkf2fs mtd netifd odhcp6c odhcpd-ipv6only opkg partx-utils ppp ppp-mod-pppoe procd ucert uci uclient-fetch urandom-seed wpad-basic-wolfssl kmod-usb-gadget-eth kmod-usb-dwc2 kmod-usb-net kmod-mii kmod-usb-core kmod-nls-base kmod-usb-net-cdc-ether kmod-usb-net-rndis luci-app-openvpn openvpn-openssl adguardhome
```
And hit 'Request Build'. For some reason, including openvpn and adguardhome sometimes gave me issues here, if this is the case you can use this package list instead and we'll add those in later:
```
base-files bcm27xx-gpu-fw busybox ca-bundle cypress-firmware-43430-sdio cypress-nvram-43430-sdio-rpi-zero-w dnsmasq dropbear e2fsprogs firewall fstools ip6tables iptables iwinfo kmod-brcmfmac kmod-fs-vfat kmod-ipt-offload kmod-nls-cp437 kmod-nls-iso8859-1 kmod-sound-arm-bcm2835 kmod-sound-core kmod-usb-hid libc libgcc libustream-wolfssl logd luci mkf2fs mtd netifd odhcp6c odhcpd-ipv6only opkg partx-utils ppp ppp-mod-pppoe procd ucert uci uclient-fetch urandom-seed wpad-basic-wolfssl kmod-usb-gadget-eth kmod-usb-dwc2 kmod-usb-net kmod-mii kmod-usb-core kmod-nls-base kmod-usb-net-cdc-ether kmod-usb-net-rndis
```
Then just download the image (I think I used 'Factory (EXT4)' but the one you choose shouldn't make a difference).The extra packages we have here will allow us to configure ethernet over USB on the Pi Zero, and are required for us to set the micro-usb port as a network interface. In the case that you're going to be baking another image, these are the additional packages that I've added on top of the default selection:
* kmod-usb-gadget-eth 
* kmod-usb-dwc2 
* kmod-usb-net 
* kmod-mii 
* kmod-usb-core 
* kmod-nls-base 
* kmod-usb-net-cdc-ether 
* kmod-usb-net-rndis 
* luci-app-openvpn 
* openvpn-openssl 
* adguardhome

With the image downloaded, it's time to flash it to our SD card. Plug the SD card for your Pi into your computer and flash the downloaded image to it using whatever flash utility you'd like (I have always used [Balena Etcher](https://www.balena.io/etcher/), so if you haven't got a preference I can recommend you use that). Your image should flash in a couple of seconds - it is ridiculously fast because of how small OpenWRT is!

## Housekeeping

Okay now you can plug your Pi in and get into it's terminal. I used a keyboard and monitor for this but you should be able to SSH, too. I'm not going to add the instructions to set up SSH here, but [this video](https://www.youtube.com/watch?v=_pBf2hGqXL8&ab_channel=DevOdyssey) does a great job of it if you need it. Remember that the OpenWRT OS does not have a GUI itself, but the router does. So even if you do use a monitor, you will only have access to a command line :)

With access to the command-line, we are going to do some general housekeeping. First, lets enable the wireless interface on our Pi by entering the following commands:

```
uci set wireless.radio0.disabled=0
```
To enable the wireless interface,
```
uci commit
```
To commit these changes, and
```
reboot
``` 
To reboot the Pi and ensure that the changes are loaded. 

Next up, we want to set a root/admin password for the router because it is unprotected right now! To do this, enter:
```
passwd
```
And enter your password/follow the prompts. Note that this is the password for the router admin, this is what we will use to log into the GUI whenever we want to connect the router to a new network and will give you full access to the Pi itself (and your VPN credentials if you add that functionality) - so make sure it's a secure one!

Next, let's jump into the boot directory and enable SSH if we haven't already. Enter the directory with
```
cd boot
```
and enable SSH with 
```
touch ssh
```

## Enable the USB network interface (specific to the Pi Zero W)

Next, we're going to configure the USB port as a network interface. This allows us to see the Pi as an ethernet device over the same port we use to provide power to the Pi.

If you're not still in the '/boot' diretory, go back there with
```
cd boot
```

Then we will want to edit the 'config.txt' file. Unfortunately, we only have Vim to edit files :( Enter the file with:
```
vi config.txt
```
scroll to the bottom of the file (after "Place your custom settings here"), hit 'i' to insert text, and add the following line:
```
dtoverlay=dwc2
``` 
Then, to save and exit hit 'esc' followed by ':wq' to 'write' the changes and 'quit' editing this file.

Next, and in the same directory, we want to edit the 'cmdline.txt' file. Enter the file with:
```
vi cmdline.txt
```
and scroll the the end of the line (after 'rootwait'), hit 'i' to insert, add a space and add the following line:
```
modules-load=dwc2,g_ether
```
Then exit with 'esc' followed by ':wq' again.

Now move back a directory with
```
cd ../
```
and enter the following commands:
```
modprobe g_ether
```
```
modinfo g_ether
```
```
echo g_ether > /etc/modules.d/56-g_ether
```
and finally
```
ls /etc/modules.d
```
to confirm that 56-g_ether is in the directory.

Let's quickly reboot the device to make sure that these changes have been made. Reboot with
```
reboot
```
and once the device has booted, if you enter 
```
ifconfig usb0
```
you should see that the interface is now available! 

## Register the network interface
With the interface visible, we are now ready to register it as a network interface. Note that if you are using a Wirelss dongle or a normal Pi (3/4) you can pick up from here, just take not of the interface name when you enter 
```
ifconfig
```

Let's change into the config directory with
```
cd /etc/config
```
Now before we start cooking up our own configurations, we're going to back up the config files that we're working on so that we can roll back to them if we make a whoopsie. Run the following three commands to do this:
```
cp network network.orig
```
```
cp wireless wireless.orig
```
```
cp firewall firewall.orig
```

Then let's enter the network configuration with
```
vi network
```
and change the lan interface ip address to 10.71.71.1 (or whatever address you'd like to assign this interface actually). then save and close the file with 'esc' and ':wq' again.

Next, let's jump into the firewall configuration with 
```
vi firewall
``` 
and under the 'wan' zone, change 'option input' to 'ACCEPT'. Save and quite with 'esc' and ':wq'

While we're here, let's give our Pi a hostname so that we don't have to use the IP address every time we want to connect to a new network. Enter the dhcp config with
```
vi /etc/config/dhcp
```
and add the following lines to the end
```
config domain
	option ip '10.71.71.1'
	option name 'router.home'
```
You can set the name to anything you would like, I just used a generic name here but why not make it something fun. Also make sure the the IP address here is the same as the one you assigned to your lan interface a couple of steps above!

Now we can reboot the Pi again to make sure that our updated configurations are loaded properly:
```
reboot
```

Once the Pi has rebooted you should be able to see the network as 'OpenWRT' and you should also be able to connect to the network too (although you won't have internet access yet)! 

## Configure the network interface

Connect to your Pi's network (OpenWRT) and, in your browser, connect to the GUI using the IP address you set (10.71.71.1) or the hostname you used (router.home). Once here, log in as admin with the password you used right at the beginning.

Now you're in the GUI, have a little explore around here if you'd like, it's a really nice interface and there is some great functionality built in!

When you're ready to carry on, go into '>network>interfaces' to configure our new USB interface. Click on 'add new interface' and name in WAN, use the DHCP client protocol, and select the usb0 interface that we've just added (or the appropriate interface if you're not using the Zero). Finally, in the firewall settings, set he zone setting to wan (the red one).

Now you have a wireless router! If you connact an ethernet cable from your home router to the USB port (using an appropriate adapter) you will have internet acces through the Pi's wifi network. But we want things to be the other way around. So while we're on the interfaces tab, we're going to switch things around. Edit the LAN configuration so taht it is assigned usb0 device, and edit the WAN interface configuration so that it is assigned to the br-lan (bridge wide-area network) device.

Now hit save and apply, and everything should be working as inteded! We could do this through command line too, but there is a lot of potential to make a mistake here and to forget something, so instead I opted to use the web (Luci) interface. If you plug the USB stem (or microUSB to USB cable) from the Pi into your laptop, it should show up as an ethernet connection. Now if you log into the Pi using the interface you can scan and connect to any network available in the area! This Pi will now act as an additional router between your device and the router providing the internet :)

## Adding a VPN
