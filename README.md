# Levitezer Protocol

## Table of contents
- [Levitezer Protocol](#levitezer-protocol)
  * [Sckeleton](#sckeleton)
    + [Header](#header)
    + [Commands](#commands)
    + [End of Message](#end-of-message)
    + [Checksum](#checksum)
  * [Command Descriptions](#command-descriptions)
    + [Gimbal Commands](#gimbal-commands)
    + [Camera commands](#camera-commands)
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

## Command Descriptions

### Gimbal Commands
 * Upate Yaw speed
 * Upate Pith speed
 * Upate Roll speed
 * Upate Yaw angle
 * Upate Pitch angle
 * Upate Roll angle
 * Real Time Data

### Camera commands
 * Zoom
 * Dist
 
## Examples
 * 255, 255, 255, Camera_ID, Counter, Data_ID, 3, Dist_L, Dist_H, 4, Zoom_L, Zoom_H, 0, Check_Sum_L Chekc_Sum_H
 * 255, 255, 255, Gimbal_ID, Counter, Data_ID, Yaw_L, Yaw_H, 2, Pitch_L, Pitch_H, 0, Check_Sum_L Chekc_Sum_H
