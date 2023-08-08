## Setting up Docker and Nvidia Container Toolkit on Ubuntu Instances

Once you have an instance running, you may need Docker for container testing:
The information below has been taken from the [AWS drivers page](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-nvidia-driver.html) and the [Nvidia Container Toolkit installation page](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html), but written to be easier to digest.

**Note:** These steps are made for use with Ubuntu AWS instances


### Installing Nvidia GRID drivers and Disabling Nouveau

1. Connect to your Linux instance. Install gcc and make, if they are not already installed.

2. Update your package cache and get the package updates for your instance.

        $ sudo apt-get update -y

3. Upgrade the linux-aws package to receive the latest version.

        $ sudo apt-get upgrade -y linux-aws


4. Reboot your instance to load the latest kernel version.

        $ sudo reboot

5. Reconnect to your instance after it has rebooted. 

6. Install the gcc compiler and the kernel headers package for the version of the kernel you are currently running.

        $ sudo apt-get install -y gcc make linux-headers-$(uname -r)

7. Disable the nouveau open source driver for NVIDIA graphics cards.
Add nouveau to the /etc/modprobe.d/blacklist.conf blacklist file. Copy the following code block and paste it into a terminal.

        $ cat << EOF | sudo tee --append /etc/modprobe.d/blacklist.conf
        blacklist vga16fb
        blacklist nouveau
        blacklist rivafb
        blacklist nvidiafb
        blacklist rivatv
        EOF

1. Edit the /etc/default/grub file and add the following line:

        GRUB_CMDLINE_LINUX="rdblacklist=nouveau"

2. Rebuild the Grub configuration.

        $ sudo update-grub

3.  Download the GRID driver installation utility using the following command:

        $ aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .

4.  Multiple versions of the GRID driver are stored in this bucket. You can see all of the available versions using the following command.

        $ aws s3 ls --recursive s3://ec2-linux-nvidia-drivers/

5.  Add permissions to run the driver installation utility using the following command.

        $ chmod +x NVIDIA-Linux-x86_64*.run

6.  Run the self-install script as follows to install the GRID driver that you downloaded. For example:

        $ sudo /bin/sh ./NVIDIA-Linux-x86_64*.run

7.  When prompted, accept the license agreement and specify the installation options as required (you can accept the default options).
    

15. Confirm that the driver is functional. The response for the following command lists the installed version of the NVIDIA driver and details about the GPUs.

        $ nvidia-smi -q | head

16. If you are using NVIDIA vGPU software version 14.x or greater on the G4dn, G5, or G5g instances, disable GSP with the following commands. For more information, on why this is required visit NVIDIA’s documentation

        $ sudo touch /etc/modprobe.d/nvidia.conf

        $ echo "options nvidia NVreg_EnableGpuFirmware=0" | sudo tee --append /etc/modprobe.d/nvidia.conf

17. Reboot the instance.

        $ sudo reboot


### Installing Nvidia Container Toolkit and Docker

1. Setup package repository and GPG key:

        $ distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
         && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
         && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

2. Install the Nvidia container toolkit:

       $ sudo apt-get update

       $ sudo apt-get install -y nvidia-container-toolkit


3. Docker-CE on Ubuntu can be setup using Docker’s official convenience script:

        $ curl https://get.docker.com | sh \
        && sudo systemctl --now enable docker

4. Configure the Docker daemon to recognize the NVIDIA Container Runtime:

       $ sudo nvidia-ctk runtime configure --runtime=docker

5. Restart the Docker daemon to complete the installation after setting the default runtime:

        $ sudo systemctl restart docker

6. At this point, a working setup can be tested by running a base CUDA container:

       $ sudo docker run --rm --runtime=nvidia --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi

    This should result in a console output similar to:

        +-----------------------------------------------------------------------------+
        | NVIDIA-SMI 450.51.06    Driver Version: 450.51.06    CUDA Version: 11.0     |
        |-------------------------------+----------------------+----------------------+
        | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
        | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
        |                               |                      |               MIG M. |
        |===============================+======================+======================|
        |   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
        | N/A   34C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
        |                               |                      |                  N/A |
        +-------------------------------+----------------------+----------------------+

        +-----------------------------------------------------------------------------+
        | Processes:                                                                  |
        |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
        |        ID   ID                                                   Usage      |
        |=============================================================================|
        |  No running processes found                                                 |
        +-----------------------------------------------------------------------------+

    Done! You can now use docker to run your applications in a container on your AWS instance.