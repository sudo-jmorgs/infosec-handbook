# infosec-handbook (Best read in raw)
Collection of bits and bobs that help me in a variety of contexts; work, dev, red, blue and osint. 

I use linux a bit so these notes are mainly for me so I don't have to rediscover everything when I need a thing and partly for you.

**Missing GPT key when running $apt update **

Message
Sub-process /usr/bin/sqv returned an error code (1), error message is: Missing key 97B32012EA1176F053727A95C048F0B49DEEC457, which is needed to verify signature.
W: An error occurred during the signature verification. The repository is not updated and the previous index files will be used. GPG error: https://downloads.metasploit.com/data/releases/metasploit-framework/apt lucid InRelease: Sub-process /usr/bin/sqv returned an error code (1), error message is: Missing key 97B32012EA1176F053727A95C048F0B49DEEC457, which is needed to verify signature.
W: Failed to fetch http://downloads.metasploit.com/data/releases/metasploit-framework/apt/dists/lucid/InRelease  Sub-process /usr/bin/sqv returned an error code (1), error message is: Missing key 97B32012EA1176F053727A95C048F0B49DEEC457, which is needed to verify signature.

Fix
Add the key manually
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 97B32012EA1176F053727A95C048F0B49DEEC457 (Substitute in the missing key key)

or if apt-key is deprecated on your linux device
curl -fsSL https://raw.githubusercontent.com/rapid7/metasploit-framework/master/RELEASE-GPG-KEY.asc | sudo gpg --dearmor -o /usr/share/keyrings/metasploit.gpg (Substitute in the missing framework repositoty - might take a bit of looking)
sudo apt update

If the repo is dead or depricated (per above) remove the repo from the sources or apt wont stop complaining. (You'll have to subsitute in the framework for the package your wanting to replace. For me it was metasploit-framework)
sudo rm /etc/apt/sources.list.d/metasploit-framework.list 2>/dev/null
sudo sed -i '/metasploit/d' /etc/apt/sources.list
sudo apt update


**What service/ports am I running and other basics**

I use multiple devices between work and home. To find out (remind myself) what each is running I use the following necessary Linux admin utilities.

sudo systemctl status _servicename_
Tells me if the service is running, enabled after reboot, is installed or disabled etc.

netstat -tulpn
or
ss -tulpn
what ports are listening and on what interface

you see a port number listening but dont know what process is it
└─$ sudo ss -tulpn | grep 9392
[sudo] password for xxx:
tcp   LISTEN 0      4096         0.0.0.0:9392       0.0.0.0:*    users:(("gsad",pid=1011,fd=4))

Shows we're running gsad and pid 1011. 

where pid = 1011
ps -p 1011 -o pid,ppid,user,cmd
    PID    PPID USER     CMD
   1011       1 _gvm     /usr/sbin/gsad --foreground --listen 0.0.0.0 --port 9392
we are running gsad from /usr/sbin/gsad under user _gvm.

top
Realtime quick glance at what processes are running also, which ones maybe crushing a machine cpu or memory

ps aux --sort=-%cpu | head
Similar to top but returns to promtp. What processes are consuming resources. 

ip a
network interfaces, ip addresses etc

linux start up logs
sudo journalctl -b

Start up error logs
sudo journalctl -b -p err

Which services slowed down boot time
systemd-analyze blame

Kernal boot messages
dmesg

filter for errors
dmesg | grep -i error

