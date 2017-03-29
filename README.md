# German DVB-T2 (HEVC) with Odroid-C2 and Si2168 based USB Stick

## 1. Preamble

### New DVB-T2 HEVC standard

Since DVB-T LiveTV is switched off in early 2017 in Germany and replaced by DVB-T2, it is time to prepare a setup that can handle the new format. So far I used Libreelec with a [Si2168 chipset](https://www.silabs.com/products/video/demodulator/Pages/Si2168.aspx) based USB-stick on a Raspberry Pi 2 in a setup like this:

``DVB-T Signal -> si2168 USB-stick -> RPI 2 -> TV``

The new DVB-T2 is based on the h265/HEVC codec, which needs far more resources for decoding than the DVB-T h264 codec. All current Raspberry models are unable to decode h265/HEVC in realtime.  

After some research I decided to go for the [Odroid-C2](http://www.hardkernel.com/main/products/prdt_info.php), since it has h265/HEVC support and can handle the DVB-T2 stream. I intended to simply replace the RPI2 with the Odroid-C2 like so:

``DVB-T2 signal -> si2168 USB-stick -> Odroid-C2 -> TV``

### Old Odroid-C2 kernel

But it turnded out to be more complex than this. A major pitfall is, that the Odroid-C2 runs on an old 3.14 kernel. And the si2168 chipset is [only supported from kernel 3.19 onwards](https://linuxtv.org/wiki/index.php/DVB-T2_USB_Devices).

~~I tried my luck with the custom built [LibreELEC 7.1.0 - media_build edition](http://forum.odroid.com/viewtopic.php?f=144&t=22887) on the Odroid-C2, since it has drivers for the si2168 chipset. But it uses Kodi 16 and produces only [choppy DVB-T2 playback](http://www.kodinerds.net/index.php/Thread/52568-ODroid-C2-TVHeadend-HD-LiveTV) which is supposedly fixed with Kodi 17.~~

*__Update:__ There are now Kodi Krypton based LibreELEC editions available in the [Odroid forum](http://forum.odroid.com/viewforum.php?f=144) which allow using a si2168 based USB stick directly connected to the Odroid-C2 and produce flawless DVB-T2 playback.*

So I was left with the options to

* spend more money and buy a different USB-stick which is supported by the old kernel or has Userspace drivers
* find a way to somehow get the DVB-T2 signal from the si2168 stick into the Odroid-C2 which runs a LibreELEC Alpha with Kodi 17

I didn't want to spend more money, so I opted for the latter. Since the RPI2 runs on a more recent kernel (4.4) it can handle the si2168 chipset. So I decided to reuse it in my DVB-T2 setup, and install the TVHeadend server on the RPI2, which would send the DVB-T2 stream to the TVHeadend client on the Odroid-C2:

``DVB-T2 signal -> si2168 USB-stick -> TVHeadend server (RPI2) -> TVheadend client (Odroid-C2) -> TV``

### Hopefully just a temporary workaround

Hopefully one day there will be a newer kernel for the Odroid-C2 (or a new LibreELEC edition with si2168 drivers and Kodi 17), so i can run my originally intended setup, with the stick directly connected to the Odroid-C2. There was an announcement for a 4.4 kernel by Hardkernel in March 2016, but nothing has been heard of it since. And then surprisingly there was Odroid-C2 support introduced with the mainline 4.7 kernel, but it is rather rudimentary so far. So there is at least [hope for the future](http://rglinuxtech.com/?p=1880).

### Only public TV stations

Since the private stations will be encrypted on DVB-T2, it is only possible to receive the public broadcast stations (like ARD,ZDF, Arte, ...) with this setup. I don't watch the private stations anyway, so I am fine with this. I did no further research on how to deal with encrypted content.

## 2. New DVB-T2 Setup

### 2.1 Used Equipment

* si2168 based USB-stick (+ antenna)
* Raspberry Pi (could be anything with a recent kernel and TVHeadend server)
* Odroid-C2

### 2.2 RPI2 (TVHeadend server) setup

The si2168 USB-stick is connected to the RPI2.

It runs a basic headless Raspbian with a 4.4 kernel (like e.g. the [raspbian-ua-netinst](https://github.com/debian-pi/raspbian-ua-netinst)).

#### TVHeadend server on RPI2

It is necessary to run a recent TVHeadend server from the unstable branch. Since the official TVHeadend repository only offers stable builds for the armhf platform, I used [this repository](https://tvheadend.org/boards/5/topics/21528?r=23476) instead. As described I did:

```
 sudo nano /etc/apt/sources.list
 ```
 and added
 ```
 deb https://dl.bintray.com/djbenson/deb wheezy unstable
 ```
then installed the repository key
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
```
and the apt-https protocoll
```
sudo apt-get install apt-transport-https
```

and finally
```
$ sudo apt-get update
$ sudo apt-get install tvheadend
```

Then there are still the firmwares

* dvb-demod-si2168-b40-01.fw
* dvb-tuner-si2158-a20-01.fw

for the si2168 chipset missing...

As described [here](https://www.linuxtv.org/wiki/index.php/Geniatech_T230#Firmware) I did

```
$ cd /lib/firmware
$ sudo wget https://github.com/OpenELEC/dvb-firmware/raw/master/firmware/dvb-demod-si2168-b40-01.fw
$ sudo wget https://github.com/OpenELEC/dvb-firmware/raw/master/firmware/dvb-tuner-si2158-a20-01.fw
```

#### TVHeadend server setup

I connected to the TVHeadend server at port 9981 from a browser like so:

``http://<RPI2-IP-ADRESSE>:9981``

Then I ran a basic setup as described [here](https://tvheadend.org/boards/4/topics/21148). It includes adding a custom mux using these settings:

```
frequency (Hz): 650000000 (depends on your location)
Bandwidth: 8Mhz
Constellation: QAM/AUTO
Delivery System: DVBT2
```

But be aware that every place in Germany has it's own frequency. You can look up the frequency for your location ~~[here](http://www.dehnmedia.de/?page=dvbt2&subpage=demokanal)~~ [here](http://www.dehnmedia.de/?page=sender&subpage=tsender)

### 2.3 Odroid-C2 (TVHeadend client) setup

* create a SD card using the [LibreELEC USB-SD Creator](https://wiki.libreelec.tv/index.php?title=LibreELEC_USB-SD_Creator). Make sure to choose the latest ~~7.90 Alpha~~ release when creating the SD card!  

* After basic setup of LibreELEC on the Odroid-C2, install the TVHeadend HTSP Client to be found under ``Add-Ons > Download Add-Ons > LibreELEC Add-ons > PVR Clients``

* configure the installed TVHeadend Client and replace the `127.0.0.1` IP address with the IP of the TVHeadend server (the RPI2 in my case).

That should be it! I got very good DVB-T2 playback on the Odroid-C2 now.
