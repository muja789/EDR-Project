**Project:**

This lab is dedicated to simulating a real cyber attack and endpoint
detection and response. Utilizing Eric Capuano\'s guide online, I will
be using virtual machines to simulate the threat & victim machines. The
attack machine will utilize \'Sliver\' as a C2 framework to attack a
Windows endpoint machine, which will be running \'LimaCharlie\' as an
EDR solution.

**Eric Capuano\'s Guide:**
[[https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro]{.underline}](https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro)?

utm_campaign=post&utm_medium=web

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image13.png){width="5.90625in"
height="3.648005249343832in"}

**VM Installation:**

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

**Setup:**

Lets setup the windows machine. At first installing sysmon.

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image1.png){width="5.40625in"
height="2.6155632108486437in"}

Created a free LimaCharlie account and logged in. Lets setup LimaCharlie

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image19.png){width="7.28125in"
height="3.4411439195100613in"}

Adding a sensor named Windows-VM using the add sensor button,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image29.png){width="8.5in"
height="3.913665791776028in"}

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image16.png){width="5.371461067366579in"
height="3.2842082239720036in"}

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image11.png){width="6.248090551181102in"
height="1.4442311898512685in"}

Now installing the sensor on the endpoint Windows-10_VM

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image20.png){width="5.958333333333333in"
height="3.439773622047244in"}
![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image22.png){width="6.390625546806649in"
height="3.1666666666666665in"}

Setting up Artifact Collection Rule from the Artifact Collection option

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image30.png){width="9.442708880139982in"
height="1.7083333333333333in"}

Naming the rule "windows-sysmon-logs"

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image2.png){width="5.8681321084864395in"
height="1.80332895888014in"}

For the pattern option choosed the sysmon option for collecting only the
sysmon logs

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image15.png){width="6.026042213473316in"
height="4.041666666666667in"}

The setup of LimaCharlie is completed here.

Now lets setup the Sliver C2 in the attacker(ubuntu) machine,

Below are the commands:

wget
https://github.com/BishopFox/sliver/releases/download/v1.5.34/sliver-server_linux
-O /usr/local/bin/sliver-server

\# Make it executable

chmod +x /usr/local/bin/sliver-server

\# install mingw-w64 for additional capabilities

apt install -y mingw-w64

Created a directory as well named sliver.

**The Attack and Defense:**

Lets generate the C2 payload,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image3.png){width="5.59375in"
height="3.8108333333333335in"}

Confirm the new implant configuration,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image10.png){width="7.661458880139983in"
height="1.042972440944882in"}

Now the payload is generated, need to drop it on the windows machine.
Lets spin up a temporary web server to transfer the payload to windows
machine.

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image23.png){width="6.633122265966755in"
height="1.3581167979002624in"}

Now starting a sliver listener,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image21.png){width="7.994792213473316in"
height="1.1580763342082239in"}

Verify the session and get the session id,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image26.png){width="8.072916666666666in"
height="0.8722233158355206in"}

Start interacting with the session,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image8.png){width="6.53125in"
height="0.8125in"}

Checking the privileges here,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image27.png){width="7.7603379265091865in"
height="3.5195581802274716in"}

Identify running processes on the remote system using "ps -T",

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image12.png){width="3.4375in"
height="0.9421839457567804in"}
![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image18.png){width="3.3634995625546806in"
height="0.8796839457567804in"}

Notice that Sliver cleverly highlights its own process in green and any
detected countermeasures (defensive tools) in red.

On the host machine we can look inside our LimaCharlie SIEM and see
telemetry from the attacker. We can identify the payload thats running
and see the IP its connected to.

On the LimaCharlie hop into the sensor we created earlier,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image17.png){width="6.942708880139983in"
height="3.144035433070866in"}

On the "Processes" tab check for the malicious process,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image32.png){width="9.0in"
height="0.9126957567804025in"}

One of the easiest ways to spot unusual processes is to simply look for
ones that are NOT signed.

We can check the network connection of this process as well,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image4.png){width="7.130208880139983in"
height="1.2916666666666667in"}
![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image7.png){width="9.463542213473316in"
height="0.9040376202974628in"}

Here we can see the destination ip is our attacker machine ip.

We can also use LimaCharlie to scan the hash of the payload through
VirusTotal,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image24.png){width="7.774071522309711in"
height="2.388907480314961in"}

From the "Timeline" tab we can see the telemetry that we generated,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image33.png){width="8.877791994750655in"
height="3.5511165791776027in"}

Now on the attack machine we can simulate an attack to steal credentials
by dumping the "LSASS" memory.

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image9.png){width="5.6875in"
height="0.65625in"}

In LimaCharlie we can check the sensors, observe the
telemetry,![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image14.png){width="9.18334864391951in"
height="1.704753937007874in"}

Start building a detection rule based on this event,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image25.png){width="7.182292213473316in"
height="1.8102504374453194in"}

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image31.png){width="7.197916666666667in"
height="4.287402668416448in"}

Now let's test our rule against the event we built it for,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image28.png){width="3.45874343832021in"
height="1.5016951006124235in"}
![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image5.png){width="3.5833333333333335in"
height="1.3350284339457568in"}

Again run the attack to steal credentials but now with the detection,

![](vertopal_fa1d1801ea58461baf81334156427f0e/media/image6.png){width="13.072916666666666in"
height="0.59375in"}

Like this, we can practice using LimaCharlie to write a rule that will
detect and block the attacks coming from the Sliver server.
