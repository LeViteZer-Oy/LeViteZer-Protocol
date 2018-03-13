# Levitezer Protocol

> Please note: Levitezer Protocol is under development, definitions are subjected to be changed

## Table of contents
- [Levitezer Protocol](#levitezer-protocol)
  * [Sckeleton](#sckeleton)
    + [Header](#header)
    + [Parameter Descriptions](#parameter-descriptions)
    + [End of Message](#end-of-message)
    + [Checksum](#checksum)
  * [Devices](#devices)
  * [Parameter Descriptions](#parameter-descriptions)
    + [Controller Data](#controller-data)  
    + [Gimbal Data](#gimbal-data)
    + [Camera Data](#camera-data)
  * [Examples](#examples)
  
## Sckeleton
Any message between two systems is compound of 

    |___Header___||___Data___||__End_of_message__||__Checksum__|

### Header
Header is made of the following fields:
```
<starting bytes> <Device_ID> <counter>
```
|       Field        | byte size | valid byte range |              Observations                |
|:------------------:|:---------:|:----------------:|:----------------------------------------:|
|`<Starting_Bytes>`  | 3         | 255-255          | Just a sequence of 3 decimal "255" or hexadecimal "FF"|
|`<Device_ID>`       | 1         | 1-254            |       |
|`<Counter>`         | 1         | 1-254            |                                          |

 * Starting_Bytes: this is a sequence of 3 bytes which values are always `255` or `FF` in hexadecimal. This identifies the beggining of the message.
 * Device_ID: Identifies the device which will receive the message (or from which the message came). 
 * Counter: a count is made to keep tracking every message and identify them.


### Data
Data is between the header and the end of the message. Here are the parameters of the device. Each parameter value is always 16 bits preceded by an 8 bit id. Therefore every data field is 3 bytes. All the parameters must be for the same device and it cannot be more than 254 parameters.

```
... <Parameter_ID0> <Parameter_value0>, <Parameter_ID1> <Parameter_value1> ... <Parameter_IDx> <Parameter_valuex> ...
```
|       Field        | byte size | valid byte range |              Observations                |
|:------------------:|:---------:|:----------------:|:----------------------------------------:|
|`<Parameter_ID>`    | 1         | 1-254            |                                          |
|`<Parameter_value>` | 2         | 0-255            | data order `<Low_byte High_byte>`        |


 * Data_ID: every piece of data has its own id.
 * Data: Value of Data itself.

Example: send the farthest focus. The id is '2'. and the max value is '2047' ('0800' in hexadecimal).
Then the data field are the three bytes following bytes (note the low byte is first):

|   id    |    low   |  high   |
|:-------:|:--------:|:-------:|
|   02    |   00     |   08    |

### End of Message
The last byte before the Checksum and after the last command is just a zero `0` byte value.

### Checksum
Optional checking method: Sum of all bytes on the message but the `<Starting_Bytes>` using modulo 256 operation. The result is one byte after the end of message.


## Devices
On the device id field it is specified the device for the message, this field identifies the device and based on the id range the type of device is specified as well.
 * Cameras 1-99
 * Gimbals 100-149
 * Controllers 224-254
 
Every type of device has its own set of parameters which is 254 parameters per device type.

## Parameter Descriptions
### Controller Data
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 1   | CONTROL_TYPE                           |        |        |                                                                        |
| 2   | JOYSTICK0_X                            | 0      | 2048   | X coordinate of joystick 0. Central value is 1024                      |
| 3   | JOYSTICK0_Y                            | 0      | 2048   | Y coordinate of joystick 0. Central value is 1024                      |
| 4   | JOYSTICK1_X                            | 0      | 2048   | Same as 2 and 3 ids.                                                   |
| 5   | JOYSTICK1_Y                            | 0      | 2048   | Same as 2 and 3 ids.                                                   |
| 6   | JOYSTICK2_X                            | 0      | 2048   | Same as 2 and 3 ids.                                                   |
| 7   | JOYSTICK2_Y                            | 0      | 2048   | Same as 2 and 3 ids.                                                   |
| 8   | JOYSTICK3_X                            | 0      | 2048   | Same as 2 and 3 ids.                                                   |
| 9   | JOYSTICK3_Y                            | 0      | 2048   | Same as 2 and 3 ids.                                                   |
| 10  | CENTRAL_POTENTIOMETER                  | 0      | 2048   |                                                                        |
| 11  | RIGHT_POTENTIOMETER                    | 0      | 2048   |                                                                        |
| 12  | LEFT_POTENTIOMETER                     | 0      | 2048   |                                                                        |
| 13  | BUTON1_BANK1                           |        |        | values   {400, 720, 1024, 1350, 1680}                                  |
| 14  | BUTON1_BANK2                           |        |        | values   {400, 720, 1024, 1350, 1680}                                  |
| 15  | BUTON2_BANK1                           |        |        | values   {400, 720, 1024, 1350, 1680}                                  |
| 16  | BUTON2_BANK2                           |        |        | values   {400, 720, 1024, 1350, 1680}                                  |
| 17  | TRIGGER                                |        |        | values   {400, 720, 1024, 1350, 1680}                                  |
| 18  | BUTTON1                                |0       |1       | will send 1 when press and 0 when released                             |
| 19  | BUTTON2                                |0       |1       | will send 1 when press and 0 when released                             |
| 20  | BUTTON3                                |0       |1       | will send 1 when press and 0 when released                             |
| 21  | BUTTON4                                |0       |1       | will send 1 when press and 0 when released                             |


### Gimbal. Incoming Data

| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 1   | IMU_ROLL                               | -720   | 720    | Imu angles are related to motors themselves. Current value. Unit: 0.02197265625 degrees |
| 2   | IMU_PITCH                              | -720   | 720    | //                                                                     |
| 3   | IMU_YAW                                | -720   | 720    | //                                                                     |
| 4   | ROLL                                   | -720   | 720    | Relative angles are related to the position of the gimbal in the world. Current value. Unit: 0.1220740379  degrees/sec |
| 5   | PITCH                                  | -720   | 720    | //                                                                     |
| 6   | YAW                                    | -720   | 720    | //                                                                     |
| 7   | TIMESTAMP                              |        |        | Timestamp of the received angles                                       |
| 13  | ACCEL_ROLL                             | 1      | 1275   | Current acceleration value                                             |
| 14  | ACCEL_PITCH                            | 1      | 1275   | //                                                                     |
| 15  | ACCEL_YAW                              | 1      | 1275   | //                                                                     |
| 18  | ANGLE_COMPLETED                        |        |        |  Notification to confirm that a new angle was set                      |
| 19  | REQUEST_REAL_TIME_DATA                 | 0      | 65536  |  Last Real Time interval that was set                                  |
| 20  | FRAME_HEADING_ANGLE                    | -180   | 180    |  Last initial angle that was set                                       |
| 21  | BOARD_VERSION                          |        |        |  Board version multiplied by 10                                        |
| 22  | FIRMWARE_VERSION                       |        |        |  Split into decimal  digits X.XX.X, e.g. 2305 means 2.30b5             |
### Gimbal. Outgoing Data
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 4   | ROLL                                   | -720   | 720    | Set this axe angle .Unit: 0.02197265625 degrees                        |
| 5   | PITCH                                  | -720   | 720    | //                                                                     |
| 6   | YAW                                    | -720   | 720    | //                                                                     |
| 10  | SPEED_ROLL                             | 0      | 2000   | Set this axe speed. Unit: 0.1220740379  degrees/sec                    |
| 11  | SPEED_PITCH                            | 0      | 2000   | //                                                                     |
| 12  | SPEED_YAW                              | 0      | 2000   | //                                                                     |
| 13  | ACCEL_ROLL                             | 1      | 1275   | Set this axe aceleration limit. Unit: 1 degree/sec^2                   |
| 14  | ACCEL_PITCH                            | 1      | 1275   | //                                                                     |
| 15  | ACCEL_YAW                              | 1      | 1275   | //                                                                     |
| 16  | COMNTROL_MODE                          |        |        | values can be: 0 - Mode no control: gimbal ignores angle and speed data</br> 1 - Mode speed: gimbal moves to speed sent</br> 2 - mode angle: gimbal goes to specified angle using specified speed (will slow down near target speed) |
| 17  | LEVEL_ROLL                             |        |        |  Sets IMU_ROLL angle to 0                                              |
| 18  | ANGLE_COMPLETED                        |        |        |  Notification to confirm that a new angle was set                      |
| 19  | REQUEST_REAL_TIME_DATA                 | 0      | 65536  |  Sets the frequency which Real time data is received in milliseconds   |
| 20  | FRAME_HEADING_ANGLE                    | -180   | 180    |  Sets the initial angle                                                |
| 21  | BOARD_VERSION                          |        |        |  Request board information. Will return board and firmware version     |
### Black Magic Camera Data (only outgoing)
Camera parameters can be set but there is no feedback of what are the current values.

#### Lens
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 2   | Focus                                  | 0      | 2047   | 0=near, 2047=far                                                       |
| 3   | Autofocus                              |        |        |                                                                        |
| 5   | Aperture (Normalised)                  | 0      | 2047   | 0=smallest, 2047=largest                                               |
| 7   | Autoaperture                           |        |        |                                                                        |
| 8   | Optical image Stabilisation            | 0      | 1      | 0=disabled, 1 or greater=enabled                                       |
| 9   | Absolute Zoom (mm)                     | 0      | 2047   | Move to specified focal in mm, from 0mm to maximum of the lens         |   
| 10  | Absolute Zoom (Normalized)             | 0      | 2047   | Move to specified normalised focal lenght: 0=wide, 2047=tele           |
| 11  | Continous Zoom (Speed)                 | -2047  | 2047   | Start/stop zooming at specified rate: -2047=zoom wider fast, 0.0=stop, +2047=zoom tele fast|

#### Color Correction
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 20  | Lift Adjust Red                        | -4096  | 4096   |   Default value: 0                                                     | 
| 21  | Lift Adjust Green                      | -4096  | 4096   |   Default value: 0                                                     |                                                                      |
| 22  | Lift Adjust Blue                       | -4096  | 4096   |   Default value: 0                                                     |                                                                      |
| 23  | Lift Adjust Luma                       | -4096  | 4096   |   Default value: 0                                                     |                                                                      |
| 24  | Gamma Adjust Red                       | -4096  | 4096   |   Default value: 0                                                     |                                                                      |
| 25  | Gamma Adjust Green                     | -8192  | 8102   |   Default value: 0                                                     |                                                                      |
| 26  | Gamma Adjust Blue                      | -8192  | 8102   |   Default value: 0                                                     |                                                                      |
| 27  | Gamma Adjust Luma                      | -8192  | 8102   |   Default value: 0                                                     |                                                                      |
| 28  | Gain Adjust Red                        |  0     | 32768  |   Default value: 2047                                                  | 
| 29  | Gain Adjust Green                      |  0     | 32768  |   Default value: 2047                                                  |
| 30  | Gain Adjust Blue                       |  0     | 32768  |   Default value: 2047                                                  |
| 31  | Gain Adjust Luma                       |  0     | 32768  |   Default value: 2047                                                  |
| 32  | Offset Adjust Red                      | -10240 | 10240  |   Default value: 0                                                     |
| 33  | Offset Adjust Green                    | -10240 | 10240  |   Default value: 0                                                     |
| 34  | Offset Adjust Blue                     | -10240 | 10240  |   Default value: 0                                                     |
| 35  | Offset Adjust Luma                     | -10240 | 10240  |   Default value: 0                                                     |
| 36  | Contrast Adjust pivot                  | 0      | 2047   |   Default value: 0                                                     |
| 37  | Contrast Adjust adj                    | 0      | 4096   |   Default value: 2047                                                  |
| 38  | Luma Mix                               | 0      | 2047   |   Default value: 0                                                     |
| 39  | Colour Adjust Hue                      | -2047  | 2047   |   Default value: 0                                                     |
| 40  | Colour Adjust Sat                      | 0      | 4096   |   Default value: 2047                                                  |
| 41  | Correction Reset Default               | 0      | 0      |   void command                                                         |

#### Video
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 50  | Video Mode                             | -      | -      |   See video mode explanation                                           |
| 51  | Sensor Gain                            | 1      | 16     |   values: 1(12dB), 2(-6dB), 4(0dB), 8(6dB), 16(12dB)                   |
| 52  | Manual White Balance                   | 2500   | 8000   |   Corresponds to color temperature in kelvins                          |
| 53  | Exposure (us)                          | 1      | 42000  |   time in us                                                           |
| 54  | Exposure (ordinal)                     | 0      | n      |   Steps through available exposure values from 0 to the maximum of the camera | 
| 55  | Dynamic Range Mode                     | 0      | 1      |   0 = film, 1 = video                                                  |
| 56  | Video Sharpening Level                 | 0      | 3      |   0=Off, 1=Low, 2=Medium, 3=High                                       |

 * Video Mode
 

#### Audio
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 69  | Mic Level                              | 0      | 2047   |                                                                        |
| 70  | Headphone Level                        | 0      | 2047   |                                                                        |
| 71  | Headphone Program Mix                  | 0      | 2047   |                                                                        |
| 72  | Speaker Level                          | 0      | 2047   |                                                                        |
| 73  | Input Type                             | 0      | 3      |   0=internal mic, 1=line level input,  2=low mic level input,  3=high mic level input |
| 74  | Input Levels ch0                       | 0      | 2047   |                                                                        |
| 75  | Input Levels ch1                       | 0      | 2047   |                                                                        |
| 76  | Phantom Power                          | 0      | 1      |   Boolean value                                                        |

#### Display
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 89  | Brightness                             | 0      | 2047   |                                                                        |
| 90  | Overlays                               | -      | -      |   0=disable, 4=zebra, 8=peaking, 61=both                               |
| 91  | Zebra Level                            | 0      | 2047   |                                                                        |
| 92  | Peaking Level                          | 0      | 2047   |                                                                        |
| 93  | Colour Bars Display Time (seconds)     | 0      | 30     |   0=disable bars, -30=enable bars with timeout (s)                     |

#### Configuration
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 109  | Tally Brightness                      | 0      | 2047   |                                                                        |
| 110  | Tally Front Brightness                | 0      | 2047   |                                                                        |
| 111  | Tally Rear Brightness                 | 0      | 2047   |                                                                        |
| 112  | Output Overlays 112
| 113  | Reference Source 113
| 114  | Reference Offset 114
| 115  | Real Time Clock_0 115
| 116  | Real Time Clock_1 116
 
## Examples
Let's analyze a message like the next one. Note that numbers are in hexadecimal format
FFFFFF0156207211210CF22200002306F900.

|       Start        | Device Id | Count |              Data                     | End Of Message |
|:------------------:|:---------:|:-----:|:-------------------------------------:|:--------------:|
|  FFFFFF            |   01      |   01  |   20 7211, 21 0CF2, 22 0000, 23 06F9  |   00  |

In this message we can see parameters 20, 21, 22, 23, and their respective 2-Byte values.
Device Id is 01 which indicates that this message is meant for a camera.
