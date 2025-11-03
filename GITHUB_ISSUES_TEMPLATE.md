# GitHub Issues Template for macOS HID Driver Port

This document contains pre-formatted GitHub issue templates for the macOS HID driver port project. Copy and paste each section below into a new GitHub issue.

---

## Phase 1: Research & Analysis

### Issue 1.1: Protocol Analysis and Documentation

**Title:** `[Phase 1.1] Complete Bluetooth LE Protocol Analysis and Documentation`

**Labels:** `research`, `documentation`, `phase-1`

**Description:**
```markdown
## Objective
Document the complete Bluetooth LE GATT protocol for the GearVR Controller to ensure accurate implementation in the native driver.

## Tasks
- [ ] Document complete Bluetooth LE GATT service structure
  - [ ] Custom service UUID: `4f63756c-7573-2054-6872-65656d6f7465`
  - [ ] Write characteristic UUID: `c8c51726-81bc-483b-a052-f7a14ea3d282`
  - [ ] Notify characteristic UUID: `c8c51726-81bc-483b-a052-f7a14ea3d281`
- [ ] Document data packet format (60-byte notifications)
  - [ ] Timestamp encoding (bytes 0-2)
  - [ ] Accelerometer data (3-axis, multiple samples)
  - [ ] Gyroscope data (3-axis, multiple samples)
  - [ ] Magnetometer data (3-axis)
  - [ ] Touchpad coordinates (bytes 54-56)
  - [ ] Temperature sensor (byte 57)
  - [ ] Button states (byte 58)
- [ ] Document command protocol
  - [ ] Power/sensor control commands
  - [ ] Calibration commands
  - [ ] Keep-alive mechanism
- [ ] Reverse engineer any undocumented protocol features

## Resources
- Existing implementation: `ControllerBluetoothInterface.js`
- Protocol documentation: `PROTOCOL_SPECIFICATION.md`
- Reverse engineering blog: http://jsyang.ca/hacks/gear-vr-rev-eng/

## Acceptance Criteria
- Complete protocol documentation verified against hardware
- All data packet fields identified and documented
- All commands tested and documented

## Estimated Effort
1-2 weeks
```

---

### Issue 1.2: macOS HID Architecture Research

**Title:** `[Phase 1.2] Research macOS HID Architecture and Frameworks`

**Labels:** `research`, `architecture`, `phase-1`

**Description:**
```markdown
## Objective
Research macOS HID frameworks and determine the best approach for implementing the driver.

## Tasks
- [ ] Study macOS IOKit HID framework
- [ ] Research DriverKit (modern approach, macOS 10.15+)
- [ ] Research IOUSBHostHIDDevice for Bluetooth HID
- [ ] Understand HID report descriptor requirements
- [ ] Study macOS Bluetooth stack integration
- [ ] Review Apple's HID specifications and guidelines

## Resources
- [IOKit Fundamentals](https://developer.apple.com/library/archive/documentation/DeviceDrivers/Conceptual/IOKitFundamentals/)
- [DriverKit Documentation](https://developer.apple.com/documentation/driverkit)
- [HID Usage Tables](https://usb.org/sites/default/files/hut1_21.pdf)

## Acceptance Criteria
- Understanding of IOKit/DriverKit architecture
- Knowledge of HID report descriptor design
- Clear comparison of implementation approaches

## Estimated Effort
1 week
```

---

### Issue 1.3: Technology Stack Decision

**Title:** `[Phase 1.3] Make Technology Stack Decisions`

**Labels:** `decision`, `architecture`, `phase-1`

**Description:**
```markdown
## Objective
Make key technology decisions for the driver implementation.

## Tasks
- [ ] Decide between:
  - [ ] Kernel Extension (KEXT) - deprecated but more powerful
  - [ ] DriverKit - modern, sandboxed approach
  - [ ] User-space daemon with IOBluetooth framework
- [ ] Define minimum macOS version support
- [ ] Plan code signing and notarization requirements

## Decision Criteria
- Future compatibility
- Development complexity
- Performance requirements
- Distribution ease
- Security considerations

## Recommended Decision
Based on `MACOS_ARCHITECTURE.md`:
- **Primary**: DriverKit + User Framework (hybrid approach)
- **Minimum OS**: macOS 10.15 (Catalina)
- **Requires**: Apple Developer Program membership

## Acceptance Criteria
- Technology stack documented
- Architecture diagram created
- Implementation approach approved

## Estimated Effort
3-5 days
```

---

## Phase 2: Development Environment Setup

### Issue 2.1: Development Tools Setup

**Title:** `[Phase 2.1] Set Up Development Tools and Environment`

**Labels:** `setup`, `environment`, `phase-2`

**Description:**
```markdown
## Objective
Set up all necessary development tools for macOS driver development.

## Tasks
- [ ] Install Xcode with required SDKs
- [ ] Set up DriverKit development environment
- [ ] Install Bluetooth development tools
- [ ] Set up code signing certificates (Apple Developer Program)
- [ ] Configure debugging environment (kernel debugging if needed)

## Prerequisites
- Apple Developer Program membership ($99/year)
- Mac running macOS 10.15 or later
- Admin access to development Mac

## Tools Required
- Xcode 13+ (for DriverKit support)
- Bluetooth PacketLogger
- Console.app for debugging
- Development certificates

## Acceptance Criteria
- Xcode builds DriverKit projects successfully
- Code signing certificates installed
- Bluetooth debugging tools working

## Estimated Effort
2-3 days
```

---

### Issue 2.2: Project Structure Creation

**Title:** `[Phase 2.2] Create Project Structure and Build System`

**Labels:** `setup`, `infrastructure`, `phase-2`

**Description:**
```markdown
## Objective
Create the foundational project structure for the driver and framework.

## Tasks
- [ ] Create Xcode project for driver
- [ ] Set up build system (Xcode or CMake)
- [ ] Create test application for driver validation
- [ ] Set up version control structure
- [ ] Create documentation directory

## Deliverables
- DriverKit Xcode project
- User framework Xcode project
- Test/demo application project
- Build scripts
- README and documentation templates

## Acceptance Criteria
- Projects build successfully
- Basic project structure in place
- Version control configured

## Estimated Effort
3-5 days
```

---

### Issue 2.3: Hardware and Testing Setup

**Title:** `[Phase 2.3] Set Up Hardware and Testing Environment`

**Labels:** `setup`, `hardware`, `phase-2`

**Description:**
```markdown
## Objective
Prepare hardware and testing infrastructure for driver development.

## Tasks
- [ ] Acquire GearVR controller for testing
- [ ] Set up Bluetooth packet capture tools (PacketLogger)
- [ ] Configure test Mac with development settings
- [ ] Enable kernel debugging if using KEXT approach

## Hardware Requirements
- GearVR Controller (SM-R324 or similar)
- Mac for development and testing
- Bluetooth LE compatible Mac

## Acceptance Criteria
- GearVR controller pairs with Mac
- Packet capture working
- Testing environment documented

## Estimated Effort
1-2 days
```

---

## Phase 3: Core Driver Implementation

### Issue 3.1: Bluetooth Communication Layer

**Title:** `[Phase 3.1] Implement Bluetooth Communication Layer`

**Labels:** `implementation`, `bluetooth`, `phase-3`, `core`

**Description:**
```markdown
## Objective
Implement the low-level Bluetooth LE communication with the GearVR controller.

## Tasks
- [ ] Implement Bluetooth device discovery
  - [ ] Filter for GearVR controller by name/UUID
  - [ ] Handle device pairing
- [ ] Implement GATT service connection
  - [ ] Connect to custom service
  - [ ] Discover characteristics
- [ ] Implement characteristic operations
  - [ ] Write commands to control characteristic
  - [ ] Subscribe to notification characteristic
  - [ ] Handle notifications/data packets

## Technical Details
- Service UUID: `4f63756c-7573-2054-6872-65656d6f7465`
- Write UUID: `c8c51726-81bc-483b-a052-f7a14ea3d282`
- Notify UUID: `c8c51726-81bc-483b-a052-f7a14ea3d281`

## Acceptance Criteria
- Driver discovers and connects to controller
- Can send commands to controller
- Receives notification data packets
- Connection stable and reliable

## Estimated Effort
1-2 weeks
```

---

### Issue 3.2: Data Parsing Layer

**Title:** `[Phase 3.2] Implement Data Parsing Layer`

**Labels:** `implementation`, `parsing`, `phase-3`, `core`

**Description:**
```markdown
## Objective
Parse raw notification data into structured sensor and button data.

## Tasks
- [ ] Port sensor data parsing functions
  - [ ] Accelerometer parsing (with scaling factors)
  - [ ] Gyroscope parsing (with scaling factors)
  - [ ] Magnetometer parsing (with scaling factors)
  - [ ] Timestamp extraction and conversion
  - [ ] Temperature reading
- [ ] Port button state parsing
  - [ ] Trigger button
  - [ ] Home button
  - [ ] Back button
  - [ ] Touchpad button
  - [ ] Volume up/down buttons
- [ ] Port touchpad coordinate parsing
  - [ ] X-axis (10-bit value)
  - [ ] Y-axis (10-bit value)
  - [ ] Coordinate normalization

## Reference
See `PROTOCOL_SPECIFICATION.md` for byte-level details and conversion factors.

## Acceptance Criteria
- All sensor data parsed correctly
- All button states detected
- Touchpad coordinates accurate
- Unit tests passing

## Estimated Effort
1 week
```

---

### Issue 3.3: HID Report Descriptor Design

**Title:** `[Phase 3.3] Design and Implement HID Report Descriptor`

**Labels:** `implementation`, `hid`, `phase-3`, `core`

**Description:**
```markdown
## Objective
Design HID report descriptor and implement report generation.

## Tasks
- [ ] Design HID report descriptor
  - [ ] Define button usage pages
  - [ ] Define axis usage pages (touchpad)
  - [ ] Define sensor data reporting (if using HID sensor spec)
  - [ ] Define vendor-specific fields for IMU data
- [ ] Implement HID report generation
  - [ ] Map parsed data to HID reports
  - [ ] Handle multiple report types if needed
  - [ ] Ensure proper report timing

## HID Strategy
- Standard game controller profile for buttons/touchpad
- Vendor-specific reports for IMU data
- Target: Work with Game Controller framework

## Acceptance Criteria
- HID descriptor validates correctly
- Reports generated at proper rate (~44 Hz for IMU)
- Device appears as game controller in System Preferences

## Estimated Effort
1-2 weeks
```

---

### Issue 3.4: Command Interface Implementation

**Title:** `[Phase 3.4] Implement Device Command Interface`

**Labels:** `implementation`, `commands`, `phase-3`

**Description:**
```markdown
## Objective
Implement command interface for controlling the GearVR controller.

## Tasks
- [ ] Implement device control commands
  - [ ] Power on/off (CMD_OFF: 0x0000)
  - [ ] Sensor enable (CMD_SENSOR: 0x0100)
  - [ ] Calibration (CMD_CALIBRATE: 0x0300)
  - [ ] Keep-alive (CMD_KEEP_ALIVE: 0x0400)
  - [ ] VR mode (CMD_VR_MODE: 0x0800)
  - [ ] Low power mode control (CMD_LPM_ENABLE/DISABLE)
- [ ] Implement initialization sequence
  - [ ] Device wake-up
  - [ ] Sensor activation
  - [ ] Calibration on startup

## Command Protocol
Commands are 2-byte little-endian hex values sent via write characteristic.

## Acceptance Criteria
- All commands functional
- Initialization sequence reliable
- Keep-alive prevents disconnection

## Estimated Effort
3-5 days
```

---

## Phase 4: HID Driver Integration

### Issue 4.1: IOKit/DriverKit Integration

**Title:** `[Phase 4.1] Implement IOKit/DriverKit Integration`

**Labels:** `implementation`, `driver`, `phase-4`

**Description:**
```markdown
## Objective
Integrate with IOKit/DriverKit to register as a HID device.

## Tasks
- [ ] Implement driver entry points
  - [ ] Start/stop methods
  - [ ] Device matching
  - [ ] Resource allocation/deallocation
- [ ] Implement HID interface
  - [ ] Register as HID device
  - [ ] Implement HID report callbacks
  - [ ] Handle HID requests from system

## Technical Requirements
- Use IOUserHIDDevice class (DriverKit)
- Implement proper device matching dictionary
- Handle driver lifecycle correctly

## Acceptance Criteria
- Driver loads successfully
- Device appears in System Preferences
- HID reports delivered to system

## Estimated Effort
1-2 weeks
```

---

### Issue 4.2: Power Management Implementation

**Title:** `[Phase 4.2] Implement Power Management`

**Labels:** `implementation`, `power`, `phase-4`

**Description:**
```markdown
## Objective
Implement robust power management for sleep/wake and connection handling.

## Tasks
- [ ] Implement sleep/wake handling
- [ ] Implement low-power mode transitions
- [ ] Handle device disconnection/reconnection
- [ ] Implement keep-alive mechanism

## Acceptance Criteria
- Driver handles Mac sleep/wake correctly
- Controller reconnects after disconnect
- No memory leaks or resource issues

## Estimated Effort
3-5 days
```

---

### Issue 4.3: Error Handling and Diagnostics

**Title:** `[Phase 4.3] Implement Error Handling and Logging`

**Labels:** `implementation`, `error-handling`, `phase-4`

**Description:**
```markdown
## Objective
Implement comprehensive error handling and diagnostic logging.

## Tasks
- [ ] Implement connection error recovery
- [ ] Handle malformed data packets
- [ ] Implement timeout handling
- [ ] Add logging and diagnostics
- [ ] Handle Bluetooth pairing failures

## Logging Strategy
- Use os_log for system logging
- Implement different log levels
- Add diagnostic counters

## Acceptance Criteria
- Driver handles all error conditions gracefully
- Logs provide useful debugging information
- No crashes or kernel panics

## Estimated Effort
1 week
```

---

## Phase 5: Sensor Fusion & Orientation

### Issue 5.1: Sensor Fusion Algorithm Implementation

**Title:** `[Phase 5.1] Implement AHRS Sensor Fusion Algorithm`

**Labels:** `implementation`, `sensor-fusion`, `phase-5`

**Description:**
```markdown
## Objective
Implement sensor fusion algorithm for accurate orientation tracking.

## Tasks
- [ ] Decide on sensor fusion approach:
  - [ ] Port existing AHRS library (Madgwick/Mahony)
  - [ ] Use Apple's CoreMotion algorithms
  - [ ] Implement custom fusion in driver vs user-space
- [ ] Implement quaternion calculation
- [ ] Handle orientation zeroing (home button)
- [ ] Implement drift compensation

## Recommended Approach
- Madgwick algorithm (beta=0.352)
- Sample interval: 68.85ms
- Implement in user-space framework (not driver)

## Acceptance Criteria
- Orientation tracking accurate and smooth
- Minimal drift over time
- Home button re-zeroing works

## Estimated Effort
1-2 weeks
```

---

### Issue 5.2: HID Sensor Data Exposure

**Title:** `[Phase 5.2] Expose Sensor Data via HID and Framework`

**Labels:** `implementation`, `api`, `phase-5`

**Description:**
```markdown
## Objective
Expose orientation and sensor data to applications via HID and custom framework.

## Tasks
- [ ] Research HID sensor usage pages
- [ ] Expose orientation as HID sensor data
- [ ] Expose raw IMU data for applications
- [ ] Consider Game Controller framework integration

## API Design
Provide both:
- Standard HID reports (for OS compatibility)
- Framework API (for advanced features)

## Acceptance Criteria
- Applications can read orientation
- Raw IMU data accessible
- Documentation complete

## Estimated Effort
1 week
```

---

## Phase 6: User-Space Application/Framework

### Issue 6.1: Swift/Objective-C Framework Development

**Title:** `[Phase 6.1] Develop User-Space Framework`

**Labels:** `implementation`, `framework`, `phase-6`

**Description:**
```markdown
## Objective
Create high-level Swift/Objective-C framework for easy integration.

## Tasks
- [ ] Create Objective-C/Swift framework
  - [ ] Device enumeration API
  - [ ] Connection management
  - [ ] Data callback interface
  - [ ] Sensor data structures
- [ ] Create C API for cross-platform compatibility
- [ ] Add documentation and examples

## API Design
```swift
class GearVRController {
    var orientation: simd_quatf { get }
    var buttons: ButtonState { get }
    var touchpad: CGPoint { get }
    
    var orientationHandler: ((simd_quatf) -> Void)?
    var buttonHandler: ((ButtonState) -> Void)?
    
    func connect()
    func disconnect()
    func calibrate()
    func resetOrientation()
}
```

## Acceptance Criteria
- Framework builds and links correctly
- API documented
- Example code provided

## Estimated Effort
2 weeks
```

---

### Issue 6.2: Demo Application Development

**Title:** `[Phase 6.2] Create Demo and Visualization Application`

**Labels:** `implementation`, `demo`, `phase-6`

**Description:**
```markdown
## Objective
Create a demo application to visualize controller data and test functionality.

## Tasks
- [ ] Port visualization demo to native app
  - [ ] 3D controller model rendering
  - [ ] Real-time sensor data display
  - [ ] Button state visualization
  - [ ] Touchpad tracking visualization
- [ ] Create configuration utility
  - [ ] Calibration UI
  - [ ] Sensitivity adjustments
  - [ ] Button mapping (if applicable)

## Technology
- SwiftUI for UI
- SceneKit for 3D rendering
- Combine for reactive updates

## Acceptance Criteria
- App shows real-time controller state
- 3D model follows orientation
- Calibration works
- User-friendly interface

## Estimated Effort
1-2 weeks
```

---

### Issue 6.3: Game Controller Framework Integration

**Title:** `[Phase 6.3] Integrate with Game Controller Framework`

**Labels:** `implementation`, `game-controller`, `phase-6`

**Description:**
```markdown
## Objective
Make the controller work with macOS Game Controller framework.

## Tasks
- [ ] Investigate GCController integration
- [ ] Map buttons to standard game controller
- [ ] Map touchpad to directional controls
- [ ] Expose motion data through framework

## Benefits
- Works with existing games without modification
- Standard API for developers
- Better OS integration

## Acceptance Criteria
- Controller appears in GCController.controllers()
- Button mapping works correctly
- Compatible with games using Game Controller framework

## Estimated Effort
1 week
```

---

## Phase 7: Testing & Validation

### Issue 7.1: Unit Testing

**Title:** `[Phase 7.1] Implement Unit Tests`

**Labels:** `testing`, `unit-tests`, `phase-7`

**Description:**
```markdown
## Objective
Create comprehensive unit tests for all components.

## Tasks
- [ ] Test data parsing functions
  - [ ] Accelerometer parsing accuracy
  - [ ] Gyroscope parsing accuracy
  - [ ] Magnetometer parsing accuracy
  - [ ] Button state parsing
  - [ ] Touchpad coordinate parsing
- [ ] Test command generation
- [ ] Test HID report generation

## Testing Framework
- XCTest for Swift/Objective-C code
- Mock Bluetooth data for testing

## Acceptance Criteria
- >80% code coverage
- All critical paths tested
- Tests run in CI/CD

## Estimated Effort
1 week
```

---

### Issue 7.2: Integration Testing

**Title:** `[Phase 7.2] Perform Integration Testing`

**Labels:** `testing`, `integration`, `phase-7`

**Description:**
```markdown
## Objective
Test the complete system with real hardware.

## Tasks
- [ ] Test Bluetooth connection stability
- [ ] Test device discovery and pairing
- [ ] Test data streaming performance
  - [ ] Verify sample rate (~60 Hz)
  - [ ] Measure latency
  - [ ] Check for data dropouts
- [ ] Test all button combinations
- [ ] Test touchpad accuracy
- [ ] Test sensor fusion accuracy

## Test Scenarios
- Normal operation
- Sleep/wake cycles
- Reconnection after disconnect
- Multiple controllers
- Interference scenarios

## Acceptance Criteria
- All features work with real hardware
- Performance meets targets
- Stability issues identified and fixed

## Estimated Effort
1-2 weeks
```

---

### Issue 7.3: System and Compatibility Testing

**Title:** `[Phase 7.3] System and Compatibility Testing`

**Labels:** `testing`, `compatibility`, `phase-7`

**Description:**
```markdown
## Objective
Test across different macOS versions and Mac models.

## Tasks
- [ ] Test with macOS HID system
- [ ] Test in various applications
- [ ] Test power management (sleep/wake)
- [ ] Test multiple controller support
- [ ] Test across different macOS versions
- [ ] Stress testing (long duration, reconnections)
- [ ] Test on different Mac models
- [ ] Test with different Bluetooth chipsets

## Test Matrix
- macOS 10.15, 11.0, 12.0, 13.0+
- Intel and Apple Silicon Macs
- Different Bluetooth hardware

## Acceptance Criteria
- Works on all supported platforms
- No platform-specific issues
- Performance consistent across platforms

## Estimated Effort
1 week
```

---

## Phase 8: Performance Optimization

### Issue 8.1: Latency and Performance Optimization

**Title:** `[Phase 8.1] Optimize Latency and Performance`

**Labels:** `optimization`, `performance`, `phase-8`

**Description:**
```markdown
## Objective
Optimize driver and framework for minimal latency and CPU usage.

## Tasks
- [ ] Minimize processing overhead
- [ ] Optimize data parsing routines
- [ ] Tune Bluetooth connection parameters
- [ ] Reduce HID report generation overhead
- [ ] Profile CPU usage
- [ ] Optimize sensor fusion calculations

## Performance Targets
- Motion-to-photon latency: <20ms (ideal), <50ms (acceptable)
- CPU usage: <2% (ideal), <5% (acceptable)
- Connection reliability: >99%

## Acceptance Criteria
- Performance targets met
- Profiling data collected
- Optimizations documented

## Estimated Effort
1 week
```

---

## Phase 9: Documentation

### Issue 9.1: Technical Documentation

**Title:** `[Phase 9.1] Write Technical Documentation`

**Labels:** `documentation`, `phase-9`

**Description:**
```markdown
## Objective
Create comprehensive technical documentation.

## Tasks
- [ ] Document driver architecture
- [ ] Document protocol specification
- [ ] Document HID report format
- [ ] Document build and installation process
- [ ] Create API reference

## Documentation Types
- Architecture diagrams
- API reference (generated from code)
- Protocol specification
- Build instructions

## Acceptance Criteria
- Documentation complete and accurate
- Examples provided
- Published online

## Estimated Effort
3-5 days
```

---

### Issue 9.2: User Documentation

**Title:** `[Phase 9.2] Write User Documentation`

**Labels:** `documentation`, `user-docs`, `phase-9`

**Description:**
```markdown
## Objective
Create user-facing documentation and guides.

## Tasks
- [ ] Write installation guide
- [ ] Write user manual
- [ ] Create troubleshooting guide
- [ ] Write FAQ

## User Documentation
- Step-by-step installation
- Getting started guide
- Troubleshooting common issues
- FAQ

## Acceptance Criteria
- Non-technical users can install
- Common issues documented
- Clear and concise

## Estimated Effort
2-3 days
```

---

### Issue 9.3: Developer Documentation

**Title:** `[Phase 9.3] Write Developer Documentation`

**Labels:** `documentation`, `developer-docs`, `phase-9`

**Description:**
```markdown
## Objective
Create documentation for developers integrating the framework.

## Tasks
- [ ] Write integration guide for apps
- [ ] Create sample code and tutorials
- [ ] Document calibration procedures
- [ ] Document extension points

## Developer Resources
- Integration tutorial
- Sample applications
- API cookbook
- Best practices

## Acceptance Criteria
- Developers can integrate easily
- Sample code works
- Common use cases covered

## Estimated Effort
3-5 days
```

---

## Phase 10: Distribution & Deployment

### Issue 10.1: Code Signing and Notarization

**Title:** `[Phase 10.1] Implement Code Signing and Notarization`

**Labels:** `distribution`, `security`, `phase-10`

**Description:**
```markdown
## Objective
Sign and notarize the driver and framework for distribution.

## Tasks
- [ ] Sign driver with Developer ID
- [ ] Notarize driver with Apple
- [ ] Handle user approval for DriverKit
- [ ] Test installation on clean system

## Requirements
- Apple Developer Program membership
- Developer ID Application certificate
- Developer ID Installer certificate

## Process
1. Sign all binaries
2. Create installer package
3. Sign installer
4. Submit for notarization
5. Staple notarization ticket

## Acceptance Criteria
- Driver signed and notarized
- Installs without warnings on clean system
- Security requirements met

## Estimated Effort
2-3 days
```

---

### Issue 10.2: Installer Package Creation

**Title:** `[Phase 10.2] Create Installation Package`

**Labels:** `distribution`, `installer`, `phase-10`

**Description:**
```markdown
## Objective
Create a user-friendly installer package.

## Tasks
- [ ] Create installation package (.pkg)
- [ ] Implement uninstaller
- [ ] Add system compatibility checks
- [ ] Create post-installation validation

## Installer Requirements
- Copy driver to system location
- Install framework
- Set up permissions
- Provide clear instructions

## Acceptance Criteria
- Installer works on clean system
- Uninstaller removes everything
- User-friendly experience

## Estimated Effort
2-3 days
```

---

### Issue 10.3: Distribution Setup

**Title:** `[Phase 10.3] Set Up Distribution Channels`

**Labels:** `distribution`, `release`, `phase-10`

**Description:**
```markdown
## Objective
Set up distribution infrastructure and release process.

## Tasks
- [ ] Set up GitHub releases
- [ ] Create website/landing page
- [ ] Prepare demo videos
- [ ] Write release notes

## Distribution Channels
- GitHub Releases (primary)
- Optional: Homebrew cask
- Website with downloads

## Acceptance Criteria
- Release process documented
- Downloads available
- Release notes complete

## Estimated Effort
2-3 days
```

---

### Issue 10.4: Support Infrastructure

**Title:** `[Phase 10.4] Set Up Support Infrastructure`

**Labels:** `infrastructure`, `support`, `phase-10`

**Description:**
```markdown
## Objective
Set up infrastructure for user support and updates.

## Tasks
- [ ] Set up issue tracking
- [ ] Create contribution guidelines
- [ ] Set up CI/CD for testing
- [ ] Plan update mechanism

## Support Channels
- GitHub Issues
- Discussion forum
- Documentation site

## Acceptance Criteria
- Issue templates created
- CI/CD running
- Update mechanism working

## Estimated Effort
2-3 days
```

---

## How to Use These Templates

1. **Create issues incrementally**: Don't create all issues at once. Start with Phase 1 and create issues as you progress.

2. **Adjust labels**: Create these labels in your repository:
   - `research`, `documentation`, `architecture`, `decision`
   - `setup`, `environment`, `infrastructure`, `hardware`
   - `implementation`, `bluetooth`, `parsing`, `hid`, `core`
   - `driver`, `framework`, `demo`, `api`
   - `sensor-fusion`, `game-controller`
   - `testing`, `unit-tests`, `integration`, `compatibility`
   - `optimization`, `performance`
   - `user-docs`, `developer-docs`
   - `distribution`, `security`, `installer`, `release`, `support`
   - `phase-1` through `phase-10`

3. **Assign milestones**: Create milestones for each phase to track progress.

4. **Link issues**: Use GitHub's task list syntax to link related issues.

5. **Update as needed**: These are templates - adjust them based on your actual needs and discoveries during implementation.

## Quick Creation Script

You can also use GitHub CLI to create issues programmatically. Here's a sample script:

```bash
#!/bin/bash

# Example: Create Phase 1.1 issue
gh issue create \
  --title "[Phase 1.1] Complete Bluetooth LE Protocol Analysis and Documentation" \
  --label "research,documentation,phase-1" \
  --body-file phase1.1.md

# Repeat for other issues...
```

---

**Total Issues to Create**: ~30 issues across 10 phases

**Recommended Creation Strategy**:
- Start: Create all Phase 1 issues (3 issues)
- Then: Create Phase 2 issues when Phase 1 is 50% complete
- Continue: Create next phase issues when current phase is 50% complete
- This prevents issue overload and allows for adjustments based on learnings
