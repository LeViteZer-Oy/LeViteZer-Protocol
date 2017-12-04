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
|`<Starting_Bytes>`  | 3         | 255-255          | Just a sequence of hexadecimal "FF FF FF"|
|`<Device_ID>`       | 1         | 1-254            |       |
|`<Counter>`         | 1         | 1-254            |                                          |

 * Starting_Bytes: this is a sequence of 3 bytes which values are always `255` or `FF` in hexadecimal. This identifies the beggining of the message.
 * Device_ID: Identifies the device which will receive the message (or from which the message came). 
 * Counter: a count is made to keep tracking every message and identify them.


### Data
Data is between the header and the end of the message. Any data field has 2 bytes and is preceeded by its ID byte. these data fields can come in any order and number (max 254).
```
... <Data_ID> <Low_Byte High_Byte> ...
```
|       Field        | byte size | valid byte range |              Observations                |
|:------------------:|:---------:|:----------------:|:----------------------------------------:|
|`<Data_ID>`         | 1         | 1-254            |                                          |
|`<Data>`            | 2         | 0-255            | data order `<Low_byte High_byte>`        |


 * Data_ID: every piece of data has its own id.
 * Data: Value of Data itself.

>Note: Data fields are always optional

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
 * Lift Adjust Red 20
 * Lift Adjust Green 21
 * Lift Adjust Blue 22
 * Lift Adjust Luma 23
 * Gamma Adjust Red  24
 * Gamma Adjust Green 25
 * Gamma Adjust Blue 26
 * Gamma Adjust Luma 27
 * Gain Adjust Red 28
 * Gain Adjust Green 29
 * Gain Adjust Blue 30
 * Gain Adjust Luma 31
 * Offset Adjust Red 32
 * Offset Adjust Green 33
 * Offset Adjust Blue 34
 * Offset Adjust Luma 35
 * Contrast Adjust pivot 36
 * Contrast Adjust adj 37
 * Luma Mix 38
 * Colour Adjust Hue 39
 * Colour Adjust Sat 40
 * Correction Reset Default 41

#### Video
 * Video Mode 50
 * Sensor Gain 51
 * Manual White Balance 52
 * Exposure (us) 53
 * Exposure (ordinal) 54
 * Dynamic Range Mode 55
 * Video Sharpening Level 56

#### Audio
 * Mic Level 69
 * Headphone Level 70
 * Headphone Program Mix 71
 * Speaker Level 72
 * Input Type 73
 * Input Levels ch0 74
 * Input Levels ch1 75
 * Phantom Power 76

#### Display
 * Brightness 89
 * Overlays 90
 * Zebra Level 91
 * Peaking Level 92
 * Colour Bars Display Time (seconds) 93

#### Configuration
 * Tally Brightness 109
 * Tally Front Brightness 110
 * Tally Rear Brightness 111
 * Output Overlays 112
 * Reference Source 113
 * Reference Offset 114
 * Real Time Clock_0 115
 * Real Time Clock_1 116
 
## Examples
Let's analyze a message like the next one. Note that numbers are in hexadecimal format
FFFFFF0156207211210CF22200002306F900.

|       Start        | Device Id | Count |              Data                     | End Of Message |
|:------------------:|:---------:|:-----:|:-------------------------------------:|:--------------:|
|  FFFFFF            |   01      |   01  |   20 7211, 21 0CF2, 22 0000, 23 06F9  |   00  |

In this message we can see parameters 20, 21, 22, 23, and their respective 2-Byte values.
Device Id is 01 which indicates that this message is meant for a camera.
