
Levitezer Protocol
==================

> Please note: Levitezer Protocol is under development, definitions are subjected to be changed

# Table of contents
- [Levitezer Protocol](#levitezer-protocol)
  * [Message Structure](#message-overview)
    + [Header](#header)
    + [Data](#data)
    + [End of Message](#end-of-message)
    + [Checksum](#checksum)
  * [Parameter Descriptions](#parameter-descriptions)
    + [Data Provided by Gimbal](#data-provided-by-gimbal)
    + [Gimbal Control Data](#gimbal-control-data)
    + [Black Magic Camera Data](#black-magic-camera-data)
    + [Controller Data](#controller-data)  
  * [Examples](#examples)

# Message Structure
Any message between two systems is compound of 

    |___Header___||___Data___||__End_of_message__||__Checksum__|

There is 2 Modes for the Messages:
 * Standard Mode
 * Binary Mode
 
 
    
The byte order is always "little endian". Meaning the least significant byte is always first in any parameter. For example a 16 bit integer must be transmitted like this
 ```c
     int16 n = 0xF137;
     byte data[]{
         n & 0xFF,  /* this is 0x37 */
         n >> 8     /* this is 0xF1 */
     }
```
Then on the other end it can be reassembled:
```c
int16 n = (data[0] | data[1] << 8);
```



 #### Message Example
|   starting bytes   | device | type |    counter   | mode |               Data                    |   Checksum     |
|:------------------:|:------:|:----:|:------------:|:----:|:-------------------------------------:|:---------------:|
|0xFF 0xFF 0xFF      | 0x07   | 0x02 |  0x71        | 0x00 | 0x02 0xAA 0x92 0x33 0x41 0x00         |   0x00 0x2C 0x02  |

This message is in standard mode, is meant for Camera id 7 and contains 2 parameters in the data payload

## Header
Header is the firs 6 bytes of the message and it tells "who" sent this message (or "who" should receive it) and how to read it.

|       Field        | bit size | valid byte range |              Observations                |
|:------------------:|:---------:|:----------------:|:----------------------------------------:|
|`<Starting_Bytes>`  | 8+8+8     | 255-255          | Just a sequence of 3 decimal "255" or hexadecimal "FF"|
|`<Device_ID>`       | 8         | 0-254            |       |
|`<Device_Type>`     | 8         | 0-254            |       |
|`<Counter>`         | 7         | 0-127            |                                                      |
|`<Mode>`            | 1         | 0-1              | 0: Standard Message, 1: Binary Message             |


 * Starting_Bytes: this is a sequence of 3 bytes which values are always `255` or `FF` in hexadecimal. This identifies the starting of the message.
 * Device_ID: Identifies the device which will receive the message (or from which the message came). This id can be given by the user 

 * Device_Type: it can be one of the following:
 
|            Type Name               | Type|
|:----------------------------------:|:---:| 
| Gimbals                            |  1  |
| Cameras  (BMD)                     |  2  |
| Controllers (i.g. Joysticks)       |  3  |
| Levitezer Lens Control             |  4  |
| Box                                | 254 |
 
Every device has its own set of parameters which is up to 254 parameters.

 * Counter+Mode: Packed on the same byte is the counter and the mode
     - counter:  a counter that overflows every 127 messages. This may be useful to keep track of the message order 
     - mode: message uses standard or binary data.


## Data
### Standard Mode
Data is between the header and the end of the message. Here are the parameters of the device. Each parameter value is always 16 bits (little endian order) preceded by an 8 bit id. Therefore every data field is 3 bytes. All the parameters must be for the same device and it cannot be more than 254 parameters.


|       Field        | byte size | valid byte range |              Observations                         |
|:------------------:|:---------:|:----------------:|:-------------------------------------------------:|
|`<ID>`              | 1         | 1-254            |                                                   |
|`<VALUE>`           | 2         | 0-255            | data order is little endian `<Low_byte High_byte>`|

```
... <ID_0> <VALUE_0>, <ID_1> <VALUE_1> ... <ID_x> <VALUE_X> ...
```

Example: send the farthest focus. The id is '2'. and the max value is '2047' ('0800' in hexadecimal).
Then the data field are the three bytes following bytes (note the low byte is first):

|   id    |    low   |  high   |
|:-------:|:--------:|:-------:|
|   02    |   00     |   08    |

### Binary Mode
This mode is meant to transmit data that is not convenient on the standard 16 bit parameter mode. Such as big, grouped parameters and the ones that required conversion.
The binary Mode uses the following structure:
```
01 MM 02 DD DD 03 DD DD 04 DD DD 05 DD DD ....
```
There is a byte number followed by 2 bytes of data, for example "02 DD DD". Where "02" is the byte number and "DD" raw data.

The data starts on "02" byte number, The first "01" identify what kind of binary message is this. After "02" these numbers don't really mean much they are only required to be in the range of [02, 254]

Then the "MM" is a 16 bit number that tells how long is data and how to parse it

The maximum data size is 500 bytes per message right now.

### End of Message (or Checksum Id)
The last byte before the Checksum and after the last command is just a zero `0` byte value.

### Checksum
Sum of all bytes on the message but the `<Starting_Bytes>` using modulo 65536 operation (0x10000). The result is a 16 bit little endian.



# Parameter Descriptions


## Data Provided by Gimbal



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
| 22  | FIRMWARE_VERSION                       |        |        |  Split into decimal  digits X.XX.X, e.g. 2305 means 2.30b5            |






## Gimbal Control Data


| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 4   | ROLL                                   | -32768 | 32767  | Set this axis angle .Unit: 0.02197265625 degrees, which gives ±720 degree range  |
| 5   | PITCH                                  | -32768 | 32767  | //                                                                     |
| 6   | YAW                                    | -32768 | 32767  | //                                                                     |
| 10  | SPEED_ROLL                             | -32768 | 32767  | Set this axis speed. Unit: 0.1220740379  degrees/sec. Note, when using angle mode (Control Mode=2), the minimum speed is 0 |
| 11  | SPEED_PITCH                            | -32768 | 32767  | //                                                                     |
| 12  | SPEED_YAW                              | -32768 | 32767  | //                                                                     |
| 13  | ACCEL_ROLL                             | 0      | 1275   | Set this axis acceleration limit. Unit: 1 degree/sec^2. Note: optimal rate of sending is 1 Hz. So it should not sent which the same frequency than others. Note 2: 0 Acceleration will disable this axis movement altogether.  |
| 14  | ACCEL_PITCH                            | 0      | 1275   | //                                                                     |
| 15  | ACCEL_YAW                              | 0      | 1275   | //                                                                     |
| 16  | CONTROL_MODE                           |        |        | values can be: 0 - Mode no control: gimbal ignores angle and speed data</br> 1 - Mode speed: gimbal moves to speed sent. Note: Optimal rate of sending speed is 50-100Hz</br> 2 - mode angle: gimbal goes to specified angle using specified speed (will slow down near target speed) |
| 17  | LEVEL_ROLL                             |        |        |  Sets IMU_ROLL angle to 0                                              |
| 18  | ANGLE_COMPLETED                        |        |        |  Notification to confirm that a new angle was set                      |
| 19  | REQUEST_REAL_TIME_DATA                 | 0      | 65536  |  Sets the frequency which Real time data is received in milliseconds   |
| 21  | BOARD_VERSION                          |        |        |  Request board information. Will return board and firmware version     |
| 23  | ROLL_OFFSET                            | -32768 |  32767 | Offsets the specified amount to the the axis                           |
| 24  | PITCH_OFFSET                           | -32768 |  32767 | //                                                                       |
| 25  | YAW_OFFSET                             | -32768 |  32767 | //                                                                       |
<!---| 60  | GIMBAL_LAT0                            | 0      | 65536  | See [Gimbal Geo Point control](#Gimbal-Geo-Point-control)              |
| 61  | GIMBAL_LAT1                            | 0      | 65536  | //                                                                     |
| 62  | GIMBAL_LON0                            | 0      | 65536  | //                                                                     |
| 63  | GIMBAL_LON1                            | 0      | 65536  | //                                                                     |
| 64  | GIMBAL_ALT0                            | 0      | 65536  | //                                                                     |
| 65  | GIMBAL_ALT1                            | 0      | 65536  | //                                                                     |
| 66  | TARGET_LAT0                            | 0      | 65536  | //                                                                     |
| 67  | TARGET_LAT1                            | 0      | 65536  | //                                                                     |
| 68  | TARGET_LON0                            | 0      | 65536  | //                                                                     |
| 69  | TARGET_LON1                            | 0      | 65536  | //                                                                     |
| 70  | TARGET_ALT0                            | 0      | 65536  | //                                                                     |
| 71  | TARGET_ALT1                            | 0      | 65536  | //                                                                     |
-->
Notes:
When controling the gimbal on speed mode you should send CONTROL_MODE=1 + speed axis. Also use a high rate send in the range of 50-100 Hz.

When controling the gimbal on angle mode you should send CONTROL_MODE=2 + speed axis + angle axis. On this case the speed will use to reach the target angle.

#### GPS Data Structure [Still on development]

GPS data is sent on Binary mode. There is 2 Ids; one for the Gimbal coordinates and the other for target coordinates.

| Binary Id  |  byte size  |             name          |                  Observations                          |
|:----------:|:-----------:|:-------------------------:|:------------------------------------------------------ |
|   502      |    40       | Gimbal GPS Coordinates    |  See Table                                             |
|   503      |    40       | Target GPS Coordinates    |  See Table                                             |

Both GPS binary message are the same structure. The parameters are based on C language types and they are packet on the same order as in the following table

| name                     |                  Observations                          |
|:------------------------:|:-------------------------------------------------------|
|  Latitude                |  64 bit IEEE-754 Double float                          |
|  Longitude               |  64 bit IEEE-754 Double float                          |
|  Altitude                |  64 bit IEEE-754 Double float                          |
|  Heading                 |  32 bit IEEE-754 Single float                          |
|  Speed                   |  32 bit IEEE-754 Single float                          |
|  Timestamp               |  32 bit Unsigned interger                              |
|  day                     |  8 bit Unsigned interger                               |
|  month                   |  8 bit Unsigned interger                               |
|  year                    |  8 bit Unsigned interger                               |
|  status                  |  8 bit Unsigned interger                               |


Since datatypes are C language based. You can take advantage of C Unions to pack and receive the data without doing any extra conversions. you just need to copy the binary data to the `dataArray` field inside the Union, after that all parameters are available. And the other way around also works; you set every parameter and `dataArray` is ready to be sent with all the data.
```c
union GpsDataUnion{
    struct  GpsData{
        double lat;
        double lon;
        double alt;
        float heading;
        float speed;
        uint32_t timestamp;
        uint8_t day;
        uint8_t month;
        uint8_t year;
        uint8_t status;
    } data;
    uint8_t dataArray[sizeof(GpsData)];
};
```
Also on python we also can use c unions through ctypes.
```python
from ctypes import (
        Union, Array, Structure,
        c_uint8, c_uint32, c_float, c_double
)

ARRAY_SIZE = 40

class uint8_array(Array):
        _type_ = c_uint8
        _length_ = ARRAY_SIZE

class gps_data(Structure):
        _fields_ = (
		("lat", c_double),
		("lon", c_double),
		("alt", c_double),
		("heading", c_float),
		("speed", c_float),
		("timestamp", c_uint32),
		("day", c_uint8),
		("month", c_uint8),
		("year", c_uint8),
		("status", c_uint8)
		)
  
class gps_data_union(Union):
    _fields_ = (
    ("data", gps_data),
    ("byteArray", uint8_array)
    )
```
<!---
#### Gimbal Geo Point control
One way of controling the gimbal is by pointing to a target geo coordinate (lat, lon, alt) relative to the own gimbal geo coordinate. It will move the gimbal yaw and pitch.

![map](map.png)

Latitudes and Longitudes are in decimal format multiplied by 10^7.
Altitudes are in decimal format multiplied by 100.
```python
lat = 60.2241444 * 1e7 # = 602241444
lon = 24.7578934 * 1e7 # = 247578934
alt = 112.55 * 100     # = 11255
```

Since every parameter in LeViteZer protocol is 16 bits, there are two ids per coordinate component. To do this we split every component to 2 16 bit variables:
```python
lat_0 = lat & 0xFFFF
lat_1 = lat >> 16
lon_0 = alt & 0xFFFF
lon_1 = alt >> 16
alt_0 = alt & 0xFFFF
alt_1 = alt >> 16
```
Latitude range is From -90 (south pole) to 90 degrees (north pole);
Longitude range is From -180 to 180 degrees

-->

## Black Magic Camera Data

Camera parameters can be set but there is no feedback of what are the current values.
Camera parameters must be send at rates below 24 Hz. If they are sent at higher frequency for short period, they will be enqueued and eventually sent to camera, but if the queue gets full then new data will be dropped. This is because a limitation on the SDI interface.

### Lens
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 2   | Focus                                  | 0      | 2047   | 0=near, 2047=far                                                       |
| 3   | Autofocus                              |        |        |                                                                        |
| 5   | Aperture (Normalised)                  | 0      | 2047   | 0=smallest, 2047=largest                                               |
| 7   | Autoaperture                           |        |        |                                                                        |
| 8   | Optical image Stabilization            | 0      | 1      | 0=disabled, 1 or greater=enabled                                       |
| 9   | Absolute Zoom (mm)                     | 0      | 2047   | Move to specified focal in mm, from 0mm to maximum of the lens         |   
| 10  | Absolute Zoom (Normalized)             | 0      | 2047   | Move to specified normalised focal lenght: 0=wide, 2047=tele           |
| 11  | Continous Zoom (Speed)                 | -2048  | 2047   | Start/stop zooming at specified rate: -2047=zoom wider fast, 0.0=stop, +2047=zoom tele fast|

### Color Correction
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

### Video
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 50  | Video Mode                             | -      | -      |   See video mode explanation                                           |
| 51  | Sensor Gain                            | 1      | 16     |   values: 1(12dB), 2(-6dB), 4(0dB), 8(6dB), 16(12dB)                   |
| 52  | Manual White Balance                   | 2500   | 8000   |   Corresponds to color temperature in kelvins                          |
| 53  | Exposure (us)                          | 1      | 42000  |   time in us                                                           |
| 54  | Exposure (ordinal)                     | 0      | n      |   Steps through available exposure values from 0 to the maximum of the camera | 
| 55  | Dynamic Range Mode                     | 0      | 1      |   0 = film, 1 = video                                                  |
| 56  | Video Sharpening Level                 | 0      | 3      |   0=Off, 1=Low, 2=Medium, 3=High                                       |

 ### Video mode
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
     colorspace =    (videomode & 0b0000111100000000) >> 8
     
     # to set the values to videomode parameter
     videomode = (fps | (mrate << 3) | (resolution << 4) | (interlased << 7) | (colorspace << 8)
     
 ```
 FPS values are from first bit to the 3rd, M-rate is the 4th bit, resolution from the 5 fifth to the 7th and so.

### Audio
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

### Display
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 89  | Brightness                             | 0      | 2047   |                                                                        |
| 90  | Overlays                               | -      | -      |   0=disable, 4=zebra, 8=peaking, 61=both                               |
| 91  | Zebra Level                            | 0      | 2047   |                                                                        |
| 92  | Peaking Level                          | 0      | 2047   |                                                                        |
| 93  | Colour Bars Display Time (seconds)     | 0      | 30     |   0=disable bars, -30=enable bars with timeout (s)                     |

### Configuration
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 109  | Tally Brightness                      | 0      | 2047   |                                                                        |
| 110  | Tally Front Brightness                | 0      | 2047   |                                                                        |
| 111  | Tally Rear Brightness                 | 0      | 2047   |                                                                        |
### PTZ control
| Id  |                 name                   |  min   | max    |                                  Observations                          |
|:---:|----------------------------------------|:------:|:------:|------------------------------------------------------------------------|
| 120  | Pan                                   | -2047      | 2047   |   Pan speed                                                                     |
| 121  | Tilt                                  | -2047      | 2047   |   Tilt speed                                                                     |
| 121  | Operation                             | 0          | 2   |  0=reset, 1=save position, 2=recall position                                  |
| 123  | Memory slot                           | 0          | 5   |    memory slot to use a operation                                   |

## Controller Data
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

 


# Examples
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
    for i in range(3, len(message)):
        checksum = (checksum + message[i]) & 0xFFFF # 16 bit overflow 
   
   #set the 16bit checksum at the end of the array
    message[-2] = checksum & 0xff
    message[-1] = checksum >> 8

    # add to the counter
    counter = (counter+1) & 127  # counter overflows after 127 (7 bit counter)

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

```
