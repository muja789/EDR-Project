# EDR Home Lab: Attack and Defense
## Project:

This lab is dedicated to simulating a real cyber attack and endpoint
detection and response. Utilizing Eric Capuano\'s guide online, I will
be using virtual machines to simulate the threat & victim machines. The
attack machine will utilize \'Sliver\' as a C2 framework to attack a
Windows endpoint machine, which will be running \'LimaCharlie\' as an
EDR solution.

**Eric Capuano\'s Guide:**
Eric Capuano's Guide: https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro?utm_campaign=post&utm_medium=web

#

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image13.png" width="567" height="346" />

## VM Installation:

The first step is to install two machines. The attacker machine will run
on Ubuntu Server, and the endpoint will be running Windows 10. I am also
going to be installing Sliver on the Ubuntu machine as my primary attack
tool, and setting up LimaCharlie on the Windows machine as an EDR
solution. LimaCharlie will have a sensor linked to the windows machine,
and will be importing sysmon logs.

After installing all the machines need to update and upgrade the ubuntu
machine:

Command : sudo apt-get update && sudo apt-get upgrade.

Now I'm creating a Nat Network profile for this lab and making sure all
the machines are using this network.
#

## Setup:

Lets setup the windows machine. At first installing sysmon.

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image1.png" width="519" height="250" />

Created a free LimaCharlie account and logged in. Lets setup LimaCharlie

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image19.png" width="692" height="327" />

Adding a sensor named Windows-VM using the add sensor button,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image29.png" width="816" height="375" />
<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image16.png" width="509" height="308" />
<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image11.png" width="596" height="135" />

Now installing the sensor on the endpoint Windows-10_VM

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image20.png" width="567" height="327" />
<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image22.png" width="605" height="298" />

#

Setting up Artifact Collection Rule from the Artifact Collection option

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image30.png" width="903" height="164" />

Naming the rule "windows-sysmon-logs"

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image2.png" width="557" height="173" />

For the pattern option choosed the sysmon option for collecting only the
sysmon logs

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image15.png" width="578" height="388" />

The setup of LimaCharlie is completed here.
#

Now lets setup the Sliver C2 in the attacker(ubuntu) machine,

#
Below are the commands:

wget
https://github.com/BishopFox/sliver/releases/download/v1.5.34/sliver-server_linux
-O /usr/local/bin/sliver-server

\# Make it executable

chmod +x /usr/local/bin/sliver-server

\# install mingw-w64 for additional capabilities

apt install -y mingw-w64
#

Created a directory as well named sliver.

## The Attack and Defense:

Lets generate the C2 payload,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image3.png" width="529" height="365" />

Confirm the new implant configuration,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image10.png" width="730" height="100" />

Now the payload is generated, need to drop it on the windows machine.
Lets spin up a temporary web server to transfer the payload to windows
machine.

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image23.png" width="634" height="130" />
#

Now starting a sliver listener,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image21.png" width="768" height="111" />

Verify the session and get the session id,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image26.png" width="775" height="84" />

Start interacting with the session,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image8.png" width="624" height="77" />

Checking the privileges here,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image27.png" width="740" height="336" />

Identify running processes on the remote system using "ps -T",

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image12.png" width="327" height="91" />
<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image18.png" width="323" height="84" />

Notice that Sliver cleverly highlights its own process in green and any
detected countermeasures (defensive tools) in red.
#

On the host machine we can look inside our LimaCharlie SIEM and see
telemetry from the attacker. We can identify the payload thats running
and see the IP its connected to.

On the LimaCharlie hop into the sensor we created earlier,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image17.png" width="663" height="298" />

On the "Processes" tab check for the malicious process,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image32.png" width="864" height="87" />

One of the easiest ways to spot unusual processes is to simply look for
ones that are NOT signed.

We can check the network connection of this process as well,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image4.png" width="682" height="124" />
<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image7.png" width="903" height="87" />

Here we can see the destination ip is our attacker machine ip.

We can also use LimaCharlie to scan the hash of the payload through
VirusTotal,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image24.png" width="740" height="221" />

From the "Timeline" tab we can see the telemetry that we generated,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image33.png" width="845" height="336" />
#

Now on the attack machine we can simulate an attack to steal credentials
by dumping the "LSASS" memory.

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image9.png" width="538" height="63" />

In LimaCharlie we can check the sensors, observe the
telemetry,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image14.png" width="874" height="164" />
#

Start building a detection rule based on this event,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image25.png" width="682" height="173" />
<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image31.png" width="691" height="411" />

Now let's test our rule against the event we built it for,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image28.png" width="332" height="144" />
<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image5.png" width="344" height="128" />
#

Again run the attack to steal credentials but now with the detection,

<img src="https://github.com/muja789/EDR-Project/blob/main/EDR-Project/media/image6.png" width="1255" height="57" />

Like this, we can practice using LimaCharlie to write a rule that will
detect and block the attacks coming from the Sliver server.
#
