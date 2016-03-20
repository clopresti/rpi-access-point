Instructions to compile and install kernel driver for rtl8192eu based WiFi dongle on the Raspberry Pi

First connect the USB wifi adapter and check if the dongle is recognized by running
lsusb

If you see ID 0bda:818b Realtek Semiconductor Corp.
Then the adapter is recognized and it uses the rtl8192eu chipset


Make sure your Pi is up to date and has build tools installed
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential

Get the driver source on your Pi
wget https://raw.githubusercontent.com/clopresti/rpi-access-point/master/rtl8192eu-driver.zip
sudo unzip rtl8192eu-driver.zip

Get the rpi-source script used to download the Pi kernel source
wget https://raw.githubusercontent.com/clopresti/rpi-access-point/master/rpi-source
chmod +x rpi-source 
sudo mv rpi-source /usr/bin/
sudo rpi-source -q --tag-update

Install the Pi kernel
sudo rpi-source --skip-gcc

Build and install the driver
sudo make ARCH=arm
sudo make ARCH=arm install
sudo bash -c 'echo "options 8192eu rtw_power_mgnt=0 rtw_enusbss=0" > /etc/modprobe.d/8192eu.conf'
modprobe 8192eu

You should now be able to use your WiFi adapter
sudo ifup wlan0

---------------------------

Instructions to turn the Pi into a WiFi access point using rtl8192eu based WiFi dongle

First make sure you can use the rtl8192eu based WiFi dongle in client mode using the above instructions

Install hostapd
sudo apt-get install hostapd

create hostapd configuration file
sudo nano /etc/hostapd/hostapd.conf

Add the following text
interface=wlan0
driver=rtl871xdrv
ssid=PiAccessPoint
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=3
wpa_passphrase=password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

Save the file Ctrl-O Enter and Exit Ctrl-X

You can change the ssid and wpa_passphrase values to what you want and channel can be changed to any value between 1 and 14

Tell the Pi where to find this configuration file.
sudo nano /etc/default/hostapd

Find the line 
#DAEMON_CONF="" 
and edit it so it says 
DAEMON_CONF="/etc/hostapd/hostapd.conf"
Don't forget to remove the # in front of the lin
Save the file Ctrl-O Enter and Exit Ctrl-X


Compile and build version of hostapd that works with rtl8192eu

wget https://github.com/clopresti/rpi-access-point/raw/master/RTL8188C_8192C_USB_linux_v4.0.2_9000.20130911.zip
sudo unzip RTL8188C_8192C_USB_linux_v4.0.2_9000.20130911.zip
cd RTL8188C_8192C_USB_linux_v4.0.2_9000.20130911/
cd wpa_supplicant_hostapd/
sudo tar -xvf wpa_supplicant_hostapd-0.8_rtw_r7475.20130812.tar.gz
sudo cp -rf wpa_supplicant_hostapd-0.8_rtw_r7475.20130812 /home/pi/wpa_supplicant_hostapd

cd /home/pi/wpa_supplicant_hostapd
cd hostapd

Build and install hostapd
sudo make
sudo make install

Move the new hostapd into the right directory
sudo mv /usr/sbin/hostapd /usr/sbin/hostapd-orig
sudo mv hostapd /usr/sbin/hostapd
sudo chown root.root /usr/sbin/hostapd
sudo chmod 755 /usr/sbin/hostapd


Edit network interfaces
sudo nano /etc/network/interfaces

Comment out all lines relating to wlan0 by placing '#' at the start of the lines
Then add the following lines at the end of the file:

allow-hotplug wlan0
iface wlan0 inet static
address 192.168.42.1
netmask 255.255.255.0

Save the file Ctrl-O Enter and Exit Ctrl-X


Assign a static IP address to the wifi adapter by running 
sudo ifconfig wlan0 192.168.42.1