Intel IoT Gateway, Arduino 101 and Microsoft Azure Hands-On-Lab
===

Overview
---

In this lab, we will unbox and set up an Intel IoT Gateway and the Arduino 101 board (with a Grove Starter kit) along with several services available in Microsoft Azure to monitor the temperature and alert maintenance of a high temperature. Using Node-RED, running on the Intel NUC Gateway, the application will read the temperature value from a Grove temperature sensor and publish that data to an Azure IoT Hub.  From there a collection of Azure services including Stream Analytics, Event Hubs, SQL Database, Web Applications and Power BI Embedded will be used to both display the temperature data as well as alert the user to temperature readings over a certain threshold. 


Prerequisites
---

In order to successfully complete this lab you will ned:

- Intel Grove Commercial IoT Developer Kit [link](https://www.seeedstudio.com/Grove-IoT-Commercial-Developer-Kit-p-2665.html)
- Arduino 101 [link](https://www.arduino.cc/en/Main/ArduinoBoard101)
- A computer.  Windows, Mac OSx or Linux
- An active Microsoft Azure Subscription [free trial](https://azure.microsoft.com/en-us/free/)

Tasks
---

1. [Getting Started with Grove IoT Commercial Developer Kit](#GettingStartedWithGrove)
1. [Intel NUC Developer Hub Overview](#IntelNucHubOverview)
1. [Connecting to your Gateway using SSH](#ConnectingWithSSH)
1. [Blinking and LED with Node-RED](#Blinky)
1. [Reading the Temperature Sensor](#ReadingTemperatures)
1. [Creating an Azure IoT Hub and Device](#CreateIoTHubAndDevice)
1. [Publishing Temperature Sensor Data to the Azure IoT Hub](#PublishToIoTHub)
1. [Processing Temperature Data with Stream Analytics](#ProcessingWithStreamAnalytics)
1. [Displaying Temperature Data with Azure Web Apps](#AzureWebApp)
1. [Sending Messages from the Azure IoT Hub to the Intel Gateway](#CloudToDeviceMessages)
1. [TIME PERMITTING - Display Temperature Data with Power BI Embedded](#PowerBIEmbedded)

___

<a name="GettingStartedWithGrove"></a>
Getting Started with Grove IoT Commercial Developer Kit
---


1. Unbox the Grove Starter Kit and Arduino 101

    ![Grove Starter Kit](images/01010-GroveStarterKit.png)
    ![Arduino 101](images/01020-Arduino101.png)

1. Remove the Grove Base Shield from the Grove Starter Kit and attach it to the Arduino 101

    ![Grove Base Shield](images/01030-AttachGroveBaseShield.png)

1. Ensure the base sheild's voltage selector switch is set to 5V

    ![Base Sheild set to 5V](images/01040-BaseShield5V.png)

1. Connect the LCD to any I2C socket and rotary angle to A0 socket with a grove connector cable as shown below, the cables are in the green box.

    ![Rotary Encoder and LCD](images/01050-RotaryEncoderAndLCD.png)
    ![Rotary Encoder and LCD Attached](images/01060-RotaryEncoderAndLCDAttached.png)

1. Connect the Arduino 101 to the Intel IoT Gateway using the USB cable provided:

    ![Attach Arduino 101](images/01070-AttachArduino101.png)

1. Connect the Intel IoT Gateway to Ethernet and Power

    ![IoT Gateway Ethernet and Power](images/01080-IoTGatewayEthernetAndPower.png)

1. Press the power button on the IoT Gateway to boot it.  It will take about two minutes or so for the device to boot.  Once it has booted the default Node-RED flow on the gateway will run and display the Gateway's IP Address on the LCD panel attached to the Arduion 101.  ***The IP Address displayed is the IP Address of your Intel IoT Gateway NUC.  You will use this to attach to your gateway throughout the rest of this lab.***  

    ![IP Address on LCD Panel](images/01090-IPAddressOnLCD.png)

1. Open your browser and go to the IP Address from the previous step (http://xxx.xxx.xxx.xxx where xxx.xxx.xxx.xxx is the IP Address from above).  If you are presented with a **"Privacy Statement"** click **"Continue"**.

    ![Privacy Statement](images/01100-PrivacyStatement.png)

1. Login is using the "**root**" as both the user name and password:

    ![Login as root](images/01110-Login.png)

1. If presented with the "**License Agreement**", click "**Agree**" to continue:

    ![License Agreement](images/01120-Eula.png)

1. Once you have successfully logged in, you should see the "**IoT Gateway Developer Hub". 

    ![IoT Gateway Developer Hub](images/01130-IoTGatewayDeveloperHub.png)
___

<a name="IntelNucHubOverview"></a>
Intel NUC Developer Hub Overview
---

The Developer Hub is a front end interface for the Gateway. It has 5 main functions:

- Display sensor data and basic Gateway information on a configurable dashboard 
- Access the Node-RED development environment 
- Add repositories, install and upgrade packages
- Administer the Gateway – update OS, configure network setting 
- Access documentation 

1. The "**Sensors**" page can be used to monitor the sensor data that is being published to the dashboard.  By default the gateway is configured to display the value of the rotary angle sensor (knob) attached to the Arduino 101.  You can twisth the knob to see the values change in the Device and Sensor Information panel on this page.  

    ![Sensor Dashboard](images/02010-Sensors.png)

1. You can collapse the Device and Sensor Information panel along the top of the page to make the other information below easier to see by clicking the arrow at the top of the navigation bar.  Click the button again to expand the panel again.  

    ![Collapsed Panel](images/02020-CollapsedDeviceAndSensorData.png)

1. The "**Packages**" page allows you to manage the various RPM package repositories and installed packages on your gateway.  We will use this page later to add the Microsoft Azure related capabilities to your gateway.

    > **Note**: PLEASE DO NOT INSTALL UPDATES AT THIS TIME.  While normally you would want to update your devices, in the lab environment this will take too long and negatively impact the available network bandwidth.  Please refrain from updating your IoT Gateway while at an IoT Camp event. You are welcome to perform the updates back on your own network. 

    ![Packages](images/02030-Packages.png)  

1. The "Administration" page provides easy access to a number of tools.  The tools most applicable to this lab are the "Quick Tools":

    > **Note**: PLEASE DO NOT INSTALL OS UPDATES OR UPGRADE TO PRO AT THIS TIME.  While these options may be desirable they will take too long for the time frame of this lab as well as have a negative impact on the available bandwidth at the event.  Please refrain from updating your IoT Gateway while at an IoT Camp event.  You are welcome to perform the updates back on your own network. 

    - Easy access to the Node-RED development environment that has been pre-installed on the IoT Gateway.  We will use this extensively in this lab.
    - A way to register your gateway with the "Wind River Helix App Cloud".  We won't be using this feature as part of this lab.
    - Access to the "LuCI" ( [link](https://github.com/openwrt/luci/wiki) ) interface to configure administrative settings on your gateway.  **PLEASE DO NOT MAKE CHANGES USING THIS TOOL.**     
    - The "Cloud Commander" web console, file manager, and editor.

    ![Administration](images/02040-Administration.png)

1. The "Documentation" page provides a wealth of documentation, community, and source code resources.

    ![Documentation](images/02050-Documentation.png)  

___

<a name="ConnectingWithSSH"></a>
Connecting to your Gateway using SSH
---

In order to perform advanced configuration of the Gateway either a monitor and keyboard, or a Secure Shell (SSH) connection is required. On OSX and Linux there are default programs that can do this - Screen and SSH respectively. However on Windows no default exists, however Putty is light weight and easy to install and can be used to SSH into the Gateway.


1. For Windows users:

    > **Note**: In the screen shots below, ***192.168.2.13*** is the IP Address of the IoT Gateway being connected to.  Replace that IP Address with the IP Address of your IoT Gateway.  That is the IP Address that should be displayed on the LCD Panel attached to your Arduino 101.

    - Visit the [PuTTY download page](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).
    - Under the "For Windows on Intel x86" heading, click on the "putty.exe" link to download the latest release version to your computer. Or if you prefer to use an installer that includes all of the additional tools like PSCP and PSFTP click the putty-0.67-installer.exe link (or latest version).

    ![PuTTY Downloads](images/03010-PuttyDownload.png)

    - Double-click putty.exe on your computer to launch PuTTY, or run the installer to install it if you chose to download it.  Then run PuTTY.
    - Enter IP address of the Gateway

    ![PuTTY Configuration](images/03020-PuTTYConfiguration.png)

    - If you are presented with a "**PuTTY Security Alert**", click "**Yes**" to continue:

    ![PuTTY Security Alert](images/03030-PuTTYSecurityAlert.png)

    - You can login with the username root and the password root.

    ![Login as root](images/03040-LoginAsRoot.png)

1. For Mac OSx and Linux users

    - Open a terminal Window
    - At the prompt, type the follwing command.  Replace `<<IP Address>>` with the IP Address of your IoT Gateway: 

        `ssh root@<<IP Address>>`

    - Enter ***root*** as the password

___

<a name="Blinky"></a>
Blinking and LED with Node-RED
---

In this exercise, you will use the Node-RED development environment pre-installed on the Intel IoT Gateway to blink the onboard LED on your Arduino 101. 

1. To open the Node-RED development environment on your IoT Gateway:

    - In your browser, navigate to http://xxx.xxx.xxx.xxx where xxx.xxx.xxx.xxx is your gateway's IP Address.
    - Click the "Administration" link
    - Click the "Launch" button under the "Node-RED" icon.

    ![Launch Node-RED](images/04010-LaunchNodeRed.png)

    > **Note**: Accessing the Node-RED environment from the Administration page leaves the IoT Gateway links, etc. still visible at the top of the page and can cause some difficulty with the Node-RED environemtn.  If you prefer to access the Node-RED environment directly, you can do so by navigating to port **1880** on your gateway using **http://xxx.xxx.xxx.xxx:1880** . Again, replace xxx.xxx.xxx.xxx with your gateway's IP Address.   

1. The Node-RED environment will show the default "Flow 1" workflow that is responsible for retrieving your gateway's IP Address and displaying it on the LCD panel as well as reading the value from the rotary angle sensor and displaying it in the charts on the IoT Gateway web portal.  Leave this flow as is for now. 

    ![Node-RED Environment](images/04020-NodeREDEnvironment.png)

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps

___

<a name="ReadingTemperatures"></a>
Reading the Temperature Sensor
---

1. Do this

    ````javascript
     var sample = 'with some code';
     console.log(sample);
    ````

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps

___

<a name="CreateIoTHubAndDevice"></a>
Creating an Azure IoT Hub and Device
---

1. Do this

    ````javascript
     var sample = 'with some code';
     console.log(sample);
    ````

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps

___

<a name="PublishToIoTHub"></a>
Publishing Temperature Sensor Data to the Azure IoT Hub
---

1. Do this

    ````javascript
     var sample = 'with some code';
     console.log(sample);
    ````

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps

___

<a name="ProcessingWithStreamAnalytics"></a>
Processing Temperature Data with Stream Analytics
---

1. Do this

    ````javascript
     var sample = 'with some code';
     console.log(sample);
    ````

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps

___

<a name="AzureWebApp"></a>
Displaying Temperature Data with Azure Web Apps
---

1. Do this

    ````javascript
     var sample = 'with some code';
     console.log(sample);
    ````

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps

___

<a name="CloudToDeviceMessages"></a>
Sending Messages from the Azure IoT Hub to the Intel Gateway
---

1. Do this

    ````javascript
     var sample = 'with some code';
     console.log(sample);
    ````

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps

___

<a name="PowerBIEmbedded"></a>
TIME PERMITTING - Display Temperature Data with Power BI Embedded
---

1. Do this

    ````javascript
     var sample = 'with some code';
     console.log(sample);
    ````

1. Show some screenshot:

    ![EULA](images/01120-Eula.png)

1. Do something else

    > **Note!**: This is a sample note!

    - With one
    - or two
    - substeps


