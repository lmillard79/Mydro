# QMydro Catchment Delineation Analysis

## Overview

The QMydro codebase is a comprehensive hydrologic modeling tool built around QGIS, consisting of:
- A QGIS Python plugin for the GUI interface
- C# backend algorithms for catchment delineation  
- Supporting scripts for rainfall design and processing

## Core Catchment Delineation Components

### C# Backend (`QMydro/scripts/CS/delineateCatch/`)

#### Program.cs - Main Entry Point
- **License Management**: Handles software licensing via HTTP API calls
- **GDAL Integration**: Configures geospatial data libraries
- **DEM Processing**: Reads elevation data and converts to processing arrays
- **Output Generation**: Creates GeoTIFF files and CSV data files
- **Main Function**: Orchestrates the entire delineation workflow

#### mainAlg.cs - Core Algorithm Implementation

**Key Classes:**
- `Subcatchment`: Represents individual drainage areas with upstream/downstream relationships
- `mainAlgorithm`: Contains the primary delineation logic

**Core Functions:**

##### `processingAlg()` - Main Processing Algorithm
```csharp
public static (int[,], int[,], int[,], List<float>, List<float>) processingAlg(
    float[,] elev, float noData_val, List<List<(int, int)>> outletCells, 
    float dx, float dy, float targetCatchSize, string model)
```

**Algorithm Steps:**
1. **Flow Direction Analysis**: Identifies drainage outlets using D8 method
2. **Watershed Delineation**: Creates initial catchment boundaries
3. **Flow Accumulation**: Calculates upstream contributing areas
4. **Subcatchment Generation**: Splits large catchments based on target size
5. **Channel Network Extraction**: Traces flow paths for hydrologic routing
6. **Hydraulic Parameter Calculation**: Computes slopes, lengths, and conveyance properties
7. **Model-Specific Output**: Generates files for Mydro or URBS hydrologic models

##### `dEightFlowAlg()` - D8 Flow Direction
- Implements the D8 (8-direction) flow algorithm
- Identifies cells where flow exits the DEM domain
- Uses elevation comparisons to determine flow directions

##### Supporting Functions:
- **BresenhamLine()**: Rasterizes vector lines to grid cells
- **Get_LineCells()**: Converts vector outlet lines to raster coordinates
- **Carve_LineCells()**: Modifies DEM elevations along specified paths
- **Shuffle()**: Randomizes cell processing order for load balancing

### QGIS Plugin Frontend (`QMydro/`)

#### QMydro.py - Main Plugin File
- QGIS plugin initialization and lifecycle management
- GUI event handling and user interaction
- Calls C# executable with processed parameters
- Handles output file management and display

#### QMydro_dockwidget.py - GUI Implementation
- PyQt-based dockable widget for user interface
- Parameter input forms and validation
- Integration with QGIS layer management

### Supporting Scripts (`QMydro/scripts/`)

#### designRainfall.py
- Rainfall intensity-duration-frequency analysis
- Design storm generation for hydrologic modeling

#### run_handler.py
- Execution management for external processes
- File I/O coordination between components

## Data Flow

```
DEM Raster → C# Processing → Catchment Delineation → 
Hydraulic Parameters → Model Files (.vec, .csv) → 
QGIS Display
```

## Modularization Strategy for Extraction

### 1. Core Algorithm Extraction

**Primary Module**: `catchment_delineation.py`
```python
import numpy as np
from typing import List, Tuple

class CatchmentDelineator:
    def __init__(self, dem: np.ndarray, cell_size_x: float, cell_size_y: float):
        self.dem = dem
        self.dx = cell_size_x
        self.dy = cell_size_y
        self.nodata_value = -9999
        
    def delineate_catchments(self, outlet_points: List[Tuple[float, float]], 
                           target_area_km2: float) -> dict:
        """Main catchment delineation method"""
        # Convert to C# algorithm logic
        pass
```

### 2. Wrapper Functions Needed

**Essential Functions to Extract:**
- `dEightFlowAlg()` → `find_flow_outlets()`
- Flow accumulation logic → `calculate_accumulation()`
- Subcatchment generation → `generate_subcatchments()`
- Channel network extraction → `extract_stream_network()`
- Hydraulic parameter calculation → `calculate_hydraulic_params()`

### 3. Dependencies to Handle

**Required Libraries:**
- NumPy (array operations)
- GDAL/OGR (spatial data handling)
- Priority queues (for flow algorithms)
- MathNet.Numerics equivalent (statistics/linear algebra)

### 4. Configuration Management

**Model-Specific Parameters:**
```python
MODEL_CONFIGS = {
    'Mydro': {
        'min_slope': 0.0005,
        'mannings_n': 0.03,
        'channel_threshold': 0.125
    },
    'URBS': {
        'min_slope': 0.0005,
        'channel_threshold': 1.0
    }
}
```

## Implementation Steps

1. **Extract Core Algorithm**: Copy `mainAlg.cs` logic to Python/C++
2. **Create Clean Interfaces**: Build wrapper functions for each major operation
3. **Handle Data Structures**: Convert C# arrays and lists to appropriate formats
4. **Implement Dependencies**: Replace GDAL calls with equivalent libraries
5. **Add Configuration**: Externalize model-specific parameters
6. **Create Tests**: Validate against known results

## Key Algorithm Characteristics

- **D8 Flow Algorithm**: Standard 8-direction flow routing
- **Priority Queue Processing**: Efficient handling of flow accumulation
- **Adaptive Subcatchment Sizing**: Dynamic splitting based on area targets
- **Channel Network Extraction**: Automated stream identification
- **Hydraulic Parameter Calculation**: Physical hydrologic properties
- **Model Compatibility**: Supports multiple hydrologic model formats

This analysis provides a foundation for extracting the catchment delineation capabilities into a standalone, reusable library suitable for integration into other hydrologic modeling projects.
