# NRC Driver Development Guide

## Overview
Comprehensive guide for developing and maintaining the NRC HaLow kernel driver within the OpenWrt environment.

## Driver Architecture

### Core Components
- **nrc-main.c**: Driver initialization and management
- **nrc-netlink.c**: Netlink communication interface
- **nrc-mac.c**: MAC layer implementation
- **nrc-vendor.c**: Vendor-specific functionality

### HaLow-Specific Features
- **802.11ah Support**: Long-range, low-power wireless
- **Vendor IE Processing**: Custom information elements
- **LED Control System**: 3-bit LED/WAKE commands
- **Power Management**: Advanced sleep modes

## Development Environment Setup

### Prerequisites
```bash
# OpenWrt development environment
cd /home/builder/workspace/halow-main

# Kernel module development tools
make menuconfig
# Navigate to: Kernel modules -> Wireless Drivers -> NRC
```

### Build Configuration
```makefile
# Key configuration options
CPTCFG_NRC=m
CPTCFG_WLAN_VENDOR_NEWRACOM=y
CONFIG_NRC_HIF_PRINT_FLOW_CONTROL=y
```

## Driver Development Workflow

### 1. Code Modification
```bash
# Edit driver sources
vim package/kernel/mac80211/src/drivers/net/wireless/nrc/
```

### 2. Compilation
```bash
# Use smart build script
./nrc-smart-build.sh smart

# Or manual compilation
make package/kernel/mac80211/compile V=s
```

### 3. Validation
```bash
# Check module generation
find build_dir -name "nrc.ko" -exec ls -lh {} \;

# Verify functionality
strings path/to/nrc.ko | grep -i "your_feature"
```

## Key Development Areas

### Netlink Interface
```c
// Example: Adding new netlink command
static int nrc_netlink_custom_cmd(struct sk_buff *skb, 
                                  struct genl_info *info) {
    // Command processing logic
    return 0;
}
```

### Vendor IE Processing
```c
// Custom IE handling
static int process_vendor_ie(struct ieee80211_vif *vif,
                           const u8 *ie_data, size_t ie_len) {
    // Vendor-specific processing
    return 0;
}
```

### LED Control Integration
```c
// LED command structure
struct nrc_led_cmd {
    uint8_t cmd_type;
    uint8_t led_data[3];
    uint16_t led_data_len;
};
```

## Debugging Techniques

### Kernel Logging
```c
// Use structured logging
NLOG_INFO("[%s] Operation completed: %d", __func__, result);
NLOG_ERR("[%s] Error condition: %d", __func__, error_code);
```

### Debug Configuration
```bash
# Enable debug output
echo 1 > /sys/kernel/debug/ieee80211/phy0/nrc/debug_level

# Monitor kernel messages
dmesg | grep nrc
```

### Wireshark Analysis
- Capture 802.11ah frames
- Analyze vendor IE content
- Verify protocol compliance

## Testing Procedures

### Unit Testing
1. **Module Loading**: Verify clean insmod/rmmod
2. **Interface Creation**: Test virtual interface setup
3. **Basic Operations**: Association, data transfer
4. **Vendor Features**: LED control, custom IEs

### Integration Testing
1. **Network Stack**: Full 802.11ah protocol testing
2. **User Applications**: RTCM IE, STA Manager compatibility
3. **Performance**: Throughput and range validation
4. **Stability**: Long-duration stress testing

## Performance Optimization

### Memory Management
- Minimize dynamic allocation in fast paths
- Use kernel memory pools for frequent operations
- Implement proper cleanup in error conditions

### Processing Efficiency
- Optimize critical packet processing paths
- Use efficient data structures
- Minimize context switches

### Power Management
- Implement proper sleep/wake cycles
- Optimize beacon processing
- Support advanced power save modes

## Common Issues and Solutions

### Build Issues
- **Silent Failures**: Use verification steps after build
- **C90 Compliance**: Declare variables at block start
- **Header Dependencies**: Include required kernel headers

### Runtime Issues
- **Module Loading**: Check kernel version compatibility
- **Memory Leaks**: Use proper cleanup procedures
- **Performance**: Profile critical code paths

## Best Practices

### Code Quality
1. **Follow Kernel Standards**: Linux kernel coding style
2. **Error Handling**: Check all return values
3. **Memory Safety**: No buffer overflows or leaks
4. **Documentation**: Comment complex algorithms

### Version Control
1. **Atomic Commits**: One feature per commit
2. **Descriptive Messages**: Clear commit descriptions
3. **Branch Management**: Feature branches for development
4. **Code Reviews**: Peer review before merge

### Testing Strategy
1. **Test Early**: Unit test during development
2. **Automate**: Scripted testing procedures
3. **Hardware Validation**: Real device testing
4. **Regression Testing**: Verify existing functionality

## Advanced Topics

### Custom Protocol Extensions
- Implementing new 802.11ah features
- Vendor-specific optimizations
- Backward compatibility considerations

### Performance Profiling
- Kernel profiling tools
- Packet processing analysis
- Memory usage optimization

### Security Considerations
- Input validation
- Buffer overflow prevention
- Privilege escalation protection

## Resources

### Documentation
- Linux Wireless Developer Documentation
- 802.11ah Standard Specifications
- OpenWrt Kernel Module Development Guide

### Tools
- Wireshark with 802.11ah support
- Kernel debugging tools
- Performance profiling utilities

---
*Development guide version: 2.0*  
*Last updated: 2025-07-31*