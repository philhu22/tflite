This guide is for beginners without experience with TensorFlow Lite on Rapsberry Pi. It is written for guys like me who had to discover how to setup a Raspberry Pi for running the [TensorFlow's Introduction to object detection on Raspberry Pi](https://www.youtube.com/watch?v=mNjXEybFn98&list=PLQY2H8rRoyvz_anznBg6y3VhuSMcpN9oe). 


- [Shopping List](#shopping-list)
- [Setting up my Raspberry Pi](#setting-up-my-raspberry-pi)
  - [Install the OS](#install-the-os)
  - [Connect with SSH](#connect-with-ssh)
    - [PuTTY](#putty)
    - [OpenSSH with Xming](#openssh-with-xming)
- [Let's go with TF Lite](#lets-go-with-tf-lite)
- [It works!](#it-works)

## Shopping List
I decided to buy a Raspberry-Pi 4 model B with 8Gb of RAM. I choose the Essential Pack from the [MC Hobby](https://shop.mchobby.be/) shop which is very close to my home. My shopping list:
- [Raspberry Pi 4 8Gb Essential Pack](https://shop.mchobby.be/en/raspberry-pi-4/1858-raspberry-pi-4-8-go-de-ram-in-stock--3232100018587.html) (includes a 3 amp USB C power supply, micro HDMI to HDMI connector and a 32Gb micro SD card with an adapter)
- [Official's Raspberry Pi 4 case](https://shop.mchobby.be/en/raspberry-pi-4-case/1607-official-s-raspberry-pi-4-case-3232100016071.html) 
- [Silent fan cooler for Raspberry-Pi](https://shop.mchobby.be/en/raspberry-pi-4/1659-silent-fan-cooler-for-raspberry-pi-3232100016590-garatronic.html?search_query=RASP-VENTILA-PI4-MK2&results=565)

Total price is 128 €. 
  
I have also purchased a [Logitech HD 720p](https://www.logitech.com/en-us/products/webcams/c270-hd-webcam.960-000694.html) webcam. Add 30 € to the budget.

## Setting up my Raspberry Pi
### Install the OS
The first thing to do after you have received your RPI is to install the OS. The easiest way is to use the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) on a computer with a SD card reader. Put the SD card you have received into the reader and run the Imager. Here are the options I have selected: 
- Raspberry PI OS (32-bit) release 2022-01-28
- hostname : rpi1.local
- enable SSH with password authentication
- choose a valid username and password
- configure the wifi SSID
- configure your time zone and keyboard layout; select skip first-run wizard

Click Write to prepare the OS on the SD card. When done, install the SD Card on your Raspberry Pi and connect the USB C power supply. The OS will boot. You can use Wifi or plug a ethernet cable to establish the connection.

### Connect with SSH
I prefer to use SSH to connect from my host machine. In that way, there is no need for additonal screen, keyboard and mouse. You have several options to establish the SSH connection. Here are 2 options from a Windows host:
#### PuTTY
There is a nice [tutorial](https://tutorials-raspberrypi.com/raspberry-pi-remote-access-by-using-ssh-and-putty/) that explains how to use SSH with PuTTY. It explains also that you need to install a X Windows server like [Xming](https://sourceforge.net/projects/xming/) to enable viewing GUI programs. We will need that tool to display the output of the webcam. Here is a summary of what you need to do:
- install [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html);
- install [Xming](https://sourceforge.net/projects/xming/) with the option to launch;
- open PuTTY and specify the RPI hostname (e.g. rpi1.local); keep the port 22;
- go to Connection > SSH > X11 and check `Enable X11 Forwarding` with `localhost:0` for the X display location;
- open the connection.

To test that X11 forwarding works properly, launch a X terminal with `lxterminal` and confirm that it opens on your desktop. 

#### OpenSSH with Xming
Using OpenSSH from a terminal is a bit more demanding. First, you need to install [OpenSSH for Windows](https://www.mls-software.com/opensshd.html). Then, install an X Windows server like [Xming](https://sourceforge.net/projects/xming/). Do not check the option to launch after the installation. When done, follow these instructions to establish the connection with X11 forwarding:
- In a windows cmd prompt, start Xwing with <br/>`"C:\Program Files (x86)\Xming\xming" :1322 -ac -rootless`
  <br/>The "-ac" parameter disables access control restrictions (that is similar to “xhost +” under unix). That enables all display access from remote machines, so to make it a bit more secure, an arbitrary display number, e.g. 1322, can be given to xming as second parameter. 
  The "-rootless" parameter specifies that a transparent root window is used.  

- Logon as root through ssh: `ssh -Y rpi1.local` (specify your RPI hostname).
- In the RPI terminal prompt, you have to set the display as it's not the default display port number: `export DISPLAY=pchost:1322.0` (specify your Windows host)
- Test that X11 forwarding is properly configured with e.g. `lxterminal &`


## Let's go with TF Lite

We are now ready to install our first TensorFlow Lite model. For this, we will follow the TF tutorial [Introduction to object detection on Raspberry Pi](https://www.youtube.com/watch?v=mNjXEybFn98&list=PLQY2H8rRoyvz_anznBg6y3VhuSMcpN9oe). Watch the 15' video to find everything you need to know. I will just repeat here the [instructions](https://gist.github.com/khanhlvg/bbeb5e4ccfca6cbcf18508a44f5964be) to execute (with minor changes in the paths): 

```bash
# Show your Raspberry Pi OS version.
cat /etc/os-release
# VERSION_ID="11"  ok

# Update packages on your Raspberry Pi OS.
sudo apt-get update

# Check your Python version. You should have Python 3.7 or later.
python3 --version
# Python 3.9.2 ok

# Install virtualenv and upgrade pip.
python3 -m pip install --user --upgrade pip
python3 -m pip install --user virtualenv

# Create a Python virtual environment for the TFLite samples (optional but strongly recommended)
mkdir venv
python3 -m venv ~/venv/tflite

# Run this command whenever you open a new Terminal window/tab to activate the environment.
source ~/venv/tflite/bin/activate

# Clone the TensorFlow example repository with the TFLite Raspberry Pi samples.
mkdir tflite
cd tflite
git clone https://github.com/tensorflow/examples.git
cd examples/lite/examples/object_detection/raspberry_pi

# Install dependencies required by the sample
sh setup.sh

# Run the object detection sample
python detect.py

####
# If you see an error running the sample:
# ImportError: libcblas.so.3: cannot open shared object file: No such file or directory
# you can fix it by installing an OpenCV dependency that is missing on your Raspberry Pi.
sudo apt-get install libatlas-base-dev
```

## It works!
You should see the object detection running:

![tflite01](/assets/images/tflite01.png)