---
sort: 1
---

# Create and Configure the new project

## Create the project
-   Create a new Zigbee SoC project based on a *ZigbeeMinimal* project
<img src="./images/CAP_1_ProjectList.png" alt="" width="500" class="center">

-   Rename your project to *Z3_TemperatureSensor*
<img src="./images/CAP_1_ProjectName.png" alt="" width="500" class="center">

-   Select the target and desired toolchain (Thunderboard Sense 2 + GCC here)
<img src="./images/CAP_1_TargetAndToolchain.png" alt="" width="500" class="center">


## Configure the project
#### Hardware
-   Open the .ISC and go to the HAL tab and click *"Open Hardware Configurator"*
Alternatively open the .hwconf file located at the root of your project
<img src="./images/CAP_2_1_HarwareConfiguratorGeneral.png" alt="" width="500" class="center">

-   Go to *"DefaultMode Peripherals"* and enable the *"I2C Sensor"*
Change its properties as follows:
* Sensor enable pin : PF9
* I2C peripheral : I2C 1

<img src="./images/CAP_2_1_HarwareConfiguratorI2CSensor.png" alt="" width="500" class="center">

-   Still in *"DefaultMode Peripherals"*, change the I2C1 Settings as follows:
* I2C SCL : PC5
* I2C SCL : PC4

<img src="./images/CAP_2_1_HarwareConfiguratorI2C1.png" alt="" width="500" class="center">

#### Application 

-   In the ISC configuration, in the *"ZCL dusters"* tab, change *"ZCL device type"* field of the *Endpoint 1* device to *"HA Temperature Sensor"*

<img src="./images/CAP_2_2_ZCLConfiguration.png" alt="" width="700" class="center">

-   In the ISC configuration, in the *"Zigbee Stack"* tab, change *"Zigbee Device Type"* field of the *ZigBee PRO network configuration* *"Sleepy End Device"*

<img src="./images/CAP_2_2_DeviceType.png" alt="" width="700" class="center">

-   Finally, still in the ISC configuration file, enable *"Printing and CLI"* options according to your needs

Note that these options are dynamically available depending on your Plugin and ZCL configurations
You might go back here at the end of your full project editions to see if you did not miss anything
