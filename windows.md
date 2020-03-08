The official ['Google Coral Getting Started Guide'](https://coral.ai/docs/dev-board/get-started/) lists a Linux or Mac computer
as a requirement. I don't have a Linux or Max computer. To begin development today, I planned to setup a Coral development environment
on a Mint Linux VM (to be replaced with an old Thinkpad running linux in the future). I had issues connecting to the
Coral board through the VM and, after realizing the scope of the hacks I'd need to get it working, decided hacking a Window's solution
would be a better alternative. Currently, there's not a Window's alternative to the Google's official Getting Started Guide, so I've put
one together below.

*Note*: this is a rough guide. Some details have been left out (such as how to add a folder to your PATH variable), but they can be 
easily found elsewhere with a quick search.

**Pre-requisites**
* python
* pip
* PuTTY or an alternate SSH / serial connection tool

**Setup**
1. Install PuTTY or an alternate SSH / serial tool.
2. Install fastboot by downloading the Android SDK Platform Tools and moving the fastboot executable to a folder listed in your system's
PATH variable. (Other tools in the download can be deleted.) Ensure fastboot has been successfully added to your system path by running

> fastboot --version

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;in CMD.

3. Install the Mendel Development Tools with pip install --user mendel-development-tool
4. Download the CP210x USB to UART driver for Silicon Labs USB-to-Serial converters (to be installed during process). 
[USB to UART Bridge VCP Driver link](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)
5. Download Enterprise Mendel software to your downloads folder and unzip by typing

> cd ~/Downloads
> curl -O https://dl.google.com/coral/mendel/enterprise/mendel-enterprise-day-13.zip
> unzip mendel-enterprise-day-13.zip

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;in CMD.

**Flashing Board**
1. Connect to board's serial port to the host PC.
2. Open Device Manager, expand the Ports tab, and note the COM port of the Coral Board (which should be listed as a serial device). Leave the Device Manager open for now.
3. Open PuTTY, select 'Serial' as connection type, enter the correct COM port, set the speed to 115200, and click 'Open'. The terminal should be completely blank at this point because the board hasn't been powered on.
4. Connect the board's power port to power with a *wall charger*. **Do not attempt to power the board from your PC.**
5. Connect the board's OTG data port to the host PC.
6. In the Device Manager, find the device names "USB Download Gadget" or similar (should be listed in the Universal Serial Bus devices flyout menu). Right click the device, select "Update Driver", then "Browse my computer for driver software", then "Let me pick", and finally select "WinUSB Device" from the left column and "ADB Device" from the right. This will correctly configure Coral's data connection on Windows.
7. Back in the PuTTY serial interface, type:

  > fastboot 0

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;and press enter.

8. Navigate to the Enterprise Mendel Linux folder in CMD (~\Downloads\mendel-enterprise-day-13 if following this guide exactly).
9. The flash.sh board flashing script cannot be executed directly on Windows, so we will enter each flash command one-at-a-time. Type these commands sequentially in CMD.

> fastboot flash bootloader0 u-boot.imx
> fastboot reboot-bootloader
> fastboot flash gpt partition-table-8gb.img
> fastboot reboot-bootloader
> fastboot erase misc
> fastboot flash boot boot_arm64.img
> fastboot flash rootfs rootfs_arm64.img
> fastboot reboot

*Note*: After the reboot, you will be able to login to the Coral via serial with user: mendel, password: mendel

**Connecting via shell**
1. Access the board via serial and run:

> hostname -I

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;and note the returned IP address (192.168.101.2 seems common).

*Note*: If the IP address begins with 192.168.100, steps 2 and 3 are not immediately necessary. That said, the board's IP is not strictly static and could change to begin with 192.168.10X (where X is any integer), which would break some of the MDT tool's functionality. You might as well complete steps 2 and 3 just in case.
2. Open {pythonpath}\Lib\site-packages\mdt\sshclient.py in your favorite text editor.
3. Edit line 86 from

> if not self.address.startswith('192.168.100'):

  to:

> if not self.address.startswith('192.168.10'):

*Note*: this edit prevents the MDT tool from throwing a 'NonLocalDeviceError' when connecting to IP addresses not in the 192.168.100 prefix. The error looks like this when it occurs:

> It looks like you're trying to connect to a device that isn't connected
to your workstation via USB and doesn't have the SSH key this MDT generated.
To connect with `mdt shell` you will need to first connect to your device
ONLY via USB.
>
> Cowardly refusing to attempt to push a key to a public machine.

4. In CMD, run

> mdt shell {ip}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;where {ip} is completely replaced with the IP address you noted in step 1. e.g.

> mdt shell 192.168.101.2

5. Login with the default auth info:

> user: mendel
> password: mendel

Congrats, you now have a Coral board shell in Windows! The MDT tool takes care of generating a SSH key pair to establish a secure shell between the host PC and Coral board. Once this has been done, you can use any SSH tool (PuTTY, for example) to connect to the Coral board **if you point the SSH tool to the correct SSH key**. This guide will be updated to include these details in the future.

References:
* [Window's Guide for older version of the Coral Board](https://blog.questionable.services/article/coral-edge-tpu-windows/) (before an SSH key pair was required to establish a shell connection)
* [OG reference to the NonLocalDeviceError hack](https://stackoverflow.com/questions/58938727/unable-to-connect-google-coral-using-otg-port)
