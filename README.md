# Houdini Studio Package Management Guide

*JSON-based environment management for small mixed-platform studios with render farms*

## Table of Contents
- [Why Packages Over houdini.env](#why-packages-over-houdinienv)
- [Studio Architecture](#studio-architecture)
- [Package Structure & Loading](#package-structure--loading)
- [Bootstrap Strategy](#bootstrap-strategy)
- [Migration Process](#migration-process)
- [Testing & Validation](#testing--validation)
- [Future Improvements](#future-improvements)
- [Debugging Commands](#debugging-commands)

---

## Why Packages Over houdini.env

### The Problem with houdini.env
- **Scattered files**: Each machine has its own env file
- **Maintenance nightmare**: Updates require touching every workstation
- **Version drift**: Hard to keep artist and farm configs in sync
- **No rollback**: Breaking changes affect everyone immediately
- **Platform mixing**: Windows/Linux path handling gets messy

### Package Benefits
- **Centralized management**: One source of truth on NAS
- **Atomic updates**: Change one file, affects all machines
- **Clear separation**: Artist vs renderfarm configurations
- **Cross-platform**: JSON handles Windows UNC + Linux paths cleanly
- **Rollback capable**: Version control friendly

---

## Studio Architecture

### Our Setup
- **Artists**: ~5 Windows workstations
- **Render Farm**: 2 Linux GPU nodes + 2 Windows GPU nodes 
- **Storage**: Central NAS with dual-platform access
- **Pipeline**: Deadline integration + custom tools

### Package Hierarchy
```
____STUDIO_PACKAGES/
├── bootstrap/           # Machine-local shortcut configs
│   ├── vvox_artist.json
│   └── vvox_renderfarm.json
├── shared/              # Common tools and assets
├── artist/              # Artist-specific configurations
└── renderfarm/          # Farm-only settings
```

---

## Package Structure & Loading

### Cross-Platform Path Handling
```json
{
  "package_path": [
    {
      "houdini_os == 'windows'": "//Vvox-nas-1/projects/_____ASSETS/3D/HOUDINI_ASSETS/____STUDIO_PACKAGES/shared"
    },
    {
      "houdini_os == 'linux'": "/mnt/VVOX-NAS-1/projects/_____ASSETS/3D/HOUDINI_ASSETS/____STUDIO_PACKAGES/shared"
    }
  ]
}
```

### Package Loading Priority
**Current Order**: Bootstrap → Shared → Artist/Renderfarm
```
1. Bootstrap (machine-local shortcuts)
2. Shared (common base)
3. Artist OR Renderfarm (role-specific)
```

**TODO**: Document behavior when packages conflict  
**TODO**: Add explicit priority handling in package metadata

### Package Metadata Template
```json
{
  "package_metadata": {
    "version": "1.0.0",
    "last_updated": "2025-08-24", 
    "validated_houdini_versions_major": ["20.5"],
    "validated_houdini_versions_minor": ["445"],
    "description": "Artist workstation base config",
    "maintainer": "[your_name]",
    "rollback_version": "0.9.0"
  },
  "package_path": [
    // actual package content
  ]
}
```

---

## Bootstrap Strategy

### Current Approach: Static Shortcuts
- `vvox_artist.json` and `vvox_renderfarm.json` in bootstrap folder
- Copied to each machine once, never edited locally
- Acts as immutable pointer to NAS-centralized configs

### Why This Works for Small Studios
- **Consistency**: No config drift between machines
- **Simple**: Artists can't accidentally break their setup
- **Rollback safe**: Changes only happen at NAS level
- **Clear separation**: Artist vs farm configs obvious

### Potential Improvements
Consider bootstrap validation script to test NAS connectivity

---

## Migration Process

### From houdini.env to Packages

**Preparation**
- Audit all existing houdini.env files
- Identify common vs machine-specific settings
- Create test packages on development machine

**Package Creation**
- Build shared package with common tools
- Create artist-specific package
- Setup renderfarm package with minimal UI tools
- Test cross-platform path resolution

**Rollout**
- Deploy to one test workstation first
- Migrate remaining artist machines
- Update render farm nodes
- Remove old houdini.env files after validation

**Emergency Rollback Plan**
1. Restore houdini.env files from backup
2. Remove/rename package files
3. Restart Houdini sessions
4. Communicate to team

---

## Testing & Validation

### Pre-deployment Checklist
- **Path Validation**: All paths accessible on Windows + Linux
- **Houdini Startup**: Clean launch on test machines
- **Tool Availability**: Custom tools load correctly
- **Performance**: Startup time acceptable vs. houdini.env
- **Asset Access**: Shared assets resolve properly

### Test Package Strategy
Create `test_artist.json` with:
```json
{
  "package_metadata": {
    "version": "test",
    "description": "Safe testing environment",
    "test_mode": true
  }
}
```

### Validation Script Ideas
```bash
check_nas_connectivity.py
validate_package_paths.py  
test_houdini_startup.py
compare_env_vs_package.py
```

---

## Future Improvements

### Short Term
- Add package versioning system
- Create validation/testing scripts  
- Document package conflict resolution

### Long Term
- Package update notification system
- GUI package manager
- Integration with pipeline tools

---

## Debugging Commands

### Python/Hython Environment Debugging
```python
import os
print(os.environ.get("HOUDINI_PATH"))
print(os.environ.get("HOUDINI_PACKAGE_PATH"))

# List all loaded packages
import hou
for package in hou.text.packageList():
    print(package)
```

### Hscript Environment Checking
```hscript
echo $HOUDINI_PATH
echo $HOUDINI_PACKAGE_PATH
echo $HIP
```

---

*This documentation grows with our pipeline. Add examples, gotchas, and improvements as encountered.*

**Last Updated**: [2025-08-25]  
**Studio Size**: <5 Houdini artists, 4 render nodes  
**Houdini Versions**: 20.5