# Portable Pi Router

This repo documents the process of configuring a Raspberry Pi Zero W as a portable router, capable of chanelling wifi signals to our device over ethernet (or more suitably, ethernet over USB). For this project, I've used OpenWRT to run the Pi as a Router, instead of configuring Raspbian itself. OpenWRT has insanely low resource requirements and runs phenominally on the Zeros - it also has a great interface and removes all the bloat we would have with Raspbian! OpenWRT is running Linux under the hood which means I immediately liked it, but also means all of our normal Raspbian commands will work in the terminal. Most of the steps are pretty standard across OpenWRT disributions so you should be able to apply these steps to most Pi's (I've tried to mark the ones specific to the Zero).

Originally I was just keen to play around with OpenWRT before overhauling my home network, but after getting aquainted with it on my Pi 4 I realised that I could set up a Zero as a super-portable, usb-powered router and got going with this project over the weekend! Towards the end of the project, the algorithms blessed me with [Network Chuck's](https://www.youtube.com/watch?v=jlHWnKVpygw&list=LL&index=7&t=1194s&ab_channel=NetworkChuck) video doing a similar thing, and I just had to add the VPN functionality too! I've included the instructions here to add the VPN but his video is really worth watching and following for that feature - you won't regret it! I should also note that all credit for the VPN addition goes to him - I just followed those steps and documented them here basically. I also added the ad-blocker on after once I'd shopped through the packages on offer, so this functionality is really easy to plug on and remove at the end of it all if you're not happy with the performance after adding it.

After following these instructions you should have a Pi Zero router that can act as a gateway between your device and a public wifi network. We will have conifigured our gateway to run a VPN for some privacy and an ad-blocker to give it a little more functionality (although both of these are optional additions so no stress if you're not keen for those features). With this, you can pretty confidently and privately make use of public networks without exposing your actual device :)

Also a quick disclaimer - the screenshots taken below were taken with a Raspberry Pi 4 running this setup so the interface is called eth0 instead of usb0. I'm a little clumsy and in removing the SD card from my Zero to reflash and do a final test of these instructions I ripped the solder pads holding the SD card reader off :confused: The instructions have been tested and these steps worked on the Zero, I was just doing that last run through as a proof read really - so I'm pretty confident everything is working. Once I've had luck with the repair (or I can get my hands on some more Zeros) I plan to update the images :)

## Requirements

1. Pi Zero W: I used a Pi Zero W and I'm sure this will work with a Zero W 2, too! Once the Pi shortage passes and I can get one in my hands I will test this out and update this repo with the results.
2. (optional) Pi Zero USB Stem: I used this so that I can just plug the Pi straight into my laptop instead of carrying a microUSB-USB adapter. Unfortunately I haven't found a usb-c stem yet, so if anyone reading this knows of one please let me know! I used [this one](https://www.sparkfun.com/products/14526) - you can get it in South Africa through [PiShop](https://www.pishop.co.za/store/pi-zero-usb-stem), [MicroRobotics](https://www.robotics.org.za/KIT-14526), or [DIY Electronics](https://www.diyelectronics.co.za/store/breakout-boards/2537-raspberry-pi-zero-usb-adapter-usb-a-connector-for-pi-zero.html?search_query=zero+usb&results=36). I believe that there are also solderless options - so have a quick Google for those if you don't have an iron or are uncomfortable soldering :)

## Bake your firmware

Alright, it's time to get started! So the first thing we want to do is load the OpenWRT firmware instead of Raspbian. [OpenWRT](https://openwrt.org/) have a [tool](https://chef.libremesh.org/?version=21.02.2&target=bcm27xx%2Fbcm2708&id=rpi) to configure your distribution and bake in packages beforehand. I've included three pre-baked packages in under [Images](https://github.com/NicholasBunn/PortableRouter/blob/main/Images). One just contains the packages required for the router, one includes OpenVPN, and the last includes OpenVPN and AdGuardHome. You can use any of the three regardless of what functionality you are adding, so I would recommend just using the one with all three if you're using these. They are all running version 21.01.2 of the "Raspberry Pi B/B+/CM/Zero/ZeroW" build. If, however, you choose to bake your own, when you're using the tool make sure you have selected "Raspberry Pi B/B+/CM/Zero/ZeroW" (if you are building for a Zero) and whatever version you would like to run (I used 21.02.2 as that was the latest version when I did this). Then, under the 'Customize' menu, we're going to specify what packages we would like to include - select all under 'Custom package selection' and paste this there instead:

```
base-files bcm27xx-gpu-fw busybox ca-bundle cypress-firmware-43430-sdio cypress-nvram-43430-sdio-rpi-zero-w dnsmasq dropbear e2fsprogs firewall fstools ip6tables iptables iwinfo kmod-brcmfmac kmod-fs-vfat kmod-ipt-offload kmod-nls-cp437 kmod-nls-iso8859-1 kmod-sound-arm-bcm2835 kmod-sound-core kmod-usb-hid libc libgcc libustream-wolfssl logd luci mkf2fs mtd netifd odhcp6c odhcpd-ipv6only opkg partx-utils ppp ppp-mod-pppoe procd ucert uci uclient-fetch urandom-seed wpad-basic-wolfssl kmod-usb-gadget-eth kmod-usb-dwc2 kmod-usb-net kmod-mii kmod-usb-core kmod-nls-base kmod-usb-net-cdc-ether kmod-usb-net-rndis luci-app-openvpn openvpn-openssl adguardhome
```

And hit 'Request Build'. For some reason, including openvpn and adguardhome sometimes gave me issues here, if this is the case you can use this package list instead and we'll add those in later:

```
base-files bcm27xx-gpu-fw busybox ca-bundle cypress-firmware-43430-sdio cypress-nvram-43430-sdio-rpi-zero-w dnsmasq dropbear e2fsprogs firewall fstools ip6tables iptables iwinfo kmod-brcmfmac kmod-fs-vfat kmod-ipt-offload kmod-nls-cp437 kmod-nls-iso8859-1 kmod-sound-arm-bcm2835 kmod-sound-core kmod-usb-hid libc libgcc libustream-wolfssl logd luci mkf2fs mtd netifd odhcp6c odhcpd-ipv6only opkg partx-utils ppp ppp-mod-pppoe procd ucert uci uclient-fetch urandom-seed wpad-basic-wolfssl kmod-usb-gadget-eth kmod-usb-dwc2 kmod-usb-net kmod-mii kmod-usb-core kmod-nls-base kmod-usb-net-cdc-ether kmod-usb-net-rndis
```

Then just download the image (I think I used 'Factory (EXT4)' but the one you choose shouldn't make a difference as long as it's a factory option).The extra packages we have here will allow us to configure ethernet over USB on the Pi Zero, and are required for us to set the micro-usb port as a network interface. In the case that you're going to be baking another image, these are the additional packages that I've added on top of the default selection:

1. For the Pi Zero

- kmod-usb-gadget-eth
- kmod-usb-dwc2
- kmod-usb-net
- kmod-mii
- kmod-usb-core
- kmod-nls-base
- kmod-usb-net-cdc-ether
- kmod-usb-net-rndis

2. For the VPN setup

- luci-app-openvpn
- openvpn-openssl

3. For the ad-blocker

- adguardhome

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

This section is only required if you're using a Pi Zero, if you're using a Pi 3/4, jump ahead to registering the network interface.

Next, we're going to configure the USB port as a network interface. This allows us to see the Pi as an ethernet device over the same port we use to provide power to the Pi.

If you're not still in the **/boot** diretory, go back there with

```
cd boot
```

Then we will want to edit the **config.txt** file. Unfortunately, we only have Vim to edit files :( so enter the file with:

```
vi config.txt
```

scroll to the bottom of the file (after "Place your custom settings here"), hit **'i'** to insert text, and add the following line:

```
dtoverlay=dwc2
```

Then, to save and exit hit **'esc'** followed by **':wq'** to _write_ the changes and _quit_ editing this file.

Next, and in the same directory, we want to edit the **cmdline.txt** file. Enter the file with:

```
vi cmdline.txt
```

and scroll the the end of the line (after **rootwait**), hit **'i'** to insert, add a space and add the following line:

```
modules-load=dwc2,g_ether
```

Then exit with **'esc'** followed by **':wq'** again.

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

to confirm that **56-g_ether** is in the directory.

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

With the interface visible, we are now ready to register it as a network interface. Note that if you are using a Wirelss dongle or a normal Pi (3/4) you can pick up from here, just take note of the interface name when you enter. If you would like to use some other interface like a Wireless dongle, these will need to be set up independently, following a similar approach as was done in the previous section. [Network Chuck's](https://www.youtube.com/watch?v=jlHWnKVpygw&list=LL&index=7&t=1194s&ab_channel=NetworkChuck) video explains the process pretty well and you should be able to get going with most interfaces following his instructions.

```
ifconfig
```

If you're still in the boot directory, head back to the root with:

```
cd ../
```

Now that we're all on the same page, let's change into the config directory with

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

and change the lan interface ip address to **10.70.70.1** (or whatever address you'd like to assign this interface actually). Remember that in Vim if you want to edit the file you will have to insert by pressing **'i'**. You can then save and close the file with **'esc'** and **':wq'** again.

Next, let's jump into the firewall configuration with

```
vi firewall
```

and under the **wan** zone, change both **option input** and **option forward** to **ACCEPT**. Save and quit with **'esc'** and **':wq'**

While we're here, let's give our Pi a hostname so that we don't have to use the IP address every time we want to connect to a new network. Enter the dhcp config with

```
vi dhcp
```

and add the following lines to the end

```
config domain
	option ip '10.70.70.1'
	option name 'router.home'
```

You can set the name to anything you would like, I just used a generic name here but why not make it something fun. Also make sure the the IP address here is the same as the one you assigned to your lan interface a couple of steps above!

Now we can reboot the Pi again to make sure that our updated configurations are loaded properly:

```
reboot
```

Once the Pi has rebooted you should be able to see the network as **OpenWRT** and you should also be able to connect to the network too (although you won't have internet access yet)!

## Configure the network interface

Connect to your Pi's network (**OpenWRT**) and, in your browser, connect to the GUI using the IP address you set (**10.70.70.1**) or the hostname you used (**router.home**). Once here, log in as admin with the password you set right at the beginning and you should end up at the page shown below.

![Overview/Landing page](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/Overview.png)

Now you're in the LuCI GUI, have a little explore around here if you'd like, it's a really nice interface and there is some great functionality built in (I personally thought the signal strength analyser/channel mapper - **Status>Channel Analysis** was really cool)! Also, while you're on the home page, take note of how little memory is being used!

When you're ready to carry on, go into >**Network**>**Interfaces** to reconfigure the existing interfaces. When you arrive, the page should look like the one below:

![Interaces page](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/Interfaces.png)

Here we are going to change the existing interface so that it is assigned to **usb0** instead of **br-lan**. Note that in the above image (the original configuration) the **LAN** interface is assigned to **br-lan**. Hit **'Edit'** and change this assignment so that it is assigned to **usb0** instead (note that in the image below it shows **eth0** because I took these screenshots on a Pi 4 setup so the interface already existed, but these steps are the same).

![Updated interface configuration](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/InterfaceConfiguration.png)

This is just switching things up so that we are using the 'ethernet' port as our router's 'output' instead of the Wifi chip. Now you can hit **'Save and apply'** and the interface should become unresponsive. This is expected thought because the Wifi interface that we have been connected over has just been changed! Plug the Pi into your main machine so that we can access it through the USB/eth and everything should be working as inteded!

We could do this through command line too, but there is a lot of potential to make a mistake here and to forget something, so instead I opted to use the web (Luci) interface. If you plug the USB stem (or microUSB to USB cable) from the Pi into your laptop, it should show up as an ethernet connection. Now if you log into the Pi using the interface you can scan and connect to any network available in the area! To do this, go into **Network>Wireless** and hit **Scan** to find all the available networks and connect to the one you'd like to connect to (this would be the public network that you'd like to use the internet from).

![Wireless Overview](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/Wireless.png)

Enter the password when the popup presents itself and under network assign it the **wan** firewall (the red one).

![Join Wireless Network](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/JoinNetwork.png)

Next, assign both **wwan** (**wan**) and **lan** to the interface - as is shown below.

![Wireless Configuration](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/WirelessConfiguration.png)

You can also **Disable** (and **Remove**) the OpenWRT interface as you won't be exposing this device over the wireless interface anymore.

Hit **'Save and Apply'** to commit all of these updates and you Pi will now act as an additional router between your device and the router providing the internet! :) You can use the same process of connecting to networks when you move to new public networks from now on

## Adding a VPN

Time to add a VPN to our router for a bit of extra privacy (this step is optional, so skip it if you don't have a VPN)! Just to re-iterate here, I've literally just documented the steps taken in [Network Chuck's video](https://www.youtube.com/watch?v=jlHWnKVpygw&list=LL&index=7&t=1194s&ab_channel=NetworkChuck) here, you may have a better experience following his video and using this repo to copy and paste codeblocks.

But in the case that you choose to follow these instructions instead - let's get into it :)

Firstly, I really recommend SSHing into the Pi for these next steps as these commands are so easy to type wrong so I'd rather copy and paste. Also, we will need to SSH soon anyways so may as well get going now. Remember that if you're pasting into a terminal window you use '**ctrl**' + '**shift**' + '**v**' instead of the normal '**ctrl**' + '**v**'! If you've plugged the Pi into you main machine, you can SSH from the terminal using:

```
sudo ssh root@10.70.70.1
```

Then hit yes if you get some fingerprint notices and enter the appropriate passwords when you're prompted for it.

Cool, now to _actually_ get started with the VPN stuff

Before we do anything else, we need to add a VPN interface, so enter the network file again using:

```
vi /etc/config/network
```

And add an openvpn network by adding the following to the end of this file:

```
config interface 'vpnclient'
	option ifname 'tun0'
	option proto 'none'
```

Remember, to add text in Vim you need to hit **'i'**, and to save and quite hit **'esc'** followed by **':wq'**.

Now, on another device we need to find your VPN server. I use NordVPN so I can use [this link](https://nordvpn.com/servers/tools/) to get mine - just find the server that your VPN offers. On the Nord site, click 'show available protocols' and download the OpenVPN UDP config.

Now we are going to open two terminal sessions on your main machine (the one that you would SSH into the Pi with - this should be the one that you donwloaded the OpenVPN UDP config on).

In the first session, SSH into the Pi and run the following command:

```
mdkir /etc/openvpn
```

Then, in the second session (the one not SSHed into the Pi) navigate into your downloads folder (**'cd Downloads'** on Linux) and enter the following command

```
scp nameOfDownloadedFile root@IPaddress:/etc/openvpn/client.conf
```

where **nameOfDownloadedFile** is the name of the OpenVPN UDP Config file you downloaded, and **IPaddress** is the IP address of your Pi (10.70.70.1). This will copy the downloaded config file over onto the Pi. You might have to sudo this, and you will have to enter your Pi's password too (the same one we used to SSH in and set wayyyyyy back at the beginning of this all).

Now, back in the SSH terminal session, we can install the packages required for OpenVPN if you were unsuccessful in baking them into the original image. Run the following

```
opkg update
```

```
opkg install luci-app-openvpn
```

```
opkg install openvpn-openssl
```

You can also install packages using LuCI, just go under **System>Software**. To update, hit **Update Lists** and then you can search and install the packages listed at the beginning of this repo under **'Download and install package'**.

![Package Management](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/Packages.png)

And then do a quick reboot with:

```
reboot
```

SSH back into the Pi now and once you're the terminal again, set your VPN configuration parameters with

```
OVPN_DIR="/etc/openvpn"
```

```
OVPN_ID="client"
```

```
OVPN_USER="USERNAME"
```

```
OVPN_PASS="PASSWORD"
```

Where **USERNAME** and **PASSWORD** are the username and password that you use to access you VPN account.

We are now going to save these credentials with the following (the following couple of steps are easiest to just copy and paste if you can):

```
umask go=
cat << EOF >${OVPN_DIR}/${OVPN_ID}.auth
${OVPN_USER}
${OVPN_PASS}
EOF
```

Then, we are going to configure out VPN service with

```
sed -i -e "
/^auth-user-pass/s/^/#/
\$a auth-user-pass ${OVPN_ID}.auth
/^redirect-gateway/s/^/#/
\$a redirect-gateway def1 ipv6
" ${OVPN_DIR}/${OVPN_ID}.conf
/etc/init.d/openvpn restart
```

And enable VPN management in the GUI with

```
ls /etc/openvpn/*conf | while read -r OVPN_CONF
do
OVPN_ID="$(basename ${OVPN_CONF%.*} | sed -e "s/\W/_/g")"
uci -q delete openvpn.${OVPN_ID}
uci set openvpn.${OVPN_ID}="openvpn"
uci set openvpn.${OVPN_ID}.enabled="1"
uci set openvpn.${OVPN_ID}.config="${OVPN_CONF}"
done
uci commit openvpn
/etc/init.d/openvpn restart
```

Now we want to just reconfigure the firewall so that our VPN works: run the following commands (TODO CHECK WHICH ONE WAS LAN AND WHICH WAS WAN)

```
uci rename firewall.@zone[0]="lan"
```

```
uci rename firewall.@zone[1]="wan"
```

```
uci del_list firewall.wan.device="tun+"
```

```
uci add_list firewall.wan.device="tun+"
```

```
uci commit firewall
```

```
/etc/init.d/firewall restart
```

And then configure Hotplug so that the VPN service isn't restarted when we lose WAN (otherwise this would happen whenever we move around!). Issue the following few commands:

```
mkdir -p /etc/hotplug.d/online
cat << "EOF" > /etc/hotplug.d/online/00-openvpn
/etc/init.d/openvpn restart
EOF
cat << "EOF" >> /etc/sysupgrade.cong
/etc/hotplug.d/online/00-openvpn
EOF
```

And your VPN should be running too! You should see a **VPN** tab next to **Network** now - which is where you can control VPN config through the GUI.

Now not only can you connect to networks through your Pi, but your device and privacy is hidden with your VPN! We can verify this by Googling "What is my IP address" and then Googling the public IP address that you are presented with. Mine was shown to belong to NordVPN (because that's the VPN service I use - so yours might be slightly different) :)

## Adding an ad-blocker

OpenVPN provides native support for running AdGuard Home as an ad-blocker - which means that you can run the ad-blocker on your router itself instead of running it on a seperate machine as you would with something like Pi-Hole! I haven't actually used Pi-Hole so I can't compare the performances, but I've enjoyed having AdGuard Home running so far!

So the first thing we want to do is update and install the AdGuard Home package if you weren't succesful in baking it in earlier. You can do this by running

```
opkg update
```

and

```
opkg install adguardhome
```

or by using LuCI (as was shown in the VPN section, above).

With the packages installed, we want to set the AdGuard Home service to start on boot with

```
service adguardhome enable
```

and then actually start it with

```
service adguardhome start
```

There is a way to set this up through command line, and it is documented [Here](https://openwrt.org/docs/guide-user/services/dns/adguard-home), but if we do it through the AdGuard GUI we get to check out all the cool monitoring features available!

Lets open up the interface by going to the below address in our browser. Replace **10.70.70.1** with whatever IP address you assigned earlier.

```
http://10.70.70.1:3000
```

You should arrive at a set up page, hit **'Get Started'** to begin. The first thing we'll do is set the Admin Web Interface to listen on 10.70.70.1 at port 8080 (instead of the default 80) - obviously change the IP address to reflect yours here. While we're on this page, also change the DNS server to listen on 10.70.70.1 at port 53. If you have an error telling you that port 53 is already in use - that means that DNSMasq is currently still running on the Pi. So back in the Pi's terminal (or on your SSH session) enter

```
service dnsmasq stop
```

and

```
service dnsmasq disable
```

and try the above steps again. Your page should look like this (without the 'bind: already in use' error):

![AdGuard Set Up - Page 2](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/AdguardConfig2.png)

You can now hit **'Next'**.

Create a username and password for your AdGuard server (it doesn't have to be the same as the one you use on your Pi, just make sure you remember it). and hit **'Next'**, again.

On the fourth page, you can leave AdGuard Home configured as a router (as that;s exactly how it's running). So just hit **'Next'** and **'Open Dashboard'**!

Enter the same credentials you just set for AdGuard Home and have a little explore around the interface! The GUI is also really cool if you like metrics, and it's quite interesting to have a look around once you've been using it for a short while.

![AdGuard Home Page](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/AdguardHome.png)

There are a whole bunch of things for you to configure here, such as your upstream DNS servers, rate limits, etc. but I'm not going to cover AdGuard's features in this guide! We are just going to do two more things for our configuration. I don't think these are strictly neessary as we will only have one device connected ata time most likely. But in the case that you have configured a Wifi dongle or are expecting multiple connections, this is good practice.

First, we are going to update Upstream DNS Server configuration so that it will accept LAN domain requests and pass the to OpenWRT itself. To do this, go to **Settings>DNS Settings>Upstream Servers** and now enter

```
[/lan/]127.0.0.1:53
[//]127.0.0.1:53
```

and hit **'Apply'**, below. You can also hit **'Test upstreams'** if you'd like to just verify things quickly.

![AdGuard DNS Settings](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/DNSSettings.png)

Next, we're going to run a Reverse DNS so that AdGuard Home picks up DHCP assignment from OpenWRT. On the same page (**Settings>DNS Settings**) scroll down until you get to **'Private reverse DNS servers'** and add

```
10.70.70.1:53
```

Also ensure that **'Use private reverse DNS resolvers'** and **'Enable reverse resolving of clients IP addresses'** are selected before hitting **'Apply'**

Finally, you can configure your filters under the **Filters** tab, to control which traffic you allow through. Go into **Filters>DNS blocklists** and scroll through to select some of the pre-configured lists. Hit **'Add blocklist'** to scroll through some more options, or to add your own!

![AdGuard DNS Blocklist](https://github.com/NicholasBunn/PortableRouter/blob/main/Figures/DNSBlocklists.png)

Just note that the more lists you use, the more memory is required. I selected all the available lists on my Zero and the memory usage did increase notably, however, it was still in the green and I didn't notice any performance issues :)

Now your ad-blocker should be working! Test it out by jumping onto a webiste that you find usually has a bunch of ads on it. Where ads used to be, there should be blank boxes as the element cannot be populaed by the ad anymore! Remember that this isn't going to work for things like Youtube ads as those ads are coming from a Google address, so they are not on our blocklist!

And that's that! You now have a portable, plug-and-play router that you can use to securely connect to public networks while being hidden by your VPN and shielded from most ads! Next up, I'm planning to add NginX as a reverse proxy so that we can resolve to 10.70.70.1:8080 using a hostname like 'router.adguard' - just need to find a couple of minuted outside of life to do that but it shouldn't be too far away!
