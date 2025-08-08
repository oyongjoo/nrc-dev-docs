# OpenWrt HaLow 빌드 가이드

## 개요
이 문서는 Docker 환경에서 OpenWrt HaLow 펌웨어를 빌드하는 완전한 가이드입니다.

## 시스템 요구사항

### 최소 사양
- **OS**: Linux, macOS, Windows (WSL2)
- **CPU**: 4코어 이상 권장
- **메모리**: 8GB 이상 (16GB 권장) 
- **디스크**: 50GB 이상 여유 공간
- **Docker**: 최신 버전
- **Docker Compose**: v2.0 이상

### 권장 사양
- **CPU**: 8코어 이상
- **메모리**: 32GB
- **디스크**: SSD 100GB 이상

## 초기 설정

### 1. 저장소 클론
```bash
git clone <repository-url>
cd openwrt-halow
```

### 2. Claude CLI 설정 (선택사항)
```bash
# 호스트에서 Claude CLI 설정
claude config
```

### 3. Docker 환경 구성
```bash
# 컨테이너 빌드 및 시작
docker-compose up -d --build

# 컨테이너 접속 확인
docker-compose exec openwrt-halow-builder bash
```

## 빌드 과정

### Phase 1: 환경 준비
```bash
# 컨테이너 내부에서 실행
cd /home/builder/workspace/halow-main

# HaLow 빌드 환경 설정
./setup_halow_build.sh
```

### Phase 2: 메뉴 설정
```bash
# 메뉴 설정 (타겟, 패키지 선택)
make menuconfig
```

**주요 설정 항목:**
- **Target System**: Ralink RT288x/RT3xxx
- **Subtarget**: MT7621 based boards  
- **Target Profile**: 사용할 보드에 맞게 선택
- **HaLow 관련 패키지**: kmod-nrc, nrc_rtcm_ie 등

### Phase 3: 의존성 다운로드
```bash
# 필요한 소스 패키지 다운로드
make download -j$(nproc)
```

### Phase 4: 빌드 실행

#### 단계별 빌드 (권장)
```bash
# 1. 호스트 도구 빌드
make tools/install -j$(nproc)

# 2. 크로스 컴파일 툴체인 빌드  
make toolchain/install -j$(nproc)

# 3. 커널 및 나머지 패키지 빌드
make -j$(nproc) V=s
```

#### 전체 빌드 (한번에)
```bash
# 백그라운드에서 전체 빌드
nohup make -j$(nproc) V=s > full_build.log 2>&1 &

# 빌드 진행 상황 모니터링
tail -f full_build.log
```

### Phase 5: 빌드 완료 확인
```bash
# 빌드 결과물 확인
ls -la bin/targets/ramips/mt7621/

# 생성된 이미지 파일들:
# - openwrt-*.bin (펌웨어 이미지)
# - openwrt-*.tar.gz (루트파일시스템)
# - *.ko 파일들 (커널 모듈)
```

## 빌드 결과물

### 주요 출력 파일
- **펌웨어 이미지**: `openwrt-ramips-mt7621-*.sysupgrade.bin`
- **팩토리 이미지**: `openwrt-ramips-mt7621-*.factory.bin`  
- **커널 모듈**: `kmod-*.ipk`
- **사용자 패키지**: `*.ipk`

### 결과물 확인
```bash
# 빌드 결과물을 호스트로 복사
cp bin/targets/ramips/mt7621/* /home/builder/output/

# 호스트에서 확인  
ls -la output/
```

## 빌드 시간 예상

### 하드웨어별 예상 시간
- **고성능 워크스테이션** (16코어, 32GB): 1-2시간
- **일반 개발머신** (8코어, 16GB): 2-4시간  
- **노트북** (4코어, 8GB): 4-8시간

### 빌드 단계별 시간
1. **tools/install**: 10-30분
2. **toolchain/install**: 20-60분
3. **커널 및 패키지**: 30분-3시간

## 성능 최적화 팁

### 빌드 속도 향상
```bash
# ccache 활성화 (menuconfig에서 설정)
Advanced configuration options > 
  Enable compiler cache (ccache)

# 병렬 빌드 최적화
make -j$(( $(nproc) + 1 ))
```

### 디스크 I/O 최적화
- SSD 사용 권장
- 빌드 디렉토리를 tmpfs에 마운트 (충분한 RAM 필요)

### 네트워크 최적화  
- 빠른 인터넷 연결 (다운로드 단계)
- 로컬 미러 설정 고려

## 부분 빌드

### 특정 패키지만 빌드
```bash
# 커널 모듈만 재빌드
make package/kernel/linux/compile -j$(nproc)

# NRC RTCM IE 패키지 재빌드 (v2.0 - debugfs 기반)
make package/network/utils/nrc_rtcm_ie/compile -j$(nproc)

# HaLow 드라이버 재빌드
make package/kernel/mac80211/compile -j$(nproc)
```

### 클린 빌드 옵션
```bash
# 특정 패키지 클린
make package/network/utils/nrc_rtcm_ie/clean

# 전체 클린 (설정 유지)
make clean

# 완전 클린 (설정 삭제)
make distclean
```

## 문제 해결

빌드 중 문제가 발생하면:

1. **로그 확인**: `full_build.log` 또는 터미널 출력
2. **에러 검색**: `grep -i "error\|failed" full_build.log`
3. **트러블슈팅 가이드**: `docs/build-troubleshooting.md` 참조
4. **클린 후 재시도**: 해당 컴포넌트 클린 후 재빌드

## 다음 단계

빌드 완료 후:
- [배포 가이드](openwrt-halow-deployment-guide.md) 참조
- 테스트 환경에서 펌웨어 검증
- 실제 하드웨어에 플래싱

---
*최종 업데이트: 2025-07-22*