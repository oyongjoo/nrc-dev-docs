# OpenWrt Package Compilation Verification Guide

## Overview
This guide explains how to verify if your OpenWrt package has been compiled successfully.

## Current Status: nrc_rtcm_vendor_ie_with_led.c

**Status**: ❌ **NOT COMPILED WITH LED SUPPORT**

The current IPK package was built on **2025-07-21 20:55** (before the LED source file was added on **2025-07-23 07:43**).

## Compilation Verification Methods

### 1. Check Build Timestamps
```bash
# Check source file modification time
stat package/network/utils/nrc_rtcm_ie/src/nrc_rtcm_vendor_ie_with_led.c

# Check compiled IPK modification time
stat bin/packages/mipsel_24kc/base/nrc_rtcm_ie_1_mipsel_24kc.ipk

# Compare: If IPK is older than source, recompilation is needed
```

### 2. Check Build Directory
```bash
# Look for compiled binary in build directory
find build_dir -name "nrc_rtcm_ie" -type f

# Check build directory contents
ls -la build_dir/target-mipsel_24kc_musl/nrc_rtcm_ie/
```

### 3. Verify Binary Contents
```bash
# Extract IPK package
mkdir -p temp_check
cd temp_check
tar -xzf ../bin/packages/mipsel_24kc/base/nrc_rtcm_ie_1_mipsel_24kc.ipk
tar -xzf data.tar.gz

# Check for LED-specific strings in binary
strings ./usr/bin/nrc_rtcm_ie | grep -E "(LED|led|send_led|station_pair|0x4C)"

# Check binary file type
file ./usr/bin/nrc_rtcm_ie
```

### 4. Check Makefile References
```bash
# Verify Makefile uses the correct source file
grep "nrc_rtcm_vendor_ie_with_led.c" package/network/utils/nrc_rtcm_ie/Makefile
```

### 5. Monitor Build Process
```bash
# Clean and rebuild with verbose output
make package/nrc_rtcm_ie/clean
make package/nrc_rtcm_ie/compile -j1 V=s
```

## How to Recompile

### Option 1: Clean Package Build
```bash
# Navigate to OpenWrt root
cd /home/builder/workspace/halow-main

# Clean the package
make package/nrc_rtcm_ie/clean

# Rebuild the package
make package/nrc_rtcm_ie/compile -j$(nproc)
```

### Option 2: Force Rebuild
```bash
# Remove build stamps
rm -rf build_dir/target-mipsel_24kc_musl/nrc_rtcm_ie/.*

# Rebuild
make package/nrc_rtcm_ie/compile -j$(nproc) V=s
```

### Option 3: Full Clean Build
```bash
# Clean everything for this package
make package/nrc_rtcm_ie/clean
make package/nrc_rtcm_ie/download
make package/nrc_rtcm_ie/prepare
make package/nrc_rtcm_ie/compile
```

## Expected Results After Successful Compilation

1. **New IPK timestamp**: Should be newer than source file
2. **Binary contains LED strings**: Should find "LED", "station_pair", "send_led_event", etc.
3. **Build log shows**: Compilation of `nrc_rtcm_vendor_ie_with_led.c`
4. **IPK size increase**: LED support adds ~2-3KB to binary size

## Common Issues

### Package Not Rebuilding
- Build system uses timestamps to determine if rebuild is needed
- Touch the source file: `touch package/network/utils/nrc_rtcm_ie/src/*.c`
- Remove build stamps: `rm build_dir/target-*/nrc_rtcm_ie/.*`

### Missing Dependencies
- Ensure libsqlite3 is available: `make package/sqlite3/compile`
- Check mosquitto library: `make package/mosquitto/compile`

### Build Errors
- Check build log: `make package/nrc_rtcm_ie/compile -j1 V=s 2>&1 | tee build.log`
- Look for linker errors related to sqlite3 or LED functions

## Verification Commands Summary
```bash
# Quick verification script
#!/bin/bash
PKG="nrc_rtcm_ie"
SRC_TIME=$(stat -c %Y package/network/utils/$PKG/src/nrc_rtcm_vendor_ie_with_led.c 2>/dev/null)
IPK_TIME=$(stat -c %Y bin/packages/mipsel_24kc/base/${PKG}_1_mipsel_24kc.ipk 2>/dev/null)

if [ -z "$SRC_TIME" ]; then
    echo "Source file not found!"
elif [ -z "$IPK_TIME" ]; then
    echo "IPK not built yet!"
elif [ $SRC_TIME -gt $IPK_TIME ]; then
    echo "Recompilation needed: Source is newer than IPK"
else
    echo "IPK is up to date"
fi
```