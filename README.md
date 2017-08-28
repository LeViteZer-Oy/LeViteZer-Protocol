# Levitezer Protocol

> Please note: Levitezer Protocol is under development, definitions are subjected to be changed

## Table of contents
- [Levitezer Protocol](#levitezer-protocol)
  * [Sckeleton](#sckeleton)
    + [Header](#header)
    + [Parameter Descriptions](#parameter-descriptions)
    + [End of Message](#end-of-message)
    + [Checksum](#checksum)
  * [Command Descriptions](#command-descriptions)
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
|`<Starting_Bytes>`  | 3         | 255-255          | just a sequence of hexadecimal "FF FF FF"|
|`<Device_ID>`       | 1         | 1-254            | 1-127 is gimbal, 128-254 is camera       |
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
Optional checking method: Sum of all bytes on the message but the `<Starting_Bytes>`

## Parameter Descriptions

### Gimbal Data
parameters received from gimbal
 * IMU Roll angle 1
 * IMU Pitch angle 2
 * IMU Yaw angle 3
 * Roll angle 4
 * Pitch angle 5
 * Yaw angle 6

### Camera Data
These control camera owns parameters, below the parameter name and the id.

#### Lens
 * Focus 2
 * Instantaneous Autofocus 3
 * Aperture (F-stop) 4
 * Aperture (Normalised) 5
 * Aperture (Ordinal) 6
 * Instantaneous Auto Aperture 7
 * Optical Image Stabilisation 8
 * Set Absolute Zoom (mm) 9
 * Set Absolute Zoom (Normalised) 10
 * Set Continuous Zoom (Speed) 11

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

In this message we can see parameters 20, 21, 22, 23, and their respective values. Device Id is 01 which indicates that this message is meant for a camera.
