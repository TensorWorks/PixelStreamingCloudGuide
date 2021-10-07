# Frequently Asked Questions

This list is to help solve a variety of issues that you may stumble across. Some of this information is also mentioned in the guides, but is here for quicker reference.

- [Frequently Asked Questions](#frequently-asked-questions)
  - [How do I install Vulkan on Linux?](#how-do-i-install-vulkan-on-linux)
  - [How do I install Pulse Audio on Linux?](#how-do-i-install-pulse-audio-on-linux)
  - [How do I install Nvidia Drivers?](#how-do-i-install-nvidia-drivers)
    - [Linux](#linux)
    - [Windows](#windows)
  - [How do I connect to my instance on the cloud?](#how-do-i-connect-to-my-instance-on-the-cloud)
    - [Linux](#linux-1)
    - [Windows](#windows-1)
  - [I can't connect to my application running on Windows!](#i-cant-connect-to-my-application-running-on-windows)
  - [How do I make my own instance image so I can skip all this setup?](#how-do-i-make-my-own-instance-image-so-i-can-skip-all-this-setup)
    - [Amazon Web Services](#amazon-web-services)
    - [Google Cloud Platform](#google-cloud-platform)



## How do I install Vulkan on Linux?
`sudo apt install vulkan-utils`

## How do I install Pulse Audio on Linux?
`sudo apt-get install -y pulseaudio`

## How do I install Nvidia Drivers?

Note: If you're running on AWS, they provide instruction on how to install GRID drivers. We recommend this option as in our experience GRID drivers provide much better performance than the standard public drivers.

Please see the relevant guides here:

Windows: https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/install-nvidia-driver.html

Linux: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-nvidia-driver.html

### Linux

You can either run: `sudo apt install nvidia-driver-470` (You can replace 470 with your desired version)

Or, you can run the "Additional Drivers" application found in your application list. This will automatically install the latest proprietary driver.

### Windows

You can visit the [Nvidia drivers download page](https://www.nvidia.com/Download/index.aspx)


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

Coming Soon!
