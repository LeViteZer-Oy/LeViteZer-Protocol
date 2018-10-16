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
    + [Data Provided by Gimbal](#data-provided-by-gimbal)
    + [Gimbal Control Data](#gimbal-control-data)
    + [Black Magic Camera Data](#black-magic-camera-data)
    + [Controller Data](#controller-data)  
  * [Future parameters](#future-parameters)
  * [Examples](#examples)
  
## Sckeleton
Any message between two systems is compound of 

    |___Header___||___Data___||__End_of_message__||__Checksum__|


 ### Message Example
|   starting bytes   | device | type |    counter   |                     Data              |   Checksum     |
|:------------------:|:------:|:----:|:------------:|:-------------------------------------:|:------------:|
|0xFF 0xFF 0xFF      | 0x07   | 0x02 |  0xA1        | (0x02 0xAA 0x92) (0x33 0x41 0x00)     |   0x00 0x5C 0x02  |

This message is for a Camera and contains 2 parameters in the data

### Header
Header is made of the following 6 bit fields:
```
<starting bytes> <Device_ID> <Device_Type_ID> <counter>
```
|       Field        | byte size | valid byte range |              Observations                |
|:------------------:|:---------:|:----------------:|:----------------------------------------:|
|`<Starting_Bytes>`  | 3         | 255-255          | Just a sequence of 3 decimal "255" or hexadecimal "FF"|
|`<Device_ID>`       | 1         | 0-254            |       |
|`<Device_Type>`     | 1         | 0-254            |       |
|`<Counter>`         | 1         | 0-254            |                                          |

 * Starting_Bytes: this is a sequence of 3 bytes which values are always `255` or `FF` in hexadecimal. This identifies the beggining of the message.
 * Device_ID: Identifies the device which will receive the message (or from which the message came). 
 * Counter: a count is made to keep tracking every message and identify them.




### Data
Data is between the header and the end of the message. Here are the parameters of the device. Each parameter value is always 16 bits (little endian order) preceded by an 8 bit id. Therefore every data field is 3 bytes. All the parameters must be for the same device and it cannot be more than 254 parameters.

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

### End of Message (or Checksum Id)
The last byte before the Checksum and after the last command is just a zero `0` byte value.

### Checksum
Sum of all bytes on the message but the `<Starting_Bytes>` using modulo 65536 operation (0x10000). The result is a 16 bit little endian.

## Devices
#### Levitezer Control Box
The protocol messages can control devices connected to the Levitezer Control Box. The Box can receive and send these messages through its serial/USB port or through Ethernet (UDP packets).

#### Available devices
Depending on the device type value in the header, the message is meant for one of the following groups:
 * Gimbals                              1
 * Cameras  (BM)                        2
 * Controllers (i.g. Joysticks)         3
 * General Purpose                      0xFE
 
Every type of device has its own set of parameters which is 254 parameters per device type.

## Parameter Descriptions

### Data Provided by Gimbal


| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 1   | IMU_ROLL                               | -32768 |   32767| Current IMU angles (relative to motors themselves). Unit: 0.02197265625 degrees, which gives ±720 degree range |
| 2   | IMU_PITCH                              | -32768 |   32767| //                                                                     |
| 3   | IMU_YAW                                | -32768 |   32767| //                                                                     |
| 4   | ROLL                                   | -32768 |   32767| Current relative angles (relative to the the gimbal frame) Unit: 0.02197265625 degrees, which gives ±720 degree range |
| 5   | PITCH                                  | -32768 |   32767| //                                                                     |
| 6   | YAW                                    | -32768 |   32767| //                                                                     |
| 7   | TIMESTAMP                              |        |        | Timestamp of the received angles                                       |
| 13  | ACCEL_ROLL                             | 0      | 1275   | Current acceleration value                                             |
| 14  | ACCEL_PITCH                            | 0      | 1275   | //                                                                     |
| 15  | ACCEL_YAW                              | 0      | 1275   | //                                                                     |
| 18  | ANGLE_COMPLETED                        |        |        |  Notification to confirm that a new angle was set                      |
| 19  | REQUEST_REAL_TIME_DATA                 | 0      | 65536  |  Last Real Time interval that was set                                  |
| 21  | BOARD_VERSION                          |        |        |  Board version multiplied by 10                                        |
| 22  | FIRMWARE_VERSION                       |        |        |  Split into decimal  digits X.XX.X, e.g. 2305 means 2.30b5             |

### Gimbal Control Data


| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 4   | ROLL                                   | -32768 | 32767  | Set this axis angle .Unit: 0.02197265625 degrees, which gives ±720 degree range  |
| 5   | PITCH                                  | -32768 | 32767  | //                                                                     |
| 6   | YAW                                    | -32768 | 32767  | //                                                                     |
| 10  | SPEED_ROLL                             | -32768 | 32767  | Set this axis speed. Unit: 0.1220740379  degrees/sec. Note, when using angle mode (Control Mode=2), the minimum speed is 0 |
| 11  | SPEED_PITCH                            | -32768 | 32767  | //                                                                     |
| 12  | SPEED_YAW                              | -32768 | 32767  | //                                                                     |
| 13  | ACCEL_ROLL                             | 0      | 1275   | Set this axis aceleration limit. Unit: 1 degree/sec^2. Note: optimal rate of sending is 1 Hz. So it should not sent which the same frequency than others. Note 2: 0 Acceleration will disable this axis movement altogether.  |
| 14  | ACCEL_PITCH                            | 0      | 1275   | //                                                                     |
| 15  | ACCEL_YAW                              | 0      | 1275   | //                                                                     |
| 16  | CONTROL_MODE                           |        |        | values can be: 0 - Mode no control: gimbal ignores angle and speed data</br> 1 - Mode speed: gimbal moves to speed sent. Note: Optimal rate of sending speed is 50-100Hz</br> 2 - mode angle: gimbal goes to specified angle using specified speed (will slow down near target speed) |
| 17  | LEVEL_ROLL                             |        |        |  Sets IMU_ROLL angle to 0                                              |
| 18  | ANGLE_COMPLETED                        |        |        |  Notification to confirm that a new angle was set                      |
| 19  | REQUEST_REAL_TIME_DATA                 | 0      | 65536  |  Sets the frequency which Real time data is received in milliseconds   |
| 21  | BOARD_VERSION                          |        |        |  Request board information. Will return board and firmware version     |

Notes:
When controling the gimbal on speed mode you should send CONTROL_MODE=1 + speed axis. Also use a high rate send in the range of 50-100 Hz.

When controling the gimbal on angle mode you should send CONTROL_MODE=2 + speed axis + angle axis. On this case the speed will use to reach the target angle.

### Black Magic Camera Data

Camera parameters can be set but there is no feedback of what are the current values.
Camera parameters must be send at rates below 24 Hz. If they are sent at higher frequency for short period, they will be enqueued and eventually sent to camera, but if the queue gets full then new data will be dropped. This is because a limitation on the SDI interface.

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
| 11  | Continous Zoom (Speed)                 | -2048  | 2047   | Start/stop zooming at specified rate: -2047=zoom wider fast, 0.0=stop, +2047=zoom tele fast|

#### Color Correction
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 20  | Lift Adjust Red                        | -4096  | 4095   |   Default value: 0                                                     | 
| 21  | Lift Adjust Green                      | -4096  | 4095   |   Default value: 0                                                     |                                                                      |
| 22  | Lift Adjust Blue                       | -4096  | 4095   |   Default value: 0                                                     |                                                                      |
| 23  | Lift Adjust Luma                       | -4096  | 4095   |   Default value: 0                                                     |                                                                      |
| 24  | Gamma Adjust Red                       | -4096  | 4095   |   Default value: 0                                                     |                                                                      |
| 25  | Gamma Adjust Green                     | -8192  | 8101   |   Default value: 0                                                     |                                                                      |
| 26  | Gamma Adjust Blue                      | -8192  | 8101   |   Default value: 0                                                     |                                                                      |
| 27  | Gamma Adjust Luma                      | -8192  | 8101   |   Default value: 0                                                     |                                                                      |
| 28  | Gain Adjust Red                        |  0     | 32767  |   Default value: 2047                                                  | 
| 29  | Gain Adjust Green                      |  0     | 32767  |   Default value: 2047                                                  |
| 30  | Gain Adjust Blue                       |  0     | 32767  |   Default value: 2047                                                  |
| 31  | Gain Adjust Luma                       |  0     | 32767  |   Default value: 2047                                                  |
| 32  | Offset Adjust Red                      | -10240 | 10240  |   Default value: 0                                                     |
| 33  | Offset Adjust Green                    | -10240 | 10240  |   Default value: 0                                                     |
| 34  | Offset Adjust Blue                     | -10240 | 10240  |   Default value: 0                                                     |
| 35  | Offset Adjust Luma                     | -10240 | 10240  |   Default value: 0                                                     |
| 36  | Contrast Adjust pivot                  | 0      | 2047   |   Default value: 0                                                     |
| 37  | Contrast Adjust adj                    | 0      | 4095   |   Default value: 2047                                                  |
| 38  | Luma Mix                               | 0      | 2047   |   Default value: 0                                                     |
| 39  | Colour Adjust Hue                      | -2047  | 2047   |   Default value: 0                                                     |
| 40  | Colour Adjust Sat                      | 0      | 4095   |   Default value: 2047                                                  |
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

 #### Video mode
 sets resolution and framerate. all the settings are in groups of bits as show from the smallest bit:
  - 3 bits -> FPS: 0=24, 1=25, 2=30, 3=50, 4=60
  - 1 bit  -> M-Rate: 0=regular, 1=M-rate
  - 3 bits -> Dimension:  0=NTSC, 1=PAL, 2=720, 3=1080, 4=2k, 5=2k DCI, 6=4k, 7=4k DCI
  - 1 bit  -> interlaced: 0=progressive, 1=interlaced
  - 4 bits -> colourspace: 0=YUV
```python
     # if videomode is a variable that represents the parameter
     videomode
     
     # to get the values from videomode parameter
     fps =            videomode & 0b0000000000000111
     mrate =         (videomode & 0b0000000000001000) >> 3
     resolution =    (videomode & 0b0000000001110000) >> 4
     interlased =    (videomode & 0b0000000010000000) >> 7
     colorspace =    (videomode & 0b0000111100000000) >> 8
     
     # to set the values to videomode parameter
     videomode = (fps | (mrate << 3) | (resolution << 4) | (interlased << 7) | (colorspace << 8)
     
 ```
 FPS values are from first bit to the 3rd, M-rate is the 4th bit, resolution from the 5 fifth to the 7th and so.

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

 
 
## Future parameters
There are some parameters that are not implemented yet. There are plans to add them as well.

### Black magic cameras
 #### Video
 * Set auto White Balance
 * Restore auto White Balance
 * Recording format
 * Set auto exposure mode (manual, iris, shutter, iris + shutter, ... etc)
 #### Output
 * Frame overlays
 * Frame guides style (Camera 3.x)
 * Frame guides opacity (Camera 3.x)
 #### Display
 * Focus Assit
 #### Reference
 * Souce
 * Offset in pixels
 #### Configuration
 * Real Time Clock
 * System languague
 * Timezone
 * Location

### Gimbal
 * Set angle 0: define where the angle 0 in the yaw axis (pan)

## Examples
Let's analyze a message like the next one. Note that numbers are in hexadecimal format
FFFFFF0156207211210CF22200002306F900.

|       Start        | Device Id | Count |              Data                     | End Of Message | Checksum |
|:------------------:|:---------:|:-----:|:-------------------------------------:|:--------------:|:--------:|  
|  FFFFFF            |   01      |   01  |   20 7211, 21 0CF2, 22 0000, 23 06F9  |   00           |    B2    |

In this message we can see parameters 20, 21, 22, 23, and their respective 2-Byte values.
Device Id is 01 which indicates that this message is meant for a camera.

To calculate the checksum sum all the bytes of all fields but the starting bytes and apply modulo by 256 to the sum.
 * `1+1+20+72+11+21+0c+f2+22+23+06+f9 = 308`
 * `308 % 256 = B2`
 
If your using a 8 bit variable for the checksum you don't need to use modulo 256, the variable overflows any time its value gets bigger than 255. For example in C language you may use any of the following data types:
* unsigned char
* uint8_t
* byte

Otherwise you will need to calculate modulo after summing. (see script example)

### Script example
#### This example is for the old version of the protocol. We will update this soon.
The following python 2.7 script moves the gimbal several times on the yaw axis, sending the control messages trough UDP using the following gimbal parameters:
 * ROLL
 * PICH
 * YAW
 * SPEED_ROLL
 * SPEED_PICH
 * SPEED_YAW
 * CONTROL_MODE
 
``` python
import socket
import array
import time
UDP_IP = "192.168.137.222"
UDP_PORT = 50505

global counter
counter = 0

def getAngleMessage(angle):
    global counter
    if angle > 720 or angle < -720:
        print "no valid angle range"
        return ""

    # convert angle to raw value
    rawYaw = int(angle / 0.02197265625)

    gimbalId = 101
    gimbalType = 1
    mode = 2
    endOfMessage = 0
    message = array.array('B', [0xff, 0xff, 0xff, gimbalId, gimbalType, counter,
            4, 0, 0, 5, 0, 0, 6, rawYaw & 0xff, rawYaw >> 8, 10, 0, 0, 11, 0, 0, 12, 255, 3, 16, mode, 0,
            endOfMessage, 0, 0])

    #calculate checksum starting in 3rd index
    checksum = 0
    aSum = 0
    for i in range(3, len(message)):
        aSum += message[i]
    #set the 16bit checksum at the end of the array
    checksum = aSum % (2**16 -1)
    message[-2] = checksum & 0xff
    message[-1] = checksum >> 8

    # add to the counter
    if counter > 255:
        counter = 0
    else:
        counter += 1

    return message.tostring()


# create socket
print "UDP target IP:", UDP_IP
print "UDP target port:", UDP_PORT
sock = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)

# move to 0
sock.sendto(getAngleMessage(0), (UDP_IP, UDP_PORT))
# move to 90
time.sleep(3)
sock.sendto(getAngleMessage(90), (UDP_IP, UDP_PORT))
# move to 180
time.sleep(2)
sock.sendto(getAngleMessage(180), (UDP_IP, UDP_PORT))
# move to 270
time.sleep(2)
sock.sendto(getAngleMessage(270), (UDP_IP, UDP_PORT))

# receive data (it blocks)
# data, addr = sock.recvfrom(1024)
# print "received message:", data

```
