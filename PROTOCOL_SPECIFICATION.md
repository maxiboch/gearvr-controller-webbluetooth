# GearVR Controller Bluetooth Protocol Specification

This document provides detailed technical specifications for the GearVR Controller Bluetooth protocol, extracted from the existing Web Bluetooth implementation.

---

## Bluetooth Low Energy (BLE) Configuration

### Device Information
- **Device Type**: Bluetooth Low Energy (BLE) HID Controller
- **Product**: Samsung Gear VR Controller
- **Bluetooth Version**: BLE 4.0+

### GATT Services

#### Custom Service
- **UUID**: `4f63756c-7573-2054-6872-65656d6f7465`
- **Description**: Primary service for controller communication
- **Characteristics**: 2 (Write, Notify)

#### Characteristics

##### Write Characteristic
- **UUID**: `c8c51726-81bc-483b-a052-f7a14ea3d282`
- **Properties**: Write
- **Usage**: Send commands to controller
- **Format**: Little-endian 2-byte hex commands

##### Notify Characteristic
- **UUID**: `c8c51726-81bc-483b-a052-f7a14ea3d281`
- **Properties**: Notify
- **Usage**: Receive sensor and button data from controller
- **Format**: 60-byte binary data packets
- **Update Rate**: ~14.55 Hz per sample, 3 samples per notification (~43.65 Hz effective)

---

## Command Protocol

### Command Format
Commands are sent as 2-byte little-endian hex values via the Write characteristic.

### Command List

| Command Name | Value (Hex) | Value (Decimal) | Description |
|--------------|-------------|-----------------|-------------|
| CMD_OFF | 0x0000 | 0 | Power off / disable sensors |
| CMD_SENSOR | 0x0100 | 256 | Enable sensor data streaming |
| CMD_UNKNOWN_FIRMWARE_UPDATE | 0x0200 | 512 | Unknown - possibly firmware related |
| CMD_CALIBRATE | 0x0300 | 768 | Trigger controller calibration |
| CMD_KEEP_ALIVE | 0x0400 | 1024 | Keep-alive ping to maintain connection |
| CMD_UNKNOWN_SETTING | 0x0500 | 1280 | Unknown setting command |
| CMD_LPM_ENABLE | 0x0600 | 1536 | Enable Low Power Mode |
| CMD_LPM_DISABLE | 0x0700 | 1792 | Disable Low Power Mode |
| CMD_VR_MODE | 0x0800 | 2048 | Enable VR mode |

### Initialization Sequence
Typical startup sequence:
1. Pair and connect to device
2. Discover services and characteristics
3. Subscribe to notify characteristic
4. Send `CMD_VR_MODE` (0x0800)
5. Send `CMD_SENSOR` (0x0100)
6. Repeat CMD_VR_MODE → CMD_SENSOR cycle 2-3 times for reliability

### Keep-Alive
- Periodic `CMD_KEEP_ALIVE` (0x0400) should be sent to maintain connection
- Recommended interval: TBD (needs testing)

---

## Data Packet Structure

### Notification Data Format
Each notification contains 60 bytes of binary data with the following structure:

#### Byte Map

| Byte Offset | Size (bytes) | Data Type | Description |
|-------------|--------------|-----------|-------------|
| 0-2 | 3 | Int32 (partial) | Timestamp (microseconds) |
| 3 | 1 | - | Unused/padding |
| 4-31 | 28 | Int16 array | IMU data (accelerometer + gyroscope, 3 samples) |
| 32-37 | 6 | Int16 array | Magnetometer data (x, y, z) |
| 38-53 | 16 | Int16 array | Additional IMU samples (2 more sets) |
| 54-56 | 3 | Packed bits | Touchpad coordinates |
| 57 | 1 | Uint8 | Temperature |
| 58 | 1 | Bitfield | Button states |
| 59 | 1 | - | Unused/padding |

### Detailed Field Descriptions

#### Timestamp (Bytes 0-2)
```
value = (buffer[0] | (buffer[1] << 8) | (buffer[2] << 16)) & 0xFFFFFFFF
timestamp_seconds = value / 1000 * 0.001
// Alternative: timestamp_seconds = value / 1000000
```
- **Type**: Unsigned 32-bit integer (only lower 24 bits stored)
- **Unit**: Microseconds
- **Conversion**: Divide by 1,000,000 for seconds
- **Factor**: 0.001 (TIMESTAMP_FACTOR)

#### Accelerometer Data

**Sampling**: 3 samples per notification
**Location**: 
- Sample 0: Bytes 4-9 (X: 4-5, Y: 6-7, Z: 8-9)
- Sample 1: Bytes 20-25
- Sample 2: Bytes 36-41

**Parsing**:
```c
// For each axis (X, Y, Z) and sample (0, 1, 2)
int16_t raw_value = *(int16_t*)(buffer + 16*sample_index + axis_offset);
float accel_g = raw_value * 10000.0 * 9.80665 / 2048.0;
// Final scaling: multiply by 0.00001 (ACCEL_FACTOR)
```

- **Type**: Signed 16-bit integer per axis
- **Axes**: X, Y, Z
- **Unit**: Raw sensor units
- **Conversion**: `raw * 10000.0 * 9.80665 / 2048.0 * 0.00001` → g-force
- **Factor**: 0.00001 (ACCEL_FACTOR)
- **Range**: ±2g (estimated)

#### Gyroscope Data

**Sampling**: 3 samples per notification
**Location**:
- Sample 0: Bytes 10-15 (X: 10-11, Y: 12-13, Z: 14-15)
- Sample 1: Bytes 26-31
- Sample 2: Bytes 42-47

**Parsing**:
```c
// For each axis (X, Y, Z) and sample (0, 1, 2)
int16_t raw_value = *(int16_t*)(buffer + 16*sample_index + axis_offset);
float gyro_rad = raw_value * 10000.0 * 0.017453292 / 14.285;
// Final scaling: multiply by 0.0001 (GYRO_FACTOR)
```

- **Type**: Signed 16-bit integer per axis
- **Axes**: X, Y, Z (angular velocity)
- **Unit**: Raw sensor units
- **Conversion**: `raw * 10000.0 * 0.017453292 / 14.285 * 0.0001` → radians/second
- **Factor**: 0.0001 (GYRO_FACTOR)
- **Range**: ±2000 degrees/second (estimated)

#### Magnetometer Data

**Sampling**: 1 sample per notification (shared across IMU samples)
**Location**: Bytes 32-37 (X: 32-33, Y: 34-35, Z: 36-37)

**Parsing**:
```c
int16_t raw_x = *(int16_t*)(buffer + 32);
int16_t raw_y = *(int16_t*)(buffer + 34);
int16_t raw_z = *(int16_t*)(buffer + 36);
float mag_x = raw_x * 0.06;
float mag_y = raw_y * 0.06;
float mag_z = raw_z * 0.06;
```

- **Type**: Signed 16-bit integer per axis
- **Axes**: X, Y, Z (magnetic field)
- **Unit**: Raw sensor units
- **Conversion**: `raw * 0.06` → microTesla (μT)
- **Factor**: 0.06

#### Touchpad Coordinates (Bytes 54-56)

**Encoding**: 10-bit values packed into 3 bytes

**Parsing**:
```c
// X coordinate (10 bits)
uint16_t axisX = ((buffer[54] & 0x0F) << 6) | ((buffer[55] & 0xFC) >> 2);
axisX &= 0x3FF; // Mask to 10 bits

// Y coordinate (10 bits)
uint16_t axisY = ((buffer[55] & 0x03) << 8) | (buffer[56] & 0xFF);
axisY &= 0x3FF; // Mask to 10 bits
```

- **Type**: 10-bit unsigned integers (packed)
- **X Range**: 0-1023 (0x000-0x3FF)
- **Y Range**: 0-1023 (0x000-0x3FF)
- **Physical Range**: ~0-315mm (touchpad dimension)
- **Center**: ~512, 512
- **Bit Layout**:
  - Byte 54 [7:4]: X[9:6]
  - Byte 54 [3:0]: X[5:2] (upper bits)
  - Byte 55 [7:2]: X[5:0] (lower bits)
  - Byte 55 [1:0]: Y[9:8]
  - Byte 56 [7:0]: Y[7:0]

#### Temperature (Byte 57)
```c
uint8_t temperature = buffer[57];
```

- **Type**: Unsigned 8-bit integer
- **Unit**: Unknown (possibly °C or raw ADC value)
- **Range**: 0-255
- **Usage**: Device thermal monitoring

#### Button States (Byte 58)

**Bit Layout**:
```
Bit 0: Trigger Button
Bit 1: Home Button
Bit 2: Back Button
Bit 3: Touchpad Button (touchpad click)
Bit 4: Volume Up Button
Bit 5: Volume Down Button
Bits 6-7: Unused
```

**Parsing**:
```c
bool trigger    = (buffer[58] & (1 << 0)) != 0;
bool home       = (buffer[58] & (1 << 1)) != 0;
bool back       = (buffer[58] & (1 << 2)) != 0;
bool touchpad   = (buffer[58] & (1 << 3)) != 0;
bool volumeUp   = (buffer[58] & (1 << 4)) != 0;
bool volumeDown = (buffer[58] & (1 << 5)) != 0;
```

---

## Sensor Specifications

### Inertial Measurement Unit (IMU)

#### Accelerometer
- **Type**: 3-axis linear acceleration
- **Sample Rate**: ~43.65 Hz (3 samples per ~68.85ms notification)
- **Resolution**: 16-bit
- **Range**: ±2g (estimated)
- **Sensitivity**: ~2048 LSB/g

#### Gyroscope
- **Type**: 3-axis angular velocity
- **Sample Rate**: ~43.65 Hz (3 samples per notification)
- **Resolution**: 16-bit
- **Range**: ±2000°/s (estimated)
- **Sensitivity**: ~14.285 LSB/(°/s)

#### Magnetometer
- **Type**: 3-axis magnetic field
- **Sample Rate**: ~14.55 Hz (1 sample per notification)
- **Resolution**: 16-bit
- **Sensitivity**: ~16.67 LSB/μT (1/0.06)

### Touchpad
- **Type**: Capacitive 2D touchpad
- **Resolution**: 10-bit (1024 levels per axis)
- **Physical Size**: ~315mm × 315mm (estimated)
- **Sample Rate**: ~14.55 Hz

### Buttons
- **Count**: 6 digital buttons
- **Types**: 
  - Trigger (analog-style trigger button)
  - Home (main menu button)
  - Back (return/back button)
  - Touchpad (touchpad click/press)
  - Volume Up
  - Volume Down
- **Sample Rate**: ~14.55 Hz

---

## Timing & Performance

### Data Rate
- **Notification Frequency**: ~14.55 Hz (one 60-byte packet every ~68.85ms)
- **Effective IMU Sample Rate**: ~43.65 Hz (3 samples per notification)
- **Magnetometer Sample Rate**: ~14.55 Hz (1 sample per notification)

### Latency
- **BLE Connection Interval**: Typical 7.5-30ms
- **Notification Latency**: ~10-50ms (depends on BLE parameters)
- **Total Motion-to-Photon**: ~20-100ms (estimated, needs measurement)

### Recommended AHRS Update
- **Sample Interval**: 68.84681583453657 ms (from existing implementation)
- **Algorithm**: Madgwick or Mahony filter
- **Beta Parameter**: 0.352 (Madgwick)

---

## Coordinate Systems

### Accelerometer
- **X-axis**: Right (when controller held upright)
- **Y-axis**: Up
- **Z-axis**: Forward (toward user)
- **Note**: Needs verification with physical testing

### Gyroscope
- **X-axis**: Roll (rotation around forward axis)
- **Y-axis**: Pitch (rotation around right axis)
- **Z-axis**: Yaw (rotation around up axis)
- **Note**: Needs verification with physical testing

### Magnetometer
- **Axes**: Same as accelerometer
- **Reference**: Earth's magnetic field
- **Note**: Calibration required for accurate heading

### Touchpad
- **Origin**: Top-left corner (0, 0)
- **X-axis**: Increases to the right
- **Y-axis**: Increases downward
- **Center**: (512, 512) approximately

---

## Error Handling

### Common Errors
1. **Connection Lost**: Controller powers off or goes out of range
2. **Invalid Data**: Malformed notification packets
3. **Service Not Found**: GATT service discovery failure
4. **Characteristic Not Found**: Missing required characteristic

### Recovery Strategies
1. **Reconnection**: Automatic retry with exponential backoff
2. **Data Validation**: Check packet size and timestamp continuity
3. **Sensor Fusion Reset**: Re-initialize orientation on connection loss
4. **Keep-Alive**: Periodic command to prevent timeout

---

## Power Management

### Power Modes
1. **Off**: Controller powered down (CMD_OFF)
2. **Low Power Mode**: Reduced sample rate (CMD_LPM_ENABLE)
3. **Normal Mode**: Full sensor streaming (CMD_LPM_DISABLE)
4. **VR Mode**: Optimized for VR applications (CMD_VR_MODE)

### Battery Considerations
- Controller uses standard battery (CR2450 or similar)
- Continuous streaming drains battery faster
- Consider implementing automatic sleep on inactivity

---

## Implementation Notes

### Calibration
- **Home Button**: Can be used to re-zero orientation (existing implementation)
- **CMD_CALIBRATE**: May trigger internal sensor calibration
- **Magnetometer**: Requires user to perform figure-8 motion for calibration

### Orientation Tracking
- Use quaternion-based sensor fusion (AHRS)
- Madgwick algorithm recommended (beta=0.352)
- Sample interval: 68.85ms (based on 3 samples per notification)
- Store zero-point quaternion for re-centering

### Data Processing Pipeline
1. Receive 60-byte notification
2. Parse all sensor data
3. Convert to physical units
4. Update sensor fusion (3 times per notification)
5. Generate quaternion/orientation
6. Apply zero-point correction
7. Generate HID reports or callback to application

---

## Known Issues & Limitations

1. **Multiple Samples**: Current implementation only uses first of 3 IMU samples
2. **Timestamp Rollover**: 24-bit timestamp will overflow after ~16.7 seconds
3. **Temperature Units**: Temperature sensor unit/calibration unknown
4. **Magnetometer Calibration**: No automatic calibration implemented
5. **Button Combinations**: No known special button combinations

---

## Testing Procedures

### Connection Test
1. Power on controller
2. Initiate BLE scan
3. Verify device appears
4. Connect to device
5. Verify GATT services

### Data Integrity Test
1. Subscribe to notifications
2. Verify 60-byte packets
3. Check timestamp increments monotonically
4. Verify button states match physical presses
5. Verify touchpad coordinates in valid range

### Sensor Test
1. Place controller on flat surface
2. Verify accelerometer reads ~1g on Z-axis
3. Rotate controller slowly
4. Verify gyroscope reads rotation
5. Verify magnetometer responds to orientation

### Orientation Test
1. Start sensor fusion
2. Place controller in known orientation
3. Press home button to zero
4. Rotate controller
5. Verify orientation follows movement
6. Check for drift over time

---

## References

### Source Code
- `ControllerBluetoothInterface.js`: Protocol implementation
- `ControllerDisplay.js`: Data processing and visualization

### External Resources
- Reverse Engineering Blog: http://jsyang.ca/hacks/gear-vr-rev-eng/
- Samsung GearVR Framework: https://github.com/Samsung/GearVRf
- AHRS Library: https://github.com/psiphi75/ahrs

### Related Standards
- Bluetooth Low Energy Specification: https://www.bluetooth.com/specifications/specs/
- HID Usage Tables: https://usb.org/sites/default/files/hut1_21.pdf
- Sensor Fusion Algorithms: https://x-io.co.uk/open-source-imu-and-ahrs-algorithms/

---

## Appendix: Data Structure Definitions

### C-Style Structures

```c
// Controller notification packet (60 bytes)
typedef struct {
    uint8_t timestamp[3];           // Bytes 0-2: timestamp (24-bit)
    uint8_t reserved1;              // Byte 3: unused
    
    // IMU Sample 0
    int16_t accel0[3];              // Bytes 4-9: accelerometer X, Y, Z
    int16_t gyro0[3];               // Bytes 10-15: gyroscope X, Y, Z
    
    // IMU Sample 1
    int16_t accel1[3];              // Bytes 16-21
    int16_t gyro1[3];               // Bytes 22-27
    
    uint8_t reserved2[4];           // Bytes 28-31: unused
    
    int16_t mag[3];                 // Bytes 32-37: magnetometer X, Y, Z
    
    uint8_t reserved3[2];           // Bytes 38-39: unused
    
    // IMU Sample 2
    int16_t accel2[3];              // Bytes 40-45
    int16_t gyro2[3];               // Bytes 46-51
    
    uint8_t reserved4[2];           // Bytes 52-53: unused
    
    uint8_t touchpad[3];            // Bytes 54-56: packed X,Y coordinates
    uint8_t temperature;            // Byte 57: temperature sensor
    uint8_t buttons;                // Byte 58: button bitfield
    uint8_t reserved5;              // Byte 59: unused
} __attribute__((packed)) GearVRControllerPacket;

// Parsed controller data
typedef struct {
    double timestamp;               // seconds
    
    float accel[3];                 // g-force (X, Y, Z)
    float gyro[3];                  // rad/s (X, Y, Z)
    float mag[3];                   // μT (X, Y, Z)
    
    uint16_t touchpad_x;            // 0-1023
    uint16_t touchpad_y;            // 0-1023
    
    uint8_t temperature;            // raw value
    
    bool trigger;
    bool home;
    bool back;
    bool touchpad_click;
    bool volume_up;
    bool volume_down;
} GearVRControllerData;
```

---

*Document Version: 1.0*
*Last Updated: 2025-11-03*
*Based on Web Bluetooth implementation analysis*
