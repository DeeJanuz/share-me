# ModularApp Framework Documentation

## Project Structure

### Root Directory Organization
- `.config/` - Engine configuration and cache
- `.docs/` - Project documentation
- `src/` - Core application logic and classes
- `plugins/` - Plugin support
  - `core_plugin/` - Contains base application resources
  - `custom_plugins/` - Directory for additional plugins
- `ui/` - UI scene files
- `assets/` - Application assets and resources

### Key Directories
```
src/
├── components/      # Component-based architecture
│   ├── properties/  # Property system classes
│   ├── resources/   # Resource system classes
│   ├── actions/     # Action system classes
│   ├── selectors/   # Selector system classes
│   └── entities/    # Entity related classes
├── resources/       # Resource data files
├── singletons/      # Singleton classes 
├── ui/              # User interface elements
├── test_scenes/     # Testing scenes
└── utils/           # Utility classes
```

## Core Systems

### 1. Entity System
#### Base Entity Class (`src/entities/entity.gd`)
- Extends: `Node2D`
- Key Features:
  - Core entity properties (health, energy, speed, etc.)
  - Movement system with acceleration and friction
  - Animation state machine
  - Resource management (primary and secondary resources)
  - AI behavior support for NPCs
  - Comprehensive movement and physics integration

### 2. Entity Component System
#### Key Components:
- **EntityComponent Class**
  - Extends: `Node`
  - Features:
    - Property infusion system
    - Visual management for models
    - Stats integration with base entity
    - Transform modifier system
    - Signal system for state changes

### 3. Action System
#### Base Action Class (`src/actions/action.gd`)
- Extends: `RefCounted`
- Properties:
  - name: String
  - description: String
  - is_active: bool
  - cooldown: float
  - property_modifiers: Array[Dictionary]
- Key Features:
  - Support for both active and passive actions
  - Property modifier system
  - Customizable activation logic

### 4. Property System
#### Tiers and Categories
1. **Tier 1 (Base) Properties:**
   - Fundamental properties (primary, secondary, tertiary, etc.)
   - Direct stat modifications
   - Quality ranks from D to S

2. **Tier 2-4 Properties:**
   - Created through T1 combinations
   - Multiple creation paths possible
   - Increased potency with higher tiers
   - Complex effect combinations

#### Property Features
- Primary category (for simplified mechanics)
- Multiple categories (for advanced mechanics)
- Quality ranks (D-S)
- Stat modifiers
- Synergy definitions
- Selector type modifiers
- Combination property requirements (for T2-T4)

#### Implementation Details
- **PropertyResource** (`plugins/core_plugin/src/resources/property_data/property_resource.gd`):
  - Extends `Resource` for serialization support
  - Defines core property features: name, tier, categories, primary_category
  - Provides rank and weight systems for T1 properties
  - Stores combination_properties for T2+ properties

- **Concrete Properties**:
  - Each property defined in its own resource file for better modularity
  - Directory structure:
    - T1: `plugins/core_plugin/src/resources/property_data/tier1/`
    - T2: `plugins/core_plugin/src/resources/property_data/tier2/`
    - T3: `plugins/core_plugin/src/resources/property_data/tier3/`
    - T4: `plugins/core_plugin/src/resources/property_data/tier4/`
  - Enables plugin extensibility with easy addition/modification of properties

- **PropertyBase** (`src/components/properties/property_base.gd`):
  - Runtime representation of properties after loading from resources
  - Handles rank-based modifier calculations
  - Provides serialization to/from dictionaries
  - Stores primary_category and combination_properties
  - Supports simplified and advanced mechanics modes

- **PropertyCategory** (`src/components/properties/property_category.gd`):
  - Defines all available categories (primary, secondary, tertiary, etc.)
  - Manages compatibility groups and incompatible pairings
  - Provides selector type modifiers for different action patterns

- **PropertyRegistry** (`src/components/properties/property_registry.gd`):
  - Loads and registers properties from plugin resources
  - Manages property combinations
  - Handles resource loading for properties
  - Maintains combination rules for different tiers

- **PropertySynergy** (`src/components/properties/property_synergy.gd`):
  - Processes synergy effects between properties
  - Handles conditional synergies based on effects
  - Manages stacking effects

### 5. Synergy System
#### Key Components:
- **SynergyResource** (`src/components/properties/synergy_resource.gd`):
  - Extends: `Resource`
  - Properties:
    - source_property: String
    - target_category: String
    - selector_effects: Dictionary
    - stack_type: StackType enum (ADDITIVE, MULTIPLICATIVE, UNIQUE)
  - Features:
    - Defines property-to-category synergy relationships
    - Maps selector types to specific effects
    - Provides stacking behavior control

- **SynergyEffect** (`src/components/properties/synergy_effect.gd`):
  - Extends: `Resource`
  - Properties:
    - effect_type: EffectType enum
    - effect_params: Dictionary
    - duration: float
    - selector_type: String
    - application_method: ApplicationMethod enum
  - Features:
    - Represents specific effects (damage, status, selector modifications, environmental)
    - Provides specialized application logic for different effect types
    - Handles serialization of effect data

- **SynergyRegistry** (`src/components/properties/synergy_registry.gd`):
  - Extends: `Node` (Singleton)
  - Key Features:
    - Manages synergy resources with efficient lookup by property, category, or property-category pair
    - Loads synergy definitions from plugin directories
    - Provides caching for frequently used synergy patterns

- **SynergyManager** (`src/components/properties/synergy_manager.gd`):
  - Extends: `Node`
  - Key Features:
    - Handles synergy detection, application, and environmental effect management
    - Manages physics-based environmental effects using Area2D
    - Provides interaction detection between actions and environmental effects
    - Caches synergies for entity components for performance optimization

- **EntitySynergyCache** (`src/components/entities/entity_synergy_cache.gd`):
  - Extends: `RefCounted`
  - Key Features:
    - Caches synergies for entity components
    - Pre-calculates potential synergy effects by selector type
    - Optimizes runtime performance during execution
    - Provides stats and monitoring capabilities

#### Directory Structure
```
plugins/
├── core_plugin/
│   ├── src/
│   │   ├── resources/
│   │   │   ├── synergy_data/
│   │   │   │   ├── primary_secondary_synergy.tres
│   │   │   │   ├── secondary_tertiary_synergy.tres
│   │   │   │   └── ...
```

#### Testing Infrastructure
- **Synergies Tab in Resource Test Scene**:
  - Located in `src/test_scenes/resource_test_scene.tscn`
  - Features:
    - Selection UI for source properties, target categories, and selector types
    - Dynamic display of synergy effects and parameters
    - Detailed view of stacking behavior and effect properties
  - Integration with singleton registries for data access

### 6. Composition System
#### Key Mechanics:
- **Resource Combination:**
  - Source elements provide base resources
  - Quality-based property extraction
  - Weight-based probability system
  
- **Entity Creation:**
  - Base template selection
  - Resource combination
  - Property synthesis
  - Action slot management
  - Stat calculation

### 7. Plugin System
#### Key Components:
- **Core Plugin Structure:**
  - Core application resources stored in plugin-like structure
  - Resource files (.tres) located in plugin directories
  
- **Plugin Loading System:**
  - DataManager loads resources from plugins
  - Configurable load order
  - Resource override capabilities
  
- **Plugin Creation Framework:**
  - Simple directory structure that mirrors core_plugin
  - API for creating custom resources
  - Integration with existing application systems

#### Directory Structure
```
plugins/
├── core_plugin/              # Contains base application resources
│   ├── src/                  # Script files for base functionality
│   │   ├── resources/        # Resource definitions and data
│   │   │   ├── property_data/       # Property resources by tier
│   │   │   ├── entity_templates/    # Templates for entity creation
│   │   │   ├── resource_data/       # Resource definitions
│   │   │   ├── synergy_data/        # Synergy effect definitions
│   │   │   ├── selectors/           # Selector type resources
│   │   │   ├── shapes/              # Physical shape definitions
│   │   │   └── resource_data_manager.gd  # Manages resources
│   ├── ui/              # UI files for base application
│   └── assets/          # Visual and audio assets
├── custom_plugins/      # Directory for additional plugins
    ├── plugin_name/     # Individual plugin folder
    │   ├── src/         # Custom scripts extending base functionality
    │   ├── ui/          # Custom UI elements
    │   ├── assets/      # Custom assets
    │   └── plugin_info.json # Plugin metadata and dependencies
```

### 8. Debug System
#### Key Components:
- **DebugManager Singleton (`src/singletons/debug_manager.gd`):**
  - Centralized logging system with customizable log levels
  - Debug visualization tools
  - Performance metrics tracking
  - Debug panel integration
  
- **Debug Panel (`src/ui/debug/debug_panel.gd`):**
  - In-app UI for monitoring and debugging application state
  - Resource inspection capabilities
  - Dynamic property viewing
  - Custom debug visualizations

- **Debug Visualizer (`src/ui/debug/debug_visualizer.gd`):**
  - Visual indicators for debugging spatial elements
  - Customizable appearance and duration

### 9. Utility Systems
#### ArrayUtils (`src/utils/array_utils.gd`)
- Type conversion functions:
  - `to_string_array()` - Convert generic Array to Array[String]
  - `to_int_array()` - Convert generic Array to Array[int]
  - `to_float_array()` - Convert generic Array to Array[float]
  - `to_bool_array()` - Convert generic Array to Array[bool]
- Array manipulation:
  - `append_to_typed_array()` - Safely append items to typed arrays
  - `copy_to_typed_array()` - Copy all items from source to typed array
  - `get_typed_array_from_dict()` - Extract and convert arrays from dictionaries

## Scene Structure
### Entity Templates
- `ui/entities/template_a.tscn`
- `ui/entities/template_b.tscn`
- Features:
  - Model integration
  - Component attachment points
  - Animation rigging
  - Visual customization options

## Implementation Progress
### Completed Features
1. Base entity system with movement and physics
2. Animation state machine
3. Basic action framework
4. Initial component system
5. Property system foundation
6. **Entity Component System Phase 1** (Completed March 15, 2025)
   - Core data structures for properties, resources, and entities
   - Resource-based serialization system
   - Plugin support infrastructure
   - Testing framework with interactive UI
   - Comprehensive debugging tools
   - Array utilities for type-safe operations
7. **Phase 2: Property System Implementation** (Completed March 16, 2025)
   - Created `PropertyResource` class for storing property definitions
   - Implemented concrete property implementations for T1-T4 properties
   - Enhanced `PropertyBase` with primary category support and multiple combination paths
   - Updated `PropertyRegistry` for multiple combination path support
   - Integrated test scene with property testing and visualization
8. **Phase 3: Resource System Implementation** (Completed March 29, 2025)
   - Created core resource system classes (`ResourceItem`, `ResourceBase`, `ResourceFactory`)
   - Implemented resource property storage with rank system (D-S)
   - Added integration with property registry for realistic resource creation
   - Developed randomized resource generation system
   - Created custom resource creation interface with validation
   - Enhanced UI for resource display with visual separation and clear formatting
   - Improved resource test scene with consistent UI behavior
9. **Phase 4: Selector Framework Implementation** (Completed April 6, 2025)
   - Implemented a flexible, resource-based selector system with multiple selector types
   - Created core components: `SelectorTypeResource`, `SelectorProperties`, `SelectorComponent`
   - Developed specialized calculation systems in `SelectorCalculator` for different patterns
   - Built visualization system in `SelectorVisualizer` for real-time feedback
   - Integrated with physics system through `SelectorPhysics` 
   - Implemented position handling with `PositionManager`
   - Created five base selector types: single_target, multi_target, area, directional, linear
   - Added advanced collision behavior for linear selection with physics-based reflections
   - Developed comprehensive testing framework with interactive UI for all selector types
   - Ensured compatibility with both grid-based and continuous entity positioning
   - Established extension points for plugin custom selector types
   - Created detailed documentation in `.docs/selector_framework.md`
10. **Synergy System Implementation** (April 7, 2025)
    - Created core synergy classes (SynergyResource, SynergyEffect, SynergyRegistry, SynergyManager)
    - Implemented physics-based environmental effect system with area detection
    - Added property-to-category synergy mapping with selector-specific effects
    - Created synergy testing infrastructure with dedicated UI tab
    - Implemented performance optimizations with entity synergy caching
    - Added plugin support through resource-based configuration
    - Fixed class_name conflicts with singleton registries
    - Simplified system by removing conditional script functionality
    - Enhanced DataManager to load synergy resources from plugins
    - Created sample synergy resources for testing and demonstration

### In Development
1. Composition system implementation
2. Advanced action effects
3. Entity customization
4. UI integration
5. Interaction mechanics

### Planned Features
1. Advanced AI behaviors
2. Extended property combinations
3. Visual effect system
4. Plugin system extensions
5. Network multiplayer support

### Recently Completed
1. Comprehensive resource system documentation (`src/resources/README.md`)
2. Initial resource and property resource classes
3. Debug panel for resource inspection
4. Entity template resource structure
5. Data manager integration with resource loading
6. Restructured resource system for plugin support
   - Reorganized directory structure with core_plugin approach
   - Added plugin loading capabilities to DataManager
   - Implemented resource override system
7. DebugManager bug fix
   - Renamed `log()` function to `log_message()` to avoid name collisions
   - Updated logging convenience methods
   - Improved error reporting and debugging capabilities
8. **ArrayUtils Implementation** (March 15, 2025)
   - Type-safe array conversion utilities
   - Solutions for strict typing system
   - Utility functions for common array operations
   - Integration with resource loading and property system
9. **Testing Framework Enhancement** (March 15, 2025)
   - Created `resource_test_scene.tscn` with tabbed interface
   - Implemented sample resource creation and inspection
10. **Dictionary-Based Combination Options** (March 16, 2025)
    - Implemented named combinations with descriptions via dictionary approach
    - Added backward compatibility with array-based combinations
    - Enhanced property validation and formation logic
    - Created example properties with descriptive combinations
11. **Property Testing UI Enhancements** (March 16, 2025)
    - Added comprehensive property details popup
    - Implemented responsive UI layout with proper sizing
    - Enhanced combination display with visual hierarchy and organization
    - Added de-duplication system for combination paths
    - Created proper error handling for property access
12. **Property Path Resolution and Code Cleanup** (March 16, 2025)
    - Fixed file path resolution in property scripts
    - Streamlined property code with better type handling
    - Removed redundant backward compatibility code
    - Implemented safer property access with null checks
    - Added custom array comparison for duplicate detection
13. **UI and Resource System Enhancements** (March 29, 2025)
    - Implemented randomized test resource generation using T1 properties from registry
    - Enhanced resource display with bordered containers and proper spacing
    - Improved custom resource form with consistent styling and better usability
    - Fixed resource test scene UI issues with proper null checks and error handling
    - Added comprehensive resource inspection with property display
    - Implemented on-demand form creation for better resource management
    - Removed automatically created test resources for cleaner UI
    - Enhanced visual distinction between UI elements for better user experience

## Development Guidelines
1. Follow SOLID principles
2. Maintain clear separation of concerns
3. Use typed arrays and dictionaries
4. Implement comprehensive error handling
5. Document all public APIs
6. Write unit tests for core systems

## Technical Notes
- Using modern framework engine
- Primary application language
- Model integration with UI
- Signal-based communication between systems
- Resource-based data management
- Comprehensive documentation in key directories (`src/resources/README.md`)
- Robust debugging system with customizable log levels and debug panel integration

## Key Documentation Files
- `.docs/project-index-example.md` - This file, overall project structure
- `docs/entity-creation-plan.md` - Implementation plan for entity creation
- `src/resources/README.md` - Resource system documentation with diagrams
- `docs/progress.md` - Development log tracking feature implementations, challenges, and solutions
