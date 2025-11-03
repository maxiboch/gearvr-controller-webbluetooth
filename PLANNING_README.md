# GearVR Controller - macOS Native HID Driver Planning Documents

This directory contains comprehensive planning and technical documentation for porting the GearVR Controller Web Bluetooth implementation to a native macOS HID driver.

## Document Overview

### 📋 [MACOS_HID_PORT_TASK_LIST.md](MACOS_HID_PORT_TASK_LIST.md)
**Complete task breakdown and project roadmap**

This document provides a comprehensive, phase-by-phase task list for implementing the native macOS driver. It covers:
- 11 development phases from research to distribution
- Detailed subtasks for each phase
- Technical challenges and considerations
- Key decisions needed
- Estimated effort (14-21 weeks)
- Success criteria

**Use this document to**: Plan the project, track progress, and understand the full scope of work required.

---

### 🔧 [PROTOCOL_SPECIFICATION.md](PROTOCOL_SPECIFICATION.md)
**Complete Bluetooth protocol documentation**

This document provides detailed technical specifications for the GearVR Controller's Bluetooth Low Energy protocol, including:
- BLE GATT service and characteristic UUIDs
- Complete command protocol (9 commands documented)
- 60-byte data packet structure with byte-level detail
- Sensor specifications (accelerometer, gyroscope, magnetometer)
- Touchpad and button encoding
- Timing, performance, and coordinate systems
- C-style data structure definitions

**Use this document to**: Understand the low-level protocol, implement data parsing, and troubleshoot communication issues.

---

### 🏗️ [MACOS_ARCHITECTURE.md](MACOS_ARCHITECTURE.md)
**Architecture recommendations and design decisions**

This document provides a complete architectural blueprint for the native macOS implementation, including:
- Analysis of 3 implementation approaches (KEXT, DriverKit, User-space)
- **Recommended**: Hybrid DriverKit + User Framework architecture
- Component specifications (Driver, Framework, Demo App)
- Complete data flow diagrams
- HID integration strategy
- 6-phase implementation plan
- Technical challenges with solutions
- Performance targets and testing strategy

**Use this document to**: Make architectural decisions, design components, and understand the overall system structure.

---

## Quick Start

### For Project Managers
1. Read **MACOS_HID_PORT_TASK_LIST.md** for the full task breakdown
2. Review the estimated effort (3.5-5 months for one developer)
3. Use the phase structure to plan sprints/milestones

### For Developers
1. Start with **PROTOCOL_SPECIFICATION.md** to understand the hardware
2. Review **MACOS_ARCHITECTURE.md** for the recommended approach
3. Use **MACOS_HID_PORT_TASK_LIST.md** as your development checklist

### For Technical Leads
1. Review **MACOS_ARCHITECTURE.md** for technology choices
2. Validate the recommended DriverKit + Framework approach
3. Review technical challenges and solutions
4. Assess resource requirements

---

## Key Decisions Made

### ✅ Technology Stack
- **Driver**: DriverKit (not deprecated KEXT)
- **Language**: C++ for driver, Swift/Objective-C for framework
- **Minimum OS**: macOS 10.15 (Catalina) or later
- **Sensor Fusion**: User-space (in framework, not driver)

### ✅ Architecture
- **Hybrid approach**: DriverKit driver + user-space framework
- **HID Strategy**: Standard game controller profile + vendor-specific reports
- **API Design**: CoreMotion-style Swift API

### ✅ Implementation Priority
1. Phase 1: Basic HID (buttons + touchpad)
2. Phase 2: Full sensor support (IMU data)
3. Phase 3: User framework (AHRS, orientation)
4. Phase 4: Demo application
5. Phase 5: Game Controller integration
6. Phase 6: Distribution

---

## Technical Highlights

### Protocol Features Documented
- ✓ Complete 60-byte packet structure
- ✓ 9 control commands
- ✓ 6 buttons + 2D touchpad
- ✓ 3-axis accelerometer (±2g, ~44Hz)
- ✓ 3-axis gyroscope (±2000°/s, ~44Hz)
- ✓ 3-axis magnetometer (~15Hz)
- ✓ Temperature sensor
- ✓ Timestamp (microseconds)

### Architecture Components
1. **DriverKit HID Driver** - Low-level BLE communication and HID reporting
2. **User Framework** - High-level API with sensor fusion (AHRS)
3. **Demo Application** - SwiftUI app for testing and visualization
4. **Game Controller Integration** - Works with existing games

### Performance Targets
- Motion-to-Photon Latency: <20ms (target), <50ms (acceptable)
- CPU Usage: <2% (target), <5% (acceptable)
- Connection Reliability: >99%
- Data Rate: 43.65 Hz (3 IMU samples per notification)

---

## Development Phases

| Phase | Duration | Key Deliverables |
|-------|----------|------------------|
| 1: Minimal Viable Driver | 2-3 weeks | Basic HID device, buttons work |
| 2: Full Sensor Support | 2-3 weeks | All sensor data accessible |
| 3: User Framework | 2-3 weeks | AHRS, orientation tracking |
| 4: Demo Application | 1-2 weeks | Visualization and testing tool |
| 5: Game Controller Integration | 1-2 weeks | Compatible with existing games |
| 6: Distribution | 1-2 weeks | Signed, notarized installer |

**Total**: 14-21 weeks (3.5-5 months)

---

## Prerequisites

### Development Requirements
- Mac running macOS 10.15+ (M1/M2 recommended)
- Xcode 13+ with DriverKit SDK
- Apple Developer Program membership ($99/year)
- GearVR Controller hardware for testing

### Skills Required
- Proficiency in C++ (for DriverKit driver)
- Proficiency in Swift/Objective-C (for framework)
- Understanding of Bluetooth Low Energy
- Understanding of HID protocol
- Experience with sensor fusion (AHRS algorithms)

### Tools Needed
- Xcode
- Bluetooth development tools (PacketLogger)
- 3D modeling tools (for demo app assets)
- Code signing certificates

---

## Resource Links

### Current Implementation
- **Repository**: maxiboch/gearvr-controller-webbluetooth
- **Web Demo**: https://jsyang.ca/gearvr-controller-webbluetooth/
- **Reverse Engineering Blog**: http://jsyang.ca/hacks/gear-vr-rev-eng/

### Apple Documentation
- [DriverKit](https://developer.apple.com/documentation/driverkit)
- [IOUserHIDDevice](https://developer.apple.com/documentation/hiddevicedriverkit/iouserhiddevice)
- [IOBluetooth](https://developer.apple.com/documentation/iobluetooth)
- [Game Controller Framework](https://developer.apple.com/documentation/gamecontroller)

### Standards & Specifications
- [Bluetooth Low Energy Spec](https://www.bluetooth.com/specifications/specs/)
- [HID Usage Tables](https://usb.org/sites/default/files/hut1_21.pdf)
- [USB HID Specification](https://www.usb.org/hid)

### Related Projects
- [Samsung GearVR Framework](https://github.com/Samsung/GearVRf)
- [OpenHMD](https://github.com/OpenHMD/OpenHMD)
- [AHRS Library](https://github.com/psiphi75/ahrs)

---

## Next Steps

### Immediate Actions
1. ✅ Review and validate the architecture (this document)
2. ✅ Review and validate the task list
3. ⬜ Set up development environment
4. ⬜ Join Apple Developer Program (if not already member)
5. ⬜ Acquire GearVR controller for testing
6. ⬜ Create DriverKit project skeleton
7. ⬜ Begin Phase 1: Minimal Viable Driver

### Key Milestones
- [ ] **M1**: DriverKit project created, builds successfully
- [ ] **M2**: BLE connection established to controller
- [ ] **M3**: First HID report sent (buttons visible in System Preferences)
- [ ] **M4**: All buttons and touchpad working
- [ ] **M5**: IMU data streaming via vendor reports
- [ ] **M6**: Framework with AHRS working
- [ ] **M7**: Demo app showing 3D controller
- [ ] **M8**: Game Controller framework integration
- [ ] **M9**: Alpha release (signed and notarized)
- [ ] **M10**: Public release

---

## Support & Questions

For questions about these planning documents or the implementation:

1. **Protocol Questions**: See PROTOCOL_SPECIFICATION.md or the original Web Bluetooth code
2. **Architecture Questions**: See MACOS_ARCHITECTURE.md
3. **Project Planning**: See MACOS_HID_PORT_TASK_LIST.md
4. **Original Implementation**: Check ControllerBluetoothInterface.js and ControllerDisplay.js

---

## Document Versions

| Document | Version | Last Updated |
|----------|---------|--------------|
| MACOS_HID_PORT_TASK_LIST.md | 1.0 | 2025-11-03 |
| PROTOCOL_SPECIFICATION.md | 1.0 | 2025-11-03 |
| MACOS_ARCHITECTURE.md | 1.0 | 2025-11-03 |
| PLANNING_README.md | 1.0 | 2025-11-03 |

---

## License

These planning documents are provided as part of the gearvr-controller-webbluetooth project. See the main repository LICENSE file for details.

---

**Ready to start? Begin with Phase 1 in MACOS_HID_PORT_TASK_LIST.md!** 🚀
