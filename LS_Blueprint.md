# LocationSimulator Blueprint

Based on the analysis of `LocationSimulator_UIReference`, here is the blueprint for the `LocationSimulator` macOS application.

## 1. Project Overview

**Goal**: Create a macOS application to spoof the location of iOS devices and Simulators.
**Target Audience**: Developers testing location-based features.
**Tech Stack**:

- **Language**: Swift
- **UI Framework**: AppKit (macOS Native) with Storyboards.
- **Architecture**: MVC (Model-View-Controller).

## 2. Directory Structure

The project should follow a clean separation of concerns:

```text
LocationSimulator/
├── Storyboard/
│   └── Main.storyboard       # Main UI Layout (Window, SplitView, Sidebar, Map)
├── ViewController/
│   ├── SplitViewController/  # Manages Sidebar and Content flow
│   ├── SidebarViewController/# Displays list of connected devices
│   ├── MapViewController/    # Main map interface and interaction logic
│   ├── Navigation/           # Navigation logic (Auto-move, Waypoints)
│   └── Onboarding/           # First-run setup or permissions
├── Views/
│   ├── MapView/              # Custom Map Views (Annotations, Overlays)
│   ├── HUD/                  # Floating controls (Walk button, Speed, Joystick)
│   └── Cells/                # Sidebar TableView cells
├── Model/
│   ├── Device.swift          # Data model for iOS Device
│   └── Location.swift        # Data model for Coordinates/GPX
├── Services/
│   ├── DeviceManager.swift   # Handles device discovery (libimobiledevice wrapper)
│   └── SimulationService.swift # Handles location spoofing logic
└── Assets.xcassets/          # App Icons, HUD Images
```

## 3. UI Architecture

### Main Window Layout

The application uses a **Split View Controller** layout:

- **Sidebar (Left)**: List of available devices.
- **Detail (Right)**: Map interface for the selected device.

### 3.1 Sidebar (`SidebarViewController`)

- **Type**: `NSOutlineView` or `NSTableView`.
- **Content**:
  - **Header**: "Devices".
  - **Rows**: Connected devices (Icon + Name + Status).
  - **Status Indicators**:
    - Green Dot: Connected/Spoofing.
    - Grey Dot: Idle.
    - Wifi Icon: Network connected.
- **Footer**: Add/Refresh buttons (optional).

### 3.2 Map Interface (`MapViewController`)

- **Base**: `MKMapView`.
- **Interactions**:
  - **Long Click**: Set spoof location (Drop pin).
  - **Drag Pin**: Adjust location.
- **Overlays**:
  - Current location (Blue Dot).
  - Simulated route (Polyline).

### 3.3 HUD Controls (Floating)

Floating views overlaid on the MapView:

- **Movement Control (Bottom-Left)**:
  - **Walk Button**: Toggle auto-walk.
  - **Speed Selector**: Walk, Cycle, Drive icons.
  - **Joystick/Pad**: Directional control.
- **Status/Action (Bottom-Right or Top-Right)**:
  - **Reset**: Stop spoofing and return to real location.
  - **Search**: Search bar for addresses (uses `MKLocalSearch` or equivalent).

### 3.4 Dialogs & Modals

- **No Device**: `NoDeviceViewController` shown in Detail pane when no device is selected.
- **Onboarding**: Instructions for "Developer Mode" (iOS 16+) and "Developer Disk Image" mounting.
- **Teleport/Navigate**: Popup when clicking a POI to "Teleport Here" or "Walk Here".

## 4. Key Features

1.  **Device Discovery**:

    - Auto-detect USB devices.
    - Network device support (via Bonjour/mDNS).
    - Filter Simulator vs Physical.

2.  **Location Spoofing**:

    - **Instant**: `LocationSimulation.set(lat, lng)`.
    - **Navigation**: Calculate routes and interpolate points over time (`LocationSimulation.set` in a loop).
    - **GPX Support**: Parse GPX files to replay routes.

3.  **Preferences**:
    - **Network**: Toggle "Allow Network Devices".
    - **Developer Disk**: Manage/Download Disk Images manually.

## 5. Assets Required

- **App Icon**: Distinctive map/location pin style.
- **HUD Icons**:
  - `walk.png`, `cycle.png`, `drive.png`.
  - `reset.png`, `play.png`, `pause.png`.
  - `pin.png` (Custom annotation).

## 6. Implementation Notes

- **AppKit vs SwiftUI**: The components suggest AppKit (Storyboards). If building new, SwiftUI could be used, but for strict "Reference" alignment, stick to AppKit constraints (Delegates, DataSources).
- **CoreLocation**: Used for map handling.
- **External Dependencies**:
  - `pymobiledevice3` (Python side for us) OR in Swift `libimobiledevice` wrapper. _Note: Original project likely wraps C-libraries directly._
