# gearvr-controller-webbluetooth

- [Gear VR controller web bluetooth demo](https://jsyang.ca/gearvr-controller-webbluetooth/index.html)
- [Reverse engineering process](http://jsyang.ca/hacks/gear-vr-rev-eng/)

This demo only works in [browsers supporting Web Bluetooth](https://caniuse.com/#feat=web-bluetooth). As of Feb 2018, this just means Chrome. 

If you're not using one watch the demo [on YouTube](https://www.youtube.com/watch?v=QGb5cKL8kZ4). 

## macOS Native HID Driver Port

Planning documents for porting this Web Bluetooth implementation to a native macOS HID driver are now available:

- **[📋 Planning Overview](PLANNING_README.md)** - Start here for quick overview and navigation
- **[📝 Complete Task List](MACOS_HID_PORT_TASK_LIST.md)** - Phase-by-phase development tasks (14-21 weeks estimated)
- **[🔧 Protocol Specification](PROTOCOL_SPECIFICATION.md)** - Detailed Bluetooth protocol documentation  
- **[🏗️ Architecture Guide](MACOS_ARCHITECTURE.md)** - Recommended DriverKit-based architecture

**Key Features of Planned Implementation:**
- Native macOS DriverKit driver (modern, sandboxed approach)
- Full HID device support (works as game controller)
- Swift/Objective-C framework with sensor fusion (AHRS)
- Complete IMU data access (accelerometer, gyroscope, magnetometer)
- Low-latency orientation tracking for VR applications
- Demo application with 3D visualization

**Estimated Effort:** 3.5-5 months for experienced developer

See [PLANNING_README.md](PLANNING_README.md) for complete details and next steps.

## Legal
Gear VR controller model from [Gear VR Framework](https://github.com/Samsung/GearVRf).