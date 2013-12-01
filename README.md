# Raspberry Pi - VPN Client/Forwarder

## Setup
This setup was used with Raspbian, the image provided with the [NOOBS](http://www.raspberrypi.org/downloads) install will work fine. For the VPN Service, I'm using [AirVPN](https://airvpn.org/) for the VPN Service. Once Raspbian is installed just follow the following steps:

1. sudo apt-get install openvpn
2. Add/Uncomment **net.ipv4.ip_forward=1** to **/etc/sysctl.conf** this allows forwarding of network traffic on boot.
3. Create startup scripts for iptables, first file needs to be [**/etc/iptables.up.rules**](iptables.up.rules)
4. Create a file **/etc/network/if-pre-up.d/iptables** with the line **/sbin/iptables-restore < /etc/iptables.up.rules** this will ensure that each time the system boots, the firewall settings are applied.
5. Create a directory under **/var/opt/** called **AirVPN** the full path should be **/var/opt/AirVPN/**
6. Login to your AirVPN Account and go to the [Config Generator](https://airvpn.org/generator/), Select Linux for your OS, Select the server you would like to connect to, Now select the **Advanced Mode** tick box, Click the port you would like to connect to the server on (I use **Direct, protocol UDP, port 443**) then tick **Separate keys/certs from .ovpn file**. Then click **Generate** once you've agreed to the Terms of service.
7. Download the resulting zip or tar file and extract it's contents into **/var/opt/AirVPN/**, there should be 4 files. Rename the *.ovpn* file to **AirVPN.ovpn** (this is so the service script in the next step can find it).
8. Create service script in [**/etc/init.d/airvpn**](airvpn) and chmod it 0755
9. Test the script works by running **sudo service airvpn start** then run **ifconfig** to see if there's a new adaptor called tun0 with an ip address of 10.x.x.x this means you are connect to the VPN service. You can also try running **curl http://checkip.dyndns.org** and see if it returns the VPN Server's IP Address.
10. If the previous step was successful, run **update-rc.d airvpn defaults** this ensures the VPN connects and starts up at boot. If the previous step didn't work look in **/var/log/syslog**, there should OpenVPN messages in there listed as the **airvpn** daemon. You can see my sample connection log under [examplesyslog](examplesyslog)

## Routing Traffic though the RPi
Now you'll probably want to route certain traffic or all traffic via your RPi, there are two levels you can do this at and different ways to do it. Now as a test we should try to access the AirVPN Speedtest page available at http://10.4.0.1/ but to do this we either need to update the routing table on you computer or we can do it a router level so every device on the network can access it. Lets do it on the computer to start. First get the local network IP of your RPi in this example we'll use 192.168.1.7

If you're using Windows you do the following:
1. Open a Command Prompt as Administrator.
2. Type **route add 10.0.0.0 mask 255.0.0.0 192.168.1.7**
3. TODO

## Notes
In addition to the above setup I also decided to make the filesystem read-only to reduce the chance of corruption in the case of power-loss. It also means I can just unplug the RPi without worrying about a fsck running on startup. You can find a guide here on this topic on the [RaspberryPi Forums](http://www.raspberrypi.org/phpBB3/viewtopic.php?p=213440). Whenever I want to edit a file on the filesystem I just run **sudo mount -o remount,rw /**
