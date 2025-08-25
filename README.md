# Houdini Studio Package Management Guide

*JSON-based environment management for small mixed-platform studios with render farms*

## Table of Contents
- [Why Packages Over houdini.env](#why-packages)
- [Studio Architecture](#studio-architecture)
- [Package Structure & Loading](#package-structure)
- [Bootstrap Strategy](#bootstrap-strategy)
- [Migration Process](#migration-process)
- [Testing & Validation](#testing-validation)
- [Troubleshooting](#troubleshooting)
- [Future Improvements](#future-improvements)

---

## Why Packages Over houdini.env {#why-packages}

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

## Studio Architecture {#studio-architecture}

### Our Setup
- **Artists**: <5 Windows workstations
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

## Package Structure & Loading {#package-structure}

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

⚠️ **TODO**: Document behavior when packages conflict
⚠️ **TODO**: Add explicit priority handling in package metadata

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

## Bootstrap Strategy {#bootstrap-strategy}

### Current Approach: Static Shortcuts
- `vvox_artist.json` and `vvox_renderfarm.json` in bootstrap folder
- Copied to each machine once, never edited locally
- Acts as immutable pointer to NAS-centralized configs

### Why This Works for Small Studios
✅ **Consistency**: No config drift between machines
✅ **Simple**: Artists can't accidentally break their setup
✅ **Rollback safe**: Changes only happen at NAS level
✅ **Clear separation**: Artist vs farm configs obvious

### Potential Improvements
```json
// Add to bootstrap configs:
{
  "bootstrap_metadata": {
    "config_type": "artist",
    "last_sync": "auto_timestamp",
    "fallback_behavior": "local_cache_if_nas_down"
  }
}
```

💡 **Consider**: Bootstrap validation script to test NAS connectivity

---

## Migration Process {#migration-process}

### From houdini.env to Packages

**Phase 1: Preparation**
- [ ] Audit all existing houdini.env files
- [ ] Identify common vs machine-specific settings
- [ ] Create test packages on development machine

**Phase 2: Package Creation**
- [ ] Build shared package with common tools
- [ ] Create artist-specific package
- [ ] Setup renderfarm package with minimal UI tools
- [ ] Test cross-platform path resolution

**Phase 3: Rollout**
- [ ] Week 1: Test on one artist workstation
- [ ] Week 2: Deploy to remaining artist machines  
- [ ] Week 3: Migrate render farm nodes
- [ ] Week 4: Remove old houdini.env files

**Emergency Rollback Plan**
1. Restore houdini.env files from backup
2. Remove/rename package files
3. Restart Houdini sessions
4. Communication to team

---

## Testing & Validation {#testing-validation}

### Pre-deployment Checklist
- [ ] **Path Validation**: All paths accessible on Windows + Linux
- [ ] **Houdini Startup**: Clean launch on test machines
- [ ] **Tool Availability**: Custom tools load correctly
- [ ] **Performance**: Startup time acceptable vs. houdini.env
- [ ] **Asset Access**: Shared assets resolve properly

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
# TODO: Create validation scripts
check_nas_connectivity.py
validate_package_paths.py  
test_houdini_startup.py
compare_env_vs_package.py
```

---

## Troubleshooting {#troubleshooting}

### Common Issues

**"Package not found" errors**
- Check NAS connectivity
- Verify mount points on Linux
- Test UNC path access on Windows

**Path resolution failures**
- Validate JSON syntax
- Check platform conditionals
- Test path existence

**Conflicting packages**
- Document loading order
- Check for duplicate settings
- Use package metadata for debugging

### Emergency Procedures
1. **NAS Down**: Document offline workflow
2. **Bad Package**: Rollback procedure
3. **Artist Blocked**: Temporary local config

---

## Future Improvements {#future-improvements}

### Short Term
- [ ] Add package versioning system
- [ ] Create validation/testing scripts  
- [ ] Document package conflict resolution
- [ ] Build rollback automation

### Medium Term  
- [ ] Package update notification system
- [ ] Automated testing pipeline
- [ ] Performance monitoring
- [ ] Multi-site package syncing

### Long Term
- [ ] GUI package manager
- [ ] Dynamic package loading
- [ ] Integration with pipeline tools
- [ ] Package sharing between studios

---

## Lessons Learned

### What Worked
- Bootstrap approach prevents config drift
- Clear separation by role (artist/farm) 
- JSON handles cross-platform paths well
- Centralized updates save maintenance time

### What Could Be Better
- Need explicit package priority rules
- Missing validation/testing automation  
- No rollback strategy initially
- Package versioning added later

### For Other Small Studios
- Start simple: shared + artist packages sufficient initially
- Test thoroughly before studio-wide deployment
- Plan rollback procedure before migration
- Document everything - you'll forget the details

---

*This documentation grows with our pipeline. Add examples, gotchas, and improvements as encountered.*

**Last Updated**: [Current Date]  
**Studio Size**: <5 artists, 4 render nodes  
**Houdini Versions**: 20.5