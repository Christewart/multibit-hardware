Status: [![Build Status](https://travis-ci.org/bitcoin-solutions/mbhd-hardware.png?branch=master)](https://travis-ci.org/bitcoin-solutions/mbhd-hardware)

### Project status

Alpha: Expect bugs and API changes. Not suitable for production, but early adopter developers should get on board.

### MultiBit Hardware

MultiBit HD (MBHD) supports hardware wallets through a common API which is provided here under the MIT license.

Hardware wallet implementers can free themselves of the burden of writing their own wallet software by using MBHD
or they can use the API provided here to create their own and take advantage of the code and utilities provided.

One example of a supported hardware wallet is the Trezor and full examples and documentation is available for review.

### Technologies

* [Java HID API](https://code.google.com/p/javahidapi/) - Java library providing USB Human Interface Device (HID) native interface
* [Google Protocol Buffers](https://code.google.com/p/protobuf/) (protobuf) - for the most efficient and flexible wire protocol
* [Google Guava](https://code.google.com/p/guava-libraries/wiki/GuavaExplained) - for excellent Java support features
* Java 7+ - to remove dependencies on JVMs that have reached end of life

### Frequently asked questions (FAQ)

#### Why Google Protocol Buffers ?

We believe that the use of Google Protocol Buffers for "on the wire" data transmission provides the best balance between an efficient
binary format and the flexibility to allow a wide range of other languages to access the device. Having your device speak "protobuf"
means that someone can rip out the `.proto` files and use them to generate the appropriate accessor code for their language (e.g. Ruby).

#### Do you support git submodules for `.proto` files ?

Yes. While you can have treat your module within the MultiBit Hardware project as the single point of (Java) reference, you don't have to.

You are also free to offer up your `.proto` files as common libraries that remain in your repo and under your control. We then include
 them as a [git submodule](http://git-scm.com/book/en/Git-Tools-Submodules). This is the approach taken by the Trezor development team
 and enables them to retain complete control over the wire protocol, while allowing MultiBit Hardware to implement support in a phased
 approach.

#### Why no listeners ?

The Guava library offers the [EventBus](https://code.google.com/p/guava-libraries/wiki/EventBusExplained) which is a much simpler way to
 manage events within an application. Since MultiBit HD uses this internally and will be the primary consumer of this library it makes
 a lot of sense to mandate its use.

#### How do I get my hardware included ?

While we welcome a wide variety of hardware wallet devices into the Bitcoin ecosystem supporting them all through MultiBit HD gives rise 
to some obvious problems:

* maintaining compatibility for legacy versions (variants, quirks, deprecated functionality etc)
* verifying the security of the wallet (robust enough for mainstream users, entropy source etc)
* simplicity of use (mainstream users must find it easy to obtain and use) 

Thus the current situation is that the MultiBit Hardware development team is only supporting the Trezor device, but in time we would like to
open up support for other leading hardware wallet vendors.

### Getting started

MultiBit Hardware uses the standard Maven build process and can be used without having external hardware attached. Just do the usual

```
$ cd <project directory>
$ mvn clean install
```

and you're good to go.

#### Collaborators and the protobuf files

If you are a collaborator (i.e. you have commit access to the repo) then you will need to perform an additional stage to ensure you have
the correct version of the protobuf files:

```
$ cd <project directory>
$ git submodule init
$ git submodule update
```
This will bring down the `.proto` files referenced in the submodules and allow you to select which tagged commit to use when generating
the protobuf files. See the "Updating protobuf files" section later.

MultiBit Hardware does not maintain `.proto` files other than for our emulator. Periodically we will update the protobuf files through
 the following process (assuming an update to the Trezor protobuf):
```
$ cd trezor/src/main/trezor-common
$ git checkout master
$ git pull origin master
$ cd ../../..
$ mvn -DupdateProtobuf=true clean compile
$ cd ..
$ git add trezor
$ git commit -m "Updating protobuf files for 'trezor'"
$ git push
```
We normally expect the HEAD of the submodule origin master branch to [represent the latest production release](http://nvie.com/posts/a-successful-git-branching-model/), but that's up to the
owner of the repo.

The new protobuf files  

#### Read the wiki for detailed instructions

Have a read of [the wiki pages](https://github.com/bitcoin-solutions/mbhd-hardware/wiki/_pages) which gives comprehensive
instructions for a variety of environments. 

#### Listing devices on a USB

Different operating systems have different methods of listing the devices attached to the USB. Here is a useful list:

* `lsusb` for Linux
* `system_profiler SPUSBDataType` for Mac
* `reg query hklm\system\currentcontrolset\enum\usbstor /s` for Windows (untested so might be a better way)

### Working with a production Trezor device (recommended)

After [purchasing a production Trezor device](https://www.buytrezor.com/) do the following (assuming a completely new :

Plug in the device to the USB port and wait for initialisation to complete.

Verify that the device is listed on the USB using one of the commands given above. An output similar to the following
should be seen:

```
TREZOR:
Product ID: 0x0001
Vendor ID: 0x534c
Version:  1.00
Serial Number: some random number
Speed: Up to 12 Mb/sec
Manufacturer: SatoshiLabs
Location ID: 0x3a200000 / 2
Current Available (mA): 500
Current Required (mA): 100
```

Attempt to discover the device using the `UsbMonitoringExample` through the command line not the IDE:
```
cd examples
mvn exec:java -Dexec.mainClass="org.multibit.hd.hardware.examples.trezor.usb.UsbMonitoringExample"
```

This will list available devices on the USB and select a Trezor if present.

A production Trezor uses an [ARM 32F205RE microcontroller](http://www.ic-on-line.cn/view_online.php?id=1820919&file=0411\stm32f205zc_4719102.pdf).

### Working with a Raspberry Pi emulation device

A low-cost introduction to the Trezor is the use of a Raspberry Pi and the Trezor Shield development hardware available from [Satoshi Labs](http://satoshilabs.com/news/2013-07-15-raspberry-pi-shield-for-developers/).

#### Configuring a RPi for socket emulation

Note your RPi's IP address so that you can open an SSH connection to it.

If you're using a laptop enable DHCP and plug in the device's network cable.

On Unix you can issue the following command to map the local network (install with brew for OSX):
```
$ nmap 192.168.0.0/24
```
Replace the IP address range with what IP address your laptop

Change the standard `rpi-serial.sh` script to use the following:

```
$ python trezor/__init__.py -s -t socket -p 0.0.0.0:3000 -d -dt socket -dp 0.0.0.0:2000
```
This will ensure that the Shield is serving over port 3000 with a debug socket on port 2000.

**Warning** Do not use this mode with real private keys since it is unsafe!

Run it up with
```
$ sudo ./rpi-serial.sh
```

You should see the Shield OLED show the Trezor logo.

#### Configuring a RPi for USB emulation

Apply power to the RPi through the USB on the Trezor Shield board. Connect a network cable as normal.

After the blinking lights have settled, test the USB device is connected as described in the earlier section.

You should see a CP2110 HID USB-to-UART device.

Establish the IP address of the RPi and SSH on to it as normal.

Use the standard `rpi-serial.sh` script but ensure that RPi Getty is turned off:
```
$ sudo nano /etc/inittab
$ #T0:23:respawn:/sbin/getty -L ttyAMA0 115200 vt100
```
This will ensure that the Shield is using the UART reach the RPi GPIO serial port with a debug socket on port 2000.

**Warning** Do not use this mode with real private keys since it is unsafe!

Run it up with
```
sudo ./rpi-serial.sh
```
You should see the Shield OLED show the Trezor logo.

### Troubleshooting

The following are known issues and their solutions or workarounds.

#### When running the examples I get errors indicating `iconv` is missing or broken

The `iconv` library is used to map character sets and is usually provided as part of the operating system. MultiBit Hardware
will work with version 1.11+. We have seen problems with running the code through an IDE where `iconv` responds with a failure
code of -1.   

#### On OS X I get the "Access denied (insufficient permissions)" from LibUsb

This is a known issue with devices that announce themselves as HID devices and thus are automatically locked by the operating
system. Attempting to unlock them from with the user space occupied by `libusb` will fail, even as root.

One workaround is to create a custom `Info.plist` file and place it in `/System/Library/Extensions/Trezor.kext/Contents/`

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<!-- This is a dummy driver which binds to Trezor.       -->
<!-- It contains no actual code; its only purpose is to  -->
<!-- prevent Apple's USBHID driver from exclusively      -->
<!-- opening the device.                                 -->
<plist version="1.0">
<dict>
	<key>CFBundleDevelopmentRegion</key>
	<string>English</string>
	<key>CFBundleIconFile</key>
	<string></string>
	<key>CFBundleIdentifier</key>
	<string>com.satoshilabs.dummy</string>
	<key>CFBundleInfoDictionaryVersion</key>
	<string>6.0</string>
	<key>CFBundlePackageType</key>
	<string>KEXT</string>
	<key>CFBundleSignature</key>
	<string>????</string>
	<key>CFBundleVersion</key>
	<string>1.0.0d1</string>
	<key>IOKitPersonalities</key>
	<dict>
		<!-- The Trezor V1 USB interface -->
		<key>TrezorV1</key>
		<dict>
			<key>CFBundleIdentifier</key>
			<string>com.apple.kpi.iokit</string>
			<key>IOClass</key>
			<string>IOService</string>
			<key>IOProviderClass</key>
			<string>IOUSBInterface</string>
			<key>bConfigurationValue</key>
			<integer>1</integer>
			<key>bInterfaceNumber</key>
			<integer>0</integer>
			<key>idProduct</key>
			<integer>1</integer>
			<key>idVendor</key>
			<integer>21324</integer>
		</dict>
	</dict>
	<key>OSBundleLibraries</key>
	<dict>
		<key>com.apple.iokit.IOUSBFamily</key>
		<string>1.8</string>
	</dict>
</dict>
</plist>
```

### Closing notes

All trademarks and copyrights are acknowledged.
