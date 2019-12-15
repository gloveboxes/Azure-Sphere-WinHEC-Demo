# Getting started with the Azure Sphere MT3620 Development Kit

| Azure Sphere   |  Image  |
| ---- | ---- |
| Azure Sphere MT3620 Development Kit | ![](resources/azure-sphere-mt3620-dev-kit.jpg) |
| Azure Sphere MT3620 Development Kit Shield | ![](resources/seeed-studio-grove-shield-and-sensors.jpg) |


## Create new Visual Studio Azure Sphere Project


![](resources/vs-create-new-project.png)

### Select Azure Sphere Project Template

![](resources/vs-select-azure-sphere-blink.png)

### Configure new Azure Sphere Project

![](resources/vs-configure-new-project.png)

### Open CMakeLists.txt

![](resources/vs-open-cmakelists.png)

### Add Reference to MT3620_Grove_Shield_Library

![](resources/vs-configure-cmakelists.png)


## Install Azure Sphere Development Tools and SDK

[Overview of set-up procedures](https://docs.microsoft.com/en-au/azure-sphere/install/overview)

## Application Manifest Capabilities by Sensor

## Define Application Capabilities Model

1. Review the [Capabilities by Sensor Type Quick Reference](#capabilities-by-sensor-type-quick-reference)
1. Open **app_manifest.json**
3. Add Uart **ISU0**
   - Note, access to the I2C SHT31 temperature/humidity sensor via the Grove Shield was built before Azure Sphere supported I2C. Hence calls to the sensor are proxied via the Uart.

```json
{
  "SchemaVersion": 1,
  "Name": "AzureSphereBlink1",
  "ComponentId": "a3ca0929-5f46-42b0-91ba-d5de1222da86",
  "EntryPoint": "/bin/app",
  "CmdArgs": [],
  "Capabilities": {
    "Gpio": [ 9 ],
    "Uart": [ "ISU0" ],
    "AllowedApplicationConnections": []
  },
  "ApplicationType": "Default"
}
```

## Read Grove Temperature/Humidity SHT31

1. Open main.c
2. add Grove Sensor headers after the _#include <applibs/gpio.h>_ header declaration.
    ```c
    #include "../MT3620_Grove_Shield/MT3620_Grove_Shield_Library/Grove.h"
    #include "../MT3620_Grove_Shield/MT3620_Grove_Shield_Library/Sensors/GroveTempHumiSHT31.h"
    ```

### Initialize the Grove Light Sensor

1. Open main.c
2. in the main method, just before _const struct timespec sleepTime = {1, 0};_

    ```c
    // Initialize Grove Shield and SHT31 Sensor
    int i2cFd;
    GroveShield_Initialize(&i2cFd, 115200);
    void* sht31 = GroveTempHumiSHT31_Open(i2cFd);
    ```

### Read SHT31 Sensor

1. In main.c inside the _while_ loop add

    ```c
	float temperature = GroveTempHumiSHT31_GetTemperature(sht31);
	float humidity = GroveTempHumiSHT31_GetHumidity(sht31);
	Log_Debug("Temperature %.2f, Humidity %.0f \n", temperature, humidity);
    ```

### Putting it all together

```c
#include <stdbool.h>
#include <errno.h>
#include <string.h>
#include <time.h>

#include <applibs/log.h>
#include <applibs/gpio.h>

#include "../MT3620_Grove_Shield/MT3620_Grove_Shield_Library/Grove.h"
#include "../MT3620_Grove_Shield/MT3620_Grove_Shield_Library/Sensors/GroveTempHumiSHT31.h"


int main(void)
{
	Log_Debug(
		"\nVisit https://github.com/Azure/azure-sphere-samples for extensible samples to use as a "
		"starting point for full applications.\n");

	// Change this GPIO number and the number in app_manifest.json if required by your hardware.
	int fd = GPIO_OpenAsOutput(9, GPIO_OutputMode_PushPull, GPIO_Value_High);
	if (fd < 0) {
		Log_Debug(
			"Error opening GPIO: %s (%d). Check that app_manifest.json includes the GPIO used.\n",
			strerror(errno), errno);
		return -1;
	}

	// Initialize Grove Shield
	int i2cFd;
	GroveShield_Initialize(&i2cFd, 115200);
	void* sht31 = GroveTempHumiSHT31_Open(i2cFd);

	const struct timespec sleepTime = { 1, 0 };
	while (true) {

		float temperature = GroveTempHumiSHT31_GetTemperature(sht31);
		float humidity = GroveTempHumiSHT31_GetHumidity(sht31);
		Log_Debug("Temperature %.2f, Humidity %.0f \n", temperature, humidity);

		GPIO_SetValue(fd, GPIO_Value_Low);
		nanosleep(&sleepTime, NULL);
		GPIO_SetValue(fd, GPIO_Value_High);
		nanosleep(&sleepTime, NULL);
	}
}
```

## Integrating with Azure IoT Central

Review [Set up Azure IoT Central to work with Azure Sphere](https://docs.microsoft.com/en-us/azure-sphere/app-development/setup-iot-central)

In summary:

1. Open an Azure Sphere Developer Command Prompt
2. Authenticate ```bash azsphere login ```
3. Download the Certificate Authority (CA) certificate for your Azure Sphere tenant ```bash azsphere tenant download-CA-certificate --output CAcertificate.cer ```
4. Upload the tenant CA certificate to Azure IoT Central and generate a verification code
5. Verify the tenant CA certificate
6. Use the validation certificate to verify the tenant identity

## Capabilities by Sensor Type Quick Reference

| Sensors  | Socket | Capabilities |
| :------------- | :------------- | :----------- |
| Grove Light Sensor  | Analog | "Gpio": [ 57, 58 ], "Uart": [ "ISU0"] |
| Grove Rotary Sensor | Analog | "Gpio": [ 57, 58 ], "Uart": [ "ISU0"] |
| Grove 4 Digit Display | GPIO0 or GPIO4 | "Gpio": [ 0, 1 ] or "Gpio": [ 4, 5 ] |
| Grove LED Button | GPIO0 or GPIO4 |  "Gpio": [ 0, 1 ] or "Gpio": [ 4, 5 ] |
| Grove Oled Display 96x96 | I2C | "Uart": [ "ISU0"]  |
| Grove Temperature Humidity SHT31 | I2C | "Uart": [ "ISU0"] |
| Grove UART3 | UART3 | "Uart": [ "ISU3"] |
| LED 1 | Red <br/> Green <br/> Blue | "Gpio": [ 8 ] <br/> "Gpio": [ 9 ] <br/> "Gpio": [ 10 ] |
| LED 2 | Red <br/> Green <br/> Blue | "Gpio": [ 15 ] <br/> "Gpio": [ 16 ] <br/> "Gpio": [ 17 ] |
| LED 3 | Red <br/> Green <br/> Blue | "Gpio": [ 18 ] <br/> "Gpio": [ 19 ] <br/> "Gpio": [ 20 ] |
| LED 4 | Red <br/> Green <br/> Blue | "Gpio": [ 21 ] <br/> "Gpio": [ 22 ] <br/> "Gpio": [ 23 ] |

For more pin definitions see the __mt3620_rdb.h__ in the MT3620_Grove_Shield/MT3620_Grove_Shield_Library folder.
