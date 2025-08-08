# Daily Build Maintenance Guide

## Overview
OpenWrt HaLow 빌드 시스템의 일상적인 유지보수 및 문제 해결 가이드입니다.

## Daily Operations

### 1. Container Status Check
```bash
# Docker 컨테이너 상태 확인
docker ps -a
docker-compose ps

# 컨테이너 재시작 (필요시)
docker-compose up -d --build
```

### 2. Build Environment Verification
```bash
# 컨테이너 내부 접속
docker-compose exec openwrt-halow-builder bash

# 작업 디렉토리 이동
cd /home/builder/workspace/halow-main

# 빌드 환경 상태 확인
ls -la build_logs/ | tail -10
```

### 3. Package Installation Verification
```bash
# 타겟 디바이스에서 설치된 패키지 확인
opkg list-installed | grep -E "(sqlite|sta|nrc|hostapd)"

# 특정 패키지 상세 정보
opkg info <package-name>

# 실행 파일 위치 확인
which sqlite3
ls -la /usr/bin/*sta*
```

## Common Issues and Solutions

### Issue 1: SQLite3/sta_manager Missing from Firmware

**증상:**
- 펌웨어 업데이트 후 SQLite3 명령어 사용 불가
- sta_manager 바이너리 누락

**원인:**
- .config에 설정되어 있지만 target/install 단계에서 누락
- 패키지가 이미지에 포함되지 않음

**해결방법:**
```bash
# 1. 컨테이너 내부에서 빌드 설정 확인
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && grep -i sqlite .config"
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && grep -i sta .config"

# 2. target/install 강제 재빌드 (verbose 모드)
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && make target/install -j1 V=s | tee build_logs/target_install_verbose_$(date +%Y%m%d_%H%M%S).log"

# 3. 펌웨어 및 패키지 파일 복사
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && cp bin/targets/ramips/mt7621/*.bin /home/builder/output/"
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && find bin/packages/mipsel_24kc/ -name '*sqlite3*' -o -name '*sta-manager*' | xargs -I {} cp {} /home/builder/output/"
```

### Issue 2: Container Offline During Build

**증상:**
- 빌드 중 컨테이너가 중단됨
- Claude CLI가 offline 상태로 표시

**해결방법:**
```bash
# 1. 컨테이너 재시작
docker-compose up -d --build

# 2. 기존 빌드 상태 확인
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && ls -la bin/targets/ramips/mt7621/"

# 3. 빌드 로그 확인으로 중단 지점 파악
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && tail -50 build_logs/*.log"
```

## Build Verification Checklist

### 1. Required Packages in .config
```bash
CONFIG_PACKAGE_libsqlite3=y
CONFIG_PACKAGE_sqlite3-cli=y
CONFIG_PACKAGE_sta-manager=y
CONFIG_PACKAGE_sta-query=y
CONFIG_PACKAGE_nrc_rtcm_ie=y
CONFIG_PACKAGE_hostapd-common=y
CONFIG_PACKAGE_hostapd-utils=y
```

### 2. Output Files Verification
```bash
# 펌웨어 이미지 확인
ls -la output/*.bin

# 패키지 파일 확인
ls -la output/ | grep -E "(sqlite3|sta|nrc)"
```

### 3. Target Device Verification
```bash
# 디바이스에서 패키지 확인
opkg list-installed | grep -E "(sqlite|sta|nrc)"

# 실행 파일 테스트
sqlite3 --version
sta-manager --help
nrc_rtcm_ie --version
```

## Build Commands Reference

### Full Build Process
```bash
# 전체 빌드 (백그라운드 실행)
make -j$(nproc) world > build_logs/world_build_$(date +%Y%m%d_%H%M%S).log 2>&1 &

# target/install 단독 실행
make target/install > build_logs/target_install_$(date +%Y%m%d_%H%M%S).log 2>&1 &
```

### Monitoring Commands
```bash
# 빌드 진행 상황 모니터링
tail -f build_logs/world_build_*.log
tail -f build_logs/target_install_*.log

# 백그라운드 작업 확인
jobs
```

### Package-specific Builds
```bash
# SQLite3 재빌드
make package/sqlite3/clean
make package/sqlite3/compile -j1 V=s

# sta_manager 재빌드
make package/sta-manager/clean
make package/sta-manager/compile -j1 V=s

# nrc_rtcm_ie 재빌드
make package/nrc_rtcm_ie/clean
make package/nrc_rtcm_ie/compile -j1 V=s
```

## File Transfer to Target Device

### Firmware Update
```bash
# sysupgrade 펌웨어 전송
cat output/openwrt-23.05-snapshot-unknown-ramips-mt7621-h1radio_11ah-squashfs-sysupgrade.bin | ssh root@192.168.10.98 "cat > /tmp/sysupgrade.bin"

# 펌웨어 업데이트 실행 (디바이스에서)
sysupgrade /tmp/sysupgrade.bin
```

### Package Installation
```bash
# 개별 패키지 전송
for file in output/*.ipk; do 
    cat "$file" | ssh root@192.168.10.98 "cat > /tmp/$(basename $file)"
done

# 패키지 설치 (디바이스에서)
opkg install /tmp/sqlite3-cli_*.ipk
opkg install /tmp/sta-manager_*.ipk
```

## Troubleshooting Logs

### Log File Locations
- Build logs: `workspace/halow-main/build_logs/`
- System logs: `docker-compose logs openwrt-halow-builder`
- Build output: `workspace/halow-main/bin/`

### Error Analysis
```bash
# 빌드 에러 검색
grep -i "error\|failed" build_logs/*.log

# 패키지별 에러 확인
grep -A5 -B5 "sqlite3\|sta-manager" build_logs/*.log
```

## Daily Maintenance Schedule

### Morning Check (9:00 AM)
1. Container status verification
2. Previous night build results check
3. Output directory file verification

### Midday Check (1:00 PM)
1. Target device connectivity test
2. Package installation verification
3. Service status check

### Evening Check (6:00 PM)
1. Build logs review
2. Disk space check
3. Backup verification

---

## Work Log

### Work Log - 2025-07-23 09:30
**Issue:** SQLite3와 sta_manager 패키지가 펌웨어 이미지에 포함되지 않음
**Solution:** 
1. .config 설정 확인 (이미 올바르게 설정됨)
2. `make target/install -j1 V=s` 명령으로 verbose 모드 재빌드
3. 펌웨어 및 패키지 파일을 output 디렉토리로 복사

**Files Modified:**
- 펌웨어 이미지: `openwrt-23.05-snapshot-unknown-ramips-mt7621-h1radio_11ah-squashfs-sysupgrade.bin`
- 패키지: `sqlite3-cli_*.ipk`, `sta-manager_*.ipk`, `libsqlite3-0_*.ipk`

**Result:** 
- 새로운 펌웨어 이미지 생성 완료
- SQLite3 및 sta_manager 패키지가 output 디렉토리에 준비됨
- 타겟 디바이스 업데이트 후 정상 작동 예상

**Commands Used:**
```bash
docker-compose up -d --build
docker-compose exec -T openwrt-halow-builder bash -c "cd /home/builder/workspace/halow-main && make target/install -j1 V=s | tee build_logs/target_install_verbose_$(date +%Y%m%d_%H%M%S).log"
```

---

## Notes
- 모든 빌드 작업은 Docker 컨테이너 내부에서 수행
- 장시간 빌드 작업은 백그라운드에서 실행하고 로그 모니터링
- 패키지 누락 시 target/install 재실행으로 해결 가능
- 정기적인 컨테이너 재시작으로 안정성 확보

Last Updated: 2025-07-23