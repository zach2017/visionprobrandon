# Vision Field Marker - Apple Vision Pro App

A spatial computing app for Apple Vision Pro that allows you to view a field tied to GPS coordinates and place markers in your field of view that return precise latitude/longitude positions.

## Features

- **GPS-Anchored AR Field**: View a virtual grid anchored to your real-world GPS location
- **Spatial Marker Placement**: Point and tap to place markers in 3D space
- **Coordinate Conversion**: Automatically converts AR world positions to GPS coordinates
- **Distance & Bearing**: See distance and direction from your origin to each marker
- **Export Options**: Export markers as JSON or GeoJSON for use in other applications
- **Compass Overlay**: Real-time heading display aligned with cardinal directions
- **Adjustable Field Size**: Configure field radius from 10m to 500m

## Requirements

- Apple Vision Pro
- visionOS 1.0+
- Xcode 15.0+
- Location Services enabled
- World Sensing permissions

## Installation

### Using Xcode

1. Open Xcode 15 or later
2. Create a new visionOS App project
3. Replace the generated files with the source files from this project:
   - `VisionFieldMarkerApp.swift` - App entry point
   - `ContentView.swift` - Main window UI
   - `ImmersiveFieldView.swift` - AR immersive space
   - `LocationManager.swift` - GPS location handling
   - `MarkerManager.swift` - Marker management & coordinate conversion
   - `Models.swift` - Data models
4. Update `Info.plist` with the required permissions
5. Build and run on Apple Vision Pro or simulator

### Project Structure

```
VisionFieldMarker/
├── VisionFieldMarkerApp.swift    # App entry point with WindowGroup & ImmersiveSpace
├── ContentView.swift             # Main UI window with controls
├── ImmersiveFieldView.swift      # AR field view with marker placement
├── LocationManager.swift         # CoreLocation integration
├── MarkerManager.swift           # Marker & coordinate management
├── Models.swift                  # Data structures
├── Info.plist                    # App configuration & permissions
└── README.md                     # This file
```

## Usage

### Getting Started

1. Launch the app on your Vision Pro
2. Grant location permissions when prompted
3. Wait for GPS lock (accuracy indicator will show)
4. Tap "Enter Field View" to open the immersive AR experience

### Placing Markers

1. In the immersive view, look at a point on the ground
2. Tap to select the position (crosshair appears)
3. The coordinate display shows the GPS lat/long
4. Tap "Place Marker" to confirm placement
5. Marker appears with coordinates visible

### Viewing Marker Data

Each marker displays:
- **Name**: Auto-generated or custom
- **Latitude**: Decimal degrees (6 decimal places)
- **Longitude**: Decimal degrees (6 decimal places)  
- **Distance**: Meters from your origin position
- **Bearing**: Direction from origin (0-360°)

### Exporting Data

1. Return to the main window
2. Tap "Export" to generate JSON/GeoJSON
3. Data includes all marker coordinates and metadata

## Technical Details

### Coordinate Conversion

The app converts between AR world positions (meters from origin) and GPS coordinates using:

```swift
// World Position → GPS Coordinate
let latDelta = latOffset / metersPerDegreeLatitude
let lonDelta = lonOffset / metersPerDegreeLongitude(at: origin.latitude)

// GPS Coordinate → World Position  
let latOffset = (coordinate.latitude - origin.latitude) * metersPerDegreeLatitude
let lonOffset = (coordinate.longitude - origin.longitude) * metersPerDegreeLongitude
```

The conversion accounts for:
- Earth's curvature (latitude-dependent longitude scaling)
- Device heading (north alignment)
- Haversine formula for accurate distances

### Accuracy Considerations

- **GPS Accuracy**: Typically ±3-10 meters outdoors
- **AR Tracking**: Sub-centimeter precision for relative positions
- **Combined**: Best results when:
  - Good GPS signal (outdoors, clear sky)
  - Stable AR tracking (textured environment)
  - Origin established at high-accuracy GPS location

### Supported Gestures

| Gesture | Action |
|---------|--------|
| Tap | Select ground position |
| Drag | Move selection |
| Pinch | Zoom field view |
| Two-finger rotate | Rotate view |

## API Reference

### Coordinate

```swift
struct Coordinate: Codable {
    let latitude: Double   // -90 to 90
    let longitude: Double  // -180 to 180
    
    var formatted: String  // "37.123456°N 122.123456°W"
    var dmsFormatted: String  // "37°7'24.42"N 122°7'24.42"W"
}
```

### FieldMarker

```swift
struct FieldMarker: Identifiable {
    let id: UUID
    var name: String
    let coordinate: Coordinate
    let worldPosition: SIMD3<Float>
    let distanceFromOrigin: Double
    let bearing: Double
    let color: UIColor
    let timestamp: Date
}
```

### MarkerManager

```swift
class MarkerManager: ObservableObject {
    func addMarker(at: Coordinate, worldPosition: SIMD3<Float>, name: String)
    func removeMarker(id: UUID)
    func coordinateToWorldPosition(_ coordinate: Coordinate) -> SIMD3<Float>
    func worldPositionToCoordinate(_ position: SIMD3<Float>) -> Coordinate?
    func exportMarkersAsJSON() -> String
    func exportMarkersAsGeoJSON() -> String
}
```

## Export Formats

### JSON Format

```json
{
  "origin": {
    "latitude": 37.7749,
    "longitude": -122.4194
  },
  "fieldRadius": 100,
  "markers": [
    {
      "id": "uuid-string",
      "name": "Marker 1",
      "latitude": 37.7750,
      "longitude": -122.4195,
      "distanceFromOrigin": 15.2,
      "bearing": 45.5,
      "timestamp": "2024-01-15T10:30:00Z"
    }
  ]
}
```

### GeoJSON Format

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "name": "Marker 1",
        "distance": 15.2,
        "bearing": 45.5,
        "timestamp": "2024-01-15T10:30:00Z"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [-122.4195, 37.7750]
      }
    }
  ]
}
```

## Troubleshooting

### GPS Not Updating
- Ensure Location Services are enabled in Settings
- Grant "While Using" permission to the app
- Move outdoors for better GPS signal

### Markers Not Placing
- Check that immersive space is fully loaded
- Ensure you're tapping on the ground plane
- Wait for coordinate display to show values

### Inaccurate Coordinates
- Establish origin with best possible GPS accuracy
- Avoid placing markers far from origin (>500m)
- Re-center periodically for long sessions

## License

MIT License - See LICENSE file for details

## Contributing

Contributions welcome! Please submit pull requests with:
- Clear description of changes
- Test coverage for new features
- Documentation updates as needed
