# Working on remote Linux machines from a Windows 11 host


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Working on remote Linux machines from a Windows 11 host](#working-on-remote-linux-machines-from-a-windows-11-host)
  - [Connect with SSH and a X Server](#connect-with-ssh-and-a-x-server)
    - [PuTTY](#putty)
    - [OpenSSH with Xming](#openssh-with-xming)
    - [OpenSSH with Cygwin/X (preferred)](#openssh-with-cygwinx-preferred)
    - [Test X11 rendering](#test-x11-rendering)
  - [Remote development](#remote-development)
    - [Visual Studio](#visual-studio)
    - [Visual Studio Code](#visual-studio-code)

<!-- /code_chunk_output -->

## Connect with SSH and a X Server
I prefer to use SSH to connect from my host machine to remote Linux systems. In that way, there is no need for additional screen, keyboard and mouse, and I can use my preferred development tools (Visual Studio and VS Code) on my Windows 11 host. If you need to display images, videos, or X11 windows generated on the remote linux system, you will need to run a X Server on Windows. 

I explain below the different options I have tried. At the end, I work mainly with OpenSSH + Cygwin/X because this solution provides the best environment for remote development with Visual Studio.
 
### PuTTY
There is a nice [tutorial](https://tutorials-raspberrypi.com/raspberry-pi-remote-access-by-using-ssh-and-putty/) that explains how to use SSH with PuTTY. It explains also that you need to install a X Windows server like [Xming](https://sourceforge.net/projects/xming/) to enable viewing GUI programs. We will need that tool to display the output of the webcam. Here is a summary of what you need to do:
- install [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html);
- install [Xming](https://sourceforge.net/projects/xming/) with the option to launch;
- open PuTTY and specify the RPI hostname (e.g. rpi1.local); keep the port 22;
- go to Connection > SSH > X11 and check `Enable X11 Forwarding` with `localhost:0` for the X display location;
- open the connection.

To test that X11 forwarding works properly, launch a X terminal with `lxterminal` and confirm that it opens on your desktop. 

### OpenSSH with Xming
Using OpenSSH from a terminal is a bit more demanding. First, you need to install [OpenSSH for Windows](https://www.mls-software.com/opensshd.html). Then, install an X Windows server like [Xming](https://sourceforge.net/projects/xming/). Do not check the option to launch after the installation. When done, follow these instructions to establish the connection with X11 forwarding:
- In a windows CMD prompt, start Xming with:
 <br/>`"C:\Program Files (x86)\Xming\xming" :1322 -ac -rootless`
  <br/>The `-ac` parameter disables access control restrictions (that is similar to “xhost +” under unix). That enables all display access from remote machines, so to make it a bit more secure, an arbitrary display number, e.g. 1322, can be given to xming as second parameter. 
  <br/>The `-rootless` parameter specifies that a transparent root window is used.  
<br/>
- Logon as root through ssh: `ssh -Y rpi1.local` (specify your RPI hostname).
- In the RPI terminal prompt, you have to set the display as it's not the default display port number:<br/> `export DISPLAY=pchost:1322.0` (specify your Windows host)
- Test that X11 forwarding is properly configured with e.g. `lxterminal &`

### OpenSSH with Cygwin/X (preferred)
Another option for a X server on Windows is [Cygwin/X](https://x.cygwin.com/) ([installation](https://x.cygwin.com/docs/ug/setup.html)). There are a few additional steps to make it work in addition to the official documentation:
- The XWin Server shortcut did not work on my Windows 11 machine. To start the X server, I use the following command:
```bash
# start X server from a cygwin64 terminal
XWin -listen tcp -multiwindow -clipboard -silent-dup-error

# or from a DOS CMD terminal
c:\cygwin64\bin\run.exe -p /usr/X11R6/bin XWin -listen tcp -multiwindow -clipboard -silent-dup-error
```
- Next, ssh to the linux host:
```bash
# from a linux command prompt (cygwin, wsl2):
export DISPLAY=localhost:0.0
ssh -Y host

# from a DOS command prompt:
set DISPLAY=localhost:0.0
ssh -Y host
```
- Check the variable DISPLAY on the remote computer:
```bash
echo $DISPLAY
# It should be set to some value like localhost:11.0
```
- Run xeyes or xclock or xterm or another X application It should display in a window on your local machine. Your Cygwin/X server is working! 

### Test X11 rendering
We can test graphic rendering with the Xming or Cygwin/X server using the [Gource](https://gource.io/) application. This application allows to visualize git repos. 
```bash
# installation on the remote host
sudo apt-get update
sudo apt-get install gource

# clone the Gource git repository
git clone 'https://github.com/acaudwell/Gource.git'

# run gource
cd Gource
gource
```
You should see a window with an animated tree of the project directory:

![xwin1](/assets/images/xwin1.png)

## Remote development

### Visual Studio

- In [this video](https://www.youtube.com/watch?v=QAliMv5_DWI) Dave shows you how to single-step C++ code live in the Visual Studio debugger as it runs remotely on a Pi running Raspbian Linux 

### Visual Studio Code