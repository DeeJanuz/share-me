# Phase 5: Body Part System Implementation Plan

## Overview
This plan outlines the implementation of the Body Part System, which is Phase 5 of our broader implementation plan. The Body Part System will build upon the previously implemented Attribute, Ingredient, and Ability/Targeting systems to create a comprehensive framework for body part creation, modification, and utilization in our game.

## Components and Dependencies

### Key Components to Implement:
1. BodyPartBase class (inherits from IngredientBase)
2. Stat calculation system with caching
3. Ability modification system
4. AbilitySlot class with segmentation logic
5. Ability assignment system
6. BodyPartFactory for creation and loading

### Dependencies:
- AttributeBase and AttributeRegistry (Phase 2) ✅
- IngredientBase and IngredientFactory (Phase 3) ✅
- AbilityBase and TargetingSystem (Phase 4) ✅
- SynergySystem (Phase 4) ✅

## User Stories

### 1. Body Part Base Implementation

**As a** developer  
**I want to** create a base class for body parts that extends the ingredient system  
**So that** body parts can store attributes while adding game-specific functionality

**Acceptance Criteria:**
- BodyPartBase inherits from IngredientBase
- BodyPartBase adds properties for stats, abilities, and ability slots
- BodyPartBase implements caching for expensive calculations
- BodyPartBase provides methods for adding/removing attributes and abilities
- BodyPartBase serializes to/from dictionary for save system compatibility

### 2. Stat Calculation System

**As a** developer  
**I want to** implement an efficient stat calculation system for body parts  
**So that** attribute modifiers correctly affect character stats without unnecessary recalculation

**Acceptance Criteria:**
- Stats are calculated based on attribute modifiers
- Modifier calculations are additive rather than compounding
- System uses lazy evaluation (only calculates when needed)
- System implements caching with dirty flags
- System provides clear methods for accessing calculated stats
- Stats are properly recalculated when attributes change

### 3. Ability Slot System

**As a** developer  
**I want to** implement an ability slot system for body parts  
**So that** abilities can be assigned to body parts with meaningful restrictions

**Acceptance Criteria:**
- AbilitySlot class manages slots for abilities
- System distinguishes between universal and unique slots
- Slot segmentation system implements efficiency penalties
- Slots connect to targeting and synergy systems
- Slots restrict which abilities can be assigned based on their properties
- Segmentation levels are dictated by how many ability slots are "segmented"

### 4. Ability Modification System

**As a** developer  
**I want to** implement a system for attributes to modify abilities  
**So that** body part attributes meaningfully affect ability behavior

**Acceptance Criteria:**
- System filters attributes based on ability's attribute blacklist
- Attributes modify abilities based on targeting type
- System implements effect application pipeline with priority
- System supports effect merging (additive, multiplicative, replacement)
- Modified properties are reset after ability use
- System connects to the synergy system for combined effects

### 5. Ability Assignment System

**As a** developer  
**I want to** implement a system for assigning abilities to body parts  
**So that** created body parts have appropriate abilities for gameplay

**Acceptance Criteria:**
- System determines available abilities for a body part
- Abilities are assigned to appropriate slots (universal or unique)
- System handles segmentation effects on ability efficiency
- System provides methods for adding/removing abilities
- System validates ability assignments based on slot compatibility

### 6. Body Part Factory

**As a** developer  
**I want to** implement a factory system for body part creation  
**So that** body parts can be created from templates with proper initialization

**Acceptance Criteria:**
- Factory generates body parts from templates
- Factory supports custom body part creation
- Factory integrates with resource and mod systems
- Factory handles initialization of stats, slots, and abilities
- Factory provides serialization for created body parts

## Technical Requirements

### 1. Stat Calculation Requirements
- **Lazy Evaluation**: Stats should only be recalculated when needed using dirty flags
- **Caching**: Calculated stats should be cached to avoid repeated calculations
- **Additive Modifiers**: Attribute stat modifiers should be additive and not compound
- **Modifier Application**: Each attribute multiplies the stat by its modifier and then adds it to the total

### 2. Ability Slot Requirements
- **Slot Types**: Support for universal slots (can hold any ability) and unique slots (restricted by ability type)
- **Segmentation**: Implementation of segmentation system that reduces efficiency when multiple slots are segmented
- **Efficiency Calculation**: Formula for efficiency penalty based on segmentation level
- **Serialization**: Slots must be serializable for save/load system

### 3. Ability Modification Requirements
- **Filtering**: Ability to filter attributes based on ability's blacklist
- **Targeting-Specific Modification**: Different modifications based on targeting type
- **Effect Pipeline**: Clear pipeline for applying effects in order of priority
- **Effect Merging**: Rules for merging effects (additive, multiplicative, replacement)
- **Reset Mechanism**: System to reset modified properties after ability use

### 4. Body Part Factory Requirements
- **Template-Based Creation**: Create body parts from pre-defined templates
- **Custom Creation**: Support for custom body part creation
- **Resource Integration**: Loading from resource system
- **Mod Support**: Integration with mod system for custom body parts
- **Serialization**: Support for saving/loading created body parts

### 5. Serialization Approach
Based on careful consideration of options, we will use a hybrid serialization approach:

- **Templates as Resources**: Use Godot's resource system (.tres) for body part templates, abilities, and attribute definitions
  - Templates will be loaded from the mod system
  - Templates will define base stats, available abilities, and slot configurations
  - Templates will be referenced by path in serialized player-created body parts

- **Player Creations as Dictionary/JSON**: Use dictionary serialization (saved as JSON) for player-created body parts
  - BodyPartBase will implement `to_dict()` and `from_dict()` methods
  - References to resources will be stored as paths rather than objects
  - Version information will be included for save compatibility

**Advantages of this approach:**
- Editor support for designing templates
- Human-readable player saves for easier debugging
- Clean separation between game data and player data
- Compatibility with the mod system
- Consistency with other parts of the system

**Implementation details:**
- Store non-resource objects directly in the dictionary
- Store references to resources as paths
- Include type information to facilitate proper deserialization
- Handle circular references appropriately
- Include version information for future-proofing

## Reference Examples and Implementation Details

### Directory Structure
Body part-related files should follow this structure:
```
scripts/
├── components/
│   ├── body_parts/
│   │   ├── body_part_base.gd       # Base class for body parts
│   │   ├── body_part_factory.gd    # Factory for creating body parts
│   │   ├── ability_slot.gd         # Class for ability slots
│   │   └── body_part_template.gd   # Class for loading templates
├── resources/                      # Resource data files
│   └── body_part_templates/        # Template definitions
mods/
├── base_mod/
│   ├── scripts/
│   │   ├── resources/
│   │   │   ├── body_part_templates/ # Mod-specific body part templates
│   │   │   │   ├── ogre_arm.tres    # Ogre arm template
│   │   │   │   └── ...
```

### Attribute Quality Ranks and Stat Modifiers
T1 attributes have tiers which dictate stat modifiers:
- F: modifier = 0.6, negative modifier = -1.0, weight = 1
- E: modifier = 0.7, negative modifier = -0.9, weight = 2
- D: modifier = 0.8, negative modifier = -0.8, weight = 3
- C: modifier = 0.9, negative modifier = -0.7, weight = 4
- B: modifier = 1.0, negative modifier = -0.6, weight = 5
- A: modifier = 1.2, negative modifier = -0.5, weight = 6
- S: modifier = 1.5, negative modifier = -0.3, weight = 7
- SS: modifier = 2.0, negative modifier = -0.2, weight = 8

### Base Body Part Structure
A base body part inherits from the ingredient class and has:
- **Base Stats**: Health, strength, dexterity, speed, efficiency, mana, stamina
- **Abilities**: List of abilities with percentage chance of availability (e.g., Smash: 30%)
- **Ability Slots**: Specified as total:universal (e.g., 3:1 means 3 total slots with 1 universal)
- **Ability Slot Segmentation (optional)**: Allows for more infusions at the cost of efficiency penalties

### Stat Calculation Formula
Stats are calculated additively using the following approach:
```
final_stat = base_stat + sum(base_stat * attribute_modifier)
```

For example, with a base health of 50 and attributes Light:B (modifier = 1.2) and Energy:C (modifier = -0.7):
```
health = 50 + (50 * 1.2) + (50 * -0.7) = 50 + 60 - 35 = 75
```

### Ability Modification
Abilities can be modified by attributes based on targeting type. Each attribute defines effects for different targeting types:
- single_target_external
- beam
- aoe
- random_area
- single_target_internal
- mass_target_external
- mass_target_internal
- cone

Abilities can blacklist certain attribute categories, preventing those attributes from modifying them.

### Ability Slot Types
Two types of ability slots:
1. **Universal Slots**: Can accept any ability
2. **Unique Slots**: For abilities specified in the body part template based on their percentage

### Concrete Example: Ogre Arm Template

```gdscript
# Example Ogre Arm Template Definition
extends "res://scripts/components/body_parts/body_part_template.gd"

func _init() -> void:
    name = "Ogre Arm"
    description = "A powerful arm from an ogre, good for smashing things."
    
    # Base stats
    base_stats = {
        "health": 50,
        "strength": 20,
        "dexterity": 5,
        "speed": 5,
        "efficiency": 0,
        "mana": 5,
        "stamina": 20
    }
    
    # Abilities with availability chance
    abilities = {
        "Smash": 30,  # 30% chance
        "Gore": 30,   # 30% chance
        "Crush": 30   # 30% chance
    }
    
    # Ability slots (total:universal)
    total_slots = 3
    universal_slots = 1
```

### Body Part Creation Example
Given the Ogre Arm template and these ingredients:
- Electric Slime Essence (energy:C, wet:D, pressure:E)
- Fire Dire Wolf Heart (darkness:B, heat:B, carbon:C)
- Angelic Feather (light:B)

The player might combine:
1. light:B + carbon:C = low_mass (T2 attribute)
2. darkness:B + heat:B + wet:D = tainted_mist (T3 attribute)
3. Keep energy:C as is

The resulting body part would have:
- **Name**: "Lightweight whispering ogre arm" (or player's choice)
- **Stats**: Modified based on the attributes (e.g., health = 75)
- **Abilities**: Those that "won" the probability check (e.g., Smash and Gore)
- **Ability Slots**: Player chooses which ability goes in which slot
- **Attributes**: low_mass, tainted_mist, energy

### Implementation Considerations

1. **Ability Assignment**:
   - When a body part is created, each ability from the template has its chance rolled
   - Successful abilities become available for assignment to slots
   - Player can choose which abilities go into which slots
   - Universal slots can accept abilities from other sources

2. **Stat Caching**:
   - Stats should only be recalculated when attributes change
   - Use dirty flags to track when recalculation is needed
   - Cache calculated values to avoid repeated calculations

3. **Ability Modification Pipeline**:
   - Abilities should be modified by attributes at the time of use
   - Filtered based on ability's blacklist
   - Modified based on targeting type
   - Synergies applied as defined in the synergy system
   - All modifications reset after ability use

4. **Serialization**:
   - Templates stored as resources (.tres)
   - Created body parts serialized to dictionaries/JSON
   - Store references to resources by path
   - Include version information for future compatibility

## Implementation Order
1. Implement BodyPartBase class with basic properties
2. Implement stat calculation system with caching
3. Implement AbilitySlot class
4. Implement body part stat system that uses attributes to calculate final stats
5. Implement ability modification system
6. Implement ability assignment system
7. Implement BodyPartFactory

## Questions and Clarifications Needed

1. **Stat Calculation Formula**: ✅ Resolved - All stats will use the identical formula, with additive modifiers from attributes.

2. **Segmentation Levels**: ✅ Resolved - We'll use a basic formula for now, with plans to implement a more complex formula in the future.

3. **Ability Assignment Probabilities**: ✅ Resolved - Abilities will be added to slots using weighted probabilities as defined in the body part template. Each ability in the template will specify a percentage chance of being available.

4. **Ability Slot Compatibility**: ✅ Resolved - Any ability specified by the body part can be placed in any slot. Additionally, abilities from other body parts can be added if that body part is used as an ingredient in creation.

5. **Body Part Templates**: ✅ Resolved - We will implement a generic "Ogre Arm" template for initial development. Modders/developers can add new body part types as resources in their modpacks, with each template file specifying the properties of the body part.

6. **Ability Modification Priority**: ✅ Resolved - Since attribute effects on stats are additive, the order of application doesn't matter. Attribute effects on abilities are handled by the synergy system and do not need to be controlled by the body part.

7. **Serialization Format**: ✅ Resolved - We'll use a hybrid approach: Godot resources for templates and dictionary/JSON for player creations.

## Testing Approach

1. **Unit Tests**:
   - Test stat calculation with different attribute combinations
   - Test ability slot segmentation with varying configurations
   - Test ability modification with different attributes and targeting types

2. **Integration Tests**:
   - Test BodyPartBase with AbilitySlot and ability assignment
   - Test BodyPartFactory with resource system and mod support
   - Test complete body part creation and modification workflow

3. **UI Testing**:
   - Create a test scene for body part creation with visual feedback
   - Implement debugging UI for inspecting body part properties
   - Validate serialization and deserialization with UI feedback

## Next Steps
After completing this phase, we will proceed to Phase 6: Body Part Creation System, which will focus on the complete body part creation flow, including the user interface for combining ingredients and the name generation system.
