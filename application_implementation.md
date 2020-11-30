---
sort: 2
---

# Implement your application
Silicon Labs App Builder projects have a very specific way of being implmented:
* Plugins are provided, and bring sources in the project that cover basic implementations for you
* Stack and Plugins both define *Callback functions* that allow you to implement event driven algorithms

Be careful:
* A lot of code is generated or linked according to your ZCL and Plugin configurations
* Callback stub function definitions are handled by Appbuilder - letting you responsible of implementing the functions you selected

Consequently, your Appbuilder based application implementation is going to be split into 2 main parts, the Plugins & Callbacks implementation, and your own code.


## Appbuilder Based Implementation
#### Plugins to add
Through the *"Plugin"* tab, add thefollowing plugins (use the search bar to find it):
* Network Steering  
    Used to join networks

* Update TC Link Key  
    Used to join Zigbee 3.0 centralized networks

* Reporting  
    Used to send periodic attribute reports
    Note that accuracy of reporting is relying on the LF clock source used (LFRCO, PLFRCO, LFXO)

* I2C Driver  
    Used to implement I2C for Si7021 communication

* Temperature Si7021  
    Driver implementation for the Si7021 (present on Thunder Board Sense boards)

* Temperature Measurement Server Cluster  
    Implementation of ZCL Temperature Measurement Server Cluster 
    This allows you to simply edit the plugin options and not waste time implementing any of it

* Zigbee PRO Leaf Library  
    Lightened Zigbee PRO Library for End Devices use only

* Find and Bind Initiator  
    Used to bind our device to a dicoverable Temperature Measurement Client 
    Mandatory as reportings only work using bindings - per spec

* Idle/Sleep  
    Used to bring the device down to EM2

* Identify Cluster

* Heartbeat  
    Blinks an LED to display stack status (only applicable when deep sleep is disabled)

<img src="./images/AI_1_1_Plugins.png" alt="" width="500" class="center">

#### Callbacks to add
Through the *"Callbacks"* tab, add the following functions (use the search bar to find it):
* Main Init  
    Will be used to add extra init code
* Complete (Network Steering)
* Complete (Find And Bind Initiator)
* Hal button ISR
* Ok To Sleep

<img src="./images/AI_1_2_Callbacks.png" alt="" width="500" class="center">

#### Custom Software Events
Finally, to safely link hardware events (i.e. button presses) to network or application actions (i.e. network joining, force sending reports) we need to declare Events in the ISC
This is done through the *"Includes"* tab of the ISC file, in the last section named *"Event Configuration"*

There add 2 new events:
* networkSteeringEventControl | networkSteeringEventHandler
* findingAndBindingEventControl | findingAndBindingEventHandler

<img src="./images/AI_1_3_Events.png" alt="" width="500" class="center">

All of the previous applies only when you click the "Generate" button on top of the ISC file


## Coded Implementation
Now that our project has its Plugins added, we will need to implement callback functions as well as events
By default, all callback implementations go to *project_name*_callbacks.c file
We will work there in this example

#### Callback implementations

-   Add the following lines at the top of the file:
```c
#include EMBER_AF_API_NETWORK_STEERING
#include EMBER_AF_API_FIND_AND_BIND_INITIATOR

#include "em_gpio.h"

#define TEMPERATURE_MEASUREMENT_ENDPOINT (1)

EmberEventControl networkSteeringEventControl;
EmberEventControl findingAndBindingEventControl;

void networkSteeringEventHandler(void);
void findingAndBindingEventHandler(void);
static void scheduleFindingAndBindingForInitiator(void);

static bool commissioning = false;
static uint8_t lastButton;
```

-   Edit the emberAfMainInitCallback implementation so that it looks like this:
```c
void emberAfMainInitCallback(void) {
    GPIO_PinModeSet(BSP_I2CSENSOR_ENABLE_PORT, BSP_I2CSENSOR_ENABLE_PIN, gpioModePushPull, 1);
}
```
This powers up the Si7021

-   Implement the HAL Button ISR so it triggers an network steering (Joining):
```c
void emberAfHalButtonIsrCallback(uint8_t button,
                                 uint8_t state)
{
  if (state == BUTTON_RELEASED) {
    lastButton = button;
    emberEventControlSetActive(networkSteeringEventControl);
  }
}
```

-   Edit the networkSteeringEventHandler as follows:
```c
void networkSteeringEventHandler(void)
{
  EmberStatus status;

  emberEventControlSetInactive(networkSteeringEventControl);

  if (emberAfNetworkState() == EMBER_JOINED_NETWORK) {
    if (lastButton == BUTTON0) {
        //To be programmed with something
    } else if (lastButton == BUTTON1) {
        //To be programmed with something
    }
  } else {

    status = emberAfPluginNetworkSteeringStart();
    emberAfCorePrintln("%p network %p: 0x%X",
                       "Join",
                       "start",
                       status);
    commissioning = true;
  }
}   
```
If the device is not joined to a network, this will trigger a Network steering procedure
Otherwise nothing (anything else can be coded)
Note that we declared a static *commissioning* variable so that the full joining proces covers both steering and binding

Finally, implement the Network steering Complete callback so it triggers a find and bind:
```c
void emberAfPluginNetworkSteeringCompleteCallback(EmberStatus status,
                                                  uint8_t totalBeacons,
                                                  uint8_t joinAttempts,
                                                  uint8_t finalState)
{
  emberAfCorePrintln("%p network %p: 0x%X", "Join", "complete", status);

  if (status != EMBER_SUCCESS) {
    commissioning = false;
  } else {
    scheduleFindingAndBindingForInitiator();
  }
}
```

-   Implement the forward declared schedule find and bind function:
```c
static void scheduleFindingAndBindingForInitiator(void)
{
  emberEventControlSetDelayMS(findingAndBindingEventControl,
                              200);
}
```
We do that to let time to the stack to initiate networking operations first

-   Also, edit the find and bind event handler:
```c
void findingAndBindingEventHandler(void)
{
  emberEventControlSetInactive(findingAndBindingEventControl);
  EmberStatus status = emberAfPluginFindAndBindInitiatorStart(TEMPERATURE_MEASUREMENT_ENDPOINT);
  emberAfCorePrintln("Find and bind initiator %p: 0x%X", "start", status);
}
```
This starts a find and bind procedure on the 1st Endpoint
In this example it is fixed, but you can have it changed at runtime as long as your local endpoint exists

-   Finally implement the find and bind complete to close the full joining procedure:
```c
void emberAfPluginFindAndBindInitiatorCompleteCallback(EmberStatus status)
{
  emberAfCorePrintln("Find and bind initiator %p: 0x%X", "complete", status);
  commissioning = false;
}
```

At this stage, network joining is complete and working
In order to use the CLI, we will disable low power by implementing the Ok To Sleep function so it returns false :
```c
bool emberAfPluginIdleSleepOkToSleepCallback(uint32_t durationMs)
{
	return false;
}
```

-   Build and flash the generated binary (do not forget to flash a bootloader)on your kit.

