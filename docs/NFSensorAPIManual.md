# NFSensor API Reference


## Introduction


The NFSensor API includes a  programming library that allow you to write custom programs to integrate with the different Sensors distributed  by NanoFocus AG. This reference guide contains information on this library and is directed toward programmers using C languages or C++ (using the C library), who want to write their own sensor control applications for a particular sensor. For languages like C# or Python, language bindings are provided, which will be refered in supplementary documentation.


## Requirements

* Operating System Windows 10 64Bit
* ANSI C99  compliant compiler
* Tested and developed under Visual Studio 2017 Professional
* All strings handled here are zero – terminated plain ASCII strings (not unicode).


## Dependencies and Distribution

NFEval Sensor API ships as a self contained zip archive and contains all binaries to operate on a specified sensor. A dependency graph can be seen in Fig. "Package Dependency". Dependend on current version, the filenames can be differ by version postfixes. In addition the package contains neccessary third party libraries as needed.

![](NFSensorDependencyGraph.svg)

Fig: Package Dependency


## Overview

### Plugin System

All sensor specific implementation are sourced out to a specific plugin dll which is usually located in the devices sub folder.

### Statemachine

The NFSensor SDK is based on a finite state machine implementation. See Fig. "State Machine Architecture" for an overview. At any given point in time the system is in a requested state or state transition and the state can be queried at any given point in time.

![](NFSensorStateMachine.svg)

Fig: State Machine Architecture

The design implies that called forbidden state transition could not be possible. For example, it is not adviseable to go from state NFSensor_Acquisition to NFSensor_Live which would result in unpredicted data. Therefore the internal design prohibits such transitions and will return an error to the caller.
All states are mapped to  state ID as shown in Table 1.
```
typedef enum
  {
    Unkown = 0,
    Error,
    Connecting,
    Connected,
    Initializing,
    Idle,
    Calibrating,
    Live,
    Acquisition,
    PreparingAcquisition,
    AcquiringData,
    PostProcessingAcquisition,
    IdleReady,
    IdleWithErrors,
    IdleAborted
  } StateID;
  ```
Table 1: Overview StateID

It is advisable to check for current state with the interface function

_NFSensor_API int NFSensor_GetStateID(SensorHandle hSensor, StateID * eStateID)_

and to use  the result StateID  into  program execution flow.

### Error Handling

In general all interface functions return integer based error code. A non zero return indicates an error. Special error codes are not provided. To obtain further information in plain text you can call the API funtion

_NFSensor_API const char * NFSensor_GetLastError()_

and evaluate the null terminated ASCII string for further error tracing and debugging. Please contact your technical manual and/or service support.


##  Interface functions 

### __General Functions__

---
>### NFSensor_API int NFSensor_Create(SensorModel eSensorModel, SensorHandle *hSensor);
---

Purpose:
* selects the specified sensor model specified by eSensorModel
* creates all of internal machinery and dependencies. Loads additional plugins if needed
* has to be called  at least once
* multiple creations of the same physical sensor is prevented by the library and returns non zero return and a null SensorHandle
* accidently creation of an already created sensor device returns non zero, but the handle is still valid

Parameters:

* eSensorModel [in]: selected Sensor Id, from available Sensor Models
* hSensor [out]: opaque identifier to sensor object, aka sensor handle

Returns:

* Zero on success

Example usage:


---
>### NFSensor_API int NFSensor_Destroy(SensorHandle hSensor);
 ---

Purpose:
* cleans up all of internal machinery and dependencies
* should be called at end of usage / end of program flow or if sensor has to be recreated by any reason

Parameters:

* hSensor  [in]: current sensor handle

Returns:

* Zero on success

Example usage:


---
>###  NFSensor_API int NFSensor_Connect(SensorHandle hSensor);
---

Purpose:
* establishes the connection to the underlying sensor device hardware
* triggers state transition from Unknown to Connected
* on non zero success, the last error can be obtained with NFSensor_GetLastError

Parameters: 

* hSensor  [in]: current sensor handle

Returns:

* Zero on success
* on non zero success, the last error can be obtained with _NFSensor_GetLastError_ 

Example usage:


---
>###  NFSensor_API int NFSensor_Init(SensorHandle hSensor);
---

Purpose:
* prepares / initializes for subsequent operations
* triggers state transition from Connected to Idle
* on non zero success, the last error can be obtained with _NFSensor_GetLastError_  

Parameters:

* hSensor  [in]: current sensor handle

Returns:

* Zero on success
* on non zero success, the last error can be obtained with _NFSensor_GetLastError_ 

Example usage:


---
>###  NFSensor_API int NFSensor_ShutDown(SensorHandle hSensor);
---

Purpose:

* triggers state transition from Idle to Connected
* on non zero success, the last error can be obtained with _NFSensor_GetLastError_

Parameters:

* hSensor  [in]: current sensor handle

Returns:

* Zero on success
* on non zero success, the last error can be obtained with _NFSensor_GetLastError_

Example usage:


---
>###  NFSensor_API int NFSensor_Disconnect(SensorHandle hSensor);
---

Purpose:
* cleans up and closes resources to underlying sensor device hardware
* triggers state transition from Connected to Unknown
* on non zero success, the last error can be obtained with _NFSensor_GetLastError_

Parameters: 

* hSensor  [in]: current sensor handle

Returns:

* Zero on success
* on non zero success, the last error can be obtained with _NFSensor_GetLastError_

Example usage:


### __Acquisition Functions__ 

---
>###  NFSensor_API int NFSensor_StartLive(SensorHandle hSensor);
---

Purpose:
 * triggers state transition from Idle to Live
 * sensor continously acquires data buffers and calls the user provided callback LiveDataReadyCallback
 * used to show signal during setup and testing

Parameters:

* hSensor  [in]: current sensor handle

Returns:

* Zero on success
 
Example usage:


---
>### NFSensor_API int NFSensor_StopLive(SensorHandle hSensor);
---

Purpose:
 * quits the live state operation
 * triggers state transition from Live to Idle
 * has to be called before data acquisition

Parameters:

* hSensor  [in]: current sensor handle
 
Returns:

* Zero on success

Example usage:


---
>###  NFSensor_API int NFSensor_StartAcquisition(SensorHandle hSensor, uint64_t NumberOfBuffers);
---

Purpose:
* triggers state transition from Idle to Acquisition
* function returns immediately
* finish of acquisition is signaled by _AcquisitionDoneCallback_

Parameters:

* hSensor  [in]: current sensor handle
* uint64_t NumberOfBuffers [in]: number of data points (buffers) to acquire

Returns:

* Zero on success

Example usage:


---
>###  NFSensor_API int NFSensor_StopAcquisition(SensorHandle hSensor);
---

Purpose:
* triggers state transition from Acquisition to Idle
* with this function user can cancel current acquisition phase
* function returns immediately, due to internal processing a synchronisation of sensor state is advisable before program flow execution 

Parameters:

* hSensor  [in]: current sensor handle

Returns:

* Zero on success
* on non zero success, the specified parameter is not available for the current sensor

Example usage:


---
>###  NFSensor_API int NFSensor_Abort(SensorHandle hSensor);
---

Purpose:

 *  user triggered stop of Live state or Acqusition state
 *  call is non blocking, a manual state synchronisation might be neccessary

Parameters: 

* hSensor  [in]: current sensor handle

Returns:

* Zero on success

Example usage:

```
StateID id(Unkown);
int rc = 0;

rc = NFSensor_StartAcquisition(hSensor, 10000));

rc = NFSensor_GetStateID(hSensor, &id));

rc =  NFSensor_Abort(hSensor));
     
while (StateID::AcquiringData == id)
{
   REQUIRE(0 == NFSensor_GetStateID(hSensor, &id));
   // Sleep(10);
}
```


---
>###   NFSensor_API int NFSensor_Acknowledge(SensorHandle hSensor);
---

Purpose:
* in case sensor device is in state IdleAborted or IdleWithErrors, state transition to Idle
* in case sensor device is in state Error, state transition to Unknown

Parameters:

* hSensor  [in]: current sensor handle

Returns:

* Zero on success
 
Example usage:


### __Configuration Functions__

To reflect on  each sensor specific parameters the NFSensor API provides a generic like interface to get and set parameters. It is organized by key - value semantic and the ParamType data structure allows to pass floating point values, integer values or string values by specifiying the apropiate data type.
The get and set functions are not bound to state machine states, so it is possible to use them at any point in the program flow. It can be 
necessary for example to set certain connection parameters (e.g. IP Address ) before calling NFSensor_Connect to transit into the idle state. Though there are common parameters for all sensors, the available parameter keys are stored in a specific header file for each sensor 
shipped with the library.

---
>### NFSensor_API int NFSensor_GetParameter(SensorHandle hSensor, const char * strParamName, ParamType * sParamValue);
---

Purpose:
 * retrieve sensor parameter by given key strParamName

Parameters:

* hSensor  [in]: current sensor handle
* const char * strParamName [in]: parameter name
* ParamType * sParamValue [out]: parameter value

Returns:

* Zero on success
* on non zero success, the specified parameter is not available for the current sensor

Example usage:

```
#include "NFScanSensorSimulParameter.h"

SensorHandle hSensor = 0;
int rc = NFSensor_Create(SCAN_SENSOR_SIMULATOR, &hSensor);

if (rc == 0)
{
  // Get pixel size in x-direction from sensor
  ParamType parameter;
  rc = NFSensor_GetParameter(hSensor, NFScanSensorSimul_PixelSizeX, &parameter);

  if (parameter.type == DataType::DOUBLE_TYPE) printf("Pixel Size X   %f", parameter.data.d);
}

rc = NFSensor_Destroy(hSensor);
```


---

>### NFSensor_API int NFSensor_SetParameter(SensorHandle hSensor, const char * >strParamName, ParamType sParamValue);

---

Purpose:
* set sensor parameter by given key strParamName and value sParamValue

Parameters: 

* hSensor  [in]: current sensor handle
* const char * strParamName [in]: parameter name
* ParamType sParamValue [in]: parameter value

Returns:

* Zero on success
* on non zero success, the specified parameter is not available for the current sensor

Example usage:

```
#include "NFScanSensorSimulParameter.h"

SensorHandle hSensor = 0;
int rc = NFSensor_Create(SCAN_SENSOR_SIMULATOR, &hSensor);

if (rc == 0)
{
  // Set new value for light intensity in percent
  ParamType parameter;
  double dLightIntensity = 31.8;
  rc = NFSensor_SetDoubleToParamType(dLightIntensity, &parameter); // Helper function
  rc = NFSensor_SetParameter(hSensor, NFScanSensorSimul_LightIntensity, parameter);
}

rc = NFSensor_Destroy(hSensor);
```

---


## Sensor Types

### C3x Sensor
