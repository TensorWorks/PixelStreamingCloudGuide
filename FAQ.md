# Frequently Asked Questions

This list is to help solve a variety of issues that you may stumble across. Some of this information is also mentioned in the guides, but is here for quicker reference.

- [Frequently Asked Questions](#frequently-asked-questions)
  - [How do I install Vulkan on Linux?](#how-do-i-install-vulkan-on-linux)
  - [How to I install Pulse Audio on Linux?](#how-to-i-install-pulse-audio-on-linux)
  - [How do I install Nvidia Drivers?](#how-do-i-install-nvidia-drivers)
    - [Linux](#linux)
    - [Windows](#windows)
  - [How do I connect to my instance on the cloud?](#how-do-i-connect-to-my-instance-on-the-cloud)
    - [Linux](#linux-1)
    - [Windows](#windows-1)

## How do I install Vulkan on Linux?
`sudo apt install vulkan-utils`

## How to I install Pulse Audio on Linux?
`sudo apt-get install -y pulseaudio`

## How do I install Nvidia Drivers?

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
You can find the .exe for RDP at `%systemroot%\system32\mstsc.exe`
Simply enter the instance IP address in the computer section (make sure to add any required ports to the end, seperated by a colon e.g `1.2.3.4:1111`)

