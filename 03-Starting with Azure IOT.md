
This is the description of my first steps with Azure IoT. I had 2 goals:
1. Connect the Raspberry Pi to an Azure IoT Hub;
2. Learn the basics of the Azure IoT SDK for C/C++

## Connect Raspberry Pi to Azure IoT Hub
Microsoft has 2 tutorials that explain the basics of connecting a Raspberry Pi to an Azure IoT Hub:
- [Connect Raspberry Pi to Azure IoT Hub (Node.js)](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-raspberry-pi-kit-node-get-started)
- [Connect Raspberry Pi to Azure IoT Hub (C)](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-raspberry-pi-kit-c-get-started) 

The tutorial for Node.js explains how to create an IoT hub, register your RPI device, and setup this device to send messages with sensor data to the IoT hub. A simulator is available if you do not have a sensor or did not want to take time to setup the I2C bus.  

### Azure IoT Tools extension for VS Code 
Messages sent from the RPI to the IoT hub can be monitored in Visual Studio Code using the [Azure IoT Tools extension](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-vscode-iot-toolkit-cloud-device-messaging). After the extension is installed, you will have a new section `AZURE IOT HUB` at the bottom of the Explorer view of VS Code. The first time you invoke a command under this section, you will have to sign in to Azure, then select your subscription and the IoT hub. After a few seconds, you will see the list of devices attached to the hub. 

Alternatively, you can open the Command Palette (`Ctrl+Shift+P`) and type the command

 `Azure IoT Hub:Set IoT Hub Connection String`

 Enter the **iothubowner** policy connection string for the IoT hub that your IoT device connects to in the pop-up window. To find this string, go to the Azure Portal and open your IoT Hub blade. Select `Shared access policies` under `Security settings`, then click on `iothubowner` and copy the Primary connection string.

### Running the Node.js sample app
Follow the instructions given in [Connect Raspberry Pi to Azure IoT Hub (Node.js)](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-raspberry-pi-kit-node-get-started). There is no difficulty. As I did not want to waste time with the installation of the sensor, I have specified `simulatedData: true` in config.json. Then, run the application with:
```bash
sudo node index.js '<YOUR AZURE IOT HUB DEVICE CONNECTION STRING>'
# Make sure you copy-paste the device connection string into the single quotes.
```
You should see the following output that shows the sensor data and the messages that are sent to your IoT hub:

![iot2](/assets/images/iot2.png)

You can now monitor the messages received by the IoT hub in VS Code:

![iot1](/assets/images/iot1.png)

![iot](/assets/images/iot.png)


### Running the C sample app
It was a bit less straighforward to run the C sample app. Many includes in `main.c` could not be resolved after running `setup.sh`. The root cause: an error in the file `setup.sh` at line 15 (in my version of March 22 2022). You must replace the line

`cd ../../cmake/iotsdk_linux`

by

`cd ../../cmake`

![iot3](/assets/images/iot3.png)

Then, build and run with
```bash
# use simulated data
sudo ./setup.sh --simulated-data

# run 
sudo ./app '<DEVICE CONNECTION STRING>'
```
You should see the sensor data and the messages that are sent to your IoT hub. See the Node.js section above to monitor these messages in VS Code.

## Setup Azure IoT C SDK
### Install the azure-iot-sdk-c package
Follow the instructions at [Setup Azure IoT C SDK vcpkg for Windows development environment](https://github.com/Azure/azure-iot-sdk-c/blob/main/doc/setting_up_vcpkg.md#setup-azure-iot-c-sdk-vcpkg-for-windows-development-environment). In summary:
   
Open PowerShell and run the following commands:
```powershell
git clone https://github.com/Microsoft/vcpkg.git vcpkg_new
pushd .\vcpkg_new\
.\bootstrap-vcpkg.bat
```
and install the azure-iot-sdk-c package in Visual Studio:
```powershell
# Install azure-iot-sdk-c package with integration
.\vcpkg.exe integrate install
.\vcpkg.exe install azure-iot-sdk-c
popd
```

### Test with a sample application
Open the Azure IoT C SDK telemetry sample solution for Visual Studio:
```powershell
git clone https://github.com/Azure/azure-iot-sdk-c.git 
pushd .\azure-iot-sdk-c\iothub_client\samples\iothub_ll_telemetry_sample\windows\
start .\iothub_ll_telemetry_sample.sln
```
Change the build target to `Debug x86` and hit `F5` to build and run. The project should compile and start. It will stop after complaining about token and IOT Hub not found. Normal as we did not configure the application.

