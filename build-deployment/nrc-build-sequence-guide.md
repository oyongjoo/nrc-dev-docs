# NRC Build Sequence Guide

## Overview
Optimal build sequence for NRC HaLow components in OpenWrt environment.

## Build Dependencies
1. **Base System**: OpenWrt toolchain and kernel
2. **MAC80211 Stack**: Wireless framework
3. **NRC Driver**: HaLow kernel module
4. **Applications**: User-space utilities

## Recommended Build Order

### Phase 1: Foundation
```bash
# 1. Toolchain preparation
make tools/install -j$(nproc)
make toolchain/install -j$(nproc)

# 2. Kernel preparation
make target/linux/prepare
```

### Phase 2: Wireless Stack
```bash
# 3. MAC80211 framework
make package/kernel/mac80211/compile -j$(nproc)

# 4. Verify wireless framework
ls build_dir/target-*/linux-*/mac80211-*/
```

### Phase 3: NRC Components
```bash
# 5. NRC kernel module
./nrc-smart-build.sh smart

# 6. NRC applications
make package/network/utils/nrc_rtcm_ie/compile -j$(nproc)
```

### Phase 4: Validation
```bash
# 7. Verify NRC module
find build_dir -name "nrc.ko" -exec ls -lh {} \;

# 8. Check applications
ls bin/packages/mipsel_24kc/utils/nrc_*
```

## Critical Build Points

### NRC Module Dependencies
- cfg80211 framework must be built first
- HaLow-specific patches applied
- LED system integration verified

### Common Issues
- **Silent failures**: Use verification steps
- **Cache conflicts**: Clear .built stamps
- **Header dependencies**: Verify include paths

## Performance Optimization
- Parallel builds where possible
- Incremental compilation
- Build cache management

---
*Build sequence validated: 2025-07-29*