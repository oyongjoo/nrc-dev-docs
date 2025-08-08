# NRC Development Documentation

Comprehensive documentation for NRC HaLow (IEEE 802.11ah) development within OpenWrt environment.

## 🚀 Quick Start

This repository contains all essential documentation for developing, building, and deploying NRC HaLow components:

- **Build System**: Complete OpenWrt build guides and troubleshooting
- **NRC Components**: Driver development, RTCM processing, and LED systems  
- **Station Management**: STA tracking and monitoring systems
- **Deployment**: Production deployment guides and CI/CD workflows

## 📁 Repository Structure

### 🔧 Build & Deployment
- [`build-deployment/`](build-deployment/) - Build system guides and troubleshooting
  - [Build Guide](build-deployment/build-guide.md) - Complete OpenWrt HaLow build instructions
  - [Build Troubleshooting](build-deployment/build-troubleshooting.md) - Solutions for common build issues
  - [Build Issue Resolution Report](build-deployment/build-issue-resolution-report.md) - Comprehensive issue analysis
  - [NRC Build Sequence Guide](build-deployment/nrc-build-sequence-guide.md) - Optimal build order for NRC components
  - [NRC Build Test Final Report](build-deployment/nrc-build-test-final-report.md) - Production readiness validation
  - [NRC Driver Development Guide](build-deployment/nrc-driver-development-guide.md) - Kernel driver development workflow

### 🧩 NRC Components
- [`nrc-components/`](nrc-components/) - Core NRC component documentation
  - [NRC RTCM Vendor IE with LED](nrc-components/nrc_rtcm_vendor_ie_with_led.md) - V3.0 LED control system
  - [NRC RTCM IE V2 Complete Guide](nrc-components/nrc-rtcm-ie-v2-complete-guide.md) - SQLite to debugfs migration
  - [NRC Enhanced Bulk RTCM Implementation](nrc-components/nrc-enhanced-bulk-rtcm-implementation.md) - Optimized data transmission
  - [NRC LED Wake Bit Systems](nrc-components/nrc-led-wake-bit-systems.md) - 3-bit LED/WAKE architecture
  - [NRC RTCM Bulk Transmission Implementation](nrc-components/nrc-rtcm-bulk-transmission-implementation.md) - Fragment processing
  - [NRC RTCM Fragmentation Analysis](nrc-components/nrc-rtcm-fragmentation-analysis.md) - Performance optimization
  - [NRC FOTA Analysis](nrc-components/nrc-fota-analysis.md) - Firmware update mechanisms
  - [NRC CLI App Command Reference](nrc-components/nrc-cli-app-command-reference.md) - Command-line interface
  - [NRC ATcmd Reference Guide](nrc-components/nrc-atcmd-reference-guide.md) - AT command documentation

### 📡 Station Management
- [`station-management/`](station-management/) - WiFi station management systems
  - [STA Manager Installation Guide](station-management/sta-manager-installation-guide.md) - Installation and configuration

## 🎯 Key Features

### Production-Ready Components ✅
- **NRC Kernel Module (nrc.ko)**: 285KB driver with LED integration
- **RTCM IE Application**: 44.4KB package with optimized fragment processing  
- **STA Manager System**: Real-time monitoring of 438+ stations
- **LED Control System**: 3-bit LED/WAKE command architecture

### Performance Achievements 📊
- **95% Performance Improvement**: SQLite to debugfs migration
- **Real-time Processing**: Fragment transmission optimization
- **Enterprise Scale**: 438 stations actively monitored
- **Build Optimization**: 98% success rate with proper procedures

### Technology Stack 🛠️
- **Platform**: OpenWrt on MT7621 (MIPS 24Kc)
- **Wireless**: IEEE 802.11ah (HaLow) long-range, low-power
- **Database**: SQLite → debugfs transition for performance
- **Communication**: Netlink sockets, vendor IE processing
- **Services**: ProCD integration, init.d service management

## 📖 Documentation Categories

### For Developers
1. **[Build System](build-deployment/)** - Complete build environment setup
2. **[Driver Development](build-deployment/nrc-driver-development-guide.md)** - Kernel module development
3. **[Component Integration](nrc-components/)** - NRC component interaction
4. **[Performance Optimization](nrc-components/nrc-rtcm-fragmentation-analysis.md)** - System performance tuning

### For System Administrators  
1. **[Installation Guides](station-management/)** - System deployment
2. **[Troubleshooting](build-deployment/build-troubleshooting.md)** - Common issues and solutions
3. **[Production Deployment](build-deployment/nrc-build-test-final-report.md)** - Enterprise deployment validation
4. **[System Monitoring](nrc-components/nrc_rtcm_vendor_ie_with_led.md)** - Real-time system monitoring

### For Architects
1. **[System Architecture](nrc-components/nrc_rtcm_vendor_ie_with_led.md)** - V3.0 system design
2. **[Performance Analysis](nrc-components/nrc-rtcm-fragmentation-analysis.md)** - Bottleneck identification
3. **[Migration Guides](nrc-components/nrc-rtcm-ie-v2-complete-guide.md)** - System evolution paths
4. **[Enhancement Strategies](nrc-components/nrc-enhanced-bulk-rtcm-implementation.md)** - Future improvements

## 🚀 Getting Started

### 1. Build Environment Setup
```bash
# Clone and set up Docker environment
git clone <your-openwrt-halow-repo>
cd openwrt-halow
docker-compose up -d --build

# Enter build environment
docker exec -it openwrt-halow-builder bash
cd /home/builder/workspace/halow-main
```

### 2. Component Build Process
```bash
# Build NRC kernel module
./nrc-smart-build.sh smart

# Build RTCM IE application  
make package/network/utils/nrc_rtcm_ie/compile V=s

# Build STA Manager
make package/network/utils/sta_manager/compile V=s
```

### 3. Deployment Validation
```bash
# Verify build outputs
find build_dir -name "nrc.ko" -exec ls -lh {} \;
ls bin/packages/mipsel_24kc/utils/nrc_*
ls bin/packages/mipsel_24kc/base/sta_manager*
```

## 📈 System Status

### Production Environment
- **Hardware**: MT7621-based OpenWrt systems
- **Network Scale**: 438+ HaLow stations monitored
- **Uptime**: >99% service availability
- **Performance**: Real-time data processing with <1% CPU impact

### Development Status
- **Build System**: ✅ Production Ready
- **NRC Driver**: ✅ Operational (285KB module)
- **RTCM Processing**: ✅ Optimized (95% improvement)
- **STA Management**: ✅ Deployed (438 stations)
- **LED Control**: ✅ 3-bit architecture active

## 🔧 Development Workflow

### Mandatory Development Process
⚠️ **CRITICAL**: Always follow this sequence for ALL code changes:
1. **Code Modification** - Make necessary source changes
2. **Build & Compile** - Verify compilation success
3. **Functional Testing** - Deploy and test on target hardware
4. **Only after successful testing** - Apply changes to upstream

### Build System Rules
- Always verify `.config` contains target packages
- Use enhanced cleanup scripts for cache invalidation
- Follow C90 standard compliance for kernel modules
- Validate all build outputs before deployment

## 📚 Learning Path

### New Developers
1. Start with [Build Guide](build-deployment/build-guide.md)
2. Review [Troubleshooting Guide](build-deployment/build-troubleshooting.md)
3. Study [Driver Development Guide](build-deployment/nrc-driver-development-guide.md)
4. Practice with [NRC Build Sequence](build-deployment/nrc-build-sequence-guide.md)

### Experienced Developers
1. Review [System Architecture](nrc-components/nrc_rtcm_vendor_ie_with_led.md)
2. Analyze [Performance Optimizations](nrc-components/nrc-rtcm-fragmentation-analysis.md)
3. Implement [Enhanced Features](nrc-components/nrc-enhanced-bulk-rtcm-implementation.md)
4. Validate with [Production Testing](build-deployment/nrc-build-test-final-report.md)

## 🤝 Contributing

### Documentation Standards
- Use clear, actionable language
- Include code examples and commands
- Provide troubleshooting sections
- Document all configuration changes

### Code Standards
- Follow Linux kernel coding standards
- Implement proper error handling
- Include comprehensive logging
- Validate all inputs and outputs

## 📞 Support

### Build Issues
- Check [Build Troubleshooting](build-deployment/build-troubleshooting.md)
- Review [Issue Resolution Report](build-deployment/build-issue-resolution-report.md)
- Follow [NRC Build Sequence](build-deployment/nrc-build-sequence-guide.md)

### Component Issues
- Study [Component Documentation](nrc-components/)
- Analyze [Performance Guides](nrc-components/nrc-rtcm-fragmentation-analysis.md)
- Review [System Architecture](nrc-components/nrc_rtcm_vendor_ie_with_led.md)

### Production Issues
- Consult [Final Test Report](build-deployment/nrc-build-test-final-report.md)
- Review [Station Management](station-management/sta-manager-installation-guide.md)
- Check system logs and monitoring data

---

## 📄 License & Acknowledgments

This documentation repository contains technical guides and implementation details for NRC HaLow development. All code examples and configurations are provided for educational and development purposes.

**Last Updated**: January 2025  
**Documentation Version**: 2.0  
**Production Status**: ✅ Active Deployment (438 stations)

For additional support or questions, refer to the specific component documentation or build troubleshooting guides.