# Frequently Asked Questions

This list is to help solve a variety of issues that you may stumble across. Some of this information is also mentioned in the guides, but is here for quicker reference.

- [Frequently Asked Questions](#frequently-asked-questions)
  - [Where do I get the Pixel Streaming servers and frontend?](#where-do-i-get-the-pixel-streaming-servers-and-frontend)
  - [How do I install Vulkan on Linux?](#how-do-i-install-vulkan-on-linux)
  - [How do I install Pulse Audio on Linux?](#how-do-i-install-pulse-audio-on-linux)
  - [How do I install Nvidia Drivers?](#how-do-i-install-nvidia-drivers)
  - [Amazon Web Services](#amazon-web-services)
    - [Linux](#linux)
    - [Windows](#windows)
  - [Microsoft Azure](#microsoft-azure)
  - [How do I confirm my drivers are installed?](#how-do-i-confirm-my-drivers-are-installed)
  - [How do I connect to my instance on the cloud?](#how-do-i-connect-to-my-instance-on-the-cloud)
    - [Linux](#linux-1)
    - [Windows](#windows-1)
  - [I can't connect to my application running on Windows!](#i-cant-connect-to-my-application-running-on-windows)
  - [I can't install anything on my Windows Instance!](#i-cant-install-anything-on-my-windows-instance)
  - [How do I make my own instance image so I can skip all this setup?](#how-do-i-make-my-own-instance-image-so-i-can-skip-all-this-setup)
    - [Amazon Web Services](#amazon-web-services-1)
    - [Google Cloud Platform](#google-cloud-platform)
    - [Microsoft Azure](#microsoft-azure-1)
  - [When I try to connect to the instance through my web browser, nothing shows up/it times out!](#when-i-try-to-connect-to-the-instance-through-my-web-browser-nothing-shows-upit-times-out)
  - [Error received = Vulkan not supported?](#error-received--vulkan-not-supported)
  - [One person on the stream gets a bad connection, then everyone does!](#one-person-on-the-stream-gets-a-bad-connection-then-everyone-does)
  - [When I pixel stream on an AMD GPU the quality is unpredictable/intermittent.](#when-i-pixel-stream-on-an-amd-gpu-the-quality-is-unpredictableintermittent)
  - [I'm getting pixellation in my pre - 4.27 build pixel stream!](#im-getting-pixellation-in-my-pre---427-build-pixel-stream)
  - [The stream quality is bad!](#the-stream-quality-is-bad)
  - [Pixel streaming shows "Streamer not connected"](#pixel-streaming-shows-streamer-not-connected)
  - [How do I test my STUN/TURN Server?](#how-do-i-test-my-stunturn-server)
    - [STUN](#stun)
    - [TURN](#turn)
  - [TURN/relay candidates are not generated in Chrome?](#turnrelay-candidates-are-not-generated-in-chrome)
  - [Getting extra information out of Chrome](#getting-extra-information-out-of-chrome)
  - [How do I stop the Pixel Stream from capturing and hiding my mouse?](#how-do-i-stop-the-pixel-stream-from-capturing-and-hiding-my-mouse)
  - [Why does my Bitrate drop heavily on an Android/Samsung mobile connected to Wifi?](#why-does-my-bitrate-drop-heavily-on-an-androidsamsung-mobile-connected-to-wifi)


## Where do I get the Pixel Streaming servers and frontend?
As of 5.1, we've moved the Pixel Streaming front end and web servers to their own repository on GitHub.
This allows us to maintain more frequent updates of these elements, independant of the Unreal Engine release cadence. 

Infrastructure Github: https://github.com/EpicGames/PixelStreamingInfrastructure

**Note:** It's vital that you pull the branch that matches your Unreal Engine version (4.27 branch to 4.27 Unreal Engine)


## How do I install Vulkan on Linux?
`sudo apt install vulkan-utils`

## How do I install Pulse Audio on Linux?
`sudo apt-get install -y pulseaudio`

## How do I install Nvidia Drivers?

## Amazon Web Services
Note: If you're running on AWS, they provide instruction on how to install GRID drivers. We recommend this option as in our experience GRID drivers provide much better performance than the standard public drivers.

Please see the relevant guides here:

Windows: https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/install-nvidia-driver.html

Linux: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-nvidia-driver.html

### Linux

You can either run: `sudo apt install nvidia-driver-470` (You can replace 470 with your desired version)

Or, you can run the "Additional Drivers" application found in your application list. This will automatically install the latest proprietary driver.

### Windows

You can visit the [Nvidia drivers download page](https://www.nvidia.com/Download/index.aspx)

Note, when it comes to GRID drivers, you can find the guide to install in each relevant guide.

## Microsoft Azure

Azure offers an incredibly useful Extensions feature. Unlike the other cloud services, you can simply add the Nvidia requirements pre-launch of the instance.

When you get to the "Advanced" step of making your instance, click "Select an extension to install" and type "Nvidia" in the search bar. You should be presented with a "NVIDIA GPU Driver Extension", select that and click Next.

For other options to install drivers, see their guide here:

Windows: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/n-series-driver-setup

Linux: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/n-series-driver-setup

## How do I confirm my drivers are installed?
Simply run `nvidia-smi` in your terminal/command window.
On windows, you'll likely need to run the command from:
```
C:\Program Files\NVIDIA Corporation\NVSMI
``` 
## How do I connect to my instance on the cloud?

This varies depending on what operating system you're using. For further elaboration, please see the full guides.

### Linux

[Use SSH to connect to your instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
The command you'll need is:
```
ssh -i /path/my-key-pair.pem my-instance-user-name@my-instance-public-dns-name
```
Simply replace the "my-key-pair" with the key pair file you created with the instance, and both the user name and "instance-public-dns with the relevant instance information. The [AWS Linux Guide](AWS%20Linux%20Guide.md) goes through the process of creating an instance, including a key pair file and connecting via SSH.

### Windows

If both your instance and local system are running on Windows, you can easily connect to your instance using RDP (Remote Desktop Protocol).

You can find the .exe for RDP at `%systemroot%\system32\mstsc.exe` or simply open the start menu and start typing "Remote desktop protocol"
Simply enter the instance IP address in the computer section (make sure to add any required ports to the end, seperated by a colon e.g `1.2.3.4:1111`)

## I can't connect to my application running on Windows!

If you have set everything up correctly and still can't connect to your application, it's likely that your instance has its public firewall enabled.
If you open your server manager and check Windows Defender Firewall in Local Server, you'll see that it is enabled. Click on it and deactivate all the firewalls. You should be able to connect now.

## I can't install anything on my Windows Instance!

Open the start menu on your instance, and open "Server Manager".

At the top left of the window, click on "Local Server"

You'll see the properties section displaying the instances security settings. You'll need to change the following:

* Windows Defender Firewall: click and disable all the firewalls in the resulting window.
* Windows Defender Antivirus: click and disable "Real-time protection".
* IE Enhance Security Configuration: click and turn off for both Administrators and Users.

The above settings stop you from installing a lot of the required programs, as well as prevent you from connecting to your pixel stream later.


## How do I make my own instance image so I can skip all this setup?

Now that you have set up your instance and have it set up the way you like (drivers updated, programs and applications installed etc) you can create a custom AMI for easy set up.
This will allow you to load your instance straight into your current set up when you next launch it.

Once your instance is set up do the following:

### Amazon Web Services
* Go to view instances.
* Right click on your instance, click Image and Templates > Create Image
* Create a suitable name for your instance (You can not change this later, so make it a suitable name!)
* Create a description, I highly recommend listing what you have set up on the instance e.g. "NodeJS, Cirrus, GRID Drivers" etc.
* Add any extra volumes as needed (If you're doing simple pixel streaming testing, the default values should be fine)
* Click Create Image


Done! As you are charged for the storage of your AMI, make sure to check any extra costs involved when it comes to keeping your custom AMI: https://aws.amazon.com/ebs/pricing/

### Google Cloud Platform

* Go to your VM instances list.
* Click the triple dots next to the instance you wish to create an image of.
* Click "Create new machine image"
* Create a suitable name for your instance (You can not change this later, so make it a suitable name!)
* Create a description, I highly recommend listing what you have set up on the instance e.g. "NodeJS, Cirrus, GRID Drivers" etc.
* Specify your desired region and encryption
* Click Create

Done!

Ensure you read the following page on any costs related to your image:
https://cloud.google.com/compute/disks-image-pricing

### Microsoft Azure

* Open the overview panel for your created instance.
* At the top of the window, click the "Capture" option.
* Specify which resource group you want this to be a part of.
* Select either an existing computer gallery, or hit "Create new" if you do not have one prior.
* Select an existing VM image definition, or hit "Create new". Enter the VM name you wish to save it as. You can leave the other values default if desired. Then click OK.
* Enter a version number you wish to mark the instance as (it needs to follow a x.x.x format)
* Specify a default replica count.
* Hit review + create
* Click create

Done!

You can refer to pricing for maintaining images and disks here: https://azure.microsoft.com/en-gb/pricing/details/managed-disks/ 

## When I try to connect to the instance through my web browser, nothing shows up/it times out!

If you're trying to join the stream and nothing happens, there are a number of things it could be. 
Check the following:

* Make sure your signalling server is properly running
* Check to make sure you have opened port 80
* Check the IP restrictions in your server groups. If they are too restrictive they may not let you connect
* Make sure your instances Windows firewall isn't in the way.

## Error received = Vulkan not supported?

This means you haven't installed either the graphic card drivers or Vulkan. Refer back to the [Installing Drivers](#how-do-i-install-nvidia-drivers) and [Installing Vulkan](#how-do-i-install-vulkan-on-linux) sections.

## One person on the stream gets a bad connection, then everyone does!

One peer is the designated "quality control host", and should he have a bad connection, the stream will appear choppy/pixellated for everyone. 

This is currently a known limitation, but will likely change in future.

## When I pixel stream on an AMD GPU the quality is unpredictable/intermittent.

This is a known issue, AMD support is experimental.
This will be improved over time as AMD improves their AMF encoder.

## I'm getting pixellation in my pre - 4.27 build pixel stream!

All versions of UE4 prior to 4.27 can suffer from pixellation issues, the only proper solution to this issue is to upgrade your project to version 4.27.

## The stream quality is bad!

Stream quality is largely dictated by the network connection, as well as the performance of the application, low FPS means the application isn't running well.

If you want to hard cap the quality, to help control the result, you can do the following:

```
-PixelStreamingEncoderMaxQP=<value>
```
and
```
-PixelStreamingEncoderMinQP=<value>
```

The lower you set these values, the higher quality the stream, but the higher the bitrate.
A comfortable setting would be to set the MaxQP to 20, and set the MinQP to 5. This should help improve the stability of the stream.

For reference, these commands default to -1, which disables any hard limits.

## Pixel streaming shows "Streamer not connected"

This is actually a blanket error for when the UE4 engine is not connected to pixel streaming.

This can be a result of of the pixel streaming not being enabled in your project, or you haven't pointed the pixel streaming UE4 instance at the signalling server (e.g. `-PixelStreamingIP=127.0.0.1` and `-PixelStreamingPort=8888`).

## How do I test my STUN/TURN Server?

### STUN

1. Run https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/ using the stun server you will be using, if there is no `srflx` component, then your connection is not able to make contact with the STUN server. If no `srflx` then confirm ports are open and remote client/server is not behind symmetric NAT.

2. Run stuntman http://www.stunprotocol.org/ with your STUN server from inside your AWS machine and confirm the binding test, for example, 
```   
./stunclient --verbosity 3 stun1.l.google.com 19302.
```   
 If failed binding test then confirm ports are open and remote client/server is not behind symmetric NAT.

3. Confirm whether remote client is not behind symmetric NAT, see console output of this example. If the remote client is behind symmetric NAT you will need to run an additional server on AWS for TURN (this guide may help you), then specify that server to cirrus in the peer connection options.

### TURN

Once your signalling server and application are up and running, there is a very easy way to test the TURN server via Firefox.

1. Open a new tab in firefox
2. Enter `about:config` in the URL
Search for:
```
media.peerconnection.ice.relay_only
```
3. Set the result to "True".

This will force all connections to use the TURN relay. 

4. Try and connect to your application in a new tab. If the TURN server is working properly, it should connect. If it does not, it will likely get stuck and fail to connect.

If you would like to test a TURN connection through Chrome, you can simply add `/?ForceTURN` to the end of your URL, e.g:
`10.0.0.0/?ForceTURN`

Alternatively, you can see our page on doing a STUN/TURN Debug using pion/turn [here](ICE%20Debugging.md).

## TURN/relay candidates are not generated in Chrome?
Chrome will only let you use TURN if you are running TURN on port 53, 80, 443, 1024, or any port above 1024 [see WebRTC source](https://chromium.googlesource.com/external/webrtc/+/refs/heads/master/p2p/base/turn_port.cc#937). We typically recommend you run TURN on UDP port 19303. However, if you desperately need to run TURN on a port below 1024 you can do so by launching Chrome with a special field trial flag, like so:

**MacOS**
`/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --force-fieldtrials="WebRTC-Turn-AllowSystemPorts/Enabled/"`

**Windows**
`Chrome.exe --force-fieldtrials="WebRTC-Turn-AllowSystemPorts/Enabled/"`

## Getting extra information out of Chrome
When a WebRTC connection is not being made for whatever reason one key point of failure is on the browser side. There are a few different places to look within Chrome. For example:

1. chrome://webrtc-internals (This should be the first place you look for problems) - FireFox has a similar (but different) page called about:webrtc that you should cross reference with - particularly for ICE candidate failures.
2. Launch Chrome through the terminal with extra logging ([full details here](https://www.chromium.org/for-testers/enable-logging)).

**Print to the console**
`chrome.exe --enable-logging=stderr --v=1`

**Print stderr and stdout to a file**
`chrome --enable-logging=stderr --v=1 > log.txt 2>&1`


## How do I stop the Pixel Stream from capturing and hiding my mouse?
With recent updates to the Pixel Streaming Infrastructure, this has never been easier!

Once you have a stream running and you are connected via browser as a peer, open the control panel (by clicking on the cog) and toggling "Control Scheme: Locked Mouse" to "Hovering Mouse". This will ensure your mouse remains visible whilst interacting with the stream.

Additionally, if you're using the Unreal Engine software cursor, you can use this setting (or the "Hide Browser Cursor" setting) to hide the Windows cursor.

To enable the Unreal Engine software cursor you'll need to:

1. Open Project Settings > Engine - User Interface > Software Cursors.

2. Add a new software cursor (click the + icon) and you'll see 2 dropdown fields. 
3. In the first field, select "Default".
4. In the second field select "DefaultCursor".
5. Save project.

Now when you run your application, you'll have the Unreal Engine cursor enabled. If you don't hide the Windows cursor via the stream control panel, you'll have 2 visible cursors.


## Why does my Bitrate drop heavily on an Android/Samsung mobile connected to Wifi?

We have recently discovered a feature in some Samsung/Android devices that can cause moderate to severe reduction stream quality. If your mobile has this setting enabled, it can compromise stream quality for you and other connected peers. 

To rectify the issue, navigate to the following on your mobile:

1. Settings > Connections > Wifi
2. Select the cog/settings icon next to your connected Wifi network.
3. Select "View more".
4. Select "Metered network".
5. Select "Treat as unmetered".
6. Restart Wifi on your mobile.

This should rectify the issue. For some reason, despite having the bandwidth to handle the stream connection, this setting causes the pixel stream to consider your connection poor. If you're the quality control host on the stream, it will degrade the stream for all the peers. Interestingly, if you are NOT the quality control host, you can enjoy the stream with no reduction in stream quality!